# KHR_character_reference_pose

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

Requires the extension(s): `KHR_character`

Assets using `KHR_character_reference_pose` MUST list `KHR_character_reference_pose` and `KHR_character` in `extensionsUsed`. They MUST contain a top-level `KHR_character` extension object.

## Overview (Informative)

The `KHR_character_reference_pose` extension identifies a standard glTF animation whose ordinary node translation, rotation, and scale channels store one static local-space pose. The optional `poseType` label records the author's classification of that pose.

The extension does not define a retargeting algorithm, joint vocabulary, anatomical requirements, skin bind pose, playback policy, or animation-mixing behavior. Applications may use the stored pose as input to systems that define those behaviors separately.

## Extension usage

The extension object MUST be attached to an animation object. The tagged animation MUST satisfy the validity requirements below.

An asset MAY list `KHR_character_reference_pose` in `extensionsRequired`. A consumer claiming support for this extension MUST discover every tagged animation and interpret its static local node TRS values as specified below. A consumer that cannot do so MUST treat an asset listing the extension in `extensionsRequired` as unsupported, according to the glTF 2.0 extension rules.

### Extension object

The following partial fragment shows the extension on a tagged animation. Accessors and buffers are omitted.

```json
{
  "animations": [
    {
      "name": "TPose",
      "extensions": {
        "KHR_character_reference_pose": {
          "poseType": "TPose"
        }
      },
      "channels": [
        { "sampler": 0, "target": { "node": 0, "path": "rotation" } },
        { "sampler": 1, "target": { "node": 0, "path": "translation" } },
        { "sampler": 2, "target": { "node": 1, "path": "rotation" } },
        { "sampler": 3, "target": { "node": 1, "path": "translation" } }
      ],
      "samplers": [
        { "input": 0, "output": 1, "interpolation": "STEP" },
        { "input": 0, "output": 2, "interpolation": "STEP" },
        { "input": 0, "output": 3, "interpolation": "STEP" },
        { "input": 0, "output": 4, "interpolation": "STEP" }
      ]
    }
  ]
}
```

### Properties

| Property   | Type   | Required | Default  | Description |
|------------|--------|----------|----------|-------------|
| `poseType` | string | No       | `TPose`  | Nonempty classification label for this reference pose. Omission resolves to `TPose`. |

### Pose labels (Informative)

The labels below are common authoring conventions. They are author assertions, not machine-verifiable anatomy. Stored local TRS values are authoritative when a label and the data appear inconsistent.

| Value    | Description |
|----------|-------------|
| `TPose`  | Commonly describes a pose with arms extended horizontally. |
| `APose`  | Commonly describes a pose with arms angled downward. |
| `IPose`  | Commonly describes a pose with arms resting beside the body. |
| *custom* | Any other nonempty author-defined label, such as `"RelaxedPose"`. |

Omitting `poseType` is equivalent to specifying `"TPose"`. Multiple tagged animations may use the same label; animation array indices remain authoritative.

## Reference-pose validity

A tagged animation MUST conform to the glTF 2.0 animation requirements and to all of the following additional requirements:

1. The animation MUST contain at least one channel.
2. Every channel MUST target an ordinary node `translation`, `rotation`, or `scale` property. The channel target MUST contain `node`; `weights`, `KHR_animation_pointer`, and other target domains MUST NOT be used.
3. The input accessor of every sampler referenced by a channel MUST have `count` equal to `1`.
4. The effective interpolation of every referenced sampler MUST be `LINEAR` or `STEP`. An omitted `interpolation` property has the core default `LINEAR`. `CUBICSPLINE` and any other interpolation mode MUST NOT be used.
5. Each referenced output accessor MUST have the type, component type, element count, and value validity required by core glTF for its channel target and interpolation mode.

The single timestamp is not required to be `0`. `STEP` interpolation is not required. With one keyframe, core endpoint behavior yields the same static sample at every requested time for `LINEAR` and `STEP` interpolation.

For each node targeted by one or more channels, its resolved reference-pose value is a local TRS transform. A targeted component uses the referenced sampler's single output value. An untargeted component uses the node's authored static value, or the corresponding core glTF default when that value is omitted.

The extension does not require a skin, require targeted nodes to be skin joints, require complete node or joint coverage, require a unique pose label, or validate the anatomical meaning of a label. Samplers not referenced by a channel do not contribute to the reference pose.

Tagging an animation does not change its ordinary core glTF playback behavior. This extension only assigns reference-pose meaning to the static local TRS values stored by the animation.

### Validation boundary

The JSON Schema enforces only the extension object's local structure: `poseType`, when present, is a nonempty string. Core glTF schemas enforce the ordinary animation object's local structure.

A glTF Validator or authoring tool that claims to validate this extension MUST perform the cross-object checks that JSON Schema cannot express:

