# Multi-Resolution Pixel Art Format (MRPAF) 仕様書 v3.1.0 ドラフト

Status: パブリックレビュー用ドラフト

Version: 3.1.0

License: 仕様テキストには CC0 1.0 Universal を推奨

## ドラフトの状態とレビュー観点

このドラフトは、リリース確定前のパブリックレビューを目的とする。

英語版ドラフトを `v3.1.0` の規範的な正本とする。翻訳版は、別途明記しない限り参考訳である。

レビュアーは次の点を重点的に確認すること。

- 規範挙動のあいまいさ
- Schema と本文の不一致
- reader / writer に必要な fail 条件の不足
- 独立実装間の相互運用リスク

本書に明記されていない限り、レビュアーは v3 より前の MRPAF ドラフトとの互換性を前提にしてはならない。

## 位置づけ

**MRPAF is a pixel-art interchange format for preserving editable layers and local structure across AI generation, tool handoff, and iterative revision, without collapsing assets into a flat PNG.**

**MRPAF は、AI生成、ツール間受け渡し、反復修正の過程で、ドット絵をフラットな PNG に潰さず、編集可能なレイヤーと部分構造を保持するための交換形式である。**

MRPAF は最終配布用の画像形式ではない。

MRPAF は完全なランタイム仕様ではない。

MRPAF は、AI-to-editor handoff と iterative revision のための、editable / layer-preserving / structure-preserving なピクセルアート交換形式である。

## 目的と適用範囲

本仕様は、編集可能なピクセルアートのための機械可読な交換形式を定義する。

コア形式は次の目的に最適化されている。

- 編集可能なレイヤーを保持すること
- 部分修正を支える局所構造を保持すること
- AI generator、editor、validator、viewer 間で安定して受け渡せること
- 意味を保った round-trip を実現すること
- 機械が生成しやすい canonical JSON を提供すること

コア形式は次の目的を持たない。

- PNG を最終配布形式として置き換えること
- 汎用アセットコンテナとして振る舞うこと
- 完全なアニメーションランタイム挙動を定義すること
- コミュニティ、ロードマップ、エコシステム文書を形式定義に含めること
- 重い provenance / rights ワークフローをコアモデルに埋め込むこと

## なぜ PNG ではないのか

MRPAF の `Why not PNG?` に対する答えは、次の3点に限定する。

1. PNG は最終ピクセルを保持するが、編集可能なレイヤーと局所構造は失う。
2. PNG は配布画像としては適切だが、反復修正のための交換形式としては適切ではない。
3. PNG は描画しやすいが、AI-to-editor handoff、部分修復、構造認識ツールにはフラットすぎる。

## PNG / Aseprite / Generic JSON との比較

| 形式 | 主な役割 | 保持が得意なもの | 失うもの / 弱いもの | MRPAF の優位点 |
| --- | --- | --- | --- | --- |
| PNG | 最終画像配布 | 最終的なラスタ出力 | レイヤー、局所編集構造、意味的な修正ポイント | MRPAF は flatten 前の編集可能構造を保持する |
| Aseprite | 制作用プロジェクトファイル | レイヤー、タイムライン、エディタ固有ワークフロー | ツール中立な交換、AI 生成しやすい canonical form | MRPAF はツール中立で、検証と生成がしやすい |
| Generic JSON | 任意データコンテナ | 柔軟性 | 各フィールドごとに個別合意しない限り相互運用意味論が定まらない | MRPAF は共有のピクセルアートデータモデルを固定する |
| MRPAF | 編集可能な交換形式 | レイヤー、局所構造、機械可読な修正状態 | 最終表示画像を目的としない | MRPAF は flatten せずに AI 生成とエディタ修正を橋渡しする |

## 規範用語

本書における `MUST`、`MUST NOT`、`SHOULD`、`SHOULD NOT`、`MAY` は RFC 2119 および RFC 8174 に従って解釈する。

本仕様では次の語を用いる。

