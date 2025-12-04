1. Speech to Text {ASR model} :
selected Deepgram, Google Cloud STT, Open AI WHisper

| ASR Provider                                | Streaming           | Latency    | Noise & Accent Handling                    | Cost               | Best Use Case                        | Notes                                                                  |
| ------------------------------------------- | ------------------- | ---------- | ------------------------------------------ | ------------------ | ------------------------------------ | ---------------------------------------------------------------------- |
| **Deepgram Nova / Aura**                    | Yes                 | 🔹Very Low | 🔹Excellent in noisy phone calls           | Mid                | Real-time calls                      | Built for call-center tier production. Diarization + telephony models. |
| **Google Cloud STT (Enhanced Phone Model)** | Yes                 | Low        | Excellent                                  | Mid–High           | Global deployments                   | Best language coverage + adaptive language models.                     |
| **Azure Speech**                            | Yes                 | Low        | Excellent                                  | Mid–High           | Enterprise compliance + SOC2 needs   | Customizable acoustic/language models.                                 |
| **OpenAI Whisper (self-hosted)**            | Community streaming | Medium     | Good (best in clean audio)                 | Low if self-hosted | Batch transcription                  | High accuracy, GPU heavy, less ideal for streaming.                    |
| **AssemblyAI** *(New)*                      | Yes                 | Low        | Very Good                                  | Mid–High           | Real-time + rich post-call analytics | Built-in summarization & toxic detection.                              |
| **Speechmatics** *(New)*                    | Yes                 | Low–Medium | Excellent multilingual & accent robustness | Mid                | Multinational contact centers        | Very strong in accents & dialects.                                     |
