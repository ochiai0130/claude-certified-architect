# Domain 1: エージェントアーキテクチャとオーケストレーション / Agent Architecture and Orchestration

> 配点比率 / Weight: **27%**
> 問題数 / Questions: **5**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- エージェントループと `stop_reason` 制御 / Agent loop and `stop_reason` control flow
- ハブアンドスポーク構成・サブエージェント分離 / Hub-and-spoke topology, subagent isolation
- 決定論的ガードレール（フック・MCP・プロンプト）の責務分離 / Deterministic guardrails — separation of concerns across hooks, MCP, prompts
- 固定パイプライン vs 動的タスク分解 / Fixed pipelines vs adaptive task decomposition
- 長時間ジョブのセッション管理（`--resume`, `fork_session`, 外部チェックポイント） / Long-running session management

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

あなたは米国の証券会社で、Claude Agent SDK を用いた株式注文執行エージェントを構築しています。エージェントは `verify_account` → `check_compliance` → `place_order` の 3 ツールを順に呼びます。本番環境で稀に、`place_order` が `tool_use` を返した直後にネットワーク断で API レスポンスが失われ、エージェントセッションがクラッシュする事象が報告されました。MiFID II / SEC Rule 15c3-5 の観点から、**同一注文が二重執行されること** は重大インシデントです。

You are building a stock order-execution agent using the Claude Agent SDK at a US broker-dealer. The agent calls `verify_account` → `check_compliance` → `place_order` in sequence. In production, occasional network drops after `place_order` emits a `tool_use` cause the session to crash before the API response is recorded. Under MiFID II / SEC Rule 15c3-5, **double execution of the same order** is a critical incident.

**設問 / Question:**

二重執行を確実に防ぐために、最も適切なアーキテクチャ上の対策はどれですか？

Which architectural countermeasure is most appropriate to reliably prevent double execution?

- A) `place_order` ツール呼び出し前にシステムプロンプトへ「同じ注文を二度送信してはならない」という強い指示を追加し、温度を 0 に下げる / Add a strong system-prompt instruction "never resubmit the same order" before each `place_order` call and set temperature to 0
- B) `place_order` のリクエストにクライアント生成の冪等キー（`Idempotency-Key`）を含めて発注 API 側で重複排除し、Agent SDK では `--resume` でセッションを復元したうえで前回の `tool_use` ID を見て再呼び出しの要否を判定する / Include a client-generated `Idempotency-Key` in the `place_order` request so the order API deduplicates server-side, then use `--resume` to restore the session and inspect the prior `tool_use` ID to decide whether to re-issue
- C) `place_order` のレスポンスを待つ間はクラッシュしないよう、上位のコーディネーターエージェントを別プロセスで動かしハートビートで監視する / Run the coordinator agent in a separate process with heartbeat monitoring so it doesn't crash while waiting for `place_order`
- D) `claude-haiku-4-5` から `claude-opus-4-6` に切り替えて、より高い指示遵守率により二重発注を回避する / Switch from `claude-haiku-4-5` to `claude-opus-4-6` so higher instruction-following reduces double submissions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

二重執行は **規制リスクであり、確率的（プロンプトベース）対策では絶対に防げません**。冪等性は (1) 注文 API 側のサーバ重複排除（業務的真実）と、(2) Agent SDK 側のセッション復元 (`--resume`) によるクライアント状態回復 の **二層**で担保するのが鉄則です。冪等キーは前回 `tool_use` 時点で生成済みであれば、リトライしても発注 API 側で `200 OK` （または `409 Conflict`）が返って二重執行は起こりません。

Double execution is a **regulatory risk and cannot be reliably prevented by probabilistic (prompt-level) controls**. Idempotency must be enforced in **two layers**: (1) server-side deduplication via an idempotency key (the source of truth), and (2) client-side recovery via `--resume` so the agent can inspect the prior `tool_use` rather than re-emit it.

- **A 不正解**: プロンプトと温度 0 では確率的にしか効かず、規制で要求される確実性は出ません。実装上のフォールバックを置き換えてはいけません。 / Prompt + temperature is probabilistic and cannot meet regulatory determinism.
- **C 不正解**: 別プロセス化はクラッシュ範囲を局所化するだけで、ネットワーク断後の冪等性は解決しません。発注 API が二重に叩かれる根本問題が残ります。 / Process isolation reduces blast radius but doesn't address idempotency at the order API.
- **D 不正解**: モデル変更は精度差はあれ二重執行を「確実に防ぐ」保証はゼロ。コンプライアンス要件はモデル能力で吸収すべきではありません。 / Model upgrades are not deterministic guarantees; compliance must not depend on model capability.

**参照 / Reference:** `guide_ja.md` 「3.1 エージェントループ」「7. 信頼性」、Anthropic API Idempotency-Key ヘッダ仕様
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

医療請求コード化 SaaS を構築中で、コーディネーターエージェントが患者カルテから ICD-10 / CPT コードを抽出します。HIPAA 準拠のため、PHI（Protected Health Information）にアクセスできるサブエージェントは最小限に制限する必要があります。現在の構成は次のとおりです：

You are building a medical billing SaaS where a coordinator extracts ICD-10 / CPT codes from patient charts. For HIPAA compliance, only the minimum number of subagents may access PHI. Current topology:

- `coordinator` (PHI 直アクセス可 / has direct PHI access)
- `icd10_lookup` サブエージェント — コード辞書を検索 / looks up code dictionary
- `cpt_lookup` サブエージェント — CPT コードを検索 / looks up CPT codes
- `audit_logger` サブエージェント — 監査ログ書き込み / writes audit log

カルテ全文をコーディネーターから 3 つのサブエージェント全てに `Task` ツールの `prompt` パラメータでそのまま渡している状態です。

The full chart text is currently passed verbatim from the coordinator to all three subagents via the `Task` tool's `prompt` parameter.

**設問 / Question:**

HIPAA の最小権限原則と監査要件を満たす上で、最も適切な改善はどれですか？

Which is the most appropriate improvement to satisfy HIPAA's minimum-necessary rule and audit requirements?

- A) コーディネーターのシステムプロンプトに「サブエージェントには必要最小限の情報のみ渡す」と書く / Add to the coordinator's system prompt: "pass only minimum necessary information to subagents"
- B) `icd10_lookup` と `cpt_lookup` には PHI を含まない**抽出済み臨床所見**（脱識別済み）のみを `Task` の `prompt` に含める。`audit_logger` には患者 ID のハッシュとタイムスタンプ・コード結果のみを渡す。各サブエージェントの `allowed_tools` は辞書検索ツールのみに絞る / Pass only **de-identified extracted findings** to `icd10_lookup` / `cpt_lookup`, and only patient-ID hash + timestamp + code result to `audit_logger`. Restrict each subagent's `allowed_tools` to its lookup tool
- C) すべてのサブエージェントを 1 つに統合し、コーディネーターから 1 回だけ呼び出す / Merge all three subagents into one and call it once from the coordinator
- D) コーディネーターを `claude-opus-4-6` で動かし、サブエージェントは `claude-haiku-4-5` にしてコストを最適化する / Run the coordinator on `claude-opus-4-6` and the subagents on `claude-haiku-4-5` for cost optimization

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

HIPAA の "minimum necessary" 原則は **アーキテクチャレベルでの情報フロー制御** を求めます。サブエージェントには **タスク遂行に必要な最小情報のみ**を `Task` の `prompt` で渡し、`allowed_tools` を絞ることで、たとえ LLM が指示を逸脱してもアクセス不可能な状態を作ります（決定論的境界）。`audit_logger` は PHI 自体を記録してはならず、ハッシュ・メタデータのみ受け取るのが正しい設計です。

HIPAA's minimum-necessary rule demands **architecture-level information flow control**. Subagents should receive only the minimum data they need via `Task`'s `prompt`, with `allowed_tools` restricted so PHI is unreachable even if the model deviates (deterministic boundary).

- **A 不正解**: プロンプト指示は確率的で HIPAA 監査では不十分。「指示したのに漏れた」は防御策になりません。 / Prompt instructions are probabilistic and inadequate for HIPAA.
- **C 不正解**: 統合するとサブエージェント全体が PHI にアクセスする逆方向の改悪。役割分離と最小権限が崩れます。 / Merging expands PHI exposure rather than minimizing it.
- **D 不正解**: モデル選定はコスト/性能のトレードオフであり、HIPAA 準拠とは無関係です。 / Model tier is unrelated to HIPAA compliance.

**参照 / Reference:** `guide_ja.md` 「3.3 ハブアンドスポーク」「3.4 Task ツール」、HIPAA §164.502(b) Minimum Necessary
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

国境を越える KYC（Know Your Customer）審査を Claude Agent SDK で自動化しています。次の 3 つの要件があります：

You are automating cross-border KYC review with the Claude Agent SDK. There are three requirements:

1. 制裁リスト国（OFAC SDN）の顧客には絶対に口座開設させてはならない / Customers from OFAC-sanctioned countries must never be onboarded
2. 高リスク業種（暗号資産・武器等）は人間レビュアーへエスカレーション / High-risk industries (crypto, arms) escalate to human reviewers
3. 推奨判定（承認/保留/拒否）は柔軟に LLM に任せたい / The recommendation (approve/hold/reject) should remain flexible LLM judgment

実装方針として、決定論が必要な責務とプロンプトに任せる責務を分けようとしています。

You want to separate deterministic responsibilities from those left to the LLM.

**設問 / Question:**

要件 1（制裁リスト遮断）の実装場所として最も適切なのはどれですか？

Where is the most appropriate place to implement requirement 1 (sanctions blocking)?

- A) システムプロンプトに OFAC SDN リストの全文を埋め込んでモデルに参照させる / Embed the full OFAC SDN list in the system prompt and have the model reference it
- B) Few-shot 例として「制裁国は拒否」のケースを 5 件追加する / Add 5 few-shot examples showing "reject sanctioned country"
- C) Agent SDK の `PreToolUse` フックで `create_account` ツール呼び出しを **インターセプト**し、制裁リスト判定を呼んで該当時はツール実行を停止する。同時に MCP サーバ側の `create_account` 実装でも同等チェックを行い、二重に防御する / Intercept `create_account` calls with a `PreToolUse` hook that runs sanctions screening and blocks execution on a hit; **also** enforce the same check inside the MCP server's `create_account` implementation as defense in depth
- D) コーディネーターのサブエージェントに `sanctions_check` を入れ、その出力を見てから次のステップに進むよう指示する / Add a `sanctions_check` subagent and instruct the coordinator to consult its output before proceeding

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

OFAC 違反は刑事責任を伴う最重大規制であり、**確率的に「ほぼ守られる」では足りません**。`PreToolUse` フックは Agent SDK が実際にツールを呼ぶ前に決定論的に介入できる唯一のポイントです。さらに **多層防御**として、MCP サーバ側の実装にも同じ判定を入れることで、エージェント以外の経路（誤呼び出し、別クライアント）からの違反も防ぎます。これは航空業界の "defense in depth" や決済業界の "PCI DSS Layered Security" と同じ思想です。

OFAC violations carry criminal liability; probabilistic enforcement is insufficient. `PreToolUse` hooks are the only point where the Agent SDK can deterministically intercept tool execution. **Defense in depth** dictates also enforcing the check server-side in the MCP implementation, so violations are blocked even from non-agent callers.

- **A 不正解**: SDN リストは数千エントリで毎週更新され、プロンプト埋め込みは現実的でなく、モデルが正確に守る保証もありません。 / The SDN list updates weekly with thousands of entries; prompts cannot reliably enforce it.
- **B 不正解**: Few-shot は意図伝達には効くが、規制遵守の確実性ゼロ。 / Few-shot is suggestive, not deterministic.
- **D 不正解**: サブエージェント＋プロンプト指示は "見るよう指示する" だけで、モデルが見ない/誤読するリスクが残ります。決定論ではない。 / The model "should" consult the result — but might not. Not deterministic.

**参照 / Reference:** `guide_ja.md` 「3.5 Agent SDK のフック」「7.2 決定論 vs 確率論」
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

SOX 法準拠の上場企業で、四半期決算前の **内部統制テスト自動化** に Claude Agent SDK を採用しようとしています。プロセスは「①証憑取得 → ②サンプリング → ③統制実施確認 → ④例外検出 → ⑤監査調書生成」の 5 ステップで、外部監査人がワークフローのトレーサビリティを要求しています。同じチームの別プロジェクトでは、新薬候補の科学文献調査エージェントも開発しており、こちらは探索的に進める必要があります。

A SOX-compliant public company is automating quarterly internal control testing with the Claude Agent SDK. The process has five fixed steps (evidence retrieval → sampling → control execution check → exception detection → audit workpaper generation), and external auditors require workflow traceability. A separate project on the same team is building a literature-research agent for drug candidates that must proceed exploratorily.

**設問 / Question:**

それぞれのプロジェクトに適したタスク分解戦略の組み合わせはどれですか？

Which combination of task-decomposition strategies best fits each project?

- A) 両方とも動的タスク分解（コーディネーターが状況に応じてサブエージェントを呼ぶ） / Both use dynamic task decomposition (coordinator dispatches subagents adaptively)
- B) 両方とも固定パイプライン（決まった順序で 5 ステップ実行） / Both use fixed pipelines (deterministic 5-step sequence)
- C) SOX 統制テストは固定パイプライン（監査追跡のため各ステップが必ず同じ順序・同じ証憑種別で実行）、文献調査は動的タスク分解（探索的にサブエージェントを呼び分ける） / SOX testing uses a fixed pipeline (auditable: same steps, same evidence types); literature research uses dynamic decomposition (exploratory subagent dispatch)
- D) SOX 統制テストは動的タスク分解（柔軟性のため）、文献調査は固定パイプライン（一貫性のため） / SOX uses dynamic decomposition (for flexibility); literature research uses a fixed pipeline (for consistency)

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

