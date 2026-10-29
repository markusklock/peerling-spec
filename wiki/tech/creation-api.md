---
title: Creation API
type: data
status: accepted
req_prefix: API
tags: [tech, server, api, creation]
sources:
  - raw/conversations/2026-10-05-formats-request.md
  - raw/conversations/2026-10-06-formats-approved.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-network-performance-approved.md
  - raw/conversations/2026-10-06-review-2-decisions.md
  - raw/conversations/2026-10-10-image-prompt-enhancer.md
  - raw/conversations/2026-10-10-no-self-hosted-llm.md
  - raw/conversations/2026-10-12-single-image-to-3d.md
  - raw/conversations/2026-10-27-high-severity-resolutions.md
  - raw/conversations/2026-10-28-medium-low-resolutions.md
related:
  - wiki/peerlings/image-prompting.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/generation-server.md
  - wiki/tech/data-formats.md
  - wiki/gameplay/onboarding.md
  - wiki/gameplay/creation-shrine.md
updated: 2026-10-28
---

# Creation API

> The HTTP interface between the game client and the operator server for
> everything that needs the server: creating Peerlings (starter and Creation
> Shrine), choosing an existing starter, checking names, and finding the
> server's current libp2p addresses. Everything else is peer-to-peer
> ([protocols](protocols.md)).

