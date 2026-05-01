# Domain 3: Claude Code の設定とワークフロー / Claude Code Configuration and Workflows

> 配点比率 / Weight: **20%**
> 問題数 / Questions: **5**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- CLAUDE.md 階層と `@path` 構文・`.claude/rules/` の `paths` フロントマター / CLAUDE.md hierarchy, `@path`, conditional rules
- `.claude/skills/` のフロントマター（`context: fork`, `allowed-tools`） / Skills frontmatter
- Plan モードと人間承認ゲート / Plan mode and human approval gates
- CI/CD 統合（`-p`, `--output-format json`, `--json-schema`） / CI/CD integration
- スラッシュコマンド・スキル・サブエージェントの責務分離 / Slash commands vs skills vs subagents

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

300 万行・サービス 80 個のモノレポで、3 つのチームが Claude Code を使っています：

A monorepo of 3M lines / 80 services is shared by three teams using Claude Code:

- Payments チーム — `services/payments/**` で動作。PCI DSS のため `console.log(card.*)` 禁止 / Payments team — operates under `services/payments/**`. PCI DSS forbids `console.log(card.*)`
- Health チーム — `services/health/**`。HIPAA のため PHI ロギング禁止 / Health team — `services/health/**`. HIPAA forbids PHI logging
- Platform チーム — `infra/**`。全社共通で SemVer + Conventional Commits / Platform team — `infra/**`. Org-wide SemVer + Conventional Commits

ルールが衝突せず、各ファイル編集時に**該当チームの規約のみ**が読み込まれる構成にしたい。

You want each file edit to load **only the relevant team's rules**, with no cross-team conflicts.

**設問 / Question:**

最も適切な設定階層はどれですか？

Which configuration layout is most appropriate?

- A) ルートの 1 つの `CLAUDE.md` に Payments / Health / Platform の全ルールを書き、モデルに「該当箇所のみ守れ」と指示する / Put all three teams' rules in a single root `CLAUDE.md` and instruct the model to apply only relevant ones
- B) 各チームメンバーが自分の `~/.claude/CLAUDE.md` に自チームのルールだけ書く / Each member puts only their team's rules in `~/.claude/CLAUDE.md`
- C) Payments / Health / Platform を 3 つの別リポジトリに分割する / Split into three separate repositories
- D) ルート `CLAUDE.md` には全社共通ルールのみを記載し、`@infra/CLAUDE.md` で Platform 規約を参照。`.claude/rules/payments.md` と `.claude/rules/health.md` をフロントマターで `paths: ["services/payments/**"]` と `paths: ["services/health/**"]` に絞り、対象ファイル編集時のみ自動読み込みされる構成にする / Root `CLAUDE.md` holds org-wide rules and uses `@infra/CLAUDE.md` for Platform. Place `.claude/rules/payments.md` and `.claude/rules/health.md` with frontmatter `paths: [...]` so they auto-load only when relevant files are edited

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

`.claude/rules/*.md` の **`paths` フロントマター**は、編集対象ファイルがマッチするときのみ自動読み込みされる仕組みで、モノレポでのチーム別規約を **コンテキスト汚染なく** 重ね合わせる正攻法です。`@path` 構文でディレクトリ別の `CLAUDE.md` を取り込めば、Platform 領域固有の SemVer 規約のみを `infra/**` 配下で有効化できます。これにより Payments のファイルを編集している最中に HIPAA ルールでコンテキストが膨らむことを防ぎます。

`paths` frontmatter on `.claude/rules/*.md` makes a rule **conditionally load** only when the edited file matches its glob — the canonical way to layer team-specific rules in a monorepo without context pollution. `@path` imports a per-directory `CLAUDE.md` so Platform's SemVer rules apply only under `infra/**`.

- **A 不正解**: 単一 CLAUDE.md は全ルールを毎回コンテキストに入れ、トークン浪費と相互干渉（PCI のルールが医療コードに誤適用、等）を生みます。 / One CLAUDE.md wastes tokens and causes cross-domain rule leakage.
- **B 不正解**: 個人の `~/.claude/CLAUDE.md` は他人の作業時に効きません。チーム規約はリポジトリに版管理する必要があります。 / Personal user-scope rules don't apply to teammates and lack version control.
- **C 不正解**: モノレポを捨てるのは過剰反応で、共通変更のレビュー・テストが分散して悪化します。 / Splitting the repo is over-reaction and harms cross-cutting changes.

