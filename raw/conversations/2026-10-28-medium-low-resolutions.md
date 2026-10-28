# 2026-10-28 — Resolving the medium- and low-severity review findings

Kind: design session, following the high-severity fixes
(raw/conversations/2026-10-27-high-severity-resolutions.md).

## Designer's request (verbatim)

> Yes, commit then start working on the rest of the findings. Resolve them if there is a clear way to do it, otherwise consult me

## Questions asked and the designer's answers

1. **REVIEW-010: fast fake "fled" encounters.** Options: accept the limit and
   state it in Known gaps (recommended), or add a walking-speed check.
   **Answer: Accept, state it.**
2. **REVIEW-022: Peerling of the Day weight.** Options: max(2 × normal,
   W div 19) (recommended); max(normal, W div 19); keep 1 in 20 exactly.
   **Answer: max(2 × normal, W div 19).**
3. **REVIEW-027: owned Peerlings of a delisted species.** Options: hidden but
   still usable: a silhouette named "Delisted Peerling", nobody keeps or serves
   its image or model, still works in battles, can't be traded, followed or put
   on a shared card (recommended); or owned ones unchanged.
   **Answer: Hidden, still usable.**
4. **REVIEW-033: originality check vs. showing the second image.** Options:
   block flagged images, so the player regenerates or edits the wish
   (recommended); or best effort.
   **Answer: Block flagged images.**
5. **REVIEW-041: UDP-blocked networks.** Options: unsupported in v1 with a
   clear message, API may fall back to HTTP/2 (recommended); or add a
   WebSocket fallback. **Answer: Unsupported in v1.**
6. **REVIEW-005: showing the recovery phrase again.** Options: keep the
   phrase's entropy on the device and in backups (recommended); or show it
   once only. **Answer: Keep it on the device.**
7. **REVIEW-025: what counts as seen.** Options: battles and ownership (wild,
   guardian and PvP opponents' Peerlings, starters, creations, trades; first
   biome = the player's tile) (recommended); wild, guardian and owned only; or
   anything seen in 3D. **Answer: Battles and ownership.**
8. **REVIEW-031/032: [proposed] items inside accepted pages** (SRV-007, the
   `name-reserved` and `wish-refused` errors, the 10-minute OpenAI outage
   retry, the concept LLM's originality rules, the provenance `imagePrompt`
   fields, and reusing the enhanced description for regenerations and check
   retries as part of IMG-001). Options: approve them, with the rest of the
   draft image-prompting page tracked as a new open question (recommended); or
   track all of them for later. **Answer: Approve them.**

## Summary of what was decided

- The eight choices above, recorded in D-0028.
- The other 23 medium- and low-severity findings have a clear fix and are
  resolved as recorded in D-0028.
- The remaining proposals on the draft image-prompting page are tracked as
  Q-059.
