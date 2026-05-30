# MRPAF v3.1.0 Draft Release Notes

## Summary

`v3.1.0` is a small forward-compatible draft on top of `v3.0.0`.

This draft keeps the `v3.0.0` core intact and adds a standardized `extensions.layerGroups` extension for named layer grouping across tool handoff.

## Main Changes

- Preserves the `v3.0.0` core object model and rendering semantics
- Adds `extensions.layerGroups` as a standard organizational extension
- Defines that layer groups do not alter core composition, visibility, placement, or animation meaning
- Adds structural schema coverage for `extensions.layerGroups`
- Adds a canonical example for the standard layer-group extension

## Design Direction

This draft favors:

- strict core semantics
- stable round-trip meaning
- machine-friendly JSON generation
- explicit interoperability boundaries

This draft does not favor:

- feature breadth for its own sake
- editor-specific timeline semantics
- speculative ecosystem features
- marketing language in place of normative precision

## Review Status

This document accompanies the `v3.1.0` public review draft.

If the layer-group extension remains stable after review, it can be promoted as a minor release without changing the `v3.0.0` core meaning.
