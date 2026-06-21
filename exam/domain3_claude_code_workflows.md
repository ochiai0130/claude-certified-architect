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

## 問題 6 / Question 6

**シナリオ / Scenario:**

新しい開発者が Claude Code のセッションで `rm -rf node_modules` を実行しようとしたら、毎回確認プロンプトが出ます。チーム全員でこのコマンドを許可したい一方、`rm -rf /` は絶対に許可したくない。

A new developer keeps getting permission prompts for `rm -rf node_modules`. The team wants to allow it but **never** `rm -rf /`.

**設問 / Question:**

最も適切な設定はどれですか？ / Best configuration?

- A) `.claude/settings.json` の `permissions.allow` に `Bash(rm -rf node_modules)` を追加し、`permissions.deny` に `Bash(rm -rf /*)` を追加する。プロジェクト共有 settings として版管理 / In `.claude/settings.json`, add `Bash(rm -rf node_modules)` to `permissions.allow` and `Bash(rm -rf /*)` to `permissions.deny`. Version-control as project settings
- B) `~/.claude/settings.json`（ユーザースコープ）に許可を追加 / Add the allow to `~/.claude/settings.json` (user scope)
- C) すべての Bash コマンドを許可する / Allow all Bash commands
- D) 開発者が個別に許可するしかない / Each developer must approve individually

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

権限制御は **プロジェクト `.claude/settings.json`** に **`allow` と `deny` の組み合わせ**で記述。版管理することでチーム全員に同じ権限ポリシーが適用され、危険コマンドは明示的にブロック。

Permissions go in **project `.claude/settings.json`** as **`allow` + `deny` rules**, version-controlled so the whole team shares them; dangerous commands are explicitly blocked.

- **B 不正解**: ユーザースコープはチーム共有にならない。 / Not shared with the team.
- **C 不正解**: 全許可は最低の選択。 / Worst choice.
- **D 不正解**: 個別承認は運用負荷。 / Operational burden.

**参照 / Reference:** Claude Code permissions
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

エージェントが書き込んだファイルや実行したコマンドの監査ログが必要です。コンプライアンス上、Claude Code の動作を確実に記録する必要があります。

You need an audit trail of every file write and command executed by the agent for compliance.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) `.claude/settings.json` の `hooks` に `PostToolUse` フックを定義し、`Edit`/`Write`/`Bash` の実行後に **改ざん耐性のある追記専用ストレージ**（WORM, 構造化 JSON ログ）に書き込む。フックは決定論的に実行されるため、モデルが「忘れる」ことが構造的に不可能 / Define a `PostToolUse` hook in `.claude/settings.json.hooks` that writes to **append-only tamper-evident storage** (WORM, structured JSON) after `Edit`/`Write`/`Bash`. Hooks run deterministically — cannot be skipped by the model
- B) Claude にログを取るよう指示 / Instruct Claude to log
- C) ログを取らない / No logs
- D) `claude` コマンドの stdout を tee する / Just tee the `claude` stdout

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

監査要件は **`PostToolUse` フックで決定論的に**実装。プロンプト指示は確率的、stdout はモデルの自由記述で構造化に不向き。

Audit requirements use **deterministic `PostToolUse` hooks**. Prompts are probabilistic; stdout lacks structure.

- **B 不正解**: 確率的。 / Probabilistic.
- **C 不正解**: 監査不可。 / Not auditable.
- **D 不正解**: 構造化されない。 / Unstructured.

**参照 / Reference:** Claude Code hooks
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

Claude Code を使う開発チームで、毎回 `npm test` を打つのが面倒。`/test` で素早く実行したい。引数として特定のテストファイル名を渡せるようにしたい。

A team wants `/test` slash command for quick `npm test` invocation, optionally with a specific test file argument.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) `.claude/commands/test.md` を作成し、本文を「Run `npm test {{args}}`」のように記述。`{{args}}` でユーザー引数を取得 / Create `.claude/commands/test.md` with body like `Run npm test {{args}}`; `{{args}}` captures user arguments
- B) `.claude/skills/test.md` を作成し、フロントマターで `argument-hint: "[test file]"` と `allowed-tools: Bash` を設定。本文に test 実行手順を記述。チーム共有・バージョン管理可能 / Create `.claude/skills/test.md` with frontmatter `argument-hint: "[test file]"` and `allowed-tools: Bash`; body describes the procedure. Team-shared and versioned
- C) 開発者全員が手で打つ / Everyone types manually
- D) `~/.bashrc` にエイリアスを追加 / Add a `~/.bashrc` alias

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

現行の Claude Code では **スキル**（`.claude/skills/`）が、フロントマター（`argument-hint`, `allowed-tools`, `context: fork` 等）を持つ構造化された再利用可能手順の標準。

Modern Claude Code uses **skills** (`.claude/skills/`) — structured, reusable procedures with frontmatter (`argument-hint`, `allowed-tools`, `context: fork`).

- **A 不正解**: 旧来のスラッシュコマンド形式は機能が限定的。 / Less expressive.
- **C 不正解**: 自動化の意味を捨てる。 / Defeats automation.
- **D 不正解**: Claude Code 統合がない。 / No integration.

**参照 / Reference:** Skills frontmatter
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

CI で Claude Code を非対話実行したい。プロンプトをファイルから読み、結果を JSON で他ツールに渡します。

You want non-interactive Claude Code in CI: read a prompt from file, output JSON to downstream tools.

**設問 / Question:**

最も適切な起動方法はどれですか？ / Best invocation?

- A) インタラクティブモードで `claude` を起動し expect スクリプトで自動化 / Interactive mode + expect scripts
- B) `claude -p "$(cat prompt.txt)" --output-format json --json-schema "$(cat schema.json)"` のように `-p`（print）モードで起動し、出力をスキーマで強制 / Use `claude -p "$(cat prompt.txt)" --output-format json --json-schema "$(cat schema.json)"` (print mode) with schema-enforced output
- C) GUI で動かして screenshot を取る / GUI + screenshots
- D) Claude Code は CI で動かない / Claude Code doesn't run in CI

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

CI 統合は **`-p` + `--output-format json` + `--json-schema`** が標準。決定論的・パース可能・型安全。

CI integration uses **`-p` + `--output-format json` + `--json-schema`** for deterministic, parseable, type-safe output.

- **A 不正解**: expect は脆弱。 / Brittle.
- **C 不正解**: GUI は CI に不向き。 / Not CI-friendly.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** Claude Code CI integration
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

100 万行のコードベースで、Claude Code に「すべての deprecated 関数を新 API に置き換えて」と頼むと、巨大すぎてコンテキストが破綻します。

In a 1M-line codebase, asking Claude Code to "replace all deprecated function calls with the new API" overwhelms context.

**設問 / Question:**

最も適切な進め方はどれですか？ / Best approach?

- A) 全コードを 1 度にコンテキストに入れる / Load everything at once
- B) `Glob` でまず deprecated 関数の使用箇所を一覧化、次に **サブエージェント（`Task`）** に「ファイル単位で置換 → 1 ファイル分の差分を構造化サマリで返す」を委譲。メインは差分メタデータ集計のみ。さらに `/compact` で進捗管理し、人間チェックポイントを定期挿入 / Use `Glob` to enumerate call sites first; delegate per-file replacement to **subagents (`Task`)** that return structured diff summaries; main aggregates metadata only; use `/compact` for progress tracking with periodic human checkpoints
- C) 諦めて手作業 / Give up and do it manually
- D) `claude-opus-4-6` の長コンテキストで 1 ターン処理 / One-shot via `claude-opus-4-6` long context

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

大規模リファクタは **列挙 → 分割サブエージェント → 構造化集約 → 進捗管理** が定石。

Large-scale refactor: **enumerate → per-task subagents → structured aggregation → progress management**.

- **A 不正解**: 物理的に不可能。 / Physically impossible.
- **C 不正解**: 自動化価値を失う。 / Loses value.
- **D 不正解**: 長コンテキストでも中間消失で精度低下。 / Drifts.

**参照 / Reference:** Subagents・/compact
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

Claude Code の `/init` を新規プロジェクトで実行すると `CLAUDE.md` のテンプレが作られます。チームでこの初期テンプレを統一したい。

`/init` creates a `CLAUDE.md` template; the team wants a unified initial template.

**設問 / Question:**

最も適切な運用はどれですか？ / Best practice?

- A) `/init` は使わず、テンプレファイルをチームの内部リポジトリに置いて手動コピー / Skip `/init`; copy a template from an internal repo manually
- B) `/init` 実行後に出力を確認し、組織共通の項目（コーディング規約、テストコマンド、デプロイ手順、関連ドキュメントへの `@path` リンク等）を **organization template** として管理。リポジトリ初期化スクリプト（`.github/templates/CLAUDE.md`）と組み合わせて自動コピー / After `/init`, augment with org-common items (coding standards, test/deploy commands, `@path` links to related docs) maintained as an **organization template**; pair with repo init scripts (`.github/templates/CLAUDE.md`) for auto-copy
- C) `CLAUDE.md` は使わない / Don't use `CLAUDE.md`
- D) 各開発者が好きに書く / Each dev writes their own

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

組織テンプレ + リポジトリ初期化スクリプトで一貫性を担保。

Org templates + repo init scripts ensure consistency.

- **A 不正解**: `/init` のメリットを捨てる。 / Loses tooling benefits.
- **C 不正解**: `CLAUDE.md` の価値を放棄。 / Throws away value.
- **D 不正解**: 一貫性なし。 / Inconsistent.

**参照 / Reference:** `/init` command・CLAUDE.md
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

