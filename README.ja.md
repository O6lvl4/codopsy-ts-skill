<h1 align="center">codopsy-ts-skill</h1>

<p align="center">
  <strong><a href="https://github.com/O6lvl4/codopsy-ts">codopsy-ts</a> の Claude Code プラグイン</strong>
</p>

<p align="center">
  Claude Code からコード品質の解析・品質トレンドの追跡・問題の自動修正を実行。
</p>

<p align="center">
  <a href="README.md">English</a>
</p>

---

## 何ができるか

このプラグインは Claude Code に `/codopsy` スキルを追加します。[codopsy-ts](https://github.com/O6lvl4/codopsy-ts) をコードベースに対して実行し、結果を解釈して、見つかった問題を自動修正できます。

**品質スコア** — ファイル・プロジェクトごとに A&ndash;F グレード（0&ndash;100 スケール）

**解析** — 循環的・認知的複雑度、13 の lint ルール、ファイルごとの内訳

**ベースライン** — 品質のスナップショットを保存し、劣化時に CI を失敗させる

**ホットスポット** — 複雑度と git チャーンを掛け合わせてリスクの高いファイルを検出

**修正** — Claude が解析結果を読み取り、各問題を理解して最小限の修正を適用

---

## インストール

```
/plugin install codopsy@O6lvl4/codopsy-ts-skill
```

> [codopsy-ts](https://www.npmjs.com/package/codopsy-ts) v1.1.0+ が必要です（`npm install -g codopsy-ts`）

---

## 使い方

```bash
# ./src を解析（デフォルト）
/codopsy

# 特定のディレクトリを解析
/codopsy ./lib

# 解析して問題を自動修正
/codopsy --fix

# 特定のディレクトリを解析して修正
/codopsy ./lib --fix

# ベースラインの保存と比較
/codopsy --baseline

# ホットスポット解析（複雑度 × git チャーン）
/codopsy --hotspots
```

---

## 動作の流れ

### レポートモード（デフォルト）

1. `codopsy-ts analyze` を `--verbose` で実行
2. **品質スコア**（A&ndash;F グレード）とグレード分布を表示
3. サマリーを表示：ファイル数、error/warning/info の件数、ホットスポット
4. 各 issue をファイル、行番号、ルール、メッセージとともに一覧表示

### ベースラインモード（`--baseline`）

1. 現在の品質メトリクスをベースラインとして保存
2. 以降の実行でベースラインと比較
3. スコア変化、issue 増減、改善/劣化ファイルを表示

### ホットスポットモード（`--hotspots`）

1. `--hotspots` フラグ付きで解析を実行
2. 複雑度スコアと git チャーン（コミット数、著者数）を組み合わせ
3. リスクレベルを表示：HIGH / MEDIUM / LOW

### 修正モード（`--fix`）

1. 解析を実行
2. 優先順位に従って問題を修正：
   - `no-var` → `let`/`const` に置換
   - `eqeqeq` → `==` を `===` に置換
   - `prefer-const` → `let` を `const` に変更
   - `no-empty-function` → 実装を追加、または空にしている理由をコメント
   - `no-param-reassign` → ローカル変数を導入
   - `max-params` → オプションオブジェクトを抽出
   - `max-depth` → ヘルパー関数の抽出、早期リターン
   - `max-complexity` / `max-cognitive-complexity` → 関数を分割
   - `max-lines` → モジュールに分離
   - `no-nested-ternary` → if/else に置換（JSX 境界の三項演算子は除外）
   - `no-any` → 型注釈を追加
3. 再解析して改善を検証
4. 品質スコアの変化を含む before/after の比較を表示

---

## 実行例

```
> /codopsy

=== Codopsy Analysis ===

Quality Score: A (95/100)
Files: 12 | Errors: 0 | Warnings: 3 | Info: 5
Grade distribution: A: 9, B: 2, C: 1

| File             | Line | Rule           | Score | Message                                    |
|------------------|------|----------------|-------|--------------------------------------------|
| src/parser.ts    | 42   | max-complexity | B (78)| Function "parse" has complexity 14 (max 10) |
| src/utils.ts     | 8    | no-var         | A (92)| Unexpected var, use let or const            |
| src/handler.ts   | 15   | eqeqeq         | A (90)| Expected "===" but found "=="              |

> /codopsy --fix

Fixed 3 issues:
  ✓ src/utils.ts:8 — var → const
  ✓ src/handler.ts:15 — == → ===
  ✓ src/parser.ts:42 — extracted helper functions

Re-analysis: A (99/100), 0 errors, 0 warnings, 5 info
```

---

## 必要なもの

- [Claude Code](https://claude.ai/code)
- [codopsy-ts](https://www.npmjs.com/package/codopsy-ts) v1.1.0+
- Node.js 18+

---

## リンク

- [codopsy-ts](https://github.com/O6lvl4/codopsy-ts) — CLI ツール本体
- [npm パッケージ](https://www.npmjs.com/package/codopsy-ts)

## ライセンス

ISC
