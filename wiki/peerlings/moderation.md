---
title: Content Moderation
type: system
status: deprecated
req_prefix: MOD
tags: [peerlings, policy]
sources:
  - raw/conversations/2026-10-04-answers-round-5.md
related:
  - wiki/decisions/D-0010-no-content-moderation.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-04
---

# Content Moderation

> **Deprecated.** The game has no content moderation
> ([D-0010](../decisions/D-0010-no-content-moderation.md)). This page is kept
> for history only.

The earlier proposal (not adopted) was to check the wish, the generated
concept and names, the generated image, and player display names, and to take
species down via registry tombstones.

What remains: the operator's emergency delisting tool, which is canonical in
[orbitdb-registry § Design](../tech/orbitdb-registry.md#design) (REG-008).

## Requirements

- ~~**MOD-001**~~ (removed 2026-10-04, see D-0010)
- ~~**MOD-002**~~ (removed 2026-10-04, replaced by REG-008 in [orbitdb-registry](../tech/orbitdb-registry.md#requirements))
