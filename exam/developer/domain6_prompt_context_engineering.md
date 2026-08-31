# Domain 6: プロンプトとコンテキスト工学 / Prompt and Context Engineering

> 配点比率 / Weight: **11.0%**
> 問題数 / Questions: **26**（基礎 17 / 発展 9）
> 形式 / Format: 単一選択（選択肢 A–D）＋複数選択（選択肢 A–E、選ぶ数を明示）

## サブスキル / Sub-skills

| サブスキル / Sub-skill | Weight | 本ファイル |
|---|---|---|
| Context Engineering | 3.8% | 9 |
| Prompt Engineering | 4.6% | 11 |
| Output Handling | 2.6% | 6 |

## 出題範囲 / Scope（公式ガイドの記述に基づく）

- **Context Engineering** — コンテキストとメモリの管理。コンテキストウィンドウの管理、コンテキストのドリフトと肥大化の防止（ツール出力の刈り込み、圧縮）、サブエージェントや多段のワークフローによるコンテキストの分離 / Context and memory management: window management, preventing context drift and bloat through tool-output pruning and compaction, and context isolation via subagents or multi-step agentic workflows
- **Prompt Engineering** — 指示の明確さ、Few-shot 例、system と user の使い分け、出力の制約、構成要素をまたぐプロンプトと指示の配置、反復的な改善、プロンプトの調整、入力のサニタイズ / Instruction clarity, few-shot examples, system versus user placement, output constraints, prompt and instruction placement across components, iterative refinement, prompt adjustment, and input sanitization
- **Output Handling** — Claude の出力を生成・検証・消費するための確立されたパターン。構造化出力のパターン、応答の検証、防御的なパース、自信のある出力への懐疑 / Established patterns for producing, validating, and consuming output: structured-output patterns, response validation, defensive parsing, and skepticism toward confident output

---

## 基礎 / Foundations level

### 問題 1 / Question 1

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

エージェントがタスクを進めるにつれて、ツールの実行結果がコンテキストに蓄積されていきます。タスク後半では、コンテキストが初期の 10 倍以上になり、レイテンシとコストが増加しています。蓄積された結果の多くは、一度参照された後は使われていません。

As an agent progresses, tool results accumulate in context. Late in a task the context is more than ten times its initial size, with latency and cost rising. Most accumulated results go unused after their first reference.

**設問 / Question:**

この現象と対処についての正しい理解はどれですか？

Which is the correct understanding of this phenomenon and its remedy?

- A) コンテキストは自動的に整理されるため、対処は不要である / Context is tidied automatically; no action is needed
- B) **コンテキストの肥大化と呼ばれる問題で、放置するとコスト・レイテンシの増加に加えて、関連情報が埋没して品質も低下する。不要になったツール出力を刈り込むことで、必要な情報だけを残せる** / **This is context bloat: left alone it raises cost and latency and degrades quality as relevant material gets buried. Pruning tool output that is no longer needed keeps only what matters**
- C) 肥大化はコストの問題だけで、品質には影響しない / Bloat is purely a cost problem and does not affect quality
- D) より大きなコンテキストウィンドウのモデルに変えれば解決する / Moving to a larger context window resolves it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

コンテキストの肥大化は、**コストとレイテンシだけでなく品質にも影響します**。無関係な情報が大量にあると、実際に必要な情報の相対的な比重が下がり、参照されにくくなります。ツールの実行結果は多くが一度使えば役目を終えるため、これを刈り込むことで必要な情報だけを残せます。コンテキストは「使い切るべき容量」ではなく「信号対雑音比を保つべき空間」として扱うのが正しい理解です。

Context bloat affects quality as well as cost and latency: with large volumes of irrelevant material, the relevant material's relative weight falls and it is referenced less reliably. Most tool results are single-use, and pruning them leaves only what matters. The context is not a quota to fill but a space whose signal-to-noise ratio must be maintained.

- **A 不正解**: 自動的に整理される仕組みを前提にすると、肥大化が放置されます。 / Assuming automatic tidying leaves the bloat in place.
- **C 不正解**: 関連情報の埋没により品質も低下します。 / Quality degrades as relevant material is buried.
- **D 不正解**: ウィンドウを広げても蓄積は続き、規模を変えて同じ問題が起きます。 / Accumulation continues at a larger scale.

**参照 / Reference:** Context Engineering — コンテキストの肥大化、ツール出力の刈り込み
</details>

---

### 問題 2 / Question 2

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

長い会話のコンテキストを管理する手法として、「不要になった内容を取り除く」方法と「内容を要約して短くする」方法があります。両者の使い分けを整理しています。

Two techniques manage a long conversation's context: removing content that is no longer needed, and summarizing content to shorten it. You are working out when each applies.

**設問 / Question:**

最も適切な理解はどれですか？

Which is the most appropriate understanding?

- A) 両者は同じ手法の別名である / They are two names for the same technique
- B) 要約のほうが常に優れている / Summarizing is always better
- C) 取り除く方法は品質を損なうので使うべきでない / Removing content harms quality and should not be used
- D) **目的が異なる。一度使えば役目を終える内容（古いツール結果など）は取り除けばよく、要約する必要すらない。一方、会話の流れのように後続で参照される可能性がある内容は、失うと文脈が切れるため要約して残す。対象の性質で使い分ける** / **They serve different purposes. Content that is finished after one use, such as old tool results, can simply be removed — no summarization needed. Content that may be referenced later, such as the flow of the conversation, would break continuity if lost and is summarized instead. Choose by the nature of the content**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

2 つの手法は**目的が異なります**。取り除く方法は「もう使わないものを消す」操作で、古いツール結果のように一度使えば役目を終える内容に適します。要約は「残すが短くする」操作で、会話の流れのように後続で参照される可能性がある内容に適します。両者を同一視して一律の要約で扱うと、消してよいものが残り、逆に保持すべき具体値が抽象化されて失われるという逆の結果になりがちです。

The two techniques serve different purposes. Removal deletes what will not be used again, which suits single-use content such as old tool results. Summarization keeps content in shortened form, which suits material that may still be referenced, such as the conversational flow. Conflating them under one generic summarization tends to invert the outcome: the disposable survives and precise values that should have been kept are abstracted away.

- **A 不正解**: 消す操作と圧縮する操作は別のものです。 / Deleting and compressing are different operations.
- **B 不正解**: 不要な内容を要約するのは無駄で、消せば済みます。 / Summarizing what can simply be deleted is wasted work.
- **C 不正解**: 不要になった内容を取り除くことは品質を損ないません。むしろ信号対雑音比を上げます。 / Removing unneeded content raises signal-to-noise.

**参照 / Reference:** Context Engineering — 刈り込みと圧縮の使い分け
</details>

---

### 問題 3 / Question 3

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

長時間の対話で、コンテキストの管理が適切でない場合に現れる症状を整理しています。

You are cataloguing the symptoms that appear in long conversations when context is poorly managed.

**設問 / Question:**

コンテキストのドリフトや肥大化の症状として適切なものを **2 つ選択してください**。

Select **2** symptoms of context drift or bloat.

- A) **会話の前半で確定した事実（金額、日付、合意事項）が後半で誤って参照される** / **Facts established early in the conversation — amounts, dates, agreements — are misquoted later**
- B) API が認証エラーを返す / The API returns authentication errors
- C) **当初のタスクの目標から外れた応答が増え、無関係な文脈に引きずられる** / **Responses increasingly stray from the original task goal, pulled by unrelated context**
- D) ネットワークのレイテンシが増加する / Network latency increases
- E) API キーが失効する / The API key expires

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, C**

**解説 / Explanation:**

コンテキストの管理不全は、**具体値の劣化**と**目標からの逸脱**として現れます。前者は、要約の過程で「月額 47 万円」が「価格について議論された」のように抽象化され、後半で正確に参照できなくなる現象です。後者は、コンテキストが長くなるにつれて当初の指示や目標が相対的に埋没し、直近の無関係な内容に引きずられる現象です。どちらも、対策として「保持すべき事実の構造化」と「目標の再提示」が有効です。認証やネットワークの問題は、コンテキスト管理とは無関係です。

Poor context management surfaces as degraded specifics and drift from the goal. The first happens when summarization abstracts "¥470,000/month" into "pricing was discussed," leaving it unquotable later. The second happens as the original instruction and goal become relatively buried in a growing context and recent, unrelated material dominates. Structured retention of facts and re-stating the goal address them respectively.

- **B 不正解**: 認証の問題はコンテキストとは無関係です。 / Unrelated to context.
- **D 不正解**: ネットワークのレイテンシは経路の問題です。 / A network-path issue.
- **E 不正解**: キーの失効は資格情報の管理の問題です。 / A credential-management issue.

**参照 / Reference:** Context Engineering — コンテキストのドリフト
</details>

---

### 問題 4 / Question 4

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

1 つのエージェントが、4 つの異なる業務領域の質問を処理しています。各領域の用語定義と注意事項をすべて 1 つのコンテキストに含めた結果、ある領域の質問に別の領域の用語で回答する事象が発生しています。

