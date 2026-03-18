# KHR_animation_type

## Contributors

- Ken Jakubzak, Meta

## Status

**Draft** – This extension is not yet ratified by the Khronos Group and is subject to change.

## Dependencies

Written against the glTF 2.0 specification.

**Optional**: `KHR_character_reference_pose` (provides rest pose for additive delta computation)

## Overview

The `KHR_animation_type` extension declares the **blend semantics** of individual animations. By tagging animations as `"override"` or `"additive"`, this extension enables animation systems to correctly combine animations in a predictable, portable manner.

- **Override animations** fully replace the base pose. This is the standard glTF 2.0 behavior and the default.
- **Additive animations** add their deltas to the base pose, allowing animations like breathing, hit reactions, or procedural offsets to be layered on top of existing animations.

## Extension Schema

This extension is applied to individual animations in the glTF file.

**Schema**: [animation.KHR_animation_type.schema.json](schema/animation.KHR_animation_type.schema.json)

```json
{
  "animations": [
    {
      "name": "IdleBase",
      "channels": [...],
      "samplers": [...]
    },
    {
      "name": "AdditiveBreathing",
      "extensions": {
        "KHR_animation_type": {
          "type": "additive"
        }
      },
      "channels": [...],
      "samplers": [...]
    }
  ]
}
```

### Properties

| Property | Type   | Required | Default      | Description |
|----------|--------|----------|--------------|-------------|
| `type`   | string | No       | `"override"` | The blend semantics of this animation. |

### type Values

| Value       | Description |
|-------------|-------------|
| `override`  | Animation values fully replace the base pose. Standard glTF behavior. |
| `additive`  | Animation values represent deltas applied on top of the base pose. |
| *custom*    | Any string for application-specific types (e.g., `"multiplicative"`). Custom types do not require extension registration. |

## Additive Animation Mathematics

Additive animations require a **reference pose** to compute deltas. The recommended reference is provided by `KHR_character_reference_pose`. If no reference pose is defined, implementations SHOULD evaluate all channels at `t=0` of the additive animation and use those values as the per-channel reference.

### Delta Computation

For each channel, the additive delta depends on the transform type:

| Transform Type   | Delta Computation |
|------------------|-------------------|
| **Translation**  | `delta = keyframe - reference` |
| **Rotation**     | `delta = inverse(reference) * keyframe` |
| **Scale**        | `delta = keyframe / reference` (component-wise) |
| **Morph Weights**| `delta = keyframe - reference` |

### Blend Operations

When blending an additive animation with weight `w` onto a base value:

| Transform Type   | Blend Operation |
|------------------|-----------------|
| **Translation**  | `result = base + delta * weight` |
| **Rotation**     | `result = normalize(base * slerp(identity, delta, weight))` |
| **Scale**        | `result = base * lerp(1.0, delta, weight)` (component-wise) |
| **Morph Weights**| `result = base + delta * weight` |

### Rotation Details

Rotation blending uses quaternion multiplication with **shortest-path slerp**:

```c
// Compute delta from reference
delta = inverse(reference_rotation) * keyframe_rotation

// Shortest-path check: negate if necessary to take shorter arc
if (dot(identity, delta) < 0) {
    delta = -delta;
}

// Apply weighted delta to base
weighted_delta = slerp(identity, delta, weight)
result = normalize(base * weighted_delta)
```

Implementations MUST normalize the resulting quaternion. Normalization SHOULD occur after accumulating 4 or more quaternion multiplications to prevent numerical drift.

### Scale Safety

When computing scale deltas, division by the reference scale may cause issues if the reference is near zero:

```c
// For each component:
if (|reference_scale| < 1e-7) {
    delta = 1.0  // Fallback to identity delta
} else {
    delta = keyframe_scale / reference_scale
}
```

### Rotation Safety

When computing rotation deltas:

- If the reference quaternion is degenerate (`|reference| < 1e-7`), use identity as reference
- If `dot(reference, reference) ≈ 0`, skip the additive contribution for this channel

### Morph Weight Behavior

Morph weight deltas are NOT automatically clamped. The resulting values may exceed the [0, 1] range:

- Applications may clamp values for display purposes
- Negative values are valid and represent "inverse" morphing
- Values > 1.0 represent exaggerated morphing

## Example: Additive Breathing

This example shows a breathing animation layered on top of any base pose:

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": ["KHR_animation_type"],
  "animations": [
    {
      "name": "IdleStand",
      "channels": [
        { "sampler": 0, "target": { "node": 1, "path": "rotation" } }
      ],
      "samplers": [
        { "input": 0, "output": 1 }
      ]
    },
    {
      "name": "AdditiveBreathing",
      "extensions": {
        "KHR_animation_type": {
          "type": "additive"
        }
      },
      "channels": [
        { "sampler": 0, "target": { "node": 2, "path": "scale" } }
      ],
      "samplers": [
        { "input": 2, "output": 3 }
      ]
    }
  ]
}
```

> **Note**: `accessors`, `bufferViews`, and `buffers` omitted for brevity.

## Relationship to KHR_animation_layers

While `KHR_animation_type` declares the **intrinsic** blend type of an animation, the `KHR_animation_layers` extension may optionally override this on a per-layer basis using the `typeOverride` property. This allows the same animation clip to be used as override in one context and additive in another.

Precedence order:

1. `typeOverride` on layer animation reference (if specified)
2. `KHR_animation_type.type` on the animation (if extension present)
3. Default: `"override"`

## Backwards Compatibility

- Animations without this extension default to `"override"` behavior, maintaining full backwards compatibility with glTF 2.0.
- This extension SHOULD be listed in `extensionsUsed`.
- **Warning**: Playing an additive animation as override will produce incorrect results (the additive pose replaces the entire character pose). For assets that require additive blending for correct display, this extension SHOULD be listed in `extensionsRequired`.
- For graceful degradation, consider including fallback override-mode animations for viewers that don't support additive blending.

## Conformance

A conforming implementation of this extension:

- MUST support both `"override"` and `"additive"` blend types
- MUST use shortest-path slerp for rotation blending
- MUST normalize final quaternion output
- MUST implement zero-scale protection as specified
- SHOULD normalize quaternions after every 4 accumulated operations
- SHOULD reject animations with NaN or Infinity in keyframe data

## Implementation Notes

- **Reference Pose**: For best results, use `KHR_character_reference_pose` to define a canonical rest pose. This ensures consistent delta computation across implementations.
- **Performance**: Implementations may precompute deltas from the reference pose during asset loading.
- **Validation**: Reject animations with NaN or Infinity values in keyframe data.

## DCC Tool Mapping

| DCC Tool | Source Feature |
|----------|----------------|
| Blender  | NLA Strip Blend Mode (Replace vs Add) |
| Maya     | Animation Layer Modes |
| Unity    | Animation Blend Mode |
| Unreal   | Additive Animation Settings |

## Known Implementations

(To be added as implementations become available)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
