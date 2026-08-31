# Domain 5: モデル選定と最適化 / Model Selection and Optimization

> 配点比率 / Weight: **16.8%**（全 8 ドメイン中 2 番目）
> 問題数 / Questions: **40**（基礎 27 / 発展 13）
> 形式 / Format: 単一選択（選択肢 A–D）＋複数選択（選択肢 A–E、選ぶ数を明示）

## サブスキル / Sub-skills

| サブスキル / Sub-skill | Weight | 本ファイル |
|---|---|---|
| LLM Fundamentals | 5.2% | 12 |
| Technical Fundamentals | 6.1% | 15 |
| Model Selection and Tradeoffs | 2.7% | 6 |
| Cost and Token Management | 2.8% | 7 |

## 出題範囲 / Scope（公式ガイドの記述に基づく）

- **LLM Fundamentals** — LLM の基礎（トークン、コンテキストウィンドウ、サンプリング、非決定性、次トークン生成）、モデルのオプション（fast mode、拡張思考、アダプティブ思考、effort の水準）、基本的なプロンプト手法（zero-shot、single-shot、multi-shot） / LLM basics, model options, and fundamental prompting techniques
- **Technical Fundamentals** — AI アプリケーション開発を支える基礎的な技術概念。REST API をラップする SDK との統合、websocket などの基本的なエンジニアリング実践 / Foundational technical concepts, including integrating with SDKs that wrap REST APIs, and websockets
- **Model Selection and Tradeoffs** — Claude の各モデルの能力（Opus / Sonnet / Haiku の用途、アダプティブ思考の対応）、品質・レイテンシ・コストのトレードオフ、モデルのリリースをまたぐ挙動変化 / Model capabilities, quality-latency-cost tradeoffs, and breaking behavior changes across releases
- **Cost and Token Management** — トークン予算とコスト管理（使用量の追跡、コストのモデリング、キャッシュ手法） / Token budgeting and cost management, including usage tracking, cost modeling, and caching techniques

---

## 基礎 / Foundations level

### 問題 1 / Question 1

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

コストを見積もるため、入力テキストのトークン数を推定する必要があります。開発者は「日本語も英語も、だいたい 4 文字で 1 トークン」という前提で計算し、実際の請求額との乖離が大きいことに気づきました。

To estimate cost, you need the token count of input text. A developer assumed roughly four characters per token for both Japanese and English, and found a large gap against the actual bill.

**設問 / Question:**

トークンについての正しい理解はどれですか？

Which is a correct understanding of tokens?

- A) 1 トークンは常に 1 文字に対応する / One token always corresponds to one character
- B) 1 トークンは常に 1 単語に対応する / One token always corresponds to one word
- C) **トークン化の結果は言語や内容によって変わるため、文字数からの一律の換算は正確でない。正確な値が必要なら、専用のトークン計測手段で数える** / **Tokenization varies by language and content, so a fixed characters-per-token ratio is not accurate; when the exact number matters, count it with a proper token-counting facility**
- D) トークン数は出力にのみ関係し、入力では課金されない / Tokens matter only for output; input is not billed

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

トークンは文字でも単語でもなく、**モデルが扱う単位**であり、その分割のされ方は言語や内容によって変わります。英語とアジア圏の言語では 1 文字あたりのトークン数が大きく異なり、コードや記号列でも変わります。したがって、文字数からの一律換算はコスト見積もりや上限判定の用途には不十分です。正確な値が必要な場面（コンテキスト上限に収まるかの判定、コスト試算）では、専用のトークン計測手段で数えるのが正しい方法です。

A token is neither a character nor a word but the unit the model operates on, and how text divides into them varies by language and content — the characters-per-token ratio differs substantially between English and Asian languages, and again for code and symbol-heavy text. A fixed ratio is therefore inadequate for cost estimation or limit checks; count with a real token-counting facility when the number matters.

- **A 不正解**: 1 文字 1 トークンではありません。 / Not one-to-one with characters.
- **B 不正解**: 1 単語 1 トークンでもありません。長い単語は複数に分割されます。 / Not one-to-one with words either.
- **D 不正解**: 入力トークンも課金対象です。むしろ多くの用途では入力側が支配的です。 / Input tokens are billed, and often dominate.

**参照 / Reference:** LLM Fundamentals — トークン
</details>

---

### 問題 2 / Question 2

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

テストで、まったく同じ入力を 2 回送ったところ、微妙に異なる応答が返ってきました。開発者は「バグではないか」と報告しています。

In testing, sending exactly the same input twice produced slightly different responses. A developer reports this as a possible bug.

**設問 / Question:**

正しい理解はどれですか？

Which is the correct understanding?

- A) **LLM の生成は本質的に非決定的で、同じ入力でも出力が完全に一致するとは限らない。これは不具合ではなく前提であり、アプリケーションはこれを踏まえて設計する必要がある** / **Generation is inherently nondeterministic: identical input does not guarantee identical output. This is a premise rather than a defect, and applications must be designed around it**
- B) 同じ入力なら必ず同じ出力になるはずなので、これは不具合である / Identical input must produce identical output, so this is a defect
- C) キャッシュが有効になっていないことが原因である / The cause is that caching is not enabled
- D) 出力が異なるのはネットワークの問題である / Differing output indicates a network problem

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**非決定性は LLM の前提であり、不具合ではありません。**この性質は設計に具体的な影響を与えます。テストは文字列の完全一致ではなく満たすべき性質で書く、監査要件は「再生成」ではなく「記録の完全性」で満たす、重要な判定はモデルの出力そのものではなく決定的な検証と組み合わせる、といった設計上の帰結が導かれます。「同じ入力なら同じ出力」という前提でシステムを組むと、本問のような報告が繰り返されます。

Nondeterminism is a premise of LLMs, not a defect, and it has concrete design consequences: write tests against properties rather than exact strings, satisfy audit requirements through record integrity rather than regeneration, and pair consequential decisions with deterministic verification. Building on an assumption of identical output guarantees a recurring stream of reports like this one.

- **B 不正解**: 完全一致は保証されません。 / Exact reproduction is not guaranteed.
- **C 不正解**: キャッシュは入力処理の再利用の仕組みで、出力を固定するものではありません。 / Caching reuses input processing; it does not fix the output.
- **D 不正解**: ネットワークは出力の内容に影響しません。 / The network does not alter the content.

**参照 / Reference:** LLM Fundamentals — 非決定性
</details>

---

### 問題 3 / Question 3

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

コンテキストウィンドウについて、チーム内で理解を揃えようとしています。

Your team is aligning its understanding of the context window.

**設問 / Question:**

コンテキストウィンドウについて正しい記述を **2 つ選択してください**。

Select **2** correct statements about the context window.

- A) コンテキストウィンドウは出力トークン数の上限を意味する / The context window is the limit on output tokens
- B) **コンテキストウィンドウは、入力（システムプロンプト、会話履歴、ツール定義、渡した文書など）と生成される出力を合わせた総量の上限である** / **The context window bounds input — system prompt, conversation history, tool definitions, attached documents — and generated output, together**
- C) ウィンドウが大きいモデルを使えば、常に品質が上がる / A model with a larger window always produces better quality
- D) **モデルによってコンテキストウィンドウの大きさは異なるため、必要な入力量に応じてモデルを選ぶ判断材料になる** / **Window size differs by model, so it is one input to model selection when the required input volume is large**
- E) 会話履歴はサーバー側に保持されるため、コンテキストウィンドウを消費しない / Conversation history is held server-side and does not consume the window

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, D**

**解説 / Explanation:**

コンテキストウィンドウは**入力と出力を合わせた総量の上限**であり、システムプロンプト、会話履歴、ツール定義、渡した文書のすべてがここに含まれます。大きさはモデルによって異なるため、扱う入力量が大きい用途ではモデル選定の判断材料になります。なお、ウィンドウが大きいことは「多く入れられる」という意味であって、多く入れれば品質が上がるという意味ではありません。無関係な情報を詰め込むと、むしろ関連情報が埋没します。

The window bounds input and output together, and everything counts: system prompt, conversation history, tool definitions, attached documents. Size varies by model, making it an input to model selection for input-heavy work. A larger window means more *can* fit, not that filling it improves quality — irrelevant material buries the relevant.

- **A 不正解**: 出力の上限は別のパラメータで指定します。 / The output ceiling is a separate parameter.
- **C 不正解**: ウィンドウの大きさは容量であり、品質を保証しません。 / Capacity, not quality.
- **E 不正解**: Messages API はステートレスで、履歴は毎回送るためウィンドウを消費します。 / The API is stateless; history is resent and consumes the window.

**参照 / Reference:** LLM Fundamentals — コンテキストウィンドウ
</details>

---

### 問題 4 / Question 4

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

社内の Q&A システムで、モデルが社内固有の製品仕様について自信を持って誤った回答をします。開発者は「モデルが学習していないなら、知らないと答えるはずだ」と考えていました。

In an internal Q&A system, the model confidently gives wrong answers about proprietary product specifications. A developer expected that "if the model hasn't learned it, it should say it doesn't know."

**設問 / Question:**

この挙動の正しい理解はどれですか？

Which is the correct understanding of this behavior?

- A) モデルは知らないことを必ず「知らない」と答える / The model always says it does not know
- B) 誤りはモデルの不具合である / The wrong answers are a defect in the model
- C) 温度を下げれば誤りはなくなる / Lowering the temperature eliminates the errors
- D) **生成は次に来るトークンを予測する処理であり、知識の有無を判定してから答える仕組みではない。したがって、根拠のない内容ももっともらしい形で生成され得る。社内固有の情報は、取得してコンテキストに入れることで対処する** / **Generation predicts the next token rather than first determining whether it knows something, so unsupported content can be produced in plausible form. Proprietary information is addressed by retrieving it into the context**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**生成は次トークンの予測であり、「知っているかを確認してから答える」処理ではありません。**このため、学習に含まれていない社内固有の情報についても、それらしい形の回答が生成され得ます。対処は 2 方向で、(1) 権威ある情報を取得してコンテキストに入れる、(2) 出典を付けられない主張を出力しない設計にする、です。「知らないなら知らないと答えるはず」という期待は、生成の仕組みに対する誤解に基づいています。

Generation predicts the next token; it does not first establish whether it knows. Proprietary information absent from training can therefore emerge in plausible form. Two remedies apply: retrieve the authoritative source into the context, and design the output so that assertions without a citation are not produced. Expecting an "I don't know" by default misunderstands the mechanism.

- **A 不正解**: 知識の有無を判定する機構が生成の中にあるわけではありません。 / There is no such determination step in generation.
- **B 不正解**: 仕組みに由来する性質であり、不具合として扱うのは誤りです。 / A property of the mechanism, not a defect.
- **C 不正解**: サンプリングの設定は、知らない情報を知っているようにする問題を解決しません。 / Sampling settings do not supply missing knowledge.

**参照 / Reference:** LLM Fundamentals — 次トークン生成
</details>

---

### 問題 5 / Question 5

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

社内の独自形式の伝票から項目を抽出するタスクで、指示だけを与えた場合（zero-shot）の精度が 71% でした。伝票の形式は社外には存在しない独自のもので、項目の位置や表記に一定の規則があります。

Extracting fields from a proprietary internal voucher format achieves 71% accuracy with instructions alone (zero-shot). The format exists nowhere outside the company, with consistent conventions for field positions and notation.

**設問 / Question:**

最も効果が期待できる改善はどれですか？

Which improvement is most likely to help?

- A) 指示をより長く詳細に書く / Write longer, more detailed instructions
- B) **実際の伝票と期待される抽出結果の対を few-shot 例として示す。独自形式のように言葉で説明しにくい規則は、例で示すほうが伝わりやすい** / **Provide few-shot examples pairing real vouchers with the expected extraction. Conventions that are hard to describe in words, as with a proprietary format, are conveyed more effectively by example**
- C) 温度を 0 にする / Set the temperature to zero
- D) より大きなコンテキストウィンドウのモデルに変更する / Switch to a model with a larger context window

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

few-shot 例が最も効くのは、**言葉で説明するより実例を示すほうが伝わる場合**です。社内独自の伝票形式は、項目の位置関係や表記の慣習を文章で網羅的に書くのが難しく、例を数件示すほうが確実に伝わります。なお、例の選定は重要で、簡単なケースばかり並べても学べることは少なく、**誤りが集中しているパターンや境界的な判断を含む例**を選ぶほうが効果が大きくなります。

