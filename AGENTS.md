# Peerlings Spec — Schema for LLM Maintainers

This file is the **schema** of this repository. Every LLM (and human) that reads,
answers questions from, or edits the specification MUST follow it. `CLAUDE.md`
imports this file, so Claude Code and other agents see the same rules.

The repository follows the *LLM Wiki* pattern
(https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), adapted for a
game design specification that grows mostly through design conversations.

---

## 1. Purpose and hard rules

- This repo contains **only the written specification** of the game *Peerlings*.
- **No implementation code.** No source files, build tooling, scripts, package
  manifests or CI. Short illustrative snippets inside markdown (JSON examples of a
  data format, pseudocode of an algorithm, Mermaid diagrams) are allowed when they
  make the spec more precise.
- The spec will be handed to *other* LLMs who will implement the game from it.
  Write for an implementer who has never seen this conversation: be explicit,
  unambiguous and self-contained.
- The human (the game designer) owns all decisions. LLMs propose; the designer
  accepts. Never present an LLM-invented idea as a decided fact (see §5).

## 2. Repository layout

```
AGENTS.md              ← this schema (canonical)
CLAUDE.md              ← imports AGENTS.md
README.md              ← human-facing intro
raw/                   ← LAYER 1: immutable sources (never edited after creation)
  conversations/       ← records of design conversations with the designer
  documents/           ← any documents/images/links the designer supplies
wiki/                  ← LAYER 2: the specification itself (LLM-maintained)
  index.md             ← catalog of every wiki page — READ THIS FIRST
  log.md               ← append-only chronological log of all changes
  overview.md          ← vision, goals, design pillars
  glossary.md          ← canonical definitions of every game term
  open-questions.md    ← every unresolved question, with stable IDs
  decisions/           ← one file per design decision (ADR style)
  gameplay/            ← what the player does: loop, exploration, battle, …
  peerlings/           ← the creatures: data model, creation, types, moves, …
  world/               ← the world: generation, visual style, audio
  tech/                ← architecture, IPFS/Helia, OrbitDB, servers, …
```

New top-level wiki folders may be added when a new area clearly doesn't fit;
record that in `log.md` and in `index.md`.

## 3. Page format

Every wiki page (except `index.md` and `log.md`) is a markdown file named in
`kebab-case.md` and starts with YAML frontmatter:

```yaml
---
title: Peerling Creation Pipeline
type: system          # overview | concept | system | data | decision | reference
status: draft         # stub | draft | proposed | accepted | deprecated
req_prefix: CRE       # requirement-ID prefix owned by this page (omit if none)
tags: [peerlings, generation]
sources:              # raw/ files this page is based on
  - raw/conversations/2026-10-03-initial-vision.md
related:              # wiki pages that are closely connected
  - wiki/peerlings/peerling-species.md
updated: 2026-10-03
---
```

