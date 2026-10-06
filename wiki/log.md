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

## [2026-10-05] design | Asset budget suggestions (Q-016)
- Source: raw/conversations/2026-10-05-asset-budgets-request.md
- Changed: tech/tech-stack.md (new Asset budgets section), peerlings/creation-pipeline.md,
  tech/ipfs-helia.md, open-questions.md, index.md
- Notes: Suggested budgets at the designer's request (model ≤ 1 MB, ≤ 20k
  triangles, 1024 px texture; species ≤ 1.2 MB; 1 GB client cache). Q-016 stays
  open until approved. Added STK-012, STK-013 (proposed).

## [2026-10-05] design | Asset budgets and individual-variation details approved
- Source: raw/conversations/2026-10-05-approvals-q016-q038.md
- Changed: tech/tech-stack.md, peerlings/peerling-species.md,
  gameplay/pvp-battles.md, decisions/D-0014, open-questions.md, index.md
- Notes: Resolved Q-016 and Q-038. No open questions remain.

## [2026-10-05] design | Grid movement, foliage encounters, battle rules proposal
- Source: raw/conversations/2026-10-05-grid-foliage-battles.md
- Changed: gameplay/exploration.md (stub → draft), gameplay/battle.md
  (stub → draft), world/procedural-generation.md, gameplay/encounters.md,
  gameplay/multiplayer.md, peerlings/moves.md, tech/player-data.md,
  tech/realtime-networking.md, glossary.md, open-questions.md, index.md
- Notes: Accepted grid movement and foliage-only encounters. Proposed grid
  numbers (Q-040) and full battle rules (Q-039). Presence now sends tile
  coordinates per step; catch evidence includes the encounter tile, which
  verifiers check is foliage. Added EXP-004…006, WGN-010/011, BTL-009/010.

## [2026-10-05] design | PvP level modes; battle rules and grid numbers approved
- Source: raw/conversations/2026-10-05-pvp-level-modes.md
- Changed: decisions/D-0015 (new), gameplay/pvp-battles.md, gameplay/battle.md,
  gameplay/multiplayer.md, gameplay/exploration.md,
  world/procedural-generation.md, tech/realtime-networking.md,
  tech/player-data.md, open-questions.md, index.md
- Notes: Resolved Q-039 and Q-040. PvP challenges now choose Fair (level 50,
  default) or Real levels (unverified, opt-in). Removed PVP-007; added PVP-010.
  No open questions remain.

## [2026-10-05] design | Proposal review part 1; unique names proposed
- Source: raw/conversations/2026-10-05-proposal-review-1.md
- Changed: 30+ pages (136 [proposed] markers flipped to [accepted]),
  peerlings/creation-pipeline.md, glossary.md, decisions/D-0005, D-0007, D-0012,
  open-questions.md, index.md
- Notes: Approved all technical proposals and design items 1–4 (house art style,
  300-character wishes, concept contents, 30 s image cooldown). Item 5 reopened:
  proposed unique, normalized species names (Q-041, CRE-025). Items 6–20 still
  pending. Fixed stale notes (D-0005, D-0007, D-0012, Region definition,
  CRE-013 link).

## [2026-10-05] design | Proposal review part 2; account recovery proposed
- Source: raw/conversations/2026-10-05-proposal-review-2.md
- Changed: ~20 pages (53 [proposed] markers flipped to [accepted]),
  tech/player-data.md (new Account recovery section), peerlings/moves.md,
  tech/ipfs-showcase.md, gameplay/creator-feedback.md, open-questions.md,
  index.md
- Notes: Resolved Q-041 (unique names). Approved items 6–16 and 18–20. Item 17
  (save contents) is pending with the designer's recovery question: proposed
  deriving save-log addresses from the player ID plus an optional Argon2id
  recovery password (Q-042, SAVE-018/019). Only the save-contents rows remain
  [proposed].

