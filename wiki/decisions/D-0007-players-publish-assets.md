---
title: "D-0007: The player's browser publishes their Peerling to IPFS"
type: decision
status: accepted
tags: [tech, ipfs, helia, creation]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/generation-server.md
updated: 2026-10-03
---

# D-0007: The player's browser publishes their Peerling to IPFS

**Status:** accepted (2026-10-03, resolves [Q-003](../open-questions.md#q-003))

## Context
The generation server produces a new Peerling's image, 3D model and data, so
it could add them to IPFS itself. But showing off IPFS is one of the game's
two main goals ([overview](../overview.md#goals)), and the brief describes the
server "autopinning all assets pushed to IPFS by users".

## Decision
[accepted] The player's browser adds their new Peerling's assets and species
record to IPFS through its own [Helia](../glossary.md#helia) node. The server
then pins them by CID, fetching the content from the player's node over IPFS.
The server still writes the registry entry
([D-0005](D-0005-server-sole-registry-writer.md)).

[proposed] The exact handshake, including the server checking that the pinned
content matches what it generated, is specified in
[creation-pipeline § Stage 7](../peerlings/creation-pipeline.md#stage-7--publish).

## Consequences
- Every player's first real IPFS act is publishing their own creation, so
  their node is the first provider of it.
- The publish step needs the player's browser online until the server has
  pinned everything. The job must survive disconnects (CRE-014).
- The server must not trust the client: it checks the fetched content before
  pinning and listing it.

## Alternatives considered
- Server adds and pins, client only fetches: more reliable but hides IPFS.
  Rejected by the designer.
