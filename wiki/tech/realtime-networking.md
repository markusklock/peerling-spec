---
title: Realtime Peer-to-Peer Networking
type: system
status: draft
req_prefix: NET
tags: [tech, libp2p, multiplayer, pubsub]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
related:
  - wiki/decisions/D-0008-shared-multiplayer-world.md
  - wiki/gameplay/multiplayer.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-04
---

# Realtime Peer-to-Peer Networking

> How players find and talk to each other in the shared world. Everything runs
> over the same libp2p node that powers each client's Helia IPFS node, with no
> game server. This is a showcase of the libp2p stack underneath IPFS.

## Building blocks (proposed)

| Need | Mechanism |
|------|-----------|
| Know who is nearby | libp2p **pubsub (gossipsub)** topic per world [region](../glossary.md#region) |
| Battle, trade, profile | Direct libp2p **streams** with custom protocol IDs |
| Epoch records | Pubsub topic `peerlings/v1/epoch`, published by the server ([player-data § Encounter seeds](player-data.md#encounter-seeds)) |
| Reaching other browsers | Circuit relay via the operator server, upgraded to direct **WebRTC** connections (preferring IPv6) when possible ([ipfs-helia § Connectivity](ipfs-helia.md#connectivity)) |

## Presence (proposed)

- The world is divided into square regions of **64 m × 64 m**
  (recommended 2026-10-04, [Q-027](../open-questions.md#q-027)). The 4 km world
  then has about 63 × 63 regions.
- A client subscribes to the presence topic of its current region and its 8
  neighbours (`peerlings/v1/presence/<rx>_<ry>`). It changes subscriptions only
  once it is 8 m past a region border, so walking along a border doesn't cause
  constant resubscribing.
- A client publishes a **presence message** to its current region topic **4
  times per second while moving**, and a heartbeat **every 5 s when idle**. Contents: peer ID, player ID, display name, avatar
  reference, position, facing, timestamp, signature.
- [accepted] There is no chat; [emotes](../glossary.md#emote) are the only
  player-to-player messages. [proposed] An emote is sent as a presence message
  with an `emote` field (an ID from the fixed set in
  [multiplayer § Communication](../gameplay/multiplayer.md#communication)),
  rate-limited like other presence messages. Receivers ignore unknown emote IDs.
- A player not heard from for **15 s** is removed from view.
- Budget: a message is about 200 bytes, so 30 visible moving players cost about
  30 × 4 × 200 B ≈ 24 KB/s of download, which is fine on desktop.
- [proposed] The operator server also joins the topics, to help gossip reach
  browsers that have few direct peers.

## Direct protocols (proposed)

| Protocol ID | Purpose | Canonical page |
|-------------|---------|----------------|
| `/peerlings/battle/1.0.0` | PvP battle session | [pvp-battles](../gameplay/pvp-battles.md) |
| `/peerlings/trade/1.0.0` | Trade session | [trading](../gameplay/trading.md) |
| `/peerlings/profile/1.0.0` | Request a player's public profile (team, created species) | [multiplayer](../gameplay/multiplayer.md) |

Message formats are still to be specified.

## Requirements

- **NET-001** [proposed] Player-to-player communication MUST use the client's libp2p node; game logic MUST NOT depend on a game server (the operator server MAY relay transport traffic).
- **NET-002** [proposed] Presence MUST be distributed via pubsub topics scoped to world regions, so a client only receives nearby players.
- **NET-003** [proposed] Presence messages MUST be signed with the player's identity key and rate-limited by the sender; receivers MUST drop messages that are unsigned, too frequent or impossibly far from the previous position.
- **NET-004** [proposed] Battles, trades and profile requests MUST use versioned libp2p protocol IDs.

## Open questions

[Q-027](../open-questions.md#q-027)

## See also

- [Architecture](architecture.md) · [Multiplayer](../gameplay/multiplayer.md)
