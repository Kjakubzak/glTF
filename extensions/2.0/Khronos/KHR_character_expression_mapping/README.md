# KHR_character_expression_mapping

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

Assets using `KHR_character_expression_mapping` MUST list `KHR_character_expression_mapping`, `KHR_character_expression`, and `KHR_character` in `extensionsUsed`. They MUST contain top-level objects for all three extensions.

## Overview (Informative)

`KHR_character_expression_mapping` provides two explicitly directed scalar mapping operations between a character's native expression drivers and an externally defined expression vocabulary:

- `expressionSetMappings` maps native expression drivers to endpoint outputs.
- `expressionSetInputMappings` maps endpoint commands to native expression drivers.

These operations are independent. Neither operation is the inverse of the other, and a consumer cannot derive one from the other.

## Extension usage

The extension object MUST be attached to the top-level glTF object. It MUST contain at least one of `expressionSetMappings` and `expressionSetInputMappings`, and MAY contain both.

An asset MAY list `KHR_character_expression_mapping` in `extensionsRequired`. A consumer claiming support for an asset using this extension MUST implement every mapping operation present in that asset. A consumer that cannot implement a present operation MUST treat the asset as unsupported when the extension is listed in `extensionsRequired`.

Mapping-set selection, endpoint integration, driver acquisition, and arbitration between direct-native and mapped input surfaces are host-defined. A consumer MUST NOT claim that a missing mapping operation is defined by this extension merely because it can derive an application-specific transform from the operation that is present.

## Mapping-set identifiers

Each property name directly inside `expressionSetMappings` or `expressionSetInputMappings` is a mapping-set identifier. It MUST be an absolute URI that identifies a specific, version-stable endpoint expression vocabulary. Consumers MUST compare mapping-set identifiers using exact string equality and MUST NOT depend on URI retrieval or redirection while loading the asset. Publishers SHOULD use immutable, content-addressed, or explicitly versioned public URIs.

A mapping-set object MUST contain at least one nonempty endpoint expression key. Keys are exact and case-sensitive. The meaning and completeness requirements of those keys are defined by the external vocabulary, not by this extension.

## Scalar domain and accumulation

All operation input values are finite scalars in the closed interval `[0, 1]`. A missing input value is `0`. Every serialized `weight` MUST also be in `[0, 1]`.

Let:

```text
clamp01(x) = min(1, max(0, x))
```

Each output is computed from an immutable input vector. All matching contributions, including repeated entries, are summed without intermediate clamping. The sum is clamped exactly once after accumulation. Therefore JSON object member order and entry-array order do not affect the mathematical result, duplicate entries contribute independently, and aggregate values may saturate at `1`.

This version does not permit a negative coefficient or a coefficient greater than `1` in one entry. It does permit multiple nonnegative entries to contribute to the same output before the final clamp.

## Native-driver to endpoint-output mapping

`expressionSetMappings` preserves the original mapping direction. For a selected mapping set, each endpoint output key contains an array of source entries:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `source` | integer | Yes | Index of a native expression in `KHR_character_expression.expressions`. |
| `name` | string | No | Diagnostic copy of the referenced expression's label. When present, it MUST match exactly and case-sensitively. |
| `weight` | number | Yes | Contribution coefficient in `[0, 1]`. |

Let `n[i]` be native expression driver `i`, and let `F[q]` be the entry array for endpoint output key `q`. The forward operation is:

```text
endpointOutput[q] = clamp01(
    sum(entry.weight * n[entry.source] for entry in F[q])
)
```

A missing native driver value is `0`. The operation produces the endpoint keys present in the selected mapping set. Selection, transport, and use of those outputs are outside this extension.

## Endpoint-command to native-driver input mapping

`expressionSetInputMappings` defines the separately authored input direction. For a selected mapping set, each endpoint command key contains an array of target entries:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `target` | integer | Yes | Index of the native expression driver that receives the contribution. |
| `name` | string | No | Diagnostic copy of the referenced expression's label. When present, it MUST match exactly and case-sensitively. |
| `weight` | number | Yes | Contribution coefficient in `[0, 1]`. |

Let `c[q]` be the endpoint command for key `q`, and let `R[q]` be that key's target-entry array. For every native expression index `i`, the input operation is:

```text
mappedNative[i] = clamp01(
    sum(
        entry.weight * c[q]
        for each q and each entry in R[q]
        where entry.target == i
    )
)
```

A missing or unrecognized endpoint command is `0`. A native expression with no matching target entry receives `0` from this operation.

The resulting `mappedNative` vector is the output of this mapping operation. Direct-native input is a separate host input surface. This extension does not define selection, arbitration, or addition between direct-native and mapped vectors.

After any host-defined selection or arbitration, the host supplies one final finite `[0, 1]` native-driver vector. When expression masks are present, that final vector—not unconditionally `mappedNative`—is their immutable pre-mask input.

