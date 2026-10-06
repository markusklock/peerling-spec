# 2026-10-05/06 — Data formats approved; one key per player

Kind: design conversation. The LLM drafted the exact formats (Q-047) and asked
about three points: (1) one key for everything, (2) the phone backup flow,
(3) the new text limits (summary 200, lore 600, move names 24).

## Designer's answers (verbatim)

> 1. What are the pros and cons of the one-key-for-everything design?
> 2. approved
> 3. approved

The LLM listed the pros (simplicity, one phrase restores everything, a clear
showcase, less code) and cons (account linked to IP address across sessions,
account key used throughout the networking stack, no rotation, a custom
OrbitDB identity provider needed, brief peer-ID clashes during a restore), and
the alternative (account key plus per-session libp2p keys with a certificate).

> keep one key, sound good

## Summary of what was decided

- One Ed25519 key per player for player ID, libp2p peer ID, OrbitDB identity
  and IPNS name (→ D-0018).
- Phone backup flow and text limits approved; with that, the three format pages
  (data-formats, protocols, creation-api) are approved (Q-047).