- `reader`: MRPAF データを読み込む実装
- `writer`: MRPAF データを出力する実装
- `viewer`: MRPAF データを描画またはプレビューする reader
- `editor`: ユーザー可視の変更を行える reader 兼 writer
- `AI generator`: モデル出力またはモデル支援編集から MRPAF を生成する writer

## コアデータモデル

MRPAF 文書は UTF-8 エンコードされた JSON オブジェクトであり、トップレベルに次のメンバーを持つ。

- `format`
- `version`
- `metadata`
- `canvas`
- `palette`
- `layers`
- `animations`
- `extensions`

トップレベル要件:

- `format` は文字列 `"MRPAF"` でなければならない
- `version` は semantic version 文字列でなければならない
- `metadata` はオブジェクトでなければならない
- `canvas` はオブジェクトでなければならない
- `palette` は配列でなければならない
- `layers` は配列でなければならない
- `animations` は省略してよい
- `extensions` は省略してよい

`extensions` の外側にある未知のトップレベルメンバーは警告対象とするべきである。reader はそれらに標準意味を与えてはならない。editor は可能な限り round-trip 時に保持するべきである。

### コアの正規ルール

- 文書は妥当な JSON でなければならない。コメントは許可しない。
- プロパティ順序に意味はない。
- writer は、意味を持たない空の任意オブジェクトや空配列を省略するべきである。
- 必須項目から再計算できる導出値をシリアライズしてはならない。
- 文書は、整形やキー順序が変わっても round-trip 後に意味を保持するべきである。

## Canvas・Palette・Layers

### 座標モデル

コア形式の座標モデルは固定である。

- 原点は左上
- x は右方向に増加
- y は下方向に増加

コア形式は代替の軸方向を定義しない。

座標は base canvas 座標空間で表現する。

### Canvas

`canvas` は基準座標空間を定義する。

必須メンバー:

- `baseWidth`: 0 より大きい整数
- `baseHeight`: 0 より大きい整数

任意メンバー:

- `backgroundColor`: hex color string (`#RRGGBB` または `#RRGGBBAA`) または `null`
- `subpixelPrecision`: 0 から 8 の整数

ルール:

- `subpixelPrecision` のデフォルト値は `0`
- `subpixelPrecision` が `0` のとき、すべての配置座標は整数でなければならない
- `subpixelPrecision` が `0` より大きいとき、配置座標はその桁数以下の小数を取ってよい
- `subpixelPrecision` が `0` なのに配置座標が非整数である場合、reader は fail しなければならない
- `subpixelPrecision` が `0` のとき、writer は小数の配置座標を出力してはならない
- `backgroundColor` が存在する場合、viewer は base canvas 全域の背面塗りつぶしとして扱わなければならない

コア形式は `pixelUnit`、`baseUnit`、`unit`、`allowFloatingPoint`、`allowSubPixel` を定義しない。

### Palette

`palette` は順序付きのパレットエントリ配列である。

各エントリは次を含まなければならない。

- `id`: 一意な文字列
- `hex`: `#RRGGBB` または `#RRGGBBAA` 形式の色文字列

各エントリは次を含んでもよい。

- `name`: 文字列

ルール:

- 文書内で palette `id` は一意でなければならない
- 透明ピクセルを使う場合、writer は透明色を含めるべきである
- reader は `#RRGGBB` を完全不透明として扱わなければならない
- `hex` の重複は許可する。名前や意味的用途が異なる場合があるためである

### Layers

`layers` は順序付き配列である。配列順は既定の背面から前面への合成順を定義する。

各 layer は次を含まなければならない。

- `id`: 一意な文字列
- `name`: 文字列
- `resolution`: オブジェクト
- `placement`: オブジェクト
- `pixels`: オブジェクト

各 layer は次を含んでもよい。

- `visible`: 真偽値
- `opacity`: 0 から 1 の数値

ルール:

- 文書内で layer `id` は一意でなければならない
- 配列順を安定参照識別子として使ってはならない
- `visible` のデフォルト値は `true`
- `opacity` のデフォルト値は `1`

#### Layer の解像度

`resolution` は内部ピクセルグリッドと、その base canvas 空間に対する倍率を定義する。

必須メンバー:

- `pixelArraySize.width`: 0 より大きい整数
- `pixelArraySize.height`: 0 より大きい整数
- `scale`: 0 より大きい数値

ルール:

- `scale` は、内部ピクセルが base-space unit にどう対応するかを定義する
- `effectiveSize` をシリアライズしてはならない
- viewer は `pixelArraySize` と `scale` から表示サイズを導出しなければならない

導出値:

- `displayWidth = pixelArraySize.width / scale`
- `displayHeight = pixelArraySize.height / scale`

writer がこれらの値を配置や描画意図と整合する形で表現できない場合、書き込み前に fail しなければならない。

#### Layer の配置

`placement` は base canvas 空間における layer 原点を定義する。

必須メンバー:

- `x`
- `y`

ルール:

- `x` と `y` は、描画される layer 境界の左上を base canvas 空間で指す
- `placement.width` と `placement.height` をシリアライズしてはならない
- layer は canvas 境界の外へはみ出してよい
- viewer は必要に応じて可視 viewport または canvas にクリップしなければならない

## ピクセルエンコーディング

コア形式は次の3種類のエンコーディングだけを定義する。

- `array`
- `rle`
- `sparse`

すべてのエンコーディングは、同一の論理結果へデコードされなければならない。

- palette entry identifier の長方形グリッド
- `resolution.pixelArraySize` と完全一致する寸法

### 共通の `pixels` オブジェクト形状

すべての layer `pixels` オブジェクトは次を含まなければならない。

- `encoding`
- `data`

ルール:

- デコード後のすべてのピクセル値は palette `id` を参照しなければならない
- デコード後のピクセルが未知の palette `id` を参照している場合、reader は fail しなければならない
- `defaultValue` は encoding ごとの規則に従う
- `defaultValue` は `sparse` では必須
- `defaultValue` は `array` では禁止
- `defaultValue` は `rle` では禁止

### `array` エンコーディング

`array` は row-major 順の palette ID 全列挙を保持する。

ルール:

- `data` は配列でなければならない
- `data.length` は `width * height` と等しくなければならない
- 値は左上から右下への row-major 順で並ばなければならない
- `defaultValue` を含めてはならない

### `rle` エンコーディング

`rle` は run-length encoded な palette ID を保持する。

`data` は2要素配列の配列でなければならない。

```json
[
  ["transparent", 8],
  ["ink", 4]
]
```

ルール:

- 各 run の1要素目は palette `id` でなければならない
- 各 run の2要素目は正の整数長でなければならない
- すべての run 長の合計は `width * height` と等しくなければならない
- 長さ 0 または負の長さの run は検証失敗としなければならない
- `defaultValue` を含めてはならない

### `sparse` エンコーディング

`sparse` は非デフォルトセルのみを保持する。

`data` はエントリの配列でなければならない。

```json
[
  { "x": 1, "y": 2, "value": "ink" }
]
```

ルール:

- `defaultValue` は `sparse` で必須
- すべての sparse entry は pixel-array の範囲内でなければならない
- 重複座標は検証失敗としなければならない
- writer は重複座標を出力してはならない

## 基本アニメーションモデル

`animations` は任意である。存在する場合、animation object の配列でなければならない。

各 animation は次を含まなければならない。

- `id`: 一意な文字列
- `name`: 文字列
- `loop`: 真偽値
- `frames`: 配列

各 frame は次を含まなければならない。

- `duration_ms`: 0 より大きい整数

各 frame は次を含んでもよい。

- `layers`: その frame で有効な layer ID の配列
- `layerOverrides`: layer ID をキーとするオブジェクト

ルール:

