# KHR_node_camera_hint

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

`KHR_node_camera_hint` lets an author describe candidate camera positions, orientations, projection data, and points of interest with annotated glTF nodes. A host may use those descriptors when presenting an asset.

Camera hints are passive and advisory. This extension does not create or activate a camera, select a hint, define a camera controller, frame geometry, follow an animated node, or prescribe adaptation, roll, transitions, smoothing, or interaction.

## Extension usage

The extension object MUST be attached to a glTF node. An asset using this extension MUST list `KHR_node_camera_hint` in `extensionsUsed`.

An asset MUST NOT list `KHR_node_camera_hint` in `extensionsRequired`. Camera hints have no mandatory presentation behavior: consumers may use, adapt, or ignore any descriptor. Ignoring a camera hint does not suppress or otherwise alter ordinary glTF content.

The annotated node remains an ordinary node. Its mesh, skin, camera, children, animation, and other extensions retain their normal meanings. In particular, this annotation does not hide geometry attached to the node and does not instantiate the camera referenced by this extension's `camera` property.

### Extension object

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `role` | string | Yes | — | Nonempty semantic role for the camera hint. Standard values are defined below. |
| `camera` | integer | No | — | Advisory index of a camera in the top-level `cameras` array. |
| `targetNode` | integer | No | — | Index of a different node whose global translation supplies an advisory forward direction. |
| `label` | string | No | — | Nonempty human-readable label for authoring and inspection. It has no camera behavior. |

## Authored descriptor

The annotated node's global transform supplies an authored camera pose using the core glTF camera convention: the local negative Z axis is forward, the local positive Y axis is up, and scaling in the global transform is ignored when deriving the pose. The translation component of the global transform supplies the authored position.

This interpretation is equivalent to deriving a core camera view matrix from the annotated node's global transform with scaling ignored. It does not modify the node transform or require the node to instantiate a core glTF camera.

When `camera` is present, it MUST be a valid index into the top-level `cameras` array. The referenced camera definition is advisory projection data associated with the hint. Referencing it does not instantiate a core camera, override `node.camera`, or change the support contracts of extensions on that camera. A host may adapt or ignore the referenced projection data.

When `targetNode` is present, it MUST be a valid index into the top-level `nodes` array and MUST NOT refer to the node containing the camera hint. Let `P` be the global translation of the hint node and `Q` the global translation of `targetNode`:

```text
targetForward = normalize(Q - P)
```

When `P` and `Q` differ, `targetForward` replaces only the descriptor's authored forward direction. It does not define a complete orientation, up vector, or roll. The authored rotation remains available as advisory orientation data, and the host chooses how, or whether, to construct a camera orientation from these inputs.

When `P` and `Q` coincide, no target direction exists. In that case the target override is ignored and the descriptor uses the authored rotation from the hint node's global transform.

Animation of either node retains ordinary core glTF meaning. This extension does not select an evaluation time or require a camera to follow an animated hint or target.

## Standard roles

The standard role strings are exact and case-sensitive. They classify authored intent without selecting a descriptor or requiring a particular camera behavior.

| `role` | Standard meaning |
|--------|------------------|
| `"default_view"` | Candidate authored camera pose for an initial or primary view. |
| `"detail"` | Candidate authored camera pose associated with a detail view. |
| `"first_person"` | Candidate authored camera pose associated with a first-person view. |
| `"framing_target"` | Authored position is a point of interest. It does not define bounds, margins, aspect ratio, distance, angle, or a framing algorithm. |
| `"full_body"` | Candidate authored camera pose classified by the author as a full-body view. |
| `"orbit_center"` | Authored position is a candidate orbit point. It does not define an orbit controller, radius, angle, or limits. |
| `"portrait"` | Candidate authored camera pose classified by the author as a portrait view. |
| `"third_person"` | Candidate authored camera pose associated with a third-person view. It does not define following or tracking. |

Multiple hints MAY use the same role. Array order, node traversal order, and role do not define priority or selection.

