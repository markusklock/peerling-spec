---
title: "D-0022: Follower, guardians, Peerling of the Day and first finds"
type: decision
status: accepted
tags: [gameplay, world, social]
sources:
  - raw/conversations/2026-10-06-v1-fun-features.md
related:
  - wiki/gameplay/exploration.md
  - wiki/gameplay/guardians.md
  - wiki/gameplay/peerling-of-the-day.md
  - wiki/gameplay/creator-feedback.md
updated: 2026-10-06
---

# D-0022: Follower, guardians, Peerling of the Day and first finds

**Status:** accepted (2026-10-06)

## Context
The designer asked for features that would make the first version more fun.
The LLM suggested seven; the designer chose four.

## Decision
[accepted]
1. **Following Peerling:** the first Peerling in the player's team walks
   behind them in the world, drawn at its size class; other players see it
   ([exploration § Following Peerling](../gameplay/exploration.md#following-peerling)).
2. **Landmark guardians:** one guardian per biome, battled for that biome's
   badge (12 badges); the guardian team is drawn each week from that biome's
   Peerlings with the shared epoch randomness; badge wins are verified by
   replay; no server needed ([guardians](../gameplay/guardians.md)).
3. **Peerling of the Day:** each in-game day the shared randomness picks one
   species that appears more often everywhere; the world feed announces it,
   its creator is told, and a pedestal near the spawn shows its 3D model,
   explained on hover ([peerling-of-the-day](../gameplay/peerling-of-the-day.md)).
4. **"First found in the wild by …":** the first player with a verified wild
   catch of a species is credited on its card, and the world feed announces
   it ([creator-feedback § First found in the wild](../gameplay/creator-feedback.md#first-found-in-the-wild)).

The exact rules on those pages are [proposed] until approved.

## Consequences
- New save events (`badge`), presence and species-stats fields, and a feed
  event kind ([data-formats](../tech/data-formats.md), [protocols](../tech/protocols.md)).
- Guardians give v1 a set of goals beyond levelling and catching.
- Peerling of the Day, guardians and first finds all put player creations in
  front of other players, which rewards creators.

## Alternatives considered
- Ghost battles against offline players' teams, hearts for species, photo
  mode and catch clips: not chosen for v1.
