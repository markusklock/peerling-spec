# 2026-10-03 — Initial vision

Kind: design conversation (first brief from the designer)

## Designer's statement (verbatim)

> This is a brand new repo, initate git and start working, the repo is called "peerlings-spec". The purpose here is to create the specification for my new game "Peerlings".
> This repo should ONLY contain the writte specification of the game. I will use LLMs to work on a detailed specification of every part of the game.
> Then later I will use several other LLMs to try to implement the game from the spec.
> But here we ONLY write the spec and no code, this will only be a design document.
> I want to create the design document using the Karpathy LLM Wiki method described here: https://gist.github.com/karpathy/442a6bf555914"893e9891c11519de94f
> So that even when the specification grow large it should be easy for an LLM to search it, find the correct details and also update all the relevant parts of the specification when I make changes to how certain things should work in the game.
> You can create a source folder for ingesting data but I will probably mostly chat with an LLM to add things to the specification rather than feeding in source documents.
>
> Lets start creating the specification.
> The game is called "Peerings" and is a Pokemon-inspired game with IPFS-tech at its core, the main gold is to equally show of the awesome IPFS-tech but also try to create a fun game people will enjoy.
> The core game mechanincs will be traveling around a procedually generate world looking for Peerlings that you can fight and catch.
> But no Peerlings will exist from start, all of them will be user generated.
> The plan is to have every new player create its character and starting Peerling. They create their Peerling by describing it from what they would like.
> I will host an small LLM on my server that will take their description and create the conecept for the Peerling from the users whish.
> Then I will feed the generated description of the Peerling to a image generator (perhaps FLUX.2 or 3 that can take json input for very precise control)
> The user will be presented with an image of their Peerling, they can accept it or regenerate.
> When they accept it I will feed the image to an image-to-3D generator like TRELLIS.2 or similar, that I also self-host, that will make a 3D asset of the generated image, the 3D asset will be used in the game.
>
> Next the LLM will create attacks and type of the Peerling. The type will be from a predefined list of Peerling-types and the attacks will have to follow a template-base to make it balanced.
> Then we take the data about the Peerling (description, type, attacks etc.) and its 3D-model and put on IPFS.
> The IPFS will also run an OrbitDB that holds all Peerlings to which the users Peerling will be added.
>
> The idea is that when a player explore the world they will encounter random Peerlings from all the player created ones that can be read from OrbitDB and downloaded via IPFS.
> The game will run in-browser and we will run Helia so that each player is its own IPFS-node that can upload and download data to the IPFS network.
> I will host a single server that will run the small LLM, image-generator and image-to-3D model and also autopin all assets pushed to IPFS by users so that each CID is reachable from at least one node.
>
> Let me know if you have any inital questions, then go ahead and create the first part of the spec and init git

## Summary of what was decided

- Repo `peerlings-spec` holds only the written spec, maintained with the LLM Wiki
  method; other LLMs will implement the game from it.
- Game name: **Peerlings** ("Peerings" in the brief is a typo; the repo name and
  the rest of the brief use Peerlings).
- Two equal goals: showcase IPFS technology, and be a genuinely fun game.
- Pokémon-inspired: travel a procedurally generated world, find, fight and catch
  Peerlings.
- No Peerlings exist at launch — all are user-generated.
- Every new player creates a character and a starting Peerling by describing it.
- Creation pipeline: self-hosted small LLM → concept; image generator (e.g.
  FLUX.2, JSON-prompted) → image; player accepts or regenerates; image-to-3D
  (e.g. TRELLIS.2) → 3D asset used in-game; LLM assigns type (from a predefined
  list) and attacks (following balance templates).
- Peerling data + 3D model are stored on IPFS; an OrbitDB database holds all
  Peerlings.
- Wild encounters draw random Peerlings from all player-created ones, read from
  OrbitDB and downloaded via IPFS.
- Runs in the browser; every player runs Helia and is a full IPFS node that
  uploads and downloads.
- One operator server runs the LLM, image generator, image-to-3D model, and
  auto-pins every asset pushed by users so every CID is reachable from at least
  one node.