Few-shot examples pay most where showing beats describing. A proprietary voucher format's field positions and notational conventions are hard to enumerate in prose and are conveyed reliably by a handful of examples. Which examples matters: a set of easy cases teaches little, while examples drawn from where errors concentrate and from genuinely borderline judgments carry far more.

- **A 不正解**: 独自形式の規則を文章で網羅するのは困難で、長さは解決になりません。 / Prose cannot readily enumerate the conventions.
- **C 不正解**: サンプリングの設定は形式の理解を助けません。 / Sampling settings do not convey the format.
- **D 不正解**: ウィンドウの大きさは今回の制約ではありません。 / Window size is not the constraint here.

**参照 / Reference:** LLM Fundamentals — zero-shot / few-shot
</details>

---

### 問題 6 / Question 6

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

複雑な推論を要するタスクと、単純な分類タスクの両方を同じモデルで処理しています。前者は品質が不足し、後者は必要以上にコストがかかっている状態です。

You run both a complex reasoning task and a simple classification task on the same model. The first falls short on quality; the second costs more than it needs to.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **タスクごとに effort の水準を使い分ける。複雑な推論には高い水準を、単純な分類には低い水準を割り当てる。effort は思考の深さと消費するトークンを調整する仕組みで、品質とコストのトレードオフを 1 つのモデルの中で動かせる** / **Use different effort levels per task: a higher level for the complex reasoning and a lower one for the simple classification. Effort tunes reasoning depth and token spend, letting you move along the quality-cost tradeoff within one model**
- B) 両方のタスクを 1 つのリクエストにまとめる / Combine both tasks into a single request
- C) `max_tokens` をタスクごとに変える / Vary `max_tokens` per task
- D) 温度をタスクごとに変える / Vary the temperature per task

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**effort の水準は、品質とコストのトレードオフを調整する第一の手段**です。複雑な推論には高い水準を割り当てて品質を確保し、単純な分類には低い水準で十分な品質をより安く得ます。同じモデルの中で調整できるため、モデルを分ける場合と違ってキャッシュの構成や運用が複雑になりません。なお、水準を上げ続けても精度はどこかで頭打ちになるため、評価で収穫逓減の点を特定してから設定するのが実務的です。

Effort is the first lever on the quality-cost tradeoff: a high level protects quality on hard reasoning, a low one gets adequate quality more cheaply on simple classification. Because it operates within one model, it avoids the cache and operational complexity of splitting across models. Accuracy plateaus as effort rises, so locate the knee empirically before fixing a level.

- **B 不正解**: 性質の異なるタスクを混ぜると、どちらの品質も安定しません。 / Mixing dissimilar tasks destabilizes both.
- **C 不正解**: `max_tokens` は出力の上限であり、推論の深さを調整する手段ではありません。 / An output ceiling, not a reasoning control.
- **D 不正解**: サンプリングの設定は、推論の深さとは別の概念です。 / Sampling is conceptually distinct from reasoning depth.

**参照 / Reference:** LLM Fundamentals — effort の水準、モデルのオプション
</details>

---

### 問題 7 / Question 7

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

複数のステップにわたる推論が必要なタスクで、モデルに十分に考えさせたいと考えています。開発者は、プロンプトに「順を追って考えてから答えてください」と書き、さらに思考のトークン数を数値で指定しようとしています。

For a task requiring multi-step reasoning, you want the model to think sufficiently. A developer writes "think step by step before answering" in the prompt and is also trying to specify a numeric thinking-token count.

**設問 / Question:**

現行モデルにおける適切な理解はどれですか？

Which is the appropriate understanding on current models?

- A) 思考のトークン数を数値で指定するのが標準的な方法である / Specifying a numeric thinking-token count is the standard approach
- B) プロンプトの指示だけで思考の量を制御できる / The prompt instruction alone controls how much the model thinks
- C) **アダプティブな思考が標準で、モデルが必要に応じて思考の量を決める。深さの調整は固定のトークン予算ではなく effort の水準で行う。プロンプトでの指示は補助的な役割にとどまる** / **Adaptive thinking is the norm: the model determines how much to think, and depth is tuned by effort level rather than a fixed token budget. A prompt instruction plays only a supporting role**
- D) 思考の機能は使わないほうが品質が安定する / Quality is more stable with thinking disabled

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

現行の Claude モデルでは**アダプティブな思考が標準**で、モデルがタスクに応じて思考の量を決めます。深さを調整したい場合は、固定のトークン予算を数値指定するのではなく **effort の水準**を使います。プロンプトでの「順を追って考えて」という指示は、出力の構成に影響を与えることはありますが、思考の量を制御する仕組みではありません。古い方式（固定の思考トークン予算）は現行モデルでは使えないため、要望はそのまま実装できず、effort による調整に翻訳する必要があります。

Current Claude models use adaptive thinking, with the model deciding how much to think per task; depth is tuned by effort level rather than a numeric budget. A "think step by step" instruction can shape the output's structure but is not a control over thinking volume. The older fixed-budget approach is unavailable on current models, so the request translates into effort tuning.

- **A 不正解**: 固定の思考トークン予算を指定する方式は、現行モデルでは使われません。 / Not how current models work.
- **B 不正解**: プロンプトの指示は思考量の制御機構ではありません。 / A prompt is not the control mechanism.
- **D 不正解**: 多段推論を要するタスクでは、思考を働かせるほうが品質が上がります。 / On multi-step reasoning, thinking improves quality.

**参照 / Reference:** LLM Fundamentals — アダプティブ思考、effort の水準
</details>

---

### 問題 8 / Question 8

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

few-shot 例を用意して抽出精度を上げようとしています。現在は、担当者が「分かりやすい」と判断した典型例を 6 件並べていますが、本番での精度は改善していません。誤りは、記載が曖昧なケースと、複数の解釈があり得るケースに集中しています。

You are adding few-shot examples to improve extraction accuracy. Currently six typical cases chosen as "clear examples" are in place, and production accuracy has not improved. Errors concentrate in ambiguously worded cases and cases with more than one possible reading.

**設問 / Question:**

few-shot 例が示すべき内容として適切なものを **2 つ選択してください**。

Select **2** things the few-shot examples should demonstrate.

- A) **本番で誤りが集中しているパターン。簡単なケースを並べても、難しい判断の手本にはならない** / **The patterns where production errors concentrate — a set of easy cases does not model the hard judgments**
- B) できるだけ多くの例（数が多いほど精度が上がる） / As many examples as possible, since more always helps
- C) 出力の文字数の目安 / A target character count for the output
- D) 例に含まれる固有名詞のバリエーション / Variations of the proper nouns appearing in the examples
- E) **どちらとも解釈できる記載をどう扱うかという、境界的な判断の基準** / **How to handle genuinely ambiguous wording — the criteria for borderline judgments**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, E**

**解説 / Explanation:**

few-shot 例の価値は**難しい判断の仕方を示すこと**にあります。誤りが曖昧なケースに集中しているなら、まさにそのケースと、そこでの判断基準を例で示すべきです。「分かりやすい例を選ぶ」という直感は、few-shot の設計としては逆方向で、簡単なケースからは学べることが少なくなります。例の選定は、評価データセットの誤答分析に基づいて行い、入れ替えた効果を測定するのが実務的な進め方です。

Few-shot examples earn their tokens by demonstrating hard judgments. When errors concentrate in ambiguous cases, those cases and the criteria for resolving them are what belong in the examples. "Pick clear examples" is the wrong instinct: easy cases teach little. Choose them from error analysis on an evaluation set and measure the effect of the change.

- **B 不正解**: 数を増やしても、同じ簡単なパターンばかりでは効果がありません。入力コストだけが増えます。 / More of the same easy pattern adds cost without effect.
- **C 不正解**: 文字数は指示で伝えられる事項で、例で示す必要はありません。 / An instruction, not something examples are needed for.
- **D 不正解**: 固有名詞のバリエーションは、判断基準とは無関係です。 / Unrelated to the judgment criteria.

**参照 / Reference:** LLM Fundamentals — few-shot 例の選定
</details>

---

### 問題 9 / Question 9

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

Python のサービスから Claude を呼び出す実装を始めます。開発者から「HTTP クライアントで直接 API を叩くのと、公式 SDK を使うのはどちらがよいか」という質問がありました。

You are starting an integration from a Python service. A developer asks whether to call the API directly with an HTTP client or use the official SDK.

**設問 / Question:**

最も適切な判断はどれですか？

Which is the most appropriate judgment?

- A) HTTP クライアントで直接叩くほうが軽量なので常に望ましい / Direct HTTP is always preferable because it is lighter
- B) **公式 SDK が提供されている言語では SDK を使う。認証、リトライ、タイムアウト、エラーの型付け、ストリーミングの扱いといった定型的な処理が実装済みで、自前で再実装すると品質と保守の負担が増える** / **Use the official SDK where one exists for the language: authentication, retries, timeouts, typed errors, and streaming handling are already implemented, and re-implementing them adds defect risk and maintenance burden**
- C) SDK は本番では使うべきでない / SDKs should not be used in production
- D) SDK と HTTP を混在させるのがよい / Mixing SDK and raw HTTP is best

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

SDK は **REST API をラップし、定型的な処理を提供する層**です。認証情報の解決、リトライ、タイムアウト、エラーの型付け、ストリーミングの扱いといった部分は、自前で書くと確実にバグが混入し、API 側の変更への追随も自分の負担になります。生の HTTP を選ぶ合理的な理由は、公式 SDK が存在しない言語である場合や、要件として明示的に HTTP を求められている場合に限られます。両者の混在は、リトライやエラー処理の方針が二重になるため避けるべきです。

An SDK wraps the REST API and supplies the boilerplate: credential resolution, retries, timeouts, typed errors, streaming. Hand-writing these reliably introduces defects and makes keeping up with API changes your problem. Raw HTTP is reasonable only where no official SDK exists for the language or where it is explicitly required. Mixing both duplicates retry and error-handling policy and should be avoided.

- **A 不正解**: 軽量さと引き換えに、定型処理を自前で実装・保守することになります。 / Trades boilerplate implementation and maintenance for marginal lightness.
- **C 不正解**: SDK は本番利用を想定して提供されています。 / SDKs are built for production use.
- **D 不正解**: 混在は方針の二重化を招き、保守を難しくします。 / Duplicates policy and complicates maintenance.

**参照 / Reference:** Technical Fundamentals — REST API をラップする SDK との統合
</details>

---

### 問題 10 / Question 10

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

本番でリクエストが長時間ハングする事象が発生しました。調査すると、SDK クライアントの生成時にタイムアウトとリトライの設定を明示しておらず、既定値がそのまま使われていました。開発者は「既定値がいくつなのか把握していなかった」と述べています。

Requests hang for a long time in production. Investigation shows the SDK client was constructed without explicit timeout and retry settings, leaving the defaults in place. The developer says they did not know what the defaults were.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) タイムアウトを無制限にする / Set the timeout to unlimited
- B) リトライを無効にする / Disable retries
- C) SDK の使用をやめる / Stop using the SDK
- D) **クライアントの設定値（タイムアウト、リトライ回数）を自分の要件に合わせて明示的に指定する。既定値は SDK ごとに異なり、単位も言語によって異なる場合があるため、把握したうえで設定する。あわせて、リトライを含めた最悪の所要時間を見積もっておく** / **Set the client's timeout and retry count explicitly to match your requirements: defaults differ per SDK and even the units differ between languages, so establish them and configure deliberately — and estimate the worst-case elapsed time including retries**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

クライアントの既定値に依存すると、**自分の要件に合っているかを検証しないまま本番に出す**ことになります。とくに注意が必要なのが、リトライとタイムアウトが掛け合わさる点です。タイムアウトが 10 分でリトライが数回あれば、最悪の所要時間はその積に近づき、呼び出し元は非常に長く待たされます。設定は明示的に行い、最悪の所要時間を見積もっておくのが正しい実践です。単位が言語によって異なる場合があることも、実際に間違いやすい点です。

Relying on client defaults ships an unverified fit with your requirements. The compounding of retries and timeouts is the specific trap: a ten-minute timeout with a few retries approaches their product as a worst case, and the caller waits all of it. Configure explicitly and estimate the worst case — and note that the units differ between languages, which is a real source of mistakes.

- **A 不正解**: 無制限のタイムアウトは、ハングの問題をさらに悪化させます。 / Makes the hang worse.
- **B 不正解**: リトライの無効化は一時的な障害に対する回復力を失わせます。設定すべきは適切な値です。 / Loses resilience; the answer is an appropriate value.
- **C 不正解**: 設定の問題であり、SDK を使わない理由にはなりません。 / A configuration problem, not a reason to drop the SDK.

