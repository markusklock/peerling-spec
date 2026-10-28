---
title: Specification Review
type: reference
status: draft
tags: [review, specification, implementation, tracking]
sources:
  - raw/conversations/2026-10-27-specification-review-tracker.md
  - raw/conversations/2026-10-27-specification-review-verification.md
  - raw/conversations/2026-10-27-high-severity-resolutions.md
related:
  - wiki/index.md
  - wiki/open-questions.md
  - wiki/tech/data-formats.md
  - wiki/tech/player-data.md
  - wiki/tech/protocols.md
  - wiki/peerlings/image-prompting.md
  - wiki/decisions/D-0027-high-severity-review-fixes.md
updated: 2026-10-27
---

# Specification Review

> Tracks all 47 findings from the specification review of 2026-10-19, with
> severity, editable status, evidence links, verification notes and resolution
> notes. Finding IDs preserve the original review numbering so each issue can
> be followed through later changes.

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

| Severity | Meaning | Findings |
|----------|---------|---------:|
| High | Can block a core flow, lose progress or ownership, invalidate verification, or make clients disagree on authoritative state. | 16 |
| Medium | Leaves a feature incomplete, contradicts intended behavior, or requires a consequential implementation choice. | 23 |
| Low | A localized tutorial, navigation, example, or documentation issue with a clear broader design. | 8 |

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

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 2

**Finding [proposed]:** The save is supposedly reconstructed by applying events, but there is no healing/rest-point event. Rest visits change HP and `lastRestPoint` through a snapshot instead. Verifier checks do not use HP or the rest point, but the completeness of a verification checkpoint's state depends on the answer.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. "Apply all events" and "latest snapshot plus later events" are never reconciled, and no heal or rest-point event exists. But EXP-008 writes a snapshot on every rest visit, so the normal load path works, and the verifier checks use only event-derived data (Peerdex, encounter numbers, epochs). Wording on the verification impact corrected.

**Resolution needed [proposed]:** Define whether snapshots are authoritative state changes or merely caches of event-derived state, and specify how healing and rest-point changes are reconstructed.

