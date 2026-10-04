---
title: "D-0006: Separate immutable species from owned instances"
type: decision
status: accepted
tags: [peerlings, data-model]
sources:
  - raw/conversations/2026-10-04-answers-round-4.md
related:
  - wiki/peerlings/peerling-species.md
updated: 2026-10-04
---

# D-0006: Separate immutable species from owned instances

**Status:** accepted (2026-10-04)

## Context
A player's creation must appear in many players' worlds, be caught many times,
and level up independently in each owner's team.

## Decision
- A **[species](../glossary.md#species)** is the immutable design, stored on IPFS
  as a [species record](../glossary.md#species-record); its CID is its identity.
- A **[Peerling instance](../glossary.md#peerling-instance)** is one individual
  (level, XP, current HP, nickname, …) owned by one player, stored in that
  player's save and referencing its species by CID.
- The starter is an instance of the species its owner created.

## Consequences
- Species data is shared and cacheable by CID across all players; instance data
  is small and private to the owner.
- Species can never be edited after publishing; a "fix" is a new species.

## Alternatives considered
- Each creation is a single unique creature (only one exists): conflicts with
  wild encounters of player-created Peerlings in every world.
