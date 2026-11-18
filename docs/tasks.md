# Master task list

High-level tasks for the engine and its production pipeline, derived from
[pipeline/](pipeline/README.md). Each task will get its own plan in `docs/plans/` before
implementation.

- **IDs** are stable; reference tasks by ID from plans and other docs.
- **Owner** is the repo the work lands in. "engine" means this project's Unreal plugins and
  tools.
- **Deps** lists tasks that must be done, or far enough along, first.
- Storylet and battle system design is out of scope here; those systems appear only where other
  tasks depend on their outputs.

## Foundations

- [ ] **F1. Asset build system.** Declarative `source + recipe -> outputs`, content-hashed,
  cached, headless, per-element granularity, diagnostics as outputs. Owner: engine.
- [ ] **F2. Export boundary.** Triangulation of the non-manifold half-edge B-rep (faces with
  holes) at export, MikkTSpace-matched tangents, persistent ID to `FGuid` mapping. Owner: engine.
  Deps: F1.
- [ ] **F3. Unreal importer.** Interchange translator or direct `FMeshDescription` builder for F2
  output. Owner: engine. Deps: F2.
- [ ] **F4. Host API.** Selection, input, undo/redo, viewport drawing, parameter UI. Shared by
  Sculptcore and UberTopo; adapters per host. Owner: engine, with Sculptcore and UberTopo.
- [ ] **F5. Source control and CI.** Large-file storage choice (Perforce or Git LFS), CI for
  code, asset builds, storylet validation, nightly cooks. Owner: engine. Deps: F1.

## Static assets

- [ ] **A1. Decimation.** QEM respecting UV seams and hard edges, for Nanite targets and LOD
  chains. Owner: Sculptcore or engine. Deps: F1.
- [ ] **A2. Automatic UV unwrapping.** Seam proposal from marked edges and group boundaries,
  flattening, packing, texel density control. Owner: Sculptcore.
- [ ] **A3. Baker.** Normal, AO, curvature, position, thickness, ID maps; cages; dilation;
  error maps. Owner: engine or Sculptcore. Deps: A2.
- [ ] **A4. LOD chain and Nanite outputs.** Recipes producing Nanite mesh, classic LODs,
  collision, optional lightmap UVs. Owner: engine. Deps: A1, A3, F3.
- [ ] **A5. Quad remeshing.** Finish Sculptcore's implementation, for deforming assets.
  Owner: Sculptcore.
- [ ] **A6. Material library.** Substrate base materials, layered materials driven by baked
  masks, named per style and biome. Owner: engine.
- [ ] **A7. Tiling material and trim sheet sources.** Choose generators and libraries; delighting
  and PBR separation for generated textures. Owner: engine.

## Sculptcore in Unreal

- [ ] **S1. Editor plugin and render proxy.** In-process plugin, custom primitive scene proxy
  uploading dirty BVH nodes. Owner: Sculptcore. Deps: F4.
- [ ] **S2. GPU backend in Unreal.** Port brush compute to RHI or use buffer interop. Owner:
  Sculptcore. Deps: S1.
- [ ] **S3. Tools and undo.** ITF tool integration; sculptcore undo entries wrapped as
  `FCommandChange`. Owner: Sculptcore. Deps: S1.
- [ ] **S4. Nanite commit.** Rebuild Nanite meshes per chunk on stroke end or idle. Owner:
  Sculptcore. Deps: S1, A4.

## Environments

- [ ] **E1. Scene description format.** UberTopo's export: kit instances, parameters, tags,
  material names, terrain, vegetation, element IDs. Owner: UberTopo and engine.
- [ ] **E2. Materializer.** Unreal plugin turning E1 into ISMs, level instances, actors and PCG;
  in-place actor updates keyed by element ID. Owner: engine. Deps: E1, F2.
- [ ] **E3. Override round trip.** Unreal edits written to UberTopo override layers; actor to
  `(element ID, override point)` mapping; base versions; orphan review UI. Owner: engine and
  UberTopo. Deps: E2.
- [ ] **E4. Annotations.** Game data parented to element IDs; export of keep-out constraints to
  UberTopo. Owner: engine. Deps: E2.
