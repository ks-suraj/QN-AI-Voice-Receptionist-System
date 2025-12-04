**Deliverable 4 : Data Flow diagram** :

I have opted to draw the data flow diagram of inbound and outbound call flow, provided in the data flow.jpg

For all the nodes where analytics is used it is better to either use snowflake (reliable) or clickhouse (self hosted burden)

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


**Outbound Campaign (Call) Flow Explaination** : 


1. Data Captured at Each Step:
- Lead Data → lead_id, phone number, metadata
- Dialing → call status events
- Conversation → transcripts, intents, replies
- Disposition → outcome codes
- Analytics → call duration, sentiment, resolution

2. Where Data is Stored:
- Lead metadata → PostgreSQL / CRM
- Transcripts → Snowflake
- Vectors → Qdrant
- Call events → Kafka
- Session state → Redis
- Recordings → S3


3.Real-Time Steps:
- Dialing, media streaming, ASR, intent detection, LLM response, TTS playback

4. Batch Steps:
- Campaign reporting
- Lead scoring updates
- Daily analytics ETL

5. Failure / Retry Points:
- Call not answered → automatic retry rules
- ASR/LLM failure → fallback templates
- CRM API failure → queued retry
- Dialer congestion → rate limiting

6. State Maintenance:
- Campaign service keeps dialing state
- Redis keeps per-call session state
- State machine manages calling workflow (intro, qualification, closing)

7. Event Triggers / Webhooks:
- Campaign start trigger
- Twilio call status events
- Kafka events for transcripts and dispositions
- CRM webhook for updating lead score or stage
