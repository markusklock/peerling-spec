---
title: Generation Server
type: system
status: draft
req_prefix: SRV
tags: [tech, server, ai, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/decisions/D-0004-single-operator-server.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/ipfs-helia.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-03
---

# Generation Server

> The single operator-hosted server: it runs the self-hosted AI models of the
> [creation pipeline](../peerlings/creation-pipeline.md), pins all game content
> on IPFS, writes the registry, and helps browser nodes connect.

## Responsibilities

| Responsibility | Provenance |
|----------------|------------|
| Run a small LLM for concepts, types and moves | [accepted] |
| Run an image generator with structured (JSON) prompting, e.g. FLUX.2 | [accepted] (model choice open) |
| Run an image-to-3D generator, e.g. TRELLIS.2 | [accepted] (model choice open) |
| Pin all assets pushed to IPFS so every CID is reachable from at least one node | [accepted] |
| Expose a creation API with a job queue | [proposed] |
| Validate generated battle data and moderate content | [proposed] |
| Sole writer of the OrbitDB registry; sign species records | [proposed] ([D-0005](../decisions/D-0005-server-sole-registry-writer.md)) |
| Bootstrap peer, circuit relay and delegated routing for browser nodes | [proposed] ([ipfs-helia](ipfs-helia.md)) |

Specific model names are examples from the designer's brief; the spec treats
each model as a replaceable component behind a stage interface.

## Capacity and fairness

[proposed] GPU time is the scarce resource. The creation API:
- runs GPU jobs through a queue and reports queue position to the client;
- rate-limits per player identity (creations and regenerations — see
  [Q-001](../open-questions.md#q-001), [Q-005](../open-questions.md#q-005));
- expires abandoned jobs.

## Requirements

- **SRV-001** [accepted] The server MUST host the concept LLM, image generator and image-to-3D generator itself.
- **SRV-002** [accepted] The server MUST pin every game asset and species record so each CID is always available from at least one node.
- **SRV-003** [proposed] Each AI model MUST be behind a stage interface so it can be swapped without changing the species record format.
- **SRV-004** [proposed] GPU work MUST go through a job queue; the client MUST be able to see job status and queue position.
- **SRV-005** [proposed] The server MUST rate-limit creation requests per player identity.
- **SRV-006** [proposed] The server MUST run a libp2p node reachable from browsers (secure WebSockets and/or WebTransport) acting as bootstrap peer and circuit relay.

## Open questions

[Q-001](../open-questions.md#q-001) · [Q-003](../open-questions.md#q-003) ·
[Q-005](../open-questions.md#q-005) · [Q-022](../open-questions.md#q-022)

## See also

- [Architecture](architecture.md)
