# 06 — Skinning, influences, LODs, bones

## Observed facts [OBSERVED]

* Stock mesh: 557 mesh bones against a 1129-bone shared skeleton; 6 LODs; 8-influence skinning.
* A donor mod's mesh carried a **2379-bone** reference skeleton (a superset of the 1129 shared skeleton). It loaded and moved in game with the shared skeleton **not shipped**. Whether the extra bones deform correctly under every animation is unobserved.
* The editor import may transiently extend the shared skeleton's bone tree in the editor session; **never save or ship the shared skeleton**; verify the on-disk original hashes after the run.
* A mesh imported from FBX with `import_mesh_lods=false` produced **one LOD**; the model-only candidate reported one LOD / many sections. Behavior at distance (LOD popping, performance cost of 1 LOD with ~550k vertices) is unobserved. [INFERENCE: expect higher GPU cost vs stock's 6 LODs]

## Guidance

* Keep **bone names/hierarchy/order** matching authority when reusing the shared skeleton.
* Keep vertex-group names equal to bone names; weights normalized; ≤ 8 non-zero.
* Budget: the observed model was ~557k vertices / ~1.04M triangles across 43 meshes — large; stock LOD0 was ~196k vertices. [OBSERVED numbers] A leaner model reduces risk and cook time. [INFERENCE]
* If LOD matters, build LODs in Blender (decimate with weight preservation) and import as LOD chain (`import_mesh_lods=true` with named LOD meshes) — untested here. [INFERENCE]
* **Morph targets/facial**: import was run with `import_morph_targets=false`; face deformation uses bones. The face mesh must keep native bone weights/UVs for facial animation to work. Bounded probes (docs/04) test it. [OBSERVED]
* **Physics-driven secondary motion** (hair/skirt/chest) is *not* produced by skin weights: it is driven by native Bonamik/Kine data attached to the mesh and bone names (docs/11). New bones for cloth need new physics data — unobserved.
