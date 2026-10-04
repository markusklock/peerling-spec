# Log

Append-only. Newest entry at the bottom. Format: see `AGENTS.md` §9.
List all entries with `grep "^## \[" wiki/log.md`.

## [2026-10-03] init | Repository, schema and first spec pass
- Source: raw/conversations/2026-10-03-initial-vision.md
- Changed: AGENTS.md, CLAUDE.md, README.md, raw/README.md, wiki/index.md,
  wiki/overview.md, wiki/glossary.md, wiki/open-questions.md,
  wiki/decisions/D-0001…D-0006, wiki/gameplay/*, wiki/peerlings/*,
  wiki/world/procedural-generation.md, wiki/tech/*
- Notes: Set up the LLM Wiki structure. Captured the initial brief as accepted
  facts; filled gaps with [proposed] items and opened Q-001…Q-022. Most
  detailed areas: creation pipeline, species data model, architecture, IPFS/
  OrbitDB. Battle, catching, exploration, world generation are stubs.

## [2026-10-03] design | Answers round 1: registry, publishing, types, stats, multiplayer
- Source: raw/conversations/2026-10-03-answers-round-1.md
- Changed: overview.md, glossary.md, open-questions.md, index.md,
  decisions/D-0002, D-0003, D-0005 (→ accepted), D-0007 (new), D-0008 (new),
  peerlings/creation-pipeline.md, peerlings/peerling-species.md,
  peerlings/types.md, peerlings/moves.md, gameplay/battle.md,
  gameplay/core-loop.md, gameplay/encounters.md, gameplay/onboarding.md,
  gameplay/exploration.md, gameplay/catching.md, gameplay/player-character.md,
  gameplay/multiplayer.md (new), gameplay/pvp-battles.md (new),
  gameplay/trading.md (new), world/procedural-generation.md,
  tech/architecture.md, tech/ipfs-helia.md, tech/generation-server.md,
  tech/orbitdb-registry.md, tech/ipfs-showcase.md,
  tech/realtime-networking.md (new)
- Notes: Resolved Q-002, Q-003, Q-004, Q-009, Q-013, Q-015, Q-020; narrowed
  Q-008 (chart only) and Q-011 (size only); opened Q-023–Q-028. The shared
  multiplayer world (D-0008) is the largest change: new pages, and the trust
  model now covers peers. Moves restructured into slots following the
  designer's tentative quick/strong/special idea (proposed, Q-024). Stats
  reduced from 5 to 4 (proposed) to avoid a clash with "special".
  Removed MOV-003 and MOV-004.

## [2026-10-04] design | Answers round 2: move slots, proximity, emotes, world size, saves, stats
- Source: raw/conversations/2026-10-04-answers-round-2.md
- Changed: open-questions.md, index.md, glossary.md (Emote; Move slot),
  peerlings/moves.md, peerlings/peerling-species.md, peerlings/moderation.md,
  gameplay/battle.md, gameplay/multiplayer.md, gameplay/pvp-battles.md,
  gameplay/trading.md, gameplay/exploration.md, gameplay/player-character.md,
  world/procedural-generation.md, tech/realtime-networking.md,
  tech/architecture.md, tech/player-data.md (new)
- Notes: Resolved Q-011 (large, finite world) and Q-028 (face to face only);
  narrowed Q-024 (3 slots accepted; move-use limits open) and Q-027 (no chat,
  emotes only; scale open). The designer asked about alternatives to browser
  saves: filed the options analysis as tech/player-data.md (status proposed;
  recommends a per-player OrbitDB log plus server-signed catches and trades).
  Suggested stat numbers (320 total, 40–130, step 5) and a damage model at the
  designer's request (Q-023 stays open until approved). Added MPL-006,
  MPL-007, WGN-005, SAVE-001…004.

## [2026-10-04] design | Answers round 3: player data, stats approved, unlimited moves
- Source: raw/conversations/2026-10-04-answers-round-3.md
- Changed: decisions/D-0009 (new), decisions/D-0008, tech/player-data.md
  (rewritten as the canonical design, with save contents), glossary.md,
  open-questions.md, index.md, peerlings/peerling-species.md,
  peerlings/moves.md, peerlings/creation-pipeline.md, gameplay/battle.md,
  gameplay/catching.md, gameplay/trading.md, gameplay/pvp-battles.md,
  gameplay/multiplayer.md, gameplay/player-character.md,
  world/procedural-generation.md, tech/architecture.md,
  tech/generation-server.md, tech/orbitdb-registry.md, tech/ipfs-helia.md
- Notes: Resolved Q-014, Q-023, Q-024, Q-025, Q-026; narrowed Q-010 (levels
  1–50 decided); opened Q-029 (encounter beacon). Answered the designer's
  question on save contents in player-data § Save contents. Added SAVE-005…009,
  BTL-005, MOV-009, PVP-007/008, CAT-002. Corrected WGN-005: the natural
  border had been marked accepted by mistake; it is a proposal.

## [2026-10-04] design | Answers round 4: species/instance, progression, type chart and encounter-seed drafts
- Source: raw/conversations/2026-10-04-answers-round-4.md
- Changed: decisions/D-0006 (→ accepted), glossary.md (drand, Epoch record;
  Beacon removed), open-questions.md, index.md, overview.md,
  peerlings/types.md, peerlings/peerling-species.md, peerlings/moves.md,
  peerlings/creation-pipeline.md, gameplay/battle.md, gameplay/encounters.md,
  gameplay/core-loop.md, gameplay/catching.md, tech/player-data.md,
  tech/orbitdb-registry.md, tech/generation-server.md,
  tech/realtime-networking.md
- Notes: Resolved Q-010 (simple XP curve, no evolution, fixed moves). Drafted
  the 12-type effectiveness chart (Q-008), the encounter-seed / epoch-record
  design using drand (Q-029), and the XP and wild-level numbers (new Q-030),
  all awaiting approval. Defined a deterministic SHA-256-based RNG (BTL-007).
  Encounter selection is now deterministic, so "prefer cached species" was
  dropped and prefetching now uses precomputed upcoming encounters. Added
  SPC-011, MOV-010, BTL-006/007, ENC-005, REG-007, SAVE-010/011.

## [2026-10-04] design | Answers round 5: approvals, no items, biomes, camera, no moderation
- Source: raw/conversations/2026-10-04-answers-round-5.md
- Changed: decisions/D-0010 (new), D-0002, D-0005, peerlings/moderation.md
  (→ deprecated), world/visual-style.md (new), world/procedural-generation.md,
  peerlings/types.md, peerlings/moves.md, peerlings/creation-pipeline.md,
  gameplay/catching.md, gameplay/battle.md, gameplay/encounters.md,
  gameplay/exploration.md, gameplay/core-loop.md, gameplay/player-character.md,
  gameplay/multiplayer.md, tech/player-data.md, tech/orbitdb-registry.md,
  tech/generation-server.md, overview.md, glossary.md, open-questions.md,
  index.md
- Notes: Resolved Q-007 (no moderation → D-0010), Q-008 (chart), Q-012
  (top-down, colorful), Q-029 (encounter seeds), Q-030 (XP numbers). No
  battle items. 12 biomes, one per type. Suggested catch chance and team size
  of 4 at the designer's request (Q-031). Opened Q-032 (healing without items)
  and Q-033 (biome details); narrowed Q-017 to approving weights. Removed
  CRE-011, PLR-003, MOD-001/002, TYP-004; added CAT-003…007, WGN-006…008,
  VIS-001…005, REG-008 (emergency delisting, proposed).
