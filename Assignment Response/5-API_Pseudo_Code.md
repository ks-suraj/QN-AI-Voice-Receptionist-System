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
