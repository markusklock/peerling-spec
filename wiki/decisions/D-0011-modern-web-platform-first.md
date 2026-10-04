---
title: "D-0011: Modern web platform first"
type: decision
status: accepted
tags: [tech, platform, networking]
sources:
  - raw/conversations/2026-10-04-tech-stack-1.md
related:
  - wiki/tech/tech-stack.md
  - wiki/tech/ipfs-helia.md
updated: 2026-10-04
---

# D-0011: Modern web platform first

**Status:** accepted (2026-10-04)

## Context
Peerlings is mainly a tech demo for IPFS and peer-to-peer technology. Older
web technology (WebSockets over TCP, IPv4-only networking, WebGL) works in more
browsers, but newer technology gives better performance and peer-to-peer
connectivity, and better shows what the modern web can do.

## Decision
- [accepted] Prefer modern, state-of-the-art web technology wherever it gives
  a real improvement, even if some older browsers or devices can't run the game.
- [accepted] **WebTransport instead of WebSockets.** Browsers connect to the
  operator server over WebTransport (HTTP/3 over QUIC), as IPFS browser nodes
  do. WebSockets are not used.
- [accepted] **IPv6 wherever possible**, to improve peer-to-peer connectivity.
- Clarification recorded with the decision: a browser can only open (dial) a
  WebTransport connection to a server; it can't accept one. Browser-to-browser
  connections therefore use WebRTC, the standard way libp2p connects two
  browsers. Both run over UDP and both benefit from IPv6.

## Consequences
- The list of technologies chosen under this principle is kept in
  [tech-stack](../tech/tech-stack.md); transport details are in
  [ipfs-helia § Connectivity](../tech/ipfs-helia.md#connectivity).
- WebTransport has worked in all major browsers since Safari 26.4 (March 2026),
  so dropping WebSockets costs little. Implementations still differ in details
  (e.g. certificate handling), so this needs testing on every browser.
- The server's libp2p node must be able to *listen* on WebTransport, which not
  every libp2p implementation supports equally well.

## Alternatives considered
- Secure WebSockets as the browser ↔ server transport (the earlier draft):
  most compatible, but TCP-based and older. Rejected by the designer.
