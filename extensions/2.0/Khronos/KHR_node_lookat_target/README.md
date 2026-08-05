# KHR_node_lookat_target

## Contributors

- Ken Jakubzak, Meta

## Status

**Draft** – This extension is not yet ratified by the Khronos Group and is subject to change.

## Conventions

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) when, and only when, they appear in all capitals, as shown here.

## Dependencies

Written against the glTF 2.0 specification.

This extension has no extension dependencies.

## Overview (Informative)

`KHR_node_lookat_target` identifies a glTF node as a passive positional marker. A host can discover the marker and supply its evaluated global-space point to a separately defined camera, constraint, animation, interaction, or application system.

The marker has no side effects. It does not select a consumer, establish a link, orient a node, move eyes or a head, run inverse kinematics (IK), activate or control a camera, follow another object, or define priority, timing, smoothing, constraints, tours, signage, networking, or shared-attention behavior.

## Extension usage

The extension object MUST be attached to a glTF node. An asset using this extension MUST list `KHR_node_lookat_target` in `extensionsUsed`.

An asset MAY list `KHR_node_lookat_target` in `extensionsRequired`. A consumer claiming support for this extension MUST recognize each marker, interpret the evaluated node instance's global translation as its target point, and make the marker and point available to host logic. Support does not require the consumer to select the marker or cause another object to respond to it. A consumer that cannot provide this minimum behavior MUST treat an asset listing the extension in `extensionsRequired` as unsupported according to the glTF 2.0 extension rules.

The annotated node remains an ordinary node. Its mesh, skin, camera, children, animation, and other extensions retain their normal meanings. In particular, this annotation does not hide or replace ordinary content attached to the node.

### Extension object

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `hint` | string | No | — | Nonempty, free-form advisory text suggesting an intended use. |

The extension object MAY be empty. Multiple nodes MAY carry the extension, and multiple markers MAY have the same `hint` value.

## Target-point semantics

For an evaluated instance `i` of an annotated node in a host-provided evaluation state `S`, let `globalTransform(i, S)` be the node instance's global transform after applying ordinary glTF scene-graph rules and the host's selected animation state. The marker's target point is the translation component of that transform:

```text
targetPoint(i, S) = translation(globalTransform(i, S))
```

Equivalently, the target point is the global-space position of the annotated node's local origin. Parent transforms therefore affect the point. The annotated node's own rotation and scale do not define a direction, volume, radius, or orientation for this extension.

The result is evaluated independently for each node instance. The extension does not select a scene, instance, animation, animation-local time, weight, or composition policy; the host supplies the evaluated state `S`. Animation of the annotated node or any ancestor retains ordinary core glTF meaning, and this extension adds no animation-mixing rules.

## Advisory hint

`hint` is free-form advisory text and has no standard vocabulary. Values such as `"eye_target"`, `"camera_target"`, or `"aim_target"` may help a host present or filter markers, but this extension assigns no behavior to those strings.

A consumer MUST NOT reject a marker because it does not recognize the `hint` value. An unrecognized hint and an omitted hint both leave the marker's target-point semantics unchanged. Any portable selection, linkage, or response behavior requires a separate specification.

## Host-defined behavior

The host decides:

- which scene, node instance, animations, local times, weights, and composition policy to use;
- whether to select, filter, or use a discovered marker;
- which system, if any, consumes a selected point;
- how a consumer establishes links or resolves multiple markers;
- orientation, axes, roll, limits, weights, priorities, activation, and constraints;
- animation drivers, gaze, IK, camera, following, and interaction behavior; and
- downstream update cadence, additional interpolation, smoothing, transitions, and networking.

These choices are not conformance behavior for this extension.

## Complete example (Informative)

The child marker below has local translation `[0.0, 1.5, 2.0]`. Its parent contributes `[1.0, 0.0, 0.0]`, so its global target point in the static scene is `[1.0, 1.5, 2.0]`.

```json
{
  "asset": { "version": "2.0" },
  "scene": 0,
  "scenes": [
    { "nodes": [0] }
  ],
  "extensionsUsed": [
    "KHR_node_lookat_target"
  ],
  "nodes": [
    {
      "name": "ObjectRoot",
      "translation": [1.0, 0.0, 0.0],
      "children": [1]
    },
    {
      "name": "FocusPoint",
      "translation": [0.0, 1.5, 2.0],
      "extensions": {
        "KHR_node_lookat_target": {
          "hint": "eye_target"
        }
      }
    }
  ]
}
```

The `eye_target` string does not cause eye movement. It remains advisory input that a host may use when selecting among markers.

## Ordinary node content example (Informative)

The following partial fragment omits the referenced mesh data. It demonstrates that a marker may be attached to a node that also has ordinary visual content.

```json
{
  "extensionsUsed": ["KHR_node_lookat_target"],
  "nodes": [
    {
      "mesh": 0,
      "extensions": {
        "KHR_node_lookat_target": {}
      }
    }
  ]
}
```

## Interaction with other systems (Informative)

Another extension or application may refer to the same node or discover it by convention. The presence of this marker does not itself establish that relationship. In particular, a `KHR_node_camera_hint.targetNode` index has its own meaning and does not require the referenced node to carry `KHR_node_lookat_target`.

## Validation boundary

The JSON Schema enforces only the extension object's local structure: `hint` is a nonempty string when present, and an empty extension object is valid.

A validator that claims to validate this extension MUST verify placement on a node and the ordinary `extensionsUsed` and `extensionsRequired` declarations. It does not select consumers, evaluate host-chosen animation state, or validate gaze, IK, camera, constraint, or application behavior.

## Known implementations (Informative)

- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and marker demo.

## Known limitations (Informative)

- The extension provides no portable link from a marker to a consumer.
- `hint` values have no portable behavior.
- Marker selection, priority, and response behavior are host-defined.

## Schema

- [node.KHR_node_lookat_target.schema.json](./schema/node.KHR_node_lookat_target.schema.json)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
