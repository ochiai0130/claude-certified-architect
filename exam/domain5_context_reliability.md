# Domain 5: コンテキスト管理と信頼性 / Context Management and Reliability

> 配点比率 / Weight: **15%**
> 問題数 / Questions: **5**
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

## 問題 31 / Question 31

**シナリオ / Scenario:**

HFT（高頻度取引）デスクで、Claude を **戦略チューニング**（夜間バッチ）に使う構成。本番取引には影響しないが、**過去市場データに対する再現性**が重要で、後日同じデータで同じ結果が出る必要があります。

An HFT desk uses Claude for **strategy tuning** (overnight batch); not in trading path, but **reproducibility on historical market data** is required.

**設問 / Question:**

最も適切な再現性設計はどれですか？ / Best reproducibility design?

- A) **完全な版管理 + 決定論的設定**：(i) Claude モデル ID（`claude-opus-4-7` 等の **完全に固定された snapshot ID**）、(ii) `temperature: 0`、(iii) プロンプト版番号、(iv) ツール定義スナップショット、(v) **市場データの版数**（snapshot timestamp + ハッシュ）、(vi) すべてを **再実行用マニフェスト**に記録、(vii) 再実行時はマニフェストから決定論的に復元。version drift の検出は **重要メトリクスで自動アラート** / **Full versioning + deterministic settings**: (i) Claude **fully fixed snapshot ID** (e.g., `claude-opus-4-7`), (ii) `temperature: 0`, (iii) prompt version, (iv) tool-definition snapshot, (v) **market-data version** (timestamp + hash), (vi) record all in a **rerun manifest**, (vii) reruns deterministically restore from it. Version drift triggers **auto-alerts on key metrics**
- B) ベストエフォートで再現 / Best-effort reproduction
- C) 再現性は不要 / No reproducibility
- D) 各セッションごとに新モデル / New model each session

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

HFT 戦略チューニングは **完全マニフェスト + 固定モデル + temp 0 + データ版数**。

HFT tuning = **complete manifest + fixed model + temp 0 + data versioning**.

- **B 不正解**: 検証不能。 / Not verifiable.
- **C 不正解**: バックテスト無効化。 / Invalidates backtests.
- **D 不正解**: 戦略の継続改善不能。 / Can't iterate.

**参照 / Reference:** HFT reproducibility・model snapshots
</details>

---

## 問題 32 / Question 32

**シナリオ / Scenario:**

銀行で **取引監視データ（transaction monitoring）** を 7 年保管が義務（米国 BSA / EU AMLD）。Claude エージェントの判定ログも対象。

A bank must retain transaction-monitoring data 7 years (US BSA / EU AMLD); Claude agent logs are in scope.

**設問 / Question:**

最も適切な保管設計はどれですか？ / Best retention design?

- A) **WORM + 階層化ストレージ + リーガルホールド対応**：(i) 直近 90 日は高速ストレージ、それ以降は WORM（S3 Object Lock / Compliance Mode）、(ii) **暗号化（KMS 経由のキーローテーション + at-rest / in-transit）**、(iii) アクセスは多要素認証 + 監査ログ、(iv) **削除は不可能化**（保管期間内）、(v) リーガルホールド時は対象期間を期限延長で物理的に保護、(vi) 規制照会時は **インデックス検索**で迅速取り出し / **WORM + tiered storage + legal-hold ready**: (i) recent 90d on hot storage, older on WORM (S3 Object Lock / Compliance Mode), (ii) **encryption (KMS rotation + at-rest / in-transit)**, (iii) access via MFA + audit log, (iv) **deletion impossible** within retention window, (v) on legal hold, extend period to physically protect, (vi) for regulator queries, **indexed search** for rapid retrieval
- B) 7 年経過したら削除 / Delete after 7 years
- C) ローカル DB に平文保管 / Plaintext local DB
- D) 保管しない / No retention

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

規制保管は **WORM + 階層 + 暗号化 + MFA + リーガルホールド + インデックス**。

Regulated retention = **WORM + tiering + encryption + MFA + legal hold + indexing**.

- **B 不正解**: リーガルホールド時に問題。 / Legal-hold conflict.
- **C 不正解**: セキュリティ違反。 / Security violation.
- **D 不正解**: 違法。 / Illegal.

**参照 / Reference:** BSA/AMLD retention・S3 Object Lock
</details>

---

## 問題 33 / Question 33

**シナリオ / Scenario:**

ウェルスマネジメントで、長期顧客（5〜20 年付き合い）の **顧客プロファイル** を Claude で活用。投資方針・リスク許容度・家族状況・税務状況など長期的な変化を文脈として保持。

Wealth management agents use Claude with **long-term client profiles** (5–20 yr relationships): policies / risk tolerance / family / tax — preserving long-term changes in context.

**設問 / Question:**

最も適切なコンテキスト戦略はどれですか？ / Best context strategy?