SOX 統制下の自動化は外部監査人が **「同じ入力に同じ手順で同じ結論」** を再現可能であることを要求します。これは固定パイプラインの典型的ユースケース（監査トレース・再現性・統制点の明示）です。一方、文献調査は仮説に応じてサブクエリを動的に組み立てる必要があり、固定化すると関連発見を逃します。**規制対応 = 固定 / 探索的タスク = 動的** が原則です。

SOX requires reproducibility: the same input must yield the same procedure and the same conclusion across audit cycles. Fixed pipelines provide that traceability. Literature research, by contrast, must adapt to emerging hypotheses; fixing it would miss relevant findings. Rule of thumb: **regulated = fixed, exploratory = dynamic**.

- **A 不正解**: SOX を動的にすると、各四半期で実行手順が変わり監査人が統制有効性を評価できません。 / Dynamic flow defeats SOX traceability.
- **B 不正解**: 文献調査を固定化すると探索の本質が失われ、関連性の高い発見を逃します。 / Fixing exploratory work loses its value.
- **D 不正解**: 完全に逆。規制対応こそ固定、探索こそ動的が原則。 / Inverted.

**参照 / Reference:** `guide_ja.md` 「3.6 タスク分解戦略」、SOX §404 内部統制要件
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

EU の GDPR 規制下で、多国籍小売チェーンの **24 時間動作する顧客データ削除リクエスト処理エージェント**を Claude Agent SDK で構築しています。1 リクエストあたり平均 3 時間、最大 18 時間の処理時間がかかり、途中でホストの再起動・依存サービスのメンテナンスが発生し得ます。GDPR Art.17（忘れられる権利）の SLA は 30 日以内ですが、監査のためには **どこまで処理が進んだかの正確な再現性** が必要です。

Under GDPR, a multinational retailer is building a Claude Agent SDK pipeline that processes customer data-deletion requests, running 24/7. Each request averages 3 hours (max 18 hours), and host restarts or dependency maintenance occur mid-flight. GDPR Art.17 requires fulfillment within 30 days, and the audit trail must precisely show **how far processing progressed** at any point.

**設問 / Question:**

クラッシュ復旧と監査再現性を両立する最も適切な設計はどれですか？

Which design best balances crash recovery with auditable reproducibility?

- A) 各サブステップ完了時に外部の永続ストレージ（DB ＋ オブジェクトストア）にチェックポイントを書き、再起動時はチェックポイントを読み込んで `--resume <session_id>` で Agent SDK セッションを復元、未完了ステップから再開する。完了済みステップは冪等チェックでスキップ / At each subtask completion, write a checkpoint to external persistent storage (DB + object store). On restart, read the checkpoint and use `--resume <session_id>` to restore the Agent SDK session, resuming from the unfinished step; completed steps are skipped via idempotency checks
- B) `fork_session` を使って 18 時間分のセッションを並列実行し、最も早く終わったものを採用する / Use `fork_session` to run 18-hour-worth of sessions in parallel and pick the fastest completion
- C) 失敗したらゼロから再実行することにし、コンテキストをすべてシステムプロンプトに保存する / On failure, restart from scratch; persist all context inside the system prompt
- D) `claude-opus-4-6` の長コンテキストウィンドウに全履歴を入れ続ければ、セッション復元は不要 / Keep all history in `claude-opus-4-6`'s long context window — session restoration becomes unnecessary

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

長時間ジョブ × 規制要件では、**Agent SDK の `--resume` だけでは不十分**です。`--resume` はモデルのセッション履歴を復元しますが、外部システム側で「どのステップが完了したか」「どの顧客レコードが削除済みか」を知る業務的真実は、**外部チェックポイント** に書く必要があります。冪等性チェックと組み合わせることで、再開時に同じ削除を二重実行することなく、正確な進捗位置から続行でき、監査人にも提示できます。

For long jobs under regulation, `--resume` alone is **not enough**. `--resume` restores the model's conversational state, but the business truth — which steps completed, which records were already deleted — must be persisted to **external checkpoints**. Combined with idempotency checks, this enables exact resumption without double-deletion and produces an auditable trail.

- **B 不正解**: `fork_session` は仮説分岐（ "what-if" 探索）の用途であり、長時間ジョブの並列高速化目的ではありません。同じ顧客データを並列に削除する設計はそれ自体不正です。 / `fork_session` is for branching hypotheses, not for parallelizing destructive operations.
- **C 不正解**: ゼロから再実行は 30 日 SLA を超過するリスクと、二重削除（既に削除した連携先との不整合）リスクがあります。 / Restarting risks SLA breach and inconsistency with already-deleted upstream systems.
- **D 不正解**: コンテキストウィンドウは "中間消失効果" やコスト爆発の問題があり、永続化の代替にはなりません。プロセスクラッシュ時には消えます。 / Context windows suffer lost-in-the-middle and cost issues, and vanish on crash.

**参照 / Reference:** `guide_ja.md` 「3.7 セッション管理（resume / fork_session）」「7. 信頼性」、GDPR Art.17
</details>

## 問題 6 / Question 6

**シナリオ / Scenario:**

決済処理エージェントが `tool_use` を 1 つ返した直後、レスポンス本文の途中で `stop_reason: "max_tokens"` で打ち切られました。その `tool_use` ブロック自体は完結している（`input` の JSON が valid）が、その後に続くはずのアシスタントの説明文が途切れています。

A payment agent emitted one `tool_use` and then `stop_reason: "max_tokens"` truncated the response. The `tool_use` block itself is complete (valid `input` JSON), but the assistant explanation that should follow is cut off.

**設問 / Question:**

最も適切な処理はどれですか？ / Which handling is most appropriate?

- A) `tool_use` の入力で示されたツールを実行し、結果を `tool_result` として返してエージェントループを継続する。打ち切られた説明文は `max_tokens` を増やして再生成する必要はない（ツール呼び出しは valid だから） / Execute the tool indicated by the `tool_use` input, return the result as `tool_result`, and continue the agent loop; no need to regenerate the truncated explanation since the tool call itself is valid
- B) 同じリクエストを `max_tokens` を増やして最初からやり直す / Restart the whole request with a higher `max_tokens`
- C) `tool_use` を破棄して `tool_result` も返さずに次のターンに進む / Discard the `tool_use` and skip directly to the next turn without a `tool_result`
- D) ツールを実行せず、ユーザーに「途中で切れました」と報告して終了する / Don't execute; report "truncation" to the user and exit

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`stop_reason: "max_tokens"` でも、`tool_use` ブロックが構文的に valid に閉じていれば、それは実行可能な意思決定です。本来、`tool_use` の後にアシスタントが説明文を出すかどうかは API の使い方次第で、業務ロジックに不要なら問題ありません。Anthropic 公式のガイドラインでは「`tool_use` が完結していれば実行 → `tool_result` で継続」が標準。

A `max_tokens` stop with a syntactically complete `tool_use` block is still a valid action: execute and continue with `tool_result`.

- **B 不正解**: 不必要なリトライでコスト・レイテンシを浪費。 / Wasteful retry.
- **C 不正解**: `tool_result` を返さないとエージェントが「結果を待つ状態」のまま壊れる。 / Skipping breaks the loop contract.
- **D 不正解**: 業務継続性を不必要に損なう。 / Hurts continuity unnecessarily.

**参照 / Reference:** `guide_ja.md` 「1.3 stop_reason」「3.1 エージェントループ」
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

不動産ローン審査エージェントが、外部信用機関 API の照会に時間がかかる際に `stop_reason: "pause_turn"` を返すケースがあります（長時間ツール実行）。SLA は 1 申請 90 秒以内、複数申請が同時に滞留すると顧客体験が悪化します。

A mortgage underwriting agent occasionally returns `stop_reason: "pause_turn"` while a slow credit-bureau API call is in flight. The SLA is 90s per application; backlogs degrade UX.

**設問 / Question:**

最も適切な対応はどれですか？ / Which is the most appropriate handling?

- A) すべての `pause_turn` で即時タイムアウトを発動し、エージェントセッションを即終了する / Immediately abort the session on every `pause_turn`
- B) `pause_turn` を受けたらクライアントは保留状態を維持し、ツール結果が返り次第エージェントループを継続する。同時に **タイムアウト + サーキットブレーカー**を別レイヤーで実装し、外部 API 障害時には `pause_turn` から決定論的に脱出して劣化応答（キャッシュ済み信用情報や手動審査エスカレーション）を返す / Hold the session on `pause_turn` and resume the loop when the tool result arrives. Independently, implement **timeout + circuit breaker** so that during external-API outages the loop deterministically exits with a degraded response (cached credit / manual-review escalation)
- C) `pause_turn` を `tool_use` と同じ意味で扱い、即座にダミー結果を返す / Treat `pause_turn` the same as `tool_use` and return a dummy result immediately
- D) リトライを 100 回まで試行する / Retry up to 100 times

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

`pause_turn` は長時間ツール実行を許容するシグナル。クライアントは結果待ちで保留する設計が正解。SLA を守るには **別レイヤーで** タイムアウトとサーキットブレーカーを実装し、確実に劣化応答へ落とせる経路を持つ必要があります。

`pause_turn` signals long-running tool execution; client holds and resumes when the result arrives. SLA enforcement requires a **separate** timeout + circuit-breaker layer that deterministically degrades.

- **A 不正解**: 即時中断は誤判定で正常な長時間処理まで壊す。 / Mass aborts kill normal long ops.
- **C 不正解**: ダミー結果は外部 API の真実と異なり、貸付判断を歪める。 / Dummy results corrupt underwriting.
- **D 不正解**: 100 回リトライはコスト爆発・SLA 違反。 / Cost and SLA disaster.

**参照 / Reference:** `guide_ja.md` 「1.3 stop_reason」「7. 信頼性 / サーキットブレーカー」
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

サプライチェーン分析のコーディネーターが 1 ターンで 3 つの `tool_use` を並列に出します（在庫照会・物流照会・取引先信用照会）。3 つのうち 1 つだけ失敗した場合の `tool_result` 設計を決めなければなりません。

A supply-chain coordinator emits 3 parallel `tool_use` blocks per turn (inventory, logistics, supplier credit). One fails; you must choose how to return `tool_result` blocks.

**設問 / Question:**

最も適切な設計はどれですか？ / Which is the most appropriate design?

- A) 失敗した 1 つの `tool_result` を省略し、成功した 2 つだけを返す / Omit the failed `tool_result`; return only the two successes
- B) 3 つすべてに同じ `tool_use_id` を付けて返す / Reuse the same `tool_use_id` for all three results
- C) 各 `tool_use` に対応する `tool_result` を **すべて 1 つの user メッセージに含めて返す**。失敗した 1 つは `is_error: true` と構造化エラー（`errorCategory`, `retryable`, `partial_results`）を含めて、コーディネーターが部分結果から推論を続けられるようにする / Return **all three `tool_result` blocks in a single user message**, with the failure marked `is_error: true` and structured error metadata (`errorCategory`, `retryable`, `partial_results`) so the coordinator can reason over partial success
- D) 失敗時はコーディネーターを終了させる / Terminate the coordinator on any failure

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

並列 `tool_use` の API 規約は「**対応する `tool_result` をすべて次の user メッセージにまとめて返す**」。失敗ケースは `is_error: true` と構造化メタデータで部分結果と再試行可否を伝え、エージェントが情報のある状態で次の判断を下せるようにします。

API contract: parallel `tool_use` requires returning **all** corresponding `tool_result` blocks in the next user message. Mark failures with `is_error: true` and structured metadata so the agent can act on partial success.

- **A 不正解**: 省略すると API が「未対応 ID」エラーで落ちる。 / Omitting causes a protocol error.
- **B 不正解**: ID は一対一対応必須。再利用は不正。 / IDs must match 1-to-1.
- **D 不正解**: 1 失敗で全停止は graceful degradation の原則に反する。 / Violates graceful degradation.

**参照 / Reference:** `guide_ja.md` 「3.4 サブエージェント・並列ツール呼び出し」
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

医療スケジューリングエージェントの `AgentDefinition` で、現在は `allowed_tools` に CRM 系・カレンダー系・処方箋発行系の 12 ツールを並べています。法務監査で「処方箋発行はこのエージェントの責務外」と指摘されました。

A medical scheduling agent's `AgentDefinition` currently lists 12 tools across CRM, calendar, and prescription-issuance categories. Legal audit found prescription issuance is **outside this agent's scope**.

**設問 / Question:**

最も適切な対応はどれですか？ / Which is the best response?

- A) システムプロンプトに「処方箋発行はしないでください」と書き加える / Add a system-prompt instruction "do not issue prescriptions"
- B) `allowed_tools` から処方箋発行系のツールを **削除**し、責務を持つ別の `AgentDefinition`（処方箋エージェント）に移動。コーディネーターからの委譲は明示的な `Task` 経由のみとする / **Remove** prescription tools from `allowed_tools` and move them to a separate `AgentDefinition` (prescription agent); coordinator delegates only via explicit `Task`
- C) 処方箋ツールを残しつつ、Few-shot で「使うな」を教える / Keep the tools but teach "don't use" via few-shot
- D) すべてのツールを 1 つのエージェントに集めて簡潔さを優先する / Consolidate all tools into one agent for simplicity

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

`allowed_tools` は **エージェントが物理的に呼び出せるツールの集合**を定めるアーキテクチャ境界です。法務監査が責務外と判断した能力はリストから削除して **不可能化**するのが最も信頼性の高い対応。プロンプトや Few-shot は確率的で、規制境界に向かない。

`allowed_tools` is an **architectural boundary** of what the agent can physically call. Removing out-of-scope tools makes misuse impossible. Prompt-level mitigations are probabilistic.

- **A 不正解**: プロンプト指示は逸脱可能、規制境界に不向き。 / Promptly bypassed.
- **C 不正解**: Few-shot も同上。 / Same issue.
- **D 不正解**: 集約は責務分離の真逆。 / Inverted from separation of duties.