A single agent handles questions across four business areas, with every area's glossary and caveats in one context. Answers to one area's questions now use another area's terminology.

**設問 / Question:**

最も適切な対処はどれですか？

Which remedy is most appropriate?

- A) **コンテキストを領域ごとに分離する。サブエージェントに分けるか、処理を多段のワークフローにして各段で扱う領域を限定すれば、各コンテキストは短く一貫したものになる** / **Isolate context per area: split into subagents, or make the processing a multi-step workflow whose steps each handle one area, so each context is short and coherent**
- B) システムプロンプトに「領域を混同しないこと」と追記する / Add "do not confuse the areas" to the system prompt
- C) 4 領域の用語を統一する / Unify the terminology across the four areas
- D) より大きなコンテキストウィンドウのモデルを使う / Use a model with a larger context window

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**コンテキストの分離**は、無関係な情報が同居することによる混同を構造的に解消する手法です。実現方法は 2 つあり、サブエージェントに分ける方法と、処理を多段のワークフローにして各段で扱う範囲を限定する方法があります。どちらも、各コンテキストが短く一貫したものになるという点で共通しています。指示を追加する対処は、汚染された同一コンテキスト内で区別の負荷をモデルに押し付けるもので、原因に対処していません。

Context isolation structurally removes the confusion caused by unrelated material coexisting. Two ways achieve it: splitting into subagents, or making the processing a multi-step workflow whose steps each cover a limited scope. Both leave each context short and coherent. Adding an instruction pushes the burden of discrimination onto the model inside the contaminated context and does not address the cause.

- **B 不正解**: 汚染されたコンテキスト内での指示追加は、原因に対処していません。 / Does not address the cause.
- **C 不正解**: 用語の統一は業務上不可能で、各領域の正確性も損ないます。 / Not possible operationally.
- **D 不正解**: ウィンドウの大きさは混同の原因ではありません。 / Window size is not the cause of the confusion.

**参照 / Reference:** Context Engineering — サブエージェントや多段ワークフローによる分離
</details>

---

### 問題 5 / Question 5

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

社内文書 Q&A で、600 件の文書をすべて毎回コンテキストに入れています。1 回の質問で実際に必要な文書は 1〜3 件です。コンテキストが大きく、コストとレイテンシが課題になっています。

An internal document Q&A places all 600 documents in context on every request, though a question typically needs one to three. The large context drives cost and latency.

**設問 / Question:**

最も適切な対処はどれですか？

Which remedy is most appropriate?

- A) 600 件の文書をすべて要約して短くする / Summarize all 600 documents to shorten them
- B) 文書の件数を 100 件に減らす / Reduce the corpus to 100 documents
- C) **質問に関連する文書だけを実行時に取得してコンテキストに入れる。必要な分だけを入れることで、コストとレイテンシが下がり、無関係な情報が減ることで回答品質も改善が見込める** / **Retrieve only the documents relevant to the question at request time: including only what is needed lowers cost and latency, and removing irrelevant material is likely to improve answer quality as well**
- D) 質問を 1 日 10 件までに制限する / Limit questions to ten per day

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

コンテキストに入れる情報は、**必要なものに絞る**のが原則です。1〜3 件で足りる用途に 600 件を入れるのは、コストとレイテンシを払って無関係な情報を大量に持ち込んでいる状態です。関連文書だけを取得すれば、入力量が必要な分に比例し、同時に信号対雑音比が上がって回答品質の改善も期待できます。コスト削減と品質改善が同じ方向に働く、望ましい対処です。

Put in context only what is needed. Sending 600 documents where one to three suffice pays cost and latency to carry large volumes of irrelevant material. Retrieving only the relevant ones makes input scale with need and simultaneously raises signal-to-noise, so cost and quality improve in the same direction.

- **A 不正解**: 要約は情報を落とし、必要な詳細が失われます。無関係な文書を送る構造も変わりません。 / Loses detail and still sends the irrelevant.
- **B 不正解**: 文書を減らすと答えられない質問が生じます。 / Some questions become unanswerable.
- **D 不正解**: 利用の制限は機能の劣化であり、対処になりません。 / Degrades the feature rather than fixing it.

**参照 / Reference:** Context Engineering — コンテキストウィンドウの管理
</details>

---

### 問題 6 / Question 6

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

長い商談支援の会話で、古いターンを自動的に要約して圧縮しています。運用すると、顧客が前半で提示した具体的な金額を後半で誤って参照する、合意済みの条件が失われる、といった問題が報告されています。要約は「これまでの会話の要点」という指示で生成しています。

A long sales-support conversation compacts older turns by summarization. In production, specific figures the customer gave early on are later misquoted and agreed terms are lost. The summary is produced with the instruction "summarize the key points so far."

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 要約の頻度を下げる / Compact less often
- B) 要約を生成するモデルを上位のものに変える / Use a stronger model to produce the summary
- C) 要約をやめて全ターンを保持する / Stop compacting and retain every turn
- D) **圧縮の対象と保持すべき情報を分ける。金額・日付・固有名詞・合意事項といった逐語で保持すべき事実は、要約の外に構造化した形で保持し、要約は会話の流れにのみ適用する** / **Separate what may be compacted from what must be preserved: hold facts that must survive verbatim — figures, dates, proper nouns, agreements — in structured form outside the summary, and apply summarization only to the conversational flow**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

汎用的な要約は、**具体的な値を抽象化する性質**を持ちます（「月額 47 万円」が「価格について議論された」になる）。商談で失われては困る情報を要約に任せると、この抽象化によって失われます。対処は、逐語で保持すべき事実を構造化して要約の外に置き、要約は文脈の連続性を保つ用途に限定することです。「要約は流れを保つのに向き、事実の正確な保持には向かない」という性質の違いを踏まえた分離が要点です。

Generic summarization abstracts precise values away — "¥470,000/month" becomes "pricing was discussed." Facts that must survive should not be entrusted to it. Hold them verbatim in structured form outside the summary and confine summarization to preserving continuity. Summaries preserve flow; structured state preserves truth.

- **A 不正解**: 頻度を下げても、圧縮された時点で同じ損失が起きます。 / The same loss occurs whenever compaction happens.
- **B 不正解**: モデルを上げても、汎用要約が具体値を抽象化する性質は残ります。 / The abstraction is inherent to generic summarization.
- **C 不正解**: 全ターン保持はコンテキスト上限に達するため成立しません。 / The limit is why compaction exists.

**参照 / Reference:** Context Engineering — 圧縮、逐語で保持すべき情報の識別
</details>

---

### 問題 7 / Question 7

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

問い合わせを要約する機能で、プロンプトに「簡潔にまとめてください」と指示しています。出力は、ある時は 1 行、ある時は 10 行と大きくばらつき、含まれる要素も一定しません。下流の処理は一定の形式を前提としています。

A summarization feature instructs the model to "summarize concisely." Output varies widely, sometimes one line and sometimes ten, with inconsistent elements. Downstream processing assumes a fixed shape.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) **「簡潔に」という曖昧な指示を、具体的な要求に置き換える。含めるべき要素（問い合わせの主題、顧客が求めている対応、緊急度）と分量の目安を明示し、可能なら構造化出力で形式を固定する** / **Replace the vague "concisely" with concrete requirements: state which elements must appear — the subject, what the customer is asking for, urgency — and the expected length, and where possible fix the shape with structured output**
- B) 「もっと簡潔に」と強調する / Emphasize "even more concisely"
- C) 温度を下げる / Lower the temperature
- D) 出力の文字数を後から切り詰める / Truncate the output afterwards

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「簡潔に」のような**主観的な形容は、解釈の幅が広く出力がばらつきます**。何を含めるべきかを具体的に列挙し、分量の目安を示せば、解釈の余地が狭まって出力が安定します。下流が一定の形式を前提としているなら、さらに構造化出力で形式そのものを固定するのが確実です。指示の明確さとは、抽象的な形容ではなく検証可能な要求として書くことを意味します。

Subjective adjectives such as "concisely" admit wide interpretation and produce variance. Enumerating what must appear and indicating expected length narrows that interpretation and stabilizes output. Where downstream assumes a fixed shape, structured output fixes it definitively. Instruction clarity means writing verifiable requirements rather than abstract adjectives.

- **B 不正解**: 強調しても曖昧さは残り、ばらつきは解消しません。 / Emphasis does not remove ambiguity.
- **C 不正解**: サンプリングの設定は、指示の曖昧さを補いません。 / Sampling settings do not compensate for vague instructions.
- **D 不正解**: 後から切り詰めると、文の途中で切れて意味が壊れます。 / Truncation breaks the content mid-sentence.

**参照 / Reference:** Prompt Engineering — 指示の明確さ
</details>

---

### 問題 8 / Question 8

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