- A) **構造化長期プロファイル + イベント年表**：(i) 顧客プロファイルを **構造化**（age, risk_tolerance, family_changes, financial_goals, tax_situation の現在値 + 履歴）、(ii) 重要イベントは **タイムスタンプ付きイベント年表**として保管（結婚 / 出産 / 退職 / 大規模売買等）、(iii) Claude セッションでは **最新のプロファイル + 直近 6 ヶ月の関連イベントのみ**を文脈に乗せる（古いものは省略可だがアクセス可能）、(iv) **fiduciary duty** 関連の決定はすべて WORM 保管 / **Structured long-term profile + event timeline**: (i) profile **structured** (age / risk tolerance / family changes / financial goals / tax) with current + history, (ii) significant events as a **timestamped timeline** (marriage / birth / retirement / major liquidity event), (iii) Claude sessions use **only the latest profile + last 6 months of relevant events** in context (older accessible on demand), (iv) all **fiduciary-duty** decisions in WORM
- B) 全履歴を毎回コンテキストに入れる / All history every time
- C) 履歴は無視 / Ignore history
- D) 顧客プロファイルは作らない / No profile

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

長期顧客は **構造化プロファイル + イベント年表 + 必要時取得 + 決定 WORM**。

Long-term clients = **structured profile + event timeline + on-demand retrieval + decision WORM**.

- **B 不正解**: トークン爆発 + 中間消失。 / Cost + drift.
- **C 不正解**: 顧客理解低下。 / Loses understanding.
- **D 不正解**: 業務不能。 / Insufficient.

**参照 / Reference:** Wealth management context
</details>

---

## 問題 34 / Question 34

**シナリオ / Scenario:**

決済プラットフォームの **24/7 サポートエージェント**で、Claude が顧客対応中に **PCI DSS 準拠**を維持。長時間対話で誤ってカード番号を発話 / 表示するリスクをどう管理するか。

A 24/7 payment-support Claude must remain **PCI DSS-compliant** in long calls; manage risk of speaking / displaying card numbers.

**設問 / Question:**

最も適切な多層防御はどれですか？ / Best layered defense?

- A) **入出力フィルタ + コンテキストガード + 監視**：(i) 入力に PAN パターンが含まれる場合は **入力レイヤーで自動マスク**（Claude のコンテキストに入る前）、(ii) 出力レイヤーでも **PAN パターン正規表現マスク**、(iii) システムプロンプトで「カード番号を絶対に表示しない」明示、(iv) 各ターンでカード番号が文脈に侵入していないかを **PreToolUse フックでスキャン**、検出時は即座にセッション中断 + インシデント記録、(v) リアルタイム DLP モニタリング / **Input/output filters + context guard + monitoring**: (i) input layer **auto-masks PAN patterns** before they enter context, (ii) output layer regex-masks PAN, (iii) system prompt forbids display, (iv) each turn **scans for PAN ingress via `PreToolUse` hook**; on detection halt session + record incident, (v) real-time DLP monitoring
- B) システムプロンプトのみ / Prompt only
- C) フィルタ不要 / No filters
- D) Claude を使わない / Don't use Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

PCI 多層防御は **入力マスク + 出力マスク + プロンプト + フックスキャン + DLP**。

PCI defense in depth = **input mask + output mask + prompt + hook scan + DLP**.

- **B 不正解**: 確率的、PCI 違反。 / Probabilistic.
- **C 不正解**: 違反。 / Breach.
- **D 不正解**: 過剰反応。 / Overreaction.

**参照 / Reference:** PCI DSS layered defense
</details>

---

## 問題 35 / Question 35

**シナリオ / Scenario:**

銀行の **顧客苦情処理（complaint resolution）** で、CFPB（消費者金融保護局）に **60 日以内の応答義務**。複雑なケースは複数の Claude セッションを跨ぐことがあります。

A bank's complaint resolution must respond within 60 days (CFPB); complex cases span multiple Claude sessions.

**設問 / Question:**

最も適切なコンテキスト管理はどれですか？ / Best context management?

- A) **永続的な complaint case ID + 構造化進捗 + SLA 監視**：(i) すべての関連セッションを `complaint_id` で紐付け、(ii) 各セッションの結論を **構造化進捗ログ**（actions_taken / pending_items / next_review_date）として永続化、(iii) 新セッション開始時はログをコンテキスト先頭に再ロード（case facts ブロック）、(iv) **SLA カウンタ**（60 日）を全セッションで参照、残日数に応じて urgency を上げる、(v) すべての顧客通信は WORM ログ / **Persistent complaint case ID + structured progress + SLA monitoring**: (i) all sessions tied via `complaint_id`, (ii) each session's outcome is a **structured progress log** (actions_taken / pending_items / next_review_date), (iii) new sessions reload it as the **case facts block** at the head of context, (iv) **SLA counter (60d)** referenced across sessions; urgency rises near deadline, (v) WORM customer comms
- B) セッションごとに記憶リセット / Reset per session
- C) Claude に毎回最初から読ませる / Re-read full history every time
- D) SLA は気にしない / Ignore SLA

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

長期苦情処理は **case ID + 構造化進捗 + case facts + SLA カウンタ + WORM**。

Long complaint flows = **case ID + structured progress + case facts + SLA counter + WORM**.

- **B 不正解**: 連続性破綻、再質問で顧客苦情。 / Loses continuity.
- **C 不正解**: トークン爆発、中間消失。 / Cost + drift.
- **D 不正解**: CFPB 違反。 / Compliance breach.

**参照 / Reference:** CFPB・case management
</details>

---

## 問題 36 / Question 36

**シナリオ / Scenario:**

