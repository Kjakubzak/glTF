# KHR_animation_mask

## Contributors

- Ken Jakubzak, Meta

## Status

**Draft** - This extension is not yet ratified by the Khronos Group and is subject to change.

## Dependencies

Written against the glTF 2.0 specification. No other extensions are required.

**Optional**: `KHR_animation_layers` (a layer stack can apply these masks as a relational pre-pass on animation weights).

## Overview

The `KHR_animation_mask` extension defines how one animation **masks (attenuates) the weight of another
animation**. It lets an asset declare, data-side, that when one animation is active it should reduce or suppress
another - resolving conflicts between animations that would otherwise combine incorrectly.

This is a **relational** mask: it operates on animation *weights* (how much each animation contributes), not on
which joints an animation affects. It is the generalized, non-character counterpart of
`KHR_character_expression_mask` (which masks facial *expressions*); here the same blend/block semantics apply to
any glTF animations.

> **Relational vs. spatial masking.** This extension is distinct from the **skeletal masks** in
> `KHR_animation_layers`, which are *spatial* (per-joint weights controlling *which nodes* a layer affects). The two
> are complementary and may be used together: skeletal masks scope *where* an animation applies; `KHR_animation_mask`
> scopes *how much* one animation applies relative to another.

### Motivation

When two animations are active at once - e.g. a "reload" upper-body action while a "wave" gesture also plays, or a
procedural override competing with an authored clip - the result may be incorrect or double-applied depending on
the runtime. Whether they should blend, and by how much, is a property of the asset, not the application. This
extension makes that relationship explicit and portable.

## Use Cases

- Reduce one animation's influence while another is active (e.g. a hit-reaction softening a locomotion gesture).
- Fully suppress an animation when a higher-priority one crosses a threshold.
- Give artists deterministic, cross-platform control over animation priority and blending.

## Behavior

An animation carrying `KHR_animation_mask` defines how it affects **target** animations. Multiple masks (from
different source animations) may target the same animation; the total influence is the **product** of all mask
influences. Let `source_w` be the weight (0.0-1.0) of the animation carrying the mask, and `target_w_in` the
target animation's incoming weight.

### "blend" Masks

A `blend` mask reduces the target proportionally to the source animation's weight:

```text
target_w_out = target_w_in * (1 - amount * source_w)
```

When the source is fully active (`1.0`) with `amount` `0.5`, the target's weight is reduced to 50%.

### "block" Masks

A `block` mask fully reduces the target (by `amount`) once the source exceeds a threshold:

```text
target_w_out = target_w_in * (1 - (source_w > threshold ? amount : 0))
```

When `threshold` is `0`, the target is blocked unless the source is completely inactive.

### Execution Order

The result is independent of evaluation order because masks read only the **input** weights of animations, and the
total influence on a target is the product of all masks targeting it (multiplication is commutative).

## Schema

```json
{
  "extensionsUsed": ["KHR_animation_mask"],
  "animations": [
    { "name": "Walk", "channels": [], "samplers": [] },
    { "name": "WaveHand", "channels": [], "samplers": [] },
    {
      "name": "HitReaction",
      "channels": [], "samplers": [],
      "extensions": {
        "KHR_animation_mask": {
          "masks": [
            { "target": 1, "type": "blend", "amount": 0.75 },
            { "target": 0, "type": "block", "threshold": 0.2 }
          ]
        }
      }
    }
  ]
}
```

Here, while `HitReaction` is active it blends `WaveHand` (animation 1) down to 25% and blocks `Walk` (animation 0)
once its own weight passes `0.2`.

## Properties

### KHR_animation_mask

| Property | Type  | Description                                                              | Required |
| -------- | ----- | ----------------------------------------------------------------------- | -------- |
| `masks`  | array | An array of mask objects defining how this animation attenuates others. | Yes      |

### Mask Object

| Property    | Type    | Description                                                                                   | Required |
| ----------- | ------- | -------------------------------------------------------------------------------------------- | -------- |
| `target`    | integer | The index of the target animation whose weight this mask attenuates.                         | Yes      |
| `type`      | string  | The type of the mask: `"blend"` or `"block"`. Default: `"blend"`.                            | No       |
| `amount`    | number  | The amount of influence (0.0-1.0). Default: `1.0`.                                           | No       |
| `threshold` | number  | For `"block"` type: the weight above which the target is blocked (0.0-1.0). Default: `0.0`.  | No       |

### Mask Type Enum

| Value   | Description                                                                                 |
| ------- | ------------------------------------------------------------------------------------------- |
| `blend` | Reduces the target proportionally to this animation's weight multiplied by the amount.      |
| `block` | Fully reduces the target by the amount when this animation's weight exceeds the threshold.   |

## Relationship to KHR_animation_layers

`KHR_animation_layers` composites multiple animations via an ordered layer stack with per-layer/per-animation
weights and **spatial** skeletal masks. `KHR_animation_mask` is a **relational** pre-pass on animation weights:
before a layer evaluates, a consuming implementation resolves all `KHR_animation_mask` influences to produce each
animation's effective weight, then composites as usual. Using both, an asset can say *where* an animation applies
(skeletal mask) and *how much it yields to another animation* (this extension), without application-specific logic.

## Implementation Notes

- Multiple masks from different animations may target the same animation; the combined effect is multiplicative.
- The `threshold` property is only meaningful for `"block"` type masks.
- Implementations should evaluate all mask influences before applying animation weights.
- Animation weights should be normalized to the [0.0-1.0] range.

## Blending Behavior Recommendations

When blending values from multiple sources, implementations **SHOULD** use traditional linear interpolation (lerp):

```text
result = lerp(base_value, blend_value, blend_weight)
       = base_value + blend_weight * (blend_value - base_value)
```

## License

This extension is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
