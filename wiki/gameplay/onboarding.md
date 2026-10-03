---
title: New Player Onboarding
type: system
status: draft
req_prefix: ONB
tags: [gameplay, onboarding, creation]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/peerlings/creation-pipeline.md
  - wiki/gameplay/player-character.md
updated: 2026-10-03
---

# New Player Onboarding

> The first-time experience: a new player creates their character and designs
> their starter Peerling, then steps into the world with it.

## Flow

[accepted] Every new player creates a player character and a starting Peerling.

[proposed] Suggested order, chosen to hide generation wait times:

1. **Welcome** — short intro to the world and the idea that every Peerling was
   made by a player.
2. **Describe your Peerling** — the player writes a wish (stage 1 of the
   [creation pipeline](../peerlings/creation-pipeline.md)).
3. **Meet your Peerling** — the image is shown; accept or regenerate (stages
   3–4).
4. **Create your character** — while the 3D model, stats and moves generate
   in the background (stages 5–6), the player creates their
   [player character](player-character.md).
5. **Reveal** — the finished Peerling appears in 3D with its name, type(s) and
   moves. The player's own browser publishes it to IPFS (stage 7). This is a
   good moment to show the player that their node now serves their creation to
   the world ([IPFS showcase](../tech/ipfs-showcase.md)). It then becomes the
   player's [starter](../glossary.md#starter) (stage 8).
6. **Into the world** — the player starts exploring; an early guaranteed
   encounter teaches battling and catching.

## Requirements

- **ONB-001** [accepted] A new player MUST create a player character and a starter Peerling before starting to explore.
- **ONB-002** [accepted] The starter MUST be an instance of the species the player designed.
- **ONB-003** [proposed] Onboarding MUST let the player do something useful (e.g. character creation) while long generation stages run.
- **ONB-004** [proposed] If the player leaves during onboarding, they MUST be able to resume the same creation job later ([CRE-014](../peerlings/creation-pipeline.md#requirements)).

## Open questions

[Q-001](../open-questions.md#q-001) · [Q-019](../open-questions.md#q-019) ·
[Q-022](../open-questions.md#q-022)
