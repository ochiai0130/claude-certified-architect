# Domain 4: プロンプトエンジニアリングと構造化出力 / Prompt Engineering and Structured Output

> 配点比率 / Weight: **20%**
> 問題数 / Questions: **5**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- 明示的基準のプロンプト設計 / Explicit-criteria prompt design
- Few-shot 例の選定（数とエッジケース偏重） / Few-shot example selection (count + edge-case skew)
- `tool_use` JSON Schema による意味エラー検出 / Semantic error detection via JSON Schema
- Message Batches API の適用条件 / Message Batches API applicability
- 検証 → 再試行 → 人間エスカレーションのループ設計 / Validate-retry-escalate loop design

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

EU 投資銀行の MiFID II 対応で、デリバティブ契約書から 12 フィールドの構造化抽出を行います（想定元本、決済日、参照金利、CSA 条項の有無 など）。現行プロンプトは「重要条項を抽出してください」という曖昧な指示で、抽出精度が 72% に留まっています。最重要要件は **「想定元本の通貨」が誤ると規制報告（EMIR Trade Repository）の自動再提出ルートに乗らないため、致命的**であることです。

For an EU investment bank's MiFID II program, you extract 12 structured fields from derivative contracts (notional, settlement date, reference rate, CSA presence, etc.). The current prompt — "extract important clauses" — yields 72% accuracy. Most critical: a wrong **notional currency** prevents automated EMIR Trade Repository re-submission and is a regulatory severity-1 incident.

**設問 / Question:**

最も効果的なプロンプト改善はどれですか？

Which prompt improvement is most effective?

- A) 12 フィールドを **明示的基準** で定義する：各フィールドの (i) 厳密な意味定義、(ii) 抽出元（条項見出し・括弧書き等）の場所優先順位、(iii) 出力フォーマット（ISO 4217 通貨コード、ISO 8601 日付）、(iv) 不在時の値（`null` or `"NOT_FOUND"`）、(v) 矛盾時の優先ルール（例：本文 > スケジュール > 注釈）。さらに `tool_use` JSON Schema で `currency` を `enum: ["USD","EUR","JPY",...]` で強制 / Define each of the 12 fields with **explicit criteria**: (i) precise semantic definition, (ii) extraction source priority (clause heading, parenthetical, etc.), (iii) output format (ISO 4217 currency, ISO 8601 date), (iv) value when absent (`null` / `"NOT_FOUND"`), (v) conflict resolution rules (body > schedule > footnote). Constrain `currency` via JSON Schema `enum`
- B) 「もっと正確に抽出してください、特に通貨に注意」と追記する / Append "extract more accurately, especially currency"
- C) 温度を 0 にして安定性を上げる / Lower temperature to 0
- D) 12 フィールドを 1 つずつ別の API 呼び出しで処理する / Process each of the 12 fields in separate API calls

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

抽出精度が 72% で止まる主因は **基準の曖昧さ** です。「重要条項」が文書ごとに揺れる、抽出元の優先順位が決まっていない、不在値の表現が定まっていない — これらを **明示的基準**として定義すると、モデルは判断の自由度を失い精度が劇的に向上します。さらに `enum` でクローズドな値域を強制すれば、誤った通貨コードは API レイヤーで弾かれ、規制報告失敗の根を絶てます。これは Anthropic 公式の "explicit criteria > vague instructions" の原則そのものです。

72% accuracy stalls due to **ambiguous criteria**. "Important clause" varies per document, source priority is undefined, and missing-value handling is inconsistent. **Explicit criteria** remove model discretion, sharply boosting accuracy. Adding `enum` constraints in JSON Schema kills wrong currency codes at the API layer.

- **B 不正解**: 「もっと正確に」は意図伝達であり基準にならず、効果は限定的。 / "More accurate" is a wish, not a criterion.
- **C 不正解**: 温度低下は揺れを抑えるが、**間違った答えに収束**することもある。基準の曖昧さは解消しない。 / Low temperature can converge confidently to wrong answers.
- **D 不正解**: 12 並列呼び出しは可能だが、**フィールド間整合性（決済日と支払日の関係等）が壊れやすい**ため逆効果になりがち。 / Splitting calls breaks cross-field consistency (e.g., settlement vs payment date).

**参照 / Reference:** `guide_ja.md` 「5.1 明示的基準」「5.4 構造化出力（JSON Schema）」、Anthropic prompt-engineering best practices
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

