---
title: Network Protocols
type: data
status: accepted
req_prefix: PRT
tags: [tech, libp2p, pubsub, protocols, formats]
sources:
  - raw/conversations/2026-10-05-formats-request.md
  - raw/conversations/2026-10-06-formats-approved.md
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
- **Every message** is a DAG-CBOR map with `v` (1) and `kind`.
- **Streams** carry DAG-CBOR messages, each prefixed with its length as an
  unsigned varint. A side that hits an error sends
  `{ "v": 1, "kind": "error", "code": <string>, "message": <string> }` and
  closes the stream.
- **Timeouts:** a stream with no message for 60 s is closed, except during PvP
  turns, which have their own timer ([battle § PvP turn timer](../gameplay/battle.md#pvp-turn-timer)).
- **Rate limits:** receivers drop pubsub messages above the rate stated per
  topic and may stop relaying for that peer.

## Pubsub topics

### `peerlings/v1/presence/<rx>_<ry>` — presence

Region coordinates `rx`, `ry` = tile ÷ 32. Rate: one per step while moving (at
most 4 per second), a heartbeat every 5 s when idle. Meaning:
[realtime-networking § Presence](realtime-networking.md#presence-proposed).

| Field | Type | Notes |
|-------|------|-------|
| `kind` | `"presence"` | |
| `gen` | uint | world-generator version (players on other versions are ignored) |
| `tile` | tile | |
| `facing` | `"n"` \| `"e"` \| `"s"` \| `"w"` | |
| `step` | time | when the current step started |
| `name` | string ≤ 20 | display name |
| `look` | bytes(8) | first 8 bytes of SHA-256 of the appearance map |
| `emote` | string, optional | emote ID ([multiplayer § Communication](../gameplay/multiplayer.md#communication)) |
| `session` | bytes(16) | device ID of the active session ([SAVE-023](player-data.md#requirements)) |
| `battle` | bytes(32), optional | battle ID while in a PvP battle that allows spectators |
| `t` | time | |

### `peerlings/v1/epoch` — epoch records
Published by the operator once per epoch: the epoch record envelope
([data-formats § Epoch record](data-formats.md#epoch-record--peerlingsepoch)).

### `peerlings/v1/feed` — world feed
Rate: receivers accept at most 1 per player per minute. Meaning:
[world-feed](../gameplay/world-feed.md).

| Field | Type | Notes |
|-------|------|-------|
| `kind` | `"species-published"` \| `"shimmer-caught"` \| `"shrine-creation"` | |
| `species` | CID | |
| `ref` | CID | the registry listing (published / shrine) or the `catch` save-log entry (shimmer) |
| `name` | string ≤ 20 | the player's display name |
| `t` | time | |

### `peerlings/v1/battle/<battle ID>` — spectating
Battle ID (hex) = SHA-256(lower player ID ‖ higher player ID ‖ battle start
time as 8-byte big-endian), where player IDs are compared as strings. Published
by both fighters. Meaning: [spectating](../gameplay/spectating.md).

| `kind` | Fields |
|--------|--------|
| `"start"` | `players` [2 player IDs], `mode` (`"fair"` \| `"real"`), `teams` (two lists of team members, as in the battle `team` message) |
| `"turn"` | `turn` (uint), `seed` (bytes(32), once revealed), `actions` (all revealed actions so far, per turn: [[action of player 0, action of player 1]]), `state` (bytes(32), state hash after this turn) |
| `"end"` | `result` (`"win"` \| `"void"` \| `"forfeit"`), `winner` (player ID or null) |

### `peerlings/v1/save-wanted` — save recovery
`{ "kind": "save-wanted", "player": <player ID>, "t": <time> }`. Rate: 1 per
player per minute. Holders reply over `/peerlings/save-backup/1.0.0`.

### `peerlings/v1/creator/<player ID>` — creator notifications
Published by the operator. Meaning: [creator-feedback](../gameplay/creator-feedback.md).
`{ "kind": "caught" | "traded" | "delisted", "species": <CID>, "by": <display name or null>, "t": <time> }`.

## Direct streams

### `/peerlings/battle/1.0.0` — PvP battle

The challenger opens the stream. Messages in order (A = challenger, B = the
other player). Meaning: [pvp-battles](../gameplay/pvp-battles.md).

| # | From | `kind` | Fields |
|---|------|--------|--------|
| 1 | A | `challenge` | `mode` (`"fair"` \| `"real"`), `spectators` (bool), `t` |
| 2 | B | `accept` / `decline` | — |
| 3 | both | `team` | `members`: [map: `instanceId`, `species`, `level`, `traits`, `shimmer`, `originProof`, `saveLog`] (up to 4, team order) |
| 4 | both | `seed-commit` | `hash`: SHA-256(value) |
| 5 | both | `seed-reveal` | `value`: bytes(32) |
| 6 | both, per decision | `commit` | `turn` (uint), `hash`: SHA-256(action ‖ nonce) |
| 7 | both, per decision | `reveal` | `turn`, `action` ([data-formats § Action](data-formats.md#catch-evidence)), `nonce` (bytes(16)) |
| 8 | both, per turn | `state` | `turn`, `hash`: SHA-256 of the battle state after the turn |
| 9 | both | `end` | `result`, `winner` |

- Step 3: each side verifies the other's team (origin and ownership,
  [player-data § Verified Peerlings](player-data.md#verified-peerlings)) before
  sending step 4; on failure it sends `error` with code `"unverified"`.
- Battle seed = SHA-256(`"peerlings/pvp-seed/v1"` ‖ value of the lower player
  ID ‖ value of the higher player ID).
- Replacing a fainted Peerling is also a commit/reveal decision (`replace`
  action), so neither side sees the other's choice first.
- A `state` hash that differs from one's own → `end` with result `"void"`.
- The battle state that is hashed is defined by the battle engine: for each
  side, the active instance, each team member's HP, stat stages and charge
  flag, plus the turn number, as a DAG-CBOR map.

### `/peerlings/trade/1.0.0` — trade

Meaning: [trading](../gameplay/trading.md).

| # | From | `kind` | Fields |
|---|------|--------|--------|
| 1 | A | `propose` | `t` |
| 2 | B | `accept` / `decline` | — |
| 3 | either, any time | `offer` | `instances` [instance ID]: the full current offer of the sender; clears both confirmations |
| 4 | either | `confirm` | `offers`: SHA-256 of both current offers (A's, then B's) |
| 5 | both, after both confirmed | `transfers` | the sender's signed transfer envelopes, sharing one `trade` ID chosen by A |
| 6 | A | `done` | `entry`: CID of the transfer-log entry A appended (both sides' transfers) |
| — | either, before 6 | `cancel` | — |

B checks the entry appears in the transfer log, then both append `trade` events
to their save logs.

### `/peerlings/profile/1.0.0` — profile

Request `{ "kind": "profile-request" }` → response `{ "kind": "profile",
"profile": <CID of the profile document>, "saveLog": <address>, "heads":
[<CID>] }`. The requester then replicates the save log, which also keeps a
backup ([SAVE-020](player-data.md#requirements)).

### `/peerlings/save-backup/1.0.0` — serving a backed-up save

Opened by the player who asked on `save-wanted`. Request `{ "kind":
"save-request", "player": <player ID> }` → response `{ "kind": "save-held",
"snapshot": <CID>, "saveLog": <address>, "heads": [<CID>] }`, or `error` with
code `"not-held"`.

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
