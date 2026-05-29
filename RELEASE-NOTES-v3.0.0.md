# MRPAF v3.0.0 Release Notes

## Summary

`v3.0.0` is a reset of MRPAF around a narrower and more defensible core.

This release defines MRPAF as an editable pixel-art interchange format for AI-to-editor handoff and iterative revision. It does not attempt to be a final delivery format, a general resource container, or a complete runtime standard.

## Main Changes

- Fixed the top-level core object model to `format`, `version`, `metadata`, `canvas`, `palette`, `layers`, `animations`, and `extensions`
- Removed alternative coordinate-system machinery from the core format
- Removed serialized derived-size concepts such as `effectiveSize`
- Removed runtime-heavy animation control from the core format
- Limited core pixel encodings to `array`, `rle`, and `sparse`
- Defined explicit handling for unknown extensions and unknown top-level members
- Tightened reader failure conditions around invalid references and invalid dimensions
- Added an embedded structural JSON Schema
- Added canonical valid examples for static, animation, and unknown-extension cases

## Design Direction

This release favors:

- strict core semantics
- stable round-trip meaning
- machine-friendly JSON generation
- explicit interoperability boundaries

This release does not favor:

- feature breadth for its own sake
- editor-specific timeline semantics
- speculative ecosystem features
- marketing language in place of normative precision

## Release Status

This document accompanies the finalized `v3.0.0` release.

Future work after `v3.0.0` should be handled as a revision or a new version, not by reopening the core meaning of this release.