対話型アプリケーションで、「常に丁寧な敬語で回答する」「回答の最後に関連する参考リンクを付ける」という 2 つの方針を全会話に適用したいと考えています。開発者は、この方針を毎回のユーザーメッセージの末尾に付加する実装をしました。

You want two policies — always answer in polite language, and end each answer with related reference links — applied to every exchange in a conversational application. A developer appends them to the end of every user message.

**設問 / Question:**

最も適切な実装はどれですか？

Which implementation is most appropriate?

- A) 方針をユーザーメッセージの先頭に移す / Move the policies to the start of each user message
- B) 方針をアシスタントの応答に書かせる / Have the assistant restate the policies in its responses
- C) **会話全体に適用される方針はシステムプロンプトに置き、ユーザーメッセージにはそのターン固有の内容だけを入れる。これにより指示の位置づけが明確になり、ユーザー入力との境界も保たれ、毎ターン同じ文言を送る無駄もなくなる** / **Put policies that apply throughout the conversation in the system prompt and leave user messages carrying only what is specific to that turn: the instructions have a clear status, the boundary with user input is preserved, and the same text is not re-sent every turn**
- D) 方針を最初のユーザーメッセージにだけ書く / Include the policies only in the first user message

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**system と user の使い分け**は、指示の適用範囲で決まります。会話全体に一貫して適用される方針はシステムプロンプトに置き、ユーザーメッセージはそのターンの内容に専念させます。ユーザーメッセージに方針を混ぜると、指示と会話の内容が同じ平面に置かれて位置づけが曖昧になり、ユーザー入力との境界も不明瞭になります。加えて、毎ターン同じ文言を送るのはトークンの無駄で、キャッシュの構成上も不利です。

The system/user split follows from the scope of the instruction. Policies that hold throughout the conversation go in the system prompt; user messages carry that turn's content. Mixing policies into user messages puts instructions and content on the same plane, blurring both their status and the boundary with user input — and re-sending the same text every turn wastes tokens and works against a stable cacheable prefix.

- **A 不正解**: 位置を変えても、ユーザーメッセージに指示を混ぜる構造は変わりません。 / The same structure in a different position.
- **B 不正解**: 応答に方針を書かせても、方針の適用は保証されません。 / Restating does not enforce.
- **D 不正解**: 1 回だけでは、長い会話で指示の効きが弱まります。 / Adherence weakens over a long conversation.

**参照 / Reference:** Prompt Engineering — system と user の使い分け
</details>

---

### 問題 9 / Question 9

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

出力の形式を安定させたいと考えています。下流の処理は、決まった項目を持つデータとして出力を消費します。

You want stable output format: downstream processing consumes the output as data with a fixed set of fields.

**設問 / Question:**

出力の制約として適切な手段を **2 つ選択してください**。

Select **2** appropriate ways to constrain the output.

- A) 「必ず正しい形式で出力してください」と強調する / Emphasize "always output in the correct format"
- B) **出力スキーマを定義し、機構として形式を強制する。必須項目と各項目の型・値域を宣言する** / **Define an output schema and enforce the format by mechanism, declaring required fields with their types and value domains**
- C) 出力形式を毎回変えて柔軟性を持たせる / Vary the output format for flexibility
- D) **期待する出力の例を示し、形式が具体的に伝わるようにする** / **Show an example of the expected output so the format is conveyed concretely**
- E) 出力を長くするよう指示する / Instruct the model to produce longer output

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, D**

**解説 / Explanation:**

出力形式の安定化には、**機構による強制**と**例による提示**の 2 つが有効です。前者が主で、スキーマを定義すれば必須項目の欠落や定義外の値が構造的に排除されます。後者は補完的で、スキーマだけでは伝わりにくい内容の粒度や表現の水準を、具体例で示すことができます。プロンプトでの強調は確率的にしか効かず、形式の保証にはなりません。

Two levers stabilize output format: mechanism-level enforcement and demonstration by example. Enforcement is primary — a schema structurally excludes missing required fields and out-of-domain values. Examples are complementary, conveying the granularity and register a schema cannot express. Emphasis in the prompt shifts probabilities and guarantees nothing.

- **A 不正解**: 強調は形式の保証にならず、機構による強制が可能な場面で確率的手段を選んでいます。 / Probabilistic where a mechanism is available.
- **C 不正解**: 形式を変えることは、下流が一定の形式を前提としている要件に反します。 / Contradicts the stated requirement.
- **E 不正解**: 出力の長さは形式の安定性と無関係です。 / Unrelated to format stability.

**参照 / Reference:** Prompt Engineering — 出力の制約、Output Handling — 構造化出力
</details>

---

### 問題 10 / Question 10

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

プロンプトの改善を続けていますが、変更するたびに「良くなった気がする」という感覚に頼っており、実際に改善しているかを確認していません。過去には、改善のつもりで加えた変更が別の面で品質を落としていたことがありました。

Prompt iteration continues, but each change rests on a sense that it "feels better," with no confirmation of actual improvement. On past occasions, a change intended as an improvement degraded quality in another respect.

**設問 / Question:**

最も適切な改善の進め方はどれですか？

Which iteration process is most appropriate?

- A) 変更のたびに開発者が数件試して判断する / Have a developer try a few cases per change and judge
- B) **評価データセットを用意し、変更の前後で測定して比較する。改善したかを数値で確認し、他の面が劣化していないかも同時に見る。感覚ではなく測定を判断の根拠にする** / **Build an evaluation set and measure before and after each change: confirm improvement numerically and check simultaneously that nothing else regressed, making measurement rather than intuition the basis for the decision**
- C) 変更を一度に多く加えて、まとめて効果を確認する / Bundle many changes and check the effect together
- D) 改善をやめて現状のプロンプトを固定する / Stop iterating and freeze the current prompt

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

プロンプトの反復的な改善には、**測定が不可欠**です。数件を目で見て判断する方法は、サンプルが偏り、他の面での劣化に気づけません。評価データセットで変更の前後を測れば、改善したかが数値で分かり、同時に別の指標が下がっていないかも確認できます。「改善したつもりで別の面を落としていた」という過去の経験は、まさに測定していないことの帰結です。変更は一度に 1 つずつ加えると、効果の切り分けができます。

Iterative prompt refinement requires measurement. Judging from a handful of cases samples narrowly and misses regressions elsewhere. Measuring an evaluation set before and after gives a number for the improvement and simultaneously shows whether another metric fell. The team's history of unnoticed regressions is precisely what not measuring produces, and applying one change at a time preserves attribution.

- **A 不正解**: 数件の目視は偏りやすく、他の面の劣化を検出できません。 / Narrow and blind to regressions.
- **C 不正解**: 変更をまとめると、どの変更が効いたか（あるいは害だったか）が分かりません。 / Destroys attribution.
- **D 不正解**: 改善の停止は、測定の仕組みを作れば解決する問題への過剰反応です。 / Overreacts to a measurable problem.

**参照 / Reference:** Prompt Engineering — 反復的な改善
</details>

---

### 問題 11 / Question 11

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

ユーザーが入力した文章を処理する機能で、プロンプトを「以下の文章を分析してください。」＋ ユーザー入力、という形で組み立てています。ユーザー入力の中に指示のような文言が含まれると、それに従ってしまう事象が起きています。

A feature processing user-submitted text assembles its prompt as "Analyze the following text." followed by the user's input. When the input contains instruction-like wording, the model follows it.

**設問 / Question:**

最も適切な対処はどれですか？

Which remedy is most appropriate?

- A) ユーザー入力から命令形の文を削除する / Strip imperative sentences from user input
- B) 機能を廃止する / Remove the feature
- C) ユーザーに指示を書かないよう注意書きを表示する / Display a notice asking users not to write instructions
- D) **ユーザー入力を明示的な区切りで囲み、それが処理対象のデータであって指示ではないことをプロンプトの構造で示す。指示は入力より前に置き、入力の扱い方を先に定義する** / **Wrap the user input in explicit delimiters and state structurally that it is data to be processed rather than instructions, placing the instructions before the input so how to treat it is defined first**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**信頼できない入力と指示が同じ平面に置かれている**のが原因です。入力を明示的な区切りで囲み、「これは分析対象のデータであり指示ではない」と構造で示すと、両者の混同が起きにくくなります。指示を入力より前に置くのも重要で、入力の扱い方が先に定義されている状態を作れます。これはプロンプト設計上の第一層で、より高いリスクを扱う場合は、ツール権限の最小化や決定的なガードレールを重ねる必要があります。

The cause is untrusted input sitting on the same plane as instructions. Delimiting the input and stating structurally that it is data to analyze makes the two much harder to conflate, and placing the instructions first establishes how the input is to be treated before it appears. This is the first layer of prompt design; higher-risk applications add least-privilege tool grants and deterministic guardrails on top.

- **A 不正解**: 命令形の除去は容易に回避され、正当な入力も壊します。 / Trivially bypassed and damages legitimate input.
- **B 不正解**: 設計で解決可能な問題への過剰反応です。 / Disproportionate to a solvable design problem.
- **C 不正解**: 注意書きは悪意ある入力には効きません。 / Ineffective against deliberate input.

