---
title: Peerling Species and Instances (Data Model)
type: data
status: draft
req_prefix: SPC
tags: [peerlings, data-model, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
related:
  - wiki/decisions/D-0006-species-vs-instance.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/peerlings/types.md
  - wiki/peerlings/moves.md
  - wiki/tech/orbitdb-registry.md
  - wiki/gameplay/trading.md
updated: 2026-10-04
---

# Peerling Species and Instances (Data Model)

> Canonical definition of the data that describes a Peerling: the immutable
> **species record** stored on IPFS, the base **stats**, and the per-player
> **instance** stored in a save.

The split between species and instance is proposed in
[D-0006](../decisions/D-0006-species-vs-instance.md).

## Species record

[accepted] The species data (description, type, attacks, …) and its 3D model
are stored on IPFS. [proposed] The species record is encoded as **DAG-CBOR**,
so the same data always produces the same bytes and therefore the same CID
(the server and the player's browser must agree on the CID during
[publishing](creation-pipeline.md#stage-7--publish)). Asset references are
real IPLD links. Its CID is the species' identity. Illustrative shape, shown as
JSON for readability:

```json
{
  "schema": "peerlings/species@1",
  "name": "Mossnap",
  "summary": "A sleepy fox made of moss that carries a glowing lantern.",
  "lore": "…",
  "types": ["Grass"],
  "baseStats": { "hp": 80, "attack": 55, "defense": 70, "speed": 45 },
  "moves": [
    { "slot": "quick", "template": "quick-jab", "name": "Moss Swipe", "type": "Normal", "description": "…" }
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

Field details (lengths, allowed characters, the exact bytes the attestation
signs) are still to be specified. The move entries follow
[moves § Move structure](moves.md#move-structure).

## Stats

[proposed] Every species has four base stats: **HP**, **Attack**, **Defense**,
**Speed**. (An earlier draft had a fifth "Special" stat. It was dropped to keep
battles simple and to avoid confusion with the *signature* (special) move slot
in [moves](moves.md).) How stats are used in damage and turn order is defined
in [battle](../gameplay/battle.md).

[accepted] *Fair by construction* ([overview](../overview.md#design-pillars)):
every species has the **same base-stat total**. [accepted] The LLM spreads
that total across the stats to fit the concept: a turtle gets Defense, a
cheetah gets Speed. [proposed] Each stat has a minimum and a maximum, so
no species has a useless stat or an extreme one.

### Suggested numbers [proposed]

Suggested on 2026-10-04 at the designer's request; awaiting approval
([Q-023](../open-questions.md#q-023)). They assume the damage model in
[battle § Suggested damage model](../gameplay/battle.md#suggested-damage-model-proposed).

| Rule | Value | Why |
|------|-------|-----|
| Base-stat total | **320** | Average 80 per stat, close to a typical fully grown Pokémon (≈ 500 over 6 stats), so the same formulas feel familiar |
| Minimum per stat | **40** | Even a turtle's Speed or a glass cannon's Defense still matters |
| Maximum per stat | **130** | Allows a clear specialty (a 130 stat is ~1.6× average) without making the other three stats useless |
| Step | **5** | Readable numbers; fewer near-identical spreads |

Example spreads (HP / Attack / Defense / Speed):

| Archetype | HP | Atk | Def | Spd |
|-----------|---:|----:|----:|----:|
| Balanced | 80 | 80 | 80 | 80 |
| Tank (turtle) | 100 | 60 | 120 | 40 |
| Glass cannon (cheetah) | 60 | 110 | 40 | 110 |
| Bruiser (bear) | 110 | 100 | 70 | 40 |
| Sweeper (falcon) | 70 | 90 | 50 | 110 |
| Wall (golem) | 130 | 50 | 100 | 40 |

The LLM picks a spread that fits the concept; the validator
([CRE-009](creation-pipeline.md#requirements)) enforces the total, bounds and
step.

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
  "origin": "starter | wild | trade",
  "originalOwner": "<player public key of whoever first obtained it>"
}
```

Progression (levels, XP, evolution): [Q-010](../open-questions.md#q-010).
Instances can change owner through [trading](../gameplay/trading.md).

## Requirements

- **SPC-001** [accepted] A species' description, type, attacks and 3D model MUST be stored on IPFS.
- **SPC-002** [proposed] A species MUST be identified by the CID of its species record.
- **SPC-003** [proposed] A species record MUST reference its image, 3D model and thumbnail by CID.
- **SPC-004** [proposed] A species record MUST carry a `schema` version string; clients MUST ignore records with unknown major versions rather than fail.
- **SPC-005** [proposed] A species' types MUST satisfy TYP-001 and TYP-002 in [types](types.md#requirements).
- **SPC-006** [proposed] A species' move set MUST satisfy the move-set rules in [moves](moves.md#requirements).
- **SPC-007** [accepted] Every species MUST have the same base-stat total. [proposed] Each stat MUST lie within its bounds.
- **SPC-008** [proposed] Clients MUST verify the attestation of a species record before using it.
- **SPC-009** [proposed] A Peerling instance MUST reference its species by CID and MUST NOT copy species data.
- **SPC-010** [proposed] Species records MUST be encoded as DAG-CBOR so the encoding, and therefore the CID, is deterministic.

## Open questions

[Q-010](../open-questions.md#q-010) ·
[Q-016](../open-questions.md#q-016) ·
[Q-018](../open-questions.md#q-018) ·
[Q-023](../open-questions.md#q-023)

## See also

- [Creation pipeline](creation-pipeline.md)
- [Registry](../tech/orbitdb-registry.md)
