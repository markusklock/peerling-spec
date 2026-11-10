---
title: "Player Data: Saves, Identity and Ownership"
type: system
status: accepted
req_prefix: SAVE
tags: [tech, saves, identity, orbitdb, ipns, security]
sources:
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-save-recovery.md
  - raw/conversations/2026-10-05-peer-save-backups.md
  - raw/conversations/2026-10-05-peer-save-backups-approved.md
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
  - raw/conversations/2026-10-05-phone-backup-approved.md
  - raw/conversations/2026-10-05-world-details.md
  - raw/conversations/2026-10-05-world-details-approved.md
  - raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-proposals-approved.md
  - raw/conversations/2026-10-06-v1-fun-features.md
  - raw/conversations/2026-10-06-fun-features-approved.md
  - raw/conversations/2026-10-06-network-performance-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-06-review-2-decisions.md
  - raw/conversations/2026-10-27-high-severity-resolutions.md
  - raw/conversations/2026-10-28-medium-low-resolutions.md
  - raw/conversations/2026-11-10-second-review-resolutions.md
related:
  - wiki/decisions/D-0009-player-data-on-orbitdb.md
  - wiki/decisions/D-0013-peer-verified-registry-catches-trades.md
  - wiki/gameplay/player-character.md
  - wiki/gameplay/catching.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/peerlings/peerling-species.md
  - wiki/tech/architecture.md
  - wiki/tech/orbitdb-registry.md
  - wiki/decisions/D-0027-high-severity-review-fixes.md
  - wiki/decisions/D-0029-second-review-fixes.md
updated: 2026-11-10
---

# Player Data: Saves, Identity and Ownership

> Canonical home for what a player's save contains, where it is stored (a
> per-player OrbitDB log), how the identity key is recovered, and how any player
> can verify another player's Peerlings and trades without a central server.
> Decided in [D-0009](../decisions/D-0009-player-data-on-orbitdb.md) and
> [D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md).

## Chosen design

[accepted] ([D-0009](../decisions/D-0009-player-data-on-orbitdb.md),
[D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md))

| Concern | Solution |
|---------|----------|
| Where the save lives | A per-player OrbitDB **save log** (an append-only event log) that only the player's identity can write. The server and other players replicate it. |
| Recovery | The identity key can be restored with a **recovery phrase** |
| Faked Peerlings | Anyone verifies a catch by **replaying the battle** from the public catch evidence |
| Duplication via trades | Ownership is a signed **transfer chain** in an open OrbitDB **transfer log**; double trades are detected and the cheater is flagged |
| Edited levels in PvP | PvP's default **Fair** mode uses level 50 for everyone; the opt-in Real-levels mode is unverified ([D-0015](../decisions/D-0015-pvp-level-modes.md)) |
| What can be traded or used in PvP | Only **verified** Peerlings |

## Save contents

The save holds everything about the player's progress. Species data (art, 3D
model, stats, moves) is **not** in the save; it lives on IPFS and the save
refers to it by CID.