**参照 / Reference:** `guide_ja.md` 「4.1 CLAUDE.md 階層」「4.2 .claude/rules/ の paths フロントマター」「@path 構文」
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

SOX 上場企業の Java 決済システムで、Claude Code が大規模リファクタリング（120 ファイル変更）を生成しました。次のステップで **同じ Claude セッション** が「コードレビュー & 統制チェック」も担当しようとしています。内部統制要件で「**生成と検証は分離されたインスタンス**」が要求されています。

In a SOX-compliant Java payments system, Claude Code generated a 120-file refactor. The next step is "code review + control check" — and the **same** session is poised to do it. Internal controls require **separation of generation and verification**.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) 同じセッションで `/review` スラッシュコマンドを実行し、システムプロンプトに「客観的に評価せよ」と書き加える / Run `/review` in the same session with a system-prompt instruction to "evaluate objectively"
- B) `.claude/skills/code-review.md` を作成し、フロントマターで `context: fork` を設定して**新しい独立コンテキスト**で起動。`allowed-tools` を Read / Grep / Glob のみに絞り（変更不可）、レビュアーが生成側のコンテキストや会話履歴を見られないようにする / Create `.claude/skills/code-review.md` with `context: fork` so it launches in a **fresh isolated context**, and restrict `allowed-tools` to Read / Grep / Glob (no write) so the reviewer cannot see the generator's context or history
- C) 別の人間レビュアーをアサインする（自動化を諦める） / Abandon automation; assign a human reviewer
- D) `--resume` で古いセッションを復元してレビューさせる / Use `--resume` to restore the older session for review

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

スキルの `context: fork` は **新しいクリーンなコンテキスト**でスキルを起動するため、生成プロセスの「自己正当化バイアス」（自分が書いたコードを自分でレビューすると見落としやすい）を排除できます。`allowed-tools` を読み取り系に絞ることでレビュアーが副作用を起こさないことを担保し、SOX の "Segregation of Duties" 統制要件を満たします。

`context: fork` launches the skill in a **fresh, isolated context**, eliminating the self-justification bias of reviewing your own work in the same session. Locking `allowed-tools` to read-only enforces SOX-style Segregation of Duties at the system layer.

- **A 不正解**: 同じセッション内では生成時のコンテキストと推論連鎖を引きずり、客観性が出ません。プロンプトでは強制できません。 / Same-session review inherits generator's reasoning bias and cannot be enforced by prompt.
- **C 不正解**: 自動化の利点を捨てる必要はなく、独立インスタンスで充分代替できます。 / Independent instances achieve the goal without abandoning automation.
- **D 不正解**: `--resume` は古いコンテキストを復元してしまい、独立性が逆方向です。 / `--resume` restores prior context — opposite of what's needed.

**参照 / Reference:** `guide_ja.md` 「4.4 スキルとコマンド」「context: fork による独立レビュー」
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

eコマースの単一データベース（MySQL 8）から **マイクロサービス分離** を行う 6 ヶ月プロジェクトがあり、初回フェーズで 45 ファイル超のスキーマ・コード変更が発生します。CTO は「**実装前に方針を確認したい**」「ロールバック可能性を担保したい」と要求しています。CI 上の自動マージは禁止です。

A 6-month project to split a MySQL 8 monolith into microservices begins with a 45+ file schema-and-code change. The CTO requires architectural review before implementation and rollback safety. Automated CI merges are not allowed.

**設問 / Question:**

Claude Code の最も適切な使い方はどれですか？

Which is the most appropriate use of Claude Code?

- A) **Plan モードで実装方針を作成 → CTO とアーキテクトが承認 → `ExitPlanMode` で実装フェーズへ移行 → 各ステップ後に `/compact` で要約と人間チェックポイントを挟む**。データベース変更は前方互換マイグレーション（expand → migrate → contract）に分割し、各段階で人間承認を得る / Use **Plan mode** to draft the approach, get CTO/architect approval, transition to execution via `ExitPlanMode`, and insert human checkpoints / `/compact` between steps. Split DB changes into expand → migrate → contract with approval at each stage
- B) いきなり実装モードで全変更を行い、最後に diff をレビューする / Implement everything in execution mode and review the diff at the end
- C) Claude Code を使わず、すべて手作業で実装する / Skip Claude Code entirely and do it manually
- D) `claude-haiku-4-5` で高速反復し、最後にまとめて承認を取る / Iterate quickly on `claude-haiku-4-5` and obtain approval at the end

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

