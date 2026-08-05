# KHR_character_expression_morphtarget

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

Requires the extensions: `KHR_character` and `KHR_character_expression`.

Assets using `KHR_character_expression_morphtarget` **MUST** list `KHR_character_expression_morphtarget`, `KHR_character_expression`, and `KHR_character` in `extensionsUsed`. They **MUST** contain the top-level `KHR_character` and `KHR_character_expression` extension objects. A `KHR_character_expression_morphtarget` object **MUST** be attached to an expression entry in that top-level `KHR_character_expression` object.

A selected channel may use `KHR_animation_pointer` as described below. Only an asset using that form is required to declare and use `KHR_animation_pointer`; it is not an unconditional dependency of this extension.

An asset MAY list `KHR_character_expression_morphtarget` in `extensionsRequired`. A consumer claiming support for this extension MUST recognize and validate its morph-weight channel classifications. Support for an optional `KHR_animation_pointer` target remains an independent capability and does not change the base evaluator's timing.

## Overview (Informative)

This extension classifies selected channels of an expression animation as node morph-weight channels. It adds validation metadata only. It does not create a separate morph-target binding, select channels for expression evaluation, change their samples, or define a property-composition policy.

The base `KHR_character_expression` extension evaluates every channel in the selected animation whether or not the channel is listed here.

## Extension schema (Normative)

The following is a partial extension fragment. Dependency declarations, character objects, animations, accessors, and buffers are omitted.

```json
{
  "extensions": {
    "KHR_character_expression": {
      "expressions": [
        {
          "expression": "smile",
          "animation": 0,
          "extensions": {
            "KHR_character_expression_morphtarget": {
              "channels": [0, 1]
            }
          }
        }
      ]
    }
  }
}
```

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `channels` | array | Nonempty array of unique channel indices in the animation selected by the containing expression entry. |

## Channel classification and validity (Normative)

Each value in `channels` **MUST** be a valid index into the `channels` array of the animation selected by the containing expression entry's `animation` property.

Each selected channel **MUST** target morph weights through one of these forms:

1. An ordinary core target with a defined `target.node` and `target.path` equal to `"weights"`.
2. A valid `KHR_animation_pointer` target resolving to the complete `/nodes/{node}/weights` property or to one element `/nodes/{node}/weights/{weight}` of that property.

For either form, `{node}` **MUST** identify a node with a defined, valid `mesh` reference, and that mesh **MUST** contain one or more morph targets. For the element form, `{weight}` **MUST** be a valid morph-target index. Core glTF requires all primitives of the mesh to contain the same number `N` of morph targets in the same order.

The selected channel and its sampler **MUST** satisfy all core animation rules, any `KHR_animation_pointer` rules, and all `KHR_character_expression` rules. In particular:

- an ordinary core `weights` output accessor has `SCALAR` type and a component type permitted by core animation;
- for a complete weights target, each key contains exactly `N` scalar weights;
- for an individual pointer target, each key contains exactly one scalar weight;
- an output accessor for `LINEAR` or `STEP` therefore has `input.count * N` elements for a complete weights target and `input.count` elements for an individual target; and
- `CUBICSPLINE` has three times the corresponding element count, ordered under the core morph-weight tangent/value rules.

The initial complete weight vector is resolved by `KHR_character_expression`: `node.weights`, then `mesh.weights`, then `N` zeros. An individual pointer target compares against the corresponding element of that vector.

Listing a channel here does not filter, weight, remap, or otherwise modify the response record produced by the base expression evaluator. Lack of support for this optional classifier does not suppress the underlying channel. Support for `KHR_animation_pointer` remains independently necessary to emit a response for the pointer form. The cross-classifier overlap prohibition is defined by `KHR_character_expression`.

## Partial animation examples (Informative)

The ordinary core form identifies the complete weight vector:

```json
{
  "sampler": 0,
  "target": {
    "node": 0,
    "path": "weights"
  }
}
```

The pointer form may identify one weight:

```json
{
  "sampler": 0,
  "target": {
    "path": "pointer",
    "extensions": {
      "KHR_animation_pointer": {
        "pointer": "/nodes/0/weights/2"
      }
    }
  }
}
```

These are partial channel fragments, not complete glTF assets. Their samplers, accessors, buffers, dependency declarations, and matching authored initial values are omitted.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js loader and VRM conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity demos.

## License

This extension is licensed under the Khronos Group Extension License.
See: https://www.khronos.org/registry/gltf/license.html
