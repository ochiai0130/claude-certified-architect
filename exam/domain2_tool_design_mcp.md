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

## 問題 6 / Question 6

**シナリオ / Scenario:**

MCP サーバが大規模監査ログ（行数 100 万件）を提供しています。これを **ツール**として公開すべきか、**リソース**として公開すべきかを設計中です。クライアントは検索クエリで特定行を取得することが多い。

An MCP server exposes 1M-row audit logs. You're deciding whether to expose them as a **tool** or a **resource**. Clients usually fetch specific rows via search.

**設問 / Question:**

最も適切な公開方式はどれですか？ / Best exposure?

- A) リソースとして全行を 1 つの URI で公開し、クライアント側でフィルタ / Single URI exposing all rows; client-side filter
- B) ツール（`search_audit_logs(query, limit)`）として公開し、サーバ側でクエリ実行 / Tool `search_audit_logs(query, limit)` with server-side query
- C) ツールとリソースの両方で同じデータを公開 / Expose via both tool and resource
- D) MCP では公開せず別途 REST API を構築 / Bypass MCP; build a separate REST API

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

MCP の **tools** は引数付きの動的アクション（検索・操作）に適合し、**resources** は静的または準静的な参照可能データ（ドキュメント、固定 URI）に適しています。1M 行をリソース化するとクライアントに膨大なデータを送ることになり非効率。検索クエリ付きツールが正解。

MCP **tools** suit dynamic actions with arguments (searches, operations); **resources** suit static or semi-static referenceable data (docs, fixed URIs). 1M rows as a single resource overwhelms the client; a parameterized tool is correct.

- **A 不正解**: 100 万行をクライアントに転送はトークン浪費。 / Wasteful transfer.
- **C 不正解**: 二重公開は混乱と不整合を招く。 / Confuses clients.
- **D 不正解**: MCP の利点（標準プロトコル・ツール統合）を捨てる。 / Loses MCP benefits.

**参照 / Reference:** `guide_ja.md` 「MCP tools vs resources」
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

社内 MCP サーバが本番運用に入り、新バージョンで `create_invoice(amount, currency)` の `currency` パラメータが必須化されました。既存クライアントは `currency` なしで呼び出しているため、即座に壊れます。

A production MCP server's new version makes `currency` required in `create_invoice(amount, currency)`. Existing clients calling without `currency` immediately break.

**設問 / Question:**

最も適切なバージョン互換戦略はどれですか？ / Best versioning strategy?

- A) 即時に必須化して全クライアントを強制移行 / Force-migrate by making it required immediately
- B) 古い名前 `create_invoice` を**そのまま**にして必須化、ドキュメントだけ更新 / Keep `create_invoice` and just update docs
- C) **新ツール `create_invoice_v2` を追加**して `currency` を必須化、古い `create_invoice` は当面残し、`description` に "deprecated, use `create_invoice_v2`" と明記。一定期間後（例：90 日後）削除する旨をリリースノートで告知 / **Add `create_invoice_v2`** with `currency` required; keep the old `create_invoice` for a deprecation window with `description` marked "deprecated, use `create_invoice_v2`"; announce removal date (e.g., 90 days)
- D) 両方のクライアントが共存できないので片方を完全削除 / Delete one of the two

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

API / ツールの破壊的変更には **Expand → Migrate → Contract** パターンが基本：新版を追加（expand）→ 旧版を deprecation（migrate 期間）→ 削除（contract）。これにより既存クライアントが壊れず順次移行できます。

Breaking changes follow **Expand → Migrate → Contract**: add v2, deprecate v1, remove later. Avoids client breakage.

- **A 不正解**: 即時必須化は本番停止を招く。 / Causes outages.
- **B 不正解**: 名前据え置きでも動作変更で既存壊れる。 / Behavior change still breaks clients.
- **D 不正解**: 完全削除は移行期間を奪う。 / No transition window.

**参照 / Reference:** API バージョニング・Expand-Contract パターン
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

3 種の MCP サーバ（社内 CRM、外部信用機関、内部監査ログ）を 1 つの Claude Code に接続。CRM はローカル stdio、信用機関は HTTPS リモート、監査ログは SSE ストリームで通知される必要があります。

Three MCP servers connect to one Claude Code: internal CRM (local stdio), external credit bureau (HTTPS remote), audit log (SSE stream notifications).

**設問 / Question:**

最も適切なトランスポート選択はどれですか？ / Best transport choice?

- A) すべて stdio に統一する / Force everything to stdio
- B) すべて HTTPS に統一する / Force everything to HTTPS
- C) MCP は stdio しかサポートしない / MCP supports only stdio
- D) **各サーバの特性に合わせる**：CRM はローカルプロセスなので stdio、信用機関はリモートかつ認証が必要なので HTTPS（OAuth/Bearer 認証）、監査ログはサーバプッシュが必要なので SSE。同じ Claude Code クライアントから 3 つを並行接続できる / **Match transport to server**: stdio for local CRM, HTTPS (OAuth/Bearer) for remote credit bureau, SSE for push notifications. All three can connect in parallel from the same client

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

MCP は **stdio・HTTPS・SSE** など複数のトランスポートをサポートし、サーバの特性に合わせて使い分けるのが正解です。

MCP supports multiple transports (stdio, HTTPS, SSE) — choose per server characteristics.

