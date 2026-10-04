---
title: Peerling Registry (OrbitDB)
type: system
status: draft
req_prefix: REG
tags: [tech, orbitdb, ipfs, data]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/decisions/D-0005-server-sole-registry-writer.md
  - wiki/peerlings/peerling-species.md
  - wiki/gameplay/encounters.md
  - wiki/peerlings/moderation.md
updated: 2026-10-04
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

- [accepted] **Writers:** only the generation server's identity can write
  ([D-0005](../decisions/D-0005-server-sole-registry-writer.md)). Clients are
  read-only replicas.

[proposed]
- **Database type:** an OrbitDB *documents* (or keyvalue) database keyed by
  species CID, so a client can look up and iterate entries cheaply.
- **Address:** one well-known database address, shipped with the client.
- **Entry contents:** small and index-like — enough to choose an encounter
  without downloading the species record:

```json
{
  "_id": "bafy…species-cid",
  "species": { "/": "bafy…species-cid" },
  "name": "Mossnap",
  "types": ["Grass"],
  "thumbnail": { "/": "bafy…" },
  "createdAt": "2026-10-03T12:00:00Z",
  "status": "active"
}
```

- **Takedown:** an update setting `"status": "removed"` acts as a tombstone
  ([moderation](../peerlings/moderation.md)).
- **Scale:** entries are small (a few hundred bytes), so even tens of thousands
  of species replicate quickly; heavy assets are fetched by CID only when
  needed.

## Other OrbitDB databases

The registry is one of three kinds of OrbitDB database in the game; the other
two are canonical in [player-data](player-data.md):
- one **save log** per player, written by that player;
- the **ownership ledger**, written only by the server.

## Requirements

- **REG-001** [accepted] All published species MUST be listed in a single OrbitDB registry database.
- **REG-002** [accepted] Clients MUST read the registry to choose wild encounters and fetch the chosen species via IPFS.
- **REG-003** [accepted] Only the generation server's identity MUST be able to write to the registry.
- **REG-004** [proposed] Registry entries MUST contain enough summary data (types, name, thumbnail CID, status) to select encounters without fetching the full species record.
- **REG-005** [proposed] Clients MUST exclude entries whose status is `removed`.
- **REG-006** [proposed] The client MUST start with its locally persisted copy of the registry and sync in the background, so the game is usable before sync completes.

## Open questions

[Q-007](../open-questions.md#q-007) ·
[Q-017](../open-questions.md#q-017) · [Q-021](../open-questions.md#q-021)

## See also

- [Peerling species & data model](../peerlings/peerling-species.md)
- [Encounters](../gameplay/encounters.md)
