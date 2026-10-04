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

### Beacon
[proposed] A random value the server publishes and signs every few minutes.
Encounter seeds include it, so players can't choose their encounters. See
[player-data](tech/player-data.md#verification).

### Biome
A region type of the procedural world (e.g. forest, desert) that influences
which Peerlings are encountered there. See
[procedural-generation](world/procedural-generation.md).

### Catch attestation
The server's signature confirming that a Peerling instance was caught
legitimately. The server checks this by replaying the battle. See
[player-data](tech/player-data.md#verification).

### CID
Content Identifier — the IPFS address of a piece of content, derived from a hash
of the content itself. The same bytes always have the same CID, so content can be
fetched from any peer and verified locally.

### Concept
The structured description of a new Peerling produced by the
[concept LLM](#concept-llm) from the player's free-text wish: name, appearance,
lore, etc. See [creation-pipeline](peerlings/creation-pipeline.md).

### Concept LLM
The small, self-hosted language model on the generation server that turns
player wishes into concepts and later assigns types and moves.

### Creator
The player who designed a Peerling [species](#species). The creator is recorded
in the species record and credited in-game.

### Emote
One of a fixed set of gestures or expressions a player can show to nearby
players. Emotes are the only way players communicate; there is no chat. See
[multiplayer](gameplay/multiplayer.md#communication).

### Encounter
A meeting with a [wild Peerling](#wild-peerling) during exploration, which
leads to a battle. See [encounters](gameplay/encounters.md).

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

### Ownership ledger
An OrbitDB database, written only by the server, that records who owns
each verified Peerling instance and every trade. See
[player-data](tech/player-data.md#ownership-ledger-and-trades-accepted-details-proposed).

### Peerling
A creature in the game. The word is ambiguous between a *species* and an
individual *instance*; when the distinction matters, the spec says
[species](#species) or [Peerling instance](#peerling-instance).
Plural: Peerlings. The game itself is also called *Peerlings*.

### Peerling instance
[proposed] One individual Peerling owned by a player (e.g. the starter, or a
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

### Signature move
[proposed] The move in a species' *signature* slot: its characteristic special
attack, always of the species' primary type. See [moves](peerlings/moves.md#move-slots).

### Species
[proposed] A Peerling design: the immutable, content-addressed definition (name,
description, types, base stats, moves, image, 3D model) created once by its
creator through the creation pipeline. See
[peerling-species](peerlings/peerling-species.md).

### Species record
[proposed] The JSON document stored on IPFS that defines a species. Its CID is
the species' identity. See [peerling-species](peerlings/peerling-species.md).

### Starter
The first Peerling a player owns: an instance of the species the player created
during onboarding. See [onboarding](gameplay/onboarding.md).

### Trade
An exchange of [Peerling instances](#peerling-instance) between two players.
See [trading](gameplay/trading.md).

### Type
An elemental category from a predefined list that determines battle
strengths and weaknesses. See [types](peerlings/types.md).

### Verified Peerling
A Peerling instance with a valid [catch attestation](#catch-attestation).
Only verified Peerlings can be traded or used in PvP.

### Wild Peerling
An unowned Peerling instance met during exploration, generated from a species
in the registry. See [encounters](gameplay/encounters.md).
