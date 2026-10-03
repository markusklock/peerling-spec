---
title: Player Character and Identity
type: system
status: stub
req_prefix: PLR
tags: [gameplay, player, identity]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/gameplay/onboarding.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-03
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
The keypair, the save (collection, team, position, progress) and settings are
stored in browser storage. Backup/recovery: [Q-014](../open-questions.md#q-014).

## Requirements

- **PLR-001** [accepted] Each player MUST have a player character created during onboarding.
- **PLR-002** [proposed] Each player MUST have a stable cryptographic identity generated client-side.

## Open questions

[Q-014](../open-questions.md#q-014) · [Q-019](../open-questions.md#q-019)
