# Domain 5: コンテキスト管理と信頼性 / Context Management and Reliability

> 配点比率 / Weight: **15%**
> 問題数 / Questions: **30**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- 逐次要約のリスクと "case facts" ブロック / Progressive summarization risks; "case facts" blocks
- 大規模コードベース調査でのコンテキスト分離 / Context isolation in large-codebase investigation
- 構造化エスカレーション・ハンドオフ / Structured escalation handoffs
- マルチエージェントの部分障害とエラー伝播 / Multi-agent partial failure and error propagation
- 出所保持と矛盾データの提示 / Provenance preservation and conflict surfacing

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

大手損害保険会社のカスタマーサポートで、Claude エージェントが顧客と平均 90 分の電話相当（テキスト換算 50,000 トークン）の長時間対話を扱います。本番で **「最初に聞いた請求金額（$8,432.17）が、対話の終盤では $8,000 と要約されてしまい、誤った金額で支払い処理が走った」** というインシデントが発生。原因はコンテキスト圧縮時の逐次要約による **数値情報の劣化（中間消失効果）** でした。

A P&C insurer's support agent handles 90-minute conversations (~50K tokens). In production, an initial claim amount of `$8,432.17` was summarized to "$8,000" by end-of-conversation, causing a wrong-amount payment. Root cause: progressive summarization caused **numeric drift via the lost-in-the-middle effect**.

**設問 / Question:**

最も効果的な対策はどれですか？

What is the most effective countermeasure?

- A) 要約を行わずすべての履歴をコンテキストに保持し続ける / Never summarize; keep the entire history in context
- B) 重要な数字は対話中に複数回モデルに復唱させる / Have the model recite key numbers multiple times during the conversation
- C) `claude-opus-4-6` の長文コンテキストに頼り、要約をオフにする / Rely on `claude-opus-4-6`'s long context and disable summarization
- D) 対話開始直後と重要事実発生時に **構造化された "case facts" ブロック** （`{ claim_amount: "8432.17", currency: "USD", reported_at: "2026-04-15T10:32:00Z", policy_id: "POL-...", ...}`）を作成し、要約サイクルでも **このブロックは劣化させず常にコンテキスト先頭に保持**。要約は会話の自由文部分のみを対象にする / Build a structured **"case facts" block** (`{ claim_amount: "8432.17", currency: "USD", reported_at: "...", policy_id: "POL-..." }`) at conversation start and on key events, **kept verbatim at the head of context** across summarization cycles. Summarize only the conversational free-text

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

逐次要約の最大の弱点は **数値・日付・固有名詞の劣化** です。これを解決する標準パターンが **"case facts" ブロック** で、要約処理から除外し、構造化された確定事実として **常に文脈先頭**（プロンプトキャッシュも効きやすい位置）に保持します。要約は自由文の会話履歴のみを対象とし、確定事実は劣化させません。これにより精度・コスト・キャッシュヒット率の三方良しになります。

The fundamental weakness of progressive summarization is **drift in numbers, dates, and proper nouns**. The canonical fix is the **"case facts" block** — structured ground-truth kept at the **head of context, exempt from summarization**, and cache-friendly. Summarization targets only conversational free-text.

- **A 不正解**: 全保持はコストが線形に膨張し、中間消失効果は消えても精度向上は限定的。 / Full retention scales linearly in cost and doesn't fully address lost-in-the-middle.
- **B 不正解**: 復唱はモデルの判断に依存し、確定事実保持の決定論的代替にはなりません。 / Recitation is model-dependent and not deterministic.
- **C 不正解**: 長コンテキストでも中間消失効果は残ります。要約を切るだけでは構造化保持の効果は得られません。 / Long context still suffers lost-in-the-middle; disabling summarization alone is insufficient.

**参照 / Reference:** `guide_ja.md` 「7.1 逐次要約のリスク」「7.2 case facts ブロック」「中間消失効果」
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

500 万行の Java + Kotlin モノレポで、Claude Code に **「決済関連のすべての入力検証ロジックを列挙し、CWE-20 の観点で脆弱なものを特定せよ」** という調査を依頼します。グレップ・ファイル読み込み・コールグラフ追跡で大量の中間データが発生しますが、最終的な主エージェントには **「脆弱な箇所のリスト + 優先度」** という凝縮した結果のみを返したい。

In a 5M-line Java + Kotlin monorepo, you ask Claude Code to "enumerate all payment-related input validation logic and identify CWE-20 weaknesses." Grep, file reads, and call-graph traversal produce massive intermediate data, but the main agent should ultimately receive **only the condensed list of weaknesses with priorities**.

**設問 / Question:**

最も適切なコンテキスト管理戦略はどれですか？

Which is the most appropriate context management strategy?

- A) 主エージェントが直接すべての検索・ファイル読み込みを実行し、最後に `/compact` で圧縮 / Main agent does all search/reading, then `/compact` to compress
- B) **`Task` でサブエージェントを起動して別コンテキストで調査**、サブエージェントは中間データを **スクラッチパッドファイル**（`./tmp/audit_findings.md`）に書き出し、メインには **構造化サマリ（脆弱箇所のリスト・該当ファイル：行・優先度・推奨修正方針）のみ** を返す。サブエージェントが大規模化したら `/compact` を併用 / **Spawn a subagent via `Task` in a separate context**, have it write intermediate data to a **scratchpad file** (`./tmp/audit_findings.md`), and return **only a structured summary** (weakness list with file:line, priority, recommended fix) to the main. Combine with `/compact` if subagent grows large
- C) すべての関連ファイルを最初からシステムプロンプトに埋め込む / Embed all relevant files in the system prompt up-front
- D) 1 ファイルずつ別 API コールで処理し、結果をクライアント側で結合する / Process one file per API call and merge client-side

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

大規模調査での原則は **「探索の汚れをサブエージェントに閉じ込める」** です。`Task` で起動したサブエージェントは別コンテキストを持ち、メインに戻すのは **凝縮された構造化結果のみ**。中間データはスクラッチパッドファイルに永続化することで、必要時に再参照でき、メインのトークン消費・中間消失効果を最小化できます。`/compact` はサブエージェント自身のコンテキストが膨らんだ際の補助。

The principle for large investigations is **"isolate exploration noise in a subagent"**. `Task`-spawned subagents have their own context and return only the **structured condensed result** to main. Intermediate data persists to a scratchpad file for later reference, minimizing main-agent token usage and lost-in-the-middle exposure. `/compact` is a complementary tool when the subagent itself grows large.

- **A 不正解**: 主エージェントを直接汚すと、後続タスクの精度低下・トークン爆発・中間消失効果が発生。 / Polluting main causes drift in downstream tasks, token explosion, and lost-in-the-middle.
- **C 不正解**: 5M 行の埋め込みは物理的に不可能であり、必要な部分のみ動的に探すのが正解。 / Impossible at scale; dynamic exploration is required.
- **D 不正解**: 1 ファイル単位ではコールグラフ・横断的な検証ロジックが見えず、調査の質が落ちます。 / File-by-file misses cross-file flow and degrades analysis quality.