## [2026-10-05] design | Save recovery settled; who holds saves corrected
- Source: raw/conversations/2026-10-05-save-recovery.md
- Changed: tech/player-data.md, open-questions.md, index.md
- Notes: Resolved Q-042: recovery by phrase or backup file only (SAVE-019
  removed); save contents approved. Corrected an overstatement: other nodes
  only hold a save log if they fetched it, so the operator server is the only
  dependable copy. Proposed keeping saves available via a save snapshot in the
  backup file and mirrors following save logs (Q-043).

## [2026-10-05] design | Peer save backups via profile inspection
- Source: raw/conversations/2026-10-05-peer-save-backups.md
- Changed: tech/player-data.md, tech/tech-stack.md, tech/resilience.md,
  tech/realtime-networking.md, gameplay/multiplayer.md, open-questions.md,
  index.md
- Notes: Community mirrors are not expected. Proposed the designer's idea:
  inspecting, trading with or battling a player keeps a backup of their save,
  served back via a `save-wanted` pubsub request; plus the save snapshot in the
  backup file (Q-043, SAVE-020/021). Answered storage persistence; proposed
  requesting persistent storage (STK-014).

## [2026-10-05] design | Peer save backups approved
- Source: raw/conversations/2026-10-05-peer-save-backups-approved.md
- Changed: tech/player-data.md, tech/tech-stack.md, open-questions.md, index.md
- Notes: Resolved Q-043 (SAVE-020, SAVE-021, STK-014 accepted). No open
  questions remain.

## [2026-10-05] design | Spectating, sharing links, device linking, world feed
- Source: raw/conversations/2026-10-05-showcase-features.md
- Changed: decisions/D-0016 (new), gameplay/spectating.md (new),
  gameplay/sharing.md (new), gameplay/world-feed.md (new), tech/player-data.md,
  tech/realtime-networking.md, tech/ipfs-showcase.md, gameplay/pvp-battles.md,
  gameplay/multiplayer.md, glossary.md, open-questions.md, index.md
- Notes: Adopted four showcase features (D-0016) and proposed their details
  (Q-044). Device linking introduces a one-device-at-a-time rule (SAVE-023),
  because two devices playing at once would fork the save log. Added SPT-001…004,
  LNK-001…003, FED-001/002, SAVE-022/023.

## [2026-10-05] design | Phone backup replaces device linking
- Source: raw/conversations/2026-10-05-phone-backup.md
- Changed: gameplay/spectating.md, gameplay/sharing.md, gameplay/world-feed.md,
  tech/player-data.md, tech/realtime-networking.md, tech/ipfs-showcase.md,
  tech/tech-stack.md, decisions/D-0016, glossary.md, open-questions.md,
  index.md
- Notes: Resolved Q-044 (spectating, sharing, world feed approved). Replaced
  device linking with a phone backup via QR code, run in the Peerlings Viewer
  (Q-045). Removed SAVE-022; added SAVE-024; SAVE-023 reworded to one computer
  per account at a time.

## [2026-10-05] design | Phone backup approved
- Source: raw/conversations/2026-10-05-phone-backup-approved.md
- Changed: tech/player-data.md, tech/tech-stack.md, decisions/D-0016,
  open-questions.md, index.md
- Notes: Resolved Q-045 (SAVE-023, SAVE-024 accepted). No open questions remain.

## [2026-10-05] design | World details: hub, landmarks, map, day/night, weather
- Source: raw/conversations/2026-10-05-world-details.md
- Changed: decisions/D-0017 (new), D-0016 (title), world/procedural-generation.md,
  world/visual-style.md, gameplay/exploration.md, gameplay/creation-shrine.md,
  tech/player-data.md, glossary.md, open-questions.md, index.md
- Notes: Accepted the spawn hub (gallery, network monument), rest-point beacons,
  named landmarks, paths and signposts, map, hand-made environment art kit,
  shared day/night and weather, and the generator-update plan (D-0017). Epoch
  records gain a `generator` field. Smaller numbers proposed (Q-046). Added
  WGN-012…016, EXP-007, VIS-006.

## [2026-10-05] design | World details approved
- Source: raw/conversations/2026-10-05-world-details-approved.md
- Changed: world/procedural-generation.md, gameplay/exploration.md,
  tech/player-data.md, decisions/D-0017, open-questions.md, index.md
