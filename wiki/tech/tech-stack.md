---
title: Tech Stack
type: system
status: accepted
req_prefix: STK
tags: [tech, platform, networking, rendering]
sources:
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-tech-stack-2.md
  - raw/conversations/2026-10-05-asset-budgets-request.md
  - raw/conversations/2026-10-05-approvals-q016-q038.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-peer-save-backups.md
  - raw/conversations/2026-10-05-phone-backup.md
  - raw/conversations/2026-10-05-phone-backup-approved.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-10-image-prompt-enhancer.md
related:
  - wiki/decisions/D-0011-modern-web-platform-first.md
  - wiki/tech/architecture.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/realtime-networking.md
  - wiki/tech/generation-server.md
updated: 2026-10-10
---

# Tech Stack

> Canonical list of the platform technologies Peerlings uses and why. The
> guiding rule ([D-0011](../decisions/D-0011-modern-web-platform-first.md)) is
> to use modern, state-of-the-art web technology wherever it gives a real
> improvement, even at some cost in browser compatibility.

## Browser target

[accepted] **Desktop only** for now: current versions of Chrome/Edge, Firefox
and Safari on desktop operating systems. Mobile is not a focus; the game may
happen to run there, but it isn't designed, tested or controlled for touch.
[accepted] No polyfills or workarounds for older browsers. The Peerlings Viewer
([sharing](../gameplay/sharing.md)) is the exception to desktop-only: [accepted] it
must also work in current mobile browsers, because it carries the phone backup.

As of 2026, WebTransport works in all of these browsers. WebGPU works in all of
them except Firefox on Linux, which is why WebGL2 is kept as a fallback (see
*Rendering*).

## Networking

| Link | Technology | Status |
|------|------------|--------|
| Browser → operator server | **WebTransport** (HTTP/3 over QUIC) | [accepted] |
| Browser → operator server, fallback | **WebRTC-direct** (libp2p's WebRTC browser-to-server transport), used when WebTransport fails. Also UDP-based, needs no TLS certificate | [accepted] |
| Browser ↔ browser | **WebRTC**, set up through the server's circuit relay, then direct | [accepted] (the only option; browsers can't accept WebTransport) |
| WebSockets | Not used | [accepted] |
| IP version | **IPv6 preferred**, IPv4 kept for players without IPv6 | [accepted] |
| Creation API (HTTPS) | HTTP/3 | [accepted] |