**参照 / Reference:** Prompt Engineering — 入力のサニタイズ、データと指示の分離
</details>

---

### 問題 12 / Question 12

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

システムプロンプトが 2 年間の運用で 12,000 トークンに肥大化しました。中身は、役割定義・出力形式に加えて、過去に発生した個別の事象への対処指示が 50 項目以上蓄積しています。最近、基本的な指示（出力形式など）の遵守率が落ちてきました。

Over two years a system prompt has grown to 12,000 tokens: role and output format plus 50-odd instructions accumulated in response to individual past incidents. Adherence to the basic instructions, such as output format, has recently degraded.

**設問 / Question:**

最も適切な対処はどれですか？

Which remedy is most appropriate?

- A) **システムプロンプトを核心部分（役割、判断基準、出力形式）に絞る。個別事象への対処のうち、決定的に扱えるものはコードやツール側に移し、判断を要するものは Few-shot 例に集約する。残す指示は効果を確認したものに限り、以後の追加も同じ基準で審査する** / **Trim the system prompt to its core — role, decision criteria, output format. Move deterministically handleable cases into code or tools, consolidate the judgment cases into few-shot examples, keep only instructions whose effect is confirmed, and apply the same bar to future additions**
- B) 50 項目の対処指示をすべて削除する / Delete all 50 incident-derived instructions
- C) 指示に優先順位の番号を振る / Number the instructions by priority
- D) システムプロンプトを 2 つに分けて 2 回の呼び出しにする / Split the prompt across two calls

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

事象のたびに指示を追加する運用は、**指示の希釈**を招きます。50 の周辺的な指示が核心的な指示と同じ重みで並べば、基本の遵守率が落ちるのは自然な帰結です。対処は 3 方向で、決定的に扱える分岐はコードへ、判断を要するものは Few-shot へ、残りは効果が確認できたものだけを残します。**追加の審査基準を設ける**ことが再肥大化を防ぐ鍵で、これがないと同じ状態に戻ります。

Appending an instruction per incident dilutes the prompt: 50 peripheral rules competing with the core ones is exactly why basic adherence decays. The remedy is three-way — deterministic branches into code, judgment cases into few-shot, and the rest kept only where the effect is confirmed — plus a bar for future additions, without which it re-inflates.

- **B 不正解**: 一括削除は、過去に実際に発生した問題を再発させます。 / Reintroduces previously observed failures.
- **C 不正解**: 番号付けは分量の問題を解決しません。50 項目は番号があっても 50 項目です。 / Does not reduce the volume causing the dilution.
- **D 不正解**: 2 回に分けても総量と希釈の問題は変わらず、コストは増えます。 / Same volume, same dilution, more cost.

**参照 / Reference:** Prompt Engineering — プロンプトの適正規模、指示の希釈
</details>

---

### 問題 13 / Question 13

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

出力に不要な前置きが含まれる問題に対して、プロンプトに「前置きを書かないでください」「挨拶を含めないでください」「余計な説明を加えないでください」という否定形の指示を並べました。前置きは減りましたが、今度は出力が不自然に短くなり、必要な補足まで省かれるようになりました。

To stop unnecessary preambles, the prompt accumulated negative instructions: do not write a preamble, do not include greetings, do not add extra explanation. Preambles decreased, but output became unnaturally terse and necessary detail was omitted.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 否定形の指示をさらに追加する / Add more negative instructions
- B) 否定形の指示をすべて削除する / Delete all the negative instructions
- C) **何を書かないかではなく、何を書くかを指定する。出力に含めるべき要素を列挙し、可能なら構造化出力で形式そのものを固定すれば、前置きは構造的に入り込まず、必要な要素も欠けない** / **Specify what to write rather than what not to write: enumerate the elements the output must contain and, where possible, fix the shape with structured output, so a preamble cannot structurally appear and required elements are not dropped**
- D) 出力から前置きを後処理で削除する / Strip preambles in post-processing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**否定形の指示を重ねると、意図した範囲を超えて出力が抑制されることがあります。**本問の「必要な補足まで省かれる」という症状はその典型です。何を書かないかではなく**何を書くかを肯定形で指定する**と、必要な要素が明示されるため過剰な抑制が起きません。構造化出力で形式を固定できるなら、前置きが入り込む余地そのものがなくなり、指示に頼る必要もなくなります。

Stacking negative instructions can suppress output beyond the intended scope, and the omitted necessary detail is the classic symptom. Specifying positively what the output must contain names the required elements and avoids over-suppression. Where structured output can fix the shape, there is no place for a preamble to appear and no need to instruct against it at all.

- **A 不正解**: 追加すると抑制がさらに強まり、症状が悪化します。 / Deepens the over-suppression.
- **B 不正解**: 削除すると前置きの問題が再発します。指定の仕方を変えるのが解です。 / Preambles return; the fix is how it is specified.
- **D 不正解**: 後処理は形式の揺れに脆く、どこまでが前置きかの判定も曖昧です。 / Brittle, and the boundary is ambiguous.

**参照 / Reference:** Prompt Engineering — 肯定形での指定、出力の制約
</details>

---

### 問題 14 / Question 14

> サブスキル / Sub-skill: Output Handling (2.6%)

**シナリオ / Scenario:**

契約書から契約金額・締結日・当事者名を抽出し、データベースに投入します。現在は自由記述のテキストで返させ、下流で正規表現を使って各項目を切り出しています。表現の揺れによりパースが失敗することがあります。

Contract value, execution date, and party names are extracted from contracts and loaded into a database. Output is returned as free text and parsed downstream with regular expressions, which sometimes fail due to variation in phrasing.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 正規表現のパターンを増やす / Add more regular-expression patterns
- B) **構造化出力にして、項目ごとにフィールドを定義する。金額や日付は型を指定し、必須項目を宣言する。表現の揺れに依存しない形で値を受け取れるため、パースが不要になる** / **Move to structured output with a field per item: type the amount and date, declare required fields, and receive values in a form independent of phrasing, removing the parsing step entirely**
- C) 出力を短くするよう指示する / Instruct the model to produce shorter output
- D) パースに失敗したら空の値を入れる / Insert empty values when parsing fails

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**構造化されたデータを自由記述で受け取り、正規表現で切り出すのが根本原因**です。構造化出力にすれば、項目ごとに値が分かれた形で返るため、パースそのものが不要になります。型の指定により金額や日付の形式も安定し、必須項目の宣言で欠落も防げます。正規表現は、表現の揺れに対して本質的に脆く、パターンを増やしても次の揺れで壊れます。

The root cause is receiving structured data as free text and extracting it with regular expressions. Structured output returns values already separated by field, eliminating parsing. Typing stabilizes amount and date formats and required declarations prevent omissions. Regular expressions are inherently brittle against phrasing variation, and more patterns simply defer the next break.

- **A 不正解**: パターンを増やしても、次の表現差異で壊れます。 / The next variation breaks it again.
- **C 不正解**: 出力を短くしても、構造がない限りパースの脆さは残ります。 / Brittleness persists without structure.
- **D 不正解**: 空の値の投入は、静かなデータ欠損という最悪の障害形態です。 / Silent data loss is the worst failure mode.

**参照 / Reference:** Output Handling — 構造化出力のパターン
</details>

---

### 問題 15 / Question 15

> サブスキル / Sub-skill: Output Handling (2.6%)

**シナリオ / Scenario:**

構造化出力を受け取って処理するコードを書いています。本番では、まれに想定外の内容が返ることがあります。

You are writing code that consumes structured output. Rarely, production returns something unexpected.

**設問 / Question:**

防御的にパースするための実践として適切なものを **2 つ選択してください**。

Select **2** practices for parsing defensively.

- A) パースに失敗したら例外を握りつぶして処理を続ける / Swallow the exception and continue when parsing fails
- B) 想定外の内容が返ったら、空のデータとして扱う / Treat unexpected content as empty data
- C) **パースの前に応答の状態を確認する。出力が途中で切れていないか、想定した形式で返っているかを判別してから、内容の処理に進む** / **Check the response's state before parsing: determine whether the output was truncated and whether it came back in the expected form before proceeding to process it**
- D) すべての値を文字列として扱い、型の検証を行わない / Treat every value as a string and skip type validation
- E) **パース後にスキーマで検証し、必須項目の存在と値域を確認する。検証に失敗したデータは下流に渡さず、隔離して検知できるようにする** / **Validate against the schema after parsing, confirming required fields and value domains, and quarantine anything that fails rather than passing it downstream — in a way that surfaces it**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, E**

**解説 / Explanation:**

防御的なパースは、**処理の前後に確認を挟む**ことです。前の確認では、応答が途中で切れていないか（切り詰め）や想定した形式かを判別します。切り詰めと形式の崩れは別の障害であり、区別することで対処も分かれます。後の確認では、スキーマに照らして必須項目と値域を検証します。重要なのは、検証に失敗したデータを**黙って通したり捨てたりしない**ことで、隔離して検知できる状態にすることで、静かなデータ欠損を防げます。

