---
title: Specification Review
type: reference
status: draft
tags: [review, specification, implementation, tracking]
sources:
  - raw/conversations/2026-10-27-specification-review-tracker.md
  - raw/conversations/2026-10-27-specification-review-verification.md
  - raw/conversations/2026-10-27-high-severity-resolutions.md
  - raw/conversations/2026-10-28-medium-low-resolutions.md
  - raw/conversations/2026-11-10-specification-review.md
related:
  - wiki/index.md
  - wiki/open-questions.md
  - wiki/tech/data-formats.md
  - wiki/tech/player-data.md
  - wiki/tech/protocols.md
  - wiki/peerlings/image-prompting.md
  - wiki/decisions/D-0027-high-severity-review-fixes.md
  - wiki/decisions/D-0028-medium-low-review-fixes.md
updated: 2026-11-10
---

# Specification Review

> Tracks the findings of the specification reviews: REVIEW-001 to REVIEW-047
> from the review of 2026-10-19 (all Fixed) and REVIEW-048 to REVIEW-082 from
> the review of 2026-11-10 (all Open). Each finding has a severity, an editable
> status, evidence links and resolution notes. Finding IDs are stable, so each
> issue can be followed through later changes.

## How to use this tracker

[accepted] The designer requested this page on 2026-10-27, with every finding
initially **Open**, categorized as **High**, **Medium** or **Low** severity.
The designer will change statuses to **Fixed** or **Will-Not-Fix** after review.

| Status | Meaning |
|--------|---------|
| Open | Awaiting review, a decision, or a specification fix. |
| Fixed | Addressed in the specification; record links to the changed pages or decision in Resolution. |
| Will-Not-Fix | Reviewed and intentionally left unchanged; record the rationale in Resolution. |

[proposed] Severity is the reviewer's assessment of implementation impact:

| Severity | Meaning | Review of 2026-10-19 | Review of 2026-11-10 |
|----------|---------|---------:|---------:|
| High | Can block a core flow, lose progress or ownership, invalidate verification, or make clients disagree on authoritative state. | 16 | 4 |
| Medium | Leaves a feature incomplete, contradicts intended behavior, or requires a consequential implementation choice. | 23 | 18 |
| Low | A localized tutorial, navigation, example, or documentation issue with a clear broader design. | 8 | 13 |

The **Status** field on each finding is its only status record on this page.
Keep IDs stable when updating severity or status; do not delete resolved items.
Record the fix or the reason for declining it in **Resolution**.

[proposed] Findings and suggested resolution work below are review observations,
not accepted changes to the game. An Open status records outstanding review
work; it does not approve the diagnosis or a suggested solution. The linked
specification pages remain the canonical homes of game rules. These REVIEW
identifiers are tracking IDs, not requirement IDs or Q-NNN design questions.
Any resulting design question belongs in [Open questions](open-questions.md),
and significant accepted changes follow the repository's decision workflow.

## Review scope

