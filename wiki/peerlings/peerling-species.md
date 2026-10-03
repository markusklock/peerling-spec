---
title: Peerling Species and Instances (Data Model)
type: data
status: draft
req_prefix: SPC
tags: [peerlings, data-model, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/decisions/D-0006-species-vs-instance.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/peerlings/types.md
  - wiki/peerlings/moves.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-03
---

# Peerling Species and Instances (Data Model)

> Canonical definition of the data that describes a Peerling: the immutable
> **species record** stored on IPFS, the base **stats**, and the per-player
> **instance** stored in a save.

The split between species and instance is proposed in
[D-0006](../decisions/D-0006-species-vs-instance.md).

## Species record

[accepted] The species data (description, type, attacks, …) and its 3D model
are stored on IPFS. [proposed] The species record is a JSON document (encoded
as DAG-JSON or DAG-CBOR so asset links are real IPLD links); its CID is the
species' identity. Illustrative shape:

```json
{
  "schema": "peerlings/species@1",
  "name": "Mossnap",
  "summary": "A sleepy fox made of moss that carries a glowing lantern.",
  "lore": "…",
  "types": ["Grass"],
  "baseStats": { "hp": 70, "attack": 45, "defense": 60, "speed": 50, "special": 75 },
  "moves": [
    { "template": "strike-basic", "name": "Moss Swipe", "type": "Grass", "description": "…" }
  ],
  "assets": {
    "image":     { "/": "bafy…" },
    "model":     { "/": "bafy…" },
    "thumbnail": { "/": "bafy…" }
  },
  "creator": { "id": "<player public key / DID>", "displayName": "…" },
  "createdAt": "2026-10-03T12:00:00Z",
  "provenance": {
    "wish": "a small sleepy fox made of moss that carries a lantern",
    "concept": { "…": "…" },
    "imagePrompt": { "…": "…" },
    "models": { "concept": "<name@version>", "image": "<name@version>", "to3d": "<name@version>" },
    "seeds": { "image": 1234567 }
  },
  "attestation": { "signer": "<server key id>", "signature": "…" }
}
```

Field details (lengths, allowed characters, exact encoding, signature scheme)
are still to be specified.

## Stats

[proposed] Every species has five base stats: **HP**, **Attack**, **Defense**,
**Special**, **Speed**. How stats are used in damage and turn order is defined
in [battle](../gameplay/battle.md).

[proposed] *Fair by construction* ([overview](../overview.md#design-pillars)):
every species gets the **same base-stat total**, distributed by the LLM to fit
the concept (a turtle gets Defense, a cheetah gets Speed) within per-stat
minimum and maximum bounds. Budget and bounds: TBD, see
[Q-009](../open-questions.md#q-009).

## Peerling instance

[proposed] An instance is one individual Peerling owned by a player, stored in
the player's save (not on the shared registry). Illustrative shape:

```json
{
  "instanceId": "<random uuid>",
  "species": { "/": "bafy…species-cid" },
  "nickname": null,
  "level": 5,
  "xp": 0,
  "currentHp": 22,
  "caughtAt": "2026-10-03T12:30:00Z",
  "origin": "starter | wild"
}
```

Progression (levels, XP, evolution): [Q-010](../open-questions.md#q-010).

## Requirements

- **SPC-001** [accepted] A species' description, type, attacks and 3D model MUST be stored on IPFS.
- **SPC-002** [proposed] A species MUST be identified by the CID of its species record.
- **SPC-003** [proposed] A species record MUST reference its image, 3D model and thumbnail by CID.
- **SPC-004** [proposed] A species record MUST carry a `schema` version string; clients MUST ignore records with unknown major versions rather than fail.
- **SPC-005** [proposed] A species' types MUST satisfy TYP-001 and TYP-002 in [types](types.md#requirements).
- **SPC-006** [proposed] A species MUST have exactly 4 moves ([moves](moves.md)).
- **SPC-007** [proposed] Base stats MUST sum to the fixed stat budget and each stat MUST lie within its bounds.
- **SPC-008** [proposed] Clients MUST verify the attestation of a species record before using it.
- **SPC-009** [proposed] A Peerling instance MUST reference its species by CID and MUST NOT copy species data.

## Open questions

[Q-009](../open-questions.md#q-009) ·
[Q-010](../open-questions.md#q-010) ·
[Q-016](../open-questions.md#q-016) ·
[Q-018](../open-questions.md#q-018)

## See also

- [Creation pipeline](creation-pipeline.md)
- [Registry](../tech/orbitdb-registry.md)