- [ ] **E5. UberTopo session server.** Out-of-process server with shared-memory geometry
  streaming to hosts. Owner: UberTopo. Deps: F4.
- [ ] **E6. Modular kits.** Kit standards (grid, pivots, naming, trims) and a first kit for one
  architectural style. Owner: content. Deps: A4, A6.
- [ ] **E7. Unreal ports of leaf stages.** Scatter and dressing as PCG nodes linking UberTopo's ID
  and RNG code. Owner: engine. Deps: E2. Optional.

## Characters and animation

- [ ] **C1. Standard skeleton decision.** UE mannequin, MetaHuman, or custom with retargeting.
  Owner: engine.
- [ ] **C2. MetaHuman workflow.** Creation, mesh-to-MetaHuman for custom sculpts, costume
  skinning. Owner: engine. Deps: C1.
- [ ] **C3. Rigging agent harness.** Joint proposal, automatic skinning, test-pose deformation,
  scoring, iteration. Owner: engine. Deps: C1, A5.
- [ ] **C4. Generative 3D intake.** Route generated characters and props through remesh, UV,
  bake and rig. Owner: engine. Deps: A2, A3, A5, C3.
- [ ] **C5. Locomotion.** Motion Matching with mocap libraries retargeted to the standard
  skeleton. Owner: engine. Deps: C1.
- [ ] **C6. Gesture library.** Conversation gestures, idles and reactions tagged by emotion and
  intensity. Owner: content. Deps: C1.
- [ ] **C7. Facial animation from audio.** Lip sync and facial performance from VO, including
  temp VO. Owner: engine. Deps: C2, N3.

## Narrative and presentation

- [ ] **N1. Storylet integration.** Runtime world state, scene requests, line IDs and location
  references from the storylet system (designed in the storylet study, with cross-storylet
  coherence). Owner: engine and storylet study.
- [ ] **N2. Dialogue cinematics director.** Staging, shots, framing, look-ats, gesture
  selection, VO timing; human overrides. Owner: engine. Deps: N1, C6, E4.
- [ ] **N3. VO pipeline.** Line ID tracking, TTS temp VO, recording management, processing.
  Owner: engine. Deps: N1.
- [ ] **N4. Localization.** String tables keyed by line ID, localized VO packs,
  pseudo-localization in CI. Owner: engine. Deps: N3.
- [ ] **N5. Narrative UI.** Dialogue choices, journal and codex driven by world state. Owner:
  per game. Deps: N1.
- [ ] **N6. Battle system hooks.** World state access, level metadata and animation reuse for
  per-game battle systems. Owner: engine. Deps: N1, C5.

## Runtime, QA and shipping

- [ ] **R1. Save system.** Derived from the world-state schema, versioned, with migrations and
  load-from-simulation tooling. Owner: engine. Deps: N1.
- [ ] **R2. Lighting decision.** Lumen only versus baked lighting per platform; determines
  lightmap UV requirements. Owner: engine.
- [ ] **R3. Performance budgets.** Per-platform budgets reported by the asset build; automated
  captures in CI. Owner: engine. Deps: F1, F5.
- [ ] **R4. Automated in-game playthroughs.** Headless runs through simulated careers, coverage
  reports. Owner: engine. Deps: N1, R1, F5.
- [ ] **R5. Packaging and patching.** Per-platform cooks, content-hash patches, language packs.
  Owner: engine. Deps: F5.

## Milestones

From [pipeline/07-roadmap.md](pipeline/07-roadmap.md):

| Milestone | Proves | Tasks |
|---|---|---|
| M1. Build system and boundary | Assets flow from source to Unreal reproducibly | F1, F2, F3 |
| M2. One hero prop | Static asset pipeline end to end | A1, A2, A3, A4, A6 |
| M3. One room | Environment generation and the override round trip | E1, E2, E3, E4, E5, E6 |
| M4. One conversation | Narrative presentation end to end | C1, C2, C6, C7, N1, N2, N3 |
| M5. Vertical slice | Everything together, with a first battle system | N6, R1, R4, plus per-game work |
