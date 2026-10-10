---
title: "D-0025: Prompt enhancer, transparent images and three views for 3D"
type: decision
status: accepted
tags: [peerlings, generation, ai, image, 3d, originality]
sources:
  - raw/conversations/2026-10-10-image-prompt-enhancer.md
related:
  - wiki/peerlings/image-prompting.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/generation-server.md
  - wiki/decisions/D-0004-single-operator-server.md
  - wiki/decisions/D-0010-no-content-moderation.md
updated: 2026-10-10
---

# D-0025: Prompt enhancer, transparent images and three views for 3D

**Status:** accepted (2026-10-10) for the points the designer set; the
details on [image-prompting](../peerlings/image-prompting.md) are
[proposed] until the designer reviews them.

## Context

- **What the spec had:** a self-hosted image model (FLUX.2 was named as an
  example) with a structured JSON prompt. The image was a single
  three-quarter view on a plain background, which the 3D stage then cut out.
- **What the designer chose:**
  - an LLM prompt enhancer in front of the image model;
  - Qwen-Image-2.1, which can draw transparent backgrounds;
  - three views for the 3D stage.
- **The designer's worry:** players could recreate existing Pokémon.

## Decision

- [accepted] Pipeline: wish → **prompt enhancer (GPT-6 Luna, OpenAI API)**
  → **Qwen-Image-2.1** → the player accepts or regenerates → **TRELLIS.2 or
  Pixal3D** make the 3D model.
- [accepted] Images have a **transparent background**.
- [accepted] The pipeline must stop players from **cloning existing
  Pokémon** by naming or describing them.
- [accepted] **Three images from different angles** are given to the
  image-to-3D generator.
- [proposed] How: the enhancer's system prompt, fixed prompt templates,
  layered originality rules, image settings and checks, the camera layout of
  the three views and a silhouette check with a single-view fallback, as
  described on [image-prompting](../peerlings/image-prompting.md).

## Consequences

- **Self-hosting:** the enhancer is the first model not hosted by the
  operator. This partly changes [D-0004](D-0004-single-operator-server.md),
  where all models were self-hosted.
  - [SRV-001](../tech/generation-server.md#requirements) names the enhancer
    as the exception.
  - If OpenAI's API is down, creation falls back to the concept's own
    appearance text.
- **Moderation:** wishes are now steered away from existing characters.
  This partly changes [D-0010](D-0010-no-content-moderation.md) ("wishes …
  are not filtered"). Nothing is rejected and taste isn't judged, so the
  game stays otherwise unmoderated.
- **Stage 5** no longer needs background removal for normal images.
- **Species record:** provenance now holds three prompts and the
  enhancer's originality notes, so the typical record grows from about 4 KB
  to about 8 KB, still within the 16 KB budget.
- **Licences:** Qwen-Image-2.1's licence is non-commercial, and the 3D
  models depend on components with restrictive licences
  ([Q-057](../open-questions.md#q-057)).

## Alternatives considered

- **Keep a single image** (the old spec). The 3D model has to invent the
  sides and back, which is the guessing the designer wants to reduce.
- **One 1 × 3 turnaround sheet in a single image.** The views are more
  alike, but each one gets only about 680 px, and the hero image the player
  reviews would be a sheet rather than a portrait.
- **Name blocklist only.** Research shows a description without the name
  still recreates the character.
- **Refusing wishes that name a character.** This is harsher on players and
  against the spirit of D-0010; rewriting keeps the player's idea.
