# 2026-10-06 — Decisions on the consistency-review findings

Kind: design conversation. After a full consistency review the LLM listed 32
findings needing the designer's decision, each with a proposed fix:
A (would break): 1 trade atomicity; 2 deterministic battle math (to be drafted
for approval); 3 exact hash encodings; 4 rules versioning; 5 verification gaps
(epoch in battle-result, client-derived epoch height, delisting keeps species
records, tombstones take a seq); 6 session heartbeat.
B (edge cases): 7 PvP HP; 8 PvP timeouts; 9 team minimum; 10 Peerdex for
non-wild Peerlings; 11 first rest point; 12 tutorial encounter; 13 blocking;
14 emote IDs; 15 crowded tiles and busy players; 16 move details; 17 world feed
for shrine creations; 18 signpost level range; 19 duplicate-name display.
C (creation lifecycle): 20 finalize step; 21 job lifetime; 22 edited wish;
23 back-to-image; 24 shrine credit; 25 one starter per player; 26 full instance
data in trade offers.
D (world): 27 inner ring too small for 12 biome areas; 28 drop world growth.
E (small): 29 relax STK-008; 30 sizeClass and temperament fields; 31 exact
presentation formulas; 32 mark fully approved pages as accepted.

## Designer's answers (verbatim)

> 12 - could maybe both players publish the winner and add it to a player stat like "number of PvP wins" or similar?
> 27 - perhaps we can build the world as 12 hexagons and 1 first central hexagon where everyone spawn is the biome where "normal" peerlings spawn more frequently?
> All others are approved

## Summary of what was decided

- All findings except 12 and 27 approved as proposed. Item 2 (battle math) and
  item 31 (presentation formulas) are new detailed content and were written as
  [proposed] for review.
- The comment under "12" concerns PvP results (not the tutorial encounter,
  which item 12 was): the designer suggests both players publish the winner,
  feeding a player stat such as "number of PvP wins". Written up as a proposal.
  The tutorial encounter (item 12) remains to be confirmed.
- Item 27: the designer suggests a world of 12 hexagons around one central
  spawn hexagon, the central one being the biome where Normal Peerlings appear
  more often. Geometry options to be discussed.
