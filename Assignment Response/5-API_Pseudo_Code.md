**Deliverable 5 : Pseudo code(Only logic)**

As far as i have the domain expertised and the research i did for our current project, the pseudo logic that i would be covering would contain : 
- Core processing loop (real-time call handling)
- Critical decision logic (how actions are chosen)
- Error handling and fallbacks
- Integration points (Twilio, SFU, ASR, Kafka, LLM, TTS, CRM, S3, Qdrant, Snowflake)
- Idempotency (safe to retry)
- Partial-failure handling
- What to do if a third-party is down
- API versioning for backward compatibility

Also assuming w.r.t the technologies i have opted for : 
- Telephony: Twilio SIP
- Media plane / SFU: mediasoup/janus
- Streaming ASR: Deepgram (fallback Google STT)
- Intent model: local quantized Llama 3 (fast)
- RAG: Qdrant + Vector retriever
- Response LLM: GPT-4o via API (fallback Mixtral, then local Llama)
- TTS: Amazon Polly (fallback Google TTS, then prerecorded)
- Queue: Kafka
- Session store / cache: Redis
- Object storage: S3
- Btch workers: Celery / K8s Jobs
- Data warehouse: Snowflake



**Pseudo Code** : 
- Main real-time call loop :
```
# handle one incoming call from start to finish
def handle_inbound_call(call_id, caller_number):
    # create or restore session in Redis
    session = redis_get_or_create_session(call_id)
    session["state"] = "connected"
    session["context"] = []      # short list of last few turns
    session["last_intent"] = None

    # subscribe to audio chunks from the media server
    for pcm_chunk in media_server.stream_audio(call_id):
        # skip periods of silence to save work
        if not voice_activity_detected(pcm_chunk):
            continue

        # try to get text for this audio chunk from ASR
        transcript = asr_stream_transcribe(pcm_chunk, call_id)
        if not transcript:
            # if ASR fails for this chunk, log and continue
            log_event("asr_chunk_failed", call_id)
            continue

        # publish transcript to analytics stream (non-blocking)
        kafka_publish("transcripts_realtime", {"call_id": call_id, "text": transcript})

        # add user text to short context
        session["context"].append({"role": "user", "text": transcript})
        redis_save_session(call_id, session)

        # quick intent detection using a small local model
        intent = detect_intent_fast(transcript)

        # if the intent is unclear, ask user to repeat
        if intent["confidence"] < 0.5:
            play_text(call_id, "Sorry, I didn't catch that. Please say that again.")
            continue

        session["last_intent"] = intent["name"]

        # if this intent maps to a deterministic flow, use state machine
        if is_simple_action(intent["name"]):
            response = run_deterministic_flow(intent["name"], session)
        else:
            # otherwise build a grounded reply using retrieval + LLM
            response = build_and_generate_response(intent, session, transcript)

        # run safety checks and add small personal touches
        response = apply_safety_and_personalization(response, session)

        # convert final text to audio and play back
        play_text(call_id, response)

        # save assistant reply in session context
        session["context"].append({"role": "assistant", "text": response})
        redis_save_session(call_id, session)

    # after stream ends (caller hung up)
    finalize_call(call_id, session)
```
