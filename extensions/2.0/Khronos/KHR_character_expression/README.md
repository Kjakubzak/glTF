# KHR_character_expression

## Contributors

- Ken Jakubzak, Meta
- Hideaki Eguchi / VirtualCast, Inc.
- K. S. Ernest (iFire) Lee, Independent Contributor / https://github.com/fire
- Shinnosuke Iwaki / VirtualCast, Inc.
- 0b5vr / pixiv Inc.
- Leonard Daly, Independent Contributor
- Nick Burkard, Meta
- Sarah Cooney, Microsoft XGTG
- Aaron Franke, Independent Contributor

## Status

**Draft** – This extension is not yet ratified by the Khronos Group and is subject to change.

## Conventions

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) when, and only when, they appear in all capitals, as shown here.

## Dependencies

Written against the glTF 2.0 specification.

Requires the extension: `KHR_character`.

Assets using `KHR_character_expression` **MUST** list `KHR_character_expression` and `KHR_character` in `extensionsUsed`. They **MUST** contain top-level `KHR_character_expression` and `KHR_character` extension objects.

Other extensions may define animation-channel targets or add typed metadata to expression entries. Their normal `extensionsUsed` and `extensionsRequired` rules continue to apply.

An asset MAY list `KHR_character_expression` in `extensionsRequired`. A consumer claiming support for this extension MUST implement the response evaluator for concrete core node translation, rotation, scale, and morph-weight targets. Support for extension-defined animation targets remains an independent capability governed by each target extension.

## Overview (Informative)

This extension associates named character-expression drivers with glTF animations and defines a normalized, stateless way to sample those animations as expression responses. A driver may come from user data, another animation system, a procedural system, or any other host source.

An expression response is a set of absolute property samples. This extension does not apply those samples to the asset state and does not define how a host combines them with ordinary animation, other expressions, or other runtime systems.

## Extension schema (Normative)

The following partial fragment omits the required `KHR_character` object and the referenced animations.

```json
{
  "extensions": {
    "KHR_character_expression": {
      "expressions": [
        {
          "expression": "smile",
          "animation": 0
        },
        {
          "expression": "blinkLeft",
          "animation": 1
        }
      ]
    }
  }
}
```

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `expressions` | array | Nonempty array of expression entries. Array indices are the authoritative identities of the entries. |
| `expression` | string | Nonempty descriptive label for the containing entry. Labels are not required to be unique. |
| `animation` | integer | Index of the glTF animation used as this entry's response curve. |

Each expression entry is a glTF property and may therefore contain `extensions` and `extras`.

The `animation` value **MUST** be a valid index into the top-level `animations` array. Duplicate `expression` labels are valid; consumers **MUST NOT** resolve an expression entry by assuming its label is unique. Extensions that refer to expression entries use their array indices unless they explicitly define another mechanism.

When two or more entries reference the same animation, each reference defines an independent evaluator input and result. Referencing an animation here neither changes nor prevents ordinary playback of that animation with its authored timeline.

## Expression response evaluator (Normative)

For an expression entry, let `A` be the animation selected by `animation` and let `x` be the current optional driver supplied to the evaluator. A supplied `x` **MUST** be finite. If no driver is supplied, `x` is `0`. The effective driver is:

```text
e = clamp(x, 0, 1)
```

If `e` is `0`, the evaluator **MUST** return an empty response and **MUST NOT** emit the authored initial values described below. Otherwise, it evaluates `A` as specified in the following sections.

Evaluation is stateless. A new evaluation replaces the previous response contribution from the same expression entry; it does not modify the glTF asset, the authored initial state, or the state used by a later evaluation. For the same asset and current driver value, the evaluator MUST return the same response regardless of prior driver values, evaluation cadence, or update history.

### Channel set and duration

Let:

- `C` be every channel in `A.channels`;
- `S` be the set of sampler indices referenced by the channels in `C`; and
- `last(s)` be the final decoded input key time of sampler `s`.

Every sampler in `S` **MUST** contain at least two input keys. Samplers in `A.samplers` that are not referenced by a channel in `C` are not members of `S` and do not affect expression evaluation.

