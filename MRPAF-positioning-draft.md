# MRPAF Positioning Draft

## Positioning Statement

**MRPAF is a pixel-art interchange format for preserving editable layers and local structure across AI generation, tool handoff, and iterative revision, without collapsing assets into a flat PNG.**

**MRPAF は、AI生成、ツール間受け渡し、反復修正の過程で、ドット絵をフラットな PNG に潰さず、編集可能なレイヤーと部分構造を保持するための交換形式である。**

## Core Thesis

MRPAF is not a final delivery image format.

MRPAF is not a complete runtime solution.

MRPAF is an editable, layer-preserving, structure-preserving interchange format for pixel art during AI-to-editor handoff and iterative revision.

## Scope

MRPAF should optimize for the following:

- preserving editable layers
- preserving local structure that supports partial revision
- stable handoff between AI generators, editors, validators, and viewers
- meaning-preserving round-trip behavior
- machine-friendly canonical JSON generation

MRPAF should not optimize for the following in the core spec:

- replacing PNG as a final distribution format
- acting as a general asset container
- defining complete animation runtime behavior
- bundling roadmap, community, or ecosystem marketing into the format definition
- embedding heavyweight provenance or rights workflows in the core model

## Why Not PNG?

MRPAF should answer `Why not PNG?` with exactly these three points:

1. PNG preserves final pixels, but it discards editable layers and local structure.
2. PNG is suitable as a delivery image, but not as an interchange format for iterative revision.
3. PNG is easy to render, but it is too flat for AI-to-editor handoff, partial repair, and structure-aware tooling.

## Comparison

| Format | Primary role | What it preserves well | What it loses or weakens | Where MRPAF wins |
| --- | --- | --- | --- | --- |
| PNG | Final image delivery | final raster output | layers, local edit structure, semantic revision points | MRPAF preserves editable structure before flattening |
| Aseprite | Authoring/project file | layers, timeline, editor-native workflow | neutral interchange across tools, AI-friendly canonical form | MRPAF is tool-neutral and easier to validate and generate |
| Generic JSON | Arbitrary data container | flexibility | interoperable semantics unless every field is separately agreed | MRPAF fixes a shared pixel-art data model |
| MRPAF | Editable interchange | layers, local structure, machine-readable revision state | not intended to be the final display image | MRPAF bridges AI generation and editor revision without flattening |

## Messaging Rules

Prefer these terms:

- `editable`
- `layer-preserving`
- `structure-preserving`
- `interchange`
- `AI-to-editor handoff`
- `iterative revision`

Avoid these terms:

- `future standard`
- `next-generation`
- `revolutionary`
- `everything format`
- `complete runtime solution`

## Editorial Rewrite Rules

When revising the specification, apply these rules consistently:

1. Replace broad visionary claims with narrow interoperability claims.
2. Introduce MRPAF first as an interchange format, not as an ecosystem.
3. Describe PNG as the flattening endpoint, not as the enemy to replace everywhere.
4. Describe Aseprite and similar tools as adjacent authoring formats, not direct one-to-one replacements.
5. Keep AI support focused on handoff, revision, and canonical generation rather than hype.

## Sections To Remove Or Demote

The following sections should be removed from the normative core or moved to non-normative appendices:

- roadmap
- community and support
- official channels
- contributor listings
- tool ecosystem promotion
- runtime-heavy animation control
- generic resource container behavior
- blockchain, 3D, collaboration, and other future-facing expansion topics

## Suggested Replacement Intro

MRPAF defines a machine-readable interchange format for editable pixel art. Its primary purpose is to preserve layers and local revision structure across AI generation, tool handoff, validation, and repeated editing, without flattening the asset into a final delivery image too early.

The format is intended for editing workflows, not final distribution. PNG remains appropriate for final raster export. MRPAF exists for the stage before flattening, where preserving editable structure matters more than preserving only final pixels.

## Suggested Replacement Use-Case Section

The primary MRPAF use case is AI-to-editor handoff for pixel art. An AI system may generate an initial layered asset, a validator may normalize or inspect it, an editor may revise only a subset of layers, and a viewer may render a preview without erasing the original editable structure. MRPAF is designed for this handoff path.

Secondary use cases include neutral interchange between pixel-art tools and structure-aware archival of in-progress assets. These are valid because they benefit from the same core property: preserving editable layers and local structure during revision.

## Suggested Spec Boundary Note

This specification does not attempt to define a complete runtime animation platform, a general-purpose asset bundle, or a final image replacement format. Features outside the interchange core should be defined as extensions or separate documents.
