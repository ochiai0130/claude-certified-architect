# Domain 2: ツール設計と MCP 統合 / Tool Design and MCP Integration

> 配点比率 / Weight: **18%**
> 問題数 / Questions: **5**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- ツールインターフェース設計（記述・スキーマ・誤呼び出し防止） / Tool interface design — descriptions, schemas, mis-routing prevention
- MCP の構造化エラー（`isError`, `errorCategory`, `retryable`） / MCP structured error responses
- `tool_choice`（`auto` / `any` / 強制選択）の適切な使い分け / `tool_choice` selection
- MCP スコープ（プロジェクト `.mcp.json` / ユーザー `~/.claude.json` / 環境変数） / MCP scopes
- 組み込みツール vs MCP ツール — トークンコスト・キャッシュ・権限分離のトレードオフ / Built-in vs MCP tools tradeoffs

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

大手損害保険会社の保険査定エージェントで、MCP サーバから 4 つのツールが提供されています：

A property-and-casualty insurance claim-assessment agent uses an MCP server exposing four tools:

- `get_policy(policy_id)` — 契約条項テキストを返す / returns policy clause text
- `lookup_policy(customer_id, claim_date)` — 顧客 ID と請求日から有効な契約を**検索する** / **searches** for the active policy by customer ID and claim date
- `get_claim_history(customer_id)` — 過去請求履歴を返す / returns past claim history
- `lookup_claim(claim_id)` — 請求の詳細を返す / returns claim details

本番ログを見ると、エージェントが本来 `lookup_policy(customer_id, claim_date)` を呼ぶべき場面で、`get_policy(claim_id)` を誤呼び出しして「契約見つからず」エラーを多発しています。

Production logs show the agent frequently calling `get_policy(claim_id)` when it should call `lookup_policy(customer_id, claim_date)`, producing "policy not found" errors.

**設問 / Question:**

最も効果的かつ持続的な改善はどれですか？

Which is the most effective and durable fix?

- A) ツール記述を **動詞ベースの一意な命名** に変更（`get_policy_by_id`, `find_active_policy_by_customer_and_date`）し、各ツールの `description` に「いつ使うか／いつ使わないか」を明示。さらに JSON スキーマで `policy_id` のフォーマット（接頭辞 `POL-`）を `pattern` で強制 / Rename to verb-based, unique names (`get_policy_by_id`, `find_active_policy_by_customer_and_date`), make each tool's `description` explicit about "when to use / when NOT to use", and constrain `policy_id` format (prefix `POL-`) via JSON schema `pattern`
- B) システムプロンプトに「`get_policy` には `policy_id` を、`lookup_policy` には `customer_id` と `claim_date` を渡すこと」と書き加える / Add to the system prompt: "pass `policy_id` to `get_policy` and `customer_id` + `claim_date` to `lookup_policy`"
- C) `tool_choice: "any"` に設定して、Claude が必ずツールのどれかを選ぶよう強制する / Set `tool_choice: "any"` to force Claude to always pick one of the tools
- D) より高性能なモデル `claude-opus-4-6` に切り替える / Upgrade to `claude-opus-4-6`

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

ツールの誤呼び出しは大半が **記述の曖昧さ・命名の衝突・スキーマの緩さ** に起因します。命名を動詞ベースで一意にし、`description` に **「いつ使う／使わない」** の境界条件を書き、スキーマで型を厳格化（接頭辞 `pattern`）する三段構えが、最も持続的でモデル横断的に効きます。`claim_id`（CLM- 接頭辞）が `policy_id`（POL- 接頭辞）の `pattern` に通らなければ、ツール呼び出しは API レベルで弾かれます。

Mis-routing usually stems from **ambiguous descriptions, naming collisions, and loose schemas**. Verb-based unique names + "when to use / when NOT" guidance + schema-level `pattern` constraints form a three-layer defense that holds across models. A `claim_id` with `CLM-` prefix simply cannot pass the `pattern` for `policy_id` (`POL-`).

- **B 不正解**: プロンプト改善は最初に試すが、根本治療ではなく、ツールが増えると破綻します。 / Prompt fixes are first-aid, not durable.
- **C 不正解**: `tool_choice: "any"` は「どれかを呼ぶ」ことを強制するだけで、**間違ったツールを選ぶ問題は解決しません**。むしろ「ツールを呼ばない」が正解の場面まで強制呼び出しになり害があります。 / `any` only forces a tool call — it doesn't fix which tool is chosen, and harms cases where no call is right.
- **D 不正解**: モデル変更でも記述の曖昧さは残り、根本治療になりません。 / Model upgrades don't fix ambiguous tool surfaces.

**参照 / Reference:** `guide_ja.md` 「2.2 ツール定義」「2.3 tool_choice」、Anthropic Tool Use docs
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

