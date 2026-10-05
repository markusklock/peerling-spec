---
title: IPFS Showcase Features
type: concept
status: draft
req_prefix: SHOW
tags: [tech, ipfs, ux, goals]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
related:
  - wiki/overview.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-05
---

# IPFS Showcase Features

> Showing off IPFS is one of the game's two equal goals
> ([overview](../overview.md#goals)). This page collects the ways the game makes
> the peer-to-peer technology *visible and meaningful* to players, without
> getting in the way of the fun.

## Principles

[accepted]
- Show the tech where it **means something to the player** ("your Peerling is
  being served to 3 other players right now") rather than as raw jargon.
- Let curious players drill down to the real thing (CIDs, peer IDs, gateway
  links), and keep it out of the way for everyone else.

## Ideas

[accepted] All of the following (approved 2026-10-05):

| Idea | What the player sees |
|------|----------------------|
| Network panel | Connected peers, data downloaded/served, number of Peerlings seeded by you |
| Species card CID | Every species card shows its CID with a "view on IPFS" link to a public gateway |
| "Fetched from a peer" moment | When a wild Peerling loads, a subtle indicator of where it came from (server vs. another player) |
| Creator pride | "Your Peerling now lives on N nodes" and live catch notifications ([creator-feedback](../gameplay/creator-feedback.md)) |
| Players found peer-to-peer | Other players appear in the world via libp2p pubsub, with no game server; a debug overlay can show the direct WebRTC connection to a nearby player |
| "You published this" | During onboarding the player watches their own node add their creation and the server pin it from them |
| Shareable links | Peerling cards and player profiles open outside the game, loaded from IPFS/IPNS in the visitor's browser ([sharing](../gameplay/sharing.md)) |
| Spectating and world feed | Battles and world events spread peer-to-peer over pubsub ([spectating](../gameplay/spectating.md), [world-feed](../gameplay/world-feed.md)) |
| Phone backup | Scan a QR code and your phone becomes an IPFS node for a moment, carrying your save as a CAR file ([player-data](player-data.md#phone-backup)) |
| Verified badge | A visible check that content was verified against its CID and the species attestation |

## Requirements

- **SHOW-001** [accepted] The game MUST provide an optional in-game view of the player's IPFS node activity (peers, data served, content held).
- **SHOW-002** [accepted] Each species MUST display its CID somewhere accessible in the UI.

## Open questions

_None at the moment._