慢性疾患（糖尿病・心不全等）の **長期患者管理**で、Claude が複数年に渡る診療履歴を扱います。新しい受診時に過去 3〜5 年の関連履歴を効率的に活用したい。

For chronic disease management (diabetes / heart failure), Claude handles multi-year histories; each visit must efficiently leverage 3–5 years of relevant history.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **構造化メディカルレコード + 関連性スコアリング**：(i) 患者プロファイルを構造化（活動疾患・服薬・直近検査値・既往）、(ii) 過去履歴を **時系列イベントストア**として保管、(iii) 受診時は MCP ツール `get_relevant_history(patient_id, current_complaint, max_events)` で **直近受診の主訴に関連する過去イベントのみ**を取得（時系列 + 臨床関連性）、(iv) 検査値は **ベースライン + 直近 + トレンド**として要約、(v) Claude のコンテキストには関連箇所のみ、PHI は最小権限でアクセス / **Structured medical records + relevance scoring**: (i) structured profile (active conditions / meds / recent labs / history), (ii) past history in a **timestamped event store**, (iii) per visit, MCP tool `get_relevant_history(patient_id, current_complaint, max_events)` returns **only events relevant to current complaint** (chronological + clinical relevance), (iv) labs as **baseline + recent + trend** summaries, (v) Claude sees only relevant parts; PHI accessed under least privilege
- B) 全カルテを毎回 / Full chart every visit
- C) 過去は無視 / Ignore history
- D) Claude を使わない / Don't use

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

慢性疾患管理は **構造化記録 + イベントストア + 関連性スコアリング + 検査値トレンド + 最小権限**。

Chronic care = **structured records + event store + relevance scoring + lab trending + least privilege**.

- **B 不正解**: トークン爆発・PHI 過大露出。 / Cost + PHI overexposure.
- **C 不正解**: 慢性疾患管理不能。 / Can't manage chronics.
- **D 不正解**: 機会損失。 / Lost value.

**参照 / Reference:** Chronic care・relevance retrieval
</details>

---

## 問題 37 / Question 37

**シナリオ / Scenario:**

製薬の **フェーズ III 臨床試験**（5 年・複数施設・数千被験者）で、Claude を試験運営支援に活用。**プロトコル修正履歴・有害事象履歴・データ凍結時点**をすべて時系列で管理。

A 5-year, multi-site, thousands-of-subject Phase III trial uses Claude for ops; all **protocol amendments / AE history / data freeze points** managed chronologically.

**設問 / Question:**

最も適切なコンテキスト戦略はどれですか？ / Best context strategy?

- A) **時点バインディング + 不変版数 + 事象別チャンク**：(i) すべての判定は **判定時点のプロトコル版数**にバインド（`protocol_v3.2 effective from 2026-01-15`）、(ii) **データロック時点**でのデータセットを永続化、(iii) 患者ごとの履歴は **イベントストア + チャンク**（事象ごとに独立、必要時に検索）、(iv) Claude は判定時点のプロトコル版を文脈先頭に保持、(v) すべての判定 + 引用 + プロトコル版を WORM、再現可能性を試験終了後 25 年保管（FDA 要件） / **Time-binding + immutable versioning + event chunks**: (i) every decision binds to the **active protocol version at decision time** (`protocol_v3.2 effective from 2026-01-15`), (ii) **data-lock snapshots** persist, (iii) per-patient history as **event store + chunks** (independent events, retrieved on demand), (iv) Claude keeps the active protocol version at head-of-context, (v) decisions + citations + protocol version go to WORM; retain 25 years post-trial (FDA)
- B) すべての履歴を毎回 / Full history every time
- C) プロトコル変更は無視 / Ignore protocol changes
- D) 短期ローカル保存のみ / Short local store only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

長期試験は **時点バインディング + 版固定 + イベントチャンク + 25 年保管**。

Long trials = **time binding + immutable versions + event chunks + 25-year retention**.

- **B 不正解**: トークン爆発。 / Cost blow-up.
- **C 不正解**: 判定の妥当性喪失。 / Validity lost.
- **D 不正解**: 規制違反。 / Regulatory breach.

**参照 / Reference:** Phase III trial・FDA retention
</details>

---

## 問題 38 / Question 38

**シナリオ / Scenario:**

総合病院の **ICU 患者モニタリング**で、Claude が 24x7 のバイタルサイン・薬剤投与・看護師記録を扱う。**重症者なので即応 SLA（数分以内）が要求**される一方、ノイズアラート過多は alarm fatigue を生む。

ICU monitoring uses Claude on 24x7 vitals / meds / nursing notes; **acute care needs minute-level SLA** but excess alerts cause alarm fatigue.

**設問 / Question:**

最も適切なバランス設計はどれですか？ / Best balance?

