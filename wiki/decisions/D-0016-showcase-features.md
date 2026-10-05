---
title: "D-0016: Spectating, shareable links, phone backup and a world feed"
type: decision
status: accepted
tags: [showcase, libp2p, ipns, pubsub, social]
sources:
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
  - raw/conversations/2026-10-05-phone-backup-approved.md
related:
  - wiki/gameplay/spectating.md
  - wiki/gameplay/sharing.md
  - wiki/gameplay/world-feed.md
  - wiki/tech/player-data.md
  - wiki/tech/ipfs-showcase.md
updated: 2026-10-05
---

# D-0016: Spectating, shareable links, phone backup and a world feed

**Status:** accepted (2026-10-05); details of 1, 2 and 4 approved the same day
([Q-044](../open-questions.md#q-044)); phone backup details approved the same day
([Q-045](../open-questions.md#q-045))

## Context
The designer asked for more peer-to-peer and modern browser technology that adds
something meaningful to the game while showing off the technology.

## Decision
[accepted] Four features are added:
1. **Spectating PvP battles** over libp2p pubsub ([spectating](../gameplay/spectating.md)).
2. **Shareable Peerling cards and player profiles** that open outside the game,
   using IPNS and in-browser IPFS retrieval ([sharing](../gameplay/sharing.md)).
3. **Phone backup** by QR code: the player's phone fetches the key and save, to
   restore them on another computer later
   ([player-data § Phone backup](../tech/player-data.md#phone-backup)). This
   replaced the first idea, linking a second computer with a short code
   (designer, 2026-10-05).
4. **A live world feed** of notable events over a world-wide pubsub topic
   ([world-feed](../gameplay/world-feed.md)).

## Consequences
- More pubsub topics and libp2p protocols ([realtime-networking](../tech/realtime-networking.md)).
- A small separate viewer web app, published on IPFS.
- An account restored on a second computer shares one save with the first, so
  only one computer may play at a time.
- The Peerlings Viewer must work on phones (for the phone backup), although the
  game itself is desktop-only.

## Alternatives considered
Battle replays as CIDs, the game rules as content-addressed WebAssembly,
scheduled events with drand timelock encryption, CAR-file export, achievements
as verifiable credentials, and WebGPU compute effects were offered and not
chosen.
