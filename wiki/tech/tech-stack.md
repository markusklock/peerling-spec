---
title: Tech Stack
type: system
status: draft
req_prefix: STK
tags: [tech, platform, networking, rendering]
sources:
  - raw/conversations/2026-10-04-tech-stack-1.md
related:
  - wiki/decisions/D-0011-modern-web-platform-first.md
  - wiki/tech/architecture.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/realtime-networking.md
  - wiki/tech/generation-server.md
updated: 2026-10-04
---

# Tech Stack

> Canonical list of the platform technologies Peerlings uses and why. The
> guiding rule ([D-0011](../decisions/D-0011-modern-web-platform-first.md)) is
> to use modern, state-of-the-art web technology wherever it gives a real
> improvement, even at some cost in browser compatibility.

## Browser target

[proposed] Current versions of Chrome/Edge, Firefox and Safari, on desktop and
mobile. No polyfills or workarounds for older browsers. As of 2026, WebTransport
works in all of these. WebGPU works in all of them except Firefox on Linux and
Android, which is why WebGL2 is kept as a fallback (see *Rendering*).

## Networking

| Link | Technology | Status |
|------|------------|--------|
| Browser → operator server | **WebTransport** (HTTP/3 over QUIC) | [accepted] |
| Browser → operator server, second option | **WebRTC-direct** (libp2p's WebRTC browser-to-server transport). Also UDP-based, needs no TLS certificate, and covers browsers whose WebTransport implementation misbehaves | [proposed] |
| Browser ↔ browser | **WebRTC**, set up through the server's circuit relay, then direct | [proposed] (the only option; browsers can't accept WebTransport) |
| WebSockets | Not used | [accepted] |
| IP version | **IPv6 preferred**, IPv4 kept for players without IPv6 | [accepted] |
| Creation API (HTTPS) | HTTP/3 | [proposed] |

Connection details (bootstrap, relay, fallbacks) are canonical in
[ipfs-helia § Connectivity](ipfs-helia.md#connectivity).

### IPv6

[accepted] IPv6 is used wherever possible. [proposed] How:
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

[proposed] libp2p's WebTransport uses short-lived self-signed certificates.
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

[proposed]
- **WebGPU** as the primary graphics API, with automatic **WebGL2 fallback**
  where WebGPU isn't available (mainly Firefox on Linux and Android). Modern
  engines provide the fallback almost for free.
- The engine itself (e.g. three.js, Babylon.js) is the implementer's choice.

## Client runtime

[proposed]

| Area | Choice | Why |
|------|--------|-----|
| Threads | Helia, libp2p and OrbitDB run in a **Web Worker** | Networking and hashing never stall rendering |
| One node per player | The **Web Locks API** ensures only one open tab runs the player's node | Two tabs with the same identity would fight over the save log |
| Storage | **OPFS** (Origin Private File System) for the IPFS blockstore and OrbitDB data | Much faster binary reads and writes than IndexedDB, especially from a worker |
| Crypto | **Ed25519 via WebCrypto** for player identities and signatures | Built into all major browsers; no JavaScript crypto library on the hot path |
| Installable app | **PWA** with a service worker caching the app itself | The game loads offline, which fits offline play ([player-data](player-data.md#encounter-seeds)) |
| Language | **TypeScript** | Helia, libp2p and OrbitDB are TypeScript/JavaScript |

## Asset formats

[proposed]
- **3D models:** binary glTF 2.0 (`.glb`) with meshopt geometry compression
  (`EXT_meshopt_compression`) and KTX2/Basis Universal textures
  (`KHR_texture_basisu`). The files are smaller to share peer-to-peer, and the
  textures stay compressed on the GPU. Budgets: [Q-016](../open-questions.md#q-016).
- **2D images and thumbnails:** AVIF.

## Requirements

- **STK-001** [accepted] Browser ↔ server connections MUST use WebTransport; WebSockets MUST NOT be used.
- **STK-002** [accepted] The game MUST use IPv6 wherever available: the server MUST be dual-stack, and clients MUST NOT suppress IPv6 connection candidates.
- **STK-003** [proposed] The server MUST also accept WebRTC-direct connections from browsers.
- **STK-004** [proposed] Clients MUST discover the server's current multiaddrs (including WebTransport certificate hashes) at startup instead of hardcoding them.
- **STK-005** [proposed] Rendering MUST use WebGPU where available and fall back to WebGL2.
- **STK-006** [proposed] The IPFS node, libp2p and OrbitDB MUST run off the main thread, and only one tab per player MUST run the node.
- **STK-007** [proposed] The client blockstore and OrbitDB storage MUST use OPFS.
- **STK-008** [proposed] Player identity keys and signatures MUST use Ed25519 via WebCrypto.
- **STK-009** [proposed] 3D models MUST be glTF 2.0 binary with meshopt compression and KTX2 textures; 2D images MUST be AVIF.

## Open questions

[Q-016](../open-questions.md#q-016) · [Q-034](../open-questions.md#q-034)

## See also

- [Architecture](architecture.md) · [Browser IPFS node](ipfs-helia.md) · [Realtime networking](realtime-networking.md)