- **A 不正解**: リモートに stdio は不可能。 / Stdio can't reach remote.
- **B 不正解**: ローカルプロセスを HTTPS にするのは過剰。 / HTTPS is overkill for local.
- **C 不正解**: 事実誤認。複数トランスポート対応。 / Factually wrong.

**参照 / Reference:** `guide_ja.md` 「MCP transports」
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

医療 MCP サーバの `prescribe_medication(patient_id, drug, dose, route)` ツールで、入力値の不正（用量負数、不存在の薬剤コード）が稀に届きます。サーバ側で防御層を作りたい。

A medical MCP tool `prescribe_medication(patient_id, drug, dose, route)` occasionally receives invalid inputs (negative dose, unknown drug code). You want server-side defense.

**設問 / Question:**

最も適切な検証戦略はどれですか？ / Best validation strategy?

- A) クライアント（Claude）に検証を任せる / Let the client (Claude) validate
- B) **JSON Schema レベル**：`dose` を `minimum: 0` と `pattern`、`drug` を `enum`（または formulary 参照）。**サーバ実装レベル**：相互依存の意味検証（妊娠中は禁忌薬を拒否、年齢×用量レンジチェック）。**両方を多層**に / **JSON Schema level**: `dose` with `minimum: 0`, `drug` with `enum` (or formulary reference). **Server implementation level**: cross-field semantic checks (contraindicated drugs in pregnancy, age × dose range). **Both layers** combined
- C) クライアントとサーバの両方で同じ検証ロジックを書く / Duplicate identical validation in client and server
- D) 検証は不要、エラーが起きてから対処 / No validation; deal with errors as they arise

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**スキーマレベル**は構文／値域、**実装レベル**はビジネスルール／意味整合性、それぞれ責務が異なるため両方必要。多層防御。

Defense in depth: schema-level for syntax/range, implementation-level for business rules / semantic integrity.

- **A 不正解**: クライアント任せは確率的。 / Probabilistic.
- **C 不正解**: 同一ロジック重複は保守性悪化。サーバを真実の源にすべき。 / Duplication; server should be source of truth.
- **D 不正解**: 後手は規制要件に不適合。 / Reactive fails compliance.

**参照 / Reference:** `guide_ja.md` 「2.4 JSON Schema」「2.5 構文 vs 意味エラー」
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

カスタマーサポートエージェントが MCP ツール `lookup_customer(email)` を呼び、結果として 50,000 文字の顧客対応履歴を取得します。コンテキストへの取り込みが大きく、後続のターンが劣化します。

A support agent calls MCP tool `lookup_customer(email)` and receives a 50K-character interaction history. Context bloat degrades subsequent turns.

**設問 / Question:**

最も適切な MCP サーバ側の改善はどれですか？ / Best server-side improvement?

- A) ツールに `summary_only: true` パラメータを追加し、サーバ側で要約・トリミングしたサブセットを返す。詳細が必要な場合のみ別ツール `get_customer_history_chunk(email, page)` で限定的に追加取得 / Add a `summary_only: true` parameter; server returns a trimmed summary. For detail, expose a separate `get_customer_history_chunk(email, page)` tool for paginated drilling
- B) ツールから 50K 文字をそのまま返し続ける / Keep returning the full 50K
- C) クライアント側で文字列を切り詰める / Truncate on the client side
- D) `claude-opus-4-6` の長コンテキストを使えば問題ない / Long context on `claude-opus-4-6` solves it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**情報のサイズと詳細度を可変にする**のは MCP ツールの基本設計。サマリと詳細チャンクの 2 段構えで必要なときだけ深掘り。

Variable size/detail is fundamental MCP tool design — summary + paginated drill-down.

- **B 不正解**: コンテキスト劣化を放置。 / Ignores degradation.
- **C 不正解**: クライアント切り詰めは情報損失が制御できない。 / Uncontrolled loss.
- **D 不正解**: 長コンテキストでも中間消失効果残存。 / Doesn't fix drift.

**参照 / Reference:** `guide_ja.md` 「2.2 ツール設計」「コンテキスト効率」
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

社内のナレッジベース MCP サーバが、`search(query)` ツールに加えて、エージェントから **「LLM にプロンプトを実行させる」** 機能を提供したい。MCP プロトコルにはこの仕組みがあります。

An internal KB MCP server, beyond `search(query)`, wants to **make the agent invoke an LLM prompt on its behalf**. MCP supports this.

**設問 / Question:**

該当する MCP の機能はどれですか？ / Which MCP capability is this?

- A) Resources — 読み取り専用データ参照 / read-only data
- B) Tools — クライアント主導のアクション / client-driven actions
- C) **Sampling** — サーバが「LLM にこのプロンプトを実行してください」とクライアントに依頼する仕組み。サーバ側で LLM 機能を活用しつつ、API キーや課金はクライアントの責任にできる / **Sampling** — the server asks the client to "please run this prompt on the LLM"; server leverages LLM capability while API key / billing remain on the client
- D) Notifications — サーバから一方向の通知 / one-way notifications

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**Sampling** は MCP の高度な機能で、サーバが LLM 推論を必要とするときにクライアントに依頼します。これにより API キー管理が中央化され、サーバ実装が軽量に。

**Sampling** lets MCP servers request LLM inference from the client, centralizing API-key management and keeping servers light.

