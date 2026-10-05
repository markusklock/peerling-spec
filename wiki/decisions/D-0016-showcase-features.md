---
title: "D-0016: Spectating, shareable links, device linking and a world feed"
type: decision
status: accepted
tags: [showcase, libp2p, ipns, pubsub, social]
sources:
  - raw/conversations/2026-10-05-showcase-features.md
related:
  - wiki/gameplay/spectating.md
  - wiki/gameplay/sharing.md
  - wiki/gameplay/world-feed.md
  - wiki/tech/player-data.md
  - wiki/tech/ipfs-showcase.md
updated: 2026-10-05
---

# D-0016: Spectating, shareable links, device linking and a world feed

**Status:** accepted (2026-10-05); details [proposed] ([Q-044](../open-questions.md#q-044))

## Context
The designer asked for more peer-to-peer and modern browser technology that adds
something meaningful to the game while showing off the technology.

## Decision
[accepted] Four features are added:
1. **Spectating PvP battles** over libp2p pubsub ([spectating](../gameplay/spectating.md)).
2. **Shareable Peerling cards and player profiles** that open outside the game,
   using IPNS and in-browser IPFS retrieval ([sharing](../gameplay/sharing.md)).
3. **Linking a device** with a short code, transferring the key over a direct,
   encrypted browser-to-browser connection
   ([player-data § Linking a device](../tech/player-data.md#linking-a-device)).
4. **A live world feed** of notable events over a world-wide pubsub topic
   ([world-feed](../gameplay/world-feed.md)).

## Consequences
- More pubsub topics and libp2p protocols ([realtime-networking](../tech/realtime-networking.md)).
- A small separate viewer web app, published on IPFS.
- Linked devices share one save, so only one device may play at a time.

## Alternatives considered
Battle replays as CIDs, the game rules as content-addressed WebAssembly,
scheduled events with drand timelock encryption, CAR-file export, achievements
as verifiable credentials, and WebGPU compute effects were offered and not
chosen.
