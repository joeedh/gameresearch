# 01 — Foundations

The infrastructure every other stage depends on. Most of it should exist before content
production starts in earnest.

## Source of truth

- **Authored source formats**
  - Meshes: Sculptcore's mesh format, and the preferred B-rep everywhere: a non-manifold,
    bmesh-style half-edge structure with support for faces with holes.
  - Worlds: UberTopo's scene graph, its execution graphs, and its override layers.
  - Narrative: storylet content (YAML today).
  - Audio: VO scripts keyed by line ID, plus raw recordings.
- **Unreal-owned data** is limited to things that only exist in the game: gameplay actors,
  triggers, Sequencer sequences, UI, per-game battle logic. When it refers to generated content
  it does so by persistent element ID (see [03-environments.md](03-environments.md)).
- **Derived data** (triangle meshes, LODs, Nanite meshes, textures, rigs, animation clips,
  string tables) is never the source of truth for anything.

## Asset build system

The most important piece of infrastructure. Without it every high-poly-to-game-mesh conversion
becomes repeated manual work.

- **Model:** a declarative graph of `source + recipe -> outputs`, similar to Bazel or Unreal's
  derived data cache, applied to art.
- **Content-hashed and cached.** An output is rebuilt only when its source, recipe or tool
  version changes.
- **Per-element granularity** for generated worlds. An UberTopo override that changes one
  building rebuilds that building's outputs only.
- **Runs headless,** both on a workstation and in CI. Editor integration triggers builds and
  shows status; it does not implement them.
- **Recipes are data.** A recipe names the steps (decimate to N triangles, unwrap with these
  seams, bake these maps at this resolution) and their parameters, and is versioned with the
  source.
- **Diagnostics are outputs too.** Bake error maps, UV distortion and stretch reports, and
  triangle budget reports are produced alongside the assets so problems are visible before
  anyone opens Unreal.

## The Unreal boundary

- **Import:** a custom Interchange translator, or a plugin that builds `FMeshDescription`
  directly.
- **Triangulation happens once, at export.** MeshDescription can't represent faces with holes
  or non-manifold topology, so the exporter triangulates (constrained triangulation, or ear
  clipping with hole bridging) and drops non-manifold features that have no game meaning.
- **Tangent space:** the exporter and baker must agree with Unreal's MikkTSpace exactly, or
  normal maps show seams.
- **Stable identity crosses the boundary.** Persistent 128-bit element IDs map directly onto
  Unreal's 128-bit `FGuid`s for actors created from generated content.

## Host API

Sculptcore and UberTopo are libraries. Each host (Blender, faberforge, Unreal, UberTopo's NW.js
UX) implements a small host API:

- selection, input and tool activation
- undo and redo (mapped onto the host's transaction system)
- viewport drawing and dirty-region updates
- parameter UI and asset browsing

**Process model:**

- **UberTopo runs out of process** as a session server, streaming geometry to the host over
  shared memory. Houdini Engine for Unreal is the precedent. A kernel crash doesn't take down
  the editor, and engine upgrades don't break the kernel build.
- **Sculptcore runs in process** as an editor plugin, because brush latency can't tolerate a
  process boundary. In Unreal it renders through a custom primitive scene proxy that uploads
  only dirty BVH nodes. Its GPU backend (WebGPU/Vulkan) must either be ported to Unreal's RHI
  compute or share buffers through interop.

## Source control and CI

- **Large binary data** (recordings, high-poly sources, textures) needs Perforce or Git LFS.
  Code and text formats (storylets, recipes, override layers) can live in Git.
- **World Partition's one-file-per-actor storage** plus deterministic actor GUIDs means that
  regenerating a level produces a small, reviewable diff.
- **CI runs:**
  - code builds and unit tests for engine plugins and kernels
  - the asset build for changed sources
  - storylet validation, including the cross-storylet coherence checks
  - automated playthroughs (see [06-runtime-and-qa.md](06-runtime-and-qa.md))
  - nightly cooks and packaged builds