Each referenced input accessor **MUST** satisfy the core animation-input requirements: `SCALAR` type, `FLOAT` component type, defined `min` and `max`, finite nonnegative values, and strictly increasing decoded key times. Its `count` **MUST** be at least two. These requirements **MUST** be checked against decoded resource data; accessor metadata alone cannot establish strict ordering or finiteness.

The expression duration is:

```text
T = max(last(s)) for all s in S
```

Core glTF requires animation input keys to be nonnegative and strictly increasing. Consequently, the requirements above make `T` finite and greater than zero.

Every channel in `C`, including a channel whose target is defined by an optional extension that a particular implementation does not support, contributes its referenced sampler to `S` and therefore to `T`.

### Channel targets

Every channel in `C` **MUST** resolve to a concrete mutable Asset Object Model property under core glTF or the normative semantics of a declared animation-target extension. Merely placing an extension object on a target does not satisfy this requirement unless that extension defines the concrete mutable property target. A core channel with no `target.node` and no valid extension-defined property target is invalid for use by this extension.

When `KHR_animation_pointer` supplies a target, its pointer **MUST NOT** resolve into any `extras` object. Application-defined animation of `extras`, although permitted by `KHR_animation_pointer`, cannot provide a portable expression response.

All target, sampler, accessor, interpolation, value, and extension-specific validity requirements from core glTF and the target extension continue to apply.

Within `A`, a concrete property **MUST NOT** be targeted by more than one channel. A channel targeting an entire array overlaps a channel targeting any element of that array. This prohibition applies across core and extension-defined target forms as well as within either form.

### Sampling

For `e > 0`, the expression sample time is:

```text
t = e * T
```

Each channel in `C` whose target is supported by the implementation is sampled at `t` using the conversion, interpolation, and endpoint-clamping rules defined by core glTF and by its target extension. Expression evaluation does not wrap, repeat, reverse, rescale, or otherwise alter the animation timeline.

Each supported channel produces a response record containing its concrete target property and the ordinary absolute glTF value sampled for that property. Samples are not deltas from the authored state, blend weights, or values to be scaled again by `e`.

Each referenced output accessor **MUST** have the component type, accessor type, dimensions, and count required by its concrete target. It supplies one complete logical target value per input key for `LINEAR` and `STEP`, and the corresponding in-tangent, value, and out-tangent records for `CUBICSPLINE`. Scalar-packed array targets, including morph weights, **MUST** include every array element for every key and three times that number of elements for `CUBICSPLINE`. Target-specific extensions may further constrain these rules.

If a channel target is defined only by an optional extension that the implementation does not support, that implementation emits no response record for the channel. The channel still contributes to `T`. Supporting `KHR_character_expression` does not make such a target extension required. If the target extension is listed in `extensionsRequired`, the ordinary glTF unsupported-required-extension rules apply.

## Authored initial state (Normative)

Every channel in `C` **MUST** sample, at time `0`, a value equivalent to the authored initial value of its target property. This invariant lets a positive driver move away from the asset's authored state while an effective driver of zero contributes no property value.

The authored initial value is resolved from the asset before ordinary animation, expression responses, or other runtime systems are applied:

1. Use the property's explicitly authored Asset Object Model value when present.
2. Otherwise, use the property's specification-defined default value.
3. For node morph weights, use `node.weights` when present, otherwise the referenced `mesh.weights` when present, otherwise a zero for each morph target.

For clarity, the core node TRS defaults are translation `[0, 0, 0]`, rotation `[0, 0, 0, 1]`, and scale `[1, 1, 1]`. Extension properties use the authored values and defaults defined by their own specifications. A target with neither an authored value nor a specification-defined default is invalid unless its target specification defines another initial-value rule.

The time-zero sample is the value obtained by sampling the channel at `t = 0` under ordinary glTF rules, including endpoint clamping when its first key time is greater than zero. For `CUBICSPLINE`, this is the sampled value, not an in-tangent or out-tangent.

### Initial-value equivalence

The authored initial value and decoded time-zero sample **MUST** have the same Asset Object Model type and dimensions. Boolean and integer values are compared exactly after the conversions required by the target extension. Floating-point scalars, vector components, array elements, and matrix components are compared as follows.

For an authored component `a` and decoded sampled component `b`, the component passes when:

```text
abs(a - b) <= 1e-6 + h + 1e-6 * max(abs(a), abs(b))
```

The accessor allowance `h` is selected from the channel's output accessor after the conversion required by the concrete target. It applies only to the decoded sampled value `b`, never to the authored-or-default AOM value `a`; it is not an allowance for JSON-side quantization. For `CUBICSPLINE`, it applies to decoded value slots and not to tangent slots:

| Output accessor encoding | `h` |
| --- | ---: |
| `FLOAT` | `0` |
| Non-normalized integer converted to a floating-point AOM value | `0` |
| Normalized `BYTE` | `0.5 / 127` |
| Normalized `UNSIGNED_BYTE` | `0.5 / 255` |
| Normalized `SHORT` | `0.5 / 32767` |
| Normalized `UNSIGNED_SHORT` | `0.5 / 65535` |

For a property whose specification defines quaternion semantics, normalize the authored quaternion `q_a` and decoded sampled quaternion `q_b`, treating `q` and `-q` as equivalent. The quaternions pass when:

```text
1 - abs(dot(q_a, q_b)) <= 1e-6 + q_h
```

For a normalized integer output accessor, `q_h = 8 * h * h` using the table above. For `FLOAT` or a non-normalized integer converted to floating point, `q_h = 0`. This quaternion rule replaces componentwise comparison for that property. Zero-length or otherwise invalid quaternions remain invalid under the property's existing rules.

These checks require decoding the referenced output accessor, including buffer data and any sparse values. More generally, validation that cannot access all referenced input and output resources is incomplete for the decoded-input, duration, target-value, and authored-initial-state requirements. It **MUST NOT** report those requirements as successfully validated solely from the JSON document.

Likewise, a validator lacking the schema or semantic support for an optional declared animation-target extension cannot completely validate the affected target, output conversion, or initial value. It **MUST NOT** reject the asset solely because that optional target extension is unsupported, and it **MUST NOT** report the affected expression checks as complete.

## Typed expression extensions (Normative)

Typed child extensions such as `KHR_character_expression_joint`, `KHR_character_expression_morphtarget`, and `KHR_character_expression_texture` classify channels and add type-specific validity requirements. They do not select the channels evaluated by the base extension.

- A listed channel index **MUST** resolve against `channels` of the animation selected by the containing entry's `animation` property.
- The same channel index **MUST NOT** occur in more than one of these three typed child extensions on the same expression entry.
- A channel not listed by a typed child extension is still a member of `C` and is evaluated normally.
- Lack of support for an optional typed child extension does not suppress or alter evaluation of the underlying channel. Ordinary `extensionsRequired` handling still applies.

## Host integration boundary (Normative)

This extension does not define an Asset Object Model property that stores `x`, nor does it define how a host obtains or updates a driver. Driver sources may include user input, tracked data, procedural logic, another animation system, or other host data.

The following remain host-defined:

- evaluation cadence and ordering;
- driver smoothing, prediction, and arbitration;
- wiring from other animations or systems into expression drivers;
- prevention or handling of host-created feedback cycles; and
- accumulation, blending, prioritization, or other composition of response records with ordinary animations, simultaneous expressions, and other property writers.

No behavior in this section changes the normative sampling and response-record rules above.

## Extension example with typed metadata (Informative)

The following partial fragment omits dependency objects and referenced animations.

```json
{
  "extensions": {
    "KHR_character_expression": {
      "expressions": [
        {
          "expression": "smile",
          "animation": 0,
          "extensions": {
            "KHR_character_expression_joint": {
              "channels": [0, 1]
            },
            "KHR_character_expression_texture": {
              "channels": [2]
            },
            "KHR_character_expression_morphtarget": {
              "channels": [3, 4]
            }
          }
        }
      ]
    }
  }
}
```

The classifications do not prevent any additional channel in animation `0` from contributing to the `smile` response.

## Known limitations (Informative)

Hosts can produce different final rendered states when they use different composition policies for multiple property writers. That variation is outside this extension's response-evaluator contract and outside the core glTF animation model.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js loader and VRM conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity demos.

## License

This extension is licensed under the Khronos Group Extension License.
See: https://www.khronos.org/registry/gltf/license.html
