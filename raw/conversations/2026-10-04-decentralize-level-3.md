# 2026-10-04 — Decentralizing registry writes, catch checks and trades

Kind: design conversation. The LLM brainstormed, at the designer's request,
whether registry writes and trades could work without the central server, and
offered four levels: 0 (current) to 3 (players write the registry with server
signatures; anyone verifies catches by replay; trades via a signed transfer
chain with double-spend detection). Level 3 was recommended.

## Designer's statement (verbatim)

> Level 3 sound good but one thing catched my eye. Every peerling in the wild is the same? In pokemon, is not randomly encountered pokemon of different level? So they can be stronger or weaker versions of the pokemon?

## Summary of what was decided

- Level 3 is accepted (→ D-0013): registry entries written by players and
  validated by server signatures; catches verifiable by anyone through replay;
  trades recorded as signed transfer chains in an open OrbitDB log, with
  double-spends detected and the cheater flagged. The server is still needed for
  creation (GPU) and for signing species, starters and shrine creations.
- The designer asked whether wild Peerlings vary. Clarified: wild Peerlings
  already differ by level (2–50 by distance, ±2). Individual variation at the
  same level (like Pokémon's IVs and natures) does not exist yet; options were
  offered (Q-037).
