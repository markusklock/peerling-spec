---
title: New Player Onboarding
type: system
status: accepted
req_prefix: ONB
tags: [gameplay, onboarding, creation]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
related:
  - wiki/peerlings/creation-pipeline.md
  - wiki/gameplay/player-character.md
updated: 2026-10-06
---

# New Player Onboarding

> The first-time experience: a new player creates their character and gets a
> starter Peerling, either by designing it or by choosing an existing one, then
> steps into the world with it.

## Flow

[accepted] Every new player creates a player character and gets a starter
Peerling, once: a player can never get a second starter. [accepted] The player either **creates** their own starter or
**chooses** one of a few random existing species
([D-0012](../decisions/D-0012-starter-choice-and-extra-creations.md)).

### Choosing an existing starter

[accepted] The server offers **3 species**, drawn at random (uniformly) from
the registry, excluding removed species. The player picks one, or switches to
creating their own instead. The server signs an attestation for the new starter
([player-data § Verification](../tech/player-data.md#verification)). Players who
choose can still create Peerlings later at the
[Creation Shrine](creation-shrine.md).

### Creating a starter

[accepted] Suggested order, chosen to hide generation wait times:

1. **Welcome** — short intro to the world and the idea that every Peerling was
   made by a player. The player chooses: create a Peerling, or pick an existing
   one.
2. **Describe your Peerling** — the player writes a wish (stage 1 of the
   [creation pipeline](../peerlings/creation-pipeline.md)).
3. **Meet your Peerling** — the image is shown; accept or regenerate (stages
   3–4).
4. **Create your character** — while the 3D model, stats and moves generate
   in the background (stages 5–6), the player creates their
   [player character](player-character.md).
5. **Reveal** — the finished Peerling appears in 3D with its type(s), stats
   and moves. The player names it, or goes back to image generation
   ([final review](../peerlings/creation-pipeline.md#final-review)). Then the
   player's own browser publishes it to IPFS (stage 7). This is a
   good moment to show the player that their node now serves their creation to
   the world ([IPFS showcase](../tech/ipfs-showcase.md)). The server then signs
   the new [starter](../glossary.md#starter)'s origin attestation, with its
   traits and shimmer roll (stage 8).
6. **Into the world** — the player starts exploring; an early guaranteed
   encounter teaches battling and catching (how it is guaranteed:
   [Q-052](../open-questions.md#q-052)).

## Requirements

- ~~**ONB-001**~~ (removed 2026-10-04, replaced by ONB-005; see D-0012)
- ~~**ONB-002**~~ (removed 2026-10-04, replaced by ONB-006; see D-0012)
- **ONB-003** [accepted] Onboarding MUST let the player do something useful (e.g. character creation) while long generation stages run.
- **ONB-004** [accepted] If the player leaves during onboarding, they MUST be able to resume the same creation job later ([CRE-014](../peerlings/creation-pipeline.md#requirements)).
- **ONB-005** [accepted] A new player MUST create a player character and get a starter Peerling before starting to explore.
- **ONB-006** [accepted] The starter MUST be either an instance of a species the player creates, or an instance of an existing species the player chooses from a few random options.
- **ONB-007** [accepted] When choosing, the player MUST be offered 3 random species from the registry.
- **ONB-008** [accepted] Each player MUST get exactly one starter, ever ([API-004](../tech/creation-api.md#requirements)).

## Open questions

- [Q-052](../open-questions.md#q-052) — how the guaranteed first encounter works
