# 2026-10-10 — Image prompt enhancer, transparent images, three views

Kind: design conversation. The designer asked for the system prompt of a new
prompt-enhancer stage and for research on the new image and 3D models.

## Designer's request (verbatim)

> Next we should work on the system promt that should be fed in to the prompt enhancer before it hands the description to the Image Generation model that will generate the images of the Peerlings before they are turned in to 3D-models.
> The full pipeline is planned like this: User description/whish of new create -> Prompt enhancer (GPT-6 Luna via API) -> Qwen Image 2.1 will generate an image, if the user is happy with the result -> TRELLIS.2 or Pixel3D turn it into a 3D-model.
> I will use the new model Qwen Image 2.1 for images, it generate great results and is capable of generating images with transparent backgrounds which should be good for the 3D-model stage later.
> Research good prompting styles for Qwen Image 2.1, how to make it generate an image of a Pokemon style creature from a user prompt with transparent background. We should also make sure the system prompt prevent the user for describing exact Pokemons to avoid cloning Pokemons in to Peerlings.
> We should also generate 3 image from different angels to feed in to TRELLIS.2/Pixel3D so that it does not have to guess as much.

## Summary of what was decided

- A **prompt enhancer** stage is added between the wish and the image model.
  It is **GPT-6 Luna**, called through OpenAI's API (not self-hosted).
- The image model is **Qwen Image 2.1**, generating images with a
  **transparent background**.
- The 3D model comes from **TRELLIS.2 or "Pixel3D"** (the research found the
  model is named Pixal3D, from TencentARC).
- The enhancer's system prompt must stop players from describing existing
  Pokémon exactly, so Pokémon aren't cloned into Peerlings.
- **Three images from different angles** of the creature are generated and
  given to the 3D model, so it has less to guess.
- The designer asked the LLM to research prompting styles and write the
  system prompt; those details are LLM proposals until approved.

## Research findings (by the LLM, checked 2026-10-10)

- **Qwen-Image-2.1** (weights 2026-09-20): a 7B model; one checkpoint does
  text-to-image, editing with 1–10 reference images, and native RGBA output.
  - **Transparency:** triggered by the prompt, using the official wording
    "This is an RGBA image with transparency. … The image has alpha channel
    and the background is transparent."
  - **Prompts:** the official prompt rewriter writes one long English
    paragraph that describes the finished image in present tense, with no
    quality boosters.
  - **Sampling:** without guidance (`true_cfg_scale` 1.0), so negative
    prompts have no effect, and the hosted API doesn't accept them for 2.1.
  - **Licence:** Qwen Research License, which is non-commercial.
  - **Other views:** a community "Multiple-Angles" LoRA turns a reference
    image to other camera angles.
  - Sources: github.com/QwenLM/Qwen-Image-2.1,
    huggingface.co/Qwen/Qwen-Image-2.1, the Qwen-Image-2.1-PE-T2I and
    PE-I2I rewriter system prompts, Alibaba Cloud Model Studio API reference,
    huggingface.co/akhaliq/Qwen-Image-2.1-Multiple-Angles-LoRA.
- **TRELLIS.2** (Microsoft, MIT):
  - **Input:** officially one image only. It uses the image's own alpha when
    present and crops to the subject.
  - **Multi-image:** only through unofficial community ports.
  - **Licences:** it depends on RMBG-2.0 (CC BY-NC) for background removal,
    nvdiffrast (NVIDIA research/evaluation licence) for texture baking, and
    DINOv3 (attribution required).
- **Pixal3D** (TencentARC and Tsinghua, SIGGRAPH 2026, MIT):
  - **Input:** single-image mode, plus an official multi-view mode since
    2026-09-01. Multi-view needs posed cameras, with the front view as the
    first frame and each view framed exactly to its camera.
  - **Tolerance:** about 5° of pose error is tolerated.
  - **Single image:** a narrow field of view (`--fov 0.2`) is recommended.
  - **Licences:** the same dependencies as TRELLIS.2.
- **GPT-6 Luna:**
  - API ID `gpt-6-luna`: "Our most efficient model for focused, high-volume
    tasks".
  - Price: $0.10 input / $0.50 output per 1M tokens.
  - Supports Structured Outputs (strict JSON schema) and reasoning effort
    `none`–`max`.
  - OpenAI advises passing untrusted input in user messages and using fixed
    schemas.
- **Stopping character clones:** "Fantastic Copyrighted Beasts" (ICLR 2025)
  found that removing names isn't enough, because a few typical keywords
  recreate a character.
  - The ChatGPT image rewriter's rule was to describe "a specific different
    character with a different specific color, hair style, or other defining
    visual characteristic".
  - The best results came from layered defences.