- **A 不正解**: Resources は静的データ参照。 / Static data only.
- **B 不正解**: Tools はクライアントが起点。 / Client-initiated.
- **D 不正解**: Notifications は LLM 呼び出しではない。 / Not LLM invocation.

**参照 / Reference:** MCP Sampling spec
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

支払い MCP サーバの `charge(amount, idempotency_key)` ツール。同じ `idempotency_key` で 2 回呼ばれた場合の挙動を設計しています。

Designing the behavior of `charge(amount, idempotency_key)` when called twice with the same `idempotency_key`.

**設問 / Question:**

最も適切な挙動はどれですか？ / Best behavior?

- A) 2 回目もそのまま処理して二重請求 / Process the second call as a fresh charge
- B) 2 回目は **既存の処理結果を返す**（成功 / 失敗 / 進行中）。状態が `in_progress` の場合はリトライ可否情報も含める。同じ `idempotency_key` で異なる `amount` が来た場合は `409 Conflict` で拒否（不整合検出） / On second call, **return the existing result** (success / failure / in_progress, with retry guidance). If the same key arrives with a different `amount`, return `409 Conflict` (inconsistency detected)
- C) 2 回目はランダムにエラー / Randomly fail the second call
- D) `idempotency_key` を無視して常に新規処理 / Ignore the key; always process fresh

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

冪等キーの正解実装は **既存結果の再返却 + 不整合の 409**。これにより安全な再試行ができ、誤利用も検出可能。

Correct idempotency: return prior result + 409 on inconsistent reuse — enables safe retries and detects misuse.

- **A 不正解**: 二重請求は致命的。 / Double-charge fatal.
- **C 不正解**: ランダム失敗は意味不明な挙動。 / Nonsense behavior.
- **D 不正解**: 鍵を無視は冪等性放棄。 / Defeats the purpose.

**参照 / Reference:** Idempotency-Key best practices
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

グローバル決済プラットフォームのカスタマーサービスで、Claude が次の 4 ツールを持ちます：`get_account`, `get_recent_transactions`, `process_refund`, `escalate_to_human`。返金処理の前に必ずアカウント検証と取引確認を要求する規制があります（PCI DSS / SOX）。

A global payments customer-service Claude has 4 tools: `get_account`, `get_recent_transactions`, `process_refund`, `escalate_to_human`. Regulation requires account verification and transaction confirmation **before** any refund (PCI DSS / SOX).

**設問 / Question:**

最も適切な決定論的順序強制はどれですか？ / Best deterministic ordering enforcement?

- A) システムプロンプトで「返金前に検証」と指示 / Prompt: "verify before refund"
- B) `process_refund` の MCP サーバ側実装で、**直前のセッション内で `get_account` と `get_recent_transactions` が成功していなかった場合に 400 エラー**を返す（前提条件チェック）。同時に Agent SDK の `PreToolUse` フックでも同等チェックを行う多層防御 / In the `process_refund` MCP server implementation, **return 400 if `get_account` and `get_recent_transactions` haven't succeeded earlier in the session** (precondition check). Also enforce the same in an Agent SDK `PreToolUse` hook for defense in depth
- C) Few-shot で順序を教える / Teach order via few-shot
- D) ツール記述で「順序を守れ」と書く / Write "follow order" in tool descriptions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

順序保証は **MCP サーバ側の前提条件チェック + クライアント側のフック**で多層防御。プロンプト・記述は確率的で規制不適合。

Order guarantees use **server preconditions + client hooks** for defense in depth. Prompts/descriptions are probabilistic.

- **A 不正解**: プロンプトは確率的。 / Probabilistic.
- **C 不正解**: Few-shot 同上。 / Same.
- **D 不正解**: 記述に頼るのは規制不適合。 / Insufficient for regulation.

**参照 / Reference:** `guide_ja.md` 「3.5 PreToolUse」「MCP 前提条件」
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

社内開発チームが共通の MCP サーバを開発中。リリース前にテストを充実させたい。サーバは `query_database`, `update_record`, `delete_record` を持ち、`delete_record` は副作用が大きい。

A team is developing a shared MCP server pre-release. They want strong testing. Tools: `query_database`, `update_record`, `delete_record`; `delete_record` has large blast radius.

**設問 / Question:**

最も適切なテスト戦略はどれですか？ / Best testing strategy?

- A) 手動でクライアントから呼んで確認 / Manual client invocation
- B) 各ツールの **単体テスト**（入力スキーマ・正常系・異常系）、**統合テスト**（実際の DB に対する一連の操作）、**契約テスト**（MCP プロトコル準拠）、**安全性テスト**（破壊的操作の引数検証）。CI で全テストを自動実行し、`delete_record` には削除前の **承認フロー**も実装 / **Unit tests** (schemas, happy + edge paths), **integration tests** (real DB sequences), **contract tests** (MCP protocol conformance), **safety tests** (destructive-arg validation). All run in CI; `delete_record` also has a pre-deletion **approval workflow**
- C) クライアント側 LLM が動けば良いのでテスト不要 / If the LLM works on the client, no tests needed
- D) `delete_record` は本番で動かして確認 / Validate `delete_record` in production

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

MCP サーバは API そのもの。**単体・統合・契約・安全性**の 4 層テストを CI 自動化、破壊的操作には承認フロー追加。

