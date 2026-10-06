---
title: Shareable Peerling Cards and Player Profiles
type: system
status: accepted
req_prefix: LNK
tags: [social, ipfs, ipns, showcase, sharing]
sources:
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-v1-fun-features.md
related:
  - wiki/decisions/D-0016-showcase-features.md
  - wiki/tech/ipfs-showcase.md
  - wiki/tech/player-data.md
  - wiki/peerlings/peerling-species.md
updated: 2026-10-06
---

# Shareable Peerling Cards and Player Profiles

> Every Peerling species and every player has a permanent link that opens
> outside the game, in any browser, loaded straight from IPFS and checked
> against its CID: good for showing creations to friends and on social media.

[accepted] Peerling cards and player profiles can be shared as links that open
outside the game ([D-0016](../decisions/D-0016-showcase-features.md)).

## Design

[accepted] Approved 2026-10-05.

### The viewer

- A small, separate static web app, the **Peerlings Viewer**, published on IPFS
  as a directory and reachable through DNSLink on the operator's domain (e.g.
  `view.<domain>`).
- It fetches everything with `@helia/verified-fetch` in the visitor's browser,
  so each byte is checked against its CID. It renders the 3D model with the
  same stack as the game (WebGPU with WebGL2 fallback).
- The Viewer also holds the **phone backup** feature: on a phone it can scan
  the computer's QR code and carry the player's key and save
  ([player-data § Phone backup](../tech/player-data.md#phone-backup)), so it
  must work in mobile browsers.
- The same links also work through the IPFS **Service Worker Gateway**
  (`inbrowser.link`), which loads and verifies content from IPFS in a service
  worker. They also work as native `ipfs://` / `ipns://` links in IPFS-aware
  browsers, so the viewer doesn't depend on the operator's web server.

### Peerling card

- Link: `https://view.<domain>/#/peerling/<species CID>`.
- Shows the rotatable 3D model, the image, name, types, base stats, moves, lore,
  creator, and, if available, the [species stats](creator-feedback.md) (met,
  caught, nodes holding it). Plus the CID and a verified badge.
- [accepted] *"First found in the wild by …"* once someone has found it
  ([creator-feedback § First found in the wild](creator-feedback.md#first-found-in-the-wild)).

### Player profile

- Link: `https://view.<domain>/#/player/<IPNS name>`. The **IPNS name** is
  derived from the player's identity key, so it never changes.
- The client publishes a small **profile document**: display name, appearance,
  team summary, created species, Peerdex counts, guardian badges and PvP
  counters (exact fields:
  [data-formats § Player profile document](../tech/data-formats.md#player-profile-document--peerlingsprofile)). It is updated with each save snapshot, and an IPNS
  record signed by the player's key points to the latest version.
- IPNS records are published from the browser through delegated routing.
  IPNS records must be republished regularly to stay findable, so the client
  republishes on every session. The operator server stores and republishes the
  latest signed record for every player, which it can do without the player's
  key. If both stop, an old profile link stops resolving until the player
  plays again.

### In the game

- Every species card and the player's own profile have a **Share** button that
  copies the link.

## Requirements

- **LNK-001** [accepted] Players MUST be able to share links to Peerling cards and player profiles that open outside the game.
- **LNK-002** [accepted] Links MUST open in a separate viewer web app published on IPFS (with DNSLink), which fetches and verifies all content by CID in the visitor's browser.
- **LNK-003** [accepted] A player's profile MUST be published as a profile document behind an IPNS name derived from their identity key; the client MUST republish it each session and the server MUST republish the latest signed record.

## Open questions

_None at the moment._

## See also

- [IPFS showcase](../tech/ipfs-showcase.md) · [Creator feedback](creator-feedback.md)
