---
title: Overview — Vision, Goals and Design Pillars
type: overview
status: draft
tags: [vision, pillars]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
related:
  - wiki/gameplay/core-loop.md
  - wiki/tech/architecture.md
  - wiki/peerlings/creation-pipeline.md
updated: 2026-10-05
---

# Overview — Vision, Goals and Design Pillars

> Peerlings is a Pokémon-inspired creature-collecting game that runs entirely in
> the browser, where every creature is designed by a player and distributed
> peer-to-peer over IPFS. This page defines the vision and the pillars every
> other part of the spec must serve.

## Elevator pitch

You arrive in a procedurally generated world with one companion: a
[Peerling](glossary.md#peerling) **you** invented. You describe it in your own
words; AI models turn that description into artwork, a 3D model, an elemental
type and a move set. Your creation is then published to IPFS — and from that
moment it lives in *everyone's* world. Exploring means meeting the imaginations
of other players: every wild Peerling you fight and catch was dreamt up by
someone else.

## Goals

The game has **two equal goals** [accepted]:

1. **Showcase IPFS.** Make content addressing, peer-to-peer distribution and
   decentralized data tangible and impressive to players. Every player's browser
   is a real IPFS node ([Helia](glossary.md#helia)).
2. **Be genuinely fun.** A game people enjoy for its own sake, not a tech demo
   with a game skin.

When the two goals conflict, look for a design that serves both; record the
trade-off in a [decision](decisions/) if one goal must yield.

## Design pillars

| # | Pillar | Meaning for the design | Provenance |
|---|--------|------------------------|------------|
| 1 | **Every creature is someone's creation** | There is no hand-designed roster. All species come from players via the [creation pipeline](peerlings/creation-pipeline.md). Even the operator's handful of launch [seed species](glossary.md#seed-species) go through the same pipeline. | [accepted] |
| 2 | **IPFS is the backbone, and it shows** | Peerling data and 3D models live on IPFS; the registry lives in [OrbitDB](glossary.md#orbitdb); players are nodes that download *and* serve content. The tech should be visible and celebrated in the UI (see [IPFS showcase](tech/ipfs-showcase.md)). | [accepted] (UI visibility: [accepted]) |
| 3 | **Familiar creature-collecting fun** | Explore a [procedural world](world/procedural-generation.md), encounter, battle and catch — the Pokémon formula players already understand — in one world shared with every other player, who you can battle and trade with ([multiplayer](gameplay/multiplayer.md)). | [accepted] |
| 4 | **Fair by construction** | Generated content is constrained by predefined [types](peerlings/types.md), [move templates](peerlings/moves.md) and the same base-stat total for every species, so no player's creation is objectively stronger because of how it was described. Individual Peerlings still differ by up to ±10% per stat ([D-0014](decisions/D-0014-individual-variation.md)). | [accepted] |
| 5 | **Minimal central infrastructure** | One operator server runs generation and pinning; everything else is peer-to-peer and client-side. See [architecture](tech/architecture.md). | [accepted] |

## Scope of the first version

The first playable version (v1) contains:

- [accepted] Player onboarding: create a player character and a starter Peerling.
- [accepted] Exploration of one procedural world shared by all players, who see
  each other ([D-0008](decisions/D-0008-shared-multiplayer-world.md)).
- [accepted] Wild encounters with player-created Peerlings, turn-based battles,
  catching.
- [accepted] PvP battles and trading between players.
- [accepted] A team/collection of caught Peerlings.
- [accepted] A top-down camera over a colorful 3D world ([visual-style](world/visual-style.md)).

[accepted] Not in v1: evolution, Peerlings learning new moves, items in
battles, and content moderation ([D-0010](decisions/D-0010-no-content-moderation.md)).

## See also

- [Core gameplay loop](gameplay/core-loop.md)
- [System architecture](tech/architecture.md)
- [Glossary](glossary.md)
