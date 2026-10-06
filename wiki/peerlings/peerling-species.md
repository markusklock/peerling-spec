---
title: Peerling Species and Instances (Data Model)
type: data
status: accepted
req_prefix: SPC
tags: [peerlings, data-model, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-approvals-q016-q038.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-proposals-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
related:
  - wiki/decisions/D-0006-species-vs-instance.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/peerlings/types.md
  - wiki/peerlings/moves.md
  - wiki/tech/orbitdb-registry.md
  - wiki/gameplay/trading.md
updated: 2026-10-06
---

# Peerling Species and Instances (Data Model)

> Canonical definition of the data that describes a Peerling: the immutable
> **species record** stored on IPFS, the base **stats**, and the per-player
> **instance** stored in a save.

[accepted] The split between species and instance is decided in
[D-0006](../decisions/D-0006-species-vs-instance.md).

## Species record

[accepted] The species data (description, type, attacks, …) and its 3D model
are stored on IPFS. [accepted] The species record is encoded as **DAG-CBOR**,
so the same data always produces the same bytes and therefore the same CID
(the server and the player's browser must agree on the CID during
[publishing](creation-pipeline.md#stage-7--publish)). Asset references are
real IPLD links. Its CID is the species' identity. Illustrative shape, shown as
JSON for readability (exact format:
[data-formats § Species record](../tech/data-formats.md#species-record--peerlingsspecies)):

```json
{
  "v": 1,
  "type": "peerlings/species",
  "signer": "<operator ID>",
  "body": {
  "name": "Mossnap",
  "summary": "A sleepy fox made of moss that carries a glowing lantern.",
  "lore": "…",
  "types": ["Grass"],
  "sizeClass": "small",
  "temperament": "Sleepy and gentle, wakes up for shiny things",
  "baseStats": { "hp": 80, "attack": 55, "defense": 70, "speed": 45 },
  "moves": [
    { "slot": "quick", "template": "quick-jab", "name": "Moss Swipe", "type": "Normal", "description": "…" },
    { "slot": "strong", "template": "strong-recoil", "name": "Lantern Slam", "type": "Grass", "description": "…" },
    { "slot": "signature", "template": "sig-weaken", "name": "Lantern Glare", "type": "Grass", "description": "…", "stat": "defense" }
  ],
  "assets": {
    "image":     { "/": "bafy…" },
    "model":     { "/": "bafy…" },
    "thumbnail": { "/": "bafy…" }
  },
  "creator": { "id": "12D3KooW…", "displayName": "…" },
  "createdAt": 1791234567000,
  "provenance": {
    "wish": "a small sleepy fox made of moss that carries a lantern",
    "concept": { "…": "…" },
    "imagePrompt": { "…": "…" },
    "models": { "concept": "<name@version>", "image": "<name@version>", "to3d": "<name@version>" },
    "seeds": { "image": 1234567 }
  }
  },
  "sig": { "/": { "bytes": "<Ed25519 signature>" } }
}
```

Field lengths, allowed values and the exact bytes the signature covers are in
[data-formats](../tech/data-formats.md#species-record--peerlingsspecies). The move entries follow
[moves § Move structure](moves.md#move-structure).

### Size and temperament

[accepted] Two presentation fields come from the
[concept](creation-pipeline.md#stage-2--concept) (approved 2026-10-06):

- **`sizeClass`** (`small`, `medium` or `large`): image-to-3D models come out
  at no particular scale, so the client scales every model to a fixed height
  for its class, in the world and in battle:

  | Size class | Model height |
  |------------|-------------:|
  | `small` | 0.6 tiles (1.2 m) |
  | `medium` | 1.0 tile (2 m) |
  | `large` | 1.4 tiles (2.8 m) |

- **`temperament`** (at most 60 characters): a short personality line shown on
  the Peerling card. It also sets the style of the idle animation (a calm
  Peerling bobs slowly, a lively one quickly,
  [battle § Presentation](../gameplay/battle.md#presentation)); how the
  client maps the text to an animation style is up to the implementer, as
  long as the same temperament always gives the same animation.

Neither field affects battles.

## Stats

[accepted] Every species has four base stats: **HP**, **Attack**, **Defense**,
**Speed**. (An earlier draft had a fifth "Special" stat. It was dropped to keep
battles simple and to avoid confusion with the *signature* (special) move slot
in [moves](moves.md).) How stats are used in damage and turn order is defined
in [battle](../gameplay/battle.md).

[accepted] *Fair by construction* ([overview](../overview.md#design-pillars)):
every species has the **same base-stat total**. [accepted] The LLM spreads
that total across the stats to fit the concept: a turtle gets Defense, a
cheetah gets Speed. [accepted] Each stat has a minimum and a maximum, so
no species has a useless stat or an extreme one.

### Stat numbers

[accepted] Approved 2026-10-04 together with the
[damage model](../gameplay/battle.md#damage-model).

| Rule | Value | Why |
|------|-------|-----|
| Base-stat total | **320** | Average 80 per stat, close to a typical fully grown Pokémon (≈ 500 over 6 stats), so the same formulas feel familiar |
| Minimum per stat | **40** | Even a turtle's Speed or a glass cannon's Defense still matters |
| Maximum per stat | **130** | Allows a clear specialty (a 130 stat is ~1.6× average) without making the other three stats useless |
| Step | **5** | Readable numbers; fewer near-identical spreads |

[accepted] Example spreads (HP / Attack / Defense / Speed), as guidance for
the LLM prompt:

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

[accepted] An instance is one individual Peerling owned by a player, stored
in the player's [save](../tech/player-data.md#save-contents) (not on the shared
registry). Illustrative shape (exact format:
[data-formats § Peerling instance](../tech/data-formats.md#peerling-instance)):

```json
{
  "id": "<instance ID: 64 hex characters>",
  "species": { "/": "bafy…species-cid" },
  "nickname": null,
  "level": 5,
  "xp": 0,
  "hp": 22,
  "obtainedAt": 1791234567000,
  "origin": "wild | starter | created",
  "originalOwner": "<player ID of whoever first obtained it>",
  "originProof": "<CID of the catch event in the catcher's save log, or the server's origin attestation>",
  "traits": { "hp": 4, "attack": -7, "defense": 10, "speed": 0 },
  "shimmer": false
}
```

[accepted] There is **no evolution** in v1: a species never changes into
another species. Levels and XP are specified in
[battle § Experience and levelling](../gameplay/battle.md#experience-and-levelling).
Instances can change owner through [trading](../gameplay/trading.md).
Instances are stored in the player's save, and their verification is
described in [player-data](../tech/player-data.md#verification).

## Individual variation

[accepted] Individual Peerlings of the same species and level differ in two
ways ([D-0014](../decisions/D-0014-individual-variation.md)).

### Stat traits

[accepted] Each individual has a random **trait** for each of its four stats,
worth up to ±10%. [accepted] Details (approved 2026-10-05):
- A trait is a whole number from −10 to +10 (percent), drawn uniformly and
  independently for HP, Attack, Defense and Speed.
- The trait multiplies the stat calculated at the Peerling's level
  ([battle § Damage model](../gameplay/battle.md#damage-model)), so a +10 Attack
  trait gives 10% more Attack at every level.
- Traits never change (no training, no items) and travel with the Peerling
  when it is traded.
- Traits are **visible**: the Peerling's card shows each trait (e.g. "Attack
  +7%") and a simple overall rating (sum of the four traits, from −40 to +40).
  This makes it clear why one Mossnap is worth more than another, which drives
  trading.
- Traits **count in PvP**, in both level modes (Fair and Real levels,
  [pvp-battles](../gameplay/pvp-battles.md#fairness)). Otherwise the choice of
  option (b) would mean nothing in PvP.

### Shimmer variants

[accepted] A rare, purely cosmetic color variant: a **shimmer**. Details
(approved 2026-10-05):
- Chance: **1 in 500** per Peerling.
- Look: the model's colors are hue-shifted by an angle derived from the species
  CID (between 90° and 270°), so every shimmer of the same species looks the
  same and players learn to recognize them. A sparkle effect plays when a
  shimmer appears in battle. It's done with a shader on the existing static
  model, so it needs no extra generated assets.
- No effect on stats or battles.
- [accepted] Exact angle, so every client draws the same colors: with h =
  SHA-256 of the species CID's bytes (the same hash the
  [cry](../world/audio.md#peerling-cries) uses), hue shift =
  90 + (h[6] × 256 + h[7]) mod 181 degrees (90°–270°).

### Where the randomness comes from

[accepted] For wild Peerlings, traits and the shimmer roll come from the
encounter seed, in the fixed order defined in
[encounters § Wild Peerling generation](../gameplay/encounters.md#wild-peerling-generation),
so they are checked when the catch is replayed. For starters and Creation Shrine
Peerlings, the server draws them and includes them in the origin attestation
([player-data](../tech/player-data.md#starters-and-shrine-creations)).

## Requirements

- **SPC-001** [accepted] A species' description, type, attacks and 3D model MUST be stored on IPFS.
- **SPC-002** [accepted] A species MUST be identified by the CID of its species record.
- **SPC-003** [accepted] A species record MUST reference its image, 3D model and thumbnail by CID.
- **SPC-004** [accepted] A species record MUST carry the version field `v` and type `peerlings/species` ([data-formats](../tech/data-formats.md#conventions)); clients MUST ignore records with an unknown `v` rather than fail.
- **SPC-005** [accepted] A species' types MUST satisfy TYP-001 and TYP-002 in [types](types.md#requirements).
- **SPC-006** [accepted] A species' move set MUST satisfy the move-set rules in [moves](moves.md#requirements).
- **SPC-007** [accepted] Every species MUST have the same base-stat total (320), and each stat MUST lie within 40–130 in steps of 5.
- **SPC-008** [accepted] Clients MUST verify the attestation of a species record before using it.
- **SPC-009** [accepted] A Peerling instance MUST reference its species by CID and MUST NOT copy species data.
- **SPC-010** [accepted] Species records MUST be encoded as DAG-CBOR so the encoding, and therefore the CID, is deterministic.
- **SPC-011** [accepted] There MUST NOT be evolution in v1.
- **SPC-012** [accepted] Every Peerling instance MUST have a fixed trait from −10% to +10% for each of its four stats; traits are whole percents drawn uniformly, visible to players, and applied in PvP.
- **SPC-013** [accepted] Peerlings MUST have a rare cosmetic shimmer variant with a chance of 1 in 500; the look is a species-specific hue shift plus a sparkle effect.
- **SPC-014** [accepted] Traits and the shimmer roll MUST come from verifiable randomness: the encounter seed for wild Peerlings, the server's origin attestation for starters and shrine creations.
- **SPC-015** [accepted] Every species record MUST have a `sizeClass` and a `temperament`, and clients MUST draw the model at the height given for its size class.

## Open questions

_None at the moment._

## See also

- [Creation pipeline](creation-pipeline.md)
- [Registry](../tech/orbitdb-registry.md)
