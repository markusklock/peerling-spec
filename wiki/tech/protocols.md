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
related:
  - wiki/tech/data-formats.md
  - wiki/tech/realtime-networking.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/gameplay/spectating.md
  - wiki/tech/player-data.md
updated: 2026-10-06
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
| `"turn"` | `turn` (uint), `seed` (bytes(32), once revealed), `actions` (all revealed decisions so far, in order: [map: `player` (0 or 1), `turn`, `action`]), `state` (bytes(32), state hash after this turn) |
| `"end"` | `result` (`"win"` \| `"void"` \| `"forfeit"`), `winner` (player ID or null) |

### `peerlings/v1/session/<player ID>` — active session
Published by the computer that is currently playing, every 15 s:
`{ "kind": "session", "device": <bytes(16)>, "t": <time> }`. A computer with
the same account subscribes to its own player's topic before starting play; a
session counts as active while heartbeats arrive, and as ended after 60 s
without one ([player-data § Using the same account on several computers](player-data.md#using-the-same-account-on-several-computers)).

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
| 6 | each deciding side, per decision | `commit` | `turn` (uint), `hash`: SHA-256(DAG-CBOR bytes of the action ‖ nonce) |
| 7 | each deciding side, per decision | `reveal` | `turn`, `action` ([data-formats § Action](data-formats.md#catch-evidence)), `nonce` (bytes(16)) |
| 8 | both, per turn | `state` | `turn`, `hash`: SHA-256 of the battle state after the turn, `sig`: state signature (below) |
| 9 | both | `end` | `result` (`"win"` \| `"void"` \| `"forfeit"`), `winner` (player ID or null), `turn` (the last resolved turn), `hash` (the battle-state hash after it), `sig`: end signature (below) |

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
  - once a side has sent or received `timeout` for a decision, both ignore any
    later `commit` for that decision, and the late player drops its own.
- Battle seed = SHA-256(`"peerlings/pvp-seed/v1"` ‖ value of the lower player
  ID ‖ value of the higher player ID).
- Replacing a fainted Peerling is also a commit/reveal decision (`replace`
  action), so neither side sees the other's choice first.
- A `state` hash that differs from one's own → `end` with result `"void"`.
- **Signatures for the win record** ([accepted] 2026-10-06,
  [pvp-battles § Win record](../gameplay/pvp-battles.md#win-record)), Ed25519
  with the sender's player key:
  - state signature over `"peerlings/pvp-state/v1"` ‖ battle ID (32 bytes) ‖
    turn (8-byte big-endian) ‖ hash;
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
- **Timers and timeouts** ([accepted] 2026-10-06; rules in
  [battle § PvP turn timer](../gameplay/battle.md#pvp-turn-timer)):
  - every decision (turn action or replacement) has a 30 s timer, started when
    the previous turn's `state` messages have been exchanged;
  - a side that hasn't received the other's `commit` in time sends
    `{ "kind": "timeout", "turn": uint }`; both sides then use the action
    `{ "kind": "timeout" }` for the late player. When the turn resolves, a
    timed-out turn action becomes a random move from the battle RNG, and a
    timed-out replacement becomes the first non-fainted team member in team
    order;
  - two timeouts in a row by the same player end the battle as a forfeit;
  - a charging Peerling's second turn needs no decision (its strike is
    automatic);
  - if the stream drops, the player who dropped may reopen it within 60 s with
    `{ "kind": "resume", "battle": <battle ID>, "turn": uint }`; otherwise the
    battle ends as a forfeit by that player.

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
| — | either, before 6 | `cancel` | — |

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

**QR code** (shown on the computer):
`peerlings-backup:?mode=<backup|restore>&peer=<computer peer ID>&addr=<relay multiaddr>&s=<secret, base64url, 32 bytes>&exp=<time>`

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

## Requirements

- **PRT-001** [accepted] All pubsub topics and their messages MUST be exactly as defined on this page, with gossipsub strict signing.
- **PRT-002** [accepted] All direct streams MUST use length-prefixed DAG-CBOR messages in the order defined per protocol.
- **PRT-003** [accepted] PvP seeds, commitments and state hashes MUST be computed exactly as defined here, so that every client and spectator gets the same results.

## Open questions

_None at the moment._

## See also

- [Data formats](data-formats.md) · [Realtime networking](realtime-networking.md) · [Creation API](creation-api.md)