Defensive parsing means checking before and after. Before, determine whether the output was truncated and whether it has the expected form — truncation and malformation are different failures and separating them separates the remedies. After, validate required fields and value domains against the schema. Critically, data that fails validation is neither passed through nor silently dropped: quarantining it surfaces the problem instead of losing it.

- **A 不正解**: 例外の握りつぶしは、失敗を検知できない状態を作ります。 / Makes failures undetectable.
- **B 不正解**: 空データとして扱うと、静かなデータ欠損になります。 / Produces silent data loss.
- **D 不正解**: 型の検証を省くと、不正な値が下流に流れます。 / Lets invalid values through.

**参照 / Reference:** Output Handling — 防御的なパース、応答の検証
</details>

---

### 問題 16 / Question 16

> サブスキル / Sub-skill: Output Handling (2.6%)

**シナリオ / Scenario:**

Claude の出力を、翌日に動く別のバッチ処理に渡します。バッチが失敗すると、翌々日まで気づきません。出力には、まれに必須項目の欠落や定義外の値が混ざります。

Claude's output feeds a batch process that runs the next day; a failure goes unnoticed for another day. The output occasionally has missing required fields or out-of-domain values.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) バッチ処理を寛容にして、どんな入力でも落ちないようにする / Make the batch lenient so it never fails on any input
- B) 検証せずに渡し、失敗したら翌々日に対応する / Pass everything unvalidated and handle failures two days later
- C) 検証に失敗したレコードを黙って破棄する / Silently discard records that fail validation
- D) **受け渡しの境界で検証する。スキーマに適合しないレコードは下流に渡さず隔離し、その存在を即座に通知する。検知が遅れる構成であるほど、境界での検証の価値が大きい** / **Validate at the handoff boundary: quarantine records that fail the schema instead of passing them, and alert immediately. The slower the detection downstream, the more the boundary check is worth**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

境界での検証は、**壊れたデータが下流に伝播するのを止める**ためにあります。とくに本問のように検知が遅れる構成（翌々日に判明）では、境界で止めることの価値が大きくなります。重要なのは、失敗したレコードを黙って捨てないことです。隔離して即座に通知すれば、データの欠落に気づける状態を保てます。バッチを寛容にする対処は、落ちなくなるだけで、不正な値が処理結果に混ざる問題は残ります。

Boundary validation stops broken data from propagating, and it is worth most where detection downstream is slow — here, two days. The critical part is not silently discarding failures: quarantine plus an immediate alert keeps the loss visible. Making the batch lenient stops it from failing without making it correct.

- **A 不正解**: 落ちないことと正しいことは別で、不正な値が結果に混ざります。 / Not failing is not the same as being correct.
- **B 不正解**: 壊れたデータが下流に入り、検知は 2 日後になります。 / Broken data lands downstream and is found two days late.
- **C 不正解**: 黙って破棄すると、静かなデータ欠損という最悪の障害形態になります。 / Silent data loss is the worst failure mode.

**参照 / Reference:** Output Handling — 応答の検証、境界での検証
</details>

---

### 問題 17 / Question 17

> サブスキル / Sub-skill: Output Handling (2.6%)

**シナリオ / Scenario:**

社内文書に基づいて質問に答える機能で、モデルが明確な口調で回答しています。利用者は、その断定的な表現から回答を確定情報として受け取っています。実際には、文書に根拠がない内容が含まれることがあります。

A feature answers questions from internal documents in a confident tone, and users take the assertive phrasing as established fact. In reality, some content has no basis in the documents.

**設問 / Question:**

最も適切な対処はどれですか？

Which remedy is most appropriate?

- A) **出力の自信の強さを正確性の指標として扱わない設計にする。すべての主張に出典（文書の識別子と該当箇所）の付与を必須とし、出典を付けられない主張は出力しない。出典が原文に実在するかは下流で機械的に検証する** / **Design so that the output's confidence is not treated as an indicator of accuracy: require a citation — document identifier and location — for every assertion, suppress anything that cannot be cited, and verify downstream that the cited text actually exists in the source**
- B) 回答の語調を弱めるよう指示する / Instruct the model to use less assertive phrasing
- C) 回答の末尾に免責文言を付ける / Append a disclaimer to each answer
- D) 利用者に「回答を鵜呑みにしないように」と周知する / Tell users not to take answers at face value

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**出力の自信の強さは、正確性の指標にはなりません。**根拠のない内容も、根拠のある内容と同じ確信を持った表現で生成され得ます。この性質を踏まえると、対処は語調の調整ではなく、**主張と根拠を機械的に結びつけること**になります。出典の付与を必須にすれば、根拠のない主張が構造的に排除され、さらに引用が原文に実在するかを下流で検証すれば、モデルの自己申告に頼らずに確認できます。

The confidence of the output is not an indicator of accuracy: unsupported content is generated with the same assurance as supported content. Given that, the remedy is not adjusting tone but binding assertions to evidence mechanically. Requiring citations suppresses unsupported claims structurally, and verifying downstream that the quoted text actually occurs in the source removes reliance on the model's self-report.

- **B 不正解**: 語調を弱めても、根拠のない内容が含まれる問題は残ります。 / The unsupported content remains.
- **C 不正解**: 免責文言は、提示された内容が正しいという印象を打ち消しません。 / Does not undo the impression of correctness.
- **D 不正解**: 周知は利用者の注意に依存し、根拠のない出力自体を減らしません。 / Depends on user vigilance and does not reduce the output.

**参照 / Reference:** Output Handling — 自信のある出力への懐疑、出典の必須化
</details>

---

## 発展 / Advanced

### 問題 18 / Question 18

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

規制対象の製品仕様書（約 150,000 トークン）に対して、10 項目のコンプライアンス要件それぞれについて「適合・不適合・記載なし」を判定させるアプリケーションを作っています。仕様書全体を 1 リクエストに入れて 10 項目を一度に判定させたところ、末尾に近い項目ほど判定が雑になり、根拠として引用された文言が仕様書に存在しないケースが監査で見つかりました。

You are building an application that judges a regulated product specification (about 150,000 tokens) against ten compliance requirements, returning compliant / non-compliant / not addressed for each. Putting the whole specification in one request and asking for all ten judgments at once produced sloppier judgments toward the end of the list, and an audit found quoted justifications that do not appear anywhere in the specification.

**設問 / Question:**

この設計を改善する最も適切な方法はどれですか？

Which is the most appropriate way to improve this design?

- A) 仕様書を要約してからコンテキストに入れ、要約に対して 10 項目を判定させる / Summarize the specification first and judge all ten requirements against the summary
- B) 温度を下げ、「仕様書にない文言を引用しないこと」という一文を system プロンプトの冒頭に追加する / Lower the temperature and add one line at the top of the system prompt saying not to quote text absent from the specification
- C) **長い文書では指示を文書の前後の両方に置き、10 項目を 1 項目ずつ（あるいは少数ずつ）の呼び出しに分割する。さらに各判定に、仕様書からの逐語引用と該当箇所の位置を必須の出力フィールドとして要求し、返ってきた引用を文字列一致で機械的に検証する** / **Place instructions both before and after the long document, split the ten requirements into one-at-a-time (or few-at-a-time) calls, require each judgment to include a verbatim quotation and its location as mandatory output fields, and mechanically verify the returned quotations by string match**
- D) 仕様書をチャンクに分割してベクトル検索し、各要件に最も近い上位 3 チャンクだけをコンテキストに入れる / Chunk the specification, retrieve the top three chunks per requirement by vector search, and put only those in context

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

症状は 2 つあります。「末尾の項目ほど雑になる」のは、1 回の呼び出しに 10 個の独立した判定タスクを詰め込んだことによる注意の希釈です。「存在しない文言の引用」は根拠が検証されていないことによるものです。それぞれに別の対処が要ります。長文では指示を文書の前後に置くこと、タスクを分割して 1 呼び出しあたりの判断数を減らすこと、そして引用を必須の構造化フィールドにして機械的に照合することで、規制対象の用途に必要な検証可能性が得られます。文字列一致による検証は決定論的なので、監査証跡として成立します。

There are two symptoms. Degradation toward the end of the list comes from packing ten independent judgment tasks into one call, which dilutes attention. Fabricated quotations come from justifications that are never verified. Each needs its own remedy: instructions before *and* after a long document, splitting the task so each call makes fewer judgments, and making the quotation a mandatory structured field that is checked by exact string match. Because the match is deterministic, it stands up as an audit artifact.

