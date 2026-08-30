# Claude Certified Architect — Professional (CCAR-P) 上級模擬試験 / Advanced Mock Exam

> Architect トラックの**上位資格** `Claude Certified Architect — Professional` のブループリントに沿った模擬問題集です。全 7 ドメイン × 30 問 = **210 問**。
>
> A mock exam set aligned to the blueprint of `Claude Certified Architect — Professional`, the **advanced tier** of the Architect track. Seven domains × 30 questions = **210 questions**.

---

## ⚠️ 出典と非公式である旨 / Sourcing and disclaimer

**本ディレクトリの内容は学習目的の非公式教材です。Anthropic 公式の試験問題ではありません。**

CCAR-P の**公式試験ガイド (PDF) は Claude Partner Network 限定の Skilljar ポータル内**で配布されており、一般公開されていません。以下に示すドメイン構成・配点は、**複数の第三者試験対策サイトで一致していた記述を突き合わせて再構成したもの**であり、Anthropic の一次情報で検証したものではありません。受験前には必ず**ポータル内の公式 Exam Guide で最新の配点と出題範囲を確認してください**。配点が改訂されていた場合、本ファイルの記述より公式ガイドが常に優先されます。

**This is unofficial study material, not actual Anthropic exam content.**

The official CCAR-P exam guide (PDF) is distributed **inside the Claude Partner Network Skilljar portal** and is not public. The domain list and weights below were **reconstructed by cross-checking multiple third-party exam-prep sources** and have **not** been verified against a primary Anthropic source. Always confirm the current blueprint in the official Exam Guide in the portal before sitting the exam; where they differ, the official guide wins.

---

## Foundations との関係 / Relationship to Foundations

| | Foundations (CCAR-F) | **Professional (CCAR-P)** |
|---|---|---|
| 問う内容 / Core question | Claude システムを**設計できるか** / Can you design a working Claude system? | 本番の Claude システムを**保有・運営できるか** / Can you *own* one in production? |
| ドメイン数 / Domains | 5 | **7** |
| 問題数 / Questions | — | **63** |
| 制限時間 / Duration | — | **120 分 / minutes** |
| 合格点 / Passing score | 720 / 1000 | **720 / 1000** |
| 受験料 / Cost | $99（一般提供時） | **$175** |

Foundations の 5 ドメインは Professional でも形を変えて残りますが、**Professional で新たに加わる本質的な差分は次の 3 領域**です。技術的正しさではなく、**組織・規制・ライフサイクルに対する責任**を問われます。

The five Foundations domains persist in altered form, but the **three domains that genuinely distinguish Professional** are the following. They test accountability to the organization, to regulators, and across the lifecycle — not technical correctness alone.

1. **Governance, Safety & Risk Management** — ガバナンス・安全性・リスク管理
2. **Stakeholder Communication & Lifecycle Management** — ステークホルダー折衝とライフサイクル管理
3. **Developer Productivity & Operational Enablement** — 開発者生産性と運用イネーブルメント

---

## ドメイン構成 / Domains

| # | ドメイン / Domain | 配点 / Weight | 本番相当問数 / Items @63 | 本模擬 / Here | ファイル / File |
|---|---|---|---|---|---|
| 1 | ソリューション設計とアーキテクチャ / Solution Design and Architecture | 17% | 11 | 30 | [`domain1_solution_design.md`](./domain1_solution_design.md) |
| 2 | Claude モデル・プロンプト・コンテキスト工学 / Claude Models, Prompting and Context Engineering | 13% | 8 | 30 | [`domain2_models_prompting_context.md`](./domain2_models_prompting_context.md) |
| 3 | 統合アーキテクチャ / Integration | **19%** | 12 | 30 | [`domain3_integration.md`](./domain3_integration.md) |
| 4 | 評価と品質保証 / Evaluation | 16% | 10 | 30 | [`domain4_evaluation.md`](./domain4_evaluation.md) |
| 5 | ガバナンス・安全性・リスク管理 / Governance, Safety and Risk Management | 14% | 9 | 30 | [`domain5_governance_risk.md`](./domain5_governance_risk.md) |
| 6 | ステークホルダー折衝とライフサイクル管理 / Stakeholder Communication and Lifecycle Management | 14% | 9 | 30 | [`domain6_stakeholder_lifecycle.md`](./domain6_stakeholder_lifecycle.md) |
| 7 | 開発者生産性と運用イネーブルメント / Developer Productivity and Operational Enablement | 7% | 4 | 30 | [`domain7_developer_enablement.md`](./domain7_developer_enablement.md) |
| **計 / Total** | | **100%** | **63** | **210** | |

