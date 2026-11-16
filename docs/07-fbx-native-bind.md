# 07 — FBX export and the native-bind post-patch

Blender's stock FBX exporter plus an import into UE 4.26 produced **bind-pose warnings** ("relative transform mismatch") — real defects, not noise. The fix: export, then **binary-patch the FBX** so its bind data equals the authoritative native pose.

## Export settings [OBSERVED]

```
global_scale 1.0; apply_unit_scale true; apply_scale_options FBX_SCALE_UNITS
use_space_transform false; bake_space_transform false
axis_forward Y; axis_up Z
object_types {MESH, ARMATURE}; use_mesh_modifiers false; use_triangles true
add_leaf_bones false; use_armature_deform_only false
armature_nodetype NULL; primary_bone_axis Y; secondary_bone_axis X
bake_anim false; path_mode STRIP; embed_textures false
rig object named exactly "Armature"; pose position REST
```

## Post-patch specification (pseudocode)

Use Blender's own `io_scene_fbx` binary parse/encode helpers (so you need no extra FBX library).

```
R = diag(1,-1,1,1)                     # handedness flip between game and Blender FBX conventions
for each native bone:
    local_fbx = R * native_local * R   # native_local from authoritative restTransform
    set Lcl Translation / Lcl Rotation (euler from quat) / Lcl Scaling on that bone's Model
set GlobalSettings up/front/coord axis props to the native axes
for each skin Cluster:
    TransformLink = transpose(world(bone))
    Transform     = transpose( inverse(world(bone)) @ world(mesh) )
update BindPose PoseNode matrices from world matrices
requirements:
    mesh world == identity (unrotated native cm)            else error
    every FBX bone parent == authoritative parent           else error
    model names unique
    digest of every FBX node not declared above is unchanged before/after
re-parse the written file; compare to authority
    tolerance: local 1e-4 absolute / 1e-5 relative; quaternion normalized; scale > 0
```

Observed result: local delta ≈ 1e-10, native global round-trip delta 0. [OBSERVED]

## Variant: animation-only FBX

Curves present, **no geometry, no skin clusters**, only the native-bone subset used by the clip; see docs/12.

## Notes

* Declare the FBX rest at frame zero and import with `use_t0_as_ref_pose=true`; do not pick a skin-cluster pose as the rest. (docs/10)
* Keep the post-patch *separate* from export so you can re-run the patch on the same FBX without re-exporting. Write the FBX once; derive a patched copy with a report; never patch in place.
* The patch report should contain the payload-unchanged digest so reviewers can see nothing else moved.