**参照 / Reference:** Technical Fundamentals — SDK のクライアント設定
</details>

---

### 問題 11 / Question 11

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

複数の環境（開発・検証・本番）で動くサービスがあり、それぞれ異なる認証情報を使います。現在は、環境ごとにソースコードを分岐させて認証情報を切り替えています。

A service runs in development, staging, and production, each with different credentials. Today the source code branches per environment to select them.

**設問 / Question:**

最も適切な実装はどれですか？

Which implementation is most appropriate?

- A) **認証情報をコードから分離し、実行環境から供給する。SDK は標準的な環境変数や資格情報の解決順序に従うため、コード側の分岐は不要になる。認証情報そのものはシークレット管理から注入する** / **Separate credentials from code and supply them from the execution environment: the SDK follows a standard credential-resolution order, so no branching is needed in code, and the credentials themselves are injected from a secret store**
- B) 環境ごとに別のリポジトリを用意する / Maintain a separate repository per environment
- C) 認証情報を暗号化してソースコードに埋め込む / Embed the credentials in source, encrypted
- D) 環境判定のロジックをより詳細にする / Make the environment-detection logic more elaborate

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**認証情報はコードから分離し、実行環境から供給する**のが原則です。SDK は標準的な資格情報の解決順序を持つため、環境ごとに正しい値を供給すれば、コード側に環境判定の分岐を書く必要がありません。分岐を書くと、環境が増えるたびにコード修正が必要になり、どの環境でどの認証情報が使われるかがコードを読まないと分からなくなります。認証情報そのものは、ファイルやコードではなくシークレット管理から注入します。

Credentials belong outside code, supplied by the execution environment. SDKs resolve credentials through a standard order, so providing the right values per environment removes the need for any branching. Branching means a code change per new environment and makes the mapping between environment and credential invisible without reading the code. The credentials themselves come from a secret store, not from files or source.

- **B 不正解**: リポジトリの分割は、コードの重複と同期の問題を生みます。 / Creates duplication and synchronization problems.
- **C 不正解**: 暗号化してもコードに埋め込む限り、鍵の管理という問題が残り、履歴にも残ります。 / Still in code and in history, with a key-management problem on top.
- **D 不正解**: 分岐を詳細にしても、コードに環境依存を持ち込む構造は変わりません。 / Keeps environment dependence in the code.

**参照 / Reference:** Technical Fundamentals — 認証情報の供給、Identity, Secrets, and Key Management
</details>

---

### 問題 12 / Question 12

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

API 呼び出しのエラー処理を実装しています。現在のコードは、例外のメッセージ文字列に「rate」という語が含まれるかどうかでレート制限を判定しています。

You are implementing error handling for API calls. The current code detects rate limiting by checking whether the exception's message string contains "rate."

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 判定する文字列のパターンを増やす / Add more string patterns to match
- B) すべての例外を同じように扱う / Treat every exception identically
- C) **SDK が提供する型付きの例外クラスで判定する。文字列によるマッチは、メッセージの文言が変わると壊れ、エラーの種別も正確に区別できない** / **Branch on the typed exception classes the SDK provides: string matching breaks when the message wording changes and cannot reliably distinguish error classes**
- D) エラーが起きたら常に再試行する / Always retry on any error

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**エラーの種別は型で判定するもの**であり、メッセージ文字列は表示のためのもので判定の根拠にはなりません。文言は変わり得るため文字列マッチは脆く、また「レート制限」「認証エラー」「サーバー障害」といった区別を正確に行えません。型付きの例外クラスで分岐すれば、再試行してよいエラーとそうでないエラーを確実に切り分けられます。SDK が提供している機能を使わずに再実装している典型例でもあります。

Error classes are determined by type; message strings exist for display and are not a basis for branching. Wording changes, so string matching is brittle, and it cannot reliably separate rate limiting from authentication failure from a server error. Typed exception classes let you distinguish retryable from non-retryable definitively — and this is a textbook case of re-implementing what the SDK already provides.

- **A 不正解**: パターンを増やしても文字列依存という脆弱性は残ります。 / The string dependence remains.
- **B 不正解**: 一律の扱いは、再試行すべきエラーとすべきでないエラーを区別できません。 / Cannot separate retryable from permanent failures.
- **D 不正解**: 恒久的な失敗への再試行は時間とコストの純損失です。 / Retrying permanent failures is pure loss.

**参照 / Reference:** Technical Fundamentals — 型付き例外、Debugging and Error Handling
</details>

---

### 問題 13 / Question 13

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

チーム内で「SDK は薄いラッパーなので、自前で HTTP クライアントを書いたほうが依存を減らせる」という意見が出ています。

Someone on the team argues that "the SDK is a thin wrapper, so hand-writing an HTTP client would reduce dependencies."

**設問 / Question:**

公式 SDK を使う理由として適切なものを **2 つ選択してください**。

Select **2** reasons to use the official SDK.

- A) SDK を使うとトークン単価が下がる / The SDK reduces the per-token price
- B) SDK を使うとモデルの精度が上がる / The SDK improves model accuracy
- C) **認証、リトライ、タイムアウト、ストリーミングの扱いといった定型処理が実装済みで、自前実装による不具合を避けられる** / **Boilerplate — authentication, retries, timeouts, streaming handling — is already implemented, avoiding defects from re-implementation**
- D) **API の型やレスポンスの構造が型として提供され、変更への追随も SDK の更新で行える** / **API types and response structures are provided as types, and keeping up with API changes comes through SDK updates**
- E) SDK を使うとレート制限が緩和される / The SDK raises your rate limits

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, D**

**解説 / Explanation:**

SDK の価値は**定型処理の実装**と**型の提供および変更への追随**にあります。前者は、自前で書けば必ずバグが入る部分（認証情報の解決順序、リトライの条件、ストリーミングのイベント処理）を肩代わりします。後者は、レスポンスの構造を型として扱えるようにし、API 側の変更に SDK の更新で追随できるようにします。「薄いラッパー」に見えても、これらを自前で維持する負担は小さくありません。一方、SDK は価格・精度・レート制限には影響しません。

The SDK's value is implemented boilerplate plus types and change tracking. The first covers what hand-written code reliably gets wrong: credential resolution order, retry conditions, streaming event handling. The second gives response structures as types and turns API changes into an SDK version bump. The "thin wrapper" framing understates the cost of maintaining that yourself. Pricing, accuracy, and rate limits are unaffected either way.

- **A 不正解**: 価格は API 側で決まり、クライアントの選択とは無関係です。 / Pricing is set by the API, not the client.
- **B 不正解**: 精度はモデルとプロンプトで決まります。 / Accuracy comes from the model and prompt.
- **E 不正解**: レート制限は契約と組織の設定で決まります。 / Rate limits follow the account, not the client.

**参照 / Reference:** Technical Fundamentals — SDK を使う理由
</details>

---

### 問題 14 / Question 14

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

Web ブラウザ上のチャット UI に、Claude の応答をリアルタイムで表示したいと考えています。サーバー側は Claude からストリーミングで応答を受け取ります。ブラウザへの配信方式を検討しています。

You want to display Claude's response in real time in a browser chat UI. The server receives the response from Claude by streaming, and you are choosing how to deliver it to the browser.

**設問 / Question:**

方式の選定について最も適切な理解はどれですか？

Which is the most appropriate understanding of the choice?

- A) ブラウザへの配信も必ず websocket でなければならない / Delivery to the browser must use websockets
- B) **この用途はサーバーからクライアントへの一方向の逐次配信なので、サーバー送信イベントのような一方向のストリーミング方式で足りる。双方向の通信が必要な要件（同時編集、双方向の制御）があるなら websocket を検討する。方式は必要な通信の向きと性質で選ぶ** / **This is one-way, incremental delivery from server to client, so a one-way streaming mechanism such as server-sent events suffices; websockets are worth considering when the requirement genuinely needs bidirectional communication. Choose by the direction and nature of the communication needed**
- C) ポーリングで 100 ミリ秒ごとに問い合わせるのが最も単純で望ましい / Polling every 100 ms is the simplest and therefore best
- D) ストリーミングは使わず、完成した応答を一度に返すべきである / Do not stream; return the completed response at once

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

配信方式は、**必要な通信の向きと性質から選びます**。チャットの応答表示はサーバーからクライアントへの一方向の逐次配信なので、一方向のストリーミング方式で要件を満たせます。websocket は双方向通信が必要な場合に価値があり、そうでない用途では接続管理の複雑さが増える分だけ不利です。「常に websocket」という判断は、要件から導かれたものではありません。

Choose the mechanism from the direction and nature of the communication. Streaming a chat response is one-way, incremental, server-to-client, which a one-way mechanism satisfies. Websockets earn their added connection-management complexity when bidirectional communication is genuinely needed. "Always websockets" is not a conclusion drawn from the requirement.

- **A 不正解**: 一方向の配信に双方向の仕組みは必須ではありません。 / A bidirectional mechanism is not required for one-way delivery.
- **C 不正解**: 高頻度ポーリングは無駄な通信が多く、逐次表示の体験も劣ります。 / Wasteful and provides a worse incremental experience.
- **D 不正解**: 一括返却では、ストリーミングを使う目的（初回表示の高速化）が失われます。 / Defeats the purpose of streaming.

**参照 / Reference:** Technical Fundamentals — websocket と配信方式の選択
</details>

---

### 問題 15 / Question 15

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

外部から呼ばれる API で、Claude を呼び出して結果を返す処理を実装しています。この API はクライアント側の再試行の対象になり得ます。処理には課金を伴う操作が含まれます。

You are implementing an externally called API that invokes Claude and returns a result. Clients may retry this API, and the flow includes a billable operation.

**設問 / Question:**

最も適切な実装はどれですか？

Which implementation is most appropriate?

- A) 再試行を禁止する旨をドキュメントに記載する / Document that retries are prohibited
- B) 処理時間を短くして再試行の可能性を減らす / Shorten processing so retries are less likely
- C) 再試行されたら 2 回目以降はエラーを返す / Return an error on the second and subsequent attempts
- D) **クライアントが付与する冪等キーを受け取り、同一キーの再送に対しては最初の結果を返す。キーと結果の対応を保存し、課金を伴う操作が重複して実行されないようにする** / **Accept an idempotency key from the client and return the original result for a repeat of the same key, storing the key-to-result mapping so the billable operation cannot execute twice**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

クライアントが再試行し得る環境で課金を伴う操作を扱うには、**冪等性の実装が必須**です。クライアントが付与するキーを受け取り、キーと結果の対応を保存しておけば、同じキーでの再送に対して最初の結果をそのまま返せます。これにより、ネットワークの問題でレスポンスが失われた場合でも、クライアントは安全に再試行でき、課金は 1 回だけ発生します。ドキュメントでの注意喚起は、実装上の保証にはなりません。

Where clients may retry and the flow bills, idempotency is mandatory. Accepting a client-supplied key and persisting the key-to-result mapping lets a repeat return the original result, so a lost response can be safely retried and the charge occurs once. A note in the documentation is not an implementation guarantee.

- **A 不正解**: ドキュメントの記載は、ネットワーク障害による再送を防げません。 / Cannot prevent retransmission after a network failure.
- **B 不正解**: 処理時間を短くしても再試行の可能性は残ります。 / Retries remain possible.
- **C 不正解**: 2 回目にエラーを返すと、レスポンスを受け取れなかったクライアントが結果を得る手段を失います。 / A client that never received the first response is left with no way to obtain the result.

**参照 / Reference:** Technical Fundamentals — 冪等性、Software Engineering Foundations
</details>

---

### 問題 16 / Question 16

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

高トラフィックのサービスで、リクエストごとに SDK クライアントを新しく生成しています。負荷が上がると、コネクション数が増加し、接続の確立に時間がかかるようになりました。

A high-traffic service constructs a new SDK client for every request. Under load, connection counts climb and connection establishment becomes slow.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) **クライアントを使い回す。SDK のクライアントは内部でコネクションを再利用するように作られており、リクエストごとに生成すると接続確立のコストが毎回発生する。アプリケーションの起動時に生成して共有する** / **Reuse the client: SDK clients are built to reuse connections internally, so constructing one per request pays connection setup every time. Create it at application startup and share it**
- B) リクエストごとの生成を維持し、コネクション数の上限を上げる / Keep per-request construction and raise the connection limit
- C) タイムアウトを延長する / Increase the timeout
- D) リクエストを直列化する / Serialize the requests

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