`@path` 構文の使い方について、チームで議論があります。

A team debates how to use the `@path` syntax.

**設問 / Question:**

`@path` の正しい用途として最も適切なのはどれですか？ / Best use of `@path`?

- A) ユーザーがプロンプト入力時に `@docs/api.md` のように打ち、ファイル内容を参照させる。`CLAUDE.md` 内でも他の md ファイルをモジュール的に取り込む（例：`@infra/CLAUDE.md`）／ Users type `@docs/api.md` in prompts to reference file contents; in `CLAUDE.md`, import other md files modularly (e.g., `@infra/CLAUDE.md`)
- B) URL を表す / Denotes a URL
- C) 環境変数 / Environment variable
- D) コメント / A comment

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`@path` は **プロンプト内のファイル参照** と **`CLAUDE.md` のモジュール取り込み** の双方で使える Claude Code の標準機能。

`@path` is Claude Code's standard syntax for **file references in prompts** and **modular `CLAUDE.md` imports**.

- **B 不正解**: URL ではない。 / Not URLs.
- **C 不正解**: 環境変数とは別。 / Different concept.
- **D 不正解**: コメントではない。 / Not comments.

**参照 / Reference:** `@path` syntax
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

スキルにフロントマターで `context: fork` を設定すると、本体とは独立したコンテキストでスキルが起動します。これをいつ使うかの判断基準を知りたい。

`context: fork` in skill frontmatter launches in an isolated context. When should it be used?

**設問 / Question:**

`context: fork` を **使うべき** ケースはどれですか？ / When SHOULD `context: fork` be used?

- A) 主セッションのコンテキストを保ちたい大量データ調査・独立レビュー・並列ワークなど、**メインの推論連鎖を汚さずに**作業させたい場合 / Heavy data exploration, independent review, parallel work — anywhere you want **isolation from main reasoning**
- B) 1〜2 行の単純なシェルコマンド / Simple 1-2 line shell command
- C) ユーザーへの応答を即時返す軽量タスク / Fast immediate user response
- D) `context: fork` は実在しない / `context: fork` does not exist

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`context: fork` は **メインを汚さない独立作業**に最適。軽量タスクで使うとオーバーヘッドが大きい。

Use `context: fork` for **isolated heavy work**; overhead hurts lightweight tasks.

- **B 不正解**: 軽量はオーバーヘッド過剰。 / Overhead too high.
- **C 不正解**: 同上。 / Same.
- **D 不正解**: 事実誤認。 / Factually wrong.

**参照 / Reference:** Skills frontmatter
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

Plan モードで作成された計画を、別セッション（あるいは別エンジニア）が実行する運用にしたい。計画は人間レビュー後に確定し、実装は別フェーズで自動・半自動進行。

You want plans created in Plan mode to be executed by a separate session (or engineer) after human review.

**設問 / Question:**

最も適切な運用はどれですか？ / Best practice?

- A) Plan 専用セッションで `ExitPlanMode` を呼び **計画ファイルを保存**（`docs/plans/<id>.md`）。人間レビュー後、別セッションで `@docs/plans/<id>.md` で計画を参照して実行。計画ファイルは PR レビューで承認、実装結果との突合で監査可能 / In a plan-only session, call `ExitPlanMode` and **persist the plan** (`docs/plans/<id>.md`); after human review, a separate session implements with `@docs/plans/<id>.md`. Plans go through PR review and can be reconciled with implementation for audit
- B) Plan モードを使わない / Skip Plan mode
- C) 同じセッションで Plan と実行を一気に / Plan + execute in one session
- D) 計画は記憶のみで実行 / Keep plans in head only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

計画の永続化 + PR レビュー + 別セッション実行で **生成と検証の分離**と **監査トレース**を両立。

Persist plans + PR review + separate execution session for **separation of duties** and **auditability**.

- **B 不正解**: Plan モードの利点を捨てる。 / Throws away value.
- **C 不正解**: 検証分離が壊れる。 / Loses separation.
- **D 不正解**: 監査・再現性ゼロ。 / No audit, no reproducibility.

**参照 / Reference:** Plan mode・ExitPlanMode
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

長時間のコード調査で、Claude Code のコンテキスト消費が気になります。`/compact` と `/clear` の使い分けをチームに教育したい。

During long investigations, context usage matters. You want to teach `/compact` vs `/clear`.

**設問 / Question:**

最も適切な使い分けはどれですか？ / Best distinction?

- A) `/compact` も `/clear` も同じ / They're the same
- B) **`/compact`** = これまでの会話を**要約圧縮**し継続作業のためにコンテキストを縮小（履歴は要約として保持）。**`/clear`** = コンテキストを**完全にリセット**して新規セッション同等にする。長時間調査の途中で文脈を保ったまま圧縮したいときは `/compact`、別タスクに完全切替するときは `/clear` / **`/compact`** = **summarize-compress** prior conversation while keeping continuity; **`/clear`** = **fully reset** context (like a fresh session). Use `/compact` to shrink while preserving thread; `/clear` when switching tasks entirely
- C) `/compact` は危険なので使わない / `/compact` is dangerous; avoid
- D) `/clear` は履歴を物理的に削除する（復元不能） / `/clear` physically deletes history with no recovery

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

`/compact` は要約圧縮、`/clear` は完全リセット。用途で使い分ける。

`/compact` summarizes; `/clear` resets — use case-dependent.

- **A 不正解**: 別物。 / Different.
- **C 不正解**: 危険ではない。 / Not dangerous.
- **D 不正解**: 物理削除の話ではない。 / Not about physical deletion.

**参照 / Reference:** `/compact`・`/clear`
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

複数の独立した実験を Claude Code で並行実施したい。同じリポジトリの異なるブランチで作業する必要があります。

You want to run multiple independent experiments in parallel from the same repo on different branches.

**設問 / Question:**

最も適切な進め方はどれですか？ / Best approach?

- A) 1 つの作業ディレクトリで `git checkout` を繰り返して切り替え / Repeatedly `git checkout` in one workdir
- B) **Git worktree** を使い、ブランチごとに独立した作業ディレクトリを作る。各ディレクトリで別の Claude Code セッションを並行起動でき、ファイルシステム競合なし / Use **`git worktree`**: each branch gets its own working directory; run separate Claude Code sessions per worktree without file-system conflict
- C) リポジトリを複数クローン / Clone the repo multiple times
- D) 並行は不可能 / Parallelism is impossible

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

`git worktree` は同一 .git に対する複数の作業ディレクトリを作る Git 標準機能で、Claude Code の並行作業に最適。

`git worktree` creates multiple working directories from one `.git` — ideal for parallel Claude Code sessions.

- **A 不正解**: 切り替えは並行性を失う。 / No parallelism.
- **C 不正解**: クローンは履歴重複と整合性問題。 / Wasteful + sync issues.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** `git worktree`
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

Claude Code を使う際、特定のファイルパス（例：`secrets/`、`.env*`）には絶対にアクセスさせたくない。秘匿情報の漏洩を防ぐ運用設計を考えています。

You must never let Claude Code touch certain paths (e.g., `secrets/`, `.env*`).

**設問 / Question:**

最も適切な対策はどれですか？ / Best safeguard?

- A) システムプロンプトで「秘匿情報を読むな」と書く / Prompt: "do not read secrets"
- B) `.claude/settings.json` の `permissions.deny` に `Read(secrets/**)`, `Read(.env*)`, `Bash(cat secrets/*)` 等を追加し、**決定論的にブロック**。`.gitignore` と OS レベルファイル権限も併用した多層防御 / Add `permissions.deny` rules in `.claude/settings.json` (e.g., `Read(secrets/**)`, `Read(.env*)`, `Bash(cat secrets/*)`) to **deterministically block**. Combine with `.gitignore` and OS file permissions for defense in depth
- C) ファイルを暗号化して読めなくする（Claude にはわからないように） / Encrypt files via obscurity
- D) 開発者の良心に任せる / Trust developer discretion

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

秘匿情報保護は **`permissions.deny` で決定論的に**ブロック + 多層防御。プロンプトは確率的、obscurity はセキュリティに非ず。

Secret protection uses **deterministic `permissions.deny`** + defense in depth. Prompts are probabilistic; obscurity isn't security.

- **A 不正解**: 確率的。 / Probabilistic.
- **C 不正解**: 業務上必要なファイルが読めなくなる。 / Breaks workflows.
- **D 不正解**: 規制不適合。 / Non-compliant.

**参照 / Reference:** Claude Code permissions
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

`.claude/skills/` と `.claude/agents/` の使い分けに迷っています。両者ともプロジェクトに常駐する手続きです。

A team is unsure when to use `.claude/skills/` vs `.claude/agents/`.

**設問 / Question:**

最も適切な使い分けはどれですか？ / Best distinction?

- A) **スキル** = 手続き的なタスクテンプレ（実行手順・引数・許可ツールを定義し、必要時にユーザー or Claude が呼ぶ）。**サブエージェント** = 別コンテキストで自律的に動く専門役（コード生成・レビュー・調査などの専任。コーディネーターから `Task` で呼び出し）。両者は補完的で、複雑なシステムでは併用 / **Skills** = procedural task templates (steps, args, allowed-tools; invoked on demand). **Subagents** = autonomous specialists in their own context (code-gen, review, investigation; called via `Task` from a coordinator). They're complementary; complex systems use both
- B) スキルとサブエージェントは同じ / They're identical
- C) スキルは UI 用、サブエージェントは CI 用 / Skills for UI, subagents for CI
- D) どちらか一方しか使えない / You can only use one

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

