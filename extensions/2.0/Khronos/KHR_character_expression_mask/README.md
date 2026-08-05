# KHR_character_expression_mask

## Contributors

- Ken Jakubzak, Meta
- Hideaki Eguchi / VirtualCast, Inc.
- K. S. Ernest (iFire) Lee, Independent Contributor / <https://github.com/fire>
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

Requires the extensions: `KHR_character` and `KHR_character_expression`.

Assets using `KHR_character_expression_mask` MUST list `KHR_character_expression_mask`, `KHR_character_expression`, and `KHR_character` in `extensionsUsed`. They MUST contain the top-level `KHR_character` and `KHR_character_expression` extension objects. A `KHR_character_expression_mask` object MUST be attached to an expression entry in the top-level `KHR_character_expression.expressions` array.

## Overview (Informative)

`KHR_character_expression_mask` lets one native expression attenuate another expression's driver before response-animation evaluation. Standard `blend` and `block` masks are deterministic scalar factors evaluated from one immutable native-driver vector.

This extension does not acquire drivers, select an expression mapping set, define final animation mixing, or prescribe how sampled property values combine with other animation systems.

## Extension usage

The extension object contains a nonempty `masks` array. Each mask's implicit source is the array index of the expression entry containing that extension object. Its required `target` is an index into the same top-level expressions array.

An asset MAY list `KHR_character_expression_mask` in `extensionsRequired`. A consumer claiming support for an asset using this extension MUST evaluate every standard mask and apply the custom-type rules below. A consumer that cannot provide that behavior MUST treat the asset as unsupported when the extension is listed in `extensionsRequired`.

### Extension object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `masks` | array | Yes | Nonempty array of mask objects originating from the containing expression. |

### Mask object

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `target` | integer | Yes | — | Index of the native expression attenuated by this mask. |
| `name` | string | No | — | Diagnostic copy of the target expression's label. When present, it MUST match exactly and case-sensitively. |
| `type` | string | No | `"blend"` | Standard `"blend"` or `"block"`, or a vendor-extension-name-shaped custom token. |
| `amount` | number | No | `1` | Attenuation amount in `[0, 1]`. |
| `threshold` | number | No | `0` | Strict activation threshold in `[0, 1]` for `"block"`. |

The `target` index is authoritative. Consumers MUST NOT resolve a mask by `name`.

## Immutable input and output vectors

The host supplies one final native-driver vector after any host-defined driver acquisition, mapping-set selection, and arbitration between direct-native and mapped input surfaces. Let `u[i]` be the finite value in `[0, 1]` for native expression index `i`. This entire vector is an immutable snapshot for one mask evaluation.

The mask operation produces an effective driver vector `v` of the same length. Every mask factor reads only `u`; no factor reads a partially computed or final value from `v`.

For a mask `m`, define:

```text
source(m) = index of the expression entry containing m
target(m) = m.target
amount(m) = m.amount when present; otherwise 1
threshold(m) = m.threshold when present; otherwise 0
```

## Standard mask factors

For a standard `"blend"` mask:

```text
factor(m, u) = 1 - amount(m) * u[source(m)]
```

For a standard `"block"` mask:

```text
factor(m, u) =
    1 - amount(m)    when u[source(m)] > threshold(m)
    1                otherwise
```

The comparison is strictly greater-than. When the source value equals the threshold, the block mask does not activate. The `threshold` property has no standard effect on a `"blend"` mask.

Given all masks in all expression entries, each effective target driver is:

```text
v[t] = u[t] * product(
    factor(m, u)
    for every mask m where target(m) == t
)
```

An empty product is `1`, so a driver with no incoming masks is unchanged. Because `u`, `amount`, `threshold`, and every supported factor are in `[0, 1]`, every value in `v` is also in `[0, 1]`.

All masks targeting an expression participate in the product. Duplicate masks contribute duplicate factors. A self-mask reads the source's unmasked value from `u` and contributes normally. Cycles do not recurse: every edge in a cycle reads the same immutable `u`. JSON object order, expression-array traversal order, and mask-array order do not affect the mathematical result.

An expression's outgoing masks always read its value from `u`, even when that expression also has incoming masks and therefore a different value in `v`.

## Custom mask types

A custom mask type is any value other than `"blend"` or `"block"`. It MUST be shaped like a glTF vendor extension name: an uppercase prefix, an underscore, and a lowercase snake-case suffix, for example `"ACME_expression_mask_curve"`.

A custom token MAY appear without a companion extension object. A bare custom token, or a custom token whose companion is not supported, has the deterministic identity factor:

```text
factor(m, u) = 1
```

