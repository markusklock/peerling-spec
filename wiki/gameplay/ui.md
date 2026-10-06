---
title: Menus, HUD and Controls
type: system
status: accepted
req_prefix: UI
tags: [gameplay, ui, controls]
sources:
  - raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
  - raw/conversations/2026-10-06-review-decisions.md
related:
  - wiki/decisions/D-0019-peerdex-ui-audio.md
  - wiki/gameplay/exploration.md
  - wiki/gameplay/battle.md
  - wiki/tech/player-data.md
updated: 2026-10-06
---

# Menus, HUD and Controls

> The screens outside the world and battles, what's always on screen, the
> keyboard controls, and the settings.

[accepted] Decided 2026-10-06 ([D-0019](../decisions/D-0019-peerdex-ui-audio.md)).
The game targets desktop browsers only, so everything is designed for keyboard
and mouse ([STK-010](../tech/tech-stack.md#requirements)). **Language:** English
only in v1.

## Title screen

**Continue** · **New game** ([onboarding](onboarding.md)) · **Recover account**
(recovery phrase, backup file, or phone,
[player-data § Account recovery](../tech/player-data.md#account-recovery)) ·
**Settings**.

## Pause menu (Esc)

| Item | Opens |
|------|-------|
| Team | The team of up to 4 ([catching § Team and collection](catching.md#team-and-collection)) |
| Collection | All other owned Peerlings |
| Peerdex | [Peerdex](peerdex.md) |
| Map | The world map ([exploration § Map](exploration.md#map)) |
| Profile | The player's profile, with Share ([sharing](sharing.md)) |
| My creations | The player's species and their stats ([creator-feedback](creator-feedback.md)) |
| Network panel | Peers, data served, Peerlings stored ([ipfs-showcase](../tech/ipfs-showcase.md)) |
| Backup | Export backup file, phone backup, show recovery phrase ([player-data](../tech/player-data.md#account-recovery)) |
| Settings | Below |

## HUD (always on screen while exploring)

- The minimap in one corner ([exploration § Map](exploration.md#map)).
- The world-feed ticker in another corner ([world-feed](world-feed.md)).
- Small HP bars for the team.
- A key prompt when the player faces something they can interact with.

## Controls

| Key | Action |
|-----|--------|
| Arrow keys or W A S D | Move ([exploration § Grid movement](exploration.md#grid-movement)) |
| Space or Enter | Interact with the faced tile |
| E | Emote wheel ([multiplayer § Communication](multiplayer.md#communication)) |
| M | Map |
| Tab | Team |
| Esc | Pause menu / back |
| 1 – 3, S, C, F | In battle: moves, switch, catch, flee ([battle § Battle screen](battle.md#battle-screen)) |

All keys can be remapped in Settings.

## Settings

- Volume sliders: music, effects, Peerling cries ([audio](../world/audio.md)).
- Graphics quality and resolution scale.
- Key remapping.
- Default for "Spectators allowed" in PvP challenges ([spectating](spectating.md)).
- Toggles: world feed, network overlay.
- Blocked players: a list with Unblock ([multiplayer § Interactions](multiplayer.md#what-players-experience)).

## Requirements

- **UI-001** [accepted] The game MUST provide the title screen, pause menu, HUD, controls and settings defined on this page, with remappable keys.
- **UI-002** [accepted] The game MUST be in English only in v1.

## See also

- [Exploration](exploration.md) · [Battle](battle.md) · [Peerdex](peerdex.md)
