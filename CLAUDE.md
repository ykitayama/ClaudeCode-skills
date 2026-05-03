# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.


---
# プロジェクト固有ガイドライン（医療情報システム担当）
# Project-Specific Guidelines for Medical IT Operations
---

## 私のコンテキスト / My Context

- 独立行政法人の医療情報システム担当（専門職）
- 西日本18病院を統括する本部支店に所属
- 担当領域：情報システム導入・運用管理、情報セキュリティ管理
- Python 中級者：主にデータ分析・集計・自動化に活用
- 生成AIを活用した医療現場の業務改善に取り組み中

## コーディングの前提条件 / Coding Prerequisites

### 言語・環境
- **Python を優先**する（他言語が適切な場合は理由を示してから提案すること）
- コードには**日本語コメント**を付ける
- 実行環境は **Windows Server / Windows 11** を想定する
- パッケージ管理は **pip** を使用（conda は使用しない）

### 推奨ライブラリ（優先順）
- データ操作：`pandas` → `polars`（大規模データの場合）
- 可視化：`matplotlib` → `plotly`（インタラクティブが必要な場合）
- ファイル操作：`openpyxl`（Excel）、`pathlib`（パス操作）
- HTTP通信：`requests`（外部API連携時）

### 禁止・注意事項
- 患者個人情報（氏名・生年月日・診察番号等）をコードにハードコードしない
- 本番データベースへの直接 UPDATE/DELETE は原則禁止（必ず確認を求める）
- 外部ネットワークへのデータ送信が発生するコードは必ず警告を表示すること

## 医療ドメイン知識の扱い / Medical Domain Knowledge

- **医療情報の規格**（HL7、FHIR、SS-MIX2、JAHIS）に関連する処理では、規格の概要を補足すること
- **診療報酬コード**（ICD-10、DPC、Kコード等）を扱う場合は、コードの意味を日本語でコメントに含める
- **診療報酬改定**に関わるロジック変更は、改定年月を必ずコメントに記載する（例：`# 2024年改定対応`）
- 略語が出てきた場合は初出時に日本語で補足すること

## 出力・ファイル管理 / Output & File Management

```
プロジェクトルート/
├── data/         # 入力データ（匿名化済みCSVなど）
├── output/       # 分析結果・レポート出力先
├── scripts/      # メインスクリプト
└── docs/         # 仕様書・説明文書
```

- 出力ファイルは必ず `output/` ディレクトリに保存する
- ファイル名には処理内容と日付を含める（例：`病院別稼働率_20250503.xlsx`）
- 大量データを扱う場合は処理進捗を `tqdm` 等で表示する

## セキュリティ要件 / Security Requirements

- 個人情報を含む可能性があるデータは処理前に匿名化ステップを挟むこと
- ログファイルに個人情報が出力されないよう注意喚起すること
- 外部サービス（API・クラウド）への接続が発生する場合は、**必ず事前に確認を求める**
- パスワード・認証情報は環境変数（`.env`）から読み込む形式にすること

## コミュニケーションスタイル / Communication Style

- 回答は**日本語**で行う
- 複数の実装方法がある場合は、**トレードオフを表にして示してから**実装に進む
- 医療制度・情報セキュリティ上のリスクがある場合は、冒頭に ⚠️ で明示する
- 18病院への横展開を想定したコードの場合、**汎用性・再利用性**を意識したコメントを付ける

## 成功基準の例 / Success Criteria Examples

| タスク | 成功基準 |
|--------|----------|
| データ集計スクリプト作成 | 全18病院のCSVで正常動作し、合計値が手計算と一致する |
| セキュリティ点検ツール | 点検項目リストの全項目をカバーし、結果をExcel出力できる |
| API連携スクリプト | タイムアウト・エラー時にログを残してリトライできる |
| レポート自動生成 | 毎月1日に前月分が自動生成され、所定フォルダに保存される |