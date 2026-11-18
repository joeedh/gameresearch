# 07 — Roadmap, risks and open questions

## Build order

Each step tests one of the risky interfaces before content production depends on it.

1. **Asset build system and the Unreal boundary.** Recipes, caching, triangulation at export,
   and an importer.
2. **One hero prop end to end.** Sculptcore high poly -> decimation -> automatic UVs -> bake ->
   Nanite mesh and classic LOD chain -> Unreal. Tests the bake, UV and LOD stages.
3. **One room end to end.** UberTopo -> scene description -> materializer -> Unreal, including
   one override made in Unreal that survives a regeneration, and one annotation that follows
   its parent.
4. **One conversation scene.** A MetaHuman and a second character, driven by a real storylet,
   with TTS temp VO, audio-driven facial animation, and a first version of the dialogue
   director.
5. **Vertical slice.** A small hub plus one mission, using the per-game battle system's first
   version, with automated playthroughs running in CI.

## Main risks

| Risk | Why it matters | Mitigation |
|---|---|---|
| Environment dressing volume | The largest share of content work in the genre | UberTopo kits plus constrained dressing; the override round trip working well |
| Conversation staging volume | Combinatorial storylets multiply scenes | The dialogue director; overrides only for key scenes |
| VO volume | Line counts grow with storylet branching | Line IDs from the start; TTS temp VO; coherence checks to prune branches early |
| Sculptcore in Unreal | GPU backend mismatch (WebGPU/Vulkan vs D3D12 RHI) | Prototype the proxy early; keep Blender and faberforge as hosts in the meantime |
| Engine upgrades | Editor plugins break between Unreal versions | Keep kernels out of process where possible; thin adapters |
| Cross-implementation divergence | Unreal ports of UberTopo stages produce different elements | Share code for IDs and RNG; port leaf stages only |

## Open questions

- Unreal as the main DCC host versus further development of faberforge. The host API keeps this
  reversible; the question is where to put effort first.
- Which standard skeleton: UE mannequin, MetaHuman, or custom.
- Baked or dynamic lighting at ship, per platform.
- Nanite budgets and disk size targets per asset class.
- How much of UberTopo's generation should run inside Unreal at all, versus only at export.
- How the dialogue director's overrides are stored, and whether they reuse the same override
  model as UberTopo.
