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
- Telephony: Twilio SIP ; incoming call webhook, call events
- Media plane / SFU: mediasoup/janus ; audio streaming, playback, recording
- Streaming ASR: Deepgram (fallback for retries) ; streaming and batch transcription
- Intent model: local quantized Llama 3 (fast) ; for intent classification
- RAG: Qdrant + Vector retriever ; vector retrieval for RAG
- Response LLM: GPT-4o via API (fallback Mixtral, then local Llama) ; high-quality response generation and personalization
- TTS: Amazon Polly (fallback Google TTS, then prerecorded) ; audio synthesis
- Queue: Kafka ; events for transcripts, recordings, dispositions
- Session store / cache: Redis ; session storage and idempotency flags
- Object storage: S3 ; store recordings and cached TTS
- Btch workers: Celery / K8s Jobs ;for job scheduling
- Data warehouse: Snowflake ; nalytics and reporting



**Pseudo Code** : 
1. Main real-time call loop :
```
# handling one incoming call from start to finish for better streamline (prefered)
def handle_inbound_call(call_id, caller_number):
    # creating or restoring session in Redis
    session = redis_get_or_create_session(call_id)
    session["state"] = "connected"
    session["context"] = []      # short list of last few turns
    session["last_intent"] = None

    # subscribe to audio chunks from the media server
    for pcm_chunk in media_server.stream_audio(call_id):
        # skipping periods of silence to save work
        if not voice_activity_detected(pcm_chunk):
            continue

        # trying to get text for this audio chunk from ASR
        transcript = asr_stream_transcribe(pcm_chunk, call_id)
        if not transcript:
            # if ASR fails for this chunk, log and continue
            log_event("asr_chunk_failed", call_id)
            continue

        # publishing transcript to analytics stream (non-blocking)
        kafka_publish("transcripts_realtime", {"call_id": call_id, "text": transcript})

        # better to add user text to short context
        session["context"].append({"role": "user", "text": transcript})
        redis_save_session(call_id, session)

        # quick intent detection using a small local model (llama)
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
            # otherwise build a grounded reply using retrieval + LLM (GPT)
            response = build_and_generate_response(intent, session, transcript)

        # run safety checks and add small personal touches (personalization addition)
        response = apply_safety_and_personalization(response, session)

        # convert final text to audio and play back
        play_text(call_id, response)

        # save assistant reply in session context
        session["context"].append({"role": "assistant", "text": response})
        redis_save_session(call_id, session)

    # after stream ends (caller hung up)
    finalize_call(call_id, session)
```
Why this : 
- Keeps a short memory of the call in Redis.
- Listens to audio, skips silence to save work.
- Converts audio to text chunk-by-chunk.
- Does a fast intent check; if intent is simple, follow a deterministic flow; otherwise call the LLM with retrieved context.
- Checks the reply for safety, personalizes it, then plays it back.
- When the call ends, it runs final steps like upload and analytics.

2. ASR wrapper (including retries and fallback) :
```
def asr_stream_transcribe(pcm_chunk, call_id, retries=2):
    # trying primary ASR first
    for attempt in range(retries):
        text = call_deepgram_stream(pcm_chunk, call_id)
        if text:
            return text
        sleep(100 * (attempt + 1))  # small backoff

    # if primary failed, trying a fallback provider
    text = call_google_stt_stream(pcm_chunk, call_id)
    return text  # may be None if fallback also failed
```
Why this : 
- First tries the main ASR. If it fails, waits a little and tries again.
- If still failing, tries one fallback provider.
- If both fail for this chunk, the loop continues; we don't abort the whole call.

3. Fast intent detection (simple classifier)
```
def detect_intent_fast(text):
    try:
        result = local_intent_model.predict(text)
        return {"name": result.intent, "confidence": result.score} #score helps in evaluation
    except Exception:
        # if local model errors, return unknown with low confidence
        return {"name": "unknown", "confidence": 0.0}
```
Why this : 
- Uses a small model kept nearby to get very fast intent results.
- If that model fails, the system treats the intent as unknown so it can ask the user to repeat or fall back.

4. Deterministic flows (state machine (only for simple actions)) :
```
def run_deterministic_flow(intent_name, session):
    # handle very specific tasks without calling LLM
    if intent_name == "check_balance":
        balance = crm_get_balance(session.get("customer_id"))
        return f"Your current balance is {balance} rupees."

    if intent_name == "pay_bill":
        # ask for a confirmation step that is deterministic
        if session.get("state") != "awaiting_payment_confirm":
            session["state"] = "awaiting_payment_confirm"
            redis_save_session(session["call_id"], session)
            return "Do you want to pay the outstanding amount now? Please say yes to confirm."
        else:
            # caller confirmed, call billing API
            success = billing_api.pay(session.get("customer_id"))
            if success:
                return "Payment successful. Anything else I can help with?"
            else:
                return "Payment failed. I will connect you to an agent."

    # fallback
    return "I will connect you to an agent for that request."
```

Why this : 
- For tasks we can complete with a few rules (like checking the balance or making a payment), we use this simple flow.
- The state is kept in the session so the bot can do multi-step tasks

