---
title: "D-0027: Fixes for the high-severity specification review findings"
type: decision
status: accepted
tags: [tech, verification, saves, trading, pvp, battle, creation]
sources:
  - raw/conversations/2026-10-27-high-severity-resolutions.md
related:
  - wiki/specification-review.md
  - wiki/tech/player-data.md
  - wiki/tech/data-formats.md
  - wiki/tech/protocols.md
  - wiki/tech/creation-api.md
  - wiki/gameplay/battle.md
  - wiki/gameplay/trading.md
updated: 2026-10-27
---

# D-0027: Fixes for the high-severity specification review findings

**Status:** accepted (2026-10-27). Changes parts of D-0013 (who loses a
Peerling after a double trade) and D-0024 (event order, day and week records,
PvP timers).

## Context
The [specification review](../specification-review.md) found 16
high-severity gaps: rules two honest clients could apply differently, states
with no defined recovery, and promises the formats could not keep. The
designer asked for the best fix for each, decided three player-facing choices
and approved the rest.

## Decision
[accepted]
1. **Catch records (REVIEW-001).** The instance inside a `catch` event has no
   `originProof`; its proof is the CID of that `catch` entry, filled in
   wherever the instance is copied ([data-formats § Save-log events](../tech/data-formats.md#save-log-events)).
2. **Encounter events and interrupted battles (REVIEW-003).** A new
   `encounter` event is written when a battle starts and consumes the
   encounter number. At the end the client writes `catch` (if caught) before
   `battle-result`. A battle interrupted by closing the game or a crash
   counts as fled; a missing `battle-result` after a `catch` is rebuilt by
   replay ([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)).
3. **Backups (REVIEW-004).** The backup holds the player's whole save log and
   the proofs of every Peerling they own, so it verifies with no network copy
   left ([player-data § Keeping saves available](../tech/player-data.md#keeping-saves-available)).
4. **Lost save (REVIEW-006).** If the key is recovered but no save exists
   anywhere, the player starts a new account; the old key stays the creator
   of its species (designer's choice).
5. **Startup race (REVIEW-007).** A starting client probes for an active
   session; if two sessions still meet, the earlier one keeps playing
   ([player-data § Using the same account on several computers](../tech/player-data.md#using-the-same-account-on-several-computers)).
6. **Catch checks (REVIEW-008).** Verifiers check the starting team against
   the catcher's own Peerlings and the caught instance against the replay.
7. **Pending results (REVIEW-009).** Verification gives *verified*,
   *invalid* or *pending*; only a check that fails on complete data is
   *invalid*.
8. **Day and week records (REVIEW-011).** Evidence is judged by the records it
   carries, even if a signed record turns up later; days and weeks before
   launch use the genesis record.
9. **Trade conflicts (REVIEW-012).** A transfer-log entry counts only as a
   whole. If any of its transfers is invalid or loses a conflict, the whole
   entry is void, every Peerling in it stays with the player who offered it,
   and entries built on it become void in turn.
10. **Unfinished trades (REVIEW-013).** Peerlings in an unfinished trade are
    locked; either player can ask the server to expire the trade, which voids
    any later entry for it without flagging anyone (designer's choice).
11. **PvP timers (REVIEW-016).** Each decision has a phase (action or
    replacement), a commit deadline and a reveal deadline
    ([protocols § PvP battle](../tech/protocols.md#peerlingsbattle100--pvp-battle)).
12. **PvP reconnect (REVIEW-017).** Either side may reconnect; both resend
    what the other hasn't confirmed. Without a reconnect, competing forfeit
    claims cancel as before.
13. **Battle edge cases (REVIEW-021).** Quick-flurry's second hit, the charge
    strike's target, a knockout on turn 200 and the same-type bonus are
    defined exactly ([battle](../gameplay/battle.md#move-mechanics)).
14. **Team invariant (REVIEW-026).** Shrine offerings follow the trade rule;
    a team that loses a member to a void trade is refilled from the
    collection ([catching § Team and collection](../gameplay/catching.md#team-and-collection)).
15. **Registry history (REVIEW-028).** Eligibility at a registry height uses
    each species' original listing `seq`; the registry index is authoritative.
16. **One starter (REVIEW-037).** One open starter job at most, atomic
    issuance, idempotent choosing and a starter status endpoint
    ([creation-api § Starters](../tech/creation-api.md#starters)).

## Consequences
- New save-log events `encounter` and `trade-void`; `battle-result` loses its
  `epoch`, `candidate`, `species` and `guardian` fields to `encounter`. New
  operator record `peerlings/trade-expired` in the transfer log.
- New messages `session-probe` and PvP `phase` fields; new endpoints
  `GET /v1/starter` and `POST /v1/trades/{trade}/expire`.
- Requirements SAVE-026 and ENC-009 are replaced by SAVE-028 and ENC-010; new
  requirements SAVE-029 to SAVE-038, BTL-015, PRT-004, TRD-006, CAT-010,
  API-007 to API-009.
- A player who received a Peerling in a void trade now gets back what they
  offered instead of only losing the received Peerling.
- Because `encounter` events log the tile, the species of non-catch
  encounters can now be checked too.

## Alternatives considered
- **Restarting on the same key after a lost save:** needs a reset record, a
  second log address and an exception to one starter per player. Rejected by
  the designer in favour of a new account.
- **Locking a half-finished trade forever, or unlocking after a wait:** the
  first can lock a Peerling for good; the second can get an honest player
  flagged. Rejected in favour of server expiry.
- **Resuming interrupted battles:** needs a save-log entry per turn. Rejected:
  fleeing always succeeds, so treating an interruption as fled gives no
  advantage.
- **Voiding only the conflicting transfer (the previous rule):** left the
  victim without the Peerling they gave and was ambiguous for multi-Peerling
  trades.