Portable custom behavior requires an extension object on the same mask object whose key exactly equals the mask's `type` value. That companion specification MUST define exactly one finite factor in `[0, 1]` for the mask. The factor MUST be a deterministic function only of the immutable vector `u` and authored data on the mask and companion objects. It MUST NOT read any value from `v`, prior evaluations, time, or unspecified host inputs; mutate a driver; directly replace `v[target(m)]`; or otherwise bypass the product equation. A supported companion's factor participates in the same product as every standard factor.

The companion extension MUST be listed in `extensionsUsed`. If correct presentation depends on its custom factor, both `KHR_character_expression_mask` and the companion extension MUST be listed in `extensionsRequired`. An unsupported optional companion uses the identity factor `1`; an unsupported required companion makes the asset unsupported under the ordinary glTF extension rules.

An unrelated or differently named extension object on the same mask does not define the custom type. It follows its own specification, while the unmatched custom token uses the identity factor.

An implementation claiming read-write or round-trip support for this extension that retains a custom mask when re-emitting an asset MUST preserve its unrecognized custom `type` token. If it retains a same-object companion extension, it MUST preserve that extension's payload. Deliberately removing a custom mask or companion extension is an authoring operation and requires the ordinary corresponding updates to `extensionsUsed` and `extensionsRequired`. Render-only consumers are not required to provide serialization.

## Response-evaluator boundary

The effective vector `v` is the mask operation's output. A character-expression response evaluator consumes `v[i]` as the current driver for expression entry `i`.

This extension does not define how simultaneous response animations are accumulated, prioritized, blended, or otherwise composed when they affect the same property. Those final property-composition choices remain host-defined.

## Standard-mask example (Informative)

The following partial fragment omits dependency objects and referenced animations. Expression indices are shown in comments outside the JSON: `aa` is index 0, `happy` is index 1, and `angry` is index 2.

```json
{
  "extensions": {
    "KHR_character_expression": {
      "expressions": [
        {
          "expression": "aa",
          "animation": 0
        },
        {
          "expression": "happy",
          "animation": 1,
          "extensions": {
            "KHR_character_expression_mask": {
              "masks": [
                {
                  "target": 0,
                  "name": "aa",
                  "type": "blend",
                  "amount": 0.5
                }
              ]
            }
          }
        },
        {
          "expression": "angry",
          "animation": 2,
          "extensions": {
            "KHR_character_expression_mask": {
              "masks": [
                {
                  "target": 0,
                  "name": "aa",
                  "type": "block",
                  "amount": 1.0,
                  "threshold": 0.2
                }
              ]
            }
          }
        }
      ]
    }
  }
}
```

For `u = [1, 0.5, 0.2]`, the blend factor is `0.75` and the block factor is `1` because the strict threshold comparison is false. Therefore `v[0] = 0.75`. If `u[2]` is greater than `0.2`, the block factor becomes `0` and `v[0]` becomes `0`.

## Custom-type examples (Informative)

This bare custom mask is valid and evaluates to identity under this extension:

```json
{
  "target": 0,
  "type": "ACME_expression_mask_curve"
}
```

The following partial fragment demonstrates where a portable companion extension would be attached. The payload is illustrative; only the `ACME_expression_mask_curve` specification can define its factor.

```json
{
  "target": 0,
  "type": "ACME_expression_mask_curve",
  "extensions": {
    "ACME_expression_mask_curve": {
      "controlPoints": [0.0, 0.25, 1.0]
    }
  }
}
```

An asset containing the companion object must list `ACME_expression_mask_curve` in `extensionsUsed`. When the custom factor is essential, it must list both `KHR_character_expression_mask` and `ACME_expression_mask_curve` in `extensionsRequired`.

## Validation boundary

The JSON Schemas enforce nonempty mask arrays, required non-negative target indices, nonempty diagnostic names, standard or vendor-extension-name-shaped type strings, and `amount` and `threshold` values in `[0, 1]`.

A validator that claims to validate this extension MUST additionally:

- verify placement on an expression entry;
- resolve every `target` against the top-level expressions array;
- verify every optional `name` against the referenced expression label;
- verify the base extension declarations and dependency objects;
- for a custom type with a same-named companion object, verify ordinary `extensionsUsed` and `extensionsRequired` declarations; and
- apply any additional structural rules defined by a supported companion specification.

Bare custom types are valid and use the identity fallback. A validator does not execute mask equations or validate host-defined final animation composition.

## Known implementations (Informative)

- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity demos.

## Schema

- [KHR_character_expression.KHR_character_expression_mask.schema.json](./schema/KHR_character_expression.KHR_character_expression_mask.schema.json)
- [KHR_character_expression_mask.mask.schema.json](./schema/KHR_character_expression_mask.mask.schema.json)

## License

This extension is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
