---
title: "D-0013: Peer-verified registry, catches and trades"
type: decision
status: accepted
tags: [tech, orbitdb, decentralization, security]
sources:
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
related:
  - wiki/tech/player-data.md
  - wiki/tech/orbitdb-registry.md
  - wiki/gameplay/trading.md
  - wiki/tech/resilience.md
  - wiki/decisions/D-0005-server-sole-registry-writer.md
  - wiki/decisions/D-0009-player-data-on-orbitdb.md
updated: 2026-10-04
---

# D-0013: Peer-verified registry, catches and trades

**Status:** accepted (2026-10-04). Supersedes the "server is the only registry
writer" part of [D-0005](D-0005-server-sole-registry-writer.md), and the
server-signed catches and ownership ledger of
[D-0009](D-0009-player-data-on-orbitdb.md). Details are [proposed].

## Context
The designer wants the game to work as far as possible without the operator
server ([resilience](../tech/resilience.md)). Under D-0005 and D-0009 the server
was needed to list new species, verify catches and record trades. Of these, only
*creating* a Peerling truly needs the server: generation needs GPUs, and AI
output can't be re-checked by anyone else.

## Decision
"Level 3" from the 2026-10-04 brainstorm:

1. **Registry writes by players.** [accepted] Any player may append a registry
   entry, but every node only accepts entries that carry a valid server
   signature over the species record. The server signs at the end of creation.
2. **Catches verified by anyone.** [accepted] A catch is deterministic and its
   evidence is public in the catcher's save log, so any player (e.g. a trade
   partner or PvP opponent) verifies it by replaying it. The server is no
   longer required for this.
3. **Trades without a server.** [accepted] Every Peerling carries a signed
   chain of ownership transfers, stored in an open OrbitDB transfer log.
   Double trades (one Peerling given to two players) can't be prevented without
   global consensus, but they are detected: the two conflicting signatures
   prove who cheated, a fixed rule decides which transfer counts, and the
   cheater is flagged.

Full details: [player-data § Verification](../tech/player-data.md#verification)
and [orbitdb-registry](../tech/orbitdb-registry.md).

## Consequences
- Without the server, everything except creating new Peerlings (and the
  Creation Shrine, onboarding new players, and creator stats) keeps working,
  trades included.
- Cheating in trades is detected after the fact rather than prevented. This is
  acceptable because nothing in the game is scarce: every species can be caught
  by anyone and all species are equally strong (but see
  [Q-037](../open-questions.md#q-037) on individual variation).
- Browsers do more work: replaying catches and checking transfer chains.
  Results can be cached per Peerling.
- The server is still trusted to sign species records, starters and shrine
  creations, and to publish epoch records (with a drand-based fallback).

## Alternatives considered
- Levels 0–2 (keeping some of the server's roles): simpler, but trades or
  catch checks stop when the server is down.
- Ownership on a blockchain (e.g. Filecoin's smart-contract layer): prevents
  double trades, but brings wallets, fees and complexity. Rejected as too heavy
  for a casual game.
