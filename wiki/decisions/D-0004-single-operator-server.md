---
title: "D-0004: One operator server for generation and pinning"
type: decision
status: accepted
tags: [tech, server]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-10-image-prompt-enhancer.md
related:
  - wiki/tech/generation-server.md
updated: 2026-10-10
---

# D-0004: One operator server for generation and pinning

**Status:** accepted (2026-10-03), partly superseded by
[D-0025](D-0025-prompt-enhancer-and-three-views.md): the prompt enhancer
(GPT-6 Luna) is called through OpenAI's API, not self-hosted.

## Context
Creature generation needs GPU models (LLM, image, image-to-3D) that can't run in
a browser. IPFS content also needs at least one always-online provider.

## Decision
The designer hosts a single server that runs the small LLM, the image generator
and the image-to-3D generator, all self-hosted, and pins every game asset so each
CID is reachable from at least one node.

## Consequences
- The server is a single point of failure for *creation*, not for *play*
  ([Q-022](../open-questions.md#q-022)).
- GPU throughput limits how fast new players can onboard; a job queue and
  rate limits are required (see [generation-server](../tech/generation-server.md)).
- The server is the natural always-online peer for bootstrap, relaying and
  registry replication.

## Alternatives considered
- Third-party hosted AI APIs: not chosen; the designer self-hosts.
