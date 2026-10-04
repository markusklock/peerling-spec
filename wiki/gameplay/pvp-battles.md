---
title: PvP Battles
type: system
status: draft
req_prefix: PVP
tags: [gameplay, multiplayer, battle, libp2p]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
related:
  - wiki/gameplay/battle.md
  - wiki/gameplay/multiplayer.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-04
---

# PvP Battles

> How two players battle each other peer-to-peer, without a game server
> deciding the outcome. The battle *rules* are the same as for wild battles
> ([battle](battle.md)); this page covers setup, fairness and the protocol.

## Overview

[accepted] Players can battle each other. [proposed] PvP battles are played
directly between the two players' browsers over a libp2p stream
([realtime-networking](../tech/realtime-networking.md)). Both clients run the
same deterministic battle engine (BTL-002) and must agree on every turn.

## Fairness (proposed)

Player saves live in the browser and can be edited, so the protocol must not
trust what a client claims. [proposed] Approach, still open in
[Q-025](../open-questions.md#q-025):

- **Species are verifiable.** Each Peerling's species record is fetched by CID
  and its attestation checked, so stats, types and moves can't be faked.
- **Levels are normalized.** All Peerlings fight at the same fixed level in
  PvP (as in Pokémon's competitive formats). Levels can't be verified, so
  edited levels then give no advantage.
- **Ownership is not verified** in v1: a modified client could field any
  species. Since every species is balanced by construction, that gives little
  advantage.

Moving saves to IPFS/OrbitDB and having the server sign catches would close the
ownership gap. Options and a recommendation are in
[player-data](../tech/player-data.md).

## Protocol (proposed)

1. **Challenge.** A sends a challenge to B while standing next to them
   (MPL-006); B accepts or declines (MPL-005).
2. **Team exchange.** Each side sends its team as a list of species CIDs, and
   each side fetches and verifies the other's species.
3. **Shared randomness.** Both commit to a random value (send its hash), then
   reveal it. The battle RNG seed is derived from both values, so neither side
   controls the random rolls.
4. **Turns.** Each turn, both sides commit to their action (send a hash), then
   reveal it. Neither can react to the other's choice. Both run the engine and
   exchange a hash of the resulting battle state. A mismatch voids the battle.
5. **End.** The result is shown to both players. A disconnect, or no action
   within a turn timer, counts as a forfeit after a grace period.

## Rewards

TBD. [proposed] No rewards that can be farmed in v1 (e.g. no XP from PvP),
since results can't be verified by a third party.

## Requirements

- **PVP-001** [accepted] Two players MUST be able to battle each other with their Peerlings.
- **PVP-002** [proposed] PvP battles MUST run peer-to-peer between the two clients, with no server deciding the outcome.
- **PVP-003** [proposed] Each client MUST verify every opposing species record (attestation) before the battle starts.
- **PVP-004** [proposed] The battle RNG seed MUST be derived from values committed and revealed by both players.
- **PVP-005** [proposed] Turn actions MUST use commit-reveal, so neither player sees the other's choice before committing.
- **PVP-006** [proposed] Clients MUST compare battle-state hashes after each turn and void the battle on mismatch.

## Open questions

[Q-025](../open-questions.md#q-025)

## See also

- [Battle rules](battle.md) · [Multiplayer](multiplayer.md)
