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
updated: 2026-10-06
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

| Event | Written when | Payload |
|-------|-------------|---------|
| `profile` | Onboarding; profile changes | display name, appearance |
| `species-created` | Creation pipeline finished | species CID |
| `starter` | Onboarding finished | the starter instance + the server's origin attestation |
| `release` | Peerlings offered at the [Creation Shrine](../gameplay/creation-shrine.md) | instance IDs, transfer-log entry reference |
| `created` | A shrine creation was published | the new instance + the server's origin attestation |
| `catch` | A wild Peerling is caught | the new instance + **catch evidence** (see below) |
| `battle-result` | After a wild battle | for each participating instance: XP gained, new level, HP |
| `team` | Team changed | ordered instance IDs |
| `session-start` | The game starts on a computer | device ID ([Using the same account on several computers](#using-the-same-account-on-several-computers)) |
| `nickname` | Peerling renamed | instance ID, nickname |
| `trade` | A trade entry was written to the transfer log | transfer-log entry reference; instances out; full data of instances in |
| `seen` | First sighting of a species, and first sighting as a shimmer | species CID, biome, shimmer flag |
| `position` | Every 30 s while moving, and on exit | position, facing |
| `explored` | The player enters a chunk for the first time | the newly revealed chunks ([exploration § Map](../gameplay/exploration.md#map)) |
| `snapshot` | Every 50 events, at every rest point visit, and on exit | CID of a DAG-CBOR document with the full current save, and the last event it includes |
| `checkpoint` | When the server has a newer verification checkpoint for this player (checked at session start and every hour) | the server-signed checkpoint |

Loading a save means reading the latest `snapshot` and applying the events
after it. The server replicates every player's log and pins the snapshots.
Other players fetch a save log when they need to verify one of its catches.

## Account recovery

[accepted] An account is a cryptographic key pair; the public key is the
player ID. The save is a per-player OrbitDB log that the server replicates and
pins (SAVE-001). The key is recovered with the **recovery phrase** or a
**backup file**; there is no password-based recovery (decided 2026-10-05).

### Getting the key back

| Method | How | Strength |
|--------|-----|----------|
| **Recovery phrase** | 12 random words shown once at onboarding; the key is derived from them. Nothing secret is stored anywhere | Very strong: the words are random |
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
- **Making a backup:** on the computer, *Settings → Back up to phone* shows a QR
  code with the computer's peer ID, a relay address and a one-time secret
  (32 random bytes, valid for 5 minutes). The phone scans it with its camera,
  connects to the computer over libp2p (WebRTC, through a relay if needed) and
  proves it knows the secret. The computer asks the player to confirm, then
  sends the **identity key** and the **save** (the latest save snapshot, packed
  as a CAR file, the IPFS archive format that keeps every CID intact; exact
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

### Finding the save again

[accepted] The save log's OrbitDB address is computed from the player ID alone
(fixed database name, type and writer), so a recovered key leads straight to
its save log; no central account directory is needed. On a new device the
player chooses *Recover account*, enters the phrase or loads the file, and the
client fetches the save log from whichever node holds it and loads the latest
snapshot.

If no node holds the save any more, the key still works: the player keeps
their identity, and every species they created still names them as creator.
Only progress is lost.

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

**2. Save snapshot in the backup file**

The backup file holds the key *and* the latest save snapshot. The game reminds
the player to refresh it now and then (at rest points,
[exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points)).
This works even if every network copy is gone.

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
   for every verifier. A catch that fails is invalid forever: the Peerling stays
   in its owner's collection but can never be traded or used in PvP.

[accepted] **Guardian badges** are verified the same way: the verifier
recomputes the week's guardian team from the `weekRecord`, replays the battle
from the `badge` event's evidence, and checks the encounter-seed rules
([guardians](../gameplay/guardians.md#badges)).

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
  behind IPFS, is one of its main contributors). It is the value of the drand round that
  starts at or just after the epoch's start time. drand values can be verified
  with drand's public key, so clients don't have to trust the server not to
  bias the randomness. Clients get drand values through the server's epoch
  record and verify the drand signature locally, so they never contact drand
  directly. Fallback if drand is unavailable or not wanted: the server
  generates the value itself, and players trust it like they trust the
  registry.
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
  (`battle-result` carries n).
- encounter seed = SHA-256("peerlings/encounter/v1" ‖ epoch randomness ‖ player
  ID ‖ n).
- The seed drives the [random number generator](../gameplay/battle.md#random-number-generator)
  for the species choice, the wild level and the whole battle
  ([encounters](../gameplay/encounters.md)).
- An encounter uses the latest epoch record the client has when the encounter
  starts. Epoch numbers must never decrease from one encounter to the next.

**Why this stops re-rolling.**
- Same epoch + same n → same encounter, so reloading the page gives exactly the
  same encounter.
- Fleeing is a logged encounter, so n can't be skipped. A gap in the encounter
  numbers makes every later catch fail verification.
- The only way to get a different encounter for the same n is to wait for a new
  epoch before triggering it, which costs up to 5 minutes. See *Known gaps*.

**Registry state.**
- The server gives each registry entry a sequence number `seq` (1, 2, 3, …)
  when adding it; a takedown records `removedAtSeq`
  ([orbitdb-registry](orbitdb-registry.md)).
- During epoch E, the eligible species are the entries with
  seq ≤ registryHeight(E), minus those with removedAtSeq ≤ registryHeight(E).
- A client that doesn't yet know the registry up to that height doesn't start
  encounters until it does. [accepted] The compact registry index named in
  the epoch record is enough, so this takes a second or two
  ([network-performance § Registry index](network-performance.md#1-registry-index)).

**When the server is offline.** [accepted] If no new server-signed epoch
record has arrived for 10 minutes (two epochs), the client derives epoch
records itself:
- `randomness` is the drand value for the epoch, fetched directly from public
  drand endpoints (run by League of Entropy members) and checked against
  drand's public key as usual.
- `registryHeight` (and the `generator` and `rules` versions and the snapshot
  roots) are copied from
  the latest server-signed epoch record in the epoch log whose epoch is lower
  than the derived record's ([accepted] 2026-10-06). Any verifier can look that
  record up, so a client can't pick an older, smaller registry state. While the
  server is down nothing can be added to the registry (only the server can sign
  listings), so nothing is missed.
- The record is marked as client-derived and has no server signature.

This keeps wild encounters working with nothing from the operator server.
Verifiers accept client-derived records whose drand value is genuine and whose
height matches the client's latest signed record. Because
the randomness still comes from drand, a client-derived record gives the player
no extra choice. See [resilience](resilience.md).

**Offline play.** Without network access, the client keeps using the last epoch
record it has. That gives no extra choice (same epoch and same n give the same
encounter), so offline play needs no time limit. Catches made offline are
verified like any other: by whoever needs to check them, from the save log.

**What a verifier checks for a catch.**
1. The epoch record is genuine: drand signature, plus either the server's
   signature or the client-derived rules above.
2. Encounter numbers run from 0 with no gaps and appear only once (counting
   `battle-result` events; a `catch` repeats the number of its own
   `battle-result`); epochs never decrease (from the newest verification checkpoint on, if there is one: the
   checkpoint covers the part before it) (every `battle-result` records its epoch, so this can be checked for
   all encounters, not just catches). (A save log with two conflicting branches shows up as duplicate
   encounter numbers, so every catch after the fork fails.)
3. The logged tile is an encounter-foliage tile (the world is deterministic,
   so any verifier can check). The species is the logged candidate from the
   deterministic candidate list, and the wild level matches, for (seed, tile,
   registry height, save log).
4. Replaying the battle with the logged actions ends in this catch.
5. The instance ID is SHA-256(player ID ‖ encounter number).

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
- Two transfers with the same `prev` are a **double trade**. Both are signed
  by the same player, so the pair is cryptographic proof of cheating.
- **Resolution:** the transfer whose log-entry CID sorts lower is the valid one;
  the other is void. Every node reaches the same result from the same data.
- **Flagging:** the cheating player ID is flagged. Clients refuse trades and
  PvP battles with flagged players.
- The player who received the void transfer loses that Peerling. This is
  acceptable: nothing in the game is scarce, every species can be caught again,
  and the cheater is exposed. Individual variation ([D-0014](../decisions/D-0014-individual-variation.md)) makes
  a lost Peerling more valuable, so this rule may need revisiting if double
  trades turn out to be common.

### PvP

[accepted] PvP uses only verified Peerlings; in Fair mode everyone fights at level 50 ([D-0015](../decisions/D-0015-pvp-level-modes.md)). [accepted] Before
the battle, each side verifies the other's team (origin and ownership chain),
using cached results where possible. No server is needed.

[accepted] PvP results are recorded as signed `pvp-result` events for the PvP
win counter ([pvp-battles § Win record](../gameplay/pvp-battles.md#win-record),
[D-0021](../decisions/D-0021-pvp-win-counter.md)). A `pvp-result` counts only
if its signatures verify as defined in
[data-formats § Save-log events](data-formats.md#save-log-events), and only
once per battle ID.

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
- **SAVE-013** [accepted] When no server-signed epoch record has arrived for two epochs, clients MUST derive epoch records from drand, copying the registry height and versions from the latest server-signed epoch record before that epoch; verifiers MUST accept such records.
- **SAVE-014** [accepted] Ownership MUST be a chain of transfers signed by the current owner and stored in an open OrbitDB transfer log; a trade MUST be a single entry holding both players' signed transfers.
- **SAVE-015** [accepted] Conflicting transfers of the same Peerling MUST be detected; [accepted] the transfer with the lower log-entry CID wins, and the signer MUST be flagged and refused for trades and PvP.
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
- **SAVE-026** [accepted] Every `battle-result` MUST record the epoch used, and verifiers MUST check that epochs never decrease across all encounters.
- **SAVE-027** [accepted] The client MUST append the newest server verification checkpoint to its save log, and verifiers MUST accept a valid checkpoint for the entries it covers and verify the entries after it.

## Open questions

_None at the moment._

## See also

- [Architecture § Trust model](architecture.md#trust-model) · [Catching](../gameplay/catching.md) · [Trading](../gameplay/trading.md) · [PvP battles](../gameplay/pvp-battles.md)
