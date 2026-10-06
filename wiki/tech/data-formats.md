---
title: Data Formats
type: data
status: accepted
req_prefix: FMT
tags: [tech, formats, ipld, orbitdb, signatures]
sources:
  - raw/conversations/2026-10-05-formats-request.md
  - raw/conversations/2026-10-06-formats-approved.md
  - raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
related:
  - wiki/tech/protocols.md
  - wiki/tech/creation-api.md
  - wiki/tech/player-data.md
  - wiki/tech/orbitdb-registry.md
  - wiki/peerlings/peerling-species.md
updated: 2026-10-06
---

# Data Formats

> Canonical, exact formats of every record the game stores or signs: the shared
> conventions (encoding, identifiers, signatures), the OrbitDB databases, and
> each record type. Other pages describe *what* these records mean and may
> show simplified examples; when they differ, this page wins. Messages sent
> over the network are in [protocols](protocols.md).

**Status: [accepted]**: drafted 2026-10-05, approved 2026-10-06 ([Q-047](../open-questions.md#q-047)).

## Conventions

### Encoding
- Every record and message is **DAG-CBOR** (the IPLD codec: deterministic CBOR
  with sorted map keys). The same data always gives the same bytes and the same
  CID.
- **No floating-point numbers** anywhere: all numbers are integers (stats,
  percentages, tile coordinates, times). This keeps hashing and replays
  identical on every platform.
- Text is UTF-8 in Unicode NFC form.
- This page shows records as DAG-JSON for readability: a link is
  `{ "/": "bafy…" }`, bytes are `{ "/": { "bytes": "<base64>" } }`.
- Unknown fields are ignored. Every record has a version field `v`. A reader
  that doesn't know a record's `v` ignores the record.

### Identifiers

| Name | Format |
|------|--------|
| **Player ID** | The libp2p peer ID of the player's Ed25519 key, as a base58btc string (`12D3KooW…`) |
| **Operator ID** | The peer ID of the operator server's signing key, built into the game app and the Peerlings Viewer |
| **CID** | CIDv1, SHA-256; written as base32 lower case (`bafy…`) |
| **Time** | Unsigned integer, milliseconds since the Unix epoch, UTC |
| **Tile** | `[x, y]`, integers, 0–1999, origin at the world's north-west corner |
| **Instance ID** | 32 bytes, written as 64 lower-case hex characters |

**One key per player** ([D-0018](../decisions/D-0018-one-key-per-player.md)). A player's single Ed25519 key pair is at the same time:
their player ID, their libp2p peer ID, their OrbitDB identity, and their IPNS
name. So pubsub messages, OrbitDB entries and IPNS records are all
authenticated by the same key with no extra mapping. (Implementers may need a
custom OrbitDB identity provider that uses this key directly.) The phone running
the Peerlings Viewer uses its own throwaway peer ID.

### Signed envelope

Records that must be verifiable on their own, outside OrbitDB, are wrapped in an
envelope:

```json
{
  "v": 1,
  "type": "peerlings/<kind>",
  "signer": "<player ID or operator ID>",
  "body": { "…": "…" },
  "sig": { "/": { "bytes": "<64-byte Ed25519 signature>" } }
}
```

- `sig` is the Ed25519 signature over the bytes
  `"peerlings-sig-v1\n"` followed by the DAG-CBOR encoding of
  `{ "v", "type", "signer", "body" }`.
- Including `type` means a signature for one kind of record can never be passed
  off as another kind.
- The CID of a signed record is the CID of the whole envelope (including `sig`).

### Hashes
`SHA-256(…)` of structured data means SHA-256 of its DAG-CBOR bytes. A list of
values joined with `‖` means plain byte concatenation.

### Size limits

| Item | Limit |
|------|-------|
| Pubsub message | 4 KiB; 16 KiB on battle topics |
| Stream message | 1 MiB (the phone backup streams its file in chunks) |
| Strings | As stated per field; longer values are invalid |

## OrbitDB databases

| Database | Name | Type | Who can write | Key |
|----------|------|------|---------------|-----|
| Registry | `peerlings-registry-v1` | documents | Anyone; entries must carry a valid operator listing signature (custom access controller) | `_id` = species CID |
| Transfer log | `peerlings-transfers-v1` | events | Anyone; every transfer must be signed by its `from` player (custom access controller) | — |
| Epoch log | `peerlings-epochs-v1` | events | Operator only | — |
| Species stats | `peerlings-species-stats-v1` | keyvalue | Operator only | species CID |
| Save log (one per player) | `peerlings-save-v1` | events | That player only | — |

The game app ships the addresses of the first four. A save log's address is
derived from its manifest (name `peerlings-save-v1`, type `events`, writer =
the player ID), so anyone can compute it from a player ID ([SAVE-018](player-data.md#requirements)).

## Records

### Species record — `peerlings/species`

Signed envelope; signer: **operator** (this is the species
[attestation](../glossary.md#attestation)). The species CID is the CID of this
envelope. Meaning: [peerling-species](../peerlings/peerling-species.md).

| Body field | Type | Rules |
|------------|------|-------|
| `name` | string | 1–20 characters, unique in normalized form ([CRE-025](../peerlings/creation-pipeline.md#requirements)) |
| `summary` | string | ≤ 200 characters |
| `lore` | string | ≤ 600 characters |
| `types` | [string] | 1–2 of `Normal Fire Water Grass Electric Earth Air Ice Metal Light Shadow Spirit`; first = primary |
| `baseStats` | map | `hp`, `attack`, `defense`, `speed`: uint, 40–130, multiples of 5, sum 320 |
| `moves` | [Move] | exactly 3, one per slot |
| `assets` | map | `image`, `model`, `thumbnail`: CID |
| `creator` | map | `id`: player ID; `displayName`: string ≤ 20 |
| `createdAt` | time | |
| `provenance` | map | `wish`: string ≤ 300; `concept`: map; `imagePrompt`: map; `models`: map of `concept`, `image`, `to3d` → string `"<name>@<version>"`; `seeds`: map of stage → uint |

**Move:** `slot` (`"quick"` \| `"strong"` \| `"signature"`), `template` (template
ID, [moves](../peerlings/moves.md#template-table)), `name` (≤ 24), `description`
(≤ 120), `type` (type name), `stat` (only for `sig-weaken` / `sig-empower`:
`"attack"` \| `"defense"` \| `"speed"`).

### Registry entry — `peerlings/listing`

Stored in the registry as `{ "_id": "<species CID>", "listing": <envelope> }`.
Signed envelope; signer: **operator**.

| Body field | Type | Rules |
|------------|------|-------|
| `species` | CID | |
| `seq` | uint | 1, 2, 3, … with no gaps |
| `name` | string | as in the species record |
| `nameKey` | string | normalized name used for uniqueness |
| `types` | [string] | |
| `thumbnail` | CID | |
| `createdAt` | time | |
| `status` | string | `"active"` or `"removed"` |
| `removedAtSeq` | uint | only when `status` is `"removed"` |

Delisting replaces the document with a new envelope that has `status:
"removed"`.

### Epoch record — `peerlings/epoch`

Signed envelope; signer: **operator**. Published every epoch on the epoch topic
and in the epoch log. Meaning:
[player-data § Encounter seeds](player-data.md#encounter-seeds).

| Body field | Type | Rules |
|------------|------|-------|
| `epoch` | uint | floor(Unix seconds ÷ 300) |
| `drandRound` | uint | the drand round used |
| `randomness` | bytes(32) | that round's randomness |
| `drandSignature` | bytes | that round's signature, for verification |
| `registryHeight` | uint | |
| `generator` | map | `version`: uint; `fromEpoch`: uint |

A **client-derived** epoch record has the same body, `"signer": null`, no
`sig`, and an extra top-level field `"derivedBy": "client"`.

### Peerling instance

Part of save records; not signed on its own.

| Field | Type | Rules |
|-------|------|-------|
| `id` | instance ID | Caught: SHA-256(player ID ‖ encounter number as 8-byte big-endian). Starter / shrine: assigned by the server |
| `species` | CID | |
| `nickname` | string or null | ≤ 20 |
| `level` | uint | 1–50 |
| `xp` | uint | |
| `hp` | uint | current HP |
| `traits` | map | `hp`, `attack`, `defense`, `speed`: int −10…10 |
| `shimmer` | bool | |
| `origin` | string | `"wild"`, `"starter"` or `"created"` (how it came into existence; trades don't change it) |
| `originalOwner` | player ID | |
| `originProof` | CID | the `catch` save-log entry, or the origin attestation |
| `obtainedAt` | time | |

### Origin attestation — `peerlings/origin`

Signed envelope; signer: **operator**. For starters and Creation Shrine
Peerlings.

| Body field | Type |
|------------|------|
| `instanceId` | instance ID |
| `species` | CID |
| `owner` | player ID |
| `origin` | `"starter"` or `"created"` |
| `level` | uint |
| `traits` | map, as in the instance |
| `shimmer` | bool |
| `t` | time |

### Catch evidence

Part of a `catch` save-log event. Meaning:
[player-data § Catches](player-data.md#catches).

| Field | Type | Rules |
|-------|------|-------|
| `encounter` | uint | the encounter number n |
| `epochRecord` | epoch record | the full record used (signed or client-derived) |
| `tile` | tile | where the encounter started |
| `candidate` | uint | 0–4, which candidate was met |
| `team` | [map] | battle-start state of each team member: `instanceId`, `species`, `level`, `hp`, `traits` |
| `actions` | [Action] | every action, in order |

**Action** (also used in PvP): `{ "kind": "move", "slot": "quick" | "strong" |
"signature" }`, `{ "kind": "switch", "to": <instance ID> }`, `{ "kind":
"replace", "to": <instance ID> }` (after a faint), `{ "kind": "catch" }`,
`{ "kind": "flee" }`.

### Save-log events

Each entry in a player's save log (already signed by OrbitDB with the player's
key) has the value `{ "v": 1, "type": "<event>", "t": <time>, …payload }`.
Meaning: [player-data § Save log](player-data.md#save-log).

| `type` | Payload fields |
|--------|----------------|
| `profile` | `displayName` (≤ 20), `appearance` (map: `body`, `hair`, `accessories` [string IDs]; `colors` map of `skin`, `hair`, `outfit` → `"#rrggbb"`) |
| `species-created` | `species` (CID) |
| `starter` | `instance`, `attestation` (origin attestation) |
| `created` | `instance`, `attestation` (origin attestation) |
| `release` | `instances` ([instance ID]), `transferEntry` (CID) |
| `catch` | `instance`, `evidence` (catch evidence) |
| `battle-result` | `encounter` (uint), `outcome` (`"won"` \| `"caught"` \| `"fled"` \| `"lost"`), `team` ([map: `instanceId`, `xpGained`, `level`, `hp`]) |
| `team` | `instances` ([instance ID], ≤ 4) |
| `nickname` | `instanceId`, `nickname` (string ≤ 20 or null) |
| `trade` | `transferEntry` (CID), `out` ([instance ID]), `in` ([instance]) |
| `seen` | `species` (CID), `biome` (biome name where first met), `shimmer` (bool: seen as a shimmer). Written the first time a species is met, and again the first time it is met as a shimmer |
| `position` | `tile`, `facing` (`"n"` \| `"e"` \| `"s"` \| `"w"`) |
| `explored` | `chunks` ([[cx, cy]]: newly revealed chunks) |
| `session-start` | `device` (16 random bytes, fixed per installation) |
| `snapshot` | `state` (CID of a save snapshot), `upTo` (CID of the last log entry it includes) |

### Save snapshot — `peerlings/save`

A plain DAG-CBOR document (not an envelope; it is referenced from the signed
save log).

| Field | Type |
|-------|------|
| `v`, `type` | `1`, `"peerlings/save"` |
| `player` | player ID |
| `profile` | as in the `profile` event |
| `instances` | [instance] (whole collection) |
| `team` | [instance ID] |
| `created` | [CID] |
| `peerdex` | map: `seen` [map: `species`, `biome`, `shimmerSeen` bool], `caught` [map: `species`, `shimmerCaught` bool] |
| `position` | map: `tile`, `facing` |
| `lastRestPoint` | tile |
| `explored` | bytes: bit set of 63 × 63 chunks, row-major, bit 1 = revealed |
| `nextEncounter` | uint |

### Transfer and transfer-log entry — `peerlings/transfer`

A **transfer** is a signed envelope; signer: the `from` player.

| Body field | Type | Rules |
|------------|------|-------|
| `instanceId` | instance ID | |
| `from` | player ID | the current owner |
| `to` | player ID or `"released"` | |
| `prev` | CID or `"origin"` | the CID of this Peerling's previous transfer envelope |
| `trade` | bytes(16) or absent | shared random ID tying the two sides of one trade together |

A **transfer-log entry** is `{ "v": 1, "type": "peerlings/transfers",
"transfers": [<transfer>, …] }`. A trade is one entry with both players'
transfers; a Creation Shrine release is one entry with exactly 3 transfers to
`"released"`. Conflict rule: two transfers with the same `prev` → the one whose
**log entry** CID is lower wins ([SAVE-015](player-data.md#requirements)).

### Species stats — `peerlings/species-stats`

Signed envelope; signer: **operator**; stored under the species CID. Meaning:
[creator-feedback](../gameplay/creator-feedback.md).

| Body field | Type |
|------------|------|
| `species` | CID |
| `encounters`, `catches`, `owners`, `trades`, `providers` | uint |
| `updatedEpoch` | uint |

### Player profile document — `peerlings/profile`

Signed envelope; signer: the **player**. Published on IPFS; the player's IPNS
record points to `/ipfs/<CID of this envelope>`. Meaning:
[sharing](../gameplay/sharing.md).

| Body field | Type |
|------------|------|
| `displayName` | string ≤ 20 |
| `appearance` | as in the `profile` event |
| `team` | [map: `species`, `level`, `traits`, `shimmer`, `nickname`] |
| `created` | [CID] |
| `peerdex` | map: `seen` uint, `caught` uint |
| `saveLog` | string: the save log's OrbitDB address |
| `updated` | time |

IPNS records: validity 30 days, TTL 1 hour, republished each session by the
client and continuously by the server.

### Backup file and phone backup payload — `peerlings/backup`

A **CAR v1** file (`.car`) whose single root is a DAG-CBOR block:

| Field | Type |
|-------|------|
| `v`, `type` | `1`, `"peerlings/backup"` |
| `player` | player ID |
| `privateKey` | bytes: the libp2p protobuf encoding of the Ed25519 private key |
| `snapshot` | CID of the latest save snapshot |
| `saveLog` | string: OrbitDB address |
| `created` | time |

The CAR also contains every block of the snapshot document. The file holds the
private key in the clear: the game must say so when exporting. File name:
`peerlings-<display name>-<YYYY-MM-DD>.car`.

### Recovery phrase

12 words from the BIP-39 English word list (128 bits of entropy). The key is
derived as: BIP-39 seed (empty passphrase) → HKDF-SHA256 with info
`"peerlings-identity-v1"` → 32-byte Ed25519 seed.

## Requirements

- **FMT-001** [accepted] All records and messages MUST be DAG-CBOR without floating-point numbers, following the conventions on this page.
- **FMT-002** [accepted] Each player MUST have one Ed25519 key that is their player ID, libp2p peer ID, OrbitDB identity and IPNS name.
- **FMT-003** [accepted] Records that need standalone verification MUST use the signed envelope with the `"peerlings-sig-v1\n"` prefix.
- **FMT-004** [accepted] The OrbitDB databases, record types and fields MUST be exactly as defined on this page; readers MUST ignore unknown fields and records with an unknown `v`.

## Open questions

_None at the moment._

## See also

- [Protocols](protocols.md) · [Creation API](creation-api.md) · [Player data](player-data.md)
