---
title: Open Questions
type: reference
status: draft
tags: [questions, backlog]
updated: 2026-10-03
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
  display names (and any chat) need moderation too.
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
- Raised: 2026-10-03

### Q-010
**Progression: levels, experience, evolution?**
- Affects: [peerling-species](peerlings/peerling-species.md), [battle](gameplay/battle.md), [core-loop](gameplay/core-loop.md), [moves](peerlings/moves.md)
- Context: Evolution would require generating additional forms (more GPU work,
  more design).
- Proposal: levels and XP in v1, no evolution in v1.
- Raised: 2026-10-03

### Q-011
**World size: finite or endless?**
- Affects: [procedural-generation](world/procedural-generation.md), [exploration](gameplay/exploration.md)
- Partly resolved 2026-10-03: one shared world for all players
  ([D-0008](decisions/D-0008-shared-multiplayer-world.md)). Still open: its size.
- Proposal: endless chunk-based world, with difficulty rising with distance from
  the start. Alternatively, a large finite world, so players crowd together more
  and meet each other more often.
- Raised: 2026-10-03

### Q-012
**Presentation: camera and visual style of the world.**
- Affects: [exploration](gameplay/exploration.md), [battle](gameplay/battle.md)
- Context: Peerlings are static 3D models and battles use simple 3D graphics.
  The world view is still open: third-person, isometric/top-down 3D, or a 2D
  world with 3D battles.
- Raised: 2026-10-03

### Q-014
**Player identity and saves: where does progress live, and how is it recovered?**
- Affects: [player-character](gameplay/player-character.md), [ipfs-helia](tech/ipfs-helia.md)
- Context: No accounts were mentioned. A keypair generated in the browser can
  identify the player, but it is lost if browser storage is cleared.
- Proposal: keypair + save in browser storage; optional export/import of a
  backup file; optional encrypted save backup on IPFS later.
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

### Q-023
**Stat budget: what is the base-stat total, and what are the per-stat bounds?**
- Affects: [peerling-species](peerlings/peerling-species.md#stats), [battle](gameplay/battle.md)
- Context: Equal totals are decided. The numbers depend on the damage formula.
- Proposal: decide together with the damage formula. For example, total 250
  across 4 stats, each stat between 35 and 100.
- Raised: 2026-10-03

### Q-024
**Move slots: confirm quick / strong / signature, and the details around them.**
- Affects: [moves](peerlings/moves.md), [battle](gameplay/battle.md), [creation-pipeline](peerlings/creation-pipeline.md)
- Context: The designer suggested "a quick attack, strong attack and a special
  attack". The draft turns that into three slots with per-slot templates.
- Sub-questions: (a) Is the three-slot structure right? (b) Add a fourth
  *support* slot (buff/debuff/heal) for more tactics? (c) Are moves unlimited,
  limited per battle (like PP), or paid from a resource (like Temtem's
  stamina)? (d) Do Peerlings ever learn or change moves (ties to Q-010)?
- Proposal: three slots plus a support slot; unlimited uses, with drawbacks
  built into the strong templates; no move changes in v1.
- Raised: 2026-10-03

### Q-025
**PvP fairness: how much cheating protection is needed?**
- Affects: [pvp-battles](gameplay/pvp-battles.md), [architecture](tech/architecture.md#trust-model)
- Context: Saves live in the browser and can be edited. Species stats can be
  verified (CID + attestation), but levels and ownership can't, without a
  server.
- Proposal: normalize all levels in PvP; don't verify ownership in v1; no
  farmable PvP rewards. Revisit if ranked play is wanted.
- Raised: 2026-10-03

### Q-026
**Trade integrity: is duplication by modified clients acceptable?**
- Affects: [trading](gameplay/trading.md), [peerling-species](peerlings/peerling-species.md#peerling-instance)
- Context: Without a central authority, a modified client can trade a Peerling
  away and keep a copy. Species aren't scarce (anyone can catch them), so the
  harm is limited.
- Options: (a) accept it for a casual game; (b) the server notarizes ownership
  (signs instances when caught or traded), which needs the server for every
  catch and trade; (c) an OrbitDB ownership log that peers check.
- Proposal: (a) for v1.
- Raised: 2026-10-03

### Q-027
**Multiplayer scale and social features.**
- Affects: [multiplayer](gameplay/multiplayer.md), [realtime-networking](tech/realtime-networking.md), [player-character](gameplay/player-character.md)
- Context: Region size, how many players are visible at once, chat or
  emotes (chat needs moderation), friends list.
- Proposal: regions of 64 × 64 m; show at most 30 nearby players; no free-text
  chat in v1, only a small set of emotes.
- Raised: 2026-10-03

### Q-028
**How are battles and trades started — only face to face, or also remotely?**
- Affects: [multiplayer](gameplay/multiplayer.md), [pvp-battles](gameplay/pvp-battles.md), [trading](gameplay/trading.md)
- Context: Face to face only makes the shared world matter. Remote play (via
  friends list or invite code) is more convenient.
- Proposal: face to face in v1 (within a short distance in the world).
- Raised: 2026-10-03

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
