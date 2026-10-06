---
title: Playing Without the Operator Server
type: system
status: accepted
req_prefix: RES
tags: [tech, resilience, ipfs, libp2p, decentralization]
sources:
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-peer-save-backups.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-network-performance-approved.md
related:
  - wiki/tech/architecture.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/generation-server.md
  - wiki/tech/player-data.md
  - wiki/gameplay/encounters.md
updated: 2026-10-06
---

# Playing Without the Operator Server

> What keeps working when the operator server is offline, and the distributed
> technology that makes it possible. The goal is to show that Peerlings is
> really peer-to-peer: the server makes things better, but the game shouldn't
> need it.

## Goal

[accepted] Use as much distributed technology as possible, so the game keeps
working when the operator server is offline. It's acceptable that some Peerlings
can't be downloaded then: the server is the only node with every Peerling
pinned. [accepted] Encounters use several candidates, so a missing species
doesn't block the game ([encounters § Candidates](../gameplay/encounters.md#candidates)).

## What works when the server is offline

[accepted]

| Feature | Without the server? | How |
|---------|:------------------:|-----|
| Loading the game | Yes | Installed PWA cache; the game app itself is also published on IPFS (see below) |
| Exploring, presence, emotes | Yes, if peers can connect | Pubsub between players; connections via public relays and peers already connected |
| Wild encounters and battles | Yes | Client-derived epoch records ([player-data § Encounter seeds](player-data.md#encounter-seeds)); species fetched from other players' nodes; 5 candidates per encounter |
| Catching | Yes | Anyone can verify a catch by replaying it (SAVE-016) |
| PvP battles | Yes | Peer-to-peer; each side verifies the other's Peerlings itself |
| Registry updates | No new species (creation needs the server) | The existing registry replicates between players; entries are signed, so any player can serve them |
| Trades | Yes | Signed transfer chains in the open transfer log ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)) |
| Creation, Creation Shrine, new players | No | GPU models and attestations live on the server |
| Creator stats | Frozen | Resume when the server is back |

## Distributed building blocks

[accepted]

### 1. Every player is a provider
Each browser keeps and serves the species it has met, caught or created
([ipfs-helia](ipfs-helia.md#content-handling)). Popular species therefore live
on many player nodes, and the 5-candidate rule makes it very likely that at
least one candidate is available somewhere. Species can be fetched directly
from connected peers via Bitswap, without any content-routing service.

### 2. Not depending on the server to find peers
- The client stores the addresses of peers it has met and reconnects to them
  on its own.
- Besides the operator server, the client's bootstrap list includes public IPFS
  bootstrap nodes that accept browser transports (WebTransport or
  WebRTC-direct). The current list must be checked at implementation time.
- Browser-to-browser WebRTC needs a relay to set up the connection. Public
  libp2p nodes (e.g. Kubo nodes with the default limited relay service) can
  serve as relays. Connections that are already open stay up when the server
  goes down.
- Content routing falls back to a public delegated routing endpoint
  (e.g. `https://delegated-ipfs.dev/routing/v1`).

### 3. Randomness without the server
Encounter seeds use drand randomness, which the client can fetch from public
drand endpoints when the server isn't publishing epoch records
([player-data § Encounter seeds](player-data.md#encounter-seeds)).

### 4. The game app on IPFS
The game client (HTML, JavaScript, authored assets) is published to IPFS as a
directory with its own CID, and the server's domain points to it with DNSLink.
Players can also load the game through any public IPFS gateway, or from the
installed PWA cache. If the operator's web server is down, the game still
loads.

### 5. Community mirrors (optional, not expected)

[accepted] The designer doesn't expect anyone to run mirrors (2026-10-05), so
nothing in the design relies on them; the option below simply stays open.

Anyone can help keep every Peerling available. The operator publishes the full
pinset (every CID the registry references) as an IPFS Cluster that others can
follow with `ipfs-cluster-follow`, or as a simple list volunteers can pin. Each
mirror is another always-on node with every Peerling.

### 6. Fast paths are only shortcuts
[accepted] The operator's HTTP fast paths, snapshots, indexes and checkpoints
make the game fast while the server is online. Each has a peer-to-peer slow
path, so everything in the table above still works without the server, just
slower ([network-performance](network-performance.md#fast-paths-through-the-operator)).

### 7. No catching up needed
Catches, trades and ownership are checked by players themselves
([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)), so
nothing waits for the server. When it is back, it simply resumes signing new
species, publishing epoch records and updating creator stats.

## Requirements

- **RES-001** [accepted] The game MUST keep working, as far as possible, while the operator server is offline: exploration, wild encounters, battles, catching, catch verification, PvP and trades MUST NOT require it.
- **RES-002** [accepted] Clients MUST persist known peers and use public bootstrap nodes, relays and delegated routing in addition to the operator server.
- **RES-003** [accepted] The game client MUST be published on IPFS (with DNSLink) and installable as a PWA, so it loads without the operator's web server.
- **RES-004** [accepted] The operator SHOULD publish the full pinset so others can run mirror nodes (e.g. IPFS Cluster followers).

## Open questions

_None at the moment._

## See also

- [Architecture](architecture.md) · [Tech stack](tech-stack.md) · [Encounters § Candidates](../gameplay/encounters.md#candidates)
