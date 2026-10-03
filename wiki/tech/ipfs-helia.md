---
title: Browser IPFS Node (Helia)
type: system
status: draft
req_prefix: NODE
tags: [tech, ipfs, helia, libp2p]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/decisions/D-0003-browser-client-is-ipfs-node.md
  - wiki/tech/architecture.md
  - wiki/tech/generation-server.md
updated: 2026-10-03
---

# Browser IPFS Node (Helia)

> How each game client runs as an IPFS node in the browser using
> [Helia](../glossary.md#helia): connecting to the network, fetching and
> verifying content, serving content to other players, and caching.

## Role

[accepted] Each player is their own IPFS node and can upload and download data
to/from the IPFS network ([D-0003](../decisions/D-0003-browser-client-is-ipfs-node.md)).

## Connectivity

Browsers cannot accept inbound TCP connections or dial plain TCP, so a browser
node needs browser-compatible transports and some help. [proposed]

- **Transports:** WebRTC (browser ↔ browser), WebTransport and secure
  WebSockets (browser ↔ server/public nodes).
- **Bootstrap:** the operator server's node is a bootstrap peer reachable over
  secure WebSockets and/or WebTransport, alongside optional public bootstrap
  nodes.
- **Relay:** the server acts as a libp2p circuit-relay so browsers can reach
  each other and upgrade to direct WebRTC connections.
- **Content routing:** browsers use delegated routing (HTTP routing API) to find
  providers, since running a full DHT client in the browser is heavy.
- **Fallback:** if no peer delivers a block in time, the client MAY fetch from
  trustless HTTP gateways (including one run by the operator) — still verified
  by CID, so trust is unchanged.

## Content handling

[proposed]
- All content is fetched by CID and **verified** locally before use.
- Fetched blocks are stored in a persistent browser blockstore (IndexedDB) and
  re-provided to other peers, so popular Peerlings spread across player nodes.
- The client **prefetches** assets it will likely need soon (e.g. species
  likely to appear in nearby biomes) so encounters don't wait on the network —
  see [encounters](../gameplay/encounters.md).
- Cache size is bounded; least-recently-used content is evicted, except the
  player's own species and the species of Peerlings in their collection.

## Requirements

- **NODE-001** [accepted] The client MUST run a Helia node that can both retrieve content from and provide content to the IPFS network.
- **NODE-002** [proposed] All content fetched from the network MUST be verified against its CID before use.
- **NODE-003** [proposed] The client MUST persist fetched blocks across sessions and keep serving them to peers while the game is open.
- **NODE-004** [proposed] The client MUST always retain (never evict) the records and assets of its own created species and of species in its collection.
- **NODE-005** [proposed] The client MUST connect to the operator server's node as a bootstrap/relay peer and SHOULD connect directly to other players via WebRTC when possible.
- **NODE-006** [proposed] The client MAY fall back to trustless HTTP gateways when peer retrieval times out.

## Open questions

[Q-003](../open-questions.md#q-003) · [Q-014](../open-questions.md#q-014) · [Q-016](../open-questions.md#q-016)

## See also

- [OrbitDB registry](orbitdb-registry.md) · [IPFS showcase](ipfs-showcase.md)