**参照 / Reference:** `guide_ja.md` 「3.2 AgentDefinition」「allowed_tools の設計」
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

企業向け契約レビューエージェントが、まれに 30 ターン以上のエージェントループに入り、コストが想定の 6 倍になることがあります。原因は曖昧な指示でループ脱出条件が決まらないこと。

An enterprise contract-review agent occasionally enters 30+ turn loops, incurring 6x expected cost. Root cause: ambiguous instructions leave loop-exit conditions undefined.

**設問 / Question:**

最も適切な対応はどれですか？ / Which is the best fix?

- A) `max_turns`（または同等のトークン/ターン上限）を Agent SDK で **明示的に設定**し、上限に達した場合はエージェントを停止して **構造化エラー（`reason: "max_turns_reached"`, 部分結果, 推奨次アクション）** を返す。同時にプロンプトでループ脱出基準（「全 5 セクションのレビュー完了で終了」）を明示 / Set `max_turns` (or token/turn cap) **explicitly** in the Agent SDK; on exhaustion, halt with **structured error** (`reason: "max_turns_reached"`, partial results, recommended next action). Also clarify exit criteria in the prompt ("stop after reviewing all 5 sections")
- B) `max_turns` は設定せず、モデルに頑張らせる / Don't set `max_turns`; let the model decide
- C) `claude-opus-4-6` にすればループに入らない / Switch to `claude-opus-4-6` to avoid loops
- D) すべての契約を 1 ターンで処理させるよう温度を 0 に / Set temperature 0 to one-shot every contract

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

エージェントループには **必ず終端保証**を入れる。`max_turns` は決定論的な安全弁、構造化エラーで上位がリカバリ可能、プロンプトの脱出基準明示で意図的なループ短縮も実現。これらは併用するのが定石。

Agent loops require **deterministic termination guarantees**. Combine `max_turns`, structured halt errors, and explicit exit criteria.

- **B 不正解**: 上限なしはコスト・SLA リスク。 / Cost and SLA risk.
- **C 不正解**: モデル変更でループ問題は解決しない。 / Model swap doesn't fix loop logic.
- **D 不正解**: 温度 0 はループの本質ではない。 / Temperature isn't the issue.

**参照 / Reference:** `guide_ja.md` 「3.1 エージェントループ」「終端保証」
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

GxP 規制下の製薬データ抽出エージェントで、すべての `tool_use` 呼び出しを **規制グレードの監査証跡** として記録する必要があります（FDA 21 CFR Part 11）。プロンプトでログ取得を指示する案が出ています。

A GxP-regulated pharma extraction agent must log every `tool_use` to a **regulatory-grade audit trail** (FDA 21 CFR Part 11). A proposal suggests instructing the model via prompt to log.

**設問 / Question:**

最も適切な実装はどれですか？ / Which implementation is most appropriate?

- A) システムプロンプトで「すべてのツール呼び出しをログに残してください」と指示 / System prompt: "log every tool call"
- B) Agent SDK の **`PostToolUse` フック**で全 `tool_use` 呼び出しと結果を **改ざん耐性のある追記専用ストレージ**（WORM、S3 Object Lock 等）に書き込み、相関 ID・ユーザー ID・タイムスタンプ・SHA-256 を必須項目化。フックは決定論的に実行されるため、モデルがログを「忘れる」ことが構造的に不可能 / Use a **`PostToolUse` hook** to write every tool call + result to **tamper-evident append-only storage** (WORM, S3 Object Lock) with required fields (correlation ID, user ID, timestamp, SHA-256). Hooks run deterministically — the model cannot "forget" to log
- C) ログを取らず、必要時に履歴から再構築 / Skip logging; reconstruct from history later
- D) `claude-opus-4-6` を使えば監査ログは自動生成される / `claude-opus-4-6` auto-generates audit logs

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

21 CFR Part 11 は **完全性・改ざん検知性・原本性**を要求します。これは確率的なプロンプトでは達成できず、**決定論的なフック**で追記専用ストレージに記録するのが正解。

21 CFR Part 11 demands integrity, tamper-evidence, and originality — achievable only via **deterministic hooks** writing to append-only stores.

- **A 不正解**: 確率的で監査では落ちる。 / Probabilistic, audit-failing.
- **C 不正解**: 履歴は失われ得る、原本性なし。 / History is lossy, not authoritative.
- **D 不正解**: モデル能力ではなくインフラの問題。 / Infrastructure, not model.

**参照 / Reference:** `guide_ja.md` 「3.5 Agent SDK のフック」「PostToolUse」、FDA 21 CFR Part 11
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

法律事務所のデューデリジェンスエージェントが、コーディネーター → 5 つのサブエージェント（契約・財務・人事・知財・税務）の構成で動作しています。コーディネーターのコンテキストには対象企業全体の概要が入っており、各サブエージェントには領域別の深掘りを依頼します。

A DD agent at a law firm uses coordinator → 5 subagents (contracts, finance, HR, IP, tax). Coordinator holds the overall company context; each subagent investigates its domain.

**設問 / Question:**

各サブエージェント起動時の最も適切なコンテキスト受け渡しはどれですか？ / Best context-passing per subagent launch?

- A) コーディネーターのコンテキスト全文をすべてのサブエージェントの `prompt` に丸ごと渡す / Pass the entire coordinator context to each subagent's `prompt`
- B) 各サブエージェントの **`prompt` に必要十分な情報のみ**を構造化して含める：①対象企業の identity（社名・業種・規模）、②領域別の調査目的、③前提条件・既知の事実、④期待される出力スキーマ。横断的な事実は再記述、無関係な領域情報は除外 / Include only **necessary-and-sufficient information** in each subagent's `prompt`: ①company identity, ②domain-specific objective, ③known facts and assumptions, ④expected output schema. Repeat shared facts; exclude irrelevant domains
- C) サブエージェントには `prompt` を空で渡し、必要な情報は彼らが自分で取りに行かせる / Pass empty `prompt`; let subagents fetch what they need
- D) サブエージェントを使わず単一エージェントで全領域を処理 / Avoid subagents; one agent handles everything

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

サブエージェントは **新しいコンテキストで起動** するため、必要な情報を `prompt` で明示的に渡す必要があります。多すぎるとトークン浪費・中間消失、少なすぎると必要なツール呼び出しを判断できない。**必要十分**＝ identity・目的・前提・期待出力の 4 点を構造化して渡すのが定石です。

Subagents start with **fresh context**; the `prompt` must explicitly carry necessary information. Too much wastes tokens; too little prevents informed decisions. Standard pattern: identity + objective + known facts + expected output schema.

- **A 不正解**: 全文転送は無関係領域でコンテキスト汚染、コスト増。 / Full transfer pollutes and inflates cost.
- **C 不正解**: 空プロンプトは「自分は何をするか」も不明。 / Empty prompts leave goals unknown.
- **D 不正解**: 5 領域を単一エージェントで処理するとコンテキスト劣化が深刻。 / One agent suffers severe context drift.

**参照 / Reference:** `guide_ja.md` 「3.3 ハブアンドスポーク」「3.4 Task の prompt 設計」
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

製造業の品質検査エージェントが、複数のサブエージェント間で「進行中の検査ロット ID と中間結果」を共有する必要があります。並列実行されるサブエージェント同士は直接通信できません。

A QC agent needs to share "active inspection lot ID + interim results" across subagents executing in parallel; subagents can't talk directly.

**設問 / Question:**

最も適切な共有状態の設計はどれですか？ / Best shared-state design?

- A) コーディネーターのプロンプトに状態を埋め込み、毎回更新する / Embed state in coordinator prompt and update each time
- B) **共有ファイル（`./state/lot_<id>.json`）またはレディスのような外部ストア**にコーディネーターが書き込み、サブエージェントは Read ツールまたは MCP 経由で読み出す。書き込みは **冪等・タイムスタンプ付き** で衝突解決可能にする / Write to a **shared file (`./state/lot_<id>.json`) or external store (Redis)** from the coordinator; subagents read via Read or MCP. Writes are **idempotent and timestamped** for conflict resolution
- C) すべてのサブエージェントを 1 つに統合 / Merge all subagents into one
- D) 環境変数で状態を共有 / Share state via environment variables

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

並列サブエージェントは **新コンテキスト**で起動するため、状態は **外部の共有ストア**経由で渡すのが正攻法です。ファイル / Redis / DB のいずれでも、書き込みの冪等性と順序性を保証する設計が要点。

Parallel subagents start fresh; share state via an **external store** (file / Redis / DB) with idempotent timestamped writes.

- **A 不正解**: コーディネーターのプロンプト埋め込みは並列サブエージェントに伝わらない。 / Coordinator prompt isn't visible inside parallel subagents.
- **C 不正解**: 統合は並列性を捨てる過剰反応。 / Loses parallelism.
- **D 不正解**: 環境変数は実行時変更しにくく、構造化データに不向き。 / Env vars don't suit dynamic structured state.

**参照 / Reference:** `guide_ja.md` 「3.3 並列サブエージェント」「外部ステート」
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

銀行の信用リスク評価で、3 つの独立スコアリング（DTI、信用履歴、担保評価）を並列で実行し、最後に統合判定します。並列実行はレイテンシ最適化のためで、各スコアリングは独立。

A bank's credit-risk evaluation runs 3 independent scorings (DTI, credit history, collateral) in parallel for latency, then merges.

**設問 / Question:**

最も適切なオーケストレーションパターンはどれですか？ / Best orchestration pattern?

- A) シーケンシャルパイプライン（DTI → 信用履歴 → 担保評価） / Sequential pipeline
- B) **Fan-out / Fan-in**：コーディネーターが 1 ターンで 3 つの `Task` を並列起動（fan-out）→ 全結果を待って統合（fan-in）。各サブエージェントは独立コンテキスト・独立失敗で graceful degradation 可能。レイテンシは max(3 並列) = 最も遅い 1 つ / **Fan-out / fan-in**: coordinator launches 3 `Task`s in one turn (fan-out) and aggregates after all complete (fan-in); each subagent has isolated context, independent failure for graceful degradation. Latency = slowest of the 3
- C) 1 つのエージェントに 3 つの計算をさせる / One agent computes all three internally
- D) ユーザーに 3 回問い合わせる / Prompt the user 3 times

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

独立スコアリングは fan-out/fan-in が最適。レイテンシが最も遅いものに収束し、コンテキストも分離できる。

Independent scoring fits fan-out/fan-in: latency bounded by the slowest, contexts isolated.

- **A 不正解**: シーケンシャルは sum(3) で遅い。 / Sequential = sum, slower.
- **C 不正解**: 1 エージェントは並列性ゼロ・コンテキスト混合。 / Loses parallelism and isolation.
- **D 不正解**: ユーザー往復はオフトピック。 / Wrong axis entirely.

**参照 / Reference:** `guide_ja.md` 「3.4 Task ツール / 並列起動」
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

セキュリティインシデント分析エージェントが、攻撃者と思われる外部ホスト群を `whois`・`abuseipdb`・`virustotal` の 3 ツールで照会します。virustotal API が応答せず 30 秒タイムアウト → リトライを 3 回繰り返してエージェントが 2 分間ハング。

A security analysis agent queries `whois` / `abuseipdb` / `virustotal` for suspect hosts. VirusTotal hangs; 30s timeout × 3 retries cause a 2-minute stall.

**設問 / Question:**

最も適切な堅牢化はどれですか？ / Best robustness improvement?

- A) リトライ回数を増やす / Increase retry count
- B) ツール呼び出しに **タイムアウト + 指数バックオフ + サーキットブレーカー**を組み合わせる：1 度目失敗で 1 秒、2 度目で 2 秒、3 度目で開放（既知の停止）。**ホストが unhealthy** とコーディネーターに `is_error: true` で通知し、残り 2 ソースで判定継続 / Combine **timeout + exponential backoff + circuit breaker**: 1s, 2s, then open. Notify coordinator with `is_error: true` "host unhealthy"; continue with the remaining 2 sources
- C) すべてのツール呼び出しを直列化 / Serialize all tool calls
- D) virustotal を必須にし、応答するまで待つ / Make VirusTotal mandatory; block until response

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

外部依存にはタイムアウト・指数バックオフ・サーキットブレーカーの三点セットが標準。失敗を構造化して上位に伝え、graceful degradation でセキュリティ判定を続行。

Standard external-dependency hardening: timeout + exponential backoff + circuit breaker, with structured failure reporting for graceful degradation.

- **A 不正解**: リトライ増は遅延を悪化。 / Worsens latency.
- **C 不正解**: 並列性を捨てるのは過剰反応。 / Loses parallelism.
- **D 不正解**: 単一障害点を作る最悪の選択。 / Creates a SPOF.

**参照 / Reference:** `guide_ja.md` 「7. 信頼性 / サーキットブレーカー」
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

カスタマーサポートエージェントのサブエージェントが「文字数 1.5 万字のシステムプロンプト」を持っています。本番ログでは、初回ターンでツール選択を 1 度間違える率が 11% と高く、トークン消費も大きい。

A support subagent has a 15K-character system prompt. Production logs show 11% wrong-tool-selection on the first turn and high token usage.

**設問 / Question:**

最も適切な改善はどれですか？ / Best fix?

- A) システムプロンプトをさらに増やしてエッジケースを網羅 / Expand the prompt with more edge cases
- B) システムプロンプトを **「役割・ツール選択基準・出力フォーマット」の核心 1〜3 千字** に削り、エッジケースは Few-shot で 3〜4 例提示。詳細仕様は MCP リソースまたは外部ドキュメントとして必要時のみ参照させる。**プロンプトキャッシュ**を有効化し冗長読み込みを削減 / Trim to a **core 1–3K chars** (role, tool-selection criteria, output format); use **3–4 few-shot examples** for edge cases; load detailed specs from MCP resources / external docs only when needed. Enable **prompt caching** to amortize reads
- C) すべての可能性を網羅した分岐を JSON で書く / Encode all branches as JSON in the prompt
- D) システムプロンプトを空にしてユーザー側で指示 / Leave system prompt empty; rely on user messages

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

