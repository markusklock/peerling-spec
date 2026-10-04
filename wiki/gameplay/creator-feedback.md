---
title: Creator Feedback
type: system
status: draft
req_prefix: CFB
tags: [gameplay, social, orbitdb, pubsub, showcase]
sources:
  - raw/conversations/2026-10-04-answers-round-7.md
related:
  - wiki/tech/player-data.md
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-showcase.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-04
---

# Creator Feedback

> How creators see what happens to their Peerlings out in the world: how often
> they're met, caught and traded, live notifications, and how many nodes keep
> them alive.

## Is it possible with IPFS?

Yes, and fairly easily. The server already sees the events that matter, and it
has already checked them:
- Every **catch** is verified by the server and recorded in the ownership
  ledger ([player-data § Verification](../tech/player-data.md#verification)).
- Every **encounter** is logged in the player's save log, which the server
  replicates.
- Every **trade** goes through the ownership ledger.

So the server can keep trustworthy counters per species and publish them with
OrbitDB and pubsub. Counters can't be inflated by fake clients, because only
verified events count. That answers the spam worry raised in Q-021.

[accepted] The feature is wanted (2026-10-04). The design below is [proposed].

## Design

[proposed]

### Species stats database
An OrbitDB keyvalue database written only by the server, keyed by species CID.
It is updated at most once per epoch (5 minutes). Each entry:

| Field | Meaning |
|-------|---------|
| `encounters` | How often players have met this species in the wild |
| `catches` | Verified catches |
| `owners` | Instances currently owned (not released) |
| `trades` | Trades involving this species |
| `providers` | Approximate number of IPFS nodes currently providing the species (from content-routing lookups) |

Any client can read any species' stats, so every species card can show them.
This is another way the game shows off OrbitDB.

### Live notifications
The server publishes a short message on the pubsub topic
`peerlings/v1/creator/<playerId>` whenever one of that player's species is
caught (after verification), traded, or delisted. A creator who is online sees
a toast such as *"Someone just caught your Mossnap!"*, with the catcher's
display name.

### While away
The client stores the stats it last showed. On the next login it compares them
with the current stats and shows a summary: *"Since you were last here, your
Mossnap was met 120 times and caught 42 times. It now lives on 37 nodes."*

### Where players see it
- A **My creations** screen listing the player's species with their stats.
- The stats on every species card ([ipfs-showcase](../tech/ipfs-showcase.md)).

## Requirements

- **CFB-001** [accepted] Creators MUST be able to see how their species are doing in the world.
- **CFB-002** [proposed] Species statistics MUST be published in a server-written OrbitDB database and MUST only count server-verified events.
- **CFB-003** [proposed] The server MUST notify online creators via a per-creator pubsub topic when their species is caught or traded.
- **CFB-004** [proposed] The client MUST show creators a summary of changes since their last session.

## See also

- [IPFS showcase](../tech/ipfs-showcase.md) · [Player data](../tech/player-data.md)
