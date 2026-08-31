# Claude Certified Architect — Professional (CCAR-P) 上級模擬試験 / Advanced Mock Exam

> Architect トラックの**上位資格** `Claude Certified Architect — Professional` のブループリントに沿った模擬問題集です。全 7 ドメイン × 30 問 = **210 問**。
>
> A mock exam set aligned to the blueprint of `Claude Certified Architect — Professional`, the **advanced tier** of the Architect track. Seven domains × 30 questions = **210 questions**.

---

## 出典と、何が公式で何が非公式か / Sourcing: what is official and what is not

配点・ドメイン名・出題範囲は **Anthropic 発行の公式 Exam Guide（Version 1.0・Effective July 2026）の一次情報**です。以下の表とドメイン別の出題範囲は、その PDF の記載に基づいています。

- 公式 Exam Guide (PDF): [`Claude Certified Architect – Professional Exam Guide`](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor/6nizmqk8tpzpfjvt6qmmav7rh/public/1783542810/Claude+Certified+Architect+%E2%80%93+Professional+Exam+Guide.pdf)
- 配布元: Anthropic Partner Academy（Skilljar）。受験登録は Pearson VUE 経由

**一方で、収録されている 210 問はすべて学習目的の独自作成であり、公式の試験問題ではありません。** 実際の試験問題は Anthropic の機密情報であり、受験時に守秘義務契約（NDA）の対象になります。本問題集は公式ブループリントの各 objective に対して独自にシナリオを書き起こしたものです。

The weights, domain names, and objectives below are transcribed from **Anthropic's official Exam Guide (Version 1.0, effective July 2026)** — a primary source. **The 210 questions themselves are independently authored study material and are not actual exam content**, which is Anthropic's confidential property and subject to an NDA at test time.

> 公式ガイドは予告なく改訂されます / The guide is subject to change without notice. 受験前に最新版を確認してください。

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
| 4 | 評価・テスト・最適化 / Evaluation, Testing & Optimization | 16% | 10 | 30 | [`domain4_evaluation.md`](./domain4_evaluation.md) |
| 5 | ガバナンス・安全性・リスク管理 / Governance, Safety and Risk Management | 14% | 9 | 30 | [`domain5_governance_risk.md`](./domain5_governance_risk.md) |
| 6 | ステークホルダー折衝とライフサイクル管理 / Stakeholder Communication and Lifecycle Management | 14% | 9 | 30 | [`domain6_stakeholder_lifecycle.md`](./domain6_stakeholder_lifecycle.md) |
| 7 | 開発者生産性と運用イネーブルメント / Developer Productivity and Operational Enablement | 7% | 4 | 30 | [`domain7_developer_enablement.md`](./domain7_developer_enablement.md) |
| **計 / Total** | | **100%** | **63** | **210** | |

配点が 7%〜19% の範囲に収まる**「広く浅くない」ブレス型**の試験です。単一ドメインの深掘りでは合格点に届きにくく、7 領域すべてでシニアアーキテクト相当の判断ができることが要求されます。

Weights span only 7%–19%, making this a **breadth exam**: depth in one domain will not carry you: senior-architect judgment is expected across all seven.

---

## ドメイン別の出題範囲 / Detailed objectives by domain（公式ガイドの記載）

公式ガイドは各ドメインについて、候補者が遂行できることを期待される **objective** を列挙しています。試験問題はこれらの objective に対して作成されます。以下は公式の記載です。

The official guide lists, per domain, the tasks a candidate is expected to perform; exam items are written against these objectives. The following is transcribed from the guide.

**Domain 1: Solution Design & Architecture (17%)**
- ビジネス課題を Claude ベースの AI ソリューションに翻訳する / Translate business problems into Claude-based AI solutions
- 入力 → 処理 → 出力 → フィードバックループの端から端までのアーキテクチャを設計する / Design end-to-end architectures (input → processing → output → feedback loops)
- 適切なアーキテクチャパターン（ワークフロー、エージェンティック、augmented LLM）を選択する / Select appropriate architectural patterns (workflow, agentic, augmented LLM)
- マルチエージェントシステムとオーケストレーション戦略を設計する / Design multi-agent systems and orchestration strategies
- 複雑な問題解決のための分解手法を適用する / Apply decomposition techniques for complex problem solving
- ソリューションをビジネス価値の柱（効率化、変革、生産性、コスト、性能 SLA）に整合させる / Align solutions to business value pillars (efficiency, transformation, productivity, cost, performance SLAs)

**Domain 2: Claude Models, Prompting & Context Engineering (13%)**
- トレードオフに基づいて適切な Claude モデルを選択する / Select appropriate Claude models based on trade-offs
- system プロンプト、テンプレート、ガードレールを設計する / Design system prompts, templates, and guardrails
- プロンプト技法（zero-shot、few-shot、chain-of-thought）を適用する / Apply prompt engineering techniques (zero-shot, few-shot, chain-of-thought)
- コンテキストウィンドウを最適化しトークン使用量を管理する / Optimize context windows and manage token usage
- プロンプト再利用の戦略（キャッシュ、モジュール化、Skills）を実装する / Implement prompt reuse strategies (caching, modular prompts, Skills)

