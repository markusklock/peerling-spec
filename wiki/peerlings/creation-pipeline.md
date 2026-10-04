---
title: Peerling Creation Pipeline
type: system
status: draft
req_prefix: CRE
tags: [peerlings, generation, ai, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
related:
  - wiki/peerlings/peerling-species.md
  - wiki/peerlings/types.md
  - wiki/peerlings/moves.md
  - wiki/decisions/D-0010-no-content-moderation.md
  - wiki/tech/generation-server.md
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-helia.md
  - wiki/gameplay/onboarding.md
  - wiki/gameplay/creation-shrine.md
  - wiki/decisions/D-0007-players-publish-assets.md
updated: 2026-10-04
---

# Peerling Creation Pipeline

> How a player's free-text wish becomes a published Peerling
> [species](../glossary.md#species): concept (including type) → image → player
> review → 3D model → stats and moves → the player's browser publishes to IPFS,
> the server pins and lists it. This page is the canonical description of the
> pipeline's stages, their order and the publish handshake.

## Why it works this way

- [accepted] All Peerlings are user-generated ([D-0002](../decisions/D-0002-all-peerlings-user-generated.md)).
  The player only describes; AI models do the art, so anyone can create.
- [accepted] The player keeps creative control at the most visible point — the
  image — by accepting or regenerating it.
- [accepted] The type is decided at the concept stage, so the image can show
  it (a Fire Peerling looks fiery).
- [accepted] Battle data (type, stats, moves) is generated within fixed rules,
  so a clever description cannot produce an overpowered creature.
- [accepted] The player's own browser publishes the result to IPFS
  ([D-0007](../decisions/D-0007-players-publish-assets.md)). Creating a Peerling
  is the player's first hands-on IPFS moment.
- [proposed] The *art style* is fixed by the system, not the player, so every
  Peerling looks like it belongs in the same game. The player controls *what*
  the creature is; the pipeline controls *how* it is drawn.

## Who uses the pipeline

- [accepted] New players who choose to create their starter during
  [onboarding](../gameplay/onboarding.md). (They may instead pick one of a few
  existing species; [D-0012](../decisions/D-0012-starter-choice-and-extra-creations.md).)
- [accepted] Players who earn an additional creation at a
  [Creation Shrine](../gameplay/creation-shrine.md).
- [accepted] The operator, who creates a handful of
  [seed species](../glossary.md#seed-species) at launch through this same
  pipeline ([D-0002](../decisions/D-0002-all-peerlings-user-generated.md)).

## Stages

| # | Stage | Runs on | Input | Output | Provenance |
|---|-------|---------|-------|--------|------------|
| 1 | Wish | client | player text | wish text | [accepted] |
| 2 | Concept | server: concept LLM | wish | [concept](../glossary.md#concept) JSON, including type(s) | [accepted] |
| 3 | Image | server: image generator | concept → structured (JSON) image prompt | 2D image | [accepted] |
| 4 | Review | client | image | accept / regenerate | [accepted] |
| 5 | 3D model | server: image-to-3D | accepted image | static 3D model (GLB) | [accepted] |
| 6 | Stats & moves | server: concept LLM + validator | concept | base stats, moves | [accepted] |
| 7 | Publish | client adds to IPFS; server verifies, pins, signs the listing; client appends it | everything above | species on IPFS + registry entry | [accepted] (handshake details: [proposed]) |
| 8 | Starter | client | published species CID | [starter](../glossary.md#starter) instance in the player's save | [accepted] |

```mermaid
sequenceDiagram
  participant P as Player browser (Helia)
  participant S as Generation server
  participant R as OrbitDB registry
  P->>S: 1. wish text
  S->>S: 2. concept LLM → concept JSON (incl. types)
  loop until accepted (no limit; cooldown CRE-021)
    S->>S: 3. image prompt JSON → image generator
    S-->>P: image + concept summary
    P->>S: 4. accept / regenerate
  end
  S->>S: 5. image-to-3D → GLB, post-process
  S->>S: 6. LLM → stats, moves → validator
  S-->>P: 7a. assets + signed species record
  P->>P: 7b. add all to own Helia node → CIDs
  P->>S: 7c. report CIDs
  S->>P: 7d. fetch by CID over IPFS, verify, pin
  S-->>P: 7e. signed registry listing
  P->>R: 7f. append registry entry
  P->>P: 8. create starter instance
```

### Stage 1 — Wish
The player describes the Peerling they want in free text (e.g. *"a small
sleepy fox made of moss that carries a lantern"*). [proposed] The client shows
a few example wishes and a character limit. [proposed] The wish is sent with the
player's identity ([player-character](../gameplay/player-character.md)) so the server can apply
rate limits.

### Stage 2 — Concept
The [concept LLM](../glossary.md#concept-llm) turns the wish into a structured
concept. [accepted] The concept LLM also decides the Peerling's
[type(s)](types.md) from the player's description at this stage. Concept
fields ([proposed] except `types`):

| Field | Description |
|-------|-------------|
| `nameSuggestions` | 3 short name ideas, offered to the player as inspiration. The player chooses the final name ([Final review](#final-review)) |
| `summary` | One sentence describing the creature |
| `lore` | 2–4 sentences of flavor text shown in-game |
| `types` | [accepted] Type(s) from the type list, chosen to fit the description (count: [TYP-002](types.md#requirements)) |
| `appearance` | Structured visual description: body plan, size class, colors, materials/textures, distinctive features, pose. Reflects the chosen type(s) |
| `temperament` | Short personality description (flavor; may inform animation style) |

The concept must stay faithful to the wish; the LLM elaborates, it does not
replace the player's idea. [proposed] The validator checks `types` against the
type list right away, so the image is never generated for an invalid concept.

### Stage 3 — Image
The concept's `appearance` is converted to a structured (JSON) prompt for the
image generator (the brief names FLUX.2 or similar models that accept JSON
prompts for precise control). [proposed] The prompt has two parts:

- **House-style block (fixed by the system):** art style matching the game's
  colorful, stylized world ([visual-style](../world/visual-style.md#visual-style)), lighting, camera,
  and constraints that make the image a good input for image-to-3D: exactly one
  creature, full body visible, centered, three-quarter front view, plain
  neutral background, no text, no ground shadows or props cut by the frame.
- **Subject block (from the concept):** the creature itself.

The seed and the full prompt are recorded for provenance.

### Stage 4 — Review
The player sees the image and either **accepts** it or **regenerates**.
[proposed] Regenerate uses a new seed; the player may adjust their wish before
regenerating.

[accepted] There is **no limit** on the number of image generations. A
**cooldown** between generations prevents spam. [proposed] The cooldown is 30
seconds per player, counted from when the previous image was delivered, and
the server's job queue keeps GPU time fair between players.

### Stage 5 — 3D model
The accepted image goes to the self-hosted image-to-3D generator (the brief
names TRELLIS.2 or similar). [accepted] The output is a **static** 3D asset,
with no rigging or skeletal animation. Motion in battles comes from simple
procedural animation ([battle § Presentation](../gameplay/battle.md#presentation)).
[proposed] Steps:
1. Background removal / subject matting of the image.
2. Image-to-3D generation → textured mesh.
3. Post-processing: normalize scale (fits a unit bounding box), orientation
   (faces +Z, up is +Y), place the lowest point at y = 0, decimate and compress
   to the asset budget ([Q-016](../open-questions.md#q-016)), export as binary
   glTF (`.glb`).
4. Render a small thumbnail of the model for lists and menus.

[accepted] There is no separate approval or retry of the 3D model: the
image-to-3D generator gives essentially the same result every time for the same
image. The player judges the model in the [final review](#final-review).

### Stage 6 — Stats & moves
[accepted] The LLM spreads the base stats to fit the concept, within the
same fixed stat total for every species
([peerling-species § Stats](peerling-species.md#stats)). It also creates
[moves](moves.md) that follow the move templates and the move-set rules there.
The LLM *proposes*; [proposed] a deterministic **validator** on the server
checks every rule and rejects (and re-prompts) or clamps anything out of bounds.
The validator, not the LLM, is the authority on balance.

### Final review

[accepted] Before publishing, the player sees the finished Peerling: the
rotatable 3D model, types, stats and moves. The player **names** it here: the
LLM's name suggestions are shown, but the name is the player's choice.
[accepted] If the player dislikes the 3D model, they can **restart from the
image generation stage** (stage 3) instead of publishing.

[proposed] Names are 1–20 characters long and need not be unique; the species
CID is its identity.

### Stage 7 — Publish
[accepted] The player's browser adds the Peerling to IPFS through its Helia
node; the server pins it and signs its registry listing; the player's browser
appends the registry entry ([D-0007](../decisions/D-0007-players-publish-assets.md),
[D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)).

[proposed] Handshake:

| Step | Actor | Action |
|------|-------|--------|
| 7a | server | Sends the client the image, model and thumbnail files plus the [species record](peerling-species.md#species-record), with asset CIDs filled in and signed ([attestation](../glossary.md#attestation)). The server computes the asset CIDs using the [fixed import parameters](../tech/ipfs-helia.md#content-import-parameters). |
| 7b | client | Adds the three asset files and the species record to its Helia node, using the same import parameters. The asset CIDs must equal those in the record. |
| 7c | client | Reports the species CID to the server. The client keeps providing the content. |
| 7d | server | Fetches the species record and every asset by CID from the network (in practice from the player's node), checks they are byte-identical to what it generated, and pins them. |
| 7e | server | Assigns the next registry `seq` and signs the registry listing ([orbitdb-registry](../tech/orbitdb-registry.md#design)); sends it to the client. |
| 7f | client | Appends the signed entry to the registry. If it doesn't appear within a minute (e.g. the client disconnected), the server appends the same entry itself. Job state `PUBLISHED`. |

If the client disconnects during 7b–7d, the job waits in `PUBLISHING` until
the client reconnects and resumes providing (CRE-014). The species becomes
visible to other players at 7f.

### Stage 8 — Starter
The client creates the player's starter
[instance](../glossary.md#peerling-instance) in its save from the species CID
([D-0006](../decisions/D-0006-species-vs-instance.md)).

## Job handling

[proposed] Stages 2, 3, 5 and 6 are GPU jobs that can take from seconds to
minutes. The pipeline is modelled as a server-side **creation job** with a state
machine; the client follows its progress.

```
WISH_SUBMITTED → CONCEPT_READY → IMAGE_READY ⇄ (regenerate)
  → IMAGE_ACCEPTED → MODEL_READY → PROFILE_READY (final review)
  → PUBLISHING → PUBLISHED
PROFILE_READY → IMAGE_READY (player restarts from image generation)
any state → FAILED (error, retryable) | EXPIRED (abandoned)
```

[proposed] Onboarding overlaps waiting time with other activity (e.g. character
creation runs while the 3D model generates) — see
[onboarding](../gameplay/onboarding.md).

## Requirements

- **CRE-001** [accepted] Every Peerling species MUST be created through this pipeline; there are no hand-authored species. This includes the operator's seed species.
- **CRE-002** [accepted] The player MUST describe the Peerling in free text; the concept MUST be generated from that description by the self-hosted LLM.
- **CRE-003** [accepted] The image MUST be generated from the concept by the image generator, and the player MUST be able to accept it or request a regeneration.
- **CRE-004** [accepted] The 3D model MUST be generated from the accepted image by the image-to-3D generator, and MUST be the asset used to show the Peerling in-game.
- **CRE-005** [accepted] Each species' type(s) MUST come from the predefined type list in [types](types.md).
- **CRE-006** [accepted] Each move MUST conform to a move template in [moves](moves.md).
- **CRE-007** [accepted] The species data and 3D model MUST be stored on IPFS and the species MUST be added to the OrbitDB registry.
- **CRE-008** [proposed] All LLM outputs MUST be requested as JSON and validated against a schema; invalid output MUST be retried, never passed on.
- **CRE-009** [proposed] A deterministic server-side validator MUST check types, stats and moves against the rules before publishing; LLM output alone MUST NOT be trusted for balance.
- **CRE-010** [proposed] The image prompt MUST include the fixed house-style block so all Peerlings share one art style and produce clean image-to-3D input.
- ~~**CRE-011**~~ (removed 2026-10-04: no content moderation, see [D-0010](../decisions/D-0010-no-content-moderation.md))
- **CRE-012** [proposed] Every species record MUST include provenance: wish text, concept, image prompt, seeds, and model names/versions used at each stage.
- **CRE-013** [proposed] The 3D model MUST be post-processed to a normalized scale, orientation and ground position, and MUST fit the asset budget ([Q-016](../open-questions.md#q-016)).
- **CRE-014** [proposed] The pipeline MUST run as a resumable server-side job; the client MUST show progress and MUST be able to reconnect to an in-progress job after a page reload.
- **CRE-015** [accepted] A published species MUST be immutable; there is no edit operation ([D-0006](../decisions/D-0006-species-vs-instance.md)).
- **CRE-016** [accepted] The concept LLM MUST determine the type(s) in stage 2, from the player's description, before the image is generated; the image MUST reflect the type(s).
- **CRE-017** [accepted] The 3D model MUST be static (unrigged); the pipeline MUST NOT depend on rigging or skeletal animation.
- **CRE-018** [accepted] The player's browser MUST add the species record and its assets to IPFS through its own Helia node.
- ~~**CRE-019**~~ (removed 2026-10-04, replaced by CRE-024; see D-0013)
- **CRE-020** [proposed] Before pinning, the server MUST verify that the content fetched by CID is byte-identical to what it generated; on mismatch, the job MUST fail and nothing is listed.
- **CRE-021** [accepted] Image generation MUST have no count limit, but MUST enforce a per-player cooldown between generations.
- **CRE-022** [accepted] Before publishing, the player MUST see the final Peerling and MUST be able to restart from image generation instead of publishing.
- **CRE-023** [accepted] The player MUST choose the name of every Peerling they create.
- **CRE-024** [accepted] The server MUST pin the species record and all its assets before signing the registry listing.

## Open questions

[Q-016](../open-questions.md#q-016)

## See also

- [Peerling species & data model](peerling-species.md)
- [Generation server](../tech/generation-server.md)
- [Onboarding](../gameplay/onboarding.md)
