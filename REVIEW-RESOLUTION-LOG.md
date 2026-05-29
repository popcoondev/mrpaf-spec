# MRPAF v3.0.0 Review Resolution Log

この文書は、`v3.0.0` ドラフトに対して出た主要レビューコメントと、その処理結果を記録する。

## Status Keys

- `accepted`: 指摘を受け入れて仕様へ反映した
- `rejected`: 指摘は採用しない
- `deferred`: 妥当だがこの版では扱わない

## Resolved Review Items

### R1. `defaultValue` の必須・任意性の統一

- Status: `accepted`
- Area: Pixel Encodings / JSON Schema / Examples
- Issue:
  - `defaultValue` の扱いが encoding ごとに曖昧だと、writer 実装と validator 実装が分岐する
- Resolution:
  - `defaultValue` は `sparse` で必須
  - `defaultValue` は `array` と `rle` で禁止
  - 本文、Schema、Review Checklist を同期

### R2. `subpixelPrecision` と placement 座標の関係

- Status: `accepted`
- Area: Canvas / Error Handling / Conformance
- Issue:
  - `subpixelPrecision = 0` のときに placement 座標が整数でなければならないことが、実装者に十分明確でない
- Resolution:
  - `subpixelPrecision = 0` では placement 座標は整数必須
  - reader は非整数座標で fail
  - writer は小数座標を出力禁止
  - Schema だけでは完全表現できないため、本文に規範要件として記載

### R3. `layerOverrides` の Schema 制約不足

- Status: `accepted`
- Area: Basic Animation Model / JSON Schema
- Issue:
  - `layerOverrides` が任意オブジェクトとして受理されると、実装間で許容範囲がぶれる
- Resolution:
  - Schema 上で `visible`, `opacity`, `placement` のみを許可
  - 本文でも override 可能項目を同じ範囲に限定

### R4. `layerOverrides` / `frames.layers` の参照整合

- Status: `accepted`
- Area: Basic Animation Model / Error Handling / JSON Schema Notes
- Issue:
  - animation frame 内の layer 参照が未知 ID を許してしまうと silent corruption の原因になる
- Resolution:
  - writer は未知 layer ID を参照禁止
  - reader は未知 layer ID 参照で fail
  - JSON Schema では参照整合を強制できないため、注記を追加

### R5. `backgroundColor` の意味のあいまいさ

- Status: `accepted`
- Area: Canvas
- Issue:
  - `backgroundColor` が palette ID なのか hex 色文字列なのか文言上ぶれる
- Resolution:
  - `backgroundColor` は hex color string (`#RRGGBB` or `#RRGGBBAA`) or `null` と明記

### R6. Schema 上の default 値の補足

- Status: `accepted`
- Area: JSON Schema
- Issue:
  - `visible`, `opacity`, `subpixelPrecision` の既定値が Schema から見えない
- Resolution:
  - Schema に `default` を追加
  - `visible: true`
  - `opacity: 1`
  - `subpixelPrecision: 0`

### R7. `sparse` 重複座標の解釈揺れ

- Status: `accepted`
- Area: Pixel Encodings / Error Handling / Review Checklist
- Issue:
  - 重複 sparse 座標を warning / last-wins にすると、reader ごとにデコード結果が分岐しうる
- Resolution:
  - duplicate sparse coordinates は validation failure に統一
  - Review Checklist も同じ前提へ更新

### R8. `backgroundColor` の描画意味の不足

- Status: `accepted`
- Area: Canvas / Viewer Conformance
- Issue:
  - `backgroundColor` が表示必須なのか、ヒントなのかが不足していた
- Resolution:
  - `backgroundColor` は full base canvas の背面塗りつぶしとして viewer が描画すると明記

### R9. frame-local layer 順の不足

- Status: `accepted`
- Area: Basic Animation Model / Viewer Conformance
- Issue:
  - `frames.layers` の存在時に既定 layer 集合を置き換えるのか、順序がどう扱われるかが不十分だった
- Resolution:
  - `frames.layers` はその frame の active layer 集合を置き換える
  - 配列順がその frame の背面から前面への順になると明記

### R10. `created` / `modified` の Schema 制約不足

- Status: `accepted`
- Area: JSON Schema
- Issue:
  - 本文では RFC 3339 だが schema では任意文字列だった
- Resolution:
  - schema に `format: date-time` を追加

### R11. 正本言語の明記不足

- Status: `accepted`
- Area: Draft Status
- Issue:
  - 翻訳版単体配布時に、どちらが正本か曖昧だった
- Resolution:
  - 英語版を規範的正本、日本語版を参考訳と明記

## Deferred Items

現時点で `deferred` 項目はない。

## Rejected Items

現時点で `rejected` 項目はない。

## Closure Assessment

このログに記載した主要論点については、`accepted` 項目が仕様本文・Schema・サンプルへ反映済みである。

公開レビューをさらに実施する場合は、この文書に追記して管理すること。
