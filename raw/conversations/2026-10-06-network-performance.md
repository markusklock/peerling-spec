# 2026-10-06 — Network performance, scale and timeouts

Kind: design conversation.

## Designer's request (verbatim)

> I think we should do one run and think about network performance, scale and timeouts. IPFS can be notoriously slow which I assume affects OrbitDB as well.
> We should think hard about what requests from the network that can be potentially slow and try to add failsafes for them so that the game does not feel super slow

## Summary

- The LLM reviewed every network-dependent operation in the spec and drafted
  wiki/tech/network-performance.md as a proposal (Q-056).
- Main findings: OrbitDB first sync of large logs takes minutes; the species
  stats keyvalue op-log and the transfer log grow too fast for browsers to
  replicate; verifying a player means fetching their whole save log; a new
  client may wait up to 5 minutes for an epoch record; libp2p relay defaults
  (2 min / 128 KB) would cut PvP battles; prefetching all 5 encounter
  candidates' models wastes ~5× bandwidth.
- Proposed failsafes: local-first play, a retrieval ladder racing peers and
  the operator's trustless gateway, request classes with timeouts, visible
  waiting states and placeholders, operator fast paths with P2P fallbacks,
  signed snapshots and indexes (registry index, stats snapshot, ownership
  index, epoch records by number, verification checkpoints), sequential
  candidate model prefetching, raised relay limits, early dialling.
- Library facts checked: libp2p circuit relay v2 default limits are 2 minutes
  and 128 KB per relayed connection; OrbitDB replicates entry by entry by
  following references.