- verify extension placement on an animation;
- resolve each channel, sampler, node, and accessor index;
- verify that every channel target is ordinary node TRS;
- verify that every channel-referenced input accessor has exactly one element;
- verify that every channel-referenced sampler's effective interpolation is `LINEAR` or `STEP`; and
- apply the core output accessor cardinality and format checks.

## Relationship to skinning and retargeting (Informative)

The reference pose is independent of skinning state. It does not modify `skin.inverseBindMatrices`, determine a bind pose, or participate in skin deformation calculations. A tagged animation may target nodes that are not skin joints, and an asset may use this extension without a skin.

A retargeter may consume the stored pose, but this extension does not define source/target correspondence, transform conversion, scale handling, constraints, or a solver. Therefore it does not by itself guarantee lossless, universal, or interoperable retargeting.

## Example

Complete example showing a T-Pose reference for a simple three-node hierarchy:

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": ["KHR_character", "KHR_character_reference_pose"],
  "scene": 0,
  "scenes": [{ "nodes": [0] }],
  "nodes": [
    { "name": "Hips", "children": [1], "translation": [0, 1, 0] },
    { "name": "Spine", "children": [2], "translation": [0, 0.3, 0] },
    { "name": "Head", "translation": [0, 0.4, 0] }
  ],
  "buffers": [
    {
      "byteLength": 88,
      "uri": "data:application/octet-stream;base64,AAAAAAAAAAAAAAAAAAAAAAAAgD8AAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAAAAAAAAACAPwAAAAAAAIA/AAAAAAAAAACamZk+AAAAAAAAAADNzMw+AAAAAA=="
    }
  ],
  "bufferViews": [
    { "buffer": 0, "byteOffset": 0, "byteLength": 4 },
    { "buffer": 0, "byteOffset": 4, "byteLength": 16 },
    { "buffer": 0, "byteOffset": 20, "byteLength": 16 },
    { "buffer": 0, "byteOffset": 36, "byteLength": 16 },
    { "buffer": 0, "byteOffset": 52, "byteLength": 12 },
    { "buffer": 0, "byteOffset": 64, "byteLength": 12 },
    { "buffer": 0, "byteOffset": 76, "byteLength": 12 }
  ],
  "accessors": [
    {
      "bufferView": 0,
      "componentType": 5126,
      "count": 1,
      "type": "SCALAR",
      "min": [0],
      "max": [0]
    },
    {
      "bufferView": 1,
      "componentType": 5126,
      "count": 1,
      "type": "VEC4"
    },
    {
      "bufferView": 2,
      "componentType": 5126,
      "count": 1,
      "type": "VEC4"
    },
    {
      "bufferView": 3,
      "componentType": 5126,
      "count": 1,
      "type": "VEC4"
    },
    {
      "bufferView": 4,
      "componentType": 5126,
      "count": 1,
      "type": "VEC3"
    },
    {
      "bufferView": 5,
      "componentType": 5126,
      "count": 1,
      "type": "VEC3"
    },
    {
      "bufferView": 6,
      "componentType": 5126,
      "count": 1,
      "type": "VEC3"
    }
  ],
  "animations": [
    {
      "name": "TPose",
      "extensions": {
        "KHR_character_reference_pose": {
          "poseType": "TPose"
        }
      },
      "channels": [
        { "sampler": 0, "target": { "node": 0, "path": "rotation" } },
        { "sampler": 1, "target": { "node": 1, "path": "rotation" } },
        { "sampler": 2, "target": { "node": 2, "path": "rotation" } },
        { "sampler": 3, "target": { "node": 0, "path": "translation" } },
        { "sampler": 4, "target": { "node": 1, "path": "translation" } },
        { "sampler": 5, "target": { "node": 2, "path": "translation" } }
      ],
      "samplers": [
        { "input": 0, "output": 1, "interpolation": "STEP" },
        { "input": 0, "output": 2, "interpolation": "STEP" },
        { "input": 0, "output": 3, "interpolation": "STEP" },
        { "input": 0, "output": 4, "interpolation": "STEP" },
        { "input": 0, "output": 5, "interpolation": "STEP" },
        { "input": 0, "output": 6, "interpolation": "STEP" }
      ]
    }
  ],
  "extensions": {
    "KHR_character": {
      "rootNode": 0
    }
  }
}
```

## Implementation notes (Informative)

- Tagged animations are discovered by animation array index. `poseType` is a label, not an identifier.
- The complete example uses `STEP` and a timestamp of `0` as convenient authoring choices, not conformance requirements.
- Authoring tools may preview a tagged animation through ordinary core animation playback, but this extension does not prescribe how that preview is mixed with other animation sources.
- Conversion from any previous bind-pose or matrix representation is format-specific and may involve non-TRS transforms or other information that this extension cannot represent. No general lossless conversion is defined here.

## Known implementations (Informative)

- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity reference-pose demo.

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