スキル = 手続きテンプレ、サブエージェント = 別コンテキストの専門役。

Skills = procedural templates; Subagents = specialists in isolated context.

- **B 不正解**: 別物。 / Different.
- **C 不正解**: 用途は UI/CI と無関係。 / Not the axis.
- **D 不正解**: 併用可能。 / Can combine.

**参照 / Reference:** Skills vs subagents
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

VS Code の Claude Code 拡張で開発しています。Claude が編集中のファイルを即座に把握できるようにしたい。

Developing with the VS Code Claude Code extension; you want Claude to immediately know the currently edited file.

**設問 / Question:**

最も適切な手段はどれですか？ / Best approach?

- A) IDE インテグレーションで **アクティブなファイル / 選択範囲 / 開いているタブ**などのコンテキストを Claude に自動共有する設定を有効化。`@path` で必要なファイルを明示参照することも可能 / Enable IDE integration so **active file / selection / open tabs** are auto-shared with Claude; also use `@path` to reference specific files explicitly
- B) ファイルパスを毎回手入力 / Always type file paths
- C) IDE は使わない / Don't use the IDE
- D) スクリーンショットを取って Claude に渡す / Take a screenshot and paste

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

IDE 統合 + `@path` でコンテキスト共有を最大化。

IDE integration + `@path` maximize context sharing.

- **B 不正解**: 手入力は非効率。 / Inefficient.
- **C 不正解**: IDE 利点を捨てる。 / Loses value.
- **D 不正解**: スクリーンショットはコンテキスト的に劣る。 / Worse than direct file access.

**参照 / Reference:** Claude Code IDE integration
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

Claude Code に新人が初日から使ってもらいたい。特定のリポジトリ規約（コミットメッセージ・PR 説明・テスト方針）を毎回プロンプトで再説明するのは非効率。

A new hire should use Claude Code on day 1 with the team's repo conventions (commit messages, PR descriptions, test policy).

**設問 / Question:**

最も適切な仕組みはどれですか？ / Best mechanism?

- A) リポジトリの `CLAUDE.md` に規約を記載し、`@docs/contributing.md` 等で詳細を取り込む。新人はクローン直後から自動的に規約を反映した提案を得られる / Put conventions in the repo's `CLAUDE.md`, importing details via `@docs/contributing.md`. From clone-time, the new hire gets convention-aligned suggestions
- B) Slack で都度教える / Tell them in Slack each time
- C) 新人はしばらく Claude Code を使わない / Don't let them use it yet
- D) 規約は不要 / No conventions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`CLAUDE.md` + `@path` インポートで規約をリポジトリに版管理。新人はクローンした瞬間からチームのプラクティスを反映できる。

`CLAUDE.md` + `@path` imports versioned in the repo — onboarding is automatic.

- **B 不正解**: スケールしない。 / Doesn't scale.
- **C 不正解**: 機会損失。 / Lost productivity.
- **D 不正解**: 一貫性なし。 / Chaos.

**参照 / Reference:** CLAUDE.md best practices
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

Claude Code でリリースノート自動生成を試みています。ただし、リリースノートは公式公開物なので **誤字・誤情報・過剰約束** を絶対に出したくない。

Auto-generating release notes — but as a public artifact, **typos, misinformation, and over-promises** are unacceptable.

**設問 / Question:**

最も適切なフローはどれですか？ / Best workflow?

- A) Claude が一気に生成して即公開 / Generate and publish immediately
- B) **生成 → 独立レビュー（`context: fork` のレビュースキルで別コンテキスト）→ 構造化検証（事実根拠の出所マッチング、変更点リストとの照合）→ 人間最終承認**。各段階を CI で自動化し、人間承認なしには公開ステージへ進まない仕組み / **Generate → independent review (review skill with `context: fork`) → structured validation (source grounding, cross-check against changelog) → human final approval**. Automate all stages in CI; nothing reaches publish without human sign-off
- C) Claude には書かせず手書きのみ / Hand-write everything
- D) 誤字は後で気づいたら直す / Fix typos when noticed

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

公開アーティファクトは **生成 / 独立レビュー / 検証 / 人間承認** の多段ゲートで品質保証。

Public artifacts require **generation / independent review / validation / human approval** as multi-stage gates.

- **A 不正解**: 誤情報リスク。 / Misinformation risk.
- **C 不正解**: 自動化価値ゼロ。 / Zero automation value.
- **D 不正解**: ブランド・信頼へのダメージ。 / Brand damage.

**参照 / Reference:** `context: fork`・人間承認ゲート
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

業務 PC への Claude Code 導入で、IT 部門が「ファイルシステムへの全アクセスを許可するのは怖い」と懸念しています。

IT worries about giving Claude Code full file-system access on corporate machines.

**設問 / Question:**

最も適切な緩和策はどれですか？ / Best mitigation?

- A) **サンドボックス**（コンテナ・専用 VM・OS のサンドボックス機能）内で Claude Code を起動。リポジトリ作業ディレクトリ以外のアクセスを OS レベルで制限。`.claude/settings.json` で許可・拒否を併設し、機微パスを `permissions.deny` で多層防御 / Run Claude Code inside a **sandbox** (container, dedicated VM, OS sandbox); restrict access to the repo workdir at OS level. Combine with `permissions.allow`/`deny` in `.claude/settings.json` for defense in depth
- B) Claude Code を使わない / Don't use Claude Code
- C) すべての権限を許可する / Grant all permissions
- D) 開発者の良心に任せる / Trust developers

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

OS レベルのサンドボックス + Claude Code の権限制御で多層防御。

OS-level sandbox + Claude Code permissions = defense in depth.

- **B 不正解**: 過剰反応。 / Overreaction.
- **C 不正解**: 最悪の選択。 / Worst.
- **D 不正解**: 規制不適合。 / Non-compliant.

**参照 / Reference:** サンドボックス・権限制御
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

Claude Code の進捗を可視化したい。マルチステップタスクで「今どこまで進んだか」が見えづらい。

You want visibility into Claude Code's progress on multi-step tasks.

**設問 / Question:**

最も適切な手段はどれですか？ / Best approach?

- A) `TodoWrite` ツールでタスクリストを宣言し、各ステップ完了時に更新。ユーザーは進行状況を一目で把握、エージェント自身も次のアクションが明確になり脱線が減る / Have the agent declare tasks via `TodoWrite` and update statuses as it proceeds. Users see progress at a glance; the agent stays focused, reducing drift
- B) ログを grep する / Grep logs
- C) 進捗は不要 / Don't bother
- D) ユーザーが何度も「進捗は？」と聞く / Manually ask repeatedly

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`TodoWrite` は **進捗の自己宣言と更新** により、UX 改善とエージェントの集中維持の両方を実現。

`TodoWrite` provides **self-declared tracked progress**, improving UX and keeping the agent focused.

- **B 不正解**: log grep は事後分析。 / Post-hoc only.
- **C 不正解**: 透明性ゼロ。 / Zero transparency.
- **D 不正解**: UX 過剰。 / Annoying UX.

**参照 / Reference:** `TodoWrite`
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

複数のスキルがあり、優先度や条件によって発動するべきものが変わります。`update-config`, `init`, `review` などはどう発動するべきか。

Multiple skills exist; which fires depends on context (`update-config`, `init`, `review`, etc.).

**設問 / Question:**

スキルの **発動契機** として最も適切なのはどれですか？ / Best triggering?

- A) **スキルのフロントマター（description / トリガー条件）**を明確に記述し、Claude がユーザーの意図やコンテキストに応じて自動的に最適なスキルを選択。明示的に呼びたい場合はユーザーが `/skill-name` で呼ぶ / Define **clear frontmatter (description / trigger conditions)**; Claude auto-selects the best skill from intent/context. Users invoke explicitly via `/skill-name` when desired
- B) スキルは常に手動で呼ぶ / Always manual invocation
- C) スキルは 1 つだけ持つ / Have only one skill
- D) スキルは使わない / Don't use skills

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

スキルは **フロントマター主導の自動選択** + **明示呼び出し** の両モード。フロントマターを丁寧に書くのが質の鍵。

Skills support both **frontmatter-driven auto-selection** and **explicit invocation**. Quality depends on careful frontmatter.

- **B 不正解**: 自動選択の利点を捨てる。 / Loses automation value.
- **C 不正解**: 制約しすぎ。 / Over-restrictive.
- **D 不正解**: スキル放棄は機能放棄。 / Throws away.

**参照 / Reference:** Skills frontmatter
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

Claude Code の出力フォーマットをユーザーが好みに合わせたい：簡潔なステータス、絵文字なし、コードブロックは Markdown で。

Users want output style customized: terse status, no emojis, code blocks in Markdown.

**設問 / Question:**

最も適切な設定はどれですか？ / Best configuration?

- A) **出力スタイル** または **ユーザー `~/.claude/CLAUDE.md`** に好みを明示記述（「絵文字を使わない」「ステータスは 1 行」「コードは fenced markdown」等）。プロジェクト規約と衝突しないよう、ユーザースコープに置く / Write preferences in **output styles** or **user `~/.claude/CLAUDE.md`** ("no emojis", "1-line status", "fenced markdown"). Keep in user scope to avoid clashing with project rules
- B) Claude Code は出力を変えられない / Output cannot be customized
- C) すべてのプロジェクトに同じ設定を強制 / Force the same setting across all projects
- D) 毎回プロンプトで指示 / Specify in each prompt

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

出力スタイル / ユーザー `CLAUDE.md` で好み記述。プロジェクトと混ぜないのが整理のコツ。

