---
title: Camera and Visual Style
type: system
status: draft
req_prefix: VIS
tags: [world, presentation, camera, art]
sources:
  - raw/conversations/2026-10-04-answers-round-5.md
related:
  - wiki/world/procedural-generation.md
  - wiki/gameplay/exploration.md
  - wiki/gameplay/battle.md
  - wiki/peerlings/creation-pipeline.md
updated: 2026-10-04
---

# Camera and Visual Style

> How the game looks: a top-down camera over a colorful 3D world, in which the
> AI-generated Peerlings must look at home.

## Camera

[accepted] The world is 3D and seen from a **top-down camera**.

[proposed] Details:
- **Tilted, not straight down.** The camera looks down at about 55° below the
  horizon. Peerling models are generated from three-quarter front images
  ([creation-pipeline § Stage 3](../peerlings/creation-pipeline.md#stage-3--image)),
  so a straight-down view would mostly show the tops of their heads.
- **Fixed orientation.** North is always up and the camera doesn't rotate,
  like classic top-down creature games. This keeps controls simple.
- **Follows the player**, keeping their character centred. The view covers
  roughly 30–40 m around the player, enough to see nearby players and
  terrain without having to load much of the world.
- **Battles:** the battle happens on the spot. The camera moves in closer and
  lower (about 30° below the horizon) and frames the two Peerlings facing
  each other: the player's Peerling in the foreground (lower left), the
  opponent further back (upper right). This is the classic creature-battle
  layout.

## Visual style

[accepted] The visual style is **colorful**.

[proposed] Details:
- Bright, saturated colors, soft lighting and simple, stylized shapes: a
  "toy-like" world rather than a realistic one. This also keeps the 3D cheap
  enough to render in any browser.
- Each of the 12 [biomes](procedural-generation.md#biomes) has its own dominant
  colors, so players can tell where they are at a glance.
- **Peerlings must fit in.** The fixed house-style block of the image prompt
  ([CRE-010](../peerlings/creation-pipeline.md#requirements)) uses the same
  look: colorful, soft-shaded, stylized 3D-render style. Generated models then
  match the world instead of looking pasted in.
- Player characters use the same stylized look.

## Requirements

- **VIS-001** [accepted] The world MUST be rendered in 3D and viewed from a top-down camera.
- **VIS-002** [accepted] The visual style MUST be colorful.
- **VIS-003** [proposed] The exploration camera MUST be tilted (about 55° below the horizon), fixed north-up, and follow the player.
- **VIS-004** [proposed] The image prompt's house style MUST match the world's colorful, stylized look.
- **VIS-005** [proposed] Each biome MUST have a visually distinct dominant color palette.

## See also

- [Procedural generation](procedural-generation.md) · [Exploration](../gameplay/exploration.md) · [Battle § Presentation](../gameplay/battle.md#presentation)