- A) **多段階フィルタ + 臨床妥当性 + 看護師フィードバック**：(i) **第 1 段**：機械的閾値（バイタル ranges）でクリティカルのみフィルタ、(ii) **第 2 段**：Claude が患者個別の文脈（既往・現治療）と照合し **臨床的に意味あるアラートのみ生成**、(iii) **第 3 段**：信頼度に応じてアラート優先度（critical / urgent / informational）、(iv) **看護師フィードバック**（false positive / true positive ラベル）を継続学習、(v) すべてのアラートは WORM ログ、(vi) **臨床決定権は医師 / 看護師**、Claude は補助 / **Multi-stage filter + clinical relevance + nurse feedback**: (i) Stage 1: numeric thresholds filter critical only, (ii) Stage 2: Claude reconciles with **patient-specific context** (history / current treatment) and **emits only clinically meaningful alerts**, (iii) Stage 3: alert priority by confidence (critical / urgent / informational), (iv) **nurse feedback labels** (FP / TP) for continuous tuning, (v) WORM-log all alerts, (vi) **clinical decisions = MD / RN**, Claude assists
- B) すべての変化でアラート / Alert on every change
- C) アラートは出さない / No alerts
- D) すべての変化を Claude に判断させる / Have Claude decide everything

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

ICU は **多段フィルタ + 臨床文脈 + 優先度 + 看護師フィードバック + 補助役**。

ICU = **multi-stage filter + clinical context + priority + nurse feedback + assistive**.

- **B 不正解**: alarm fatigue で本物見逃し。 / Fatigue, missed events.
- **C 不正解**: 救命機会喪失。 / Loses life-saving signal.
- **D 不正解**: 越権。 / Out of scope.

**参照 / Reference:** Alarm fatigue・clinical context
</details>

---

## 問題 39 / Question 39

**シナリオ / Scenario:**

医療請求の **データバックフィル（過去 5 年分の再処理）** に Claude を使う計画。本番運用は別系統で、バックフィルは **数百万件**を扱います。再処理結果を本番に統合する前の検証が必要。

A medical claims **backfill (5 years)** uses Claude; production runs separately. Backfill processes millions of records; results need validation before merging into production.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **シャドウモード + 段階統合**：(i) Backfill 結果を **本番と並列のシャドウテーブル**に書き込み、(ii) 既存判定との **差分分析**（一致 / 軽微差 / 重大差をカテゴリ別に集計）、(iii) サンプル N 件を医療コーディネーターが **手動レビュー**、(iv) 重大差の根本原因を分析（プロンプト改善 / モデル更新 / データ品質）、(v) 検証済み部分のみ **段階的に本番に統合**（カナリア → 全件）、(vi) すべての処理ログを WORM 保管 / **Shadow mode + staged merge**: (i) backfill writes to a **shadow table parallel to prod**, (ii) **diff vs existing** judgments (agree / minor diff / major diff) categorized, (iii) human medical coders review **N samples**, (iv) RCA on major diffs (prompt / model / data quality), (v) merge **only validated subsets** into prod (canary → full), (vi) WORM-log everything
- B) いきなり本番統合 / Cut over directly
- C) シャドウなしで本番上書き / Overwrite prod directly
- D) 検証なしで信頼 / Trust without validation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

大規模バックフィルは **シャドウ + 差分分析 + 手動レビュー + RCA + 段階統合 + WORM**。

Large backfill = **shadow + diff analysis + human review + RCA + staged merge + WORM**.

- **B 不正解**: 大量誤判定でビジネスインパクト。 / Mass error risk.
- **C 不正解**: 同上 + 監査不能。 / Same + not auditable.
- **D 不正解**: 規制不適合。 / Non-compliant.

**参照 / Reference:** Shadow mode・data backfill
</details>

---

## 問題 40 / Question 40

**シナリオ / Scenario:**

医療系 SaaS で、複数クライアント（病院・診療所・保険会社）に提供しているサービスで **Claude モデル更新**（4.5 → 4.7 等）を実施する場合の影響評価。クライアントごとに異なる業務フロー。

A medical SaaS serves hospitals / clinics / insurers; **Claude model updates** (4.5 → 4.7) require impact assessment across diverse client workflows.

**設問 / Question:**

最も適切な手順はどれですか？ / Best procedure?

- A) **クライアント別影響評価 + コンセント駆動展開**：(i) クライアント毎にユースケース別の **回帰評価セット**を保有、(ii) 新モデルでの **全クライアント評価結果**をスコアカード化、(iii) **クライアント承認制**でロールアウト（早期採用希望クライアントから順次）、(iv) 全クライアントが **fallback** 可能（前モデル snapshot を一時的に維持）、(v) クライアントごとの本番でのキー指標を継続監視、(vi) 複数の HIPAA / FDA 対応文書を更新 / **Per-client impact + consent-driven rollout**: (i) per-client use-case-specific **regression eval sets**, (ii) score-cards for **per-client results on the new model**, (iii) **opt-in rollout** by client (early adopters first), (iv) all clients can **fall back** (prior model snapshot preserved), (v) continuously monitor per-client KPIs, (vi) update HIPAA / FDA documentation
- B) 全クライアント一斉に切替 / Cut over all clients at once
- C) モデル更新しない / Never update
- D) クライアントには無断で切替 / Switch without consent

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

ヘルスケア SaaS のモデル更新は **クライアント別評価 + コンセント + フォールバック + 監視 + ドキュメント更新**。

Healthcare SaaS model updates = **per-client eval + consent + fallback + monitoring + documentation**.

- **B 不正解**: ビッグバンリスク。 / Big-bang risk.
- **C 不正解**: 改善機会喪失。 / Lost gains.
- **D 不正解**: 契約違反、信頼喪失。 / Breach + trust loss.

**参照 / Reference:** Healthcare SaaS rollout
</details>

---

