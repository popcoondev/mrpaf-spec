# Multi-Resolution Pixel Art Format (MRPAF) Specification v3.1.0 Draft

Status: Public review draft

Version: 3.1.0

License: CC0 1.0 Universal recommended for specification text

## Draft Status and Review Focus

This draft is intended for public review before release finalization.

The English draft is the normative reference text for `v3.1.0`. Any translated draft is informative unless stated otherwise.

Reviewers should focus on:

- ambiguity in normative behavior
- schema-to-prose mismatches
- missing failure conditions for readers and writers
- interoperability risks across independent implementations

Reviewers should not assume compatibility with pre-v3 MRPAF drafts unless compatibility is stated explicitly in this document.

## Positioning Statement

**MRPAF is a pixel-art interchange format for preserving editable layers and local structure across AI generation, tool handoff, and iterative revision, without collapsing assets into a flat PNG.**

**MRPAF は、AI生成、ツール間受け渡し、反復修正の過程で、ドット絵をフラットな PNG に潰さず、編集可能なレイヤーと部分構造を保持するための交換形式である。**

MRPAF is not a final delivery image format.

MRPAF is not a complete runtime solution.

MRPAF is an editable, layer-preserving, structure-preserving interchange format for pixel art during AI-to-editor handoff and iterative revision.

## Purpose and Scope

This specification defines a machine-readable interchange format for editable pixel art.

The core format is optimized for:

- preserving editable layers
- preserving local structure that supports partial revision
- stable handoff between AI generators, editors, validators, and viewers
- meaning-preserving round-trip behavior
- machine-friendly canonical JSON generation

The core format is not intended to:

- replace PNG as a final distribution format
- act as a general asset container
- define complete animation runtime behavior
- bundle community, roadmap, or ecosystem documentation into the format definition
- embed heavyweight provenance or rights workflows into the core model

## Why Not PNG?

MRPAF answers `Why not PNG?` with exactly these three points:

1. PNG preserves final pixels, but it discards editable layers and local structure.
2. PNG is suitable as a delivery image, but not as an interchange format for iterative revision.
3. PNG is easy to render, but it is too flat for AI-to-editor handoff, partial repair, and structure-aware tooling.

## Comparison with PNG, Aseprite, and Generic JSON

| Format | Primary role | What it preserves well | What it loses or weakens | Where MRPAF wins |
| --- | --- | --- | --- | --- |
| PNG | Final image delivery | Final raster output | Layers, local edit structure, semantic revision points | MRPAF preserves editable structure before flattening |
| Aseprite | Authoring and project file | Layers, timeline, editor-native workflow | Neutral interchange across tools, AI-friendly canonical form | MRPAF is tool-neutral and easier to validate and generate |
| Generic JSON | Arbitrary data container | Flexibility | Interoperable semantics unless every field is separately agreed | MRPAF fixes a shared pixel-art data model |
| MRPAF | Editable interchange | Layers, local structure, machine-readable revision state | Not intended to be the final display image | MRPAF bridges AI generation and editor revision without flattening |

## Normative Terms

The key words `MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY` in this document are to be interpreted as described in RFC 2119 and RFC 8174.

In this specification:

- `reader` means an implementation that loads MRPAF data
- `writer` means an implementation that emits MRPAF data
- `viewer` means a reader that renders or previews MRPAF data
- `editor` means a reader and writer that allows user-visible modification
- `AI generator` means a writer that emits MRPAF from model output or model-guided editing

## Core Data Model

An MRPAF document is a UTF-8 encoded JSON object with the following top-level members:

- `format`
- `version`
- `metadata`
- `canvas`
- `palette`
- `layers`
- `animations`
- `extensions`

Top-level requirements:

- `format` MUST be the string `"MRPAF"`
- `version` MUST be a semantic version string
- `metadata` MUST be an object
- `canvas` MUST be an object
- `palette` MUST be an array
- `layers` MUST be an array
- `animations` MAY be omitted
- `extensions` MAY be omitted

Unknown top-level members outside `extensions` SHOULD trigger a warning. Readers MUST NOT assign standard meaning to them. Editors SHOULD preserve them during round-trip handling when practical.

### Canonical Core Rules

- Documents MUST be valid JSON. Comments are not allowed.
- Property order is not semantically meaningful.
- Writers SHOULD omit empty optional objects and arrays unless they carry meaning.
- Derived values MUST NOT be serialized when they can be recomputed from required fields.
- Documents SHOULD preserve meaning across round-trip saves even if formatting or key order changes.