**Domain 3: Integration (19%)**
- ツール／エージェントの構成を capability bloat の観点で評価する / Evaluate tool/agent configuration for capability bloat
- 認証・認可の要件を分析してセキュリティギャップを特定する / Analyze authentication and authorization requirements to identify security gaps
- 精度とレイテンシのトレードオフを評価し、構成上の判断を根拠づける / Evaluate accuracy-latency trade-offs and justify configuration decisions
- 大規模環境における可観測性の課題を分析し、監視戦略を選択する / Analyze observability challenges and select monitoring strategies at scale
- 適切なチャンク分割とインデックス戦略を伴う RAG パイプラインを設計する / Design a RAG pipeline with appropriate chunking and indexing strategies
- データの形とクエリのパターンに合わせた検索戦略を適用する / Apply retrieval strategies matched to data shape and query pattern
- 接続プロトコルを評価し、適切な統合機構（MCP、API/CLI、エージェント間連携）を選択する / Evaluate connection protocols and select the appropriate integration mechanism (MCP, API/CLI, agent-to-agent)
- 段階的な探索とモノリシックなコンテキスト戦略を比較評価する / Evaluate progressive discovery vs. monolithic context strategy

**Domain 4: Evaluation, Testing & Optimization (16%)**
- 評価指標（精度、レイテンシ、コスト、安全性、セキュリティ）を定義する / Define evaluation metrics (accuracy, latency, cost, safety, security)
- 複数の方法論を組み合わせた評価データセットとテストフレームワークを設計する / Design evaluation datasets and test frameworks using mixed methodologies
- A/B テストと反復的な改善を実施する / Conduct A/B testing and iterative improvements
- システムの問題（プロンプトの失敗、ハルシネーション、モデルのミスマッチ）を診断する / Diagnose system issues (prompt failure, hallucinations, model mismatch)
- トークン使用量、レイテンシ、コストと性能のトレードオフを最適化する / Optimize token usage, latency, and cost-performance trade-offs
- ロギングと可観測性ツールでシステム性能を監視する / Monitor system performance using logging and observability tools

**Domain 5: Governance, Safety & Risk Management (14%)**
- ガードレールと安全性統制を実装する / Implement guardrails and safety controls
- LLM システムのリスク・限界・失敗モードを特定する / Identify risks, limitations, and failure modes of LLM systems
- human-in-the-loop の検証戦略を適用する / Apply human-in-the-loop validation strategies
- 規制（GDPR、HIPAA、FedRAMP など）への準拠を確保する / Ensure compliance with regulations (e.g., GDPR, HIPAA, FedRAMP)
- 倫理的な AI の考慮事項（バイアス、公平性、透明性）に対処する / Address ethical AI considerations (bias, fairness, transparency)

**Domain 6: Stakeholder Communication & Lifecycle Management (14%)**
- 構造化されたディスカバリーと要件収集を実施する / Conduct structured discovery and requirement gathering
- アーキテクチャ上の決定とトレードオフを伝える / Communicate architectural decisions and trade-offs
- ステークホルダーのフィードバックループと期待値の調整（SLA を含む）を管理する / Manage stakeholder feedback loops and expectation alignment (including SLAs)
- アーキテクチャを文書化し、実装のガイダンスを提供する / Document architectures and provide implementation guidance
- ライフサイクルの各フェーズ（ディスカバリー、設計、引き継ぎ、監視、反復）を支援する / Support lifecycle phases (discovery, design, handoff, monitoring, iteration)

**Domain 7: Developer Productivity & Operational Enablement (7%)**
- チーム向けに Claude のツールと環境（Claude Code など）を構成する / Configure Claude tools and environments for teams (e.g., Claude Code)
- AI 支援ツールで開発者のワークフローを改善する / Improve developer workflows using AI-assisted tooling
- デバッグと運用上の問題解決を支援する / Support debugging and operational issue resolution

---

## 形式 / Format

> **本番の出題形式について / About the live item format**
> 公式ガイドによれば、本番は**多肢選択に加えて複数選択（複数の正答を選ぶ設問）も出題され、設問ごとに選ぶ数が明示されます**。本模擬セットは全 210 問が 4 択・単一選択なので、複数選択の練習は Developer セット（[`../developer/`](../developer/)、複数選択を約 20% 含む）で補ってください。
>
> Per the official guide, the live exam includes **multiple-response items as well as multiple-choice, and each item states how many responses to select**. All 210 questions here are 4-option single-answer, so practice multiple-response items with the Developer set ([`../developer/`](../developer/)), which includes about 20%.

- **4 択・単一選択** / 4 options, single answer（本模擬セットの形式。本番の形式は上記を参照）
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
| [4. 評価・テスト・最適化 / Evaluation, Testing & Optimization](./domain4_evaluation.md) | 16% |
| [5. ガバナンス・安全性・リスク管理 / Governance, Safety and Risk Management](./domain5_governance_risk.md) | 14% |
| [6. ステークホルダー折衝とライフサイクル管理 / Stakeholder Communication and Lifecycle Management](./domain6_stakeholder_lifecycle.md) | 14% |
| [7. 開発者生産性と運用イネーブルメント / Developer Productivity and Operational Enablement](./domain7_developer_enablement.md) | 7% |

**Foundations 模擬試験 / Foundations mock exam:** [`../README.md`](../README.md)