**参照 / Reference:** `guide_ja.md` 「7.4 サブエージェントによる調査の分離」「スクラッチパッドファイル」「/compact の使い方」
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

大手証券会社のリテール対応エージェントが、Claude では判断不能なケース（例：500 万円超の特殊取引・コンプライアンス関連の質問）で人間オペレータにエスカレーションします。現状は **会話履歴全体（10,000 トークン）をそのまま貼り付け**ていて、オペレータの「30 秒以内に把握して対応」という SLA を超過しがち、また顧客 ID と取引 ID の特定にも時間がかかっています。

A retail brokerage agent escalates to human operators for cases Claude cannot handle (e.g., transactions > ¥5M, compliance questions). Currently it pastes the entire conversation history (10K tokens), often exceeding the operator's "comprehend and respond in 30 seconds" SLA, and operators struggle to find customer/transaction IDs.

**設問 / Question:**

最も適切なエスカレーションハンドオフ設計はどれですか？

Which escalation handoff design is best?

- A) **構造化ハンドオフスキーマ**を定義：①顧客 ID・口座種別、②直近 5 ターンの要約、③エスカレーション理由（enum）、④推奨アクション、⑤関連取引 ID リスト、⑥モデルの自信度、⑦該当する規制カテゴリ（enum）。会話全文は折りたたみリンクとして添付（必要時に展開可能） / Define a **structured handoff schema**: ①customer ID / account type, ②last-5-turn summary, ③escalation reason (enum), ④recommended action, ⑤related transaction IDs, ⑥model confidence, ⑦applicable regulatory category (enum). Attach the full transcript as a collapsible link for on-demand drill-down
- B) 会話履歴をすべて渡し、「重要箇所はオペレータが見つけてください」と運用ルールで対応 / Continue passing the whole transcript and instruct operators to find the key parts
- C) Claude に「短く要約してオペレータに渡す」と指示する / Instruct Claude to "summarize briefly for the operator"
- D) エスカレーション時はその場で人間にライブコールバックを依頼する / On escalation, request a live human callback in the moment

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

構造化ハンドオフは **オペレータが 30 秒で意思決定**できる情報密度を実現する設計です。鍵は (1) 機械可読な ID 群（顧客・取引）、(2) enum 化された理由・規制カテゴリ（ダッシュボードで集計可能）、(3) 推奨アクション（オペレータの認知負荷低減）、(4) **必要時に深掘りできる全文添付**（情報の取りこぼし防止）。これは医療の SBAR（Situation-Background-Assessment-Recommendation）ハンドオフと同じ思想です。

Structured handoffs target **30-second operator decisions**: machine-readable IDs, enumerated reasons (also dashboard-aggregatable), explicit recommended action (reduces cognitive load), and **on-demand full transcript** (no information loss). Mirrors medical SBAR handoffs.

- **B 不正解**: 10K トークンを毎回読ませる運用は SLA も品質も両立できません。 / 10K-token reads neither hit SLA nor maintain quality.
- **C 不正解**: 自由文要約はオペレータごとに解釈がぶれ、ID の取りこぼしや enum 集計不能で運用悪化。 / Free-text summaries lack ID precision and aggregation.
- **D 不正解**: コールバックは負荷を増やすだけで、情報構造化の本質を解決しません。 / Callbacks add load without addressing structuring.

**参照 / Reference:** `guide_ja.md` 「7.5 構造化エスカレーション」、医療 SBAR ハンドオフ
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

サプライチェーン分析のマルチエージェント調査で、コーディネーターが 4 つのサブエージェントを並列起動：① ERP 在庫照会、② 物流 API 照会、③ 仕入先信用調査、④ 規制制裁リスト照会。仕入先信用調査の外部 API がタイムアウトし **③ だけが失敗**、他は成功しました。コーディネーターは何らかの結論を出す必要があります（夜間バッチ）。

In a supply-chain investigation, the coordinator launches four subagents in parallel: ① ERP inventory, ② logistics API, ③ supplier credit check, ④ sanctions list. The supplier-credit external API times out, so **only ③ fails**; ①②④ succeed. The coordinator must reach a conclusion (overnight batch).

**設問 / Question:**

最も適切なエラー伝播設計はどれですか？

Which error-propagation design is most appropriate?

- A) 1 つでも失敗したらコーディネーター全体を即時失敗させる / Fail the entire coordinator on any subagent failure
- B) サブエージェント ③ は **構造化エラー**（`{ status: "partial_failure", error_type: "external_api_timeout", retryable: true, attempted_query: "...", partial_results: [...], alternatives: ["use cached_credit_score from 7 days ago", "skip supplier and use country-level risk"] }`）を返し、コーディネーターは「キャッシュ済み信用スコアでフォールバック」を選択して **部分結果を明示**した結論を出す。最終出力には「③ は劣化データを使用」と監査追跡可能な注記を残す / Have ③ return **structured error** (`status: "partial_failure"`, `error_type`, `retryable`, `attempted_query`, `partial_results`, `alternatives: [...]`); coordinator selects "use 7-day-old cached credit score" as fallback, **explicitly marks** results as partial, and logs "③ used degraded data" for audit
- C) ③ の失敗を無視して残り 3 つで結論を出す / Silently ignore ③ and conclude from the other three
- D) ③ を成功するまで無限リトライする / Retry ③ infinitely until it succeeds

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

マルチエージェントの正しい部分障害設計は **構造化エラー + 代替案 + 部分結果 + 監査注記** です。コーディネーターが LLM 判断ではなく決定論的にフォールバック戦略（キャッシュ・別ソース・スキップ）を選べる情報構造を返すことが要点。最終出力に **劣化データを使ったことを明示** することで、下流の意思決定者と監査人が品質低下を認識できます。これは航空業界の "graceful degradation" の思想です。

Correct partial-failure design: **structured error + alternatives + partial results + audit annotation**. Returning enough structure for the coordinator to deterministically choose a fallback (cache, alternate source, skip) — not LLM judgment — is the key. **Explicitly marking degraded data** in the final output lets downstream consumers and auditors detect quality drops. This is "graceful degradation."

- **A 不正解**: 1 失敗で全失敗にすると業務継続性が崩壊し、ビジネスインパクトが過大。 / All-fail-on-any breaks business continuity unnecessarily.
- **C 不正解**: 失敗を黙殺すると劣化結果が下流で「正常データ」として扱われ、誤判断の温床。 / Silent failure pollutes downstream as if it were clean data.
- **D 不正解**: 無限リトライはコスト爆発・SLA 超過・サーキットブレーカー不在の典型的アンチパターン。 / Infinite retries explode cost and miss SLAs.

**参照 / Reference:** `guide_ja.md` 「7.6 マルチエージェントエラー伝播」「graceful degradation」「構造化エラー」
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

監査法人のデューデリジェンス調査で、Claude が複数のソース（決算開示・有価証券報告書・ニュース・社内議事録）から「対象企業の 2025 年通期売上高」を収集します。**ソース間で値が食い違う**ことが判明しました：