## Canvas, Palette, and Layers

### Coordinate Model

The coordinate model is fixed in the core format:

- origin is top-left
- x increases to the right
- y increases downward

The core format does not define alternative axis directions.

Coordinates are expressed in the base canvas coordinate space.

### Canvas

`canvas` defines the base coordinate space.

Required members:

- `baseWidth`: integer, greater than 0
- `baseHeight`: integer, greater than 0

Optional members:

- `backgroundColor`: hex color string (`#RRGGBB` or `#RRGGBBAA`) or null
- `subpixelPrecision`: integer from 0 to 8

Rules:

- `subpixelPrecision` defaults to `0`
- if `subpixelPrecision` is `0`, all placement coordinates MUST be integers
- if `subpixelPrecision` is greater than `0`, placement coordinates MAY be decimal values with at most that many fractional digits
- a reader MUST fail if `subpixelPrecision` is `0` and any placement coordinate is non-integer
- a writer MUST NOT emit fractional placement coordinates when `subpixelPrecision` is `0`
- if `backgroundColor` is present, viewers MUST treat it as a flat fill behind all layers across the full base canvas

The core format does not define `pixelUnit`, `baseUnit`, `unit`, `allowFloatingPoint`, or `allowSubPixel`.

### Palette

`palette` is an ordered array of palette entries.

Each entry MUST contain:

- `id`: unique string
- `hex`: color string in `#RRGGBB` or `#RRGGBBAA`

Each entry MAY contain:

- `name`: string

Rules:

- palette `id` values MUST be unique within the document
- writers SHOULD include a transparent color when transparent pixels are used
- readers MUST treat `#RRGGBB` as fully opaque
- duplicate `hex` values are allowed, because names or semantic use may differ

### Layers

`layers` is an ordered array. Array order defines the default back-to-front composition order.

Each layer MUST contain:

- `id`: unique string
- `name`: string
- `resolution`: object
- `placement`: object
- `pixels`: object

Each layer MAY contain:

- `visible`: boolean
- `opacity`: number from 0 to 1

Rules:

- layer `id` values MUST be unique within the document
- array order MUST NOT be used as a stable reference identifier
- `visible` defaults to `true`
- `opacity` defaults to `1`

#### Layer Resolution

`resolution` defines the internal pixel grid and its scale relative to the base canvas space.

Required members:

- `pixelArraySize.width`: integer, greater than 0
- `pixelArraySize.height`: integer, greater than 0
- `scale`: number, greater than 0

Rules:

- `scale` defines how many internal pixels map to one base-space unit
- `effectiveSize` MUST NOT be serialized
- viewers MUST derive displayed size from `pixelArraySize` and `scale`

Derived values:

- `displayWidth = pixelArraySize.width / scale`
- `displayHeight = pixelArraySize.height / scale`

If a writer cannot represent these values consistently with placement or rendering intent, it MUST fail before writing.

#### Layer Placement

`placement` defines the layer origin in base canvas space.

Required members:

- `x`
- `y`

Rules:

- `x` and `y` refer to the top-left of the rendered layer bounds in base canvas space
- `placement.width` and `placement.height` MUST NOT be serialized
- layers MAY extend outside the canvas bounds
- viewers MUST clip to the visible viewport or canvas as needed

## Pixel Encodings

The core format defines exactly three encodings:

- `array`
- `rle`
- `sparse`

All encodings MUST decode to the same logical result:

- a rectangular grid of palette entry identifiers
- dimensions exactly equal to `resolution.pixelArraySize`

### Common Pixel Object Shape

Every layer `pixels` object MUST contain:

- `encoding`
- `data`

Rules:

- every decoded pixel value MUST refer to a palette `id`
- if a decoded pixel refers to an unknown palette `id`, readers MUST fail
- `defaultValue` is encoding-specific
- `defaultValue` MUST be present for `sparse`
- `defaultValue` MUST NOT be present for `array`
- `defaultValue` MUST NOT be present for `rle`

### `array` Encoding

`array` stores a full row-major list of palette IDs.

Rules:

- `data` MUST be an array
- `data.length` MUST equal `width * height`
- values MUST be ordered row-major from top-left to bottom-right
- `defaultValue` MUST NOT be present

### `rle` Encoding

`rle` stores run-length encoded palette IDs.

`data` MUST be an array of two-element arrays:

```json
[
  ["transparent", 8],
  ["ink", 4]
]
```

Rules:

- the first element of each run is a palette `id`
- the second element of each run is a positive integer length
- the sum of all run lengths MUST equal `width * height`
- zero-length or negative-length runs MUST fail validation
- `defaultValue` MUST NOT be present

### `sparse` Encoding

`sparse` stores only non-default cells.

`data` MUST be an array of entries:

```json
[
  { "x": 1, "y": 2, "value": "ink" }
]
```

Rules:

- `defaultValue` MUST be present for `sparse`
- every sparse entry MUST be within pixel-array bounds
- duplicate coordinates MUST fail validation
- writers MUST NOT emit duplicate coordinates

## Basic Animation Model

`animations` is optional. If present, it MUST be an array of animation objects.

Each animation MUST contain:

- `id`: unique string
- `name`: string
- `loop`: boolean
- `frames`: array

Each frame MUST contain:

- `duration_ms`: integer, greater than 0

Each frame MAY contain:

- `layers`: array of active layer IDs
- `layerOverrides`: object keyed by layer ID

Rules:

- animation `id` values MUST be unique within the document
- `fps` MUST NOT be serialized in the core format
- `tweens`, `timeline`, `events`, `animationController`, and `transitions` are outside the core format

### Layer State in Frames

If `layers` is present, it replaces the default active layer set for that frame.

The order of `frames.layers` defines the back-to-front composition order for that frame.

If `layerOverrides` is present, it MAY override:

- `visible`
- `opacity`
- `placement.x`
- `placement.y`

Writers SHOULD use `layerOverrides` only for frame-local state.
Writers MUST NOT emit override members other than `visible`, `opacity`, and `placement`.
Writers MUST NOT reference unknown layer IDs from `frames.layers` or `layerOverrides`.
Readers MUST fail if `frames.layers` or `layerOverrides` reference an unknown layer ID.
Core readers SHOULD warn on unknown override members.

## Metadata

`metadata` MUST contain:

- `title`: string

`metadata` MAY contain:

- `author`: string
- `license`: string
- `source`: string
- `created`: string in RFC 3339 format
- `modified`: string in RFC 3339 format
- `description`: string
- `tags`: array of strings

The core format intentionally keeps metadata lightweight. Heavy provenance and workflow tracking belong in extensions.

## Extensions

`extensions` is an optional object whose members define namespaced extension payloads.

Rules:

- extension keys SHOULD use stable names such as `ai`, `rights`, `provenance`, or vendor-specific keys
- readers MUST NOT fail solely because an unknown extension exists
- editors SHOULD preserve unknown extension payloads when saving
- writers MUST NOT place standard core semantics only inside `extensions`

### Standard Extension: `extensions.layerGroups`

`extensions.layerGroups` defines named organizational groups of layers.

This extension is intended to preserve editor-facing layer grouping across tool handoff without changing core rendering semantics.

`extensions.layerGroups` SHOULD contain:

- `groups`: array of group objects

Each group object MUST contain:

- `id`: unique string within `extensions.layerGroups.groups`
- `members`: array of declared layer IDs

Each group object MAY contain:

- `name`: string

Rules:

- group membership is organizational metadata and MUST NOT change core composition, visibility, opacity, placement, or animation meaning
- the order of `members` is organizational only and MUST NOT override layer composition order
- nested groups are not defined by this extension
- a layer MAY appear in zero or more groups
- writers supporting this extension MUST NOT emit duplicate group IDs
- writers supporting this extension MUST NOT emit duplicate layer IDs within one group's `members`
- implementations supporting this extension SHOULD warn if a group references an unknown layer ID
- implementations supporting this extension MAY ignore invalid group references without failing core document loading

Recommended extension buckets:

- `extensions.ai`
- `extensions.layerGroups`
- `extensions.rights`
- `extensions.provenance`

## Versioning and Compatibility

MRPAF uses semantic versioning.

Compatibility policy for this line:

- major version changes MAY introduce incompatible changes
- minor version changes MUST preserve forward compatibility for core readers that ignore unknown optional fields and unknown extensions
- patch version changes MUST NOT change core meaning

For the purposes of this specification, a change is forward compatible only if:

- all previously required members remain required with compatible meaning
- existing member meaning does not change
- new standard members are optional for readers
- unknown standard additions can be ignored without corrupting existing decoded image or basic animation meaning
- unknown extensions remain ignorable without changing core interpretation

Reader behavior:

- same major, newer minor: MAY warn, SHOULD continue when unknown additions can be safely ignored without changing decoded image, layer order, or basic animation meaning
- different major: MUST fail unless explicitly configured otherwise
- malformed semantic version string: MUST fail

