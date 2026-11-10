# 2026-11-10 — Resolving the findings of the second specification review

Kind: design session, following the review recorded in
raw/conversations/2026-11-10-specification-review.md (REVIEW-048 to
REVIEW-082, all Open).

## Designer's request (verbatim)

> Check the open issues in the review-file, try to solve them all the best you see fit but ask me if you require my input

## Questions asked and the designer's answers

1. **REVIEW-050: forks of a save log.** A player can sign a second branch of
   their own save log, which today makes every catch "after the fork" invalid,
   including Peerlings already traded to honest players. Options: checkpoints
   protect: a fork invalidates only catches with an encounter number at or
   above the lowest duplicated number, and never a catch already covered by a
   server verification checkpoint; the rest is an accepted risk (recommended);
   accept the risk and only define "after the fork"; or forks never invalidate
   anything. **Answer: Checkpoints protect.**
2. **REVIEW-066: pinning down the world generator.** Options: one binding
   module per generator version, the same code and constants in the client,
   the server and any verifier, identified by its hash, with published test
   vectors; the spec lists what the module must output but not how
   (recommended); or the full integer algorithm written into the spec as a
   proposal. **Answer: One binding module.**
3. **REVIEW-056: forfeit wins claimed from void or drawn battles.** Options:
   both players also write void and drawn battles to their save log (not shown
   in any counter), and a forfeit claim is cancelled when the opponent's log
   holds a void, a draw or a competing forfeit for that battle (recommended);
   or accept it as a known gap. **Answer: Record void and draw.**
4. **REVIEW-075: when a shrine credit's 30 days start.** Options: from when
   the first shrine job for that offering was abandoned or expired, never
   renewed (recommended); or from the offering. **Answer: End of first job.**

## Summary of what was decided

- The four choices above, recorded in D-0029.
- The other 31 findings were resolved by the LLM under the designer's
  instruction to solve them "the best you see fit". They are listed in D-0029
  as clear fixes approved with the request. The ones that add a rule rather
  than repair wording were pointed out to the designer in the report.
- The new job-object fields for failed image checks stay proposed under Q-059,
  like the image checks they report.
