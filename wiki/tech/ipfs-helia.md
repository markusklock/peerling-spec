---
title: Browser IPFS Node (Helia)
type: system
status: draft
req_prefix: NODE
tags: [tech, ipfs, helia, libp2p]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/decisions/D-0003-browser-client-is-ipfs-node.md
  - wiki/decisions/D-0007-players-publish-assets.md
  - wiki/tech/architecture.md
  - wiki/tech/generation-server.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-04
---

# Browser IPFS Node (Helia)

> How each game client runs as an IPFS node in the browser using
> [Helia](../glossary.md#helia): connecting to the network, publishing the
> player's own creation, fetching and verifying content, serving content to
> other players, and caching.

## Role

[accepted] Each player is their own IPFS node and can upload and download data
to/from the IPFS network ([D-0003](../decisions/D-0003-browser-client-is-ipfs-node.md)).
[accepted] The player's node publishes the player's newly created Peerling
([D-0007](../decisions/D-0007-players-publish-assets.md)). [proposed] The same
libp2p node carries [realtime multiplayer traffic](realtime-networking.md).
[accepted] It also hosts the player's OrbitDB save log
([player-data](player-data.md)).

## Connectivity

Browsers cannot accept inbound TCP connections or dial plain TCP, so a browser
node needs browser-compatible transports and some help. [proposed]

- **Transports:** WebRTC (browser ↔ browser), WebTransport and secure
  WebSockets (browser ↔ server/public nodes).
- **Bootstrap:** the operator server's node is a bootstrap peer reachable over
  secure WebSockets and/or WebTransport, alongside optional public bootstrap
  nodes.
- **Relay:** the server acts as a libp2p circuit relay, so browsers can reach
  each other and upgrade to direct WebRTC connections.
- **Content routing:** browsers use delegated routing (the HTTP routing API) to
  find providers, since running a full DHT client in the browser is heavy.
- **Fallback:** if no peer delivers a block in time, the client MAY fetch from
  trustless HTTP gateways (including one run by the operator). Content is still
  verified by CID, so the trust model doesn't change.

## Content import parameters

[proposed] The server and the client must produce **identical CIDs** for the
same file, because the server computes the asset CIDs that go into the signed
species record and the client then imports the same files
([creation-pipeline § Stage 7](../peerlings/creation-pipeline.md#stage-7--publish)).
All game files are therefore imported with these fixed parameters, never with
library defaults:

| Parameter | Value |
|-----------|-------|
| CID version | 1 |
| Hash | sha2-256 |
| Assets (image, model, thumbnail) | UnixFS file, fixed-size chunker 1 MiB (1,048,576 bytes), raw leaves, balanced layout, max 1024 links per node, no wrapping directory |
| Species record | single DAG-CBOR block ([SPC-010](../peerlings/peerling-species.md#requirements)) |

## Content handling

[proposed]
- **Publishing:** during creation, the client adds its new species' record and
  assets, keeps them permanently, and provides them until the server has pinned
  them, and afterwards like any other content.
- **Verification:** all content is fetched by CID and verified locally before
  use.
- **Serving:** fetched blocks are stored in a persistent browser blockstore
  (IndexedDB) and provided to other peers, so popular Peerlings spread across
  player nodes.
- **Prefetching:** the client fetches assets it will likely need soon (e.g.
  species likely to appear in nearby biomes) ahead of time, so encounters don't
  wait on the network. See [encounters](../gameplay/encounters.md).
- **Eviction:** cache size is bounded; least-recently-used content is evicted,
  except the player's own species and the species of Peerlings in their
  collection.

## Requirements

- **NODE-001** [accepted] The client MUST run a Helia node that can both retrieve content from and provide content to the IPFS network.
- **NODE-002** [proposed] All content fetched from the network MUST be verified against its CID before use.
- **NODE-003** [proposed] The client MUST persist fetched blocks across sessions and keep serving them to peers while the game is open.
- **NODE-004** [proposed] The client MUST always retain (never evict) the records and assets of its own created species and of species in its collection.
- **NODE-005** [proposed] The client MUST connect to the operator server's node as a bootstrap/relay peer and SHOULD connect directly to other players via WebRTC when possible.
- **NODE-006** [proposed] The client MAY fall back to trustless HTTP gateways when peer retrieval times out.
- **NODE-007** [accepted] The client MUST add its newly created species record and assets to IPFS through its own node. [proposed] It MUST provide them at least until the server confirms pinning.
- **NODE-008** [proposed] Server and client MUST import game content with the fixed parameters in [Content import parameters](#content-import-parameters).

## Open questions

[Q-016](../open-questions.md#q-016)

## See also

- [OrbitDB registry](orbitdb-registry.md) · [Realtime networking](realtime-networking.md) · [IPFS showcase](ipfs-showcase.md)
