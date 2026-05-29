---
name: mrpaf-json-schema-reviewer
description: Use when reviewing MRPAF from the perspective of a JSON Schema and data-format implementer. Focus on mismatches between prose and schema, unenforced invariants, and places where readers must apply semantic validation beyond standard schema validation.
---

# MRPAF JSON Schema Reviewer

Review MRPAF as a schema-focused implementer.

Focus on:

- prose-to-schema mismatches
- structural constraints that the schema misses
- fields that can safely be enforced in schema but currently are not
- schema notes needed to prevent false bug reports

Prefer concrete fixes that improve validation without changing the core model.
