# 2026-10-03 — Answers to the first open questions (round 1)

Kind: design conversation. The designer answered eight numbered questions the
LLM asked after the first spec pass. The questions are paraphrased for context;
the answers are verbatim.

## Questions asked (paraphrased)

1. Who may write to the OrbitDB registry — only the server? (Q-002)
2. Who adds generated assets to IPFS — the server, or the player's browser with
   the server pinning afterwards? (Q-003)
3. Should the type be decided at the concept stage, before the image? (Q-004)
4. Which types — are 12 classic elements OK, or something IPFS-themed? (Q-008)
5. Should every Peerling have the same base-stat total, spread by the LLM? (Q-009)
6. Cold start — will the operator create a seed batch of Peerlings? (Q-020)
7. One shared world or one per player? Is multiplayer (trading, PvP) in the
   first version? (Q-011, Q-013)
8. Static 3D models — use simple procedural animation? (Q-015)

## Designer's answers (verbatim)

> 1. Only my server
> 2. The players browser
> 3. Yeah at the concept stage, the LLM will determine type based on the users description
> 4. The 12 classic ones are perfect
> 5. Yes, same total for all. Perhaps a quick attack, strong attack and a special attack? And the LLM balance them by spreading the stats as approriate from the Peerling concept. Or how does similar pokemon-games spread the different creatures attacks?
> 6. Yeah, I will probably create a handful of Peerlings myself at launch to populate the world a bit.
> 7. Yes, all players move around in the same world. They should be able to battle and trade
> 8. Yes, static 3D-assets so we will use pretty simple 3D graphics during battles

## Summary of what was decided

- Only the operator's server writes to the registry (Q-002 → D-0005 accepted).
- The player's browser adds the generated assets to IPFS via its Helia node;
  the server pins them (Q-003 → D-0007).
- The concept LLM determines the Peerling's type(s) from the player's
  description at the concept stage, before the image is generated (Q-004).
- The type list is the 12 classic elements proposed earlier: Normal, Fire,
  Water, Grass, Electric, Earth, Air, Ice, Metal, Light, Shadow, Spirit (part of
  Q-008; the effectiveness chart is still open).
- Every species has the same base-stat total; the LLM spreads stats to fit the
  concept (Q-009).
- Tentative, needs confirmation: move set built from a quick attack, a strong
  attack and a special attack. The designer asked how Pokémon-like games
  structure creatures' attacks.
- Cold start: the operator creates a handful of Peerlings at launch through
  the normal pipeline (Q-020).
- One shared world for all players; players see each other and can battle
  (PvP) and trade (Q-011 partly, Q-013 → D-0008). Taken as in scope for the
  first version, since the question asked about the first version.
- Peerling models are static 3D assets; battles use simple 3D graphics with
  procedural animation and no rigging (Q-015).
