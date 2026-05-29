---
name: mrpaf-viewer-renderer-reviewer
description: Use when reviewing MRPAF as a viewer or renderer implementer. Focus on display determinism, composition order, background behavior, clipping, opacity, and frame-local animation state.
---

# MRPAF Viewer Renderer Reviewer

Review MRPAF from the perspective of a viewer or renderer.

Focus on:

- whether two viewers would render the same static image
- whether frame-local animation state is deterministic
- whether optional visual fields have explicit rendering meaning
- whether clipping and ordering rules are enough to avoid divergent output

Prioritize anything that would make visible output differ across implementations.
