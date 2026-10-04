---
title: "Player Data: Saves, Identity and Ownership"
type: system
status: draft
req_prefix: SAVE
tags: [tech, saves, identity, orbitdb, ipns, security]
sources:
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-7.md
related:
  - wiki/decisions/D-0009-player-data-on-orbitdb.md
  - wiki/gameplay/player-character.md
  - wiki/gameplay/catching.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/peerlings/peerling-species.md
  - wiki/tech/architecture.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-04
---

# Player Data: Saves, Identity and Ownership

> Canonical home for what a player's save contains, where it is stored (a
> per-player OrbitDB log), how the identity key is recovered, and how catches
> and trades are verified by the server so other players can trust them.
> Decided in [D-0009](../decisions/D-0009-player-data-on-orbitdb.md).

## Chosen design

[accepted] ([D-0009](../decisions/D-0009-player-data-on-orbitdb.md))

| Concern | Solution |
|---------|----------|
| Where the save lives | A per-player OrbitDB **save log** (an append-only event log) that only the player's identity can write. The server replicates and pins it. |
| Recovery | The identity key can be restored with a **recovery phrase** |
| Faked Peerlings | The server verifies each catch by **replaying the battle** and signs it |
| Duplication via trades | Trades complete only when recorded in the server's **ownership ledger** |
| Edited levels in PvP | PvP uses **level 50** for everyone |
| What can be traded or used in PvP | Only **verified** Peerlings |

## Save contents

The save holds everything about the player's progress. Species data (art, 3D
model, stats, moves) is **not** in the save; it lives on IPFS and the save
refers to it by CID.