MCP servers are APIs — apply unit + integration + contract + safety tests in CI, with approval flows for destructive ops.

- **A 不正解**: 手動は再現性なし。 / Not reproducible.
- **C 不正解**: サーバの責務はサーバで検証。 / Server contracts must be verified.
- **D 不正解**: 本番で破壊的検証は致命的。 / Catastrophic.

**参照 / Reference:** ソフトウェアテスト原則・MCP testing
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

エージェントが `report_generator(template, params)` ツールを呼ぶと、サーバは PDF 生成に 30〜90 秒かかります。クライアントの API タイムアウト（60 秒）に引っかかる可能性があります。

`report_generator(template, params)` takes 30–90s server-side. Client API timeout (60s) may trigger.

**設問 / Question:**

最も適切な MCP サーバ設計はどれですか？ / Best MCP server design?

- A) 同期で 90 秒待たせる / Sync wait 90s
- B) **非同期パターン**：`report_generator` は即時に `{ status: "processing", report_id }` を返し、エージェントは `get_report_status(report_id)` で完了をポーリング、または完了時にサーバが通知（notification）を送る。長時間処理を別エンドポイントに分離することでタイムアウト問題を構造的に回避 / **Async pattern**: `report_generator` returns `{ status: "processing", report_id }` immediately; the agent polls `get_report_status(report_id)` or receives a server notification on completion. Decouples long-running work from request timeout
- C) タイムアウトを 5 分に伸ばす / Bump the timeout to 5 min
- D) PDF 生成を諦めて HTML にする / Drop PDFs; switch to HTML

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

長時間処理は **非同期 + ポーリング / 通知**で疎結合に。タイムアウト延長は対症療法で根本解決にならない。

Long-running work belongs in **async + polling/notifications**, decoupled from request timeouts.

- **A 不正解**: タイムアウトに直面。 / Timeout collision.
- **C 不正解**: 別の長時間ジョブで再発。 / Recurs for any longer job.
- **D 不正解**: 機能を犠牲にする過剰反応。 / Sacrifices functionality.

**参照 / Reference:** 非同期パターン・MCP notifications
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

セキュリティ運用センター（SOC）の Claude エージェントが、内部 SIEM への MCP ツール `query_siem(rule_id, time_range)` を呼びます。本番で、攻撃時刻に大量の query が短時間に走り、SIEM に過負荷を与えました。

A SOC's agent calls MCP tool `query_siem(rule_id, time_range)`. During an incident, query bursts overloaded the SIEM.

**設問 / Question:**

最も適切な対策はどれですか？ / Best mitigation?

- A) MCP サーバ側で **レートリミット**（per-session, per-user, per-tool）を実装し、超過時は `429 Too Many Requests` と `retry_after_ms`、`errorCategory: "rate_limited"` を構造化エラーで返す。エージェントは指数バックオフで再試行可能 / Server-side **rate limiting** (per-session, per-user, per-tool); on overage return `429` + `retry_after_ms` + `errorCategory: "rate_limited"` as structured error so the agent can back off
- B) クライアント側で「レートリミットを守れ」と指示 / Instruct the client to "respect limits"
- C) SIEM のスケールアップ / Scale up the SIEM
- D) クエリを禁止 / Forbid queries

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

外部依存の保護は **サーバ側レートリミット + 構造化エラー**でクライアントが安全に再試行できるようにするのが定石。

Protect external dependencies via **server-side rate limits + structured errors** enabling safe client retries.

- **B 不正解**: クライアント任せは確率的。 / Probabilistic.
- **C 不正解**: スケールアップは根本解決でなくコスト増。 / Cost without root cause fix.
- **D 不正解**: 機能放棄は過剰。 / Eliminates value.

**参照 / Reference:** Rate limiting・MCP 構造化エラー
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

`get_pricing(product_id, region, tier)` ツールが、同じ引数に対して 1 時間以内なら同じ結果を返します（価格は時間単位で更新）。同じセッション中に同じツールを 12 回呼ぶケースがある。

`get_pricing(product_id, region, tier)` returns the same result within 1 hour (prices update hourly). The same call happens 12 times in a session.

**設問 / Question:**

最も適切な最適化はどれですか？ / Best optimization?

- A) ツールの呼び出しを禁止 / Forbid the call
- B) MCP サーバ側で **TTL 付きキャッシュ**（1 時間）を実装。クライアント側でも prompt cache を利用し、重複した tool_result が cache hit になるよう構成。**ETag / Last-Modified** を返してクライアントが条件付き取得できるようにする / Server-side **TTL cache (1 hour)**; client-side prompt cache so repeated `tool_result`s hit cache. Return **ETag / Last-Modified** for conditional fetching
- C) クライアント側でランダムに省略 / Randomly skip on the client
- D) すべてのツールキャッシュは禁止すべき / All tool caching is forbidden

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

サーバ TTL キャッシュ + ETag + クライアント prompt cache の組み合わせがコスト・レイテンシの両方を最適化。

Combine server TTL cache + ETag + client prompt cache for cost and latency wins.

- **A 不正解**: 機能放棄。 / Loss of function.
- **C 不正解**: ランダム省略はビジネスロジックを壊す。 / Breaks logic.
- **D 不正解**: 事実誤認。冪等で TTL がある呼び出しはキャッシュ可。 / Factually wrong.