In an audit DD review, Claude collects "FY2025 revenue" from multiple sources (financial filings, securities report, news, internal minutes). **Sources disagree**:

- 決算短信（確定値）：¥45,210M / Earnings release (final): ¥45,210M
- 有価証券報告書（監査済）：¥45,210M / Annual securities report (audited): ¥45,210M
- 業界ニュース記事：¥46,000M（概数） / Trade press: ¥46,000M (rounded)
- 社内議事録（プレリミナリー）：¥44,800M / Internal minutes (preliminary): ¥44,800M

監査人は **「どのソースから何を引用したか・矛盾がある場合はそれをどう扱ったか」** を明示することを要求しています。

The auditor requires: **"For each claim, show which source it came from and how disagreements were handled."**

**設問 / Question:**

最も適切な出力設計はどれですか？

Which output design is most appropriate?

- A) 4 ソースの平均（¥45,302.5M）を「売上高」として返す / Return the 4-source average (¥45,302.5M)
- B) 一番新しいソース（業界ニュース）の値を採用する / Use the most recent source (trade press)
- C) 各 claim に **`{ value, source, source_type, confidence, retrieved_at }` の構造化メタデータ**を付与し、矛盾がある場合は **「採用値・採用理由（監査済 > 確定 > 概数 > プレリミナリーの優先順位）・除外した値とその理由」を明示**。最終的な claim には監査人が再現できるよう全ソースの footnote を残す / Attach **structured metadata `{ value, source, source_type, confidence, retrieved_at }` to each claim**. On disagreement, **explicitly state the chosen value, the reasoning (priority: audited > final > rounded > preliminary), and the excluded values with reasons**. Attach a full source footnote so the auditor can reproduce the chain
- D) 矛盾があるので「不明」と返す / Return "unknown" because of the disagreement

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

監査・コンプライアンスでは **claim → source の出所保持（provenance）** が最重要であり、矛盾は **隠さず・平均化せず・選択理由を明示**するのが原則です。ソースには階層があり（監査済 > 確定値 > 概数 > プレリミナリー）、選択ロジックを明示することで監査人が判断を再現・検証できます。これは法廷会計・ジャーナリズムでも使われる "show your work" の原則そのものです。

Audit/compliance demands **provenance (claim → source)**; disagreements must not be hidden or averaged — instead, the chosen value, the selection rule, and excluded values with reasons must be explicit. Sources have a hierarchy (audited > final > rounded > preliminary); making the rule explicit allows auditors to reproduce and verify. This is "show your work."

- **A 不正解**: 平均化はソースの重み（監査済の重要性）を消し、規制に通らない数字を作ります。 / Averaging erases source weight and produces a value of no regulatory standing.
- **B 不正解**: "新しさ" は監査の優先順位ではなく、概数を採用するのは事実誤認のもと。 / Recency isn't the audit hierarchy; rounded press numbers aren't authoritative.
- **D 不正解**: 「不明」は調査を放棄しているのに等しく、利用可能な確定値を活用していません。 / "Unknown" abandons the investigation despite having authoritative data.

**参照 / Reference:** `guide_ja.md` 「7.7 出所保持（provenance）」「矛盾データの提示」、監査調書記録の原則
</details>

---

## 問題 6 / Question 6

**シナリオ / Scenario:**

長時間 Claude Code セッションで、複数ファイルを Read した後、メインのコンテキスト消費が 80% を超えています。続行したいが精度劣化が懸念されます。

A long Claude Code session has consumed >80% of main context after reading many files. Continuing risks accuracy drop.

**設問 / Question:**

最も適切な対応はどれですか？ / Best response?

- A) `/compact` で過去の会話を要約圧縮し、未読の重要事実を **構造化メタデータ**として保持。圧縮後も関連ファイル参照は `@path` で再ロード可能。圧縮タイミングは "アクション直前のまとまり" の境界が望ましい / Use `/compact` to summarize-compress prior conversation, keeping key facts as **structured metadata**. Files can be re-loaded via `@path` after compaction. Compact at clean action-boundary moments
- B) コンテキストを無視して続行 / Ignore and continue
- C) セッションを破棄 / Drop the session
- D) `claude-opus-4-6` の長コンテキストに切り替え / Switch to `claude-opus-4-6` long context

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`/compact` でコンテキストを圧縮し継続性を保つ。境界タイミングが質を左右。

`/compact` preserves continuity while shrinking context — boundary timing matters.

- **B 不正解**: 精度劣化が深刻化。 / Accelerates drift.
- **C 不正解**: 進捗喪失。 / Loses progress.
- **D 不正解**: 中間消失効果は残る。 / Doesn't fix drift.

**参照 / Reference:** `/compact`・context boundary
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

サポートエージェントが API レートリミット（`429`）を時々受けます。即座に再試行すると同じエラー。

A support agent occasionally hits API `429`; immediate retry repeats it.

**設問 / Question:**

最も適切な対応はどれですか？ / Best response?

- A) サーバが返す `retry-after` ヘッダ（または `_meta.retry_after_ms`）を尊重し、**指数バックオフ + ジッタ**で再試行。**サーキットブレーカー**を併用し、長時間 429 が続く場合は劣化応答（キャッシュやテンプレ）にフォールバック。すべての試行を相関 ID 付きでログ / Honor server-provided `retry-after` (or `_meta.retry_after_ms`) with **exponential backoff + jitter**. Combine with a **circuit breaker** to fall back to degraded responses (cache, templates) on prolonged 429s. Log all attempts with correlation IDs
- B) 即座に再試行を 100 回 / Retry 100x immediately
- C) 失敗を無視 / Ignore
- D) ユーザーに「やり直し」と言う / Tell user to retry

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

レートリミット対応は **retry-after 尊重 + 指数バックオフ + ジッタ + サーキットブレーカー** が標準。

Rate-limit handling = **respect retry-after + exponential backoff + jitter + circuit breaker**.

- **B 不正解**: 同じ 429 連発。 / Re-triggers 429.
- **C 不正解**: ビジネスインパクト。 / Business impact.
- **D 不正解**: UX 悪化。 / Poor UX.

**参照 / Reference:** Rate limit handling
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

API 呼び出しの **トークン使用量とコスト**を可視化したい。本番でプロンプトの効率改善を継続的に測定する必要がある。

You want token usage and cost visibility to continuously measure prompt efficiency in prod.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) **API レスポンスの `usage`** フィールド（`input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`）を **すべてのリクエストでログ**。プロンプト版・モデル版・ユースケースのタグを付け、**ダッシュボード**で集計。コスト = 入力 × 単価 + 出力 × 単価（キャッシュは割引）で算出。異常値はアラート / Log **`usage`** from every API response (`input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`) with prompt/model/use-case tags. Aggregate in a **dashboard**; cost = input×price + output×price (cache discounted). Alert on anomalies
- B) コストは Anthropic 請求書を見るだけ / Look at the Anthropic invoice only
- C) トークン数は気にしない / Ignore tokens
- D) `claude-opus-4-6` だけ使ってコスト一定 / Always use `claude-opus-4-6` for fixed cost

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