長すぎるシステムプロンプトは **中間消失効果**で精度低下、コストも上昇。核心を絞り Few-shot でエッジケース、詳細は外部参照、キャッシュ有効化が定石。

Overly long system prompts degrade accuracy (lost-in-the-middle) and inflate cost. Lean core + few-shot + external docs + caching is the standard.

- **A 不正解**: 拡大は逆効果。 / Counter-productive.
- **C 不正解**: 巨大 JSON は同様に劣化。 / Same drift problem.
- **D 不正解**: 空は役割定義の喪失。 / Loses role definition.

**参照 / Reference:** `guide_ja.md` 「3.2 AgentDefinition」「5. プロンプト設計」「プロンプトキャッシュ」
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

複雑な税務計算エージェントで、推論ステップが長く正確性が要求される場合、Extended thinking（拡張思考）を有効化することを検討中。一方、単純な分類タスクには使うべきではない。

A complex tax-calculation agent considers enabling extended thinking for accurate multi-step reasoning. Simple classification tasks should not use it.

**設問 / Question:**

最も適切な使い分けはどれですか？ / Best application?

- A) すべてのタスクで拡張思考を有効化 / Enable for everything
- B) 拡張思考は **明確に多段推論を要する高精度ユースケース**（複雑税務計算・コンプライアンス推論・難易度の高いコード生成）で有効化し、単純タスク（分類・要約・抽出）では無効化。コストとレイテンシのトレードオフを **タスクごとに評価**し、品質指標で正当化する / Enable extended thinking only for **clearly multi-step reasoning, high-precision use cases** (complex tax, compliance reasoning, hard code-gen); disable for simple tasks (classification, summarization, extraction). Evaluate cost/latency tradeoff **per task** with quality metrics
- C) 拡張思考は使わない / Never enable
- D) `claude-haiku-4-5` で常に有効化 / Always enable on `claude-haiku-4-5`

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

拡張思考は **多段推論で精度向上が見込めるが、コストとレイテンシが増える**。タスク特性で使い分け、品質メトリクスで効果を確認するのが原則。

Extended thinking improves multi-step reasoning at cost/latency expense — apply only where benefit is demonstrable.

- **A 不正解**: 全有効化はコスト過剰。 / Wasteful.
- **C 不正解**: 必要な所で使わないのは機会損失。 / Misses gains where needed.
- **D 不正解**: モデル指定は本質ではなくタスク特性。 / Task type is the axis.

**参照 / Reference:** `guide_ja.md` 「extended thinking の使い分け」
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

コーディネーター + 4 サブエージェント（コード生成、テスト生成、ドキュメント生成、レビュー）の構成。ある日「サブエージェントの責務範囲が重なり、同じ作業を 2 つのエージェントが繰り返している」と報告された。

In a coordinator + 4 subagents (code-gen, test-gen, docs-gen, review), responsibilities overlap and two agents redo the same work.

**設問 / Question:**

最も適切な責務分離はどれですか？ / Best clarification of responsibilities?

- A) すべてのサブエージェントに「他と被らないこと」と指示 / Tell each "don't overlap"
- B) 各サブエージェントの **入力契約・出力契約・前提条件**を明文化したスキーマに落とし、`AgentDefinition` の system_prompt と `allowed_tools` を契約に整合させる。コーディネーターは契約に従ってのみ委譲。重複は **構造的に不可能**にする / Express each subagent's **input/output contracts and preconditions** as schemas; align `AgentDefinition` system_prompt and `allowed_tools` to those contracts. Coordinator delegates only per contract — overlap becomes **structurally impossible**
- C) 4 つを 1 つに統合 / Merge to one agent
- D) 重複してもよいことにする / Accept the duplication

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

責務分離は **契約駆動**：入力・出力・前提を明示し、`allowed_tools` でできる範囲を絞ることで構造的に重複を防ぎます。プロンプトでの注意は確率的。

Separation of duties is contract-driven: explicit I/O + preconditions + tool restrictions make overlap structurally impossible.

- **A 不正解**: プロンプト指示は逸脱可能。 / Promptly violated.
- **C 不正解**: 統合は責務分離を捨てる。 / Loses separation.
- **D 不正解**: コスト・レイテンシ・整合性が悪化。 / Worsens cost and consistency.

**参照 / Reference:** `guide_ja.md` 「3.2 AgentDefinition」「責務分離」
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

監査機関のリサーチエージェントが、コーディネーター → ドメインコーディネーター（金融・税務・法務）→ 専門サブエージェント（各 3 つ）という 3 階層構成です。本番で「中間ドメインコーディネーターがコンテキストを大量に飲み込み、最終結果が劣化」する問題が発生。

A 3-tier architecture: coordinator → domain coordinators (finance, tax, legal) → specialist subagents. The intermediate domain coordinators bloat their context and final output degrades.

**設問 / Question:**

最も適切な是正はどれですか？ / Best corrective action?

- A) 3 階層を 2 階層に圧縮（コーディネーター → 専門サブエージェント直結） / Flatten to 2 tiers (coordinator → specialists)
- B) 中間コーディネーターを廃止し、トップコーディネーターが直接全 9 専門エージェントを呼ぶ / Replace intermediates with top-level direct dispatch to 9 specialists
- C) 中間コーディネーターを残しつつ、彼らの責務を **「集約 + サマリ生成」のみ**に絞る。詳細データは専門サブエージェントが **スクラッチパッドファイル**に書き、中間は構造化メタデータのみハンドリング。トップコーディネーターには **凝縮された結果のみ**を返す / Keep the intermediate tier but narrow its scope to **aggregation + summary**; specialists write detailed data to **scratchpad files**; intermediates handle only structured metadata; top-level receives only condensed results
- D) すべての階層で同じプロンプトを使う / Use the same prompt at every tier

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

階層が深いほど **中間で凝縮**することが重要。中間は集約専任にし、詳細は外部ストレージ、メタデータのみで通信するのが定石。

Deep hierarchies require **intermediate condensation** — intermediates aggregate; details live in scratchpads; only metadata flows up.

- **A 不正解**: 9 並列を 1 つで管理は中間消失効果が悪化。 / 9-way fan-out into one degrades.
- **B 不正解**: 同上。 / Same.
- **D 不正解**: 同一プロンプトは責務分離が崩れる。 / Loses separation.

**参照 / Reference:** `guide_ja.md` 「3.3 階層構造」「7.4 サブエージェントによる調査の分離」
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

労務管理エージェントの「給与計算」サブエージェントが、特定エッジケース（時短勤務 + 育休 + 賞与）で計算誤りを返すケースが見つかった。コーディネーターは結果を信用してそのまま支給額として確定。

A payroll subagent miscalculates a specific edge case (reduced hours + parental leave + bonus). The coordinator trusts and finalizes the amount.

**設問 / Question:**

最も適切な堅牢化はどれですか？ / Best hardening?

- A) サブエージェントに「ミスをするな」と強く指示 / Tell the subagent "do not err"
- B) サブエージェントの出力に **検証層**を挟む：①独立した検証ロジック（決定論的計算ライブラリでクロスチェック）、②エッジケースを明示的にカバーするテストスイート、③不一致時はコーディネーター経由で**人間レビューにエスカレーション**。重要な判定は決して LLM 単独で確定させない / Insert a **validation layer** after the subagent: ①independent deterministic library cross-check, ②explicit edge-case test suite, ③escalate to human review on mismatch. Critical determinations are **never** finalized by LLM alone
- C) コーディネーターに同じ計算をやらせて二重チェック / Have the coordinator redo the same calc
- D) `claude-opus-4-6` に切り替えれば直る / Switch to `claude-opus-4-6` to fix it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

ビジネスクリティカルな確定判定は LLM 単独に任せず **決定論的検証層** を挟むのが原則。エッジケースは明示テストで網羅。

Critical decisions never finalize on LLM alone — pair with a deterministic validation layer and explicit edge-case tests.

- **A 不正解**: 確率的指示は規制要件に不適合。 / Probabilistic.
- **C 不正解**: 同じバイアスで同じ誤りを繰り返す。 / Same bias, same error.
- **D 不正解**: モデル変更は決定論的保証なし。 / Not deterministic.

**参照 / Reference:** `guide_ja.md` 「7.3 検証層」「人間レビューの設計」
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

大規模クライアント対応で、コーディネーターから `email_send` ツールを呼べる構成。誤送信時の影響が大きいため、1 セッションあたりの `email_send` 呼び出し回数を制限したい。

In a large enterprise setup, the coordinator can call `email_send`. Misfires have large blast radius; you want to cap `email_send` calls per session.

**設問 / Question:**

最も適切な制限手段はどれですか？ / Best way to cap?

- A) システムプロンプトで「メール送信は最大 5 回まで」と指示 / Instruct in system prompt: "max 5 emails"
- B) `PreToolUse` フックでセッション内の `email_send` 呼び出しをカウントし、**5 回目以降は決定論的にブロック**してエージェントに `is_error: true` と「上限到達 — 人間承認が必要」を返す。同時に **`tool_use_id` ベースの冪等キー**で同じメールの二重送信を防ぐ / Use a `PreToolUse` hook to count `email_send` calls per session and **deterministically block from the 6th onward**, returning `is_error: true` "limit reached — human approval required". Combine with a `tool_use_id`-based idempotency key to prevent duplicates
- C) `email_send` を `allowed_tools` から削除 / Remove `email_send` from `allowed_tools`
- D) ユーザーに送信可否を毎回問う / Ask the user each time

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

ツール呼び出し回数の上限は **フックで決定論的に**実装。プロンプトでは確実性が出ない。冪等キーで二重送信も防ぐ。

Tool-call caps are enforced via **deterministic hooks**, with idempotency keys preventing duplicates.

- **A 不正解**: 確率的で誤送信を防げない。 / Probabilistic.
- **C 不正解**: 完全削除では業務不能。 / Breaks functionality.
- **D 不正解**: UX 過剰で運用が回らない。 / Operational nightmare.

**参照 / Reference:** `guide_ja.md` 「3.5 PreToolUse フック」「冪等性」
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

医療チャット相談エージェントで、応答の長文化により最初のトークンまでのレイテンシ（TTFT）が悪化。一方、最終的な構造化出力（処方推奨 JSON）の正確性は最重要。

A medical chat agent has slow TTFT due to long responses, but the final structured output (prescription recommendation JSON) must remain accurate.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) 応答全体を非ストリーミングで生成して TTFT を犠牲 / Non-streaming throughout; sacrifice TTFT
- B) 説明文部分は **ストリーミング**でユーザーに即時表示し、構造化判定は別ターンの `tool_use` で取得して**独立に検証**してから確定。ユーザー体験と業務ロジックを分離 / **Stream the explanation** for instant TTFT and obtain the structured decision in a separate `tool_use` turn that's **validated independently** before commitment. Separate UX from business logic
- C) ストリーミングだけで JSON も含めて生成 / Stream the entire response including the JSON
- D) すべて短縮して 1 行で返す / Squash everything to a single line

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

UX 用テキストはストリーミング、業務確定は構造化出力 + 検証で分離。混在させるとパース失敗・部分書き込みリスク。

Stream the prose for UX; obtain the decision via a structured tool call validated independently. Mixing risks parse failures.

- **A 不正解**: TTFT 悪化で UX 低下。 / Hurts UX.
- **C 不正解**: ストリーミング中の JSON は不完全な状態を露出。 / Exposes incomplete JSON.
- **D 不正解**: 情報密度が落ちる。 / Loses informativeness.

**参照 / Reference:** `guide_ja.md` 「ストリーミング」「構造化出力の分離」
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

ロジスティクス最適化エージェントは「3 拠点で在庫が同時に不足したらリバランス」というワークフロー。動的にサブステップが変わる可能性は低く、業務監査が厳しい。

A logistics rebalancer triggers when 3 sites are simultaneously low. Dynamic substeps are rare; audit is strict.

**設問 / Question:**

最も適切なオーケストレーションはどれですか？ / Best orchestration?

- A) **固定パイプライン**：在庫照会 → 不足判定 → リバランス計画 → 承認 → 実行 → 監査記録、の各ステップを `AgentDefinition` で明示し、決定論的に順序を強制 / **Fixed pipeline**: inventory check → shortage detection → rebalance plan → approval → execution → audit log, with each step explicitly defined and order enforced
- B) コーディネーターに自由に動的タスク分解させる / Let the coordinator improvise tasks
- C) 1 つの巨大エージェントが全工程を実行 / One mega-agent handles everything
- D) 人間が全ステップを手動実行 / Humans do every step manually

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

業務監査が厳しく、フロー固定 → **固定パイプライン**で再現性・監査性を最大化。動的分解は監査トレースを壊す。

Strict audit + stable flow = **fixed pipeline** for reproducibility and audit traceability.

- **B 不正解**: 動的分解は監査再現性を壊す。 / Breaks reproducibility.
- **C 不正解**: 巨大エージェントは可観測性が低下。 / Less observable.
- **D 不正解**: 自動化の意味を失う。 / Defeats automation.

**参照 / Reference:** `guide_ja.md` 「3.6 タスク分解戦略」「固定パイプライン」
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

財務アナリストエージェントが、サブエージェント「決算データ取得」が `parsing_error`（PDF 構造異常）で失敗。コーディネーターは代替戦略を選ぶ必要がある。

An equity research agent's "earnings extraction" subagent fails with `parsing_error` on a malformed PDF. Coordinator must choose a fallback.

**設問 / Question:**

最も適切なリカバリ戦略はどれですか？ / Best recovery strategy?

