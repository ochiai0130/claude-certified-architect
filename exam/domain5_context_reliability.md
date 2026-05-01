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

> **前のドメイン / Previous:** [`domain4_prompt_structured_output.md`](./domain4_prompt_structured_output.md) | **目次 / Index:** [`README.md`](./README.md)