`usage` フィールドの活用 + 構造化タグ + ダッシュボード + アラートが運用標準。

Operational standard: `usage` + structured tags + dashboard + alerts.

- **B 不正解**: 粒度不足。 / Too coarse.
- **C 不正解**: 改善サイクル不能。 / No iteration.
- **D 不正解**: コストは可変。 / Cost varies.

**参照 / Reference:** Usage field・cost monitoring
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

エージェントが **ストリーミング応答**を返している途中でクライアントの接続が切れることがあります。サーバ側で何が起きているか把握したい。

Client connections drop mid-stream; you need server-side visibility.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) `message_delta`・`message_stop` イベントを **完全に処理**してリクエスト完了を確認。途中切断時は **`stop_reason` が出ない / 出ても切断時刻と一致しない**ため、**サーバで使用トークンと出力部分を相関 ID 付きで記録**。ユーザー側にもリクエスト ID を返して再試行可能に / Process **`message_delta` and `message_stop`** to confirm completion. On drop, **`stop_reason` won't arrive cleanly**; **server records used tokens + partial output with correlation ID**. Surface request IDs to clients for safe retry
- B) ストリーミングは使わない / Don't stream
- C) クライアントには「切れたら諦めろ」と言う / Tell clients "give up on drop"
- D) サーバ側ログは取らない / No server logs

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

ストリーミングの可観測性は **イベントモデル + 相関 ID + 部分結果ログ** で実現。

Streaming observability = **event model + correlation ID + partial-result logs**.

- **B 不正解**: UX 悪化。 / Worse UX.
- **C 不正解**: UX 最悪。 / Worst.
- **D 不正解**: 事後分析不能。 / No post-mortem.

**参照 / Reference:** Streaming events・observability
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

長時間 Claude Code セッションで、初期に作成された設計判断を後から参照したいが、要約により失われています。

Long Claude Code sessions lose early design decisions to summarization.

**設問 / Question:**

最も適切な対策はどれですか？ / Best mitigation?

- A) 重要な決定は **scratchpad ファイル**（`./decisions/<topic>.md`）に書き出し、`@decisions/<topic>.md` で必要時に再ロード。設計判断は `/compact` 対象外。決定にはタイムスタンプ・理由・代替案を構造化記述 / Persist key decisions to **scratchpad files** (`./decisions/<topic>.md`); reload via `@decisions/<topic>.md`. Decisions are exempt from `/compact`. Structure each entry with timestamp, rationale, alternatives
- B) 全履歴をシステムプロンプトに残す / Keep full history in system prompt
- C) 決定は記録しない / Don't record
- D) すべて記憶に頼る / Rely on memory

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

設計判断は **永続スクラッチパッド** に外出し、必要時に再ロード。

Design decisions belong in **persistent scratchpads**, reloaded when needed.

- **B 不正解**: トークン爆発。 / Token blow-up.
- **C 不正解**: 後追い不能。 / Untraceable.
- **D 不正解**: 客観性ゼロ。 / Zero objectivity.

**参照 / Reference:** Scratchpad pattern
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

医療チャットの自信度（confidence）出力が **過信**しがちで、規制リスクとなっています。

Medical chat confidence outputs trend overconfident — a regulatory risk.

**設問 / Question:**

最も適切なキャリブレーション手法はどれですか？ / Best calibration?

- A) confidence は使わない / Don't output confidence
- B) **本番代表サンプル**で confidence と実際の正答率を比較し、**信頼度キャリブレーション曲線**（reliability diagram）を作成。系統的過信を検出したら、プロンプトで「強い証拠なしに高 confidence を付与しない」「対立する証拠を必ず明示」などの **過信抑制ルール**を追加。さらに **confidence 閾値**を本番のリスク許容度に合わせて調整 / Compare confidence vs actual accuracy on representative samples to build a **reliability diagram**. On detecting systematic overconfidence, add **anti-overconfidence rules** in the prompt ("require strong evidence for high confidence", "always state contradicting evidence") and **adjust confidence thresholds** to match risk tolerance
- C) confidence を一律 0.5 にする / Force confidence = 0.5 always
- D) confidence は無視 / Ignore confidence

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

信頼度はキャリブレーション曲線で評価し、過信検出時に明示ルール追加 + 閾値調整。

Calibrate confidence via reliability diagrams; on overconfidence add explicit rules + threshold tuning.

- **A 不正解**: トリアージ判断不能。 / Loses triage signal.
- **C 不正解**: 情報量ゼロ。 / Zero info.
- **D 不正解**: 機会損失。 / Missed signal.

**参照 / Reference:** Confidence calibration
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

法務契約抽出で 100 件のサンプル評価を行ったが、契約種別による精度差が確認しづらい。

In a 100-sample legal extraction eval, accuracy variance by contract type is hard to see.

**設問 / Question:**

最も適切な評価手法はどれですか？ / Best eval method?

- A) **層化サンプリング**（stratified sampling）で契約種別ごとに最低 N 件を確保し、種別別精度を **個別に集計**。全体精度が高くても種別差で重大インシデントが起きるため、**最弱種別の精度** をモニタリング指標に。プロダクション分布の変化検知も種別単位で / **Stratified sampling**: ensure ≥N per contract type and report **per-type accuracy**. Even with high global accuracy, weak types cause incidents. Monitor **worst-type accuracy** as a KPI; detect distribution drift per type
- B) ランダムサンプリングで全体精度のみ / Random sampling, overall only
- C) 契約種別は気にしない / Ignore types
- D) すべての種別を均等に扱う / Treat all types equally

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

層化サンプリング + 種別別評価 + 最弱種別 KPI が業務クリティカル評価の標準。

Stratified sampling + per-type evaluation + worst-type KPI = critical-eval standard.

- **B 不正解**: 種別差を見落とす。 / Misses cohort gaps.
- **C 不正解**: 規制不適合。 / Non-compliant.
- **D 不正解**: 過重平均で重大失敗を覆い隠す。 / Hides critical failures.

**参照 / Reference:** Stratified sampling
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

エージェントの応答時間（レイテンシ）が、本番で SLO（p95 < 3s）を超過。

Agent latency exceeds the SLO (p95 < 3s) in prod.

**設問 / Question:**

最も適切な改善はどれですか？ / Best improvement?

- A) **レイテンシのドメイン分解**：①プロンプト処理時間（キャッシュ効きやすさ）、②モデル選定（Haiku vs Sonnet vs Opus）、③ストリーミング有無、④並列ツール呼び出し、⑤プロンプト長、⑥ツール呼び出しチェーンの深さ。各要因を計測し ROI 高い順に最適化（多くの場合：プロンプトキャッシュ → モデル軽量化 → 並列化）。SLO 超過時はアラート + 自動的にモデル軽量化フォールバック / **Decompose latency**: ①prompt processing (cache hit rate), ②model tier (Haiku/Sonnet/Opus), ③streaming, ④parallel tools, ⑤prompt length, ⑥tool-call chain depth. Measure each, optimize by ROI (often: caching → smaller model → parallelize). On SLO breach, alert and auto-fallback to a smaller model
- B) すべて Opus で統一 / Force everything to Opus
- C) レイテンシは無視 / Ignore latency
- D) ストリーミングを切る / Disable streaming

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