PCI DSS 準拠が必要な決済プラットフォームで、MCP サーバーが `charge_card`、`refund`、`tokenize_card` ツールを提供しています。あるトランザクションで `charge_card` がカード会社側でレートリミットに当たり、5 秒後に再試行すれば成功するケースが頻発します。一方、CVV 不一致やカード期限切れのケースは再試行しても無意味です。MCP からのエラー応答で、上位コーディネーターエージェントが**カード番号や CVV を絶対にログ・コンテキストに残さず**、適切に再試行 / 失敗判定できるようにしたい。

A PCI DSS-compliant payments platform exposes `charge_card`, `refund`, `tokenize_card` via MCP. `charge_card` often hits issuer rate limits and succeeds on retry after 5 seconds, while CVV mismatches or expired cards cannot be remediated by retry. The error response must let the coordinator decide retry vs fail **without ever putting card numbers or CVVs into logs or model context**.

**設問 / Question:**

MCP サーバの `charge_card` ツールが返す**最も適切なエラー応答構造**はどれですか？

What is the most appropriate error response structure from the MCP server's `charge_card` tool?

- A) `{ "isError": true, "content": [{"type": "text", "text": "Card 4111-1111-1111-1111 declined: CVV mismatch"}] }` / The same with full PAN and CVV in text
- B) `{ "isError": true, "content": [{"type": "text", "text": "Charge failed"}] }` のみ / `isError: true` with just "Charge failed"
- C) `{ "isError": true, "content": [{"type": "text", "text": "Charge failed: rate_limited"}], "_meta": { "errorCategory": "rate_limited", "retryable": true, "retryAfterMs": 5000, "correlationId": "tx_abc123" } }` のように、**カード番号は含めず**、エラーカテゴリ・再試行可能性・推奨待機時間・相関 ID を構造化フィールドで返す / Return structured metadata (`errorCategory`, `retryable`, `retryAfterMs`, `correlationId`) **without card data**, e.g., `{ "isError": true, "content": [{"type": "text", "text": "Charge failed: rate_limited"}], "_meta": {...} }`
- D) HTTP ステータスコード 429 を返してエージェント側で解釈させる / Return HTTP 429 and let the agent interpret it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

MCP の構造化エラーは **(1) 機械可読な分類（`errorCategory`）、(2) 再試行可能性（`retryable`）、(3) 待機指示（`retryAfterMs`）、(4) 監査用相関 ID** を返すのが正解で、これによりコーディネーターは LLM の判断ではなく決定論的に再試行ロジックを実行できます。PCI DSS は **PAN（カード番号）と CVV をコンテキストやログに残してはならない** と要求するため、`content` のテキストには絶対に含めません。エラーメッセージから何かを学ばせる必要は一切なく、機械処理可能な構造に切り出すのが原則です。

MCP error design should expose **(1) machine-readable category, (2) retryability, (3) retry timing, (4) correlation ID for audit**, enabling the coordinator to act deterministically rather than via LLM judgment. PCI DSS forbids PAN/CVV in context or logs — so card data must never appear in `content` text.

- **A 不正解**: PAN と CVV をコンテキストに入れた瞬間 PCI DSS 違反。最重大インシデント。 / Embedding PAN/CVV in context is an immediate PCI DSS violation.
- **B 不正解**: コンプライアンスは満たすが、エージェントが「再試行すべきか」を**確率的に**推測することになり、信頼性が出ません。 / Compliant but forces probabilistic retry decisions.
- **D 不正解**: HTTP ステータスは MCP プロトコル外で、エージェントに直接届きません。MCP の `isError` + 構造化メタデータが正しいレイヤー。 / HTTP status codes don't surface through MCP — wrong layer.

**参照 / Reference:** `guide_ja.md` 「2.5 MCP の構造化エラー」、PCI DSS v4.0 Req. 3.3 (PAN protection)
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

法務契約書から **支払条項** を抽出するパイプラインを設計中です。100% の契約書から構造化された JSON を返す必要があり、抽出失敗は許容できますが「自然言語の長文応答」は下流処理が壊れるため許容できません。一方、別のサポートチャットエージェントでは、ユーザーの質問が雑談か業務質問かを判別し、業務質問の場合のみ `lookup_order` ツールを呼ぶ仕様です。

You are designing two systems:

1. **Contract extraction**: Must return structured JSON for every contract; missing extraction is OK, but free-text responses break downstream parsing.
2. **Support chat**: Determines whether a user message is small talk or a business question; calls `lookup_order` **only** for business questions.

**設問 / Question:**

`tool_choice` の設定として最も適切な組み合わせはどれですか？

What is the most appropriate `tool_choice` combination?