## 問題 41 / Question 41

**シナリオ / Scenario:**

法律事務所で、**マター（matter / 案件）** ごとに **数年に渡る** 契約交渉履歴・会議メモ・草案を保管。新しい弁護士が引き継ぐ際の **オンボーディング**に Claude を活用。

A law firm uses Claude for **onboarding new attorneys** to multi-year matters with contract negotiations / meeting notes / drafts.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **マター別ナレッジベース + 階層的サマリ**：(i) マターごとに **専用 MCP サーバ**（あるいはディレクトリ + skill）、(ii) 全文書を時系列 + ドキュメント種別で構造化、(iii) **階層的サマリ**（エグゼクティブ / 詳細 / 全文）を予め生成、(iv) 新弁護士のオンボーディング時はエグゼクティブから始め、必要時に詳細展開、(v) **守秘義務**遵守のため、他マター情報は `context: fork` で隔離、(vi) すべてのアクセスは **倫理ウォール**（Chinese wall）でテナント分離 / **Per-matter knowledge base + hierarchical summaries**: (i) per-matter **MCP server** (or dir + skill), (ii) docs structured by date + type, (iii) **hierarchical summaries** (executive / detail / full), (iv) onboarding starts from executive, drills down on demand, (v) **confidentiality**: other-matter info isolated via `context: fork`, (vi) **ethical wall** (Chinese wall) tenant isolation
- B) 全マター 1 つの DB / All matters in one DB
- C) オンボーディングは口頭のみ / Verbal onboarding only
- D) Claude を使わない / Don't use Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

マター別ナレッジは **専用サーバ + 階層サマリ + 倫理ウォール + 守秘**。

Matter knowledge = **per-matter server + hierarchical summaries + ethical wall + confidentiality**.

- **B 不正解**: 倫理違反。 / Ethical breach.
- **C 不正解**: スケールしない。 / Doesn't scale.
- **D 不正解**: 効率損失。 / Lost efficiency.

**参照 / Reference:** Matter-centric knowledge management
</details>

---

## 問題 42 / Question 42

**シナリオ / Scenario:**

法律事務所の **タイムシート管理**で、Claude が会議録音 / メール履歴から作業時間を推定。**請求の正確性が顧客信頼の根幹**で、過剰請求は懲戒、過少請求は事務所の損失。

A law firm estimates billable hours from meeting recordings / email via Claude. **Billing accuracy** is foundational to client trust; over-billing risks discipline, under-billing loses revenue.

**設問 / Question:**

最も適切な信頼性設計はどれですか？ / Best reliability design?

- A) **複数情報源クロスチェック + 弁護士最終承認**：(i) 各タスクの推定時間に対して **複数情報源**（カレンダー・メール・ドキュメント編集ログ）からクロスチェック、(ii) 不一致は **構造化アラート**（情報源 A：30 分、情報源 B：45 分、矛盾）、(iii) **confidence band** を表示し弁護士判断を仰ぐ、(iv) 弁護士が確定するまで請求書には反映されない、(v) すべての推定 + 確定の差分を WORM、長期トレンドで継続改善 / **Cross-source check + attorney sign-off**: (i) cross-check estimates against **multiple sources** (calendar / email / doc-edit log), (ii) discrepancies as **structured alerts** (source A: 30m vs source B: 45m, conflict), (iii) **confidence band** shown for attorney judgment, (iv) no invoice line until attorney confirms, (v) WORM-log estimate vs final diffs; iterate
- B) Claude が請求書を直接発行 / Claude issues invoices directly
- C) 推定は不要 / No estimation
- D) 平均値で請求 / Average-bill

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

タイムシートは **クロスチェック + 矛盾検出 + 弁護士確定 + 継続改善**。

Timesheets = **cross-check + conflict detection + attorney sign-off + iterate**.

- **B 不正解**: 過剰請求 / 倫理違反。 / Over-billing / ethical issue.
- **C 不正解**: 業務価値喪失。 / Loses value.
- **D 不正解**: 過剰または過少請求。 / Over / under-billing.

**参照 / Reference:** Legal billing
</details>

---

## 問題 43 / Question 43

**シナリオ / Scenario:**

国際的なリーガルテックで、複数法域（米・EU・日・中）の規制を Claude で扱う際、各法域の **個人情報保護法** が異なります。GDPR / CCPA / 個人情報保護法 / 個人情報保護法（中国 PIPL）など。

International legaltech operates across US / EU / Japan / China; per-jurisdiction privacy laws differ (GDPR / CCPA / Japan APPI / China PIPL).

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **法域認識データ管理 + クロスボーダー転送制限**：(i) すべてのデータに **データオリジン**（顧客の所在国）と **適用法**タグ、(ii) クロスボーダー転送（例：EU→US）は **明示同意 + SCC + 適切な保護措置**、(iii) Claude API のリージョン選択も **データ residency** に対応、(iv) 削除権（GDPR / CCPA / APPI）は **法域別の SLA**（GDPR 30 日・CCPA 45 日等）で実装、(v) 監査証跡は法域別に分離保管 / **Jurisdiction-aware data + cross-border transfer controls**: (i) tag every datum with **origin (customer country)** and **applicable law**, (ii) cross-border transfers (EU→US) require **explicit consent + SCC + safeguards**, (iii) Claude API region selection respects **data residency**, (iv) erasure rights (GDPR / CCPA / APPI) per-jurisdiction SLA (GDPR 30d, CCPA 45d), (v) audit trails kept per jurisdiction
- B) 全顧客に一律 GDPR 適用 / Apply GDPR to all
- C) 個人情報保護は無視 / Ignore privacy laws
- D) 法域ごとに別システム / Separate systems per jurisdiction

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