5. Building a response with retrival (LLM and fallbacks) :
```
def build_and_generate_response(intent, session, recent_text):
    # get relevant documents from vector store
    docs = qdrant_search(recent_text, top_k=5)

    # build prompt using recent context and retrieved docs
    prompt = compose_prompt(session["context"], docs, recent_text, intent["name"])

    # try the primary cloud LLM
    try:
        answer = gpt4o_generate(prompt)
    except TimeoutError:
        # if cloud LLM times out, try a mid-tier model
        try:
            answer = mixtral_generate(prompt)
        except Exception:
            # last resort: local short-answer model or template
            answer = local_llama_short_generate(prompt)

    # quickly check for obvious factual errors or policy triggers
    if not passes_basic_checks(answer, docs):
        return "I am not confident in that answer. Let me connect you to an agent."

    return answer
```
Why this : 
- We first fetch helpful documents to ground the LLM.
- We call the high-quality model. If it fails, we try a cheaper fallback, and finally use a local model or template.
- If the result looks unsafe or made-up, we refuse and escalate.

6. TTS with caching and fallback :
```
def play_text(call_id, text):
    # create a cache key for this text
    key = "tts_cache:" + simple_hash(text)

    # check cached audio first
    s3_key = redis_get(key)
    if s3_key:
        audio = s3_download(s3_key)
        media_server.play_audio(call_id, audio)
        return

    # try generating audio with primary provider
    audio = None
    try:
        audio = amazon_polly_synthesize(text)
    except Exception:
        try:
            audio = google_tts_synthesize(text)
        except Exception:
            audio = load_local_prerecorded("fallback_message.wav")

    # save audio to S3 and cache pointer
    s3_key = s3_upload(audio, f"tts/{key}.mp3")
    redis_set(key, s3_key, ttl=86400)

    media_server.play_audio(call_id, audio)
```
Why this :
- Reuses previously generated audio for the same text to save cost and time.
- If the main TTS fails, tries a fallback; finally uses a recorded message.

7. Finalizing the call and initiating batch work :
```
def finalize_call(call_id, session):
    # ask media server to close and upload the recording
    s3_recording_key = media_server.finalize_and_upload(call_id)

    # notify the batch pipeline to transcribe the full recording
    if s3_recording_key:
        kafka_publish("recordings_completed", {"call_id": call_id, "s3_key": s3_recording_key})

    # save call summary and disposition (idempotent)
    op_id = f"crm_update:{call_id}"
    write_once(op_id, lambda: crm_update_call_log(session))

    # remove session from Redis
    redis_delete(f"session:{call_id}")
```
Why this : 
- Uploads the full call recording to S3.
- Posts an event so batch workers will do full transcription and analytics.
- Updates CRM once and only once, even if the function is retried.

8. Idempotency helpers (safe area to retry) :
```
def write_once(op_id, action):
    # try to reserve this operation
    reserved = redis_setnx(op_id, "in-progress", ttl=3600)
    if not reserved:
        # someone else already did this operation
        return "already_done"

    try:
        action()
        # mark as finished
        redis_set(op_id, "done", ttl=86400)
    except Exception:
        # clear reservation so retry can happen later
        redis_delete(op_id)
        raise
```
Why this :
- Ensures critical write operations (like updating CRM or billing) only happen once, even if requests are retried.
- Simple pattern: attempt to reserve, run action, mark done.

9. Handling partial failures
```
# policy summary in code form

if asr_primary_unhealthy():
    for i in 3 retries :
        restart_asr_stream()
        log_event("asr_failover")

if lmm_primary_unhealthy():
    enable_local_llama_mode()   # degrade gracefully
    show_message_to_user("We are experiencing a temporary issue. We may call you back.")

if tts_primary_unhealthy():
    use_prerecorded_prompts_for_now()
if CRM_unhealthy :
    queue_tasks()
    kafka_retry_policy() && task_pending == True
if Qdrant_unhealthy :
    switch_to_NO_RAG(Llama_call && RAG == False)
```
Why this : 
- Detect when external services become unhealthy and flip to a lower-quality but available option.
- Tell the user politely if needed, and queue any offline work for later.




10. Third party failure handling : 
- If ASR provider is down: give it 3 retries and ask the user to repeat if not understood, worst case scenario switch it to google STT.
- If LLM provider is down: route to lower-quality local Llama for short templates or use deterministic state machine. Mark call for QA review.
- If TTS provider is down: use alternative TTS or prerecorded audio. For prolonged outage, switch to SMS follow-up or schedule callback.
- If CRM is down: queue CRM writes in Kafka with retry policy, tag events as pending; notify ops.
- If Qdrant (RAG) is down: degrade to no-RAG mode (LLM allowed but higher hallucination risk) and limit actions; escalate to human for sensitive queries.

11. How to ensure idempotency (safe to retry)
- All external-write operations (CRM updates, billing API calls, database writes) use operation_id and write_once semantics (Redis setnx + persistent marker).
- Use unique request ids for LLM calls if they trigger side effects.
- For outbound side-effects (billing), apply two-phase commit / saga pattern:
- Reserve -> confirm -> commit. Each step is idempotent.
- Use Kafka with exactly-once semantics where possible; consumers should be idempotent.

12. Handling partial failures
- Keep session state in Redis. If a transient worker fails, another worker can resume using session state.
- Use short-lived checkpoints: after each assistant reply persist session snapshot to Redis.
- For long-running operations (file uploads, embeddings), use job states (pending -> processing -> completed -> failed) stored in Postgres and requeued on failure.
- Alerts: if retries > threshold, create incident ticket and tag call for human review.

13. API versioning and schema :
- external as well as few third party apis go with /v1/calls , /v2/calls
- kafka gives out schema_version
- keep a flag check list for any roll outs
- also best practice to keep older apis running while new pop up

14. Monitoring(thinks to keep an eye on) :
- ASR latency
- LLM response time and number of time it fallbacks
- kafka logs
- Daily cost of ASR, TTS and LLM