**References:** [Player data: Save log](tech/player-data.md#save-log); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave).

**Resolution:** —

### REVIEW-005: Showing the recovery phrase again is unsupported

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 5

**Finding [proposed]:** The UI offers "show recovery phrase," but the phrase is shown once at onboarding and no storage of the mnemonic or its entropy is specified. Backups store only the derived private key, from which the BIP-39 phrase cannot be reconstructed. The action is therefore unsupported on every device, not only after file or phone restoration.

**Verification (2026-10-27) [proposed]:** Confirmed and broader than originally stated: the phrase is shown once at onboarding and nothing secret is stored, so even the original device cannot show it again. Title and wording corrected.

**Resolution needed [proposed]:** Specify whether the client stores the mnemonic or entropy (and whether backups carry it), or whether the action is removed or limited to onboarding.

**References:** [UI: Pause menu](gameplay/ui.md#pause-menu-esc); [Data formats: Recovery phrase](tech/data-formats.md#recovery-phrase); [Backup format](tech/data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup).

**Resolution:** —

### REVIEW-010: Anti-reroll guarantees exceed what the evidence proves

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 10

**Finding [proposed]:** A modified client can append invented non-catch `battle-result` events to advance encounter numbers quickly; those events contain no replay evidence. This allows searching for desirable traits or shimmers without the claimed five-minute reroll cost. Likewise, client-derived-record validity does not prove that the server was offline. These are broader gaps than the currently described risks.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from High. `battle-result` carries no replay evidence, and fleeing always succeeds, so even honest play advances encounter numbers cheaply. The known-gap text "only while the server is offline" is inaccurate, because the client-derived validity rule does not check that. Verifiers still agree on results, so this weakens an anti-cheat promise rather than making clients disagree.

**Resolution needed [proposed]:** Explicitly accept these broader limits or define stronger verification rules; align the anti-reroll explanation and known-risk descriptions with the chosen behavior.

**References:** [Player data: Encounter seeds](tech/player-data.md#encounter-seeds); [Known gaps](tech/player-data.md#known-gaps-accepted-risks); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** —

### REVIEW-014: Equivalent previous-transfer entries can produce different next-transfer signatures

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 14

**Finding [proposed]:** Identical transfers may appear in multiple OrbitDB entries, but the next transfer's `prev` points to an entry CID. Choosing different equivalent entries changes the signed envelope, so any future retry or re-signing that picks a different equivalent entry would look like a double trade. No currently specified flow re-signs a transfer.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. `prev` is not canonical when the same envelope sits in several entries, but every trade uses a fresh random trade ID and nothing re-signs after step 5, so no specified flow produces an identical retry. Verifiers resolve `prev` to the envelope, so verification does not diverge. Wording corrected.

**Resolution needed [proposed]:** Specify a canonical `prev` or an equivalence rule for these retries.

**References:** [Data formats: Transfer and transfer-log entry](tech/data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer).

**Resolution:** —

### REVIEW-015: Trade confirmation binds IDs rather than the offered instance state

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 15

**Finding [proposed]:** The offers hash covers only instance IDs, while the receiver obtains level, XP, HP, and nickname from the offered data. The transfer envelopes carry none of that data. The receiver's `trade` save-log event stores it, but the received offer data is not required to be persisted before that event is written, so recovery at session start or on another computer has no source for it.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. The receiver's `trade` event stores the full received instances, so the state is permanent once written; the gap is the window before that event. Unverified levels are already an accepted risk. Wording corrected.

**Resolution needed [proposed]:** Define what exact state confirmation commits to and how that state remains recoverable.

**References:** [Protocols: Trade and offers hash](tech/protocols.md#peerlingstrade100--trade); [Trading: Flow](gameplay/trading.md#flow).

**Resolution:** —

### REVIEW-018: PvP result proofs leave important cases unresolved

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 18

**Finding [proposed]:** Forfeits need the loser's signed state, but none exists before the first completed turn. State signatures do not bind the level mode, so a winner-signed forfeit can label a loser's state with a different mode. Conflict cancellation also requires discovering claims in both players' logs, while profile counting describes fetching the inspected player's log.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from High, because the win counter is a stat only with no reward ([D-0021](decisions/D-0021-pvp-win-counter.md)). Wording corrected: D-0024 accepted a forged-forfeit mitigation, not a listed accepted risk.

**Resolution needed [proposed]:** Define these cases consistently with the forged-forfeit mitigation accepted in [D-0024](decisions/D-0024-review-2-decisions.md), including how competing claims are discovered and how mode and initial state are bound to the result.

**References:** [Protocols: PvP signatures](tech/protocols.md#peerlingsbattle100--pvp-battle); [Data formats: Save-log events](tech/data-formats.md#save-log-events); [PvP: Win record](gameplay/pvp-battles.md#win-record).

**Resolution:** —

### REVIEW-019: Late spectators cannot obtain the initial battle setup

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 19

**Finding [proposed]:** Turn messages contain cumulative actions and the seed, but teams, mode, and rules version appear only in the initial `start` message. A spectator joining later has no specified request or replay mechanism for retrieving that message.

**Verification (2026-10-27) [proposed]:** Confirmed. It also contradicts the spectating page's statement that a late spectator catches up from any single message. Authentication is straightforward once the message is retrieved (strict signing plus state hashes).

**Resolution needed [proposed]:** Define how late spectators retrieve and authenticate the initial setup.

**References:** [Protocols: Spectating](tech/protocols.md#peerlingsv1battlebattle-id--spectating); [Spectating design](gameplay/spectating.md#design).

**Resolution:** —

### REVIEW-020: Spectator messages exceed their mandatory size limit

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 20

**Finding [proposed]:** Every turn message must contain all previous actions, and battles may last 200 turns. The review calculated 18,065 bytes for a 200-turn message containing only quick moves, before pubsub overhead; the battle-topic limit is 16,384 bytes (16 KiB). Switch-heavy battles exceed the limit much earlier (from turn 112 if every action is a switch).

**Verification (2026-10-27) [proposed]:** Confirmed: 18,065 bytes is exact for the DAG-CBOR encoding, and the message first exceeds the limit at turn 182. Understated: a switch action encodes to 50 bytes, so an all-switch battle exceeds 16 KiB from turn 112, and long switch-heavy battles are the case the 200-turn draw exists for. The spectating page's "this stays small" is contradicted.

**Resolution needed [proposed]:** Specify chunking, history retrieval, or a different message format that respects the limit throughout a valid battle.

**References:** [Spectating requirements](gameplay/spectating.md#requirements); [Data formats: Size limits](tech/data-formats.md#size-limits); [Protocols: Spectating](tech/protocols.md#peerlingsv1battlebattle-id--spectating).

**Resolution:** —

### REVIEW-022: Peerling of the Day can become less common instead of more common

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 22

**Finding [proposed]:** Its replacement weight is `max(1, W div 19)`. With three unseen Normal species in Plains, each ordinarily has weight 12. Featuring one changes its first-draw chance from 33.3% to 4%, contrary to the stated increase.

**Verification (2026-10-27) [proposed]:** Confirmed; arithmetic checked. The featured species replaces its own weight: W = 24, its weight is max(1, 24 div 19) = 1, giving 1/25 = 4% instead of 12/36 = 33.3%. This contradicts POD-001 ("MUST appear more often"). The featured species is penalised whenever W div 19 is below its natural weight (W < 228 for an unseen native species), which is exactly the small cold-start registry.

**Resolution needed [proposed]:** Decide whether the goal is an approximately 5% share or a guaranteed increase, and align the formula with that goal.

**References:** [Peerling of the Day: In encounters](gameplay/peerling-of-the-day.md#in-encounters); [Encounters: Selection](gameplay/encounters.md#selection).

**Resolution:** —

### REVIEW-025: Seen is not fully defined outside wild encounters

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 25

**Finding [proposed]:** Clarify whether PvP opponents, guardians, followers, gallery models, and opened species cards count as sightings. Starters, creations, and trades explicitly count, but their events do not consistently supply the mandatory first-met biome stored in the Peerdex.

**Verification (2026-10-27) [proposed]:** Confirmed. A guardian `battle-result` has no `species` field and the guardians page never mentions `seen`; `starter`, `created` and `trade` events carry no biome, while the snapshot's Peerdex entry requires one.

**Resolution needed [proposed]:** Define sighting sources and the first-biome value for every non-wild acquisition or sighting.

**References:** [Peerdex: What it lists](gameplay/peerdex.md#what-it-lists); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave); [Encounters: Selection](gameplay/encounters.md#selection).

**Resolution:** —

### REVIEW-027: Delisting has inconsistent scope

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 27

**Finding [proposed]:** Some text says clients stop showing a delisted species; other rules retain owned assets permanently and keep delisted guardian members for the week. `SRV-002` also requires pinning every asset, whereas `REG-010` permits unpinning delisted assets.

**Verification (2026-10-27) [proposed]:** Confirmed. SRV-002 ("MUST pin every game asset") directly contradicts REG-010 ("MAY unpin its assets").

**Resolution needed [proposed]:** Specify treatment in collections, PvP, trades, followers, shared cards, and cached encounters, and reconcile the server-pinning requirements.

**References:** [D-0010: Decision](decisions/D-0010-no-content-moderation.md#decision); [Registry: Design](tech/orbitdb-registry.md#design); [Generation server requirements](tech/generation-server.md#requirements); [Guardians: Weekly team](gameplay/guardians.md#the-weekly-team).

**Resolution:** —

### REVIEW-029: World updates need rules for ongoing battles and saved map state

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 29

**Finding [proposed]:** Generator changes can move guardian statues (and with them their level) and relocate rest points. Guardian evidence carries both an epoch record and a week record, and the spec does not say which record's generator version validates the statue tile. Relocation goes to the "nearest" walkable tile or rest point without saying nearest to what. The snapshot has no explicit generator-version field, although it is derivable from each encounter's epoch record.

**Verification (2026-10-27) [proposed]:** Partly confirmed. Players on newly blocked tiles and missing rest points are already relocated; the generator version is derivable from each encounter's epoch record; the guardian level changes only if the statue tile moves; explored chunks are a positional bitset. The remaining gap is which record's generator version validates guardian evidence. Wording corrected.

**Resolution needed [proposed]:** Specify which version determines a guardian battle spanning an update and how map and rest-point state migrate.

**References:** [Procedural generation: Generator updates](world/procedural-generation.md#generator-updates); [Data formats: Save snapshot](tech/data-formats.md#save-snapshot--peerlingssave); [Guardians: Weekly team](gameplay/guardians.md#the-weekly-team).

**Resolution:** —

### REVIEW-031: Implementation-critical proposals remain unapproved inside accepted pages

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 31

**Finding [proposed]:** Examples include `SRV-007`, API refusal and outage handling, and provenance fields. The image-prompting page correctly remains draft, but the open-question list tracks only licences and the default 3D generator, not approval of the broader proposed pipeline behavior.

**Verification (2026-10-27) [proposed]:** Confirmed. creation-api, creation-pipeline, generation-server and data-formats have status `accepted` but contain `[proposed]` items (for example SRV-007, `name-reserved`, outage and `wish-refused` handling, provenance `imagePrompt` fields). Only Q-057 and Q-058 are open.

**Resolution needed [proposed]:** Make the decision boundary explicit for implementers and track approval or rejection of the remaining proposals through the existing question and decision workflow.

**References:** [Generation server requirements](tech/generation-server.md#requirements); [Creation pipeline: Job handling](peerlings/creation-pipeline.md#job-handling); [Open questions](open-questions.md#open).

**Resolution:** —

### REVIEW-033: The originality guarantee conflicts with proposed fail-open behavior

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 33

**Finding [proposed]:** `IMG-003` says the pipeline MUST stop character cloning, but after one retry the second image is shown regardless of the resemblance result.

**Verification (2026-10-27) [proposed]:** Confirmed. IMG-003 is `[accepted]`; the fail-open flow and IMG-010 (a SHOULD) are `[proposed]`. The second check's result is not kept in provenance. The other originality layers still apply.

**Resolution needed [proposed]:** Decide whether originality checking is best-effort or whether known failed images must be blocked from publication.

**References:** [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules); [Requirements](peerlings/image-prompting.md#requirements).

**Resolution:** —

### REVIEW-035: Franchise matching uses incompatible normalization and matching rules

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 35

**Finding [proposed]:** Franchise matching applies species-name normalization, which removes spaces and punctuation, then requires whole-word or adjacent-word matching. Word boundaries are gone by then.

**Verification (2026-10-27) [proposed]:** Confirmed. Also undefined: the order of the leetspeak mapping relative to NFKC and confusable mapping; what "words" mean for Japanese written without spaces; whether accent removal strips Japanese voicing marks. CRE-025 itself does not mention spaces; only the prose does.

**Resolution needed [proposed]:** Define tokenization order, Unicode normalization details, and what happens when a species name normalizes to an empty string.

**References:** [Image prompting: Originality rules](peerlings/image-prompting.md#originality-rules); [Creation pipeline: Final review and naming](peerlings/creation-pipeline.md#final-review).

**Resolution:** —

### REVIEW-036: Creation jobs lack complete retry and recovery operations

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 36

**Finding [proposed]:** `FAILED` is retryable, but no retry transition or endpoint exists; the only retry is starting a new job, which loses the wish and image. Recovering a starter job after losing its locally stored job ID is also undefined. Automatic polling continually extends expiry, while a long-lived SSE connection may not.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. There is no exit from `FAILED`, no retry endpoint and no starter-job discovery, although ONB-004 requires resuming the same job. But starting a new job is a working (lossy) retry and a failed shrine job grants a credit (SHR-005), so no player is stuck. Losing a starter after its `origin` was issued belongs to REVIEW-037. Wording corrected.

**Resolution needed [proposed]:** Define retry transitions, job discovery after local data loss, and a deliberate distinction between player activity and request activity for expiry.

**References:** [Creation pipeline: Job handling](peerlings/creation-pipeline.md#job-handling); [Creation API: Job object](tech/creation-api.md#the-job-object); [Creating a Peerling](tech/creation-api.md#creating-a-peerling).

**Resolution:** Partly addressed by [D-0027](decisions/D-0027-high-severity-review-fixes.md): `GET /v1/starter` finds a lost starter job. Retry transitions and the expiry distinction remain open.

### REVIEW-038: The operator's seed-species workflow is absent from the API

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 38

**Finding [proposed]:** Seed species must use the normal pipeline and receive no instance, but job kinds are only `starter` and `shrine`, with their associated restrictions.

**Verification (2026-10-27) [proposed]:** Confirmed; kept Medium. Also unspecified: which player key and `creator.id` a seed species uses. That value ends up in signed public records, and seed species are needed at launch.

**Resolution needed [proposed]:** Specify the operator-only creation path or an explicit administrative exception.

**References:** [Creation pipeline: Who uses the pipeline](peerlings/creation-pipeline.md#who-uses-the-pipeline); [Stage 8](peerlings/creation-pipeline.md#stage-8--the-new-peerling-joins-the-player); [Creation API](tech/creation-api.md#creating-a-peerling).

**Resolution:** —

### REVIEW-039: Shrine credits lack lifecycle details

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 39

**Finding [proposed]:** A replacement job still needs the original offering's average level, but the credit description exposes only expiry.

**Verification (2026-10-27) [proposed]:** Partly confirmed. When a credit is consumed is implied (a credit-funded job omits `release`; SHR-006 allows one open shrine job or credit) but not stated. The reward-level and renewal gaps hold.

**Resolution needed [proposed]:** Define retention of offering provenance and reward level, when a credit is consumed, and whether repeated failure or abandonment renews its 30-day lifetime.

**References:** [Creation Shrine: Rules](gameplay/creation-shrine.md#rules); [Creation API: Shrine status](tech/creation-api.md#creating-a-peerling).

**Resolution:** —

### REVIEW-040: Several exact schemas remain descriptive rather than exact

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 40

**Finding [proposed]:** `concept` is an unspecified map; `appearance` is described as structured data but shown as a string; duplicate species types are not prohibited; and character-count units are only explicit for species names. Provenance requires models for every stage but has no distinct stats-and-moves model field (`models.concept` arguably covers it, because the concept LLM also writes stats and moves). Only the character-count units can make clients disagree on whether a signed record is valid.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. Concept fields are listed in the pipeline but without types, lengths or extra-field rules; `appearance` is a string in every example; duplicate types are not forbidden (and would multiply to 4× under TYP-003); the stats-and-moves model is ambiguous rather than missing. The server signs these records and clients fetch exact bytes, so only the character-count point can make clients disagree. Wording corrected.

**Resolution needed [proposed]:** Provide concrete validation rules and reconcile the schemas and examples, including provenance for every required stage.

**References:** [Creation pipeline: Concept](peerlings/creation-pipeline.md#stage-2--concept); [Data formats: Species record](tech/data-formats.md#species-record--peerlingsspecies); [Image prompting: Input object](peerlings/image-prompting.md#input-object).

**Resolution:** —

### REVIEW-041: Network fallback policy leaves UDP-blocked environments unresolved

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 41

**Finding [proposed]:** The spec identifies blocked UDP as a WebTransport failure case, while its WebRTC-direct fallback is also described as UDP-based. HTTP/3 is mandatory for the API, with no stated HTTP fallback.

**Verification (2026-10-27) [proposed]:** Partly confirmed. Not mentioned in the finding: blocks can still come from trustless HTTP gateways (NODE-006), and D-0011 accepts that the game may not run everywhere, without naming UDP-blocked networks. Browsers also fall back from HTTP/3 to HTTP/2 on their own.

**Resolution needed [proposed]:** Clarify whether these environments are unsupported or which permitted path serves them.

**References:** [Tech stack: Networking](tech/tech-stack.md#networking); [Network performance: What can be slow](tech/network-performance.md#what-can-be-slow); [Creation API: General rules](tech/creation-api.md#general-rules).

**Resolution:** —

### REVIEW-042: Unlimited ownership and permanent retention need a storage-exhaustion policy

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 42

**Finding [proposed]:** Collections are unlimited, owned and created species must never be evicted, and the cache is described as capped at 1 GB. Retention is per species, so growth is bounded by the number of distinct species owned.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from High. Retention is per species, not per Peerling (about 1,500 typical species fill 1 GB). Whether exempt content counts toward the cap and what happens on a quota error remain unspecified. Wording corrected.

**Resolution needed [proposed]:** Clarify whether pinned ownership content sits outside that cap and what happens when browser quota is exhausted during a catch, trade, or creation.

**References:** [Tech stack: Asset budgets](tech/tech-stack.md#asset-budgets); [IPFS node: Retention requirements](tech/ipfs-helia.md#requirements); [Catching: Team and collection](gameplay/catching.md#team-and-collection).

**Resolution:** —

### REVIEW-043: Profile links can expire despite continuous server republication

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 43

**Finding [proposed]:** IPNS records have 30-day validity, while the server republishes an existing player-signed record without holding the player's key. Republishing does not extend its signed expiry, yet the sharing page promises that "every player has a permanent link".

**Verification (2026-10-27) [proposed]:** Confirmed. The sharing page's summary promises a permanent link, and its player-profile section implies that resolution ends only if both the server and the player stop. Viewer behaviour after expiry is unspecified. Wording and references corrected.

**Resolution needed [proposed]:** Specify what users see after an absent player's record expires, and qualify the promise of permanent profile links.

**References:** [Sharing: summary](gameplay/sharing.md); [Data formats: Player profile document](tech/data-formats.md#player-profile-document--peerlingsprofile); [Sharing: Player profile](gameplay/sharing.md#player-profile).

**Resolution:** —

### REVIEW-045: Creator statistics claim stronger verification than source events support

- **Status:** Open
- **Severity:** Medium
- **Original review item:** 45

**Finding [proposed]:** Non-catch encounters lack the evidence needed to replay them, yet encounter counts are described as resistant to fake clients. The counting unit for a trade containing multiple instances of one species and the effect of later transfer conflicts are also unspecified.

**Verification (2026-10-27) [proposed]:** Confirmed. `battle-result` has no tile, so the species of a non-catch encounter cannot be checked against the candidate list; only the gap-free and epoch checks apply.

**Resolution needed [proposed]:** Define how non-catch encounters are validated, whether trades increment once per transaction or per instance, and when conflicting transfers reverse counters.

**References:** [Creator feedback: Feasibility](gameplay/creator-feedback.md#is-it-possible-with-ipfs); [Species stats snapshot](gameplay/creator-feedback.md#species-stats-snapshot); [Data formats: Save-log events](tech/data-formats.md#save-log-events).

**Resolution:** Partly addressed by [D-0027](decisions/D-0027-high-severity-review-fixes.md): `encounter` events now log the tile, so a non-catch encounter's species can be checked. Trade counting and conflict reversal remain open.

## Low severity

### REVIEW-023: The guaranteed first wild encounter conflicts with guardians consuming encounter zero

- **Status:** Open
- **Severity:** Low
- **Original review item:** 23

**Finding [proposed]:** A player can reach a guardian before entering foliage, and guardian battles consume the next encounter number. Under EXP-011 the first-encounter guarantee then silently lapses, so the tutorial's promise of an ordinary first wild encounter is broken.

**Verification (2026-10-27) [proposed]:** Partly confirmed. The ordering is possible (the Plains guardian is about 350 m from the centre, beyond the 300 m spawn hexagon, and paths mostly avoid foliage), but EXP-011 already decides the outcome: the guarantee lasts until encounter 0 is logged, so it silently lapses. Wording corrected.

**Resolution needed [proposed]:** Decide whether the guarantee should survive a guardian battle, or prevent that ordering.

**References:** [Exploration: Wild encounters in foliage](gameplay/exploration.md#wild-encounters-in-foliage); [Guardians: Badges and logging](gameplay/guardians.md#badges).

**Resolution:** —

### REVIEW-024: Encounter prefetching needs invalidation rules

- **Status:** Open
- **Severity:** Low
- **Original review item:** 24

**Finding [proposed]:** Three upcoming encounters are prefetched, but each earlier encounter can change the seen set and therefore later candidate weights. No recomputation trigger is stated for that case; epoch, day and biome changes are already covered.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from Medium. Candidates are worked out at trigger time (ENC-005), a new epoch record already triggers recomputation for the current and neighbouring biomes, and assets are content-addressed. Only the seen-set dependency lacks a recompute trigger, and a stale prediction only costs a cache miss. Wording corrected.

**Resolution needed [proposed]:** Specify recomputation of predicted candidates after a first sighting.

**References:** [Encounters: Selection](gameplay/encounters.md#selection); [Latency](gameplay/encounters.md#latency).

**Resolution:** —

### REVIEW-030: Movement and interaction rules have uncovered boundary cases

- **Status:** Open
- **Severity:** Low
- **Original review item:** 30

**Finding [proposed]:** "Within 3 metres" is defined as a neighbouring tile without saying whether diagonal tiles (2.83 m) count, yet interaction only targets the cardinal tile being faced. Presence validation also rejects implausible jumps, yet legitimate defeat respawns and generator relocations can jump long distances without a corresponding presence-message flag.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from Medium. "Neighbouring tile" never says 4 or 8 neighbours; the diagonal reading was the reviewer's inference. Presence is not authoritative, so the worst case is a player who is briefly invisible to others (removed after 15 s). Wording corrected.

**Resolution needed [proposed]:** Define adjacency precisely and specify how legitimate relocation passes presence validation.

**References:** [Multiplayer: What players experience](gameplay/multiplayer.md#what-players-experience); [Realtime networking requirements](tech/realtime-networking.md#requirements); [Protocols: Presence](tech/protocols.md#peerlingsv1presencerx_ry--presence).

**Resolution:** —

### REVIEW-032: Enhancer invocation rules conflict

- **Status:** Open
- **Severity:** Low
- **Original review item:** 32

**Finding [proposed]:** Accepted `IMG-001` requires every new image request to pass through GPT-6 Luna. Proposed regeneration reuses an existing enhanced description, and proposed failure handling bypasses the enhancer altogether.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from Medium. The enhancer-failure fallback is a real bypass and is also written into accepted D-0025; regeneration reusing the enhanced description arguably still goes through the enhancer. Image-check retries are a third reuse case. Every exception is written down, so only IMG-001's wording needs aligning.

**Resolution needed [proposed]:** Define the intended exceptions explicitly.

**References:** [Image prompting: Pipeline placement](peerlings/image-prompting.md#where-it-sits-in-the-pipeline); [Failure handling](peerlings/image-prompting.md#failure-handling); [Requirements](peerlings/image-prompting.md#requirements).

**Resolution:** —

### REVIEW-034: Image-quality checks contradict their own requirement

- **Status:** Open
- **Severity:** Low
- **Original review item:** 34

**Finding [proposed]:** Proposed `IMG-013` requires every image to pass the checks before use, but the failure flow shows an image anyway after retries.

**Verification (2026-10-27) [proposed]:** Confirmed; severity lowered from Medium. IMG-013 and the failure flow are both `[proposed]` on a draft page, so a proposal contradicts itself; only a missing alpha channel is repaired, and the player can regenerate.

**Resolution needed [proposed]:** Specify whether the player can approve failed framing, disconnected geometry, or other failed checks, and which failures are hard blockers.

**References:** [Image prompting: Image checks and clean-up](peerlings/image-prompting.md#image-checks-and-clean-up); [Requirements](peerlings/image-prompting.md#requirements).

**Resolution:** —

### REVIEW-044: Phone-backup entry into the Viewer needs clarification

- **Status:** Open
- **Severity:** Low
- **Original review item:** 44

**Finding [proposed]:** The QR payload is a custom `peerlings-backup:` URI rather than an HTTPS Viewer URL. The pages imply opening the Viewer first and scanning inside it, but a scan with the phone's normal camera launches nothing (a web app can only register `web+` schemes), and the computer is not required to tell the player where the Viewer is.

**Verification (2026-10-27) [proposed]:** Partly confirmed. Read together, the pages imply opening the Viewer first and scanning inside it, which is fairly clear. The real gaps are that a normal camera scan of the custom URI launches nothing and that the player is never shown the Viewer's address. Wording corrected.

**Resolution needed [proposed]:** Specify whether users first open the Viewer and use its scanner, or whether scanning with the phone's normal camera should launch it.

**References:** [Protocols: Phone backup](tech/protocols.md#peerlingsphone-backup100--phone-backup); [Player data: Phone backup](tech/player-data.md#phone-backup).

**Resolution:** —

### REVIEW-046: First-finder finalization needs a collection window

- **Status:** Open
- **Severity:** Low
- **Original review item:** 46

**Finding [proposed]:** Lowest epoch, then lowest catch CID, and final once announced are compatible only after defining the set of catches considered before announcement. The timing determines whether arrival order or catch epoch actually decides the winner.

**Verification (2026-10-27) [proposed]:** Partly confirmed; severity lowered from Medium. `firstWild` lives in the hourly stats snapshot and announcements happen once the stats name a finder, so the snapshot is effectively the naming point. Only the set of catches it considers is undefined, and only the server decides.

**Resolution needed [proposed]:** Specify whether selection happens immediately, at an epoch boundary, or during the hourly snapshot.

**References:** [Creator feedback: First found in the wild](gameplay/creator-feedback.md#first-found-in-the-wild).

**Resolution:** —

### REVIEW-047: Examples and historical summaries contradict current rules

- **Status:** Open
- **Severity:** Low
- **Original review item:** 47

**Finding [proposed]:** The species JSON example totals 250 base-stat points, violating `SPC-007`'s 320. D-0006 calls instance data private to the owner, although saves are publicly replicated, and still describes only created starters. D-0025 retains a consequence saying provenance contains three prompts after D-0026 removed those views.

**Verification (2026-10-27) [proposed]:** Confirmed: the example totals 80 + 55 + 70 + 45 = 250. Also stale: D-0006's starter statement has no note about D-0012, and D-0025 keeps a bullet about the three-view camera layout and silhouette check. Decision records changed in part are expected to carry notes (AGENTS.md §7), so these are real defects.

**Resolution needed [proposed]:** Correct the invalid example and clearly mark or update superseded details wherever they remain.

**References:** [Peerling species: Example](peerlings/peerling-species.md#species-record); [D-0006](decisions/D-0006-species-vs-instance.md); [D-0025](decisions/D-0025-prompt-enhancer-and-three-views.md); [D-0026](decisions/D-0026-single-image-to-3d.md).

**Resolution:** —

## See also

- [Wiki index](index.md)
- [Open questions](open-questions.md)
- [Change log](log.md)

