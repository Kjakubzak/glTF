# KHR_animation_layers

## Contributors

- Ken Jakubzak, Meta

## Status

**Draft** – This extension is not yet ratified by the Khronos Group and is subject to change.

## Dependencies

Written against the glTF 2.0 specification.

**Optional**: `KHR_animation_type` (for intrinsic animation blend types)

**Optional**: `KHR_character_expression_mask` (for expression pre-pass integration)

**Optional**: `KHR_animation_events` (for sync markers and event integration)

## Overview

The `KHR_animation_layers` extension defines an **ordered animation layer stack** that enables sophisticated animation blending, prioritization, and masking. This provides a portable, declarative way to describe how multiple animations combine to produce final character poses.

Key features:

- **Layer Stack**: Ordered layers evaluated from base (index 0) to top (highest index)
- **Layer Weights**: Per-layer blend weights for cross-fading and dynamic blending
- **Skeletal Masks**: Restrict layer influence to specific joints (define-then-reference pattern)
- **Animation References**: Multiple animations per layer with individual weights
- **Type Override**: Override intrinsic animation blend type on a per-reference basis

## Extension Schema

This extension is applied at the root glTF level (or optionally per-scene).

**Schema**: [scene.KHR_animation_layers.schema.json](schema/scene.KHR_animation_layers.schema.json)

```json
{
  "extensions": {
    "KHR_animation_layers": {
      "masks": [
        {
          "name": "UpperBody",
          "nodes": [5, 6, 7, 8, 9, 10, 11, 12],
          "weights": [1.0, 1.0, 1.0, 1.0, 0.8, 0.8, 0.5, 0.5],
          "includeDescendants": true
        },
        {
          "name": "LowerBody",
          "nodes": [0, 1, 2, 3, 4],
          "weights": [1.0, 1.0, 1.0, 1.0, 1.0],
          "includeDescendants": true
        }
      ],
      "layers": [
        {
          "name": "BaseLocomotion",
          "weight": 1.0,
          "animations": [
            { "animation": 0, "weight": 0.7 },
            { "animation": 1, "weight": 0.3 }
          ]
        },
        {
          "name": "UpperBodyGesture",
          "weight": 1.0,
          "mask": 0,
          "animations": [
            { "animation": 2 }
          ]
        },
        {
          "name": "AdditiveBreathing",
          "weight": 0.5,
          "animations": [
            { "animation": 3, "typeOverride": "additive" }
          ]
        }
      ]
    }
  }
}
```

## Define-Then-Reference Mask Pattern

Following the pattern established by `KHR_lights_punctual`, masks are **defined once** at the extension root and **referenced by index** from layers. This provides:

- **Reusability**: Same mask shared across multiple layers without duplication
- **Consistency**: Single source of truth for mask definitions
- **Efficiency**: Reduced file size and simplified validation

### Mask Definition (Root Level)

```json
{
  "extensions": {
    "KHR_animation_layers": {
      "masks": [
        {
          "name": "UpperBody",
          "nodes": [5, 6, 7, 8, 9],
          "weights": [1.0, 1.0, 1.0, 0.8, 0.5],
          "includeDescendants": true
        }
      ],
      "layers": [...]
    }
  }
}
```

### Mask Reference (Layer Level)

```json
{
  "layers": [
    {
      "name": "GestureLayer",
      "mask": 0,
      "animations": [...]
    },
    {
      "name": "CombatOverride",
      "mask": 0,
      "animations": [...]
    }
  ]
}
```

## Extension Properties

### Root Extension Object

| Property | Type   | Required | Description |
|----------|--------|----------|-------------|
| `masks`  | array  | No       | Array of mask definitions. Referenced by index from layers. |
| `layers` | array  | Yes      | Ordered array of layer objects. Index 0 is base layer. |

### Mask Object

| Property             | Type    | Required | Default | Description |
|----------------------|---------|----------|---------|-------------|
| `name`               | string  | No       |         | Human-readable mask name. |
| `nodes`              | array   | Yes      |         | Array of node indices affected by this mask. Must be unique. |
| `weights`            | array   | Yes      |         | Parallel array of weights [0,1] for each node. MUST have same length as `nodes`. |
| `includeDescendants` | boolean | No       | `true`  | If true, mask affects descendants of specified nodes. |

### Layer Object

| Property           | Type    | Required | Default | Description |
|--------------------|---------|----------|---------|-------------|
| `name`             | string  | No       |         | Human-readable layer name. |
| `weight`           | number  | No       | `1.0`   | Layer blend weight [0,1]. Affects all animations in this layer. |
| `mask`             | integer | No       |         | Index into root `masks` array. If omitted, layer affects all nodes. |
| `blendMorphTargets`| boolean | No       | `true`  | If false, morph target channels in this layer's animations are ignored; morph weights from lower layers pass through unchanged. |
| `animations`       | array   | Yes      |         | Array of animation references in this layer. |

