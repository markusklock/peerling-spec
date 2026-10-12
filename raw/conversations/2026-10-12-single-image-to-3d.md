# 2026-10-12 — Single image to 3D; 3D model format

Kind: design conversation, after the designer dry-ran the image pipeline from
[2026-10-10-image-prompt-enhancer](2026-10-10-image-prompt-enhancer.md).

## Context from the dry run (reported by the designer earlier)

- The hero-image prompt worked: Qwen-Image-2.1 produced a transparent image
  of a new Peerling for all three test creatures.
- The edit prompts for the right-side and back views (without the
  multiple-angles LoRA, 40 steps, Hugging Face Space, prompt extend off)
  returned the input image unchanged instead of a rotated view.

## Designer's statements (verbatim)

> Great, what format do we want the 3D model in? STL, OBJ, GLB or FBX?
> After some testing I think we can skip the multi-image part and just have Qwen Image 2.1 generate a single image and feed it to either Pixal3D or TRELLIS.2
> I tried the first image from each of our 3 examples with TRELLIS.2 and the results are very good

## Summary of what was decided

- The **multi-image step is dropped**: Qwen-Image-2.1 generates a single
  image (the hero image the player accepts), and that one image goes to the
  image-to-3D generator. This replaces "three images from different angles"
  from 2026-10-10.
- The image-to-3D generator is **Pixal3D or TRELLIS.2**, in single-image
  mode. The designer's test of TRELLIS.2 with the hero image of each of the
  three example creatures gave very good results.
- **3D format:** the question was answered from the existing spec: binary
  glTF 2.0 (`.glb`) with meshopt compression and KTX2 textures was already
  approved on 2026-10-04 (STK-009). No change.
