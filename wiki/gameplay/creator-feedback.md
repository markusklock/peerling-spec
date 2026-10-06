---
title: Creator Feedback
type: system
status: accepted
req_prefix: CFB
tags: [gameplay, social, orbitdb, pubsub, showcase]
sources:
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-v1-fun-features.md
  - raw/conversations/2026-10-06-fun-features-approved.md
related:
  - wiki/tech/player-data.md
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-showcase.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-06
---

# Creator Feedback

> How creators see what happens to their Peerlings out in the world: how often
> they're met, caught and traded, live notifications, and how many nodes keep
> them alive.

## Is it possible with IPFS?

Yes, and fairly easily. The server already sees the events that matter, and it
can check them:
- Every **catch** is in the catcher's save log, which the server replicates
  and can verify by replay ([player-data § Verification](../tech/player-data.md#verification)).
- Every **encounter** is logged in the player's save log, which the server
  replicates.
- Every **trade** is in the open transfer log.

So the server can keep trustworthy counters per species and publish them with
OrbitDB and pubsub. Counters can't be inflated by fake clients, because only
events that pass verification count. That answers the spam worry raised in
Q-021.

[accepted] The feature is wanted (2026-10-04). The design below was approved 2026-10-05.

## Design

[accepted]

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

### First found in the wild

[accepted] The first player with a verified wild catch of a species is
credited on its card as **"First found in the wild by …"**, and the world
feed announces it ([D-0022](../decisions/D-0022-v1-fun-features.md)). The
wording makes clear that this player found it in the wild, not created it.
Every new species becomes a small race.

[accepted] Details (approved 2026-10-06):
- **Who decides:** the server, which already verifies every catch for the
  species stats. It records the finder in the species stats (`firstWild`,
  [data-formats § Species stats](../tech/data-formats.md#species-stats--peerlingsspecies-stats)).
- **What counts:** only wild catches (not starters, shrine creations or
  trades). [accepted] A species' creator can never earn its credit, even by
  catching it in the wild (confirmed by the designer): the credit is for
  finding someone else's creation.
- **First** = the verified catch with the lowest epoch; ties go to the lower
  CID of the `catch` save-log entry. A catch made offline and synced later
  can still take the title if its epoch is earlier; this is rare and
  accepted.
- **Checkable:** the stats name the `catch` entry's CID, so any client can
  verify the catch by replay.
- **Card:** *"First found in the wild by Mia · 2026-10-07"*
  ([sharing § Peerling card](sharing.md#peerling-card)).
- **Announcements:** once the stats name them, the finder's client posts a
  `first-found` world feed event ([world-feed](world-feed.md)) and shows
  *"You are the first to find Mossnap in the wild!"*. The creator gets
  *"Mia was the first to find your Mossnap in the wild!"* on their creator
  topic.
- While the server is offline, the credit appears once it is back.

### Where players see it
- A **My creations** screen listing the player's species with their stats.
- The stats on every species card ([ipfs-showcase](../tech/ipfs-showcase.md)).

## Requirements

- **CFB-001** [accepted] Creators MUST be able to see how their species are doing in the world.
- **CFB-002** [accepted] Species statistics MUST be published in a server-written OrbitDB database and MUST only count events that pass verification.
- **CFB-003** [accepted] The server MUST notify online creators via a per-creator pubsub topic when their species is caught, traded or delisted.
- **CFB-004** [accepted] The client MUST show creators a summary of changes since their last session.
- **CFB-005** [accepted] Each species card MUST show "First found in the wild by …" naming the first player with a verified wild catch of it, and the world feed MUST announce it.
- **CFB-006** [accepted] The first wild finder MUST be determined and announced as in [First found in the wild](#first-found-in-the-wild).
- **CFB-007** [accepted] A species\' creator MUST NOT be credited as its first wild finder.

## Open questions

- [Q-056](../open-questions.md#q-056) — network performance, scale and timeouts ([network-performance](../tech/network-performance.md))

## See also

- [IPFS showcase](../tech/ipfs-showcase.md) · [Player data](../tech/player-data.md)