## Independence of the two operations

An asset may define only the operation it needs. When both properties use the same mapping-set identifier, they refer to the same external vocabulary but remain separately authored operations. They are not required to be algebraic inverses, transposes, pseudoinverses, or numerically symmetric.

Consumers MUST NOT interpret or advertise the following as behavior defined by this extension:

- applying `expressionSetMappings` in reverse as though it were `expressionSetInputMappings`;
- deriving `expressionSetInputMappings` from forward weights;
- deriving forward endpoint-output mappings from input mappings; or
- auto-selecting an operation from asset shape or runtime values.

A host MAY perform application-specific transforms outside this extension. Such transforms are not described by the glTF asset, are not portable mapping behavior, and MUST NOT be reported or serialized as one of these operations unless the corresponding operation is authored explicitly.

## Complete example (Informative)

This example defines both independent directions for one versioned endpoint vocabulary. The three native expressions share a valid two-key response animation; each expression entry still has independent driver identity.

```json
{
  "asset": { "version": "2.0" },
  "scene": 0,
  "scenes": [
    { "nodes": [0] }
  ],
  "extensionsUsed": [
    "KHR_character",
    "KHR_character_expression",
    "KHR_character_expression_mapping"
  ],
  "nodes": [
    { "name": "CharacterRoot" }
  ],
  "buffers": [
    {
      "byteLength": 32,
      "uri": "data:application/octet-stream;base64,AAAAAAAAgD8AAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAA="
    }
  ],
  "bufferViews": [
    { "buffer": 0, "byteOffset": 0, "byteLength": 8 },
    { "buffer": 0, "byteOffset": 8, "byteLength": 24 }
  ],
  "accessors": [
    {
      "bufferView": 0,
      "componentType": 5126,
      "count": 2,
      "type": "SCALAR",
      "min": [0],
      "max": [1]
    },
    {
      "bufferView": 1,
      "componentType": 5126,
      "count": 2,
      "type": "VEC3"
    }
  ],
  "animations": [
    {
      "channels": [
        { "sampler": 0, "target": { "node": 0, "path": "translation" } }
      ],
      "samplers": [
        { "input": 0, "output": 1, "interpolation": "LINEAR" }
      ]
    }
  ],
  "extensions": {
    "KHR_character": {
      "rootNode": 0
    },
    "KHR_character_expression": {
      "expressions": [
        { "expression": "smileLeft", "animation": 0 },
        { "expression": "smileRight", "animation": 0 },
        { "expression": "jawOpen", "animation": 0 }
      ]
    },
    "KHR_character_expression_mapping": {
      "expressionSetMappings": {
        "https://example.com/expression-vocabularies/example/v1": {
          "Smile": [
            { "source": 0, "name": "smileLeft", "weight": 0.8 },
            { "source": 1, "name": "smileRight", "weight": 0.8 }
          ],
          "MouthOpen": [
            { "source": 2, "name": "jawOpen", "weight": 1.0 }
          ]
        }
      },
      "expressionSetInputMappings": {
        "https://example.com/expression-vocabularies/example/v1": {
          "Smile": [
            { "target": 0, "name": "smileLeft", "weight": 0.5 },
            { "target": 1, "name": "smileRight", "weight": 0.5 }
          ],
          "MouthOpen": [
            { "target": 2, "name": "jawOpen", "weight": 1.0 }
          ]
        }
      }
    }
  }
}
```

For native drivers `smileLeft=0.5`, `smileRight=0.25`, and `jawOpen=1`, the forward outputs are `Smile=0.6` and `MouthOpen=1`. For endpoint commands `Smile=1` and `MouthOpen=0.25`, the input operation produces `smileLeft=0.5`, `smileRight=0.5`, and `jawOpen=0.25`. The differing weights demonstrate that the two operations are not inferred from each other.

## Validation boundary

The JSON Schema enforces:

- presence of at least one mapping operation;
- nonempty mapping-set objects, endpoint keys, and entry arrays;
- URI-scheme-prefixed mapping-set identifiers, with the JSON Schema `uri` format annotation;
- required, non-negative expression indices; and
- weights in `[0, 1]`.

A validator that claims to validate this extension MUST additionally:

- verify that every mapping-set identifier is a valid absolute URI;
- resolve every `source` and `target` against `KHR_character_expression.expressions`;
- verify every optional `name` against the referenced expression label; and
- verify the required extension declarations and top-level dependency objects.

Validators do not select a mapping set or operation and do not infer one mapping direction from the other. The scalar equations are evaluated by consumers and test oracles.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - TypeScript schema types and conversion tooling.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, and sample assets.

## Schema

- [glTF.KHR_character_expression_mapping.schema.json](./schema/glTF.KHR_character_expression_mapping.schema.json)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
