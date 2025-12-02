# How it works
1. Call arrives - When the call arrives the PSTN/SIP provider recieves it and forwards a SIP invitation to the telephony gateway
2. Call session created - Once the SIP invitation is accepted, the LOAD balancer gives a raincheck on load or number of session being handled and the SIP gateway routes the call to media plane {SFU(Janus / Mediasoup)} on Kubernetes, where the media plane handles the RTP and session setup.
3. Audio pre-processing - Media workers ensure echo cancel using AEC, noise suppression, voice activity detection (VAD) and converts the audio to PCM chunks or packets.
4. Streaming - PCM is streamed to low latency ASR (Whisper/Google), to ensure streaming the the partial transcripts are sent or emitted in real-time.
5. Intent Detection - these transcripts are pushed to event schedule (Queues) (using Kafka), an intent engine (NLU) determinse the user intent.
6. Decision & Response generation - Since using kafka, once the intent is configured, the event trigger Response Generation via a LLM (INtelligence Layer) + RAG (Vector DB (consisting details and data)) and produces a reply, this response is checked through hallucianation triggers, saftety, compilance and a personalization layer
7. TTS & Playback - The approved response is coverted from text to audio via TTS tools (ElevenLabs) and also connected to media plane for playbacks to the caller.
8. Async recording and analytics - The entire call recordings are saved in S3 Buckets, converted into vectors for RAG and pushed into data warehouse (Snowflake) for future analytics, also they are integrated to CRM which has read access via webhooks and API connectors.


# Critical Data Paths
RTP → Media Router → Preprocessing → Streaming ASR → Intent Engine → LLM (fast) → TTS → Caller
this specific path handles the latency on how quick does the caller get a reply
- to solve this we are using streaming ASR/TTS rather than batch models

# Bottlenecks and mitigation
ASR congestion
- as it is CPU/GPU heavy, handling multiple calls can create congestion
- using VAD helps us discard noise and background before sending it to ASR, hence less data but more accurate data packets



# Scaling -
- LLMs vs SLMs - LLMs are huge to play with whereas SLMs specifically trained or quantized LLMs can do the processing and response generation with very less latency and quick response
- Google ADK - Google ADK has inbuilt voice agent generation which not only benefits with higher model leverage but also larger context window