SDK のクライアントは、**内部でコネクションを保持して再利用するように設計されています**。リクエストごとに生成すると、この再利用が働かず、毎回 TLS ハンドシェイクを含む接続確立のコストが発生します。高トラフィック環境ではこれが顕著なオーバーヘッドになります。アプリケーションの起動時に生成して共有するのが標準的な使い方で、SDK が提供する機能を活かす実装でもあります。

SDK clients are designed to hold and reuse connections. Constructing one per request defeats that and pays connection establishment, including the TLS handshake, every time — a significant overhead at high traffic. Creating the client at startup and sharing it is the standard pattern and uses what the SDK already provides.

- **B 不正解**: 上限を上げても接続確立のコストは毎回発生し、資源も浪費します。 / Setup cost recurs regardless, and resources are wasted.
- **C 不正解**: タイムアウトは接続確立の遅さを許容するだけです。 / Tolerates the slowness rather than removing it.
- **D 不正解**: 直列化はスループットを大きく落とし、原因への対処にもなりません。 / Cripples throughput and does not address the cause.

**参照 / Reference:** Technical Fundamentals — クライアントのライフサイクル
</details>

---

### 問題 17 / Question 17

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

Claude から返された構造化データを、アプリケーション内部のデータ構造に変換して扱っています。開発者は、SDK が提供する型を使わず、独自に同等の型を定義していました。SDK の更新でレスポンスに新しいフィールドが追加された際、独自定義の型がそれを取りこぼしていることに気づきました。

Structured data returned by Claude is converted into internal data structures. A developer defined equivalent types by hand rather than using the SDK's. When an SDK update added a new response field, the hand-written types silently dropped it.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 独自定義の型に新しいフィールドを追加する / Add the new field to the hand-written types
- B) SDK の更新を止める / Stop updating the SDK
- C) **SDK が提供する型をそのまま使う。API のデータ構造に対応する型は SDK に用意されており、独自に再定義するとフィールドの追加や変更に追随できず、型安全性の利点も失われる** / **Use the SDK's types directly: the SDK provides types for the API's data structures, and redefining them means falling behind on field additions and changes while losing the type-safety benefit**
- D) 型を使わず、すべて辞書型で扱う / Drop types entirely and use dictionaries

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**SDK が提供している型を再定義するのは、機能の重複実装**です。API のデータ構造に対応する型は SDK に含まれており、API 側の変更は SDK の更新で型に反映されます。独自定義すると、この追随が自分の作業になり、本問のように取りこぼしが生じます。型安全性の観点でも、SDK の型を使えばコンパイル時や静的解析で不整合を検出できます。アプリケーション固有の内部構造が必要なら、SDK の型から変換する層を明示的に置くのが適切です。

Redefining types the SDK already provides duplicates its functionality. The SDK carries types for the API's data structures, and API changes reach you through an SDK update. Hand-written equivalents make that tracking your job, which is how the field went missing. Where an application-specific internal structure is genuinely needed, convert from the SDK's types in an explicit layer.

- **A 不正解**: 今回の漏れは埋まりますが、次の変更でも同じことが起きます。 / Fixes this instance; the next change repeats it.
- **B 不正解**: 更新の停止は、セキュリティ修正や新機能も受け取れなくなります。 / Forgoes security fixes and new capability.
- **D 不正解**: 型を捨てると、静的な検出ができなくなり品質が下がります。 / Loses static detection and lowers quality.

**参照 / Reference:** Technical Fundamentals — SDK の型の利用
</details>

---

### 問題 18 / Question 18

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

Claude を呼び出すサービスの信頼性を高めたいと考えています。現在は、SDK の既定設定のまま使い、エラー処理は最上位で一括して例外を捕捉するだけです。

You want to improve the reliability of a service that calls Claude. Today it runs on the SDK's default settings, with error handling limited to a single catch-all at the top level.

**設問 / Question:**

信頼性を高める実践として適切なものを **2 つ選択してください**。

Select **2** practices that improve reliability.

- A) 例外をすべて握りつぶして処理を継続する / Swallow all exceptions and continue processing
- B) **エラーを種別ごとに扱い分ける。再試行が有効なもの（レート制限、一時的なサーバー障害）と、そうでないもの（不正なリクエスト、認証エラー）を型で区別して処理する** / **Handle errors by class, distinguishing by type those worth retrying — rate limits, transient server failures — from those that are not, such as malformed requests and authentication errors**
- C) タイムアウトを無制限にする / Set timeouts to unlimited
- D) すべてのエラーで同じ回数だけ再試行する / Retry every error the same number of times
- E) **タイムアウトとリトライを要件に合わせて明示的に設定し、リトライを含めた最悪の所要時間を把握しておく** / **Set timeouts and retries explicitly to match requirements, and know the worst-case elapsed time including retries**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, E**

**解説 / Explanation:**

信頼性を高める実践の中心は、**エラーの種別に応じた扱い**と**タイムアウト・リトライの明示的な設定**です。前者により、回復可能な失敗には再試行が働き、恒久的な失敗には無駄な待ち時間が発生しません。後者は、既定値に依存せず自分の要件に合わせることと、リトライが掛け合わさった最悪の所要時間を把握しておくことの 2 つを含みます。最上位での一括捕捉だけでは、この区別ができず、呼び出し元がどれだけ待たされるかも制御できません。

The core practices are class-aware error handling and explicit timeout and retry configuration. The first retries what can recover and stops wasting time on what cannot. The second means not inheriting defaults and knowing the worst-case elapsed time once retries compound. A single top-level catch provides neither the distinction nor control over how long the caller waits.

- **A 不正解**: 例外の握りつぶしは、失敗を検知できない状態を作ります。 / Makes failures undetectable.
- **C 不正解**: 無制限のタイムアウトは、ハングをそのまま許容します。 / Tolerates hangs indefinitely.
- **D 不正解**: 一律の再試行は、恒久的な失敗にも時間を費やします。 / Wastes time on permanent failures.

**参照 / Reference:** Technical Fundamentals — エラー処理、クライアント設定
</details>

---

### 問題 19 / Question 19

> サブスキル / Sub-skill: Model Selection and Tradeoffs (2.7%)

**シナリオ / Scenario:**

問い合わせを 5 カテゴリに分類するタスクを、1 日 40 万件処理します。評価したところ、軽量なモデルでも精度 99.1%、上位モデルで 99.4% でした。分類結果は担当チームへの振り分けに使われ、誤分類は担当者が気づいて手動で振り替えます。

A task classifying inquiries into five categories runs 400,000 times a day. Evaluation shows 99.1% accuracy on a light model and 99.4% on a stronger one. The classification routes items to the owning team, and a misroute is noticed and manually corrected by staff.

**設問 / Question:**

最も適切なモデル選定はどれですか？

Which model selection is most appropriate?

- A) 常に最上位のモデルを使う / Always use the strongest model
- B) 精度が高いほうを無条件に選ぶ / Choose the higher accuracy unconditionally
- C) モデルは選ばず、両方をランダムに使う / Alternate randomly between them
- D) **軽量なモデルを選ぶ。精度差 0.3 ポイントは 1 日 1,200 件の誤分類差に相当するが、誤分類は担当者が気づいて振り替えられる回復可能な誤りであり、40 万件分の単価差に見合わない可能性が高い。判断は、精度差の業務価値とコスト差を突き合わせて行う** / **Choose the light model: 0.3 points is about 1,200 more misroutes a day, but a misroute is a recoverable error that staff catch and correct, and that is unlikely to justify the unit-price difference across 400,000 items. Decide by comparing the business value of the accuracy difference against the cost difference**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

モデル選定は「精度が高いほうを選ぶ」ではなく、**精度差の業務価値とコスト差を突き合わせる判断**です。本問では、誤分類が回復可能（担当者が気づいて振り替える）である点が決定的で、誤りのコストは振り替えの手間に限られます。0.3 ポイントの差を 1 日 1,200 件という業務上の量に換算し、その手間と 40 万件分の単価差を比較して初めて、根拠のある選定になります。誤りが不可逆であったり検知できない用途であれば、同じ 0.3 ポイントでも結論は変わります。

Model selection is not "pick the higher number" but a comparison between the business value of the accuracy difference and the cost difference. What decides it here is that a misroute is recoverable: its cost is bounded by the effort to re-route. Converting 0.3 points into 1,200 items a day and weighing that effort against the unit-price difference over 400,000 items is what makes the choice defensible. Where errors were irreversible or undetected, the same 0.3 points could point the other way.

- **A 不正解**: 用途に関わらず最上位を選ぶのは、コスト構造を検討していない判断です。 / Ignores the cost structure entirely.
- **B 不正解**: 精度差の業務価値を評価せずに選ぶと、この規模ではコストが大きく変わります。 / At this volume, ignoring the value of the difference is expensive.
- **C 不正解**: ランダムな使い分けは挙動を予測不能にし、選定の根拠になりません。 / Makes behavior unpredictable and is not a basis.

**参照 / Reference:** Model Selection and Tradeoffs — 品質・コストのトレードオフ
</details>

---

### 問題 20 / Question 20

> サブスキル / Sub-skill: Model Selection and Tradeoffs (2.7%)

**シナリオ / Scenario:**

新機能のモデル選定を求められました。チームからは「一番いいモデルを使えばよいのでは」という声があります。この機能は対話型で、ユーザーが応答を待ちます。

You are asked to select a model for a new feature. The team suggests simply using the best one. The feature is interactive, with users waiting on the response.

**設問 / Question:**

モデル選定の考え方として最も適切なものはどれですか？

Which is the most appropriate way to approach model selection?

- A) 常に最も安価なモデルから試す / Always start with the cheapest model
- B) **品質・レイテンシ・コストの 3 軸で要件を先に定め、それを満たす候補の中から選ぶ。対話型では応答時間が体験を左右するため、品質だけで選ぶと要件を満たさないことがある。候補は自社の評価データセットで比較する** / **Define requirements on three axes first — quality, latency, cost — and choose among candidates that satisfy them. In an interactive feature, response time drives the experience, so selecting on quality alone can fail the requirement. Compare candidates on your own evaluation set**
- C) 公開されているベンチマークのスコアが最も高いモデルを選ぶ / Choose the model with the highest published benchmark scores
- D) チームの多数決で決める / Decide by team vote

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

モデル選定は**多目的の判断**であり、品質・レイテンシ・コストのどれか 1 つだけで決めるものではありません。対話型の機能では応答時間が体験に直結するため、品質が最も高くてもレイテンシ要件を満たさなければ採用できません。順序としては、まず 3 軸の要件（許容レイテンシ、最低品質、単価上限）を定めて候補を絞り、残った候補を自社の評価データセットで比較します。公開ベンチマークは汎用能力の指標で、自社タスクの性能を保証しません。

Model selection is a multi-objective decision, not a single-axis one. In an interactive feature, response time is the experience, so the highest-quality model is unusable if it misses the latency requirement. The order is: fix requirements on all three axes, eliminate candidates that fail them, then compare the survivors on your own evaluation set. Public benchmarks measure general capability and do not guarantee performance on your task.

- **A 不正解**: 安さから始めるのも 1 軸だけの判断で、品質要件を無視しています。 / Also single-axis, ignoring quality.
- **C 不正解**: ベンチマークは汎用能力の指標で、自社タスクの性能を予測しません。 / Does not predict task-specific performance.
- **D 不正解**: 多数決は要件との適合を反映しません。 / A vote does not reflect fit to requirements.

**参照 / Reference:** Model Selection and Tradeoffs — 品質・レイテンシ・コスト
</details>

---

### 問題 21 / Question 21

> サブスキル / Sub-skill: Model Selection and Tradeoffs (2.7%)

**シナリオ / Scenario:**

使用しているモデルを新しいバージョンに更新したところ、全体の精度は向上しましたが、出力の文体が変わり、下流の正規表現ベースの処理が一部で失敗するようになりました。また、境界的な入力での判定が旧バージョンと食い違うケースが見つかりました。

After updating to a new model version, overall accuracy improved but the output's phrasing changed, breaking some downstream regex-based processing. Borderline inputs also produce decisions that differ from the previous version.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **モデルの更新は挙動の変化を伴い得ると理解し、更新を計画的な移行として扱う。下流の脆さ（文体依存）は構造化出力で解消し、判定の差分は事前に評価データセットで比較して、変化の内容を把握したうえで切り替える** / **Treat a model update as a deliberate migration on the premise that behavior can change: fix the downstream brittleness by moving to structured output, compare decisions on an evaluation set beforehand, and switch with the differences understood**
- B) モデルの更新を今後行わない / Never update the model again
- C) 新モデルに旧モデルと同じ文体で出力するよう指示する / Instruct the new model to match the old phrasing
- D) 下流の正規表現をより複雑にする / Make the downstream regular expressions more elaborate

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

