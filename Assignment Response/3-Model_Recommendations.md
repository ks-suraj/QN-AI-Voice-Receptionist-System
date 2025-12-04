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
