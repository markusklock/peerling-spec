# 2026-10-06 — Network performance design approved

Kind: design conversation. The LLM presented the network performance, scale
and timeout proposal (Q-056, wiki/tech/network-performance.md), including two
changes to earlier decisions: species stats become hourly snapshots instead of
an OrbitDB database, and verifiers trust the operator's signed checkpoints for
the older part of a save log.

## Designer's answer (verbatim)

> Approved

## Summary of what was decided

- Q-056 approved as proposed, including the two changes above.
- Implementation detail added while applying it: the epoch record carries a
  single CID for the registry index manifest (not the chunk list), so it
  stays within the 4 KiB pubsub message limit; a `GET /v1/checkpoints/{player}`
  endpoint lets clients fetch their newest checkpoint.