Use output styles / user `CLAUDE.md` — keep separate from project scope.

- **B 不正解**: 事実誤認。 / Wrong.
- **C 不正解**: プロジェクト規約と競合。 / Collides with project.
- **D 不正解**: 毎回繰り返しは非効率。 / Inefficient.

**参照 / Reference:** Output styles
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

`.claude/settings.json` を変更したらその場で反映させたい。再起動なしで設定の試行錯誤をしたい。

You want `.claude/settings.json` changes to apply without restart for fast iteration.

**設問 / Question:**

最も適切な運用はどれですか？ / Best practice?

- A) Claude Code は設定変更を **次回新セッションから反映**。試行錯誤は新セッションで素早くやり、プロジェクト変更は **PR レビュー** を通して版管理。長時間セッション中の設定変更は副作用が読みにくいので避ける / Claude Code applies setting changes **from the next new session**. Iterate quickly with fresh sessions; commit project changes via **PR review**. Avoid mid-session edits — side effects are hard to predict
- B) 設定ファイルは決して変えない / Never change settings
- C) 全員で同じグローバル設定を使う / Force one global setting for everyone
- D) 設定ファイルを編集すると即時にすべての過去セッションが書き換わる / Editing rewrites all past sessions instantly

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

設定変更は新セッションから反映、PR レビューで版管理、セッション中変更は避ける。

Settings apply to new sessions; version-control via PRs; avoid mid-session edits.

- **B 不正解**: 改善を阻害。 / Halts improvement.
- **C 不正解**: 個別性を奪う。 / Loses flexibility.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** Claude Code settings
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

ペネトレーションテストエージェントを構築中。`Bash` ツールで `nmap` や `sqlmap` を実行する必要があるが、間違って本番ネットワークに対して実行されると重大インシデント。

A pentest agent uses `Bash` for `nmap` / `sqlmap`. Accidentally targeting prod is critical.

**設問 / Question:**

最も適切な安全策はどれですか？ / Best safeguard?

- A) `Bash` を **特定パターンのみ許可**（`Bash(nmap target.test.local *)`, `Bash(sqlmap -u http://staging.* *)` 等）し、本番ホストへの実行を `permissions.deny` で明示ブロック。さらに **隔離ネットワーク**（テスト VLAN）でのみ実行可能にする多層防御 / **Allow only specific patterns** (`Bash(nmap target.test.local *)`, `Bash(sqlmap -u http://staging.* *)`) and explicitly `permissions.deny` production. Combine with **isolated network** (test VLAN) for defense in depth
- B) `Bash` を全許可 / Allow all `Bash`
- C) `Bash` を全禁止 / Deny all `Bash`
- D) システムプロンプトで「本番に向けるな」と指示 / Prompt: "do not target prod"

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

セキュリティテストツールの危険性に対しては **パターン許可 + 本番拒否 + ネットワーク隔離** の多層防御。

Defense in depth = pattern allow + prod deny + network isolation.

- **B 不正解**: 全許可は最悪。 / Worst.
- **C 不正解**: 機能放棄。 / Loses function.
- **D 不正解**: プロンプトは確率的。 / Probabilistic.

**参照 / Reference:** Permissions・network segmentation
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

Claude Code の `PreCompact` フックがあれば、コンテキスト圧縮前に追加処理（重要事実の抽出など）を入れたい。フックの理解を確認します。

`PreCompact` hook fires before compaction; you may want to extract key facts beforehand.

**設問 / Question:**

最も適切な使い方はどれですか？ / Best usage?

- A) `PreCompact` で重要事実を **case facts ファイル**として永続化し、`/compact` 後もコンテキストに復元できるよう設定する。これにより数値や日付など要約で劣化しがちな情報を保護 / Use `PreCompact` to persist key facts to a **case facts file**, restoring them post-compaction so numbers/dates survive
- B) フックは存在しない / Hooks don't exist
- C) `PreCompact` でコンテキストを増やす（圧縮効果を打ち消す） / Use it to bloat context (defeats compaction)
- D) 圧縮前に Claude を停止する / Halt Claude before compaction

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`PreCompact` は要約による情報劣化を防ぐ重要な拡張点。case facts 抽出に最適。

`PreCompact` mitigates summarization drift — ideal for case-fact extraction.

- **B 不正解**: 事実誤認。 / Wrong.
- **C 不正解**: 圧縮効果を打ち消すアンチパターン。 / Anti-pattern.
- **D 不正解**: 停止は不適切。 / Wrong action.

**参照 / Reference:** Hooks・case facts
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

オープンソースプロジェクトで、貢献者が Claude Code を使うかどうかは自由ですが、**プロジェクト共通の規約**は全員が守るべきです。プライベートな個人ルールは個人スコープ。

In an OSS project, using Claude Code is optional, but **project conventions** must be honored. Personal rules stay personal.

**設問 / Question:**

最も適切なスコープ設計はどれですか？ / Best scope design?

- A) プロジェクト規約は **`.claude/settings.json` & `CLAUDE.md`**（リポジトリにコミット）、個人プリファレンスは **`~/.claude/settings.json` & `~/.claude/CLAUDE.md`**。**`.claude/settings.local.json`** はリポジトリ内の個人ローカル設定（gitignore）で、貢献者が共通設定を上書きせずに微調整可能 / Project conventions in **`.claude/settings.json` & `CLAUDE.md`** (committed); personal in **`~/.claude/...`**; **`.claude/settings.local.json`** is gitignored per-clone overrides for tweaks without touching shared config
- B) 全部 1 つのファイルに / Everything in one file
- C) プロジェクト規約はメンバーのみに伝達 / Convey conventions verbally
- D) 設定はしない / No settings

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

3 層スコープ（プロジェクト共有 / 個人 / プロジェクト内ローカル）で OSS の個別事情に対応。

Three-tier scoping (project-shared / personal / repo-local) accommodates OSS variance.

- **B 不正解**: 衝突発生。 / Causes clashes.
- **C 不正解**: スケールしない。 / Doesn't scale.
- **D 不正解**: 一貫性なし。 / Inconsistent.

**参照 / Reference:** Settings scopes
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

長時間動く Claude Code セッションが、特定タイミング（コミット直前・テスト直後）に追加処理を行う必要があります。例えばコミット前に lint 必須、テスト後にカバレッジ抽出。

A long Claude Code session must run extra steps at specific moments (lint before commit, coverage after tests).

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) `.claude/settings.json` の `hooks` で `PreToolUse(Bash(git commit *))` で lint 実行、`PostToolUse(Bash(* test *))` でカバレッジ抽出を **決定論的に**自動実行。フック失敗時はツール実行をブロックし開発者に通知 / Use `.claude/settings.json.hooks`: `PreToolUse(Bash(git commit *))` runs lint, `PostToolUse(Bash(* test *))` runs coverage extraction — **deterministically**. Failures block tool execution and notify the developer
- B) Claude にプロンプトで「忘れずに lint してください」と頼む / Prompt: "remember to lint"
- C) 開発者が手動実行 / Manual execution
- D) フックは不可能 / Hooks are impossible

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

タイミング駆動の追加処理は **`PreToolUse`/`PostToolUse` フック**で決定論的に。失敗ブロックも併用。

Timing-driven extras = **`PreToolUse`/`PostToolUse` hooks** with failure-block semantics.

- **B 不正解**: 確率的、忘れる。 / Probabilistic, forgettable.
- **C 不正解**: 自動化価値ゼロ。 / Zero automation.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** Claude Code hooks
</details>

---

## 問題 31 / Question 31

**シナリオ / Scenario:**

トレーディングデスクの **アルゴ取引コードベース**で Claude Code を導入。本番取引中の事故は数百万ドル規模。コードレビュー・テスト・デプロイのガードレールを設計します。

A trading desk adopts Claude Code for **algo-trading code**; production incidents cost millions. Design code review / test / deploy guardrails.

**設問 / Question:**

最も適切な構成はどれですか？ / Best configuration?

- A) 通常の開発フローで十分 / Normal flow suffices
- B) **多層ガード**：(i) `.claude/settings.json` の `permissions.deny` で `Edit(prod_*/**)`, `Bash(deploy_prod*)` を完全ブロック、(ii) ステージング環境への変更のみ許可、(iii) `.claude/skills/code-review.md` を `context: fork` で独立レビュー実行、(iv) PR には **必ずバックテスト結果**（過去 N 日間の市場データでの再現）を添付するスキル、(v) 本番デプロイは別の人間オペレータの **2-eyes 承認** + リアルタイム P&L 監視 + サーキットブレーカー / **Multi-layered guards**: (i) `permissions.deny` blocks `Edit(prod_*/**)`, `Bash(deploy_prod*)`, (ii) only staging changes allowed, (iii) `.claude/skills/code-review.md` runs `context: fork` independent review, (iv) PRs require backtest attachments via a skill, (v) production deploys need a second human's **two-eyes approval** + real-time P&L monitoring + circuit breaker
- C) Claude Code に本番権限 / Grant production access
- D) Claude Code は使わない / Don't use Claude Code

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

アルゴ取引のコード安全性は **多層 deny + 独立レビュー + バックテスト + 2-eyes + サーキットブレーカー**。

Algo-trading safety = **layered deny + independent review + backtest + two-eyes + circuit breaker**.

- **A 不正解**: 事故コストが大きすぎる。 / Risk too high.
- **C 不正解**: 単一誤操作で大損失。 / Single mistake = huge loss.
- **D 不正解**: 自動化価値喪失。 / Loses value.

