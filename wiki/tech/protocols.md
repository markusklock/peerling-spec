---
title: Network Protocols
type: data
status: accepted
req_prefix: PRT
tags: [tech, libp2p, pubsub, protocols, formats]
sources:
  - raw/conversations/2026-10-05-formats-request.md
  - raw/conversations/2026-10-06-formats-approved.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-proposals-approved.md
  - raw/conversations/2026-10-06-v1-fun-features.md
  - raw/conversations/2026-10-06-fun-features-approved.md
  - raw/conversations/2026-10-06-review-2-decisions.md
  - raw/conversations/2026-10-27-high-severity-resolutions.md
  - raw/conversations/2026-10-28-medium-low-resolutions.md
  - raw/conversations/2026-11-10-second-review-resolutions.md
related:
  - wiki/tech/data-formats.md
  - wiki/tech/realtime-networking.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/gameplay/spectating.md
  - wiki/tech/player-data.md
updated: 2026-11-10
---

# Network Protocols

> Canonical, exact formats of every message players' browsers and the server
> exchange over libp2p: pubsub topics and direct streams. Encoding, identifiers,
> signatures and record formats follow [data-formats](data-formats.md). What the
> protocols are *for* is described on the linked gameplay and tech pages.

**Status: [accepted]**: drafted 2026-10-05, approved 2026-10-06 ([Q-047](../open-questions.md#q-047)).

## General rules

- **Pubsub** is gossipsub with strict signing: every message is signed by the
  sender's peer key, which is the player's key
  ([FMT-002](data-formats.md#requirements)). This also satisfies
  [NET-003](realtime-networking.md#requirements) without an extra signature.
  Receivers drop messages whose sender isn't who the message claims to be.
- **Every message** is a DAG-CBOR map with `v` (1) and `kind`, except on the
  epoch topic, which carries the epoch record envelope itself.
- **Streams** carry DAG-CBOR messages, each prefixed with its length as an
  unsigned varint. A side that hits an error sends
  `{ "v": 1, "kind": "error", "code": <string>, "message": <string> }` and
  closes the stream.
- **Timeouts:** a stream with no message for 60 s is closed, except during PvP
  turns, which have their own timer ([battle § PvP turn timer](../gameplay/battle.md#pvp-turn-timer)).
  While a player is deciding (choosing a trade offer, answering a challenge),
  their client sends `{ "v": 1, "kind": "ping" }` every 20 s to keep the
  stream open; receivers ignore it.
- **Rate limits:** receivers drop pubsub messages above the rate stated per
  topic and may stop relaying for that peer.

## Pubsub topics

### `peerlings/v1/presence/<rx>_<ry>` — presence

Region coordinates `rx`, `ry` = tile ÷ 32. Rate: one per step while moving (at
most 4 per second), a heartbeat every 5 s when idle. Meaning:
[realtime-networking § Presence](realtime-networking.md#presence).

| Field | Type | Notes |
|-------|------|-------|
| `kind` | `"presence"` | |
| `gen` | uint | world-generator version (players on other versions are ignored) |
| `tile` | tile | |
| `facing` | `"n"` \| `"e"` \| `"s"` \| `"w"` | |
| `step` | time | when the current step started |
| `name` | string ≤ 20 | display name |
| `look` | bytes(8) | first 8 bytes of SHA-256 of the appearance map |
| `emote` | string, optional | emote ID: `wave`, `heart`, `laugh`, `wow`, `thumbs-up`, `thumbs-down`, `challenge` or `trade` ([multiplayer § Communication](../gameplay/multiplayer.md#communication)) |
| `session` | bytes(16) | device ID of the active session ([SAVE-023](player-data.md#requirements)) |
| `battle` | bytes(32), optional | battle ID while in a PvP battle that allows spectators |
| `follower` | map, optional | [accepted] `species` (CID), `shimmer` (bool): the player's [following Peerling](../gameplay/exploration.md#following-peerling); omitted when hidden |
| `jump` | bool, optional | [accepted] (2026-10-28) true on the first message after a respawn at a rest point or a move by a generator update, so receivers accept the long move ([NET-005](realtime-networking.md#requirements)) |
| `t` | time | |

### `peerlings/v1/epoch` — epoch records
Published by the operator once per epoch: the epoch record envelope
([data-formats § Epoch record](data-formats.md#epoch-record--peerlingsepoch)).

### `peerlings/v1/feed` — world feed
Rate: receivers accept at most 1 per player per minute. Meaning:
[world-feed](../gameplay/world-feed.md).

| Field | Type | Notes |
|-------|------|-------|
| `kind` | `"species-published"` \| `"shimmer-caught"` \| `"shrine-creation"` \| `"first-found"` \| `"all-badges"` ([accepted]) | |
| `species` | CID | omitted for `all-badges` |
| `ref` | CID | the registry listing (published / shrine), the `catch` save-log entry (shimmer, first-found) or the 12th `badge` entry (all-badges) |
| `name` | string ≤ 20 | the player's display name |
| `t` | time | |

### `peerlings/v1/battle/<battle ID>` — spectating
Battle ID = SHA-256(lower player ID ‖ higher player ID ‖ the challenge
message's `t`), where player IDs are compared as strings; in the topic name it
is written as lower-case hex. In every
two-player list (`players`, `teams`, the action pairs) the player with the
lower ID comes first. Published
by both fighters. Meaning: [spectating](../gameplay/spectating.md).

| `kind` | Fields |
|--------|--------|
| `"start"` | `players` [2 player IDs], `mode` (`"fair"` \| `"real"`), `rules` (uint), `teams` (two lists of team members, as in the battle `team` message) |
| `"turn"` | `turn` (uint), `seed` (bytes(32), once revealed), `actions` ([accepted] 2026-10-28: this turn's revealed decisions, in order: [map: `player` (0 or 1), `turn`, `action`]), `state` (bytes(32), state hash after this turn), `start` (CID of the `start` message as a DAG-CBOR block), `history` (CID of the [battle history](#battle-history-block) after this turn) |
| `"end"` | `result` (`"win"` \| `"void"` \| `"forfeit"` \| `"draw"`), `winner` (player ID or null) |

[accepted] **Late spectators** (2026-10-28, [D-0028](../decisions/D-0028-medium-low-review-fixes.md)). A `turn` message
stays small at any length of battle. A spectator who joins late gets the
two blocks it names from either fighter. The CIDs come from a message signed
by a fighter, so the blocks are authenticated by their hashes. [accepted]
(2026-11-10, [D-0029](../decisions/D-0029-second-review-fixes.md)) The
spectator asks for them over the direct stream
[`/peerlings/battle-blocks/1.0.0`](#peerlingsbattle-blocks100--blocks-for-late-spectators),
which is one of the game's own streams and so also works over a relayed
connection ([network-performance § Connections](network-performance.md#connections)).
Fighters also provide both blocks over bitswap, for peers with a direct
connection.

#### Battle history block
A DAG-CBOR block `{ "v": 1, "type": "peerlings/battle-history", "battle":
<battle ID>, "actions": [all revealed decisions so far, as in `turn`] }`. Each
fighter builds it after every turn; both get the same bytes and CID, since the
encoding is deterministic.

### `peerlings/v1/session/<player ID>` — active session
Published by the computer that is currently playing, every 15 s:
`{ "kind": "session", "device": <bytes(16)>, "since": <time>, "t": <time> }`,
where `since` ([accepted] 2026-10-27) is when this session started. A computer
with the same account subscribes to its own player's topic before starting
play; a session counts as active while heartbeats arrive, and as ended after
60 s without one ([player-data § Using the same account on several computers](player-data.md#using-the-same-account-on-several-computers)).

[accepted] (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)) Before starting play, a computer also
publishes `{ "kind": "session-probe", "device": <bytes(16)>, "t": <time> }`;
an active session answers at once with a `session` heartbeat. The prober
starts play if no heartbeat arrives within 3 s. A playing computer that
receives a heartbeat from another device keeps playing only if its own
`since` is lower (on a tie, its device ID is lower); otherwise it stops.

### `peerlings/v1/save-wanted` — save recovery
`{ "kind": "save-wanted", "player": <player ID>, "t": <time> }`. Rate: 1 per
player per minute. Holders answer by opening `/peerlings/save-backup/1.0.0`
to the asking player.

### `peerlings/v1/creator/<player ID>` — creator notifications
Published by the operator. Meaning: [creator-feedback](../gameplay/creator-feedback.md).
`{ "kind": "caught" | "traded" | "delisted" | "first-found", "species": <CID>, "by": <display name or null>, "t": <time> }`. For `first-found`, `by` is the finder's display name.

## Direct streams

### `/peerlings/battle/1.0.0` — PvP battle

The challenger opens the stream. Messages in order (A = challenger, B = the
other player). Meaning: [pvp-battles](../gameplay/pvp-battles.md).

| # | From | `kind` | Fields |
|---|------|--------|--------|
| 1 | A | `challenge` | `mode` (`"fair"` \| `"real"`), `spectators` (bool), `t`, `rules` (uint: the battle-rules version A's client will use, [battle § Rules versions](../gameplay/battle.md#rules-versions)) |
| 2 | B | `accept` / `decline` | `decline` carries `reason`: `"declined"`, `"busy"` (already in a battle or trade; also used for blocked players) or `"version"` (B's rules version differs; the player with the older app is asked to reload) |
| 3 | both | `team` | `members`: [map: `instanceId`, `species`, `level`, `traits`, `shimmer`, `originProof`, `saveLog` (address of the original owner's save log, where `originProof` is)] (up to 4, team order) |
| 4 | both | `seed-commit` | `hash`: SHA-256(value) |
| 5 | both | `seed-reveal` | `value`: bytes(32) |
| 6 | each deciding side, per decision | `commit` | `turn` (uint), `phase` (`"action"` \| `"replace"`), `hash`: SHA-256(DAG-CBOR bytes of the action ‖ nonce) |
| 7 | each deciding side, per decision | `reveal` | `turn`, `phase`, `action` ([data-formats § Action](data-formats.md#catch-evidence)), `nonce` (bytes(16)) |
| 8 | both, per turn | `state` | `turn`, `hash`: SHA-256 of the battle state after the turn, `sig`: state signature (below) |
| 9 | both | `end` | `result` (`"win"` \| `"void"` \| `"forfeit"` \| `"draw"`), `winner` (player ID or null), `turn` (the last resolved turn), `hash` (the battle-state hash after it), `sig`: end signature (below) |

- Steps 1–2: an unanswered `challenge` counts as busy, and crossing requests
  between the same two players are settled by player ID
  ([multiplayer § What players experience](../gameplay/multiplayer.md#what-players-experience)).
  The sender withdraws a request by closing the stream.
- Step 3: each side verifies the other's team (origin and ownership,
  [player-data § Verified Peerlings](player-data.md#verified-peerlings)) before
  sending step 4; on failure it sends `error` with code `"unverified"`, and if
  the other player is flagged for a double trade, `error` with code
  `"flagged"`.
- **Turns and decisions** ([accepted] 2026-10-06):
  - turns are numbered from 1; in each turn, every side that has to choose a
    turn action commits and reveals one. A side whose Peerling is on the
    second turn of a charge move doesn't choose, so it sends no `commit`;
  - after the moves, each side whose active Peerling fainted chooses a
    `replace` action, with the same `turn` number; only those sides commit
    and reveal;
  - the `state` message for a turn is exchanged after its replacements. In the
    battle state, `active` is the Peerling currently out (a fainted one stays
    `active` until it is replaced);
  - [accepted] (2026-10-27) a decision is identified by its `turn` and its
    `phase`: `"action"` for the turn action, `"replace"` for a replacement
    after a faint. `commit`, `reveal` and `timeout` all carry both;
  - once a side has sent or received `timeout` for a decision, both ignore any
    later `commit` or `reveal` for that decision, and the late player drops
    its own;
  - [accepted] (2026-11-10) a decision **requires** a `commit` and a `reveal`
    from each side that decides in it, and from nobody else. If only one side
    decides (the other is charging, or only one side replaces), that side
    sends its `commit` and then its `reveal` at once. A turn in which both
    sides are charging has no action decision.
- Battle seed = SHA-256(`"peerlings/pvp-seed/v1"` ‖ value of the lower player
  ID ‖ value of the higher player ID).
- Replacing a fainted Peerling is also a commit/reveal decision (`replace`
  action), so neither side sees the other's choice first.
- A `state` hash that differs from one's own → `end` with result `"void"`.
- **Signatures for the win record** ([accepted] 2026-10-06,
  [pvp-battles § Win record](../gameplay/pvp-battles.md#win-record)), Ed25519
  with the sender's player key:
  - state signature over `"peerlings/pvp-state/v1"` ‖ battle ID (32 bytes) ‖
    mode (`"fair"` or `"real"`, as UTF-8) ‖ turn (8-byte big-endian) ‖ hash
    ([accepted] mode added 2026-10-28, so a state can't be relabelled either);
  - [accepted] (2026-10-28) right after `seed-reveal`, both sides send a
    signed `state` for turn 0, the starting state, so a forfeit can be proven
    even before the first turn is resolved;
  - end signature over `"peerlings/pvp-end/v1"` ‖ battle ID ‖
    DAG-CBOR(`{ "result", "winner", "turn", "hash", "mode" }`), where `mode`
    is the challenge's mode, so a result can't be relabelled.
  A side that receives a `state` or `end` with a bad signature sends `error`
  with code `"bad-signature"` and ends the battle as void.
- **Battle state** (exact, [accepted] 2026-10-06): the hashed value is the
  DAG-CBOR map
  `{ "turn": uint, "sides": [side, side] }` with the lower player ID's side
  first, where `side` = `{ "active": <instance ID bytes>, "members": [member, …] }`
  in team order and `member` = `{ "id": <instance ID bytes>, "hp": uint,
  "stages": [attack, defense, speed] (ints −3…3), "charging": bool }`.
- **Timers and timeouts** ([accepted] 2026-10-06, deadlines made exact
  2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md); rules in
  [battle § PvP turn timer](../gameplay/battle.md#pvp-turn-timer)):
  - **commit deadline:** every decision has a 30 s timer. It starts, for the
    action of a turn, when both `state` messages of the previous turn have
    been exchanged ([accepted] 2026-11-10: for turn 1 these are the turn-0
    states, no longer the `seed-reveal` messages); for a replacement, when
    that turn's moves are resolved, which is when every action `reveal` the
    turn requires has been exchanged (in a turn with no action decision, at
    the same moment as the turn itself starts);
  - **reveal deadline:** a side sends its `reveal` as soon as it has every
    `commit` the decision requires; every other required `reveal` must arrive
    within 10 s after that;
  - a side that hasn't received the other's `commit` or `reveal` in time sends
    `{ "kind": "timeout", "turn": uint, "phase": …, "stage": "commit" |
    "reveal" }`; both sides then use the action `{ "kind": "timeout" }` for the
    late player in that decision. When the turn resolves, a timed-out turn
    action becomes a random move from the battle RNG, and a timed-out
    replacement becomes the first non-fainted team member in team order;
  - [accepted] **Late and disputed timeouts** (2026-11-10,
    [D-0029](../decisions/D-0029-second-review-fixes.md)). A side waits 2 s
    past a deadline before it sends `timeout`, and a side whose own deadline
    has passed on its own clock no longer sends that `commit` or `reveal`: it
    uses `timeout` for itself and waits for the other's `timeout` message. A
    side that receives a `timeout` for a `commit` or `reveal` it had already
    sent before the `timeout` arrived doesn't accept it: it sends `error`
    with code `"timeout-disputed"` and ends the battle as void. So a
    `timeout` can never turn a choice the other side already saw into a
    random move; the worst a false claim does is void the battle;
  - two timeouts in a row by the same player (in any phase) end the battle as
    a forfeit;
  - a charging Peerling's second turn needs no decision (its strike is
    automatic);
  - **state deadline:** each side sends its `state` as soon as the turn,
    including its replacements, is resolved; a `state` that hasn't arrived
    within 10 s counts as a dropped stream.
- **Disconnects** ([accepted] 2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)):
  - if the stream drops, both sides try to reopen it for 60 s. Either may
    dial; if two streams open, the one opened by the lower player ID is kept;
  - each side then sends `{ "kind": "resume", "battle": <battle ID>, "have":
    uint or absent }`, where `have` is the last turn whose `state` it has
    received from the other side. [accepted] (2026-11-10) 0 means the turn-0
    state; `have` is absent when it has received no `state` at all;
  - on receiving `resume`, a side resends, in their original order, all the
    messages it sent after its own `state` for turn `have`; for an absent
    `have`, every message it sent after `accept` (its `team`, `seed-commit`,
    `seed-reveal` and turn-0 `state`, as far as it had got). Receivers ignore
    messages they already have. The RNG
    position and timeout counts follow from the replayed actions, and the
    current decision's timer restarts at 30 s;
  - [accepted] **When a battle has started** (2026-11-10,
    [D-0029](../decisions/D-0029-second-review-fixes.md)): for a side, the
    battle has started once it has sent its turn-0 `state`. A stream that
    drops before `accept` just ends the request. A stream that drops after
    `accept` can be resumed as above at any point;
  - if no stream is reopened within 60 s:
    - a side for which the battle hasn't started drops it; nothing is
      recorded;
    - a side that holds a signed `state` from the other player ends the
      battle as a forfeit by the other player;
    - a side for which the battle has started but that holds no signed
      `state` from the other can't prove a forfeit; it ends the battle as
      void and records that.

    Nobody can tell objectively who dropped: if both clients record a forfeit
    win, the two claims name different winners and cancel each other, and a
    recorded void cancels the other side's forfeit claim too
    ([data-formats § Save-log events](data-formats.md#save-log-events)), so
    a drop on both sides counts for nobody.
- [accepted] **Recording void and drawn battles** (2026-11-10, designer's
  choice, [D-0029](../decisions/D-0029-second-review-fixes.md)): a side that
  ends a started battle as void (a state mismatch, a bad signature, a disputed
  timeout, or the case above) or as a draw signs an `end` with that result and
  `winner` null, sends it if the stream is still open, and writes a
  `pvp-result` with it. The other side doesn't need to agree.

### `/peerlings/trade/1.0.0` — trade

Meaning: [trading](../gameplay/trading.md).

| # | From | `kind` | Fields |
|---|------|--------|--------|
| 1 | A | `propose` | `t`, `trade` (bytes(16): the random trade ID both sides' transfers will carry) |
| 2 | B | `accept` / `decline` | `decline` carries `reason` (`"declined"` or `"busy"`) |
| 3 | either, any time | `offer` | `instances` [instance]: the full instance data of the sender's current offer ([data-formats § Peerling instance](data-formats.md#peerling-instance)); clears both confirmations |
| 4 | either | `confirm` | `offers`: the offers hash (below) |
| 5 | both, after both confirmed | `transfers` | the sender's signed transfer envelopes, carrying the `trade` ID from `propose` and the `offers` hash ([data-formats § Transfer](data-formats.md#transfer-and-transfer-log-entry--peerlingstransfer): trades are all or nothing) |
| 6 | A | `done` | `entry`: CID of the transfer-log entry A appended (both sides' transfers) |
| — | either, before 5 | `cancel` | — |

After step 5 the trade can't be cancelled. If B doesn't receive `done` (or
the stream drops), B appends the entry itself; both entries hold identical
transfers, which never count as a conflict. [accepted] (2026-10-27) Until the
entry appears in the transfer log, each side's offered Peerlings are locked,
and either side may ask the server to expire the trade
([player-data § Transfer log and trades](player-data.md#transfer-log-and-trades)).
[accepted] (2026-11-10) Before a side sends its `transfers` (step 5), it
appends a `trade-pending` event to its save log, which holds the lock, its own
signed transfers and the other side's final offer data. An unanswered
`propose` counts as busy, like an unanswered `challenge`.

- **Offers hash** = SHA-256 of the DAG-CBOR list `[offer of the lower player
  ID, offer of the higher player ID]`, where each offer is the list of its
  instance IDs (bytes(32)) sorted byte by byte. Each offer holds at least one
  Peerling (both players sign, [TRD-003](../gameplay/trading.md#requirements)).
  The transfer-log access controller recomputes it from the transfers in the
  entry.
- B checks the entry appears in the transfer log, then both append `trade`
  events to their save logs.
- Each side checks the received instance data against what it can verify:
  origin from the **original owner's** save log (`originalOwner`, where the
  catch is) or the origin attestation, ownership from the transfer log, and
  traits; level, XP and HP are taken from the offer.
- [accepted] (2026-10-28, [D-0028](../decisions/D-0028-medium-low-review-fixes.md)) The confirmation commits to the
  instance IDs only; the rest of the offer is unverified data, like levels. A
  client MUST store the other side's final offer data durably before it sends
  its own `transfers` ([accepted] 2026-11-10: in the `in` field of its
  `trade-pending` event). If that copy can't be read (e.g. a backup made
  before the event), the client rebuilds each received instance from the sender's save
  log: its state there just before the sender's `trade` event for this entry,
  or at the log's head if the sender hasn't written one.

### `/peerlings/battle-blocks/1.0.0` — blocks for late spectators

[accepted] (2026-11-10, [D-0029](../decisions/D-0029-second-review-fixes.md))
Opened by a spectator to either fighter of a battle that allows spectators.
Request `{ "kind": "battle-blocks", "battle": <battle ID>, "want": [<CID>] }`
(at most 2 CIDs, taken from a `turn` message) → response `{ "kind":
"battle-blocks", "blocks": [<bytes>] }` with the raw blocks in the order
asked, or `error` with code `"not-found"`. The spectator checks each block
against its CID. A fighter answers at most one request per peer every 5 s and
only for its own current battle.

### `/peerlings/profile/1.0.0` — profile

Request `{ "kind": "profile-request" }` → response `{ "kind": "profile",
"profile": <CID of the profile document>, "saveLog": <address>, "heads":
[<CID>] }`. The requester then replicates the save log, which also keeps a
backup ([SAVE-020](player-data.md#requirements)).

### `/peerlings/save-backup/1.0.0` — serving a backed-up save

Opened by a player holding a backup, to the player who sent `save-wanted`
(the sender is known from the signed pubsub message). The holder sends one
message `{ "kind": "save-held", "player": <player ID>, "snapshot": <CID>,
"saveLog": <address>, "heads": [<CID>] }` and closes the stream.

### `/peerlings/phone-backup/1.0.0` — phone backup

Meaning: [player-data § Phone backup](player-data.md#phone-backup).

**QR code** (shown on the computer; [accepted] a Viewer URL since
2026-10-28, so the phone's normal camera opens it):
`https://view.<domain>/#/backup?mode=<backup|restore>&peer=<computer peer ID>&addr=<relay multiaddr>&s=<secret, base64url, 32 bytes>&exp=<time>`

Everything after `#` stays on the phone: browsers never send the fragment to
the web server. The Viewer's own scanner reads the same URL. Next to the QR
code, the computer shows *"Scan this with your phone's camera"* and the
Viewer's address.

The phone always dials the computer.

| # | From | `kind` | Fields |
|---|------|--------|--------|
| 1 | phone | `hello` | `mode`, `proof`: HMAC-SHA256(secret, `"peerlings-phone-backup-v1"` ‖ phone peer ID ‖ computer peer ID) |
| 2 | computer | `ready` / `denied` | after the player confirms on screen |
| 3 | sender | `chunk` | `n` (uint), `data`: encrypted bytes of the backup CAR file ([data-formats § Backup file](data-formats.md#backup-file-and-phone-backup-payload--peerlingsbackup)) |
| 4 | sender | `end` | `total` (uint, chunks), `sha256` (bytes(32), of the plain CAR) |

- The sender is the computer for `backup` and the phone for `restore`.
- Encryption: AES-256-GCM, key = HKDF-SHA256(secret, info
  `"peerlings-phone-backup-v1/key"`), 64 KiB chunks, nonce = chunk number as a
  12-byte big-endian integer.
- The secret is single-use: the computer rejects a second `hello` with the same
  secret, or any `hello` after `exp`.
- [accepted] **Continuing an interrupted transfer** (2026-11-10,
  [D-0029](../decisions/D-0029-second-review-fixes.md)). A backup holds the
  whole save log, so it can be larger than one relayed connection may carry
  ([network-performance § Connections](network-performance.md#connections)).
  The one exception to the single-use rule: the same phone peer ID may send
  `hello` again with the same secret, up to 30 minutes after `exp`, to
  continue a transfer that broke off. The receiver says how many chunks it
  already holds in a field `have` (uint): the phone in its `hello` (backup),
  the computer in its `ready` (restore). The sender continues with chunk
  `have`; the player isn't asked to confirm again. The sender keeps the same
  CAR file for that time, so the chunks and the final `sha256` match.

## Requirements

- **PRT-001** [accepted] All pubsub topics and their messages MUST be exactly as defined on this page, with gossipsub strict signing.
- **PRT-002** [accepted] All direct streams MUST use length-prefixed DAG-CBOR messages in the order defined per protocol.
- **PRT-003** [accepted] PvP seeds, commitments and state hashes MUST be computed exactly as defined here, so that every client and spectator gets the same results.
- **PRT-004** [accepted] Every PvP decision MUST carry its turn and phase and follow the commit, reveal and state deadlines defined here, and a resumed battle MUST resend the messages the other side hasn't confirmed, as defined in [PvP battle](#peerlingsbattle100--pvp-battle).
- **PRT-005** [accepted] A side that receives a `timeout` for a `commit` or `reveal` it had already sent MUST end the battle as void, and MUST NOT accept the timeout action.
- **PRT-006** [accepted] A PvP battle MUST be resumable from any point after `accept`; a side MUST claim a forfeit only if it holds a signed `state` from the other player, and MUST record a started battle that ends void or drawn.
- **PRT-007** [accepted] Fighters MUST serve the `start` and battle-history blocks of their current battle over `/peerlings/battle-blocks/1.0.0`.
- **PRT-008** [accepted] An interrupted phone-backup transfer MUST be continuable from the last chunk received, by the same phone and with the same secret.

## Open questions

_None at the moment._

## See also

- [Data formats](data-formats.md) · [Realtime networking](realtime-networking.md) · [Creation API](creation-api.md)
