# 2026-10-27 — Resolving the high-severity review findings

Kind: design session, following the verification of the specification review
(raw/conversations/2026-10-27-specification-review-verification.md).

## Designer's request (verbatim)

> Great, now its time to resolve them.
> Start with the high prio ones.
> Figure out the best way to solve them or ask for my opinion if needed

## Questions asked and the designer's answers

1. **REVIEW-006: a recovered key with no save left anywhere.**
   Options: a new account (recommended), or a server-signed reset that lets
   the same key restart with a new save log and a second starter.
   **Answer: New account.**
2. **REVIEW-013: a trade whose signatures were only partly exchanged.**
   Options: the offered Peerlings are locked and the uncertain player can ask
   the server to cancel (expire) the trade, after which a late entry is void
   and both keep what they offered (recommended); locked until the trade
   appears; or unlock after 7 days at the risk of being flagged.
   **Answer: Server can cancel it.**
3. **REVIEW-003: a wild battle interrupted by closing or a crash.**
   Options: counts as fled (recommended), or log every turn and resume.
   **Answer: Counts as fled.**
4. **The other 13 high-severity fixes,** listed in chat as: catch-event
   instance without `originProof` (001); `encounter` event at battle start and
   `catch` before `battle-result` (003); backup holds the full save log and
   ownership proofs (004); session probe and tie-break (007); explicit
   catch-verification checks (008); verified / invalid / pending results
   (009); evidence judged by the records it carried, genesis record (011);
   trade entries valid only as a whole, cascading voids, `trade-void` event
   (012); PvP decision phases and deadlines (016); PvP resume and mutual
   disconnects (017); battle edge cases (021); team invariant for shrine
   offerings and void trades (026); historical eligibility by listing `seq`
   (028); one open starter job, atomic issuance, starter status endpoint (037).
   Options: apply as accepted (recommended), or apply as proposed.
   **Answer: Apply as accepted.**

## Summary of what was decided

- All 16 high-severity findings are resolved as listed above, recorded in
  D-0027 and marked Fixed in the specification review.
- A lost save with a surviving key leads to a new account; the old key stays
  the creator of its species.
- An unresolved trade can be expired by the server; a void trade entry returns
  every Peerling in it to the player who offered it, and nobody is flagged for
  an expired trade.
- An interrupted wild battle counts as fled.
