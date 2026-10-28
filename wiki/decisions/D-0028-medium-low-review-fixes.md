---
title: "D-0028: Fixes for the medium- and low-severity specification review findings"
type: decision
status: accepted
tags: [tech, verification, saves, trading, pvp, creation, world]
sources:
  - raw/conversations/2026-10-28-medium-low-resolutions.md
related:
  - wiki/specification-review.md
  - wiki/decisions/D-0027-high-severity-review-fixes.md
  - wiki/tech/player-data.md
  - wiki/tech/data-formats.md
  - wiki/peerlings/image-prompting.md
updated: 2026-10-28
---

# D-0028: Fixes for the medium- and low-severity specification review findings

**Status:** accepted (2026-10-28). Changes parts of D-0010 (what delisting
does to owned Peerlings), D-0011 (UDP-blocked networks), D-0022 (Peerling of
the Day weight) and D-0025 (resemblance check, IMG-001 exceptions).

## Context
After [D-0027](D-0027-high-severity-review-fixes.md) fixed the high-severity
findings of the [specification review](../specification-review.md), 31
medium- and low-severity findings remained. The designer asked for each to be
resolved where the fix is clear, and decided eight choices.

## Decision
[accepted] Designer's choices:
1. **Fast fled encounters (REVIEW-010)** are an accepted gap: a modified
   client can skip walking and battles, so rerolling costs only time.
2. **Peerling of the Day (REVIEW-022)** weight = max(2 × its normal weight,
   W div 19).
3. **Delisting (REVIEW-027):** owned Peerlings of a delisted species are
   shown as a silhouette named "Delisted Peerling"; nobody keeps or serves its
   image or model; it still battles, but can't be traded, followed or shared.
4. **Clones (REVIEW-033):** a hero image still flagged by the resemblance
   check after the automatic retry can't be accepted.
5. **UDP-blocked networks (REVIEW-041)** are unsupported in v1; the client
   says so, and the API may fall back to HTTP/2.
6. **Recovery phrase (REVIEW-005):** the device and backups keep the
   phrase's entropy, so it can be shown again.
7. **Seen (REVIEW-025):** wild, guardian and PvP opponents' Peerlings, and
   starters, creations and trades; the first biome is the player's tile.
8. **Proposals in accepted pages (REVIEW-031, 032)** listed by the designer
   are approved; the rest of the draft image-prompting page is
   [Q-059](../open-questions.md#q-059).

[accepted] Clear fixes, approved with the request:
9. **Save reconstruction (REVIEW-002):** a new `rest` event; a lost battle
   heals and returns the player; snapshots are caches of the events.
10. **Equivalent `prev` (REVIEW-014):** transfers that differ only in which
    equivalent entry `prev` names are one transfer.
11. **Offer state (REVIEW-015):** received offer data is stored before
    signing and can be rebuilt from the sender's save log.
12. **PvP proofs (REVIEW-018):** a signed state for turn 0, the mode in every
    state signature, and profiles check the opponent's log for forfeits.
13. **Spectating (REVIEW-019, 020):** turn messages carry only that turn's
    actions plus the CIDs of the start message and of the action history.
14. **First wild encounter (REVIEW-023):** a guardian battle doesn't use up
    the guarantee.
15. **Prefetching (REVIEW-024):** recompute after a new sighting.
16. **World updates (REVIEW-029):** the battle's epoch record fixes the
    generator version; "nearest" is defined; snapshots name their version.
17. **Adjacency (REVIEW-030):** the 4 tiles directly next to a player; a
    `jump` flag for respawns and relocations.
18. **Image checks (REVIEW-034):** which failed checks the player may accept
    and which block the image.
19. **Name matching (REVIEW-035):** exact normalization order, tokenization,
    scripts without spaces, empty names.
20. **Creation jobs (REVIEW-036):** a retry endpoint; an open event stream
    counts as activity.
21. **Seed species (REVIEW-038):** a `seed` job kind for operator accounts.
22. **Shrine credits (REVIEW-039)** keep their level and offering and are
    returned with their original expiry.
23. **Schemas (REVIEW-040):** an exact concept schema, distinct types,
    code-point lengths everywhere, a `profile` model in provenance.
24. **Storage (REVIEW-042):** retained content sits outside the 1 GB cap,
    with a defined order when storage is full.
25. **Profile links (REVIEW-043):** the link never changes; after 30 days
    without play it shows the last known profile with its date.
26. **Phone backup (REVIEW-044):** the QR code is a Viewer URL.
27. **Creator stats (REVIEW-045):** encounters are checked from `encounter`
    events; trades count once per trade entry; void trades drop out.
28. **First finder (REVIEW-046):** each hourly snapshot considers the catches
    verified by then.
29. **Examples (REVIEW-047):** the species example sums to 320; D-0006 and
    D-0025 carry notes on what changed.

## Consequences
- New save-log event `rest`; new fields `generator` (snapshot), `entropy`
  (backup), `jump` (presence), `start` and `history` (spectating), `level` and
  `releaseEntry` (shrine credit), `models.profile` (provenance).
- New endpoint `POST /v1/jobs/{id}/retry`; new job kind `seed`.
- Requirements SRV-002, REG-010, NODE-004, EXP-011 and SPT-002 are replaced
  by SRV-008, REG-011, NODE-009, EXP-014 and SPT-005; IMG-001's text names its
  exceptions.

## Alternatives considered
The rejected options are listed with each question in the
[source](../../raw/conversations/2026-10-28-medium-low-resolutions.md): a
walking-speed check, other day weights, unchanged owned Peerlings after
delisting, a best-effort originality check, a WebSocket fallback, a phrase
shown only once, narrower or wider "seen", and leaving the proposals open.