- A) 両方とも `tool_choice: "auto"` / Both `"auto"`
- B) 契約抽出は `tool_choice: "any"`（または `extract_payment_terms` ツールへ強制）、サポートチャットは `tool_choice: "auto"` / Contract extraction: `"any"` (or forced to `extract_payment_terms`); Support chat: `"auto"`
- C) 契約抽出は `tool_choice: "auto"`、サポートチャットは `tool_choice: "any"` / Contract extraction: `"auto"`; Support chat: `"any"`
- D) 両方とも `tool_choice: "any"` / Both `"any"`

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

`tool_choice: "any"`（または特定ツールへの強制）は **「必ずツール呼び出しを返させる」** ことで構造化出力を保証する設計です。契約抽出のように **下流が JSON しか受け付けない** ケースでは適切。一方、サポートチャットでは雑談に対して `lookup_order` を呼ばせるとハルシネーション化（存在しない注文を捏造）しやすいため、**「呼ぶべきでない時は呼ばない」を許す `auto`** が正解。

`tool_choice: "any"` (or forcing a specific tool) **guarantees a tool call**, which is correct when downstream consumers accept only JSON. For chat, allowing the model to **not** call a tool (`"auto"`) prevents hallucinated lookups for small talk.

- **A 不正解**: 契約抽出で `auto` だと「契約に支払条項がありません（自由文）」が返り、JSON パースが壊れます。 / `"auto"` lets the model emit free text and break parsing.
- **C 不正解**: 完全に逆。抽出を `auto` にし、雑談を `any` で強制呼び出しさせるとハルシネーションが頻発します。 / Inverted — causes hallucinations on small talk.
- **D 不正解**: チャットで `any` を使うと存在しない注文 ID をでっち上げる "tool hallucination" が起きます。 / `"any"` on chat invites fabricated tool inputs.

**参照 / Reference:** `guide_ja.md` 「2.3 tool_choice パラメータ」「2.4 構造化出力」
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

マルチテナント B2B SaaS で、各顧客が **自社専用の MCP サーバ**（社内 CRM、社内ナレッジ、業務 API）への接続情報を Claude Code 経由で利用します。社内監査では「顧客 A の API キーが顧客 B の Claude セッションから到達不可能であること」を立証する必要があり、また、開発者個人のローカル設定（個人用 GitHub MCP など）と顧客の本番接続情報を混ぜてはならないという要件があります。

In a multi-tenant B2B SaaS, each customer connects to **their own** MCP servers (internal CRM, knowledge base, business APIs) via Claude Code. Audit requires proving Customer A's API keys are unreachable from Customer B's session, and developer personal MCPs (e.g., personal GitHub) must not mix with customer production credentials.

**設問 / Question:**

MCP スコープと秘匿情報の適切な配置はどれですか？

Which MCP scope and secret-handling design is correct?

- A) すべての MCP サーバ設定を `~/.claude.json`（ユーザースコープ）に集約し、API キーも直接記述する / Put all MCP configs and keys directly in `~/.claude.json` (user scope)
- B) MCP は使わず、すべてのツールを Agent SDK の `customTools` として直接実装する / Avoid MCP entirely; implement all tools as Agent SDK `customTools`
- C) 顧客ごとに別の Claude API キーを使えば MCP は分離不要 / Use a separate Claude API key per customer; no MCP separation needed
- D) 顧客固有の MCP は **顧客プロジェクトディレクトリの `.mcp.json`** に定義し、API キーは平文で書かず **環境変数参照（`${CRM_API_KEY}`）** にする。環境変数は顧客テナントごとの隔離環境（コンテナ・専用ホスト）から注入。開発者個人の MCP は `~/.claude.json` に置く / Define per-customer MCPs in the customer project's **`.mcp.json`** with keys referenced as **environment variables** (`${CRM_API_KEY}`); inject env vars from per-tenant isolated environments. Personal developer MCPs live in `~/.claude.json`

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

MCP は **プロジェクトスコープ `.mcp.json`（チーム共有・顧客固有）** と **ユーザースコープ `~/.claude.json`（個人用）** が分離されており、これを使い分けることで「顧客本番」と「開発者個人」の混在を防ぎます。秘匿情報は設定ファイルに平文で書かず **`${VAR}` 構文で環境変数参照** とし、テナント分離されたコンテナや CI 環境から注入することで、顧客 A のキーが顧客 B のプロセスから物理的に到達不可能になります。これは "secrets out of config files" という業界標準の原則です。

MCP separates **project scope (`.mcp.json`)** from **user scope (`~/.claude.json`)**. Customer-specific configs belong in the project; personal tools belong in user scope. Secrets must be **environment-variable references** injected from tenant-isolated runtimes, ensuring physical isolation between customers.

