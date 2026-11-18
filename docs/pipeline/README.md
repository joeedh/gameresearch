# End-to-end production pipeline

A high-level outline of how a narrative game (Mass Effect / Persona-style) gets made with this
engine, from authored source data to a shipped build. It names the stages, what each one owns,
and how they connect. It deliberately stops short of implementation detail.

Two systems are only mentioned, not designed, here:

- **The storylet system.** Narrative content is authored as storylets with **cross-storylet
  narrative coherence**: storylets are validated against each other and against the shared
  world state, not just individually. It is being developed in
  `/c/dev/mass-effect-storylet-study`.
- **The battle system.** Implemented per game. The engine provides hooks for it (world state,
  level metadata, character and animation pipelines) but no combat design.

## Files

| File | Covers |
|---|---|
| [01-foundations.md](01-foundations.md) | Principles, source of truth, asset build system, host API, source control, CI |
| [02-assets.md](02-assets.md) | Sculpting, static mesh pipeline (Nanite and classic LODs), UVs, baking, materials |
| [03-environments.md](03-environments.md) | UberTopo, modular kits, the materializer, overrides and the round trip |
| [04-characters-and-animation.md](04-characters-and-animation.md) | Character creation, remeshing, rigging, locomotion, gestures, faces |
| [05-narrative-and-presentation.md](05-narrative-and-presentation.md) | Storylets (brief), dialogue cinematics, VO, localization, UI |
| [06-runtime-and-qa.md](06-runtime-and-qa.md) | Gameplay integration, save system, lighting, performance, testing, shipping |
| [07-roadmap.md](07-roadmap.md) | Build order, vertical slice, risks, open questions |

## The pipeline at a glance

```
 AUTHORING (outside Unreal, or Unreal as an editing host)
 ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
 │ Sculptcore   │ │ UberTopo     │ │ Characters   │ │ Storylets    │ │ Audio / VO   │
 │ props, hero  │ │ worlds, kits │ │ sculpt / gen │ │ + coherence  │ │ scripts, TTS │
 │ assets       │ │ overrides    │ │ AI, MetaHuman│ │ validation   │ │ recordings   │
 └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
        └────────────────┴───────┬────────┴────────────────┴────────────────┘
                                 ▼
 ASSET BUILD SYSTEM (source + recipe -> derived outputs, content-hashed, cached)
   triangulate · decimate · quad remesh · UV · bake · LOD chain · Nanite · collision
   rig + skin · retarget · facial anim from audio · line IDs · localization tables
                                 │
                                 ▼
 UNREAL INTEGRATION
   importers · materializer (scene description -> actors) · dialogue director
   per-game battle system · save system · UI · lighting
                                 │
                                 ▼
 VALIDATION AND SHIPPING
   coherence checks · automated playthroughs · perf budgets · cook · package
```

## Core principles

- **Source data lives outside Unreal.** Sculptcore meshes, UberTopo worlds and storylet content
  stay in their own formats. Unreal holds derived data, plus game-only data that refers to
  generated content by ID.
- **Derived data is a build output.** It is never edited by hand, and it can always be
  regenerated from source plus recipe.
- **UberTopo works without Unreal.** Anything Unreal needs from it comes through an exported
  scene description. Some UberTopo techniques may be reimplemented in Unreal; those ports share
  UberTopo's ID and RNG code.
- **Kernels don't know about their host.** Sculptcore and UberTopo talk to a host API, so
  Blender, faberforge and Unreal are thin adapters and the choice of host stays reversible.
- **Automate the volume, hand-craft the peaks.** Locomotion, conversation staging, lip sync,
  dressing and LODs are generated; hero props, key scenes and important beats get manual work.