Body structure (omit sections that don't apply, keep the order):

1. `# Title`
2. A **summary** blockquote (1–3 sentences) — what this page defines.
3. Main content sections.
4. `## Requirements` — numbered, testable statements (see §4).
5. `## Open questions` — links to entries in `wiki/open-questions.md`.
6. `## See also` — related pages.

**Page status meanings**

| Status       | Meaning                                                         |
|--------------|-----------------------------------------------------------------|
| `stub`       | Placeholder; scope described, content not yet written.          |
| `draft`      | Being worked on; mixes accepted facts and proposals.            |
| `proposed`   | Complete proposal awaiting designer review.                     |
| `accepted`   | Designer has approved the whole page.                           |
| `deprecated` | Superseded; keep for history, link to the replacement.          |

Decision records (`wiki/decisions/`) use the decision statuses from §7 in
their `status:` field instead: `proposed`, `accepted`,
`superseded by D-XXXX`, or `accepted` with a note in the body when a later
decision changed only part of it ("partly superseded by D-XXXX").

## 4. Requirements

- Requirements use RFC 2119 keywords: **MUST**, **MUST NOT**, **SHOULD**,
  **SHOULD NOT**, **MAY**.
- Each requirement has a stable ID `<PREFIX>-<NNN>` where the prefix is the page's
  `req_prefix` (e.g. `CRE-004`). IDs are **never reused or renumbered**; when a
  requirement is removed, keep the line struck through with a note:
  `~~CRE-004~~ (removed 2026-11-02, see D-0009)`.
- Format: `- **CRE-001** [accepted] The client MUST …`
- Implementers will cite these IDs, so keep each requirement atomic and testable.
- Registered prefixes are listed in `wiki/index.md`. Check there before choosing a
  new prefix to avoid collisions.

## 5. Provenance markers — who decided this?

Every requirement, and every non-obvious design statement, carries a marker:

- **[accepted]** — stated or approved by the designer (traceable to a `raw/`
  conversation or a decision record).
- **[proposed]** — suggested by an LLM to fill a gap; NOT yet approved.

When the designer approves a proposal, flip it to `[accepted]`, cite the
conversation in `sources`, and log it. Never silently upgrade a proposal.

## 6. Linking and the canonical-home rule

- Use **relative standard markdown links** (`[Types](../peerlings/types.md)`) so
  links work on GitHub, in Obsidian and for any LLM. Link to headings with
  `#anchor` when useful.
- **Every fact has exactly one canonical home page.** Other pages *link* to it
  instead of restating it. This applies especially to numbers (stat budgets,
  limits, sizes, timings), lists (the type list, the move templates) and data
  schemas. A one-line paraphrase plus a link is fine; copying the details is not.
- The first mention of a glossary term on a page SHOULD link to
  `glossary.md#term`.
- Every page must be listed in `wiki/index.md` and be linked from at least one
  other page (no orphans).

## 7. Open questions and decisions

- **Open questions** live only in `wiki/open-questions.md` with stable IDs
  `Q-NNN`. Pages reference them (`[Q-007](../open-questions.md#q-007)`). When a
  question is answered, move it to the *Resolved* section with a link to the
  decision or page that answers it — never delete it.
- **Decisions** are files `wiki/decisions/D-NNNN-short-slug.md` with:
  Context, Decision, Consequences, Alternatives considered, Status
  (`proposed | accepted | superseded by D-XXXX`; a decision changed only in part
  stays `accepted` and its Status line names the later decision). Significant design choices —
  especially ones that change existing behavior — get a decision record.

## 8. Workflows

### 8.1 Design session (the most common workflow)

The designer describes or changes how the game works in chat.

1. **Capture the source.** Create
   `raw/conversations/YYYY-MM-DD-short-slug.md` containing the designer's
   statements (verbatim where it matters) and a short bullet summary of what was
   decided. This is the provenance trail; it is immutable once written.
2. **Find affected pages.** Read `wiki/index.md`, then search
   (`grep -ril "<term>" wiki/`) for every term, entity, number and requirement
   ID touched by the change. Also check the `related` frontmatter of each hit.
3. **Update every affected page**, respecting the canonical-home rule: change the
   canonical page, then fix every page that links to or paraphrases it.
4. **Record** a decision file if the change is significant; resolve or add open
   questions; update `glossary.md` for new/changed terms.
5. **Bookkeeping:** update `updated:` dates, `wiki/index.md`, and append to
   `wiki/log.md`.
6. **Report back** to the designer: which pages changed, which questions were
   opened/closed, and any conflicts found.

### 8.2 Ingest (a document is added to `raw/`)

Same as 8.1, starting from the document: read it, discuss key takeaways with the
designer, then update/create pages, index and log. Sources are never modified.

### 8.3 Change propagation checklist

When anything changes, before finishing confirm that:
- [ ] the canonical page is updated;
- [ ] all pages that mention the old behavior are updated (grep for old terms,
      old numbers and the affected requirement IDs);
- [ ] no requirement now contradicts another;
- [ ] removed requirements are struck through, not deleted;
- [ ] glossary, open questions, decisions, index and log are updated.

### 8.4 Query

To answer a question about the game: read `wiki/index.md`, open the relevant
pages, answer with links to the pages/requirement IDs used. If the answer
required non-trivial synthesis that would help future readers, offer to file it
back into the wiki. If the spec doesn't answer it, say so and offer to add an
open question.

### 8.5 Lint (on request, or after large changes)

Check for and report/fix:
- contradictions between pages or requirements;
- facts duplicated outside their canonical home;
- orphan pages, pages missing from `index.md`, broken relative links;
- glossary terms used but not defined;
- `[proposed]` items that are old and should be raised with the designer;
- open questions that have actually been answered somewhere;
- requirement ID collisions or gaps in prefix registration;
- stubs that block other pages.
Append a `lint` entry to `log.md` with the findings.

## 9. Log format

`wiki/log.md` is append-only, newest entry at the bottom. Each entry:

```
## [YYYY-MM-DD] <op> | <short title>
- Source: raw/conversations/… (if any)
- Changed: wiki/a.md, wiki/b.md
- Notes: one or two lines
```

`<op>` is one of: `init`, `design`, `ingest`, `query`, `lint`, `refactor`.
This makes the log greppable: `grep "^## \[" wiki/log.md`.

## 10. Writing style

- Write for implementers: concrete, precise, no marketing language.
- Prefer tables and lists for data; prose for intent and rationale.
- Explain the **why** (design intent) next to the **what** — implementers make
  better choices when they know the goal.
- Name technologies only where they are part of the design (IPFS, Helia,
  OrbitDB are core). Leave other technology choices (rendering engine, UI
  framework) to the implementer unless the designer has decided.
- Units always explicit (ms, MB, px, metres). Dates in ISO format.
- The game is called **Peerlings** (one creature: a *Peerling*).