Selecting a `"first_person"` or `"third_person"` hint does not implicitly select a context for `KHR_node_visibility_hint` or `KHR_mesh_primitive_visibility_hint`. A host may coordinate those features, but that coordination is outside the glTF asset contract.

## Custom roles

A custom role is any nonempty string other than the standard values. Custom roles SHOULD be vendor-qualified, for example `"ACME_inspection_view"`, to reduce collisions.

Unrecognized custom roles remain valid. Their descriptor data retains the generic meaning defined above, but this extension does not define additional behavior for them.

## Host-defined behavior

The host decides:

- whether to use a hint and which hint to select;
- when to resolve animated global transforms;
- whether or how to adapt the authored position, forward direction, rotation, or projection data;
- how to choose an up vector and roll when `targetNode` supplies a forward direction;
- camera creation, activation, following, tracking, constraints, and controllers;
- orbit, fitting, framing, margins, aspect-ratio handling, transitions, smoothing, and user interaction; and
- coordination with visibility contexts or other presentation systems.

These choices are not conformance behavior for this extension.

## Complete example (Informative)

This example provides a candidate default view aimed at a separate point of interest. The referenced core camera supplies advisory projection data.

```json
{
  "asset": { "version": "2.0" },
  "scene": 0,
  "scenes": [
    { "nodes": [0, 1, 2] }
  ],
  "extensionsUsed": [
    "KHR_node_camera_hint"
  ],
  "cameras": [
    {
      "type": "perspective",
      "perspective": {
        "yfov": 0.785398,
        "znear": 0.01,
        "zfar": 100.0
      }
    }
  ],
  "nodes": [
    {
      "name": "DefaultView",
      "translation": [0.0, 1.5, 3.0],
      "extensions": {
        "KHR_node_camera_hint": {
          "role": "default_view",
          "camera": 0,
          "targetNode": 1,
          "label": "Authored default view"
        }
      }
    },
    {
      "name": "PointOfInterest",
      "translation": [0.0, 1.0, 0.0],
      "extensions": {
        "KHR_node_camera_hint": {
          "role": "framing_target"
        }
      }
    },
    {
      "name": "OrbitPoint",
      "translation": [0.0, 0.75, 0.0],
      "extensions": {
        "KHR_node_camera_hint": {
          "role": "orbit_center"
        }
      }
    }
  ]
}
```

The default-view descriptor supplies an authored position and a forward direction toward `PointOfInterest`. It does not supply a complete target-derived orientation because roll remains host-defined. The framing and orbit roles identify points only; they do not perform framing or create an orbit controller.

## Ordinary node content example (Informative)

This partial fragment omits the referenced mesh data. It demonstrates that a camera hint does not suppress ordinary content attached to its node.

```json
{
  "extensionsUsed": ["KHR_node_camera_hint"],
  "nodes": [
    {
      "mesh": 0,
      "extensions": {
        "KHR_node_camera_hint": {
          "role": "detail"
        }
      }
    }
  ]
}
```

## Validation boundary

The JSON Schema enforces only the extension object's local structure: `role` is required and nonempty, `camera` and `targetNode` are non-negative indices, and `label` is nonempty when present.

A validator that claims to validate this extension MUST additionally:

- verify placement on a node;
- verify the `extensionsUsed` declaration and reject `KHR_node_camera_hint` in `extensionsRequired`;
- resolve `camera` against the top-level `cameras` array when present;
- resolve `targetNode` against the top-level `nodes` array when present; and
- reject a `targetNode` that refers to the node containing the extension.

Coincident hint and target positions are valid and use the authored-rotation fallback. Validators do not select hints, compute framing, or validate host camera behavior.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js loader, helper, and VRM conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity camera-hint demo.

## Known limitations (Informative)

- The extension does not define hint priority or selection.
- A point alone cannot define framing or a complete look-at orientation.
- Custom roles have no portable behavior beyond the generic descriptor.
- Advisory hints cannot guarantee presentation across hosts.

## Schema

- [node.KHR_node_camera_hint.schema.json](./schema/node.KHR_node_camera_hint.schema.json)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
