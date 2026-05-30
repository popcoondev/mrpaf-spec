# MRPAF v3.0.0

MRPAF is an editable, layer-preserving, structure-preserving interchange format for pixel art during AI-to-editor handoff and iterative revision.

このディレクトリには、MRPAF `v3.0.0` の正式仕様書と関連文書を置く。

## Files

- [MRPAF-v3.0.0.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/MRPAF-v3.0.0.md)
  English specification. This is the normative text.
- [MRPAF-v3.0.0.ja.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/MRPAF-v3.0.0.ja.md)
  日本語版。英語版の参考訳として使う。
- [PUBLIC-REVIEW-NOTES.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/PUBLIC-REVIEW-NOTES.md)
  レビュー観点、既知の前提、レビュー投稿時の着眼点。
- [RELEASE-NOTES-v3.0.0.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/RELEASE-NOTES-v3.0.0.md)
  `v3.0.0` リリースで何を確定したかの公開向け要約。
- [GITHUB-RELEASE-v3.0.0.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/GITHUB-RELEASE-v3.0.0.md)
  GitHub Release にそのまま使える公開文面。
- [GITHUB-REPO-METADATA.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/GITHUB-REPO-METADATA.md)
  About 欄と Topics の推奨値。
- [REVIEW-RESOLUTION-LOG.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/REVIEW-RESOLUTION-LOG.md)
  受けた主要レビューコメントと、その反映状況。
- [DRAFT-EXIT-CRITERIA.md](/Users/mn/Documents/Codex/2026-05-28/files-mentioned-by-the-user-multi/DRAFT-EXIT-CRITERIA.md)
  `draft` を外して正式版へ進めるための判定基準。

## Scope

MRPAF `v3.0.0` is intentionally narrow.

- It is not a final delivery image format.
- It is not a complete runtime solution.
- It is an interchange core for editable pixel art.
- It prioritizes structure-preserving handoff over feature breadth.

## Review Priorities

- 規範文の曖昧さ
- JSON Schema と本文の不一致
- fail / warn / recover の境界
- 独立実装間で分岐しそうな箇所
- AI generator / editor / viewer の相互運用性

## Validation Status

英語版・日本語版ともに埋め込み JSON Schema に対して主要サンプル 4 件の検証を通過している。

## Notes

- 英語版を正本とし、日本語版は参考訳として扱う。
- `v3.0.0` より前の MRPAF 草案との互換性は前提にしない。
- 公開前に仕様の意味を変える変更を入れる場合は、英語版と日本語版の両方を同期すること。