- **A 不正解**: 要約は情報を落とします。「記載なし」の判定は、記載がないことを仕様書全体に対して確認して初めて成り立つため、要約に対する判定は規制用途では成立しません / A summary drops information. A "not addressed" verdict requires checking the *whole* specification, so judging against a summary is invalid for a regulated use
- **B 不正解**: プロンプトによる約束は確率的な統制で、監査証跡になりません。温度を下げても、検証されない根拠が正しくなるわけではありません / A promise in the prompt is a probabilistic control and not an audit artifact. Lowering the temperature does not make unverified justifications correct
- **D 不正解**: 上位 3 チャンクに絞ると、該当箇所が検索で拾えなかった場合に「記載なし」と誤判定します。網羅性が要件である判定タスクに、再現率が保証されない検索を挟むのは危険です / Limiting to three chunks yields a false "not addressed" whenever retrieval misses the relevant passage. Inserting retrieval with unguaranteed recall into a task whose requirement is completeness is unsafe

**参照 / Reference:** Context Engineering — 長文での指示配置、タスク分割、根拠の機械的検証
</details>

---

### 問題 19 / Question 19

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

インシデント対応の支援ツールが、(1) 数千行のログから異常箇所を抽出、(2) 抽出結果を既知の障害パターンに分類、(3) 分類に基づく復旧手順の提案、という 3 段階を 1 つの会話で実行しています。段階 3 の提案が、段階 1 で見た生ログの細部に引きずられ、分類結果と食い違うことがあります。

An incident-response assistant runs three stages in a single conversation: (1) extract anomalies from thousands of log lines, (2) classify the extraction against known failure patterns, (3) propose recovery steps based on the classification. Stage 3's proposals sometimes get pulled toward details of the raw logs seen in stage 1 and contradict the classification.

**設問 / Question:**

この問題に対する最も適切なコンテキスト設計はどれですか？

Which context design best addresses this?

- A) **3 段階を別々の呼び出しに分け、各段階には前段の構造化された出力だけを渡す。段階 3 のコンテキストには生ログを入れず、分類結果と、分類が参照した抽出行だけを渡す** / **Split the three stages into separate calls and pass each stage only the previous stage's structured output. Stage 3's context excludes the raw logs, carrying only the classification and the extracted lines it cites**
- B) 段階 3 のプロンプトに「段階 1 の生ログは無視してください」と明記する / Add "ignore the raw logs from stage 1" to stage 3's prompt
- C) 3 段階を 1 呼び出しのまま、生ログを会話の末尾に移動する / Keep one call but move the raw logs to the end of the conversation
- D) 段階 3 だけ別のモデルに切り替える / Switch only stage 3 to a different model

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

これはコンテキストの分離が必要な典型例です。段階ごとに必要な情報は異なり、段階 3 に必要なのは分類結果とその根拠だけです。生ログを持ち込まないことで、混入自体が起こり得なくなります。多段のワークフローとして分割すると、各段階の入力が明示的になり、段階ごとに評価もできます。「分類が参照した抽出行」を残すのは、提案の根拠を追跡可能にするためで、これは生ログ全体を渡すのとは量も性質も異なります。

This is a textbook case for context isolation. Each stage needs different information, and stage 3 needs only the classification and its supporting evidence. Not carrying the raw logs forward makes the contamination structurally impossible. Splitting into a multi-step workflow also makes each stage's input explicit and independently evaluable. Retaining the cited extracted lines keeps the proposal traceable — different in both volume and kind from passing the entire log.

- **B 不正解**: コンテキストに存在する情報を「無視せよ」と指示しても、影響を確実には排除できません。混入させないことと、混入した上で無視させることは同じではありません / Telling the model to ignore information that is present in context does not reliably remove its influence. Not introducing it is not the same as introducing it and asking for it to be ignored
- **C 不正解**: 位置を変えても情報はコンテキストにあり、しかも末尾は注意が向きやすい位置です。改善どころか悪化しかねません / Moving it leaves the information in context, and the end of the context is a position that attracts attention — this may make things worse
- **D 不正解**: モデルを変えても、同じコンテキストを渡す限り同じ混入が起こります。原因はモデルの能力ではなく渡している情報です / A different model receiving the same context suffers the same contamination. The cause is what is passed, not model capability

**参照 / Reference:** Context Engineering — 多段ワークフローによるコンテキストの分離
</details>

---

### 問題 20 / Question 20

> サブスキル / Sub-skill: Context Engineering (3.8%)

**シナリオ / Scenario:**

長時間動作するエージェントで、コンテキストが上限に近づいたときに圧縮（コンパクション）を行います。圧縮後にエージェントが同じツールを再実行したり、すでに確定した事実を再確認したりする退行が観測されています。

A long-running agent compacts its context as it approaches the window limit. After compaction the agent regresses: it re-runs tools it already ran and re-verifies facts that were already settled.

**設問 / Question:**

圧縮の設計として適切なものを **2 つ選択してください**。

Select **2** appropriate compaction design choices.

- A) **圧縮時に、元のゴール（タスク定義と成功条件）を要約せず原文のまま保持する** / **Preserve the original goal — the task definition and success criteria — verbatim rather than summarizing it during compaction**
- B) 会話の古い順に一定割合を機械的に削除する / Mechanically delete a fixed proportion of the conversation, oldest first
- C) 圧縮後に、それまでの全ツール出力を再取得して差し替える / After compaction, re-fetch and replace all previous tool output
- D) 圧縮の頻度を上げ、常にコンテキストを最小に保つ / Compact more frequently to keep context minimal at all times
- E) **確定した事実・決定・未解決の課題を、後から参照できる構造で保持し、それ以外（冗長なツール出力の本文など）を落とす** / **Retain settled facts, decisions, and open questions in a structure that can be referenced later, and drop the rest — such as the bodies of verbose tool output**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, E**

**解説 / Explanation:**

退行の原因は、圧縮が「何を残すか」を決めずに量だけを減らしていることです。A のゴールの原文保持は、圧縮を繰り返すたびに要約の要約が生まれてタスク定義がぼやける（意味のドリフト）のを防ぎます。E は残すべき情報の種類を明示するもので、確定事実を構造化して残せば「すでに確定した事実の再確認」が起こりません。この 2 つはどちらも「量を減らす」ではなく「何を残すか決める」という同じ原則の適用です。

The regression comes from compaction that reduces volume without deciding what to keep. Preserving the goal verbatim (A) prevents summary-of-summary drift blurring the task definition across repeated compactions. Retaining settled facts, decisions, and open questions in referenceable structure (E) is what stops the agent re-verifying what is already settled. Both apply the same principle: decide what to keep, don't just cut.

- **B 不正解**: 古さは重要度の代理指標になりません。序盤に確定した重要な決定ほど古く、最初に消えます / Age is not a proxy for importance. The most important early decisions are the oldest and get deleted first
- **C 不正解**: 再取得はコストとレイテンシを増やすだけでなく、退行そのもの（同じツールの再実行）を制度化してしまいます / Re-fetching adds cost and latency and institutionalizes the very regression in question — re-running the same tools
- **D 不正解**: 頻度を上げても、残す情報を決めていなければ情報の損失が加速するだけです / More frequent compaction without a retention policy only accelerates information loss

**参照 / Reference:** Context Engineering — コンパクション設計、ゴールの保持、意味のドリフト防止
</details>

---

### 問題 21 / Question 21

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

2 年運用してきた分類プロンプトがあります。この間に何度も個別事例への対処が追加され、system プロンプトは「〜の場合は必ず〜と出力すること」という例外規則が 40 行以上並ぶ状態です。より新しいモデルに移行したところ、精度が期待ほど上がらず、一部のカテゴリではむしろ悪化しました。

You have a classification prompt that has been in production for two years. Patches for individual cases were added repeatedly, and the system prompt now contains more than forty lines of exception rules of the form "if X, always output Y." After migrating to a newer model, accuracy improved less than expected and got worse in some categories.

**設問 / Question:**

最も適切な対応はどれですか？

Which is the most appropriate response?

- A) 旧モデルに戻す / Revert to the old model
- B) 例外規則をさらに詳細にし、悪化したカテゴリ向けの規則を追加する / Make the exception rules more detailed and add rules for the categories that got worse
- C) プロンプトはそのままにして、出力を後処理で整える / Leave the prompt as-is and fix the output in post-processing
- D) **プロンプトをモデル移行の一部として見直す。旧モデル向けに積み重ねた回避策や過剰に規定的な指示を棚卸しし、新モデルでは不要になったものを削る。変更は評価データセットで測定しながら段階的に行う** / **Treat the prompt as part of the migration: inventory the workarounds and over-prescriptive instructions accumulated for the old model, remove those the new model no longer needs, and make the changes incrementally, measuring against an evaluation dataset**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

古いプロンプトには、当時のモデルの弱点を補うための回避策が層になって残っています。新しいモデルではその弱点がなくなっている一方、回避策は残り、過剰に規定的な指示としてモデルの判断を縛ります。これがモデル移行で精度が伸びない典型的な原因です。プロンプトはモデルと組で設計されるものなので、モデルを変えたらプロンプトも見直す対象になります。ただし 40 行を一度に消すと何が効いていたか分からなくなるため、評価データセットで測定しながら段階的に削るのが正しい進め方です。

