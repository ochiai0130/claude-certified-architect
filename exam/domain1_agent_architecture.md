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

> **次のドメイン / Next domain:** [`domain2_tool_design_mcp.md`](./domain2_tool_design_mcp.md)

