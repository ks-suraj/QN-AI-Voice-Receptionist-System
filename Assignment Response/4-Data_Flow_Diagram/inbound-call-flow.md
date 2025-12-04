Caller (PSTN)
      |
      v
Telephony Provider (Twilio SIP)
      |
      v
SIP Gateway / Load Balancer
      |
      v
Media Plane (WebRTC SFU: AEC, VAD, Noise Suppression)
      |
      v
PCM Audio Frames
      |
      v
Streaming ASR (Deepgram / Google)
      |
      v
Partial Transcripts
      |
      v
Kafka Topic: transcripts_realtime
      |
      v
Intent Engine (Llama 3 classifier)
      |
      |----> If deterministic intent ----> State Machine ----> Template Response
      |
      |----> If complex intent ----------> RAG (Qdrant) ----> LLM (GPT-4o) ----> Response Filter
      |
      v
TTS (Amazon Polly)
      |
      v
Audio Playback (SFU)
      |
      v
Call Resolution
      |
      v
CRM Sync + Snowflake Analytics