| Part | Contents | Provenance |
|------|----------|------------|
| Collection | Every Peerling the player owns, each with its **current level**, XP, current HP, nickname, origin and verification. Format: [data-formats § Peerling instance](data-formats.md#peerling-instance) | [accepted] |
| Team | Ordered list of the instance IDs in the active team (size: [catching](../gameplay/catching.md)) | [accepted] |
| Profile | Player ID (public key), display name, character appearance | [accepted] |
| Created species | CID(s) of the species this player created | [accepted] |
| Peerdex | Species seen and species caught (CIDs) | [accepted] |
| Position | Last position and facing in the world | [accepted] |
| Map | Chunks the player has explored, as a bit set ([exploration § Map](../gameplay/exploration.md#map)) | [accepted] |
| Inventory | [accepted] No battle items in the first version ([CAT-003](../gameplay/catching.md#requirements)). [accepted] No inventory at all in the first version, so this part is empty | [accepted] |

Not in the save: the identity **private key** (stays on the device; restored
with the recovery phrase), and the authoritative owner of each Peerling (that is
the [transfer log](#transfer-log-and-trades)).

## Save log

[accepted] The save is stored as *events*, not as one file that gets
overwritten. The current save is what you get by applying all events in
order. OrbitDB *events* databases are append-only and every entry is signed by
the writer, so the log is also a tamper-evident history.

Exact payloads: [data-formats § Save-log events](data-formats.md#save-log-events).
That page holds the complete event list; this table says when each event is
written.

| Event | Written when | Payload |
|-------|-------------|---------|
| `profile` | Onboarding; profile changes | display name, appearance |
| `species-created` | Creation pipeline finished | species CID |
| `starter` | Onboarding finished | the starter instance + the server's origin attestation |
| `release` | Peerlings offered at the [Creation Shrine](../gameplay/creation-shrine.md) | instance IDs, transfer-log entry reference |
| `created` | A shrine creation was published | the new instance + the server's origin attestation |
| `catch` | A wild Peerling is caught | the new instance + **catch evidence** (see below) |
| `encounter` | A wild or guardian battle starts | encounter number, epoch used, tile, and the candidate and species met (or the guardian's biome) |
| `battle-result` | A wild or guardian battle ends | outcome; for each participating instance: XP gained, new level, HP |
| `team` | Team changed | ordered instance IDs |
| `session-start` | The game starts on a computer | device ID ([Using the same account on several computers](#using-the-same-account-on-several-computers)) |
| `nickname` | Peerling renamed | instance ID, nickname |
| `rest` | The player uses a rest point | the rest point's tile |
| `badge` | The first win against a biome's guardian, before that battle's `battle-result` ([guardians § Badges](../gameplay/guardians.md#badges)) | biome index, guardian evidence |
| `pvp-result` | A PvP battle ends, with any result ([PvP](#pvp)) | battle ID, players, mode, result and the signatures that prove it |
| `generator` | The client starts using a world-generator version: before a new player's first step, and whenever it switches to a newer one ([procedural-generation § Generator updates](../world/procedural-generation.md#generator-updates)) | the version number |
| `trade-pending` | Just before the client sends its signed transfers in a trade ([Transfer log and trades](#transfer-log-and-trades)) | trade ID, partner, instances out, own signed transfers, the other side's offer data |
| `trade` | A trade entry was written to the transfer log | transfer-log entry reference; instances out; full data of instances in |
| `trade-void` | A trade the player took part in turned out void ([Transfer log and trades](#transfer-log-and-trades)) | the void entry; own Peerlings restored; received Peerlings removed |
| `seen` | First sighting of a species, and first sighting as a shimmer | species CID, biome, shimmer flag |
| `position` | Every 30 s while moving, and on exit | position, facing |
| `explored` | The player enters a chunk for the first time | the newly revealed chunks ([exploration § Map](../gameplay/exploration.md#map)) |
| `snapshot` | Every 50 events, at every rest point visit, and on exit | CID of a DAG-CBOR document with the full current save, and the last event it includes |
| `checkpoint` | When the server has a newer verification checkpoint for this player (checked at session start and every hour) | the server-signed checkpoint |

Loading a save means reading the latest `snapshot` and applying the events
after it. The server replicates every player's log and pins the snapshots.

[accepted] **Events are the save; snapshots are caches** (2026-10-28,
[D-0028](../decisions/D-0028-medium-low-review-fixes.md)). Every change to the save comes from an event:
- a `rest` event heals the whole team and makes that rest point the last rest
  point ([exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points));
- a `battle-result` with outcome `lost` (wild or guardian) heals the whole
  team and moves the player to the last rest point;
- otherwise HP comes from the `team` field of each `battle-result`.

A snapshot holds the result of applying every event up to its `upTo` entry.
Readers may load from it, but if it ever differs from the events, the events
win. The server computes a checkpoint's `state` from the events.
Other players fetch a save log when they need to verify one of its catches.

[accepted] **What each event does to the save** (2026-11-10,
[D-0029](../decisions/D-0029-second-review-fixes.md)). The client and the
server must build the same snapshot from the same events, so every effect is
exact. Fields are those of the
[save snapshot](data-formats.md#save-snapshot--peerlingssave).

*Before any event:* no Peerlings, an empty team and Peerdex, no badges, PvP
counters 0, `nextEncounter` 0, `position` and `lastRestPoint` as for a new
player ([exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points)),
and `explored` holding the chunks revealed from the start
([exploration § Map](../gameplay/exploration.md#map)).

| Event | Effect |
|-------|--------|
| `profile` | Sets `profile` |
| `generator` | Sets `generator`. If the version is newer than the one before, `position` and `lastRestPoint` are moved by the "nearest" rule of [procedural-generation § Generator updates](../world/procedural-generation.md#generator-updates) |
| `species-created` | Adds the CID to `created` |
| `starter`, `created`, `catch`, incoming `trade` | Adds each instance to `instances`. It joins the **end of the team** if the team holds fewer than 4 Peerlings, otherwise it is in the collection only. A `catch` also marks the species as caught in the Peerdex |
| `release`, outgoing `trade` | Removes the instances from `instances` and from the team; the other team members keep their order |
| `team` | Sets the team to the listed order |
| `nickname` | Sets that instance's nickname |
| `encounter` | Sets `nextEncounter` to its number + 1 |
| `battle-result` | For each listed instance: adds `xpGained`, applies level-ups ([battle § Experience and levelling](../gameplay/battle.md#experience-and-levelling)) and sets its HP. Outcome `lost`: the whole team is set to full HP and `position` becomes the respawn position of `lastRestPoint` |
| `rest` | Sets the whole team to full HP, sets `lastRestPoint`, and reveals the chunk of that area's landmark in `explored` |
| `seen` | Adds the species to the Peerdex `seen` list with its biome, or sets its `shimmerSeen` |
| `position` | Sets `position` |
| `explored` | Sets the listed chunks in `explored` |
| `badge` | Adds the biome index to `badges` |
| `pvp-result` | Updates the `pvp` counters, if the result counts ([PvP](#pvp)) |
| `trade-pending` | Locks its outgoing instances until a `trade` or `trade-void` event for the same trade follows, or the trade expires ([Transfer log and trades](#transfer-log-and-trades)). Locks aren't part of the snapshot; a client works them out from the events |
| `trade-void` | Removes the `removed` instances (also from the team) and adds the `restored` ones like received ones; then the team is refilled as in [catching § Team and collection](../gameplay/catching.md#team-and-collection) |
| `session-start`, `snapshot`, `checkpoint` | No effect on the snapshot fields |

## Account recovery

[accepted] An account is a cryptographic key pair; the public key is the
player ID. The save is a per-player OrbitDB log that the server replicates and
pins (SAVE-001). The key is recovered with the **recovery phrase** or a
**backup file**; there is no password-based recovery (decided 2026-10-05).

### Getting the key back

| Method | How | Strength |
|--------|-----|----------|
| **Recovery phrase** | 12 random words shown at onboarding; the key is derived from them. Nothing secret is stored on any server. [accepted] The device keeps the phrase's entropy next to the key, and backups carry it, so *Backup → Show recovery phrase* works on every device the account is on (2026-10-28) | Very strong: the words are random |
| **Backup file** | Export or import the key as a file | Strong, if the file is kept safe |
| **Phone backup** | Scan a QR code with your phone; it carries the key and the save ([Phone backup](#phone-backup)) | Strong; whoever has the phone has the account |

### Phone backup

[accepted] Players can back up their account to their phone by scanning a QR
code. The phone keeps a copy of the save, which can be imported on another
computer or after clearing browser data. The phone is only a backup carrier;
the game isn't played on it (designer, 2026-10-05; replaces the earlier
"link a device" idea).

[accepted] Approved 2026-10-05.
- **On the phone:** the phone opens the [Peerlings Viewer](../gameplay/sharing.md)
  (the small web app published on IPFS), which has a **Backup** section. It
  runs its own Helia node in the phone's browser, so the phone is another IPFS
  node for a moment. It can be installed on the home screen.
- **Making a backup:** on the computer, *Backup → Back up to phone* (in the
  pause menu, [ui § Pause menu](../gameplay/ui.md#pause-menu-esc)) shows a QR
  code with the computer's peer ID, a relay address and a one-time secret
  (32 random bytes, valid for 5 minutes). The phone scans it with its camera,
  connects to the computer over libp2p (WebRTC, through a relay if needed) and
  proves it knows the secret. The computer asks the player to confirm, then
  sends the **identity key** and the **save** (the latest save snapshot, packed
  as a CAR file, the IPFS archive format that keeps every CID intact,
  together with the save log and the proofs of the player's Peerlings; exact
  contents: [data-formats § Backup file](data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup)),
  encrypted with a key derived from the secret. The phone stores it in its
  browser storage and shows the date of the backup.
- **Restoring:** on the new (or wiped) computer, *Recover account → Restore from
  phone* shows a QR code of the same kind. The phone scans it, connects, and
  sends the key and save. The computer imports both, then syncs the save log
  from the network to pick up anything newer than the backup.
- **Why the phone always scans:** phones have cameras, desktop computers often
  don't, so the computer only ever *shows* QR codes.
- **Keeping it fresh:** scanning again replaces the backup. The game reminds the
  player now and then (e.g. after a level-up milestone).
- **Security:** the QR code carries a full-strength random secret and is only
  shown on the player's own screen, so nobody else can join the transfer. The
  backup on the phone contains the private key, so the Viewer shows a warning
  that whoever has the phone has the account, and offers to delete the backup.

### Using the same account on several computers

[accepted] A restored account can end up on two computers at once (the old one
still has it). Playing on both at the same time would create duplicate
encounter numbers, which verification treats as a forked save
([Verified Peerlings](#verified-peerlings)). So a device appends a
`session-start` event when the game starts and, while playing, publishes a
heartbeat every 15 s on its own session topic
([protocols](protocols.md#peerlingsv1sessionplayer-id--active-session)).
Another computer with the same account first syncs the save log and listens on
that topic; if a heartbeat has arrived within the last 60 s, a session is
active elsewhere and it refuses to start play (*"You're playing on another
computer"*). A crashed computer's session simply stops sending heartbeats and
counts as ended after 60 s. Two computers playing while both are offline
can't be detected in time; restoring shows a warning about this.

[accepted] **Starting and competing sessions** (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md);
messages in [protocols](protocols.md#peerlingsv1sessionplayer-id--active-session)):
- Before starting play, the client also publishes a `session-probe` on that
  topic. An active session answers at once with a heartbeat, so the client
  waits only 3 s for an answer instead of a full heartbeat interval.
- Every heartbeat carries the time its session started (`since`). If a playing
  client receives a heartbeat from another device of the same account, the
  session that started earlier keeps playing (on a tie, the lower device ID).
  The other stops at once, writes no further events and shows *"You're playing
  on another computer"*.
- What remains is two computers starting within the same few seconds, or
  playing while offline. A fork there is an accepted risk.

### Finding the save again

[accepted] The save log's OrbitDB address is computed from the player ID alone
(fixed database name, type and writer), so a recovered key leads straight to
its save log; no central account directory is needed. On a new device the
player chooses *Recover account*, enters the phrase or loads the file, and the
client fetches the save log from whichever node holds it and loads the latest
snapshot.

If no node holds the save any more, the key still works as an identity: every
species the player created still names it as creator. The progress can't be
rebuilt, and the key can't start a new save either: the save log's address is
fixed by the player ID ([SAVE-018](#requirements)), so a fresh log would clash
with any old copy that turns up later, and the player has already had their one
starter. [accepted] So (2026-10-27, designer's choice, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)) the client
explains that the progress is lost and offers two choices:
- **Keep looking:** try the server and the `save-wanted` request again later.
- **Start a new account:** a new key and recovery phrase, so a new player ID
  and a new starter, as for any new player. The old key isn't used to play
  again. The client MAY keep the old player ID to show the
  [creator notifications](../gameplay/creator-feedback.md) of the species it
  created.

### Who actually holds a save

IPFS and OrbitDB nodes only hold what they have asked for, so a save log
exists only where something has a reason to fetch it:

| Holder | Why it has the save | How reliable |
|--------|---------------------|--------------|
| **The operator server** | Replicates and pins every save log (SAVE-001) | The main copy; always online, but a single machine |
| **The player's own browsers** | Every device the player plays on keeps its own log | Lost if browser data is cleared |
| **Other players' backups** | Inspecting a player, trading or battling keeps a backup of their save ([Keeping saves available](#keeping-saves-available)) | Sporadic; only while the holder's game is open |
| **Community mirrors** | Not expected (designer, 2026-10-05) | Not relied on |

So in practice the operator server is the only dependable network copy. Ways
to make saves more durable are in [Keeping saves available](#keeping-saves-available).

### Keeping saves available

[accepted] Community mirrors are not expected, so nothing relies on them
(2026-10-05). [accepted] Two complementary measures (approved 2026-10-05):

**1. Peer backups through profile inspection** (the designer's idea)

- **Inspecting fetches the save.** When a player inspects a nearby player
  ([multiplayer](../gameplay/multiplayer.md): stand on a neighbouring tile,
  face them, choose *View profile*), the profile screen appears at once from
  the profile stream ([protocols § profile](protocols.md#peerlingsprofile100--profile)),
  and the client then fetches that player's **full save log** (the latest
  snapshot plus the events after it) in the background
  ([network-performance](network-performance.md#operation-by-operation)). The
  verification marks on their team fill in once it has arrived.
- **Trades and PvP fetch it too.** Verifying a trade partner's or opponent's
  Peerlings already fetches their save log; that copy is kept the same way.
- **Kept as a backup.** The client stores these logs persistently, apart from
  the normal content cache, up to **100 other players or 100 MB**, whichever
  comes first, dropping the save it has gone longest without refreshing. Each
  backup is refreshed whenever that player is inspected, traded with or battled
  again.
- **Tamper-proof.** Every save-log entry is signed by its owner, so a backup
  holder can't change anyone's save.
- **Serving it back.** A player recovering their account publishes a request on
  the pubsub topic `peerlings/v1/save-wanted` with their player ID. Every online
  client holding a backup of that save answers over the direct protocol
  `/peerlings/save-backup/1.0.0` with its snapshot CID and log heads. The
  recovering client fetches them, merges them with the server's copy (OrbitDB
  merges logs automatically), and continues from the newest state.
- **Honest limits:** backups are sporadic. A player who never meets anyone has
  none, and a holder can only help while their game is open. This is a
  safety net next to the server, not a replacement.

**2. The save in the backup file**

The backup file holds the key, the latest save snapshot and [accepted]
(2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)) the whole save log plus the proofs of every Peerling
the player owns (exact contents:
[data-formats § Backup file](data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup)).
The phone backup carries the same file. The game reminds the player to refresh
it now and then (at rest points,
[exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points)).
This works even if every network copy of the save is gone:
- the save loads, and the signed log proves the snapshot is the player's own;
- the player's own catches, starters, creations, badges and PvP results can be
  verified again by anyone the restored client shares the log with;
- every owned Peerling keeps its ownership chain. A Peerling received in a
  trade is verified against its original owner's save log, so its origin can
  be checked again once that log is reachable (from the server, the original
  owner or a peer backup).

Not recommended: buddy pinning of assigned saves (browsers are offline most of
the time), and paid pinning or Filecoin storage (costs money).

## Verification

### Verified Peerlings

[accepted] A Peerling is **verified** when anyone can check two things
([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)):
1. **Its origin is genuine:** a catch that replays correctly, or a starter or
   shrine creation with a valid server origin attestation (below).
2. **Its ownership chain is valid:** every transfer since its origin is
   signed by the owner at the time, and it ends at the player who holds it
   ([Transfer log and trades](#transfer-log-and-trades)).

Who checks: [accepted] a trade partner before trading, a PvP opponent before
battling, and the Creation Shrine before accepting an offering. The client
caches results per Peerling and checks only the new part of an ownership chain
later. The server also verifies catches it sees, for the species stats
([creator-feedback](../gameplay/creator-feedback.md)).

[accepted] **Results** (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)). A check gives one of three
results:

| Result | When | Effect |
|--------|------|--------|
| **Verified** | Every check passed | Usable in trades and PvP |
| **Invalid** | A check failed on complete data: the verifier had every save-log entry in the causal history of the entry it checked (back to the start of the log, or to the `upTo` of the checkpoint it uses), every block the checks need, and a rules version it supports. A server checkpoint's `invalid` list also counts | Never usable in trades or PvP; it stays in its owner's collection |
| **Pending** | Something is missing: a log entry or block that can't be fetched yet, an epoch record, or a rules version newer than the verifier's game (the player is then asked to reload the game) | Not usable for now; checked again the next time it is needed. The UI says *"This Peerling can't be checked right now"* |

Verified and invalid results are cached by entry CID. A cached verified catch
is checked again only for what can still change: save-log entries the verifier
didn't have before (a fork shows up as duplicate encounter numbers) and new
transfers.

[accepted] **Forks** (2026-11-10, designer's choice,
[D-0029](../decisions/D-0029-second-review-fixes.md)). A save log is forked
when two different `encounter` entries carry the same encounter number. Let
*d* be the lowest encounter number that appears more than once in the entries
a verifier has.
- Every catch and badge of that log with an encounter number of *d* or higher
  is invalid, in every branch. Catches and badges with a lower number are not
  affected.
- **A checkpoint protects what it covers.** A catch or badge whose entry is in
  the causal history of the `upTo` of any valid server checkpoint for that
  player, and that isn't on that checkpoint's `invalid` list, stays verified
  whatever is appended or found later, even if *d* is at or below its number.
  So once the server has checked a catch, its catcher can no longer make it
  invalid by signing a new branch.
- The server keeps to this itself: a later checkpoint never lists as
  `invalid` a Peerling that an earlier checkpoint for the same player covered
  as valid.
- What remains is a catch no checkpoint covers yet; see
  [Known gaps](#known-gaps-accepted-risks).

[accepted] **Verification checkpoints** (2026-10-06,
[D-0023](../decisions/D-0023-network-performance.md)): the server signs a
checkpoint whenever it has verified a save log up to some entry, and the
player's client appends its newest checkpoint to its own log as a `checkpoint`
event; the checkpoint's `player` must be the log's owner. A verifier accepts the newest checkpoint for everything up to its
`upTo` entry and checks only the entries after it (those not in the causal history of
`upTo`, so a forked log is still caught), so verifying a long-time
player takes seconds, not minutes. Peerlings listed as `invalid` in it fail.
This trusts the operator's signature for the older part of the log, as players
already trust it for species listings; without a checkpoint, verifiers check
the whole log ([network-performance § Verification checkpoints](network-performance.md#5-verification-checkpoints)).

### Catches

1. When a player catches a Peerling, the client appends a `catch` event with
   the **catch evidence**: the encounter number, the epoch record used, the
   tile where it started, which of the encounter's candidates was met
   ([encounters § Candidates](../gameplay/encounters.md#candidates)), the
   battle's starting state, and every action taken. The Peerling can be used
   straight away in exploration and wild battles.
2. [accepted] The instance ID of a caught Peerling is
   SHA-256(player ID ‖ encounter number), so it is unique automatically and
   can't be reused.
3. Any verifier fetches the catcher's save log, replays the battle with the
   deterministic engine ([BTL-002](../gameplay/battle.md#requirements)), and runs
   the checks in [Encounter seeds](#encounter-seeds). The result is the same
   for every verifier that has the same data. A catch that is
   [invalid](#verified-peerlings) stays invalid: the Peerling stays in its
   owner's collection but can never be traded or used in PvP. A *pending*
   check is tried again later.

[accepted] **Guardian badges** are verified the same way: the verifier
recomputes the week's guardian team from the `weekRecord`, replays the battle
from the `badge` event's evidence, and checks the encounter-seed rules
([guardians](../gameplay/guardians.md#badges)). [accepted] Exactly
(2026-11-10, [D-0029](../decisions/D-0029-second-review-fixes.md)), a badge is
valid when, on complete data:
1. checks 1, 2 and 4 of [What a verifier checks for a catch](#encounter-seeds)
   pass, with the `weekRecord` checked like the day record;
2. the `encounter` event with the evidence's number has `guardian` equal to
   the badge's `biome`, and the same epoch and tile as the evidence;
3. the tile is that biome's statue tile in the generator version of the
   evidence's epoch record;
4. the guardian team and level are the ones the week record and that tile
   give ([guardians § The weekly team](../gameplay/guardians.md#the-weekly-team));
5. replaying the battle with the logged actions ends with every guardian
   Peerling fainted.

### Encounter seeds

[accepted] Approved 2026-10-04, including drand as the randomness source.

Goal: players can't choose or re-roll their wild encounters, play keeps working
without the server, and the server can check everything afterwards.

**Epochs and the epoch record.**
- Time is divided into 5-minute **epochs**: epoch number E = floor(Unix time in
  seconds ÷ 300).
- At the start of each epoch the server publishes a signed
  [epoch record](../glossary.md#epoch-record) (illustrative; exact format:
  [data-formats § Epoch record](data-formats.md#epoch-record--peerlingsepoch)):

  ```json
  {
    "epoch": 5873210,
    "drandRound": 1234567,
    "randomness": "<32 bytes, hex>",
    "drandSignature": "<hex>",
    "registryHeight": 1842,
    "generator": { "version": 2, "fromEpoch": 5873400 }
  }
  ```

- `randomness` is taken from **[drand](../glossary.md#drand)**, the public
  randomness beacon run by the League of Entropy (Protocol Labs, the company
  behind IPFS, is one of its main contributors). drand values can be verified
  with drand's public key, so clients don't have to trust the server not to
  bias the randomness. Clients get drand values through the server's epoch
  record and verify the drand signature locally, so they normally never
  contact drand directly.
- [accepted] **The drand network** (2026-10-06) is **quicknet**:

  | Parameter | Value |
  |-----------|-------|
  | Chain hash | `52db9ba70e0cc0f6eaf7803dd07447a1f5477735fd3f661792ba94600c84e971` |
  | Scheme | `bls-unchained-g1-rfc9380` (each round is verifiable on its own) |
  | Period | 3 s |
  | Genesis time | 1692803367 (Unix seconds) |
  | Public key | `83cf0f2896adee7eb8b5f01fcad3912212c437e0073e911fb90022d3e760183c8c4b450b6a0a6c3ac6a5776a2d1064510d1fec758c921cc22b0e17e63aaf4bcb5ed66304de9cf809bd274ca73bab4af5a6e9c76a4bc09e76eae8991ef5ece45a` |

  The round for epoch E is **(E × 300 − 1692803367) ÷ 3 + 1**. It is a whole
  number, because both terms are divisible by 3, and that round is emitted
  exactly at the epoch's start. `randomness` is that round's randomness. There
  is no fallback where the server makes up its own value: nobody could verify
  it. If drand is down, no new epoch records appear and players keep using the
  last one, as in offline play.
- `generator` announces the current world-generator version and the epoch it
  takes effect ([procedural-generation § Generator updates](../world/procedural-generation.md#generator-updates)).
- `registryHeight` fixes which species are eligible during that epoch (see
  *Registry state* below).
- Distribution: live on the pubsub topic `peerlings/v1/epoch`
  ([realtime-networking](realtime-networking.md)), and in an OrbitDB
  **epoch log** (events database, written only by the server) for clients that
  were offline and for the server's own verification. That is 288 small entries
  per day.

**Seeds.**
- Every encounter has an **encounter number** n: 0 for the player's first
  encounter, increasing by exactly 1 for each encounter. *Every* encounter is
  logged in the save log, including ones the player flees from or loses
  (its `encounter` event carries n).
- encounter seed = SHA-256("peerlings/encounter/v1" ‖ epoch randomness ‖ player
  ID ‖ n).
- The seed drives the [random number generator](../gameplay/battle.md#random-number-generator)
  for the species choice, the wild level and the whole battle
  ([encounters](../gameplay/encounters.md)).
- An encounter uses the latest epoch record the client has when the encounter
  starts. Epoch numbers must never decrease from one encounter to the next.
- [accepted] (2026-11-10, [D-0029](../decisions/D-0029-second-review-fixes.md))
  That same record also fixes the world and the battle rules the client plays
  with: the client shows the world-generator version, and uses the rules
  version, that are active at the epoch of the latest epoch record it has,
  never the ones its clock suggests. So an encounter's tile is always checked
  against the world the player actually walked in
  ([procedural-generation § Generator updates](../world/procedural-generation.md#generator-updates)).

**Recording an encounter** ([accepted] 2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md); exact
events: [data-formats § Save-log events](data-formats.md#save-log-events)).
- When a wild or guardian battle starts (a candidate has arrived, or the
  player accepted a guardian challenge), the client appends an `encounter`
  event with n, the epoch used, the tile, and the candidate and species met (or
  the guardian's biome). This uses up n and fixes the tile and epoch, even if
  the game reloads.
- [accepted] (2026-11-10, [D-0029](../decisions/D-0029-second-review-fixes.md))
  If the epoch record or the day record the encounter uses is client-derived,
  the `encounter` event also names the epoch of its base record (`base`,
  `dayBase`). Every base record is server-signed and can be fetched by its
  epoch number, so the server and any other verifier can rebuild the derived
  record and check the species of an encounter that didn't end in a catch.
- A first sighting's `seen` event follows at once
  ([encounters § Selection](../gameplay/encounters.md#selection)).
- When the battle ends, the client appends the `catch` event (if caught) or
  the `badge` event (a first guardian win), then the `battle-result`, so the
  evidence is saved before the result.
- **Interrupted battles** (designer's choice). If the game closes or crashes
  during a battle, the next start finds an `encounter` with no
  `battle-result`. If a `catch` for n exists, or a `badge` whose evidence has
  the number n, the client replays its evidence and writes the matching
  `battle-result` (`caught` or `won`, with the XP the replay gives). Otherwise
  the battle counts as fled: the client writes a `battle-result` with outcome
  `fled`, no XP, and each Peerling's HP as it was when the battle started.
  Fleeing always succeeds anyway, so closing the game during a battle gains
  nothing.

**Why this stops re-rolling.**
- Same epoch + same n → same encounter, so reloading the page gives exactly the
  same encounter.
- Fleeing is a logged encounter, so n can't be skipped. A gap in the encounter
  numbers makes every later catch fail verification.
- The only way to get a different encounter for the same n is to wait for a new
  epoch before triggering it, which costs up to 5 minutes. See *Known gaps*.
- [accepted] This stops re-rolling the *same* encounter. It doesn't stop a
  player from moving on to the next one: fleeing always succeeds, so anyone can
  flee until a good trait or a shimmer turns up, and a modified client can do
  it without walking or battling (2026-10-28; see *Known gaps*).

**Registry state.**
- The server gives each registry entry a sequence number `seq` (1, 2, 3, …)
  when adding it; a takedown records `removedAtSeq`
  ([orbitdb-registry](orbitdb-registry.md)).
- [accepted] **Eligible at registry height H** (exact, 2026-10-27,
  [D-0027](../decisions/D-0027-high-severity-review-fixes.md)): a species is eligible if its listing has a `seq` ≤ H and it has
  no delisting with a `seq` ≤ H. Its **listing seq** is the `seq` of its
  original listing: a tombstone keeps it as `listedSeq`, and the registry index
  has the listing as its own entry. During epoch E, H = registryHeight(E).
  Every draw that sorts eligible species by registry `seq` (encounter
  candidates, the Peerling of the Day, guardian teams) sorts them by listing
  seq.
- [accepted] The [registry index](network-performance.md#1-registry-index) is
  authoritative for this: it holds every listing and delisting by `seq`, and
  the signed epoch record names it. A reader that uses the OrbitDB registry
  instead takes `listedSeq` and `removedAtSeq` from the tombstones, never just
  the current documents' `seq`.
- A client that doesn't yet know the registry up to that height doesn't start
  encounters until it does. [accepted] The compact registry index named in
  the epoch record is enough, so this takes a second or two
  ([network-performance § Registry index](network-performance.md#1-registry-index)).

**When the server is offline.** [accepted] If no new server-signed epoch
record has arrived for 10 minutes (two epochs), the client derives epoch
records itself (rules tightened 2026-10-06):
- `randomness` is the drand value for the epoch, fetched directly from public
  drand endpoints (run by League of Entropy members) and checked against
  drand's public key as usual.
- `registryHeight` (and the `generator` and `rules` versions and the snapshot
  roots) are copied from a server-signed epoch record, the **base record**.
  While the server is down nothing can be added to the
  registry (only the server can sign listings), so nothing is missed.
- [accepted] **Which base record** (2026-11-10,
  [D-0029](../decisions/D-0029-second-review-fixes.md)): for a derived record
  of epoch E, the base record is the server-signed epoch record with the
  **highest epoch that is at most E − 2** among those the client holds or can
  get (its own store, `GET /v1/epochs/{n}`, the epoch log). So that it holds
  one, a client keeps every server-signed epoch record it received during the
  last 8 days, and always the newest one it ever received. If it can get no
  such record, it can't derive one and doesn't start encounters until it can,
  as with an unknown registry.
- The record is marked as client-derived and has no server signature. Evidence
  that uses it also carries its base record (`baseRecord`), so verifiers never
  search the epoch log.
- [accepted] **Validity:** verifiers accept a client-derived record for epoch
  E if its drand value is genuine for E, its base record is validly
  server-signed with an epoch of at most E − 2, and the copied fields equal
  the base record's. A cheater could choose an older base record during an
  outage; that is accepted (see [Known gaps](#known-gaps-accepted-risks)).
- [accepted] **Day and week records** (Peerling of the Day, guardian teams):
  for its own new encounters, a client uses the server-signed record for that
  epoch if it exists. A client that doesn't have it tries `GET /v1/epochs/{E}`
  and the epoch log; if it still has none, it derives the record for that epoch
  at once (without the 10-minute wait), and switches to the signed record for
  later encounters if one turns up. [accepted] (2026-11-10) The derived day
  or week record follows the same base rule: for the day record of epoch
  288 × D, the newest signed record with an epoch of at most 288 × D − 2, even
  if the client holds newer signed records. Honest clients that can reach the
  same records therefore derive the same day and week record.
- [accepted] **The server fills in missed day and week records**
  (2026-11-10): if the server published no record for the first epoch of a
  day or week (epoch 288 × D or 2016 × W), it publishes it as soon as it can,
  signed, with that epoch's drand value and the registry height, versions and
  roots as they stood at that epoch's start. It is added to the epoch log and
  served by `GET /v1/epochs/{E}`; it isn't sent on the epoch topic.
- [accepted] **Evidence keeps its records** (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)): a catch
  or badge is verified with the epoch, day and week records its evidence
  carries. A valid client-derived record stays valid even if a signed record
  for the same epoch turns up later, so finished catches and badges never
  change meaning.
- [accepted] **Launch** (2026-10-27): the game app ships the CID of the
  [genesis record](../glossary.md#genesis-record), the first epoch record the
  operator published. A day or week that began before it uses the genesis
  record as its day or week record. Clients can't derive records until a
  server-signed record at least 2 epochs old exists, so right at launch
  encounters need the server.

This keeps wild encounters working with nothing from the operator server.
Because the randomness still comes from drand, a client-derived record gives
the player almost no extra choice. See [resilience](resilience.md).

**Offline play.** Without network access, the client keeps using the last epoch
record it has. That gives no extra choice (same epoch and same n give the same
encounter), so offline play needs no time limit. Catches made offline are
verified like any other: by whoever needs to check them, from the save log.

**What a verifier checks for a catch** ([accepted]; checks 3, 4 and 6 made
exact 2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)). The verifier first needs complete data
([Results](#verified-peerlings)); then:
1. The epoch record is genuine: drand signature, plus either the server's
   signature or the client-derived validity rule above. The same holds for the
   day record.
2. Encounter numbers run from 0 with no gaps and appear only once, counting
   `encounter` events; each `catch` and `battle-result` repeats the number of
   the `encounter` before it. Epochs never decrease (from the newest
   verification checkpoint on, if there is one: the checkpoint covers the
   part before it); every `encounter` records its epoch, so this is checked
   for all encounters, not just catches. (A save log with two conflicting
   branches shows up as duplicate encounter numbers; which catches then fail
   is defined under [Forks](#verified-peerlings).)
3. The `encounter` event agrees with the evidence (number, epoch, tile,
   candidate). The tile is an encounter-foliage tile (the world is
   deterministic, so any verifier can check). The species is the logged
   candidate from the deterministic candidate list, and the wild level
   matches, for (seed, tile, registry height, save log).
4. **The starting team.** Each member of the evidence `team` is a Peerling the
   catcher owned when the encounter started (in their save state before the
   `encounter` event), with the same instance ID, species and traits. Its
   level must be 1–50 and its HP between 0 and its max HP at that level
   ([accepted] 2026-11-10: a team may enter a battle with fainted members,
   which have 0 HP); at least one member must have more than 0 HP. Levels
   aren't checked further ([Known gaps](#known-gaps-accepted-risks)).
   The team members' own verification isn't required: like levels, they only
   affect the catcher's own battle.
5. Replaying the battle with the logged actions ends in this catch.
6. **The caught instance** is exactly what the replay gives: `id` =
   SHA-256(player ID ‖ encounter number); `species` = the candidate met;
   `level` = the wild level; `traits` and `shimmer` = the encounter's draws;
   `hp` = the wild Peerling's HP at the end of the replay; `xp` = 0;
   `nickname` = null; `origin` = `"wild"`; `originalOwner` = the log's owner.
   Any difference makes the catch invalid.

### Starters and shrine creations

[accepted] Starters ([onboarding](../gameplay/onboarding.md)) and Peerlings
created at the [Creation Shrine](../gameplay/creation-shrine.md) don't come from a
catch, so there's no battle to replay. Instead the server, which is involved
in both anyway, signs an **origin attestation**: instance ID, species CID, first
owner, origin (`starter` or `created`), level, and the randomly drawn traits and
shimmer flag ([peerling-species § Individual variation](../peerlings/peerling-species.md#individual-variation)). These Peerlings are verified
from the start.

### Transfer log and trades

[accepted] Ownership is a chain of signed transfers in an open OrbitDB
**transfer log** ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)).
[accepted] Details:

- **The log:** an OrbitDB *events* database that any player may append to.
  Its access controller accepts an entry only if every transfer in it is
  signed by the `from` player.
- **A transfer:** `{ instanceId, from, to, prev, trade, offers }`, signed by `from`
  ([data-formats § Transfer](data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)). `prev` is the CID
  of the previous transfer of this Peerling, or `origin` for its first
  transfer. `to` is a player ID, or `released` for Peerlings given up at the
  [Creation Shrine](../gameplay/creation-shrine.md).
- **A trade** is a single log entry holding both players' transfers and both
  signatures, so either both sides happen or neither does
  ([trading](../gameplay/trading.md)). [accepted] This is enforced: a transfer
  with a trade ID only counts inside an entry that holds the complete transfer
  set for both confirmed offers
  ([data-formats § Transfer](data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)).
- **Current owner:** follow the Peerling's chain from its origin; the last `to`
  is the owner. A Peerling with no transfers belongs to its first owner.
- **Before trading,** each side looks up the current owners in the ownership
  index and checks the transfer-log entries newer than it
  ([network-performance § Ownership index](network-performance.md#3-ownership-index));
  without the server it syncs the full transfer log instead. It checks that the
  other player is the current owner of what they offer and isn't flagged.

**Double trades.** Without a central authority, two conflicting transfers
can't be prevented: a cheater could give the same Peerling to two players at
once, e.g. while they are not connected to each other. They are detected
instead:
- Two *different* transfers of the same Peerling that follow the same previous
  transfer are a **double trade**. Both are signed by the same player, so the
  pair is cryptographic proof of cheating. Identical copies of one transfer
  (re-posted, or appended by both trade partners), and [accepted] copies that
  differ only in which equivalent entry `prev` names, count once and are never
  a conflict ([data-formats § Transfer](data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)).
- **Resolution:** the transfer whose envelope CID sorts lower is the valid
  one; the other loses. Every node reaches the same result from the same
  data.
- **Flagging:** the cheating player ID is flagged. Clients refuse trades and
  PvP battles with flagged players.
- [accepted] **The whole trade is undone** (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)): a trade
  entry with a losing or invalid transfer is void as a whole, so every
  Peerling in it stays with the player who offered it, and trades that built
  on a void entry become void in turn (exact rules:
  [data-formats § Transfer](data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer)).
  Each player in a void trade appends a `trade-void` event that puts back their
  own Peerlings and removes the ones they received. Clients look for void
  entries at every session start and whenever they sync the transfer log. A
  team that loses a member this way is refilled
  ([catching § Team and collection](../gameplay/catching.md#team-and-collection)).
- The victim loses the Peerling they received but gets back what they gave.
  This is acceptable: nothing in the game is scarce, every species can be
  caught again, and the cheater is exposed. Individual variation
  ([D-0014](../decisions/D-0014-individual-variation.md)) makes a lost
  Peerling more valuable, so this rule may need revisiting if double trades
  turn out to be common.

[accepted] **Unfinished trades** (2026-10-27, designer's choice,
[D-0027](../decisions/D-0027-high-severity-review-fixes.md)). If a trade stream drops while the signed transfers are being
exchanged, one player may hold both sides' transfers and the other only their
own, and the second can't tell whether the trade will happen.
- **Locked:** from the moment a client sends its signed transfers until the
  trade's entry appears in the transfer log or the trade expires, the Peerlings
  it offered are locked: they can't be offered in another trade, released at
  the shrine or used in PvP. Exploration and wild battles are unaffected.
- [accepted] **The lock is in the save log** (2026-11-10,
  [D-0029](../decisions/D-0029-second-review-fixes.md)): before it sends its
  signed transfers, the client appends a `trade-pending` event with the trade
  ID, the partner, the instances it gives, its own signed transfers and the
  other side's final offer data
  ([data-formats § Save-log events](data-formats.md#save-log-events)). The
  lock follows from that event, so it survives a reload, cleared browser data,
  a restore and a move to another computer, and any of the player's devices
  can finish or expire the trade. The lock ends with the player's `trade` or
  `trade-void` event for that trade, or when the transfer log holds a
  trade-expired record for it.
- **Finishing:** a player who holds both sides appends the entry at once
  ([TRD-005](../gameplay/trading.md#requirements)).
- **Expiring:** either player can ask the server to expire the trade
  ([creation-api § Trades](creation-api.md#trades)). If the server holds no
  entry for that trade ID, it appends a signed
  [trade-expired record](data-formats.md#trade-expired-record--peerlingstrade-expired)
  to the transfer log. Any entry for that trade that arrives later is void,
  both players keep what they offered, and a player who already wrote a
  `trade` event for it appends `trade-void`. An expired trade never counts as
  a double trade, so nobody is flagged. If the server already holds the entry,
  it returns it and the trade is complete.
- The client offers *Cancel this trade* 10 minutes after the stream dropped if
  the entry still hasn't appeared. Expiring needs the server; until it is
  reachable, the Peerlings stay locked.

### PvP

[accepted] PvP uses only verified Peerlings; in Fair mode everyone fights at level 50 ([D-0015](../decisions/D-0015-pvp-level-modes.md)). [accepted] Before
the battle, each side verifies the other's team (origin and ownership chain),
using cached results where possible. No server is needed.

[accepted] PvP results are recorded as signed `pvp-result` events for the PvP
win counter ([pvp-battles § Win record](../gameplay/pvp-battles.md#win-record),
[D-0021](../decisions/D-0021-pvp-win-counter.md)). A `pvp-result` counts only
if its signatures verify as defined in
[data-formats § Save-log events](data-formats.md#save-log-events), and only
once per battle ID. [accepted] Void and drawn battles are recorded too
(2026-11-10, designer's choice,
[D-0029](../decisions/D-0029-second-review-fixes.md)): they change no counter,
but they cancel a forfeit win the other player claims for the same battle.

## Known gaps (accepted risks)

[accepted]
- **Positions aren't fully verified.** The encounter tile must be foliage
  (checked), but a modified client could claim a foliage tile it never walked
  to, to target a biome or wild level. Possible mitigation: verifiers check that
  consecutive `position` events are reachable at walking speed (3 tiles per
  second).
- **Small choice among recent epochs.** A player who knows the upcoming
  encounter (the client computes it in advance for prefetching) can stall until
  a new epoch. That is at most one re-roll per 5 minutes. Since D-0014 this can
  also be used to chase good traits or a shimmer; the 5-minute cost keeps it
  slow.
- **Levels aren't verified.** XP from wild battles isn't replayed. An edited
  level matters in the player's own wild battles, in a Peerling they trade away
  (the receiver gets the level shown), and in Real-levels PvP, which both
  players opt into ([D-0015](../decisions/D-0015-pvp-level-modes.md)).
- **Double trades** are detected, not prevented ([Transfer log and trades](#transfer-log-and-trades)).
- **Base record choice.** A client-derived epoch record may copy any
  server-signed record at least 2 epochs older, so a cheater could pick an
  older registry state (fewer species). [accepted] Verifiers can't tell
  whether the server was reachable, so a modified client can do this at any
  time, not only during an outage (corrected 2026-10-27). [accepted]
  (corrected 2026-11-10) The derived record also copies the `generator` and
  `rules` versions, so an old base record selects the species list, the world
  and the battle rules of that time. All three were once valid for everyone,
  so this gives an old state of the game, never a made-up one.
- **Forks before a checkpoint** ([accepted] 2026-11-10, designer's choice,
  [D-0029](../decisions/D-0029-second-review-fixes.md)). A player can sign a
  second branch of their own save log and so make their own catches invalid
  from the first duplicated encounter number on. A catch that a server
  checkpoint already covers is safe ([Forks](#verified-peerlings)). A catch
  that was traded away before any checkpoint covered it is not: its new owner
  can lose the use of it in trades and PvP. The server checkpoints a log
  whenever it has verified it, so the window is short while the server is up.
- **Shrine offering levels aren't verified** (levels in general aren't, see
  above). An edited level lets a player meet the level-20 rule early, and the
  new Peerling's level comes from the offered levels. Accepted: the offering is
  still given up, and the 7-day limit still applies.
- **Fast fled encounters** ([accepted] 2026-10-28, designer's choice,
  [D-0028](../decisions/D-0028-medium-low-review-fixes.md)). A `battle-result` that isn't a catch carries no battle to
  replay, and fleeing always succeeds. So a modified client can log one fled
  encounter after another without walking or battling, until the encounter
  seed gives a trait or shimmer it wants. Honest players can do the same by
  fleeing, only more slowly; the cost is time, not a re-roll of one encounter.
- **Choice among encounter candidates.** A modified client could claim the first
  candidates failed to download and pick a later one: at most a choice of 1 in
  5. Only the species differs between candidates: level, traits and shimmer
  belong to the encounter, so this only lets a player favour a species they like.

## Storage options considered

Background for D-0009. Moving the save to OrbitDB or IPFS fixes *durability*,
not *integrity*: whoever holds the write key can write anything. Integrity comes
from replayable evidence and signatures (originally server signatures; since
D-0013, mostly checks any player can run).

| Option | How it works | Durable / cross-device | IPFS showcase | Cost / risk |
|--------|--------------|------------------------|---------------|-------------|
| S1. Browser only | IndexedDB, with optional export to a file | No | None | Simplest |
| S2. IPFS snapshots + IPNS | Save snapshots on IPFS; the latest one is published under the player's IPNS name | Yes | Good | Light. **Fallback** if S3 doesn't scale |
| **S3. Per-player OrbitDB log** (chosen) | Player-signed event log, replicated and pinned by the server | Yes | Very good | The server keeps one database open per player |

| Integrity option | Result |
|------------------|--------|
| I1. Trust clients | Nothing prevented |
| **I2. Neutralize** (chosen, for PvP levels) | Edited levels give no PvP advantage |
| I3. Server-attested events (chosen in D-0009, replaced by D-0013) | Faked catches and trade duplication prevented, but needs the server |
| **I5. Peer-verified events** (chosen in D-0013) | Faked catches prevented by replay; double trades detected and flagged |
| I4. Server-authoritative game | Rejected: against pillar 5 |

## Requirements

- **SAVE-001** [accepted] A player's save MUST be stored as a per-player OrbitDB log, writable only by the player's identity and replicated and pinned by the server, so it survives cleared browser storage and can be loaded on another device.
- **SAVE-002** [accepted] The client MUST let the player back up their identity key with a recovery phrase and restore it on another device. The private key MUST NOT leave the device in any other form than the recovery phrase, the backup file (SAVE-021) and the phone backup (SAVE-024).
- **SAVE-003** [accepted] Only verified Peerling instances MUST be usable in trades and PvP battles; verification follows [Verified Peerlings](#verified-peerlings).
- ~~**SAVE-004**~~ (removed 2026-10-04, replaced by SAVE-014 and SAVE-015; see D-0013)
- **SAVE-005** [accepted] The save MUST contain every owned Peerling instance with its current level and XP, and MUST also contain the parts listed in [Save contents](#save-contents).
- ~~**SAVE-006**~~ (removed 2026-10-04, replaced by SAVE-016; see D-0013)
- **SAVE-007** [accepted] The save log MUST be event-based as listed in [Save log](#save-log), with periodic snapshots so loading doesn't replay the full history.
- **SAVE-008** [accepted] Encounter seeds MUST be derived as in [Encounter seeds](#encounter-seeds): from the epoch record's randomness, the player ID and a gap-free encounter number.
- **SAVE-009** [accepted] Catches MUST be playable while unverified; verification MAY happen later (e.g. when the server is reachable again).
- **SAVE-010** [accepted] Every wild encounter, including fled and lost ones, MUST be recorded in the save log with its encounter number.
- **SAVE-011** [accepted] The server MUST publish a signed epoch record every 5 minutes, on pubsub and in a server-written OrbitDB epoch log.
- **SAVE-012** [accepted] Starters and shrine-created Peerlings MUST receive a server origin attestation when they are created.
- ~~**SAVE-013**~~ (removed 2026-11-10, replaced by SAVE-042; see D-0029)
- **SAVE-014** [accepted] Ownership MUST be a chain of transfers signed by the current owner and stored in an open OrbitDB transfer log; a trade MUST be a single entry holding both players' signed transfers.
- **SAVE-015** [accepted] Conflicting (different) transfers of the same Peerling MUST be detected, while identical copies of one transfer MUST NOT count as a conflict; [accepted] the transfer with the lower envelope CID wins, and the signer MUST be flagged and refused for trades and PvP.
- **SAVE-016** [accepted] Any client MUST be able to verify a catch by replaying it from the catcher's save log; no server signature is required.
- **SAVE-017** [accepted] A caught Peerling's instance ID MUST be SHA-256(player ID ‖ encounter number).
- **SAVE-018** [accepted] A save log's OrbitDB address MUST be derivable from the player ID alone, so a recovered key can find its save on any node that holds it.
- ~~**SAVE-019**~~ (removed 2026-10-05: no password-based recovery; the designer chose the recovery phrase and backup file only)
- **SAVE-020** [accepted] Inspecting another player's profile, trading or battling with them MUST fetch and keep a persistent backup of their save log (up to 100 players or 100 MB); clients MUST answer `save-wanted` requests for saves they hold.
- **SAVE-021** [accepted] The backup file MUST contain the key and the latest save snapshot.
- ~~**SAVE-022**~~ (removed 2026-10-05: device linking replaced by the phone backup, SAVE-024)
- **SAVE-023** [accepted] Only one computer per player MUST be able to play at a time: a client MUST sync the save log and MUST refuse to start play while another computer's session for the same account is active.
- **SAVE-024** [accepted] A player MUST be able to back up the identity key and save to a phone by scanning a QR code (one-time secret, 5 minutes) shown on the computer, and restore them to a computer the same way; the phone side runs in the Peerlings Viewer.
- **SAVE-025** [accepted] A transfer that carries a trade ID MUST only be valid inside a transfer-log entry that holds the complete set of transfers for both confirmed offers.
- ~~**SAVE-026**~~ (removed 2026-10-27, replaced by SAVE-028; see D-0027)
- **SAVE-027** [accepted] The client MUST append the newest server verification checkpoint to its save log, and verifiers MUST accept a valid checkpoint for the entries it covers and verify the entries after it.
- **SAVE-028** [accepted] Every wild and guardian battle MUST be recorded by an `encounter` event written when it starts, with its encounter number and the epoch used; when it ends, the client MUST write the `catch` event (if any) before the `battle-result`. Verifiers MUST count encounter numbers from `encounter` events and MUST check that epochs never decrease across all encounters.
- ~~**SAVE-029**~~ (removed 2026-11-10, replaced by SAVE-043; see D-0029)
- **SAVE-030** [accepted] The backup file and the phone backup MUST contain the whole save log and, for every owned Peerling, its origin proof and its ownership-chain entries, as defined in [data-formats § Backup file](data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup).
- **SAVE-031** [accepted] Before starting play the client MUST probe for an active session of the same account; when two sessions meet, the one that started later MUST stop at once.
- **SAVE-032** [accepted] Verifiers MUST check a catch's starting team and caught instance as listed in [What a verifier checks for a catch](#encounter-seeds).
- **SAVE-033** [accepted] Verification MUST give verified, invalid or pending as defined in [Verified Peerlings](#verified-peerlings); a result MUST be invalid only when a check fails on complete data, and pending checks MUST be retried.
- **SAVE-034** [accepted] Verifiers MUST verify evidence with the epoch, day and week records it carries, accepting a valid client-derived record even if a signed record exists for that epoch; a day or week that began before the genesis record MUST use the genesis record.
- **SAVE-035** [accepted] A trade entry with an invalid or losing transfer MUST be void as a whole, entries that depend on it MUST be void in turn, and each player in a void trade MUST append a `trade-void` event.
- **SAVE-036** [accepted] Peerlings in a trade whose entry hasn't appeared MUST be locked against trades, releases and PvP until the entry appears or the trade expires; an entry for an expired trade MUST be void and MUST NOT count as a double trade.
- **SAVE-037** [accepted] If no copy of a recovered player's save exists, the client MUST NOT start a new save on that key; it MUST offer to keep looking or to start a new account.
- **SAVE-038** [accepted] Species eligibility at a registry height MUST use listing and delisting `seq` values as defined in [Encounter seeds](#encounter-seeds), and draws MUST sort eligible species by listing seq.
- **SAVE-039** [accepted] Every change to the save MUST come from an event as defined in [Save log](#save-log), including a `rest` event for each rest-point visit; a snapshot MUST equal the result of the events up to its `upTo`, and where they differ the events MUST win.
- **SAVE-040** [accepted] The client MUST keep the recovery phrase's entropy on the device next to the private key and in every backup, so the phrase can be shown again.
- **SAVE-041** [accepted] Transfers that differ only in which equivalent entry `prev` names MUST count as one transfer and MUST NOT count as a double trade.
- **SAVE-042** [accepted] When no server-signed epoch record has arrived for two epochs, or the record of a day's or week's first epoch is missing, clients MUST derive the epoch record from drand, copying the other fields from the base record defined in [Encounter seeds](#encounter-seeds): the server-signed record with the highest epoch at most E − 2 that the client holds or can get. Verifiers MUST accept such records under the validity rule there. The server MUST publish a missed day or week record once it can.
- **SAVE-043** [accepted] The client MUST write a first guardian win's `badge` event before its `battle-result`. At start, the client MUST complete an `encounter` that has no `battle-result`: by replay from its `catch` or its `badge` if there is one, otherwise as fled with no XP and the HP from the battle's start.
- **SAVE-044** [accepted] A fork MUST invalidate only the catches and badges with an encounter number at or above the lowest duplicated number, and MUST NOT invalidate one covered as valid by a server checkpoint, as defined in [Forks](#verified-peerlings).
- **SAVE-045** [accepted] Applying a save-log event MUST have exactly the effect listed in [What each event does to the save](#save-log).
- **SAVE-046** [accepted] An `encounter` event that uses a client-derived epoch or day record MUST name the epoch of its base record.
- **SAVE-047** [accepted] A client MUST append a `trade-pending` event before sending its signed transfers, and the lock on the offered Peerlings MUST follow from that event on every device.
- **SAVE-048** [accepted] A client MUST show the world-generator version and use the battle-rules version active at the epoch of the latest epoch record it has.
- **SAVE-049** [accepted] Verifiers MUST check a badge as listed in [Catches](#catches).

## Open questions

_None at the moment._

## See also

- [Architecture § Trust model](architecture.md#trust-model) · [Catching](../gameplay/catching.md) · [Trading](../gameplay/trading.md) · [PvP battles](../gameplay/pvp-battles.md)
