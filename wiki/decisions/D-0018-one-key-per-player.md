---
title: "D-0018: One key per player for every identity"
type: decision
status: accepted
tags: [tech, identity, libp2p, orbitdb, ipns, security]
sources:
  - raw/conversations/2026-10-06-formats-approved.md
related:
  - wiki/tech/data-formats.md
  - wiki/tech/player-data.md
  - wiki/gameplay/player-character.md
updated: 2026-10-06
---

# D-0018: One key per player for every identity

**Status:** accepted (2026-10-06)

## Context
A player needs an identity in several systems: the game (player ID), libp2p
(peer ID), OrbitDB (writer of their save log) and IPNS (their profile link).

## Decision
[accepted] Each player has **one Ed25519 key pair**, and its public key is at the
same time their player ID, libp2p peer ID, OrbitDB identity and IPNS name
([data-formats § Identifiers](../tech/data-formats.md#identifiers), FMT-002).

## Consequences
- Simple: no mapping between identities; every pubsub message, OrbitDB entry,
  IPNS record and connection is signed by the player's own key.
- One recovery phrase restores the account, the save-log address and the
  profile link.
- A clear showcase: the player ID *is* the network identity *is* the profile link.
- Accepted downsides:
  - the player ID is visible on every connection, so it can be linked to the
    player's IP address across sessions (WebRTC play already shows IP addresses
    to directly connected players);
  - the account key is used throughout the networking stack, so a bug there
    could leak it, and it can't be rotated without becoming a new player;
  - implementers need a custom OrbitDB identity provider;
  - during a restore, the same peer ID may briefly be online twice.

## Alternatives considered
- An account key plus a fresh libp2p key per session, linked by a short
  certificate signed by the account key: better privacy and key isolation, but
  one more record type and a certificate check in every peer interaction.
  Rejected in favour of simplicity.
