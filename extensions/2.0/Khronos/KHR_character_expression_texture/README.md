# KHR_character_expression_texture

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

Requires the extensions: `KHR_character`, `KHR_character_expression`, `KHR_animation_pointer`, and `KHR_texture_transform`.

Assets using `KHR_character_expression_texture` **MUST** list all five extensions in `extensionsUsed`. They **MUST** contain the top-level `KHR_character` and `KHR_character_expression` extension objects. A `KHR_character_expression_texture` object **MUST** be attached to an expression entry in that top-level `KHR_character_expression` object.

Every selected channel uses `KHR_animation_pointer` to target a property of an authored `KHR_texture_transform` object as described below.

An asset MAY list `KHR_character_expression_texture` in `extensionsRequired`. A consumer claiming support for this extension MUST recognize and validate its texture-transform channel classifications. Emitting response records for those channels also requires support for the independently declared `KHR_animation_pointer` and `KHR_texture_transform` targets.

## Overview (Informative)

This extension classifies selected channels of an expression animation as texture-transform channels. It adds validation metadata only. It does not bind an expression to a texture or texture index, select channels for expression evaluation, change their samples, or define a property-composition policy.

The concrete target is supplied entirely by the selected channel's `KHR_animation_pointer` pointer. The base `KHR_character_expression` extension evaluates every channel in the selected animation whether or not the channel is listed here.

## Extension schema (Normative)

The following is a partial extension fragment. Dependency declarations, character objects, materials, texture-transform objects, animations, accessors, and buffers are omitted.

```json
{
  "extensions": {
    "KHR_character_expression": {
      "expressions": [
        {
          "expression": "happy",
          "animation": 0,
          "extensions": {
            "KHR_character_expression_texture": {
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

Each selected channel **MUST** have:

- `target.path` equal to `"pointer"`;
- no `target.node`; and
- a valid `KHR_animation_pointer` target whose pointer resolves to the `offset`, `scale`, or `rotation` property of a `KHR_texture_transform` extension object actually authored in the asset.

It is not sufficient for the pointed-to property to have the same shape or name elsewhere. The owning object **MUST** be an actual `KHR_texture_transform` object attached at a location allowed by that extension.

The selected channel and its sampler **MUST** satisfy all core animation, `KHR_animation_pointer`, `KHR_texture_transform`, and `KHR_character_expression` rules. In particular:

| Target property | AOM value | Output accessor type |
| --- | --- | --- |
| `offset` | two floating-point components | `VEC2` |
| `scale` | two floating-point components | `VEC2` |
| `rotation` | one floating-point component in radians | `SCALAR` |

The output contains one logical value per input key for `LINEAR` or `STEP`, and three logical records per input key for `CUBICSPLINE`. Its component type and values **MUST** satisfy `KHR_animation_pointer` and the target property's constraints.

The authored initial values are the explicitly authored property values when present. Within an authored `KHR_texture_transform` object, omitted `offset`, `scale`, and `rotation` resolve to `[0, 0]`, `[1, 1]`, and `0`, respectively, as defined by `KHR_texture_transform`.

Listing a channel here does not filter, bind, or otherwise modify the response record produced by the base expression evaluator. Lack of support for this optional classifier does not suppress the underlying channel; support for its independently declared target extensions remains necessary to emit that channel's response. The cross-classifier overlap prohibition is defined by `KHR_character_expression`.

## Partial animation-channel example (Informative)

```json
{
  "sampler": 0,
  "target": {
    "path": "pointer",
    "extensions": {
      "KHR_animation_pointer": {
        "pointer": "/materials/2/pbrMetallicRoughness/baseColorTexture/extensions/KHR_texture_transform/scale"
      }
    }
  }
}
```

This is a partial channel fragment, not a complete glTF asset. In a valid asset, the pointed-to `KHR_texture_transform` object is authored at that location and the omitted sampler, accessors, buffers, dependency declarations, and initial-value match satisfy their respective specifications.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js loader and VRM conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity demos.

## License

This extension is licensed under the Khronos Group Extension License.
See: https://www.khronos.org/registry/gltf/license.html
