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

### Q-007
**Content moderation policy and takedowns.**
- Affects: [moderation](peerlings/moderation.md), [orbitdb-registry](tech/orbitdb-registry.md), [player-character](gameplay/player-character.md)
- Context: Players can describe anything: NSFW, hateful content, real people,
  copyrighted characters (e.g. existing Pokémon). IPFS content can't be deleted
  from the network, but the registry can stop listing it. With a shared world,
  display names need moderation too (there is no chat: MPL-007).
- Proposal: moderate both the text wish and the generated image on the server;
  takedown = a tombstone entry in the registry, clients stop showing the species
  and the server unpins it.
- Raised: 2026-10-03

### Q-008
**The type effectiveness chart.**
- Affects: [types](peerlings/types.md), [battle](gameplay/battle.md)
- Partly resolved 2026-10-03: the type list is the 12 classic elements
  ([types](peerlings/types.md#the-type-list)). Still open: which type is strong
  or weak against which.
- Draft chart written 2026-10-04 at the designer's request:
  [types § Effectiveness chart](peerlings/types.md#effectiveness-chart).
  Awaiting approval.
- Raised: 2026-10-03

### Q-012
**Presentation: camera and visual style of the world.**
- Affects: [exploration](gameplay/exploration.md), [battle](gameplay/battle.md)
- Context: Peerlings are static 3D models and battles use simple 3D graphics.
  The world view is still open: third-person, isometric/top-down 3D, or a 2D
  world with 3D battles.
- Raised: 2026-10-03

### Q-016
**Asset budgets: maximum model size, texture resolution, image size.**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [peerling-species](peerlings/peerling-species.md), [ipfs-helia](tech/ipfs-helia.md)
- Context: Every encounter downloads a model peer-to-peer; size drives load
  time.
- Proposal: GLB ≤ 2 MB with mesh compression, textures ≤ 1024 px, a small
  thumbnail for lists.
- Raised: 2026-10-03

### Q-017
**How are wild Peerlings chosen from the registry?**
- Affects: [encounters](gameplay/encounters.md), [orbitdb-registry](tech/orbitdb-registry.md)
- Context: Uniform random, weighted by biome/type, favor new or rarely-seen
  species, rarity tiers? Also: with thousands of species, the client must not
  download every model up front.
- Raised: 2026-10-03

### Q-018
**Who names the Peerling, and must names be unique?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [peerling-species](peerlings/peerling-species.md)
- Proposal: the LLM suggests names, the player picks or types one (moderated);
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

### Q-029
**Encounter seeds: approve the epoch-record design?**
- Affects: [player-data](tech/player-data.md#encounter-seeds), [encounters](gameplay/encounters.md), [generation-server](tech/generation-server.md), [orbitdb-registry](tech/orbitdb-registry.md)
- Context: Catch verification needs encounter seeds that players can't choose.
- Draft written 2026-10-04 at the designer's request:
  [player-data § Encounter seeds](tech/player-data.md#encounter-seeds). Every
  5 minutes the server publishes a signed epoch record with drand randomness and
  the registry height; seeds use a gap-free encounter number; the server
  re-checks everything when verifying a catch. Awaiting approval, including
  whether to use drand or the server's own random value.
- Raised: 2026-10-04
### Q-030
**Approve the XP curve and wild-level numbers?**
- Affects: [battle § Experience and levelling](gameplay/battle.md#experience-and-levelling), [encounters § Wild level](gameplay/encounters.md#wild-level)
- Context: A simple XP curve was accepted (Q-010); these are the concrete
  numbers.
- Draft 2026-10-04: XP to next level 5 × L²; 10 × wild level XP per defeat or
  catch; starter at level 5; wild level rises from 2 at the spawn to 50 at 2 km
  out (±2). About 600 wild battles from level 5 to 50.
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
