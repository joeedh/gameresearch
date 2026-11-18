# 06 — Runtime integration, QA and shipping

## Gameplay integration

- **World state** is the shared contract between storylets, the battle system, UI and saves.
- **Scene anchors** (where a conversation happens, where characters stand) are Unreal-side
  annotations parented to UberTopo element IDs (see [03-environments.md](03-environments.md)).
- **Streaming:** World Partition and data layers, with data layers also used for narrative state
  (e.g. a district before and after an attack).

## Save system

- Derived from the world-state schema, so the saved state and the narrative state can't drift
  apart.
- Versioned, with migrations when the schema changes during development.
- Debug tooling to load a save from a simulated playthrough, which is how designers jump to a
  given point in the story.

## Lighting

- **Lumen** during development, for fast iteration.
- **Decide early** whether shipping uses baked lighting (GPU Lightmass) on some platforms or
  areas, because that decision determines whether every static mesh needs lightmap UVs.
- Conversation scenes get lighting rigs from the dialogue director's templates, with manual
  adjustments for key scenes.

## Performance budgets

- Budgets per platform: triangles (Nanite and non-Nanite), draw calls, texture memory, actor
  counts, animation cost, and streaming bandwidth.
- The asset build reports against budgets per asset and per level, before anything is cooked.
- Automated performance captures along fixed camera paths in CI.

## Testing

- **Narrative:** storylet validation, including the cross-storylet coherence checks, plus
  simulated playthroughs at the data level (already done in the storylet study).
- **Automated playthroughs in the game:** drive the real build headless through simulated
  careers, to find storylets that are reachable in data but broken in the game (missing anchors,
  unreachable locations, softlocks).
- **Coverage reports:** which storylets, lines and scenes have been reached in-game.
- **Asset checks:** missing LODs, budget overruns, orphaned overrides, orphaned annotations,
  broken references.
- **Battle system:** tested per game.

## Shipping

- Cook and package per platform from CI.
- Patches built from content hashes, so regenerated-but-unchanged assets don't enter a patch.
- Localization and VO packs per language.
