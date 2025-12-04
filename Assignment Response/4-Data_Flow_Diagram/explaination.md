**Deliverable 4 : Data Flow diagram** :

I have opted to draw the data flow diagram of inbound and outbound call flow, provided in the data flow.jpg

**Inbound Call Flow Explaination** : 

1. Data Captured at Each Step:
- Caller → raw audio
- Media Plane → processed PCM audio
- ASR → partial transcripts
- Intent Engine → intent labels, confidence scores
- LLM → response text
- TTS → generated audio
- CRM Sync → call outcome, disposition

2. Where Data is Stored:
- Recordings → S3
- Transcripts → Kafka → Snowflake
- Metrics → Prometheus
- Vectors (RAG) → Qdrant
- Session State → Redis

3. Real-Time Steps:
- Audio processing, ASR, transcripts, intent detection, LLM response, TTS playback

4. Batch Steps:
- Call recordings upload, long-form transcription, analytics, data warehouse updates

5. Failure / Retry Points:
- ASR timeout → retry → fallback to Google STT
- LLM error → retry → fallback to Mixtral → fallback to static templates
- TTS failure → fallback Google TTS → pre-recorded audio
- Kafka lag → retry consumer
- CRM webhook failure → retry via queue

6. State Maintenance:
- Session state maintained in Redis (call_id, last transcript, last intent, session context)
- State machine tracks conversation stage

7. Event Triggers / Webhooks:
- Twilio webhook on call start
- Kafka event for transcript
- Kafka event for intent
- CRM webhook on call completion
- S3 event when recording uploaded