- A) 同じサブエージェントを 10 回リトライ / Retry the same subagent 10 times
- B) 構造化エラーから **代替手段**（HTML 開示版・XBRL データ・ニュース要約）を選び、各代替の信頼度を出力に注記。**N 回失敗で人間にエスカレーション**し、原本リンクと推奨アクションを構造化ハンドオフで提供 / From the structured error pick a **fallback** (HTML disclosure / XBRL / news summary), annotate the result with per-source confidence, and **escalate to human after N failures** with a structured handoff (source links + recommended action)
- C) 失敗を無視して結果を空で返す / Silently return empty
- D) コーディネーター自身が PDF パースを引き受ける / Have the coordinator parse PDFs itself

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

構造化エラー → 代替経路 → 信頼度注記 → 人間エスカレーション、の連鎖が graceful degradation の標準。

Structured error → alternative path → confidence annotation → human escalation = standard graceful degradation.

- **A 不正解**: 同じ条件のリトライは結果が変わらない。 / Same conditions, same failure.
- **C 不正解**: 黙殺は下流に劣化を伝えず危険。 / Silent failure is dangerous.
- **D 不正解**: コーディネーターの責務を肥大化させる。 / Bloats coordinator scope.

**参照 / Reference:** `guide_ja.md` 「7.6 マルチエージェントエラー伝播」
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

クラウドコスト分析エージェントが、毎ターン 4 つのツールを順次直列に呼んでいる（料金照会・使用量照会・予測・推奨生成）。それぞれ独立で副作用なし。レイテンシが 18 秒もかかる。

A cloud-cost analysis agent calls 4 tools sequentially each turn (pricing, usage, forecast, recommendation). All are independent and side-effect-free, but total latency is 18s.

**設問 / Question:**

最も適切な改善はどれですか？ / Best improvement?

- A) 4 ツールを **同一ターン内で並列の `tool_use` ブロック**として返すよう設計し、`tool_result` も並列で受け取る。レイテンシが max(4) ≒ 5〜6s に短縮 / Emit all 4 as **parallel `tool_use` blocks in one turn** and receive `tool_result`s together; latency drops to max(4) ≈ 5–6s
- B) ツールを 1 つに統合 / Merge into one tool
- C) `claude-opus-4-6` で速度向上 / Speed up via `claude-opus-4-6`
- D) ストリーミングを使う / Use streaming

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

独立ツールは並列 `tool_use` で処理。Claude API は 1 ターンで複数 `tool_use` を返すことができ、`tool_result` も並列で受け取れる。

Independent tools should fire in parallel: emit multiple `tool_use` blocks in one turn; collect `tool_result`s together.

- **B 不正解**: 統合は責務分離を犠牲。 / Loses separation.
- **C 不正解**: モデル変更で直列性は変わらない。 / Doesn't fix serialization.
- **D 不正解**: ストリーミングは別軸。 / Different axis.

**参照 / Reference:** `guide_ja.md` 「3.4 並列 tool_use」
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

複雑なエージェントシステムでインシデント発生。コーディネーター + 7 サブエージェントの動作を時系列で追いたい。現状はログが各エージェントごとに分散していて再構成が難しい。

A complex multi-agent system had an incident. You want a chronological trace of coordinator + 7 subagents, but logs are scattered.

**設問 / Question:**

最も適切な可観測性設計はどれですか？ / Best observability design?

- A) 各エージェントの個別ログをそのまま使う / Use per-agent logs as-is
- B) **session_id・parent_session_id・correlation_id・trace_id・span_id** を全イベントに付与し、`PostToolUse` フック等から構造化ログ（JSON）として **集約バックエンド**に送る。OpenTelemetry のような標準スキーマに合わせ、ダッシュボードで親子関係を辿れるようにする / Emit structured logs (JSON) with `session_id`, `parent_session_id`, `correlation_id`, `trace_id`, `span_id` from hooks (`PostToolUse` etc.) into a **central backend**, conforming to OpenTelemetry-style schemas so parent-child traces are navigable
- C) ログを自由文で書き、必要時に grep で探す / Write free-text logs and grep when needed
- D) ログを取らずインシデント時のみ調査 / No logs; investigate only on incident

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

マルチエージェントの可観測性は **トレース ID + 親子関係 + 構造化ログ** で確立する。OpenTelemetry のような標準に合わせると外部ツール連携も容易。

Multi-agent observability requires trace IDs + parent-child links + structured logs, ideally OTel-compatible.

- **A 不正解**: 分散ログは再構成困難。 / Hard to reconstruct.
- **C 不正解**: 自由文は集計不可。 / Not aggregatable.
- **D 不正解**: ログなしは事後分析不能。 / No post-mortem possible.

**参照 / Reference:** `guide_ja.md` 「監査ログ・可観測性」
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

研究エージェントの 1 セッションあたりのトークン消費が想定より 3 倍になり、月次予算を超過。原因はサブエージェントが冗長に呼び合うこと。

A research agent burns 3x the expected tokens per session, blowing the monthly budget. Subagents call each other redundantly.

**設問 / Question:**

最も適切な制御はどれですか？ / Best control?

- A) **セッション全体のトークン上限**と **サブエージェント単位の上限**を Agent SDK で設定し、超過時は構造化エラー（`reason: "budget_exhausted"`, 部分結果）を返す。同時にサブエージェント呼び出しグラフを可観測性で可視化し、冗長呼び出しを設計レベルで除去 / Set **session-wide and per-subagent token caps** in the Agent SDK; on exhaustion return a structured error (`reason: "budget_exhausted"`, partial result). Visualize the call graph via observability and **eliminate redundant calls at the design level**
- B) 月次予算を 3 倍に増やす / Triple the monthly budget
- C) すべてのサブエージェントを `claude-haiku-4-5` に統一 / Force all subagents to `claude-haiku-4-5`
- D) ツールを使わない / Disable tools entirely

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

コスト制御は **トークン上限 + 構造化エラー + 設計レベルの冗長性除去**。可観測性で根本原因を見つけて修正。

Cost control combines token caps + structured halt + design-level dedup, surfaced via observability.

- **B 不正解**: 根本原因に未対処。 / Doesn't address root cause.
- **C 不正解**: 冗長呼び出しは残る。 / Doesn't dedup.
- **D 不正解**: 機能を捨てる過剰反応。 / Throws away function.

**参照 / Reference:** `guide_ja.md` 「コスト管理」「トークン上限」
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

新薬候補のプロパティ予測エージェントで、複数の異なる戦略（分子動力学的試算、機械学習モデル、文献根拠による類推）を並列で試したい。同じセッションコンテキストから分岐して比較する。

A drug-property predictor wants to explore multiple strategies (MD simulation, ML model, literature analogy) in parallel from the same session context for comparison.

**設問 / Question:**

最も適切な機能はどれですか？ / Best feature?

- A) `--resume` でセッションを直列に再開 / Sequentially `--resume` the same session
- B) **`fork_session`** で同一の親コンテキストから **3 つの独立分岐**を起動し、各分岐で異なる戦略を実行。結果を最後に統合判定する。これは "what-if" 探索の典型的ユースケース / Use **`fork_session`** to spawn **3 independent branches** from the same parent context, each running a different strategy; merge results at the end. Canonical "what-if" exploration
- C) 単一の長いセッション内で順次試す / Run sequentially in a single long session
- D) 別の Anthropic アカウントで並行実行 / Run on a different Anthropic account in parallel

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

`fork_session` は同一親コンテキストから複数分岐させ、独立に進めて比較する用途に最適。

`fork_session` is designed for what-if branching from a shared parent context.

- **A 不正解**: 直列再開は並列性なし。 / No parallelism.
- **C 不正解**: コンテキスト混在で比較が困難。 / Mixed context, poor comparison.
- **D 不正解**: アカウント分離は管理上の悪手。 / Operational mess.

**参照 / Reference:** `guide_ja.md` 「3.7 fork_session」
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

医療画像診断エージェントが、最終診断前に必ず人間の放射線科医の確認を要する規制要件あり。エージェントは候補診断と根拠を提示するが、決定はしない。

A radiology agent must obtain a radiologist's review before any final diagnosis (regulation). The agent suggests candidates with rationale but never decides.

**設問 / Question:**

最も適切な人間 in the loop 設計はどれですか？ / Best human-in-the-loop design?

- A) エージェントが診断を確定し、後で放射線科医がレビュー / Agent finalizes; radiologist reviews after
- B) エージェントの最終ステップで `Task` 完了前に **`PreToolUse` フック**または **明示的な人間承認ステップ**を挟み、放射線科医がレビュー画面で承認するまで `commit_diagnosis` ツールが呼ばれないようにする。承認内容と差分は監査ログに保存 / Insert a **`PreToolUse` hook** (or explicit human-approval step) before the final `Task` completes, blocking `commit_diagnosis` until the radiologist approves. Log approval and diffs to the audit trail
- C) システムプロンプトで「人間に確認しろ」と指示 / Prompt: "ask a human to confirm"
- D) 放射線科医をエージェントの一員として組み込む / Embed the radiologist as an agent

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

規制要件の人間承認は **フック / 明示ステップで決定論的に**実装。プロンプトでは確実性なし。

Regulated human-approval gates are implemented deterministically via hooks or explicit steps — never via prompt.

- **A 不正解**: 順序が逆で規制違反。 / Inverted, regulatory violation.
- **C 不正解**: 確率的で規制不適合。 / Probabilistic.
- **D 不正解**: エージェント化は責務分離を崩す。 / Mis-frames the role.

**参照 / Reference:** `guide_ja.md` 「3.5 PreToolUse」「人間承認ゲート」
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

製造業の予知保全システムが、複数の生産ラインから同時に「異常値」アラートを受け、各ラインに対応するエージェントセッションを並列起動します。同じ部品の同時注文が走り **発注重複**が発生しました。

A predictive-maintenance system receives anomaly alerts from multiple lines and launches per-line agent sessions in parallel. Concurrent orders for the same part **duplicate**.

**設問 / Question:**

最も適切な防止策はどれですか？ / Best prevention?

- A) 並列起動を禁止し、すべてのラインを直列処理 / Disallow parallelism; serialize all lines
- B) 並列性を維持しつつ、`order_part` ツールに **分散ロック（Redis / DB の SELECT FOR UPDATE）** と **冪等キー**（`part_id + date` 等）を実装。重複発注は注文 API 側で 409 Conflict として弾く。エージェントには `is_error: true` で構造化通知し、リカバリさせる / Keep parallelism; add a **distributed lock (Redis / SELECT FOR UPDATE)** and **idempotency key** (`part_id + date`) to `order_part`; the order API returns 409 Conflict on duplicates; agents receive `is_error: true` for structured recovery
- C) システムプロンプトで「他のラインと協調しろ」と指示 / Prompt: "coordinate across lines"
- D) すべてのラインで `claude-opus-4-6` を使う / Use `claude-opus-4-6` everywhere

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

並列性を保ったまま重複を防ぐには、**分散ロック + 冪等キー + サーバ側 409** の組み合わせ。エージェント側ではなく **注文 API レイヤー**で唯一の真実を担保するのが正攻法。

Preserve parallelism; enforce uniqueness at the **order API** with a distributed lock + idempotency key + server-side 409.

- **A 不正解**: 直列化はスループットを犠牲にする。 / Sacrifices throughput.
- **C 不正解**: プロンプト協調は確率的。 / Probabilistic.
- **D 不正解**: モデル変更は競合を解決しない。 / Doesn't fix concurrency.

**参照 / Reference:** `guide_ja.md` 「冪等性・分散ロック」「concurrency control」
</details>

---

## 問題 31 / Question 31

**シナリオ / Scenario:**

ティア 1 投資銀行の **約定後処理（post-trade settlement）** で、Claude Agent SDK を使った再照合エージェントを設計中です。約定 → 確認書突合 → 例外処理 → CSD（証券保管振替機関）連携というフローで、決済リスクは T+1 で解消されなければなりません。日中で 50 万件の約定を処理し、例外発生は 0.3%（1,500 件）。例外の自動解決と人間オペレータへのエスカレーションを両立させる必要があります。

A tier-1 investment bank's **post-trade settlement** uses a Claude Agent SDK reconciliation agent: trade → confirmation matching → exception handling → CSD interaction, with settlement risk that must clear by T+1. Daily 500K trades, 0.3% (1,500) exceptions. Auto-resolve and human-escalate must coexist.

**設問 / Question:**

最も適切なオーケストレーションはどれですか？ / Best orchestration?

- A) 単一エージェントが全 50 万件を直列処理 / One agent serially handles all 500K
- B) 例外（1,500 件）のみ Agent SDK で扱い、残りはルールベースで処理。例外用コーディネーターは **3 段階エスカレーションパス**（Tier 1: 自動修正 / Tier 2: シニアオペレータ / Tier 3: ミドルオフィス）を持ち、各段で `confidence` と **修正手順の構造化ログ**を残す。T+1 SLA を超過する例外は自動的に Tier 3 へ昇格 / Process the 1,500 exceptions with the Agent SDK; rules handle the rest. The exception coordinator has a **3-tier escalation path** (Tier 1 auto-fix / Tier 2 senior op / Tier 3 middle office) with `confidence` and a **structured log of remediation steps** at each tier. Exceptions risking T+1 SLA escalate automatically to Tier 3
- C) すべてを Tier 1 で自動修正 / Auto-fix everything in Tier 1
- D) 例外は無視して T+2 で処理 / Skip exceptions; process at T+2

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

ポストトレード例外処理では **大半をルールで処理し、エージェントは例外領域に集中**するのが定石。エスカレーションパスは決済リスクとオペレータ負荷のバランスで設計し、SLA 監視で動的に Tier 昇格させる。各段の構造化ログは T+1 内の解消・監査再現性に必須。

Post-trade exception handling pattern: **rules for the bulk, agent for exceptions**. Tiered escalation balances risk and operator load; SLA monitoring drives dynamic promotion. Structured logs per tier are essential for T+1 closure and audit reproducibility.