| Part | Contents | Provenance |
|------|----------|------------|
| Collection | Every Peerling the player owns, each with its **current level**, XP, current HP, nickname, origin and verification. Format: [Peerling instance](../peerlings/peerling-species.md#peerling-instance) | [accepted] |
| Team | Ordered list of the instance IDs in the active team (size: [catching](../gameplay/catching.md)) | [proposed] |
| Profile | Player ID (public key), display name, character appearance | [proposed] |
| Created species | CID(s) of the species this player created | [proposed] |
| Peerdex | Species seen and species caught (CIDs) | [proposed] |
| Position | Last position and facing in the world | [proposed] |
| Inventory | [accepted] No battle items in the first version ([CAT-003](../gameplay/catching.md#requirements)). [proposed] No inventory at all in the first version, so this part is empty | [proposed] |

Not in the save: the identity **private key** (stays on the device; restored
with the recovery phrase), and the authoritative owner of each Peerling (that is
the ownership ledger).

## Save log

[proposed] The save is stored as *events*, not as one file that gets
overwritten. The current save is what you get by applying all events in
order. OrbitDB *events* databases are append-only and every entry is signed by
the writer, so the log is also a tamper-evident history.

| Event | Written when | Payload |
|-------|-------------|---------|
| `profile` | Onboarding; profile changes | display name, appearance |
| `species-created` | Creation pipeline finished | species CID |
| `starter` | Onboarding finished | the starter instance + the server's attestation |
| `release` | Peerlings offered at the [Creation Shrine](../gameplay/creation-shrine.md) | instance IDs, ledger entry reference |
| `created` | A shrine creation was published | the new instance + the server's attestation |
| `catch` | A wild Peerling is caught | the new instance + **catch evidence** (see below) |
| `catch-verified` | The server's verification arrives | instance ID, catch attestation |
| `battle-result` | After a wild battle | for each participating instance: XP gained, new level, HP |
| `team` | Team changed | ordered instance IDs |
| `nickname` | Peerling renamed | instance ID, nickname |
| `trade` | The ownership ledger recorded a trade | ledger entry reference; instances out; full data of instances in |
| `seen` | First sighting of a species | species CID |
| `position` | Every 30 s while moving, and on exit | position, facing |
| `snapshot` | Every 50 events, and on exit | CID of a DAG-CBOR document with the full current save, and the last event it includes |

Loading a save means reading the latest `snapshot` and applying the events
after it. The server replicates every player's log and pins the snapshots.

## Verification

### Catches [accepted, details proposed]

1. When a player catches a Peerling, the client appends a `catch` event with
   the **catch evidence**: the encounter number, the epoch record used, the
   position, which of the encounter's candidates was met
   ([encounters § Candidates](../gameplay/encounters.md#candidates)), the
   battle's starting state, and every action taken. The Peerling can be used straight away in
   exploration and wild battles. It is *unverified* until step 3.
2. The server sees the event (it replicates the save log), replays the battle
   with the deterministic engine ([BTL-002](../gameplay/battle.md#requirements)),
   and runs the checks in [Encounter seeds](#encounter-seeds).
3. If valid, the server signs a **catch attestation** (instance ID, species CID,
   owner, encounter seed) and records the instance in the ownership ledger. The
   client appends `catch-verified`. If invalid, the instance is permanently
   *unverified*: it stays in the collection but can never be traded or used in
   PvP.

### Encounter seeds

[accepted] Approved 2026-10-04, including drand as the randomness source.

Goal: players can't choose or re-roll their wild encounters, play keeps working
without the server, and the server can check everything afterwards.

**Epochs and the epoch record.**
- Time is divided into 5-minute **epochs**: epoch number E = floor(Unix time in
  seconds ÷ 300).
- At the start of each epoch the server publishes a signed
  [epoch record](../glossary.md#epoch-record):

  ```json
  {
    "epoch": 5873210,
    "drandRound": 1234567,
    "randomness": "<32 bytes, hex>",
    "drandSignature": "<hex>",
    "registryHeight": 1842,
    "serverSignature": "<hex>"
  }
  ```

- `randomness` is taken from **[drand](../glossary.md#drand)**, the public
  randomness beacon run by the League of Entropy (Protocol Labs, the company
  behind IPFS, is one of its main contributors). It is the value of the drand round that
  starts at or just after the epoch's start time. drand values can be verified
  with drand's public key, so clients don't have to trust the server not to
  bias the randomness. Clients get drand values through the server's epoch
  record and verify the drand signature locally, so they never contact drand
  directly. Fallback if drand is unavailable or not wanted: the server
  generates the value itself, and players trust it like they trust the
  registry.
- `registryHeight` fixes which species are eligible during that epoch (see
  *Registry state* below).
- Distribution: live on the pubsub topic `peerlings/v1/epoch`
  ([realtime-networking](realtime-networking.md)), and in an OrbitDB
  **epoch log** (events database, written only by the server) for clients that
  were offline and for the server's own verification. That is 288 small entries
  per day.

**Seeds.**
- Every encounter has an **encounter number** n: 0 for the player's first
  encounter, increasing by exactly 1 for each encounter. *Every* encounter is
  logged in the save log, including ones the player flees from or loses
  (`battle-result` carries n).
- encounter seed = SHA-256("peerlings/encounter/v1" ‖ epoch randomness ‖ player
  ID ‖ n).
- The seed drives the [random number generator](../gameplay/battle.md#random-number-generator)
  for the species choice, the wild level and the whole battle
  ([encounters](../gameplay/encounters.md)).
- An encounter uses the latest epoch record the client has when the encounter
  starts. Epoch numbers must never decrease from one encounter to the next.

**Why this stops re-rolling.**
- Same epoch + same n → same encounter, so reloading the page gives exactly the
  same encounter.
- Fleeing is a logged encounter, so n can't be skipped. A gap in the encounter
  numbers makes every later catch fail verification.
- The only way to get a different encounter for the same n is to wait for a new
  epoch before triggering it, which costs up to 5 minutes. See *Known gaps*.

**Registry state.**
- The server gives each registry entry a sequence number `seq` (1, 2, 3, …)
  when adding it; a takedown records `removedAtSeq`
  ([orbitdb-registry](orbitdb-registry.md)).
- During epoch E, the eligible species are the entries with
  seq ≤ registryHeight(E), minus those with removedAtSeq ≤ registryHeight(E).
- A client that hasn't yet synced the registry up to that height doesn't start
  encounters until it has. Registry entries are small, so this is brief.

**When the server is offline.** [proposed] If no new server-signed epoch
record has arrived for 10 minutes (two epochs), the client derives epoch
records itself:
- `randomness` is the drand value for the epoch, fetched directly from public
  drand endpoints (run by League of Entropy members) and checked against
  drand's public key as usual.
- `registryHeight` is the height from the latest server-signed epoch record the
  client has. While the server is down nothing can be added to the registry
  (only the server writes it), so nothing is missed.
- The record is marked as client-derived and has no server signature.

This keeps wild encounters working with nothing from the operator server. When
the server is back, it accepts client-derived records whose drand value is
genuine and whose height matches the client's latest signed record. Because
the randomness still comes from drand, a client-derived record gives the player
no extra choice. See [resilience](resilience.md).

**Offline play.** Without network access, the client keeps using the last epoch
record it has. That gives no extra choice (same epoch and same n give the same
encounter), so offline play needs no time limit. Catches made offline are
verified when the server next sees the save log.

**What the server checks when verifying a catch.**
1. The epoch record is genuine: drand signature, plus either the server's
   signature or the client-derived rules above.
2. Encounter numbers run from 0 with no gaps; epochs never decrease.
3. The species is the logged candidate from the deterministic candidate list,
   and the wild level matches, for (seed, position, registry height, save log).
4. Replaying the battle with the logged actions ends in this catch.
5. The instance ID isn't already in the ownership ledger.

### Starters and shrine creations

[proposed] Starters ([onboarding](../gameplay/onboarding.md)) and Peerlings
created at the [Creation Shrine](../gameplay/creation-shrine.md) don't come from a
catch, so there's no battle to replay. Instead the server creates them: it signs
an attestation of the same form as a catch attestation (with origin `starter` or
`created` instead of an encounter seed) and records the instance in the
ownership ledger. These Peerlings are verified from the start.

### Ownership ledger and trades [accepted, details proposed]

[proposed] The **ownership ledger** is an OrbitDB keyvalue database, keyed by
instance ID, that only the server can write (like the
[registry](orbitdb-registry.md)). Each entry: species CID, current owner,
attestation, trade history, and a `released` flag for Peerlings given up at the
[Creation Shrine](../gameplay/creation-shrine.md). A trade ([trading](../gameplay/trading.md)):

1. Both players sign the trade record and send it to the server.
2. The server checks that each offered instance is verified and currently owned
   by the player offering it.
3. The server updates the ledger, then both clients append a `trade` event.

So a modified client that "keeps a copy" can never trade that copy again: the
ledger says someone else owns it.

### PvP

[accepted] PvP uses level 50 and only verified Peerlings. [proposed] The
opponent checks each catch attestation's signature, which needs no server
during the battle. It does not check the ledger, so a player who traded a
Peerling away could still battle with a stale copy. This is an accepted gap,
since that Peerling was legitimately caught and gives no advantage.

## Known gaps (accepted risks)

[proposed]
- **Positions aren't verified.** A modified client could claim a different
  position to target a biome or wild level. Possible mitigation: the server
  checks that consecutive `position` events are reachable at walking speed.
- **Small choice among recent epochs.** A player who knows the upcoming
  encounter (the client computes it in advance for prefetching) can stall until
  a new epoch. That is at most one re-roll per 5 minutes, which seems acceptable.
- **Levels outside PvP aren't verified.** XP from wild battles isn't replayed.
  An edited level only matters in the player's own wild battles, and in a
  Peerling they trade away. The receiving player gets the level shown.
- **Stale copies in PvP** (above).
- **Choice among encounter candidates.** A modified client could claim the first
  candidates failed to download and pick a later one: at most a choice of 1 in
  5. All species are equally strong, so this only lets a player favour a species
  they like.

## Storage options considered

Background for D-0009. Moving the save to OrbitDB or IPFS fixes *durability*,
not *integrity*: whoever holds the write key can write anything. Integrity comes
from server signatures.

| Option | How it works | Durable / cross-device | IPFS showcase | Cost / risk |
|--------|--------------|------------------------|---------------|-------------|
| S1. Browser only | IndexedDB, with optional export to a file | No | None | Simplest |
| S2. IPFS snapshots + IPNS | Save snapshots on IPFS; the latest one is published under the player's IPNS name | Yes | Good | Light. **Fallback** if S3 doesn't scale |
| **S3. Per-player OrbitDB log** (chosen) | Player-signed event log, replicated and pinned by the server | Yes | Very good | The server keeps one database open per player |

| Integrity option | Result |
|------------------|--------|
| I1. Trust clients | Nothing prevented |
| **I2. Neutralize** (chosen, for PvP levels) | Edited levels give no PvP advantage |
| **I3. Server-attested events** (chosen) | Faked catches and trade duplication prevented |
| I4. Server-authoritative game | Rejected: against pillar 5 |

## Requirements

- **SAVE-001** [accepted] A player's save MUST be stored as a per-player OrbitDB log, writable only by the player's identity and replicated and pinned by the server, so it survives cleared browser storage and can be loaded on another device.
- **SAVE-002** [accepted] The client MUST let the player back up their identity key with a recovery phrase and restore it on another device. The private key MUST NOT leave the device in any other form.
- **SAVE-003** [accepted] Only verified Peerling instances MUST be usable in trades and PvP battles.
- **SAVE-004** [accepted] Instance ownership MUST be recorded in a server-written ownership ledger; a trade MUST NOT complete until the ledger records it.
- **SAVE-005** [accepted] The save MUST contain every owned Peerling instance with its current level and XP. [proposed] It MUST also contain the parts listed in [Save contents](#save-contents).
- **SAVE-006** [accepted] The server MUST verify a catch by replaying the battle from the catch evidence before signing a catch attestation.
- **SAVE-007** [proposed] The save log MUST be event-based as listed in [Save log](#save-log), with periodic snapshots so loading doesn't replay the full history.
- **SAVE-008** [accepted] Encounter seeds MUST be derived as in [Encounter seeds](#encounter-seeds): from the epoch record's randomness, the player ID and a gap-free encounter number.
- **SAVE-009** [proposed] Catches MUST be playable while unverified; verification MAY happen later (e.g. when the server is reachable again).
- **SAVE-010** [accepted] Every wild encounter, including fled and lost ones, MUST be recorded in the save log with its encounter number.
- **SAVE-011** [accepted] The server MUST publish a signed epoch record every 5 minutes, on pubsub and in a server-written OrbitDB epoch log.
- **SAVE-012** [proposed] Starters and shrine-created Peerlings MUST receive a server attestation and a ledger entry when they are created.
- **SAVE-013** [proposed] When no server-signed epoch record has arrived for two epochs, clients MUST derive epoch records from drand and their latest signed registry height, and the server MUST accept such records when verifying.

## Open questions

_None at the moment._

## See also

- [Architecture § Trust model](architecture.md#trust-model) · [Catching](../gameplay/catching.md) · [Trading](../gameplay/trading.md) · [PvP battles](../gameplay/pvp-battles.md)
