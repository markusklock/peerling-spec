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

## [2026-10-04] design | Approvals round 6 and tech stack (WebTransport, IPv6)
- Source: raw/conversations/2026-10-04-tech-stack-1.md
- Changed: decisions/D-0010, D-0011 (new), tech/tech-stack.md (new),
  tech/ipfs-helia.md, tech/generation-server.md, tech/architecture.md,
  tech/realtime-networking.md, tech/orbitdb-registry.md, gameplay/catching.md,
  gameplay/exploration.md, gameplay/battle.md, gameplay/encounters.md,
  world/procedural-generation.md, world/visual-style.md, glossary.md,
  open-questions.md, index.md
- Notes: Resolved Q-017, Q-031, Q-032, Q-033; emergency delisting kept
  (REG-008 accepted). New principle D-0011: modern web platform first;
  WebTransport replaces WebSockets for browser → server; IPv6 preferred.
  Recorded that browsers can't accept WebTransport, so browser ↔ browser stays
  WebRTC. Proposed the rest of the stack (WebRTC-direct, WebGPU + WebGL2
  fallback, Web Worker, OPFS, Ed25519 WebCrypto, PWA, meshopt/KTX2, AVIF) as
  Q-034. Added EXP-002/003, WGN-009, STK-001…009.

## [2026-10-04] design | Tech stack round 2: desktop only, WebGPU fallback, stack approved
- Source: raw/conversations/2026-10-04-tech-stack-2.md
- Changed: tech/tech-stack.md, tech/ipfs-helia.md,
  decisions/D-0011-modern-web-platform-first.md, open-questions.md, index.md
- Notes: Resolved Q-034. Desktop browsers only (mobile not a target); WebGPU
  with WebGL2 fallback; WebRTC-direct as the fallback browser → server
  transport. Worker, OPFS, Ed25519, PWA, asset formats and TypeScript accepted.
  Added STK-010, STK-011. Asset size budgets remain open (Q-016).

## [2026-10-04] design | Answers round 7: creations, naming, creator feedback, resilience
- Source: raw/conversations/2026-10-04-answers-round-7.md
- Changed: decisions/D-0012 (new), D-0002, gameplay/creation-shrine.md (new),
  gameplay/creator-feedback.md (new), tech/resilience.md (new),
  peerlings/creation-pipeline.md, peerlings/peerling-species.md,
  gameplay/onboarding.md, gameplay/encounters.md, gameplay/catching.md,
  gameplay/core-loop.md, gameplay/player-character.md, gameplay/multiplayer.md,
  tech/player-data.md, tech/realtime-networking.md, tech/orbitdb-registry.md,
  tech/generation-server.md, tech/ipfs-showcase.md, tech/architecture.md,
  tech/ipfs-helia.md, glossary.md, open-questions.md, index.md
- Notes: Resolved Q-001 (D-0012), Q-005, Q-006, Q-018, Q-021. Q-016 deferred
  by the designer. Wrote the requested suggestions: character creation (Q-019),
  multiplayer scale (Q-027), Creation Shrine balancing (new Q-035), resilience
  design (Q-022). Adapted the designer's 5-candidate encounter idea into a
  verifiable ordered list (new Q-036). Removed ONB-001/002 (replaced by
  ONB-005/006). Added CRE-021…023, ONB-005…007, SHR-001…004, CFB-001…004,
  RES-001…004, ENC-006/007, SAVE-012/013, PLR-004.

## [2026-10-04] design | Answers round 8: approvals; decentralization brainstorm opened
- Source: raw/conversations/2026-10-04-answers-round-8.md
- Changed: gameplay/creation-shrine.md, gameplay/player-character.md,
  gameplay/onboarding.md, gameplay/multiplayer.md, gameplay/encounters.md,
  tech/realtime-networking.md, open-questions.md, index.md
- Notes: Resolved Q-019, Q-027, Q-035, Q-036. Q-022 stays open while the
  designer explores decentralizing registry writes and trades.

## [2026-10-04] design | Level 3 decentralization: peer-verified registry, catches and trades
- Source: raw/conversations/2026-10-04-decentralize-level-3.md
- Changed: decisions/D-0013 (new), D-0005 (→ superseded), D-0009 (partly
  superseded), tech/player-data.md, tech/orbitdb-registry.md,
  tech/generation-server.md, tech/architecture.md, tech/resilience.md,
  peerlings/creation-pipeline.md, peerlings/peerling-species.md,
  gameplay/trading.md, gameplay/pvp-battles.md, gameplay/catching.md,
  gameplay/creation-shrine.md, gameplay/creator-feedback.md, glossary.md,
  open-questions.md, index.md
- Notes: Players append registry entries carrying a server listing signature;
  anyone verifies catches by replay; ownership is a signed transfer chain in an
  open OrbitDB transfer log, with double trades detected (lower CID wins) and
  the cheater flagged. The ownership ledger and server catch attestations are
  gone. Resolved Q-022; opened Q-037 (individual variation). Removed REG-003,
  CRE-019, SAVE-004, SAVE-006, PVP-008; added REG-009, CRE-024,
  SAVE-014…017, PVP-009, TRD-004.

## [2026-10-05] design | Individual variation: stat traits and shimmer variants
- Source: raw/conversations/2026-10-05-individual-variation.md
- Changed: decisions/D-0014 (new), peerlings/peerling-species.md,
  gameplay/battle.md, gameplay/encounters.md, gameplay/pvp-battles.md,
  gameplay/trading.md, tech/player-data.md, overview.md, glossary.md,
  open-questions.md, index.md
- Notes: Resolved Q-037 (options b + d). Traits (−10…+10% per stat) multiply
  stats after the level formula; shimmer variants are cosmetic. Defined the
  fixed order in which a wild Peerling is generated from the encounter seed;
  level, traits and shimmer belong to the encounter, not the candidate, so
  candidate choice can't be used to chase them. Opened Q-038 (rarity 1 in 500,
  visible traits, traits in PvP). Added SPC-012…014, BTL-008, ENC-008.
