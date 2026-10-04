---
title: "Player Data: Saves, Identity and Ownership"
type: concept
status: proposed
req_prefix: SAVE
tags: [tech, saves, identity, orbitdb, ipns, security]
sources:
  - raw/conversations/2026-10-04-answers-round-2.md
related:
  - wiki/gameplay/player-character.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/tech/architecture.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-04
---

# Player Data: Saves, Identity and Ownership

> Canonical home for where a player's save lives and how others can trust what
> it says. This page compares the options, including OrbitDB- and IPFS-based
> ones, and gives a recommendation. Nothing here is decided yet: it answers the
> designer's question from 2026-10-04 and feeds [Q-014](../open-questions.md#q-014),
> [Q-025](../open-questions.md#q-025) and [Q-026](../open-questions.md#q-026).

## Two separate problems

Keeping saves in the browser causes two different problems, and each needs its
own solution:

| Problem | Question | Example |
|---------|----------|---------|
| **Durability and portability** | *Where* is the data stored? | Clearing browser data deletes everything; a save can't move to another device. |
| **Integrity** | *Who* can vouch for the data? | A player edits their save to own a Peerling they never caught, or trades one away and keeps a copy. |

**Key point:** moving the save to OrbitDB or IPFS solves the first problem,
not the second. Whoever holds the write key (the player) can still write
anything into their own OrbitDB database or IPNS record. Signatures only prove
*who* wrote something, not that it is *true*. Integrity needs someone other
than the player to vouch for the important events. In practice that is the
operator server, which is already trusted for the registry
([D-0005](../decisions/D-0005-server-sole-registry-writer.md)), or
re-verification by replaying deterministic game logic.

## Storage options

| Option | How it works | Durable / cross-device | IPFS showcase | Cost / risk |
|--------|--------------|------------------------|---------------|-------------|
| **S1. Browser only** | IndexedDB in the browser, with optional export to a file | No (unless the player exports) | None | Simplest |
| **S2. IPFS snapshots + IPNS** | Each save is a DAG on IPFS; the latest root CID is published under the player's IPNS name (derived from their identity key). Each snapshot links to the previous one, so history forms a hash-linked chain. The server pins the latest snapshot. | Yes: any device with the key can resolve the IPNS name and load the save | Good (IPNS + content addressing) | Light. IPNS publishing from the browser goes through delegated routing |
| **S3. Per-player OrbitDB log** | Each player has their own OrbitDB *events* database, writable only by their identity. Every state change is a signed log entry. The server replicates and pins every player's database. | Yes: open the same database address with the key on any device | Very good: OrbitDB used for player data, not just the registry | Heavier: the server keeps one database open per player. Scale needs testing |

All three need the **identity key** to be recoverable, because losing it means
losing the save. Options: a recovery phrase (a list of words shown once and
written down by the player), a backup file, or a passkey (WebAuthn) that
derives the key. The key itself is never sent to the server.

## Integrity options

| Option | How it works | Stops edited levels | Stops fake ownership | Stops trade duplication | Needs server |
|--------|--------------|:---:|:---:|:---:|---|
| **I1. Trust** | Accept whatever a client says | ✗ | ✗ | ✗ | No |
| **I2. Neutralize** | PvP normalizes levels; ownership isn't checked; duplication is accepted. Species are balanced, so faking gains little. | ✓ (in PvP) | ✗ | ✗ | No |
| **I3. Server-attested events** | The server co-signs the events that create scarcity. *Catch:* the client sends the encounter seed and its battle actions; the server replays the deterministic battle (BTL-002) and, if valid, signs the new instance. *Trade:* both players sign the trade, and the server records the change of owner in an **ownership ledger** (an OrbitDB database only the server can write, like the registry) and signs it. | ✓ (in PvP, with I2) | ✓ | ✓ | Only for catches (can happen later) and trades |
| **I4. Server-authoritative game** | The server runs all game logic | ✓ | ✓ | ✓ | Always: against pillar 5, rejected |

Replaying a battle is cheap CPU work (no GPU), so I3 adds little server load.
Catch attestation can be **asynchronous**: a caught Peerling is usable at once
in exploration and wild battles, and becomes *verified* when the server has
signed it. Only verified instances can be traded or used in PvP.

One detail I3 still needs: to stop players re-rolling encounters until they
get one they like, the encounter seed must include randomness the player can't
choose, e.g. a value the server publishes and signs every few minutes. This is
to be specified if I3 is chosen.

## Recommendation [proposed]

**S3 + I2 + I3:**

1. **Storage (S3):** the save is a player-signed OrbitDB event log, replicated
   and pinned by the server, with periodic snapshots for fast loading. Recovery
   uses a recovery phrase. This makes OrbitDB central to *both* the registry and
   every player's progress, which is the strongest IPFS showcase. If per-player
   databases don't scale on the server, fall back to S2 (IPNS snapshots); the
   integrity design is the same either way.
2. **Integrity (I2 + I3):** PvP uses normalized levels. Catches are verified by
   server replay, and trades go through the server's ownership ledger. Only
   verified instances can be traded or used in PvP. That stops faked Peerlings
   and duplication without a game server: the server signs results, but play
   itself stays peer-to-peer and client-side.
3. **Verification by peers:** an opponent or trade partner checks a Peerling's
   server signatures (catch, and the latest ledger entry) themselves. The
   server doesn't need to be online during the battle.

Cheaper alternative: **S2 + I2**. Saves are portable and survive cleared
browsers, but cheating and duplication are accepted as harmless in a casual
game.

## Requirements

All [proposed]; they apply only if the recommendation is accepted.

- **SAVE-001** [proposed] A player's save MUST be stored on IPFS (per-player OrbitDB log, or IPNS-addressed snapshots) and pinned or replicated by the server, so it survives cleared browser storage and can be loaded on another device.
- **SAVE-002** [proposed] The client MUST let the player back up their identity key (recovery phrase and/or file) and restore it on another device.
- **SAVE-003** [proposed] Only server-verified Peerling instances MUST be usable in trades and PvP battles.
- **SAVE-004** [proposed] Instance ownership MUST be recorded in a server-written ownership ledger; a trade MUST NOT complete until the ledger records it.

## Open questions

[Q-014](../open-questions.md#q-014) · [Q-025](../open-questions.md#q-025) ·
[Q-026](../open-questions.md#q-026)

## See also

- [Architecture § Trust model](architecture.md#trust-model) · [PvP battles](../gameplay/pvp-battles.md) · [Trading](../gameplay/trading.md)