Plan モードは **実装前にアプローチを文書化し人間承認を得る** ためのフェーズで、後戻りコストの大きい変更（マイクロサービス分離・スキーマ変更）に最適です。`ExitPlanMode` は明示的承認後の実装移行を表し、監査トレースになります。データベースは expand → migrate → contract の三段階に分割し、各段階で人間承認を挟むことで、いつでもロールバック可能な状態を維持できます。

Plan mode **documents the approach for human approval before any implementation**, ideal for high-blast-radius changes (microservice extraction, schema changes). `ExitPlanMode` is an explicit transition that creates an audit trail. Splitting DB changes into expand → migrate → contract with approval at each stage preserves rollback safety throughout.

- **B 不正解**: 45 ファイル一括変更を後でレビューは現実的でなく、ロールバックも困難。 / Reviewing 45 files post-hoc is impractical and rollback is hard.
- **C 不正解**: ツールを捨てる必要はなく、Plan モードで適切に運用すれば自動化の恩恵を得られます。 / No need to discard tooling; Plan mode handles this safely.
- **D 不正解**: モデル選択は本質ではなく、承認ゲートのなさが致命的。 / Model choice doesn't address the missing approval gates.

**参照 / Reference:** `guide_ja.md` 「4.5 Plan モード」「ExitPlanMode の運用」
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

GitHub Actions で、PR ごとに Claude Code が **「破壊的変更検出 → 影響範囲レポート生成 → reviewers 自動アサイン」** を実行します。レポートは下流のスクリプトが JSON でパースし、CODEOWNERS と突合してレビュアーを決定します。レポートのフィールドが欠けたり型が崩れたりすると CI 全体が落ち、夜間デプロイが遅延します。

In GitHub Actions, Claude Code runs on each PR to **detect breaking changes → emit an impact report → auto-assign reviewers**. A downstream script parses the JSON, joins with CODEOWNERS, and assigns reviewers. Missing fields or type drift breaks the entire CI and delays nightly deploys.

**設問 / Question:**

最も信頼性の高い実装はどれですか？

Which is the most reliable implementation?

- A) `claude` を引数なしで起動し、生成された Markdown を正規表現でパースする / Invoke `claude` without flags and parse the generated Markdown via regex
- B) `claude -p "$PROMPT" --output-format json --json-schema "$SCHEMA"` を使い、出力スキーマをファイルで定義（必須フィールド・enum・型を厳格化）。失敗時は同じセッションを再試行せず、**指数バックオフで新規セッションをリトライ**し、N 回失敗時は人間にエスカレーション / Use `claude -p "$PROMPT" --output-format json --json-schema "$SCHEMA"` with a strict schema (required fields, enums, types). On failure, retry **with a new session and exponential backoff**; after N failures, escalate to a human
- C) 出力を YAML にして、エラー時は `null` を返すよう指示 / Use YAML output and instruct the model to emit `null` on errors
- D) Claude Code を CI に組み込まず、ローカル実行のみにする / Don't integrate Claude Code into CI; run locally only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

CI/CD では **構造化出力の確実性**が信頼性の根幹です。`-p`（print・非対話モード）+ `--output-format json` + `--json-schema` で出力スキーマを **API レイヤーで強制**することで、欠損フィールドや型崩れを根絶します。失敗時は **同じセッションでの再試行**は同じ誤りを繰り返しがちなので、**新規セッション + 指数バックオフ + 上限回数でエスカレーション**が定石です（Idempotency の観点でも同じ）。

For CI/CD, **deterministic structured output** is the core of reliability. Combining `-p` (non-interactive), `--output-format json`, and `--json-schema` enforces the contract at the API layer, eliminating missing-field and type-drift bugs. On failure, retry with a **fresh session and exponential backoff** (same-session retries tend to repeat errors), capped with human escalation.