医療機関の臨床ノート（自由記述）から **「アレルギー」「現病歴」「処方薬」** を抽出するエージェントを開発中です。テスト集合 200 件のうち、**多数派の典型的なノート（160 件）では精度 94%** ですが、**残り 40 件のエッジケース**（2 つのアレルギーをスラッシュ区切り、薬剤名の略称、否定形「ペニシリンアレルギー**なし**」）で精度 51% に落ちます。Few-shot を追加して改善したい。

You extract `allergies`, `present_illness`, `medications` from clinical free-text notes. Out of 200 test samples, 160 typical notes hit 94% accuracy; the remaining 40 edge cases (slash-delimited multi-allergies, abbreviated drug names, negations like "**no** penicillin allergy") drop to 51%.

**設問 / Question:**

最も効果的な Few-shot 戦略はどれですか？

Which is the most effective few-shot strategy?

- A) 多数派の典型的なノートから 8 例を Few-shot に追加 / Add 8 typical notes as few-shot examples
- B) Few-shot を **2〜4 例**にとどめ、内容は **エッジケース偏重**にする：①スラッシュ区切り複数アレルギー、②略称薬剤、③否定形（「なし」）、④（任意で）多数派の典型 1 例。各例は入力 → 期待出力（JSON）と「なぜこの出力なのか」の短い理由を併記 / Keep to **2–4 examples**, **skewed toward edge cases**: ①slash-delimited multi-allergy, ②abbreviated drug, ③negation ("none"), ④(optionally) one typical case. Each example pairs input → expected JSON with a short rationale
- C) Few-shot を 30 例まで増やしてカバレッジを上げる / Increase to 30 examples to maximize coverage
- D) Few-shot ではなく、`claude-opus-4-6` に切り替える / Skip few-shot; switch to `claude-opus-4-6`

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

Few-shot の正解は **数より失敗モードへの偏重**です。多数派ですでに 94% 出ている領域に例を追加しても限界効用は低く、コスト（コンテキスト・レイテンシ）だけ上がります。**失敗モード（エッジケース）に偏らせた 2〜4 例**で、モデルに「これも対象範囲だ」と境界を学ばせるのが最も効率的。短い理由併記により否定形の扱いなど暗黙基準を顕在化できます。Anthropic の公式ガイドラインでも 2〜4 例が標準で、それ以上は **diminishing returns** が顕著です。

The right approach is **skewed selection over volume**. Adding more typical-case examples to a domain already at 94% adds cost without benefit. **2–4 edge-case-skewed examples** teach the model the boundary cases. Brief rationales surface implicit criteria (e.g., negation handling). 2–4 is Anthropic's standard; beyond that yields diminishing returns.

- **A 不正解**: 多数派偏重は既に出来ている領域を強化するだけで、エッジケースは改善しません。 / Reinforces the strong area; weakest area remains weak.
- **C 不正解**: 30 例はコンテキストコストとレイテンシが急上昇し、効果は頭打ち。Anthropic ガイドラインから外れます。 / 30 examples explode cost with little gain.
- **D 不正解**: モデル変更で改善しても、明示的なエッジケース提示の方が再現性高く安価に効きます。 / Model upgrades are less reproducible and more expensive than targeted few-shot.

**参照 / Reference:** `guide_ja.md` 「5.2 Few-shot プロンプティング」、Anthropic Prompt Engineering Guide
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

医療保険会社で、**保険金支払い判定**（承認 / 一部承認 / 拒否 / 要審査）を Claude が出力する `tool_use` ツール `decide_claim` を実装中。スキーマには `decision` (enum)、`amount` (number)、`reason_codes` (array of enum)、`reviewer_handoff` (boolean) が含まれます。最近の本番データで、`decision = "approved"` なのに `amount = 0` という **意味的に矛盾する出力**が稀に発生し、下流の支払いシステムが落ちる事故がありました。

A health insurer implements a `decide_claim` `tool_use` tool with `decision` (enum), `amount` (number), `reason_codes` (array of enum), and `reviewer_handoff` (boolean). Production data revealed rare **semantically inconsistent outputs** like `decision="approved"` with `amount=0`, crashing the downstream payment system.

**設問 / Question:**

最も適切な防御策はどれですか？

Which defense is most appropriate?

