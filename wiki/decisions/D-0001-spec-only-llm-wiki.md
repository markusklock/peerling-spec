---
title: "D-0001: Spec-only repository maintained as an LLM Wiki"
type: decision
status: accepted
tags: [process]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
updated: 2026-10-03
---

# D-0001: Spec-only repository maintained as an LLM Wiki

**Status:** accepted (2026-10-03)

## Context
The designer will develop the spec by chatting with LLMs, then have several
other LLMs implement the game from it. The spec will grow large; any LLM must be
able to find details and update every affected part when a rule changes.

## Decision
- The repo contains only the written specification — no code.
- It follows the LLM Wiki pattern: immutable sources in `raw/`, the spec in
  `wiki/`, rules in `AGENTS.md` (imported by `CLAUDE.md`).
- Conversations are captured as sources in `raw/conversations/`, since most
  input arrives through chat rather than documents.

## Consequences
- Every change goes through the workflows in `AGENTS.md` §8, including the change
  propagation checklist.
- Canonical-home rule, stable requirement IDs, provenance markers and an
  append-only log make the spec searchable and traceable for implementers.

## Alternatives considered
- One large design document: hard for LLMs to update consistently as it grows.
- A spec with embedded reference code: rejected by the designer — implementers
  should work from the written spec alone.