- 文書内で animation `id` は一意でなければならない
- `fps` をコア形式にシリアライズしてはならない
- `tweens`、`timeline`、`events`、`animationController`、`transitions` はコア形式の外側にある

### Frame における Layer 状態

`layers` が存在する場合、それはその frame における既定の有効 layer 集合を置き換える。

`frames.layers` の配列順は、その frame における背面から前面への合成順を定義する。

`layerOverrides` が存在する場合、次を上書きしてよい。

- `visible`
- `opacity`
- `placement.x`
- `placement.y`

writer は `layerOverrides` を frame ローカル状態にのみ使うべきである。
writer は `visible`、`opacity`、`placement` 以外の override member を出力してはならない。
writer は `frames.layers` または `layerOverrides` から未知の layer ID を参照してはならない。
`frames.layers` または `layerOverrides` が未知の layer ID を参照している場合、reader は fail しなければならない。
コア reader は未知の override member を警告するべきである。

## メタデータ

`metadata` は次を含まなければならない。

- `title`: 文字列

`metadata` は次を含んでもよい。

- `author`: 文字列
- `license`: 文字列
- `source`: 文字列
- `created`: RFC 3339 形式の文字列
- `modified`: RFC 3339 形式の文字列
- `description`: 文字列
- `tags`: 文字列配列

コア形式の metadata は意図的に軽量に保つ。重い provenance や workflow tracking は extensions に属する。

## 拡張

`extensions` は任意オブジェクトであり、そのメンバーは namespaced extension payload を定義する。

ルール:

- extension key は `ai`、`rights`、`provenance` のような安定名、または vendor-specific key を用いるべきである
- reader は未知の extension が存在することだけを理由に fail してはならない
- editor は保存時に未知の extension payload を保持するべきである
- writer は標準コア意味論を `extensions` の中だけに置いてはならない
- core 適合 reader および viewer は、将来の conformance profile が明示的に要求しない限り、既知・未知を問わず extension payload を無視してよい

### 標準拡張: `extensions.layerGroups`

`extensions.layerGroups` は、名前付きの layer 組を定義する組織化用拡張である。

この拡張は、ツール間受け渡しで editor 向けの layer grouping を保持するためのものであり、コア描画意味を変えない。

`extensions.layerGroups` は次を含むべきである。

- `groups`: group object の配列

各 group object は次を含まなければならない。

- `id`: `extensions.layerGroups.groups` 内で一意な文字列
- `members`: 宣言済み layer ID の配列

各 group object は次を含んでもよい。

- `name`: 文字列

ルール:

- group membership は組織化メタデータであり、コアの合成順、visibility、opacity、placement、animation meaning を変えてはならない
- `groups` の順序は組織化用途のみであり、描画順や階層を定義してはならない
- `members` の順序は組織化用途のみであり、layer composition order を上書きしてはならない
- nested groups はこの拡張では定義しない
- layer は 0 個以上の group に属してよい
- この拡張をサポートする writer は重複 group ID を出力してはならない
- この拡張をサポートする writer は、1つの group の `members` 内に重複 layer ID を出力してはならない
- この拡張をサポートする実装は、未知の layer ID を参照する group を警告するべきである
- この拡張をサポートする実装は、無効な group 参照があってもコア文書の読み込み自体は fail せず無視してよい

推奨される extension bucket:

- `extensions.ai`
- `extensions.layerGroups`
- `extensions.rights`
- `extensions.provenance`

## バージョニングと互換性

MRPAF は semantic versioning を採用する。

この系統における互換性ポリシー:

- major version の変更は非互換変更を導入してよい
- minor version の変更は、未知の任意フィールドと未知の extension を無視できる core reader に対する forward compatibility を保たなければならない
- patch version の変更はコア意味を変えてはならない

本仕様において、ある変更が forward compatible であるのは次の条件を満たす場合に限る。

