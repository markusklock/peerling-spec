---
title: Glossary
type: reference
status: draft
tags: [reference, terminology]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
updated: 2026-10-04
---

# Glossary

> Canonical definitions of every game and technology term used in the spec.
> Pages link here on first use of a term. Terms are alphabetical; each heading is
> a link anchor.

### Attestation
[proposed] A signature by the [generation server](#generation-server) over a
[species record](#species-record), proving the record was produced by the
official pipeline and has not been altered. See
[orbitdb-registry](tech/orbitdb-registry.md).

### Biome
One of the 12 region types of the world, one per Peerling type (e.g. Forest
for Grass). Peerlings of a biome's type are more likely to be encountered
there. See [procedural-generation](world/procedural-generation.md#biomes).

### Catch evidence
The record of a catch in the catcher's save log (encounter number, epoch
record, position, candidate, battle actions). Anyone can replay it to verify
the catch. See [player-data](tech/player-data.md#catches-accepted-details-proposed).

### CID
Content Identifier — the IPFS address of a piece of content, derived from a hash
of the content itself. The same bytes always have the same CID, so content can be
fetched from any peer and verified locally.

### Collection
All the Peerlings a player owns that are not in their [team](#team). It has no
size limit. See [catching](gameplay/catching.md#team-and-collection).

### Concept
The structured description of a new Peerling produced by the
[concept LLM](#concept-llm) from the player's free-text wish: name, appearance,
lore, etc. See [creation-pipeline](peerlings/creation-pipeline.md).

### Concept LLM
The small, self-hosted language model on the generation server that turns
player wishes into concepts and later assigns types and moves.

### Creation Shrine
A place at the world's spawn where a player gives up 3 Peerlings in exchange
for creating a new species. See [creation-shrine](gameplay/creation-shrine.md).

### Creator
The player who designed a Peerling [species](#species). The creator is recorded
in the species record and credited in-game.

### drand
A public, verifiable randomness beacon run by the League of Entropy. The
[epoch record](#epoch-record) takes its randomness from drand. See
[player-data](tech/player-data.md#encounter-seeds).

### Emote
One of a fixed set of gestures or expressions a player can show to nearby
players. Emotes are the only way players communicate; there is no chat. See
[multiplayer](gameplay/multiplayer.md#communication).

### Encounter
A meeting with a [wild Peerling](#wild-peerling) during exploration, which
leads to a battle. See [encounters](gameplay/encounters.md).

### Encounter candidates
The ordered list of up to 5 species drawn for one wild encounter; the
encounter uses the first one that could be fetched. See
[encounters § Candidates](gameplay/encounters.md#candidates).

### Encounter foliage
The biome-specific tall grass (or similar) where wild encounters happen. See
[exploration § Wild encounters in foliage](gameplay/exploration.md#wild-encounters-in-foliage).

### Epoch record
[proposed] A record the server signs and publishes every 5 minutes (one
*epoch*). It holds a drand random value and the current registry height, and
encounter seeds are derived from it, so players can't choose their encounters.
See [player-data](tech/player-data.md#encounter-seeds).

### Generation server
The single operator-hosted server that runs the concept LLM, the image
generator, the image-to-3D generator, and pins all game content on IPFS. See
[generation-server](tech/generation-server.md).

### Helia
A TypeScript implementation of IPFS that runs in the browser. Every game client
runs a Helia node. See [ipfs-helia](tech/ipfs-helia.md).

### IPFS
The InterPlanetary File System — a peer-to-peer network for storing and sharing
content addressed by [CID](#cid).

### Move
An attack or action a Peerling can use in battle. Every move is an instance of a
[move template](#move-template). See [moves](peerlings/moves.md).

### Move slot
One of the three fixed roles in every species' move set: *quick*, *strong*
and *signature*. See [moves](peerlings/moves.md#move-slots).

### Move template
A predefined, balanced pattern (power range, accuracy, effects, …) that
generated moves must follow. See [moves](peerlings/moves.md).

### Operator
The person running the game's [generation server](#generation-server): the
game's designer. The operator also creates the launch
[seed species](#seed-species).

### OrbitDB
A peer-to-peer database built on IPFS and libp2p. The game's
[registry](#registry) of all Peerling species is an OrbitDB database. See
[orbitdb-registry](tech/orbitdb-registry.md).

### Origin attestation
The server's signature on a starter or a Creation Shrine Peerling, proving
where it came from (these don't come from a catch, so there's nothing to
replay). See [player-data](tech/player-data.md#starters-and-shrine-creations).

### Peerling
A creature in the game. The word is ambiguous between a *species* and an
individual *instance*; when the distinction matters, the spec says
[species](#species) or [Peerling instance](#peerling-instance).
Plural: Peerlings. The game itself is also called *Peerlings*.

### Peerling instance
One individual Peerling owned by a player (e.g. the starter, or a
caught wild Peerling), with its own level, experience, current HP, etc. Many
instances can exist of the same species. See
[peerling-species](peerlings/peerling-species.md).

### Pin / pinning
Telling an IPFS node to keep a piece of content permanently and serve it to
others. The generation server pins all game content so every CID is always
reachable from at least one node.

### Player character
The avatar a player controls in the world. See
[player-character](gameplay/player-character.md).

### Presence
[proposed] The live broadcast of a player's position in the shared world, sent
to nearby players over libp2p pubsub. See
[realtime-networking](tech/realtime-networking.md#presence-proposed).

### PvP battle
A battle between two players' Peerlings, played peer-to-peer. See
[pvp-battles](gameplay/pvp-battles.md).

### Recovery phrase
A list of words shown to the player once, from which their identity key can
be restored on another device. See [player-data](tech/player-data.md).

### Region
[proposed] A square area of the world made of several chunks. It is the unit
for presence topics in multiplayer. See
[procedural-generation](world/procedural-generation.md).

### Registry
The OrbitDB database that lists every published Peerling species. See
[orbitdb-registry](tech/orbitdb-registry.md).

### Save log
A player's save: a per-player OrbitDB event log, written only by that
player and replicated by the server. See [player-data](tech/player-data.md#save-log).

### Seed species
One of the handful of species the [operator](#operator) creates at launch,
through the normal creation pipeline, so the first players have Peerlings to
meet. See [D-0002](decisions/D-0002-all-peerlings-user-generated.md).

### Shimmer
A rare (1 in 500), purely cosmetic color variant of an individual Peerling. See
[peerling-species § Individual variation](peerlings/peerling-species.md#individual-variation).

### Signature move
[proposed] The move in a species' *signature* slot: its characteristic special
attack, always of the species' primary type. See [moves](peerlings/moves.md#move-slots).

### Species
A Peerling design: the immutable, content-addressed definition (name,
description, types, base stats, moves, image, 3D model) created once by its
creator through the creation pipeline. See
[peerling-species](peerlings/peerling-species.md).

### Species record
The document stored on IPFS that defines a species. Its CID is
the species' identity. See [peerling-species](peerlings/peerling-species.md).

### Species stats
Per-species counters (encounters, catches, owners, trades, providers) that the
server publishes in an OrbitDB database for creators and species cards. See
[creator-feedback](gameplay/creator-feedback.md).

### Starter
The first Peerling a player owns: an instance of a species the player created
during onboarding, or of an existing species they chose instead. See
[onboarding](gameplay/onboarding.md).

### Team
The Peerlings a player brings into battles: up to 4. See
[catching](gameplay/catching.md#team-and-collection).

### Tile
One cell of the invisible grid the world is laid out on; players move one tile
at a time. See [exploration § Grid movement](gameplay/exploration.md#grid-movement).

### Trade
An exchange of [Peerling instances](#peerling-instance) between two players.
See [trading](gameplay/trading.md).

### Trait
An individual Peerling's fixed modifier of −10% to +10% on one of its four
stats. See [peerling-species § Individual variation](peerlings/peerling-species.md#individual-variation).

### Transfer log
The open OrbitDB log of signed ownership transfers. Following a Peerling's
chain of transfers gives its current owner; conflicting transfers expose a
cheater. See [player-data](tech/player-data.md#transfer-log-and-trades).

### Type
An elemental category from a predefined list that determines battle
strengths and weaknesses. See [types](peerlings/types.md).

### Verified Peerling
A Peerling whose origin is genuine (its catch replays correctly, or it has a
valid [origin attestation](#origin-attestation)) and whose ownership chain in
the [transfer log](#transfer-log) is valid. Only verified Peerlings can be
traded or used in PvP. Any player can check this.

### WebRTC
A browser technology for direct, UDP-based connections between two browsers.
libp2p uses it to connect players' browsers directly. See
[tech-stack](tech/tech-stack.md#networking).

### WebTransport
A modern browser API for connecting to a server over HTTP/3 (QUIC, which runs
on UDP). Browsers use it to connect to the operator server; it replaces
WebSockets. See [tech-stack](tech/tech-stack.md#networking).

### Wild Peerling
An unowned Peerling instance met during exploration, generated from a species
in the registry. See [encounters](gameplay/encounters.md).
