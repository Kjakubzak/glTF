# KHR_character_skeleton_mapping

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

Assets using `KHR_character_skeleton_mapping` MUST list `KHR_character_skeleton_mapping` and `KHR_character` in `extensionsUsed`. They MUST contain top-level `KHR_character_skeleton_mapping` and `KHR_character` extension objects.

## Overview (Informative)

The `KHR_character_skeleton_mapping` extension associates roles from one or more externally defined skeletal vocabularies with nodes in a glTF asset. A mapping can help tools discover which node an author associates with a role such as `hips`, `head`, or `leftHand`.

The extension stores associations only. It does not define a skeleton topology, require a skin, identify skin joints, define inverse kinematics (IK), or provide an animation-retargeting algorithm.

## Extension usage

The `KHR_character_skeleton_mapping` object MUST be attached to the top-level glTF object. An asset MAY list the extension in `extensionsRequired`.

A consumer claiming support for this extension MUST recognize every mapping set, preserve the association between each vocabulary role and its referenced node while processing the asset, and resolve each `node` index. Support for this extension does not imply that the consumer understands a particular external vocabulary or can perform retargeting or IK. A consumer that cannot recognize and resolve the associations MUST treat an asset listing `KHR_character_skeleton_mapping` in `extensionsRequired` as unsupported, according to the glTF 2.0 extension rules.

Requiring this extension guarantees only the generic association contract defined here. It does not require support for any particular vocabulary identified by a mapping-set URI.

## Mapping-set and role semantics

The `skeletalRigMappings` property is a nonempty object. Each property name in that object is a mapping-set identifier, and its value is a nonempty role-association object.

Each mapping-set identifier MUST be an absolute URI that identifies a specific, version-stable skeletal vocabulary. Consumers MUST compare mapping-set identifiers using exact string equality and MUST NOT depend on URI retrieval or redirection while loading the asset. Publishers SHOULD use immutable, content-addressed, or explicitly versioned public URIs.

Each property name in a role-association object MUST be a nonempty role identifier defined by the mapping set's vocabulary. Its value associates that role with one glTF node:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `node` | integer | Yes | Index of the associated node in the top-level `nodes` array. |
| `name` | string | No | Diagnostic copy of the referenced node's `name`. When present, it MUST match that node's `name` exactly and case-sensitively. |

The `node` index is authoritative. Consumers MUST NOT resolve an association by `name`. Multiple vocabulary roles MAY refer to the same node, and corresponding roles in different mapping sets MAY refer to the same node.

Associated nodes are not required to appear in a `skin.joints` array. This extension does not assert skin ownership, node hierarchy, role completeness, anatomical correctness, transform compatibility, or a one-to-one correspondence between roles and distinct nodes. Requirements defined by an external vocabulary remain outside this extension.

## Complete example

The mapping-set identifier below is a public, commit-pinned URI for the [VRM 1.0 Humanoid vocabulary](https://github.com/vrm-c/vrm-specification/blob/70ae16e93abd6da727fdf641b67aa41010c6d933/specification/VRMC_vrm-1.0/humanoid.md). The example contains no `skin`; the associations remain valid generic role-to-node metadata.

```json
{
  "asset": { "version": "2.0" },
  "scene": 0,
  "scenes": [
    { "nodes": [0] }
  ],
  "extensionsUsed": [
    "KHR_character",
    "KHR_character_skeleton_mapping"
  ],
  "nodes": [
    { "name": "CharacterRoot", "children": [1] },
    { "name": "Hips", "children": [2, 3, 4, 5, 6] },
    { "name": "Head" },
    { "name": "LeftFoot" },
    { "name": "RightFoot" },
    { "name": "LeftHand" },
    { "name": "RightHand" }
  ],
  "extensions": {
    "KHR_character": {
      "rootNode": 0
    },
    "KHR_character_skeleton_mapping": {
      "skeletalRigMappings": {
        "https://github.com/vrm-c/vrm-specification/blob/70ae16e93abd6da727fdf641b67aa41010c6d933/specification/VRMC_vrm-1.0/humanoid.md": {
          "hips": { "node": 1, "name": "Hips" },
          "head": { "node": 2, "name": "Head" },
          "leftFoot": { "node": 3, "name": "LeftFoot" },
          "rightFoot": { "node": 4, "name": "RightFoot" },
          "leftHand": { "node": 5, "name": "LeftHand" },
          "rightHand": { "node": 6, "name": "RightHand" }
        }
      }
    }
  }
}
```

## Validation boundary

The JSON Schemas enforce nonempty mapping sets, URI-scheme-prefixed mapping-set identifiers, nonempty role identifiers, association shape, and non-negative `node` indices. They also annotate mapping-set identifiers with the JSON Schema `uri` format. A validator that claims to validate this extension MUST additionally verify that every mapping-set identifier is a valid absolute URI, resolve every `node` index against the top-level `nodes` array, and verify each optional `name` against the referenced node.

Validators are not required to retrieve a mapping-set URI, understand external role semantics, verify role completeness, identify skin joints, or run a retargeter.

## Implementation notes (Informative)

- Mapping-set and role strings are identifiers, not display labels.
- A consumer can expose associations for inspection without implementing profile-specific animation behavior.
- Profile-specific adapters may interpret recognized roles, but their retargeting, transform, scale, and IK behavior is outside this extension.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js loader, retargeting helpers, and VRM conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and rig-switching demo.

## License

This extension specification is licensed under the Khronos Group Extension License.
See: https://www.khronos.org/registry/gltf/license.html