- 以前に必須だったメンバーが、互換な意味を持つ必須メンバーのままであること
- 既存メンバーの意味が変わらないこと
- 新しい標準メンバーが reader にとって任意であること
- 未知の標準追加を無視しても、既存のデコード結果や基本アニメーション意味を壊さないこと
- 未知の extension を無視してもコア解釈が変わらないこと

reader の挙動:

- same major かつ newer minor: 未知の追加を無視しても decoded image、layer order、basic animation meaning が変わらないなら、警告してよく、継続するべきである
- different major: 明示的な設定がない限り fail しなければならない
- semantic version 文字列が不正な場合: fail しなければならない

## エラー処理と回復

コア方針は次の通りである。構造は厳格に扱い、安全な場合に限り内容回復を許す。

reader は次の場合に fail しなければならない。

- 必須トップレベルメンバーの欠落
- 不正な JSON
- 必須フィールドの型不正
- 不正な寸法
- 宣言寸法と一致しないピクセル数
- 未知の palette ID 参照
- 未知の layer ID を参照する animation frame
- 重複する palette ID
- 重複する layer ID または animation ID

reader は次の場合に警告し、回復してよい。

- 未知の任意フィールド
- 未知の extension payload
- 重要ではないが不正な任意 metadata 値

editor は可能な限り回復可能な未知データを保持するべきである。
editor は、ユーザーが明示的に正規化による削除を要求しない限り、未知のトップレベルメンバーと未知の extension を round-trip 中に保持するべきである。

## 適合レベル

### Reader 適合

適合する reader は次を満たさなければならない。

- 妥当な MRPAF core JSON を解析する
- 必須構造を強制する
- `array`、`rle`、`sparse` をデコードする
- 未知の extension があっても fail しない
- `extensions` の外側にある未知のトップレベルメンバーを警告する
- `subpixelPrecision` が `0` で placement coordinate が非整数なら fail する

### Writer 適合

適合する writer は次を満たさなければならない。

- 妥当な JSON を出力する
- 必須トップレベルメンバーを出力する
- `effectiveSize` のような導出フィールドを避ける
- 定義済みの core encoding のみを出力する
- `array` と `rle` では `defaultValue` を省く
- `sparse` では `defaultValue` を出力する

### Viewer 適合

適合する viewer は次を満たさなければならない。

- 静的な layer 文書を描画する
- layer order、visibility、opacity、placement を尊重する
- `backgroundColor` が存在する場合、canvas 全域の塗りつぶしとして描画する
- frame が `layers` を定義している場合、その frame ローカルの layer 集合と順序を使う
- `duration_ms` を用いた基本 frame animation を描画する

### Editor 適合

適合する editor は次を満たさなければならない。

- 妥当な MRPAF を読み書きする
- round-trip で既知の意味を保持する
- 可能な限り未知の extension を保持する
- 可能な限り未知のトップレベルメンバーを保持する

### AI Generator 適合

適合する AI generator は次を満たさなければならない。

- 妥当な MRPAF JSON を出力する
- 参照される layer と animation に対して安定した文字列識別子を出力する
- 非コアのランタイム意味論をコア schema に混入させない

## JSON Schema

以下の schema は、このドラフトに整合する最小構造コア schema である。

これは構造上の基準線であり、完全な semantic validator ではない。

本文の規範要件には、JSON Schema だけでは完全に表現できないものがあり、少なくとも次を含む。

- palette、layer、animation identifier の一意性
- animation が宣言済み layer identifier のみを参照していること
- 宣言寸法に対する decoded pixel-count の一致
- 導出フィールドの禁止
- 警告と回復の挙動
- round-trip 中の未知メンバー保持
- `subpixelPrecision` が `0` のときの整数座標強制
- minor / patch 更新で意味を変えてはならないこと
- sparse 座標重複時の拒否挙動
- `extensions.layerGroups.groups[].id` の一意性
- 1つの `extensions.layerGroups` group 内での重複 member 検出