レイテンシ最適化は **ドメイン分解 + 計測 + ROI 順最適化 + アラート**。

Latency optimization = **decompose + measure + ROI-optimize + alert**.

- **B 不正解**: コスト爆発、レイテンシ悪化。 / Worsens cost & latency.
- **C 不正解**: SLA 違反。 / Breaches SLA.
- **D 不正解**: TTFT 悪化。 / Worsens TTFT.

**参照 / Reference:** Latency optimization
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

エージェントが **同じ質問を 2 回続けて受け取る**ケース（ユーザーがリロードや再送信）があり、毎回フル処理してコスト増。

The agent receives **the same question twice in a row** (user reload / re-submit), incurring full cost each time.

**設問 / Question:**

最も適切な対策はどれですか？ / Best mitigation?

- A) **レスポンスキャッシュ**を実装：入力ハッシュをキー、レスポンスを短期 TTL（数分）で保持。同一入力には保存済み応答を返す。**冪等キー** をクライアント側で生成して衝突を防ぎ、ユーザー側にも見えるリクエスト ID として活用。プロンプトキャッシュ（モデル側）と組み合わせて二重に効率化 / Implement a **response cache** keyed on input hash with short TTL (minutes); same input returns the stored answer. Generate an **idempotency key** on the client; expose it as request ID. Combine with prompt caching (model-side) for compound savings
- B) 同じ質問でも毎回フル処理 / Fully process each time
- C) 質問を変えるようユーザーに依頼 / Ask user to vary
- D) リロードを禁止 / Disallow reloads

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

応答キャッシュ + 冪等キーで重複処理を排除し、プロンプトキャッシュと併用で最適化。

Response cache + idempotency keys eliminate duplicates; combine with prompt caching.

- **B 不正解**: コスト浪費。 / Wasteful.
- **C 不正解**: UX 悪化。 / Bad UX.
- **D 不正解**: 機能制限。 / Functional limit.

**参照 / Reference:** Response caching・idempotency
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

Anthropic API がリージョン障害を起こした場合の **災害復旧（DR）** を設計しています。

Designing DR for an Anthropic API regional outage.

**設問 / Question:**

最も適切な戦略はどれですか？ / Best strategy?

- A) **マルチリージョン / マルチ提供**：①Anthropic API のリージョンフェイルオーバー、②AWS Bedrock 経由 / Google Vertex 経由などの **代替経路**を冗長化、③重要処理は **テンプレ応答 / 簡易ロジック**で degraded mode 動作、④全障害時は **graceful failure** で UX を保つ（"only retry on stable status" UI）。DR テストを定期実行 / **Multi-region / multi-provider**: ①Anthropic API region failover, ②redundant alternate paths via AWS Bedrock / Google Vertex, ③critical paths fall back to **template / simple-logic** in degraded mode, ④total-outage **graceful failure** preserves UX. Regularly run DR drills
- B) DR は不要 / DR isn't needed
- C) 障害時は手動対応 / Manual response on incident
- D) `claude-opus-4-6` が落ちなければ大丈夫 / Trust `claude-opus-4-6` not to fail

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

DR は **冗長化 + 劣化モード + テスト**が基本。クラウド・LLM 共通の運用原則。

DR = **redundancy + degraded mode + drills** — universal cloud / LLM ops.

- **B 不正解**: 規制不適合・SLA 違反。 / Non-compliant.
- **C 不正解**: 大規模障害時不可能。 / Doesn't scale.
- **D 不正解**: SPOF 思想。 / SPOF mindset.

**参照 / Reference:** DR・multi-region
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

**カナリアデプロイ** で新プロンプトを 5% トラフィックに流して品質を検証したい。

You want canary deploys to test a new prompt on 5% traffic.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) **トラフィックの 5% を新プロンプトにルーティング**（ハッシュベース安定割当）。**品質メトリクス** とコスト・レイテンシを **同じセグメント条件**で比較。閾値超過で自動ロールバック、閾値以内で段階的に 25% / 50% / 100% に拡大。プロンプト版番号と各リクエストの紐付けを記録 / **Route 5% to the new prompt** via stable-hash assignment. Compare **quality metrics + cost + latency** on **matched segments**. Auto-rollback on threshold breach; graduate to 25% / 50% / 100%. Tag every request with prompt version
- B) いきなり 100% に切り替え / Cut over to 100% immediately
- C) カナリアは不要 / Skip canary
- D) ランダムに切り替え / Random switching

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

カナリアは **安定割当 + メトリクス比較 + 自動ロールバック + 段階拡大** が標準。

Canary = **stable assignment + metric comparison + auto-rollback + graduation**.

- **B 不正解**: ビッグバンは失敗時影響大。 / Big-bang risk.
- **C 不正解**: 検証なしは規制不適合。 / Non-compliant.
- **D 不正解**: 不安定。 / Unstable.

**参照 / Reference:** Canary deploy
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

エージェントを **複数顧客** で運用しており、顧客 A の問題で顧客 B が影響を受けないようにしたい。

Multi-tenant agent: customer A's issue must not impact customer B.

**設問 / Question:**

最も適切な隔離設計はどれですか？ / Best isolation design?

- A) **テナント隔離**：①顧客ごとに別 Anthropic API キー（または別組織契約）、②セッション ID とテナント ID の厳密な紐付け、③レートリミット・コスト上限を **テナント単位**で実装、④ログとメトリクスはテナントタグで分離、⑤障害発生時は影響範囲を **テナント単位で局所化**できる体制。SOC 2 / ISO 27001 のテナント分離要件を満たす / **Tenant isolation**: ①per-tenant Anthropic API key (or separate org), ②strict session-tenant binding, ③per-tenant rate limits and cost caps, ④tenant-tagged logs and metrics, ⑤failures localized **per tenant**. Satisfies SOC 2 / ISO 27001 tenant-isolation requirements
- B) すべて 1 つのキーで運用 / One shared key
- C) 顧客分離は不要 / No isolation needed
- D) 顧客ごとに別アプリを書く / Separate apps per customer

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

マルチテナントは **キー分離 + ID 厳密紐付け + 上限分離 + ログ分離 + 障害局所化**が SOC 2 / ISO 27001 標準。

Multi-tenant = **key separation + strict ID binding + per-tenant caps + log separation + blast-radius containment** — SOC 2 / ISO 27001.

- **B 不正解**: 単一障害で全顧客影響。 / Cross-tenant blast radius.
- **C 不正解**: 規制不適合。 / Non-compliant.
- **D 不正解**: 過剰、保守性悪化。 / Overkill.

**参照 / Reference:** Tenant isolation・SOC 2
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

監査人が「12 月 3 日の 14:23 に xxx ユーザーが行った操作の **完全な再現性**」を要求しました。

An auditor demands **complete reproducibility** of an action at 14:23 on Dec 3 by user xxx.

