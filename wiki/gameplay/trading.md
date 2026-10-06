---
title: Trading
type: system
status: accepted
req_prefix: TRD
tags: [gameplay, multiplayer, trading, libp2p]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-06-review-2-decisions.md
related:
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/peerling-species.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-06
---

# Trading

> How two players exchange Peerlings. Trading moves a
> [Peerling instance](../glossary.md#peerling-instance) from one player's save
> to the other's; the species itself lives on IPFS and needs no transfer.

## Overview

[accepted] Players can trade Peerlings. [accepted] Trades happen peer-to-peer
over a direct libp2p stream between the two players
([realtime-networking](../tech/realtime-networking.md)).

## Flow

Exact messages: [protocols](../tech/protocols.md).

1. A proposes a trade to B while standing next to them (MPL-006); B accepts
   the session (MPL-005).
2. Both pick the instance(s) they offer. Each offer carries the full instance
   data, so the other side can show it at once and only needs to check it
   against the original owner's save log and the transfer log ([accepted]
   2026-10-06; fields in [protocols](../tech/protocols.md#peerlingstrade100--trade)).
   Both see the other's offer live,
   including each Peerling's level, stat traits and whether it is a shimmer
   ([peerling-species § Individual variation](../peerlings/peerling-species.md#individual-variation)). Each
   client fetches the offered species by CID and **verifies the offered
   Peerlings**: genuine origin, a valid ownership chain ending at the other
   player, and the other player not flagged
   ([player-data § Verified Peerlings](../tech/player-data.md#verified-peerlings)).
3. Both confirm. Any change to an offer resets both confirmations.
4. Both sign their transfers (each one names the hash of both final offers, so
   a trade is all or nothing:
   [data-formats § Transfer](../tech/data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)), and all signed transfers are written as one
   entry to the open **transfer log**
   ([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades)).
   No server is involved.
   [accepted] Once both sides have exchanged their signed transfers, the trade
   can no longer be cancelled: either side may write the entry, and does so if
   it hasn't appeared, even if the session dropped (2026-10-06).
5. Each side then appends a `trade` event to its save, removing the outgoing
   instances and adding the incoming ones (their `origin` stays as it was;
   trades don't change it). A client keeps the received offer data until it
   has written this event, and at every session start it checks the transfer
   log (or the ownership index) for its own Peerlings and writes any `trade`
   event that is missing.
6. After the trade, the receiving player's node fetches and keeps
   ([NODE-004](../tech/ipfs-helia.md#requirements)) the species content of what
   it received.

## Integrity

[accepted] Only verified Peerlings can be traded, and ownership is a signed
transfer chain that any player can check ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)). Trades work
without the operator server.

A modified client that "keeps a copy" can't trade it again: the transfer log
shows it already belongs to someone else. A player who gives the same Peerling
to two people at once is detected by their two conflicting signatures and
flagged; one transfer stays valid and the other is void. Re-posting someone's
old, identical transfer is never a conflict, so it can't get an honest player
flagged
([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades)).

## Requirements

- **TRD-001** [accepted] Two players MUST be able to trade Peerlings with each other.
- **TRD-002** [accepted] A trade MUST complete only after both players confirm the final offers; changing an offer MUST clear both confirmations.
- **TRD-003** [accepted] A completed trade MUST produce a trade record signed by both players.
- **TRD-004** [accepted] Trades MUST NOT require the operator server; each side MUST verify the other's offered Peerlings before signing.
- **TRD-005** [accepted] A trade MUST NOT be cancellable once both sides have exchanged signed transfers; either side MUST append the entry if it is missing, and each client MUST at session start write any `trade` event missing for its own Peerlings.

## Open questions

_None at the moment._

## See also

- [Multiplayer](multiplayer.md) · [Catching](catching.md)
