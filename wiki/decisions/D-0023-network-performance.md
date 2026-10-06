---
title: "D-0023: Network performance failsafes, snapshots and indexes"
type: decision
status: accepted
tags: [tech, networking, performance, orbitdb]
sources:
  - raw/conversations/2026-10-06-network-performance.md
  - raw/conversations/2026-10-06-network-performance-approved.md
related:
  - wiki/tech/network-performance.md
  - wiki/gameplay/creator-feedback.md
  - wiki/tech/player-data.md
updated: 2026-10-06
---

# D-0023: Network performance failsafes, snapshots and indexes

**Status:** accepted (2026-10-06). Partly supersedes the species-stats design
of 2026-10-05 (CFB-002) and the full-registry-sync rule in
[player-data § Encounter seeds](../tech/player-data.md#encounter-seeds).

## Context
IPFS and OrbitDB can be slow. A review of every network request found steps
that can take minutes (first sync of large OrbitDB logs, verifying a player
by downloading their whole save log), a database that grows far too fast for
browsers (species stats as an OrbitDB keyvalue op-log), a possible 5-minute
wait for the first epoch record, relay limits that would cut PvP battles, and
wasteful encounter prefetching.

## Decision
[accepted] Adopt [network-performance](../tech/network-performance.md):
- local-first play; a retrieval ladder racing peers, the operator's trustless
  gateway and public gateways; request classes with timeouts; waiting states
  and placeholders;
- operator fast paths (HTTP) for epoch records, log downloads, checkpoints,
  IPNS records and creation uploads, each with a peer-to-peer fallback;
- signed snapshots and indexes instead of replicating growing logs: a registry
  index, hourly species stats snapshots (**replacing the species stats OrbitDB
  database**), an ownership index, epoch records by number, and **verification
  checkpoints** signed by the operator;
- candidate models prefetched one at a time; raised operator relay limits;
  dialling nearby players early.

## Consequences
- Verifiers trust the operator's checkpoint for the older part of a save log,
  as they already trust its listing signatures; full verification stays
  possible for anyone.
- The game uses four OrbitDB database kinds instead of five.
- New formats: epoch record fields, registry index, stats and ownership
  snapshots, checkpoints ([data-formats](../tech/data-formats.md)); new HTTP
  endpoints ([creation-api](../tech/creation-api.md)).

## Alternatives considered
- **Keep full OrbitDB replication everywhere:** simplest and most
  decentralized, but first syncs and verification would take minutes as the
  game grows.
- **Verify everything in full, with progress bars:** no extra trust, but
  trades and PvP with long-time players would be slow.
- **Serve everything over HTTP from the operator:** fast, but loses the
  peer-to-peer showcase and the offline resilience.
