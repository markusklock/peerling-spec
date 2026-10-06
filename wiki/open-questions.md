---
title: Open Questions
type: reference
status: draft
tags: [questions, backlog]
updated: 2026-10-04
---

# Open Questions

> Every unresolved design question, with a stable ID. Pages link here instead of
> keeping their own question lists. Answered questions move to *Resolved* with a
> link to the answer; they are never deleted.

Each entry: the question, the pages it affects, context, and an LLM proposal
(which is **not** a decision until the designer accepts it).

## Open

## Resolved

### Q-002
**Who is allowed to write to the registry?** Only the operator's server.
Resolved 2026-10-03 → [D-0005](decisions/D-0005-server-sole-registry-writer.md).

### Q-003
**Who adds the generated assets to IPFS?** The player's browser; the server
pins. Resolved 2026-10-03 → [D-0007](decisions/D-0007-players-publish-assets.md),
[creation-pipeline § Stage 7](peerlings/creation-pipeline.md#stage-7--publish).

### Q-004
**Should the type be decided before the image is generated?** Yes. The
concept LLM determines the type from the player's description at the concept
stage. Resolved 2026-10-03 →
[creation-pipeline § Stage 2](peerlings/creation-pipeline.md#stage-2--concept) (CRE-016).

### Q-009
**Does every species get the same base-stat total?** Yes. The LLM spreads
stats to fit the concept. Resolved 2026-10-03 →
[peerling-species § Stats](peerlings/peerling-species.md#stats). The numbers
are still open in [Q-023](#q-023).

### Q-013
**Multiplayer scope.** All players share one world and can battle and trade,
in the first version. Resolved 2026-10-03 →
[D-0008](decisions/D-0008-shared-multiplayer-world.md).

### Q-015
**How are static 3D models animated?** Models stay static; battles use simple
3D graphics with procedural animation. Resolved 2026-10-03 →
[battle § Presentation](gameplay/battle.md#presentation) (CRE-017, BTL-003).

### Q-020
**Cold start.** The operator creates a handful of seed species at launch
through the normal pipeline. Resolved 2026-10-03 →
[D-0002](decisions/D-0002-all-peerlings-user-generated.md),
[encounters § Cold start](gameplay/encounters.md#cold-start).

### Q-011
**World size.** Large but finite, with one shared world for all players.
Resolved 2026-10-04 →
[procedural-generation](world/procedural-generation.md) (WGN-005; the 4 km ×
4 km size was approved on 2026-10-04).

### Q-028
**How are battles and trades started?** Only when the two players are next to
each other in the world. Resolved 2026-10-04 →
[multiplayer](gameplay/multiplayer.md) (MPL-006).

### Q-014
**Player identity and saves.** A per-player OrbitDB save log replicated by the
server; the key can be restored with a recovery phrase. Resolved 2026-10-04 →
[D-0009](decisions/D-0009-player-data-on-orbitdb.md),
[player-data](tech/player-data.md).

### Q-023
**Stat budget.** Total 320, each stat 40–130 in steps of 5, with the approved
damage model. Resolved 2026-10-04 →
[peerling-species § Stat numbers](peerlings/peerling-species.md#stat-numbers),
[battle § Damage model](gameplay/battle.md#damage-model).

### Q-024
**Move slots and move use.** Three slots (quick, strong, signature); moves
can be used without limit. Resolved 2026-10-04 →
[moves](peerlings/moves.md) (MOV-006, MOV-009). Whether moves change when
levelling moved to [Q-010](#q-010).

### Q-025
**PvP fairness.** Level 50 for everyone, and only Peerlings with a server catch
attestation. (Since D-0013: only verified Peerlings, checked by the opponent
itself.) Resolved 2026-10-04 →
[D-0009](decisions/D-0009-player-data-on-orbitdb.md),
[pvp-battles](gameplay/pvp-battles.md#fairness).

### Q-026
**Trade integrity.** Trades complete only when the server's ownership ledger
records them. (Superseded 2026-10-04 by
[D-0013](decisions/D-0013-peer-verified-registry-catches-trades.md): signed
transfer chains; double trades detected and flagged.) Resolved 2026-10-04 →
[D-0009](decisions/D-0009-player-data-on-orbitdb.md),
[trading](gameplay/trading.md#integrity).

### Q-010
**Progression.** A simple XP curve; levels 1–50; no evolution; Peerlings never
learn or change moves. Resolved 2026-10-04 →
[battle § Experience and levelling](gameplay/battle.md#experience-and-levelling),
[SPC-011](peerlings/peerling-species.md#requirements),
[MOV-010](peerlings/moves.md#requirements). The concrete numbers are in
[Q-030](#q-030).

### Q-007
**Content moderation.** None. The game is a free hobby project; if
inappropriate content becomes a real problem, the operator will shut the game
down. Resolved 2026-10-04 → [D-0010](decisions/D-0010-no-content-moderation.md).

### Q-008
**Type effectiveness chart.** The draft chart was approved. Resolved 2026-10-04 →
[types § Effectiveness chart](peerlings/types.md#effectiveness-chart).

### Q-012
**Camera and visual style.** Top-down camera over a 3D world, colorful style.
Resolved 2026-10-04 → [visual-style](world/visual-style.md).

### Q-029
**Encounter seeds.** The epoch-record design with drand randomness was approved.
Resolved 2026-10-04 →
[player-data § Encounter seeds](tech/player-data.md#encounter-seeds).

### Q-030
**XP curve and wild levels.** The draft numbers were approved. Resolved
2026-10-04 →
[battle § Experience and levelling](gameplay/battle.md#experience-and-levelling),
[encounters § Wild level](gameplay/encounters.md#wild-level).

### Q-017
**Encounter selection weights.** Biome-type match × 6, never-seen species × 2.
Resolved 2026-10-04 → [encounters § Selection](gameplay/encounters.md#selection).

### Q-031
**Catch chance and team size.** Catch as a battle action, chance
0.6 × (3 × maxHP − 2 × currentHP) ÷ (3 × maxHP) × level factor, team of 4,
unlimited collection. Resolved 2026-10-04 → [catching](gameplay/catching.md).

### Q-032
**Healing without items.** Rest points in every biome area; a fainted team
returns to the last rest point, fully healed. Resolved 2026-10-04 →
[exploration § Healing and rest points](gameplay/exploration.md#healing-and-rest-points).

### Q-033
**Biome names, looks and layout.** Approved as proposed. Resolved 2026-10-04 →
[procedural-generation § Biomes](world/procedural-generation.md#biomes).

### Q-034
**Tech stack proposals.** Desktop browsers only; WebGPU with WebGL2 fallback;
WebRTC-direct as fallback transport; Web Worker, OPFS, Ed25519 WebCrypto, PWA,
glTF meshopt + KTX2, AVIF and TypeScript approved. Resolved 2026-10-04 →
[tech-stack](tech/tech-stack.md).

### Q-001
**More than one species per player?** Yes: additional creations at a place on
the map where players give something up; new players may also choose an
existing species as their starter. Resolved 2026-10-04 →
[D-0012](decisions/D-0012-starter-choice-and-extra-creations.md). Balancing:
[Q-035](#q-035).

### Q-005
**Regeneration rules.** No limit on image generations; a cooldown prevents
spam. Resolved 2026-10-04 →
[creation-pipeline § Stage 4](peerlings/creation-pipeline.md#stage-4--review) (CRE-021).

### Q-006
**3D model approval.** No separate approval or retry; the player can restart
from image generation in the final review. Resolved 2026-10-04 →
[creation-pipeline § Final review](peerlings/creation-pipeline.md#final-review) (CRE-022).

### Q-018
**Naming.** The player names the Peerlings they create. Resolved 2026-10-04 →
[creation-pipeline § Final review](peerlings/creation-pipeline.md#final-review) (CRE-023).

### Q-021
**Creator feedback.** Yes: server-verified species stats in OrbitDB, live pubsub
notifications, and a "since you were last here" summary. Resolved 2026-10-04 →
[creator-feedback](gameplay/creator-feedback.md) (design details proposed).

### Q-019
**Player character creation.** Parts-based customizer with game-made, rigged
bodies (option A); an AI-generated 2D portrait may come later. Resolved
2026-10-04 → [player-character](gameplay/player-character.md#decision).

### Q-027
**Multiplayer scale.** 64 m regions (3 × 3 subscribed), 30 nearest players
drawn, 4 updates/s moving, 5 s heartbeat, 15 s timeout. Resolved 2026-10-04 →
[realtime-networking § Presence](tech/realtime-networking.md#presence-proposed),
[multiplayer](gameplay/multiplayer.md#scale-and-visibility-proposed).

### Q-035
**Creation Shrine cost and limits.** Approved as proposed. Resolved 2026-10-04 →
[creation-shrine](gameplay/creation-shrine.md#rules).

### Q-036
**Encounter candidate order.** Ordered list; earliest already-fetched
candidate, else first to arrive. Resolved 2026-10-04 →
[encounters § Candidates](gameplay/encounters.md#candidates).

### Q-022
**Playing without the server.** As much as possible works peer-to-peer: public
bootstrap, relays and routing; drand-based epoch records; the game app on IPFS;
optional mirrors. Since D-0013, catch verification and trades also work without
the server; only creation needs it. Resolved 2026-10-04 →
[resilience](tech/resilience.md),
[D-0013](decisions/D-0013-peer-verified-registry-catches-trades.md).

### Q-037
**Individual variation.** Stat traits of up to ±10% per stat, and rare
cosmetic shimmer variants. Resolved 2026-10-05 →
[D-0014](decisions/D-0014-individual-variation.md). Details: [Q-038](#q-038).

### Q-016
**Asset budgets.** Model ≤ 1 MB (≤ 20,000 triangles, one 1024 px texture),
card image ≤ 150 KB, thumbnail ≤ 15 KB, whole species ≤ 1.2 MB, 1 GB client
cache. Resolved 2026-10-05 → [tech-stack § Asset budgets](tech/tech-stack.md#asset-budgets).

### Q-038
**Individual variation details.** Shimmer 1 in 500; traits visible with an
overall rating; traits apply in PvP; species-specific shimmer look. Resolved
2026-10-05 → [peerling-species § Individual variation](peerlings/peerling-species.md#individual-variation).

### Q-039
**Battle rules.** Approved as proposed; PvP got a choice of level modes
(D-0015). Resolved 2026-10-05 → [battle § Rules](gameplay/battle.md#rules).

### Q-040
**Grid and encounter numbers.** 2 m tiles, 4-directional movement at 3
tiles/s, 1 in 10 encounter chance per foliage step with 3 grace steps, 20–30%
foliage, 32 × 32-tile chunks. Resolved 2026-10-05 →
[exploration § Grid movement](gameplay/exploration.md#grid-movement).

### Q-041
**Unique species names.** Unique in normalized form, enforced by the server
with a live check and reservation; names stay taken after delisting. Resolved
2026-10-05 → [creation-pipeline § Final review](peerlings/creation-pipeline.md#final-review).

### Q-042
**Account recovery and save contents.** Recovery phrase and backup file only;
no password recovery. The save contents list is approved. Resolved 2026-10-05 →
[player-data § Account recovery](tech/player-data.md#account-recovery).

### Q-043
**Keeping saves available.** Inspecting, trading with or battling a player
keeps a persistent backup of their save, served back on request via pubsub; the
backup file holds the latest save snapshot; the client requests persistent
storage. No reliance on community mirrors. Resolved 2026-10-05 →
[player-data § Keeping saves available](tech/player-data.md#keeping-saves-available).

### Q-044
**Details of spectating, sharing links and the world feed.** Approved as
proposed; device linking was replaced by a phone backup ([Q-045](#q-045)).
Resolved 2026-10-05 → [D-0016](decisions/D-0016-showcase-features.md).

### Q-045
**Phone backup and one computer at a time.** Approved as proposed. Resolved
2026-10-05 → [player-data § Phone backup](tech/player-data.md#phone-backup).

### Q-046
**World details.** Gallery, monument, hub, landmark names, paths, day length,
weather, map: all approved as proposed. Resolved 2026-10-05 →
[procedural-generation](world/procedural-generation.md),
[exploration § Map](gameplay/exploration.md#map).

### Q-047
**Exact data formats, protocols and creation API.** Approved, including one key
per player (D-0018), the phone backup flow and the text limits. Resolved
2026-10-06 → [data-formats](tech/data-formats.md), [protocols](tech/protocols.md),
[creation-api](tech/creation-api.md).

### Q-048
**Peerling cry details.** Approved as proposed. Resolved 2026-10-06 →
[audio § Peerling cries](world/audio.md#peerling-cries).
