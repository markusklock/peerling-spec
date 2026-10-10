# 2026-10-10 — No self-hosted LLM; GPT-6 Luna for every LLM step

Kind: design conversation. This answers the question in the report on the
image-prompting work: should GPT-6 Luna also do the concept stage?

## Designer's answer (verbatim)

> Yes, I decided to not use a self-hosted LLM as GPT-6 Luna is very cheap, that unlocks more VRAM on my server GPU for Pixal3D and Qwen Image 2.1

## Summary of what was decided

- The server runs **no self-hosted LLM**. Every LLM step uses **GPT-6 Luna**
  through OpenAI's API:
  - the concept (stage 2);
  - the prompt enhancer (stage 3);
  - stats and moves (stage 6).
- **Why:** GPT-6 Luna is very cheap, and the server's GPU memory is freed
  for Pixal3D and Qwen-Image-2.1.
- The GPU models that stay self-hosted are **Qwen-Image-2.1** and
  **Pixal3D**. The designer names Pixal3D as the 3D model running on the
  server.
