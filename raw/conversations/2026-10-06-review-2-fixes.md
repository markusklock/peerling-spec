# 2026-10-06 — Second full review: mechanical fixes

Kind: review at the designer's request.

## Designer's request (verbatim)

> Launch a full review to catch anything that is ambiguous or that contradicts something else

## What happened

Four parallel LLM reviewers (gameplay; social and creation; data and
protocols; infrastructure, world and overview) reported about 80 findings.
Findings where the intended behaviour was already decided were fixed directly
as mechanical clarifications (exact encodings, missing fields, stale text,
contradictory wording, glossary). Findings that need a design choice were
collected for the designer.

Mechanical fixes include: instance IDs as bytes in records; hash input rules
for CIDs, indices and structured values; registry access-controller rules
(no DEL, highest `seq` wins) and tombstone `listedSeq`; starter/shrine
instance IDs; `xp` and caught `hp` meaning; biome index in `seen`; explored
bit order; `battle-result` gains `candidate` and `species`; `pvp-result` gains
`t` and owner check; guardian evidence base records; shard key hashing; ping
keepalive; battle ID hex; spectator action list; `first-found` creator
message; save-backup direction; trade ID in `propose`; canonical offers
hash; original owner's save log for origin checks; rules version in the PvP
challenge; PvP turn and decision bookkeeping; end signature covers `mode`;
API signing string, envelopes in JSON, media types, log path; `rand_int`
mapping; integer expected-damage score; draw-order edge cases; min(5,
eligible) candidate draws; candidate 0–4; encounter retry; "fetched"
definition; following Peerling wording; first-encounter guarantee; guardian
biome indices, week of a battle, every guardian battle logged; shrine leave
vs abandon wording; stale save-fetch text; spectating battle ID; species
example with 3 moves; `species-created` event; name length unit; PvP proof
wording and flagged players; relay limits per connection; bandwidth figures;
stale encounter cost; team protected in cache; server verification not
optional; trust model; overview scope and pillar 5; cry hash input; guardian
site counts as landmark; resilience table; glossary links and terms.