**参照 / Reference:** Algo trading safety・two-eyes
</details>

---

## 問題 32 / Question 32

**シナリオ / Scenario:**

銀行の **モデルリスク管理（MRM, SR 11-7）** で、Claude が生成したコードや分析を扱う際の管理体制を設計します。Fed / OCC のモデルリスク基準に準拠する必要あり。

A bank's **Model Risk Management (SR 11-7)** governs Claude-generated code / analysis. Must satisfy Fed / OCC standards.

**設問 / Question:**

最も適切な体制はどれですか？ / Best framework?

- A) **モデルライフサイクル管理**：(i) Claude のプロンプト・モデル版数・出力を **MRM レジストリ**に登録（"AI-assisted artifact" として分類）、(ii) **independent validation team** が `context: fork` で独立検証、(iii) 本番投入前に `model_card` 相当のドキュメント（用途・限界・既知のバイアス・検証結果）作成、(iv) 定期的な **再検証**（市場環境変化に応じて）、(v) すべての Claude 利用は **MRM 監査トレース**に紐付け / **Model lifecycle**: (i) register Claude prompts / model version / outputs in the **MRM registry** (classified as "AI-assisted artifact"), (ii) **independent validation team** uses `context: fork`, (iii) before prod, create a `model_card`-equivalent (use, limits, known biases, validation results), (iv) periodic **revalidation** with market shifts, (v) tie all Claude usage to **MRM audit trail**
- B) MRM 対象外として扱う / Treat outside MRM
- C) Claude 利用を禁止 / Forbid Claude
- D) 監査時だけ申告 / Disclose only at audit

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

MRM SR 11-7 は **登録・独立検証・モデルカード・再検証・監査トレース**を要求。

SR 11-7 = **registration + independent validation + model card + revalidation + audit trail**.

- **B 不正解**: 規制違反。 / Breach.
- **C 不正解**: 過剰反応。 / Overreaction.
- **D 不正解**: 規制不適合。 / Non-compliant.

**参照 / Reference:** SR 11-7・MRM
</details>

---

## 問題 33 / Question 33

**シナリオ / Scenario:**

ヘッジファンドのクオンツチームで、Claude Code を使って研究 → 戦略実装。**戦略の漏洩**は競争優位性の致命傷。社内ローカル開発のセキュリティ設計が必要です。

A quant team uses Claude Code for research → strategy implementation. **Strategy leakage** is fatal to competitiveness. Need secure local-dev posture.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **隔離開発環境**：(i) 専用 VPN / コンテナ / VDI で開発、(ii) Anthropic API は **VPC エンドポイント / 自社専用キー**で接続、(iii) `.claude/settings.json` の `permissions.deny` で外部書き込み（Slack 通知ツール、Public GitHub 等）をブロック、(iv) 戦略コードは git LFS + 専用 Git サーバ、(v) Claude のコンテキストに入る情報は **DLP 監視**、(vi) すべての Claude セッション ID と利用者を **コンプライアンス DB に紐付け** / **Isolated dev**: (i) dedicated VPN / container / VDI, (ii) Anthropic via **VPC endpoint / dedicated key**, (iii) `permissions.deny` blocks external writes (Slack, public GitHub), (iv) strategy code in git LFS + dedicated server, (v) **DLP monitors** what enters Claude's context, (vi) every session ID + user tied to a **compliance DB**
- B) 通常のクラウド開発 / Normal cloud dev
- C) Claude Code を使わない / Don't use Claude Code
- D) 漏洩は気にしない / Ignore leakage

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

戦略 IP 保護は **隔離環境 + プライベート接続 + 外部書き込み禁止 + DLP + 利用者紐付け**。

Strategy IP = **isolated env + private connectivity + external-write deny + DLP + user binding**.

- **B 不正解**: 漏洩リスク。 / Leak risk.
- **C 不正解**: 機会損失。 / Lost gains.
- **D 不正解**: 競争優位性喪失。 / Lost edge.

**参照 / Reference:** Strategy IP protection・DLP
</details>

---

## 問題 34 / Question 34

**シナリオ / Scenario:**

決済プラットフォームで、PCI DSS 監査対応のコードレビューに Claude Code を活用。**カード番号がコードやログに混入していないか**を継続的にチェックしたい。

A payments platform uses Claude Code for PCI DSS audit-prep code reviews; continuously check for **card numbers in code or logs**.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) **`.claude/skills/pci-scan.md`** スキル：①コードベース全体に対して `Grep` で PAN / CVV パターン（`\d{13,19}`、`CVV?\s*[:=]\s*['"]?\d{3,4}`）を検索、②検出時は構造化レポート（ファイル：行：抜粋（マスク済み））を出力、③`PostToolUse` フックで毎回のファイル編集後に **自動 PCI スキャン**を実行、④違反検出時はコミットを **`PreToolUse` フックでブロック**、⑤定期スキルスケジュール（CI cron）で全体チェック / **`.claude/skills/pci-scan.md` skill**: ①`Grep` for PAN / CVV patterns across the codebase, ②structured report on hits (file:line:masked-snippet), ③`PostToolUse` hook auto-runs **PCI scan** after every file edit, ④violations **block commits via `PreToolUse` hook**, ⑤scheduled CI scan
- B) PCI スキャンは別ツール / External PCI tool only
- C) 開発者の良心 / Trust developers
- D) 監査時だけスキャン / Scan only at audit

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

PCI コード監査は **スキル + フック + 自動ブロック + CI 定期スキャン**。

PCI code audit = **skill + hook + auto-block + scheduled CI scan**.

- **B 不正解**: Claude Code 統合の利点を捨てる。 / Loses integration.
- **C 不正解**: 規制不適合。 / Non-compliant.
- **D 不正解**: 後追い、流出済みリスク。 / Reactive, leaks already happen.

**参照 / Reference:** PCI DSS code review・hooks
</details>

---

## 問題 35 / Question 35

**シナリオ / Scenario:**

銀行の **コンプライアンス部門**が、既存 Java コードベースから Basel III の自己資本比率計算ロジックを抽出 → ドキュメント化したい。100 万行超のレガシー。

A bank's compliance dept wants to extract + document Basel III capital ratio logic from a 1M-line legacy Java codebase.

**設問 / Question:**

最も適切なフローはどれですか？ / Best flow?

- A) **`.claude/skills/extract-basel.md`** スキル：①Plan モードで対象パッケージ・クラス候補を探索、②サブエージェント（`Task` + `context: fork`）が領域別（リスク資産計算 / 自己資本算定 / レポート生成）で並列分析、③各サブエージェントの出力を **scratchpad ファイル**（`./compliance/basel_extraction/<area>.md`）に保存、④メインエージェントは構造化サマリのみ集約、⑤コンプライアンス担当者がレビュー後、ドキュメントを最終化、⑥本番コード変更がドキュメントに反映されているか **PostToolUse フック**で常時チェック / **`.claude/skills/extract-basel.md`**: ①Plan mode explores candidate packages/classes, ②subagents (`Task` + `context: fork`) in parallel by area (RWA calc / capital / reporting), ③each writes to **scratchpad files** (`./compliance/basel_extraction/<area>.md`), ④main aggregates structured summaries, ⑤compliance reviews + finalizes docs, ⑥`PostToolUse` hook continuously verifies that production code changes flow into the docs
- B) 1 ファイルずつ Claude に読ませる / One file at a time
- C) Claude Code は不適 / Unsuitable
- D) Word でゼロから手書き / Hand-write in Word

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

大規模レガシー解析は **Plan + 並列サブエージェント + scratchpad + 同期チェック**で堅実に。

Legacy at scale = **Plan + parallel subagents + scratchpad + sync check**.

- **B 不正解**: 文脈劣化、進捗把握不能。 / Drift, no progress.
- **C 不正解**: 機会損失。 / Loses value.
- **D 不正解**: 効率最悪。 / Worst efficiency.

**参照 / Reference:** Legacy analysis・Plan mode・subagents
</details>

---

## 問題 36 / Question 36

**シナリオ / Scenario:**

電子カルテシステムの **HIPAA 準拠開発**で、Claude Code を導入。開発環境にも **本番 PHI を一切持ち込まない**運用。テストデータは合成データを使うが、人為ミスでの本番データ流入を防ぎたい。

EHR development under HIPAA — **no production PHI in dev environments**. Synthetic test data; prevent accidental prod-data ingress.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) (i) **`.claude/settings.json` の `permissions.deny`** で本番 DB / S3 / FHIR エンドポイントへの読み取りを完全ブロック、(ii) `.gitignore` + Git pre-commit フックで PHI らしき値（SSN・MRN・特定氏名パターン）の混入を機械検査、(iii) 開発用 MCP は **合成データソースのみ**接続、(iv) すべての Claude セッションを **データクラス（dev / prod）でタグ付け**し、prod タグのセッションは制限環境でのみ起動可能、(v) PHI スキャナを `PreToolUse` フックで自動実行 / (i) `permissions.deny` blocks reads to prod DB / S3 / FHIR; (ii) `.gitignore` + pre-commit hook detect PHI-shaped values (SSN, MRN, name patterns); (iii) dev MCP only connects to **synthetic data sources**; (iv) **tag every Claude session by data class** (dev/prod); prod-tagged sessions only run in restricted environments; (v) PHI scanner via `PreToolUse` hook
- B) 開発者の注意に任せる / Trust developers
- C) PHI が入っても問題ない / PHI-in-dev is fine
- D) Claude Code 禁止 / Forbid Claude Code

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

HIPAA dev は **多層防御 + データクラスタグ + 合成データ + フックスキャナ**。

