# 2026-10-05 — Request to specify exact formats

Kind: design conversation. The LLM listed the remaining work: (1) exact message
and data formats for implementers, (2) smaller gaps. After the world details,
the designer picked (1).

## Designer's statement (verbatim)

> Lets look at 1 here

## Summary

- The LLM drafted the exact formats as three new pages, all [proposed]:
  `wiki/tech/data-formats.md` (conventions, OrbitDB databases, every record),
  `wiki/tech/protocols.md` (pubsub topics and libp2p streams) and
  `wiki/tech/creation-api.md` (the HTTP API to the operator server) (Q-047).
- Notable proposal: one Ed25519 key per player serves as player ID, libp2p
  peer ID, OrbitDB identity and IPNS name.
