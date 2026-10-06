---
title: PvP Battles
type: system
status: accepted
req_prefix: PVP
tags: [gameplay, multiplayer, battle, libp2p]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-approvals-q016-q038.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-pvp-wins-hex-world.md
  - raw/conversations/2026-10-06-proposals-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-06-review-2-decisions.md
related:
  - wiki/gameplay/battle.md
  - wiki/gameplay/multiplayer.md
  - wiki/tech/realtime-networking.md
updated: 2026-10-06
---

# PvP Battles

> How two players battle each other peer-to-peer, without a game server
> deciding the outcome. The battle *rules* are the same as for wild battles
> ([battle](battle.md)); this page covers setup, fairness and the protocol.

## Overview

[accepted] Players can battle each other. [accepted] PvP battles are played
directly between the two players' browsers over a libp2p stream
([realtime-networking](../tech/realtime-networking.md)). Both clients run the
same deterministic battle engine (BTL-002) and must agree on every turn.

## Fairness

A modified client could lie about its Peerlings, so the protocol checks
everything it can (decided in [D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md) and
[D-0015](../decisions/D-0015-pvp-level-modes.md)):

- [accepted] **Species are verifiable.** Each Peerling's species record is
  fetched by CID and its attestation checked, so stats, types and moves can't
  be faked.
- [accepted] **Level mode** ([D-0015](../decisions/D-0015-pvp-level-modes.md)). The challenger picks a mode,
  and the other player sees it before accepting:
  - **Fair** (default): all Peerlings fight at level 50 (as in Pokémon's
    competitive formats), so edited levels give no advantage.
  - **Real levels:** all Peerlings fight at their actual level. Levels aren't
    verified ([player-data § Known gaps](../tech/player-data.md#known-gaps-accepted-risks)),
    so a modified client could fake them; by agreeing to this mode, both
    players accept that.
  [accepted] Individual stat traits still apply
  ([peerling-species § Individual variation](../peerlings/peerling-species.md#individual-variation));
  they are part of the verified Peerling, so they can't be faked.
- [accepted] **Only verified Peerlings.** Every Peerling in a PvP team must be
  verified: genuine origin and a valid ownership chain ending at its player
  ([player-data § Verified Peerlings](../tech/player-data.md#verified-peerlings)).
  Each side checks the other itself, so faked or traded-away Peerlings can't be
  used and no server is needed.

## Protocol

Exact messages: [protocols](../tech/protocols.md).

1. **Challenge.** A sends a challenge, including the level mode (Fair or Real
   levels) and whether spectators are allowed ([spectating](spectating.md)), to B while standing next to them (MPL-006); B accepts or declines
   (MPL-005).
2. **Team exchange.** Each side sends its team (species, instance and
   verification details per Peerling; exact fields in
   [protocols](../tech/protocols.md#peerlingsbattle100--pvp-battle)). Each side fetches and verifies the other's species and
   verifies each Peerling (cached results are reused).
3. **Shared randomness.** Both commit to a random value (send its hash), then
   reveal it. The battle RNG seed is derived from both values, so neither side
   controls the random rolls.
4. **Turns.** Each turn, both sides commit to their action (send a hash), then
   reveal it. Neither can react to the other's choice. Both run the engine and
   exchange a hash of the resulting battle state. A mismatch voids the battle.
5. **End.** The result is shown to both players. Timeouts follow
   [battle § PvP turn timer](battle.md#pvp-turn-timer) (a random move, two in a
   row forfeit); a player who disconnects has 60 s to resume, otherwise they
   forfeit. Every Peerling starts at full HP and the battle changes nothing
   afterwards ([battle § Ending a battle](battle.md#ending-a-battle)).

## Rewards

[accepted] No rewards that can be farmed: no XP or items from PvP.

[accepted] Each player has a **counter of PvP battles won**, shown on their
profile ([D-0021](../decisions/D-0021-pvp-win-counter.md)). In the designer's
words, with nothing to lose or gain PvP would feel meaningless; the counter
gives players something to show for it.

### Win record

[accepted] How wins are proven and recorded, so anyone can check a player's
count (approved 2026-10-06; exact messages in
[protocols](../tech/protocols.md#peerlingsbattle100--pvp-battle), exact event
in [data-formats § Save-log events](../tech/data-formats.md#save-log-events)):

- **Signed states.** Each `state` message (the per-turn battle-state hash) is
  signed by its sender with the player key. The `end` message names the
  result, the winner and the final state hash, also signed.
- **Proof of a result.** A normal win counts when the **loser** signed an
  `end` naming the winner (the loser has no reason to fake their own loss).
  A forfeit by timeout or disconnect needs the winner's signed `end` plus the
  loser's last signed state; it counts as a win, but is shown separately as
  "won by forfeit". A loser who closes the game instead of sending `end`
  therefore turns a normal win into a win by forfeit. Exact rules:
  [data-formats § Save-log events](../tech/data-formats.md#save-log-events).
- **Recording.** Each player, winner and loser, appends a `pvp-result` event
  to their own [save log](../tech/player-data.md#save-log) with the battle
  ID, both player IDs, the mode, the winner and the signatures that prove it.
  Verifiers check the signatures, so anyone can count a player's wins. A void
  battle is not recorded.
- **Showing it.** Wins and losses (Fair and Real-levels counted separately)
  appear on the player profile (in game and in the shared
  [profile document](../tech/data-formats.md#player-profile-document--peerlingsprofile)),
  with wins by forfeit and the number of different opponents beaten.
- [accepted] **Conflicting claims** (2026-10-06): a win the loser signed always
  beats a forfeit claim for the same battle, and two forfeit claims naming
  different winners cancel each other. The **in-game profile screen** counts
  only `pvp-result` events that verify, from the save log it fetches; the
  numbers in the shared profile document are the player's own claim.
- **No rewards.** It is a stat only; there are still no XP or items for PvP.
- **Known gap:** two friends (or one person with two accounts) can still
  play fixed battles to raise a win count. Since it gives no reward, this is
  accepted; the profile shows the number of different opponents beaten next
  to the total, which makes farming visible.

## Requirements

- **PVP-001** [accepted] Two players MUST be able to battle each other with their Peerlings.
- **PVP-002** [accepted] PvP battles MUST run peer-to-peer between the two clients, with no server deciding the outcome.
- **PVP-003** [accepted] Each client MUST verify every opposing species record (attestation) before the battle starts.
- **PVP-004** [accepted] The battle RNG seed MUST be derived from values committed and revealed by both players.
- **PVP-005** [accepted] Turn actions MUST use commit-reveal, so neither player sees the other's choice before committing.
- **PVP-006** [accepted] Clients MUST compare battle-state hashes after each turn and void the battle on mismatch.
- ~~**PVP-007**~~ (removed 2026-10-05, replaced by PVP-010; see D-0015)
- ~~**PVP-008**~~ (removed 2026-10-04, replaced by PVP-009; see D-0013)
- **PVP-009** [accepted] Each client MUST reject an opposing Peerling that is not verified or not owned by the opponent ([SAVE-003](../tech/player-data.md#requirements)), and MUST refuse a battle with a player flagged for a double trade ([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades)).
- **PVP-010** [accepted] A PvP challenge MUST state a level mode, Fair (all Peerlings at level 50, the default) or Real levels (actual levels, unverified), and both players MUST agree to it.
- **PVP-011** [accepted] The game MUST count each player's PvP wins and show the count on their profile.
- **PVP-012** [accepted] Only wins proven as in [Win record](#win-record) MUST be counted.

## Open questions

_None at the moment._

## See also

- [Battle rules](battle.md) · [Multiplayer](multiplayer.md)