**参照 / Reference:** HTTP caching・prompt caching
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

法律事務所の MCP サーバが、`search_case_law(query, jurisdiction)` ツールに加え、よく使われる **プロンプトテンプレート**（「契約条項のリスク要約を Anglo-American 標準で」）を共有したい。

A law-firm MCP server, alongside `search_case_law`, wants to share **prompt templates** (e.g., "summarize contract clause risks per Anglo-American standard").

**設問 / Question:**

最も適切な MCP の機能はどれですか？ / Best MCP feature?

- A) Tools — 引数付きアクション / actions with args
- B) Resources — 静的データ / static data
- C) **Prompts** — サーバが提供する **再利用可能なプロンプトテンプレート**。引数を持ち、クライアント（Claude Desktop / Claude Code）でユーザーが選択して使える / **Prompts** — reusable prompt templates with arguments, surfaced for users to select in clients (Claude Desktop / Code)
- D) Notifications — 通知 / one-way push

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

MCP の **Prompts** は再利用可能なプロンプトテンプレート提供機能で、クライアントから選択・呼び出し可能。チームでベストプラクティスを共有するのに最適。

MCP **Prompts** are reusable templates surfaced to clients — ideal for sharing team best practices.

- **A 不正解**: ツールは動的アクション。 / Actions, not templates.
- **B 不正解**: リソースは静的データ。 / Static data.
- **D 不正解**: 通知は別目的。 / Different purpose.

**参照 / Reference:** MCP Prompts spec
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

監査ログを供給する MCP サーバが、新しいログエントリ発生時にクライアントに **能動的に通知**する必要があります。クライアントが定期ポーリングするとレイテンシと負荷が問題。

An audit-log MCP server must **push** new log entries to the client; periodic polling adds latency and load.

**設問 / Question:**

最も適切な MCP の機能はどれですか？ / Best MCP feature?

- A) ポーリングを定期的に行う / Use periodic polling
- B) **Resource subscriptions / Notifications**：クライアントがリソース URI を `subscribe` し、サーバは更新時に `notifications/resources/updated` を送信。リアルタイム性とコスト効率が両立 / **Resource subscriptions / notifications**: client `subscribe`s to a resource URI; server emits `notifications/resources/updated` on changes — realtime and efficient
- C) クライアント側で WebSocket を直接張る（MCP 外） / Open a separate WebSocket outside MCP
- D) 通知は MCP では不可能 / Notifications aren't possible in MCP

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

MCP は **resource subscription** と **notification** をサポートしており、サーバプッシュのリアルタイム連携に対応します。

MCP supports **resource subscriptions** and **notifications** for server-pushed real-time integration.

- **A 不正解**: ポーリングはレイテンシと負荷。 / Latency and load issues.
- **C 不正解**: 標準外の経路は管理を分散させる。 / Fragments management.
- **D 不正解**: 事実誤認。 / Factually wrong.

**参照 / Reference:** MCP notifications spec
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

新人エンジニアが MCP サーバを書きました。`tools/list` が 73 個のツールを返し、エージェントがどれを使うか迷い精度が落ちています。

A junior engineer's MCP server returns 73 tools from `tools/list`; the agent struggles to choose, accuracy drops.

**設問 / Question:**

最も適切なリファクタリングはどれですか？ / Best refactoring?

- A) ツール数を増やしてさらに細分化 / Add more granular tools
- B) **責務でグルーピング**して 1 サーバ → 複数 MCP サーバに分割（顧客サーバ・注文サーバ・分析サーバなど）。クライアント側では用途に応じて必要なサーバのみ接続。1 サーバあたりのツール数は **5〜15 程度**を目安に絞り、各ツールの記述を「いつ使うか／いつ使わないか」明示 / **Group by responsibility** and split into multiple MCP servers (customer / orders / analytics); clients connect only those needed. Aim for **5–15 tools per server**; sharpen "when to use / when NOT" in each description
- C) 73 個のツールをすべて 1 つに統合 / Merge all 73 into one mega-tool
- D) MCP を捨てる / Drop MCP

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

ツール数が多すぎると選択精度が下がるのは LLM の既知特性。**サーバ分割 + 用途別接続 + 各 5〜15 ツール**が実務上の経験則。

Too-many-tools degrades LLM tool selection — split servers by responsibility, connect on demand, 5–15 tools each is the rule of thumb.

- **A 不正解**: 細分化はさらに混乱を招く。 / Worse confusion.
- **C 不正解**: 巨大ツールは責務不明。 / Loses clarity.
- **D 不正解**: 過剰反応。 / Throws away value.

**参照 / Reference:** ツール選択精度の経験則
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

社内の機密 MCP サーバが、`get_employee_salary(employee_id)` というツールを公開しています。HR 部門のみアクセス可能であるべきですが、誰でも `tools/list` で見えてしまうのが懸念です。

An internal MCP server exposes `get_employee_salary(employee_id)`; only HR should access, but anyone seeing `tools/list` is a concern.

**設問 / Question:**

最も適切なアクセス制御はどれですか？ / Best access control?