国際リーガルテックは **法域タグ + 越境移転制御 + リージョン選択 + 法域別 SLA**。

International legaltech = **jurisdiction tags + cross-border controls + region selection + per-jurisdiction SLA**.

- **B 不正解**: 過剰だが残り法域は不適合。 / Over-applies + still non-compliant.
- **C 不正解**: 違法。 / Illegal.
- **D 不正解**: 重複コスト。 / Cost duplication.

**参照 / Reference:** GDPR / CCPA / APPI / PIPL
</details>

---

## 問題 44 / Question 44

**シナリオ / Scenario:**

刑事弁護で、**証拠開示（discovery）** を Claude が分析。**無罪を支持する証拠**（exculpatory evidence）を見落とすと弁護過誤・倫理違反。

Criminal defense uses Claude for discovery. Missing **exculpatory evidence** is malpractice + ethical violation.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **多面探索 + 高 recall + 弁護士最終確認**：(i) 1 回の検索ではなく **複数仮説**（被告の不在 / 動機の欠如 / 識別の問題 / 証拠の信用性 / etc.）に基づく **多面検索**、(ii) 各仮説で **high recall** 優先（false negative 最小化）、(iii) 候補証拠は **すべて構造化サマリ**（証拠 / 出所 / 関連仮説 / 強度）、(iv) **弁護士が一つ一つ確認**、(v) Claude は提案役、最終判断は弁護士、(vi) 倫理規則に基づき重大な見落としは **WORM 保管 + 内部レビュー** / **Multi-perspective search + high recall + attorney review**: (i) **multi-perspective queries** (alibi / motive absence / ID issues / evidence credibility / etc.), (ii) prefer **high recall** per perspective (minimize FN), (iii) candidates as **structured summaries** (evidence / source / hypothesis / strength), (iv) **attorney reviews each**, (v) Claude proposes; attorney decides, (vi) per ethics rules, significant misses → **WORM + internal review**
- B) 1 つの検索で済ます / Single search
- C) Claude が無罪証拠を直接判定 / Claude decides exculpatory directly
- D) Discovery は手動のみ / Manual only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

刑事弁護は **多面探索 + 高 recall + 弁護士確定 + 倫理ログ**。

Criminal defense = **multi-perspective + high recall + attorney sign-off + ethics logging**.

- **B 不正解**: 見落としリスク。 / Miss risk.
- **C 不正解**: 弁護過誤、UPL。 / Malpractice / UPL.
- **D 不正解**: 効率損失。 / Inefficient.

**参照 / Reference:** Brady・exculpatory evidence
</details>

---

## 問題 45 / Question 45

**シナリオ / Scenario:**

法律事務所のクラウド移行で、**機密性の高いマター情報**を扱う Claude エージェントを **Amazon Bedrock 経由** または **Google Vertex 経由** で運用検討中。両クラウドで **データ residency** と **VPC 連携**が異なります。

A law firm migrating to cloud considers running confidentiality-sensitive Claude agents via **Bedrock** or **Vertex**; data residency and VPC integration differ.

**設問 / Question:**

最も適切な比較・選択基準はどれですか？ / Best comparison criteria?

- A) **法律事務所固有の評価軸**：(i) **データ residency**（クライアントの法域要件と一致）、(ii) **データ非保持契約**（Bedrock / Vertex の Anthropic との契約条件確認）、(iii) **VPC 連携**（ファイアウォール内での通信）、(iv) **既存セキュリティ認証**（SOC 2 / ISO 27001）、(v) **マター別の AWS / GCP プロジェクト分離**、(vi) コスト・レイテンシ・モデル可用性。すべての評価を **マター別の倫理リスク + 監査適合性** で総合判定 / **Law-firm-specific axes**: (i) **data residency** (matches client jurisdictional requirements), (ii) **no-retention agreements** (verify Bedrock / Vertex's Anthropic terms), (iii) **VPC integration** (in-firewall comms), (iv) **certifications** (SOC 2 / ISO 27001), (v) **per-matter project isolation** (AWS / GCP), (vi) cost / latency / model availability. Decide based on **per-matter ethical risk + audit fit**
- B) 安いほうを選ぶ / Pick the cheaper
- C) クラウド使わない / On-prem only
- D) Anthropic 直接のみ / Anthropic API only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

法律事務所のクラウド選定は **residency + 非保持 + VPC + 認証 + マター分離 + 倫理**で多軸評価。

Law-firm cloud selection = **residency + retention + VPC + certifications + matter isolation + ethics** — multi-axis.

- **B 不正解**: 倫理 / 規制不適合リスク。 / Risk.
- **C 不正解**: 過剰、コスト爆発。 / Overkill.
- **D 不正解**: 機能 / 統合不足。 / Lacks integration.

**参照 / Reference:** Cloud selection for legal
</details>

---

## 問題 46 / Question 46

**シナリオ / Scenario:**

