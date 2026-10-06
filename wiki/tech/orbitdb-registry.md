---
title: Peerling Registry (OrbitDB)
type: system
status: accepted
req_prefix: REG
tags: [tech, orbitdb, ipfs, data]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-network-performance-approved.md
related:
  - wiki/decisions/D-0005-server-sole-registry-writer.md
  - wiki/peerlings/peerling-species.md
  - wiki/gameplay/encounters.md
  - wiki/decisions/D-0010-no-content-moderation.md
updated: 2026-10-06
---

# Peerling Registry (OrbitDB)

> The [registry](../glossary.md#registry) is the OrbitDB database that lists
> every published Peerling species. Clients replicate it peer-to-peer and draw
> wild encounters from it.

## Role

[accepted] An OrbitDB database holds all Peerlings; each new player's Peerling is
added to it. When exploring, players encounter random Peerlings from all
player-created ones, read from OrbitDB and downloaded via IPFS.

## Design

- [accepted] **Writers:** any player may append an entry, but every node only
  accepts entries that carry a valid server signature
  ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md); this replaced the earlier "server is the only writer" rule,
  [D-0005](../decisions/D-0005-server-sole-registry-writer.md)).
- [accepted] **Listing signature:** at the end of publishing, the server signs
  the entry's contents (species CID, `seq`, name, types, thumbnail CID,
  `createdAt`). The registry's OrbitDB access controller checks this signature
  on every entry, both when appending and when replicating, so invalid entries
  never spread. The player's browser appends the entry. If the entry hasn't
  appeared within a minute, the server appends the identical entry itself, so
  the sequence numbers never have gaps.

[accepted]
- **Database type:** an OrbitDB *documents* database keyed by
  species CID, so a client can look up and iterate entries cheaply.
- **Address:** one well-known database address, shipped with the client.
- **Entry contents** (illustrative; exact format:
  [data-formats § Registry entry](data-formats.md#registry-entry--peerlingslisting)): small and index-like — enough to choose an encounter
  without downloading the species record:

```json
{
  "_id": "bafy…species-cid",
  "listing": {
    "v": 1, "type": "peerlings/listing", "signer": "<operator ID>",
    "body": {
      "species": { "/": "bafy…species-cid" }, "seq": 1842,
      "name": "Mossnap", "nameKey": "mossnap", "types": ["Grass"],
      "thumbnail": { "/": "bafy…" }, "createdAt": 1791234567000,
      "status": "active"
    },
    "sig": { "/": { "bytes": "<Ed25519 signature>" } }
  }
}
```

- **Emergency delisting:** there is no content moderation
  ([D-0010](../decisions/D-0010-no-content-moderation.md)), but the operator
  can delist a species by appending a server-signed update to its entry with
  `"status": "removed"` (a tombstone). A tombstone takes the next `seq` of its
  own, and records it as `removedAtSeq`. [accepted] The server then unpins the
  species' **image, model and thumbnail**, but keeps the small **species
  record** pinned: old catches of the species stay verifiable by replay, which
  needs its stats and moves (2026-10-06).
- **Sequence numbers:** the server gives each new entry the next `seq` (1, 2,
  3, …) when it signs the listing. Together with `removedAtSeq`, this lets every client and the server
  agree exactly on which species were eligible at a given registry height
  ([player-data § Encounter seeds](player-data.md#encounter-seeds)).
- **Scale:** entries are small (a few hundred bytes), but a first sync of tens
  of thousands of entries still takes minutes, because OrbitDB fetches them
  block by block. [accepted] So a new client starts from the compact
  **registry index** and syncs the full registry in the background, or
  downloads it in one go from the server's log endpoint
  ([network-performance](network-performance.md#1-registry-index),
  [D-0023](../decisions/D-0023-network-performance.md)). Heavy assets are
  fetched by CID only when needed.

## Other OrbitDB databases

The registry is one of four kinds of OrbitDB database in the game:
- one **save log** per player, written by that player ([player-data](player-data.md));
- the **transfer log** of ownership changes, open to every player but
  accepting only correctly signed transfers ([player-data](player-data.md#transfer-log-and-trades));
- the **epoch log**, written only by the server ([player-data](player-data.md)).

[accepted] Species stats were a fifth, but are now published as hourly
snapshots instead ([creator-feedback](../gameplay/creator-feedback.md#species-stats-snapshot)).
Browsers don't replicate the epoch log or the transfer log in full either;
they read them through snapshots, indexes and the server's log endpoint
([network-performance § Scale](network-performance.md#scale)).

## Requirements

- **REG-001** [accepted] All published species MUST be listed in a single OrbitDB registry database.
- **REG-002** [accepted] Clients MUST read the registry to choose wild encounters and fetch the chosen species via IPFS.
- ~~**REG-003**~~ (removed 2026-10-04, replaced by REG-009; see D-0013)
- **REG-004** [accepted] Registry entries MUST contain enough summary data (types, name, thumbnail CID, status) to select encounters without fetching the full species record.
- **REG-005** [accepted] Clients MUST exclude entries whose status is `removed`.
- **REG-006** [accepted] The client MUST start with its locally persisted copy of the registry and sync in the background, so the game is usable before sync completes.
- **REG-007** [accepted] Each registry entry MUST carry a unique, gap-free sequence number `seq` assigned by the server; takedowns MUST record `removedAtSeq`.
- **REG-008** [accepted] The operator MUST be able to delist a species with a tombstone entry, and clients MUST honor it (REG-005).
- **REG-009** [accepted] Any player MAY append registry entries, but nodes MUST accept (and replicate) only entries carrying a valid server listing signature.
- **REG-010** [accepted] A tombstone MUST take its own new `seq`; after delisting, the server MUST keep the species record pinned and MAY unpin its assets.

## Open questions

_None at the moment._

## See also

- [Peerling species & data model](../peerlings/peerling-species.md)
- [Encounters](../gameplay/encounters.md)
