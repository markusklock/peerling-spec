---
title: "D-0010: No content moderation"
type: decision
status: accepted
tags: [peerlings, generation, policy]
sources:
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-10-image-prompt-enhancer.md
  - raw/conversations/2026-10-10-no-self-hosted-llm.md
related:
  - wiki/peerlings/moderation.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-10
---

# D-0010: No content moderation

**Status:** accepted (2026-10-04, resolves [Q-007](../open-questions.md#q-007)),
partly superseded by [D-0025](D-0025-prompt-enhancer-and-three-views.md):
wishes and designs are steered away from existing characters such as
Pokémon ([originality rules](../peerlings/image-prompting.md#originality-rules)).

## Context
Earlier drafts proposed moderating players' wishes, generated images, names and
display names ([moderation](../peerlings/moderation.md), now deprecated).
Peerlings is a free, self-hosted hobby project: no company or brand can suffer
backlash, and unusual or meme Peerlings may even help the game's popularity.

## Decision
- [accepted] The game has **no content moderation**: wishes, generated
  concepts, images, Peerling and move names, and player display names are not
  filtered or reviewed.
- [accepted] If inappropriate content becomes a real problem, the operator will
  most likely shut the game down rather than moderate.
- [accepted] The operator keeps one **emergency delisting tool**: the server can
  mark a registry entry as removed (a tombstone, REG-008), and clients stop
  showing that species. This is not moderation; it is a cheap lever that could
  avoid shutting down the whole game over a single species, e.g. if the operator
  receives a legal request to remove something.

## Consequences
- The creation pipeline has no moderation stage and no `REJECTED` state.
- Self-hosted image models may have their own built-in safety behaviour; the
  spec neither requires nor relies on it. Since 2026-10-10 the LLM steps use
  GPT-6 Luna through OpenAI's API, whose usage policies apply and which may
  refuse a wish (`wish-refused`, [D-0025](D-0025-prompt-enhancer-and-three-views.md)).
- The operator is responsible for what their server generates and hosts.
  Even a non-commercial project may receive legal requests to remove clearly
  illegal content, which is the reason for the delisting tool above.

## Alternatives considered
- Text and image moderation at each pipeline stage (the earlier proposal):
  rejected by the designer.