モデルのリリースをまたぐと**挙動が変わり得る**というのは前提であり、これを踏まえた移行の進め方が必要です。本問には 2 つの問題があります。文体への依存という下流の脆さは、構造化出力に変えることで根本的に解消できます。判定の差分は、事前に評価データセットで新旧を比較して把握すべき事項で、とくに境界的な入力での変化は業務上の意味を確認する必要があります。全体精度の向上は、個別の判定が変わらないことを意味しません。

Behavior can change across model releases, and migration must be conducted on that premise. Two problems appear here: the downstream dependence on phrasing, which structured output removes at the root, and the changed decisions, which should be characterized on an evaluation set beforehand — borderline changes especially warrant a check on what they mean operationally. An aggregate accuracy gain does not imply that individual decisions held.

- **B 不正解**: 更新の停止は、いずれ非推奨化で強制的な移行に直面します。 / Deprecation eventually forces the migration anyway.
- **C 不正解**: 文体を模倣させても、正規表現依存という脆弱性は残り、次の更新で再発します。 / Leaves the brittleness for the next update.
- **D 不正解**: 正規表現を複雑にしても、文体に依存する構造は変わりません。 / The dependence on phrasing remains.

**参照 / Reference:** Model Selection and Tradeoffs — モデルのリリースをまたぐ挙動変化
</details>

---

### 問題 22 / Question 22

> サブスキル / Sub-skill: Model Selection and Tradeoffs (2.7%)

**シナリオ / Scenario:**

社内文書 Q&A の回答品質が低いため、より上位のモデルに変更しました。しかし品質はほとんど改善しませんでした。調査すると、誤答の大半は「回答に必要な文書がコンテキストに入っていなかった」ケースでした。

To improve poor answer quality in an internal document Q&A, you switched to a stronger model, but quality barely changed. Investigation shows most wrong answers occurred when the document needed to answer was never in the context.

**設問 / Question:**

この結果の正しい理解はどれですか？

Which is the correct understanding of this result?

- A) 上位モデルの選定が誤っていた / The wrong stronger model was chosen
- B) さらに上位のモデルを試すべきである / An even stronger model should be tried
- C) **ボトルネックがモデルの能力ではなく取得の側にあるため、モデルを変えても改善しない。必要な文書が取得されるかを別途測定し、取得側（インデックス、チャンク、検索）を改善する必要がある** / **The bottleneck is retrieval, not model capability, so changing the model cannot help. Measure retrieval separately — whether the needed document is retrieved — and improve the indexing, chunking, and search**
- D) 文書 Q&A には LLM が適さない / LLMs are unsuited to document Q&A

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**モデルの能力が制約でない場面でモデルを上げても改善しません。**回答に必要な文書がコンテキストに入っていなければ、どれだけ能力の高いモデルでも正しく答えられません。この状況を見分けるには、評価を「必要な情報が取得されたか」と「取得された情報から正しい回答が作れたか」に分解して測る必要があります。分解せずに一次元の精度だけを見ていると、本問のように効かない改善に投資することになります。

Raising model capability cannot help where capability is not the constraint: no model answers correctly from a context that lacks the necessary document. Distinguishing this requires decomposing the evaluation into whether the needed information was retrieved and whether a correct answer was produced from what was retrieved. Measuring only end-to-end accuracy leads to exactly this kind of ineffective investment.

- **A 不正解**: モデルの選び方の問題ではなく、ボトルネックの所在の問題です。 / A question of where the bottleneck is, not which model.
- **B 不正解**: さらに上位のモデルでも、存在しない情報からは答えられません。 / No model answers from information that is absent.
- **D 不正解**: 取得を改善すれば成立する用途であり、適不適の問題ではありません。 / The use case works once retrieval improves.

**参照 / Reference:** Model Selection and Tradeoffs — ボトルネックの特定
</details>

---

### 問題 23 / Question 23

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

Claude の利用コストが想定より高いことが分かりました。どの処理がコストを消費しているかを把握したいのですが、現在は月次の請求総額しか見ていません。

Claude spend is higher than expected. You want to know which processing drives it, but only the monthly total is currently visible.

**設問 / Question:**

最初に行うべきことはどれですか？

What should be done first?

- A) すべての処理を最も安価なモデルに切り替える / Switch everything to the cheapest model
- B) プロンプトを短くする / Shorten the prompts
- C) 利用を一時停止する / Suspend usage
- D) **各リクエストの応答に含まれる使用量情報（入力・出力・キャッシュのトークン数）を記録し、機能や処理の種別と紐付けて集計する。どこが支配的かを把握してから対策を選ぶ** / **Record the usage information each response carries — input, output, and cache token counts — tag it by feature or processing type, and aggregate it, so the dominant contributors are known before choosing a remedy**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**測定なしに最適化はできません。**応答には使用量の情報（入力トークン、出力トークン、キャッシュの読み書き）が含まれるので、これを機能や処理の種別と紐付けて記録すれば、どこが支配的かが分かります。多くの場合、コストは特定の処理に偏っており、そこに集中して対策するのが効率的です。測定せずにモデル変更やプロンプト短縮を行うと、支配的でない部分に労力を使い、品質だけを落とすことになりかねません。

Optimization requires measurement. Each response carries usage information — input and output tokens, cache reads and writes — and tagging it by feature or processing type reveals what dominates. Cost is usually concentrated, and concentrating the remedy there is what makes it efficient. Changing models or shortening prompts before measuring risks spending effort on a minor contributor while degrading quality.

- **A 不正解**: 品質要件を確認せずに全処理を最安モデルに落とすのは、コスト以外の要件を無視した対応です。 / Ignores quality requirements across every feature at once.
- **B 不正解**: プロンプト短縮が有効かは、入力側が支配的かどうかによります。測定が先です。 / Whether it helps depends on measurement.
- **C 不正解**: 一時停止は機能の提供を止めるだけで、原因の把握になりません。 / Stops the feature without revealing the cause.

**参照 / Reference:** Cost and Token Management — 使用量の追跡
</details>

---

### 問題 24 / Question 24

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

日次バッチ処理のコストを下げたいと考えています。各リクエストは同一の 20,000 トークンの指示を接頭辞に持ち、その後に個別のデータが続きます。処理は翌朝までに終わればよく、リアルタイム性は不要です。

You want to reduce the cost of a daily batch. Each request carries an identical 20,000-token instruction prefix followed by individual data. Results are needed by the next morning, with no real-time requirement.

**設問 / Question:**

品質を犠牲にせずコストを下げる手段を **2 つ選択してください**。

Select **2** ways to reduce cost without sacrificing quality.

- A) **共通する 20,000 トークンの接頭辞にプロンプトキャッシュを適用する** / **Apply prompt caching to the shared 20,000-token prefix**
- B) 処理対象の件数を半分に減らす / Halve the number of items processed
- C) **遅延許容があるため、バッチ処理として投入する** / **Exploit the latency tolerance by submitting the work as a batch**
- D) 出力トークンの上限を厳しく制限する / Impose a tight cap on output tokens
- E) 入力データを要約してから処理する / Summarize the input data before processing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, C**

**解説 / Explanation:**

このワークロードは、**品質を落とさずにコストを下げる 2 つの条件を備えています**。長い共通接頭辞があるのでプロンプトキャッシュが効き、遅延許容があるのでバッチ処理が使えます。どちらも出力の内容には影響せず、効果は独立に作用するため組み合わせられます。一方、件数削減・出力の切り詰め・入力の要約は、いずれも処理する情報量を減らす手段で、品質やカバレッジを犠牲にします。品質を落とさない手段を尽くしてから、初めてトレードオフを伴う手段を検討するのが順序です。

This workload has two conditions for reducing cost without touching quality: a long shared prefix (caching) and latency tolerance (batch). Neither changes the output, and they compose. Reducing volume, capping output, and summarizing input all reduce the information processed and cost quality or coverage. Exhaust the free levers before considering any that trade quality away.

- **B 不正解**: 件数削減はカバレッジの放棄であり、コスト最適化ではありません。 / Abandons coverage rather than optimizing.
- **D 不正解**: 出力の切り詰めは結果を途中で切る危険があり、削減幅も入力側に比べて小さいです。 / Risks truncation and saves little against a 20,000-token input.
- **E 不正解**: 要約は情報を落とし、処理結果の品質を直接損ないます。 / Drops information and degrades the result.

**参照 / Reference:** Cost and Token Management — キャッシュ、Claude API Mechanics — batch
</details>

---

### 問題 25 / Question 25

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

新機能のコストを見積もっています。1 リクエストあたり、入力が 30,000 トークン、出力が 500 トークンの見込みです。開発者は「出力を短くすればコストが下がる」と考え、出力を 300 トークンに削る案を検討しています。

You are estimating the cost of a new feature: about 30,000 input tokens and 500 output tokens per request. A developer proposes trimming output to 300 tokens on the assumption that shorter output reduces cost.

**設問 / Question:**

最も適切な指摘はどれですか？

What is the most appropriate observation?

- A) 出力を削れば大幅にコストが下がる / Trimming output reduces cost substantially
- B) **このワークロードは入力側が支配的であり、出力を 200 トークン削っても全体への影響は小さい。削減効果の大きい入力側（キャッシュの活用、不要なコンテキストの除去）から着手すべきである。出力単価は入力より高いが、量の差がそれを上回っている** / **This workload is input-dominated: trimming 200 output tokens barely moves the total. Start where the savings are — the input side, through caching and removing unnecessary context. Output is priced higher per token, but the volume difference outweighs that**
- C) 入力と出力の単価は同じなので、どちらを削っても同じである / Input and output are priced identically, so it makes no difference which you trim
- D) コストは出力トークン数のみで決まる / Cost is determined solely by output tokens

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

コスト最適化では、**どちらの側が支配的かを量と単価の両方で見る**必要があります。出力トークンは入力より単価が高く設定されていますが、本問では入力が出力の 60 倍あるため、総額は入力側に支配されています。200 トークンの削減より、入力側の対策（共通部分のキャッシュ、不要なコンテキストの除去、取得の絞り込み）のほうがはるかに効果が大きくなります。単価だけを見て「出力が高いから出力を削る」と判断すると、効果の小さい対策に労力を使うことになります。

Cost optimization requires looking at both volume and unit price to see which side dominates. Output is priced higher per token, but here input is sixty times the volume, so the total is input-dominated. Caching the shared prefix, removing unnecessary context, and tightening retrieval all move the number far more than 200 output tokens. Reasoning from unit price alone points at the wrong lever.

- **A 不正解**: 200 トークンの削減は、30,500 トークンの構成では影響が小さいです。 / Negligible against 30,500 tokens.
- **C 不正解**: 単価は同じではありません。出力のほうが高く設定されています。 / They are not identical; output is priced higher.
- **D 不正解**: 入力トークンも課金対象で、この構成ではむしろ支配的です。 / Input is billed and dominates here.

**参照 / Reference:** Cost and Token Management — コストのモデリング
</details>

---

### 問題 26 / Question 26

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

プロンプトを次の要素で構成しています。(1) 役割と出力形式の定義（2,000 トークン、ほぼ変わらない）、(2) 参照資料（25,000 トークン、週次更新）、(3) 会話履歴（可変）、(4) ユーザーの質問。キャッシュを効かせたいと考えています。

Your prompt consists of: (1) role and output-format definitions (2,000 tokens, nearly static), (2) reference material (25,000 tokens, updated weekly), (3) conversation history (variable), (4) the user's question. You want caching to be effective.

**設問 / Question:**

最も適切な構成はどれですか？

Which arrangement is most appropriate?

- A) **(1) → (2) → (3) → (4) の順に、変化が遅いものから速いものへ並べる。キャッシュの境界は (2) の後ろに置き、週次更新までの間は 27,000 トークン分が再利用される** / **Order them (1) → (2) → (3) → (4), from slowest-changing to fastest, with the cache boundary after (2) so 27,000 tokens are reused until the weekly update**
- B) (4) → (3) → (2) → (1) の順に並べる / Order them (4) → (3) → (2) → (1)
- C) (3) を先頭に置く / Put (3) at the head
- D) 順序は影響しないので現状のままでよい / Order does not matter; leave it as is

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