工場の **24x7 生産ライン**で、生産制御エージェントの **異常検知 SLA は 99.99% で 5 秒以内**。Anthropic API の応答が遅い場合のフォールバックが必要です。

A 24x7 production line's anomaly-detection agent needs 99.99% within-5-seconds SLA; need fallback when Anthropic API is slow.

**設問 / Question:**

最も適切なフォールバックはどれですか？ / Best fallback?

- A) **多段フォールバック**：(i) **第 1 段** Claude API（通常パス、100ms 以内に応答）、(ii) **第 2 段** Bedrock 経由 Claude（地域フェイルオーバー）、(iii) **第 3 段** 軽量モデル / ML パイプライン（事前訓練）、(iv) **第 4 段** ルールベースシステム（最低限の安全動作）、(v) サーキットブレーカーで段階移行、(vi) 各段階の判定を **WORM 監査ログ**に記録、(vii) フォールバック発動率 / 復旧時間を **SLO 監視** / **Multi-tier fallback**: (i) Tier 1 Claude API (normal, <100ms), (ii) Tier 2 Bedrock Claude (regional failover), (iii) Tier 3 lightweight model / ML pipeline (pre-trained), (iv) Tier 4 rule-based (minimum-safe behavior), (v) circuit breaker advances tiers, (vi) WORM-log per-tier decisions, (vii) **SLO-monitor** fallback rate / recovery time
- B) Claude のみで十分 / Claude alone suffices
- C) フォールバックなし / No fallback
- D) ライン停止 / Halt the line

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

製造業の高 SLA は **多段フォールバック + サーキットブレーカー + 監視**。

Manufacturing high-SLA = **multi-tier fallback + CB + monitoring**.

- **B 不正解**: 単一障害で停止。 / SPOF.
- **C 不正解**: 99.99% 不可。 / Can't meet SLA.
- **D 不正解**: ビジネスインパクト過大。 / Costs too much.

**参照 / Reference:** Manufacturing SLO・multi-tier fallback
</details>

---

## 問題 47 / Question 47

**シナリオ / Scenario:**

サプライチェーンで **港湾ストライキ・地震・洪水**などの突発イベント時、即座に複数サプライヤーを **代替評価**する必要があります。情報源は混乱しており品質も悪い。

On disruptions (port strikes / quakes / floods), supply chain agents must rapidly **evaluate alternative suppliers** with chaotic, low-quality info.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **情報源信頼度 + 多源クロスチェック + 確度マーキング**：(i) 各情報源（ニュース・公式発表・現地担当者・サードパーティ監視サービス）に **事前の信頼度スコア**、(ii) 同じイベントが **複数源で確認**された場合のみ高確度、(iii) 矛盾は **明示的に提示**（"NYT は 3 日停止、現地は 1 週間"）、(iv) **時間経過で更新**（情報の鮮度減衰）、(v) Claude の判定はすべて **意思決定者向けに信頼度バンド付き**、(vi) すべて WORM ログ / **Source confidence + multi-source check + confidence marking**: (i) **prior confidence** per source (news / officials / on-the-ground / 3rd-party monitoring), (ii) high confidence only when **confirmed across sources**, (iii) **explicitly surface conflicts** ("NYT: 3 days; local: 1 week"), (iv) **freshness decay** with time, (v) all judgments come with **confidence bands** for decision-makers, (vi) WORM-log all
- B) 最初に見つけた情報を採用 / Use first source found
- C) すべての情報を等しく扱う / Treat all sources equally
- D) 情報源は 1 つだけ / Single source only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

危機時情報処理は **信頼度 + クロスチェック + 矛盾提示 + 鮮度 + 信頼度バンド**。

Crisis info = **confidence + cross-check + conflict surfacing + freshness + bands**.

- **B 不正解**: 誤情報リスク。 / Misinfo risk.
- **C 不正解**: 質の悪い情報に流される。 / Pulled by noise.
- **D 不正解**: 情報不足。 / Under-informed.

**参照 / Reference:** Crisis info management
</details>

---

## 問題 48 / Question 48

**シナリオ / Scenario:**

**化学プラント運転**で Claude を補助監視に。**プラント運転は数十年連続稼働**し、設備の経年変化・運転パラメータの drift が起きる。Claude のモデルやプロンプトも数年ごとに更新される。

A chemical plant runs continuously for decades; equipment ages, parameters drift; Claude models / prompts update every few years.

**設問 / Question:**

最も適切な長期運用はどれですか？ / Best long-term ops?

- A) **モデルバージョン記録 + 設備状態履歴 + 定期再校正**：(i) すべての判定に Claude モデル ID + プロンプト版 + 設備状態スナップショット、(ii) **定期校正サイクル**（年次）でセンサーキャリブレーション + モデル再評価、(iii) 経年劣化を考慮した動的閾値、(iv) **長期トレンド分析**で運転改善、(v) 重大設備変更時は再ベースライン、(vi) すべての履歴は **プラント寿命** までの保管 / **Model version + equipment-state history + periodic recalibration**: (i) every decision tagged with Claude model ID + prompt version + equipment-state snapshot, (ii) **annual calibration cycle** (sensor calibration + model re-eval), (iii) dynamic thresholds accounting for aging, (iv) **long-term trend analysis** for ops improvement, (v) major equipment changes → re-baseline, (vi) retention through **plant lifetime**
- B) モデル更新は無視 / Ignore model updates
- C) 設備の経年は無視 / Ignore aging
- D) Claude を使わない / Don't use Claude

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