- A) MCP サーバ側で **認証・認可**を実装：クライアント認証時にロールを取得し、`tools/list` 時に **ロールに応じてツールをフィルタリング**して返す。`tools/call` 時にも認可チェック。HR でないユーザーには `get_employee_salary` 自体が存在として見えない / Implement **authn + authz** server-side: detect role at client auth, **filter `tools/list`** based on role, and re-check on `tools/call`. Non-HR users don't even see `get_employee_salary`
- B) クライアント側のシステムプロンプトで「HR でなければ呼ぶな」と指示 / Prompt: "don't call unless HR"
- C) ツール名を難読化 / Obfuscate the tool name
- D) MCP では権限制御不可能 / Permission control isn't possible in MCP

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

機密ツールは **サーバ側で認証・認可・フィルタリング** を実装。クライアント任せは確率的、難読化はセキュリティに非ず（"security through obscurity" は不可）。

Sensitive tools require **server-side authn/authz/filtering**. Client-side is probabilistic; obfuscation is not security.

- **B 不正解**: 確率的、規制不適合。 / Probabilistic.
- **C 不正解**: 難読化はセキュリティではない。 / Not security.
- **D 不正解**: 事実誤認。 / Factually wrong.

**参照 / Reference:** MCP authentication / authorization
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

医療系 MCP サーバ `lookup_patient(name)` が、同名患者がいる場合に複数結果を返します。クライアントエージェントがどの患者か誤判定するリスクがあります。

A medical MCP `lookup_patient(name)` returns multiple matches for same-name patients; the agent risks misidentification.

**設問 / Question:**

最も適切な対応はどれですか？ / Best response?

- A) 1 件目を必ず返す / Always return the first match
- B) ランダムに選ぶ / Pick randomly
- C) すべての候補を構造化（`{ patient_id, name, dob, last4_ssn, last_visit }` のリスト）して返し、**確実な追加識別子**（生年月日や last4 SSN）の確認を要求するメタデータを含める。エージェントには曖昧さを構造化エラー（`needs_disambiguation: true`）で通知 / Return all candidates as structured list (`patient_id, name, dob, last4_ssn, last_visit`) with metadata requesting **definitive identifier** (DOB or last-4 SSN). Signal ambiguity to the agent via `needs_disambiguation: true`
- D) エラーで返さない / Just error out

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

曖昧さは **構造化して上位に伝える**のが正解。患者誤認は重大事故、確実な識別子確認のフローを設計に組み込む。

Ambiguity must be **surfaced structurally** so the caller can disambiguate. Mis-identification is critical.

- **A 不正解**: 1 件目固定は誤識別の温床。 / Sets up errors.
- **B 不正解**: ランダムは論外。 / Unsafe.
- **D 不正解**: エラーは情報を失う。 / Loses info.

**参照 / Reference:** 曖昧性の構造化伝達
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

`run_sql(query)` ツールが任意 SQL を実行できるよう設計されており、エージェントが間違って `DROP TABLE customers;` を実行する可能性があります。

`run_sql(query)` accepts arbitrary SQL; the agent could execute `DROP TABLE customers;`.

**設問 / Question:**

最も適切な堅牢化はどれですか？ / Best hardening?

- A) システムプロンプトで「DROP は使うな」と書く / Prompt: "do not use DROP"
- B) **MCP サーバ側で SQL を解析**し、`SELECT` のみ許可（DDL/DML は拒否）。さらに **読み取り専用の DB ユーザー**で接続し、権限レイヤーでも DROP を不可能化。重要操作は別ツール `migrate_schema(plan)` で承認フロー付き / **Parse SQL server-side** and allow only `SELECT` (reject DDL/DML). Connect as a **read-only DB user** so DROP is impossible at the privilege layer. Sensitive ops go through a separate `migrate_schema(plan)` tool with approval workflow
- C) Few-shot で安全な例を見せる / Show safe examples via few-shot
- D) 任意 SQL を許容するのが本来の柔軟性 / Arbitrary SQL is the desired flexibility

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

任意 SQL は危険。**サーバ側パース + 最小権限 DB ユーザー + 操作別ツール分離**で多層防御。

Arbitrary SQL is dangerous. Defend with **server-side parsing + least-privilege DB user + tool separation by op**.

- **A 不正解**: プロンプトは確率的。 / Probabilistic.
- **C 不正解**: Few-shot 同上。 / Same.
- **D 不正解**: 柔軟性の名で不正コマンドを許容するのは論外。 / Unsafe.

**参照 / Reference:** SQL injection・最小権限原則
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

ロジスティクス MCP サーバが Python と Go の両方で実装されています（チームの言語選択）。クライアントから見て、どちらが動作しているかは透明であるべきです。

A logistics MCP exists in both Python and Go (team choice). The client must remain agnostic to which runs.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) 言語によって異なるツール名を公開 / Different tool names per language
- B) **MCP プロトコル仕様への厳密準拠**で言語透過性を担保。ツール名・スキーマ・エラーモデル・トランスポートを一致させ、契約テストで両実装の挙動同等性を CI で検証 / **Strict MCP protocol conformance** ensures language transparency. Same tool names, schemas, error models, transports; CI runs **contract tests** verifying behavior equivalence
- C) クライアントを Python 専用にする / Restrict client to Python
- D) 1 つの言語に統一する / Force one language

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

MCP は言語非依存のプロトコル。実装言語に関わらず **契約に厳密準拠** + **契約テスト**で透過性を担保。

