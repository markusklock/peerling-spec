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
| [onboarding](gameplay/onboarding.md) | draft | ONB | New player creates a character and creates or chooses a starter Peerling |
| [player-character](gameplay/player-character.md) | stub | PLR | Avatar options (recommended: parts-based customizer), identity keypair |
| [exploration](gameplay/exploration.md) | draft | EXP | Tile-by-tile grid movement, encounters in foliage, rest points and healing |
| [encounters](gameplay/encounters.md) | draft | ENC | Choosing wild Peerlings: weights, 5 ordered candidates, wild levels, prefetching |
| [battle](gameplay/battle.md) | draft | BTL | Turns, action order, move mechanics, stat stages, damage model, XP, RNG, battle screen |
| [catching](gameplay/catching.md) | draft | CAT | Catching as a battle action (no items), catch chance, team of 4, collection |
| [multiplayer](gameplay/multiplayer.md) | draft | MPL | Shared world: seeing other players, face-to-face interaction, emotes (no chat) |
| [pvp-battles](gameplay/pvp-battles.md) | draft | PVP | Peer-to-peer battles: Fair / Real-levels modes, verification, commit-reveal protocol |
| [trading](gameplay/trading.md) | draft | TRD | Peer-to-peer trades of Peerling instances |
| [creation-shrine](gameplay/creation-shrine.md) | draft | SHR | Giving up 3 Peerlings to create a new species |
| [creator-feedback](gameplay/creator-feedback.md) | draft | CFB | Species stats in OrbitDB, live creator notifications |
| [spectating](gameplay/spectating.md) | draft | SPT | Watching PvP battles live over pubsub |
| [sharing](gameplay/sharing.md) | draft | LNK | Shareable Peerling cards and IPNS player profiles, Peerlings Viewer |
| [world-feed](gameplay/world-feed.md) | draft | FED | Live ticker of notable world events over pubsub |

## Peerlings

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [creation-pipeline](peerlings/creation-pipeline.md) | draft | CRE | Wish → concept (+type) → image (no limit, cooldown) → 3D → stats & moves → final review and naming → player publishes, server pins |
| [peerling-species](peerlings/peerling-species.md) | draft | SPC | Species record (DAG-CBOR on IPFS), stats (total 320, 40–130), instance data model, individual traits and shimmers |
| [types](peerlings/types.md) | draft | TYP | The 12 types, primary/secondary type, effectiveness chart |
| [moves](peerlings/moves.md) | draft | MOV | Three move slots (quick/strong/signature), unlimited use, templates, how Pokémon-like games do it |
| [moderation](peerlings/moderation.md) | deprecated | MOD | No content moderation (D-0010); kept for history |

## World

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [procedural-generation](world/procedural-generation.md) | draft | WGN | One shared 4 × 4 km seeded world on a tile grid; 12 biomes with their foliage and layout |
| [visual-style](world/visual-style.md) | draft | VIS | Top-down tilted camera, battle camera, colorful stylized look |

