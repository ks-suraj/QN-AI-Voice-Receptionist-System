| Component Category             | **My Choice Choice**                                       | **Why This?**                                                                                                                                               | **Trade-offs**                                                                         |
| ------------------------------ | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Backend Runtime**            | **Python (FastAPI microservices)**                    | AI ecosystem, multiple ML,AI,Automation libraries, supports async programming, easy and faster development, personal experience  | lower throughput compared to Go/Rust, yet acceptable as AI ecosystem would dominate |
| **Web Framework**              | **FastAPI**                                           | Aync I/O availability, supports pydantic, developer friendly and faster, easier and adaptable                 | a sacrifice on latency w.r.t Go servers yet faster developer ecosystem                            |
| **Primary Database**           | **PostgreSQL**                                        | Best for production, ACID properties, accepts relational query, extensions availability                                                | not good for large document searching, but since we have vector dbs, we can offload there.                       |
| **Cache Layer**                | **Redis**                                             | very low latency, session store option, good for streaming processes                                     | Careful and detailed planning, should have a backup scenari on ready               |
| **Message Queue**              | **Kafka**                                             | High throughput, durable, best for event-driven pipelines (transcripts, intents, dispositions).             | Operationally complexity and managment costs but helps for throughput.                     |
| **Vector / Search DB**         | **Qdrant**                              | Cost efficient, heavy data and workloads and Open source.                                                                | Not simple as Pinecone and not hybrid search as Weaviate but performance wins for Qdrant                       |
| **Job Scheduler**              | **Celery + Redis / Kubernetes CronJobs**              | asynchronous task execution, analytics ETL, scheduling enable and retries.                                                                     |needs heavy monitoring extra moving nodes.                         |
| **Monitoring & Logging**       | **Prometheus + Grafana + ELK (ElasticSearch / Loki)** | SIP metrics, ASR latency, LLM latency, queue lag, TTS error, log search, traces.                                                                       | Operational overhead, storage-heavy, complexity but good for heavy production.                              |
| **Deployment & Orchestration** | **Kubernetes (EKS/GKE)**                              |auto-scaling media servers, ASR workers, LLM inference microservices, stateless APIs.          | Harder DevOps maturity, costly at initially stage but better for production.                                       |
| **Telephony Provider** | **Twilio SIP + Voice API**                              |Global PSTN , webhook, low friction, real time         | Higher per call cost, chances of vendor lock in, but reliable and higher market reach                                       |
| **Real-Time protocol** | **WebRTC(for media) + WebSockets(for signlaing)**                              |Jitter buffering, secure media, SFU compatability         |More Infra and more complex but better for grade audio                                       |
| **Dialogue Framework** | **Hybrid : State Machine + LLM Agent**                              |Determinism (state machine), intelligent layer (LLM), escalation flow        |Higher desing complexity, Higher computational better for multi approach, might have hallucinations but reliability and naturalnaess factor would dominate                                       |




**Telephony Provider** - 
After assessoing the use case, it would be better to go with  - #Twilio SIP Trunks + Voice API
Coming to WHY :
 - Twilio is a global provider, especially reliable for PSTN -> SIP connectvity
 - has built in call routing, recording, phone number provision
 - supports webhoooks and low latency streaming, which helps us for ASR -> LLM -> TTS
 - simple to handle (less complex infra)
 - costs dirtectly dependent on call duration as it would be acceptable for revenue to call to revenue ratio
 - best part , no telecom expertise required
 - Alternatives rejected : 1 - SignalWare (less scalable for AI centric bots) ; 2 - FreeSwitch (high maintenance + reliability risk) ; 3 - Direct carrier SIP trunks (requires telecom engineers)

**Real-Time Protocol** - 
As per our requirment , a major part of WebRTC and also benefits of Websockets, basically a hybrid system

Coming to WHY :
- WebRTC - For low latency, helps in bidirectional transfer of audio packets, handles jitter buffer, packet loss recovery, ICE/STUN/TURN for NAT traversal which gives us a better grade of telephony system, smoother integration with SFU
- WebSockets - only used for metadata signaling just to avoid media blockage channel
- problems that can be encountered - higher complexity ; CPU load ; additional infra for monitoring
- but these problems are comprimised for better audio quality and ASR which forwards to LLM and NLU for intent , which is the main core.
- acc to my research WebRTC overhead gives around 10 to 15% more CPU , which shall be acceptable for quality
- Alternatives rejected : 1 - Only WebSockets (cannot handle jitter - poor audio) ; 2 - gRPC (not optimized friendly for real time)
