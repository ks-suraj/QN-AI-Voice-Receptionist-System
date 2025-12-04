# Deliverable 7 : Challenges and its mitigation stratergies

This section is where i actually used AI, i had not much of a clarity where the actual challenges might pop up, but based on my experience and the intent i had put on desgining i felt challenges would pop majorly on latency, audio packets, job scheduling, inconsistent response (user response), noise, third party, and other i figured and understood from AI.

I. Technical Challenges : 
1. Challenge: LLM hallucinations or incorrect responses
- Why it matters: The system may give wrong information to customers, hurting trust and compliance.
- Risk Level: High
- Mitigation: Use RAG grounding, strict response filters, short prompts, fallback templates for sensitive topics, and route low-confidence answers to human agents.

2. Challenge: Real-time latency (ASR → Intent → LLM → TTS chain)
- Why it matters: If replies take longer than 600–800 ms, callers feel the bot is slow or broken.
- Risk Level: High
- Mitigation: Use streaming ASR and TTS, lightweight intent model, short LLM prompts, local caching, and ensure SFU/audio pipeline is optimized.

3. Challenge: ASR accuracy in noisy or accented environments
- Why it matters: Misheard input leads to wrong intents and confusion in conversations.
- Risk Level: Medium
- Mitigation: Use AEC/VAD in media plane, Deepgram’s telephony models, fallback ASR provider, and threshold-based clarification prompts.

4. Challenge: Third-party API downtime (ASR, LLM, TTS, CRM)
- Why it matters: If any provider goes down, the call flow breaks.
- Risk Level: High
- Mitigation: Use fallback models (ASR fallback, LLM fallback, TTS fallback), offline templates, cached responses, and circuit-breaker logic.

5. Challenge: Scaling under high call volume
- Why it matters: System may drop calls or lag under peak load.
- Risk Level: Medium
- Mitigation: Autoscaling K8s, load testing, Kafka backpressure monitoring, separate real-time vs batch workloads, and isolate media services from LLM services.

6. Challenge: Recording storage and data handling at scale
- Why it matters: Thousands of calls generate large audio files and transcripts, which can overwhelm storage and pipelines.
- Risk Level: Medium
- Mitigation: Lifecycle policies on S3, compressed formats, scheduled ETL jobs, partitioned Snowflake tables, and proper indexing for Qdrant.

7. Challenge: Cost overruns due to ASR/LLM/TTS usage
- Why it matters: Per-minute and per-token costs can grow quickly with volume.
- Risk Level: High
- Mitigation: VAD to reduce ASR minutes, caching common TTS responses, LLM routing (local → mid-tier → premium), token limits, and monthly cost monitoring dashboards.

8. Challenge: Privacy & compliance (GDPR, CCPA, TCPA for calls)
- Why it matters: Wrong handling of recordings or personal information can cause legal penalties.
- Risk Level: Critical
- Mitigation: Encrypt data at rest/in transit, minimize PII storage, auto-delete after retention period, obtain consent in IVR, and maintain audit logs.

  
II. Business Challenges ( used AI) : 
1. Challenge: Measuring ROI and real effectiveness of the system
- Why it matters: Without clear ROI, leadership may not continue investing.
- Risk Level: Medium
- Mitigation: Track metrics such as call deflection rate, successful resolutions, reduced agent minutes, cost-per-call, and compare against baseline human operations.

2. Challenge: Customer satisfaction and call quality perception
- Why it matters: Even if the system works technically, poor tone or slow responses lower satisfaction.
- Risk Level: Medium
- Mitigation: Use natural voices (e.g., Polly Neural or ElevenLabs), smooth prompt design, short responses, sentiment monitoring, and escalate frustrated callers.

3. Challenge: Competitive feature pressure
- Why it matters: Stakeholders may request rapid new features because other AI voice systems appear more advanced.
- Risk Level: Medium
- Mitigation: Maintain clear product roadmap, implement plugin-based architecture for adding skills, and prioritize only high-impact features.

4. Challenge: Team skills and hiring for voice + LLM + telephony stack
- Why it matters: Voice systems require specialized knowledge (SIP, WebRTC, ASR, LLM orchestration). Hard to hire for.
- Risk Level: Low–Medium
- Mitigation: Document architecture well, create internal training guides, modularize components so general engineers can contribute, and rely on managed services for telephony/ASR/TTS when possible.
