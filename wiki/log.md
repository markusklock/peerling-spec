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
