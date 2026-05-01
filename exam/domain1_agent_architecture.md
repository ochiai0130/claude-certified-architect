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

---

> **次のドメイン / Next domain:** [`domain2_tool_design_mcp.md`](./domain2_tool_design_mcp.md)
