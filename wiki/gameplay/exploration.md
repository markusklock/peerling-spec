---
title: Exploration
type: system
status: stub
req_prefix: EXP
tags: [gameplay, world]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-tech-stack-2.md
related:
  - wiki/world/procedural-generation.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/multiplayer.md
  - wiki/world/visual-style.md
updated: 2026-10-04
---

# Exploration

> How the player moves through and experiences the world. Status: stub.

[accepted] The player travels around a [procedurally generated world](../world/procedural-generation.md)
looking for Peerlings. [accepted] The world is large but finite. [accepted] The world is shared, so other players
exploring nearby are visible ([multiplayer](multiplayer.md)).

[accepted] The world is seen from a top-down camera in a colorful style
([visual-style](../world/visual-style.md)).

## Healing and rest points

[accepted] There are no healing items. A Peerling's HP carries over between
battles and is restored at **rest points**:
- Every biome area has a rest point
  ([procedural-generation § Layout](../world/procedural-generation.md#layout)).
  Visiting it fully heals the whole team, and it becomes the player's *last
  rest point*.
- If the player's whole team faints, the player returns to their last rest
  point with the team fully healed. Nothing is lost.

[accepted] The game targets desktop browsers only
([STK-010](../tech/tech-stack.md#requirements)), so controls are designed for
keyboard and mouse.

To be specified: the exact controls, movement, what triggers encounters (e.g. tall grass,
visible roaming Peerlings), points of interest, and the look of rest points.

## Requirements

- **EXP-001** [accepted] The player MUST be able to travel freely around the procedurally generated world.
- **EXP-002** [accepted] Visiting a rest point MUST fully heal the player's team and record it as the last rest point.
- **EXP-003** [accepted] When the whole team faints, the player MUST return to the last rest point with the team fully healed, losing nothing.

## Open questions

_None at the moment._
