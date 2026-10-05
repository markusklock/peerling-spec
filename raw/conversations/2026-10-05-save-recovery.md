# 2026-10-05 — Save recovery: phrase and file only; where save copies live

Kind: design conversation.

## Questions asked (paraphrased)

1. Add the optional recovery password, or keep the recovery phrase and backup
   file only?
2. Is the save contents list fine (team, profile, created species, Peerdex,
   position, no inventory in v1)?

## Designer's answers (verbatim)

> Phrase and backup file is enough. The content is fine.
> But IPFS CIDs are only fetched by other nodes when requested and no other node would have a reason to request a users saved content right? So there is no chance that the network would hold any copies of the save file unless we force other nodes to pin?

## Summary of what was decided

- No recovery password: the key is recovered with the recovery phrase or a
  backup file only.
- The save contents list is approved.
- The designer correctly pointed out that IPFS nodes only hold content they
  requested, so the "any node can serve your save" claim was too strong. The
  LLM corrected the spec and proposed ways to keep saves available (Q-043).
