# 03 — Environments

How worlds are generated, brought into Unreal, and edited without losing work on
regeneration.

## UberTopo's role

- **Standalone generator.** UberTopo builds worlds without Unreal, for this engine and for its
  other uses. Its data and execution graphs are never represented in Unreal's structures.
- **Constrained, not random.** It builds environments that follow architectural styles and
  natural biomes through rules and constraints, rather than open-ended procedural generation.
- **Stable 128-bit persistent IDs** derived from provenance, so the same element keeps its ID
  across re-evaluation. Overrides, caching and Unreal-side references all depend on this.

## How environments are composed

Unique per-asset baking doesn't scale to whole buildings. Environments are built from:

- **Modular kits:** walls, floors, trims, doors and windows on a metric grid, per style.
- **Trim sheets and tiling materials** shared across many meshes.
- **Decals, vertex-painted blends and runtime virtual textures** to break up repetition.
- **Unique hero props** from the static asset pipeline ([02-assets.md](02-assets.md)).
- **Terrain and vegetation** per biome.

UberTopo decides which kit pieces go where. The kit pieces themselves are normal assets built
by the asset build system.

## Layout versus style

- **Gameplay layout** (navigation, sightlines, cover, combat spaces, pacing, quest-critical
  locations) is designed by hand for hubs and missions, typically from a blockout.
- **Style** (architecture, dressing, materials, wear, vegetation) is filled in by UberTopo under
  constraints derived from the layout.
- Storylets and the per-game battle system attach to layout, not to generated detail.

## The scene description and the materializer

- **UberTopo exports a semantic scene description,** not unique geometry: kit piece instances,
  their parameters and tags (style, era, wear), material assignments, terrain, vegetation, and a
  persistent ID for every element.
- **An Unreal plugin (the materializer)** turns the description into instanced static meshes,
  level instances, actors and PCG graphs, organized with World Partition and data layers.
- **Actors are updated in place.** The materializer never deletes and recreates an actor whose
  element ID still exists, so Sequencer bindings, soft references and storylet anchors survive.
  Actor GUIDs are derived from element IDs.
- **Unreal-side ports** of UberTopo stages (e.g. scatter and dressing as PCG nodes) link
  UberTopo's ID derivation and keyed RNG code, and are kept to leaf stages so that
  floating-point divergence stays local.

## Overrides and the round trip

Designers in Unreal will edit generated content. When UberTopo's inputs change and the world
regenerates, those edits must survive or fail loudly. UberTopo's own design already covers
override layers, override points, tombstones, fallback locators, and classifying each override
as resolved, re-anchored or orphaned. The Unreal-specific rules:

- **Edits to generated content become entries in an UberTopo override layer.** Unreal edits the
  layer; it doesn't own it. The override applies at the operation that created the element, so
  later stages react to it (e.g. moving a tree also moves the cleared area around it).
- **Game data stays in Unreal as annotations.** Gameplay actors, triggers, storylet scene anchors
  and cover points are parented to element IDs with local transforms. Where layout depends on
  them, they are exported to UberTopo as keep-out constraints.
- **The materializer maps each actor or instance to `(element ID, override point)`,** so a
  gesture in Unreal (moving one chair, moving a whole room) edits the right level of the
  derivation. Structural edits, such as cutting a new doorway, are refused in Unreal and made
  as UberTopo inputs instead.
- **Overrides record the base version** they were made against, so re-anchoring can compare
  with what the designer saw.
- **Orphans are reviewed, never dropped silently.** That applies to orphaned overrides and to
  annotations whose parent disappeared.
- **Derived assets are invalidated per element.** An override that changes a building's geometry
  rebuilds that building's meshes, LODs and bakes only.

## Lighting and atmosphere

Lighting is placed partly by rule (fixtures from kits carry their lights) and partly by hand for
key spaces. See [06-runtime-and-qa.md](06-runtime-and-qa.md) for the baked-versus-dynamic
decision.