キャッシュは**接頭辞の一致**で効くため、構成の原則は「変化が遅いものを先に、速いものを後に」です。役割定義（ほぼ不変）と参照資料（週次更新）を先頭に置けば、週次更新までの間は 27,000 トークン分の接頭辞が一致し続けます。会話履歴とユーザーの質問はリクエストごとに変わるため、キャッシュ境界より後ろに置きます。可変部分を先頭に置くと、その時点で接頭辞が一致しなくなり、後続にどれだけ共通の内容があっても効きません。

Caching matches on a prefix, so the rule is slowest-changing first. Putting the near-static role definition and the weekly reference material at the head keeps a 27,000-token prefix matching until the weekly update. History and the question vary per request and belong behind the cache boundary. Any variable content placed at the head breaks the match immediately, regardless of what shared content follows.

- **B 不正解**: 最も変化が速いものを先頭に置く構成で、キャッシュがまったく効きません。 / Puts the fastest-changing content first; nothing caches.
- **C 不正解**: 会話履歴はリクエストごとに変わるため、先頭に置くと接頭辞が一致しません。 / History varies per request and breaks the prefix.
- **D 不正解**: 順序はキャッシュの成否を決める最も重要な要素です。 / Order is the decisive factor.

**参照 / Reference:** Cost and Token Management — キャッシュ手法、接頭辞の構成
</details>

---

### 問題 27 / Question 27

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

社内 Q&A のコストを分析したところ、入力トークンの大半が参照資料で占められていました。現在は 800 件の社内文書をすべて毎回コンテキストに入れています。1 回の質問で実際に必要な文書は 1〜3 件です。

Cost analysis of an internal Q&A shows input tokens dominated by reference material: all 800 internal documents are placed in context on every request, though a question typically needs one to three of them.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 800 件の文書を要約して短くする / Summarize the 800 documents to make them shorter
- B) より安いモデルに変更する / Switch to a cheaper model
- C) **質問に関連する文書だけを実行時に取得してコンテキストに入れる。入力トークンが必要な分だけになり、無関係な情報が減ることで回答の品質も改善が見込める** / **Retrieve only the documents relevant to the question at request time, so input tokens scale with what is needed — and with less irrelevant material, answer quality is likely to improve as well**
- D) 質問できる件数を制限する / Limit how many questions can be asked

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

1〜3 件で足りる用途で 800 件すべてを毎回送るのは、**入力トークンの大半が使われないまま課金されている**状態です。関連文書だけを取得する構成にすれば、入力量が必要な分に比例し、コストが大幅に下がります。加えて、無関係な 797 件が減ることで関連情報が埋没しにくくなり、回答品質の改善も期待できます。コスト削減と品質改善が同じ方向に働く、望ましい改善です。

Sending all 800 when one to three suffice means paying for input that goes unused. Retrieving only the relevant documents makes input scale with need and cuts cost substantially. It also removes 797 documents of noise, so the relevant material is less likely to be buried — cost and quality improve in the same direction.

- **A 不正解**: 要約は情報を落とし、必要な詳細が失われます。無関係な文書を送る構造も変わりません。 / Loses detail and still sends irrelevant documents.
- **B 不正解**: 無駄な入力を安い単価で送るだけで、構造的な無駄は残ります。 / Sends the same waste at a lower price.
- **D 不正解**: 利用の制限は機能の劣化であり、コスト最適化ではありません。 / Degrades the feature rather than optimizing.

**参照 / Reference:** Cost and Token Management — 入力トークンの削減、取得による絞り込み
</details>

---

## 発展 / Advanced

### 問題 28 / Question 28

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

複雑な判定タスクで effort の水準を段階的に上げて測定しました。低い水準で精度 74%、中程度で 86%、高い水準で 91%、最も高い水準で 91.3% でした。処理コストは水準にほぼ比例して増加します。このタスクは 1 日 5,000 件処理し、誤りは修正作業（1 件あたり平均 30 分）につながります。

Measuring accuracy across effort levels on a complex decision task: 74% at low, 86% at medium, 91% at high, and 91.3% at the highest. Cost scales roughly with the level. The task runs 5,000 times a day and each error causes about 30 minutes of correction work.

**設問 / Question:**

最も適切な判断はどれですか？

Which decision is most appropriate?

- A) 最も高い水準を選ぶ / Choose the highest level
- B) **高い水準あたりを選ぶ。最上位への引き上げは 0.3 ポイントの改善にとどまり、収穫が明確に逓減している。0.3 ポイントは 1 日 15 件の誤り削減（約 7.5 時間の作業）に相当するため、その価値と増加するコストを比較して最終決定し、その計算を記録しておく** / **Choose around the high level: the step to the highest buys 0.3 points, a clear point of diminishing returns. That 0.3 points is 15 fewer errors a day, about 7.5 hours of correction work, so decide by comparing that against the added cost — and record the calculation**
- C) 最も低い水準を選ぶ / Choose the lowest level
- D) 水準は測定せず、既定のままにする / Leave it at the default without measuring

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

effort は「高いほど良い」ものではなく、**収穫逓減の点を測定で特定する**対象です。高い水準から最上位への 0.3 ポイントは典型的な飽和で、コスト増に見合いません。一方、中程度から高い水準への 5 ポイントは 1 日 250 件の誤り削減に相当し、明らかに価値があります。ここで重要なのは、**精度差を業務上の量と時間に換算して比較する**ことで、この換算があることで技術的な設定がステークホルダーに説明可能になります。

Effort is not monotonically worth raising; the job is locating the knee empirically. The 0.3 points from high to highest is saturation and does not repay the cost, while the 5 points from medium to high is 250 fewer errors a day and clearly does. What makes the decision defensible is converting accuracy into operational quantities — errors and hours — which is also what makes the setting explainable to stakeholders.

- **A 不正解**: 0.3 ポイントのために大幅なコスト増を払う判断で、費用対効果の検討がありません。 / Pays substantially more for 0.3 points with no cost-benefit basis.
- **C 不正解**: 低い水準では精度 74% で、高い水準との差は 1 日 850 件の誤りに相当します。 / The gap to high is 850 errors a day.
- **D 不正解**: 測定しなければ、どの水準が適切かを判断できません。 / Without measurement there is no basis.

**参照 / Reference:** LLM Fundamentals — effort の調整、収穫逓減
</details>

---

### 問題 29 / Question 29

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

対話型のコーディング支援機能で、応答の生成速度を上げたいという要望があります。現在は品質に問題はなく、体感速度だけが課題です。バッチ処理での利用や、複数のクラウドプラットフォーム経由での提供も将来的に検討しています。

For an interactive coding-assist feature, there is a request to increase generation speed. Quality is fine; only perceived speed is the problem. Batch usage and delivery through several cloud platforms are also under future consideration.

**設問 / Question:**

高速な生成モードの検討にあたり、最も適切な理解はどれですか？

Which is the most appropriate understanding when evaluating a faster generation mode?

- A) 高速モードはすべてのモデルとすべての経路で使える / A fast mode is available on every model and every path
- B) 高速モードを使えば品質が上がる / A fast mode improves quality
- C) 高速モードを使えばコストが下がる / A fast mode reduces cost
- D) **高速な生成モードは、対応するモデルと提供経路が限られ、価格も通常と異なる。バッチ処理や一部のプラットフォーム経由では利用できない場合があるため、将来の利用形態も含めて対応状況を確認したうえで採用を判断する** / **A fast generation mode is limited in which models and delivery paths support it and is priced differently; it may be unavailable through batch processing or certain platforms, so confirm availability against your intended usage — including future forms — before adopting it**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

高速な生成モードのような**特定の条件でのみ使える機能**を採用する際は、対応するモデル・提供経路・価格・併用できない機能を事前に確認する必要があります。本問では、バッチ処理での利用と複数プラットフォーム経由での提供が将来の検討事項として挙がっているため、これらで使えるかどうかが採用判断に直結します。「速くなる」という効果だけを見て採用すると、後から利用形態を広げるときに制約が顕在化します。

Adopting a capability that is available only under specific conditions requires checking which models and paths support it, how it is priced, and what it cannot be combined with. Here, batch usage and multi-platform delivery are already anticipated, so their support status bears directly on the decision. Adopting on the speed benefit alone surfaces the constraint later, when usage expands.

- **A 不正解**: 対応するモデルと経路は限られます。 / Support is limited by model and path.
- **B 不正解**: 生成速度の話であり、品質を上げる機能ではありません。 / A speed feature, not a quality one.
- **C 不正解**: 高速な生成は通常より高い価格が設定されるのが一般的で、コスト削減の手段ではありません。 / Typically priced above standard; not a cost lever.

**参照 / Reference:** LLM Fundamentals — モデルのオプション、Model Selection and Tradeoffs
</details>

---

### 問題 30 / Question 30

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

規制対象の業務で Claude を使う設計をしています。監査部門から「同じ入力に対して同じ出力が得られることを保証してほしい」という要求がありました。

You are designing a regulated workflow using Claude. The audit function asks for a guarantee that identical input produces identical output.

**設問 / Question:**

非決定性を踏まえた設計上の対応として適切なものを **2 つ選択してください**。

Select **2** appropriate design responses given nondeterminism.

- A) **生成物とその生成条件（入力、モデルとプロンプトの版数、設定値）を保存し、監査要件を「再生成」ではなく「記録の完全性」で満たす** / **Retain the artifact together with the conditions that produced it — input, model and prompt versions, settings — satisfying the audit requirement through record integrity rather than regeneration**
- B) 同じ出力が出るまで再実行する / Re-run until the same output appears
- C) 出力のハッシュを保存し、一致しなければエラーとする / Store a hash of the output and error when it does not match
- D) **判定など重要な結論は、モデルの出力そのものではなく、決定的に検証できる形（列挙された値、スキーマ適合、突合可能な数値）で受け取り、下流の処理は検証済みの値に対して行う** / **Take consequential conclusions in a deterministically verifiable form — enumerated values, schema conformance, reconcilable figures — rather than as raw model output, and run downstream processing on the verified values**
- E) 温度を 0 に設定すれば完全に決定的になるので、それで要件を満たす / Set temperature to zero, which makes it fully deterministic and satisfies the requirement

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, D**

**解説 / Explanation:**

非決定性を前提とすると、監査要件は 2 方向で満たします。第一に、**記録の完全性**です。生成物と生成条件を揃えて保存すれば、数年後でも「この出力はこの条件から生まれた」と示せます。監査が本当に必要としているのは再生成ではなく、当時の判断の検証可能性です。第二に、**決定的に検証できる形での受け取り**です。列挙された値やスキーマ適合、突合可能な数値として受け取れば、下流の処理は検証済みの値に対して決定的に動きます。

Given nondeterminism, the audit requirement is met from two directions. First, record integrity: retaining the artifact alongside its conditions establishes years later which conditions produced which output, and verifiability of the original decision is what audit actually needs. Second, taking conclusions in deterministically verifiable form — enumerations, schema conformance, reconcilable figures — so downstream processing operates deterministically on verified values.

- **B 不正解**: 一致するまで再実行するのは非決定性を隠す操作で、監査上むしろ不誠実です。 / Conceals nondeterminism and is audit-adverse.
- **C 不正解**: 正常な非決定性を障害として扱うことになり、システムが不安定になります。 / Treats normal behavior as a fault.
- **E 不正解**: サンプリング設定を変えてもビット単位の決定性は保証されません。 / Sampling settings do not guarantee bit-exact determinism.

**参照 / Reference:** LLM Fundamentals — 非決定性、監査要件の翻訳
</details>

---

### 問題 31 / Question 31

> サブスキル / Sub-skill: LLM Fundamentals (5.2%)

**シナリオ / Scenario:**

few-shot 例を 4 件から 30 件に増やしたところ、評価データセットでの精度がわずかに下がり、1 リクエストあたりの入力トークンが 6 倍になりました。追加した 26 件は、既存の 4 件と似た内容です。

Increasing few-shot examples from 4 to 30 slightly reduced accuracy on the evaluation set and multiplied input tokens per request by six. The 26 added examples resemble the original four.

**設問 / Question:**

最も適切な理解と対応はどれですか？

Which understanding and response is most appropriate?

