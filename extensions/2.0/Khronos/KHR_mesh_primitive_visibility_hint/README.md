# KHR_mesh_primitive_visibility_hint

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

This extension has no extension dependencies. It composes with [`KHR_node_visibility_hint`](../KHR_node_visibility_hint/README.md) and [`KHR_node_visibility`](../KHR_node_visibility/README.md) when the interacting extensions are supported, but does not require them.

## Overview (Informative)

Some assets combine geometry with different view-context roles into separate primitives of one mesh. This extension associates a mesh primitive with a view-context role so that its visual contribution can be evaluated independently for each render view.

The role is a pure per-view predicate. The extension does not mutate a material, renderer, primitive, mesh, or node; select the active context; create or activate a camera; or define transitions between contexts.

## Extension usage

The extension object MUST be attached to an object in a mesh's `primitives` array. An asset using this extension MUST list `KHR_mesh_primitive_visibility_hint` in `extensionsUsed`.

An asset MAY list `KHR_mesh_primitive_visibility_hint` in `extensionsRequired` when correct visual presentation depends on the standard behavior defined below. A consumer claiming support for required use MUST evaluate standard roles and omit hidden primitive instances from visual rendering as specified. A consumer that cannot provide that behavior MUST treat the asset as unsupported according to the glTF 2.0 extension rules.

If the extension appears only in `extensionsUsed`, an implementation that does not support it ignores the hint. Ignoring a hint MUST NOT suppress a primitive; it is equivalent to treating the primitive role as visible.

### Extension object

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `role` | string | Yes | — | Nonempty view-context role. Standard values are defined below. |
| `label` | string | No | — | Nonempty human-readable label for authoring and inspection. It has no effect on visibility. |

## View-context input

The host may supply an active view-context string independently for each render view. Context selection, naming of custom contexts, update cadence, and transitions are host-defined.

The host may also supply no active view context. When no context is supplied, this extension MUST NOT suppress any primitive. This rule applies to standard and custom roles.

Selecting or activating a `KHR_node_camera_hint` does not implicitly select a context for this extension. A host may coordinate those features, but that coordination is outside the glTF asset contract.

## Standard role predicate

The standard role strings are exact and case-sensitive. Given a supplied context `C`, the primitive role predicate is:

| `role` | No context supplied | `C == "first_person"` | Any other supplied `C` |
|--------|---------------------|-------------------------|------------------------|
| `"always"` | visible | visible | visible |
| `"first_person"` | visible | visible | hidden |
| `"third_person"` | visible | hidden | visible |
| unrecognized role | visible | visible | visible |

An unannotated primitive is equivalent to `role: "always"`. A differently cased token such as `"First_Person"` is not a standard role and therefore uses the unrecognized-role fallback.

The `"third_person"` role means “hidden when the supplied context is exactly `"first_person"`,” not “visible only when the supplied context is exactly `"third_person"`.”

## Primitive-instance composition

Let `primitiveHintVisible(p, C)` be the role predicate for primitive `p` and the supplied context. If `p` has no hint, or no context is supplied, this value is `true`.

For an instance of primitive `p` through containing node `n`, let `nodeHintVisible(n, C)` be the resolved predicate from `KHR_node_visibility_hint` when that extension is present and supported; otherwise it is `true`. Let `coreVisible(n)` be the ancestor-inclusive predicate from `KHR_node_visibility` when that extension is present and supported; otherwise it is `true`.

The visual-rendering predicate for the primitive instance is:

```text
renderPrimitiveInstance(n, p, C) =
    primitiveHintVisible(p, C)
    AND nodeHintVisible(n, C)
    AND coreVisible(n)
```

A primitive hint cannot make a primitive visible when its containing node is hidden. Conversely, a visible containing node does not override a hidden primitive predicate.

When interacting extensions are used, the asset MUST list each one in `extensionsUsed`. If correct presentation depends on their combined behavior, it MUST list each extension whose behavior is required in `extensionsRequired`.

## Per-view and per-instance evaluation

