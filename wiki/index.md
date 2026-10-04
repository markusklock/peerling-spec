# Wiki Index

Catalog of every page in the Peerlings specification. **LLMs: read this first**,
then open only the pages you need. Keep it updated on every change
(`AGENTS.md` §8).

Status legend: `stub` · `draft` · `proposed` · `accepted` · `deprecated`

## Start here

| Page | Status | Summary |
|------|--------|---------|
| [overview](overview.md) | draft | Vision, the two equal goals, design pillars, v1 scope |
| [glossary](glossary.md) | draft | Canonical definitions of all terms |
| [open-questions](open-questions.md) | draft | All unresolved questions (Q-NNN) |
| [log](log.md) | — | Append-only change log |

## Gameplay

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [core-loop](gameplay/core-loop.md) | draft | — | Explore → encounter → battle → catch; player motivations |
| [onboarding](gameplay/onboarding.md) | draft | ONB | New player creates character + starter Peerling |
| [player-character](gameplay/player-character.md) | stub | PLR | Avatar, identity keypair (save details: player-data) |
| [exploration](gameplay/exploration.md) | stub | EXP | Moving through the world |
| [encounters](gameplay/encounters.md) | draft | ENC | Choosing wild Peerlings deterministically from the registry; wild levels; prefetching |
| [battle](gameplay/battle.md) | stub | BTL | Battle rules: damage model, XP and levelling, deterministic RNG; procedural animation of static models |
| [catching](gameplay/catching.md) | stub | CAT | Catching wild Peerlings; collection |
| [multiplayer](gameplay/multiplayer.md) | draft | MPL | Shared world: seeing other players, face-to-face interaction, emotes (no chat) |
| [pvp-battles](gameplay/pvp-battles.md) | draft | PVP | Peer-to-peer battles: fairness, commit-reveal protocol |
| [trading](gameplay/trading.md) | draft | TRD | Peer-to-peer trades of Peerling instances |

## Peerlings

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [creation-pipeline](peerlings/creation-pipeline.md) | draft | CRE | Wish → concept (+type) → image → review → 3D → stats & moves → player publishes, server pins → starter |
| [peerling-species](peerlings/peerling-species.md) | draft | SPC | Species record (DAG-CBOR on IPFS), stats (total 320, 40–130), Peerling instance data model |
| [types](peerlings/types.md) | draft | TYP | The 12 types, primary/secondary type, effectiveness chart (draft) |
| [moves](peerlings/moves.md) | draft | MOV | Three move slots (quick/strong/signature), unlimited use, templates, how Pokémon-like games do it |
| [moderation](peerlings/moderation.md) | stub | MOD | Content checks and takedown via registry tombstones |

## World

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [procedural-generation](world/procedural-generation.md) | stub | WGN | One shared, large but finite, seeded, chunked world; regions; biomes |

## Tech

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [architecture](tech/architecture.md) | draft | ARC | Components, data flows, trust model |
| [ipfs-helia](tech/ipfs-helia.md) | draft | NODE | Browser IPFS node: connectivity, publishing, CID import parameters, caching |
| [orbitdb-registry](tech/orbitdb-registry.md) | draft | REG | OrbitDB database of all species (plus an overview of the game's other OrbitDB databases) |
| [generation-server](tech/generation-server.md) | draft | SRV | Self-hosted AI models, pinning, registry writer, job queue, relay |
| [realtime-networking](tech/realtime-networking.md) | draft | NET | libp2p pubsub presence, emotes, direct protocols for battles/trades |
| [player-data](tech/player-data.md) | draft | SAVE | Save contents, per-player OrbitDB save log, key recovery, catch verification, encounter seeds (epoch records, drand), ownership ledger |
| [ipfs-showcase](tech/ipfs-showcase.md) | draft | SHOW | Making IPFS visible and meaningful to players |

## Decisions

| ID | Status | Title |
|----|--------|-------|
| [D-0001](decisions/D-0001-spec-only-llm-wiki.md) | accepted | Spec-only repository maintained as an LLM Wiki |
| [D-0002](decisions/D-0002-all-peerlings-user-generated.md) | accepted | All Peerlings are user-generated |
| [D-0003](decisions/D-0003-browser-client-is-ipfs-node.md) | accepted | Every browser client is a Helia IPFS node |
| [D-0004](decisions/D-0004-single-operator-server.md) | accepted | One operator server for generation and pinning |
| [D-0005](decisions/D-0005-server-sole-registry-writer.md) | accepted | The generation server is the only registry writer |
| [D-0006](decisions/D-0006-species-vs-instance.md) | accepted | Separate immutable species from owned instances |
| [D-0007](decisions/D-0007-players-publish-assets.md) | accepted | The player's browser publishes their Peerling to IPFS |
| [D-0008](decisions/D-0008-shared-multiplayer-world.md) | accepted | One shared world with PvP battles and trading |
| [D-0009](decisions/D-0009-player-data-on-orbitdb.md) | accepted | Player saves on OrbitDB, with server-verified catches and trades |

## Registered requirement prefixes

ARC, BTL, CAT, CRE, ENC, EXP, MOD, MOV, MPL, NET, NODE, ONB, PLR, PVP, REG,
SAVE, SHOW, SPC, SRV, TRD, TYP, WGN. Next free decision ID: D-0010. Next free question
ID: Q-031.

## Sources

| Source | Date | Summary |
|--------|------|---------|
| [2026-10-03-initial-vision](../raw/conversations/2026-10-03-initial-vision.md) | 2026-10-03 | The designer's initial brief for the whole game |
| [2026-10-03-answers-round-1](../raw/conversations/2026-10-03-answers-round-1.md) | 2026-10-03 | Answers to Q-002/003/004/008/009/011/013/015/020: server-only registry, browser publishes, type at concept stage, 12 types, equal stats, seed species, shared multiplayer world, static models |
| [2026-10-04-answers-round-2](../raw/conversations/2026-10-04-answers-round-2.md) | 2026-10-04 | 3 move slots, face-to-face battles/trades, emotes only, large finite world; designer asked for save-storage options and stat suggestions |
| [2026-10-04-answers-round-3](../raw/conversations/2026-10-04-answers-round-3.md) | 2026-10-04 | Saves on OrbitDB + server-verified catches/trades accepted; stats and damage model approved; unlimited moves; 3 m, emotes, 4 × 4 km accepted |
| [2026-10-04-answers-round-4](../raw/conversations/2026-10-04-answers-round-4.md) | 2026-10-04 | D-0006 accepted; simple XP curve, no evolution, fixed moves; requests to draft the type chart and the encounter-seed design |