- A) 例が足りないので 100 件に増やす / Increase to 100, since there are still too few
- B) few-shot は効果がないので廃止する / Abandon few-shot as ineffective
- C) 精度低下はノイズなので無視する / Dismiss the drop as noise
- D) **例は多いほど良いわけではない。似た内容の例を増やしても新しい情報が加わらず、入力コストと処理する情報量だけが増える。誤りが集中しているパターンや境界的な判断を含む、内容の異なる少数の例に絞るほうが効果が大きい** / **More examples are not automatically better: near-duplicate examples add no information while multiplying input cost and material to process. A smaller set of genuinely different examples — drawn from where errors concentrate and from borderline judgments — is more effective**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

few-shot 例の効果は**件数ではなく内容の多様性**で決まります。似た内容の例を並べても、示している判断は同じなので新しい情報が加わりません。一方、入力トークンは確実に増え、コストとレイテンシに直結します。精度がわずかに下がったのも、有効な情報が増えないまま処理する量だけが増えた結果として説明できます。例の選定は、誤答分析に基づいて内容の異なるものを選び、入れ替えの効果を測定するのが実務的です。

Few-shot effectiveness comes from the diversity of the examples, not their count. Near-duplicates demonstrate the same judgment and add nothing, while input tokens rise reliably and with them cost and latency. The small accuracy drop is consistent with more material to process and no more information in it. Select examples from error analysis for genuine variety, and measure the effect of the change.

- **A 不正解**: 同じ内容の例をさらに増やしても、同じ結果が拡大するだけです。 / More of the same produces more of the same.
- **B 不正解**: 元の 4 件は効果があったと考えられ、廃止は行き過ぎです。 / The original four were working; abandoning is an overreaction.
- **C 不正解**: 入力トークン 6 倍という明確なコスト増を無視できません。 / A sixfold input cost increase is not dismissible.

**参照 / Reference:** LLM Fundamentals — multi-shot、例の選定
</details>

---

### 問題 32 / Question 32

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

Python のサービスと TypeScript のサービスの両方から Claude を呼び出しています。両者で「タイムアウトを 30 に設定」という同じ意図の実装をしたところ、片方は 30 秒、もう片方は 30 ミリ秒として扱われ、後者で大量のタイムアウトが発生しました。

Both a Python service and a TypeScript service call Claude. Each was implemented with the same intent — "set the timeout to 30" — but one was treated as 30 seconds and the other as 30 milliseconds, causing mass timeouts in the latter.

**設問 / Question:**

この事象からの適切な教訓はどれですか？

What is the appropriate lesson?

- A) 両方のサービスを同じ言語で書き直す / Rewrite both services in the same language
- B) タイムアウトは設定しないほうがよい / It is better not to set timeouts at all
- C) **SDK の設定値は、言語によって単位や既定値が異なる場合がある。数値をそのまま移植せず、各言語のドキュメントで単位と既定値を確認したうえで設定する。設定値は環境ごとに検証し、意図した時間で動作しているかを確認する** / **SDK settings can differ in units and defaults between languages: do not port a number verbatim — confirm the unit and default in each language's documentation, and verify in each environment that the configured value behaves as intended**
- D) タイムアウトの値をすべて同じにする / Use the same numeric value everywhere

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**同じ意図の設定でも、言語や SDK によって単位や既定値が異なることがあります。**数値をそのまま移植するのは、単位が同じであるという未検証の前提に立った実装です。設定を行う際は各言語のドキュメントで単位と既定値を確認し、実際に意図した時間で動作しているかを検証する必要があります。この種の差異は、コードレビューでも見落とされやすく、本番で初めて顕在化することが多い問題です。

The same intent can map to different units and defaults across languages and SDKs. Porting a number verbatim rests on an unverified assumption that the unit is the same. Confirm the unit and default per language when configuring, and verify that the effective duration matches the intent. This class of difference is easy to miss in review and typically surfaces first in production.

- **A 不正解**: 言語の統一は現実的でなく、単位を確認するという教訓を得ていません。 / Impractical, and misses the lesson.
- **B 不正解**: タイムアウトの未設定は、ハングを許容することになります。 / Leaving timeouts unset tolerates hangs.
- **D 不正解**: 単位が違えば同じ数値でも意味が違います。それがこの事象の原因です。 / Identical numbers mean different things when units differ.

**参照 / Reference:** Technical Fundamentals — SDK の設定値、言語間の差異
</details>

---

### 問題 33 / Question 33

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

1 リクエストで Claude を 20 回呼び出す処理があります。呼び出しは相互に独立しています。現在はスレッドプールで並列化していますが、スレッド数を増やすとメモリ使用量が急増し、それ以上並列度を上げられません。処理は I/O 待ちが支配的です。

A request makes 20 independent calls to Claude. They are currently parallelized with a thread pool, but raising the thread count spikes memory usage and caps further parallelism. The work is dominated by I/O waiting.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 並列化をやめて逐次実行する / Abandon parallelism and run sequentially
- B) **I/O 待ちが支配的な処理では、スレッドごとにスタックを確保する方式より、非同期の並行処理のほうが少ないリソースで高い並行度を実現できる。非同期に切り替え、あわせて同時実行数の上限を設けてレート制限に配慮する** / **For I/O-bound work, asynchronous concurrency achieves higher parallelism with fewer resources than allocating a stack per thread. Switch to async and bound the concurrency to respect rate limits**
- C) メモリを増設する / Add more memory
- D) 20 回の呼び出しを 1 回にまとめる / Collapse the 20 calls into one

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**I/O 待ちが支配的な処理**では、スレッドを増やす方式はスレッドごとのリソース確保が制約になります。非同期の並行処理は、待ち時間の間にリソースを保持し続けないため、同じリソースでより高い並行度を実現できます。ただし並行度を無制限に上げるとレート制限に当たるため、同時実行数の上限を設けることが必要です。処理が CPU 負荷中心であれば結論は変わりますが、本問は I/O 待ちが支配的と明示されています。

For I/O-bound work, the per-thread resource allocation becomes the constraint. Asynchronous concurrency does not hold those resources during the wait and reaches higher parallelism on the same footprint. Concurrency still needs a bound, or rate limits become the next wall. The conclusion would differ for CPU-bound work, but the scenario states I/O dominates.

- **A 不正解**: 逐次実行は所要時間を 20 回分の合計に戻します。 / Restores the sum of all 20 as the elapsed time.
- **C 不正解**: メモリ増設は制約を先送りするだけで、方式の非効率は残ります。 / Defers the limit without addressing the inefficiency.
- **D 不正解**: 独立した 20 のタスクを 1 回にまとめると、それぞれの品質が不安定になります。 / Merging 20 independent tasks destabilizes each.

**参照 / Reference:** Technical Fundamentals — 非同期処理、同時実行制御
</details>

---

### 問題 34 / Question 34

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

ストリーミング応答をブラウザに配信する構成を本番に出したところ、社内ネットワークの一部の利用者で、応答が途中で止まる事象が発生しました。経路にはリバースプロキシとロードバランサが含まれます。

After deploying streaming delivery to browsers, some users on the corporate network see responses stop partway. The path includes a reverse proxy and a load balancer.

**設問 / Question:**

調査と対策で考慮すべき事項を **2 つ選択してください**。

Select **2** considerations for investigation and remediation.

- A) ストリーミングをやめれば根本的に解決する / Abandoning streaming resolves it at the root
- B) ブラウザのバージョンを統一する / Standardize browser versions
- C) **経路上の中間装置がレスポンスをバッファリングしていないか。バッファリングされると逐次配信が働かず、まとめて届くか途中で切られる** / **Whether intermediaries on the path are buffering the response: buffering defeats incremental delivery, so output arrives in bulk or is cut off**
- D) 応答のトークン数を減らす / Reduce the number of response tokens
- E) **中間装置やクライアントのアイドルタイムアウト。生成の合間に無通信の時間が続くと、接続が切断されることがある** / **Idle timeouts on intermediaries and clients: a gap with no traffic during generation can cause the connection to be dropped**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, E**

**解説 / Explanation:**

ストリーミングを本番の経路に載せると、**中間装置の挙動が問題になります**。バッファリングは逐次配信そのものを無効化し、アイドルタイムアウトは生成の合間の無通信時間で接続を切ります。どちらも「一部の利用者だけで起きる」という症状と整合し、経路上の装置の設定に依存します。調査ではまず、直接接続した場合に再現するかを確認して、経路の問題かアプリケーションの問題かを切り分けるのが有効です。

Putting streaming into a production path makes intermediary behavior the issue. Buffering defeats incremental delivery outright, and idle timeouts drop the connection during gaps in generation. Both are consistent with the symptom affecting only some users and both depend on device configuration. A useful first step is checking whether it reproduces on a direct connection, separating a path problem from an application one.

- **A 不正解**: ストリーミングをやめると、初回表示の高速化という目的が失われます。 / Forfeits the purpose of streaming.
- **B 不正解**: 症状は経路依存であり、ブラウザのバージョンでは説明できません。 / Path-dependent, not browser-dependent.
- **D 不正解**: 出力を短くしても、バッファリングやタイムアウトの原因は残ります。 / Neither cause is addressed.

**参照 / Reference:** Technical Fundamentals — ストリーミングと中間装置、websocket / SSE の運用
</details>

---

### 問題 35 / Question 35

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

本番で Claude 呼び出しの一部が遅い、あるいは失敗するという報告があります。アプリケーションのログには「API 呼び出し失敗」としか記録されておらず、どのリクエストが、どのくらいの時間で、どのエラーで失敗したのかが分かりません。

Reports say some Claude calls are slow or failing in production. The application log records only "API call failed," with no indication of which request, how long it took, or which error occurred.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) ログのレベルをデバッグに変更する / Change the log level to debug
- B) 失敗したら自動的に再試行する / Retry automatically on failure
- C) すべてのリクエストとレスポンスを丸ごと記録する / Log every request and response in full
- D) **呼び出しの境界に計装を入れる。リクエスト識別子、所要時間、エラーの種別、使用したモデル、応答の使用量情報を構造化して記録し、集計と検索ができるようにする。個人情報を含み得る本文は記録対象から外すか、統制のもとで扱う** / **Instrument the call boundary: record the request identifier, duration, error class, model used, and the response's usage information in structured form so it can be aggregated and searched — excluding message bodies that may contain personal data, or handling them under controls**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

調査に必要なのは、**構造化された計装**です。所要時間があれば遅い呼び出しを特定でき、エラーの種別があれば原因を分類でき、リクエスト識別子があれば他のログやトレースと突き合わせられます。使用量情報を併せて記録すると、コスト分析にもそのまま使えます。一方、本文を丸ごと記録すると、個人情報がログ基盤に蓄積される問題が生じるため、記録対象から外すか統制のもとで扱う必要があります。デバッグレベルへの変更は、量が増えるだけで構造は得られません。

Investigation needs structured instrumentation: duration identifies slow calls, error class categorizes causes, and a request identifier correlates with other logs and traces. Recording usage information alongside makes the same data serve cost analysis. Logging bodies wholesale, by contrast, accumulates personal data in the logging platform and must be excluded or controlled. Raising the log level adds volume without structure.

- **A 不正解**: 量が増えるだけで、必要な項目が構造化されるわけではありません。 / More volume, no structure.
- **B 不正解**: 再試行は対処であって、原因を知る手段ではありません。 / A remedy, not a diagnostic.
- **C 不正解**: 本文の全記録は、個人情報の蓄積という別の問題を生みます。 / Creates a personal-data accumulation problem.

**参照 / Reference:** Technical Fundamentals — 計装と可観測性、ログの機微データ統制
</details>

---

### 問題 36 / Question 36

> サブスキル / Sub-skill: Technical Fundamentals (6.1%)

**シナリオ / Scenario:**

使用している SDK のメジャーバージョンが上がりました。変更内容には、非推奨だったパラメータの削除、一部メソッドの戻り値の変更、対応する言語バージョンの引き上げが含まれます。現在のコードは旧バージョンに依存しています。

A major version of your SDK has been released, removing deprecated parameters, changing some methods' return values, and raising the minimum language version. Your code depends on the old version.

**設問 / Question:**

最も適切な進め方はどれですか？

Which approach is most appropriate?

- A) **変更点の一覧を確認して影響範囲を特定し、依存する言語バージョンの要件を先に満たしたうえで、変更を分割して適用する。各段階でテストを通し、動作を確認してから次に進む。旧バージョンのサポート期間を確認して計画を立てる** / **Review the change list to identify what is affected, satisfy the language-version requirement first, then apply the changes in stages — running tests and confirming behavior at each step — and plan against the old version's support window**
- B) バージョンを上げずに使い続ける / Stay on the old version indefinitely
- C) すべての変更を一度に適用し、動かなければ元に戻す / Apply everything at once and revert if it does not work
- D) SDK を使わない実装に切り替える / Switch to an implementation without the SDK

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