An old prompt carries layers of workarounds for the weaknesses of the model of the day. The newer model no longer has those weaknesses, but the workarounds remain as over-prescriptive instructions that constrain its judgment — a classic reason migrations under-deliver. A prompt is designed jointly with a model, so changing the model puts the prompt in scope. Deleting all forty lines at once would destroy the record of what was actually load-bearing, so remove them incrementally while measuring against an evaluation set.

- **A 不正解**: 移行の目的を放棄しています。原因はモデルではなくモデルに合っていないプロンプトです / This abandons the purpose of the migration. The cause is a prompt that no longer fits, not the model
- **B 不正解**: 問題の原因である規則の積み重ねを増やす方向で、悪化を加速させます / This adds to the accumulation that caused the problem and accelerates the degradation
- **C 不正解**: 後処理は症状を隠すだけで、分類そのものの精度は変わりません。分類の誤りは後処理では検出できません / Post-processing hides the symptom without improving classification accuracy; a misclassification is not detectable downstream

**参照 / Reference:** Prompt Engineering — モデル移行に伴うプロンプトの見直し、過剰に規定的な指示の棚卸し
</details>

---

### 問題 22 / Question 22

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

社内エージェントには、(a) 役割と一般的な方針を書いた system プロンプト、(b) 各ツールの `description`、(c) リポジトリごとの `CLAUDE.md` があります。「削除系のツールを使う前に必ず対象件数を確認して報告する」という規則を、どこに書くべきか設計チームで議論になっています。この規則は全リポジトリ共通で、削除系ツールにのみ適用されます。

An internal agent has (a) a system prompt with its role and general policy, (b) a `description` on each tool, and (c) a per-repository `CLAUDE.md`. The design team is debating where to put the rule "before using any deletion tool, always confirm and report the number of affected records." The rule is common to all repositories and applies only to deletion tools.

**設問 / Question:**

指示の配置として最も適切なものはどれですか？

Which placement of the instruction is most appropriate?

- A) `CLAUDE.md` に書く。運用ルールはリポジトリの文書に置くのが自然だから / Put it in `CLAUDE.md`, since operational rules naturally belong in repository documentation
- B) **削除系ツールの `description` に、使用条件として書く。適用範囲がそのツールに限定されており、モデルがツールを選ぶ時点で必ず読む位置だから。あわせて、不可逆な操作である以上、確認をプロンプトの約束だけに頼らず、ツール側でも件数の返却と確認ステップを強制する** / **Put it in each deletion tool's `description` as a usage condition: the scope is limited to that tool, and it is the text the model necessarily reads when choosing the tool. Additionally, because the operation is irreversible, do not rely on a prompt-level promise alone — enforce the count return and confirmation step in the tool itself**
- C) 3 か所すべてに同じ文言を書く。重複させるほど従う確率が上がるから / Put the same wording in all three places, since duplication raises compliance
- D) system プロンプトに書く。最も優先度が高い指示だから / Put it in the system prompt, since it is the highest-priority instruction

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

指示は「適用範囲と一致する場所」に置きます。この規則は削除系ツールにのみ適用されるので、ツールの `description` が正しい位置です。`description` はモデルがそのツールを選択・使用する判断をする、まさにその時点で読まれます。さらに重要なのは後半で、不可逆な操作の統制をプロンプトの約束（確率的統制）に委ねてはいけないという点です。件数の返却と確認をツールの実装で強制すれば、モデルが指示に従わなかった場合でも安全側に倒れます。

Place an instruction where its scope lives. This rule applies only to deletion tools, so the tool `description` is the right location — it is read exactly when the model is deciding whether and how to use that tool. The second half matters more: control over an irreversible operation must not rest on a prompt-level promise, which is a probabilistic control. Enforcing the count return and confirmation in the tool implementation keeps the system safe even when the model does not follow the instruction.

- **A 不正解**: `CLAUDE.md` はリポジトリ固有の文脈のための場所です。全リポジトリ共通の規則を各リポジトリに書くと、更新漏れで規則が食い違います / `CLAUDE.md` is for repository-specific context. Putting a rule common to all repositories in each one guarantees they drift apart when it is updated
- **C 不正解**: 3 か所に散らすと更新時の不整合が生まれ、どれが正なのか判断できなくなります。重複は遵守率ではなく保守負債を増やします / Scattering it across three places creates inconsistency on update and no clear source of truth. Duplication buys maintenance debt, not compliance
- **D 不正解**: system プロンプトは常に読まれるため、特定ツールにのみ適用される条件を置くと、無関係な場面でも文脈を占有します。優先度の高さは配置の理由になりません / The system prompt is always in context, so a condition scoped to one tool occupies context in every unrelated situation. Importance is not a placement criterion

**参照 / Reference:** Prompt Engineering — 構成要素をまたぐ指示の配置、決定論的統制の優先
</details>

---

### 問題 23 / Question 23

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

チームでプロンプトの改善サイクルを回していますが、「良くなった気がする」で変更を入れてしまい、数週間後に精度が下がっていることに気づく、ということを繰り返しています。プロンプトは複数人が編集し、変更履歴とデプロイの対応も曖昧です。

Your team iterates on prompts, but changes keep going in on the strength of "it feels better," and weeks later you discover accuracy has dropped. Several people edit the prompt, and the mapping between edits and deployments is unclear.

**設問 / Question:**

反復的な改善を規律あるものにするために有効な取り組みを **2 つ選択してください**。

Select **2** practices that make iterative refinement disciplined.

- A) プロンプトを 1 人の担当者だけが編集できるようにする / Restrict prompt edits to a single owner
- B) 変更のたびに、より強い表現（「必ず」「絶対に」）へ書き換える / Rewrite toward stronger wording ("always," "never") with each change
- C) **代表的な入力と期待される振る舞いからなる評価データセットを用意し、プロンプトの変更前後で同じデータセットに対して測定する。失敗例が見つかるたびにデータセットへ追加する** / **Maintain an evaluation dataset of representative inputs and expected behavior, measure the same dataset before and after each prompt change, and add every newly found failure case to it**
- D) **プロンプトをアプリケーションコードと同じようにバージョン管理し、変更をレビューの対象にして、どのバージョンがどの環境で動いているかを追跡できるようにする** / **Version-control the prompt like application code, put changes through review, and track which version is running in which environment**
- E) 変更は本番でのみ検証し、影響が出たら戻す / Validate changes only in production and roll back if impact appears

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, D**

**解説 / Explanation:**

「良くなった気がする」を排除するには、測定（C）と追跡（D）の 2 つが要ります。評価データセットは変更の効果を主観から切り離し、失敗例を追加していくことで回帰テストとしても機能します。バージョン管理とレビューは、誰がいつ何を変えたかと、どのバージョンが本番にあるかを対応づけ、精度低下を特定の変更まで遡れるようにします。プロンプトは振る舞いを決める本番アーティファクトなので、コードと同じ構成管理の下に置きます。

Eliminating "it feels better" needs two things: measurement (C) and traceability (D). An evaluation dataset separates the effect of a change from subjective impression, and grows into a regression suite as failure cases are added. Version control with review ties who changed what to which version is in production, so a drop in accuracy can be traced back to a specific change. A prompt is a production artifact that determines behavior, so it belongs under the same configuration management as code.

- **A 不正解**: 単独所有者はボトルネックを作るだけで、変更が良かったかどうかは依然として測定されません / A single owner creates a bottleneck and still leaves changes unmeasured
- **B 不正解**: 表現を強めることは改善の手法ではありません。強い指示の積み重ねは、問題 21 の過剰に規定的なプロンプトそのものです / Stronger wording is not a refinement method; accumulating emphatic instructions produces exactly the over-prescriptive prompt of Question 21
- **E 不正解**: 本番のみでの検証はユーザーに影響が出てから気づくことを意味し、まさに現状の失敗パターンです / Validating only in production means noticing after users are affected — precisely the failure pattern described

**参照 / Reference:** Prompt Engineering — 反復的な改善、評価データセット、プロンプトの構成管理
</details>

---

### 問題 24 / Question 24

> サブスキル / Sub-skill: Prompt Engineering (4.6%)

**シナリオ / Scenario:**

サポート回答の生成プロンプトについて、現行版（A）と改訂版（B）のどちらが優れているかを判断したいと考えています。オフライン評価では B がわずかに上回りましたが、差は小さく、実ユーザーの満足度に効くかは分かりません。1 日あたりの対象トラフィックは十分にあります。

You want to decide whether the current prompt (A) or a revision (B) is better for generating support replies. Offline evaluation gives B a slight edge, but the margin is small and it is unclear whether it moves real user satisfaction. Daily traffic on the affected path is ample.

**設問 / Question:**

最も適切な進め方はどれですか？

Which is the most appropriate approach?