HIPAA dev = **defense in depth + data-class tags + synthetic data + hook scanners**.

- **B 不正解**: 規制不適合。 / Non-compliant.
- **C 不正解**: 違法。 / Illegal.
- **D 不正解**: 過剰反応。 / Overreaction.

**参照 / Reference:** HIPAA dev environment
</details>

---

## 問題 37 / Question 37

**シナリオ / Scenario:**

製薬の **臨床試験ソフトウェア** 開発で、FDA 21 CFR Part 11 + GAMP 5 の妥当性確認（CSV / Computer System Validation）が必要。Claude Code が生成したコードもすべて妥当性確認の対象。

For clinical trial software, FDA 21 CFR Part 11 + GAMP 5 require **CSV** (Computer System Validation). Claude-generated code is in scope.

**設問 / Question:**

最も適切な統合はどれですか？ / Best integration?

- A) **CSV 統合スキル**：(i) `.claude/skills/csv-evidence.md` がコード変更ごとに **要件 → 実装 → テスト** のトレーサビリティマトリクスを自動更新、(ii) すべての Claude 出力を WORM ストレージに保存（`session_id`・モデル版・プロンプト・diff）、(iii) PR には **IQ / OQ / PQ テスト結果**を必須添付、(iv) `PreToolUse` フックで本番デプロイを QA 部門の電子署名なしにブロック、(v) **モデル変更時は再検証**（同じ要件 → 同じ結果が出るか） / **CSV integration skill**: (i) `.claude/skills/csv-evidence.md` auto-updates the **requirement → implementation → test** traceability matrix on changes, (ii) all Claude outputs in WORM (session ID, model version, prompt, diff), (iii) PRs require **IQ/OQ/PQ test results**, (iv) `PreToolUse` blocks prod deploy without QA e-signature, (v) **revalidate on model change** (same requirements → same results)
- B) CSV は手動 / Manual CSV only
- C) AI コードは CSV 対象外 / AI code is out of scope
- D) FDA 規制を無視 / Ignore FDA

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

GAMP 5 + 21 CFR Part 11 は **トレーサビリティ + WORM + IQ/OQ/PQ + 電子署名 + モデル変更再検証**。

GAMP 5 + 21 CFR Part 11 = **traceability + WORM + IQ/OQ/PQ + e-sig + revalidation on model change**.

- **B 不正解**: 効率損失。 / Inefficient.
- **C 不正解**: 規制誤解。 / Misreading.
- **D 不正解**: 違法。 / Illegal.

**参照 / Reference:** GAMP 5・CSV
</details>

---

## 問題 38 / Question 38

**シナリオ / Scenario:**

医療機器ソフトウェア（IEC 62304 Class C）の開発に Claude Code を導入。**Class C は患者死亡または重大障害に至り得るソフトウェア**で、最高水準のプロセスが必要。

For medical device software at IEC 62304 Class C (death / serious injury possible), highest process rigor is required.

**設問 / Question:**

最も適切な体制はどれですか？ / Best framework?

- A) **Class C 統合フロー**：(i) Claude は **アシスタント役のみ**（実装提案 / レビュー / テスト案）、(ii) すべての Claude 出力は **`AI_GENERATED` タグ**で管理し、人間レビュー必須、(iii) Plan モードで **設計仕様（DSP）** を必ずドラフト → 規制対応専門家がレビュー、(iv) コードレビューは **2 名の有資格エンジニア**による独立レビュー（うち 1 名は Claude を使わない）、(v) すべての変更は **リスク管理ファイル（ISO 14971）** に紐付け、(vi) リリース前に **完全な再検証** / **Class C flow**: (i) Claude is **assistant only** (proposal / review / test ideas), (ii) tag all output `AI_GENERATED`; mandatory human review, (iii) Plan mode drafts the **Design Specification (DSP)** for regulatory experts to review, (iv) code review by **two qualified engineers** (one not using Claude), (v) every change tied to **risk file (ISO 14971)**, (vi) full revalidation pre-release
- B) 通常開発と同じ / Same as normal dev
- C) Class C で AI 禁止 / Ban AI for Class C
- D) Claude が承認まで実行 / Claude approves to release

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

Class C は **AI アシスタント + タグ管理 + 設計仕様 + 二重独立レビュー + リスクファイル紐付け + 再検証**。

Class C = **AI as assistant + tagging + DSP + dual independent review + risk-file linkage + revalidation**.

- **B 不正解**: 規格違反。 / Standard breach.
- **C 不正解**: 効率の機会損失。 / Lost efficiency.
- **D 不正解**: 致命的。 / Catastrophic.

**参照 / Reference:** IEC 62304 Class C・ISO 14971
</details>

---

## 問題 39 / Question 39

**シナリオ / Scenario:**

病院の **電子カルテ統合プロジェクト**で、HL7 v2 → FHIR R4 への移行コードを Claude Code で生成。マッピングは数百項目で、誤マッピングは患者安全リスク。

A hospital's HL7 v2 → FHIR R4 migration uses Claude Code; hundreds of mappings; misallocations risk patient safety.

**設問 / Question:**

最も適切な進め方はどれですか？ / Best approach?

- A) **段階的検証フロー**：(i) Plan モードで全マッピング表をドラフト、(ii) `Task` で領域別サブエージェント（patient / encounter / observation / medication）が並列で詳細化、(iii) 各サブエージェントの出力に対して **臨床情報学スペシャリスト**が `context: fork` で独立レビュー、(iv) 段階的に **shadow mode**（並行運用で結果差分を比較）→ canary（5% トラフィック）→ 段階拡大、(v) すべてのマッピングを **規制提出用ドキュメント**として版管理 / **Staged validation**: (i) Plan mode drafts the full mapping table, (ii) `Task` subagents in parallel by domain (patient / encounter / observation / medication), (iii) **clinical informaticists** review independently via `context: fork`, (iv) **shadow mode** (compare in parallel) → canary 5% → graduate, (v) mappings as **regulatory submission docs** under version control
- B) 一度に切り替え / Cut over at once
- C) 自動マッピングで信頼 / Trust auto mappings
- D) 移行しない / Don't migrate

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

医療データマイグレーションは **Plan + 並列領域 + 独立レビュー + shadow → canary → 段階拡大 + 文書化**。

Healthcare migrations = **Plan + parallel domains + independent review + shadow → canary → graduation + docs**.

- **B 不正解**: 安全性リスク。 / Safety risk.
- **C 不正解**: 検証なしは不可。 / Insufficient.
- **D 不正解**: 機会損失。 / Loses progress.

**参照 / Reference:** HL7→FHIR migration
</details>

---

## 問題 40 / Question 40

**シナリオ / Scenario:**

医療機関の **個人情報保護対応**で、ログから自動的に PHI を除去するスキルを作成。すべての開発ログ・本番ログ・監査ログから PHI が漏れていないかを検査。

A clinic auto-redacts PHI from dev / prod / audit logs via a skill, and verifies no PHI leaks across them.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **`.claude/skills/phi-redaction.md`**：(i) 検出パターン（SSN・MRN・電話・メール・氏名 NER）、(ii) 検出時は **トークン化置換**（`[PHI-EMAIL-12]`）でログ書き込み、(iii) 元値マッピングは **HIPAA-compliant 暗号化ストア**にのみ保管、(iv) PHI スキャナを CI で全ログに適用、(v) フローでは検出時に **アラート + 自動 redaction + インシデント記録**、(vi) 定期的に過去ログを再スキャン（パターン更新で過去漏洩検出） / **`.claude/skills/phi-redaction.md`**: (i) detection patterns (SSN, MRN, phone, email, NER for names), (ii) replace with **tokenized values** (`[PHI-EMAIL-12]`), (iii) original-value mapping in a **HIPAA-compliant encrypted store**, (iv) CI runs PHI scanner across all logs, (v) on detection: **alert + auto-redact + incident record**, (vi) periodic rescan of past logs (pattern updates surface old leaks)
- B) PHI 検出は手動 / Manual detection
- C) ログを取らない / No logs
- D) PHI を平文で残す / Keep PHI plain

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

PHI 除去は **検出 + トークン化 + 暗号化マッピング + CI スキャン + アラート + 過去再スキャン**。

PHI redaction = **detect + tokenize + encrypted mapping + CI scan + alerts + rescan**.

- **B 不正解**: 規制不適合。 / Non-compliant.
- **C 不正解**: 監査ログは必須。 / Audit logs required.
- **D 不正解**: 違法。 / Illegal.

**参照 / Reference:** HIPAA log redaction
</details>

---

## 問題 41 / Question 41

**シナリオ / Scenario:**

法律事務所のドキュメント自動化で、Claude Code を使って **契約書ドラフト生成**。クライアントごとに異なる **契約テンプレート + 過去契約パターン**があり、適切なテンプレートを選択する必要があります。

Law-firm document automation drafts contracts via Claude Code. Per-client **template + historical patterns** must be selected.

**設問 / Question:**

最も適切な構成はどれですか？ / Best configuration?

- A) **クライアント別スキル + ナレッジ**：(i) `.claude/skills/draft-contract-{client}.md` を顧客別に作成、`argument-hint` で契約種別を受け取る、(ii) 各スキル内で `@clients/{client}/templates/...` でテンプレを参照、(iii) `@clients/{client}/clauses_library.md` で過去契約パターンライブラリ、(iv) `context: fork` で他クライアント情報の漏洩防止、(v) 弁護士が最終承認するまで PDF 化しない仕組み / **Per-client skill + knowledge**: (i) `.claude/skills/draft-contract-{client}.md` per client; `argument-hint` for contract type, (ii) `@clients/{client}/templates/...` for templates, (iii) `@clients/{client}/clauses_library.md` for historical patterns, (iv) `context: fork` to prevent cross-client leakage, (v) no PDF until attorney approves
- B) 1 つの汎用スキル / One generic skill
- C) ドラフトは手書き / Hand-draft
- D) クライアント情報を共有プロンプト / Shared prompt across clients

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

