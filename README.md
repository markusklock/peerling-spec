# Peerlings — Game Specification

**Peerlings** is a Pokémon-inspired, in-browser creature-collecting game built on
IPFS. Every creature in the game is designed by a player: you describe your
Peerling, AI models turn the description into artwork, a 3D model, a type and a
move set, and the result is published to IPFS for every other player to
encounter, battle and catch.

This repository contains **only the design specification** — no code. The spec is
written and maintained with LLMs, and is intended to be detailed enough for other
LLMs (or people) to implement the game from it.

## Where to start

- [wiki/overview.md](wiki/overview.md) — vision, goals and design pillars
- [wiki/index.md](wiki/index.md) — catalog of every spec page
- [wiki/open-questions.md](wiki/open-questions.md) — what is still undecided
- [wiki/glossary.md](wiki/glossary.md) — terminology
- [wiki/decisions/](wiki/decisions/) — every design decision, starting with
  [D-0001](wiki/decisions/D-0001-spec-only-llm-wiki.md), how this repo works

## How this repo is organized

The spec follows the [LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f):

| Layer   | Location     | Role                                                          |
|---------|--------------|---------------------------------------------------------------|
| Sources | `raw/`       | Immutable inputs: design conversations, documents, references |
| Wiki    | `wiki/`      | The specification itself, maintained by LLMs                  |
| Schema  | `AGENTS.md`  | Rules every LLM follows when reading or editing the spec      |

To work on the spec, open the repo with an LLM agent and just talk about the game
— the agent follows `AGENTS.md` to record the conversation, update every
affected page and log the change. The wiki also works well when browsed in
Obsidian (standard relative links, YAML frontmatter).
