# 06 — Skinning, influences, LODs, bones

## Observed facts [OBSERVED]

* Stock mesh: mesh-bone list is a subset of a large shared skeleton; multiple LODs; 8-influence skinning.
* A donor mod's mesh carried a reference skeleton that was a **superset** of the shared skeleton. It loaded and moved in game with the shared skeleton **not shipped**. Whether the extra bones deform correctly under every animation is unobserved.
* The editor import may transiently extend the shared skeleton's bone tree in the editor session; **never save or ship the shared skeleton**; verify the on-disk original hashes after the run.
* A mesh imported from FBX with `import_mesh_lods=false` produced **one LOD**; the model-only candidate reported one LOD / many sections. Behavior at distance (LOD popping, GPU cost of one very dense LOD) is unobserved. [INFERENCE: expect higher GPU cost vs stock's multiple LODs]

## Guidance

* Keep **bone names/hierarchy/order** matching authority when reusing the shared skeleton.
* Keep vertex-group names equal to bone names; weights normalized; ≤ 8 non-zero.
* Budget: the observed custom model was several times denser than the stock LOD0 (hundreds of thousands of vertices across dozens of meshes). A leaner model reduces risk and cook time. [OBSERVED order of magnitude; INFERENCE on risk]
* If LOD matters, build LODs in Blender (decimate with weight preservation) and import as LOD chain (`import_mesh_lods=true` with named LOD meshes) — untested here. [INFERENCE]
* **Morph targets/facial**: import was run with `import_morph_targets=false`; face deformation uses bones. The face mesh must keep native bone weights/UVs for facial animation to work. Bounded probes (docs/04) test it. [OBSERVED]
* **Physics-driven secondary motion** (hair/skirt/chest) is *not* produced by skin weights: it is driven by native Bonamik/Kine data attached to the mesh and bone names (docs/11). New bones for cloth need new physics data — unobserved.