MCP is language-agnostic — strict spec conformance + contract tests preserve client transparency.

- **A 不正解**: 言語別命名は契約違反。 / Violates contract.
- **C 不正解**: クライアント制約は MCP の利点を放棄。 / Defeats MCP value.
- **D 不正解**: チームの選択を奪う過剰反応。 / Overreach.

**参照 / Reference:** MCP spec conformance・契約テスト
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

トランザクション処理 MCP サーバの `process_payment` ツールが、たまに 1〜2 秒応答しないことがあります。エージェントは即座にリトライしてしまい、二重決済リスクがあります。

A `process_payment` MCP tool occasionally hangs 1–2 seconds. The agent retries instantly, risking double payment.

**設問 / Question:**

最も適切な対策はどれですか？ / Best mitigation?

- A) クライアントは絶対にリトライしない / Never retry
- B) サーバ側で必ずタイムアウトしないようにする / Make the server never time out
- C) ツールを **冪等**に設計（`idempotency_key` 必須）し、エラー応答に **`retryable: true/false`** と `retry_after_ms` を含める。クライアントは `retryable: true` のときのみ指数バックオフでリトライし、同じ冪等キーを使うことでサーバ側で重複排除 / Design the tool **idempotently** (require `idempotency_key`); return `retryable: true/false` and `retry_after_ms` in errors. Client retries with exponential backoff **only when `retryable: true`**, reusing the same key so the server dedupes
- D) `claude-opus-4-6` にすれば二重決済は起きない / `claude-opus-4-6` prevents double payment

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

二重決済防止は **冪等キー + retryable フラグ + 指数バックオフ** が標準。サーバ・クライアント両側で冪等性を担保。

Double-payment safety = idempotency key + retryable flag + exponential backoff, enforced on both sides.

- **A 不正解**: 完全リトライ禁止は瞬間障害でビジネスインパクト。 / Hurts continuity.
- **B 不正解**: 「絶対にタイムアウトしない」は技術的に不可能。 / Impossible.
- **D 不正解**: モデル変更は冪等性に無関係。 / Irrelevant.

**参照 / Reference:** Idempotency-Key・サーキットブレーカー・retryable
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

社内 MCP サーバが Claude Code に接続されているが、ユーザーが認証情報を入力しないと使えないツールが含まれています。初回接続時にどのようにフローを設計するか。

An internal MCP server requires user credentials for some tools. How should first-connection flow be designed?

**設問 / Question:**

最も適切なフローはどれですか？ / Best flow?

- A) MCP の **OAuth 2.0 / Bearer 認証**フローでユーザーをブラウザに誘導しトークン取得、トークンは安全なストレージ（OS キーチェーン / 暗号化ファイル）に保存。トークン期限切れ時は自動リフレッシュ、リフレッシュ失敗時は再認証フロー。**設定ファイルにトークンを直接書かない** / Use MCP **OAuth 2.0 / Bearer** flow: redirect user to browser for token, store in OS keychain / encrypted store. Auto-refresh on expiry; re-auth on refresh failure. **Never write tokens directly into config files**
- B) 設定ファイルにトークンを平文で書く / Plaintext token in config
- C) ユーザーに毎回トークンを手入力させる / Have the user paste a token each time
- D) MCP は認証をサポートしない / MCP doesn't support auth

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

セキュアな MCP 認証は **OAuth/Bearer + キーチェーン保存 + 自動リフレッシュ**が標準。

Secure MCP auth: **OAuth/Bearer + OS keychain + auto-refresh**.

- **B 不正解**: 平文保存はセキュリティ違反。 / Insecure.
- **C 不正解**: UX 過剰、運用上不可能。 / UX disaster.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** MCP authentication
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

MCP サーバの `list_files(directory)` ツールが、ディレクトリ中の 50,000 ファイルすべてを返してコンテキストを破壊することがあります。

`list_files(directory)` returns 50,000 entries, blowing up context.

**設問 / Question:**

最も適切な API 設計はどれですか？ / Best API design?

- A) すべてを 1 度に返す / Return all at once
- B) **ページネーション**（`page`, `page_size`）と **フィルタ**（`name_pattern`, `modified_after`）と **デフォルト上限**（例：100 件）を実装。`has_more` フラグと `next_cursor` で続きを取得可能に。**結果がデフォルト上限を超える場合**はメタデータで通知 / Add **pagination** (`page`, `page_size`), **filters** (`name_pattern`, `modified_after`), and **default cap** (e.g., 100). Return `has_more` + `next_cursor`; signal via metadata when results exceed defaults
- C) クライアントが切り詰める / Client-side truncation
- D) ディレクトリリスト機能を捨てる / Drop the feature

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

大規模リスト返却は **ページネーション + フィルタ + デフォルト上限**が API 設計の基本。

Large-list APIs require **pagination + filters + default caps**.

- **A 不正解**: コンテキスト破壊。 / Blows up context.
- **C 不正解**: 制御不能な切り詰め。 / Uncontrolled.
- **D 不正解**: 機能放棄。 / Loss of value.

**参照 / Reference:** REST/MCP API 設計
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

Claude Agent SDK で 7 個の MCP サーバ + 5 個の組み込みツールを使う構成。エージェントの **ツール選択精度**が想定より 20% 低い。

An Agent SDK setup has 7 MCP servers + 5 built-in tools. Tool-selection accuracy is 20 points lower than expected.

