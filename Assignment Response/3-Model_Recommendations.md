Deliverable 3 :  LLM/ASR/TTS Model Recommendations


Since our project relies on latency, audio processing, real time decisions, LLM (intelligent) conversations, so based on that, the main factors to be taken in consideration are : 
- Speed (On quick actions and communications, with reduced latency)
- Stability (no hallucinations)
- Scalability (handling larger number of calls per day)



   
1. Speech to Text {ASR model} :
selected Deepgram, Google Cloud STT, Open AI WHisper

| ASR Provider                                | Streaming           | Latency    | Noise & Accent Handling                    | Cost               | Best Use Case                        | Notes                                                                  |
| ------------------------------------------- | ------------------- | ---------- | ------------------------------------------ | ------------------ | ------------------------------------ | ---------------------------------------------------------------------- |
| **Deepgram Nova / Aura**                    | Yes                 | Very Low | Excellent in noisy phone calls           | Mid                | Real-time calls                      | Built for call-center tier production, telephony models. |
| **Google Cloud STT ** | Yes                 | Low        | Excellent                                  | Mid–High           | Global deployments                   | Best language coverage and adaptive language models.                     |
| **Azure Speech**                            | Yes                 | Low        | Excellent                                  | Mid–High           | Enterprise compliane   | Customizable language models.                                 |
| **OpenAI Whisper **            | Community streaming | Medium     | Good (best in clean audio)                 | Low | Batch transcription                  | High accuracy, GPU heavy, less ideal for streaming.(w.r.t self hosted ONLY                    |
| **AssemblyAI**                       | Yes                 | Low        | Very Good                                  | Mid–High           | Real-time + rich after-call analytics | Built-in summarization + toxic detection.                              |
| **Speechmatics**                     | Yes                 | Low–Medium | Excellent multilingual & accent robustness | Mid                | Multinational contact centers        | Very strong in accents & dialects.                                     |


My recommendations : 
- For inbound calls -  Deepgram Nova - Best for real-time latency, streaming, better for telephony (if not then Google cloud STT for simpler infra)
- Batch Transcription - Only if we self host, then OpenAI whisper for large scale, if no self hosting then batch transcription in Deepgrams pre recorded ASR API, better for telephony audio support, lower cost compared to google and azyre
- For handling noise - we already opted for AEC, VAD and noise suppresion in SFY, that should take care of it
- For accent - Deepgram (which i recommended) would solve that issue too.


2. TTS model comparision :


| TTS Provider                        | Naturalness          | Latency  | Cost     | Voice Customization | Best Use Case                 | Notes                                    |
| ----------------------------------- | -------------------- | -------- | -------- | ------------------- | ----------------------------- | ---------------------------------------- |
| **Amazon Polly Neural **      | Very Good            | Very Low | Low–Mid  | Good                | Default production TTS        | Most stable + scalable option.           |
| **ElevenLabs**                      | Excellent  | Low      | High     | Excellent + cloning | Premium user-facing AI agents | Human-like voices but pricey.            |
| **Google WaveNet**                  | Very Good            | Low      | Mid      | Good                | Global deployments            | Clean waveforms & stable.                |
| **Azure Neural TTS**                | Very Good            | Low      | Mid      | Good                | Enterprise apps               | Custom neural voice models available.    |
| **Play.ht**                  | Very High            | Low      | Mid–High | Excellent           | Marketing/brand personas      | Voice cloning + expressive tone.         |
| **Coqui TTS**  | High                 | Medium   | Very Low | Good with training  | Self-hosting & custom voices  | Good for companies wanting full control. |

My Recommendation : 
- Amazon polly Neural : lower latency, lower cost, highly reliable, and also heard from direct experienced clients, can also go with google WaveNet (go for ElevenLabs for premium servies, but costly)

3. LLMs for intent recognition and response :
   
we have to opt for lightweight intent classifier, llm for response generation and adaptable to state machine too

| Model                               | Latency             | Quality        | Cost                    | Safety      | Best Use Case                      | Notes                                         |
| ----------------------------------- | ------------------- | -------------- | ----------------------- | ----------- | ---------------------------------- | --------------------------------------------- |
| **OpenAI GPT-4o / 4.1**             | Low–Medium          | 🔹Best         | High                    | Excellent   | Complex reasoning + safe responses | Best overall for natural voice agents.        |
| **Claude 3.5 Sonnet**               | Low                 | Excellent      | High                    | Best safety | Complex dialogues                  | Very stable + less hallucinations.            |
| **Llama 3 (7B/70B quantized)**      | Very Low (if local) | Good–Very Good | Low (self-hosted infra) | Medium      | Fast intent + short replies        | Your architecture supports this nicely.       |
| **Mixtral 8x7B**                    | Low                 | Good           | Low                     | Medium      | Cheaper LLM fallback               | Good latency on GPUs, cheaper inference.      |
| **Google Gemini 1.5 Flash**  | Low                 | High           | Medium                  | High        | Fast real-time reasoning           | Best for “fast but smart” tasks.              |
| **Mistral Nemo**             | Low                 | Good           | Low                     | Medium      | Self-hosting                       | Good for on-prem latency-sensitive workloads. |


My Recommendations : 
- Using seperate models for intent classification (detection) and Response Generation would be better, as the former takes charge of recognition, hence should be deteministic,cheap,accurate and fast and the later deals with response which invlovles expressivness, context rich and safety.
- for intent detection - going with Llama 3 8b (locally qquantized one) - cheap, super fast, stable
- for response generation - since this carries compilance and involves in direct human interactions, going with cloud used (API based) GPT 4o (complex queries handling) or Mixtral 56B (covers mid tier segment)


Cost Implications :

Since assuming 1000s of daily operations : 

1. ASR :
   Deepgram Nove ~ $0.0043 per min
   Daily cost ~ $25 ($20(standar) + $5(buffer)
   Monthly cost ~ $750
   Assuming roughly ~ $1000
2. TTS :
   Amazon Polly ~ $0.001 (Neural)
   Daily cost ~ $50 per day
   Monthly ~ $1500

3. LLMs :
   around 8 to 12 LLM interactions per call ~ 10000 llm calls per day
   GPT 4o - depends on token ~ $0.005 to $0.03
   daily ~ $50 to $300 per day
   monthly ~ $1500 to $7500

Cost Optimization :

ASR : Using VAD can reduc upto 20 to 30% less audio, enabling streaming only when caller speaks
TTS : Caching common phrases around 50% less TTS usage
LLM : small Llama for simple replies, sending fewer tokens, caching repetitive repsonses


Fallback Strategy :

ASR Fallbacks : 
- Primary → Deepgram
- Backup → Google STT
- If both fail → ask “Can you repeat?”
- If still fails → send to human agent

TTS Fallbacks : 
- Primary → Amazon Polly
- Backup → Google TTS
- Last fallback → Pre-recorded audio messages

LLM Fallbacks
- Primary → GPT-4o / Claude
- Backup → Mixtral 8x7B
- Local fallback → Llama 3
- Hard fallback → Static templates
- Final → Human agent

Open-Source / Self-Hosted :

When YES (Use Open-Source):
- Intent detection → Small local model
- Cheap fallback model → Llama 3 / Mixtral

When NO (Avoid Self-Hosting):
- ASR (hard to match Deepgram/Google quality)
- TTS (premium voices impossible to match)
- Primary LLM (GPT/Claude give better quality + safety)

→ Hybrid:
Cloud for ASR, TTS, main LLM
Local for intent + fallback LLM
