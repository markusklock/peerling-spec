---
title: "D-0003: Every browser client is a Helia IPFS node"
type: decision
status: accepted
tags: [tech, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/tech/ipfs-helia.md
updated: 2026-10-03
---

# D-0003: Every browser client is a Helia IPFS node

**Status:** accepted (2026-10-03)

## Context
Showcasing IPFS is one of the game's two main goals. Fetching content through an
HTTP gateway would hide the technology.

## Decision
The game runs in the browser, and every client runs [Helia](../glossary.md#helia)
as a full IPFS node that both downloads and serves (uploads) game content.

## Consequences
- Clients need browser-compatible libp2p transports and help from the server
  for bootstrap/relay (see [ipfs-helia](../tech/ipfs-helia.md)).
- Content popular with players spreads across player nodes, reducing server load.
- Content fetched from untrusted peers is verified by CID.

## Alternatives considered
- Gateway-only HTTP fetching: simpler but defeats the showcase goal. May still be
  used as a fallback ([proposed], see [ipfs-helia](../tech/ipfs-helia.md)).
