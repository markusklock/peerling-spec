---
title: Content Moderation
type: system
status: stub
req_prefix: MOD
tags: [peerlings, safety, generation]
related:
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/orbitdb-registry.md
updated: 2026-10-03
---

# Content Moderation

> How user-generated Peerlings are kept appropriate for a broad audience, and
> how a published species can be taken down even though IPFS content is
> permanent. Status: stub — policy pending [Q-007](../open-questions.md#q-007).

## Scope (proposed)

[proposed] Moderation checkpoints in the [creation pipeline](creation-pipeline.md):

| Checkpoint | What is checked |
|------------|-----------------|
| Wish (stage 1) | Hateful, sexual, violent-extreme content; real people; obvious copyrighted characters |
| Concept & names (stages 2, 6) | Same as above, applied to generated text and move names |
| Image (stage 3) | NSFW / unsafe image classifier before the player sees it |

[proposed] Takedown: IPFS content cannot be deleted from the network, but the
registry decides what the game shows. The server appends a **tombstone** entry
for the species to the [registry](../tech/orbitdb-registry.md); clients stop
offering it in encounters and hide it in collections; the server unpins its
assets.

## Requirements

- **MOD-001** [proposed] Content MUST be moderated before it is shown to other players.
- **MOD-002** [proposed] The operator MUST be able to take down a published species via a registry tombstone, and clients MUST honor tombstones.

## Open questions

[Q-007](../open-questions.md#q-007)