## Error Handling and Recovery

The core policy is: structure is strict, content is recoverable when safe.

Readers MUST fail on:

- missing required top-level members
- invalid JSON
- invalid types for required fields
- invalid dimensions
- pixel counts that do not match declared dimensions
- references to unknown palette IDs
- animation frame references to unknown layer IDs
- duplicate palette IDs
- duplicate layer IDs or animation IDs

Readers SHOULD warn and MAY recover on:

- unknown optional fields
- unknown extension payloads
- optional metadata fields with malformed but non-critical values

Editors SHOULD preserve recoverable unknown data where possible.
Editors SHOULD preserve unknown top-level members and unknown extensions during round-trip handling unless the user explicitly requests normalization that removes them.

## Conformance Levels

### Reader Conformance

A conforming reader MUST:

- parse valid MRPAF core JSON
- enforce required structure
- decode `array`, `rle`, and `sparse`
- support unknown extensions without failing
- warn on unknown top-level members outside `extensions`
- fail when `subpixelPrecision` is `0` and a placement coordinate is non-integer

### Writer Conformance

A conforming writer MUST:

- emit valid JSON
- emit required top-level members
- avoid derived fields such as `effectiveSize`
- emit only defined core encodings
- omit `defaultValue` from `array` and `rle`
- emit `defaultValue` for `sparse`

### Viewer Conformance

A conforming viewer MUST:

- render static layered documents
- respect layer order, visibility, opacity, and placement
- render `backgroundColor` as a full-canvas fill when present
- when a frame defines `layers`, use that frame-local layer membership and order
- render basic frame animation using `duration_ms`

### Editor Conformance

A conforming editor MUST:

- read and write valid MRPAF
- preserve known semantics on round-trip
- preserve unknown extensions when practical
- preserve unknown top-level members when practical

### AI Generator Conformance

A conforming AI generator MUST:

- emit valid MRPAF JSON
- emit stable string identifiers for referenced layers and animations
- avoid injecting non-core runtime semantics into the core schema

## JSON Schema

The following schema is a minimal structural core schema aligned with this draft.

It is a structural baseline, not a complete semantic validator.

The normative text still defines constraints that are only partially expressible in JSON Schema, including:

- uniqueness of palette, layer, and animation identifiers
- validity of animation references to declared layer identifiers
- decoded pixel-count equality against declared dimensions
- derived-field prohibition
- warning and recovery behavior
- preservation of unknown members during round-trip handling
- placement integer enforcement when `subpixelPrecision` is `0`
- prohibition of semantic meaning changes across minor and patch version updates
- duplicate sparse-coordinate rejection behavior
- uniqueness of `extensions.layerGroups.groups[].id`
- duplicate member detection inside one `extensions.layerGroups` group