**設問 / Question:**

最も適切な記録項目はどれですか？ / Best recordable items?

- A) **完全再現に必要なすべて**：①入力プロンプト全文、②システムプロンプト版番号、③モデル ID（`claude-opus-4-6` 等）、④モデルパラメータ（temperature, max_tokens 等）、⑤ツール定義スナップショット、⑥`tool_use` 入出力、⑦最終 `text`、⑧`stop_reason`、⑨usage、⑩タイムスタンプ・リクエスト ID・ユーザー ID。これらを **改ざん耐性のある追記専用ストレージ**に保存 / **Everything needed for full reproducibility**: ①full input prompt, ②system prompt version, ③model ID (e.g., `claude-opus-4-6`), ④parameters (temperature, max_tokens, ...), ⑤tool-definition snapshot, ⑥`tool_use` inputs/outputs, ⑦final `text`, ⑧`stop_reason`, ⑨usage, ⑩timestamp / request ID / user ID. Store in **tamper-evident append-only** storage
- B) 最終応答だけ / Final response only
- C) ログは不要 / No logs
- D) 自由記述メモ / Free-text notes

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

完全再現には **全 10 項目 + 改ざん耐性ストレージ**。

Full reproducibility = **all 10 items + tamper-evident store**.

- **B 不正解**: 入力なしで再現不能。 / Can't reproduce.
- **C 不正解**: 規制不適合。 / Non-compliant.
- **D 不正解**: 検索不能。 / Not searchable.

**参照 / Reference:** Audit logging
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

サーキットブレーカーが「**開放（open）**」状態に入った場合の挙動を設計しています。

You're designing circuit-breaker open-state behavior.

**設問 / Question:**

最も適切な挙動はどれですか？ / Best behavior?

- A) サーキットブレーカーは不要 / No circuit breaker
- B) 開放中は **一定時間（cooldown）すべての呼び出しを即座に劣化応答**で返し、依存先を回復させる。**half-open** 試験的呼び出しで成功率を測定し、回復したら **closed** に戻す。閉じる前に成功率閾値（例：5/5 連続成功）を要求する。**メトリクス**（オープン回数・継続時間・降格率）を監視 / On open, **all calls return a degraded response immediately** for the cooldown to let the dependency recover. **Half-open** probes measure recovery; close back when above a success threshold (e.g., 5/5). Monitor **metrics** (open count, duration, degradation rate)
- C) 開放中もすべての呼び出しを試行 / Try every call even when open
- D) 開放後は永遠に開放 / Stay open forever

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

サーキットブレーカーの **open / half-open / closed** 三状態 + メトリクス監視が標準。

Standard CB pattern: **open / half-open / closed** + metrics.

- **A 不正解**: 障害伝播を放置。 / Allows cascade.
- **C 不正解**: ブレーカー意味なし。 / Defeats purpose.
- **D 不正解**: 永久障害扱い。 / Wrong behavior.

**参照 / Reference:** Circuit breaker pattern
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

メッセージキューを介して **非同期に大量リクエスト**を Claude に送る構成。バックプレッシャー（負荷制御）を考えなければなりません。

A queue-driven async architecture sends bulk requests to Claude; backpressure must be handled.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **DLQ + バックプレッシャー**：①ワーカーの並列度を Anthropic API のレートリミットに合わせて調整、②失敗（429・5xx・タイムアウト）はリトライ後に **デッドレターキュー（DLQ）** へ、③DLQ 内容を分析し恒常的失敗パターンを特定、④流量制御は **トークンバケット**でグローバル制御。SLA を超えるリクエストは事前に拒否（fail-fast） / **DLQ + backpressure**: ①worker concurrency aligned with API rate limits, ②failures (429/5xx/timeout) go to **DLQ** after retry, ③analyze DLQ for systemic patterns, ④global flow control via **token bucket**; reject above-SLA requests fast (fail-fast)
- B) ワーカーを無制限にスケール / Unbounded worker scale-out
- C) 失敗を黙殺 / Silently drop failures
- D) リトライを 1000 回 / Retry 1000x

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

非同期キュー設計の標準は **DLQ + バックプレッシャー + トークンバケット + fail-fast**。

Async queue standard = **DLQ + backpressure + token bucket + fail-fast**.

- **B 不正解**: API rate limit 突破で全停止。 / Cascade failure.
- **C 不正解**: 損失無視は規制不適合。 / Non-compliant.
- **D 不正解**: コスト爆発・無限ループ。 / Cost & loops.

**参照 / Reference:** Queue design・DLQ
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

ステークホルダーから「Claude エージェントの **品質指標**を毎月レポートしてほしい」と要請。何を測ればよいか。

Stakeholders want a monthly **quality report**.

**設問 / Question:**

最も適切な指標セットはどれですか？ / Best metric set?

- A) **多面的指標**：①精度（正解率・F1・タスク特性別）、②ハルシネーション率、③false refusal 率、④ユーザー満足度（CSAT / NPS）、⑤トークン使用量とコスト、⑥レイテンシ（p50 / p95 / p99）、⑦エラー率（5xx・スキーマ違反・意味エラー）、⑧人間エスカレーション率、⑨カバレッジ（ユースケース別）、⑩計測可能な業務 KPI 連動。各指標の **トレンドと閾値超過アラート**を追跡 / **Multi-dimensional**: ①accuracy / F1 (per task), ②hallucination rate, ③false-refusal rate, ④CSAT/NPS, ⑤tokens & cost, ⑥latency (p50/p95/p99), ⑦errors (5xx, schema violations, semantic), ⑧human escalation rate, ⑨coverage by use case, ⑩business KPI linkage. Track **trends + threshold alerts**
- B) 精度だけ / Accuracy only
- C) コストだけ / Cost only
- D) ユーザー満足度だけ / CSAT only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

品質は単一指標で表現不能。多面評価が必要。

Quality requires multi-dimensional measurement.

- **B 不正解**: コスト・UX・安全性が見えない。 / Misses cost / UX / safety.
- **C 不正解**: 品質指標ではない。 / Not a quality metric.
- **D 不正解**: 主観のみ。 / Subjective only.

**参照 / Reference:** Quality metrics framework
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

Claude に **長文を要約**させる際、本文の **冒頭と末尾**は精度高いが、中間部の重要事実が落ちる "lost in the middle" が深刻。

Long-document summarization shows strong start/end fidelity but **middle drift** (lost-in-the-middle).

**設問 / Question:**

最も適切な緩和策はどれですか？ / Best mitigation?

- A) **チャンク分割 + 個別要約 + 階層集約**：①長文を意味単位（章・節）でチャンク化、②各チャンクを独立に要約、③要約を **階層的に統合**して全体要約。各チャンクが「先頭・末尾」になるため lost-in-the-middle 影響を分散。重要事実は **構造化抽出フェーズ**を別に通す / **Chunk + per-chunk summarize + hierarchical merge**: ①chunk by semantic units (chapters/sections), ②summarize each independently, ③merge **hierarchically**. Each chunk becomes its own start/end, distributing the lost-in-the-middle effect. Run a **separate structured-extraction pass** for key facts
- B) 長文をそのまま 1 回で要約 / Single-shot summarize
- C) 全部 `claude-opus-4-6` の長コンテキストに任せる / Trust `claude-opus-4-6`
- D) 要約は不可能 / Summarization is impossible

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