### Animation Reference Object

| Property       | Type    | Required | Default | Description |
|----------------|---------|----------|---------|-------------|
| `animation`    | integer | Yes      |         | Index into glTF `animations` array. |
| `weight`       | number  | No       | `1.0`   | Animation weight [0,1] within this layer. |
| `typeOverride` | string  | No       |         | Override the animation's intrinsic type (`"override"` or `"additive"`). |

## Evaluation Pipeline

### Layer Evaluation Order

Layers are evaluated in **array index order** (0 = base, N = top):

```text
Final Pose = evaluate(layers[N], evaluate(layers[N-1], ... evaluate(layers[1], layers[0])))
```

### Per-Layer Evaluation

For each layer:

1. **Sample animations**: Evaluate all animations at current time
2. **Blend within layer**: Combine animations (see Intra-Layer Blending below)
3. **Apply mask**: Restrict influence to masked nodes
4. **Apply layer weight**: Scale contribution by layer weight
5. **Blend with base**: Combine with accumulated result from lower layers

### Blend Type Resolution

The blend type for each animation is resolved in this order:

1. `typeOverride` on animation reference (if specified)
2. `KHR_animation_type.type` on animation (if extension present)
3. Default: `"override"`

## Intra-Layer Animation Blending

When a layer contains multiple animations, they are blended together before the layer is composited.

### For Override Animations

Weights are **normalized** to sum to 1.0 before blending:

```c
total_weight = sum(animation[i].weight for all i)
normalized_weight[i] = animation[i].weight / total_weight

// For each transform type:
result = sum(animation[i].value * normalized_weight[i])
```

| Transform Type   | Blend Operation |
|------------------|-----------------|
| **Translation**  | `lerp` with normalized weights |
| **Rotation**     | `slerp` with shortest path and normalized weights |
| **Scale**        | `lerp` with normalized weights |
| **Morph Weights**| `lerp` with normalized weights |

### For Additive Animations

Weights are **NOT normalized**. Each animation's delta is scaled independently and accumulated:

```c
result = base + sum(delta[i] * weight[i])
```

See `KHR_animation_type` for detailed additive math.

### For Mixed Layers (Override + Additive)

Override animations blend first (normalized), then additive animations apply their deltas:

```c
override_result = blend(override_animations)  // normalized
final_result = override_result + sum(additive_delta[i] * additive_weight[i])
```

## Layer Composition

### Override Layers

```c
result = lerp(base, layer_value, layer_weight * mask_weight)
```

For rotations, use `slerp` with shortest-path:

```c
result = slerp(base, layer_value, layer_weight * mask_weight)
```

### Additive Layers

```c
result = base + (layer_delta * layer_weight * mask_weight)
```

See `KHR_animation_type` for detailed additive formulas.

### Effective Weight

The effective weight for a node combines layer, animation, and mask weights:

```c
effective_weight = layer_weight * animation_weight * mask_weight
```

## Skeletal Masks

Masks restrict which nodes a layer affects. Nodes not in the mask receive no contribution from the layer.

### Mask Weight Semantics

| Weight | Meaning |
|--------|---------|
| `1.0`  | Full layer influence |
| `0.5`  | Half layer influence (soft blend) |
| `0.0`  | No layer influence (equivalent to not being in mask) |

### Mask Weight Inheritance Rules

When `includeDescendants: true`:

1. Explicitly listed nodes use their specified weight
2. Unlisted descendants inherit weight from nearest listed ancestor
3. Nodes with no listed ancestor in the mask receive weight `0.0` (not affected)

When `includeDescendants: false`:

- Only explicitly listed nodes are affected
- Descendants not in the list receive weight `0.0`

### Descendant Override Example

```json
{
  "masks": [
    {
      "name": "ArmWithGradient",
      "nodes": [5, 6, 7, 8],
      "weights": [0.3, 0.6, 1.0, 1.0],
      "includeDescendants": true
    }
  ]
}
```

If node 5 (shoulder) has children [6 (upper arm), 9 (clavicle)]:

- Node 5: weight `0.3` (explicitly listed)
- Node 6: weight `0.6` (explicitly listed, overrides inheritance)
- Node 9: weight `0.3` (inherits from ancestor node 5)

### Example: Soft Upper-Body Mask

```json
{
  "masks": [
    {
      "name": "UpperBodySoft",
      "nodes": [5, 6, 7, 8, 9],
      "weights": [0.3, 0.6, 1.0, 1.0, 1.0],
      "includeDescendants": true
    }
  ]
}
```

This creates a gradient from spine (30% influence) through chest (60%) to full influence on shoulders/arms.

