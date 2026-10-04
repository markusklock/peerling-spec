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

### Q-001
**Can a player create more than one Peerling species?**
- Affects: [onboarding](gameplay/onboarding.md), [creation-pipeline](peerlings/creation-pipeline.md), [generation-server](tech/generation-server.md)
- Context: The brief says every new player creates a starter. Unclear whether
  more creations are possible later (e.g. as a reward), which matters for GPU
  cost, registry growth and progression design.
- Proposal: one species per player at onboarding in v1; additional creations as
  a later, earned reward.
- Raised: 2026-10-03

### Q-005
**Regeneration rules: how many image regenerations, and can the player edit their description in between?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [generation-server](tech/generation-server.md)
- Proposal: up to 5 image generations per creation; the player may tweak the
  description between attempts.
- Raised: 2026-10-03

### Q-006
**Does the player see and approve the 3D model, or only the image?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md)
- Context: Image-to-3D can fail in ways the 2D image doesn't show (missing back
  side, broken geometry).
- Proposal: show a rotatable preview; allow one 3D retry; if it still fails,
  return to image selection.
- Raised: 2026-10-03

### Q-016
**Asset budgets: maximum model size, texture resolution, image size.**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [peerling-species](peerlings/peerling-species.md), [ipfs-helia](tech/ipfs-helia.md)
- Context: Every encounter downloads a model peer-to-peer; size drives load
  time.
- Proposal: GLB ≤ 2 MB with mesh compression, textures ≤ 1024 px, a small
  thumbnail for lists.
- Raised: 2026-10-03

### Q-018
**Who names the Peerling, and must names be unique?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [peerling-species](peerlings/peerling-species.md)
- Proposal: the LLM suggests names, the player picks or types one;
  names need not be unique since the CID is the identity.
- Raised: 2026-10-03

### Q-019
**Player character creation: how much customization? Is the avatar AI-generated too?**
- Affects: [player-character](gameplay/player-character.md), [onboarding](gameplay/onboarding.md)
- Context: Avatars are now seen by other players in the shared world.
- Raised: 2026-10-03

### Q-021
**Do creators get feedback when others meet or catch their Peerling?**
- Affects: [orbitdb-registry](tech/orbitdb-registry.md), [ipfs-showcase](tech/ipfs-showcase.md)
- Context: "Your Peerling has been caught 42 times" is a strong social hook and
  could use an OrbitDB event log, but needs anti-spam thought.
- Raised: 2026-10-03

### Q-022
**What happens when the generation server is down?**
- Affects: [architecture](tech/architecture.md), [onboarding](gameplay/onboarding.md)
- Context: Creation needs the server. Play could continue from peers and cached
  data, which is a good demonstration of decentralization. The server is
  also the main relay between browsers, so multiplayer would degrade.
- Proposal: creation unavailable (queue/waitlist message); everything else keeps
  working from peers and local cache as far as connectivity allows.
- Raised: 2026-10-03

### Q-027
**Multiplayer scale: region size and how many players are visible at once.**
- Affects: [multiplayer](gameplay/multiplayer.md), [realtime-networking](tech/realtime-networking.md)
- Partly resolved 2026-10-04: no chat, only emotes (MPL-007).
- Proposal: regions of 64 × 64 m; show at most 30 nearby players.
- Raised: 2026-10-03

### Q-034
**Tech stack: approve the proposals in tech-stack?**
- Affects: [tech-stack](tech/tech-stack.md), [ipfs-helia](tech/ipfs-helia.md), [generation-server](tech/generation-server.md)
- Context: The principle (modern web first), WebTransport and IPv6 are
  decided (D-0011). The rest of the stack is proposed.
- Sub-questions: (a) browser target: current Chrome/Edge, Firefox, Safari,
  desktop and mobile? (b) WebGPU with WebGL2 fallback, or WebGPU only (which
  drops Firefox on Linux and Android)? (c) WebRTC-direct as a second
  browser → server transport? (d) Web Worker + OPFS + Ed25519 WebCrypto + PWA?
  (e) glTF with meshopt and KTX2, and AVIF images?
- Raised: 2026-10-04

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
4 km size is still [proposed]).

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
attestation. Resolved 2026-10-04 →
[D-0009](decisions/D-0009-player-data-on-orbitdb.md),
[pvp-battles](gameplay/pvp-battles.md#fairness).

### Q-026
**Trade integrity.** Trades complete only when the server's ownership ledger
records them. Resolved 2026-10-04 →
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