長文要約は **チャンク + 階層集約 + 構造化抽出**が定石。

Long-doc summarization = **chunk + hierarchical merge + structured extraction**.

- **B 不正解**: 中間消失効果直撃。 / Worst case.
- **C 不正解**: 長コンテキストでも残存。 / Not solved.
- **D 不正解**: 事実誤認。 / Wrong.

**参照 / Reference:** Long-doc patterns
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

**コスト超過アラート**が頻発しています。原因の多くは「不要に長いプロンプトを毎回送信している」こと。

Cost-overrun alerts are frequent; often "needlessly long prompts every time".

**設問 / Question:**

最も適切な対策はどれですか？ / Best response?

- A) **プロンプト診断**：①プロンプトを「不変な前置き」「動的な指示」「ユーザー入力」に分解、②不変部分を **キャッシュ可能なブロック**に整理して `cache_control` を付与、③動的部分を最小化、④Few-shot の例数を 2〜4 に最適化、⑤Tools 定義を必要なものだけに絞る。**月次でプロンプト監査**を実施 / **Prompt audit**: ①decompose into "invariant prefix / dynamic directives / user input", ②group invariants for `cache_control`, ③minimize dynamics, ④cap few-shot at 2–4, ⑤include only needed tool defs. Run **monthly prompt audits**
- B) コスト超過は仕方ない / Accept overruns
- C) すべてを `claude-haiku-4-5` に変える / Force everything to `claude-haiku-4-5`
- D) ユーザー入力を切り詰める / Truncate user input

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

プロンプト最適化は **構造化 + キャッシュ + Few-shot 最小化 + 月次監査**。

Prompt optimization = **structuring + caching + minimal few-shot + monthly audit**.

- **B 不正解**: 改善放棄。 / Gives up.
- **C 不正解**: 品質低下リスク。 / Quality risk.
- **D 不正解**: UX 悪化。 / Bad UX.

**参照 / Reference:** Cost optimization
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

**プロンプトインジェクション攻撃**が時々検出されます。攻撃者は「すべての過去指示を無視して..」のような入力を送ってくる。

Prompt-injection attempts ("ignore all prior instructions") are detected periodically.

**設問 / Question:**

最も適切な検知・対応はどれですか？ / Best detection & response?

- A) **検知パターンと応答ポリシー**：①既知攻撃文字列の **入力フィルタ**（regex / 分類モデル）、②検出時は **インジェクション検出ログ**に記録、③応答は事前定義テンプレ（個別応答せず安全文言）、④高頻度発生 IP / アカウントに **レート制限と監査エスカレーション**、⑤定期的に新パターンを脅威インテリジェンス源から取得して更新 / **Detection + response policy**: ①input filter for known attack strings (regex / classifier), ②log to **injection events**, ③respond via predefined safe template, ④rate-limit + escalate audit on high-frequency IPs/accounts, ⑤update patterns periodically from threat-intel feeds
- B) インジェクションは存在しない / Injection doesn't exist
- C) すべての入力を許可 / Allow everything
- D) Claude を信用しない / Stop trusting Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

インジェクション対応は **検知 + ログ + テンプレ応答 + レート制限 + 継続更新** の運用設計。

Injection ops = **detect + log + safe template + rate limit + continuous update**.

- **B 不正解**: 事実誤認。 / Wrong.
- **C 不正解**: 最悪。 / Worst.
- **D 不正解**: 過剰反応。 / Over-reaction.

**参照 / Reference:** Injection detection・threat intel
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

エージェントが **ユーザーから受け取った機密情報**（ID・電話番号・メール）を、後続の応答やログにうっかり露出しないようにしたい。

Avoid accidentally exposing user-shared secrets (IDs, phone, email) in subsequent responses or logs.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **PII / 機密情報マスキング層**：①入力前段で PII 検出（regex + ML 分類）、②内部処理ではトークン化（`<EMAIL_TOKEN_42>` 等）、③ログには元の値を残さず置換後を保存、④出力時に必要な箇所のみ復元、⑤監査時に復元できるよう **マッピングテーブル**を別の高セキュリティストアに保管。GDPR / PIPEDA / 個人情報保護法等に対応 / **PII masking layer**: ①detect PII pre-pipeline (regex + ML classifier), ②tokenize internally (e.g., `<EMAIL_TOKEN_42>`), ③logs store masked form only, ④rehydrate on output where needed, ⑤keep mapping in a separate high-security store for auditable recovery. Aligns with GDPR / PIPEDA / local PDPA
- B) PII を平文でログに残す / Plaintext PII in logs
- C) 検出は不要 / No detection
- D) すべての PII を消す（復元不能） / Erase all PII forever

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

PII 対応は **検出 + トークン化 + マスク済みログ + 復元マップ別保管**が標準。

PII handling = **detect + tokenize + masked logs + separate mapping store**.

- **B 不正解**: 規制違反。 / Regulatory breach.
- **C 不正解**: リスク放置。 / Unmitigated risk.
- **D 不正解**: 業務に必要な復元ができない。 / Loses functionality.

**参照 / Reference:** PII handling・GDPR
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

エージェントの **応答時間が突然 4 倍**に遅くなりました。原因切り分けが必要です。

Agent latency suddenly 4x slower.

**設問 / Question:**

最も適切な切り分け手順はどれですか？ / Best triage?

- A) **観測駆動の切り分け**：①ダッシュボードで **どのレイヤー**（API・MCP・自前ロジック）が遅いか確認、②同じプロンプトで **直接 API**を叩いて Anthropic 側か自社側かを切り分け、③プロンプト長 / モデル / `usage` の変化を確認、④外部依存（DB / MCP）の応答時間を確認、⑤キャッシュヒット率の変化を確認、⑥ロールバック候補（直近の変更）を確認。各ステップを **時系列のメトリクス**で裏付け / **Observation-driven**: ①identify which layer (API / MCP / own logic) is slow via dashboards, ②probe API directly to isolate Anthropic vs self, ③check prompt length / model / `usage` deltas, ④measure external deps (DB / MCP), ⑤check cache hit rate, ⑥inspect recent changes for rollback. Back each step with **time-series metrics**
- B) 推測でモデルを変える / Swap model on a hunch
- C) サーバを再起動 / Restart server
- D) 何もしない / Do nothing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

レイテンシトラブルシューティングは **計測駆動** で原因を切り分けるのが鉄則。

Latency triage = **measurement-driven** isolation.

- **B 不正解**: 当て推量は時間浪費。 / Guesswork.
- **C 不正解**: 根本原因不明のまま。 / Doesn't fix root.
- **D 不正解**: SLA 違反継続。 / SLA breach continues.

**参照 / Reference:** Production triage
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

