---
title: "Player Data: Saves, Identity and Ownership"
type: system
status: draft
req_prefix: SAVE
tags: [tech, saves, identity, orbitdb, ipns, security]
sources:
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
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
| Inventory | Items (e.g. catching items; to be specified in [catching](../gameplay/catching.md)) | [proposed] |

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
   the **catch evidence**: the encounter seed, the battle's starting state, and
   every action taken. The Peerling can be used straight away in
   exploration and wild battles. It is *unverified* until step 3.
2. The server sees the event (it replicates the save log), replays the battle
   with the deterministic engine ([BTL-002](../gameplay/battle.md#requirements)),
   and checks the encounter seed (see below).
3. If valid, the server signs a **catch attestation** (instance ID, species CID,
   owner, encounter seed) and records the instance in the ownership ledger. The
   client appends `catch-verified`. If invalid, the instance is permanently
   *unverified*: it stays in the collection but can never be traded or used in
   PvP.

**Encounter seed.** [proposed] To stop players re-rolling encounters until they
get a good one, the seed is derived from a random value the player can't
choose: `seed = hash(beacon, playerId, encounterCounter)`. The **beacon** is a
random value the server publishes and signs every 5 minutes. Details:
[Q-029](../open-questions.md#q-029).

### Ownership ledger and trades [accepted, details proposed]

[proposed] The **ownership ledger** is an OrbitDB keyvalue database, keyed by
instance ID, that only the server can write (like the
[registry](orbitdb-registry.md)). Each entry: species CID, current owner,
catch attestation, trade history. A trade ([trading](../gameplay/trading.md)):

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
- **Levels outside PvP aren't verified.** XP from wild battles isn't replayed.
  An edited level only matters in the player's own wild battles, and in a
  Peerling they trade away. The receiving player gets the level shown.
- **Stale copies in PvP** (above).

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
- **SAVE-008** [proposed] Encounter seeds MUST include a server-published random beacon value, so players can't choose their encounters.
- **SAVE-009** [proposed] Catches MUST be playable while unverified; verification MAY happen later (e.g. when the server is reachable again).

## Open questions

[Q-029](../open-questions.md#q-029)

## See also

- [Architecture § Trust model](architecture.md#trust-model) · [Catching](../gameplay/catching.md) · [Trading](../gameplay/trading.md) · [PvP battles](../gameplay/pvp-battles.md)
