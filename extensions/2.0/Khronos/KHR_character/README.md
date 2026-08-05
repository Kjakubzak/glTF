# KHR_character

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

This extension has no extension dependencies.

## Overview (Informative)

The `KHR_character` extension provides one author-selected character designation for a glTF asset. It provides a common discovery point for independently defined character-related extensions.

This extension does not define rigging, expressions, retargeting, metadata, or runtime behavior. Those capabilities are defined, when present, by their respective extensions.

Identifying additional independently addressable characters is outside the scope of this extension.

## Extension usage

The `KHR_character` object MUST be attached to the top-level glTF object. The asset MUST list `KHR_character` in `extensionsUsed` and MAY list it in `extensionsRequired` according to the rules of the glTF 2.0 specification.

This extension provides exactly one character designation per asset. It neither prohibits unrelated or additional scene content nor identifies additional characters.

The `rootNode` property MUST contain a valid index into the top-level `nodes` array. It designates the root node chosen by the author for the character. It does not assert scene membership, skin ownership, skeleton membership, or that every character-related node is its descendant.

A consumer claiming support for this extension MUST recognize the `KHR_character` object and resolve `rootNode` to the designated node. The extension does not require traversal, rendering, animation, or other character-specific behavior. A consumer that cannot recognize and resolve the designation MUST treat an asset listing `KHR_character` in `extensionsRequired` as unsupported, according to the glTF 2.0 extension rules.

## Extension schema

```json
{
  "asset": {
    "version": "2.0"
  },
  "nodes": [
    {
      "name": "characterRoot"
    }
  ],
  "extensionsUsed": [
    "KHR_character"
  ],
  "extensions": {
    "KHR_character": {
      "rootNode": 0
    }
  }
}
```

### Properties

| Property   | Type    | Description                                                                                                                                                                                  |
| ---------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rootNode` | integer | Index of the glTF `node` designated as the character root. |

## Metadata guidance (Informative)

This section is informative and does not define conformance requirements for `KHR_character`.

Character metadata can be expressed using the `KHR_xmp_json_ld` format, a structured mechanism for attaching JSON-LD metadata blocks to glTF files. In the context of `KHR_character`, this can describe character provenance, licensing, creator, versioning, and intended use, among others.

When used, the `KHR_xmp_json_ld` block is placed according to the rules defined by that extension. This extension does not define a character-specific metadata vocabulary.

The following properties are examples that authoring pipelines may find useful. Their presence is not required by this extension. For these examples, the `khr` prefix uses the proposed registry namespace `https://www.khronos.org/registry/glTF/character/metadata/`. This registry namespace is an example that has not yet been implemented or published; it must be established before these terms can be treated as part of a Khronos-managed vocabulary.

| DC/XMP_JSON_LD Property | Example use                                                                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| dc:title                | Human-readable title for the character.                                                                                                   |
| dc:creator              | Person or organization that created the character.                                                                                        |
| dc:license              | License governing use of the character.                                                                                                   |
| dc:rights               | Copyright or other rights information.                                                                                                    |
| dc:created              | Date on which the asset was created.                                                                                                      |
| dc:publisher            | Entity responsible for making the resource available.                                                                                     |
| dc:description          | Context and a summary of the content.                                                                                                     |
| dc:subject              | Content tags or subject classifications.                                                                                                  |
| dc:source               | Source information for provenance and attribution.                                                                                        |
| khr:version             | Character or asset version as a string.                                                                                                   |
| khr:thumbnailImage      | Zero-based index of an image in the top-level glTF `images` array to use as a character thumbnail.                                        |

### Optional metadata example

```json
{
  "asset": {
    "version": "2.0",
    "extensions": {
      "KHR_xmp_json_ld": {
        "packet": 0
      }
    }
  },
  "scene": 0,
  "scenes": [
    {
      "nodes": [0]
    }
  ],
  "nodes": [
    {
      "name": "characterRoot"
    }
  ],
  "images": [
    {
      "name": "Character Thumbnail",
      "uri": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNk+A8AAQUBAScY42YAAAAASUVORK5CYII="
    }
  ],
  "extensionsUsed": ["KHR_character", "KHR_xmp_json_ld"],
  "extensions": {
    "KHR_character": {
      "rootNode": 0
    },

    "KHR_xmp_json_ld": {
      "packets": [
        {
          "@context": {
            "dc": "http://purl.org/dc/elements/1.1/",
            "khr": "https://www.khronos.org/registry/glTF/character/metadata/"
          },
          "dc:title": "Example Model",
          "dc:creator": {
            "@list": [
              "Author1",
              "AuthorEmail1@email.com",
              "Author2",
              "AuthorEmail2@email.com"
            ]
          },
          "dc:license": {
            "@list": [
              "https://vrm.dev/licenses/1.0/",
              "https://example.com/third-party-license"
            ]
          },
          "dc:created": "2023-05-05",
          "dc:rights": "Copyright information about the model",
          "dc:publisher": "Imaginary Corporation A, LLC",
          "dc:description": "A sentence, or paragraph describing the character at hand",
          "dc:subject": {
            "@list": ["Example trait", "Another example trait"]
          },
          "dc:source": "https://example.com/characters/example-model",
          "khr:version": "1.0",
          "khr:thumbnailImage": 0
        }
      ]
    }
  }
}
```

## Implementation notes (Informative)

- `rootNode` provides a convenient starting point for character-oriented traversal, but consumers cannot infer relationships that are not represented by the node hierarchy or another extension.
- Authors are encouraged to choose a node that is a common ancestor of character-related meshes and joints when the asset hierarchy permits it.
- Consumers can use the marker as a signal to inspect `extensionsUsed` for independently defined character-related extensions.

## Known implementations (Informative)

- [0b5vr/khr-character-testbed](https://github.com/0b5vr/khr-character-testbed) - Three.js viewer and VRM-to-KHR_character converter.
- [Kjakubzak/khr_character_testbed](https://github.com/Kjakubzak/khr_character_testbed) - UnityGLTF importer, exporter, sample assets, and Unity demos.

## License

This extension specification is licensed under the Khronos Group Extension License.
See: https://www.khronos.org/registry/gltf/license.html
