# 2026-10-04 — Answers to open questions (round 2)

Kind: design conversation. The designer answered the seven numbered questions
asked after round 1. The questions are paraphrased; the answers are verbatim.

## Questions asked (paraphrased)

1. Move slots: keep three (quick/strong/signature) or add a fourth support slot? (Q-024)
2. PvP cheating: is level normalization + no farmable rewards enough? (Q-025)
3. Trade duplication: accept it, or have the server notarize trades? (Q-026)
4. Start battles/trades only face to face, or also remotely? (Q-028)
5. Multiplayer scale: how many visible players; chat or just emotes? (Q-027)
6. World size: endless, or large but finite? (Q-011)
7. Stat numbers: base-stat total and per-stat bounds? (Q-023)

## Designer's answers (verbatim)

> 1. Keep 3.
> 2, 3. What alternatives do we have to keep saves in the browser? Keeping them in OrbitDB? Any other IPFS-related alternative?
> 4. only when next to each other
> 5. No chat, just emotes.
> 6. Large but finite
> 7. Give me suggestions

## Summary of what was decided

- Exactly three move slots: quick, strong, signature. No support slot (Q-024
  partly; move-use limits still open).
- PvP battles and trades can only be started when the two players are next to
  each other in the world (Q-028).
- No chat; players communicate only with emotes (Q-027 partly; scale numbers
  still open).
- The world is large but finite (Q-011).
- Not decided, the designer asked for input:
  - Alternatives to keeping saves only in the browser, e.g. OrbitDB or other
    IPFS-based storage (relates to Q-014, Q-025, Q-026). The analysis was filed
    as `wiki/tech/player-data.md`.
  - Suggested stat numbers (Q-023). The suggestion was filed in
    `wiki/peerlings/peerling-species.md` and `wiki/gameplay/battle.md`.
