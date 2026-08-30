# 上級模擬試験 / Advanced Mock Exam

> Claude を **金融・医療・法務・規制対応・大規模分散システム** などの**ビジネスクリティカル領域**に投入するエキスパートエンジニア向け模擬問題集です。
>
> A mock exam set for expert engineers deploying Claude in **business-critical domains** (finance, healthcare, legal, regulated industries, large-scale distributed systems).

## 位置づけ / Positioning

公式の `Claude Certified Architect — Foundations` 認定試験は入門〜中級レベルです。本模擬問題集は、その**ドメイン構成（5ドメイン・配点比率）を枠組みとして踏襲**しつつ、**Foundations を超える上級レベル**のシナリオで構成しています。

The official `Claude Certified Architect — Foundations` exam is entry-to-intermediate level. This mock exam **follows the same five-domain blueprint** but is designed at an **advanced/expert level** that goes beyond Foundations.

各問は次のうち **2 つ以上** の上級者要素を含みます：

Each question incorporates **two or more** of the following advanced elements:

1. 規制・コンプライアンス制約 (HIPAA / PCI DSS / SOX / GDPR / MiFID II など)
2. マルチサブシステム連携と障害伝播
3. 部分障害・劣化挙動・冪等性
4. コスト・レイテンシ・SLA トレードオフ
5. 監査ログ・可観測性・再現性
6. 複数の妥当に見える設計案からのトレードオフ判断

## ドメイン構成 / Domains

| # | ドメイン / Domain | 配点 / Weight | ファイル / File |
|---|---|---|---|
| 1 | エージェントアーキテクチャとオーケストレーション / Agent Architecture and Orchestration | 27% | [`domain1_agent_architecture.md`](./domain1_agent_architecture.md) |
| 2 | ツール設計と MCP 統合 / Tool Design and MCP Integration | 18% | [`domain2_tool_design_mcp.md`](./domain2_tool_design_mcp.md) |
| 3 | Claude Code の設定とワークフロー / Claude Code Configuration and Workflows | 20% | [`domain3_claude_code_workflows.md`](./domain3_claude_code_workflows.md) |
| 4 | プロンプトエンジニアリングと構造化出力 / Prompt Engineering and Structured Output | 20% | [`domain4_prompt_structured_output.md`](./domain4_prompt_structured_output.md) |
| 5 | コンテキスト管理と信頼性 / Context Management and Reliability | 15% | [`domain5_context_reliability.md`](./domain5_context_reliability.md) |
| **計 / Total** | | **100%** | 150 問 / questions |

## 形式 / Format

- **4 択・単一選択** / 4 options, single answer
- **解説付き** — 各問に正解理由と各誤答の不適切な点を記載
- **二言語併記** — 日本語と英語（用語の正確性確保のため）

## 推奨される使い方 / How to Use

1. ドメインごとに 1 ファイルずつ取り組む（配点が高い順を推奨：1 → 3 → 4 → 2 → 5）
2. 各問の `<details>` を開く前に自分の回答を決める
3. 解説を読んで誤答パターンを確認する
4. ガイド本編 (`../guide_ja.md` / `../guide_en.MD`) の参照節に戻って深掘り
5. 8 割（30 問中 24 問）以上を各ドメインで正解できれば、上級レベルとして安定的な実力

Recommended approach:

1. Work through one domain file at a time (suggested order by weight: 1 → 3 → 4 → 2 → 5)
2. Decide your answer before opening the `<details>` block
3. Review the explanations including why each distractor is wrong
4. Return to the relevant section of the main guide (`../guide_ja.md` / `../guide_en.MD`) for deeper study
5. Scoring ≥80% per domain (24/30) consistently indicates solid advanced-level proficiency

## 上位資格 / Professional tier

Architect トラックの上位資格 `Claude Certified Architect — Professional` (CCAR-P) 向けの模擬問題集を [`professional/`](./professional/) に用意しています。全 7 ドメイン × 30 問 = 210 問。Foundations の 5 ドメインに加えて、**ガバナンス・リスク管理**、**ステークホルダー折衝とライフサイクル管理**、**開発者生産性と運用イネーブルメント** の 3 領域が加わります。

A mock exam for the advanced tier, `Claude Certified Architect — Professional` (CCAR-P), is available in [`professional/`](./professional/): seven domains × 30 questions = 210. Beyond the Foundations five, it adds governance and risk, stakeholder communication and lifecycle, and developer enablement.

## 注意 / Disclaimer

本模擬問題集は学習目的の**非公式**コンテンツです。Anthropic 公式の試験問題ではなく、公開ガイドの内容に基づき独自に作成しています。

This is **unofficial** study material. These are not actual Anthropic exam questions; they are independently authored based on publicly available guide content.