法令変更で「**生成 AI による意思決定の説明可能性**」が義務化（EU AI Act 等）。エージェントが下した判断について、後から **理由**を提示できる必要があります。

A regulation (EU AI Act etc.) mandates **explainability** of AI decisions; you must produce reasons after the fact.

**設問 / Question:**

最も適切な実装はどれですか？ / Best implementation?

- A) **意思決定 + 説明をペアで構造化**：①`tool_use` で `{ decision, reasoning_steps[], evidence_sources[], counterfactuals[], confidence }` を出力、②**reasoning_steps** は具体的根拠と推論連鎖、③**evidence_sources** はソース ID と引用箇所、④**counterfactuals** は「もし入力が違ったら結論はどう変わるか」、⑤すべて監査ログに保存。後から第三者が再現・説明可能 / **Decision + explanation as a structured pair**: ①`tool_use` returns `{ decision, reasoning_steps[], evidence_sources[], counterfactuals[], confidence }`; ②**reasoning_steps** = concrete grounds and chain; ③**evidence_sources** = source IDs + spans; ④**counterfactuals** = "if input differed, would the decision change?"; ⑤full audit log. Third parties can replay and explain
- B) 説明は不要 / No explanation
- C) 説明は自由文で / Free-text explanation
- D) `claude-opus-4-6` なら自動で説明可能 / `claude-opus-4-6` auto-explains

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

説明可能性は **構造化された決定 + 推論連鎖 + 証拠 + 反事実 + 監査ログ** で達成。

Explainability = **structured decision + reasoning chain + evidence + counterfactuals + audit log**.

- **B 不正解**: EU AI Act 違反。 / Regulatory breach.
- **C 不正解**: 検証性が低い。 / Not verifiable.
- **D 不正解**: モデル能力では構造化保証なし。 / Not guaranteed.

**参照 / Reference:** EU AI Act・explainability
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

エージェントの **モデル移行**（例：Claude 4.5 → 4.6）を計画。互換性破壊や品質劣化のリスクをどう抑えるか。

Migrating models (e.g., Claude 4.5 → 4.6); how to mitigate compatibility / quality regressions.

**設問 / Question:**

最も適切な手順はどれですか？ / Best sequence?

- A) **段階移行**：①回帰テストスイートを新モデルで実行、②品質メトリクス（精度・ハルシネーション・false refusal）の差分確認、③カナリア（5%）で段階的トラフィック切替、④しきい値超過で自動ロールバック、⑤問題なければ 25/50/100% に拡大。**プロンプトの微調整**が必要な場合は新モデル向け版を A/B、版番号を全リクエストに記録 / **Staged migration**: ①run regression suite on the new model, ②diff quality metrics (accuracy / hallucination / false refusal), ③canary 5%, ④auto-rollback on breach, ⑤graduate to 25/50/100. If prompt tweaks are needed, A/B-test and tag every request with prompt version
- B) 一気に切り替え / Cut over all at once
- C) 移行しない / Never migrate
- D) ランダムに切り替え / Randomly switch

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

モデル移行は **回帰テスト + 段階展開 + 自動ロールバック + 版管理**。

Model migration = **regression + staged rollout + auto-rollback + versioning**.

- **B 不正解**: ビッグバンリスク。 / Big-bang risk.
- **C 不正解**: 改善機会喪失。 / Missed gains.
- **D 不正解**: 不安定。 / Unstable.

**参照 / Reference:** Model migration
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

事業継続のために **エージェントの SLA**（例：可用性 99.9%、p95 レイテンシ 3 秒、月次コスト上限）を定義したい。

Define agent **SLAs** for business continuity (availability 99.9%, p95 3s, monthly cost cap).

**設問 / Question:**

最も適切な SLO/SLI 設計はどれですか？ / Best SLO/SLI design?

- A) **SLO（目標）+ SLI（指標）+ エラーバジェット**：①可用性 SLO 99.9% に対する SLI として "5xx 以外で意味的に有効な応答" を定義、②p95 レイテンシ SLI を測定、③月次コスト上限を予算 SLO に、④エラーバジェット消費を可視化し、消費過多のときは **新機能ロールアウトを停止**して安定化に集中、⑤定期レビューで SLO を見直す。SRE 原則に準拠 / **SLO + SLI + error budget**: ①availability SLO 99.9% with SLI "non-5xx, semantically valid response", ②p95 latency SLI, ③monthly cost cap as budget SLO, ④visualize error-budget burn — **freeze new rollouts** when over-burning, focus on stability, ⑤review SLOs periodically. Aligned with SRE
- B) SLA は不要 / No SLAs
- C) SLA は宣言だけ / Declare without measurement
- D) 100% を目指す / Aim for 100%

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

SRE 原則の **SLO + SLI + エラーバジェット**で運用品質を担保。

SRE pattern = **SLO + SLI + error budget** for operational quality.

- **B 不正解**: 運用基準なし。 / No baseline.
- **C 不正解**: 宣言のみは無意味。 / Mere declaration.
- **D 不正解**: 100% は経済合理性なし。 / Not economically rational.

**参照 / Reference:** SRE / SLO / SLI
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

最後の問題。エージェントが運用に乗ってから **6 ヶ月**経過。継続改善のために何をすべきか。

After 6 months in production, what should you do for continuous improvement?

**設問 / Question:**

最も適切なルーチンはどれですか？ / Best routine?

- A) 改善は不要 / No improvement needed
- B) **継続改善ルーチン**：①月次の **品質メトリクスレビュー** で劣化トレンドを検出、②本番ログから **失敗事例**を抽出して評価集合に追加、③**プロンプト・ツール・モデル・スキーマ**の各次元で A/B 改善、④**エッジケース** が増えるごとに Few-shot や検証ルールを更新、⑤コンプライアンス・規制変更への追従、⑥**新モデル**の定期評価と移行、⑦**コスト最適化**監査、⑧チームへの **継続教育**。すべて版管理 + メトリクス比較で科学的に / **Continuous improvement routine**: ①monthly **quality metric review** to detect drift, ②mine production logs for **failure cases** added to the eval set, ③A/B across **prompts / tools / models / schemas**, ④update few-shot / validation rules as edge cases accumulate, ⑤track regulatory updates, ⑥periodic **new-model** evals and migration, ⑦**cost-optimization** audits, ⑧team **continuous education**. All version-controlled with metric comparison
- C) 6 ヶ月後はそのまま運用 / Just run it
- D) 全部作り直す / Rewrite everything

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

LLM システムは生き物：**継続的な計測・学習・改善**のサイクルが運用品質を決める。

LLM systems are living: **measure / learn / iterate** cycles determine operational quality.

- **A 不正解**: 環境変化で劣化。 / Drifts with environment.
- **C 不正解**: 改善機会喪失。 / Missed gains.
- **D 不正解**: 過剰反応。 / Overreaction.

**参照 / Reference:** Continuous improvement・LLM ops
</details>

---

> **前のドメイン / Previous:** [`domain4_prompt_structured_output.md`](./domain4_prompt_structured_output.md) | **目次 / Index:** [`README.md`](./README.md)
