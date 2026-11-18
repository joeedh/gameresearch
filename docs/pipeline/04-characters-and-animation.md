# 04 — Characters and animation

The goal is to keep manual animation to a minimum by standardizing on one skeleton, using mocap
libraries and generated motion, and hand-animating only key beats.

## Characters

### Humans

- **MetaHuman** for human characters: in-engine creation, mesh-to-MetaHuman for custom sculpts,
  a standard face rig, and grooms.
- Custom costume and armor pieces come through the static asset pipeline, then get skinned to
  the MetaHuman body.

### Non-humans and stylized characters

- **Sources:** Sculptcore sculpts, or generative 3D AI output treated as high-poly input.
- **Pipeline:** quad remesh (deformation-friendly edge flow around joints and the face) -> UVs
  -> bake -> rig -> skin -> deformation tests.

## Rigging

- **One standard skeleton** (the UE mannequin, MetaHuman, or a custom skeleton retargetable to
  them through IK Retargeter). Every mocap library and shared animation then works on every
  compatible character.
- **Rigging agent:** an agentic harness, built if none exists.
  - Proposes joint placement for a mesh against the standard skeleton.
  - Runs automatic skinning (e.g. geodesic voxel binding or bounded biharmonic weights).
  - Deforms the mesh through a fixed set of test poses.
  - Scores the result: volume loss, candy-wrapping at twist joints, self-intersection, weight
    smoothness.
  - Iterates until the scores pass, and flags failures for a human.
- Recent learned auto-riggers are candidates for the proposal step.
- **Secondary rigs** (cloth, hair, armor plates, alien appendages) use physics or control rigs
  rather than hand-keyed animation.

## Body animation

- **Locomotion:** Motion Matching, seeded with Epic's Game Animation Sample data and other mocap
  libraries, retargeted onto the standard skeleton.
- **Conversation body language:** a library of gesture, idle and reaction clips tagged by
  emotion, intensity and stance, combined with layered additive blending. The dialogue director
  (see [05-narrative-and-presentation.md](05-narrative-and-presentation.md)) picks from it.
- **Interactions** (sitting, leaning, using objects) come from mocap, with procedural IK to fit
  the props placed by UberTopo.
- **Combat animation** belongs to the per-game battle system, which reuses the same skeleton and
  retargeting.
- **Hand animation** only for hero cinematics and the most important beats.

## Facial animation

- **Lip sync and facial performance generated from audio** (MetaHuman Animator's audio-to-face,
  or FaceFX-style systems). This is the volume path for thousands of VO lines.
- **Performance capture** (phone or webcam with MetaHuman Animator) for important scenes.
- **Temp VO through TTS** produces temp facial animation, so scenes are watchable before
  recording.
- Non-human faces need their own face rigs and a mapping from the audio-driven output.

## Validation

- Deformation test poses run in the asset build for every rigged character.
- A retargeting check plays a standard clip set on every character and flags foot sliding,
  penetration and broken joints.