法律事務所は **クライアント別スキル + 隔離 + 弁護士承認**で運用。

Law firms = **per-client skills + isolation + attorney sign-off**.

- **B 不正解**: クライアント間の漏洩リスク。 / Cross-client leakage.
- **C 不正解**: 効率損失。 / Inefficient.
- **D 不正解**: 利益相反 / 守秘義務違反。 / Conflict / breach.

**参照 / Reference:** Per-client knowledge isolation
</details>

---

## 問題 42 / Question 42

**シナリオ / Scenario:**

法務 GRC（Governance, Risk, Compliance）部門が **規制変更を社内ポリシーに反映** する作業を Claude Code で支援。新規制 → ポリシー差分作成 → 関連手順書更新 → 担当者通知。

A GRC team automates **regulatory change → policy diff → procedure update → owner notification** via Claude Code.

**設問 / Question:**

最も適切なフローはどれですか？ / Best flow?

- A) (i) **規制変更を `MCP resource subscription` で常時監視**、(ii) 新規制検出時に `.claude/skills/policy-impact.md` 自動起動、(iii) 影響評価結果を Plan モードで **段階適用案**（A 案：表面のみ変更 / B 案：構造変更 / C 案：全体改訂）として提示、(iv) GRC 担当者が承認した案で **`Edit` ツールで該当ポリシーを更新**（diff として PR 化）、(v) `PostToolUse` フックで担当者リスト（CODEOWNERS）に通知、(vi) すべての変更ログは WORM 保管 / (i) Watch regulatory updates via **`MCP resource subscription`**, (ii) on detection trigger `.claude/skills/policy-impact.md`, (iii) Plan mode produces **graded options** (A surface, B structural, C full rewrite), (iv) GRC owner approves; **`Edit` updates policies** (PR diffs), (v) `PostToolUse` notifies owners (CODEOWNERS), (vi) WORM logs everything
- B) 規制変更を毎月手動チェック / Monthly manual check
- C) ポリシーは更新しない / Don't update policies
- D) 規制変更は他部門の問題 / Someone else's problem

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

GRC 自動化は **常時監視 + 影響評価 + 段階提案 + 承認後適用 + 通知 + WORM**。

GRC automation = **continuous monitoring + impact + graded options + post-approval edit + notify + WORM**.

- **B 不正解**: タイムリー性ゼロ。 / Untimely.
- **C 不正解**: コンプライアンス違反。 / Non-compliant.
- **D 不正解**: ガバナンス機能不全。 / Governance failure.

**参照 / Reference:** GRC automation
</details>

---

## 問題 43 / Question 43

**シナリオ / Scenario:**

法務でリーガルホールド（litigation hold）が発行されたら、関連する **すべての文書・コミュニケーション**を即座に保全する必要があります。Claude Code を使って漏れなく実施したい。

When a legal hold issues, **all related docs / communications must be immediately preserved**. Use Claude Code to ensure completeness.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **`.claude/skills/legal-hold.md`** スキル：(i) ホール対象（custodian、件名、期間、キーワード）を引数で受け取る、(ii) 対象システム（メール / Slack / Drive / コードリポジトリ / DB）を **パイプラインで保全 API 呼び出し**、(iii) 保全結果を構造化（システム / 件数 / ハッシュ / タイムスタンプ）、(iv) **PreToolUse フック**で `delete_*` 系操作を該当 custodian に対し **物理的にブロック**、(v) すべての保全アクションを WORM ログに記録、(vi) リリース時は別承認フロー / **`.claude/skills/legal-hold.md`**: (i) args = custodians, subject, period, keywords, (ii) preserve via APIs across email / Slack / Drive / repos / DB, (iii) structured result (system / counts / hashes / timestamps), (iv) `PreToolUse` hook **physically blocks `delete_*`** for affected custodians, (v) WORM-log all preservation actions, (vi) release through a separate approval flow
- B) 弁護士が手動で各システムに依頼 / Lawyers email each system manually
- C) 自動保全は不可能 / Auto preservation infeasible
- D) リーガルホールドは無視 / Ignore legal hold

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

リーガルホールドは **構造化 + 全システム連携 + 削除ブロック + WORM + 承認解除**で漏れなく。

Legal hold = **structured + all-system reach + delete-block + WORM + release approval**.

- **B 不正解**: 漏れ・遅延・抜けやすい。 / Gaps + delays.
- **C 不正解**: 事実誤認。 / Wrong.
- **D 不正解**: 民事訴訟法違反。 / Spoliation.

**参照 / Reference:** Legal hold automation
</details>

---

## 問題 44 / Question 44

**シナリオ / Scenario:**

特許事務所で、出願明細書の **形式チェック**（USPTO・JPO・EPO の形式要件に準拠）を Claude Code で自動化。各庁の細かい形式（フォント・余白・行番号・図面表示）が異なる。

A patent firm automates application **formatting checks** (USPTO / JPO / EPO) — fonts / margins / line numbers / drawings differ.

**設問 / Question:**

最も適切な構成はどれですか？ / Best configuration?

- A) **庁別スキル + 共通テストハーネス**：(i) `.claude/skills/format-check-{office}.md` で各庁の規則をエンコード、(ii) `Glob` + `Grep` で図面・特定タグを抽出、(iii) 違反検出は構造化（場所 / 規則 / 修正案）、(iv) 弁理士は修正案を一括確認、(v) **複数庁に同時出願**する場合は各スキル並列実行 → 結果統合、(vi) 規則更新は `@offices/{office}/rules.md` で版管理 / **Per-office skills + common harness**: (i) `.claude/skills/format-check-{office}.md` encodes each office's rules, (ii) `Glob` + `Grep` extract drawings / tags, (iii) violations structured (location / rule / fix), (iv) attorney bulk-reviews fixes, (v) **multi-office filings** run skills in parallel and merge, (vi) rule updates versioned via `@offices/{office}/rules.md`
- B) 1 スキルで全庁対応 / One skill for all offices
- C) 形式チェックは手動 / Manual format check
- D) 形式は気にしない / Don't bother

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

複数庁対応は **庁別スキル + 共通ハーネス + 並列 + 版管理** が定石。

Multi-office = **per-office skills + common harness + parallel + versioned rules**.

- **B 不正解**: 庁の差異で精度低下。 / Drifts on differences.
- **C 不正解**: 効率損失。 / Inefficient.
- **D 不正解**: 出願却下リスク。 / Rejection risk.

**参照 / Reference:** Patent multi-office automation
</details>

---

## 問題 45 / Question 45

**シナリオ / Scenario:**

国際法律事務所で、マルチ言語（英・日・中・独）のクライアント対応がある。Claude Code を使った **法的助言の言語ローカライズ**で、法的概念の正確性を保つ必要があります。

International law firm has multilingual clients (en / ja / zh / de). Claude Code helps localize legal advice; **legal concepts must stay precise**.

**設問 / Question:**

最も適切なアプローチはどれですか？ / Best approach?

- A) (i) **法的概念辞書**を `@knowledge/legal-glossary.md` で管理（管轄ごとに定訳）、(ii) `.claude/skills/localize-advice.md` がドラフトを生成 → 辞書を厳格に参照、(iii) 法的概念のミスマッチは **構造化エラー**（`{ source_concept, target_locale, dictionary_match: enum["exact","approximate","not_found"], legal_caveat }`）として出力、(iv) `not_found` 時は **必ず弁護士レビュー**、(v) ローカライズ後の文書は管轄の現地弁護士が最終承認 / (i) Maintain a **legal glossary** in `@knowledge/legal-glossary.md` (per-jurisdiction canonical translations), (ii) `.claude/skills/localize-advice.md` drafts strictly referencing the glossary, (iii) mismatches return a **structured error** (`{ source_concept, target_locale, dictionary_match: enum["exact","approximate","not_found"], legal_caveat }`), (iv) `not_found` triggers **mandatory attorney review**, (v) local-jurisdiction lawyer finalizes
- B) 機械翻訳で十分 / MT suffices
- C) 法的概念は翻訳しない / Don't translate legal concepts
- D) 多言語対応はしない / Skip multilingual

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

法的多言語対応は **辞書 + 概念マッチング + 構造化エラー + 弁護士承認**。

Legal multilingual = **glossary + concept matching + structured errors + attorney sign-off**.

- **B 不正解**: 法的精度欠如。 / Lacks legal precision.
- **C 不正解**: ローカライズ意義喪失。 / Defeats purpose.
- **D 不正解**: 顧客対応不能。 / Loses clients.

**参照 / Reference:** Legal localization
</details>

---

## 問題 46 / Question 46

**シナリオ / Scenario:**

自動車製造の **PLC（Programmable Logic Controller）コード**生成 / レビューに Claude Code を導入。**ISO 26262** の機能安全規格 + **ISO 13849** の機械安全規格に準拠する必要あり。

Automotive PLC code dev with Claude Code under **ISO 26262** + **ISO 13849**.

**設問 / Question:**

最も適切な体制はどれですか？ / Best framework?

