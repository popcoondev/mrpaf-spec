# MRPAF v3.0.0 Public Review Notes

Status: Archived after release

## What This Draft Is

MRPAF `v3.0.0` was reviewed publicly as an editable pixel-art interchange core before final release.

The target use case is:

- AI-generated pixel art handoff into editors
- iterative revision without flattening into PNG
- structure-preserving exchange between independent tools

## What This Draft Is Not

- a final delivery image format
- a general asset container
- a runtime-heavy animation system
- a compatibility promise for pre-v3 experimental drafts

## Reviewer Questions

Use these questions when reviewing:

1. Can an independent reader, writer, and viewer implement this without private assumptions?
2. Are there any places where the JSON Schema implies behavior different from the normative text?
3. Are the failure conditions strict enough to prevent silent corruption?
4. Are any core fields still semantically redundant?
5. Do the examples prove the core behaviors the text claims to define?

## Review Boundaries

Feedback is most useful when it focuses on:

- normative ambiguity
- data-model contradictions
- encoding edge cases
- layer-reference integrity
- versioning and forward-compatibility semantics
- round-trip preservation expectations

Feedback is less useful when it asks this draft to become:

- a full editor project file
- a full runtime playback standard
- a generic bundle format
- a rights-management platform

## Current Validation Baseline

- The embedded schema parses successfully.
- The four embedded MRPAF examples validate against the embedded schema.
- The draft explicitly documents semantic rules that standard JSON Schema cannot enforce alone.

## Submission Guidance

When filing feedback, include:

- section name
- quoted sentence or schema location
- why the current text is ambiguous, inconsistent, or risky
- proposed replacement text when possible