The original review covered all 72 wiki pages then present, including the
decision records, and relevant prior-review sources. Its structural checks found
no broken local links or heading targets, unindexed or orphan pages, duplicate
requirement IDs, or unregistered prefixes across 325 declared requirement IDs
(including removed ones). These are historical results for 2026-10-19, recorded
in the [review log](log.md#2026-10-19-lint--full-specification-implementation-readiness-review).

[proposed] The core design is understandable, but implementation needs the
clarifications below. The recommended first priority is persistence and
protocol correctness, followed by shared conformance examples for hashes,
signatures, battle replays, historical registry selection and world generation.
The world algorithm is explicitly delegated to its implementation; its first
version should be frozen before independent clients and verifiers are built.

## Verification of 2026-10-27

[proposed] At the designer's request, every finding was checked again against
the current pages. All 47 were confirmed in full or in part. No open question,
accepted risk or decision record already answers any of them. Thirteen severities
were lowered: REVIEW-002, 010, 014, 015, 018, 036, 040 and 042 from High to
Medium, and REVIEW-024, 030, 032, 034 and 046 from Medium to Low. No severity
was raised. Finding text that misstated the spec was corrected, and each
finding's **Verification** note records the verdict, extra evidence and any
change. All reference links resolved to existing headings.

## Review of 2026-11-10

[proposed] At the designer's request the whole specification (75 wiki pages)
was reviewed again after D-0027 and D-0028. Four reviewers each read one area
line by line: persistence and encounters; battle, PvP, trading and protocols;
the creation pipeline and registry; world, UI, glossary, questions and
decisions. Every finding they reported was checked against the cited text
before it was recorded here. Things already fixed, already listed as an open
question or already recorded as an accepted risk were left out.

Mechanical checks found no broken local links or heading targets, no page
missing from the index, no duplicate requirement IDs and no unregistered
prefixes, across 370 declared requirement IDs (29 of them removed). The one
reference to a removed requirement outside history text is in
[REVIEW-079](#review-079-decision-status-lines-and-resolved-questions-miss-later-changes).

The 35 new findings are REVIEW-048 to REVIEW-082, in the three
[second-review sections](#second-review-high-severity) below. About half are
places where a D-0027 or D-0028 fix is incomplete or was not carried to every
page; each of those names the earlier finding. The recommended first priority
is the four High findings, which can make honest catches invalid or make
battle replays diverge, followed by the PvP protocol findings (REVIEW-056 to
REVIEW-058).

## High severity

### REVIEW-001: Catch records contain a circular CID reference

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 1

**Finding [proposed]:** A `catch` event contains an `instance`, whose `originProof` must be the CID of that same catch event. Including that CID changes the event's content and therefore its CID.

**Verification (2026-10-27) [proposed]:** Confirmed. `originProof` is defined as the CID of the catch entry that contains the instance (restated in [Peerling species](peerlings/peerling-species.md#peerling-instance)). Starter and created instances are unaffected because their origin attestation is a separate envelope.

**Resolution needed [proposed]:** Define an event-specific instance format that omits this field, or a separate origin-proof record.

**References:** [Data formats: Peerling instance](tech/data-formats.md#peerling-instance); [Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): the instance inside a `catch` event has no `originProof`; its proof is the CID of that entry, filled in wherever the instance is copied ([Data formats: Save-log events](tech/data-formats.md#save-log-events)).

### REVIEW-003: Crashes during battles and between save events have no recovery rule

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 3

**Finding [proposed]:** Encounters are recorded at battle end, using separate `battle-result`, `catch`, and `seen` events. A crash between them can consume an encounter number without saving the caught Peerling. Reloading during a battle can also change the tile or epoch before that encounter is recorded.

**Verification (2026-10-27) [proposed]:** Confirmed. The three entries are written separately at battle end and position is saved only every 30 s; the verifier rules do not cover a `caught` battle result without a `catch`. The accepted "small choice among recent epochs" risk covers stalling before an encounter, not reloading mid-battle. A crash after `catch` but before `seen` is harmless, because a catch counts as seen.

**Resolution needed [proposed]:** Specify durable pending-battle state and how partially written results are completed.

**References:** [Encounters: Selection](gameplay/encounters.md#selection); [Player data: Catches](tech/player-data.md#catches).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): a new `encounter` event at battle start uses up the number and fixes tile and epoch; `catch` is written before `battle-result`; an interrupted battle counts as fled, and a missing `battle-result` after a `catch` is rebuilt by replay (SAVE-028, SAVE-029, ENC-010; [Player data: Encounter seeds](tech/player-data.md#encounter-seeds)).

### REVIEW-004: Backups do not clearly preserve everything needed for verification

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 4

**Finding [proposed]:** The backup CAR contains the private key and snapshot, but does not explicitly require the catch history, origin attestations, transfer chains, checkpoint, or signed snapshot anchor. A snapshot could restore visible progress while leaving owned Peerlings unverifiable after all network copies disappear.

**Verification (2026-10-27) [proposed]:** Confirmed. The CAR is required to contain only the snapshot document's blocks; the snapshot is not signed on its own and its instances carry only `originProof` CIDs. This contradicts the promise that restoring a backup works even if every network copy is gone.

**Resolution needed [proposed]:** Define the backup's required block closure and its guarantees after network history is lost.

**References:** [Data formats: Backup file and phone backup payload](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup); [Player data: Keeping saves available](tech/player-data.md#keeping-saves-available).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): the backup holds the whole save log and each owned Peerling's origin proof and ownership chain, with its guarantees after network loss stated (SAVE-030; [Data formats: Backup file](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup); [Player data: Keeping saves available](tech/player-data.md#keeping-saves-available)).

### REVIEW-006: A recovered identity with no remaining save has no defined restart path

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 6

**Finding [proposed]:** The statement "Only progress is lost" does not explain how play resumes if the key survives but every save copy is gone. The player has already consumed their one lifetime starter. Restarting encounter numbering at zero would also reuse instance IDs.

**Verification (2026-10-27) [proposed]:** Confirmed; kept High (rare precondition, but the player cannot play at all). Also: the save-log address is fixed per player ID (SAVE-018), so a fresh log is the same database; if an old copy resurfaces from a peer backup it merges as a fork with duplicate encounter numbers, and every later catch fails.

**Resolution needed [proposed]:** Define the recovery or reset behavior for this case without silently violating starter or instance-ID rules.

**References:** [Player data: Finding the save again](tech/player-data.md#finding-the-save-again); [Onboarding requirements](gameplay/onboarding.md#requirements); [Data formats: Peerling instance](tech/data-formats.md#peerling-instance).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): designer's choice: the client offers to keep looking or to start a new account; the old key isn't used to play again and stays the creator of its species (SAVE-037; [Player data: Finding the save again](tech/player-data.md#finding-the-save-again)).

### REVIEW-007: The one-computer rule has an online startup race

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 7

**Finding [proposed]:** Two computers can start before either receives the other's heartbeat. The spec defines refusal when an existing heartbeat is observed, but no startup listening period, competing-session tie-break, or action when another active session appears later. The acknowledged offline limitation does not cover this online race.

**Verification (2026-10-27) [proposed]:** Confirmed. Heartbeats are sent only while playing, every 15 s, and no listening duration is given, so two clients starting together both observe nothing. No rule covers a heartbeat or presence session from another device that arrives later. A fork fails every later catch.

**Resolution needed [proposed]:** Specify startup coordination and the behavior of both clients when competing online sessions are detected.

**References:** [Player data: Using the same account on several computers](tech/player-data.md#using-the-same-account-on-several-computers); [Protocols: Active session](tech/protocols.md#peerlingsv1sessionplayer-id--active-session).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): `session-probe` with a 3 s wait, `since` in heartbeats, and the later-started session stops when two meet (SAVE-031; [Protocols: Active session](tech/protocols.md#peerlingsv1sessionplayer-id--active-session)).

### REVIEW-008: Catch verification needs explicit input and output checks

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 8

**Finding [proposed]:** Evidence supplies the starting team's species, levels, HP, and traits. The spec does not fully distinguish values trusted from the evidence, values checked only for valid ranges, and values checked against origins or ownership. The resulting caught instance also needs explicit comparison with the replay. Implementers can otherwise produce materially different definitions of "verified."

**Verification (2026-10-27) [proposed]:** Partly confirmed. The verifier steps never check the evidence team against instances the catcher owns or their origins, and never compare the resulting instance's `hp`, `origin` and `originalOwner` with the replay. Traits and shimmer are checked through the replay, though not as a listed step.

**Resolution needed [proposed]:** Define checks for starting-team data and require the resulting instance's species, traits, shimmer, owner, and level to match the replay, consistently with the already accepted unverified-level policy.

**References:** [Player data: Encounter seeds and verifier checks](tech/player-data.md#encounter-seeds); [Data formats: Catch evidence](tech/data-formats.md#catch-evidence).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): explicit checks for the starting team and for every field of the caught instance (SAVE-032; [Player data: Encounter seeds](tech/player-data.md#encounter-seeds)).

### REVIEW-009: Temporary unavailability is not distinguished from invalid evidence

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 9

**Finding [proposed]:** The spec says a catch that fails verification is invalid forever. Missing blocks, incomplete log synchronization, unsupported rules versions, and an actual invalid replay need different outcomes.

**Verification (2026-10-27) [proposed]:** Confirmed. The UI already treats a 15 s timeout as retryable, but the verification rules have no pending or unavailable state. A `battle-result` missing through partial sync looks like a gap that fails every later catch; BTL-011 keeps only past rules versions, so a newer version is unhandled; the results cache has no invalidation rule.

**Resolution needed [proposed]:** Define at least pending or unavailable versus conclusively invalid, including cache invalidation after new evidence arrives.

**References:** [Player data: Catches](tech/player-data.md#catches); [Verified Peerlings](tech/player-data.md#verified-peerlings).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): verified / invalid / pending results; invalid only on complete data, pending retried, caching rules defined (SAVE-033; [Player data: Verified Peerlings](tech/player-data.md#verified-peerlings)).

### REVIEW-011: Day and week record replacement can change the validity of past play

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 11

**Finding [proposed]:** A client may use a derived day or week record, but a signed record "always wins" when discovered later. Different registry heights can change encounter candidates or guardian teams. Launch-day behavior is also undefined when no signed base record exists from before that day or week began.

**Verification (2026-10-27) [proposed]:** Confirmed. Only `registryHeight` differs between a derived and a signed record (the drand randomness is the same). It is unclear whether verifiers that know the signed record reject evidence that used a derived one, and the launch day or week has no base record.

**Resolution needed [proposed]:** Specify whether completed catches and badges retain their original interpretation, and define initialization when the required older base record does not exist.

**References:** [Player data: Encounter seeds](tech/player-data.md#encounter-seeds); [Peerling of the Day: Choosing it](gameplay/peerling-of-the-day.md#choosing-it); [Guardians: Weekly team](gameplay/guardians.md#the-weekly-team).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): evidence is verified with the records it carries, even if a signed record turns up later; days and weeks before launch use the genesis record (SAVE-034; [Player data: Encounter seeds](tech/player-data.md#encounter-seeds)).

### REVIEW-012: Trade atomicity conflicts with per-instance conflict resolution

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 12

**Finding [proposed]:** A trade is all-or-nothing, but double trades are resolved separately by each transfer envelope's CID. If one transfer in a multi-Peerling trade loses a conflict, the spec does not say whether the entire trade is void, what happens to its other transfers, or how subsequent trades are rolled back.

**Verification (2026-10-27) [proposed]:** Confirmed. Also: "valid only if the entry contains the complete set" can be read as voiding the whole entry once one transfer is void, so implementers can diverge on ownership.

**Resolution needed [proposed]:** Define transaction-level conflict resolution and the effects on every transfer and descendant trade.

**References:** [Data formats: Transfer and transfer-log entry](tech/data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer); [Player data: Transfer log and trades](tech/player-data.md#transfer-log-and-trades).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): a trade entry with an invalid or losing transfer is void as a whole, entries built on it fail in turn, and players write `trade-void` (SAVE-035; [Data formats: Transfer](tech/data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)).

### REVIEW-013: Partially exchanged trade signatures need a recovery state

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 13

**Finding [proposed]:** One player can receive both sides' signatures while the other receives only one side before disconnecting. "Cannot cancel once both sides exchanged signatures" is not a state both clients can reliably observe.

**Verification (2026-10-27) [proposed]:** Confirmed, with a further consequence: a player holding only their own half may assume the trade failed and trade the same Peerling again; when the other player later appends the first entry, the honest player has signed two transfers with the same `prev` and is flagged for a double trade. Nothing locks offered instances and signed offers never expire.

**Resolution needed [proposed]:** Define when offered instances become locked, whether signed offers expire, and how clients recover or abandon pending trades.

**References:** [Protocols: Trade](tech/protocols.md#peerlingstrade100--trade); [Trading: Flow](gameplay/trading.md#flow).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): designer's choice: offered Peerlings are locked until the entry appears, and either player can ask the server to expire the trade; a later entry is then void and nobody is flagged (SAVE-036, TRD-006, API-009; [Player data: Transfer log and trades](tech/player-data.md#transfer-log-and-trades)).

### REVIEW-016: PvP timers do not define a complete decision state machine

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 16

**Finding [proposed]:** Timers start after the preceding turn's state exchange, leaving turn one undefined. Replacement decisions happen before that exchange, so their timer start is also unclear. There is no explicit reveal deadline after a valid commitment, and action and replacement decisions share a turn number without a phase identifier.

**Verification (2026-10-27) [proposed]:** Confirmed. Also: with no reveal deadline, and the stream timeout suspended during PvP turns, a player who has committed can stall the battle indefinitely.

**Resolution needed [proposed]:** Define decision phases, deadlines, and late-message handling for initial actions, replacements, commitments, and reveals.

**References:** [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle); [Battle: PvP turn timer](gameplay/battle.md#pvp-turn-timer).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): decisions carry a `phase`; commit, reveal and state deadlines are defined for turn 1, replacements and reveals (PRT-004; [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle)).

### REVIEW-017: PvP reconnect lacks sufficient reconciliation rules

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 17

**Finding [proposed]:** The specified `resume` message contains only battle ID and turn. There is no reconciliation protocol for pending commitments, revealed actions, or replacements. Nor is there an objective way to identify "the player who dropped" during a network partition.

**Verification (2026-10-27) [proposed]:** Partly confirmed; kept High. The RNG position and timeout streak can be recomputed by replaying the agreed actions (timeouts are recorded as actions), so they were removed from the finding. The other gaps hold and the two clients can reach different outcomes live; the rule voiding conflicting forfeit claims keeps stored records consistent.

**Resolution needed [proposed]:** Define the state exchanged during resume, disagreement handling, and the outcome of ambiguous disconnects.

**References:** [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle); [Battle: PvP turn timer](gameplay/battle.md#pvp-turn-timer).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): either side may reconnect; `resume` says which turn's state it has and each side resends what followed; without a reconnect, competing forfeit claims cancel (PRT-004; [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle)).

### REVIEW-021: Battle edge cases can produce divergent replays

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 21

**Finding [proposed]:** The rules do not fully settle whether `quick-flurry` draws or applies its second hit after the first hit faints the target; what the strike turn of a charge move targets when the original target has fainted or been replaced; or whether a knockout on turn 200 takes precedence over the draw. The integer damage algorithm also states its same-type multiplier without an explicit condition.

**Verification (2026-10-27) [proposed]:** Partly confirmed; kept High. Quick-flurry after a faint, the turn-200 knockout versus draw, and the unconditional same-type step hold. A Speed change cannot reorder an already ordered one-against-one turn, so that claim was removed; the charge-turn claim was narrowed to a target that faints or is replaced before the strike.

**Resolution needed [proposed]:** Specify these edge cases and explicitly make the same-type multiplier conditional so independent battle engines consume identical RNG values and produce identical results.

**References:** [Battle: Move mechanics](gameplay/battle.md#move-mechanics); [Deterministic arithmetic](gameplay/battle.md#deterministic-arithmetic); [Ending a battle](gameplay/battle.md#ending-a-battle).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): quick-flurry's second hit, the charge strike's target and rolls, fixed move order, a turn-200 knockout and the conditional same-type step are defined (BTL-015; [Battle: Move mechanics](gameplay/battle.md#move-mechanics)).

### REVIEW-026: Shrine offerings can violate the team invariant

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 26

**Finding [proposed]:** Swaps and trades are blocked when they leave no healthy team member, but shrine releases are not covered explicitly. The same invariant needs handling after a previously accepted ownership transfer becomes void.

**Verification (2026-10-27) [proposed]:** Confirmed. A soft lock is possible: a player whose only Peerlings are the three offered ones keeps only a credit if the job fails, and a starter is granted once ever (ONB-008).

**Resolution needed [proposed]:** Define whether team members may be offered, how the remaining team is chosen, and how ownership reversals preserve or restore a usable team.

**References:** [Catching: Team and collection](gameplay/catching.md#team-and-collection); [Creation Shrine: Rules](gameplay/creation-shrine.md#rules).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): shrine offerings follow the trade rule, and a team that loses members to a void trade is refilled from the collection (CAT-010; [Catching: Team and collection](gameplay/catching.md#team-and-collection)).

### REVIEW-028: Historical registry eligibility needs an explicit algorithm

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 28

**Finding [proposed]:** Readers select the highest-`seq` envelope, but a tombstone replaces the original `seq` and preserves it as `listedSeq`. Historical eligibility still uses the expression `seq ≤ registryHeight`. Following that literally against the current document view can exclude a species that was eligible before delisting.

**Verification (2026-10-27) [proposed]:** Partly confirmed; kept High. The registry index keeps the listing and the delisting as separate entries and carries `listedSeq`, so the correct algorithm can be worked out. But the stated rule, applied literally to the documents view, yields different encounter candidates, and how clients check the index against the registry is unspecified.

**Resolution needed [proposed]:** Specify historical reconstruction using original listings or `listedSeq`.

**References:** [Data formats: Registry entry](tech/data-formats.md#registry-entry--peerlingslisting); [Player data: Encounter seeds and registry state](tech/player-data.md#encounter-seeds).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): eligibility at height H uses each species' listing and delisting `seq`, draws sort by listing seq, and the registry index is authoritative (SAVE-038; [Player data: Encounter seeds](tech/player-data.md#encounter-seeds)).

### REVIEW-037: Exactly-one-starter behavior is incomplete under concurrency and lost responses

- **Status:** Fixed
- **Severity:** High
- **Original review item:** 37

**Finding [proposed]:** Multiple creation jobs can start before a player has a starter, and choosing an existing starter can race with publishing one. If `/starters/choose` succeeds but its response is lost, a retry returns `starter-taken`; there is no specified endpoint to recover the issued attestation.

**Verification (2026-10-27) [proposed]:** Confirmed. Also unspecified: a second concurrent starter job can publish a species whose instance must then be refused. A lost response permanently blocks onboarding (ONB-005).

**Resolution needed [proposed]:** Define atomic issuance and idempotent result retrieval.

**References:** [Creation API: Starters](tech/creation-api.md#starters); [Creating a Peerling](tech/creation-api.md#creating-a-peerling); [Onboarding requirements](gameplay/onboarding.md#requirements).

**Resolution:** Fixed 2026-10-27 in [D-0027](decisions/D-0027-high-severity-review-fixes.md): one open starter job per player, atomic issuance at the attestation, choosing abandons an open starter job and is idempotent, and `GET /v1/starter` recovers the state (API-007, API-008; [Creation API: Starters](tech/creation-api.md#starters)).

## Medium severity

### REVIEW-002: Save reconstruction is not fully specified

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 2

**Finding [proposed]:** The save is supposedly reconstructed by applying events, but there is no healing/rest-point event. Rest visits change HP and `lastRestPoint` through a snapshot instead. Verifier checks do not use HP or the rest point, but the completeness of a verification checkpoint's state depends on the answer.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. "Apply all events" and "latest snapshot plus later events" are never reconciled, and no heal or rest-point event exists. But EXP-008 writes a snapshot on every rest visit, so the normal load path works, and the verifier checks use only event-derived data (Peerdex, encounter numbers, epochs). Wording on the verification impact corrected.

**Resolution needed [proposed]:** Define whether snapshots are authoritative state changes or merely caches of event-derived state, and specify how healing and rest-point changes are reconstructed.

**References:** [Player data: Save log](tech/player-data.md#save-log); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): a `rest` event; a lost battle heals and returns the player; events define the save and snapshots are caches (SAVE-039; [Player data: Save log](tech/player-data.md#save-log)).

### REVIEW-005: Showing the recovery phrase again is unsupported

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 5

**Finding [proposed]:** The UI offers "show recovery phrase," but the phrase is shown once at onboarding and no storage of the mnemonic or its entropy is specified. Backups store only the derived private key, from which the BIP-39 phrase cannot be reconstructed. The action is therefore unsupported on every device, not only after file or phone restoration.

**Verification (2026-10-27) [proposed]:** Confirmed and broader than originally stated: the phrase is shown once at onboarding and nothing secret is stored, so even the original device cannot show it again. Title and wording corrected.

**Resolution needed [proposed]:** Specify whether the client stores the mnemonic or entropy (and whether backups carry it), or whether the action is removed or limited to onboarding.

**References:** [UI: Pause menu](gameplay/ui.md#pause-menu-esc); [Data formats: Recovery phrase](tech/data-formats.md#recovery-phrase); [Backup format](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: the device and every backup keep the phrase's entropy, so it can be shown again (SAVE-040; [Data formats: Recovery phrase](tech/data-formats.md#recovery-phrase)).

### REVIEW-010: Anti-reroll guarantees exceed what the evidence proves

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 10

**Finding [proposed]:** A modified client can append invented non-catch `battle-result` events to advance encounter numbers quickly; those events contain no replay evidence. This allows searching for desirable traits or shimmers without the claimed five-minute reroll cost. Likewise, client-derived-record validity does not prove that the server was offline. These are broader gaps than the currently described risks.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from High. `battle-result` carries no replay evidence, and fleeing always succeeds, so even honest play advances encounter numbers cheaply. The known-gap text "only while the server is offline" is inaccurate, because the client-derived validity rule does not check that. Verifiers still agree on results, so this weakens an anti-cheat promise rather than making clients disagree.

**Resolution needed [proposed]:** Explicitly accept these broader limits or define stronger verification rules; align the anti-reroll explanation and known-risk descriptions with the chosen behavior.

**References:** [Player data: Encounter seeds](tech/player-data.md#encounter-seeds); [Known gaps](tech/player-data.md#known-gaps-accepted-risks); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: accepted as a known gap (fast fled encounters), with the anti-reroll text and the base-record gap corrected ([Player data: Known gaps](tech/player-data.md#known-gaps-accepted-risks)).

### REVIEW-014: Equivalent previous-transfer entries can produce different next-transfer signatures

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 14

**Finding [proposed]:** Identical transfers may appear in multiple OrbitDB entries, but the next transfer's `prev` points to an entry CID. Choosing different equivalent entries changes the signed envelope, so any future retry or re-signing that picks a different equivalent entry would look like a double trade. No currently specified flow re-signs a transfer.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. `prev` is not canonical when the same envelope sits in several entries, but every trade uses a fresh random trade ID and nothing re-signs after step 5, so no specified flow produces an identical retry. Verifiers resolve `prev` to the envelope, so verification does not diverge. Wording corrected.

**Resolution needed [proposed]:** Specify a canonical `prev` or an equivalence rule for these retries.

**References:** [Data formats: Transfer and transfer-log entry](tech/data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): transfers that differ only in an equivalent `prev` are one transfer; signers name the lowest-CID entry (SAVE-041; [Data formats: Transfer](tech/data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)).

### REVIEW-015: Trade confirmation binds IDs rather than the offered instance state

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 15

**Finding [proposed]:** The offers hash covers only instance IDs, while the receiver obtains level, XP, HP, and nickname from the offered data. The transfer envelopes carry none of that data. The receiver's `trade` save-log event stores it, but the received offer data is not required to be persisted before that event is written, so recovery at session start or on another computer has no source for it.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. The receiver's `trade` event stores the full received instances, so the state is permanent once written; the gap is the window before that event. Unverified levels are already an accepted risk. Wording corrected.

**Resolution needed [proposed]:** Define what exact state confirmation commits to and how that state remains recoverable.

**References:** [Protocols: Trade and offers hash](tech/protocols.md#peerlingstrade100--trade); [Trading: Flow](gameplay/trading.md#flow).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): confirmation commits to IDs only; the receiver stores the offer data before signing and can rebuild it from the sender's save log (TRD-007; [Protocols: Trade](tech/protocols.md#peerlingstrade100--trade)).

### REVIEW-018: PvP result proofs leave important cases unresolved

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 18

**Finding [proposed]:** Forfeits need the loser's signed state, but none exists before the first completed turn. State signatures do not bind the level mode, so a winner-signed forfeit can label a loser's state with a different mode. Conflict cancellation also requires discovering claims in both players' logs, while profile counting describes fetching the inspected player's log.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from High, because the win counter is a stat only with no reward ([D-0021](decisions/D-0021-pvp-win-counter.md)). Wording corrected: D-0024 accepted a forged-forfeit mitigation, not a listed accepted risk.

**Resolution needed [proposed]:** Define these cases consistently with the forged-forfeit mitigation accepted in [D-0024](decisions/D-0024-review-2-decisions.md), including how competing claims are discovered and how mode and initial state are bound to the result.

**References:** [Protocols: PvP signatures](tech/protocols.md#peerlingsbattle100--pvp-battle); [Data formats: Save-log events](tech/data-formats.md#save-log-events); [PvP: Win record](gameplay/pvp-battles.md#win-record).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): a signed turn-0 state, the mode in every state signature, and profiles check the opponent's log before counting a forfeit (PVP-013; [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle)).

### REVIEW-019: Late spectators cannot obtain the initial battle setup

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 19

**Finding [proposed]:** Turn messages contain cumulative actions and the seed, but teams, mode, and rules version appear only in the initial `start` message. A spectator joining later has no specified request or replay mechanism for retrieving that message.

**Verification (2026-10-27) [proposed]:** Confirmed. It also contradicts the spectating page's statement that a late spectator catches up from any single message. Authentication is straightforward once the message is retrieved (strict signing plus state hashes).

**Resolution needed [proposed]:** Define how late spectators retrieve and authenticate the initial setup.

**References:** [Protocols: Spectating](tech/protocols.md#peerlingsv1battlebattle-id--spectating); [Spectating design](gameplay/spectating.md#design).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): turn messages name the `start` message and the battle history by CID; late spectators fetch both blocks (SPT-005; [Protocols: Spectating](tech/protocols.md#peerlingsv1battlebattle-id--spectating)).

### REVIEW-020: Spectator messages exceed their mandatory size limit

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 20

**Finding [proposed]:** Every turn message must contain all previous actions, and battles may last 200 turns. The review calculated 18,065 bytes for a 200-turn message containing only quick moves, before pubsub overhead; the battle-topic limit is 16,384 bytes (16 KiB). Switch-heavy battles exceed the limit much earlier (from turn 112 if every action is a switch).

**Verification (2026-10-27) [proposed]:** Confirmed: 18,065 bytes is exact for the DAG-CBOR encoding, and the message first exceeds the limit at turn 182. Understated: a switch action encodes to 50 bytes, so an all-switch battle exceeds 16 KiB from turn 112, and long switch-heavy battles are the case the 200-turn draw exists for. The spectating page's "this stays small" is contradicted.

**Resolution needed [proposed]:** Specify chunking, history retrieval, or a different message format that respects the limit throughout a valid battle.

**References:** [Spectating requirements](gameplay/spectating.md#requirements); [Data formats: Size limits](tech/data-formats.md#size-limits); [Protocols: Spectating](tech/protocols.md#peerlingsv1battlebattle-id--spectating).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): turn messages carry only that turn's actions; the full history is a separate block, so messages stay small at 200 turns (SPT-005; [Spectating: Design](gameplay/spectating.md#design)).

### REVIEW-022: Peerling of the Day can become less common instead of more common

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 22

**Finding [proposed]:** Its replacement weight is `max(1, W div 19)`. With three unseen Normal species in Plains, each ordinarily has weight 12. Featuring one changes its first-draw chance from 33.3% to 4%, contrary to the stated increase.

**Verification (2026-10-27) [proposed]:** Confirmed; arithmetic checked. The featured species replaces its own weight: W = 24, its weight is max(1, 24 div 19) = 1, giving 1/25 = 4% instead of 12/36 = 33.3%. This contradicts POD-001 ("MUST appear more often"). The featured species is penalised whenever W div 19 is below its natural weight (W < 228 for an unseen native species), which is exactly the small cold-start registry.

**Resolution needed [proposed]:** Decide whether the goal is an approximately 5% share or a guaranteed increase, and align the formula with that goal.

**References:** [Peerling of the Day: In encounters](gameplay/peerling-of-the-day.md#in-encounters); [Encounters: Selection](gameplay/encounters.md#selection).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: weight = max(2 × normal weight, W div 19) ([Peerling of the Day: In encounters](gameplay/peerling-of-the-day.md#in-encounters)).

### REVIEW-025: Seen is not fully defined outside wild encounters

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 25

**Finding [proposed]:** Clarify whether PvP opponents, guardians, followers, gallery models, and opened species cards count as sightings. Starters, creations, and trades explicitly count, but their events do not consistently supply the mandatory first-met biome stored in the Peerdex.

**Verification (2026-10-27) [proposed]:** Confirmed. A guardian `battle-result` has no `species` field and the guardians page never mentions `seen`; `starter`, `created` and `trade` events carry no biome, while the snapshot's Peerdex entry requires one.

**Resolution needed [proposed]:** Define sighting sources and the first-biome value for every non-wild acquisition or sighting.

**References:** [Peerdex: What it lists](gameplay/peerdex.md#what-it-lists); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave); [Encounters: Selection](gameplay/encounters.md#selection).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: wild, guardian and PvP opponents' Peerlings and every received Peerling write `seen`, with the biome of the player's tile (ENC-011; [Encounters: Selection](gameplay/encounters.md#selection)).

### REVIEW-027: Delisting has inconsistent scope

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 27

**Finding [proposed]:** Some text says clients stop showing a delisted species; other rules retain owned assets permanently and keep delisted guardian members for the week. `SRV-002` also requires pinning every asset, whereas `REG-010` permits unpinning delisted assets.

**Verification (2026-10-27) [proposed]:** Confirmed. SRV-002 ("MUST pin every game asset") directly contradicts REG-010 ("MAY unpin its assets").

**Resolution needed [proposed]:** Specify treatment in collections, PvP, trades, followers, shared cards, and cached encounters, and reconcile the server-pinning requirements.

**References:** [D-0010: Decision](decisions/D-0010-no-content-moderation.md#decision); [Registry: Design](tech/orbitdb-registry.md#design); [Generation server requirements](tech/generation-server.md#requirements); [Guardians: Weekly team](gameplay/guardians.md#the-weekly-team).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: owned Peerlings become a silhouette named "Delisted Peerling", still battle, can't be traded, followed or shared; everyone deletes the assets; SRV-002, REG-010 and NODE-004 replaced by SRV-008, REG-011 and NODE-009 ([Registry: Design](tech/orbitdb-registry.md#design)).

### REVIEW-029: World updates need rules for ongoing battles and saved map state

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 29

**Finding [proposed]:** Generator changes can move guardian statues (and with them their level) and relocate rest points. Guardian evidence carries both an epoch record and a week record, and the spec does not say which record's generator version validates the statue tile. Relocation goes to the "nearest" walkable tile or rest point without saying nearest to what. The snapshot has no explicit generator-version field, although it is derivable from each encounter's epoch record.

**Verification (2026-10-27) [proposed]:** Partly confirmed. Players on newly blocked tiles and missing rest points are already relocated; the generator version is derivable from each encounter's epoch record; the guardian level changes only if the statue tile moves; explored chunks are a positional bitset. The remaining gap is which record's generator version validates guardian evidence. Wording corrected.

**Resolution needed [proposed]:** Specify which version determines a guardian battle spanning an update and how map and rest-point state migrate.

**References:** [Procedural generation: Generator updates](world/procedural-generation.md#generator-updates); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave); [Guardians: Weekly team](gameplay/guardians.md#the-weekly-team).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): a guardian battle uses its epoch record's generator version; "nearest" is defined; snapshots name their `generator` (WGN-022; [Procedural generation: Generator updates](world/procedural-generation.md#generator-updates)).

### REVIEW-031: Implementation-critical proposals remain unapproved inside accepted pages

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 31

**Finding [proposed]:** Examples include `SRV-007`, API refusal and outage handling, and provenance fields. The image-prompting page correctly remains draft, but the open-question list tracks only licences and the default 3D generator, not approval of the broader proposed pipeline behavior.

**Verification (2026-10-27) [proposed]:** Confirmed. creation-api, creation-pipeline, generation-server and data-formats have status `accepted` but contain `[proposed]` items (for example SRV-007, `name-reserved`, outage and `wish-refused` handling, provenance `imagePrompt` fields). Only Q-057 and Q-058 are open.

**Resolution needed [proposed]:** Make the decision boundary explicit for implementers and track approval or rejection of the remaining proposals through the existing question and decision workflow.

**References:** [Generation server requirements](tech/generation-server.md#requirements); [Creation pipeline: Job handling](peerlings/creation-pipeline.md#job-handling); [Open questions](open-questions.md#open).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: the listed proposals in accepted pages are approved; the remaining image-pipeline proposals are tracked as [Q-059](open-questions.md#q-059).

### REVIEW-033: The originality guarantee conflicts with proposed fail-open behavior

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 33

**Finding [proposed]:** `IMG-003` says the pipeline MUST stop character cloning, but after one retry the second image is shown regardless of the resemblance result.

**Verification (2026-10-27) [proposed]:** Confirmed. IMG-003 is `[accepted]`; the fail-open flow and IMG-010 (a SHOULD) are `[proposed]`. The second check's result is not kept in provenance. The other originality layers still apply.

**Resolution needed [proposed]:** Decide whether originality checking is best-effort or whether known failed images must be blocked from publication.

**References:** [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules); [Requirements](peerlings/image-prompting.md#requirements).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: a hero image flagged again after the automatic retry can't be accepted (IMG-018; [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules)).

### REVIEW-035: Franchise matching uses incompatible normalization and matching rules

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 35

**Finding [proposed]:** Franchise matching applies species-name normalization, which removes spaces and punctuation, then requires whole-word or adjacent-word matching. Word boundaries are gone by then.

**Verification (2026-10-27) [proposed]:** Confirmed. Also undefined: the order of the leetspeak mapping relative to NFKC and confusable mapping; what "words" mean for Japanese written without spaces; whether accent removal strips Japanese voicing marks. CRE-025 itself does not mention spaces; only the prose does.

**Resolution needed [proposed]:** Define tokenization order, Unicode normalization details, and what happens when a species name normalizes to an empty string.

**References:** [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules); [Creation pipeline: Final review and naming](peerlings/creation-pipeline.md#final-review).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): exact normalization order, word splitting before normalization, scripts without spaces, and empty names refused (CRE-026; [Creation pipeline: Final review](peerlings/creation-pipeline.md#final-review); [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules)).

### REVIEW-036: Creation jobs lack complete retry and recovery operations

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 36

**Finding [proposed]:** `FAILED` is retryable, but no retry transition or endpoint exists; the only retry is starting a new job, which loses the wish and image. Recovering a starter job after losing its locally stored job ID is also undefined. Automatic polling continually extends expiry, while a long-lived SSE connection may not.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. There is no exit from `FAILED`, no retry endpoint and no starter-job discovery, although ONB-004 requires resuming the same job. But starting a new job is a working (lossy) retry and a failed shrine job grants a credit (SHR-005), so no player is stuck. Losing a starter after its `origin` was issued belongs to REVIEW-037. Wording corrected.

**Resolution needed [proposed]:** Define retry transitions, job discovery after local data loss, and a deliberate distinction between player activity and request activity for expiry.

**References:** [Creation pipeline: Job handling](peerlings/creation-pipeline.md#job-handling); [Creation API: Job object](tech/creation-api.md#the-job-object); [Creating a Peerling](tech/creation-api.md#creating-a-peerling).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): `POST /v1/jobs/{id}/retry`, an open event stream counts as activity, and `GET /v1/starter` finds a lost starter job (API-010; [Creation API: The job object](tech/creation-api.md#the-job-object)).

### REVIEW-038: The operator's seed-species workflow is absent from the API

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 38

**Finding [proposed]:** Seed species must use the normal pipeline and receive no instance, but job kinds are only `starter` and `shrine`, with their associated restrictions.

**Verification (2026-10-27) [proposed]:** Confirmed; kept Medium. Also unspecified: which player key and `creator.id` a seed species uses. That value ends up in signed public records, and seed species are needed at launch.

**Resolution needed [proposed]:** Specify the operator-only creation path or an explicit administrative exception.

**References:** [Creation pipeline: Who uses the pipeline](peerlings/creation-pipeline.md#who-uses-the-pipeline); [Stage 8](peerlings/creation-pipeline.md#stage-8--the-new-peerling-joins-the-player); [Creation API](tech/creation-api.md#creating-a-peerling).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): a `seed` job kind for operator accounts, with no instance and the operator account as creator (API-011; [Creation API: Creating a Peerling](tech/creation-api.md#creating-a-peerling)).

### REVIEW-039: Shrine credits lack lifecycle details

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 39

**Finding [proposed]:** A replacement job still needs the original offering's average level, but the credit description exposes only expiry.

**Verification (2026-10-27) [proposed]:** Partly confirmed. When a credit is consumed is implied (a credit-funded job omits `release`; SHR-006 allows one open shrine job or credit) but not stated. The reward-level and renewal gaps hold.

**Resolution needed [proposed]:** Define retention of offering provenance and reward level, when a credit is consumed, and whether repeated failure or abandonment renews its 30-day lifetime.

**References:** [Creation Shrine: Rules](gameplay/creation-shrine.md#rules); [Creation API: Shrine status](tech/creation-api.md#creating-a-peerling).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): a credit holds the reward level, the release entry and its expiry; it is used up when a job starts and returned with its original expiry ([Creation Shrine: Rules](gameplay/creation-shrine.md#rules)).

### REVIEW-040: Several exact schemas remain descriptive rather than exact

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 40

**Finding [proposed]:** `concept` is an unspecified map; `appearance` is described as structured data but shown as a string; duplicate species types are not prohibited; and character-count units are only explicit for species names. Provenance requires models for every stage but has no distinct stats-and-moves model field (`models.concept` arguably covers it, because the concept LLM also writes stats and moves). Only the character-count units can make clients disagree on whether a signed record is valid.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. Concept fields are listed in the pipeline but without types, lengths or extra-field rules; `appearance` is a string in every example; duplicate types are not forbidden (and would multiply to 4× under TYP-003); the stats-and-moves model is ambiguous rather than missing. The server signs these records and clients fetch exact bytes, so only the character-count point can make clients disagree. Wording corrected.

**Resolution needed [proposed]:** Provide concrete validation rules and reconcile the schemas and examples, including provenance for every required stage.

**References:** [Creation pipeline: Concept](peerlings/creation-pipeline.md#stage-2--concept); [Data formats: Species record](tech/data-formats.md#species-record--peerlingsspecies); [Image prompting: Input object](peerlings/image-prompting.md#input-object).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): exact concept schema, distinct types, code-point lengths everywhere, `models.profile` in provenance (CRE-027, FMT-005; [Creation pipeline: Stage 2](peerlings/creation-pipeline.md#stage-2--concept)).

### REVIEW-041: Network fallback policy leaves UDP-blocked environments unresolved

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 41

**Finding [proposed]:** The spec identifies blocked UDP as a WebTransport failure case, while its WebRTC-direct fallback is also described as UDP-based. HTTP/3 is mandatory for the API, with no stated HTTP fallback.

**Verification (2026-10-27) [proposed]:** Partly confirmed. Not mentioned in the finding: blocks can still come from trustless HTTP gateways (NODE-006), and D-0011 accepts that the game may not run everywhere, without naming UDP-blocked networks. Browsers also fall back from HTTP/3 to HTTP/2 on their own.

**Resolution needed [proposed]:** Clarify whether these environments are unsupported or which permitted path serves them.

**References:** [Tech stack: Networking](tech/tech-stack.md#networking); [Network performance: What can be slow](tech/network-performance.md#what-can-be-slow); [Creation API: General rules](tech/creation-api.md#general-rules).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): designer's choice: unsupported in v1 with a clear message; the API is also served over HTTP/2 (STK-016, API-012; [Tech stack: Networking](tech/tech-stack.md#networking)).

### REVIEW-042: Unlimited ownership and permanent retention need a storage-exhaustion policy

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 42

**Finding [proposed]:** Collections are unlimited, owned and created species must never be evicted, and the cache is described as capped at 1 GB. Retention is per species, so growth is bounded by the number of distinct species owned.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. Retention is per species, not per Peerling (about 1,500 typical species fill 1 GB). Whether exempt content counts toward the cap and what happens on a quota error remain unspecified. Wording corrected.

**Resolution needed [proposed]:** Clarify whether pinned ownership content sits outside that cap and what happens when browser quota is exhausted during a catch, trade, or creation.

**References:** [Tech stack: Asset budgets](tech/tech-stack.md#asset-budgets); [IPFS node: Retention requirements](tech/ipfs-helia.md#requirements); [Catching: Team and collection](gameplay/catching.md#team-and-collection).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): retained content, the save log and peer backups sit outside the 1 GB cap; a defined order for a full quota (STK-015; [Tech stack: Asset budgets](tech/tech-stack.md#asset-budgets)).

### REVIEW-043: Profile links can expire despite continuous server republication

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 43

**Finding [proposed]:** IPNS records have 30-day validity, while the server republishes an existing player-signed record without holding the player's key. Republishing does not extend its signed expiry, yet the sharing page promises that "every player has a permanent link".

**Verification (2026-10-27) [proposed]:** Confirmed. The sharing page's summary promises a permanent link, and its player-profile section implies that resolution ends only if both the server and the player stop. Viewer behaviour after expiry is unspecified. Wording and references corrected.

**Resolution needed [proposed]:** Specify what users see after an absent player's record expires, and qualify the promise of permanent profile links.

**References:** [Sharing: summary](gameplay/sharing.md); [Data formats: Player profile document](tech/data-formats.md#player-profile-document--peerlingsprofile); [Sharing: Player profile](gameplay/sharing.md#player-profile).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): the link never changes; after expiry the Viewer shows the last signed profile with its date (LNK-004; [Sharing: Player profile](gameplay/sharing.md#player-profile)).

### REVIEW-045: Creator statistics claim stronger verification than source events support

- **Status:** Fixed
- **Severity:** Medium
- **Original review item:** 45

**Finding [proposed]:** Non-catch encounters lack the evidence needed to replay them, yet encounter counts are described as resistant to fake clients. The counting unit for a trade containing multiple instances of one species and the effect of later transfer conflicts are also unspecified.

**Verification (2026-10-27) [proposed]:** Confirmed. `battle-result` has no tile, so the species of a non-catch encounter cannot be checked against the candidate list; only the gap-free and epoch checks apply.

**Resolution needed [proposed]:** Define how non-catch encounters are validated, whether trades increment once per transaction or per instance, and when conflicting transfers reverse counters.

**References:** [Creator feedback: Feasibility](gameplay/creator-feedback.md#is-it-possible-with-ipfs); [Species stats snapshot](gameplay/creator-feedback.md#species-stats-snapshot); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): encounters are checked from `encounter` events, trades count once per trade entry, void trades drop out of the hourly counts (CFB-009; [Creator feedback: Species stats snapshot](gameplay/creator-feedback.md#species-stats-snapshot)).

## Low severity

### REVIEW-023: The guaranteed first wild encounter conflicts with guardians consuming encounter zero

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 23

**Finding [proposed]:** A player can reach a guardian before entering foliage, and guardian battles consume the next encounter number. Under EXP-011 the first-encounter guarantee then silently lapses, so the tutorial's promise of an ordinary first wild encounter is broken.

**Verification (2026-10-27) [proposed]:** Partly confirmed. The ordering is possible (the Plains guardian is about 350 m from the centre, beyond the 300 m spawn hexagon, and paths mostly avoid foliage), but EXP-011 already decides the outcome: the guarantee lasts until encounter 0 is logged, so it silently lapses. Wording corrected.

**Resolution needed [proposed]:** Decide whether the guarantee should survive a guardian battle, or prevent that ordering.

**References:** [Exploration: Wild encounters in foliage](gameplay/exploration.md#wild-encounters-in-foliage); [Guardians: Badges and logging](gameplay/guardians.md#badges).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): a guardian battle doesn't end the first-encounter guarantee; EXP-011 replaced by EXP-014 ([Exploration: Wild encounters in foliage](gameplay/exploration.md#wild-encounters-in-foliage)).

### REVIEW-024: Encounter prefetching needs invalidation rules

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 24

**Finding [proposed]:** Three upcoming encounters are prefetched, but each earlier encounter can change the seen set and therefore later candidate weights. No recomputation trigger is stated for that case; epoch, day and biome changes are already covered.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from Medium. Candidates are worked out at trigger time (ENC-005), a new epoch record already triggers recomputation for the current and neighbouring biomes, and assets are content-addressed. Only the seen-set dependency lacks a recompute trigger, and a stale prediction only costs a cache miss. Wording corrected.

**Resolution needed [proposed]:** Specify recomputation of predicted candidates after a first sighting.

**References:** [Encounters: Selection](gameplay/encounters.md#selection); [Latency](gameplay/encounters.md#latency).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): candidate lists are worked out again after every new `seen` event; fetched species are reused by CID ([Encounters: Latency](gameplay/encounters.md#latency)).

### REVIEW-030: Movement and interaction rules have uncovered boundary cases

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 30

**Finding [proposed]:** "Within 3 metres" is defined as a neighbouring tile without saying whether diagonal tiles (2.83 m) count, yet interaction only targets the cardinal tile being faced. Presence validation also rejects implausible jumps, yet legitimate defeat respawns and generator relocations can jump long distances without a corresponding presence-message flag.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from Medium. "Neighbouring tile" never says 4 or 8 neighbours; the diagonal reading was the reviewer's inference. Presence is not authoritative, so the worst case is a player who is briefly invisible to others (removed after 15 s). Wording corrected.

**Resolution needed [proposed]:** Define adjacency precisely and specify how legitimate relocation passes presence validation.

**References:** [Multiplayer: What players experience](gameplay/multiplayer.md#what-players-experience); [Realtime networking requirements](tech/realtime-networking.md#requirements); [Protocols: Presence](tech/protocols.md#peerlingsv1presencerx_ry--presence).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): "within 3 m" means the 4 tiles directly next to the player; presence `jump` for respawns and relocations (MPL-010, NET-005; [Multiplayer](gameplay/multiplayer.md#what-players-experience)).

### REVIEW-032: Enhancer invocation rules conflict

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 32

**Finding [proposed]:** Accepted `IMG-001` requires every new image request to pass through GPT-6 Luna. Proposed regeneration reuses an existing enhanced description, and proposed failure handling bypasses the enhancer altogether.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from Medium. The enhancer-failure fallback is a real bypass and is also written into accepted D-0025; regeneration reusing the enhanced description arguably still goes through the enhancer. Image-check retries are a third reuse case. Every exception is written down, so only IMG-001's wording needs aligning.

**Resolution needed [proposed]:** Define the intended exceptions explicitly.

**References:** [Image prompting: Pipeline placement](peerlings/image-prompting.md#where-it-sits-in-the-pipeline); [Failure handling](peerlings/image-prompting.md#failure-handling); [Requirements](peerlings/image-prompting.md#requirements).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): IMG-001 names its exceptions (reused descriptions for regenerations and check retries, and the D-0025 fallback) ([Image prompting: Requirements](peerlings/image-prompting.md#requirements)).

### REVIEW-034: Image-quality checks contradict their own requirement

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 34

**Finding [proposed]:** Proposed `IMG-013` requires every image to pass the checks before use, but the failure flow shows an image anyway after retries.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from Medium. IMG-013 and the failure flow are both `[proposed]` on a draft page, so a proposal contradicts itself; only a missing alpha channel is repaired, and the player can regenerate.

**Resolution needed [proposed]:** Specify whether the player can approve failed framing, disconnected geometry, or other failed checks, and which failures are hard blockers.

**References:** [Image prompting: Image checks and clean-up](peerlings/image-prompting.md#image-checks-and-clean-up); [Requirements](peerlings/image-prompting.md#requirements).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): only a missing transparency after matting blocks an image; the other failed checks are shown and the player decides (IMG-013, still proposed under Q-059; [Image prompting: Image checks](peerlings/image-prompting.md#image-checks-and-clean-up)).

### REVIEW-044: Phone-backup entry into the Viewer needs clarification

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 44

**Finding [proposed]:** The QR payload is a custom `peerlings-backup:` URI rather than an HTTPS Viewer URL. The pages imply opening the Viewer first and scanning inside it, but a scan with the phone's normal camera launches nothing (a web app can only register `web+` schemes), and the computer is not required to tell the player where the Viewer is.

**Verification (2026-10-27) [proposed]:** Partly confirmed. Read together, the pages imply opening the Viewer first and scanning inside it, which is fairly clear. The real gaps are that a normal camera scan of the custom URI launches nothing and that the player is never shown the Viewer's address. Wording corrected.

**Resolution needed [proposed]:** Specify whether users first open the Viewer and use its scanner, or whether scanning with the phone's normal camera should launch it.

**References:** [Protocols: Phone backup](tech/protocols.md#peerlingsphone-backup100--phone-backup); [Player data: Phone backup](tech/player-data.md#phone-backup).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): the QR code is a Viewer URL with the data in the fragment, so the phone's camera opens the Viewer ([Protocols: Phone backup](tech/protocols.md#peerlingsphone-backup100--phone-backup)).

### REVIEW-046: First-finder finalization needs a collection window

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 46

**Finding [proposed]:** Lowest epoch, then lowest catch CID, and final once announced are compatible only after defining the set of catches considered before announcement. The timing determines whether arrival order or catch epoch actually decides the winner.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from Medium. `firstWild` lives in the hourly stats snapshot and announcements happen once the stats name a finder, so the snapshot is effectively the naming point. Only the set of catches it considers is undefined, and only the server decides.

**Resolution needed [proposed]:** Specify whether selection happens immediately, at an epoch boundary, or during the hourly snapshot.

**References:** [Creator feedback: First found in the wild](gameplay/creator-feedback.md#first-found-in-the-wild).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): each hourly snapshot considers the catches verified by then (CFB-009; [Creator feedback: First found in the wild](gameplay/creator-feedback.md#first-found-in-the-wild)).

### REVIEW-047: Examples and historical summaries contradict current rules

- **Status:** Fixed
- **Severity:** Low
- **Original review item:** 47

**Finding [proposed]:** The species JSON example totals 250 base-stat points, violating `SPC-007`'s 320. D-0006 calls instance data private to the owner, although saves are publicly replicated, and still describes only created starters. D-0025 retains a consequence saying provenance contains three prompts after D-0026 removed those views.

**Verification (2026-10-27) [proposed]:** Confirmed: the example totals 80 + 55 + 70 + 45 = 250. Also stale: D-0006's starter statement has no note about D-0012, and D-0025 keeps a bullet about the three-view camera layout and silhouette check. Decision records changed in part are expected to carry notes (AGENTS.md §7), so these are real defects.

**Resolution needed [proposed]:** Correct the invalid example and clearly mark or update superseded details wherever they remain.

**References:** [Peerling species: Example](peerlings/peerling-species.md#species-record); [D-0006](decisions/D-0006-species-vs-instance.md); [D-0025](decisions/D-0025-prompt-enhancer-and-three-views.md); [D-0026](decisions/D-0026-single-image-to-3d.md).

**Resolution:** Fixed 2026-10-28 in [D-0028](decisions/D-0028-medium-low-review-fixes.md): the species example sums to 320 and names the `profile` model; D-0006 and D-0025 carry notes on what changed ([Peerling species](peerlings/peerling-species.md#species-record)).

## Second review: High severity

### REVIEW-048: The catch check rejects an honest catch made with a fainted team member

- **Status:** Open
- **Severity:** High
- **Review:** 2026-11-10 (regression from the REVIEW-008 fix)

**Finding [proposed]:** Verifier check 4 requires every member of the evidence `team` to have HP between 1 and its max HP. The evidence `team` holds the battle-start state of every team member, and a team may start a wild battle with fainted members (HP carries over; only one non-fainted member is required, CAT-009). Such a catch fails on complete data, so it is invalid for good and can never be traded or used in PvP.

**Checked:** No rule heals or removes fainted members before a battle, and nothing says the evidence team leaves them out. Guardian evidence is verified "the same way", so badges are affected too.

**Resolution needed [proposed]:** Allow HP from 0 to max in check 4 (with at least one member above 0), or define the evidence team as the non-fainted members only.

**References:** [Player data: Catches](tech/player-data.md#catches) (check 4, SAVE-032); [Data formats: Catch evidence](tech/data-formats.md#catch-evidence); [Battle: Formats](gameplay/battle.md#formats); [Catching: Team and collection](gameplay/catching.md#team-and-collection).

**Resolution:** —

### REVIEW-049: Day and week records have no usable base-record rule

- **Status:** Open
- **Severity:** High
- **Review:** 2026-11-10

**Finding [proposed]:** A client-derived record copies its fields from "the latest server-signed epoch record the client has", but verifiers accept only a base record with an epoch of at most E − 2. A missing day or week record (epoch 288 × D or 2016 × W) is derived "at once", usually by a client whose latest signed record is newer than that epoch. Followed literally, this gives a day or week record that fails the validity rule, so every catch of that day and every badge of that week is invalid. If the implementer picks an older record instead, honest clients can pick different ones and disagree on the Peerling of the Day or the guardian team. SAVE-013 adds a third wording ("the latest server-signed epoch record before that epoch", which can be E − 1). Nothing says the server publishes a missed day or week epoch later, so a short server outage across 00:00 UTC is enough to trigger this.

**Checked:** The normal 10-minute path always has a base at E − 2 or older, so only the day and week path is affected. The REVIEW-011 fix (evidence keeps its records) covers later replacement, not which base to pick. The accepted "base record choice" gap covers cheaters, not honest clients.

**Resolution needed [proposed]:** Define one base record for a derived record of epoch E (for example the newest signed record with epoch ≤ E − 2, and how a client gets it), align SAVE-013 with it, and say whether the server fills in missed day and week epochs.

**References:** [Player data: Encounter seeds](tech/player-data.md#encounter-seeds) (SAVE-013); [Data formats: Epoch record](tech/data-formats.md#epoch-record--peerlingsepoch); [Peerling of the Day](gameplay/peerling-of-the-day.md); [Creation API: Network](tech/creation-api.md#network).

**Resolution:** —

### REVIEW-050: A save-log owner can invalidate Peerlings they have already traded away

- **Status:** Open
- **Severity:** High
- **Review:** 2026-11-10

**Finding [proposed]:** A fork of a save log shows up as duplicate encounter numbers, "so every catch after the fork fails", and cached verified catches are checked again against save-log entries the verifier didn't have before. The original catcher can at any time sign a new branch with an `encounter` whose number is at or below an old catch's number. A Peerling they caught and then traded to an honest player becomes invalid for every verifier that receives the branch, while other verifiers still hold it as verified. The trade is not void, nobody is flagged and the new owner gets nothing back. It is also not defined which catches count as "after the fork", or what happens when the duplicate number lies below a checkpoint's range.

**Checked:** Known gaps, D-0027, D-0028 and the open questions have no entry for this. Only the accidental two-computer fork is accepted, and that concerns the player's own Peerlings.

**Resolution needed [proposed]:** Define exactly which catches a fork invalidates, and either protect catches already transferred or covered by a checkpoint, or record this as an accepted risk.

**References:** [Player data: Verified Peerlings](tech/player-data.md#verified-peerlings); [Player data: Catches](tech/player-data.md#catches) (check 2); [Player data: Known gaps](tech/player-data.md#known-gaps-accepted-risks).

**Resolution:** —

### REVIEW-051: A move against a Peerling that already fainted this turn is undefined

- **Status:** Open
- **Severity:** High
- **Review:** 2026-11-10 (a case next to those fixed for REVIEW-021)

**Finding [proposed]:** With `strong-recoil` the first mover can hit, leave the opponent standing and faint from its own recoil. The second mover then has no standing target, and replacements happen only after the turn. The spec does not say whether that second move runs, or whether its accuracy, damage and stat-effect rolls are drawn. It also does not say whether a charge setup or strike is used up, whether `sig-empower` still triggers, or whether `sig-drain` heals. Two engines that differ here draw different RNG values, so a catch replay fails or a PvP battle ends void.

**Checked:** "A Peerling that faints before its move doesn't act and draws nothing" covers the fainted Peerling, not the move aimed at it. The D-0027 rules cover `quick-flurry`'s second hit and the charge strike on a replacement. Recoil is the only way a Peerling other than the target faints during the move phase, so this is the whole gap.

**Resolution needed [proposed]:** State whether a move against a fainted opposing Peerling fails, and exactly which rolls and side effects (charge state, drain, empower) still happen.

**References:** [Battle: A turn](gameplay/battle.md#a-turn); [Battle: Move mechanics](gameplay/battle.md#move-mechanics); [Battle: Random number generator](gameplay/battle.md#random-number-generator) (BTL-012, BTL-015).

**Resolution:** —

## Second review: Medium severity

### REVIEW-052: Encounters that aren't catches can't be checked when their epoch record is client-derived

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-003 and REVIEW-045 fixes incomplete)

**Finding [proposed]:** The `encounter` event stores only the epoch number. Recomputing its candidate list needs the epoch record's registry height and the day record. When either was client-derived (any server outage), the base record is in the log only for catches, inside the catch evidence. For fled, lost and won encounters nobody can recompute the species, yet the creator-stats rule counts `encounter` events "whose tile, epoch and candidate the server has checked", and D-0027 says the species of non-catch encounters can now be checked.

**Checked:** Verifier checks 1 to 6 require the species check only for catches; `GET /v1/epochs/{E}` returns only signed records.

**Resolution needed [proposed]:** Add the record references to the `encounter` event when a record is client-derived, or state that such encounters are not checked and not counted.

**References:** [Data formats: Save-log events](tech/data-formats.md#save-log-events); [Data formats: Catch evidence](tech/data-formats.md#catch-evidence); [Creator feedback: Species stats snapshot](gameplay/creator-feedback.md#species-stats-snapshot); [D-0027](decisions/D-0027-high-severity-review-fixes.md).

**Resolution:** —

### REVIEW-053: Guardian badges have no write order or crash recovery

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-003 fix incomplete)

**Finding [proposed]:** `catch` is ordered before `battle-result`, but nothing orders `badge` against `battle-result`, and start-up recovery (SAVE-029) knows only `catch`. After a crash at the end of a won guardian battle, either a `badge` exists whose battle is then logged as fled with no XP, or the badge is lost. Badge verification is only "like a catch": nothing says the `encounter` event must name the same `guardian` biome or that the replay must end in a win.

**Checked:** No other ordering or recovery rule mentions `badge`.

**Resolution needed [proposed]:** Write `badge` before `battle-result`, extend SAVE-029 to rebuild a guardian result from the badge evidence, and list the badge-specific checks.

**References:** [Guardians: Badges](gameplay/guardians.md#badges); [Player data: Encounter seeds](tech/player-data.md#encounter-seeds) (SAVE-029); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** —

### REVIEW-054: Several state changes have no event rule, though events are the save

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-002 fix incomplete)

**Finding [proposed]:** SAVE-039 requires every change to come from an event, and the server computes a checkpoint's `state` from the events. Several changes have no defined event or implied rule, so a client and the server can build different snapshots:
- **Team membership:** whether `catch`, `starter`, `created` or an incoming `trade` adds the Peerling to the team by itself, or a `team` event is needed. Catching says only that a catch with a full team goes to the collection.
- **Refill after a void trade** (CAT-010): no event is named, there is no tie-break for equal `obtainedAt`, and the case where every collection Peerling has fainted is open.
- **Generator update:** a last rest point that no longer exists "becomes the nearest one" with no event (a `rest` event would also heal).
- **Snapshot field `generator`:** no event sets it.

**Checked:** Trading step 5, D-0028 item 9 and the save-log event list.

**Resolution needed [proposed]:** Give the effect of each event type on team, collection and position, and an event or derivation rule for a moved rest point and for `generator`.

**References:** [Player data: Save log](tech/player-data.md#save-log) (SAVE-039); [Catching: Team and collection](gameplay/catching.md#team-and-collection); [Procedural generation: Generator updates](world/procedural-generation.md#generator-updates); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave).

**Resolution:** —

### REVIEW-055: A stale or chosen epoch record also selects the world and rules version

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-029 fix incomplete)

**Finding [proposed]:** Verifiers take the generator and rules version from the epoch of the epoch record an encounter used. Two cases are not covered:
- **Honest stale record.** Offline, or while drand is down, a client keeps using its last record. If that record is from before a generator switch and the clock has passed the switch ("every client switches at that epoch"), the spec does not say which world the client shows. If it switches by the clock, an encounter on a tile that is foliage only in the new version is checked against the old version and fails; a guardian's statue tile and level have the same problem.
- **Known gap understated.** A client-derived record copies `generator` and `rules` from its base record, so a cheater choosing an old base record also chooses older battle rules and an older world. The known gap says it "only changes which species can appear".

**Checked:** WGN-022 and D-0028 item 16 cover a guardian battle that spans a switch and the snapshot `generator` field, not these cases.

**Resolution needed [proposed]:** State that the world a client plays in is the version active at the epoch of the record it uses for encounters (or forbid encounters with a pre-switch record after the switch), and correct the known-gap text.

**References:** [Procedural generation: Generator updates](world/procedural-generation.md#generator-updates); [Battle: Rules versions](gameplay/battle.md#rules-versions); [Player data: Encounter seeds](tech/player-data.md#encounter-seeds); [Player data: Known gaps](tech/player-data.md#known-gaps-accepted-risks).

**Resolution:** —

### REVIEW-056: A forfeit win can be recorded from a battle that ended void or drawn

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-018 fix incomplete)

**Finding [proposed]:** A `forfeit` result is valid with the claimant's own end signature plus any state signature by the opponent. Nothing ties that state to the claimed turn, or shows that a timeout or drop happened. Since D-0028 every battle gives both sides a signed turn-0 state. A modified client can start a battle, receive the turn-0 state and then send a wrong state hash or a bad signature. The honest client ends the battle void and records nothing, so the check for a conflicting claim (PVP-013) finds none and the forged forfeit counts. The same works after a 200-turn draw. The victims are real, different opponents, so the "different opponents beaten" figure does not show it.

**Checked:** The recorded known gap covers only two colluding players; "conflicting forfeit claims cancel" needs the honest side to have recorded something.

**Resolution needed [proposed]:** Make void and draw outcomes provable or recorded, or define what a forfeit proof must show beyond a state the opponent once signed.

**References:** [PvP battles: Win record](gameplay/pvp-battles.md#win-record) (PVP-012, PVP-013); [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** —

### REVIEW-057: A `timeout` message binds the accused side without any check

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-016 fix incomplete)

**Finding [proposed]:** Either side declares the other late by sending `timeout`, and "both sides then use the action `timeout` for the late player". The only rule about late messages covers a `commit` or `reveal` that arrives after the timeout. Nothing covers a `timeout` sent for a message the accused side delivered in time. A modified client can read the opponent's reveal, then send `timeout` for it and turn that action into a random move; twice in a row forces a forfeit that passes the forfeit proof. If the honest client instead rejects the claim, the two states differ and the battle ends void. The spec picks neither, and the first reading breaks "neither can react to the other's choice" (PVP-005).

**Checked:** Disconnects are handled evenly (claims cancel); timeout claims have no such rule, and the case is not a recorded known gap.

**Resolution needed [proposed]:** Define what a side does when it receives a `timeout` for a decision whose message it already sent, and whether a disputed timeout voids the battle.

**References:** [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle); [PvP battles: Protocol](gameplay/pvp-battles.md#protocol); [Battle: PvP turn timer](gameplay/battle.md#pvp-turn-timer).

**Resolution:** —

### REVIEW-058: PvP resume is undefined before the turn-0 state

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-017 fix not reconciled with the turn-0 state of D-0028)

**Finding [proposed]:** In `resume`, `have` is "the last turn whose `state` it has received (0 if none)", so 0 means both "I have your turn-0 state" and "I have nothing". The reply to 0 resends only what followed `seed-reveal`, so a stream that drops during the team exchange, team verification or the seed exchange never gets `team`, `seed-commit` or `seed-reveal` again. The "forfeit by the other player after 60 s" rule has no starting condition; in that window one side may hold the other's turn-0 state while the other holds none, so only one side can claim a forfeit and the "claims cancel" safeguard fails. The turn-1 timer also still starts at the `seed-reveal` messages, not at the turn-0 states, unlike every later turn.

**Checked:** Battle § PvP turn timer, PvP battles § Protocol and PRT-004; none says when a battle counts as started.

**Resolution needed [proposed]:** Define the point from which a battle can be resumed and forfeited, what a drop before it means, and a `have` value that tells "no state" from "turn-0 state".

**References:** [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle) (PRT-004).

**Resolution:** —

### REVIEW-059: The unfinished-trade lock exists only in local client state

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-013 fix incomplete)

**Finding [proposed]:** The lock on offered Peerlings, and the data needed to expire a trade (the trade ID and the client's own signed transfer), are in no save-log event, snapshot or backup. Only the other side's offer data must be stored durably (TRD-007). A player who sends signed transfers, loses the stream and continues on a second computer, after cleared storage or after a restore has a client that knows nothing of the pending trade and lets the Peerling be traded or released again. When the partner later appends the original entry, the player has two transfers with the same `prev` and is flagged for a double trade, which bars trading and PvP. That client also can't call the expire endpoint, which needs one of the caller's own signed transfers.

**Checked:** The session-start check covers only entries already in the transfer log; no rule removes a flag.

**Resolution needed [proposed]:** Record the pending trade (trade ID, own signed transfers, locked instance IDs) in the save log before sending `transfers`, or define another way for the lock to survive a device change or storage loss.

**References:** [Player data: Transfer log and trades](tech/player-data.md#transfer-log-and-trades) (SAVE-036); [Trading: Requirements](gameplay/trading.md#requirements) (TRD-006, TRD-007); [Creation API: Trades](tech/creation-api.md#trades).

**Resolution:** —

### REVIEW-060: The relay rule conflicts with late-spectator block fetches and the larger phone backup

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (side effects of the REVIEW-019 and REVIEW-004 fixes)

**Finding [proposed]:** Clients may use a relayed connection only for the five game streams, "never for gossipsub, Bitswap or OrbitDB sync" (PERF-006). D-0028 makes a late spectator fetch the `start` and history blocks by bitswap from a fighter, and those blocks exist only on the two fighters. A spectator whose connections to both fighters stay relayed has no allowed path to them. Separately, the relay must carry at least 4 MB per connection, phone backup may run over a relay, and since D-0027 the backup holds the whole save log plus proofs. A long-time player's backup can pass 4 MB (the page's own scale figure is about 20,000 entries after 100 hours; the size per entry is the reviewer's estimate), and the phone-backup stream has no resume.

**Checked:** The retrieval ladder has no step that reaches a block held only by a relayed browser peer, and nothing says the operator fetches or pins battle blocks.

**Resolution needed [proposed]:** Give late spectators a relay-safe source for the two blocks (the operator, or a stated exception), and size the relay limit or the phone-backup transfer for a whole-log backup.

**References:** [Network performance: Connections](tech/network-performance.md#connections) (PERF-006); [Protocols: spectating](tech/protocols.md#peerlingsv1battlebattle-id--spectating) (SPT-005); [Data formats: Backup file](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup); [Player data: Phone backup](tech/player-data.md#phone-backup).

**Resolution:** —

### REVIEW-061: The wild Peerling's "best move" target is unclear on a turn where the player switches

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10

**Finding [proposed]:** The wild or guardian Peerling picks the move with the highest expected damage "against the player's active Peerling", and that choice is the first draw of the turn. Switches resolve before moves in the same turn. The text does not say whether the score uses the Peerling that is out when actions are chosen or the one that comes in. The two give different moves whenever the switch changes types, so a verifier's replay can differ from the player's battle.

**Checked:** Guardians and encounters add nothing; no page says "before the switch" or "after the switch". Ties between equal scores are defined.

**Resolution needed [proposed]:** State whether the score is computed against the Peerling that is out before or after this turn's switch.

**References:** [Battle: Wild Peerling behaviour](gameplay/battle.md#wild-peerling-behaviour); [Battle: A turn](gameplay/battle.md#a-turn) (BTL-009, BTL-012).

**Resolution:** —

### REVIEW-062: A refused wish has no job transition, and the FAILED state is unclear in the job object

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-036 fix incomplete)

**Finding [proposed]:** `wish-refused` is listed as an HTTP error code, but the concept stage runs after `POST /v1/jobs` has returned, so it can only appear in the job's `error` field, whose format is not defined. The player is to "edit" the wish, but no endpoint takes a new wish in that state: `retry` runs the same step with the same wish, and `regenerate` exists only from `IMAGE_READY`. For a starter, a new `POST /v1/jobs` is blocked by `starter-job-open`. Also, "keeps the state it failed from" does not say whether `state` reads `FAILED` or the earlier state, and the job object has no field for the failed-from state, so a client that reconnects (CRE-014) can't tell which step failed.

**Checked:** No page defines the `error` object or a wish-edit transition; deleting the job and starting again is not stated, and for a shrine job it would go through the credit.

**Resolution needed [proposed]:** Define the `error` field, how the failed-from state appears in the job object, and the endpoint that submits an edited wish after `wish-refused`.

**References:** [Creation API: The job object](tech/creation-api.md#the-job-object) (API-010); [Creation pipeline: Job handling](peerlings/creation-pipeline.md#job-handling).

**Resolution:** —

### REVIEW-063: The job object can't carry the image-check results the client must show

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-033 and REVIEW-034 fixes not carried to the API)

**Finding [proposed]:** D-0028 made two image outcomes visible to the player: an image that "can't be accepted" (a second resemblance hit, IMG-018, or a transparency failure after matting) and an image "shown with a note of what failed" that may still be accepted. The job object has only `originalityChanged`. No field says which checks failed, or that the current image is blocked and why, so the client can't show the notices or disable Accept. `accept-image` names an error only for the resemblance case.

**Checked:** Q-059 lists `originalityChanged` and IMG-013 as awaiting approval but not the missing fields; IMG-018 is accepted.

**Resolution needed [proposed]:** Add job-object fields for failed image checks and for a blocked image with its reason, and name the `accept-image` error for a transparency-blocked image.

**References:** [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules); [Image prompting: Image checks and clean-up](peerlings/image-prompting.md#image-checks-and-clean-up); [Creation API: The job object](tech/creation-api.md#the-job-object).

**Resolution:** —

### REVIEW-064: The creator's display name in the species record has no source

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10

**Finding [proposed]:** The species record, signed by the server at `finalize`, requires `creator.displayName`. No creation-API request carries a display name and no rule says where the server takes it from. The `profile` save-log event is not a safe source for starters: the wish is sent before the character exists, and save replication runs in the background and never blocks. A source is stated only for seed jobs.

**Checked:** Every other use of a display name (first finder, creator topic, presence) comes from logs or messages the server already holds.

**Resolution needed [proposed]:** State where the server gets the creator's display name at `finalize` and what happens when none is available yet.

**References:** [Data formats: Species record](tech/data-formats.md#species-record--peerlingsspecies); [Creation API: Creating a Peerling](tech/creation-api.md#creating-a-peerling); [Onboarding: Creating a starter](gameplay/onboarding.md#creating-a-starter).

**Resolution:** —

### REVIEW-065: The Viewer can't tell that a species is delisted

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-027 fix incomplete)

**Finding [proposed]:** D-0028 requires an existing card link to show only "This Peerling has been delisted". The Viewer fetches by CID with verified-fetch; nothing says it reads the registry, the registry index or any other listing status. A delisted species' record stays pinned and correctly signed, so by the stated mechanism its card loads with a verified badge. The same holds for a record signed at `finalize` whose job never reached listing: it carries a valid signature, and its name is later freed.

**Checked:** No page gives the Viewer a registry lookup, and the species-stats value has no delisted flag.

**Resolution needed [proposed]:** Specify how the Viewer and the in-game species card check a species CID's listing status, including while the server is offline.

**References:** [Sharing: The viewer](gameplay/sharing.md#the-viewer); [Sharing: Peerling card](gameplay/sharing.md#peerling-card); [OrbitDB registry: Design](tech/orbitdb-registry.md#design).

**Resolution:** —

### REVIEW-066: The world generator is not specified, yet every verifier must reproduce it exactly

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10

**Finding [proposed]:** The page gives determinism rules (integer maths, SHA-256 noise, Voronoi ties) but no algorithm: noise parameters, area seed points, foliage patches, heights, paths, rest-point and statue placement and the integer sector constants are absent. It says the algorithm "is defined by that version's implementation, which every client ships". But the server also verifies every save log, and no rule makes one generator module or a set of test vectors binding on the client, the server and other verifiers. Two generators written from this page would disagree on which tiles are foliage, and honest catches would fail. The "exact" 30° sectors and pointy-top hexagon (WGN-018) can't be exact in integer vectors either; the constants that approximate them are not given. The first review's scope note raised this as advice; it was never tracked as a finding.

**Checked:** No page mentions a reference implementation, shared module or conformance vectors for the generator. The battle engine, by contrast, is fully specified in integers.

**Resolution needed [proposed]:** Either specify the verification-relevant functions exactly (tile kind, biome, area, rest-point and statue tiles, with constants), or state that one generator module per version is the single artefact every client, the server and any verifier must run, with test vectors.

**References:** [Procedural generation: Deterministic world maths](world/procedural-generation.md#deterministic-world-maths) (WGN-002, WGN-018, WGN-021); [Player data: Catches](tech/player-data.md#catches); [Guardians: Guardian sites](gameplay/guardians.md#guardian-sites).

**Resolution:** —

### REVIEW-067: Respawn and start tiles are undefined

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-002 fix incomplete)

**Finding [proposed]:** D-0028 made "a lost battle moves the player to the last rest point" a state change that follows from events and that the server recomputes for a checkpoint. But `lastRestPoint` and the `rest` event hold "the rest point's tile", and a rest point is used by facing its beacon, so that tile is not clearly one the player can stand on. No page says which tile and facing the player has after a respawn. The same gap covers a new player's first position and the tile of the spawn hub's rest point as the first `lastRestPoint`.

**Checked:** The "nearest" rule of the generator-update section covers only relocations after an update.

**Resolution needed [proposed]:** Define the rest-point tile versus the tile the player stands on, the position and facing after a respawn, and the new player's start tile and first `lastRestPoint`.

**References:** [Player data: Save log](tech/player-data.md#save-log); [Exploration: Healing and rest points](gameplay/exploration.md#healing-and-rest-points); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** —

### REVIEW-068: The map's first revealed area and its landmark marks are not defined in the save

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10

**Finding [proposed]:** "The spawn hub and the paths leading out of it are revealed from the start", but paths connect every rest point in the world, so the first set of revealed chunks has no stated end and is not given as starting `explored` state. A rest point "marks the landmark on the player's map" and the map shows "discovered rest points and landmarks", but the save holds only the chunk bit set and `lastRestPoint`. "Discovered" is never defined, and a mark that doesn't follow from revealed chunks is lost when the save is loaded.

**Checked:** EXP-007 only restates the feature; the snapshot field list has no landmark field.

**Resolution needed [proposed]:** Define the chunks revealed at the start exactly, and define "discovered" (for example, derived from revealed chunks) or add the marks to the snapshot.

**References:** [Exploration: Map](gameplay/exploration.md#map); [Exploration: Healing and rest points](gameplay/exploration.md#healing-and-rest-points); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave).

**Resolution:** —

### REVIEW-069: Four glossary definitions still describe behaviour from before D-0027 and D-0028

- **Status:** Open
- **Severity:** Medium
- **Review:** 2026-11-10 (REVIEW-004, REVIEW-005 and REVIEW-027 fixes not carried to the glossary)

**Finding [proposed]:** The glossary is the canonical home of term definitions, and four entries contradict their pages:
- **Recovery phrase:** "shown to the player once". The phrase can be shown again on every device (SAVE-040).
- **Backup file:** "private key and latest save snapshot". It holds the whole save log plus origin proofs and ownership chains (SAVE-030).
- **Delisting:** "clients stop showing it". Owned Peerlings stay, are shown as "Delisted Peerling", still battle, and can't be traded, followed or shared.
- **XP:** "earns from wild battles". Guardian wins give XP too.

**Checked:** The other entries touched by the two decisions (encounter number, double trade, genesis record, shrine credit) match their pages.

**Resolution needed [proposed]:** Update the four definitions and the glossary's sources and date.

**References:** [Glossary](glossary.md); [Player data: Account recovery](tech/player-data.md#account-recovery); [Data formats: Backup file](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup); [OrbitDB registry: Design](tech/orbitdb-registry.md#design); [Guardians: The battle](gameplay/guardians.md#the-battle).

**Resolution:** —

## Second review: Low severity

### REVIEW-070: The save-log table in player-data omits `badge` and `pvp-result`

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** SAVE-007 ("event-based as listed in Save log") and SAVE-039 point to the Save log table as the event list. The table lacks two events that the format page defines and that change snapshot fields (`badges`, `pvp`).

**Checked:** Data formats wins on conflict, so this is a documentation gap only.

**Resolution needed [proposed]:** Add the two rows, or make the table an explicit pointer to data-formats.

**References:** [Player data: Save log](tech/player-data.md#save-log); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** —

### REVIEW-071: The onboarding flow has no step for the key and the recovery phrase

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** Player data says the 12 words are "shown at onboarding", and every server call needs the player key, but the six-step flow has no such step. The "choose an existing starter" path also doesn't say when the character is created relative to the choice (ONB-005 requires both).

**Checked:** The phrase appears elsewhere only for title-screen recovery and the Backup menu.

**Resolution needed [proposed]:** Add the key and phrase step to the flow, and the step order for the choose path.

**References:** [Onboarding: Flow](gameplay/onboarding.md#flow); [Player data: Account recovery](tech/player-data.md#account-recovery).

**Resolution:** —

### REVIEW-072: PvP deadlines are written only for decisions both sides make

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10 (REVIEW-016 fix incomplete)

**Finding [proposed]:** A side sends its `reveal` "as soon as it has both commits", and the replacement timer starts "when both reveals of that turn's actions have been exchanged". The same section defines decisions only one side makes (the other is charging, or only one side replaces) and turns with no decision (both charging). Read literally, a single decider never has "both commits", and a replacement after a turn with fewer than two action reveals has no timer start.

**Checked:** Battle § PvP turn timer says only that the charge turn has no timer.

**Resolution needed [proposed]:** Word the deadlines as "all commits or reveals this decision requires", and define the replacement-timer start for turns with fewer than two action reveals.

**References:** [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle).

**Resolution:** —

### REVIEW-073: Crossing or pending requests are not covered by the "busy" rule

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** A request is declined as "busy" only when the player is "already in a battle or trade" (MPL-009). A player with an unanswered `challenge` or `propose`, sent or received, is in neither. Two players challenging each other at once, or a third player's request arriving while one is pending, has no defined outcome.

**Checked:** Multiplayer, UI, PvP battles, trading and protocols say nothing about pending or simultaneous requests.

**Resolution needed [proposed]:** Say whether a pending request counts as busy, and how two crossing requests between the same pair resolve.

**References:** [Multiplayer: Communication](gameplay/multiplayer.md#communication) (MPL-009); [Protocols: PvP battle](tech/protocols.md#peerlingsbattle100--pvp-battle).

**Resolution:** —

### REVIEW-074: Who "took part" in a battle for XP is not defined

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** XP goes to every Peerling "that took part in the battle and didn't faint". "Took part" could mean in the team at the start, sent out, or having used a move. The `xpGained` values of `battle-result` depend on it, and so does the replay that rebuilds a missing `battle-result` after a crash (SAVE-029).

**Checked:** This is the only use of the phrase in a battle context.

**Resolution needed [proposed]:** Define taking part exactly (for example "was the active Peerling at any point in the battle").

**References:** [Battle: Experience and levelling](gameplay/battle.md#experience-and-levelling).

**Resolution:** —

### REVIEW-075: SHR-005 contradicts the shrine-credit text, and the 30 days have two start points

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10 (REVIEW-036 and REVIEW-039 fixes not carried to the requirement)

**Finding [proposed]:** SHR-005 still gives a credit when a shrine job "fails". The text above it now says a failed job stays open and can be retried, and the credit comes only when it is abandoned or expires. Read literally, SHR-005 gives an open job and a credit together, against SHR-006. Separately, the credit lasts 30 days from when the job ends, yet the returned-credit rule says it is gone "30 days after the offering"; the two differ by the first job's lifetime, which has no upper limit while the player is active.

**Checked:** Creation API and D-0028 item 22 don't fix the start of the 30 days.

**Resolution needed [proposed]:** Reword SHR-005 to match the text, and state whether the 30 days run from the offering or from the end of the first job.

**References:** [Creation Shrine: Rules](gameplay/creation-shrine.md#rules) (SHR-005, SHR-006).

**Resolution:** —

### REVIEW-076: The enhancer system prompt has a broken example and lengths that don't match the schema

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** IMG-005 requires the system prompt word for word, and it contains errors:
- the third example output is not valid JSON: the `originality` object is never closed, so `subject` sits inside it;
- that example's `subject` is 588 characters, under the schema's minimum of 600, so the page's own failure handling would reject it;
- the prompt asks for "100-250 words" while the schema requires 600 to 1,600 characters; short answers within the word range fall under 600 characters;
- the three example inputs have an `appearance` of 72 to 88 characters, under the concept schema's minimum of 100 (CRE-027).

**Checked:** Each example line was parsed and counted by script; examples 1 and 2 parse and are in range. The page is still proposed under Q-059, which does not list these errors.

**Resolution needed [proposed]:** Fix example 3, align the word and character ranges, and make the example concepts valid.

**References:** [Image prompting: System prompt](peerlings/image-prompting.md#system-prompt); [Image prompting: Output schema](peerlings/image-prompting.md#output-schema); [Creation pipeline: Stage 2](peerlings/creation-pipeline.md#stage-2--concept).

**Resolution:** —

### REVIEW-077: The registry page lists fewer signed listing fields than the format

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** The registry page says the server signs "species CID, `seq`, name, types, thumbnail CID, `createdAt`". Data formats signs the whole envelope body, which also holds `nameKey`, `status`, `removedAtSeq` and `listedSeq`. An access controller built from the registry page would leave `status` unsigned, so a tombstone could be forged or stripped.

**Checked:** Data formats wins on conflict, which keeps this Low.

**Resolution needed [proposed]:** Replace the field list on the registry page with "the whole listing body" and a link.

**References:** [OrbitDB registry: Design](tech/orbitdb-registry.md#design); [Data formats: Registry entry](tech/data-formats.md#registry-entry--peerlingslisting); [Data formats: Signed envelope](tech/data-formats.md#signed-envelope).

**Resolution:** —

### REVIEW-078: The species-stats table in creator-feedback was split in two

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10 (regression from the REVIEW-045 fix)

**Finding [proposed]:** The "How they are counted" block was inserted in the middle of the stats table. The `providers` row now follows a bullet list with no table header, so it renders as stray text and the table appears to define four fields instead of five.

**Checked:** Read in the raw file: no blank line or header precedes the row.

**Resolution needed [proposed]:** Move the `providers` row back into the table, above the counting rules.

**References:** [Creator feedback: Species stats snapshot](gameplay/creator-feedback.md#species-stats-snapshot).

**Resolution:** —

### REVIEW-079: Decision status lines and resolved questions miss later changes

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** Bookkeeping that the schema requires (AGENTS.md §7) is missing in two places:
- **Decisions.** D-0024 says it changes parts of D-0013, D-0017, D-0021 and D-0022. None of their Status lines names D-0024 (D-0013 and D-0022 name only D-0027 or D-0028; D-0017 and D-0021 name nothing), and the index rows follow them.
- **Resolved questions.** Q-052 cites EXP-011, which was removed and replaced by EXP-014, and says the first encounter is always number 0, which a guardian battle can now use up. Q-043 says the backup file holds the latest snapshot (now the whole log). Q-021 says the species stats are in OrbitDB (replaced by the snapshot in D-0023). Other resolved entries carry a "since" note for such changes; these three don't.

**Checked:** The four decision texts are high-level and not contradicted in substance; the other resolved questions checked still match their pages.

**Resolution needed [proposed]:** Add "partly changed by D-0024" to the four Status lines and index rows, add notes to the three resolved questions, and point Q-052 at EXP-014.

**References:** [D-0024](decisions/D-0024-review-2-decisions.md); [D-0013](decisions/D-0013-peer-verified-registry-catches-trades.md); [D-0017](decisions/D-0017-world-features.md); [D-0021](decisions/D-0021-pvp-win-counter.md); [D-0022](decisions/D-0022-v1-fun-features.md); [Open questions: Q-052](open-questions.md#q-052); [Q-043](open-questions.md#q-043); [Q-021](open-questions.md#q-021).

**Resolution:** —

### REVIEW-080: The signpost example can't occur under the stated range rule

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** A signpost's range is the lowest base level in the area minus 2 to the highest plus 2. Areas are about 300 to 500 m across and the base level rises with distance, so an area inside the level-scaling zone spans clearly more than 4 levels. The example "level 22–26" needs one base level across the whole area, so a reader may take the range as tile-local.

**Checked:** Only areas where the base level is constant (at the cap) give a 4-level span, and there the range would end at 50.

**Resolution needed [proposed]:** Replace the example with a range the rule can produce.

**References:** [Procedural generation: Points of interest](world/procedural-generation.md#points-of-interest); [Encounters: Wild level](gameplay/encounters.md#wild-level).

**Resolution:** —

### REVIEW-081: The list of cry triggers omits petting

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** Audio lists when a cry plays: entering a battle, opening a card, and being chosen as starter. Exploration adds a fourth trigger: petting the following Peerling plays its cry.

**Checked:** No other trigger list exists.

**Resolution needed [proposed]:** Add petting to the list in audio.

**References:** [Audio: Peerling cries](world/audio.md#peerling-cries); [Exploration: Following Peerling](gameplay/exploration.md#following-peerling).

**Resolution:** —

### REVIEW-082: The phone-backup menu path contradicts the UI page

- **Status:** Open
- **Severity:** Low
- **Review:** 2026-11-10

**Finding [proposed]:** Player data puts phone backup under "Settings → Back up to phone". The UI page puts it under the pause menu's Backup item, has no backup entry in Settings, and player data itself uses "Backup → Show recovery phrase".

**Checked:** No other page names the menu path.

**Resolution needed [proposed]:** Change the path in player data to the Backup menu.

**References:** [Player data: Phone backup](tech/player-data.md#phone-backup); [UI: Pause menu](gameplay/ui.md#pause-menu-esc).

**Resolution:** —

## See also

- [Wiki index](index.md)
- [Open questions](open-questions.md)
- [Change log](log.md)