- **A 不正解**: Markdown を正規表現でパースするのは脆弱で、フォーマットの揺れに耐えられません。 / Regex-on-Markdown is brittle and breaks on format variation.
- **C 不正解**: YAML はスキーマ強制が弱く、`null` を返す指示は確率的で監査要件を満たしません。 / YAML lacks strict schema enforcement; "emit null" is probabilistic.
- **D 不正解**: 自動化の価値を捨てるのは過剰反応。スキーマ強制で安全に統合可能です。 / Avoiding CI integration sacrifices automation value unnecessarily.

**参照 / Reference:** `guide_ja.md` 「4.6 CI/CD 統合」「-p と --output-format json」「--json-schema」
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

24x7 オンコール体制の SaaS で、インシデント対応中に Claude Code を活用したい。次の 3 用途があります：

A 24x7 SaaS team wants to use Claude Code during incident response for three use cases:

(α) よく使う 1 行コマンド「現在の P0 インシデントの Slack スレッドを取得」 / Frequent one-liner — "fetch current P0 Slack thread"
(β) 「障害報告書（RCA）テンプレートを過去のインシデント DB から構築」 — 引数：インシデント ID。共有・チェックイン可能・改訂履歴必要 / "Build RCA from past incident DB" — takes an incident ID; must be shared, checked in, and versioned
(γ) 「ログ全文 50GB を解析して原因仮説を 3 つ立てる」 — メインコンテキストを汚染したくない / "Analyze 50GB of logs and produce 3 hypotheses" — must not pollute the main context

**設問 / Question:**

最も適切な責務分離はどれですか？

Which responsibility allocation is most appropriate?

- A) 3 つともスラッシュコマンドで実装 / All three as slash commands
- B) 3 つともサブエージェントで実装 / All three as subagents
- C) (α) スラッシュコマンド（簡潔・即時）、(β) スキル（フロントマター付き・引数・チームでバージョン管理）、(γ) サブエージェント（`Task` で起動して別コンテキストで処理し、要約のみメインに戻す） / (α) Slash command (concise, instant), (β) Skill (frontmatter, args, team-versioned), (γ) Subagent (launched via `Task` in a separate context, returning only a summary)
- D) (α) スキル、(β) サブエージェント、(γ) スラッシュコマンド / (α) Skill, (β) Subagent, (γ) Slash command

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

責務の使い分けは次が原則です：

The principle is:

- **スラッシュコマンド (Slash command)** = 簡潔で即時的なテンプレート化された呼び出し。引数なし・最小ロジック。 / Concise instant template invocations.
- **スキル (Skill)** = フロントマター（`argument-hint`、`allowed-tools`、`context: fork` 等）で構造化され、チームで共有・版管理される手続き。 / Structured procedure with frontmatter, argument hints, allowed-tools — team-versioned and reusable.
- **サブエージェント (Subagent)** = 大量データやマルチステップ処理を **別コンテキストで分離**し、メインのコンテキスト劣化を防ぐ。`Task` ツール経由で起動。 / Separate context for heavy data or multi-step work; preserves main context.

(α) は即時の固定動作 → スラッシュコマンド。(β) は引数 + 共有 + 改訂 → スキル。(γ) は 50GB のコンテキスト汚染回避 → サブエージェント、が正解。

- **A 不正解**: スラッシュコマンドは引数や `allowed-tools` の柔軟性が低く、(β) や (γ) には不向き。 / Slash commands lack the flexibility for (β)/(γ).
- **B 不正解**: 単純な (α) にサブエージェントは過剰でレイテンシが悪化。 / Subagents are overkill for simple (α) and add latency.
- **D 不正解**: 役割が逆。50GB ログをスラッシュコマンドにするとメイン汚染必至。 / Inverted; 50GB logs in a slash command pollute main context.

**参照 / Reference:** `guide_ja.md` 「4.3 スラッシュコマンド」「4.4 スキル」「3.4 サブエージェント (Task)」
</details>

---

> **前のドメイン / Previous:** [`domain2_tool_design_mcp.md`](./domain2_tool_design_mcp.md) | **次のドメイン / Next:** [`domain4_prompt_structured_output.md`](./domain4_prompt_structured_output.md)
