---
title: Glossary
type: reference
status: accepted
tags: [reference, terminology]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-pvp-wins-hex-world.md
  - raw/conversations/2026-10-06-v1-fun-features.md
  - raw/conversations/2026-10-06-fun-features-approved.md
  - raw/conversations/2026-10-06-network-performance-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
updated: 2026-10-06
---

# Glossary

> Canonical definitions of every game and technology term used in the spec.
> Pages link here on first use of a term. Terms are alphabetical; each heading is
> a link anchor.

### Attestation
[accepted] A signature by the [generation server](#generation-server) over a
[species record](#species-record), proving the record was produced by the
official pipeline and has not been altered. See
[orbitdb-registry](tech/orbitdb-registry.md).

### Backup file
A `.car` file the player can export, holding their private key and latest save
snapshot. See [data-formats § Backup file](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup).

### Badge
One of 12 rewards, one per biome, for beating that biome's guardian. See
[guardians](gameplay/guardians.md).

### Biome
One of the 12 area types of the world, one per Peerling type (e.g. Forest
for Grass). Peerlings of a biome's type are more likely to be encountered
there. See [procedural-generation](world/procedural-generation.md#biomes).

### Biome area
One area of the world (about 300–500 m across) with a single biome, its own
rest point and its own landmark. See [procedural-generation §
Layout](world/procedural-generation.md#layout).

### Biome index
A biome's number 0–11, its row in the biome table (Plains = 0); used in hashes and data formats. See
[procedural-generation § Biomes](world/procedural-generation.md#biomes).

### Biome sector
One of the 12 wedge-shaped slices of the world, one per biome, running from
the spawn hexagon to the border. See [procedural-generation §
Layout](world/procedural-generation.md#layout).

### Block list
A player's local list of blocked players: their characters, names and emotes are hidden and their requests are declined as "busy". See
[multiplayer § What players experience](gameplay/multiplayer.md#what-players-experience).

### Bootstrap peer
A known node a client connects to first, to find other peers. The operator
server is the main one; public IPFS bootstrap nodes are fallbacks. See [ipfs-
helia § Connectivity](tech/ipfs-helia.md#connectivity).

### Catch evidence
The record of a catch in the catcher's save log (encounter number, epoch
record, position, candidate, battle actions). Anyone can replay it to verify
the catch. See [player-data](tech/player-data.md#catches).

### Chunk
A 32 × 32-tile (64 m × 64 m) piece of the world, generated on demand. Each
chunk is also a [region](#region). See [procedural-
generation](world/procedural-generation.md#generation-basics).

### CID
Content Identifier — the IPFS address of a piece of content, derived from a hash
of the content itself. The same bytes always have the same CID, so content can be
fetched from any peer and verified locally.

### Collection
All the Peerlings a player owns that are not in their [team](#team). It has no
size limit. See [catching](gameplay/catching.md#team-and-collection).

### Concept
The structured description of a new Peerling produced by the
[concept LLM](#concept-llm) from the player's free-text wish: name
suggestions, summary, lore, types, appearance and temperament. See
[creation-pipeline § Stage 2](peerlings/creation-pipeline.md#stage-2--concept).

### Concept LLM
The small, self-hosted language model on the generation server. It turns
player wishes into concepts (including the Peerling's types) and later spreads
the stats and writes the moves.

### Creation Shrine
A place at the world's spawn where a player gives up 3 Peerlings in exchange
for creating a new species. See [creation-shrine](gameplay/creation-shrine.md).

### Creator
The player who designed a Peerling [species](#species). The creator is recorded
in the species record and credited in-game.

### Cry
The sound a Peerling makes, synthesized in the browser from its species CID and
type. See [audio § Peerling cries](world/audio.md#peerling-cries).

### Day
Two different days: the **in-game day** of the day/night cycle lasts 2 hours (24 epochs); the **Peerling of the Day** changes every real 24 hours (288 epochs, from 00:00 UTC). See
[procedural-generation § Day and night](world/procedural-generation.md#day-and-night), [peerling-of-the-day](gameplay/peerling-of-the-day.md).

### Delisting
The operator's emergency removal of a species from the registry with a signed
tombstone entry; clients stop showing it. Not moderation. See [orbitdb-registry
§ Design](tech/orbitdb-registry.md#design).

### Double trade
Two different transfers of the same Peerling with the same `prev`, both signed by its owner: proof of cheating. The lower one wins and the owner is flagged. See
[player-data § Transfer log and trades](tech/player-data.md#transfer-log-and-trades).

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

### Encounter number
A player's gap-free counter of wild (and guardian) encounters, 0, 1, 2, …, used in the encounter seed. See
[player-data § Encounter seeds](tech/player-data.md#encounter-seeds).

### Encounter seed
The random value that determines one wild encounter (its candidates, level,
traits, shimmer roll and battle rolls), derived from the epoch record, the
player ID and the encounter number. See [player-data § Encounter
seeds](tech/player-data.md#encounter-seeds).

### Epoch
A 5-minute time slot (Unix seconds ÷ 300). The server publishes an [epoch
record](#epoch-record) for each one. See [player-data § Encounter
seeds](tech/player-data.md#encounter-seeds).

### Epoch log
The server-written OrbitDB events database of all signed epoch records. See
[player-data § Encounter seeds](tech/player-data.md#encounter-seeds).

### Epoch record
[accepted] A record the server signs and publishes every 5 minutes (one
*epoch*). It holds a drand random value, the current registry height, the
rules and world-generator versions and the roots of the server's snapshots and
indexes; encounter seeds are derived from it, so players can't choose their
encounters. When the server is offline, clients derive the record themselves
(a *client-derived* epoch record).
See [player-data](tech/player-data.md#encounter-seeds).

### Faint
A Peerling at 0 HP faints and can't fight until healed at a rest point. See
[battle § Rules](gameplay/battle.md#rules).

### Fair mode
The default PvP level mode: every Peerling fights at level 50. See [pvp-battles
§ Fairness](gameplay/pvp-battles.md#fairness).

### Fast path
An HTTP shortcut through the operator server (latest epoch record, log download, checkpoints, IPNS records) that returns signed or content-addressed data and always has a peer-to-peer fallback. See
[network-performance](tech/network-performance.md#fast-paths-through-the-operator).

### First found in the wild
The credit on a species card naming the first player with a verified wild catch of it. See
[creator-feedback § First found in the wild](gameplay/creator-feedback.md#first-found-in-the-wild).

### Flagged player
A player proven to have made a double trade; clients refuse trades and PvP battles with them. See
[player-data § Transfer log and trades](tech/player-data.md#transfer-log-and-trades).

### Following Peerling
The first non-fainted Peerling in the player's team, which walks behind them in the world and is visible to others. See
[exploration § Following Peerling](gameplay/exploration.md#following-peerling).

### Generation server
The single operator-hosted server that runs the concept LLM, the image
generator, the image-to-3D generator, and pins all game content on IPFS. See
[generation-server](tech/generation-server.md).

### Generator version
The version of the world generator. All clients switch at an epoch announced in
the epoch records. See [procedural-generation § Generator
updates](world/procedural-generation.md#generator-updates).

### Guardian
The keeper of a biome's guardian site, whose team of 4 of that biome's Peerlings changes weekly; beating it earns the biome's badge. See
[guardians](gameplay/guardians.md).

### Guardian site
The stone platform in each biome sector where its guardian is challenged; it counts as its area's landmark. See
[guardians § Guardian sites](gameplay/guardians.md#guardian-sites).

### Helia
A TypeScript implementation of IPFS that runs in the browser. Every game client
runs a Helia node. See [ipfs-helia](tech/ipfs-helia.md).

### IPFS
The InterPlanetary File System — a peer-to-peer network for storing and sharing
content addressed by [CID](#cid).

### IPNS
The InterPlanetary Name System: a fixed name, derived from a key, that points to
changing IPFS content through signed records. Player profiles use it. See
[sharing](gameplay/sharing.md).

### Landmark
A large, unique feature with a generated name in each biome area (e.g.
"Whispering Falls"), used for orientation. See
[procedural-generation § Points of interest](world/procedural-generation.md#points-of-interest).

### Level
A Peerling instance's level, 1–50, raised by XP. It scales its stats. See
[battle § Experience and levelling](gameplay/battle.md#experience-and-levelling).

### libp2p
The peer-to-peer networking library under IPFS. Each client runs a libp2p node
for connections, pubsub and direct streams. See [realtime-
networking](tech/realtime-networking.md).

### Move
An attack or action a Peerling can use in battle. Every move is an instance of a
[move template](#move-template). See [moves](peerlings/moves.md).

### Move slot
One of the three fixed roles in every species' move set: *quick*, *strong*
and *signature*. See [moves](peerlings/moves.md#move-slots).

### Move template
A predefined, balanced pattern (power range, accuracy, effects, …) that
generated moves must follow. See [moves](peerlings/moves.md).

### Network monument
A crystal tree in the spawn hub that shows the player's live network activity.
See [procedural-generation § Spawn hub](world/procedural-generation.md#spawn-hub).

### New Peerlings gallery
Pedestals in the spawn hub showing the newest published Peerlings, loaded live
from IPFS. See [procedural-generation § Spawn hub](world/procedural-generation.md#spawn-hub).

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

### Ownership index
The operator's hourly sharded map from instance ID to its latest transfer, so verifiers needn't replicate the whole transfer log. See
[network-performance](tech/network-performance.md#3-ownership-index).

### Peerdex
The player's index of species they have seen and caught, plus their own
creations. See [peerdex](gameplay/peerdex.md).

### Peerling
A creature in the game. The word is ambiguous between a *species* and an
individual *instance*; when the distinction matters, the spec says
[species](#species) or [Peerling instance](#peerling-instance).
Plural: Peerlings. The game itself is also called *Peerlings*.

### Peerling card
The detail view of a species: 3D model, stats, moves, lore, creator, world
stats and CID. Shareable as a link. See [peerdex](gameplay/peerdex.md) and
[sharing](gameplay/sharing.md).

### Peerling instance
One individual Peerling owned by a player (e.g. the starter, or a
caught wild Peerling), with its own level, experience, current HP, etc. Many
instances can exist of the same species. See
[peerling-species](peerlings/peerling-species.md).

### Peerling of the Day
The species picked each day (24 hours) by the shared randomness; it appears more often everywhere and stands on a pedestal near the spawn. See
[peerling-of-the-day](gameplay/peerling-of-the-day.md).

### Peerlings Viewer
A small web app, published on IPFS, that shows shared Peerling cards and player
profiles outside the game. See [sharing](gameplay/sharing.md).

### Phone backup
A copy of a player's key and save carried on their phone, made and restored by
scanning a QR code on the computer. See
[player-data § Phone backup](tech/player-data.md#phone-backup).

### Pin / pinning
Telling an IPFS node to keep a piece of content permanently and serve it to
others. The generation server pins all game content so every CID is always
reachable from at least one node.

### Player character
The avatar a player controls in the world. See
[player-character](gameplay/player-character.md).

### Player ID
A player's identity: the libp2p peer ID of their Ed25519 key, which is also
their OrbitDB identity and IPNS name. See [D-0018](decisions/D-0018-one-key-per-player.md).

### Presence
[accepted] The live broadcast of a player's position in the shared world, sent
to nearby players over libp2p pubsub. See
[realtime-networking](tech/realtime-networking.md#presence).

### Pubsub
Publish/subscribe messaging (libp2p gossipsub): messages on a topic reach
everyone subscribed. Used for presence, epoch records, the world feed,
spectating and more. See [protocols](tech/protocols.md#pubsub-topics).

### PvP battle
A battle between two players' Peerlings, played peer-to-peer. See
[pvp-battles](gameplay/pvp-battles.md).

### Real-levels mode
The opt-in PvP level mode where Peerlings fight at their actual (unverified)
levels. See [pvp-battles § Fairness](gameplay/pvp-battles.md#fairness).

### Recovery phrase
A list of words shown to the player once, from which their identity key can
be restored on another device. See [player-data](tech/player-data.md#account-recovery).

### Region
[accepted] A 64 m × 64 m square of the world (one chunk of 32 × 32 tiles). It is the unit
for presence topics in multiplayer. See
[procedural-generation](world/procedural-generation.md).

### Registry
The OrbitDB database that lists every published Peerling species. See
[orbitdb-registry](tech/orbitdb-registry.md).

### Registry height
The highest registry `seq` in force at an epoch, which fixes which species are eligible. See
[player-data § Encounter seeds](tech/player-data.md#encounter-seeds).

### Registry index
A compact, chunked list of every registry entry that lets a new client start encounters within seconds. See
[network-performance](tech/network-performance.md#1-registry-index).

### Relay
A libp2p node (mainly the operator server) that passes traffic between browsers
so they can find each other and set up direct WebRTC connections. See [ipfs-
helia § Connectivity](tech/ipfs-helia.md#connectivity).

### Rest point
A beacon in every biome area that fully heals the player's team and becomes
their respawn point. See
[exploration § Healing and rest points](gameplay/exploration.md#healing-and-rest-points).

### Retrieval ladder
The order and delays in which the client asks local storage, peers, the operator's gateway and public gateways for content by CID. See
[network-performance](tech/network-performance.md#retrieval-ladder).

### Rules version
The version number of the battle rules. Epoch records announce it, and every battle uses the version active in its epoch, so old catches still verify. See
[battle § Rules versions](gameplay/battle.md#rules-versions).

### Save log
A player's save: a per-player OrbitDB event log, written only by that
player and replicated by the server. See [player-data](tech/player-data.md#save-log).

### Save snapshot
A full copy of a player's save at one moment, referenced from the save log so
loading doesn't replay everything. See [data-formats § Save
snapshot](tech/data-formats.md#save-snapshot--peerlingssave).

### Seed species
One of the handful of species the [operator](#operator) creates at launch,
through the normal creation pipeline, so the first players have Peerlings to
meet. See [D-0002](decisions/D-0002-all-peerlings-user-generated.md).

### Shimmer
A rare (1 in 500), purely cosmetic color variant of an individual Peerling. See
[peerling-species § Individual variation](peerlings/peerling-species.md#individual-variation).

### Shrine credit
The right to one new Creation Shrine job without a new offering, kept for 30 days when a shrine job is abandoned or expires. See
[creation-shrine](gameplay/creation-shrine.md).

### Signature move
[accepted] The move in a species' *signature* slot: its characteristic special
attack, always of the species' primary type. See [moves](peerlings/moves.md#move-slots).

### Signed envelope
The common wrapper for records that must be verifiable on their own: version,
type, signer, body and an Ed25519 signature. See
[data-formats § Signed envelope](tech/data-formats.md#signed-envelope).

### Size class
A species' `small`, `medium` or `large` class, which sets the height its model is drawn at. See
[peerling-species § Size and temperament](peerlings/peerling-species.md#size-and-temperament).

### Spawn hexagon
The Plains hexagon in the middle of the world where everyone spawns; the spawn
hub sits at its centre. See [procedural-generation §
Layout](world/procedural-generation.md#layout).

### Spawn hub
The centre of the world, where every player starts: Creation Shrine, rest
point, New Peerlings gallery, Peerling of the Day pedestal and network
monument. See
[procedural-generation § Spawn hub](world/procedural-generation.md#spawn-hub).

### Species
A Peerling design: the immutable, content-addressed definition (name,
description, types, base stats, moves, image, 3D model) created once by its
creator through the creation pipeline. See
[peerling-species](peerlings/peerling-species.md).

### Species record
The document stored on IPFS that defines a species. Its CID is
the species' identity. See [peerling-species](peerlings/peerling-species.md).

### Species stats
Per-species counters (encounters, catches, owners, trades, providers, first
wild finder) that the server publishes as an hourly snapshot for creators and
species cards. See
[creator-feedback](gameplay/creator-feedback.md).

### Spectator
A nearby player watching a PvP battle live. See
[spectating](gameplay/spectating.md).

### Starter
The first Peerling a player owns: an instance of a species the player created
during onboarding, or of an existing species they chose instead. See
[onboarding](gameplay/onboarding.md).

### Stat stage
A temporary battle modifier on Attack, Defense or Speed, from −3 to +3. See
[battle § Stat stages](gameplay/battle.md#stat-stages).

### Team
The Peerlings a player brings into battles: up to 4. See
[catching](gameplay/catching.md#team-and-collection).

### Temperament
A species' short personality line (at most 60 characters), shown on its card and used to style its idle animation. See
[peerling-species § Size and temperament](peerlings/peerling-species.md#size-and-temperament).

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

### Verification checkpoint
An operator-signed statement that a player's save log verifies up to a given entry; verifiers check only what comes after it. See
[network-performance](tech/network-performance.md#5-verification-checkpoints).

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

### Wild level
A wild Peerling's level: the base level of its tile (rising with distance from the world centre) plus a random −2…+2. See
[encounters § Wild level](gameplay/encounters.md#wild-level).

### Wild Peerling
An unowned Peerling instance met during exploration, generated from a species
in the registry. See [encounters](gameplay/encounters.md).

### World feed
The live ticker of notable events (new Peerlings, shrine creations, shimmer
catches, first wild finds, all 12 badges, and the Peerling of the Day) spread
over a world-wide pubsub topic or added locally. See [world-feed](gameplay/world-feed.md).

### XP
Experience points a Peerling earns from wild battles; enough XP raises its
level. See [battle § Experience and levelling](gameplay/battle.md#experience-and-levelling).
