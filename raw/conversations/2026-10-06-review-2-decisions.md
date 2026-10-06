# 2026-10-06 — Second review: design decisions

Kind: design conversation. After the second full review, the LLM listed 14
questions needing a decision, each with a recommendation:

1. World generation identical in every browser: integer maths for tile kind,
   height and biome; fixed world seed per generator version; floats only for
   visuals.
2. drand network: quicknet with an exact round formula; drop the
   server-generated randomness fallback.
3. Client-derived epoch records: valid only if copying a server-signed record
   at least 2 epochs older; for day and week records the signed record wins;
   the remaining small gap accepted.
4. Exact "seen" set for the novelty weight (starters, shrine creations and
   trades count); Peerling of the Day record in catch evidence.
5. Trade integrity: identical re-posted transfers never conflict; no cancel
   after signatures are exchanged; either side may record; `prev` points to
   the transfer-log entry.
6. PvP: recoil not applied when the hit ended the battle; draw after 200 turns.
7. Forged forfeits: a loser-signed win beats a forfeit claim; conflicting
   forfeit claims cancel; in-game profile counts only verified wins.
8. Shrine: one open job or credit at a time; failed job gives a credit;
   shrine status endpoint; unverified offering levels accepted as a known gap.
9. First-find credit final once announced.
10. Guardians: XP only for a win, at the end; smaller team if fewer than 4
    species; delisted species keep their place for the week (silhouette).
11. Peerling of the Day delisted mid-day: boost stops, pedestal empty.
12. Block swaps that would leave only fainted Peerlings in the team.
13. "Show my follower" off also hides it from others.
14. World edge: 60 m ocean; guardian sites in the nearest area with walkable
    land; the spawn hub is the hexagon's landmark and rest point.

## Designer's answer (verbatim)

> Approved

## Summary of what was decided

All 14 recommendations accepted as written. Implementation detail added:
drand quicknet parameters (chain hash, scheme, period, genesis, public key)
taken from the official endpoint; round for epoch E = (E × 300 −
1692803367) ÷ 3 + 1, emitted exactly at the epoch start. Verification
checkpoints gain `state` (a server-computed snapshot) so verifiers can get the
seen set; encounter events are written in the order battle-result, catch, seen.