- **A 不正解**: 50 万件の直列処理は SLA 不可能。 / Serial 500K can't meet SLA.
- **C 不正解**: 高難度例外を Tier 1 で誤修正するリスク。 / Misfix risk in Tier 1.
- **D 不正解**: T+1 規制違反。 / Regulatory breach.

**参照 / Reference:** Post-trade settlement・T+1 規制・段階エスカレーション
</details>

---

## 問題 32 / Question 32

**シナリオ / Scenario:**

大手クレジットカード会社の **リアルタイム不正検知**で、Claude エージェントを補助判断に使う構想。1 件あたり 50ms 以内に「承認 / 拒否 / 追加認証要求（3DS）」を判定する必要があります。ML モデルが第一段階、エージェントは ML モデルの境界例（confidence 0.4〜0.6）のみ取り扱う方針。

A real-time fraud detection at a major card issuer plans Claude as auxiliary judgment for borderline ML cases (confidence 0.4–0.6); approve / decline / step-up (3DS) decision must complete within 50ms.

**設問 / Question:**

最も適切なエージェント設計はどれですか？ / Best agent design?

- A) Claude を毎件呼び出して 50ms 以内に判断させる / Call Claude every transaction in 50ms
- B) `claude-haiku-4-5` を使い、すべての transaction にツールチェーンを呼ばせる / Use `claude-haiku-4-5` with full tool chain on every tx
- C) ML 第一段で **確信度高**は即時判定、**境界例のみ**を非同期キューに投入し Claude が分析、結果は **次回以降の同一カードの判定にフィードバック**（リアルタイムには間に合わない設計）。リアルタイム経路には **キャッシュされた過去判断 + 動的ルール**を活用。Claude のレイテンシは別 SLA（数秒〜数十秒）で運用 / ML first-pass for high-confidence; **borderline cases** are pushed to an async queue for Claude analysis, results **feed forward to subsequent decisions** for the same card (not real-time). Real-time path uses **cached prior decisions + dynamic rules**. Claude operates under a separate SLA (seconds-to-tens-of-seconds)
- D) Claude を使わない / Don't use Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

50ms SLA 下で Claude を真のリアルタイム判定に使うのは現実的ではありません。**ハイブリッド設計**：(i) 高確信度はルール / ML、(ii) 境界例は非同期で深掘り、(iii) 結果は次回以降にフィードバックする。リアルタイム判定は ML + 動的ルール + キャッシュで完結させ、Claude は **戦術的（次の判断に効く）** な役割に限定するのがプロのパターン。

A 50ms SLA isn't a realistic Claude budget. **Hybrid**: rules/ML for high confidence; async deep analysis for borderlines; results feed forward. Real-time path = ML + dynamic rules + cache; Claude provides **tactical** advantage for subsequent decisions.

- **A 不正解**: 50ms で API ラウンドトリップを含めるのは事実上不可能。 / Practically impossible.
- **B 不正解**: ツールチェーン呼び出しは数百ms〜秒単位。 / Adds seconds.
- **D 不正解**: Claude を使わない選択肢で問題は解決するが、機会損失。 / Misses value.

**参照 / Reference:** リアルタイムシステム設計・ハイブリッド ML/LLM
</details>

---

## 問題 33 / Question 33

**シナリオ / Scenario:**

国際銀行の **規制報告**（FRY-9C・FFIEC・COREP・LCR）の自動生成に Claude を導入。月次・四半期で**数十のテンプレート**を、勘定系・市場リスク・与信リスクなど複数システムから集計したデータで埋めます。報告書には **誤りが許されず**（罰金リスク）、生成ロジックの **完全な監査トレース**が必要。

An international bank automates regulatory reports (FRY-9C, FFIEC, COREP, LCR) using Claude; dozens of monthly/quarterly templates aggregate from core banking, market-risk, and credit-risk systems. **Zero-error tolerance** (fines) and **complete audit trace** required.

**設問 / Question:**

最も適切なエージェントトポロジはどれですか？ / Best topology?

- A) 自由記述で 1 つのエージェントが全部処理 / One agent free-forms everything
- B) **固定パイプライン**：①データソース取得 → ②検証（クロスシステム整合性） → ③テンプレート埋め込み → ④数値検証（前期比・規制定義との一致） → ⑤独立レビューエージェント（`context: fork`）→ ⑥承認待ち。各ステップで **入出力ハッシュを WORM ログ**に記録、再実行で同じハッシュなら確定。逸脱時は人間レビュー必須 / **Fixed pipeline**: ①source ingest → ②validation (cross-system consistency) → ③template fill → ④numeric checks (period-over-period, against regulatory defs) → ⑤independent review agent (`context: fork`) → ⑥approval. Each step writes **input/output hashes to WORM logs**; deterministic re-run reproduces the same hashes. Deviations trigger mandatory human review
- C) 動的に行動を決めさせる / Let the agent decide dynamically
- D) 報告書は手書きに戻す / Revert to hand-written reports

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

規制報告は **再現可能性が最重要**：固定パイプライン + 各段階のハッシュベース監査 + 独立レビュー + 承認ゲート。動的タスク分解は監査トレースを破壊するため不適。

Regulatory reports demand **reproducibility**: fixed pipeline + per-stage hash audit + independent review + approval. Dynamic task decomposition destroys audit traces.

- **A 不正解**: 自由記述は再現性ゼロ。 / Zero reproducibility.
- **C 不正解**: 監査不可能。 / Not auditable.
- **D 不正解**: 自動化価値を失う。 / Loses value.

**参照 / Reference:** 規制報告・固定パイプライン・WORM
</details>

---

## 問題 34 / Question 34

**シナリオ / Scenario:**

ヘッジファンドの **クオンツリサーチ**で、Claude が市場ニュース・SEC 提出書類・ブローカーレポートから **アルファシグナル候補**を抽出。複数戦略チームが並行で利用し、戦略ごとに同じデータの解釈が異なる場合があります。

A hedge fund's quant research has Claude extract alpha signal candidates from news / SEC filings / broker notes. Multiple strategy teams use it in parallel; same data may be interpreted differently per strategy.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) 戦略ごとに **専用のサブエージェント定義**を作成（`AgentDefinition` ごとに異なるシステムプロンプト・許可ツール・出力スキーマ）。共有データ取得は **共通抽出エージェント**で行い、その出力を各戦略エージェントが独立解釈。**戦略間で結果がリークしないよう** `Task` の `prompt` で送る情報を最小化。各戦略の解釈履歴は **scratchpad ファイル**で版管理 / Per-strategy `AgentDefinition`s (distinct system prompts / allowed tools / output schemas). A **shared extraction agent** retrieves data; each strategy reinterprets independently. **Prevent cross-strategy leakage** via minimal `Task` prompts. Versioned scratchpad files per strategy
- B) 1 つのエージェントが全戦略の判断 / One agent for all strategies
- C) 戦略ごとに別 Anthropic アカウント / Separate Anthropic accounts
- D) 戦略は分離せずチームに任せる / Leave separation to teams

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

戦略の **解釈差異の独立性**は競争優位性。`AgentDefinition` 分離 + 共通抽出 + 最小情報送信で機械的に分離。情報リークは規制（front-running 等）にも関わる。

Independent strategy interpretations are competitive edge. `AgentDefinition` separation + shared extraction + minimal prompts mechanically isolate. Leakage also has regulatory implications.

- **B 不正解**: 解釈バイアス共有・リーク。 / Shared bias / leakage.
- **C 不正解**: 過剰、運用負荷。 / Overkill.
- **D 不正解**: 一貫性なし。 / Inconsistent.

**参照 / Reference:** AgentDefinition・戦略分離・情報リーク
</details>

---

## 問題 35 / Question 35

**シナリオ / Scenario:**

商業銀行で **法人融資の与信審査**を支援するエージェントを構築。財務諸表分析・業界比較・経営者面談ノートのレビューを並列実行し、最後に統合スコアを生成します。融資判断の最終決定権は **必ず人間の与信担当者**。

A commercial bank's corporate-loan underwriting agent runs financial statement analysis, peer comparison, and management-interview review in parallel, producing a consolidated score. **Final lending decision must be a human underwriter.**

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) Claude が融資承認可否を出力 / Have Claude output approve/decline
- B) 並列サブエージェントの統合結果に対して **構造化ハンドオフ**：`{ overall_score, strengths[], red_flags[], peer_comparison_table, evidence_links[], confidence, recommended_action: "review_recommended_approve" | "review_recommended_decline" | "additional_data_needed" }`。**「approve/decline」ではなく「推奨」**として表現し、人間の判断材料として提示。`PreToolUse` フックで `commit_credit_decision` ツールを **必ず人間承認後にのみ実行可能**にし、最終決定権を技術的に保証 / **Structured handoff** of merged results: `{ overall_score, strengths[], red_flags[], peer_comparison_table, evidence_links[], confidence, recommended_action: "review_recommended_approve" | "review_recommended_decline" | "additional_data_needed" }`. Express as **recommendation, not decision**. A `PreToolUse` hook ensures `commit_credit_decision` only fires **after human approval** — technically guaranteed
- C) ML スコアだけ使う / Use only an ML score
- D) Claude は使わない / Don't use Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

人間判断必須の領域では LLM 出力は **意思決定そのものではなく決定支援**。フックで決定論的にゲートを設置し、構造化ハンドオフで根拠を可視化。

Where human decision is mandated, LLM output is **decision support, not the decision**. Hooks deterministically gate; structured handoff surfaces evidence.

- **A 不正解**: 規制（公平貸付・ECOA 等）違反リスク。 / Regulatory risk.
- **C 不正解**: 質的情報の活用が抜ける。 / Misses qualitative info.
- **D 不正解**: 機会損失。 / Missed value.

**参照 / Reference:** 与信審査・人間承認・PreToolUse
</details>

---

## 問題 36 / Question 36

**シナリオ / Scenario:**

総合病院の **臨床決定支援（CDS）** で、Claude エージェントが医師の処方判断を補助します。電子カルテから患者プロファイル取得 → 薬物相互作用チェック → アレルギーチェック → 推奨用量計算という 4 サブエージェント。**EHR システム（Epic/Cerner）への書き込みは絶対に禁止**、読み取り専用。

A hospital CDS agent assists prescribing: 4 subagents — patient profile retrieval / drug-drug interaction / allergy check / dose calculation — from the EHR. **Writes to EHR (Epic / Cerner) are forbidden**; read-only.

**設問 / Question:**

最も適切な保証はどれですか？ / Best safeguard?

- A) システムプロンプトで「書き込まない」と指示 / Prompt: "do not write"
- B) 各 `AgentDefinition` の `allowed_tools` から **書き込み系ツールを完全削除**し、MCP サーバ側でも書き込みエンドポイントを **HIPAA ロール（read-only）** で接続。さらに `PreToolUse` フックで MCP 書き込みを多層防御。EHR 監査ログ（HIPAA 必須）と Agent SDK 側 `PostToolUse` ログを照合し、不一致時はインシデント / **Remove write tools entirely from `allowed_tools`** in each `AgentDefinition`; connect to MCP using a **HIPAA read-only role** server-side. Add a `PreToolUse` hook for defense in depth. Reconcile EHR audit logs (HIPAA mandatory) with Agent SDK `PostToolUse` logs; mismatches are incidents
- C) Claude を信頼して書き込み許可 / Trust Claude with writes
- D) EHR 統合をしない / Don't integrate with EHR

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

医療 EHR への書き込み制御は **多層防御**：`allowed_tools` で物理的不可能化、サーバ側ロールで二重防御、フックで三重、ログ照合で四重。HIPAA 監査での説明可能性を確保。

EHR write control demands **defense in depth**: `allowed_tools` removal, server-side role, hook, log reconciliation. Ensures HIPAA audit explainability.

- **A 不正解**: プロンプトは確率的、HIPAA 不適合。 / Probabilistic.
- **C 不正解**: 規制違反リスク。 / Regulatory risk.
- **D 不正解**: 統合価値を捨てる。 / Loses value.

**参照 / Reference:** HIPAA・最小権限・defense in depth
</details>

---

## 問題 37 / Question 37

**シナリオ / Scenario:**

大手保険会社の **保険金請求処理**を Claude で自動化。事故報告 → 保険契約照合 → 損害評価 → 過去類似請求検索 → 支払推奨という 5 ステップ。**FRA / 保険金詐欺対策法**により、疑わしい請求は **専任 SIU（Special Investigations Unit）** にエスカレーション必須。

A major insurer automates claim processing with Claude: incident → policy match → damage assessment → similar-claim lookup → payment recommendation. Suspicious claims must escalate to the **SIU (Special Investigations Unit)** per fraud regulations.

**設問 / Question:**

最も適切なエスカレーション設計はどれですか？ / Best escalation design?

- A) 疑わしい場合に SIU 担当者に Slack で連絡 / Slack the SIU
- B) **疑わしさスコア**を構造化（`{ score: 0..1, fraud_indicators: [enum...], evidence_quotes: [{quote, source, location}], confidence, recommended_siu_action: enum }`）。スコア閾値超過時は **`PreToolUse` フック**で `commit_payment` を決定論的にブロックし、自動的に SIU エスカレーションキューに入れる。**ハンドオフ全体を 30 秒で SIU が判断できる構造**にし、原本リンクと過去類似請求 ID を含める / Structure a **suspicion score** (`{ score: 0..1, fraud_indicators: [enum...], evidence_quotes: [{quote, source, location}], confidence, recommended_siu_action: enum }`). On threshold breach, **`PreToolUse` hook** deterministically blocks `commit_payment` and queues SIU escalation. Format the handoff so SIU can decide in 30s — include source links and similar-claim IDs
- C) Claude が SIU 業務も実行 / Have Claude do SIU work
- D) すべての請求を SIU で確認 / SIU reviews everything

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

