---
title: "D-0005: The generation server is the only registry writer"
type: decision
status: proposed
tags: [tech, orbitdb, security]
related:
  - wiki/tech/orbitdb-registry.md
  - wiki/peerlings/creation-pipeline.md
updated: 2026-10-03
---

# D-0005: The generation server is the only registry writer

**Status:** proposed (awaiting designer review — [Q-002](../open-questions.md#q-002))

## Context
The [registry](../glossary.md#registry) decides which species appear in every
player's world. Clients run in the browser and can be modified by anyone, so a
client-written entry could contain arbitrary stats, moves or unmoderated assets.

## Decision (proposed)
The OrbitDB registry's access controller grants write access only to the
generation server's OrbitDB identity. Clients replicate and read the registry;
they never write to it. Every species record is additionally signed by the
server ([attestation](../glossary.md#attestation)) so the record stays verifiable
even when fetched from a peer outside OrbitDB.

## Consequences
- Only content that went through the pipeline, validation and moderation can
  appear in the game.
- Takedowns are possible by the server appending a tombstone entry.
- The registry is still fully peer-to-peer for *reading* and replication.

## Alternatives considered
- Open write access with client-side validation of every entry: every client
  must re-validate everything, and moderation is impossible.
- Clients write, but entries require a server attestation: equivalent security
  with more moving parts; could be revisited if player-written databases are
  wanted for other features (e.g. encounter stats, [Q-021](../open-questions.md#q-021)).