The annotation belongs to the mesh primitive and therefore supplies the same primitive role to every node that references the mesh. The complete predicate MUST nevertheless be evaluated independently for each containing node instance because node hints and ancestor `KHR_node_visibility` values may differ.

Implementations MUST also evaluate the predicate independently for each render view. Two views rendered during the same frame may supply different contexts and obtain different primitive sets without changing asset state.

An implementation may use any internal technique that is observably equivalent to the predicate. It MUST NOT observably overwrite authored materials or visibility properties, retain a resolved hint as persistent global primitive or mesh state, or allow one node instance or render view to contaminate another. Any temporary internal mutation MUST be removed before it can affect another instance or view or become externally observable.

When evaluation makes `renderPrimitiveInstance` false, that primitive instance MUST NOT contribute to any visual render pass for that view, including color, depth, or shadow output. Merely replacing its surface color with a transparent material is not sufficient conformance. Picking, selection, physics, animation evaluation, and arbitrary application queries are outside this extension's visual-presentation contract.

## Custom roles

A custom role is any nonempty role string other than the three standard values. Unrecognized custom roles use the visible fallback in the standard role table.

Custom roles SHOULD be vendor-qualified, for example `"ACME_mirror_only"`, to reduce collisions. Their interpretation is host-defined and is not portable under this extension alone.

If correct presentation depends on custom-role semantics, those semantics MUST be defined by a separate specification, such as a vendor extension or application profile. That specification defines its own association with role tokens, data placement, behavior, and support contract; this extension does not prescribe a companion object or naming relationship. Any glTF extension used for that purpose MUST follow the ordinary `extensionsUsed` and `extensionsRequired` rules. Without separately supported semantics, the visible fallback applies.

## Examples (Informative)

The following partial fragment marks a head primitive as hidden in first-person. Accessors and buffers are omitted.

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": ["KHR_mesh_primitive_visibility_hint"],
  "meshes": [
    {
      "name": "AvatarBody",
      "primitives": [
        { "attributes": { "POSITION": 0 }, "material": 0 },
        {
          "attributes": { "POSITION": 1 },
          "material": 1,
          "extensions": {
            "KHR_mesh_primitive_visibility_hint": {
              "role": "third_person",
              "label": "Head"
            }
          }
        }
      ]
    }
  ]
}
```

With context `"first_person"`, the second primitive is omitted from visual rendering. With any other supplied context, or no supplied context, both primitives are hint-visible.

The following partial fragment demonstrates node-plus-primitive composition. Mesh attributes, accessors, and buffers are omitted.

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": [
    "KHR_node_visibility_hint",
    "KHR_mesh_primitive_visibility_hint"
  ],
  "nodes": [
    {
      "mesh": 0,
      "extensions": {
        "KHR_node_visibility_hint": { "role": "first_person" }
      }
    }
  ],
  "meshes": [
    {
      "primitives": [
        {
          "extensions": {
            "KHR_mesh_primitive_visibility_hint": {
              "role": "third_person"
            }
          }
        }
      ]
    }
  ]
}
```

When a context is supplied, the standard node and primitive predicates cannot both be true for this instance, so the primitive is hidden. With no supplied context, neither hint suppresses it.

## Validation boundary

The JSON Schema enforces only the extension object's local structure: `role` is required and nonempty, and `label` is nonempty when present.

A validator that claims to validate this extension MUST verify placement on a mesh primitive and the ordinary `extensionsUsed` and `extensionsRequired` declarations. Custom roles remain structurally valid and use the visible fallback; validation of separately specified custom semantics is outside this extension.

## Known implementations (Informative)

- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and primitive-visibility demo.

## Known limitations (Informative)

- Per-primitive visual suppression may require splitting draw submissions in engines that otherwise batch mesh primitives.
- The host chooses whether and when to supply a context.
- The standard predicate is binary; fades and transitions are host-defined.
- Custom-role behavior is portable only through a separately supported specification.

## Schema

- [mesh.primitive.KHR_mesh_primitive_visibility_hint.schema.json](./schema/mesh.primitive.KHR_mesh_primitive_visibility_hint.schema.json)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
