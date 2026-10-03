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

### Q-002
**Who is allowed to write to the registry?**
- Affects: [orbitdb-registry](tech/orbitdb-registry.md), [creation-pipeline](peerlings/creation-pipeline.md)
- Context: If any browser can append to the OrbitDB registry, anyone can inject
  overpowered or offensive species that bypassed the pipeline.
- Proposal: only the generation server's OrbitDB identity can write (see
  [D-0005](decisions/D-0005-server-sole-registry-writer.md)); clients replicate
  and read.
- Raised: 2026-10-03

### Q-003
**Who adds the generated assets to IPFS — the server or the player's browser node?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [ipfs-helia](tech/ipfs-helia.md), [generation-server](tech/generation-server.md)
- Context: The brief says the server "autopins all assets pushed to IPFS by
  users", suggesting the client publishes. But the server generated the assets
  and already has them.
- Options: (a) server adds and pins, client then fetches them over IPFS
  (reliable); (b) server hands files to the client, the client adds them to its
  Helia node, the server pins by CID by fetching from the client (better IPFS
  showcase, more failure modes).
- Proposal: (a) for the authoritative copy, plus the client keeps and serves
  its own creation from its Helia node from then on.
- Raised: 2026-10-03

### Q-004
**Should the type be decided before the image is generated?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [types](peerlings/types.md)
- Context: The brief orders it: concept → image → 3D → type and moves. If the
  type is assigned after the image, a creature that *looks* like fire might get
  a water type. Deciding the type in the concept step lets the image reflect it.
- Proposal: concept step picks the type(s); the final step only re-validates.
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
- Affects: [moderation](peerlings/moderation.md), [orbitdb-registry](tech/orbitdb-registry.md)
- Context: Players can describe anything: NSFW, hateful content, real people,
  copyrighted characters (e.g. existing Pokémon). IPFS content can't be deleted
  from the network, but the registry can stop listing it.
- Proposal: moderate both the text wish and the generated image on the server;
  takedown = a tombstone entry in the registry, clients stop showing the species
  and the server unpins it.
- Raised: 2026-10-03

### Q-008
**The type list and the type effectiveness chart.**
- Affects: [types](peerlings/types.md), [battle](gameplay/battle.md), [procedural-generation](world/procedural-generation.md)
- Context: A predefined list is decided; its contents are not. Classic elements
  are easy to understand; IPFS-themed types would be more original.
- Raised: 2026-10-03

### Q-009
**Stat model: does every species get the same base-stat total?**
- Affects: [peerling-species](peerlings/peerling-species.md), [battle](gameplay/battle.md)
- Proposal: yes — a fixed budget distributed by the LLM within per-stat
  min/max bounds, so no description produces an objectively stronger creature.
- Raised: 2026-10-03

### Q-010
**Progression: levels, experience, evolution?**
- Affects: [peerling-species](peerlings/peerling-species.md), [battle](gameplay/battle.md), [core-loop](gameplay/core-loop.md)
- Context: Evolution would require generating additional forms (more GPU work,
  more design).
- Proposal: levels and XP in v1, no evolution in v1.
- Raised: 2026-10-03

### Q-011
**World: one shared world seed for everyone, or a world per player? Finite or endless?**
- Affects: [procedural-generation](world/procedural-generation.md), [exploration](gameplay/exploration.md)
- Proposal: one shared global seed (everyone explores the same world, enables
  future multiplayer and shared landmarks); endless chunk-based world with
  difficulty rising with distance from the start.
- Raised: 2026-10-03

### Q-012
**Presentation: camera and visual style of the world.**
- Affects: [exploration](gameplay/exploration.md), [battle](gameplay/battle.md)
- Context: Peerlings are 3D models, so the world is presumably 3D. Options:
  third-person, isometric/top-down 3D, or 2D world with 3D battles.
- Raised: 2026-10-03

### Q-013
**Multiplayer scope: seeing other players, trading, PvP — when?**
- Affects: [overview](overview.md), [architecture](tech/architecture.md)
- Context: Peer-to-peer play (libp2p pubsub/WebRTC) would be a strong IPFS
  showcase but adds a lot of scope.
- Proposal: not in v1; design v1 data so it doesn't block it later.
- Raised: 2026-10-03

### Q-014
**Player identity and saves: where does progress live, and how is it recovered?**
- Affects: [player-character](gameplay/player-character.md), [ipfs-helia](tech/ipfs-helia.md)
- Context: No accounts were mentioned. A keypair generated in the browser can
  identify the player, but it is lost if browser storage is cleared.
- Proposal: keypair + save in browser storage; optional export/import of a
  backup file; optional encrypted save backup on IPFS later.
- Raised: 2026-10-03

### Q-015
**How are static 3D models animated?**
- Affects: [creation-pipeline](peerlings/creation-pipeline.md), [battle](gameplay/battle.md)
- Context: Image-to-3D generators (e.g. TRELLIS.2) output a static, unrigged
  mesh. Battles need some motion.
- Proposal: procedural animation only (idle bob/breathe, squash-and-stretch,
  lunge, shake on hit, faint tip-over); auto-rigging as a later option.
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
- Raised: 2026-10-03

### Q-020
**Cold start: what do the first players encounter when the registry is nearly empty?**
- Affects: [encounters](gameplay/encounters.md), [overview](overview.md)
- Context: "No Peerlings exist from start" — but the very first player would
  meet only their own species. Options: the operator creates a seed batch through
  the same pipeline before launch; allow repeats of few species; closed beta.
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
  data — a good demonstration of decentralization.
- Proposal: creation unavailable (queue/waitlist message); everything else keeps
  working from peers and local cache.
- Raised: 2026-10-03

## Resolved

_None yet._
