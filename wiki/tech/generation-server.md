---
title: Generation Server
type: system
status: accepted
req_prefix: SRV
tags: [tech, server, ai, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-network-performance-approved.md
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-10-image-prompt-enhancer.md
  - raw/conversations/2026-10-10-no-self-hosted-llm.md
related:
  - wiki/decisions/D-0004-single-operator-server.md
  - wiki/decisions/D-0005-server-sole-registry-writer.md
  - wiki/decisions/D-0007-players-publish-assets.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/orbitdb-registry.md
  - wiki/peerlings/image-prompting.md
updated: 2026-10-10
---

# Generation Server

> The single operator-hosted server. It runs the self-hosted image and 3D
> models of the [creation pipeline](../peerlings/creation-pipeline.md), calls
> GPT-6 Luna for its LLM steps, pins all game content
> on IPFS, signs registry listings and Peerling origins, and helps browser nodes connect.

## Responsibilities

| Responsibility | Provenance |
|----------------|------------|
| Call GPT-6 Luna through OpenAI's API for every LLM step: concepts (including type), the [prompt enhancer](../glossary.md#prompt-enhancer) ([image-prompting](../peerlings/image-prompting.md)), stats and moves. No LLM runs on the server, which leaves its GPU memory to the image and 3D models | [accepted] ([D-0025](../decisions/D-0025-prompt-enhancer-and-three-views.md)) |
| Run the image generator, Qwen-Image-2.1 (hero image and reference views, transparent background) | [accepted] ([D-0025](../decisions/D-0025-prompt-enhancer-and-three-views.md)) |
| Run the image-to-3D generator, Pixal3D | [accepted] ([D-0025](../decisions/D-0025-prompt-enhancer-and-three-views.md)); [proposed] TRELLIS.2 as the alternative |
| Pin all assets players push to IPFS, so every CID is reachable from at least one node | [accepted] |
| Sign registry listings (and append them if the player's browser doesn't) | [accepted] ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)) |
| Sign species records (attestation) | [accepted] |
| Replicate and pin every player's save log; verify every save log by replay, for the species stats, first wild finds and verification checkpoints (players don't depend on it: anyone can verify catches) | [accepted] ([player-data](player-data.md#verification)) |
| Publish the signed epoch record every 5 minutes (drand randomness + registry height) | [accepted] ([player-data § Encounter seeds](player-data.md#encounter-seeds)) |
| Expose a creation API with a job queue ([creation-api](creation-api.md)) | [accepted] |
| Sign origin attestations for starters and Creation Shrine Peerlings; check shrine offerings | [accepted] ([player-data](player-data.md#starters-and-shrine-creations)) |
| Store and republish every player's latest signed IPNS profile record | [accepted] ([sharing](../gameplay/sharing.md), LNK-003) |
| Maintain species stats and send creator notifications | [accepted] ([creator-feedback](../gameplay/creator-feedback.md)) |
| Publish the registry index, hourly species stats snapshots and ownership index, and sign verification checkpoints | [accepted] ([network-performance § Snapshots and indexes](network-performance.md#snapshots-and-indexes)) |
| Serve the fast-path endpoints and a trustless HTTP gateway | [accepted] ([creation-api § Network](creation-api.md#network), [network-performance](network-performance.md#fast-paths-through-the-operator)) |
| Validate generated battle data | [accepted] |
| Bootstrap peer, circuit relay (with raised limits for game protocols), delegated routing and pubsub helper for browser nodes | [accepted] ([ipfs-helia](ipfs-helia.md), [realtime-networking](realtime-networking.md)) |

[accepted] The designer chose these models on 2026-10-10. The spec still
treats each model as a replaceable component behind a stage interface.

## Pinning

[accepted] Players publish their creations from their browsers; the server
pins them ([D-0007](../decisions/D-0007-players-publish-assets.md)). The
handshake, including the server checking that the pinned content matches what
it generated, is canonical in
[creation-pipeline § Stage 7](../peerlings/creation-pipeline.md#stage-7--publish).

## Capacity and fairness

[accepted] GPU time is the scarce resource. The creation API:
- runs GPU jobs through a queue and reports queue position to the client;
- rate-limits per player identity: the image-generation cooldown
  ([CRE-021](../peerlings/creation-pipeline.md#requirements)) and the Creation
  Shrine limit ([SHR-003](../gameplay/creation-shrine.md#requirements));
- expires abandoned jobs.

## Requirements

- **SRV-001** [accepted] The server MUST host the image generator and image-to-3D generator itself. [accepted] Every LLM step (concept, prompt enhancer, stats and moves) MUST use GPT-6 Luna through OpenAI's API ([D-0025](../decisions/D-0025-prompt-enhancer-and-three-views.md)).
- **SRV-002** [accepted] The server MUST pin every game asset and species record so each CID is always available from at least one node.
- **SRV-003** [accepted] Each AI model MUST be behind a stage interface so it can be swapped without changing the species record format.
- **SRV-004** [accepted] GPU work MUST go through a job queue; the client MUST be able to see job status and queue position.
- **SRV-005** [accepted] The server MUST rate-limit creation requests per player identity.
- **SRV-006** [accepted] The server MUST run a dual-stack (IPv6 + IPv4) libp2p node reachable from browsers over WebTransport and WebRTC-direct ([tech-stack](tech-stack.md#networking)), acting as bootstrap peer and circuit relay.
- **SRV-007** [proposed] Only the server MAY call OpenAI's API (the API key never leaves it). Requests MUST carry only the game data the step needs (wish, concept, hints, generated text, or the hero image), never the player ID, keys, display name or IP address, and MUST set `store: false`.

## Open questions

- [Q-057](../open-questions.md#q-057) — Licences of the chosen models.

## See also

- [Architecture](architecture.md)
