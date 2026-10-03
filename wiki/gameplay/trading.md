---
title: Trading
type: system
status: draft
req_prefix: TRD
tags: [gameplay, multiplayer, trading, libp2p]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/peerling-species.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-03
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

1. A proposes a trade to B; B accepts the session (MPL-005).
2. Both pick the instance(s) they offer. Both see the other's offer live, with
   the species fetched and verified by CID.
3. Both confirm. Any change to an offer resets both confirmations.
4. Both sign a **trade record** (both player IDs, the instances exchanged, a
   timestamp). Each side removes its outgoing instances and adds the incoming
   ones, with `origin: "trade"`.
5. After the trade, the receiving player's node fetches and keeps
   ([NODE-004](../tech/ipfs-helia.md#requirements)) the species content of what
   it received.

## Integrity

Saves live in the browser, so a modified client could "trade" a Peerling and
keep a copy. Whether that matters, and how to prevent it, is open:
[Q-026](../open-questions.md#q-026).

## Requirements

- **TRD-001** [accepted] Two players MUST be able to trade Peerlings with each other.
- **TRD-002** [proposed] A trade MUST complete only after both players confirm the final offers; changing an offer MUST clear both confirmations.
- **TRD-003** [proposed] A completed trade MUST produce a trade record signed by both players.

## Open questions

[Q-026](../open-questions.md#q-026) · [Q-028](../open-questions.md#q-028)

## See also

- [Multiplayer](multiplayer.md) · [Catching](catching.md)