- A) **安全機能 + 非安全機能のリポジトリ分離**：(i) 安全関連コード（SIL3 以上）は **別リポジトリ** + **より厳しい settings.json**（permissions.deny で実行系を制限）、(ii) 安全機能の変更は **2 名の有資格機能安全エンジニア**による独立レビュー、(iii) `.claude/skills/safety-trace.md` で **要件 → 実装 → V&V**トレーサビリティを自動更新、(iv) すべての変更を **HARA（Hazard Analysis & Risk Assessment）** に紐付け、(v) Claude 出力は補助のみ、最終決定は人間 / Separate **safety vs non-safety repos**: (i) safety code (≥ SIL3) lives in a **separate repo** with stricter `settings.json` (`permissions.deny` restricts executes), (ii) safety changes need **two qualified functional-safety engineers** independent review, (iii) `.claude/skills/safety-trace.md` auto-maintains requirement → implementation → V&V traceability, (iv) all changes linked to **HARA**, (v) Claude output is assistive; humans decide
- B) 安全機能と非安全機能を混在 / Mix safety and non-safety
- C) 安全機能で AI 禁止 / Ban AI for safety
- D) 規制を無視 / Ignore standards

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

機能安全は **リポ分離 + 厳格な権限 + 二重レビュー + トレーサビリティ + HARA 紐付け**。

Functional safety = **repo separation + strict permissions + dual review + traceability + HARA linkage**.

- **B 不正解**: 規格違反。 / Standard breach.
- **C 不正解**: 過剰、効率損失。 / Overreaction.
- **D 不正解**: 違法。 / Illegal.

**参照 / Reference:** ISO 26262・ISO 13849
</details>

---

## 問題 47 / Question 47

**シナリオ / Scenario:**

製造ラインの **品質管理ドキュメント**（FMEA, PPAP, 8D レポート）を Claude Code で生成。**自動車業界 IATF 16949** に準拠するためフォーマットと内容に厳格な規定がある。

Manufacturing quality docs (FMEA, PPAP, 8D) generated with Claude Code; **IATF 16949** strictly governs format / content.

**設問 / Question:**

最も適切な構成はどれですか？ / Best configuration?

- A) **テンプレ + 検証スキル**：(i) `.claude/skills/generate-fmea.md` 等が IATF 16949 テンプレに沿って生成、(ii) 生成後 **`.claude/skills/qms-validate.md`** で必須項目（severity / occurrence / detection / RPN ・root cause / containment / corrective action）の充足を検証、(iii) 不足は構造化エラー、(iv) **品質管理者が最終承認**してからリリース、(v) すべてのドキュメントは **顧客監査用に WORM 保管** / **Templates + validation skill**: (i) `.claude/skills/generate-fmea.md` etc. follow IATF 16949 templates, (ii) `.claude/skills/qms-validate.md` checks required fields (severity / occurrence / detection / RPN; root cause / containment / corrective action), (iii) gaps are structured errors, (iv) **Quality Manager approves** before release, (v) WORM-archive for customer audits
- B) 自由フォーマットで生成 / Free format
- C) 手書きに戻す / Hand-write
- D) IATF は無視 / Ignore IATF

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

QMS ドキュメントは **テンプレ + 検証 + 必須項目 + 承認 + 監査保管**。

QMS docs = **templates + validation + required fields + approval + audit retention**.

- **B 不正解**: 顧客監査で却下。 / Audit failures.
- **C 不正解**: 効率損失。 / Inefficient.
- **D 不正解**: 顧客契約違反。 / Contract breach.

**参照 / Reference:** IATF 16949・FMEA・8D
</details>

---

## 問題 48 / Question 48

**シナリオ / Scenario:**

化学プラントの **OT セキュリティ**で、Claude Code を使った PLC / SCADA コード変更管理。**IEC 62443**（産業制御システム・サイバーセキュリティ）の要件を満たす必要があります。

A chemical plant's OT security manages PLC / SCADA changes with Claude Code under **IEC 62443**.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) **OT/IT 分離 + 厳格な変更管理**：(i) Claude Code は **IT 側のみ**で動作、(ii) PLC / SCADA への変更は **オフラインで生成 → 物理メディアで OT 側に持ち込み → 二重承認**、(iii) すべての変更コードは **デジタル署名 + ハッシュ検証**、(iv) Claude 利用は変更管理システムに登録（誰がいつどのプロンプトで何を生成したか）、(v) 変更後は **退出時のセキュリティ監査**で意図しないサイドチャネル流出をチェック / **OT/IT separation + strict change control**: (i) Claude Code runs **IT-side only**, (ii) PLC/SCADA changes are **generated offline, ferried via physical media to OT, dual-approved**, (iii) all change code is **signed + hash-verified**, (iv) Claude usage logged in change management (who, when, what prompt, what output), (v) post-change **exit security audit** checks for unintended side-channel leakage
- B) Claude Code から PLC へ直接変更 / Direct PLC change from Claude Code
- C) 認証なしで PLC コード書き換え / Write PLC without auth
- D) Claude Code を使わない / Don't use Claude Code

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

IEC 62443 は **OT/IT 分離 + オフライン生成 + デジタル署名 + 二重承認 + 監査** を要求。

IEC 62443 = **OT/IT separation + offline generation + digital signature + dual approval + audit**.

- **B 不正解**: 規格違反、攻撃面拡大。 / Standard breach + attack surface.
- **C 不正解**: 致命的。 / Catastrophic.
- **D 不正解**: 効率損失。 / Inefficient.

**参照 / Reference:** IEC 62443・OT change control
</details>

---

## 問題 49 / Question 49

**シナリオ / Scenario:**

サプライチェーン管理 SaaS の開発で、複数顧客（マルチテナント）の **業務データを扱うコード**を Claude Code で生成。テナント分離を物理レベルで保証したい。

A multi-tenant SCM SaaS dev uses Claude Code for code handling **per-tenant business data**; physical tenant isolation required.

**設問 / Question:**

最も適切な構成はどれですか？ / Best configuration?

- A) **テナント別開発環境 + テスト境界強制**：(i) 開発はテナント別ディレクトリ + git ブランチで分離、(ii) `.claude/settings.json` の `permissions.deny` で他テナントディレクトリへのアクセスをブロック、(iii) `.claude/skills/tenant-boundary-check.md` がコード変更の **クロステナント参照**を自動検出、(iv) テストデータは **テナント ID プレフィックス**で物理隔離、(v) CI で **テナント越境クエリ**（`tenant_id IS NULL` のような危険パターン）を自動検査 / **Per-tenant dev + test boundary enforcement**: (i) per-tenant directories + git branches, (ii) `permissions.deny` blocks access to other tenants' dirs, (iii) `.claude/skills/tenant-boundary-check.md` auto-detects **cross-tenant references**, (iv) test data physically isolated with tenant-ID prefixes, (v) CI flags **cross-tenant queries** (e.g., `tenant_id IS NULL` patterns)
- B) テナント分離は本番だけ / Isolation only in prod
- C) すべてのテナントを混在 / Mix all tenants
- D) テナント分離は不要 / No isolation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

マルチテナント開発は **物理分離 + 権限制限 + 越境検査 + CI 自動検出**で開発時から防御。

Multi-tenant dev = **physical separation + permissions + boundary checks + CI detection**.

- **B 不正解**: 開発漏洩リスク。 / Dev leak risk.
- **C 不正解**: 致命的。 / Catastrophic.
- **D 不正解**: 規制違反。 / Non-compliant.

**参照 / Reference:** Multi-tenant SaaS dev
</details>

---

## 問題 50 / Question 50

**シナリオ / Scenario:**

製造業の **デジタルツイン**プロジェクトで、現場 OT センサーからのデータを取り込み、シミュレーションコードを Claude Code で生成。**現実とシミュレーションの整合性**が重要。

A manufacturing digital-twin ingests OT data; simulation code is built with Claude Code. **Reality / sim parity** is critical.

**設問 / Question:**

最も適切な構成はどれですか？ / Best configuration?

- A) **シミュレーション開発 + 検証スキル**：(i) Plan モードでシミュレーションのドメインモデルをドラフト、(ii) Claude Code で実装、(iii) `.claude/skills/twin-validate.md` が **実機データとのデルタ計算**（同入力で実機 vs シム → 差分を統計的に評価）、(iv) 大きな差分は **モデル校正タスク**として開発フローに自動エスカレーション、(v) すべてのシミュレーションバージョンと実機データを **DataVersion + Hash で紐付け**、再現可能性を保証 / **Sim dev + validation skill**: (i) Plan mode drafts the domain model, (ii) Claude Code implements, (iii) `.claude/skills/twin-validate.md` computes **delta vs reality** (same inputs to reality and sim, statistical comparison), (iv) large deltas auto-escalate as **model-calibration tasks**, (v) every sim version + reality data are **DataVersion + Hash bound** for reproducibility
- B) シミュレーションは現実から独立 / Sim independent of reality
- C) 検証は手動 / Manual validation
- D) デジタルツインは無理 / Twins are infeasible

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

デジタルツインは **Plan + 実装 + デルタ検証 + 校正タスク + 版・ハッシュ紐付け**。

Digital twins = **Plan + impl + delta validation + calibration tasks + version-hash binding**.

- **B 不正解**: 整合性喪失で価値喪失。 / Loses parity, value.
- **C 不正解**: 規模で破綻。 / Doesn't scale.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** Digital twin validation
</details>

---

> **前のドメイン / Previous:** [`domain2_tool_design_mcp.md`](./domain2_tool_design_mcp.md) | **次のドメイン / Next:** [`domain4_prompt_structured_output.md`](./domain4_prompt_structured_output.md)