## Complete Example

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": ["KHR_animation_layers", "KHR_animation_type"],
  "nodes": [
    { "name": "Hips", "children": [1, 4] },
    { "name": "Spine", "children": [2] },
    { "name": "Chest", "children": [3] },
    { "name": "Head" },
    { "name": "LeftLeg" },
    { "name": "RightLeg" }
  ],
  "animations": [
    { "name": "Idle", "channels": [], "samplers": [] },
    { "name": "Walk", "channels": [], "samplers": [] },
    { "name": "WaveHand", "channels": [], "samplers": [] },
    { "name": "AdditiveBreathing", "extensions": { "KHR_animation_type": { "type": "additive" } }, "channels": [], "samplers": [] }
  ],
  "extensions": {
    "KHR_animation_layers": {
      "masks": [
        {
          "name": "UpperBody",
          "nodes": [1, 2, 3],
          "weights": [0.5, 1.0, 1.0],
          "includeDescendants": true
        }
      ],
      "layers": [
        {
          "name": "BaseLocomotion",
          "weight": 1.0,
          "animations": [
            { "animation": 0, "weight": 0.3 },
            { "animation": 1, "weight": 0.7 }
          ]
        },
        {
          "name": "UpperBodyGesture",
          "weight": 1.0,
          "mask": 0,
          "animations": [
            { "animation": 2 }
          ]
        },
        {
          "name": "Breathing",
          "weight": 0.4,
          "animations": [
            { "animation": 3 }
          ]
        }
      ]
    }
  }
}
```

> **Note**: Animation `channels` and `samplers`, `accessors`, `bufferViews`, and `buffers` omitted for brevity.

## Relationship to Other Extensions

### KHR_animation_type

`KHR_animation_type` is **optional** but recommended. It provides intrinsic blend semantics for animations. The `typeOverride` property allows overriding the intrinsic type when referencing animations, so animations can work without `KHR_animation_type` if all references use `typeOverride`.

### KHR_animation_events

Events fire based on animation playback independent of layer weights and masks:

- Events fire even when layer weight is 0
- Events fire even when animation is fully masked
- Applications needing weight-dependent events should implement custom logic

### KHR_character_expression_mask

Expression masks operate at a different semantic level than skeletal masks:

- **Expression masks**: Control blending between facial expressions (pre-pass)
- **Skeletal masks**: Control which joints a layer affects (layer evaluation)

Both can be used together in the same asset.

### KHR_animation_pointer

`KHR_animation_layers` is fully compatible with `KHR_animation_pointer`. Layers can contain animations that target arbitrary JSON properties, not just node transforms.

## Safety & Conformance

### Limits

| Resource | Limit |
|----------|-------|
| Maximum layers | 32 |
| Maximum animations per layer | 1024 |
| Maximum masks | 256 |

### Numeric Safety

- **Weight range**: All weights MUST be in [0, 1]. Values outside this range SHOULD be clamped.
- **NaN/Infinity**: Reject any NaN or Infinity values in weights.
- **Mask validation**: `weights.length` MUST equal `nodes.length`.
- **Quaternion normalization**: Normalize after every 4 quaternion composition operations; MUST normalize final output.
- **Zero-scale protection**: When computing additive scale deltas, use fallback if reference ≈ 0 (see `KHR_animation_type`).

### Shortest-Path Slerp

Rotation blending MUST use shortest-path quaternion interpolation:

```c
if (dot(q1, q2) < 0) {
    q2 = -q2;  // Negate to take shorter arc
}
result = slerp(q1, q2, weight);
```

## Conformance

A conforming implementation of this extension:

- MUST evaluate layers in array index order (0 first)
- MUST support at least 32 layers, 1024 animations per layer, and 256 masks
- MUST normalize weights for override animations within a layer
- MUST NOT normalize weights for additive animations
- MUST use shortest-path slerp for all rotation blending
- MUST normalize final quaternion output
- MUST enforce `weights.length === nodes.length` for masks
- SHOULD clamp weights outside [0, 1] range
- SHOULD reject NaN/Infinity values during validation

## Implementation Notes

- **Precomputation**: Implementations may precompute mask node sets and weights at load time.
- **Sparse evaluation**: Layers with weight 0 may be skipped entirely.
- **Animation caching**: Animation values may be cached when weights/time haven't changed.
- **Morph target handling**: When `blendMorphTargets: false`, skip morph weight channels.

## DCC Tool Mapping

| DCC Tool | Source Feature |
|----------|----------------|
| Blender | NLA Strips, Action Layers |
| Maya | Animation Layers |
| Unity | Animator Layers, Avatar Masks |
| Unreal | Anim Layers, Layer Blend |

## Backwards Compatibility

- If `KHR_animation_layers` is not supported, animations can still be played independently using standard glTF playback.
- Implementations SHOULD play the first animation from the first layer as fallback.
- Alternatively, implementations MAY present a list of all animations for user selection.
- The extension SHOULD be listed in `extensionsRequired` if the asset relies on layer blending for correct presentation.

## Known Implementations

(To be added as implementations become available)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
