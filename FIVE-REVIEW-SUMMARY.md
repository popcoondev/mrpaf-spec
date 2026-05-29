# MRPAF v3.0.0 Five-Reviewer Pass

この文書は、5種類の実装者視点で `MRPAF v3.0.0` ドラフトをレビューした要約である。

## Reviewer Set

1. First-read implementer
2. JSON Schema reviewer
3. Editor/save-format reviewer
4. Viewer/renderer reviewer
5. AI pipeline reviewer

## Findings Before Fixes

### 1. First-read implementer

- Finding:
  - `frames.layers` が存在する場合に、既定の layer 集合を置き換えるのか、既存 layer 順に対する部分指定なのかが明確でなかった
- Risk:
  - frame ごとの合成順が viewer 実装ごとに分岐する
- Action:
  - `frames.layers` はその frame の既定 active layer 集合を置き換え、配列順が frame 内の背面から前面への順になると明記

### 2. JSON Schema reviewer

- Finding:
  - `created` / `modified` は本文で RFC 3339 としているが、schema はただの string だった
- Risk:
  - schema 準拠でも日時フォーマット違反が通る
- Action:
  - schema に `format: date-time` を追加

### 3. Editor/save-format reviewer

- Finding:
  - `sparse` の重複座標が warning / last-wins 扱いだと、保存時の再エンコードで意味がぶれうる
- Risk:
  - editor によって読み込み結果や再保存結果が変わる
- Action:
  - duplicate sparse coordinates は validation failure に統一

### 4. Viewer/renderer reviewer

- Finding:
  - `backgroundColor` が viewer にとって描画必須なのか、単なるヒントなのかが十分に明記されていなかった
- Risk:
  - viewer ごとに背景色の有無が変わる
- Action:
  - `backgroundColor` は full base canvas の背面塗りつぶしとして扱うと明記

### 5. AI pipeline reviewer

- Finding:
  - 英語版が正本で日本語版が参考訳であることが仕様書単体では読めなかった
- Risk:
  - 翻訳版単体配布時に、どちらを正本とみなすか曖昧になる
- Action:
  - 両仕様書の冒頭に正本・参考訳の関係を明記

## Post-Fix Assessment

上記5点はすべて仕様へ反映済みである。

レビュー後の残評価:

- First-read implementer: blocking issue なし
- JSON Schema reviewer: blocking issue なし
- Editor/save-format reviewer: blocking issue なし
- Viewer/renderer reviewer: blocking issue なし
- AI pipeline reviewer: blocking issue なし

## Result

5人分の観点で見た限り、`v3.0.0` は公開前レビューを通過できる水準に達していた。

その後、レビュー完了として正式公開へ進んだ。
