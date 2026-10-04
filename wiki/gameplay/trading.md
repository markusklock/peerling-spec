---
title: Trading
type: system
status: draft
req_prefix: TRD
tags: [gameplay, multiplayer, trading, libp2p]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/peerling-species.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-04
---

# Trading

> How two players exchange Peerlings. Trading moves a
> [Peerling instance](../glossary.md#peerling-instance) from one player's save
> to the other's; the species itself lives on IPFS and needs no transfer.

## Overview

[accepted] Players can trade Peerlings. [proposed] Trades happen peer-to-peer
over a direct libp2p stream between the two players
([realtime-networking](../tech/realtime-networking.md)).

## Flow (proposed)

1. A proposes a trade to B while standing next to them (MPL-006); B accepts
   the session (MPL-005).
2. Both pick the instance(s) they offer. Both see the other's offer live, with
   the species fetched and verified by CID.
3. Both confirm. Any change to an offer resets both confirmations.
4. Both sign a **trade record** (both player IDs, the instances exchanged, a
   timestamp) and send it to the server. The server checks that every offered
   instance is verified and owned by the player offering it, then records the
   trade in the ownership ledger
   ([player-data § Ownership ledger and trades](../tech/player-data.md#ownership-ledger-and-trades-accepted-details-proposed)).
5. Each side then appends a `trade` event to its save, removing the outgoing
   instances and adding the incoming ones with `origin: "trade"`.
6. After the trade, the receiving player's node fetches and keeps
   ([NODE-004](../tech/ipfs-helia.md#requirements)) the species content of what
   it received.

## Integrity

[accepted] A trade completes only when the server's ownership ledger records
it, and only verified Peerlings can be traded
([D-0009](../decisions/D-0009-player-data-on-orbitdb.md)). A modified client
that keeps a copy of a traded Peerling can never trade that copy again. Trades
therefore need the server to be reachable.

## Requirements

- **TRD-001** [accepted] Two players MUST be able to trade Peerlings with each other.
- **TRD-002** [proposed] A trade MUST complete only after both players confirm the final offers; changing an offer MUST clear both confirmations.
- **TRD-003** [proposed] A completed trade MUST produce a trade record signed by both players.

## Open questions

_None at the moment._

## See also

- [Multiplayer](multiplayer.md) · [Catching](catching.md)