Note: reference integrity for layer IDs used in `frames.layers` and `layerOverrides` must be validated by the reader implementation. Standard JSON Schema cannot enforce that those keys or values match the declared layer IDs in `layers`.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://mrpaf.org/schemas/v3.1.0/mrpaf.schema.json",
  "title": "MRPAF v3.1.0 core schema",
  "type": "object",
  "additionalProperties": true,
  "required": ["format", "version", "metadata", "canvas", "palette", "layers"],
  "properties": {
    "format": {
      "const": "MRPAF"
    },
    "version": {
      "type": "string",
      "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$"
    },
    "metadata": {
      "type": "object",
      "additionalProperties": false,
      "required": ["title"],
      "properties": {
        "title": { "type": "string", "minLength": 1 },
        "author": { "type": "string" },
        "license": { "type": "string" },
        "source": { "type": "string" },
        "created": { "type": "string", "format": "date-time" },
        "modified": { "type": "string", "format": "date-time" },
        "description": { "type": "string" },
        "tags": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
    },
    "canvas": {
      "type": "object",
      "additionalProperties": false,
      "required": ["baseWidth", "baseHeight"],
      "properties": {
        "baseWidth": { "type": "integer", "minimum": 1 },
        "baseHeight": { "type": "integer", "minimum": 1 },
        "backgroundColor": {
          "type": ["string", "null"],
          "pattern": "^#([0-9A-Fa-f]{6}|[0-9A-Fa-f]{8})$"
        },
        "subpixelPrecision": { "type": "integer", "minimum": 0, "maximum": 8, "default": 0 }
      }
    },
    "palette": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "hex"],
        "properties": {
          "id": { "type": "string", "minLength": 1 },
          "hex": {
            "type": "string",
            "pattern": "^#([0-9A-Fa-f]{6}|[0-9A-Fa-f]{8})$"
          },
          "name": { "type": "string" }
        }
      }
    },
    "layers": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "name", "resolution", "placement", "pixels"],
        "properties": {
          "id": { "type": "string", "minLength": 1 },
          "name": { "type": "string", "minLength": 1 },
          "visible": { "type": "boolean", "default": true },
          "opacity": { "type": "number", "minimum": 0, "maximum": 1, "default": 1 },
          "resolution": {
            "type": "object",
            "additionalProperties": false,
            "required": ["pixelArraySize", "scale"],
            "properties": {
              "pixelArraySize": {
                "type": "object",
                "additionalProperties": false,
                "required": ["width", "height"],
                "properties": {
                  "width": { "type": "integer", "minimum": 1 },
                  "height": { "type": "integer", "minimum": 1 }
                }
              },
              "scale": { "type": "number", "exclusiveMinimum": 0 }
            }
          },
          "placement": {
            "type": "object",
            "additionalProperties": false,
            "required": ["x", "y"],
            "properties": {
              "x": { "type": "number" },
              "y": { "type": "number" }
            }
          },
          "pixels": {
            "type": "object",
            "oneOf": [
              {
                "additionalProperties": false,
                "required": ["encoding", "data"],
                "properties": {
                  "encoding": { "const": "array" },
                  "data": {
                    "type": "array",
                    "items": { "type": "string" }
                  }
                }
              },
              {
                "additionalProperties": false,
                "required": ["encoding", "data"],
                "properties": {
                  "encoding": { "const": "rle" },
                  "data": {
                    "type": "array",
                    "items": {
                      "type": "array",
                      "minItems": 2,
                      "maxItems": 2,
                      "prefixItems": [
                        { "type": "string" },
                        { "type": "integer", "minimum": 1 }
                      ]
                    }
                  }
                }
              },
              {
                "additionalProperties": false,
                "required": ["encoding", "defaultValue", "data"],
                "properties": {
                  "encoding": { "const": "sparse" },
                  "defaultValue": { "type": "string" },
                  "data": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "additionalProperties": false,
                      "required": ["x", "y", "value"],
                      "properties": {
                        "x": { "type": "integer", "minimum": 0 },
                        "y": { "type": "integer", "minimum": 0 },
                        "value": { "type": "string" }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "animations": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "name", "loop", "frames"],
        "properties": {
          "id": { "type": "string", "minLength": 1 },
          "name": { "type": "string", "minLength": 1 },
          "loop": { "type": "boolean" },
          "frames": {
            "type": "array",
            "items": {
              "type": "object",
              "additionalProperties": false,
              "required": ["duration_ms"],
              "properties": {
                "duration_ms": { "type": "integer", "minimum": 1 },
                "layers": {
                  "type": "array",
                  "items": { "type": "string" }
                },
                "layerOverrides": {
                  "type": "object",
                  "additionalProperties": {
                    "type": "object",
                    "additionalProperties": false,
                    "properties": {
                      "visible": { "type": "boolean" },
                      "opacity": { "type": "number", "minimum": 0, "maximum": 1 },
                      "placement": {
                        "type": "object",
                        "additionalProperties": false,
                        "properties": {
                          "x": { "type": "number" },
                          "y": { "type": "number" }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "extensions": {
      "type": "object",
      "properties": {
        "layerGroups": {
          "type": "object",
          "additionalProperties": true,
          "required": ["groups"],
          "properties": {
            "groups": {
              "type": "array",
              "items": {
                "type": "object",
                "additionalProperties": false,
                "required": ["id", "members"],
                "properties": {
                  "id": { "type": "string", "minLength": 1 },
                  "name": { "type": "string" },
                  "members": {
                    "type": "array",
                    "items": { "type": "string", "minLength": 1 }
                  }
                }
              }
            }
          }
        }
      },
      "additionalProperties": true
    }
  }
}
```

## Canonical Examples

### Minimal Valid Example

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Minimal Sprite"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" }
  ],
  "layers": [
    {
      "id": "base",
      "name": "Base",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "rle",
        "data": [
          ["transparent", 256]
        ]
      }
    }
  ]
}
```

### Normal Static Example

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Two-Layer Portrait",
    "author": "Example Artist",
    "license": "CC-BY-4.0"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16,
    "backgroundColor": "#00000000"
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "skin", "hex": "#F0C0A0" },
    { "id": "ink", "hex": "#202020" }
  ],
  "layers": [
    {
      "id": "body",
      "name": "Body",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "rle",
        "data": [
          ["transparent", 40],
          ["skin", 8],
          ["transparent", 208]
        ]
      }
    },
    {
      "id": "face_detail",
      "name": "Face Detail",
      "resolution": {
        "pixelArraySize": { "width": 8, "height": 8 },
        "scale": 2
      },
      "placement": {
        "x": 4,
        "y": 4
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 1, "y": 2, "value": "ink" },
          { "x": 5, "y": 2, "value": "ink" }
        ]
      }
    }
  ]
}
```

### Normal Basic Animation Example

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Blink Animation"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "ink", "hex": "#202020" }
  ],
  "layers": [
    {
      "id": "face_open",
      "name": "Face Open",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 5, "y": 6, "value": "ink" },
          { "x": 10, "y": 6, "value": "ink" }
        ]
      }
    },
    {
      "id": "face_closed",
      "name": "Face Closed",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 5, "y": 6, "value": "ink" },
          { "x": 6, "y": 6, "value": "ink" },
          { "x": 10, "y": 6, "value": "ink" },
          { "x": 11, "y": 6, "value": "ink" }
        ]
      }
    }
  ],
  "animations": [
    {
      "id": "blink",
      "name": "Blink",
      "loop": true,
      "frames": [
        {
          "duration_ms": 700,
          "layers": ["face_open"]
        },
        {
          "duration_ms": 100,
          "layers": ["face_closed"]
        }
      ]
    }
  ]
}
```

### Standard Layer Group Extension Example

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Grouped Portrait"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "skin", "hex": "#F0C0A0" },
    { "id": "ink", "hex": "#202020" }
  ],
  "layers": [
    {
      "id": "body",
      "name": "Body",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "rle",
        "data": [
          ["transparent", 40],
          ["skin", 8],
          ["transparent", 208]
        ]
      }
    },
    {
      "id": "face_detail",
      "name": "Face Detail",
      "resolution": {
        "pixelArraySize": { "width": 8, "height": 8 },
        "scale": 2
      },
      "placement": {
        "x": 4,
        "y": 4
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 1, "y": 2, "value": "ink" },
          { "x": 5, "y": 2, "value": "ink" }
        ]
      }
    }
  ],
  "extensions": {
    "layerGroups": {
      "groups": [
        {
          "id": "character",
          "name": "Character",
          "members": ["body", "face_detail"]
        }
      ]
    }
  }
}
```

### Compatible Unknown Extension Example

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "AI Handoff Example"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "accent", "hex": "#FF00AA" }
  ],
  "layers": [
    {
      "id": "base",
      "name": "Base",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 7, "y": 7, "value": "accent" }
        ]
      }
    }
  ],
  "extensions": {
    "ai": {
      "model": "example-model",
      "prompt": "single accent pixel in center",
      "seed": 42
    }
  }
}
```

## Draft Change Summary

This draft intentionally extends MRPAF with a standardized organizational layer-group extension while keeping core rendering semantics unchanged.

Compared with `v3.0.0`, this draft:

- keeps the `v3.0.0` core object model and rendering semantics
- adds `extensions.layerGroups` as a standard organizational extension
- keeps layer grouping out of the core rendering model
- preserves forward-compatible handling for implementations that ignore unknown extensions

## Review Checklist

Use this checklist before publishing a new draft:

- Positioning Statement matches the Purpose and Scope section
- `Why Not PNG?` contains exactly three points and no extra marketing claims
- Comparison table does not overclaim against PNG or Aseprite
- Top-level object members match the schema
- No removed fields such as `coordinateSystem`, `pixelUnit`, `fps`, `effectiveSize`, or `resources` appear in normative examples
- `array`, `rle`, and `sparse` are the only core encodings described
- `defaultValue` is required only for `sparse` and forbidden for `array` and `rle`
- Duplicate sparse coordinates are rejected, not recovered
- Animation examples use `duration_ms` and not `fps`
- `layerOverrides` is schema-constrained to `visible`, `opacity`, and `placement`
- Animation frames reference only declared layer IDs
- `backgroundColor` rendering semantics are stated explicitly
- `extensions.layerGroups` is defined as organizational and not rendering-affecting
- Unknown extension behavior is described in both narrative text and examples
- Required metadata is limited to `title`
- Derived size rules are stated once and not contradicted elsewhere
- All JSON examples are valid JSON
- No section claims MRPAF is a final delivery image format or complete runtime solution