- Notes: Resolved Q-046. No open questions remain.

## [2026-10-05] design | Exact data formats, protocols and creation API drafted
- Source: raw/conversations/2026-10-05-formats-request.md
- Changed: tech/data-formats.md (new), tech/protocols.md (new),
  tech/creation-api.md (new), peerlings/peerling-species.md,
  peerlings/moves.md, tech/orbitdb-registry.md, tech/player-data.md,
  tech/realtime-networking.md, tech/generation-server.md, tech/architecture.md,
  gameplay/pvp-battles.md, gameplay/trading.md, gameplay/spectating.md,
  glossary.md, open-questions.md, index.md
- Notes: Drafted exact formats for implementers (Q-047): DAG-CBOR without
  floats, one Ed25519 key per player for all identities, a signed envelope,
  five OrbitDB databases, every record, all pubsub topics and stream message
  sequences, and the HTTP/3 creation API. Example records on other pages now
  point to the exact formats. Instance origin values aligned (wild, starter,
  created). Added FMT, PRT and API requirements (all proposed).

## [2026-10-06] design | Formats approved; one key per player
- Source: raw/conversations/2026-10-06-formats-approved.md
- Changed: decisions/D-0018 (new), tech/data-formats.md, tech/protocols.md,
  tech/creation-api.md, tech/architecture.md, open-questions.md, index.md
- Notes: Resolved Q-047. The three format pages are accepted. Kept one Ed25519
  key per player for all identities after weighing pros and cons (D-0018).
  No open questions remain.

## [2026-10-06] design | Peerdex, menus, audio, rest points; no fast travel
- Source: raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
- Changed: decisions/D-0019 (new), gameplay/peerdex.md (new), gameplay/ui.md
  (new), world/audio.md (new), gameplay/exploration.md, gameplay/catching.md,
  tech/data-formats.md, tech/player-data.md, glossary.md, open-questions.md,
  index.md
- Notes: Accepted the Peerdex (seen/caught, filters, shimmer badges, My
  creations), menus/HUD/controls/settings (English only), hand-made or licensed
  music and effects, Peerling cries synthesized from the CID, rest-point
  behaviour (heal, respawn, snapshot, backup reminder), and no fast travel. The
  `seen` event and the snapshot's Peerdex now record the biome and shimmer
  sightings. Cry parameter details proposed (Q-048). Added DEX, UI, AUD,
  EXP-008/009.

## [2026-10-06] design | Peerling cry details approved
- Source: raw/conversations/2026-10-06-cry-details-approved.md
- Changed: world/audio.md, open-questions.md, index.md
- Notes: Resolved Q-048. No open questions or proposals remain.

