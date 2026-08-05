# KHR_character_expression_joint

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

Assets using `KHR_character_expression_joint` **MUST** list `KHR_character_expression_joint`, `KHR_character_expression`, and `KHR_character` in `extensionsUsed`. They **MUST** contain the top-level `KHR_character` and `KHR_character_expression` extension objects. A `KHR_character_expression_joint` object **MUST** be attached to an expression entry in that top-level `KHR_character_expression` object.

An asset MAY list `KHR_character_expression_joint` in `extensionsRequired`. A consumer claiming support for this extension MUST recognize and validate its node-TRS channel classifications. This support does not change or filter the base response evaluator.

## Overview (Informative)

This extension classifies selected channels of an expression animation as node-TRS channels. It adds validation metadata only. It does not select channels for expression evaluation, change their samples, require the targeted nodes to be skin joints, or define a property-composition policy.

The base `KHR_character_expression` extension evaluates every channel in the selected animation whether or not the channel is listed here.

## Extension schema (Normative)

The following is a partial extension fragment. Dependency declarations, character objects, accessors, buffers, and the referenced animation are omitted.

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
              "channels": [0, 1, 2]
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

Each selected channel **MUST** use an ordinary core animation target with:

- a defined, valid `target.node`; and
- `target.path` equal to `"translation"`, `"rotation"`, or `"scale"`.

An extension-defined target, including an equivalent-looking `KHR_animation_pointer` target, does not satisfy this classifier. The targeted node **MUST NOT** define `matrix`, because core glTF prohibits animating TRS properties of a node that supplies its transform with `matrix`.

The target node is not required to occur in any `skin.joints` array and is not required to influence a skinned mesh. The word “joint” in this extension's name is a classification label, not an additional skin-membership constraint.

The selected channel and its sampler **MUST** satisfy all core animation rules and all `KHR_character_expression` rules. In particular:

- translation and scale outputs use the core `VEC3` representation;
- rotation outputs use the core quaternion `VEC4` representation and quaternion validity rules; and
- each output contains one logical value per input key for `LINEAR` or `STEP`, or three logical records per input key for `CUBICSPLINE`.

Listing a channel here does not filter, weight, retarget, or otherwise modify the response record produced by the base expression evaluator. Lack of support for this optional classifier does not suppress the underlying core channel. The cross-classifier overlap prohibition is defined by `KHR_character_expression`.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js loader and VRM conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity demos.

## License

This extension is licensed under the Khronos Group Extension License.
See: https://www.khronos.org/registry/gltf/license.html
