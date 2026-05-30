# MRPAF v3.1.0 Release Notes

## Summary

`v3.1.0` is a small forward-compatible release on top of `v3.0.0`.

This release keeps the `v3.0.0` core intact and adds a standardized `extensions.layerGroups` extension for named layer grouping across tool handoff.

## Main Changes

- Preserves the `v3.0.0` core object model and rendering semantics
- Adds `extensions.layerGroups` as a standard organizational extension
- Defines that layer groups do not alter core composition, visibility, placement, or animation meaning
- Adds structural schema coverage for `extensions.layerGroups`
- Adds a canonical example for the standard layer-group extension

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

This document accompanies the finalized `v3.1.0` release.

Future work after `v3.1.0` should extend grouping behavior in a new draft or version rather than retroactively changing the meaning of this release.
