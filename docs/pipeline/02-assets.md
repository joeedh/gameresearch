# 02 — Static assets

How props and hero assets go from high-poly source to shipping meshes and materials.

## Authoring

- **Sculpting** in Sculptcore, hosted in Blender, faberforge or (eventually) Unreal.
- **Generative 3D AI** is a source of high-poly input, not a finished asset. Its output goes
  through the same remesh, UV and bake stages as a sculpt.
- **Typical source density:** millions of triangles for a hero prop. A couch might be authored
  at around 4 million triangles.
- **Scan data and libraries** (photogrammetry, Fab/Megascans subject to licensing) enter the
  same pipeline as sculpts.

## From high poly to game meshes

Each source asset produces several outputs from one recipe:

| Output | Purpose |
|---|---|
| Nanite mesh | Decimated to a budget (typically a few hundred thousand triangles), plus a normal map for the remaining detail. Disk size and streaming are the real limits, not triangle count. |
| Classic LOD chain | LOD0 at a conventional budget (e.g. 40k for the couch) with normal maps, plus lower LODs. Needed for cases Nanite doesn't cover: some platforms, translucency, some deformation. |
| Collision | Convex decomposition, or simple primitives for gameplay-relevant shapes. |
| Lightmap UVs | Only if the game ships baked lighting (see [06-runtime-and-qa.md](06-runtime-and-qa.md)). |

### Reduction

- **Static meshes use decimation** (QEM, respecting UV seams and hard edges). Static props
  don't need quad topology.
- **Quad remeshing is for anything that deforms** (characters, cloth, soft props) and for
  assets that will be subdivided. Sculptcore has a partial implementation.
- Sculptcore's dynamic tessellation preserves marked edges, poly group boundaries, existing
  boundaries and UV chart boundaries, which supports both remeshing and unwrapping.

### UV unwrapping

- **Fully automatic unwrapping** is assembled from Sculptcore's existing tools: seam proposal
  from marked edges and group boundaries, then chart flattening and packing.
- **Quality targets:** low stretch, few seams in visible areas, tight packing, consistent texel
  density across a set of assets that appear together.
- **Manual overrides** (marking seams) feed back into the recipe rather than editing the output.

### Baking

- **Maps:** normal (tangent space, MikkTSpace-matched), ambient occlusion, curvature, position,
  thickness, and ID/mask maps for material layering.
- **Handled details:** cage generation, ray misses and overlaps, padding and dilation across
  UV seams, and error maps as a build diagnostic.

## Materials and textures

- **Material model:** Unreal's Substrate.
- **Unique assets** use baked masks driving layered materials, so most texturing is parametric
  rather than painted.
- **Architecture and large surfaces** use tiling materials, trim sheets, decals, vertex-painted
  blends and runtime virtual textures (see [03-environments.md](03-environments.md)).
- **Sources of tiling materials:** Substance-style procedural generators, scan libraries,
  in-house generators, and generative textures run through a delighting and PBR-separation step.
- **Material library:** a curated set of named materials per architectural style and biome that
  UberTopo can refer to by name.

## Stylized games

A Persona-style game changes this stage more than any other: toon shading, lower polygon
counts, hand-tuned normals and outlines, and little use of high-poly baking. The build system and
the Unreal boundary stay the same; the recipes branch.