保険金詐欺対策では **構造化スコア + 決定論的ブロック + 構造化ハンドオフ**が標準。Slack 通知だけでは支払いが先行するリスク。

Anti-fraud workflow = **structured score + deterministic block + structured handoff**. Slack alone risks payment before review.

- **A 不正解**: 通知だけでは支払い先行リスク。 / Race against payment.
- **C 不正解**: SIU の専門性は LLM では代替不能。 / Beyond LLM scope.
- **D 不正解**: 運用不能、SIU 過負荷。 / Operationally infeasible.

**参照 / Reference:** Insurance fraud・SIU・PreToolUse
</details>

---

## 問題 38 / Question 38

**シナリオ / Scenario:**

製薬会社の **臨床試験モニタリング**で、複数の試験施設（site）からのデータを Claude が監視。プロトコル逸脱（protocol deviation）を検知したら、**FDA 21 CFR Part 11 準拠**の電子記録を残し、Sponsor の Medical Monitor に通知する必要があります。試験は GCP（Good Clinical Practice）下で 2〜5 年継続。

A pharma's clinical trial monitor uses Claude to watch multiple sites for protocol deviations. Deviations require **FDA 21 CFR Part 11-compliant** e-records and Medical Monitor notification. Trials run 2–5 years under GCP.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) ツール呼び出しの後、自由文で記録 / Free-text post-call notes
- B) **すべての判定とアクション**を `PostToolUse` フックで **電子署名付き WORM ログ**（タイムスタンプ・ユーザー ID・操作・SHA-256・電子署名）に書き込み。長期セッションは `--resume` + 外部チェックポイントで連続性を保ち、各時点の **試験プロトコル版本**もログに紐付け。Medical Monitor への通知は **構造化ハンドオフ**（site / subject / deviation type / severity / regulatory implication）。プロトコル変更時は **メジャー版** として再 baseline 化 / **Every decision and action** is written to **e-signed WORM log** via `PostToolUse` hook (timestamp, user, operation, SHA-256, signature). Long sessions use `--resume` + external checkpoints; each log entry binds the **active trial protocol version**. Medical Monitor notifications use **structured handoff** (site / subject / deviation type / severity / regulatory implication). Protocol changes trigger **major version** re-baseline
- C) 紙のログだけ残す / Only paper logs
- D) FDA 21 CFR Part 11 は無視 / Ignore the regulation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

臨床試験は超長期 + 強規制。WORM ログ + 電子署名 + プロトコル版数紐付け + 構造化エスカレーション + チェックポイント連続性が要件。

Clinical trials are super-long + heavily regulated: WORM + e-sig + protocol version + structured escalation + checkpoint continuity.

- **A 不正解**: 構造化なしで監査不能。 / Not auditable.
- **C 不正解**: 21 世紀の規制は電子記録対応。 / E-records mandated.
- **D 不正解**: 違法・出荷停止。 / Illegal.

**参照 / Reference:** FDA 21 CFR Part 11・GCP・clinical trial monitoring
</details>

---

## 問題 39 / Question 39

**シナリオ / Scenario:**

精神科クリニックで Claude が患者対話のサマリと **リスクスクリーニング**（自殺念慮など）を補助。誤検出は致命的（患者離脱）、検出漏れも致命的（救命機会損失）。**HIPAA + 各州の精神保健法**に準拠する必要があります。

A psych clinic uses Claude for visit summaries and **risk screening** (e.g., suicidal ideation). False positives drive patient drop-off; misses cost lives. Must comply with HIPAA + state mental-health laws.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) Claude のリスク判定をそのまま採用 / Trust Claude's risk verdict
- B) Claude は **検出補助のみ**（最終判断は臨床医）。リスク指標が出た場合は (i) **構造化されたエビデンス**（発言の原文引用、文脈、スクリーニングスケール対応）、(ii) **複数の独立判定**（self-consistency / 複数サンプル）、(iii) **明確な不確実性表現**を出力し、(iv) 人間臨床医にハンドオフ。すべての判定セッションは **暗号化 PHI ストア**に保存し、患者同意の範囲でのみ利用 / Claude is **detection-aid only**; clinicians decide. On positive: (i) **structured evidence** (verbatim quotes, context, scale alignment), (ii) **multi-sample self-consistency**, (iii) **explicit uncertainty**, (iv) clinician handoff. Sessions stored in **encrypted PHI store**, used within patient consent boundaries
- C) リスクが疑われたら警察に直接通報 / Call police on suspected risk
- D) リスクスクリーニングは行わない / Skip risk screening

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

メンタルヘルスは **臨床判断必須 + 法的義務（duty to warn 等）+ 患者プライバシー**の三重制約。Claude は補助役、判断は人間。

Mental health = **clinical judgment required + legal duties (duty to warn) + patient privacy**. Claude assists; humans decide.

- **A 不正解**: 規制不適合 + 倫理問題。 / Regulatory + ethical issue.
- **C 不正解**: 越権行為で別の法的問題。 / Out of scope, legal issue.
- **D 不正解**: 機会損失（救命）。 / Loses life-saving signal.

**参照 / Reference:** HIPAA mental health・duty to warn・clinical decision support
</details>

---

## 問題 40 / Question 40

**シナリオ / Scenario:**

希少疾患の **新薬探索** で、Claude が論文・特許・社内実験データから候補化合物の作用機序仮説を生成。仮説は **科学的に検証可能** でなければならず、根拠論文は必ず引用、内部データソースは **企業秘密**として外部に漏らさない。

For rare-disease drug discovery, Claude generates mechanism-of-action hypotheses from papers / patents / internal experiment data. Hypotheses must be **scientifically testable**; cite papers; keep internal data **confidential**.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) すべてのデータを 1 つのプロンプトに混ぜる / Mix everything in one prompt
- B) **データ層分離アーキテクチャ**：①公開データ（論文・特許）取得用エージェント（**Citations 機能で出所付与**）、②社内データ取得用エージェント（**社内 MCP のみアクセス可能**、出力に "INTERNAL" タグ）、③仮説生成エージェント（両者の出力を受け、仮説には公開根拠を必須・社内根拠は別フィールドで分離管理）、④検証可能性チェック（実験で反証可能か、定量的か）。社外提出向け出力からは "INTERNAL" タグを **構造的に除去**できる設計 / **Data-layer separation**: ①public-data agent (papers / patents) with **Citations**, ②internal-data agent (MCP-only) tagging output as "INTERNAL", ③hypothesis-gen agent consuming both — public grounding required, internal grounding in a separate field, ④falsifiability check (experimentally testable, quantitative). External outputs **structurally strip** "INTERNAL"
- C) 社内秘密も論文も区別なく出力 / Mix internal and public freely
- D) 公開データだけ使う / Use only public data

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

知財管理 + 科学的厳密性のため、**データ層分離 + Citations + 検証可能性チェック + 構造的タグ除去**が標準。

IP + scientific rigor: **layer separation + Citations + falsifiability + structural tag stripping**.

- **A 不正解**: 知財漏洩リスク。 / IP leakage risk.
- **C 不正解**: 同上。 / Same.
- **D 不正解**: 競争優位性を捨てる。 / Loses edge.

**参照 / Reference:** IP isolation・citations・falsifiability
</details>

---

## 問題 41 / Question 41

**シナリオ / Scenario:**

国際法律事務所の **e-Discovery** で、訴訟関連文書 200 万件から関連性のある文書を抽出。**特権文書（attorney-client privilege）** は絶対に開示してはならず、誤って開示すると訴訟戦略が崩壊します。

International law firm's e-Discovery sifts 2M docs for relevance. **Attorney-client privilege docs must never be disclosed**; accidental release ruins strategy.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) 1 つのエージェントが関連性判定と特権判定を同時実行 / One agent does relevance + privilege jointly
- B) **二段階パイプライン + 多重チェック**：①特権判定エージェント（高 recall 優先 — false negative を最小化、specialty: 弁護士間メール・work product・interview notes 等のパターン検出）、②関連性判定エージェント（特権文書は **入力に来ない** よう前段でフィルタ）、③人間レビュアーが特権マークを最終確認、④TAR（Technology Assisted Review）統計で **隠れた特権率**を推計し、サンプルレビューで担保 / **Two-stage + multi-check**: ①privilege detector (high-recall — minimize false negatives; expert in attorney-attorney email, work product, interview notes), ②relevance detector (privilege docs **never enter** its input — filtered upstream), ③human reviewer confirms privilege marks, ④TAR statistics estimate **latent privilege rate** with sample review
- C) 関連文書だけ見て特権判定は省略 / Skip privilege check
- D) 全文書を一斉に開示 / Disclose everything

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

特権漏洩は法律事務所の致命傷。**前段で物理的にフィルタ + 高 recall + 人間最終確認 + TAR 統計**。

Privilege leak is fatal: **upstream physical filter + high recall + human final + TAR statistics**.

- **A 不正解**: 同一エージェントの混在は漏洩リスク。 / Co-mingling risk.
- **C 不正解**: 違法・致命的。 / Illegal, catastrophic.
- **D 不正解**: 論外。 / Unthinkable.

**参照 / Reference:** e-Discovery・attorney-client privilege・TAR
</details>

---

## 問題 42 / Question 42

**シナリオ / Scenario:**

レギュレーター（金融・医療・環境）が頻繁に規則を更新します。法律事務所で **規制変更の継続的トラッキングエージェント**を構築。クライアントへの **影響度評価レポート**を自動生成し、変更の **発効日（effective date）** までに対応案を提示する必要があります。

Regulators (finance / health / environment) issue frequent rule updates. A law firm builds an agent for **continuous regulatory tracking**, auto-generating client impact assessments before each **effective date**.

**設問 / Question:**

最も適切なエージェント設計はどれですか？ / Best design?

- A) 月 1 回エージェントが手動で全規制を読み込む / Monthly manual full read
- B) **イベント駆動型**：①官報・規制機関の RSS / API を **MCP リソース subscription** で監視（変更通知をリアルタイム受信）、②変更検知エージェントが新旧差分を抽出、③影響評価エージェントがクライアント別の影響を判定（クライアントの業種・規制ステータス・契約内容との照合）、④発効日カウントダウン付きでクライアント担当弁護士にハンドオフ。すべての追跡履歴は WORM で保管 / **Event-driven**: ①monitor official gazettes / agency APIs via **MCP resource subscription** (real-time updates), ②change-detection agent extracts old/new diff, ③impact-assessment agent maps to client-specific implications (industry / status / contracts), ④hand off to engagement attorney with effective-date countdown. All tracking history in WORM
- C) 規制変更は無視 / Ignore changes
- D) クライアントに自分で確認してもらう / Tell clients to track themselves

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

規制トラッキングは **イベント駆動 + 差分抽出 + 影響評価 + 発効日管理 + 監査保管** が標準。

Regulatory tracking = **event-driven + diff extraction + impact assessment + effective-date management + audit retention**.

- **A 不正解**: 月 1 ではタイムリー性ゼロ。 / Untimely.
- **C 不正解**: 弁護過誤リスク。 / Malpractice risk.
- **D 不正解**: 法律事務所の付加価値を捨てる。 / Loses value-add.

**参照 / Reference:** Regulatory tracking・MCP subscriptions
</details>

---

## 問題 43 / Question 43

**シナリオ / Scenario:**

M&A 案件の **デューデリジェンス**で、Claude が **VDR（Virtual Data Room）** の数千文書を分析。買い手 / 売り手それぞれに別の Claude エージェントが配置され、**情報壁（Chinese Wall）** を維持しなければなりません。

In M&A due diligence, Claude analyzes thousands of VDR docs; **separate agents for buy-side and sell-side** must maintain **Chinese walls**.

**設問 / Question:**

最も適切な情報壁実装はどれですか？ / Best Chinese wall implementation?

- A) 同じエージェントを順番に使う / One agent serially for both sides
- B) **物理的隔離**：別組織契約 / 別 Anthropic 環境 / 別 API キー / 別 MCP サーバ / 別ストレージ。各サイドの **session_id** にサイドタグを付与し、コードレベルでクロス参照不可能化。法的にも「**別チーム / 別ベンダー**」として位置付け。**監査時に分離が立証可能** / **Physical isolation**: separate org agreements / Anthropic environments / API keys / MCP servers / storage. Side-tag every `session_id`; code-level cross-reference is impossible. Legally framed as "**separate teams / vendors**". Separation provable to audit
- C) システムプロンプトで「相手側を見るな」 / Prompt: "don't peek the other side"
- D) 情報壁は不要 / No Chinese wall

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

M&A 情報壁は **物理的隔離 + 法的位置付け + 監査可能性**で実現。プロンプトでは規制不適合。

M&A walls = **physical isolation + legal framing + auditability**. Prompts are insufficient.

- **A 不正解**: 規制違反 + 倫理違反。 / Regulatory + ethical breach.
- **C 不正解**: 確率的、立証不能。 / Probabilistic, not provable.
- **D 不正解**: 訴訟リスク甚大。 / Massive litigation risk.

**参照 / Reference:** M&A Chinese walls・legal ethics
</details>

---

## 問題 44 / Question 44

**シナリオ / Scenario:**

特許事務所で **先行技術調査（prior art search）** をエージェント化。発明書類 → 関連分野特定 → 既存特許 / 論文検索 → 類似性分析 → 特許性意見書ドラフト、というフロー。**専門的な誤判定は出願却下や訴訟敗訴**につながります。