## [2026-10-06] lint | Full consistency review (four parallel reviewers)
- Source: none (review at the designer's request)
- Changed: ~35 pages across gameplay, peerlings, world, tech, decisions,
  glossary, open-questions, index, AGENTS.md, README.md
- Notes: Fixed mechanical issues: leftovers of the superseded server-only
  registry, server catch checks and `origin: "trade"`; examples that drifted
  from data-formats (species record, instance, registry entry, epoch record);
  SAVE-002 vs backups; STK-001 scope; ARC-005 drand; stale "(proposed)"
  headings (anchors relinked); page statuses for player-character and
  procedural-generation; overview v1 scope; creation stage 8 (origin
  attestation); job object `origin`/`releaseEntry`; D-0009/D-0008/D-0002/D-0013
  notes; missing Consequences/Context in D-0017/D-0019; 23 glossary terms added;
  AGENTS decision-status convention. Substantive findings (trade atomicity,
  deterministic arithmetic and hashing, rules versioning, epoch/delisting
  checks, gameplay edge cases, creation job lifecycle, world geometry) were
  collected for the designer.

## [2026-10-06] design | Consistency-review findings decided
- Source: raw/conversations/2026-10-06-review-decisions.md
- Changed: gameplay/battle.md, pvp-battles.md, catching.md, peerdex.md,
  exploration.md, multiplayer.md, ui.md, trading.md, world-feed.md,
  creation-shrine.md, onboarding.md; peerlings/creation-pipeline.md,
  peerling-species.md; world/procedural-generation.md, audio.md;
  tech/data-formats.md, protocols.md, player-data.md, orbitdb-registry.md,
  creation-api.md, tech-stack.md; glossary.md, open-questions.md, index.md
- Notes: Applied the approved fixes: exact hash encodings, rules versions
  (BTL-011), trade all-or-nothing (SAVE-025), epoch checks (SAVE-026), tombstone
  seq and kept species records (REG-010), session heartbeat, PvP full HP and
  timeouts (BTL-013), team minimum (CAT-008), Peerdex for non-wild Peerlings
  (DEX-004), first rest point (EXP-010), blocking and busy players
  (MPL-008/009), emote IDs, move details, shrine feed event (FED-003),
  signpost ranges, duplicate-name suffix, finalize endpoint, 24 h job expiry
  (API-005), one starter ever (API-004, ONB-008), shrine credit (SHR-005),
  full trade offers, no world growth in v1, relaxed STK-008, size class and
  temperament (SPC-015). Pages with no proposals left are now `accepted`.
  New proposals: integer battle maths (BTL-012, Q-049), PvP win record
  (Q-050), presentation formulas (Q-051). New questions: first encounter
  (Q-052), hexagon world layout (Q-053).

## [2026-10-06] design | PvP win counter, first encounter, hexagon world
- Source: raw/conversations/2026-10-06-pvp-wins-hex-world.md
- Changed: world/procedural-generation.md, gameplay/pvp-battles.md,
  gameplay/battle.md, gameplay/exploration.md, gameplay/onboarding.md,
  decisions/D-0020-hexagon-spawn-biome-sectors.md (new),
  decisions/D-0021-pvp-win-counter.md (new), glossary.md, open-questions.md,
  index.md
- Notes: Central Plains spawn hexagon with 12 biome sectors (D-0020; WGN-007
  replaced by WGN-017; exact geometry proposed as WGN-018, Q-054). PvP win
  counter accepted (D-0021, PVP-011); proof mechanism still proposed (Q-050,
  PVP-012). Guaranteed first encounter approved (EXP-011); resolved Q-052 and
  Q-053.

## [2026-10-06] design | Remaining proposals approved
- Source: raw/conversations/2026-10-06-proposals-approved.md
- Changed: gameplay/battle.md, gameplay/pvp-battles.md,
  peerlings/peerling-species.md, world/audio.md,
  world/procedural-generation.md, tech/protocols.md, tech/data-formats.md,
  tech/player-data.md, decisions/D-0020, decisions/D-0021, open-questions.md,
  index.md
- Notes: Resolved Q-049 (integer battle maths, BTL-012), Q-050 (PvP win
  record, PVP-012; signed `state`/`end` messages and the `pvp-result` event and
  `pvp` profile counters added to protocols and data-formats), Q-051
  (presentation formulas) and Q-054 (hexagon geometry, WGN-018). No proposals
  or open questions remain.

## [2026-10-06] design | Following Peerling, guardians, Peerling of the Day, first finds
- Source: raw/conversations/2026-10-06-v1-fun-features.md
- Changed: gameplay/guardians.md (new), gameplay/peerling-of-the-day.md (new),
  decisions/D-0022-v1-fun-features.md (new), gameplay/exploration.md,
  creator-feedback.md, encounters.md, battle.md, world-feed.md, sharing.md,
  ui.md, core-loop.md; world/procedural-generation.md; tech/data-formats.md,
  protocols.md, player-data.md; overview.md, glossary.md, open-questions.md,
  index.md
- Notes: Four v1 features approved (D-0022): following Peerling (EXP-012),
  landmark guardians with 12 badges (GRD), Peerling of the Day with a spawn
  pedestal (POD, WGN-019), "First found in the wild by …" (CFB-005).
  Registered prefixes GRD and POD. Exact rules proposed (Q-055).

## [2026-10-06] design | Fun-feature rules approved
- Source: raw/conversations/2026-10-06-fun-features-approved.md
- Changed: gameplay/peerling-of-the-day.md, guardians.md, exploration.md,
  creator-feedback.md, world-feed.md; world/procedural-generation.md;
  tech/data-formats.md, protocols.md, player-data.md;
  decisions/D-0022-v1-fun-features.md, glossary.md, open-questions.md, index.md
- Notes: Resolved Q-055. Peerling of the Day now lasts a full 24-hour day
  (288 epochs from 00:00 UTC) instead of a 2-hour in-game day. Creators
  can't earn first-finder credit for their own species (CFB-007). No
  proposals or open questions remain.

## [2026-10-06] design | Network performance, scale and timeouts (proposal)
- Source: raw/conversations/2026-10-06-network-performance.md
- Changed: tech/network-performance.md (new), open-questions.md, index.md;
  Q-056 linked from ipfs-helia, orbitdb-registry, player-data, resilience,
  realtime-networking, creator-feedback, encounters
- Notes: Whole-spec review of slow network requests and data growth. New page
  with all-[proposed] failsafes (PERF prefix registered). Would change
  CFB-002 (species stats as an OrbitDB database) and the full-registry-sync
  rule before encounters; awaiting approval (Q-056).

## [2026-10-06] design | Network performance design approved
- Source: raw/conversations/2026-10-06-network-performance-approved.md
- Changed: tech/network-performance.md, decisions/D-0023 (new),
  tech/data-formats.md, creation-api.md, player-data.md, orbitdb-registry.md,
  ipfs-helia.md, realtime-networking.md, generation-server.md,
  ipfs-showcase.md, resilience.md; gameplay/creator-feedback.md,
  encounters.md, world-feed.md; peerlings/creation-pipeline.md; glossary.md,
  open-questions.md, index.md
- Notes: Resolved Q-056. Species stats are now hourly snapshots (CFB-002
  removed, CFB-008 added); verification checkpoints (SAVE-027); fast-path
  endpoints and creation upload (API-006); epoch record gains registryIndex,
  statsRoot, ownersRoot; catch evidence gains baseRecord. No proposals or
  open questions remain.

## [2026-10-06] lint | Second full review (four parallel reviewers): mechanical fixes
- Source: raw/conversations/2026-10-06-review-2-fixes.md
- Changed: tech/data-formats.md, protocols.md, creation-api.md, player-data.md,
  network-performance.md, tech-stack.md, ipfs-helia.md, generation-server.md,
  architecture.md, resilience.md; gameplay/battle.md, encounters.md,
  exploration.md, guardians.md, creation-shrine.md, trading.md, world-feed.md,
  multiplayer.md, spectating.md, creator-feedback.md, pvp-battles.md;
  peerlings/peerling-species.md, creation-pipeline.md; world/audio.md,
  procedural-generation.md; overview.md, glossary.md, index.md
- Notes: ~80 findings. Mechanical ones fixed (see the source file for the
  list). Design questions (world-generation determinism, drand network,
  client-derived records, novelty seen-set, trade integrity, PvP edge cases,
  shrine limits, guardian and Peerling of the Day edge cases, all-fainted
  team, world border) collected for the designer.

## [2026-10-06] design | Second review: design decisions applied
- Source: raw/conversations/2026-10-06-review-2-decisions.md
- Changed: decisions/D-0024 (new), world/procedural-generation.md,
  tech/player-data.md, data-formats.md, protocols.md, creation-api.md,
  network-performance.md; gameplay/encounters.md, peerling-of-the-day.md,
  trading.md, battle.md, pvp-battles.md, creator-feedback.md, guardians.md,
  exploration.md, catching.md, creation-shrine.md; index.md
- Notes: 14 decisions (D-0024). New requirements WGN-021, ENC-009, TRD-005,
  BTL-014, CAT-009, SHR-006; SAVE-015 and SHR-005 reworded. drand quicknet
  parameters pinned. No proposals or open questions remain.