- A) `tool_use` の JSON Schema を厳密化（`amount` を `minimum: 0` に）するだけで十分 / Tighten the JSON Schema (e.g., `amount: { minimum: 0 }`) — that's enough
- B) Claude に毎回「この出力は矛盾していないか？」と自己検証させる / Have Claude self-check "is this output inconsistent?" each time
- C) システムプロンプトに「approved なら amount は 0 より大きく」と注意を書く / Add a system prompt note: "if approved, amount must be > 0"
- D) JSON Schema の構文検証に加え、**意味検証層**を実装：`decision="approved"` なら `amount > 0` 必須、`decision="rejected"` なら `amount = 0` 必須、`reviewer_handoff=true` なら `decision="manual_review"` 必須、等のルールエンジンを通す。違反時は **構造化エラーを Claude に返して再試行**（フィードバック付き）し、N 回失敗で人間にエスカレーション / Beyond JSON Schema **syntax** validation, run a **semantic validation** layer: if `decision="approved"` then `amount > 0`; if `decision="rejected"` then `amount = 0`; if `reviewer_handoff=true` then `decision="manual_review"`; etc. On violation, **return structured feedback to Claude and retry**; after N failures, escalate to a human

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

JSON Schema は **構文（型・enum・range）** は強制できますが、**意味的整合性（フィールド間の論理関係）** は表現しきれません。これは外部の **意味検証層** で検査し、違反時は **エラー内容をフィードバックとして Claude に渡し、修正版を再生成させる**のが正しい設計です（"validate → retry with feedback" ループ）。N 回失敗で人間にエスカレーションすることでコスト爆発と無限ループを防ぎます。

JSON Schema enforces **syntax** (types, enums, ranges) but cannot express **inter-field semantic constraints**. A separate **semantic validation layer** must enforce these and feed violations back to Claude as structured feedback for retry ("validate → retry with feedback" loop). Cap retries to prevent runaway loops; escalate to humans on persistent failure.

- **A 不正解**: スキーマだけでは `decision="approved"` AND `amount=0` を防げません（両方とも単体では妥当）。 / Schema alone allows the inconsistent combination since each field is individually valid.
- **B 不正解**: 自己検証は同じバイアスで見逃すことが多く、外部の決定論的検証が必要。 / Self-check often misses with the same bias; external deterministic validation is needed.
- **C 不正解**: プロンプト指示は確率的で、本番事故率を許容範囲まで下げられません。 / Prompts cannot reach the required precision.

**参照 / Reference:** `guide_ja.md` 「2.5 構文エラーと意味エラー」「5.5 検証 → 再試行 → エスカレーション」
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

5,000 万件の過去問い合わせメールを **業務カテゴリ（10 種類）に分類** し直すマイグレーションが必要です。要件は次のとおり：

You must reclassify 50M historical support emails into 10 business categories. Requirements:

- 結果は 24 時間以内に揃えばよい / Results within 24 hours is acceptable
- 単発のバッチで、リアルタイム応答は不要 / One-off batch; no real-time response
- マルチターン対話やツール使用は不要（プロンプト → 1 回のレスポンスで完結） / No multi-turn or tool use needed (prompt → single response)
- コスト最適化が最優先 / Cost optimization is the top priority

別途、**カスタマーサポート Live エージェント**（リアルタイム応答・ツール使用あり）も構築中です。

You're also building a **live support agent** that requires real-time responses and tool use.

**設問 / Question:**

最も適切な API 選択はどれですか？

Which API choice is most appropriate?

- A) 両方とも通常の Messages API を使う / Both via the regular Messages API
- B) 両方とも Message Batches API を使う / Both via Message Batches API
- C) マイグレーションは **Message Batches API**（最大 24 時間ウィンドウ・約 50% コスト削減・マルチターンやツール使用は非対応）、Live エージェントは Messages API + Streaming / Migration via **Message Batches API** (up to 24-hour window, ~50% cost reduction, no multi-turn or tool use); Live agent via Messages API + streaming
- D) マイグレーションは Messages API（リアルタイム性のため）、Live エージェントは Message Batches API（低コストのため） / Migration via Messages API (for real-time), Live agent via Message Batches API (for cost)

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

Message Batches API は **(1) 最大 24 時間の処理ウィンドウ、(2) 約 50% のコスト削減、(3) マルチターン会話・ツール使用は非対応、(4) 単発の独立リクエストに最適** という特性があります。5,000 万件の単発分類はこの条件にぴったり合致。一方、Live エージェントはリアルタイム応答とツール使用が必須なので Batches API は使えず、Messages API + Streaming が正解。

