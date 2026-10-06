---
title: Multiplayer — The Shared World
type: system
status: draft
req_prefix: MPL
tags: [gameplay, multiplayer, social]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-05-peer-save-backups.md
  - raw/conversations/2026-10-05-showcase-features.md
related:
  - wiki/decisions/D-0008-shared-multiplayer-world.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/tech/realtime-networking.md
  - wiki/world/procedural-generation.md
updated: 2026-10-05
---

# Multiplayer — The Shared World

> All players explore one shared world, see each other, and can battle and
> trade ([D-0008](../decisions/D-0008-shared-multiplayer-world.md)). This page
> covers what players experience; the networking underneath is in
> [realtime-networking](../tech/realtime-networking.md).

## What players experience

- [accepted] Everyone moves around in the **same world**: the same terrain,
  biomes and places for every player.
- [accepted] Players can start a **[PvP battle](pvp-battles.md)** with another
  player.
- [accepted] Players can **[trade](trading.md)** Peerlings with each other.
- [accepted] Nearby players are visible as their
  [player character](player-character.md), with a display name above them.
  Their position and movement update live.
- [accepted] Battles and trades can only be started when the two players are
  **next to each other** in the world, meaning within 3 metres.
- [accepted] With grid movement, "within 3 metres" means standing on a
  neighbouring tile ([exploration § Grid movement](exploration.md#grid-movement)).
  The player starts an interaction by facing the other player and pressing
  the interact key.
- [accepted] Interacting with an adjacent player's character opens a menu:
  *Challenge to battle* (choosing Fair or Real-levels mode, [pvp-battles](pvp-battles.md#fairness)), *Propose trade*, *View profile* (fetches their save, which also keeps a backup of it: [player-data § Keeping saves available](../tech/player-data.md#keeping-saves-available); their team, and the
  species they created). Once a battle or trade has started, it continues even
  if a player moves away.
- [accepted] Nearby PvP battles can be watched ([spectating](spectating.md)), and
  notable events across the world appear in a live feed ([world-feed](world-feed.md)).
- [accepted] Wild encounters are **per player**: each player meets their own
  wild Peerlings, even when standing next to someone else. This avoids
  competing for the same creature.

## Communication

[accepted] There is **no chat**. Players communicate only with
[emotes](../glossary.md#emote). [accepted] The fixed set below, sent through the
presence channel ([realtime-networking](../tech/realtime-networking.md#presence))
and shown as a bubble or animation above the player's character:

| Emote | Meaning |
|-------|---------|
| Wave | Hello / goodbye |
| Heart | Like / thanks |
| Laugh | Fun |
| Wow | Surprise / admiration |
| Thumbs up | OK / agree |
| Thumbs down | No / disagree |
| Challenge | "Want to battle?" |
| Trade | "Want to trade?" |

Why no chat: emotes keep interactions light and friendly between strangers,
and they work in every language.

## Scale and visibility

[accepted] Approved 2026-10-04:
- A client shows players in its own and the 8 neighbouring
  [regions](../glossary.md#region): a 192 m × 192 m area, well beyond the
  camera's 30–40 m view ([visual-style](../world/visual-style.md#camera)), so
  players appear before they come on screen.
- At most the **30 nearest** players are drawn. If there are more, a small
  indicator shows "+N players nearby". This matters mostly at the busy spawn.
- Other players' movement is smoothed (interpolated) between presence
  updates and drawn about 150 ms behind real time.

Network rates and region size are in
[realtime-networking § Presence](../tech/realtime-networking.md#presence).

## Requirements

- **MPL-001** [accepted] All players MUST share one world.
- **MPL-002** [accepted] Players MUST be able to battle each other and trade Peerlings.
- **MPL-003** [accepted] A client MUST show other players who are near it in the world, with live movement.
- **MPL-004** [accepted] Wild encounters MUST be local to each player; other players MUST NOT be able to interfere with them.
- **MPL-005** [accepted] Battle and trade requests MUST require explicit acceptance by the receiving player, who MUST be able to block or ignore a player.
- **MPL-006** [accepted] A battle or trade request MUST only be possible when the two players are within 3 m of each other in the world.
- **MPL-007** [accepted] There MUST NOT be free-text chat between players; players communicate only with emotes from a fixed set.

## Open questions

_None at the moment._

## See also

- [PvP battles](pvp-battles.md) · [Trading](trading.md) · [Realtime networking](../tech/realtime-networking.md)
