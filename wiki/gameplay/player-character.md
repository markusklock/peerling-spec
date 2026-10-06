---
title: Player Character and Identity
type: system
status: accepted
req_prefix: PLR
tags: [gameplay, player, identity]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
related:
  - wiki/gameplay/onboarding.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-06
---

# Player Character and Identity

> The avatar the player controls, and the player's identity and save data.

## Character

[accepted] Every new player creates a character.

### Options considered

Suggested 2026-10-04 at the designer's request:

| Option | How | Pros | Cons |
|--------|-----|------|------|
| **A. Parts-based customizer** (recommended) | A few game-made, rigged, stylized base bodies; the player picks colors (skin, hair, outfit), a hairstyle and accessories (hats, glasses, backpacks; about 10 of each) | Walks, idles and plays emotes properly (rigged); stored as a tiny JSON, so other players render it instantly with no download; consistent with the colorful style | Characters are not AI-generated |
| B. AI-generated avatar | The player describes their character; the Peerling pipeline makes a 3D model | Fits the "everything is generated" theme | Generated models are static and unrigged, so walking looks like a sliding statue; every nearby player downloads 1–2 MB per avatar; extra GPU load |
| C. Hybrid | A + an AI-generated 2D portrait for the profile card (image generation only, no 3D) | A personal touch at low cost | Another generation step during onboarding |

### Decision

[accepted] Approved 2026-10-04: option **A** for the first version, optionally
with **C**'s portrait later.
The character's appearance is a small JSON document (body, colors, hairstyle,
accessories) stored in the `profile` event of the save log
([player-data](../tech/player-data.md#save-log)). Presence messages carry a
short hash of it, and other players fetch the full JSON from the player's
profile when the hash changes. The base bodies and accessories ship with the
game app.

## Identity and save

[accepted] No traditional accounts. On first launch the client generates a
cryptographic keypair; its public key is the player's identity (used as the
[creator](../glossary.md#creator) ID on species and for server rate limits).
[accepted] The same key is also the player's libp2p peer ID, OrbitDB identity
and IPNS name ([D-0018](../decisions/D-0018-one-key-per-player.md)).
[accepted] The save is a per-player OrbitDB log, and the key can be restored
with a recovery phrase. What the save contains, and how it is stored and
verified, is canonical in [player-data](../tech/player-data.md).

[accepted] Because the world is shared ([multiplayer](multiplayer.md)), the
identity also signs presence messages, PvP commitments and trade records. The
display name and avatar are visible to other players. [accepted] Display names
are not moderated ([D-0010](../decisions/D-0010-no-content-moderation.md)).

## Requirements

- **PLR-001** [accepted] Each player MUST have a player character created during onboarding.
- **PLR-002** [accepted] Each player MUST have a stable cryptographic identity generated client-side.
- ~~**PLR-003**~~ (removed 2026-10-04: no content moderation, see [D-0010](../decisions/D-0010-no-content-moderation.md))
- **PLR-004** [accepted] The player character MUST be built from game-made, rigged parts (body, colors, hairstyle, accessories) described by a small JSON document.

## Open questions

_None at the moment._