注記: `frames.layers` と `layerOverrides` における layer ID の参照整合性は、reader 実装で検証しなければならない。標準 JSON Schema では、それらのキーや値が `layers` に宣言された layer ID と一致することを強制できない。

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://mrpaf.org/schemas/v3.1.0/mrpaf.schema.json",
  "title": "MRPAF v3.1.0 core schema",
  "type": "object",
  "additionalProperties": true,
  "required": ["format", "version", "metadata", "canvas", "palette", "layers"],
  "properties": {
    "format": {
      "const": "MRPAF"
    },
    "version": {
      "type": "string",
      "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$"
    },
    "metadata": {
      "type": "object",
      "additionalProperties": false,
      "required": ["title"],
      "properties": {
        "title": { "type": "string", "minLength": 1 },
        "author": { "type": "string" },
        "license": { "type": "string" },
        "source": { "type": "string" },
        "created": { "type": "string", "format": "date-time" },
        "modified": { "type": "string", "format": "date-time" },
        "description": { "type": "string" },
        "tags": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
    },
    "canvas": {
      "type": "object",
      "additionalProperties": false,
      "required": ["baseWidth", "baseHeight"],
      "properties": {
        "baseWidth": { "type": "integer", "minimum": 1 },
        "baseHeight": { "type": "integer", "minimum": 1 },
        "backgroundColor": {
          "type": ["string", "null"],
          "pattern": "^#([0-9A-Fa-f]{6}|[0-9A-Fa-f]{8})$"
        },
        "subpixelPrecision": { "type": "integer", "minimum": 0, "maximum": 8, "default": 0 }
      }
    },
    "palette": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "hex"],
        "properties": {
          "id": { "type": "string", "minLength": 1 },
          "hex": {
            "type": "string",
            "pattern": "^#([0-9A-Fa-f]{6}|[0-9A-Fa-f]{8})$"
          },
          "name": { "type": "string" }
        }
      }
    },
    "layers": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "name", "resolution", "placement", "pixels"],
        "properties": {
          "id": { "type": "string", "minLength": 1 },
          "name": { "type": "string", "minLength": 1 },
          "visible": { "type": "boolean", "default": true },
          "opacity": { "type": "number", "minimum": 0, "maximum": 1, "default": 1 },
          "resolution": {
            "type": "object",
            "additionalProperties": false,
            "required": ["pixelArraySize", "scale"],
            "properties": {
              "pixelArraySize": {
                "type": "object",
                "additionalProperties": false,
                "required": ["width", "height"],
                "properties": {
                  "width": { "type": "integer", "minimum": 1 },
                  "height": { "type": "integer", "minimum": 1 }
                }
              },
              "scale": { "type": "number", "exclusiveMinimum": 0 }
            }
          },
          "placement": {
            "type": "object",
            "additionalProperties": false,
            "required": ["x", "y"],
            "properties": {
              "x": { "type": "number" },
              "y": { "type": "number" }
            }
          },
          "pixels": {
            "type": "object",
            "oneOf": [
              {
                "additionalProperties": false,
                "required": ["encoding", "data"],
                "properties": {
                  "encoding": { "const": "array" },
                  "data": {
                    "type": "array",
                    "items": { "type": "string" }
                  }
                }
              },
              {
                "additionalProperties": false,
                "required": ["encoding", "data"],
                "properties": {
                  "encoding": { "const": "rle" },
                  "data": {
                    "type": "array",
                    "items": {
                      "type": "array",
                      "minItems": 2,
                      "maxItems": 2,
                      "prefixItems": [
                        { "type": "string" },
                        { "type": "integer", "minimum": 1 }
                      ]
                    }
                  }
                }
              },
              {
                "additionalProperties": false,
                "required": ["encoding", "defaultValue", "data"],
                "properties": {
                  "encoding": { "const": "sparse" },
                  "defaultValue": { "type": "string" },
                  "data": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "additionalProperties": false,
                      "required": ["x", "y", "value"],
                      "properties": {
                        "x": { "type": "integer", "minimum": 0 },
                        "y": { "type": "integer", "minimum": 0 },
                        "value": { "type": "string" }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "animations": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "name", "loop", "frames"],
        "properties": {
          "id": { "type": "string", "minLength": 1 },
          "name": { "type": "string", "minLength": 1 },
          "loop": { "type": "boolean" },
          "frames": {
            "type": "array",
            "items": {
              "type": "object",
              "additionalProperties": false,
              "required": ["duration_ms"],
              "properties": {
                "duration_ms": { "type": "integer", "minimum": 1 },
                "layers": {
                  "type": "array",
                  "items": { "type": "string" }
                },
                "layerOverrides": {
                  "type": "object",
                  "additionalProperties": {
                    "type": "object",
                    "additionalProperties": false,
                    "properties": {
                      "visible": { "type": "boolean" },
                      "opacity": { "type": "number", "minimum": 0, "maximum": 1 },
                      "placement": {
                        "type": "object",
                        "additionalProperties": false,
                        "properties": {
                          "x": { "type": "number" },
                          "y": { "type": "number" }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "extensions": {
      "type": "object",
      "properties": {
        "layerGroups": {
          "type": "object",
          "additionalProperties": true,
          "required": ["groups"],
          "properties": {
            "groups": {
              "type": "array",
              "items": {
                "type": "object",
                "additionalProperties": false,
                "required": ["id", "members"],
                "properties": {
                  "id": { "type": "string", "minLength": 1 },
                  "name": { "type": "string" },
                  "members": {
                    "type": "array",
                    "items": { "type": "string", "minLength": 1 }
                  }
                }
              }
            }
          }
        }
      },
      "additionalProperties": true
    }
  }
}
```

## 正規サンプル

### 最小有効サンプル

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Minimal Sprite"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" }
  ],
  "layers": [
    {
      "id": "base",
      "name": "Base",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "rle",
        "data": [
          ["transparent", 256]
        ]
      }
    }
  ]
}
```

### 標準的な静止画サンプル

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Two-Layer Portrait",
    "author": "Example Artist",
    "license": "CC-BY-4.0"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16,
    "backgroundColor": "#00000000"
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "skin", "hex": "#F0C0A0" },
    { "id": "ink", "hex": "#202020" }
  ],
  "layers": [
    {
      "id": "body",
      "name": "Body",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "rle",
        "data": [
          ["transparent", 40],
          ["skin", 8],
          ["transparent", 208]
        ]
      }
    },
    {
      "id": "face_detail",
      "name": "Face Detail",
      "resolution": {
        "pixelArraySize": { "width": 8, "height": 8 },
        "scale": 2
      },
      "placement": {
        "x": 4,
        "y": 4
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 1, "y": 2, "value": "ink" },
          { "x": 5, "y": 2, "value": "ink" }
        ]
      }
    }
  ]
}
```

### 標準的な基本アニメーションサンプル

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Blink Animation"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "ink", "hex": "#202020" }
  ],
  "layers": [
    {
      "id": "face_open",
      "name": "Face Open",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 5, "y": 6, "value": "ink" },
          { "x": 10, "y": 6, "value": "ink" }
        ]
      }
    },
    {
      "id": "face_closed",
      "name": "Face Closed",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 5, "y": 6, "value": "ink" },
          { "x": 6, "y": 6, "value": "ink" },
          { "x": 10, "y": 6, "value": "ink" },
          { "x": 11, "y": 6, "value": "ink" }
        ]
      }
    }
  ],
  "animations": [
    {
      "id": "blink",
      "name": "Blink",
      "loop": true,
      "frames": [
        {
          "duration_ms": 700,
          "layers": ["face_open"]
        },
        {
          "duration_ms": 100,
          "layers": ["face_closed"]
        }
      ]
    }
  ]
}
```

### 標準 layer group 拡張サンプル

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "Grouped Portrait"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "skin", "hex": "#F0C0A0" },
    { "id": "ink", "hex": "#202020" }
  ],
  "layers": [
    {
      "id": "body",
      "name": "Body",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "rle",
        "data": [
          ["transparent", 40],
          ["skin", 8],
          ["transparent", 208]
        ]
      }
    },
    {
      "id": "face_detail",
      "name": "Face Detail",
      "resolution": {
        "pixelArraySize": { "width": 8, "height": 8 },
        "scale": 2
      },
      "placement": {
        "x": 4,
        "y": 4
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 1, "y": 2, "value": "ink" },
          { "x": 5, "y": 2, "value": "ink" }
        ]
      }
    }
  ],
  "extensions": {
    "layerGroups": {
      "groups": [
        {
          "id": "character",
          "name": "Character",
          "members": ["body", "face_detail"]
        }
      ]
    }
  }
}
```

### 互換性のある未知 extension サンプル

```json
{
  "format": "MRPAF",
  "version": "3.1.0",
  "metadata": {
    "title": "AI Handoff Example"
  },
  "canvas": {
    "baseWidth": 16,
    "baseHeight": 16
  },
  "palette": [
    { "id": "transparent", "hex": "#00000000" },
    { "id": "accent", "hex": "#FF00AA" }
  ],
  "layers": [
    {
      "id": "base",
      "name": "Base",
      "resolution": {
        "pixelArraySize": { "width": 16, "height": 16 },
        "scale": 1
      },
      "placement": {
        "x": 0,
        "y": 0
      },
      "pixels": {
        "encoding": "sparse",
        "defaultValue": "transparent",
        "data": [
          { "x": 7, "y": 7, "value": "accent" }
        ]
      }
    }
  ],
  "extensions": {
    "ai": {
      "model": "example-model",
      "prompt": "single accent pixel in center",
      "seed": 42
    }
  }
}
```

## ドラフト変更要約

このドラフトは、コア描画意味を変えずに標準の layer-group 拡張を追加する。

`v3.0.0` と比べて、このドラフトは次を行っている。

- `v3.0.0` の core object model と rendering semantics を維持する
- `extensions.layerGroups` を標準の組織化拡張として追加する
- layer grouping をコア描画モデルの外側に保つ
- 未知 extension を無視する実装との forward-compatible な扱いを維持する

## レビューチェックリスト

新しいドラフトを公開する前に、次のチェックリストを使うこと。

- 位置づけが Purpose and Scope と一致している
- `Why Not PNG?` がちょうど3点で、余計なマーケティング表現を含まない
- 比較表が PNG や Aseprite に対して過剰な主張をしていない
- トップレベルオブジェクトのメンバーが schema と一致している
- `coordinateSystem`、`pixelUnit`、`fps`、`effectiveSize`、`resources` のような削除済みフィールドが規範サンプルに出てこない
- core encoding として `array`、`rle`、`sparse` のみを説明している
- `defaultValue` が `sparse` でのみ必須で、`array` と `rle` では禁止されている
- 重複する sparse 座標が回復対象ではなく拒否対象になっている
- アニメーション例が `fps` ではなく `duration_ms` を使っている
- `layerOverrides` が schema 上 `visible`、`opacity`、`placement` に制約されている
- animation frame が宣言済み layer ID のみを参照している
- `backgroundColor` の描画意味が明示されている
- `extensions.layerGroups` が組織化用途であり、描画意味を持たないことが明示されている
- `extensions.layerGroups.groups` の順序が組織化用途のみであることが明示されている
- 未知 extension の扱いが本文とサンプルの両方で説明されている
- 必須 metadata が `title` に限定されている
- 導出サイズルールが一度だけ定義され、他所で矛盾していない
- すべての JSON サンプルが妥当な JSON である
- MRPAF を最終配布用画像形式または完全なランタイム仕様だと主張する節が存在しない
