---
title: Spectating PvP Battles
type: system
status: draft
req_prefix: SPT
tags: [gameplay, multiplayer, pvp, pubsub, showcase]
sources:
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
related:
  - wiki/decisions/D-0016-showcase-features.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/battle.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-05
---

# Spectating PvP Battles

> Nearby players can watch a PvP battle live. The two fighters publish each
> turn on a per-battle pubsub topic, and every spectator replays the battle
> with the same deterministic engine, with no server involved.

[accepted] Players can watch other players' PvP battles
([D-0016](../decisions/D-0016-showcase-features.md)).

## Design

Exact messages: [protocols](../tech/protocols.md).

[accepted] Approved 2026-10-05.

- **Finding a battle.** A PvP battle happens on the spot in the world; the two
  fighters' characters show a battle indicator. A nearby player interacts with
  either fighter (or clicks the indicator) and chooses *Watch*.
- **The battle topic.** Each PvP battle has a pubsub topic
  `peerlings/v1/battle/<battleId>`, where `battleId` is the hash of both player
  IDs and the battle start time. The fighters announce it in their presence
  messages while the battle runs.
- **What the fighters publish.** At the start: both teams (species CIDs,
  instance IDs), the level mode and the shared RNG seed (once revealed). After
  each turn: both revealed actions and the resulting battle-state hash. Each
  turn message also carries all earlier actions, so a spectator who joins late
  catches up from any single message. Battles are short, so this stays small.
- **What spectators do.** They run the same deterministic battle engine
  ([BTL-002](battle.md#requirements)) on the published actions, so they see
  exactly what the fighters see: the battle screen without controls, from the
  battle camera. If a state hash doesn't match their own result, they show the
  battle as void, just like the fighters.
- **No advantage to fighters.** Actions are only published after both sides
  have revealed them (commit-reveal, [pvp-battles](pvp-battles.md#protocol-proposed)),
  so a spectator can't leak a choice before it is made. Teams are visible to
  both fighters anyway.
- **Opting out.** The challenge screen has a *Spectators allowed* switch, on by
  default. When it's off, the fighters don't announce or publish the topic.
- **Showing the crowd.** Fighters see a count of spectators: the number of
  peers subscribed to the battle topic.

## Requirements

- **SPT-001** [accepted] Players MUST be able to watch nearby PvP battles live.
- **SPT-002** [accepted] Fighters MUST publish the teams, level mode, RNG seed and each turn's revealed actions with the battle-state hash on a per-battle pubsub topic; each turn message MUST include all earlier actions.
- **SPT-003** [accepted] Spectators MUST render battles by running the deterministic battle engine on the published actions and MUST check the state hashes.
- **SPT-004** [accepted] A PvP challenge MUST include a *Spectators allowed* option, on by default.

## Open questions

_None at the moment._

## See also

- [PvP battles](pvp-battles.md) · [Battle](battle.md) · [Realtime networking](../tech/realtime-networking.md)
