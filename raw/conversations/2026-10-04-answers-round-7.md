# 2026-10-04 — Answers to open questions (round 7)

Kind: design conversation. The designer answered the six topics listed after
the tech-stack rounds. The questions are paraphrased; the answers are verbatim.

## Questions asked (paraphrased)

1. Asset size budgets (Q-016).
2. Creation limits: more than one Peerling per player (Q-001)? Image
   regenerations (Q-005)? Does the player approve the 3D model (Q-006)? Who
   names the Peerling (Q-018)?
3. Player character creation: how much customisation; AI-generated avatar? (Q-019)
4. Creator feedback, e.g. "your Peerling has been caught 42 times" (Q-021).
5. Multiplayer scale: region size and visible players (Q-027).
6. What works when the operator server is down? (Q-022)

## Designer's answers (verbatim)

> 1. TBD
> 2. I think it would be nice to be able to create additional peerlings (and also choose from a few random ones instead of creating your own if the player perfers) but I am unsure how to balance that. Perhaps a place on the map where the players can trade 3 different peerlings or something else for the possibility to create a new one? To limit it a bit.
> No limit on image generations, perhaps a cool down to prevent spam.
> No the image-to-3D will probably be the same each time. If the player dislike the 3D asset they can restart from image generation stage.
> The player names its created Peerlings.
> 3. I am unsure, give suggestions.
> 4. Is that easily possible with IPFS? If so it sounds awesome.
> 5. Give recommendations
> 6. What is feasible? I would prefer to use as much distributed tech as possible to make the game work even if my server is offline. Perhaps not every Peerling is downloadable when the server is offline as its the only node with all peerlings pinned.
> But we should build a fall back in to the game. Perhaps when a random encounter is triggered the game picks not one Peerling but maybe 5 from the database that are possible candidates to encounter, then it will try to fetch them via IPFS. The first that is successfully download is the one the player is encountering.
> Should speed up the game and make it more robust

## Summary of what was decided

- Q-016 (asset budgets): deferred by the designer.
- Players can create additional Peerlings, limited by a place on the map where
  they give up something (e.g. 3 different Peerlings) for the right to create a
  new one. The balancing is not decided; the designer asked for a proposal.
- New players may choose a starter from a few random existing Peerlings instead
  of creating their own.
- Image generation: no limit on the number of generations; a cooldown prevents
  spam.
- No separate 3D-model approval or 3D retry (image-to-3D gives the same result
  each time). If the player dislikes the 3D model, they restart from the image
  generation stage.
- Players name the Peerlings they create.
- Character creation: the designer asked for suggestions.
- Creator feedback: wanted, if feasible with IPFS ("sounds awesome").
- Multiplayer scale: the designer asked for recommendations.
- Server offline: use as much distributed technology as possible so the game
  keeps working when the operator server is offline; accept that some
  Peerlings may not be downloadable then. Fallback: an encounter picks about 5
  candidate Peerlings and tries to fetch them via IPFS; a candidate that
  arrives is the one encountered. Goals: faster and more robust encounters.