配点が 7%〜19% の範囲に収まる**「広く浅くない」ブレス型**の試験です。単一ドメインの深掘りでは合格点に届きにくく、7 領域すべてでシニアアーキテクト相当の判断ができることが要求されます。

Weights span only 7%–19%, making this a **breadth exam**: depth in one domain will not carry you: senior-architect judgment is expected across all seven.

---

## 形式 / Format

- **4 択・単一選択** / 4 options, single answer
- **解説付き** — 正解の理由と、各誤答が不適切な理由を記載 / Explanations covering the key and every distractor
- **二言語併記** — 日本語と英語 / Bilingual (Japanese / English)
- 各問は次のうち **2 つ以上** の上級者要素を含みます / Each question incorporates **two or more** of:
  1. 規制・コンプライアンス制約 (GDPR / EU AI Act / HIPAA / PCI DSS / SOX / DORA / MiFID II など)
  2. 組織横断の利害調整とエスカレーション経路
  3. マルチテナント・マルチリージョン・大規模分散環境
  4. コスト・レイテンシ・品質・SLA のトレードオフ判断
  5. 監査可能性・可観測性・再現性
  6. モデル更新やベンダー変更を含むライフサイクル移行

---

## 推奨される使い方 / How to Use

1. **配点が高い順に取り組む** — 3 (19%) → 1 (17%) → 4 (16%) → 5 (14%) → 6 (14%) → 2 (13%) → 7 (7%)
2. Foundations 未受験・未学習なら、先に [`../README.md`](../README.md) の 5 ドメインを済ませる（ドメイン 1〜3 は Foundations の知識を前提にしています）
3. 各問の `<details>` を開く前に自分の回答を決める
4. 解説を読み、**なぜ他の 3 択が「もっともらしく見えるのに不適切なのか」**を言語化できるか確認する
5. 各ドメインで **8 割（30 問中 24 問）** 以上を安定して正解できれば、本番の 720 点ラインに対して余裕があります

Recommended order by weight: **3 → 1 → 4 → 5 → 6 → 2 → 7**. Scoring ≥ 24/30 consistently in each domain indicates comfortable margin against the 720 line.

---

## 新規 3 ドメインの学習で特に注意すべき点 / Traps in the three new domains

Foundations 合格者が Professional で最も失点しやすいのは、**「技術的に最も洗練された選択肢」を選んでしまう**パターンです。新規 3 ドメインでは、正解はしばしば次の性質を持ちます。

- **決定論的な統制が、確率的な統制に優先する** — プロンプトによる約束は監査証跡にならない
- **不可逆な影響には人間の承認ゲートが要る** — 自動化の度合いは技術的可能性ではなくリスクで決める
- **説明できない改善は改善ではない** — ステークホルダーに根拠を提示できない最適化は採用されない
- **最小限の変更が最良であることが多い** — 全面刷新は「もっともらしい誤答」の定番

The most common Professional-level mistake is picking the **technically most sophisticated** option. In the three new domains the correct answer usually satisfies: deterministic controls beat probabilistic ones; irreversible impact requires a human approval gate; an improvement you cannot explain to stakeholders will not ship; and the minimal change is usually right.

---

## 目次 / Index

| ドメイン / Domain | 配点 / Weight |
|---|---|
| [1. ソリューション設計とアーキテクチャ / Solution Design and Architecture](./domain1_solution_design.md) | 17% |
| [2. Claude モデル・プロンプト・コンテキスト工学 / Models, Prompting and Context Engineering](./domain2_models_prompting_context.md) | 13% |
| [3. 統合アーキテクチャ / Integration](./domain3_integration.md) | 19% |
| [4. 評価と品質保証 / Evaluation](./domain4_evaluation.md) | 16% |
| [5. ガバナンス・安全性・リスク管理 / Governance, Safety and Risk Management](./domain5_governance_risk.md) | 14% |
| [6. ステークホルダー折衝とライフサイクル管理 / Stakeholder Communication and Lifecycle Management](./domain6_stakeholder_lifecycle.md) | 14% |
| [7. 開発者生産性と運用イネーブルメント / Developer Productivity and Operational Enablement](./domain7_developer_enablement.md) | 7% |

**Foundations 模擬試験 / Foundations mock exam:** [`../README.md`](../README.md)
