---
title: Network Performance, Scale and Timeouts
type: system
status: accepted
req_prefix: PERF
tags: [tech, networking, performance, ipfs, orbitdb]
sources:
  - raw/conversations/2026-10-06-network-performance.md
  - raw/conversations/2026-10-06-network-performance-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-06-review-2-decisions.md
related:
  - wiki/tech/ipfs-helia.md
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/player-data.md
  - wiki/tech/resilience.md
  - wiki/tech/realtime-networking.md
  - wiki/gameplay/encounters.md
updated: 2026-10-06
---

# Network Performance, Scale and Timeouts

> Which network requests can be slow, how big the shared data grows, and the
> failsafes that keep the game feeling fast: local-first play, racing several
> sources for the same content, prefetching, snapshots instead of replicating
> ever-growing logs, and a time budget with a visible state for every wait.

[accepted] The designer's concern (2026-10-06): IPFS can be notoriously slow,
which probably affects OrbitDB as well. Every request that can be slow needs
a failsafe so the game doesn't feel slow.

[accepted] Everything below was approved on 2026-10-06
([D-0023](../decisions/D-0023-network-performance.md)). Where it changed an
earlier rule, it says so. The latency numbers are estimates and must be
measured in an early prototype; the budgets are starting values to tune.

## Principles

[accepted]
1. **Local first.** Walking, the world, wild battles and the player's own save
   run entirely in the browser. The network only adds things: new species,
   other players, verification. Nothing in the core loop waits for it once
   the content is cached.
