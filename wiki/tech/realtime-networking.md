---
title: Realtime Peer-to-Peer Networking
type: system
status: accepted
req_prefix: NET
tags: [tech, libp2p, multiplayer, pubsub]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-peer-save-backups.md
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-network-performance-approved.md
related:
  - wiki/decisions/D-0008-shared-multiplayer-world.md
  - wiki/gameplay/multiplayer.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-06
---

# Realtime Peer-to-Peer Networking

> How players find and talk to each other in the shared world. Everything runs
> over the same libp2p node that powers each client's Helia IPFS node, with no
> game server. This is a showcase of the libp2p stack underneath IPFS.

## Building blocks

| Need | Mechanism |
|------|-----------|
| Know who is nearby | libp2p **pubsub (gossipsub)** topic per world [region](../glossary.md#region) |
| Battle, trade, profile | Direct libp2p **streams** with custom protocol IDs |
| Watching PvP battles | Pubsub topic `peerlings/v1/battle/<battleId>` ([spectating](../gameplay/spectating.md)) |
| World feed | Pubsub topic `peerlings/v1/feed` ([world-feed](../gameplay/world-feed.md)) |
| Creator notifications | Pubsub topic `peerlings/v1/creator/<player ID>`, published by the server ([creator-feedback](../gameplay/creator-feedback.md)) |
| Save recovery requests | Pubsub topic `peerlings/v1/save-wanted` ([player-data](player-data.md#keeping-saves-available)) |
| Epoch records | Pubsub topic `peerlings/v1/epoch`, published by the server ([player-data § Encounter seeds](player-data.md#encounter-seeds)) |
| Reaching other browsers | Circuit relay via the operator server, upgraded to direct **WebRTC** connections (preferring IPv6) when possible ([ipfs-helia § Connectivity](ipfs-helia.md#connectivity)); dialled early when an interaction is likely, with raised relay limits ([network-performance § Connections](network-performance.md#connections)) |

## Presence

[accepted] The region size, rates and timeout below were approved 2026-10-04.

- The world is divided into square regions of **64 m × 64 m**. The 4 km world
  then has about 63 × 63 regions.
- A client subscribes to the presence topic of its current region and its 8
  neighbours (`peerlings/v1/presence/<rx>_<ry>`). It changes subscriptions only
  once it is 8 m past a region border, so walking along a border doesn't cause
  constant resubscribing.
- A client publishes a **presence message** to its current region topic **4
  times per second while moving**, and a heartbeat **every 5 s when idle**.
  [accepted] With grid movement ([exploration § Grid movement](../gameplay/exploration.md#grid-movement))
  a moving player sends one message per step (3 per second, within the
  limit). Contents: tile, facing, step start time, display name, appearance
  hash, generator version, session and (in a PvP battle) battle ID; exact fields
  in [protocols](protocols.md#peerlingsv1presencerx_ry--presence). The message is
  signed by gossipsub with the player's key ([D-0018](../decisions/D-0018-one-key-per-player.md)).
  Receivers animate the step from the previous tile to the new one, so movement
  looks smooth without extra messages.
- [accepted] There is no chat; [emotes](../glossary.md#emote) are the only
  player-to-player messages. [accepted] An emote is sent as a presence message
  with an `emote` field (an ID from the fixed set in
  [multiplayer § Communication](../gameplay/multiplayer.md#communication)),
  rate-limited like other presence messages. Receivers ignore unknown emote IDs.
- A player not heard from for **15 s** is removed from view.
- Budget: a message is about 200 bytes, so 30 visible moving players cost about
  30 × 4 × 200 B ≈ 24 KB/s of download, which is fine on desktop.
- [accepted] The operator server also joins the topics, to help gossip reach
  browsers that have few direct peers.

## Direct protocols

| Protocol ID | Purpose | Canonical page |
|-------------|---------|----------------|
| `/peerlings/battle/1.0.0` | PvP battle session | [pvp-battles](../gameplay/pvp-battles.md) |
| `/peerlings/trade/1.0.0` | Trade session | [trading](../gameplay/trading.md) |
| `/peerlings/profile/1.0.0` | Request a player's public profile (team, created species) | [multiplayer](../gameplay/multiplayer.md) |
| `/peerlings/phone-backup/1.0.0` | Encrypted transfer of key and save between a computer and the player's phone | [player-data](player-data.md#phone-backup) |
| `/peerlings/save-backup/1.0.0` | Answer a `save-wanted` request with the snapshot CID and log heads of a held save backup | [player-data](player-data.md#keeping-saves-available) |

Exact message formats: [protocols](protocols.md).

## Requirements

- **NET-001** [accepted] Player-to-player communication MUST use the client's libp2p node; game logic MUST NOT depend on a game server (the operator server MAY relay transport traffic).
- **NET-002** [accepted] Presence MUST be distributed via pubsub topics scoped to world regions, so a client only receives nearby players.
- **NET-003** [accepted] Presence messages MUST be signed with the player's identity key and rate-limited by the sender; receivers MUST drop messages that are unsigned, too frequent or impossibly far from the previous position.
- **NET-004** [accepted] Battles, trades and profile requests MUST use versioned libp2p protocol IDs.

## Open questions

_None at the moment._

## See also

- [Architecture](architecture.md) · [Multiplayer](../gameplay/multiplayer.md)
