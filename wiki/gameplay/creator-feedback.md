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
  - raw/conversations/2026-10-06-network-performance-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-06-review-2-decisions.md
  - raw/conversations/2026-10-28-medium-low-resolutions.md
related:
  - wiki/tech/player-data.md
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-showcase.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-28
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
IPFS snapshots and pubsub. Only events that pass verification count, so catch,
owner and trade counts can't be inflated by fake clients. That answers the spam
worry raised in Q-021. [accepted] Encounter counts are checked too, but a
modified client can push them up by logging fast fled encounters, an accepted
gap ([player-data § Known gaps](../tech/player-data.md#known-gaps-accepted-risks);
corrected 2026-10-28).

[accepted] The feature is wanted (2026-10-04). The design below was approved 2026-10-05.

## Design

[accepted]

### Species stats snapshot
[accepted] (2026-10-06, [D-0023](../decisions/D-0023-network-performance.md);
replaces the earlier OrbitDB keyvalue database, whose op-log would grow far too
fast for browsers.) The server publishes the stats every 12 epochs (1 hour) as
a signed, sharded snapshot; the epoch record names its root (`statsRoot`), and
a species card fetches only the shard holding its species
([network-performance § Species stats snapshot](../tech/network-performance.md#2-species-stats-snapshot),
[data-formats § Snapshots and indexes](../tech/data-formats.md#snapshots-and-indexes)).
Each species' stats:

| Field | Meaning |
|-------|---------|
| `encounters` | How often players have met this species in the wild |
| `catches` | Verified catches |
| `owners` | Instances currently owned (not released) |
| `trades` | Trades involving this species |

[accepted] **How they are counted** (2026-10-28, [D-0028](../decisions/D-0028-medium-low-review-fixes.md)):
- `encounters`: `encounter` events of wild encounters whose tile, epoch and
  candidate the server has checked (as for a catch, without a battle replay).
- `trades`: one per valid trade entry that moves at least one Peerling of the
  species, however many it moves.
- Each hourly snapshot counts from the transfer log as it then stands, so a
  trade that becomes void ([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades))
  drops out of `trades`, and `owners` follows the restored owners.
| `providers` | Approximate number of IPFS nodes currently providing the species (from content-routing lookups) |

Any client can read any species' stats, so every species card can show them.

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
  CID of the `catch` save-log entry. [accepted] Once the server has named a
  finder, the title is **final**: a catch made offline and synced later
  doesn't take it, even if its epoch is earlier (2026-10-06).
- [accepted] **When it is decided** (2026-10-28, [D-0028](../decisions/D-0028-medium-low-review-fixes.md)): at each
  hourly stats snapshot, the server looks at every wild catch of the species
  it has verified by then and applies the rule above to them. The first
  snapshot that finds any catch names the finder, so within that hour the
  epoch decides, not the order in which catches arrived.
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
- ~~**CFB-002**~~ (removed 2026-10-06, replaced by CFB-008; see D-0023)
- **CFB-003** [accepted] The server MUST notify online creators via a per-creator pubsub topic when their species is caught, traded or delisted, or first found in the wild.
- **CFB-004** [accepted] The client MUST show creators a summary of changes since their last session.
- **CFB-005** [accepted] Each species card MUST show "First found in the wild by …" naming the first player with a verified wild catch of it, and the world feed MUST announce it.
- **CFB-006** [accepted] The first wild finder MUST be determined and announced as in [First found in the wild](#first-found-in-the-wild).
- **CFB-007** [accepted] A species' creator MUST NOT be credited as its first wild finder.
- **CFB-008** [accepted] Species statistics MUST be published by the server as the hourly species stats snapshot and MUST only count events that pass verification.
- **CFB-009** [accepted] Encounters, trades and the first wild finder MUST be counted as defined in [How they are counted](#species-stats-snapshot) and [First found in the wild](#first-found-in-the-wild).

## Open questions

_None at the moment._

## See also

- [IPFS showcase](../tech/ipfs-showcase.md) · [Player data](../tech/player-data.md)
