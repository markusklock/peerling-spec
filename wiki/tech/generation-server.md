---
title: Generation Server
type: system
status: draft
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
related:
  - wiki/decisions/D-0004-single-operator-server.md
  - wiki/decisions/D-0005-server-sole-registry-writer.md
  - wiki/decisions/D-0007-players-publish-assets.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-05
---

# Generation Server

> The single operator-hosted server. It runs the self-hosted AI models of the
> [creation pipeline](../peerlings/creation-pipeline.md), pins all game content
> on IPFS, signs registry listings and Peerling origins, and helps browser nodes connect.

## Responsibilities

| Responsibility | Provenance |
|----------------|------------|
| Run a small LLM for concepts (including type), stats and moves | [accepted] |
| Run an image generator with structured (JSON) prompting, e.g. FLUX.2 | [accepted] (model choice open) |
| Run an image-to-3D generator, e.g. TRELLIS.2 | [accepted] (model choice open) |
| Pin all assets players push to IPFS, so every CID is reachable from at least one node | [accepted] |
| Sign registry listings (and append them if the player's browser doesn't) | [accepted] ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)) |
| Sign species records (attestation) | [accepted] |
| Replicate and pin every player's save log | [accepted] ([player-data](player-data.md)) |
| Replicate save logs and replay catches for the species stats (optional; anyone can verify catches) | [accepted] ([player-data](player-data.md#verification)) |
| Publish the signed epoch record every 5 minutes (drand randomness + registry height) | [accepted] ([player-data § Encounter seeds](player-data.md#encounter-seeds)) |
| Expose a creation API with a job queue | [accepted] |
| Sign origin attestations for starters and Creation Shrine Peerlings; check shrine offerings | [accepted] ([player-data](player-data.md#starters-and-shrine-creations)) |
| Maintain species stats and send creator notifications | [proposed] ([creator-feedback](../gameplay/creator-feedback.md)) |
| Validate generated battle data | [accepted] |
| Bootstrap peer, circuit relay, delegated routing and pubsub helper for browser nodes | [accepted] ([ipfs-helia](ipfs-helia.md), [realtime-networking](realtime-networking.md)) |

The specific model names are examples from the designer's brief. The spec
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

- **SRV-001** [accepted] The server MUST host the concept LLM, image generator and image-to-3D generator itself.
- **SRV-002** [accepted] The server MUST pin every game asset and species record so each CID is always available from at least one node.
- **SRV-003** [accepted] Each AI model MUST be behind a stage interface so it can be swapped without changing the species record format.
- **SRV-004** [accepted] GPU work MUST go through a job queue; the client MUST be able to see job status and queue position.
- **SRV-005** [accepted] The server MUST rate-limit creation requests per player identity.
- **SRV-006** [accepted] The server MUST run a dual-stack (IPv6 + IPv4) libp2p node reachable from browsers over WebTransport and WebRTC-direct ([tech-stack](tech-stack.md#networking)), acting as bootstrap peer and circuit relay.

## Open questions

_None at the moment._

## See also

- [Architecture](architecture.md)