メジャーバージョンの更新は**破壊的変更を含む計画的な作業**です。最初に変更点の一覧から影響範囲を特定し、前提条件（言語バージョンの要件）を先に満たします。そのうえで変更を分割して適用し、各段階でテストを通せば、問題が起きたときに原因を特定できます。一括適用は、複数の破壊的変更が同時に効いて切り分けが困難になります。サポート期間の確認は、計画の期限を決めるために必要です。

A major version upgrade is planned work containing breaking changes. Identify what is affected from the change list, satisfy the prerequisite language-version requirement first, then apply changes in stages with tests at each step so a failure is attributable. Applying everything at once lets several breaking changes interact and defeats isolation. The support window sets the deadline for the plan.

- **B 不正解**: サポート終了後は、セキュリティ修正も受け取れなくなります。 / Loses security fixes after end of support.
- **C 不正解**: 一括適用は、複数の破壊的変更が絡んで原因の切り分けができません。 / Multiple interacting changes destroy attribution.
- **D 不正解**: SDK の放棄は、定型処理を自前で実装・保守する負担を負う選択です。 / Takes on the boilerplate as your own burden.

**参照 / Reference:** Technical Fundamentals — SDK のバージョン更新
</details>

---

### 問題 37 / Question 37

> サブスキル / Sub-skill: Model Selection and Tradeoffs (2.7%)

**シナリオ / Scenario:**

1 つのアプリケーション内に、性質の異なる 3 つの処理があります。(1) 入力の言語判定（1 日 50 万件、単純）、(2) 問い合わせの要約（1 日 8 万件、中程度）、(3) 契約書の条項リスク分析（1 日 2,000 件、複雑で高い品質が必要）。現在はすべてを同一の上位モデルで処理しており、コストが問題になっています。

One application contains three processes of different character: language detection (500,000/day, simple), inquiry summarization (80,000/day, moderate), and contract clause-risk analysis (2,000/day, complex and quality-critical). All three run on the same strong model, and cost has become a problem.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) すべてを最も軽量なモデルに統一する / Standardize everything on the lightest model
- B) **処理の性質でモデルを使い分ける。単純な言語判定は軽量なモデルに、要約は中間の選択に、条項リスク分析は上位モデルに割り当てる。振り分けは処理の種別から決定的に行い、各層の品質は評価データセットで継続的に確認する** / **Tier models by the nature of each process: a light model for language detection, an intermediate choice for summarization, and the strong model for clause-risk analysis — routing deterministically by process type and continuously verifying each tier's quality against an evaluation set**
- C) すべてを上位モデルのまま維持する / Keep everything on the strong model
- D) 処理ごとにモデルをランダムに選ぶ / Choose a model at random per process

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

難易度と量が明確に分かれている場合、**モデルの階層化はコストと品質を同時に最適化します**。トラフィックの大部分を占める単純な処理を軽量モデルに移せばコストの大半が削減され、少量で高品質が必要な処理には上位モデルを維持できます。振り分けを処理の種別から決定的に行うのが要点で、これにより振り分け自体のコストとレイテンシがゼロに近くなり、挙動も予測可能になります。各層の品質は評価で継続的に確認し、境界が妥当かを検証します。

Where difficulty and volume are clearly stratified, tiering optimizes cost and quality together: moving the high-volume simple work to a light model captures most of the savings while the low-volume quality-critical work keeps the strong one. Routing deterministically by process type keeps the router itself free, fast, and predictable, and per-tier evaluation verifies that the boundaries hold.

- **A 不正解**: 条項リスク分析は品質要件を満たさなくなります。 / Clause-risk analysis would fail its quality bar.
- **C 不正解**: 50 万件の単純な処理に上位モデルを使い続けるのは、最大の削減余地を放置しています。 / Leaves the largest saving untouched.
- **D 不正解**: ランダムな選択は品質もコストも予測不能にします。 / Makes both quality and cost unpredictable.

**参照 / Reference:** Model Selection and Tradeoffs — モデル階層化
</details>

---

### 問題 38 / Question 38

> サブスキル / Sub-skill: Model Selection and Tradeoffs (2.7%)

**シナリオ / Scenario:**

新しいモデルが公開され、公開ベンチマークで高いスコアを出しています。経営層から「すぐに切り替えよう」という指示がありました。自社のシステムは、社内固有の用語を含む保守作業指示書を構造化するもので、公開ベンチマークとはタスクの性質が異なります。

A new model has been released with high public-benchmark scores, and leadership directs an immediate switch. Your system structures maintenance work orders containing proprietary terminology — a task unlike the public benchmarks.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 指示どおり即座に切り替える / Switch immediately as directed
- B) ベンチマークスコアから自社タスクの性能を推定する / Estimate task performance from the benchmark scores
- C) **公開ベンチマークは汎用能力の指標であり、社内固有の用語や出力形式を含むタスクの性能を保証しないことを説明したうえで、既存の評価データセットで新旧を比較する。数日で判断材料が揃うため、その結果をもって切り替えの可否と時期を決めることを提案する** / **Explain that public benchmarks measure general capability and do not guarantee performance on a task with proprietary terminology and output formats, then compare old and new on your existing evaluation set — evidence available in days, on which the decision and its timing should rest**
- D) 切り替えを拒否し、現行モデルを維持する / Refuse the switch and keep the current model

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**公開ベンチマークは汎用能力の指標**であり、社内固有の用語や出力形式を含むタスクでは、順位と実性能が一致しないことが普通にあります。ここでの適切な対応は、指示を拒否することでも盲従することでもなく、**数日で判断材料を用意して意思決定を根拠のあるものにする**ことです。評価データセットが整備されていれば、この提案が現実的な期間で成立します。評価基盤への投資が効いてくる場面でもあります。

Public benchmarks measure general capability, and on a task involving proprietary terminology and formats, ranking and real performance routinely diverge. The right move is neither refusal nor compliance but producing evidence in days so the decision has a basis — feasible precisely because an evaluation set exists. This is where investment in evaluation infrastructure pays.

- **A 不正解**: 根拠のないまま切り替えると、品質が下がる可能性を検証していません。 / Ships without verifying the regression risk.
- **B 不正解**: ベンチマークスコアからタスク性能を推定する妥当な方法は存在しません。 / No valid mapping exists.
- **D 不正解**: 拒否は経営層の関心に応えておらず、実際に改善する可能性も捨てています。 / Ignores a legitimate interest and forfeits a possible gain.

**参照 / Reference:** Model Selection and Tradeoffs — ベンチマークとタスク固有評価
</details>

---

### 問題 39 / Question 39

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

マルチテナント SaaS で、Claude の利用コストが予算を超過しました。財務部門から「どのテナントがいくら使っているか」を示すよう求められましたが、現在は組織全体の合計しか把握できていません。

In a multi-tenant SaaS, Claude spend has exceeded budget. Finance asks which tenants drive it, but only the organization-wide total is currently visible.

**設問 / Question:**

コストを配賦できるようにするために必要な実装を **2 つ選択してください**。

Select **2** implementations needed to make cost attributable.

- A) テナントごとに別々の API キーを発行し、それだけで配賦する / Issue a separate API key per tenant and attribute on that alone
- B) **すべての呼び出しに、テナントと機能を識別する情報を付与して記録する** / **Tag and record every call with information identifying the tenant and the feature**
- C) 月次の請求書を細かく分析する / Analyze the monthly invoice in more detail
- D) **各応答に含まれる使用量情報（入力・出力・キャッシュのトークン数）を、その識別情報とあわせて永続化し、集計できるようにする** / **Persist the usage information from each response — input, output, and cache token counts — alongside those identifiers so it can be aggregated**
- E) テナントごとに利用の上限を設ける / Impose a usage cap per tenant

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, D**

**解説 / Explanation:**

コスト配賦には、**識別情報の付与**と**使用量の記録**の両方が必要です。テナントと機能を識別する情報がなければ集計の軸がなく、使用量の情報がなければ集計する値がありません。両方を呼び出し単位で記録して初めて、「どのテナントのどの機能がいくら使ったか」が分かります。この可視化ができて初めて、値上げ・機能制限・最適化のいずれを行うかを根拠を持って判断できます。上限の設定は、配賦ができるようになった後の施策です。

Attribution needs both identifiers and usage. Without tenant and feature identifiers there is no axis to aggregate on; without usage information there is no value to aggregate. Recording both per call is what answers "which tenant's which feature spent what," and that visibility is the precondition for deciding between repricing, limiting, and optimizing. Caps are a measure that follows attribution, not a substitute for it.

- **A 不正解**: キーの分離だけでは機能別の内訳が得られず、運用負荷も増えます。 / Gives no per-feature breakdown and adds operational burden.
- **C 不正解**: 請求書は組織単位の合計であり、テナント別の内訳を含みません。 / An organization-level total with no tenant breakdown.
- **E 不正解**: 上限は配賦ができてから検討する施策で、可視化の手段ではありません。 / A measure that follows attribution, not a means of achieving it.

**参照 / Reference:** Cost and Token Management — 使用量の追跡、コスト配賦
</details>

---

### 問題 40 / Question 40

> サブスキル / Sub-skill: Cost and Token Management (2.8%)

**シナリオ / Scenario:**

コスト削減の施策として、次の 3 案が挙がっています。(1) 共通接頭辞へのキャッシュ適用（品質影響なし、削減見込み 35%）、(2) 上位モデルから中位モデルへの変更（評価で精度 2.4 ポイント低下、削減見込み 40%）、(3) 取得する文書数を 20 件から 8 件に削減（評価で精度 0.3 ポイント低下、削減見込み 22%）。予算超過分は約 30% です。

Three cost-reduction options are on the table: (1) caching the shared prefix (no quality impact, ~35% saving), (2) moving from the strong model to a mid tier (2.4 points lower on evaluation, ~40% saving), (3) reducing retrieved documents from 20 to 8 (0.3 points lower, ~22% saving). The budget overrun is about 30%.

**設問 / Question:**

最も適切な進め方はどれですか？

Which approach is most appropriate?

- A) **品質への影響がない (1) を先に実施し、効果を実測する。単独で予算超過分を上回る見込みであるため、品質を犠牲にする (2) や (3) の判断はその結果を見てから行う。品質を落とさない手段を尽くしてから、トレードオフを伴う手段を検討する** / **Apply (1) first and measure the realized saving: it alone is projected to exceed the overrun, so decisions on the quality-costing options (2) and (3) can wait for that result. Exhaust the levers that cost no quality before considering those that do**
- B) 削減幅が最大の (2) を実施する / Apply (2), the largest saving
- C) 3 案すべてを同時に実施する / Apply all three simultaneously
- D) 精度低下が小さい (3) のみを実施する / Apply only (3), whose accuracy impact is smallest

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

コスト最適化の順序は、**品質を犠牲にしない手段が先**です。キャッシュの適用は出力の内容に影響せず、しかも単独で予算超過分（30%）を上回る削減が見込めます。この時点で、精度を 2.4 ポイント落とす選択をする理由がありません。3 案の同時実施は、どの施策がどれだけ効いたかを切り分けられず、必要以上に品質を落とす可能性もあります。まず品質影響のない施策を実施して効果を実測し、それでも不足する場合に初めてトレードオフを検討するのが正しい順序です。

Cost optimization starts with levers that cost no quality. Caching does not change the output and is alone projected to exceed the 30% overrun, so there is no reason at this point to accept a 2.4-point accuracy loss. Applying all three at once forfeits attribution and risks giving up more quality than necessary. Apply the free lever, measure the realized saving, and consider tradeoffs only if a gap remains.

- **B 不正解**: 品質影響のない手段を検討せずに、2.4 ポイントの低下を受け入れています。 / Accepts a quality loss before trying the free lever.
- **C 不正解**: 同時実施は効果の切り分けができず、過剰な品質低下を招き得ます。 / No attribution, and likely gives up more quality than needed.
- **D 不正解**: 22% では予算超過分の 30% に届かず、品質も落としています。 / Falls short of the overrun while still costing quality.

**参照 / Reference:** Cost and Token Management — 施策の優先順位、効果の実測
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **前 / Previous:** [`domain2_applications_integration.md`](./domain2_applications_integration.md) | **次 / Next:** [`domain1_agents_workflows.md`](./domain1_agents_workflows.md)
