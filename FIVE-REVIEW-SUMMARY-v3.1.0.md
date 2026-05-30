# MRPAF v3.1.0 Draft Five-Reviewer Pass

この文書は、`v3.1.0 draft` を 5 本のレビュースキル視点で点検した結果をまとめる。

## Reviewer Set

1. First-read implementer
2. JSON Schema reviewer
3. Editor/save-format reviewer
4. Viewer/renderer reviewer
5. AI pipeline reviewer

## Findings

### 1. `extensions.layerGroups.groups` の順序意味が不足していた

- Reviewer:
  - first-read implementer
  - viewer/renderer reviewer
- Risk:
  - group 配列順を階層や描画順と誤解する実装が出る
- Resolution:
  - `groups` の順序は組織化用途のみであり、描画順や階層を定義しないと明記

### 2. extension 無視の適合範囲をより明示した方がよかった

- Reviewer:
  - first-read implementer
  - AI pipeline reviewer
- Risk:
  - `layerGroups` を知らない reader/viewer が非適合だと誤解される
- Resolution:
  - core-conforming reader/viewer は extension payload を無視してよいと明記

## Post-Fix Assessment

- No blocking findings
- No schema/example mismatches detected
- `layerGroups` は組織化用途に限定され、コア描画意味を汚染していない

## Validation

- English draft: embedded schema + 5 MRPAF examples validated
- Japanese draft: embedded schema + 5 MRPAF examples validated

## Conclusion

`v3.1.0 draft` is ready for focused review of the `extensions.layerGroups` addition.
