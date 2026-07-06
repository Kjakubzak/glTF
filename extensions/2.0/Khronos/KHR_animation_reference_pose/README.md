# KHR_animation_reference_pose

## Contributors

- Ken Jakubzak, Meta

## Status

**Draft** - This extension is not yet ratified by the Khronos Group and is subject to change.

## Dependencies

Written against the glTF 2.0 specification. No other extensions are required.

**Optional**: `KHR_animation_type` (additive animations use a reference pose as their per-channel baseline).

## Overview

The `KHR_animation_reference_pose` extension defines a canonical **reference (rest) pose** for a node hierarchy
by tagging a standard glTF animation with a semantic pose type. A reference pose is a neutral baseline that
supports:

- **Additive animation** — the per-channel baseline from which additive deltas are computed (see `KHR_animation_type`).
- **Retargeting** — a shared rest pose that source and target rigs can be aligned against.
- **Validation & tooling** — a canonical pose for deformation checks and consistency across toolchains.

This is a **generalized, non-character** extension: it applies to any node hierarchy (skeletal or not) and does
not require any character-specific extension. It is the general counterpart to the character-specific
`KHR_character_reference_pose`.

Reference poses are stored as single-keyframe animations using the standard glTF animation infrastructure,
providing:

- **TRS format**: Local-space translation, rotation, and scale - the format retargeting systems require.
- **Binary storage**: Efficient accessor-based storage via existing animation buffers.
- **Compression support**: Normalized quaternions and other animation compression techniques.
- **Universal tooling**: 100% compatibility with existing glTF viewers and tools (it is a normal animation).

## Extension Schema

This extension is applied to individual animations, marking them as reference poses with a semantic `poseType`.

```json
{
  "animations": [
    {
      "name": "RestPose",
      "extensions": {
        "KHR_animation_reference_pose": {
          "poseType": "rest"
        }
      },
      "channels": [
        { "sampler": 0, "target": { "node": 0, "path": "rotation" } },
        { "sampler": 1, "target": { "node": 0, "path": "translation" } }
      ],
      "samplers": [
        { "input": 0, "output": 1, "interpolation": "STEP" },
        { "input": 0, "output": 2, "interpolation": "STEP" }
      ]
    }
  ]
}
```

### Properties

| Property   | Type   | Required | Default | Description |
|------------|--------|----------|---------|-------------|
| `poseType` | string | No       | `rest`  | The semantic classification of this reference pose. Open-ended. |

### poseType Values

The `poseType` property has predefined values but accepts any custom string for extensibility:

| Value    | Description |
|----------|-------------|
| `rest`   | Neutral rest/reference pose. The general default. |
| `bind`   | A pose matching the skinning bind pose (see "Relationship to glTF 2.0 Skinning"). |
| `TPose`  | Humanoid convention: arms extended horizontally. |
| `APose`  | Humanoid convention: arms angled downward (~45°). |
| `IPose`  | Humanoid convention: arms resting at sides. |
| *custom* | Any string for poses that do not fit the predefined values (e.g., `"CombatStance"`). |

> The humanoid conventions (`TPose`/`APose`/`IPose`) are listed for interoperability but carry no character-specific
> requirement; this extension does not depend on any humanoid or character semantics.

## Animation Structure

Reference poses use the standard glTF animation structure with these conventions:

### Channels

Each node participating in the reference pose should have channels for the transforms that define its pose:

- **`rotation`**: VEC4 quaternion (typical for most nodes).
- **`translation`**: VEC3 position (required for the root, optional for others).
- **`scale`**: VEC3 scale (optional, typically omitted when uniform).

### Samplers

- **`interpolation`**: Should be `STEP` since poses are static snapshots.
- **`input`**: Time accessor with a single value (typically `0.0`).
- **`output`**: Transform data accessor.

### Accessors

| Channel Path  | Accessor Type | Component Type(s) |
|---------------|---------------|-------------------|
| `translation` | `VEC3`        | float |
| `rotation`    | `VEC4`        | float, or normalized byte/short |
| `scale`       | `VEC3`        | float |

## Relationship to KHR_animation_type

`KHR_animation_type` marks animations as `override` or `additive`. Additive animations are evaluated as **deltas
over a baseline**. When a `KHR_animation_reference_pose` animation is present, implementations **SHOULD** use it as
that per-channel baseline. If no reference pose is available, implementations **SHOULD** fall back to evaluating
the additive animation's channels at `t = 0` and using those values as the baseline.

## Relationship to glTF 2.0 Skinning

A reference pose (for retargeting/additive baselining) is distinct from the skinning **bind pose**:

| Concept | Definition | Location in glTF |
|---------|------------|------------------|
| **Bind Pose** | The pose used when skinning weights were painted | `inverse(skin.inverseBindMatrices)` |
| **Reference Pose** | A canonical pose for retargeting / additive baselining | This extension |

These may or may not be the same pose. Reference poses defined by this extension are **not** used for skinning
calculations; they provide semantic information for retargeting and other toolchain operations.

## Implementation Notes

- **Single keyframe**: Reference poses typically have one keyframe at `t = 0`; they represent a static pose, not motion.
- **Local space**: All transforms are local to the parent node, matching standard glTF animation behavior.
- **Node coverage**: Include channels for all nodes relevant to the pose. Nodes without channels retain their default node transforms.
- **Multiple poses**: Tag multiple animations with different `poseType` values to define several reference poses in one asset.
- **Discovery**: Iterate the `animations` array and check for the presence of this extension.
- **Playback**: A reference-pose animation can be "played" like any animation to set the hierarchy to the pose - useful for visualization/debugging.

## Known Implementations

(To be added as implementations become available.)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