## Tech

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [architecture](tech/architecture.md) | draft | ARC | Components, data flows, trust model |
| [ipfs-helia](tech/ipfs-helia.md) | draft | NODE | Browser IPFS node: connectivity, publishing, CID import parameters, caching |
| [orbitdb-registry](tech/orbitdb-registry.md) | draft | REG | OrbitDB database of all species, written by players with server signatures (plus the game's other OrbitDB databases) |
| [generation-server](tech/generation-server.md) | draft | SRV | Self-hosted AI models, pinning, registry writer, job queue, relay |
| [realtime-networking](tech/realtime-networking.md) | draft | NET | libp2p pubsub presence, emotes, direct protocols for battles/trades |
| [player-data](tech/player-data.md) | draft | SAVE | Save contents, per-player OrbitDB save log, account recovery (phrase, file, phone backup), who holds saves, peer verification by replay, encounter seeds (epoch records, drand), transfer log |
| [tech-stack](tech/tech-stack.md) | draft | STK | Platform technologies: WebTransport, WebRTC, IPv6, WebGPU, OPFS, asset formats and budgets |
| [resilience](tech/resilience.md) | draft | RES | What works without the operator server, and how |
| [ipfs-showcase](tech/ipfs-showcase.md) | draft | SHOW | Making IPFS visible and meaningful to players |

## Decisions

| ID | Status | Title |
|----|--------|-------|
| [D-0001](decisions/D-0001-spec-only-llm-wiki.md) | accepted | Spec-only repository maintained as an LLM Wiki |
| [D-0002](decisions/D-0002-all-peerlings-user-generated.md) | accepted | All Peerlings are user-generated |
| [D-0003](decisions/D-0003-browser-client-is-ipfs-node.md) | accepted | Every browser client is a Helia IPFS node |
| [D-0004](decisions/D-0004-single-operator-server.md) | accepted | One operator server for generation and pinning |
| [D-0005](decisions/D-0005-server-sole-registry-writer.md) | superseded by D-0013 | The generation server is the only registry writer |
| [D-0006](decisions/D-0006-species-vs-instance.md) | accepted | Separate immutable species from owned instances |
| [D-0007](decisions/D-0007-players-publish-assets.md) | accepted | The player's browser publishes their Peerling to IPFS |
| [D-0008](decisions/D-0008-shared-multiplayer-world.md) | accepted | One shared world with PvP battles and trading |
| [D-0009](decisions/D-0009-player-data-on-orbitdb.md) | accepted (partly superseded by D-0013) | Player saves on OrbitDB, with server-verified catches and trades |
| [D-0010](decisions/D-0010-no-content-moderation.md) | accepted | No content moderation |
| [D-0011](decisions/D-0011-modern-web-platform-first.md) | accepted | Modern web platform first (WebTransport, IPv6) |
| [D-0012](decisions/D-0012-starter-choice-and-extra-creations.md) | accepted | Starter choice and additional creations |
| [D-0013](decisions/D-0013-peer-verified-registry-catches-trades.md) | accepted | Peer-verified registry, catches and trades |
| [D-0014](decisions/D-0014-individual-variation.md) | accepted | Individual variation: stat traits and shimmer variants |
| [D-0015](decisions/D-0015-pvp-level-modes.md) | accepted | PvP level modes: Fair or Real levels |
| [D-0016](decisions/D-0016-showcase-features.md) | accepted | Spectating, shareable links, device linking and a world feed |

## Registered requirement prefixes

ARC, BTL, CAT, CFB, CRE, ENC, EXP, FED, LNK, MOD, MOV, MPL, NET, NODE, ONB, PLR,
PVP, REG, RES, SAVE, SHOW, SHR, SPC, SPT, SRV, STK, TRD, TYP, VIS, WGN. Next free decision ID:
D-0017. Next free question ID: Q-046.

## Sources

| Source | Date | Summary |
|--------|------|---------|
| [2026-10-03-initial-vision](../raw/conversations/2026-10-03-initial-vision.md) | 2026-10-03 | The designer's initial brief for the whole game |
| [2026-10-03-answers-round-1](../raw/conversations/2026-10-03-answers-round-1.md) | 2026-10-03 | Answers to Q-002/003/004/008/009/011/013/015/020: server-only registry, browser publishes, type at concept stage, 12 types, equal stats, seed species, shared multiplayer world, static models |
| [2026-10-04-answers-round-2](../raw/conversations/2026-10-04-answers-round-2.md) | 2026-10-04 | 3 move slots, face-to-face battles/trades, emotes only, large finite world; designer asked for save-storage options and stat suggestions |
| [2026-10-04-answers-round-3](../raw/conversations/2026-10-04-answers-round-3.md) | 2026-10-04 | Saves on OrbitDB + server-verified catches/trades accepted; stats and damage model approved; unlimited moves; 3 m, emotes, 4 × 4 km accepted |
| [2026-10-04-answers-round-4](../raw/conversations/2026-10-04-answers-round-4.md) | 2026-10-04 | D-0006 accepted; simple XP curve, no evolution, fixed moves; requests to draft the type chart and the encounter-seed design |
| [2026-10-04-answers-round-5](../raw/conversations/2026-10-04-answers-round-5.md) | 2026-10-04 | Chart, XP and encounter seeds approved; no battle items; 12 biomes; top-down colorful 3D; no content moderation |
| [2026-10-04-tech-stack-1](../raw/conversations/2026-10-04-tech-stack-1.md) | 2026-10-04 | Catching, healing, biomes, weights, delisting approved; tech-stack principle, WebTransport instead of WebSockets, IPv6 |
| [2026-10-04-tech-stack-2](../raw/conversations/2026-10-04-tech-stack-2.md) | 2026-10-04 | Desktop only; WebGPU + WebGL2 fallback; WebRTC-direct fallback; rest of the stack approved |
| [2026-10-04-answers-round-7](../raw/conversations/2026-10-04-answers-round-7.md) | 2026-10-04 | Extra creations and starter choice; unlimited images with cooldown; restart from images; players name Peerlings; creator feedback wanted; server-offline resilience with 5 encounter candidates |
| [2026-10-04-answers-round-8](../raw/conversations/2026-10-04-answers-round-8.md) | 2026-10-04 | Shrine, character creation, multiplayer scale, encounter candidates approved; brainstorm on decentralizing registry writes and trades |
| [2026-10-04-decentralize-level-3](../raw/conversations/2026-10-04-decentralize-level-3.md) | 2026-10-04 | Level 3 decentralization accepted (D-0013); question about individual variation of wild Peerlings |
| [2026-10-05-individual-variation](../raw/conversations/2026-10-05-individual-variation.md) | 2026-10-05 | Individual variation: ±10% stat traits and rare color variants (options b and d) |
| [2026-10-05-asset-budgets-request](../raw/conversations/2026-10-05-asset-budgets-request.md) | 2026-10-05 | Designer asked for asset budget suggestions (Q-016) and for Q-038 to be explained |
| [2026-10-05-approvals-q016-q038](../raw/conversations/2026-10-05-approvals-q016-q038.md) | 2026-10-05 | Asset budgets and individual-variation details approved |
| [2026-10-05-grid-foliage-battles](../raw/conversations/2026-10-05-grid-foliage-battles.md) | 2026-10-05 | Game Boy-style grid movement; encounters in biome foliage; battle design requested |
| [2026-10-05-pvp-level-modes](../raw/conversations/2026-10-05-pvp-level-modes.md) | 2026-10-05 | PvP level modes (Fair / Real levels); battle rules and grid numbers approved |
| [2026-10-05-proposal-review-1](../raw/conversations/2026-10-05-proposal-review-1.md) | 2026-10-05 | Proposal review: technical proposals and design items 1–4 approved; unique names requested |
| [2026-10-05-proposal-review-2](../raw/conversations/2026-10-05-proposal-review-2.md) | 2026-10-05 | Unique names and design items 6–16, 18–20 approved; account recovery via password asked |
| [2026-10-05-save-recovery](../raw/conversations/2026-10-05-save-recovery.md) | 2026-10-05 | Recovery by phrase and file only; save contents approved; who actually holds save copies |
| [2026-10-05-peer-save-backups](../raw/conversations/2026-10-05-peer-save-backups.md) | 2026-10-05 | No community mirrors; peer save backups via profile inspection; storage persistence |
| [2026-10-05-peer-save-backups-approved](../raw/conversations/2026-10-05-peer-save-backups-approved.md) | 2026-10-05 | Peer save backups and save snapshot in backup file approved |
| [2026-10-05-showcase-features](../raw/conversations/2026-10-05-showcase-features.md) | 2026-10-05 | Adopted spectating, shareable links, device linking and a world feed |
| [2026-10-05-phone-backup](../raw/conversations/2026-10-05-phone-backup.md) | 2026-10-05 | Spectating, sharing and world feed details approved; device linking replaced by phone backup via QR |
