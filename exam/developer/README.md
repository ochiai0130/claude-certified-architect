# Claude Certified Developer — Foundations (CCDV-F) 模擬試験 / Mock Exam

> Anthropic の **Claude Certified Developer – Foundations** 認定（試験コード `CCDV-F`）の**公式 Exam Guide に記載されたブループリントに厳密に沿った**模擬問題集です。全 8 ドメイン・25 サブスキル、**計 240 問**。
>
> A mock exam set built **strictly to the blueprint published in the official Exam Guide** for Anthropic's **Claude Certified Developer – Foundations** certification (exam code `CCDV-F`): eight domains, 25 sub-skills, **240 questions**.

---

## 出典と、何が公式で何が非公式か / Sourcing: what is official and what is not

配点と出題範囲は **Anthropic 発行の公式 Exam Guide（Version 1.0・Effective July 2026）の一次情報**です。本 README のドメイン名・サブスキル名・weight・試験形式は、その PDF の記載をそのまま転記しています。

- 公式 Exam Guide (PDF): [`Claude Certified Developer – Foundations Exam Guide`](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor/6nizmqk8tpzpfjvt6qmmav7rh/public/1783542875/Claude+Certified+Developer+%E2%80%93+Foundations+Exam+Guide.pdf)
- 配布元: Anthropic Partner Academy（Skilljar）。受験登録は Pearson VUE 経由

**一方で、収録されている 240 問はすべて学習目的の独自作成であり、公式の試験問題ではありません。** 実際の試験問題は Anthropic の機密情報であり、受験時に守秘義務契約（NDA）の対象になります。本問題集は公式ブループリントの各サブスキルに対して独自にシナリオを書き起こしたものです。

The weights and objectives below are transcribed from **Anthropic's official Exam Guide (Version 1.0, effective July 2026)** — a primary source. **The 240 questions themselves are independently authored study material and are not actual exam content**, which is Anthropic's confidential property and subject to an NDA at test time.

> 公式ガイドは予告なく改訂されます / The guide is subject to change without notice. 受験前に最新版を確認してください。

---

## 試験形式 / Exam details at a glance

| 項目 / Item | 内容 / Value |
|---|---|
| 資格名 / Credential | Claude Certified Developer – Foundations |
| 試験コード / Exam code | **CCDV-F** |
| 問題数 / Number of items | **53** |
| 出題形式 / Item format | **多肢選択＋複数選択**（設問ごとに選ぶ数を明示）/ Multiple-choice and multiple-response; each item states how many responses to select |
| 制限時間 / Time limit | **120 分 / minutes** |
| 実施方式 / Delivery | Pearson VUE（オンライン監督試験またはテストセンター）/ Proctored |
| 合格点 / Passing score | **720**（100–1,000 スケール）/ Scaled score of 720 on a 100–1,000 scale |
| 受験料 / Exam fee | **$125 USD** |
| 有効期間 / Validity | **12 か月 / months**（期限内なら無料の非監督アセスメントで更新）|
| 結果 / Result reporting | 合否＋スケールドスコア、およびドメイン別の正答率 / Pass-fail with scaled score, plus percent-correct by domain |
| 再受験 / Retake | 1 回目不合格後 14 日、2 回目後 30 日、3 回目後 90 日。12 か月間で最大 4 回 |
| 前提条件 / Prerequisites | なし（受験資格に必須要件はない）/ None required |

---

## 対象者 / Intended audience

公式ガイドが定める「最低限適格な候補者（MQC）」は、**Claude を使ったアプリケーション・エージェント・ワークフローを実際に構築して本番に出す技術者**です。設計だけでなく実装まで担う点が Architect トラックとの違いです。

推奨経験（必須ではありません）:

- ソフトウェアエンジニアリング **1〜5 年**
- Claude または同等の LLM ベースシステムでの実務 **6 か月以上**
- **Python および／または TypeScript** の習熟
- REST API と CLI ツールの実務的な理解
- LLM の基礎、エージェント、コンテキスト管理、MCP の動作理解

対象外とされているのは、非技術者・カジュアル利用者、およびプロンプト作成のみでアプリケーション開発の責任を持たない役割です。

---

## ドメイン構成と本セットの問題数 / Domains and question counts

