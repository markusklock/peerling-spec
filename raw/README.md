# raw/ — Immutable sources

Layer 1 of the LLM Wiki. Files here are **never edited after they are created**;
the wiki is derived from them. If a source turns out to be wrong, a newer source
supersedes it — the old one stays for history.

- `conversations/` — one file per design conversation:
  `YYYY-MM-DD-short-slug.md`, containing the designer's statements (verbatim
  where it matters) and a bullet summary of what was decided.
- `documents/` — documents, images, links and reference material supplied by the
  designer. Prefer local copies over links (links rot).

See `AGENTS.md` §8 for how sources are processed into the wiki.
