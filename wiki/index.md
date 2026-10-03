# Wiki Index

Catalog of every page in the Peerlings specification. **LLMs: read this first**,
then open only the pages you need. Keep it updated on every change
(`AGENTS.md` §8).

Status legend: `stub` · `draft` · `proposed` · `accepted` · `deprecated`

## Start here

| Page | Status | Summary |
|------|--------|---------|
| [overview](overview.md) | draft | Vision, the two equal goals, design pillars, v1 scope |
| [glossary](glossary.md) | draft | Canonical definitions of all terms |
| [open-questions](open-questions.md) | draft | All unresolved questions (Q-NNN) |
| [log](log.md) | — | Append-only change log |

## Gameplay

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [core-loop](gameplay/core-loop.md) | draft | — | Explore → encounter → battle → catch; player motivations |
| [onboarding](gameplay/onboarding.md) | draft | ONB | New player creates character + starter Peerling |
| [player-character](gameplay/player-character.md) | stub | PLR | Avatar, identity keypair, save data |
| [exploration](gameplay/exploration.md) | stub | EXP | Moving through the world |
| [encounters](gameplay/encounters.md) | draft | ENC | Choosing wild Peerlings from the registry; prefetch pool |
| [battle](gameplay/battle.md) | stub | BTL | Turn-based fights |
| [catching](gameplay/catching.md) | stub | CAT | Catching wild Peerlings; collection |

## Peerlings

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [creation-pipeline](peerlings/creation-pipeline.md) | draft | CRE | Wish → concept → image → review → 3D → battle profile → publish → starter |
| [peerling-species](peerlings/peerling-species.md) | draft | SPC | Species record (IPFS), stats, Peerling instance data model |
| [types](peerlings/types.md) | draft | TYP | Predefined type list and effectiveness chart |
| [moves](peerlings/moves.md) | draft | MOV | Move templates that keep generated attacks balanced |
| [moderation](peerlings/moderation.md) | stub | MOD | Content checks and takedown via registry tombstones |

## World

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [procedural-generation](world/procedural-generation.md) | stub | WGN | Seeded, chunked world; biomes |

## Tech

| Page | Status | Req prefix | Summary |
|------|--------|-----------|---------|
| [architecture](tech/architecture.md) | draft | ARC | Components, data flows, trust model |
| [ipfs-helia](tech/ipfs-helia.md) | draft | NODE | Browser IPFS node: connectivity, verification, caching |
| [orbitdb-registry](tech/orbitdb-registry.md) | draft | REG | OrbitDB database of all species |
| [generation-server](tech/generation-server.md) | draft | SRV | Self-hosted AI models, pinning, job queue, relay |
| [ipfs-showcase](tech/ipfs-showcase.md) | draft | SHOW | Making IPFS visible and meaningful to players |

## Decisions

| ID | Status | Title |
|----|--------|-------|
| [D-0001](decisions/D-0001-spec-only-llm-wiki.md) | accepted | Spec-only repository maintained as an LLM Wiki |
| [D-0002](decisions/D-0002-all-peerlings-user-generated.md) | accepted | All Peerlings are user-generated |
| [D-0003](decisions/D-0003-browser-client-is-ipfs-node.md) | accepted | Every browser client is a Helia IPFS node |
| [D-0004](decisions/D-0004-single-operator-server.md) | accepted | One operator server for generation and pinning |
| [D-0005](decisions/D-0005-server-sole-registry-writer.md) | proposed | The generation server is the only registry writer |
| [D-0006](decisions/D-0006-species-vs-instance.md) | proposed | Separate immutable species from owned instances |

## Registered requirement prefixes

ARC, BTL, CAT, CRE, ENC, EXP, MOD, MOV, NODE, ONB, PLR, REG, SHOW, SPC, SRV,
TYP, WGN. Next free decision ID: D-0007. Next free question ID: Q-023.

## Sources

| Source | Date | Summary |
|--------|------|---------|
| [2026-10-03-initial-vision](../raw/conversations/2026-10-03-initial-vision.md) | 2026-10-03 | The designer's initial brief for the whole game |
