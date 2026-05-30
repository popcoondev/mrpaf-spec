# MRPAF v3.0.0

MRPAF v3.0.0 is the first finalized public specification release of the Multi-Resolution Pixel Art Format.

MRPAF is an editable, layer-preserving, structure-preserving interchange format for pixel art during AI-to-editor handoff and iterative revision. It is not a final delivery image format, and it is not a complete runtime solution.

## Highlights

- Defines a fixed core object model: `format`, `version`, `metadata`, `canvas`, `palette`, `layers`, `animations`, `extensions`
- Narrows the format to an editable interchange core instead of a general asset container
- Removes derived-size serialization and runtime-heavy animation control from the core
- Limits core pixel encodings to `array`, `rle`, and `sparse`
- Defines explicit behavior for unknown extensions, unknown top-level members, and round-trip preservation
- Includes embedded JSON Schema and canonical examples

## Included in this release

- English specification: `MRPAF-v3.0.0.md`
- Japanese reference translation: `MRPAF-v3.0.0.ja.md`
- Release notes: `RELEASE-NOTES-v3.0.0.md`
- Review notes and resolution log
- Reviewer skill profiles and issue templates for future maintenance

## Positioning

MRPAF is designed for:

- AI-generated pixel art handoff into editors
- iterative revision without flattening into PNG
- interoperable exchange of editable layered pixel-art data

MRPAF is not designed to replace:

- PNG as a final delivery image
- editor-native project files
- full runtime playback specifications

## Validation

- The English and Japanese specifications both include embedded schema and examples
- The primary MRPAF examples validate against the embedded schema

## Notes

- The English text is the normative specification for `v3.0.0`
- The Japanese text is an informative reference translation
- Compatibility with pre-v3 experimental drafts is not assumed