2. **Never depend on one source.** Content is requested from several sources
   at once, with short delays between them; the first verified answer wins
   ([Retrieval ladder](#retrieval-ladder)).
3. **Fetch before it's needed.** Anything predictable is prefetched: encounter
   candidates, guardian teams, the Peerling of the Day, opponents' teams.
4. **Every wait has a budget, a visible state and a fallback.** No request can
   hang forever, and the player always sees what is happening
   ([Waiting states](#waiting-states)).
5. **Don't replicate what grows without limit.** Browsers fully replicate only
   small OrbitDB databases. Data that keeps growing is read through signed
   snapshots and indexes ([Snapshots and indexes](#snapshots-and-indexes)).
6. **Shortcuts through the operator never become requirements.** Every fast
   path through the operator server has a fully peer-to-peer slow path, so
   nothing in [resilience](resilience.md) changes when the server is offline.
7. **Show the path.** The network panel shows where each piece of content came
   from and how long it took ([ipfs-showcase](ipfs-showcase.md)), so the fast
   paths stay honest and the peer-to-peer part stays visible.

## What can be slow

[accepted] Estimates for a desktop browser on broadband:

| Network step | Usually | Bad case | Why |
|--------------|---------|----------|-----|
| Connecting to the operator (WebTransport) | 100–300 ms | Fails if UDP is blocked; then WebRTC-direct, 1–2 s | QUIC/UDP filtered on some networks |
| Browser ↔ browser WebRTC through a relay | 1–3 s | Fails behind some NATs; the connection stays relayed | ICE negotiation; symmetric NATs |
| Relayed connection | Works | **Cut after 2 min or 128 KB** with default relay limits | libp2p circuit relay v2 defaults |
| Bitswap from a connected peer that has the block | 1 round trip + transfer: 50–300 ms for 1 MB | Peer is busy or slow | A model is 1 block (1 MiB chunks, [ipfs-helia](ipfs-helia.md#content-import-parameters)) |
| Bitswap for content no connected peer has | Needs content routing: 1–10 s | Often fails: most IPFS providers can't be dialled from a browser | Provider records mostly point to TCP/QUIC nodes |
| Delegated routing lookup | 100 ms–2 s | Rate-limited public endpoints | HTTP round trip plus the endpoint's own DHT lookup |
| IPNS resolution | 0.5–3 s | Old or missing record | DHT lookup behind delegated routing |
| OrbitDB first sync of a log | A few seconds for hundreds of entries | **Minutes** for tens of thousands | Entries are fetched block by block, following each entry's back-references; only partly parallel |
| Gossipsub message | 50–300 ms | Lost if a browser has no mesh peers | The operator joins all topics to help |
| Epoch record over pubsub | At the next epoch start | **Up to 5 min** for a client that just started | Records are only published once per epoch |

## Scale

[accepted] Sizing assumptions: up to **1,000 players online at once**,
100,000 players in total and about 20,000 species after the first year.

How each shared database grows, and whether a browser can replicate it all:

| Database | Grows by | After a year (estimate) | Full replication in a browser? |
|----------|----------|------------------------|--------------------------------|
| Registry | 1 entry per species or delisting | ~20,000 entries, ~8 MB | In the background, yes; at first start, too slow → [registry index](#1-registry-index) |
| Epoch log | 288 entries per day | ~105,000 entries | No → [fetch single records](#4-epoch-records-by-number) |
| Species stats (OrbitDB keyvalue) | 1 entry per species update | With 1,000 active species updated every epoch: **~288,000 entries per day** | **No, and it would swamp the server too** → [stats snapshot](#2-species-stats-snapshot) |
| Transfer log | 1 entry per trade or release | 50,000–500,000 entries | No → [ownership index](#3-ownership-index) |
| A player's save log | ~200 events per hour of play | ~20,000 entries after 100 hours | The owner keeps it locally; others can't fetch it all quickly → [verification checkpoints](#5-verification-checkpoints) |

## Retrieval ladder

[accepted] How the client fetches any content by CID. Each step starts after
the delay shown unless an earlier step has already delivered; steps run in
parallel, and the first **verified** block wins and cancels the rest.

| Step | Source | Starts after (player waiting) | Starts after (prefetch) |
|------|--------|------------------------------:|------------------------:|
| 1 | Local blockstore (OPFS) | 0 | 0 |
| 2 | Bitswap to connected peers (the operator is always one of them) | 0 | 0 |
| 3 | The operator's trustless HTTP gateway ([ipfs-helia § Connectivity](ipfs-helia.md#connectivity)) | 500 ms | 2 s |
| 4 | Delegated routing → dial up to 3 browser-dialable providers | 2 s | 5 s |
| 5 | Public trustless gateways (a short list shipped with the app) | 4 s | 10 s |

- If the operator connection is known to be down, steps 2 (for the operator)
  and 3 are skipped at once instead of waiting.
- All content is verified by CID whichever source delivered it, so no step
  changes the trust model.
- The network panel records which step delivered.

## Request classes

[accepted] Every network request belongs to a class. Higher classes go first,
and a pending interactive request pauses background work.

| Class | Examples | Parallel requests | Timeout | Retries |
|-------|----------|------------------:|--------:|---------|
| **Interactive:** the player is waiting | Opening a card's model, verifying a trade or PvP opponent, joining a battle, publishing a creation | 6 | 15 s, then a message with *Retry* | On request |
| **Prefetch:** needed soon | Encounter candidates, guardian teams, Peerling of the Day, followers, gallery models | 3 | 30 s per item | After 30 s, 2 min, 10 min |
| **Background:** nobody is waiting | OrbitDB sync, peer save backups, save replication, providing content | 2, at most ~25% of measured bandwidth | None (resumable) | Continuous |

## Waiting states

[accepted] What the player sees while something loads:

| Waiting time | Shown |
|--------------|-------|
| Under 300 ms | Nothing |
| 300 ms – 2 s | A small spinner on the thing that is loading |
| Over 2 s | A short line saying what is loading and from where, e.g. *"Fetching Mossnap from 3 peers…"* (this doubles as an IPFS showcase) |
| Timeout | A friendly message with *Retry*; the rest of the game keeps working |

Content that isn't loaded yet uses **placeholders**, never an empty screen:
a glowing orb for a missing model, the thumbnail before the model, a
silhouette avatar before a player's appearance has loaded, and "—" for stats.

## Fast paths through the operator

[accepted] HTTP endpoints on the operator server (exact requests and responses:
[creation-api § Network](creation-api.md#network)). All of them return signed or
content-addressed data, so clients check them exactly as they would check the
same data from peers. Each has a peer-to-peer slow path.

| Endpoint | Returns | Slow path when the server is offline |
|----------|---------|----------------------------------------|
| `GET /v1/epochs/latest` | The latest server-signed epoch record | Wait for pubsub; client-derived records ([player-data § Encounter seeds](player-data.md#encounter-seeds)) |
| `GET /v1/epochs/{E}` | The server-signed record for epoch E | Search the epoch log |
| `GET /v1/logs/{address}/entries?after=<heads>` | A CAR file with the log entries after the given heads (OrbitDB entry blocks) | Normal OrbitDB sync |
| `GET /v1/checkpoints/{player}` | The newest [verification checkpoint](#5-verification-checkpoints) for that player | Full verification |
| `GET /v1/ipns/{name}` | The latest signed IPNS record the server republishes ([sharing § Player profile](../gameplay/sharing.md#player-profile)) | Delegated routing / DHT |
| `POST /v1/jobs/{id}/upload` | (creation only) The client uploads its new species as a CAR when the server can't fetch it peer to peer within 20 s ([creation-pipeline § Stage 7](../peerlings/creation-pipeline.md#stage-7--publish)) | None needed: creation needs the server anyway |

The log endpoint matters most: it turns a sync of thousands of entries, block
by block, into one download. The client checks every entry's hash and
signature before adding it to its OrbitDB storage, as OrbitDB itself would.

## Snapshots and indexes

[accepted] Signed, content-addressed summaries that the server publishes so
browsers never have to replicate the large databases. The epoch record names
their root CIDs, so they are as trustworthy as the epoch record itself. Exact
formats: [data-formats § Snapshots and indexes](data-formats.md#snapshots-and-indexes).

### 1. Registry index
- A compact, append-only list of every registry entry in `seq` order:
  `[seq, species CID, type indices]`, or `[seq, species CID, null]` for a
  delisting, which is its own entry as in the registry. About 45 bytes per entry.
- Split into **chunks of 1,000 entries**. A full chunk never changes, so it is
  cached forever and served by any peer. The epoch record names the index
  manifest (`registryIndex`), which lists the chunks; a single CID keeps the
  epoch record within the 4 KiB pubsub message limit.
- A new player fetches about 20 small chunks (~1 MB in total for 20,000
  species) in a second or two, and can start encounters right away. It holds
  everything candidate selection needs: `seq`, types and delistings.
- The OrbitDB registry stays the source of truth and still replicates fully
  in the background, so names and thumbnails are available and the registry
  survives without the server. Clients check that the index matches the
  registry entries they have.
- This replaced the earlier rule that a client waits for a full registry sync
  before starting encounters ([player-data § Encounter seeds](player-data.md#encounter-seeds)):
  the index is enough.

### 2. Species stats snapshot
- Instead of the OrbitDB keyvalue database, the server publishes the stats as
  a **sharded map** every 12 epochs (1 hour): 256 shards by the first byte of
  SHA-256(species CID), each a DAG-CBOR map of species CID →
  [stats](data-formats.md#species-stats--peerlingsspecies-stats). The root
  goes in the epoch record (`statsRoot`).
- A species card fetches only its shard (a few KB).
- Live creator notifications stay on pubsub, unchanged.
- This **replaced CFB-002**, which made the stats an OrbitDB database
  ([creator-feedback](../gameplay/creator-feedback.md#species-stats-snapshot)).
  The keyvalue op-log would grow by up to ~288,000 entries a day, too much
  for browsers and a burden for the server. OrbitDB stays showcased
  by the registry, transfer log, epoch log and save logs.

### 3. Ownership index
- Every 12 epochs the server publishes a sharded map of **instance ID → CID of
  the transfer-log entry holding its latest transfer**, plus the list of flagged players. The root goes in
  the epoch record (`ownersRoot`).
- A verifier looks up an instance in its shard, then walks the transfer chain
  backwards by CID to its origin (chains are short), and checks transfer-log
  entries newer than the snapshot (from the log endpoint, or live from
  pubsub).
- Double trades are still detected by anyone who replicates the full transfer
  log, including the server. Without the server, verifiers use the last
  ownership index they know (client-derived epoch records copy its root) plus
  the transfer-log entries newer than its `heads`; only without any index do
  they sync the full log (slow path).

### 4. Epoch records by number
- Verifiers and features that need an older epoch record (guardian weeks,
  Peerling of the Day, catches) fetch it with `GET /v1/epochs/{E}`, or from the
  catch evidence itself.
- Catch evidence that uses a **client-derived** record also includes the
  server-signed record its registry height was copied from (`baseRecord`), so
  verifiers never have to search the epoch log.

### 5. Verification checkpoints
- The server already replicates and verifies every save log
  ([creator-feedback](../gameplay/creator-feedback.md)). Each time it has
  verified a player's log up to some entry, it signs a **checkpoint**:
  `{ player, upTo (entry CID), nextEncounter, lastEpoch, invalid (instance
  IDs whose origin failed) }`.
- The player's client fetches its newest checkpoint and appends it to its own
  save log as a `checkpoint` event. Anyone fetching the save log then finds a
  recent checkpoint near the head.
- A verifier (trade partner, PvP opponent, world feed) accepts the checkpoint
  for everything up to `upTo` and verifies only the entries after it, usually
  a few hundred at most. Verifying a player then takes seconds, not minutes.
- **Trust:** this trusts the operator's signature for the older part of a log,
  as players already trust it for species listings. Full verification stays
  possible for anyone, and is used when there is no checkpoint (e.g. the
  server has been offline a long time).

### 6. Light verification for the world feed
A shimmer-catch feed message makes every online client verify one catch.
Receivers do a **light check**: fetch the single `catch` entry by its CID and
replay its battle from the evidence, plus the checkpoint if there is one. They
skip the full encounter-number history and the candidate-list check, verify at most 2 feed messages at a
time, and drop a message that isn't verified within 30 s. A feed message is
shown only, never trusted for ownership, so a light check is enough.

## Operation by operation

[accepted]

| Operation | Needs from the network | Failsafe | Player sees |
|-----------|------------------------|----------|-------------|
| **Starting the game** | Operator connection, an epoch record, the registry index, the local save | The world loads from the local save at once. Epoch record from `GET /v1/epochs/latest` (budget 2 s), otherwise pubsub or client-derived. Encounters switch on as soon as an epoch record and the registry index are there | The world straight away; a small "Connecting…" indicator until online |
| **Walking, presence, emotes** | Pubsub | Nothing waits; players not heard from for 15 s fade out | — |
| **Other players' looks and followers** | Appearance map, species models | Prefetch class; silhouette avatar and glowing-orb follower until loaded | Placeholders that fill in |
| **Wild encounter** | Candidate species record + model | Keep at least 3 upcoming encounters ready. Fetch all 5 candidates' species records (tiny), but **models one at a time in list order**, moving on only if one fails or takes over 5 s. This cuts the cost of an encounter from ~3 MB to ~0.6 MB and fits the "earliest ready in the list" rule ([encounters § Candidates](../gameplay/encounters.md#candidates)) | Instant; if nothing is ready, the 1.5 s battle intro covers the wait, then up to the existing 10 s, then *"The wild Peerling slipped away"* and the encounter number isn't used |
| **Wild battle and catching** | Nothing | Fully local; the save log syncs in the background | — |
| **Species card** | Record, thumbnail, model, stats shard | Shown at once from cache; model and stats fill in | Thumbnail first, then the 3D model |
| **View profile** | Profile, team, save log | The profile screen comes from the direct `/peerlings/profile/1.0.0` stream (one round trip). The full save log (the peer backup) is fetched in the background | Profile in under a second |
| **Starting PvP or a trade** | WebRTC connection, the other's team, verification | Dial early: when a player faces another within 3 tiles or opens the menu. If the direct upgrade fails, stay on the operator relay ([Connections](#connections)). Verification starts as soon as the request is sent, while the other player is still deciding, using checkpoints. Budget 15 s | Progress: *"Verifying Mia's Peerlings (2 of 4)…"*; on timeout, *Retry* or cancel |
| **PvP turns** | Small messages | Existing 30 s turn timer and 60 s resume ([battle § PvP turn timer](../gameplay/battle.md#pvp-turn-timer)) | — |
| **Spectating** | Pubsub, both teams' models | Prefetch class; orbs until loaded | — |
| **World feed** | Pubsub, light verification | [Light verification](#6-light-verification-for-the-world-feed); unverified messages are dropped | Messages may appear a few seconds late |
| **Guardian site** | 4 species models | Prefetched when the player is within 300 m of the site | Plinths fill in before the player arrives |
| **Peerling of the Day** | 1 species model | Prefetched at start | — |
| **New Peerlings gallery** | 12 thumbnails, 12 models | Thumbnails at start; models when within 200 m of the hub | — |
| **Publishing a creation** | Server fetches the client's content (stage 7d) | If the server hasn't got everything within 20 s, the client uploads a CAR file over HTTP; the server checks it is byte-identical as before ([CRE-020](../peerlings/creation-pipeline.md#requirements)) | One progress bar; no failure from slow peer-to-peer |
| **Save replication** | Background push to the server | Never blocks; unsent events wait and are retried; the backup file is the last resort | "Saved locally · syncing" in the network panel |
| **Account recovery** | The save log | Log endpoint (one download), plus `save-wanted` answers within 10 s | Progress bar |
| **Shared links (viewer)** | IPNS record, profile, species | IPNS from `GET /v1/ipns/{name}` first, delegated routing in parallel after 1 s; content via the retrieval ladder | — |

## Connections

[accepted]
- **The operator connection** is kept open at all times and protected from
  connection pruning. If it drops, the client reconnects with backoff (1, 2,
  4, 8, then every 30 s), trying WebTransport, then WebRTC-direct.
- **Connection limit:** about 60 open connections, preferring the operator,
  nearby players and peers that recently delivered content.
- **Dialling early:** the client dials a nearby player when a direct
  interaction becomes likely (they are within 3 tiles and one faces the
  other, or the interaction menu opens). At most 5 of these early dials at a
  time.
- **Relay limits:** a relay only sees an encrypted connection, not the
  protocols inside it, so its limits apply per relayed connection. The
  operator's circuit relay **must** allow each relayed connection to last at
  least 60 minutes and carry at least 4 MB; the default limits (2 minutes,
  128 KB) would cut a PvP battle short. To stay within that, clients use a
  relayed connection **only** for the game's own streams (battle, trade,
  profile, save-backup, phone-backup): never for gossipsub, Bitswap or OrbitDB
  sync. Large content (models, save logs) comes from the operator, a gateway
  or a direct connection instead.
- Public relays (used while the server is offline) keep their default limits.
  So does a battle that never gets a direct connection while the server is
  offline: it may be cut off, and the resume rule then applies.

## Bandwidth and server load

[accepted] Rough figures for planning:

- **A player** downloads presence (up to ~24 KB/s in a crowd,
  [realtime-networking § Presence](realtime-networking.md#presence)) and new
  species: about 40 encounters an hour × half of them uncached × ~0.6 MB ≈
  **12 MB per hour** with one-at-a-time model prefetching.
- **The operator**, with 1,000 players online, serves about 12 GB per hour
  (~27 Mbit/s) of encounter species content if no peer helps, plus the
  smaller amounts for followers, galleries, guardian teams, the Peerling of the
  Day and opponents' teams; say **15–20 GB per hour (35–45 Mbit/s)** in all.
  In practice other players' nodes serve popular species.
- **Gossip:** with the operator on every presence topic, 1,000 moving players
  send it about 3,000 messages per second (~5 Mbit/s in), and it forwards each
  to several mesh peers: roughly **25 Mbit/s out**.
- **Optional:** species content never changes, so the operator's trustless
  gateway can sit behind a free or cheap CDN that caches it forever.

## Measuring

[accepted] The client records, for every fetch, the time and the source that
delivered it. The network panel shows recent ones (*"Mossnap: 180 ms from
Mia's node"*). An early prototype must measure the numbers in
[What can be slow](#what-can-be-slow) and tune the budgets on this page.

## Requirements

- **PERF-001** [accepted] The core loop (walking, the world, wild battles with cached species, the player's own save) MUST work without waiting for the network.
- **PERF-002** [accepted] Content fetches MUST follow the [retrieval ladder](#retrieval-ladder), and every network request MUST belong to a [request class](#request-classes) with its timeout.
- **PERF-003** [accepted] Every wait over 300 ms MUST show a waiting state as in [Waiting states](#waiting-states), and content not yet loaded MUST use placeholders.
- **PERF-004** [accepted] The operator MUST provide the [fast paths](#fast-paths-through-the-operator), and every fast path MUST have a peer-to-peer fallback.
- **PERF-005** [accepted] Browsers MUST NOT need to fully replicate the epoch log, species stats or transfer log; the server MUST publish the [snapshots and indexes](#snapshots-and-indexes).
- **PERF-006** [accepted] The operator's circuit relay MUST allow relayed connections of at least 60 minutes and 4 MB, and clients MUST use relayed connections only for the game's own streams.
- **PERF-007** [accepted] Encounter prefetching MUST fetch candidate models one at a time in list order, keeping at least 3 upcoming encounters ready.

## Open questions

_None at the moment._

## See also

- [IPFS in the browser](ipfs-helia.md) · [OrbitDB registry](orbitdb-registry.md) · [Player data](player-data.md) · [Resilience](resilience.md) · [Realtime networking](realtime-networking.md)
