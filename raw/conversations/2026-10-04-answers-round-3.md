# 2026-10-04 — Answers to open questions (round 3)

Kind: design conversation. The designer answered the four numbered questions
asked after round 2. The questions are paraphrased; the answers are verbatim.

## Questions asked (paraphrased)

1. Saves: the full recommendation (per-player OrbitDB save log + server-signed
   catches and trades + normalized PvP levels), the cheaper IPNS-only option,
   or something in between? (Q-014, Q-025, Q-026)
2. Approve the suggested stat numbers (total 320, 40–130, step 5) and the
   damage model (levels 1–50, simplified Pokémon formula, 1.5× same-type
   bonus)? If so, PvP level is 50. (Q-023)
3. Move use: unlimited, limited per battle (PP), or stamina? (Q-024)
4. Are 3 m for "next to each other", the 8 emotes, and a 4 km × 4 km world
   fine?

## Designer's answers (verbatim)

> 1. The recommendation sound good. So the player save will contain the list of Peerlings the player has and their current level?
> 2. approved
> 3. without limit
> 4. yes

## Summary of what was decided

- The recommended player-data design is accepted: the save is a per-player,
  player-signed OrbitDB event log replicated and pinned by the server; the
  identity key can be recovered with a recovery phrase; the server verifies
  catches by replaying the battle and signs them; trades are recorded in a
  server-written ownership ledger; only verified Peerlings can be traded or
  used in PvP; PvP uses normalized levels (→ D-0009; resolves Q-014, Q-025,
  Q-026).
- The designer asked what the save contains. Answer: yes, the list of owned
  Peerlings with their current level (and XP, nickname, verification), plus
  team, position, Peerdex, profile, etc. The answer was filed in
  `wiki/tech/player-data.md § Save contents`.
- The stat numbers and damage model are approved: total 320, per-stat 40–130,
  step 5; levels 1–50; PvP at level 50 (resolves Q-023).
- Moves can be used without limit (resolves Q-024).
- Accepted: "next to each other" = within 3 m; the 8 proposed emotes; a world
  of 4 km × 4 km.
