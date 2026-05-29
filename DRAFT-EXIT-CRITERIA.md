# MRPAF v3.0.0 Draft Exit Criteria

この文書は、`Public review draft` を外して正式版へ進めるための判定基準を定義する。

## Release Gate

次の条件をすべて満たした場合に限り、`draft` 表記を外してよい。

1. 規範本文が凍結されている
2. JSON Schema と本文の不一致が解消されている
3. すべての正規サンプルが機械検証を通過している
4. 公開レビューで出た意味的論点が解消されている
5. 互換性ポリシーがこの版から先に対して確定している
6. 英語版と日本語版の同期状態が確認されている

## Detailed Checklist

### 1. Normative Stability

- `MUST` / `MUST NOT` / `SHOULD` / `MAY` の使い分けに未解決論点がない
- コアデータモデルのトップレベル構造が固定されている
- 削除済み概念が規範本文に再混入していない
- `fail / warn / recover` の境界が未決着ではない

### 2. Schema Alignment

- 埋め込み JSON Schema が本文の構造要件と一致している
- Schema で表現できない意味制約が本文に明記されている
- `defaultValue`、`subpixelPrecision`、`layerOverrides` など既知の難所に矛盾がない
- 参照整合のような Schema 非表現項目が注記されている

### 3. Example Validation

- 英語版の全 JSON サンプルが構文上有効である
- 日本語版の全 JSON サンプルが構文上有効である
- 英語版の全 MRPAF サンプルが埋め込み Schema に適合する
- 日本語版の全 MRPAF サンプルが埋め込み Schema に適合する

### 4. Review Closure

- 公開レビューコメントが一覧化されている
- 各コメントが `accepted`, `rejected`, `deferred` のいずれかに分類されている
- `accepted` 項目が仕様へ反映済みである
- `deferred` 項目が正式版の blocking issue ではない
- コア意味を揺らす未解決論点が残っていない

### 5. Compatibility Commitment

- `major`, `minor`, `patch` の意味が固定されている
- `unknown top-level members` の扱いが固定されている
- `unknown extensions` の扱いが固定されている
- round-trip における意味保存方針が固定されている

### 6. Publication Readiness

- 英語版を正本とするか、日本語版と同格にするかが明記されている
- README に公開導線がある
- 公開レビュー用ノートがある
- リリースノート草案がある
- ファイル名、版番号、本文内 version 表記が一致している

## Blockers

次のいずれかが残る場合、`draft` は外してはならない。

- コアデータモデルの変更提案が未決着
- 必須項目の追加・削除が未決着
- エンコーディング意味の変更提案が未決着
- 互換性ポリシーの変更提案が未決着
- サンプルと Schema の不一致
- 英語版と日本語版で意味差がある

## Promotion Rule

次の条件を満たした場合、`MRPAF-v3.0.0` を正式版へ昇格してよい。

- このチェックリストの全項目が満たされている
- 変更が文言整理または注記追加に限られ、コア意味を変えない

もし昇格前にコア意味が変わる場合は、`draft` を維持したまま版を見直すこと。

## Suggested Final Step

正式化直前に次を一度だけ実施する。

1. 英語版と日本語版の最終差分確認
2. JSON Schema とサンプルの再検証
3. Release notes の最終更新
4. `Status: Final` への更新
5. ファイル名から `draft` を外す
