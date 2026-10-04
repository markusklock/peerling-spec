---
title: Creation Shrine
type: system
status: draft
req_prefix: SHR
tags: [gameplay, creation, progression]
sources:
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
related:
  - wiki/decisions/D-0012-starter-choice-and-extra-creations.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/tech/player-data.md
  - wiki/world/procedural-generation.md
updated: 2026-10-04
---

# Creation Shrine

> A special place in the world where a player gives up some of their
> Peerlings in exchange for the right to create a new Peerling species.

## Purpose

[accepted] Players can create additional Peerlings, but in a limited way: at
a place on the map they give something up, e.g. 3 different Peerlings, for
the possibility to create a new one
([D-0012](../decisions/D-0012-starter-choice-and-extra-creations.md)).

## Rules

[accepted] Approved 2026-10-04:

| Rule | Value | Why |
|------|-------|-----|
| Location | One shrine, at the spawn in the world centre | Everyone knows where it is; returning to the busy centre is social |
| Offering | 3 **verified** Peerlings of **3 different primary types**, each **level 20 or higher** | Takes real play: levelling or exploring to about 750 m out, and catching across different biomes. Turns creation into a mid-game goal rather than a quick repeat |
| What happens to the offering | The 3 Peerlings are **released**: removed from the player's collection by a signed transfer to `released` in the [transfer log](../glossary.md#transfer-log) | Makes it a real sacrifice, and stops the same Peerlings from being offered twice |
| Reward | One run of the [creation pipeline](../peerlings/creation-pipeline.md); the new Peerling joins the player's collection | |
| Level of the new Peerling | The average level of the 3 offered Peerlings, rounded down | Sacrificing three level-30s shouldn't hand back a level-5 |
| Limit | One shrine creation per player per 7 days | Caps GPU load and registry growth at about one species per active player per week |

[accepted] Flow:
1. The player walks to the shrine and chooses 3 Peerlings to offer.
2. The client signs a transfer to `released` for each of the 3 Peerlings and
   sends them to the server. The server verifies each Peerling (origin,
   ownership chain, not already released) and checks the rules above.
3. The server appends the release transfers to the transfer log and starts a
   creation job. The client appends a `release` event to its save log.
4. The creation pipeline runs as usual, including the final review and naming.
5. On publishing, the server signs an origin attestation for the new Peerling
   ([player-data § Verification](../tech/player-data.md#verification)), and the
   client appends a `created` event.

If the player abandons the creation job, the offering is not refunded, but the
right to create is kept: the player can resume the job later (CRE-014).

The server must be online to use the shrine.

## Requirements

- **SHR-001** [accepted] There MUST be a place in the world where a player can give up Peerlings in exchange for creating a new species.
- **SHR-002** [accepted] The offering MUST be 3 verified Peerlings of 3 different primary types, each at least level 20; they MUST be released (signed transfers to `released` in the transfer log) when the creation starts.
- **SHR-003** [accepted] A player MUST be limited to one shrine creation per 7 days.
- **SHR-004** [accepted] The new Peerling MUST start at the average level of the offered Peerlings, rounded down.

## Open questions

_None at the moment._