配点は極端に偏っています。**Applications and Integration の 1 ドメインだけで試験の 3 分の 1（33.1%）**を占め、下位 4 ドメイン（Claude Code・Eval・Security・Tools）の合計を上回ります。本セットは**この比率をそのまま問題数に反映**しているため、問題数の配分がそのまま学習配分の指針になります。

| # | ドメイン / Domain | Weight | 本番相当<br>問数 @53 | 本セット | ファイル |
|---|---|---|---|---|---|
| 1 | Agents and Workflows | 14.7% | 8 | **35** | [`domain1_agents_workflows.md`](./domain1_agents_workflows.md) |
| 2 | Applications and Integration | **33.1%** | 18 | **82** | [`domain2_applications_integration.md`](./domain2_applications_integration.md) |
| 3 | Claude Code | 3.1% | 2 | **7** | [`domain3_claude_code.md`](./domain3_claude_code.md) |
| 4 | Eval, Testing, and Debugging | 2.6% | 1 | **6** | [`domain4_eval_testing_debugging.md`](./domain4_eval_testing_debugging.md) |
| 5 | Model Selection and Optimization | 16.8% | 9 | **40** | [`domain5_model_selection_optimization.md`](./domain5_model_selection_optimization.md) |
| 6 | Prompt and Context Engineering | 11.0% | 6 | **26** | [`domain6_prompt_context_engineering.md`](./domain6_prompt_context_engineering.md) |
| 7 | Security and Safety | 8.1% | 4 | **19** | [`domain7_security_safety.md`](./domain7_security_safety.md) |
| 8 | Tools and MCPs | 10.6% | 6 | **25** | [`domain8_tools_mcps.md`](./domain8_tools_mcps.md) |
| | **計 / Total** | **100%** | **53** | **240** | |

---

## サブスキル別の配点 / Sub-skill weights

公式ブループリントは 2 階層で、8 ドメインが **25 のサブスキル**に分割され、それぞれに weight が付いています。各問には対象サブスキルをタグ付けしてあるので、**弱点をサブスキル単位で特定**できます。

| ドメイン | サブスキル / Sub-skill | Weight | 本セット |
|---|---|---|---|
| 1 | Agent Architecture | 4.5% | 11 |
| 1 | Agent Construction with Claude | 5.3% | 13 |
| 1 | Agent Patterns and Frameworks | 4.9% | 11 |
| 2 | Understanding Requirements | 3.4% | 8 |
| 2 | Systems Life Cycle | 2.8% | 7 |
| 2 | Claude API Mechanics | 6.8% | 17 |
| 2 | Software Engineering Foundations | 7.4% | 18 |
| 2 | **Claude Application Design** | **8.6%** | 22 |
| 2 | Configuration Management | 4.1% | 10 |
| 3 | Claude Code Operation | 3.1% | 7 |
| 4 | Debugging and Error Handling | 2.6% | 6 |
| 5 | LLM Fundamentals | 5.2% | 12 |
| 5 | Technical Fundamentals | 6.1% | 15 |
| 5 | Model Selection and Tradeoffs | 2.7% | 6 |
| 5 | Cost and Token Management | 2.8% | 7 |
| 6 | Context Engineering | 3.8% | 9 |
| 6 | Prompt Engineering | 4.6% | 11 |
| 6 | Output Handling | 2.6% | 6 |
| 7 | AI Application Security | 3.2% | 7 |
| 7 | Guardrails and Safe Deployment | 2.3% | 5 |
| 7 | Claude Hooks | 1.0% | 3 |
| 7 | Identity, Secrets, and Key Management | 1.6% | 4 |
| 8 | Tool Implementation | 4.4% | 10 |
| 8 | MCP Server Development | 2.1% | 5 |
| 8 | Agentic Customization | 4.1% | 10 |

**Claude Application Design (8.6%) が単一サブスキルとして最大**です。Claude が各インターフェース（Claude Code / Desktop / claude.ai / API / SDK）で指示をどう解釈するか、コンテンツ境界、スキーマ設計、セッション衛生、プラグイン管理が範囲です。

---

## 本セットの構成 / How this set is organized

各ドメインファイルは 2 節に分かれています。