**Status: [accepted]**: drafted 2026-10-05, approved 2026-10-06 ([Q-047](../open-questions.md#q-047)).

## General rules

- **Transport:** HTTPS over HTTP/3 ([tech-stack](tech-stack.md#networking));
  [accepted] the server also serves HTTP/2, which browsers fall back to on
  their own (2026-10-28),
  base path `/v1`. Request and response bodies are JSON (`application/json`);
  CIDs are strings; byte values are base64url strings (no padding). Signed
  envelopes and other DAG-CBOR records inside a JSON body (e.g. `release`,
  `listing`, `origin`) are carried as base64url strings of their DAG-CBOR
  bytes, so they keep their exact bytes and signatures. Binary files (image,
  model, thumbnail, species record) are served raw with their own media types;
  DAG-CBOR responses use `application/vnd.ipld.dag-cbor`.
- **Who's asking:** every request that acts for a player is signed with the
  player's key:
  - headers `Peerlings-Player` (player ID), `Peerlings-Time` (time, ms) and
    `Peerlings-Signature` (base64url Ed25519 signature);
  - the signature covers the UTF-8 bytes of `"peerlings-api-v1\n"` ‖ method
    (upper case) ‖ `"\n"` ‖ path including any query string ‖ `"\n"` ‖ time
    (decimal digits, the same value as the header) ‖ `"\n"` ‖ the SHA-256 of
    the body as 64 lower-case hex characters (of zero bytes for an empty
    body);
  - the server rejects requests whose time is more than 60 s in the past or
    more than 10 s in the future, and replays of a signature it has seen.
- **Errors:** status code plus `{ "error": "<code>", "message": "<text>" }`.
  Codes include `rate-limited` (with `Retry-After`), `cooldown`, `name-taken`,
  `name-reserved` ([accepted] the name of an existing character, [IMG-015](../peerlings/image-prompting.md#requirements)),
  `wish-refused` ([accepted] GPT-6 Luna declined the wish; the player edits it,
  [creation-pipeline § Job handling](../peerlings/creation-pipeline.md#job-handling)),
  `invalid`, `not-found`, `job-expired`, `verification-failed`,
  `starter-taken` (the player already has a starter), `starter-job-open`
  ([accepted] a starter job is already open; the body also carries its `job`
  ID), `shrine-limit` (the
  7-day limit hasn't ended, or a shrine job or credit is already open; with
  `Retry-After` when it is the time limit).
- **Progress:** `GET /v1/jobs/{id}/events` is a Server-Sent Events stream that
  sends the job object again whenever it changes. Clients may poll
  `GET /v1/jobs/{id}` instead.

## The job object

```json
{
  "id": "job_7f3a…",
  "kind": "starter",
  "state": "IMAGE_READY",
  "queuePosition": 0,
  "wish": "a small sleepy fox made of moss that carries a lantern",
  "concept": { "summary": "…", "lore": "…", "types": ["Grass"], "nameSuggestions": ["Mossnap", "Lumifox", "Glowtail"] },
  "image": "/v1/jobs/job_7f3a…/files/image",
  "originalityChanged": false,
  "nextImageAt": 1791234567000,
  "model": null,
  "profile": null,
  "name": null,
  "listing": null,
  "origin": null,
  "releaseEntry": null,
  "error": null,
  "expiresAt": 1791238167000
}
```

[accepted] A job expires (`job-expired`) after 24 h without a request from
the player; `expiresAt` moves forward with every request. [accepted] An open
`GET /v1/jobs/{id}/events` stream counts as one long request: the job doesn't
expire while it is open, and the 24 h run from when it closes (2026-10-28,
[D-0028](../decisions/D-0028-medium-low-review-fixes.md)).

[accepted] A job in `FAILED` has `error` set and keeps the state it failed
from; `POST /v1/jobs/{id}/retry` returns it there and runs the failed step
again. A failed job stays open and can be retried until it expires.

`state` follows the job state machine in
[creation-pipeline § Job handling](../peerlings/creation-pipeline.md#job-handling).
[proposed] `originalityChanged` is true when the prompt enhancer steered the
design away from an existing character; the client then shows the fixed
notice from [image-prompting § Originality rules](../peerlings/image-prompting.md#originality-rules).
`profile` (once `PROFILE_READY`) holds the types, base stats and moves.
`listing` (once signed at stage 7e) is the registry listing envelope.
`origin` (once the new Peerling exists) is its origin attestation, for starters
and Creation Shrine creations. `releaseEntry` (shrine jobs) is the CID of the
transfer-log entry that released the offering.

## Endpoints

### Creating a Peerling

| Method and path | Body | Does |
|-----------------|------|------|
| `POST /v1/jobs` | `kind` (`"starter"` \| `"shrine"` \| [accepted] `"seed"`: operator accounts only, see below), `wish` (≤ 300 characters), `release` (shrine only: the 3 signed transfers to `"released"`; omitted when using a kept shrine credit) | Starts a job. For `starter`, fails with `starter-taken` if the player already has a starter, and with `starter-job-open` if a starter job is already open. For `shrine`, checks the offering or the player's shrine credit first ([creation-shrine](../gameplay/creation-shrine.md)) |
| `GET /v1/jobs/{id}` | — | The job object |
| `GET /v1/shrine` | — | The player's shrine status: `{ "openJob": <job ID or null>, "credit": { "expiresAt": <time>, "level": <uint: the new Peerling's level>, "releaseEntry": <CID of the offering's release entry> } or null, "nextShrineAt": <time or null: when the 7-day limit ends> }` ([accepted] `level` and `releaseEntry` added 2026-10-28) |
| `GET /v1/jobs/{id}/events` | — | Server-Sent Events, as above |
| `POST /v1/jobs/{id}/regenerate` | `wish` (optional: an edited wish) | New image; respects the 30 s cooldown (`nextImageAt`) |
| `POST /v1/jobs/{id}/accept-image` | — | Starts 3D generation, then stats and moves; [accepted] `invalid` if the image is flagged by the resemblance check ([IMG-018](../peerlings/image-prompting.md#requirements)) |
| `POST /v1/jobs/{id}/retry` | — | [accepted] Only in `FAILED`: back to the state the job failed from, running the failed step again |
| `POST /v1/jobs/{id}/back-to-image` | — | From the final review, back to image generation; keeps the name |
| `GET /v1/names/{name}` | — | `{ "available": bool, "nameKey": "…" }` |
| `POST /v1/jobs/{id}/name` | `name` | Reserves the name for as long as the job lives; `name-taken` if not free; [accepted] `name-reserved` for an existing character's name; `invalid` if the name's normalized form is empty |
| `POST /v1/jobs/{id}/finalize` | — | [accepted] The player confirms publishing (needs a name). The server signs the species record; the job goes from `PROFILE_READY` to `PUBLISHING`, and `files/record` becomes available |
| `GET /v1/jobs/{id}/files/{file}` | — | `image`, `model`, `thumbnail` or `record` (the signed species record, DAG-CBOR) |
| `POST /v1/jobs/{id}/upload` | CAR file (`application/vnd.ipld.car`) with the species record and assets | [accepted] Fallback when the server can't fetch the content peer to peer within 20 s of `published` ([network-performance](network-performance.md#fast-paths-through-the-operator)). The server checks the blocks against the expected CIDs, then continues as for `published` |
| `POST /v1/jobs/{id}/published` | `species` (CID) | Called after the client added everything to IPFS (stage 7). The server fetches, compares, pins and signs the listing; returns the job with `listing` |
| `DELETE /v1/jobs/{id}` | — | Abandons the job (frees the name; a shrine job's offering is not refunded, but the player keeps a shrine credit for 30 days: [creation-shrine](../gameplay/creation-shrine.md)) |

[accepted] **Seed species** (2026-10-28, [D-0028](../decisions/D-0028-medium-low-review-fixes.md)): a job of kind `seed`
is accepted only from player IDs on the server's list of operator accounts.
It runs the normal pipeline with no starter or shrine rules and creates no
instance (no stage 8). The species' `creator` is that operator account, whose
display name the operator chooses. The operator publishes it (stage 7) from
the game client signed in with that account.

### Starters

| Method and path | Body | Does |
|-----------------|------|------|
| `POST /v1/starters/options` | — | Returns 3 random species (CIDs) for a new player ([onboarding](../gameplay/onboarding.md#choosing-an-existing-starter)). One set per player; asking again returns the same set |
| `POST /v1/starters/choose` | `species` (one of the offered CIDs) | Returns the starter's origin attestation. Abandons the player's open starter job, if any (its name is freed; a species it already published stays published, with no instance). Asking again for the same species after success returns the same attestation; `starter-taken` if the player's starter is a different one |
| `GET /v1/starter` | — | [accepted] The player's starter status: `{ "state": "none" \| "job" \| "issued", "job": <open starter job ID or null>, "attestation": <origin attestation or null> }`. A client that lost its local data uses it to find its open job or its issued starter |
| — | — | A created starter (or Creation Shrine Peerling) gets its origin attestation in the job object's `origin` field |

[accepted] **One starter, exactly** (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md)): a player has at
most one open starter job. The server records a starter in one atomic step,
when it signs the origin attestation (stage 8 of a starter job, or
`/starters/choose`); that step fails with `starter-taken` if the player already
has one. So a lost response never costs the starter: the client asks
`GET /v1/starter` again.

### Trades

[accepted] (2026-10-27, [D-0027](../decisions/D-0027-high-severity-review-fixes.md);
[player-data § Transfer log and trades](player-data.md#transfer-log-and-trades))

| Method and path | Body | Does |
|-----------------|------|------|
| `POST /v1/trades/{trade}/expire` | `transfer` (one of the caller's own signed transfers carrying this trade ID, proving they are in the trade) | `{trade}` is the trade ID in lower-case hex. If the server's transfer log holds an entry for this trade, returns `{ "entry": <CID> }` and the trade is complete. Otherwise the server appends a signed [trade-expired record](data-formats.md#trade-expired-record--peerlingstrade-expired) and returns `{ "expired": <CID of that log entry> }` |

### Network

| Method and path | Does |
|-----------------|------|
| `GET /v1/bootstrap` | The server's current libp2p multiaddrs, including WebTransport certificate hashes ([tech-stack § WebTransport certificates](tech-stack.md#webtransport-certificates)). Same content as the `dnsaddr` record; no signature needed |

[accepted] Fast paths ([network-performance § Fast paths through the operator](network-performance.md#fast-paths-through-the-operator),
[D-0023](../decisions/D-0023-network-performance.md)). They need no player
signature; every response is signed or content-addressed and checked by the
client:

| Method and path | Does |
|-----------------|------|
| `GET /v1/epochs/latest` | The latest server-signed [epoch record](data-formats.md#epoch-record--peerlingsepoch) envelope (DAG-CBOR) |
| `GET /v1/epochs/{E}` | The server-signed epoch record for epoch E; `not-found` if the server published none |
| `GET /v1/logs/{manifest CID}/entries?after=<CID>,<CID>…` | For the OrbitDB log whose address is `/orbitdb/<manifest CID>`: a CAR file (`application/vnd.ipld.car`) of its entry blocks newer than the given heads (all entries if `after` is empty), oldest first, at most 5,000 entries per response; the header `Peerlings-More: true` means the client should ask again with the new heads. Works for the registry, transfer log, epoch log and any save log |
| `GET /v1/checkpoints/{player}` | The newest [verification checkpoint](data-formats.md#verification-checkpoint--peerlingscheckpoint) envelope (DAG-CBOR) for that player; `not-found` if none |
| `GET /v1/ipns/{name}` | The latest signed IPNS record the server holds for that name (`application/vnd.ipfs.ipns-record`) |

## Requirements

- **API-001** [accepted] The client and operator server MUST communicate for creation, starters and names through the endpoints on this page, over HTTP/3.
- **API-002** [accepted] Every request acting for a player MUST carry a fresh Ed25519 signature by the player's key as defined on this page.
- **API-003** [accepted] Job progress MUST be available as a Server-Sent Events stream and by polling.
- **API-004** [accepted] The server MUST give each player at most one starter, ever, whether created or chosen.
- **API-005** [accepted] A creation job MUST expire after 24 h without a request from the player, and its name reservation MUST last until then.
- **API-006** [accepted] The server MUST provide the fast-path endpoints and the creation upload fallback listed on this page.
- **API-007** [accepted] The server MUST allow at most one open starter job per player, MUST record a starter atomically when signing its origin attestation, and MUST answer a repeated `/starters/choose` for the issued species with the same attestation.
- **API-008** [accepted] The server MUST provide `GET /v1/starter` with the player's starter state, open starter job and issued attestation.
- **API-009** [accepted] The server MUST expire a trade on request from one of its players only if its transfer log holds no entry for that trade ID, and otherwise MUST return the entry.
- **API-010** [accepted] A job in `FAILED` MUST be retryable with `POST /v1/jobs/{id}/retry` until it expires, and an open event stream MUST count as activity for expiry.
- **API-011** [accepted] The server MUST accept jobs of kind `seed` only from its operator accounts, and MUST NOT create an instance for them.
- **API-012** [accepted] The server MUST also serve the API over HTTP/2.

## Open questions

_None at the moment._

## See also

- [Creation pipeline](../peerlings/creation-pipeline.md) · [Generation server](generation-server.md) · [Data formats](data-formats.md)
