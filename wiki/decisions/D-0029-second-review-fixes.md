---
title: "D-0029: Fixes for the findings of the second specification review"
type: decision
status: accepted
tags: [tech, verification, saves, trading, pvp, creation, world]
sources:
  - raw/conversations/2026-11-10-specification-review.md
  - raw/conversations/2026-11-10-second-review-resolutions.md
related:
  - wiki/specification-review.md
  - wiki/decisions/D-0027-high-severity-review-fixes.md
  - wiki/decisions/D-0028-medium-low-review-fixes.md
  - wiki/tech/player-data.md
  - wiki/tech/data-formats.md
  - wiki/tech/protocols.md
  - wiki/world/procedural-generation.md
updated: 2026-11-10
---

# D-0029: Fixes for the findings of the second specification review

**Status:** accepted (2026-11-10). Changes parts of D-0013 (forks), D-0017
(world generator), D-0021 and D-0024 (PvP result claims, base records, shrine
credits), D-0027 and D-0028 (details listed in their Status lines).

## Context
The [specification review](../specification-review.md) of 2026-11-10 found 35
new issues (REVIEW-048 to REVIEW-082) after
[D-0027](D-0027-high-severity-review-fixes.md) and
[D-0028](D-0028-medium-low-review-fixes.md). The designer asked for all of
them to be solved as seen fit, with questions only where a decision was
needed, and decided four choices.

## Decision
[accepted] Designer's choices:
1. **Forks (REVIEW-050):** a forked save log invalidates only the catches and
   badges from the lowest duplicated encounter number on, and never one that
   a server checkpoint already covers as valid. A catch traded away before any
   checkpoint covered it stays exposed: an accepted risk.
2. **World generator (REVIEW-066):** each generator version is one binding
   generator module, the same artefact in the client, the server and every
   verifier, identified by its hash and published with test vectors. The spec
   lists what the module answers, not its algorithm.
3. **PvP forfeits (REVIEW-056):** void and drawn battles are written to the
   save log (no counter changes), and a forfeit claim is cancelled by a void,
   a draw or a competing forfeit in the other player's log.
4. **Shrine credit (REVIEW-075):** its 30 days start when the first job for
   that offering was abandoned or expired, and are never renewed.

[accepted] Fixes made under the request ("solve them all the best you see
fit"). The ones marked **new rule** add behaviour rather than repair wording:
5. **Fainted team members (REVIEW-048):** the catch check allows 0 HP for
   team members, with at least one above 0.
6. **Base records (REVIEW-049), new rule:** a derived record of epoch E copies
   the newest signed record with an epoch of at most E − 2 that the client
   can get; clients keep 8 days of signed records; the server publishes a
   missed day or week record later.
7. **No target (REVIEW-051), new rule:** a move against a Peerling that
   already fainted this turn fails and draws nothing; a charge setup still
   charges, a charge strike is used up.
8. **Derived records in encounters (REVIEW-052):** an `encounter` event names
   the epoch of the base record of a client-derived epoch or day record.
9. **Badges (REVIEW-053):** `badge` is written before `battle-result`; a
   crashed first win is rebuilt from the badge; the badge checks are listed.
10. **Event effects (REVIEW-054), new rule:** an exact effect for every
    save-log event, including who joins the team, the refill after a void
    trade, and a new `generator` event.
11. **World and rules version (REVIEW-055), new rule:** a client plays in the
    generator and rules version of its latest epoch record, not of its clock;
    the base-record known gap now names the world and the rules too.
12. **Timeouts (REVIEW-057), new rule:** a 2 s grace before a `timeout`; a
    `timeout` for a message that was already sent ends the battle void.
13. **Resume (REVIEW-058):** a battle can be resumed from any point after
    `accept`; `have` tells "no state" from "turn-0 state"; a forfeit needs a
    signed state from the other player; the turn-1 timer starts at the turn-0
    states.
14. **Unfinished trades (REVIEW-059):** a `trade-pending` event holds the lock,
    the client's own transfers and the received offer data.
15. **Relays (REVIEW-060):** a `battle-blocks` stream for late spectators; a
    phone backup can continue after a cut connection.
16. **Wild move choice (REVIEW-061):** scored against the Peerling that is out
    before the turn's switch.
17. **Failed jobs (REVIEW-062):** `failedFrom`, an `error` object with codes,
    and `retry` with an edited wish.
18. **Image checks in the job (REVIEW-063):** `imageChecks.blocked`
    (accepted) and `imageChecks.failed` (proposed, Q-059); the error
    `image-blocked`.
19. **Creator name (REVIEW-064):** `finalize` carries the display name.
20. **Delisted cards (REVIEW-065), new rule:** the Viewer checks a species'
    listing with `GET /v1/listings/{species CID}`; the game uses its registry.
21. **Rest-point tiles (REVIEW-067), new rule:** a rest point's tile is its
    beacon tile; players respawn and start on the arrival tile south of it.
22. **Map (REVIEW-068):** the chunks revealed from the start, and
    "discovered", are defined from revealed chunks.
23. **Requests (REVIEW-073), new rule:** a pending request counts as busy;
    crossing requests are settled by player ID; unanswered requests are
    withdrawn after 60 s.
24. **XP (REVIEW-074):** a Peerling took part if it was active at any point.
25. **Deadline wording (REVIEW-072)**, the **glossary (REVIEW-069)**, the
    **event table (REVIEW-070)**, **onboarding's key and phrase steps
    (REVIEW-071)**, the **enhancer prompt examples (REVIEW-076)**, the
    **registry's signed fields (REVIEW-077)**, the **stats table
    (REVIEW-078)**, **decision and question notes (REVIEW-079)**, the
    **signpost example (REVIEW-080)**, the **cry triggers (REVIEW-081)** and
    the **phone-backup menu path (REVIEW-082)** were corrected.

## Consequences
- New save-log events `generator` and `trade-pending`; new `encounter` fields
  `base` and `dayBase`; `pvp-result` results `void` and `draw`.
- New stream `/peerlings/battle-blocks/1.0.0`; `have` in the phone-backup
  stream; the `timeout-disputed` error.
- New endpoint `GET /v1/listings/{species CID}`; job fields `failedFrom` and
  `imageChecks`; `wish` on `retry`; `displayName` on `finalize`; the error
  `image-blocked`.
- Requirements SAVE-013, SAVE-029 and SHR-005 are replaced by SAVE-042,
  SAVE-043 and SHR-007. New: SAVE-044 to SAVE-049, BTL-016 to BTL-018, PRT-005 to
  PRT-008, PVP-014, TRD-008, MPL-011, ONB-009, EXP-016, EXP-017, API-013 to
  API-016, LNK-005, WGN-023 to WGN-025.
- The server has two new duties: publishing missed day and week records, and
  never withdrawing a checkpoint's verdict on a catch.
- Building the game now includes building one generator module per version
  and its test vectors before any client or verifier work that depends on the
  world.

## Alternatives considered
The rejected options are listed with each question in the
[source](../../raw/conversations/2026-11-10-second-review-resolutions.md):
accepting forks as a plain risk or never letting them invalidate anything,
writing the full generator algorithm into the spec, leaving forged forfeits
as a known gap, and counting a shrine credit's 30 days from the offering.