Message Batches API offers **24-hour window, ~50% cost discount, no multi-turn or tool use**, ideal for one-off independent requests. The 50M classification job fits perfectly. The live agent needs real-time + tool use, so it cannot use Batches.

- **A 不正解**: マイグレーションを Messages API で流すと約 2 倍のコストになり、最適化要件を満たしません。 / Doubles cost for the migration unnecessarily.
- **B 不正解**: Live エージェントは Batches API のレイテンシ（最大 24 時間）に耐えられず、ツール使用もできません。 / Live agent cannot tolerate batch latency or lack of tool use.
- **D 不正解**: 完全に逆。要件適合性が反転しています。 / Inverted.

**参照 / Reference:** `guide_ja.md` 「5.6 Message Batches API」、Anthropic Batches API docs
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

金融機関の **AML（マネーロンダリング検知）アラート要約** を Claude が生成し、調査担当者にエスカレーションするシステムを設計中です。要件：

You're designing an AML alert-summarization system that escalates to investigators. Requirements:

- 出力に**捏造（hallucination）**があると規制リスク / Hallucinations carry regulatory risk
- 自信度が低いケースは**人間レビュー必須** / Low-confidence cases must trigger human review
- 全件の入出力・推論ステップ・リトライ履歴を **監査ログ**に保存 / All input/output, reasoning steps, retries must go to an audit log
- 調査担当者の負荷を抑えるため、**過剰な人間エスカレーションは避ける** / Avoid over-escalation to keep investigator load reasonable

**設問 / Question:**

最も適切なループ設計はどれですか？

Which loop design is most appropriate?

- A) 1 回生成して即座に人間に渡す（自信度判定なし） / Generate once and immediately route to humans (no confidence gating)
- B) **生成 → 検証（事実根拠の出所マッチング、構造化スキーマ、自信度算出）→ 自信度が閾値未満なら最大 N 回まで「具体的な不足点」をフィードバックして再生成 → 依然不十分なら人間エスカレーション**。すべての試行を相関 ID 付きで監査ログに保存。閾値は本番データのキャリブレーションで決定（過剰エスカレーション抑制） / **Generate → validate (source grounding, schema, confidence) → if confidence < threshold, feed back specific deficiencies and regenerate up to N times → escalate to humans if still insufficient**. Persist all attempts with correlation IDs in the audit log. Tune the threshold via calibration to balance escalation volume
- C) 自信度が低ければ温度を上げて再試行する / On low confidence, raise temperature and retry
- D) すべての出力を必ず人間が確認する（ループなし） / Always have a human review every output (no loop)

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

ビジネスクリティカル領域での信頼性ループは **生成 → 検証 → 構造化フィードバック付き再試行 → 人間エスカレーション + 監査ログ** の組み合わせが基本形です。重要なのは「具体的な不足点をフィードバック」すること（「もう一度頑張れ」では効果が出ない）と、**自信度の閾値を実データでキャリブレーション**することです（業務負荷と精度のトレードオフ）。すべての試行を相関 ID 付きで監査ログに保存することで、コンプライアンス調査・モデル改善・障害分析の三役を兼ねます。

Business-critical reliability loops follow **generate → validate → retry-with-specific-feedback → human-escalate + full audit log**. Two keys: feed back **specific deficiencies** (not "try again"), and **calibrate the confidence threshold** on production data to balance accuracy and investigator load.

- **A 不正解**: 自信度判定なしでは投資家負荷が爆発し、運用破綻。 / Without gating, escalation volume becomes unmanageable.
- **C 不正解**: 温度を上げると揺れが増えて精度が下がります。低自信度の答えは温度では救えません。 / Higher temperature increases variance, not accuracy.
- **D 不正解**: 全件人間確認は規制要件以上の負荷で持続不能。Claude を使う意味が薄い。 / 100% human review is unsustainable and defeats automation.

**参照 / Reference:** `guide_ja.md` 「5.5 検証・再試行・フィードバックループ」「7.3 信頼度キャリブレーション」「監査ログ」
</details>

---

> **前のドメイン / Previous:** [`domain3_claude_code_workflows.md`](./domain3_claude_code_workflows.md) | **次のドメイン / Next:** [`domain5_context_reliability.md`](./domain5_context_reliability.md)
