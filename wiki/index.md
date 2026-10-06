# Wiki Index

Catalog of every page in the Peerlings specification. **LLMs: read this first**,
then open only the pages you need. Keep it updated on every change
(`AGENTS.md` §8).

Status legend: `stub` · `draft` · `proposed` · `accepted` · `deprecated`

## Start here

| Page | Status | Summary |
|------|--------|---------|
| [overview](overview.md) | accepted | Vision, the two equal goals, design pillars, v1 scope |
| [glossary](glossary.md) | accepted | Canonical definitions of all terms |
| [open-questions](open-questions.md) | draft | All unresolved questions (Q-NNN) |
| [log](log.md) | — | Append-only change log |

## Gameplay

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [core-loop](gameplay/core-loop.md) | accepted | — | Explore → encounter → battle → catch; player motivations |
| [onboarding](gameplay/onboarding.md) | accepted | ONB | New player creates a character and creates or chooses a starter Peerling |
| [player-character](gameplay/player-character.md) | accepted | PLR | Parts-based avatar, identity key (one key per player) |
| [exploration](gameplay/exploration.md) | accepted | EXP | Tile-by-tile grid movement, map, encounters in foliage, rest points, no fast travel |
| [encounters](gameplay/encounters.md) | accepted | ENC | Choosing wild Peerlings: weights, 5 ordered candidates, wild levels, prefetching |
| [battle](gameplay/battle.md) | accepted | BTL | Turns, action order, move mechanics, stat stages, damage model, XP, RNG, integer maths, rules versions, battle screen |
| [catching](gameplay/catching.md) | accepted | CAT | Catching as a battle action (no items), catch chance, team of 4, collection |
| [multiplayer](gameplay/multiplayer.md) | accepted | MPL | Shared world: seeing other players, face-to-face interaction, emotes (no chat) |
| [pvp-battles](gameplay/pvp-battles.md) | accepted | PVP | Peer-to-peer battles: Fair / Real-levels modes, verification, commit-reveal protocol |
| [trading](gameplay/trading.md) | accepted | TRD | Peer-to-peer trades of Peerling instances |
| [creation-shrine](gameplay/creation-shrine.md) | accepted | SHR | Giving up 3 Peerlings to create a new species |
| [creator-feedback](gameplay/creator-feedback.md) | accepted | CFB | Hourly species stats snapshots, live creator notifications, first wild finder |
| [spectating](gameplay/spectating.md) | accepted | SPT | Watching PvP battles live over pubsub |
| [sharing](gameplay/sharing.md) | accepted | LNK | Shareable Peerling cards and IPNS player profiles, Peerlings Viewer |
| [peerdex](gameplay/peerdex.md) | accepted | DEX | Index of seen and caught species, own creations |
| [ui](gameplay/ui.md) | accepted | UI | Title screen, pause menu, HUD, controls, settings |
| [guardians](gameplay/guardians.md) | accepted | GRD | One guardian per biome with a weekly team; 12 badges verified by replay |
| [peerling-of-the-day](gameplay/peerling-of-the-day.md) | accepted | POD | A species of the day (24 h) that appears more often everywhere, with a pedestal at the spawn |
| [world-feed](gameplay/world-feed.md) | accepted | FED | Live ticker of notable world events over pubsub |

## Peerlings

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [creation-pipeline](peerlings/creation-pipeline.md) | accepted | CRE | Wish → concept (+type) → image (no limit, cooldown) → 3D → stats & moves → final review and naming → player publishes, server pins |
| [peerling-species](peerlings/peerling-species.md) | accepted | SPC | Species record (DAG-CBOR on IPFS), stats (total 320, 40–130), instance data model, individual traits and shimmers |
| [types](peerlings/types.md) | accepted | TYP | The 12 types, primary/secondary type, effectiveness chart |
| [moves](peerlings/moves.md) | accepted | MOV | Three move slots (quick/strong/signature), unlimited use, templates, how Pokémon-like games do it |
| [moderation](peerlings/moderation.md) | deprecated | MOD | No content moderation (D-0010); kept for history |

