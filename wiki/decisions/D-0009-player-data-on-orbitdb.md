---
title: "D-0009: Player saves on OrbitDB, with server-verified catches and trades"
type: decision
status: accepted
tags: [tech, saves, orbitdb, security]
sources:
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/tech/player-data.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/gameplay/catching.md
updated: 2026-10-04
---

# D-0009: Player saves on OrbitDB, with server-verified catches and trades

**Status:** accepted (2026-10-04, resolves [Q-014](../open-questions.md#q-014),
[Q-025](../open-questions.md#q-025), [Q-026](../open-questions.md#q-026))

## Context
Browser-only saves are lost when browser data is cleared, can't move between
devices, and can be edited freely. With a shared world, PvP and trading
([D-0008](D-0008-shared-multiplayer-world.md)), edited saves would affect other
players. The options were analysed in [player-data](../tech/player-data.md).

## Decision
[accepted] The recommended combination from [player-data](../tech/player-data.md):

1. **Storage:** each player's save is a player-signed OrbitDB event log. The
   server replicates and pins every player's log. The player's identity key can
   be restored with a recovery phrase.
2. **Catches:** the server verifies each catch by replaying the deterministic
   battle and signs the new Peerling. Verification may happen after the catch.
3. **Trades:** a trade completes only when the server has recorded the change of
   owner in its ownership ledger (an OrbitDB database only the server writes).
4. **PvP:** levels are normalized (level 50), and only verified Peerlings can be
   traded or used in PvP.

## Consequences
- Saves survive cleared browsers and can be loaded on any device that has the
  key, and OrbitDB carries both the registry and every player's progress.
- Faked Peerlings and trade duplication are prevented without a game server.
  The server signs results; it does not run the game.
- Trades need the server to be reachable. Catches work offline and are
  verified later.
- New server workload: one replicated OrbitDB database per player, plus
  battle replays (CPU only). If per-player databases don't scale, the fallback is
  IPNS snapshots ([player-data § Storage options](../tech/player-data.md#storage-options-considered));
  the integrity design stays the same.
- The battle engine must be deterministic (BTL-002).

## Alternatives considered
- Browser-only saves (S1), IPNS snapshots (S2), accepting cheating (I1/I2 only),
  and a server-authoritative game (I4). See [player-data](../tech/player-data.md).
