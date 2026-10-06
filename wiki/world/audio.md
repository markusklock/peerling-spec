---
title: Music, Sound Effects and Peerling Cries
type: system
status: accepted
req_prefix: AUD
tags: [world, audio, presentation]
sources:
  - raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
  - raw/conversations/2026-10-06-cry-details-approved.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-proposals-approved.md
related:
  - wiki/decisions/D-0019-peerdex-ui-audio.md
  - wiki/world/visual-style.md
  - wiki/peerlings/types.md
updated: 2026-10-06
---

# Music, Sound Effects and Peerling Cries

> What the game sounds like: hand-made music and sound effects shipped with the
> app, and a unique cry for every Peerling species, synthesized in the browser
> from its CID.

[accepted] Decided 2026-10-06 ([D-0019](../decisions/D-0019-peerdex-ui-audio.md)).

## Music

Hand-made or licensed royalty-free music, shipped with the game app (like the
environment art, [visual-style § Environment art](visual-style.md#environment-art)):
- one looping theme per biome;
- a theme for the spawn hub;
- battle music, one track for wild battles and one for PvP.

## Sound effects

Also hand-made or licensed and shipped with the app:
- footsteps per tile surface (grass, sand, snow, stone, wood);
- move effects per type;
- UI sounds;
- jingles for a successful catch, a level-up, a shimmer appearing, and a rest
  point healing the team.

## Peerling cries

Every species has its own cry, **synthesized in the browser** with the Web
Audio API, so no audio files are generated or downloaded.

- **Deterministic:** all cry parameters are derived from SHA-256 of the
  species CID's bytes and from its primary type, so every player hears the
  same cry for the same species.
- **Parameters** (approved 2026-10-06) from the hash bytes, in this order:

  | Byte | Parameter | Range |
  |------|-----------|-------|
  | 0 | Base pitch | 110–880 Hz (spread evenly on a log scale) |
  | 1 | Number of syllables | 1–3 |
  | 2 | Syllable length | 80–250 ms |
  | 3 | Pitch contour | rising, falling, rise-fall, or flat |
  | 4 | Vibrato depth | 0–1 semitone |
  | 5 | Noise mix | 0–40% |

  [accepted] Exact mapping, with b = the byte's value (0–255):

  | Parameter | Formula |
  |-----------|---------|
  | Base pitch | 110 × 2^(3 × b ÷ 255) Hz (110 Hz at 0, 880 Hz at 255) |
  | Syllables | 1 + (b mod 3) |
  | Syllable length | 80 + b × 170 div 255 ms |
  | Contour | b mod 4: 0 rising, 1 falling, 2 rise-fall, 3 flat |
  | Vibrato depth | b ÷ 255 semitone |
  | Noise mix | b × 40 div 255 % |

  Cries are never verified, so floating point is fine here.

- **Type flavour** [accepted] (the timbre each type adds; the exact sound design is the
  implementer's choice, as long as it depends only on the parameters above and
  the type):

  | Type | Flavour |
  |------|---------|
  | Normal | Plain, soft tone |
  | Fire | Crackling noise bursts |
  | Water | Bubbly, gurgling modulation |
  | Grass | Soft, breathy |
  | Electric | Buzzing, square-wave crackle |
  | Earth | Low rumble |
  | Air | Airy whistle |
  | Ice | Glassy, bell-like |
  | Metal | Metallic ring |
  | Light | Shimmering chorus |
  | Shadow | Low, distorted |
  | Spirit | Echoing, reverberant |

- **When it plays** [accepted]: when a Peerling enters a battle, when its card is opened,
  and when the player's own Peerling is chosen as starter.

## Requirements

- **AUD-001** [accepted] Music and sound effects MUST be hand-made or licensed assets shipped with the game app, including one theme per biome, a hub theme and wild and PvP battle music.
- **AUD-002** [accepted] Every species MUST have a cry synthesized in the browser, derived deterministically from the species CID and its primary type.
- **AUD-003** [accepted] Cry parameters and type flavours MUST follow the tables on this page.

## Open questions

_None at the moment._

## See also

- [Visual style](visual-style.md) · [Types](../peerlings/types.md) · [UI § Settings](../gameplay/ui.md#settings)
