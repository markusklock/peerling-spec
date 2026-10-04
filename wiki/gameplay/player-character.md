---
title: Player Character and Identity
type: system
status: stub
req_prefix: PLR
tags: [gameplay, player, identity]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/gameplay/onboarding.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-04
---

# Player Character and Identity

> The avatar the player controls, and the player's identity and save data.
> Status: stub.

## Character

[accepted] Every new player creates a character. Customization options and
whether the avatar is also AI-generated: [Q-019](../open-questions.md#q-019).

## Identity and save (proposed)

[proposed] No traditional accounts. On first launch the client generates a
cryptographic keypair; its public key is the player's identity (used as the
[creator](../glossary.md#creator) ID on species and for server rate limits).
[accepted] The save is a per-player OrbitDB log, and the key can be restored
with a recovery phrase. What the save contains, and how it is stored and
verified, is canonical in [player-data](../tech/player-data.md).

[proposed] Because the world is shared ([multiplayer](multiplayer.md)), the
identity also signs presence messages, PvP commitments and trade records. The
display name and avatar are visible to other players, so the display name
must be moderated. There is no chat to moderate ([MPL-007](multiplayer.md#requirements)).

## Requirements

- **PLR-001** [accepted] Each player MUST have a player character created during onboarding.
- **PLR-002** [proposed] Each player MUST have a stable cryptographic identity generated client-side.
- **PLR-003** [proposed] The player's display name MUST be moderated before other players can see it.

## Open questions

[Q-019](../open-questions.md#q-019) · [Q-027](../open-questions.md#q-027)