- **A 不正解**: ユーザースコープに顧客本番キーを集約すると、開発者の個人マシン上で全顧客の鍵が露出。監査で確実に落ちます。 / Aggregating production keys into user scope exposes all customers on a single dev machine.
- **B 不正解**: MCP の利点（プロセス分離・標準プロトコル・スコープ管理）を捨てる過剰反応です。 / Throwing away MCP gives up process isolation and standard protocol benefits.
- **C 不正解**: Anthropic API キーの分離は LLM 呼び出しの分離であって、CRM や業務 API の分離とは別問題です。 / Anthropic API key separation does not isolate downstream MCP server credentials.

**参照 / Reference:** `guide_ja.md` 「2.6 MCP サーバの設定とスコープ」「環境変数による秘匿情報注入」
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

製造業の大規模データレイク（PB 級・日次更新・PII 混在）に対して、Claude Code でアドホックな調査を行う体制を作ります。検索パターンとして以下が想定されます：

You are setting up Claude Code for ad-hoc investigation against a petabyte-scale manufacturing data lake (daily updates, PII mixed in). Expected search patterns:

(a) ファイル名パターンでログを探す（`*.error.log`） / Locate logs by filename pattern (`*.error.log`)
(b) コードベース全体から特定の例外メッセージを検索 / Search the codebase for a specific exception message
(c) PII を含むカラムから集計クエリを実行 / Aggregate from PII-containing columns
(d) 個別ファイル数件の中身を読む / Read a few specific files

**設問 / Question:**

トークンコスト・キャッシュヒット率・権限分離の三軸で最適な**ツール選択戦略**はどれですか？

Which is the optimal tool-selection strategy across token cost, cache hit rate, and permission isolation?

- A) すべて Bash ツール 1 本で `grep`、`find`、`head`、`psql` をパイプで組み合わせて実行 / Use the Bash tool alone with `grep`, `find`, `head`, `psql` pipelines
- B) (a) は組み込みの Glob、(b) は組み込みの Grep、(d) は Read を使う。(c) の PII 集計は **専用 MCP サーバ** を介し、サーバ側で行レベル・カラムレベルのアクセス制御と監査ログを強制。Bash は必要最小限に絞り、`allowed_tools` で制限 / (a) Glob, (b) Grep, (d) Read for built-ins; (c) PII aggregation through a **dedicated MCP server** that enforces row/column-level ACLs and audit logging server-side. Restrict Bash to minimum via `allowed_tools`
- C) 全ファイルをまずコンテキストに読み込み、Claude に検索させる / Load all files into context first and let Claude search
- D) Glob・Grep・Read は遅いので使わず、すべて MCP ツールで自前実装する / Avoid Glob/Grep/Read (too slow) and reimplement everything as MCP tools

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**役割で分ける**のが正解です。組み込み Glob / Grep / Read は ① プロンプトキャッシュが効きやすく、② 大量ファイル走査でトークン消費が少なく（マッチ部のみ返却）、③ ローカル FS への安全なアクセスを提供します。一方、PII を含む集計は LLM のコンテキストに PII を入れずにサーバ側で **ACL + 監査ログ + 集計後の脱識別** を強制する必要があり、これは MCP サーバの責務です。Bash を絞る (`allowed_tools` で制限) のは権限分離のため。

The right answer is **role-based decomposition**. Built-in Glob/Grep/Read are cache-friendly, token-efficient (returning only matches), and FS-safe. PII aggregation must enforce ACLs, audit logging, and post-aggregation de-identification **server-side** — a perfect fit for a dedicated MCP server that prevents PII from entering model context. Restricting Bash via `allowed_tools` reinforces permission isolation.

- **A 不正解**: Bash 1 本だと PII が直接コンテキストに流れ込み、監査も困難。権限境界が曖昧。 / A single Bash channel pulls PII into context and blurs audit boundaries.
- **C 不正解**: PB 級データを文脈に入れるのは現実的でなく、コストと中間消失効果で破綻します。 / Loading PB-scale data into context is impossible and counter-productive.
- **D 不正解**: 組み込みツールはローカル探索で最も効率的。MCP で再実装するのは車輪の再発明で、キャッシュ効率も悪化します。 / Built-ins are most efficient for local search; reimplementing them via MCP wastes cache and effort.

**参照 / Reference:** `guide_ja.md` 「2.7 組み込みツール」「2.6 MCP サーバ」「6.x 権限とセキュリティ」
</details>

---

> **前のドメイン / Previous:** [`domain1_agent_architecture.md`](./domain1_agent_architecture.md) | **次のドメイン / Next:** [`domain3_claude_code_workflows.md`](./domain3_claude_code_workflows.md)