**設問 / Question:**

最も効果的な改善はどれですか？ / Best improvement?

- A) より高性能なモデルに切り替え / Switch to a higher-tier model
- B) **タスクごとに必要なツールサブセットだけを `allowed_tools` で公開**する。コーディネーターは特定タスク用のサブエージェントへ `Task` で委譲し、各サブエージェントは自分の責務に必要な 3〜7 ツールのみアクセス可。さらに各ツールの `description` を「いつ使う／使わない」で改善 / **Expose per-task tool subsets via `allowed_tools`**. Coordinator delegates via `Task` to specialist subagents, each with only 3–7 tools relevant to scope. Tighten descriptions with "when to use / when NOT"
- C) すべてのツールをマージしてカテゴリ別に整理 / Merge tools into mega-tools by category
- D) ツールを増やしてカバレッジ向上 / Add more tools

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

ツール多すぎ問題は **責務別サブエージェント + 限定 `allowed_tools` + 明確な記述**で構造的に解決。

Tool-overload is solved via **specialist subagents with narrow `allowed_tools` + clear descriptions**.

- **A 不正解**: モデルアップは選択肢爆発を解決しない。 / Doesn't fix overload.
- **C 不正解**: 巨大ツールは責務不明。 / Unclear.
- **D 不正解**: 増やすのは逆効果。 / Worse.

**参照 / Reference:** ツール選択精度・サブエージェント設計
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

`get_user(user_id)` ツールの結果が長大で、`tool_result` が 80,000 トークンを占めます。エージェントは数フィールド（名前、メール、最終ログイン）しか必要としていません。

`get_user(user_id)` returns 80K tokens; the agent needs only `name`, `email`, `last_login`.

**設問 / Question:**

最も適切な改善はどれですか？ / Best improvement?

- A) クライアントで全結果を受け取り、後で必要な分だけ使う / Receive all then use what's needed
- B) ツールに `fields` パラメータを追加（GraphQL 的）：`get_user(user_id, fields=["name","email","last_login"])` で **必要なフィールドのみサーバ側で返す**。デフォルトは最小セット、フル取得は明示要求時のみ / Add a `fields` parameter (GraphQL-like): `get_user(user_id, fields=["name","email","last_login"])` so the server returns only requested fields. Default to a minimal set; full payload only on explicit request
- C) 大きすぎるからこのツールは使わない / Avoid this tool
- D) 切り詰めて表示 / Truncate display-side

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**フィールド射影**（projection）でサーバ側からトークン浪費を抑える。GraphQL のフィールド選択や JSON:API のスパース・フィールドセットと同じ思想。

**Field projection** at the server avoids token waste — same idea as GraphQL field selection / JSON:API sparse fieldsets.

- **A 不正解**: 取得後の選別ではトークン消費は変わらない。 / Doesn't save tokens.
- **C 不正解**: ツールの利点を放棄。 / Loss of value.
- **D 不正解**: 表示切り詰めは LLM コンテキスト消費を変えない。 / Doesn't help context.

**参照 / Reference:** Field projection / sparse fieldsets
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

複数の MCP サーバを持つチームが、組織全体で **ツール命名規約**を統一したい。現状は `getOrder`, `fetch_order`, `lookup-order`, `order_get` などバラバラ。

A team with multiple MCP servers wants org-wide **tool naming conventions**. Today: `getOrder`, `fetch_order`, `lookup-order`, `order_get` — all over the map.

**設問 / Question:**

最も適切な命名規約はどれですか？ / Best naming convention?

- A) **動詞_対象**（snake_case）で統一：`get_order`, `create_invoice`, `cancel_subscription`。動詞で意図を明示し、対象で何のリソースかを示す。boolean を返すツールは `is_*` / `has_*`、検索系は `find_*` / `search_*`、非破壊は `get_*` / `list_*`、破壊的は `create_*` / `update_*` / `delete_*` で揃える / **`verb_object`** in snake_case: `get_order`, `create_invoice`, `cancel_subscription`. Verb signals intent, object names the resource. Booleans use `is_*` / `has_*`; queries use `find_*` / `search_*`; safe reads use `get_*` / `list_*`; mutations use `create_*` / `update_*` / `delete_*`
- B) 各エンジニアが好きな名前を付ける / Each engineer picks freely
- C) `tool1`, `tool2` のように番号で / Number them: `tool1`, `tool2`
- D) 命名規約は不要 / Conventions aren't needed

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

LLM のツール選択精度は **命名の一貫性に大きく依存**します。動詞_対象パターンは可読性・意図伝達・予測可能性で最良。

LLM tool selection depends heavily on **naming consistency**. `verb_object` snake_case maximizes readability, intent, and predictability.

- **B 不正解**: 揺れは選択精度を下げる。 / Inconsistency degrades selection.
- **C 不正解**: 番号は意図伝達ゼロ。 / Zero semantics.
- **D 不正解**: 規約なしは混乱を生む。 / Causes confusion.

**参照 / Reference:** API 命名規約・MCP best practices
</details>

---

> **前のドメイン / Previous:** [`domain1_agent_architecture.md`](./domain1_agent_architecture.md) | **次のドメイン / Next:** [`domain3_claude_code_workflows.md`](./domain3_claude_code_workflows.md)
