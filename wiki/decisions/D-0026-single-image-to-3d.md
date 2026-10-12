---
title: "D-0026: One image to 3D"
type: decision
status: accepted
tags: [peerlings, generation, ai, image, 3d]
sources:
  - raw/conversations/2026-10-12-single-image-to-3d.md
related:
  - wiki/peerlings/image-prompting.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/generation-server.md
  - wiki/decisions/D-0025-prompt-enhancer-and-three-views.md
updated: 2026-10-12
---

# D-0026: One image to 3D

**Status:** accepted (2026-10-12). Partly supersedes
[D-0025](D-0025-prompt-enhancer-and-three-views.md) (its three views). The
details on [image-prompting § Image to 3D](../peerlings/image-prompting.md#image-to-3d)
are [proposed].

## Context

- [D-0025](D-0025-prompt-enhancer-and-three-views.md) gave the image-to-3D
  generator three images: the hero image plus a right-side and a back view,
  drawn by editing the hero image with Qwen-Image-2.1.
- In the designer's dry run, the hero prompt gave good transparent images,
  but the side and back edits returned the hero image unchanged (no
  multiple-angles LoRA, prompt rewriting off).
- The designer then gave TRELLIS.2 only the hero image of each of the three
  test creatures, and the models were very good.

## Decision

- [accepted] The multi-image step is dropped. Qwen-Image-2.1 generates a
  single image, the hero image the player accepts, and that image alone goes
  to the image-to-3D generator.
- [accepted] The image-to-3D generator is **TRELLIS.2 or Pixal3D**, in
  single-image mode. Which one runs by default is
  [Q-058](../open-questions.md#q-058).
- [proposed] What goes with the views: the side and back templates, the
  LoRA, the enhancer's `rightSideDetails` and `backDetails`, the camera
  file, the silhouette check with its single-view fallback (IMG-014), the
  job's `views` and the provenance fields `side`, `back` and `viewsUsed`.
- [proposed] How the model's front is found for a generator that picks its
  own orientation: see
  [image-prompting § Image to 3D](../peerlings/image-prompting.md#image-to-3d)
  (IMG-017).

## Consequences

- **Less to run:** two fewer Qwen-Image-2.1 generations after each accepted
  image, and one generator call instead of up to two (multi-view plus the
  single-view fallback).
- **The model invents the sides and back.** The player judges them in the
  [final review](../peerlings/creation-pipeline.md#final-review) and can go
  back to image generation there.
- **Pixal3D's advantage is gone:** it was named because it is the only one
  of the two with an official multi-view input. In single-image mode both
  are candidates ([Q-058](../open-questions.md#q-058)).
- **Smaller records:** one image prompt instead of three in provenance, so
  a typical species record is about 6 KB instead of 8 KB
  ([tech-stack § Asset budgets](../tech/tech-stack.md#asset-budgets)).
- **Unchanged:** the 3D format stays binary glTF 2.0 with meshopt and KTX2
  (STK-009), and the hero image still has a transparent background, which
  both generators use as the mask.

## Alternatives considered

- **Better edit prompts for the views** (leading with "rotate the camera",
  without "unchanged"), or the community multiple-angles LoRA. Untested; the
  LoRA only runs self-hosted.
- **One turnaround sheet** with all three views from one text-to-image
  prompt (2688 × 896). Untested; about 896 px per view and roughly 2.5 times
  the GPU time for every generation.
- The designer chose the single image, which already gave very good models.
