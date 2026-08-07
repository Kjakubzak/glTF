# KHR_node_visibility_hint

## Contributors

- Ken Jakubzak, Meta
- 0b5vr / pixiv Inc.
- Aaron Franke, Independent Contributor
- Hideaki Eguchi / VirtualCast, Inc.
- Leonard Daly, Independent Contributor
- Nick Burkard, Meta

## Status

**Draft** – This extension is not yet ratified by the Khronos Group and is subject to change.

## Conventions

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) when, and only when, they appear in all capitals, as shown here.

## Dependencies

Written against the glTF 2.0 specification.

This extension has no extension dependencies. It composes with [`KHR_node_visibility`](../KHR_node_visibility/README.md) when both extensions are supported, but does not require it.

## Overview (Informative)

Many applications present the same asset in more than one view context. For example, character head geometry may be useful in a third-person view but obstruct a first-person camera. This extension associates a node and, by inheritance, its descendants with a view-context role.

The role defines a pure predicate for each node instance and render view. Evaluating it does not alter glTF-authored or animated node state, select the active context, create or activate a camera, or define transitions between contexts. Implementations may use temporary engine state as described under [Per-view and per-instance evaluation](#per-view-and-per-instance-evaluation).

## Extension usage

The extension object MUST be attached to a glTF node. An asset using this extension MUST list `KHR_node_visibility_hint` in `extensionsUsed`.

An asset MAY list `KHR_node_visibility_hint` in `extensionsRequired` when correct visual presentation depends on the standard behavior defined below. A consumer claiming support for required use MUST evaluate standard roles and inheritance as specified. When the consumer also supports `KHR_node_visibility`, it MUST apply the interaction defined below. A consumer that cannot provide the required hint behavior MUST treat the asset as unsupported according to the glTF 2.0 extension rules.

If the extension appears only in `extensionsUsed`, an implementation that does not support it ignores the hint. Ignoring a hint MUST NOT suppress visual content; it is equivalent to treating the hint role as visible.

### Extension object

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `role` | string | Yes | — | Nonempty view-context role. Standard values are defined below. |
| `label` | string | No | — | Nonempty human-readable label for authoring and inspection. It has no effect on visibility. |

## View-context input

The host may supply an active view-context string independently for each render view. Context selection, naming of custom contexts, update cadence, and transitions are host-defined.

The host may also supply no active view context. When no context is supplied, this extension MUST NOT suppress any node. This rule applies to standard and custom roles.

Selecting or activating a `KHR_node_camera_hint` does not implicitly select a context for this extension. A host may coordinate those features, but that coordination is outside the glTF asset contract.

## Standard role predicate

The standard role strings are exact and case-sensitive. Given a supplied context `C`, the base role predicate is:

| `role` | No context supplied | `C == "first_person"` | Any other supplied `C` |
|--------|---------------------|-------------------------|------------------------|
| `"always"` | visible | visible | visible |
| `"first_person"` | visible | visible | hidden |
| `"third_person"` | visible | hidden | visible |
| unrecognized role | visible | visible | visible |

A node with neither its own hint nor an annotated ancestor is equivalent to `role: "always"`. An explicit `role: "always"` overrides an inherited role. A differently cased token such as `"First_Person"` is not a standard role and therefore uses the unrecognized-role fallback.

The `"third_person"` role means “hidden when the supplied context is exactly `"first_person"`,” not “visible only when the supplied context is exactly `"third_person"`.” This preserves visibility in other supplied contexts unless a custom role defines different behavior.

## Inheritance and composition

For a node `n`, `resolvedHint(n)` is the hint object on `n` when present; otherwise it is the nearest hint object on an ancestor of `n`; otherwise it is an implicit object with `role: "always"`. A hint on a descendant replaces the inherited hint object for that descendant and its subtree.

Let `hintVisible(n, C)` be the standard role predicate for the role of `resolvedHint(n)` and the supplied context. If no context is supplied, `hintVisible(n, C)` is `true`.

When `KHR_node_visibility` is present and supported, let `coreVisible(n)` be the logical AND of the current `KHR_node_visibility.visible` values on `n` and all its ancestors, using that extension's default of `true` where the property or extension is absent. Otherwise, `coreVisible(n)` is `true`. If `KHR_node_visibility` is separately listed in `extensionsRequired` and the consumer does not support it, the asset is unsupported under ordinary glTF extension rules.

Because `coreVisible(n)` is a logical AND over a node and all its ancestors, an invisible node higher in the hierarchy always takes priority: a `visible: false` at any level hides that node and its entire subtree, and no `visible: true` on a descendant can restore it. The following cases summarize this interaction for an ancestor–descendant pair, independent of any hint role:

| Ancestor `visible` | Descendant `visible` | Result |
|--------------------|----------------------|--------|
| `true`  | `false` | Descendant hidden. |
| `false` | `true`  | Ancestor and descendant hidden. |
| `false` | `false` | Ancestor and descendant hidden. |
| `true`  | `true`  | Both visible, subject to the hint predicate. |

This mirrors the normative semantics of `KHR_node_visibility`; it is restated here only to make the composition explicit and does not modify that extension.

The visual-content predicate for `n` is:

```text
renderNodeVisualContent(n, C) = hintVisible(n, C) AND coreVisible(n)
```

Hint inheritance is metadata resolution, not persistent subtree mutation. A hint-hidden ancestor MUST NOT prevent a descendant's nearer hint from making that descendant's visual content visible. In contrast, `KHR_node_visibility.visible: false` on an ancestor hides descendant visual content according to the normative semantics of that extension and cannot be overridden by a descendant hint.

When both extensions are used, the asset MUST list both in `extensionsUsed`. If correct presentation depends on both behaviors, it MUST list both in `extensionsRequired`.

## Per-view and per-instance evaluation

Implementations MUST evaluate the predicate independently for each render view and node instance. Two views rendered during the same frame may supply different contexts and therefore obtain different results without changing asset state.

When multiple nodes reference the same mesh, each containing node uses its own resolved hint role and ancestor `KHR_node_visibility` state. Implementations MUST NOT cache one node's result as visibility state on the shared mesh.

Implementations MAY use material or shader substitution, renderer filtering, draw-list filtering, node or primitive splitting, or any other technique that produces the required result independently for each view and node instance. Implementation state MUST NOT modify glTF-authored or animated data, be written back as authored asset state on export, or allow evaluation for one view or instance to affect another. If an implementation temporarily changes an engine representation such as a material binding or renderer flag, it MUST restore or isolate that state before evaluating another view or instance.

When evaluation makes a node's visual-content predicate false, the node's visual features MUST NOT contribute to visual rendering for that view. Picking, selection, physics, audio, animation evaluation, and arbitrary application queries are outside this extension's visual-presentation contract.

## Custom roles

A custom role is any nonempty role string other than the three standard values. Unrecognized custom roles use the visible fallback in the standard role table.

Custom roles SHOULD be vendor-qualified, for example `"ACME_mirror_only"`, to reduce collisions. Their interpretation is host-defined and is not portable under this extension alone.

If correct presentation depends on custom-role semantics, those semantics MUST be defined by a separate specification, such as a vendor extension or application profile. That specification defines its own association with role tokens, data placement, behavior, and support contract; this extension does not prescribe a companion object or naming relationship. Any glTF extension used for that purpose MUST follow the ordinary `extensionsUsed` and `extensionsRequired` rules. Without separately supported semantics, the visible fallback applies.

## Interaction with primitive visibility

[`KHR_mesh_primitive_visibility_hint`](../KHR_mesh_primitive_visibility_hint/README.md) defines the corresponding predicate at mesh-primitive granularity. For a primitive instance, the containing node predicate and the primitive predicate are combined with logical AND. A primitive hint cannot make visual content visible when its containing node is hidden.

## Examples (Informative)

The following partial fragment demonstrates inheritance and a descendant override. Mesh, accessor, and buffer definitions are omitted.

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": ["KHR_node_visibility_hint"],
  "nodes": [
    {
      "name": "CharacterRoot",
      "mesh": 0,
      "children": [1],
      "extensions": {
        "KHR_node_visibility_hint": { "role": "third_person" }
      }
    },
    {
      "name": "FirstPersonHands",
      "mesh": 1,
      "extensions": {
        "KHR_node_visibility_hint": { "role": "first_person" }
      }
    }
  ]
}
```

With context `"first_person"`, visual content attached directly to `CharacterRoot` is hint-hidden while `FirstPersonHands` is hint-visible because its nearer role overrides the inherited role. With no supplied context, neither node is suppressed by hints.

The following partial fragment demonstrates composition with `KHR_node_visibility`. Mesh, accessor, and buffer definitions are omitted.

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": [
    "KHR_node_visibility_hint",
    "KHR_node_visibility"
  ],
  "nodes": [
    {
      "mesh": 0,
      "extensions": {
        "KHR_node_visibility_hint": { "role": "third_person" },
        "KHR_node_visibility": { "visible": false }
      }
    }
  ]
}
```

The node is hidden in every context because the two predicates are conjoined and the `KHR_node_visibility` predicate is false.

## Validation boundary

The JSON Schema enforces only the extension object's local structure: `role` is required and nonempty, and `label` is nonempty when present.

A validator that claims to validate this extension MUST verify placement on a node and the ordinary `extensionsUsed` and `extensionsRequired` declarations. Custom roles remain structurally valid and use the visible fallback; validation of separately specified custom semantics is outside this extension.

## Known implementations (Informative)

- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and first-/third-person visibility demo.

## Known limitations (Informative)

- The host chooses whether and when to supply a context.
- The standard predicate is binary; fades and transitions are host-defined.
- Custom-role behavior is portable only through a separately supported specification.
- Restructuring a node hierarchy may change inherited roles, so authoring tools should preserve or deliberately relocate annotations when reparenting nodes.

## Schema

- [node.KHR_node_visibility_hint.schema.json](./schema/node.KHR_node_visibility_hint.schema.json)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
