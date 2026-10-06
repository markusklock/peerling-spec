---
title: Browser IPFS Node (Helia)
type: system
status: accepted
req_prefix: NODE
tags: [tech, ipfs, helia, libp2p]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-tech-stack-2.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-05-asset-budgets-request.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-06-review-decisions.md
related:
  - wiki/decisions/D-0003-browser-client-is-ipfs-node.md
  - wiki/decisions/D-0007-players-publish-assets.md
  - wiki/tech/architecture.md
  - wiki/tech/generation-server.md
  - wiki/tech/realtime-networking.md
  - wiki/tech/tech-stack.md
  - wiki/decisions/D-0011-modern-web-platform-first.md
updated: 2026-10-06
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
([D-0007](../decisions/D-0007-players-publish-assets.md)). [accepted] The same
libp2p node carries [realtime multiplayer traffic](realtime-networking.md).
[accepted] It also hosts the player's OrbitDB save log
([player-data](player-data.md)).

## Connectivity

Browsers can't accept inbound connections or dial plain TCP or UDP, so a
browser node needs browser-compatible transports and some help from the
server. Technology choices follow
[D-0011](../decisions/D-0011-modern-web-platform-first.md) and
[tech-stack § Networking](tech-stack.md#networking).

- **Transports:** [accepted] WebTransport for browser → server; no WebSockets.
  [accepted] WebRTC-direct as the fallback browser → server transport.
  [accepted] WebRTC for browser ↔ browser.
- **IPv6:** [accepted] preferred wherever available; see
  [tech-stack § IPv6](tech-stack.md#ipv6).
- **Bootstrap:** [accepted] the operator server's node is the bootstrap peer,
  reachable over WebTransport and WebRTC-direct. The client looks up the
  server's current multiaddrs at startup
  ([tech-stack § WebTransport certificates](tech-stack.md#webtransport-certificates)).
  Public IPFS bootstrap nodes that support browser transports, and peers the
  client has met before, are also used so the game works when the server is
  offline ([resilience](resilience.md)).
- **Relay:** [accepted] the server acts as a libp2p circuit relay, so browsers can reach
  each other and upgrade to direct WebRTC connections.
- **Content routing:** [accepted] browsers use delegated routing (the HTTP routing API) to
  find providers, since running a full DHT client in the browser is heavy.
- **Fallback:** [accepted] if no peer delivers a block in time, the client MAY fetch from
  trustless HTTP gateways (including one run by the operator). Content is still
  verified by CID, so the trust model doesn't change.

## Content import parameters

[accepted] The server and the client must produce **identical CIDs** for the
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

[accepted]
- **Publishing:** during creation, the client adds its new species' record and
  assets, keeps them permanently, and provides them until the server has pinned
  them, and afterwards like any other content.
- **Verification:** all content is fetched by CID and verified locally before
  use.
- **Serving:** fetched blocks are stored in a persistent browser blockstore
  (OPFS, [tech-stack](tech-stack.md#client-runtime)) and provided to other peers, so popular Peerlings spread across
  player nodes.
- **Prefetching:** the client fetches assets it will likely need soon (e.g.
  species likely to appear in nearby biomes) ahead of time, so encounters don't
  wait on the network. See [encounters](../gameplay/encounters.md).
- **Eviction:** cache size is bounded ([tech-stack § Asset budgets](tech-stack.md#asset-budgets)); least-recently-used content is evicted,
  except the player's own species and the species of Peerlings in their
  collection.

## Requirements

- **NODE-001** [accepted] The client MUST run a Helia node that can both retrieve content from and provide content to the IPFS network.
- **NODE-002** [accepted] All content fetched from the network MUST be verified against its CID before use.
- **NODE-003** [accepted] The client MUST persist fetched blocks across sessions and keep serving them to peers while the game is open.
- **NODE-004** [accepted] The client MUST always retain (never evict) the records and assets of its own created species and of species in its collection.
- **NODE-005** [accepted] The client MUST connect to the operator server's node as a bootstrap/relay peer (over WebTransport or WebRTC-direct) and SHOULD connect directly to other players via WebRTC when possible.
- **NODE-006** [accepted] The client MAY fall back to trustless HTTP gateways when peer retrieval times out.
- **NODE-007** [accepted] The client MUST add its newly created species record and assets to IPFS through its own node. [accepted] It MUST provide them at least until the server confirms pinning.
- **NODE-008** [accepted] Server and client MUST import game content with the fixed parameters in [Content import parameters](#content-import-parameters).

## Open questions

- [Q-056](../open-questions.md#q-056) — network performance, scale and timeouts ([network-performance](../tech/network-performance.md))

## See also

- [OrbitDB registry](orbitdb-registry.md) · [Realtime networking](realtime-networking.md) · [IPFS showcase](ipfs-showcase.md)