| 節 | 位置づけ | 全体の問数 |
|---|---|---|
| **基礎 / Foundations level** | 公式サンプル問題と同じ認知レベル。実際の受験対策の中核 | 161 |
| **発展 / Advanced** | 規制制約・スケール・トレードオフを含む上級シナリオ。合格ラインに余裕を作るための追加演習 | 79 |

**設問形式:** 公式の item format に合わせ、単一選択（選択肢 A–D）と複数選択（選択肢 A–E、「2 つ選択してください」等を明示）を混在させています。全体の約 2 割が複数選択です。

各問の構成は既存の Architect セットと同じです。

```
## 問題 N / Question N
> サブスキル / Sub-skill: Claude API Mechanics (6.8%)
**シナリオ / Scenario:**   … 日本語 → 英語
**設問 / Question:**       … 日本語 → 英語（複数選択なら選ぶ数を明示）
- A) 〜 - D)（単一選択）または - A) 〜 - E)（複数選択）
<details> 正解と解説（正解の理由＋各誤答が不適切な理由）＋ 参照 </details>
```

---

## 推奨される使い方 / How to Use

1. **配点順に取り組む** — 2 (33.1%) → 5 (16.8%) → 1 (14.7%) → 6 (11.0%) → 8 (10.6%) → 7 (8.1%) → 3 (3.1%) → 4 (2.6%)
2. ドメイン 2 だけで試験の 3 分の 1 を占めるため、**ここに最も多くの時間を割く**。逆にドメイン 3・4 は合わせて 5.7%（本番で 3 問程度）なので、深追いしない
3. まず各ドメインの **基礎** 節を通し、8 割を超えてから **発展** 節に進む
4. 各問の `<details>` を開く前に自分の回答を決める。複数選択問題は「選ぶ数」を必ず守る
5. 誤答はサブスキルのタグを控えておき、同じサブスキルの問題を横断して復習する
6. 公式ガイドの Section 6 を読み、各サブスキルの記述に対して自己評価する

**合格ラインの目安:** 本番は 720/1000。本セットでは、**基礎節で各ドメイン 8 割以上**を安定して取れることを目標にしてください。

---

## Architect トラックとの違い / How this differs from the Architect track

| | **Developer (CCDV-F)** | Architect (CCAR-F / CCAR-P) |
|---|---|---|
| 問われること | **実装して出荷できるか** — API 統合、エージェント構築、ツール／MCP 実装、Claude Code 運用 | 設計判断とトレードオフ、組織・規制への責任 |
| 前提スキル | Python / TypeScript、REST API、CLI | アーキテクチャ設計、ステークホルダー折衝 |
| 特徴的な領域 | Software Engineering Foundations、Claude API Mechanics、Configuration Management | Governance & Risk、Stakeholder Communication |
| 受験料 | $125 | $125（Foundations）／ $175（Professional）|
| 問題数 | 53 | — ／ 63（Professional）|

Anthropic の認定は 4 資格構成です: **Claude Certified Associate – Foundations** / **Claude Certified Developer – Foundations** / **Claude Certified Architect – Foundations** / **Claude Certified Architect – Professional**。Developer は Foundations 階層のみで、Professional 階層は現時点で Architect トラックにのみ存在します。

**このリポジトリの他の模擬試験:**

- Architect Foundations（上級レベル・5 ドメイン 150 問）: [`../README.md`](../README.md)
- Architect Professional（7 ドメイン 210 問）: [`../professional/README.md`](../professional/README.md)

---

## 目次 / Index

| ドメイン / Domain | Weight | 問数 |
|---|---|---|
| [2. Applications and Integration](./domain2_applications_integration.md) | 33.1% | 82 |
| [5. Model Selection and Optimization](./domain5_model_selection_optimization.md) | 16.8% | 40 |
| [1. Agents and Workflows](./domain1_agents_workflows.md) | 14.7% | 35 |
| [6. Prompt and Context Engineering](./domain6_prompt_context_engineering.md) | 11.0% | 26 |
| [8. Tools and MCPs](./domain8_tools_mcps.md) | 10.6% | 25 |
| [7. Security and Safety](./domain7_security_safety.md) | 8.1% | 19 |
| [3. Claude Code](./domain3_claude_code.md) | 3.1% | 7 |
| [4. Eval, Testing, and Debugging](./domain4_eval_testing_debugging.md) | 2.6% | 6 |

（配点順に並べています / Ordered by weight）