長期プラント運用は **モデル版 + 設備状態 + 定期校正 + トレンド + 寿命保管**。

Long-term plant ops = **model version + equipment state + periodic recalibration + trends + lifetime retention**.

- **B 不正解**: drift で精度低下。 / Drift.
- **C 不正解**: 安全性低下。 / Safety degradation.
- **D 不正解**: 価値喪失。 / Lost value.

**参照 / Reference:** Plant long-term ops
</details>

---

## 問題 49 / Question 49

**シナリオ / Scenario:**

**多国籍製造企業のグローバル品質管理**で、世界 30 工場のデータを Claude で集約。**各工場の業務文化・言語・規制要件が異なる**ため、グローバル指標の比較が難しい。

Global QC across 30 factories: cultures / languages / regulations vary, making global metric comparison hard.

**設問 / Question:**

最も適切な設計はどれですか？ / Best design?

- A) **正規化 + 文脈保持 + 多言語対応**：(i) 各工場のローカルデータを **共通スキーマに正規化**（単位・分類・命名）、(ii) **元の言語と訳された英語を併記**（誤訳のリスク管理）、(iii) **ローカル規制文脈**は別フィールドで保持（インド工場の安全規制 vs ドイツ工場 etc.）、(iv) ローカル工場のフィードバックループで正規化精度改善、(v) グローバルダッシュボードでは正規化値で比較、必要時にローカル文脈に展開可能、(vi) 言語ローカリゼーションされたアラート / **Normalization + context preservation + multilingual**: (i) normalize local data into a **common schema** (units / categories / names), (ii) **bilingual original + English** (manage translation risk), (iii) **local regulatory context** in separate fields (e.g., India safety regs vs Germany), (iv) feedback loop with local plants improves normalization, (v) global dashboards compare on normalized values; drill-down to local context, (vi) localized alerts
- B) 各工場 1 言語のみ / One language per plant
- C) 正規化なしで集約 / Aggregate without normalization
- D) グローバル比較しない / No global comparison

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

グローバル QC は **正規化 + バイリンガル + 文脈保持 + フィードバック + ローカリゼーション**。

Global QC = **normalization + bilingual + context preservation + feedback + localization**.

- **B 不正解**: 言語バリアで分析不能。 / Lang barrier.
- **C 不正解**: 比較精度ゼロ。 / Incomparable.
- **D 不正解**: グローバル可視性喪失。 / Loses visibility.

**参照 / Reference:** Global QC・i18n
</details>

---

## 問題 50 / Question 50

**シナリオ / Scenario:**

最後の問題。あなたは大手製造業の CTO として、**全社の AI / Claude 活用戦略**を 5 年計画で策定しました。実装は順調ですが、**継続的な価値創出と倫理 / 規制遵守の両立**が課題です。

You are a CTO at a major manufacturer with a 5-year AI / Claude strategy. Implementation goes well, but **sustaining value creation while maintaining ethics / compliance** is the challenge.

**設問 / Question:**

最も適切な長期運用フレームワークはどれですか？ / Best long-term framework?

- A) **AI 統治フレームワーク**：(i) **AI 利用の責任部門**（CDO / CTO 直下）と **倫理委員会**（社外有識者含む）、(ii) **AI ユースケース登録制**（業務領域・データ種別・リスク評価・利用モデル / 提供者）、(iii) **定期的なアルゴリズム監査**（外部監査人）、(iv) **継続教育**（全社員に AI リテラシー、エンジニアに技術深化）、(v) **インシデント対応プロセス**（AI 起因の事故への対応）、(vi) **規制対応（EU AI Act 等）への先回り体制**、(vii) **取締役会レベルでの AI 監督** / **AI governance framework**: (i) **AI accountability** (CDO/CTO + ethics board with external experts), (ii) **AI use-case registry** (domain / data class / risk / model / provider), (iii) **periodic algorithmic audit** (external auditors), (iv) **continuous education** (firmwide AI literacy + engineering depth), (v) **AI incident response**, (vi) **proactive regulatory tracking** (EU AI Act etc.), (vii) **board-level AI oversight**
- B) AI 戦略は技術部門だけで完結 / Tech dept handles AI alone
- C) 規制対応は受動的 / Reactive compliance
- D) インシデント対応は事後 / Post-incident response only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

エンタープライズ AI 戦略は **責任部門 + 倫理委員会 + ユースケース登録 + 監査 + 教育 + インシデント対応 + 規制対応 + 取締役会監督**。

Enterprise AI = **accountability + ethics board + use-case registry + audit + education + incident response + regulatory + board oversight**.

- **B 不正解**: 部門サイロ化、ガバナンス不全。 / Silos, governance failure.
- **C 不正解**: 規制違反リスク。 / Compliance risk.
- **D 不正解**: 大事故誘発。 / Major-incident risk.

**参照 / Reference:** AI governance・EU AI Act
</details>

---

> **前のドメイン / Previous:** [`domain4_prompt_structured_output.md`](./domain4_prompt_structured_output.md) | **目次 / Index:** [`README.md`](./README.md)