- A) オフライン評価で B が上回ったのだから、そのまま B に全面切り替えする / Since B won offline, switch fully to B
- B) 判断がつかないので、A と B の文言を混ぜた C を作る / Since it is a toss-up, create a C that blends the wording of A and B
- C) **トラフィックを分割して A と B を並行運用し、事前に決めた指標（解決率・エスカレーション率など）で有意差が出るまで測定する。プロンプト以外の条件（モデル、ツール、対象ユーザー層）は揃える** / **Split traffic and run A and B in parallel, measuring against metrics agreed in advance — resolution rate, escalation rate — until the difference is significant. Hold everything else constant: model, tools, and user segment**
- D) 社内メンバーに両方の出力を見せて多数決を取る / Show both outputs to internal staff and take a vote

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

オフライン評価で差が小さいときこそ、実トラフィックでの A/B テストが要ります。重要なのは、指標を事前に決めること（後から都合の良い指標を選ぶと、どちらでも勝たせられます）と、プロンプト以外の条件を揃えることです。モデルやツール構成が同時に変わると、差がどちらに由来するか分からなくなります。トラフィックが十分あるという前提は、有意差を検出できる条件が揃っていることを意味します。

When the offline margin is small, an A/B test on real traffic is exactly what is needed. Two things matter: agreeing the metrics in advance (choosing them afterward lets you declare either winner), and holding everything but the prompt constant. If model or tool configuration changes at the same time, the difference cannot be attributed. Ample traffic is what makes detecting a significant difference feasible.

- **A 不正解**: 小さいオフライン差は実運用での優位を意味しません。オフライン評価は実ユーザーの分布や満足度を完全には代表しません / A small offline margin does not imply an advantage in production. Offline evaluation does not fully represent real user distribution or satisfaction
- **B 不正解**: 混ぜたものは A でも B でもない第 3 の未評価プロンプトで、どちらの評価結果も適用できません / A blend is a third, unevaluated prompt to which neither set of results applies
- **D 不正解**: 社内の多数決は実ユーザーの成果指標の代理になりません。特に「解決したか」は社内では判定できません / An internal vote is not a proxy for real user outcomes — whether an issue was actually resolved cannot be judged internally

**参照 / Reference:** Prompt Engineering — A/B テスト、事前に定めた指標、変数の統制
</details>

---

### 問題 25 / Question 25

> サブスキル / Sub-skill: Output Handling (2.6%)

**シナリオ / Scenario:**

日次バッチで 50,000 件の文書からメタデータを構造化抽出しています。全体の約 0.3%（150 件前後）でスキーマ検証に失敗するか、必須フィールドが業務上ありえない値になります。現在の実装は検証失敗時に例外を投げ、バッチ全体が停止します。抽出結果は下流の請求処理に流れます。

A daily batch extracts structured metadata from 50,000 documents. About 0.3% (roughly 150 records) either fail schema validation or carry required fields with values that are impossible for the business. The current implementation throws on validation failure, halting the whole batch. Extraction results flow downstream into billing.

**設問 / Question:**

最も適切な出力処理の設計はどれですか？

Which is the most appropriate output-handling design?

- A) スキーマ検証を緩め、型が合わない値は文字列として通す / Loosen schema validation and let type-mismatched values through as strings
- B) 検証に失敗した文書だけを自動で無制限に再試行し、成功するまで繰り返す / Automatically retry only the failing documents, without limit, until they succeed
- C) 検証失敗をログに記録して当該レコードをスキップし、残りをそのまま下流に流す / Log the failure, skip the record, and pass the rest downstream unchanged
- D) **失敗したレコードを隔離キューに退避してバッチは継続し、成功分のみ下流に渡す。隔離分は有限回の再試行後に人間のレビュー対象とし、隔離件数と理由の内訳を監視して閾値超過でアラートを出す。業務上ありえない値は、スキーマ検証とは別に業務ルール検証として明示的に検出する** / **Quarantine failing records and continue the batch, passing only successes downstream. After a bounded number of retries, route the quarantine to human review; monitor quarantine volume and reason breakdown, alerting when a threshold is exceeded. Detect business-impossible values explicitly as business-rule validation, separate from schema validation**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

大規模バッチでは、一部の失敗が全体を止めない設計が要ります。同時に、失われたレコードが見えなくなってもいけません。隔離キューは「継続する」と「失われない」を両立させ、人間のレビュー経路があることで請求処理という不可逆な下流に対する統制になります。件数と理由の監視は、0.3% が 3% に増えたときに気づくための仕組みです。もう 1 点重要なのは、スキーマ検証と業務ルール検証を分けることです。「型は正しいが業務上ありえない」値はスキーマでは捕まらないため、別の検証層が必要です。

A large batch needs a design where partial failure does not stop the whole run — and where dropped records do not become invisible. A quarantine queue satisfies both, and the human review path is the control over an irreversible downstream process like billing. Monitoring volume and reason breakdown is how you notice when 0.3% becomes 3%. The other key point is separating schema validation from business-rule validation: a value that is well-typed but impossible for the business will never be caught by a schema, so it needs its own layer.

- **A 不正解**: 検証を緩めることは、不正な値を検出せずに請求処理へ流すことを意味します。防御的パースの目的と正反対です / Loosening validation means passing invalid values into billing undetected — the opposite of defensive parsing
- **B 不正解**: 無制限の再試行は、決定論的に失敗する入力（文書自体が不完全な場合など）に対して無限に回り、バッチを止めます / Unbounded retries spin forever on deterministically failing input — an incomplete document, say — and stall the batch
- **C 不正解**: ログだけではレコードが実質的に失われます。請求処理では、抽出できなかったことが検知・回収されないまま進むのは許容できません / Logging alone effectively loses the record. In billing, an extraction failure that is never surfaced or recovered is not acceptable

**参照 / Reference:** Output Handling — 応答の検証、防御的パース、隔離と人間のレビュー
</details>

---

### 問題 26 / Question 26

> サブスキル / Sub-skill: Output Handling (2.6%)

**シナリオ / Scenario:**

社内ナレッジベースに対する質問応答アプリで、Claude は常に自信のある断定的な文体で回答します。利用者からは「もっともらしいが実際には社内文書に書かれていない内容が混じることがある」という報告が上がっています。文体からは、どこまでが文書に基づいているのか区別がつきません。

In a Q&A application over an internal knowledge base, Claude always answers in a confident, assertive tone. Users report that plausible-sounding content sometimes turns out not to be in the internal documents at all. The tone gives no signal about which parts are grounded in the documents.

**設問 / Question:**

この問題への最も適切な対処はどれですか？

Which is the most appropriate remedy?

- A) 回答の冒頭に「この回答は誤っている可能性があります」という定型文を付ける / Prepend a boilerplate disclaimer saying the answer may be incorrect
- B) **回答の各主張に、根拠となった文書の識別子と該当箇所を構造化された形で必ず添えさせ、参照先が実在すること・引用文が原文と一致することをアプリ側で検証する。文書に根拠がない場合は「該当なし」と返す経路を明示的に用意する** / **Require every claim to carry the source document identifier and passage in structured form, verify application-side that the reference exists and the quoted text matches the original, and provide an explicit path to return "not found" when the documents do not support an answer**
- C) 温度を 0 にして、断定的な文体を抑える / Set temperature to 0 to suppress the assertive tone
- D) 回答の末尾に、モデル自身が申告した確信度スコアを表示する / Append a confidence score that the model reports for itself

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

「自信のある出力への懐疑」が求められるのは、文体が正しさの指標にならないからです。利用者に懐疑を促すのではなく、検証可能にすることが対処になります。主張ごとに出典を構造化して要求すれば、アプリ側で「その文書 ID は存在するか」「引用文は原文と一致するか」を機械的に検証できます。もう 1 点重要なのが「該当なし」の経路です。根拠がないときに返す先が用意されていないと、モデルは何かを答えようとして埋め合わせます。

Skepticism toward confident output is required precisely because tone is not a signal of correctness. The fix is not to ask users to be skeptical but to make the output verifiable. Requiring a structured citation per claim lets the application mechanically check that the document ID exists and that the quotation matches the source. The explicit "not found" path matters just as much: with nowhere to land when there is no support, the model fills the gap.

- **A 不正解**: 定型的な免責は、どの部分が根拠を欠くのかを示しません。全体に付く警告は無視されるようになるだけです / A blanket disclaimer does not identify which part lacks grounding, and a warning attached to everything is soon ignored
- **C 不正解**: 温度は出力の多様性に影響しますが、根拠のない内容が出ること自体は防げません。文体の問題でもありません / Temperature affects output diversity but does not prevent ungrounded content, and the issue is not stylistic
- **D 不正解**: モデルの自己申告した確信度は、根拠の有無とは独立に高く出ることがあります。検証されていない自己申告を根拠にするのは、問題の言い換えにすぎません / Self-reported confidence can be high independently of whether grounding exists. Basing trust on an unverified self-report merely restates the problem

**参照 / Reference:** Output Handling — 自信のある出力への懐疑、出典の構造化と機械的検証
</details>

---
