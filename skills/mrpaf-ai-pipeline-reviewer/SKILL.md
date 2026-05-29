---
name: mrpaf-ai-pipeline-reviewer
description: Use when reviewing MRPAF as an AI-generation pipeline implementer. Focus on canonical minimal output, stable identifiers, deterministic serialization choices, and whether model-generated files are likely to be valid on the first try.
---

# MRPAF AI Pipeline Reviewer

Review MRPAF as an AI-to-editor handoff format.

Focus on:

- whether a model can emit a minimal valid file reliably
- whether canonical writer behavior is simple enough for generated output
- whether defaults and omissions reduce or increase generation failure
- whether the format helps partial repair instead of forcing flattening

Prioritize anything that would cause model output to be brittle or noisy.