Connection details (bootstrap, relay, fallbacks) are canonical in
[ipfs-helia § Connectivity](ipfs-helia.md#connectivity).

### IPv6

[accepted] IPv6 is used wherever possible. [accepted] How:
- The operator server is dual-stack: it has public IPv6 and IPv4 addresses,
  listens on both, and publishes multiaddrs for both. Its DNS names have AAAA
  records.
- Browser ↔ browser WebRTC gathers IPv6 candidates along with IPv4 ones, and
  ICE (the connection-setup process inside WebRTC) prefers IPv6 when both work.
  Many home IPv6 connections have no NAT, so two players can connect
  directly far more often than over IPv4. The client must not filter out IPv6
  candidates.
- The STUN servers used by WebRTC are dual-stack.
- No TURN servers: when a direct connection fails, the fallback is the libp2p
  circuit relay on the operator server.

### WebTransport certificates

[accepted] libp2p's WebTransport uses short-lived self-signed certificates.
Browsers only accept such certificates by their hash and only if they are valid
for at most 14 days. The hash is part of the server's multiaddr (`/certhash/…`),
so the server's addresses change every time its certificate rotates. Clients
therefore must not hardcode the server's multiaddrs. They look them up at
startup from a DNS `dnsaddr` TXT record (or an HTTPS endpoint) that the server
keeps up to date.

### Server implementation note

The server's libp2p node must *listen* on WebTransport and WebRTC-direct. Not
every libp2p implementation supports listening equally well. go-libp2p (used by
Kubo) has supported WebTransport listening for longest, while js-libp2p
historically focused on dialling from browsers. Implementers must check the
current state. A Kubo/go-libp2p node in front of the server's JavaScript
services (OrbitDB) is an acceptable setup.

## Rendering

- [accepted] **WebGPU** as the primary graphics API, with a **WebGL2
  fallback** where WebGPU isn't available (mainly Firefox on Linux).
  [accepted] Modern engines provide the fallback almost for free.
- The engine itself (e.g. three.js, Babylon.js) is the implementer's choice.

## Client runtime

[accepted] Approved 2026-10-04.

| Area | Choice | Why |
|------|--------|-----|
| Threads | Helia, libp2p and OrbitDB run in a **Web Worker** | Networking and hashing never stall rendering |
| One node per player | The **Web Locks API** ensures only one open tab runs the player's node | Two tabs with the same identity would fight over the save log |
| Storage | **OPFS** (Origin Private File System) for the IPFS blockstore and OrbitDB data | Much faster binary reads and writes than IndexedDB, especially from a worker |
| Crypto | **Ed25519** for player identities and signatures, via WebCrypto where the browser supports it ([accepted] 2026-10-06) | WebCrypto needs no JavaScript crypto library on the hot path; a library fallback covers browsers without Ed25519 in WebCrypto, and some libp2p code paths use their own implementation |
| Persistent storage | [accepted] The client calls `navigator.storage.persist()` at first launch, and again after the player installs the PWA | See [Does stored data survive a restart?](#does-stored-data-survive-a-restart) |
| Installable app | **PWA** (installable desktop web app) with a service worker caching the app itself | The game loads offline, which fits offline play ([player-data](player-data.md#encounter-seeds)) |
| Language | **TypeScript** | Helia, libp2p and OrbitDB are TypeScript/JavaScript |

### Does stored data survive a restart?

Yes. OPFS (where the IPFS blocks, OrbitDB logs, save backups and key live) is
persistent storage: it survives closing the browser and restarting the
computer, as long as browser data isn't cleared. Two caveats:
- **Eviction under pressure.** By default, browser storage is "best effort":
  when the disk runs low, the browser may delete an origin's data. Asking for
  *persistent* storage (`navigator.storage.persist()`) prevents that. Browsers
  grant it more readily to installed web apps, which is one more reason the
  game is a PWA.
- **Safari's 7-day rule.** Safari deletes a website's stored data if the site
  isn't used for 7 days. Web apps added to the Dock count as installed and keep
  their data longer, but rules differ per browser and change over time, so
  implementers must check the current behaviour.

The player's own data is protected by the recovery phrase, the backup file and
the server's copy anyway.

## Asset formats

[accepted] Approved 2026-10-04. Size budgets: [Asset budgets](#asset-budgets).
- **3D models:** binary glTF 2.0 (`.glb`) with meshopt geometry compression
  (`EXT_meshopt_compression`) and KTX2/Basis Universal textures
  (`KHR_texture_basisu`). The files are smaller to share peer-to-peer, and the
  textures stay compressed on the GPU. Budgets: [Asset budgets](#asset-budgets).
- **2D images and thumbnails:** AVIF.

## Asset budgets

[accepted] Approved 2026-10-05.

**What drives the numbers:**
- Every wild encounter prefetches candidate species (records for all 5, models
  one at a time, [network-performance](network-performance.md#operation-by-operation)),
  and a PvP battle downloads the opponent's team of 4, all peer-to-peer. Small files mean
  encounters start instantly and more players can serve each species.
- On screen, Peerlings are small: about 30–40 m of world is visible while
  exploring, and even the closer battle camera shows a Peerling at roughly
  300–400 px tall on a 1080p screen ([visual-style](../world/visual-style.md#camera)).
- The style is colorful and stylized, not photorealistic, so fine surface
  detail (normal maps, high-res textures) adds little.

| Asset | Budget | Typical | Notes |
|-------|--------|---------|-------|
| 3D model (`.glb`) | **≤ 1 MB** | ~500 KB | Meshopt-compressed geometry, KTX2 texture |
| Triangles | **≤ 20,000** | 10,000–15,000 | Image-to-3D output is decimated to fit ([CRE-013](../peerlings/creation-pipeline.md#requirements)) |
| Materials and textures | **1 material, 1 base-color texture, 1024 × 1024** | | No normal or roughness maps; the stylized shading doesn't need them. Power-of-two size, mipmapped |
| 2D image (species card) | **≤ 150 KB**, 1024 × 1024 AVIF with alpha | ~80 KB | The image the player accepted in creation, on a transparent background ([image-prompting](../peerlings/image-prompting.md#image-checks-and-clean-up)) |
| Thumbnail | **≤ 15 KB**, 256 × 256 AVIF | ~8 KB | Rendered from the 3D model; used in lists and the registry |
| Species record (DAG-CBOR) | **≤ 16 KB** | ~8 KB | Lore and descriptions have length limits; about half is the image prompts kept for provenance |
| Registry entry | **≤ 1 KB** | ~400 B | |
| **Whole species** | **≤ 1.2 MB** | **~0.6 MB** | |

What that means in practice:
- An uncached encounter costs about 0.6 MB with typical species (one model
  plus five small records), at most about 1.3 MB; most species near the spawn
  will already be cached.
- A PvP battle against an unknown team costs about 2.5 MB with typical
  species, at most 4.8 MB.
- **Client cache:** the browser keeps up to **1 GB** of game content
  (roughly 1,500 species at typical size) and evicts the least recently used
  beyond that, except the player's own creations and collection
  ([NODE-004](ipfs-helia.md#requirements)).
- **Server storage:** about 0.6 MB per species, so even 100,000 species are
  about 60 GB pinned.

**Enforcement:** the server makes every asset fit before publishing. Clients
refuse to download anything above the budget, which also protects players from
oversized files.

## Requirements

- **STK-001** [accepted] The browser's libp2p connections to the server MUST use WebTransport, falling back to WebRTC-direct (STK-003); WebSockets MUST NOT be used. (The creation API is plain HTTPS over HTTP/3.)
- **STK-002** [accepted] The game MUST use IPv6 wherever available: the server MUST be dual-stack, and clients MUST NOT suppress IPv6 connection candidates.
- **STK-003** [accepted] The server MUST also accept WebRTC-direct connections from browsers, and clients MUST fall back to WebRTC-direct when WebTransport fails.
- **STK-004** [accepted] Clients MUST discover the server's current multiaddrs (including WebTransport certificate hashes) at startup instead of hardcoding them.
- **STK-005** [accepted] Rendering MUST use WebGPU where available and fall back to WebGL2.
- **STK-006** [accepted] The IPFS node, libp2p and OrbitDB MUST run off the main thread, and only one tab per player MUST run the node.
- **STK-007** [accepted] The client blockstore and OrbitDB storage MUST use OPFS.
- **STK-008** [accepted] Player identity keys and signatures MUST use Ed25519, through WebCrypto where the browser supports it.
- **STK-009** [accepted] 3D models MUST be glTF 2.0 binary with meshopt compression and KTX2 textures; 2D images MUST be AVIF.
- **STK-010** [accepted] The game MUST target current desktop versions of Chrome/Edge, Firefox and Safari; mobile is not a target.
- **STK-011** [accepted] The client MUST be written in TypeScript and be installable as a PWA.
- **STK-012** [accepted] Every species asset MUST fit the [asset budgets](#asset-budgets); clients MUST refuse content that exceeds them.
- **STK-013** [accepted] The client content cache MUST be capped at 1 GB, evicting least-recently-used content except the player's own creations, team and collection.
- **STK-014** [accepted] The client MUST request persistent storage (`navigator.storage.persist()`) and SHOULD encourage installing the PWA.

## Open questions

_None at the moment._

## See also

- [Architecture](architecture.md) · [Browser IPFS node](ipfs-helia.md) · [Realtime networking](realtime-networking.md)
