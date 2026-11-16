# 08 — UV and material baking (Blender → PBR texture set)

Goal: reduce arbitrary Blender node shaders to a **fixed PBR texture set per material** the native shader can consume, without disturbing authored UVs.

## Algorithm [OBSERVED]

```
for each material actually used:
    require explicit linked surface; only Principled BSDF supported, else error  # refuse placeholders
    add generated UV layer "BakeUV" (unwrap with island margin 0.001, pack margin 0.001, no rotation, AABB shape)
        overlapping faces isolated; require non-overlapping and inside 0..1
    original UV layers are preserved byte-for-byte (verify)
    bake at 1024 px:
        BaseColor, Normal, Roughness, Metallic, Opacity (via emission route, Cycles samples = 1)
    constant (unlinked) inputs -> 1x1 textures
    pack ORM: R = AO = 1, G = roughness, B = metallic
```

## Key rules

* **Bake from UV0-compatible layout.** The cooked material samples the mesh's UV; if the bake layout is a new UV, that UV must be what the mesh carries, or the bake texture will not line up. The source project baked to `BakeUV` and kept original layers; the final game mesh UV choice must be consistent with the bake — record which layer index the game mesh uses. [OBSERVED partially; verify mapping in your slice]
* **Opacity** is needed only for masked (cutout) materials; threshold from the source clip value is carried to the native cutoff (docs/09).
* **Colour space**: bake colours as sRGB; Normal/Roughness/Metallic as non-colour data. A 1×1 constant colour written without conversion is *linear* bytes and must be sRGB-encoded for a Color parameter (docs/09 P11).
* **Normal maps**: confirm convention (green channel up/down) on the first test material; the native shader and Blender may differ. [GENERAL / untested here]
* Refuse unsupported shader graphs rather than approximating; unsupported = a human decision.
* Texture count: the observed model produced roughly four texture packages per material binding — a lot of bulk data; share textures where possible. [OBSERVED]