## World

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [procedural-generation](world/procedural-generation.md) | accepted | WGN | Shared 4 × 4 km world: tiles, 12 biomes, terrain, spawn hub, landmarks, paths, day/night, weather, generator updates |
| [audio](world/audio.md) | accepted | AUD | Music, sound effects, Peerling cries synthesized from the CID |
| [visual-style](world/visual-style.md) | accepted | VIS | Top-down tilted camera, battle camera, colorful stylized look, environment art kit |

## Tech

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [architecture](tech/architecture.md) | accepted | ARC | Components, data flows, trust model |
| [ipfs-helia](tech/ipfs-helia.md) | accepted | NODE | Browser IPFS node: connectivity, publishing, CID import parameters, caching |
| [orbitdb-registry](tech/orbitdb-registry.md) | accepted | REG | OrbitDB database of all species, written by players with server signatures (plus the game's other OrbitDB databases) |
| [generation-server](tech/generation-server.md) | accepted | SRV | Self-hosted AI models, pinning, listing and origin signer, job queue, relay |
| [realtime-networking](tech/realtime-networking.md) | accepted | NET | libp2p pubsub presence, emotes, direct protocols for battles/trades |
| [player-data](tech/player-data.md) | accepted | SAVE | Save contents, per-player OrbitDB save log, account recovery (phrase, file, phone backup), who holds saves, peer verification by replay, encounter seeds (epoch records, drand), transfer log |
| [data-formats](tech/data-formats.md) | accepted | FMT | Exact formats: encoding, identifiers, signed envelope, OrbitDB databases, every record |
| [protocols](tech/protocols.md) | accepted | PRT | Exact pubsub and libp2p stream messages |
| [creation-api](tech/creation-api.md) | accepted | API | HTTP/3 API between client and operator server |
| [tech-stack](tech/tech-stack.md) | accepted | STK | Platform technologies: WebTransport, WebRTC, IPv6, WebGPU, OPFS, asset formats and budgets |
| [resilience](tech/resilience.md) | accepted | RES | What works without the operator server, and how |
| [network-performance](tech/network-performance.md) | accepted | PERF | Slow requests, data growth, retrieval ladder, timeouts, waiting states, snapshots and indexes |
| [ipfs-showcase](tech/ipfs-showcase.md) | accepted | SHOW | Making IPFS visible and meaningful to players |

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
| [D-0009](decisions/D-0009-player-data-on-orbitdb.md) | accepted (partly superseded by D-0013, D-0015) | Player saves on OrbitDB, with server-verified catches and trades |
| [D-0010](decisions/D-0010-no-content-moderation.md) | accepted | No content moderation |
| [D-0011](decisions/D-0011-modern-web-platform-first.md) | accepted | Modern web platform first |
| [D-0012](decisions/D-0012-starter-choice-and-extra-creations.md) | accepted | Starter choice and additional creations |
| [D-0013](decisions/D-0013-peer-verified-registry-catches-trades.md) | accepted | Peer-verified registry, catches and trades |
| [D-0014](decisions/D-0014-individual-variation.md) | accepted | Individual variation: stat traits and shimmer variants |
| [D-0015](decisions/D-0015-pvp-level-modes.md) | accepted | PvP level modes: Fair or Real levels |
| [D-0016](decisions/D-0016-showcase-features.md) | accepted | Spectating, shareable links, phone backup and a world feed |
| [D-0017](decisions/D-0017-world-features.md) | accepted | World features: spawn hub, landmarks, map, day/night, weather |
| [D-0018](decisions/D-0018-one-key-per-player.md) | accepted | One key per player for every identity |
| [D-0019](decisions/D-0019-peerdex-ui-audio.md) | accepted | Peerdex, menus, audio and rest points |
| [D-0020](decisions/D-0020-hexagon-spawn-biome-sectors.md) | accepted | Central Plains spawn hexagon with 12 biome sectors |
| [D-0021](decisions/D-0021-pvp-win-counter.md) | accepted | PvP win counter on the player profile |
| [D-0022](decisions/D-0022-v1-fun-features.md) | accepted | Following Peerling, guardians, Peerling of the Day, first finds |
| [D-0023](decisions/D-0023-network-performance.md) | accepted | Network performance failsafes, snapshots and indexes |
| [D-0024](decisions/D-0024-review-2-decisions.md) | accepted | Determinism, verification and edge-case rules from the second review |

## Registered requirement prefixes

API, ARC, AUD, BTL, CAT, CFB, CRE, DEX, ENC, EXP, FED, FMT, GRD, LNK, MOD, MOV, MPL, NET, NODE, ONB, PLR,
PERF, POD, PRT, PVP, REG, RES, SAVE, SHOW, SHR, SPC, SPT, SRV, STK, TRD, TYP, UI, VIS, WGN. Next free decision ID:
D-0025. Next free question ID: Q-057.

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
| [2026-10-05-phone-backup-approved](../raw/conversations/2026-10-05-phone-backup-approved.md) | 2026-10-05 | Phone backup and one-computer-at-a-time rule approved |
| [2026-10-05-world-details](../raw/conversations/2026-10-05-world-details.md) | 2026-10-05 | Spawn hub, landmarks, paths, map, hand-made art kit, day/night, weather, generator updates |
| [2026-10-05-world-details-approved](../raw/conversations/2026-10-05-world-details-approved.md) | 2026-10-05 | Smaller world details approved |
| [2026-10-05-formats-request](../raw/conversations/2026-10-05-formats-request.md) | 2026-10-05 | Designer asked for the exact data and message formats |
| [2026-10-06-formats-approved](../raw/conversations/2026-10-06-formats-approved.md) | 2026-10-06 | Formats approved; one key per player kept after weighing pros and cons |
| [2026-10-06-peerdex-ui-audio-restpoints](../raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md) | 2026-10-06 | Peerdex, menus/HUD/controls, audio with CID-synthesized cries, rest points; no fast travel |
| [2026-10-06-cry-details-approved](../raw/conversations/2026-10-06-cry-details-approved.md) | 2026-10-06 | Peerling cry details approved |
| [2026-10-06-review-decisions](../raw/conversations/2026-10-06-review-decisions.md) | 2026-10-06 | Consistency-review findings approved; PvP win stat suggested; hexagon world layout suggested |
| [2026-10-06-pvp-wins-hex-world](../raw/conversations/2026-10-06-pvp-wins-hex-world.md) | 2026-10-06 | PvP win counter; first encounter approved; hexagon + 12 biome sectors (option B) |
| [2026-10-06-proposals-approved](../raw/conversations/2026-10-06-proposals-approved.md) | 2026-10-06 | Battle maths, PvP win record, presentation formulas and hexagon geometry approved |
| [2026-10-06-v1-fun-features](../raw/conversations/2026-10-06-v1-fun-features.md) | 2026-10-06 | Following Peerling, guardians, Peerling of the Day (with spawn pedestal), "First found in the wild by …" |
| [2026-10-06-fun-features-approved](../raw/conversations/2026-10-06-fun-features-approved.md) | 2026-10-06 | Fun-feature rules approved; Peerling of the Day lasts 24 h; creators excluded from first-finder credit |
| [2026-10-06-network-performance](../raw/conversations/2026-10-06-network-performance.md) | 2026-10-06 | Network performance, scale and timeouts reviewed; failsafes proposed |
| [2026-10-06-network-performance-approved](../raw/conversations/2026-10-06-network-performance-approved.md) | 2026-10-06 | Network performance design approved |
| [2026-10-06-review-2-fixes](../raw/conversations/2026-10-06-review-2-fixes.md) | 2026-10-06 | Second full review; mechanical fixes applied, design questions collected |
| [2026-10-06-review-2-decisions](../raw/conversations/2026-10-06-review-2-decisions.md) | 2026-10-06 | 14 design decisions from the second review approved |
