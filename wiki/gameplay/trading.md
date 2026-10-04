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
  - raw/conversations/2026-10-04-decentralize-level-3.md
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
2. Both pick the instance(s) they offer. Both see the other's offer live. Each
   client fetches the offered species by CID and **verifies the offered
   Peerlings**: genuine origin, a valid ownership chain ending at the other
   player, and the other player not flagged
   ([player-data § Verified Peerlings](../tech/player-data.md#verified-peerlings)).
3. Both confirm. Any change to an offer resets both confirmations.
4. Both sign their transfers, and the two signed transfers are written as one
   entry to the open **transfer log**
   ([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades)).
   No server is involved.
5. Each side then appends a `trade` event to its save, removing the outgoing
   instances and adding the incoming ones with `origin: "trade"`.
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
flagged; one transfer stays valid and the other is void
([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades)).

## Requirements

- **TRD-001** [accepted] Two players MUST be able to trade Peerlings with each other.
- **TRD-002** [proposed] A trade MUST complete only after both players confirm the final offers; changing an offer MUST clear both confirmations.
- **TRD-003** [proposed] A completed trade MUST produce a trade record signed by both players.
- **TRD-004** [accepted] Trades MUST NOT require the operator server; each side MUST verify the other's offered Peerlings before signing.

## Open questions

_None at the moment._

## See also

- [Multiplayer](multiplayer.md) · [Catching](catching.md)
