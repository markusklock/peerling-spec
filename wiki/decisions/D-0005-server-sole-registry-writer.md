---
title: "D-0005: The generation server is the only registry writer"
type: decision
status: superseded by D-0013
tags: [tech, orbitdb, security]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-05-proposal-review-1.md
related:
  - wiki/tech/orbitdb-registry.md
  - wiki/peerlings/creation-pipeline.md
updated: 2026-10-05
---

# D-0005: The generation server is the only registry writer

**Status:** superseded by [D-0013](D-0013-peer-verified-registry-catches-trades.md)
(2026-10-04): players now append registry entries, which are valid only with a
server signature. Originally accepted 2026-10-03, resolving
[Q-002](../open-questions.md#q-002).
The attestation (signature) part below was accepted on 2026-10-05.

## Context
The [registry](../glossary.md#registry) decides which species appear in every
player's world. Clients run in the browser and can be modified by anyone, so a
client-written entry could contain arbitrary stats, moves or unmoderated assets.

## Decision
- [accepted] The OrbitDB registry's access controller grants write access only
  to the generation server's OrbitDB identity. Clients replicate and read the
  registry; they never write to it.
- [accepted] Every species record is additionally signed by the server
  ([attestation](../glossary.md#attestation)), so the record stays verifiable
  when it is fetched from a peer outside OrbitDB, e.g. during a
  [PvP battle](../gameplay/pvp-battles.md) or [trade](../gameplay/trading.md).

## Consequences
- Only content that went through the pipeline and validation can appear in
  the game. (There is no moderation stage: [D-0010](D-0010-no-content-moderation.md).)
- Takedowns are possible by the server appending a tombstone entry.
- The registry is still fully peer-to-peer for *reading* and replication.
- Players still publish the *content* to IPFS themselves
  ([D-0007](D-0007-players-publish-assets.md)); the server controls only the
  *listing*.

## Alternatives considered
- Open write access with client-side validation of every entry: every client
  must re-validate everything, and moderation is impossible.
- Clients write, but entries require a server attestation: equivalent security
  with more moving parts; could be revisited if player-written databases are
  wanted for other features (e.g. encounter stats, [Q-021](../open-questions.md#q-021)).