A patent firm builds an agent for prior-art search: invention disclosure → field identification → patent / paper search → similarity analysis → patentability opinion. **Expert errors cause rejection or litigation losses.**

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) 1 エージェントですべて処理 / One agent handles all
- B) **特化サブエージェント + 弁理士レビュー**：①機械学習分野・薬学分野・機械工学分野など **分野別の検索エージェント**（各分野の検索戦略・データベース・術語に最適化）、②類似性分析は **構造化された請求項マッピング**（element-by-element）、③弁理士が **claim chart** を最終確認、④意見書は **テンプレート + 引用付き** で人間が承認後発行。エージェントは下書き専属 / **Specialist subagents + patent-attorney review**: ①field-specific search agents (ML, pharma, mech eng) tuned to each field's strategy / database / vocabulary, ②similarity = structured claim mapping (element-by-element), ③attorney finalizes the **claim chart**, ④opinion drafted via **template + citations**, issued only after human approval. Agents draft only
- C) Claude に意見書を直接出させる / Have Claude issue the opinion
- D) AI を使わず手作業 / Manual only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

特許実務は **分野専門性 + claim chart + 弁理士最終承認**。LLM は下書き役。

Patent practice = **field expertise + claim chart + attorney sign-off**. LLM drafts.

- **A 不正解**: 分野横断は精度劣化。 / Cross-field drift.
- **C 不正解**: 弁理士業法違反リスク。 / UPL risk.
- **D 不正解**: 効率損失。 / Efficiency loss.

**参照 / Reference:** Prior art・claim chart
</details>

---

## 問題 45 / Question 45

**シナリオ / Scenario:**

金融機関の **AML（マネーロンダリング検知）** で、SAR（Suspicious Activity Report）の起草を Claude が補助。AML 担当者が最終承認するが、**起草過程で false positive が多すぎると担当者が疲弊**し、true positive を見落とすリスク。

For AML, Claude drafts SARs (Suspicious Activity Reports); AML officers finalize. **Too many false positives fatigue officers** and they miss true positives.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) すべての疑わしい取引で SAR を起草させる / Draft a SAR for every suspicious tx
- B) **重み付け閾値 + 動的キャリブレーション**：①取引リスクスコアに加え、**過去の類似取引のオフィサー判断結果**を学習に取り込み、起草対象を動的に絞る、②起草された SAR には **confidence + 主要根拠 3 点 + 反証 1 点**を必ず含め、オフィサーが **30 秒で判断**できる構造、③オフィサーフィードバックを **継続的に閾値調整**にフィードバックする運用ループ、④四半期ごとの過検出率 / 過小検出率レビュー / **Weighted threshold + dynamic calibration**: ①risk score + **historical officer outcomes for similar txs** narrow drafting, ②every draft includes **confidence + top-3 grounds + one counter-evidence** so officers decide in **30s**, ③officer feedback continuously tunes thresholds, ④quarterly review of over- vs under-detection rates
- C) 閾値は固定 / Fix the threshold
- D) すべての取引で人間レビュー / Human-review every tx

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

AML 運用は **オフィサー疲労を最小化しつつ true positive を逃さない**バランス。動的キャリブレーション + 構造化根拠が要点。

AML balances officer fatigue and miss rate via **dynamic calibration + structured grounding**.

- **A 不正解**: 過検出で疲労、ミス増。 / Over-alert fatigue.
- **C 不正解**: 環境変化に追従できない。 / Static, drifts.
- **D 不正解**: 不可能、コスト過大。 / Infeasible.

**参照 / Reference:** AML・SAR・calibration
</details>

---

## 問題 46 / Question 46

**シナリオ / Scenario:**

自動車製造ラインで **予知保全（predictive maintenance）** にエージェント導入。SCADA から振動センサー値を読み取り、異常パターンを検知すると保全計画と部品発注を提案します。**ISO 26262 ASIL-D（最高安全水準）** の対象機械なので、誤った保全推奨で事故が起きると人命にかかわります。

An automotive line adopts predictive maintenance: read vibration sensors via SCADA, detect anomalies, propose maintenance + parts orders. The machinery is **ISO 26262 ASIL-D** (highest safety integrity); wrong recommendations risk lives.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) Claude の推奨を即座に実行 / Execute Claude's recommendation immediately
- B) Claude は **Tier 1 トリアージ専属**：(i) 異常パターン検出と分類のみ、(ii) 保全推奨は **「物理的検証要 / 予防交換可 / 監視継続」** の 3 区分の構造化提案、(iii) 物理的検証要は **必ず保全エンジニア**が確認、(iv) 部品発注は人間承認必須、(v) 過去の判定 vs 実際の故障状況をフィードバックループに組み込み継続的に精度評価。**ASIL-D 機器の制御変更は LLM の責任範囲外** / Claude is **Tier-1 triage only**: (i) detection + classification, (ii) recommendation in 3 structured classes — **physical verification needed / preventive swap allowed / continued monitoring**, (iii) physical-verification cases **always go to a maintenance engineer**, (iv) parts ordering needs human approval, (v) outcomes vs predictions feed continuous accuracy review. **ASIL-D control changes are out of LLM scope**
- C) Claude が部品発注を直接実行 / Have Claude order parts directly
- D) 予知保全をやめる / Drop predictive maintenance

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

ISO 26262 ASIL-D では **LLM は意思決定者ではなくトリアージ補助**。物理検証 + 人間承認が標準。

Under ISO 26262 ASIL-D, **LLM is triage aid, not decider**; physical verification + human approval are mandatory.

- **A 不正解**: 安全性規格違反。 / Standard breach.
- **C 不正解**: 人間承認なき発注は規制違反。 / Compliance breach.
- **D 不正解**: 価値喪失。 / Loses value.

**参照 / Reference:** ISO 26262・predictive maintenance
</details>

---

## 問題 47 / Question 47

**シナリオ / Scenario:**

電子部品メーカーで **製品リコール調査**を Claude で支援。発生した不具合事例を製造ロット履歴・サプライヤーデータ・出荷記録と突合し、影響範囲を特定。**規制報告（CPSC・各国当局）** には正確な範囲特定が必須。

An electronics maker uses Claude for **recall investigation**: cross-check defect cases with manufacturing lots / supplier data / shipping records to bound impact. **Regulatory reporting (CPSC, national authorities)** requires precise bounding.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) Claude が独断で影響範囲を確定 / Claude solely sets scope
- B) **多段証拠統合 + 出所保持**：①各データソース（ERP・MES・供給品質・FA データ）に対して並列サブエージェント、②結果は **claim → source（ロット ID・装置 ID・出荷先）の構造化マッピング**で統合、③矛盾は明示し優先順位ルール（一次データ > 集計データ）で解決、④最終範囲は **品質保証部門と法務が承認**してから規制当局に報告、⑤継続調査で範囲拡大のシグナルがあれば再評価フローに入る / **Multi-source evidence + provenance**: ①parallel subagents per source (ERP / MES / supplier quality / failure analysis), ②merge as **claim → source (lot ID / equipment ID / shipping destination)**, ③surface conflicts; resolve with priority rules (primary > aggregate), ④Quality and Legal approve before regulatory submission, ⑤scope reopens if new signals emerge
- C) 影響範囲を最大に広げて safe side / Maximally wide scope for "safety"
- D) 範囲を最小限に絞ってコスト最小化 / Minimally narrow to save cost

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

リコール調査は **証拠統合 + 出所保持 + 矛盾解決 + 多部門承認**。範囲最大 / 最小は両方とも企業リスク。

Recall investigation = **evidence merge + provenance + conflict resolution + multi-dept approval**. Max-wide / min-narrow are both wrong.

- **A 不正解**: 規制不適合・訴訟リスク。 / Compliance + litigation risk.
- **C 不正解**: コスト爆発・市場信頼失墜。 / Cost + reputation damage.
- **D 不正解**: 安全リスク・後続リコールで信用失墜。 / Safety + reputation.

**参照 / Reference:** Recall investigation・CPSC
</details>

---

## 問題 48 / Question 48

**シナリオ / Scenario:**

グローバルサプライチェーンで、**地政学リスク（経済制裁・関税変更・港湾ストライキ）** に対する代替調達計画をエージェントで支援。リスク発生時に 24 時間以内に **代替サプライヤー候補と影響シミュレーション**を出すことが求められます。

A global supply chain agent supports geopolitical-risk replanning (sanctions, tariff changes, port strikes); on triggers, 24-hour SLA to produce **alternative suppliers + impact simulation**.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) 平時から **常時稼働の監視エージェント**で世界各地のリスクシグナルを監視（ニュース・政府発表・港湾運行情報）、②シグナル検知時に **代替サプライヤー検索エージェント**を fan-out で起動（地域別・部品別）、③並列で **影響シミュレーションエージェント**（リードタイム・コスト・在庫影響）が動作、④すべての候補を **構造化スコアリング**（リスク・コスト・納期・コンプライアンス・既存契約）で優先順位付け、⑤調達責任者に **24 時間以内の意思決定**用ハンドオフ。OFAC / EU 制裁リストとの **自動照合**は必須 / Always-on signal monitor (news, government, port ops); on detection, fan-out **alternative-supplier search agents** (by region / part) and parallel **impact simulators** (lead time, cost, inventory). Score candidates **structurally** (risk, cost, ETA, compliance, existing contracts) for **24-hour decision** handoff. **OFAC / EU sanctions screening** is mandatory
- B) リスクが起きてから手動で対応 / React manually post-event
- C) 1 エージェントに毎日全部チェックさせる / One agent rechecks everything daily
- D) 代替調達は不要 / No alternative sourcing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

サプライチェーン地政学対応は **常時監視 + 並列調査 + 構造化スコアリング + 制裁チェック + 24h SLA** が標準。

Geopolitical SCM = **always-on watch + parallel research + structured scoring + sanctions check + 24h SLA**.

- **B 不正解**: 24h SLA 未達。 / Misses SLA.
- **C 不正解**: 非効率。 / Inefficient.
- **D 不正解**: ビジネス継続性を捨てる。 / No BCP.

**参照 / Reference:** SCM resilience・sanctions
</details>

---

## 問題 49 / Question 49

**シナリオ / Scenario:**

工場の **OT（Operational Technology）ネットワーク**は IT ネットワークから物理的に分離されています。Claude エージェントは IT 側で動き、OT 側のデータは **データダイオード経由**でしか入手できません（書き込み禁止・読み取りのみ）。

A factory's **OT network** is physically separated from IT. Claude runs on IT side; OT data arrives only via a **data diode** (read-only).

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) Claude を OT 側で直接動かす / Run Claude inside OT
- B) **IT 側で読み取り専用エージェント**を構築：①データダイオード経由で OT テレメトリを受信、②MCP サーバ（OT 側）は **read-only API** のみ公開、③IT 側 Claude は **書き込み系ツールを `allowed_tools` から完全除外**、④異常検知時の指示は **人間運用者経由で OT 側にコマンド入力**（人間が物理的にギャップを越える）、⑤すべての判定は IT 側 WORM ログに保存。**Purdue モデル**遵守 / Build a **read-only IT-side agent**: ①OT telemetry via data diode, ②OT-side MCP exposes **read-only API only**, ③IT-side Claude has **no write tools in `allowed_tools`**, ④on anomalies, instructions go via **human operators bridging the gap**, ⑤all judgments go to IT-side WORM logs. Compliant with **Purdue model**
- C) データダイオードを取り外して双方向通信 / Remove the diode for two-way
- D) Claude を使わない / Don't use Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

OT/IT 分離（Purdue モデル）下では **読み取り専用エージェント + 書き込み不可 + 人間ブリッジ**が標準。データダイオードは物理的セキュリティの根幹。

Under Purdue / OT-IT separation: **read-only agent + no writes + human bridge**. The data diode is core physical security.

- **A 不正解**: OT 側に Claude を入れるのは攻撃面拡大。 / Attack-surface expansion.
- **C 不正解**: セキュリティ設計を破壊。 / Destroys security.
- **D 不正解**: 価値喪失。 / Loses value.

**参照 / Reference:** Purdue model・OT/IT separation
</details>

---

## 問題 50 / Question 50

**シナリオ / Scenario:**

家電メーカーの **製品サポート** で、Claude エージェントが顧客対応 + ファームウェア診断 + 修理予約を実行。ファームウェア更新は **物理製品にプッシュされる**ため、誤った更新は大量回収につながりかねません。

A consumer-electronics support uses Claude for customer ops + firmware diagnostics + repair scheduling. Firmware updates **push to physical devices**; bad updates cause mass recall.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) Claude が直接ファームウェア更新を発火 / Claude triggers firmware updates directly
- B) Claude には **ファームウェア更新の起票権限のみ**を与え（チケット作成）、更新の実発火は **品質保証チームの承認 + カナリアリリース（数百台）+ 数日のソーク + 段階拡大** という既存リリースパイプラインに従う。Claude は **アシスタント役**：症状から候補ビルドを推奨、関連 KB を検索、修理予約のスケジュール提案。**`PreToolUse` フック**で `dispatch_firmware` 等の高影響ツール呼び出しを **必ずチケット ID 紐付け** にし、未承認チケットは実行不可 / Grant Claude **only ticket-creation permission**; actual rollout follows the existing pipeline (QA approval + canary on hundreds of units + multi-day soak + graduation). Claude is **assistant**: suggest candidate builds from symptoms, retrieve KB, schedule repairs. A `PreToolUse` hook ties high-impact calls (e.g., `dispatch_firmware`) to a **ticket ID**; unapproved tickets cannot execute
- C) Claude にすべての権限を与える / Grant Claude full power
- D) 何も自動化しない / Automate nothing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

物理製品への影響を伴う操作は **既存の承認・カナリア・ソークパイプラインを遵守**し、LLM はあくまで支援役。

Operations affecting physical devices follow **existing approval / canary / soak pipelines**; LLM remains assistant.

- **A 不正解**: ブリックリスク・大量回収。 / Brick risk.
- **C 不正解**: 高影響操作の暴発。 / Catastrophic potential.
- **D 不正解**: 効率損失。 / Inefficient.

**参照 / Reference:** Firmware OTA・canary release・PreToolUse
</details>

---

> **次のドメイン / Next domain:** [`domain2_tool_design_mcp.md`](./domain2_tool_design_mcp.md)

