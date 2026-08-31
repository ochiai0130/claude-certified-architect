# Domain 1: エージェントとワークフロー / Agents and Workflows

> 配点比率 / Weight: **14.7%**（全 8 ドメイン中 3 番目）
> 問題数 / Questions: **35**（基礎 23 / 発展 12）
> 形式 / Format: 単一選択（選択肢 A–D）＋複数選択（選択肢 A–E、選ぶ数を明示）

## サブスキル / Sub-skills

| サブスキル / Sub-skill | Weight | 本ファイル |
|---|---|---|
| Agent Architecture | 4.5% | 11 |
| Agent Construction with Claude | 5.3% | 13 |
| Agent Patterns and Frameworks | 4.9% | 11 |

## 出題範囲 / Scope（公式ガイドの記述に基づく）

- **Agent Architecture** — エージェントとワークフローのアーキテクチャの原則・パターン・トレードオフ。ワークフローとエージェントのどちらを使うかの判断基準、マネージャー／スーパーバイザー階層の構造、タスク実行を改善するサブエージェントの役割 / Principles, patterns, and tradeoffs of agent and workflow architecture, including the criteria for choosing a workflow versus an agent, the structure of manager/supervisor hierarchies, and the role of subagents
- **Agent Construction with Claude** — Claude のエージェントを構築する方法・ツール・プラットフォーム。Claude Agent SDK、独自のエージェントループとハーネス、マネージド／自己ホストのデプロイ形態、決定的な動作のためのフック / Methods, tools, and platforms for constructing Claude agents: the Claude Agent SDK, custom agent loops and harnesses, managed versus self-hosted deployment, and hooks for deterministic actions
- **Agent Patterns and Frameworks** — 一般的なエージェント設計パターン（ツール使用ループ、サブエージェント、メモリ、コンテキストウィンドウ管理）と、多段タスクのためのエージェント抽象化フレームワーク / Common agent design patterns (tool-use loops, sub-agents, memory, context-window management) and agentic abstraction frameworks for multi-step tasks

---

## 基礎 / Foundations level

### 問題 1 / Question 1

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

新しい機能について、固定のワークフローとして実装するか、エージェントとして実装するかを判断する必要があります。

You must decide whether to implement a new feature as a fixed workflow or as an agent.

**設問 / Question:**

判断基準として最も適切なものはどれですか？

Which is the most appropriate criterion?

- A) エージェントのほうが新しい技術なので、常にエージェントを選ぶ / Agents are the newer technology, so always choose an agent
- B) 実装が簡単なほうを選ぶ / Choose whichever is easier to implement
- C) **処理の手順を事前に確定できるかで判断する。手順が確定しており毎回同じ順序でよいなら固定のワークフロー、必要な手順が入力によって変わり事前に列挙できないならエージェントが適合する** / **Decide by whether the steps can be determined in advance: a fixed workflow when the sequence is known and identical each time, an agent when the necessary steps vary with the input and cannot be enumerated beforehand**
- D) コストが安いほうを選ぶ / Choose whichever costs less

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**ワークフローとエージェントの本質的な違いは、制御フローを誰が決めるか**です。手順が事前に確定しているなら、その順序をコードで書くほうが予測可能で、テストしやすく、コストも低くなります。一方、必要な手順が入力によって変わり、事前に列挙できない場合は、モデルに次の行動を決めさせるエージェントが適合します。この判断軸は、実装の難易度やコストより先に来るもので、それらは選択の結果として付いてくる性質です。

The essential difference is who determines the control flow. When the sequence is fixed in advance, writing it in code is more predictable, easier to test, and cheaper. When the necessary steps vary with the input and cannot be enumerated, letting the model decide the next action is the fit. This criterion comes before implementation difficulty and cost, which follow from the choice rather than determining it.

- **A 不正解**: 新しさは適合性の判断基準になりません。 / Novelty is not a fitness criterion.
- **B 不正解**: 実装の容易さは、要件への適合とは別の軸です。 / Ease of implementation is a separate axis from fit.
- **D 不正解**: コストは重要ですが、まず要件に適合する方式を選んだうえで最適化する対象です。 / Important, but optimized after choosing a fitting approach.

**参照 / Reference:** Agent Architecture — ワークフローとエージェントの判断基準
</details>

---

### 問題 2 / Question 2

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

金融機関の口座開設処理を実装します。処理は法令により「本人確認 → 反社チェック → 与信照会 → 口座生成」の順で必ず実施する必要があり、順序の入れ替えや省略は監督官庁への説明が不可能です。各ステップの入出力は保存義務があります。

You are implementing account opening at a financial institution. Regulation requires the sequence identity verification → sanctions screening → credit check → account creation, always in that order; reordering or skipping cannot be defended to the regulator, and each step's inputs and outputs must be retained.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) **コードで記述した固定のワークフローとして実装する。順序が規制で決まっている以上、モデルの判断に委ねる余地がなく、コードで書けば順序違反が構造的に起こり得ない。各ステップの境界で入出力を永続化する** / **Implement it as a fixed, code-defined workflow: with the order fixed by regulation there is nothing to delegate to model judgment, and encoding it in code makes an ordering violation structurally impossible, with inputs and outputs persisted at each boundary**
- B) エージェントに 4 つのツールを与え、システムプロンプトで順序を指示する / Give an agent four tools and instruct the order in the system prompt
- C) 4 つのステップを並列に実行する / Execute the four steps in parallel
- D) エージェントが状況に応じて順序を決める構成にする / Let an agent decide the order adaptively

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**規制で順序が固定されている制御フローは、モデルの判断に委ねてはならない領域**です。コードで順序を記述すれば、順序違反は構造的に起こり得ず、各境界での永続化がそのまま監査証跡になります。プロンプトによる順序の指示は確率的で、「順序は必ず守られる」と監督官庁に説明できません。この用途にはエージェントの柔軟性が価値を持たず、むしろ違反のリスク要因になります。

Control flow fixed by regulation must not be delegated to model judgment. Encoding the order in code makes a violation structurally impossible, and persisting at each boundary yields the audit trail directly. A prompt-level instruction is probabilistic and cannot be defended as a guarantee. Agentic flexibility carries no value here and is purely a source of risk.

- **B 不正解**: プロンプトによる順序保証は確率的で、規制上の説明責任を果たせません。 / Probabilistic; not defensible to a regulator.
- **C 不正解**: 並列実行は順序要件に真っ向から反します。 / Directly violates the ordering requirement.
- **D 不正解**: 順序が固定されている以上、動的な判断は違反の原因にしかなりません。 / With the order fixed, adaptivity is only a source of violations.

**参照 / Reference:** Agent Architecture — 固定ワークフローの適合条件
</details>

---

### 問題 3 / Question 3

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

エージェントとして実装すべきかを検討しています。

You are assessing whether a task warrants an agent.

**設問 / Question:**

エージェントの採用を正当化する条件を **2 つ選択してください**。

Select **2** conditions that justify choosing an agent.

- A) 処理の手順が完全に決まっている / The steps are entirely fixed
- B) **入力によって必要な手順や回数が変わり、事前に列挙できない** / **The necessary steps and their number vary with the input and cannot be enumerated in advance**
- C) 最新の技術を採用したい / You want to adopt the newest technology
- D) **途中の結果を見て次の行動を決める必要があり、その判断を事前にルール化できない** / **The next action must be chosen based on intermediate results, and that judgment cannot be reduced to rules in advance**
- E) コストを最小にしたい / You want to minimize cost

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, D**

**解説 / Explanation:**

エージェントが正当化されるのは、**制御フローを事前に決められない場合**です。必要な手順が入力によって変わる、途中の結果を見て次を判断する必要がある、という 2 つはいずれもこの性質を表しています。逆に、手順が決まっているならワークフローのほうが予測可能で安価です。エージェントは柔軟性と引き換えに、コスト・レイテンシ・予測可能性を犠牲にするため、その柔軟性が実際に必要な場合にのみ採用します。

An agent is justified when the control flow cannot be determined in advance. Steps that vary with the input, and next actions that depend on intermediate results, are both expressions of that property. Where the sequence is known, a workflow is more predictable and cheaper. An agent trades cost, latency, and predictability for flexibility, so adopt it only where that flexibility is genuinely needed.

- **A 不正解**: 手順が決まっているならワークフローが適合し、エージェントは過剰です。 / Fixed steps point to a workflow.
- **C 不正解**: 技術の新しさは適合性の根拠になりません。 / Novelty is not a justification.
- **E 不正解**: エージェントは一般にワークフローよりコストが高くなります。 / Agents generally cost more than workflows.

**参照 / Reference:** Agent Architecture — エージェントの適合条件
</details>

---

### 問題 4 / Question 4

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

複数の専門サブエージェント（文献調査、データ分析、レポート生成）を束ねる構成を設計しています。上位のコーディネーターの役割を定義する必要があります。

You are designing a system that coordinates several specialist subagents — literature research, data analysis, report generation — and must define the coordinator's role.

**設問 / Question:**

コーディネーターの役割として最も適切なものはどれですか？

Which best describes the coordinator's role?

- A) すべての実作業を自分で行い、サブエージェントは使わない / Do all the work itself and not use the subagents
- B) サブエージェントの出力をそのままユーザーに転送する / Forward the subagents' output to the user unchanged
- C) サブエージェントと同じ作業を並行して行い、結果を比較する / Perform the same work in parallel and compare results
- D) **タスクを分解してどのサブエージェントに何を委譲するかを決め、各サブエージェントに必要な情報だけを渡し、返ってきた結果を統合して全体の目標に対する進捗を判断する** / **Decompose the task, decide what to delegate to which subagent, pass each only the information it needs, integrate the returned results, and judge progress against the overall goal**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

マネージャー／スーパーバイザー階層におけるコーディネーターの役割は、**分解・委譲・統合・進捗判断**です。とくに重要なのが「各サブエージェントに必要な情報だけを渡す」点で、これによりサブエージェント側のコンテキストが小さく保たれ、専門タスクへの集中が生まれます。全体をコーディネーターが把握し、サブエージェントは局所的な作業に専念するという役割分担が、この構成の価値の源泉です。

In a manager/supervisor hierarchy the coordinator decomposes, delegates, integrates, and judges progress. Passing each subagent only what it needs is the essential part: it keeps each subagent's context small and focused on its specialty. The coordinator holds the whole while subagents work locally — that division is where the topology's value comes from.

- **A 不正解**: サブエージェントを使わないなら、この構成を採る意味がありません。 / Defeats the purpose of the topology.
- **B 不正解**: 統合を行わない転送だけでは、全体の目標に対する判断ができません。 / Forwarding without integration cannot judge progress toward the goal.
- **C 不正解**: 同じ作業の重複はコストを倍増させ、分業の利点を失います。 / Duplicates work and forfeits the division of labor.

**参照 / Reference:** Agent Architecture — マネージャー／スーパーバイザー階層
</details>

---

### 問題 5 / Question 5

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

1 つのエージェントに、5 つの異なる業務領域（人事・会計・在庫・法務・営業）の質問を処理させています。各領域の用語定義と注意事項をすべてシステムプロンプトに含めた結果、8,000 トークンになりました。運用すると、ある領域の質問に別の領域の用語で回答する、無関係な注意事項に引きずられる、といった問題が出ています。

A single agent handles questions across five business areas — HR, accounting, inventory, legal, sales — with every area's glossary and caveats in one 8,000-token system prompt. In production, answers to one area's questions use another area's terminology, and irrelevant caveats influence the output.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) システムプロンプトをさらに詳細にして、領域の区別を明確に書く / Expand the system prompt with more detail distinguishing the areas
- B) **領域ごとにサブエージェントを分け、それぞれが自領域の指示とツールだけを持つ構成にする。各サブエージェントのコンテキストが短く一貫したものになり、無関係な情報による影響がなくなる** / **Split into per-area subagents, each holding only its own instructions and tools, so each context is short and coherent and unrelated material cannot influence it**
- C) 5 領域の用語を統一する / Unify the terminology across the five areas
- D) 質問できる領域を 1 つに限定する / Restrict the system to one area

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

症状は典型的な**コンテキストの汚染**です。無関係な領域の指示と用語が同一のコンテキストに同居していると、モデルはそれらを確実には区別できません。指示を足して解決しようとするのは、原因（同居そのもの）ではなく症状への対処です。領域ごとにサブエージェントを分ければ、各コンテキストは短く一貫したものになり、精度とコストの両方が改善します。**サブエージェントの価値の 1 つは、このコンテキスト分離**にあります。

The symptom is context contamination: unrelated instructions and glossaries sharing one context cannot be reliably separated by the model. Adding more instructions addresses the symptom rather than the cause. Splitting per area makes each context short and coherent, improving both accuracy and cost — context isolation being one of the main things subagents buy you.

- **A 不正解**: 汚染された同一コンテキスト内での指示追加は、区別の負荷をモデルに押し付けるだけです。 / Pushes the burden of discrimination onto the model.
- **C 不正解**: 用語の統一は業務上不可能で、各領域の正確性も損ないます。 / Not possible operationally, and damages per-area accuracy.
- **D 不正解**: 機能の縮小であり、設計で解決可能な問題への過剰反応です。 / Disproportionate to a solvable design problem.

**参照 / Reference:** Agent Architecture — サブエージェントによるコンテキスト分離
</details>

---

### 問題 6 / Question 6

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

「問い合わせを 4 カテゴリに分類し、担当キューに振り分ける」という要件に対して、意図分類エージェント・感情分析エージェント・緊急度判定エージェント・振り分けエージェントの 4 つをコーディネーターが束ねる構成が提案されました。現状は担当者が件名を読んで 10 秒で振り分けています。

For the requirement "classify inquiries into four categories and route them to the owning queue," a design is proposed with four subagents — intent, sentiment, urgency, routing — behind a coordinator. Today a person reads the subject line and routes it in ten seconds.

**設問 / Question:**

レビュアーとして最も適切な指摘はどれですか？

What is the most appropriate review comment?

- A) **要件に対して構成が過剰である。分類と振り分けは、必要な項目を 1 回の構造化出力で受け取り、コードでキューに投入すれば足りる。5 コンポーネントの構成はレイテンシ・コスト・障害点・運用負荷を増やすが、精度上の利得は示されていない。まず単純な構成を評価データセットで測り、不足が実証された部分にだけ複雑さを足すべきである** / **The design is disproportionate: one structured-output call returning the needed fields, plus code to enqueue, suffices. Five components add latency, cost, failure points, and operational burden with no demonstrated accuracy benefit. Measure the simple design on an evaluation set first and add complexity only where a gap is proven**
- B) エージェントを 6 つに増やすべきである / Increase to six agents
- C) コーディネーターを 2 段構成にすべきである / Make the coordinator two-tiered
- D) 各エージェントを別々のサービスとしてデプロイすべきである / Deploy each agent as a separate service

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**複雑さは要件が要求したときにだけ足す**というのがアーキテクチャの原則です。ここでは人間が 10 秒で行える分類に対し、5 コンポーネントの構成が提案されています。マルチエージェントが正当化されるのは、コンテキスト分離や並列化に実際の必要がある場合ですが、この要件はいずれにも当たりません。分類対象が同一の入力に対する独立した複数の判定であることを踏まえれば、1 回の構造化出力にまとめるのが自然です。まず単純な構成を測定し、実証された不足に対してのみ複雑さを追加するのが正しい順序です。

Complexity is added when a requirement demands it. Here a task a person completes in ten seconds draws five components. Multi-agent designs earn their cost when context isolation or parallelism is genuinely needed; neither applies. Since these are independent classifications over the same input, one structured-output call is the natural shape. Measure the simple design first and add complexity only against a demonstrated gap.

- **B 不正解**: 過剰な構成にさらに要素を足す提案です。 / Adds to an already disproportionate design.
- **C 不正解**: 存在しない拡張要件のために階層を増やす投機的な過剰設計です。 / Speculative complexity.
- **D 不正解**: 分割デプロイは、不要なコンポーネントの運用負荷を確定させるだけです。 / Cements the burden of components that should not exist.

**参照 / Reference:** Agent Architecture — 適正な複雑さ、マルチエージェントの正当化条件
</details>

---

### 問題 7 / Question 7

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

コーディネーターが 4 つのサブエージェントに調査を委譲し、結果を統合してレポートを作ります。現在、各サブエージェントは調査で参照した資料の全文を含む長大なテキストをコーディネーターに返しており、統合の段階でコーディネーターのコンテキストが上限に近づいています。

A coordinator delegates research to four subagents and integrates their results into a report. Each subagent currently returns long text including the full documents it consulted, and by the integration step the coordinator's context approaches its limit.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) コーディネーターをより大きなコンテキストウィンドウのモデルに変更する / Move the coordinator to a model with a larger context window
- B) サブエージェントの数を 2 つに減らす / Reduce the subagents to two
- C) **サブエージェントの返却内容を構造化した所見に限定する。発見事項、根拠となる出典の識別子、重要度といった統合に必要な要素だけを返し、参照した資料の全文は返さない** / **Restrict what subagents return to structured findings: the finding, an identifier for its source, and its significance — the elements integration needs — without the full text of the documents consulted**
- D) 統合を行わず、4 つの結果をそのまま並べて出力する / Skip integration and output the four results side by side

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

サブエージェントを使う目的の 1 つは、**各サブエージェントが大量の資料を処理し、コーディネーターには要点だけを返すこと**でコンテキストを分離することです。全文を返してしまうと、この分離が働かず、結局コーディネーター側に全資料が集まって同じ問題が起きます。返却内容を構造化した所見に限定すれば、統合に必要な情報を保ちながらコンテキストを小さく保てます。出典の識別子を含めることで、必要なら元資料を辿れる経路も残せます。

Part of the point of subagents is that each processes a large body of material and returns only what matters, isolating context. Returning full text defeats that isolation and reassembles everything in the coordinator. Restricting the return to structured findings preserves what integration needs while keeping context small, and including a source identifier keeps a path back to the original material when needed.

- **A 不正解**: ウィンドウを広げても全資料が集まる構造は変わらず、コストも増えます。 / The topology and its cost remain.
- **B 不正解**: サブエージェントを減らすとカバレッジが落ち、原因にも対処していません。 / Reduces coverage without addressing the cause.
- **D 不正解**: 統合はこのシステムの目的そのもので、省略できません。 / Integration is the system's purpose.

**参照 / Reference:** Agent Architecture — サブエージェントの結果の受け渡し
</details>

---

### 問題 8 / Question 8

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

独自のエージェントループを実装しています。Claude にツールを渡してリクエストを送り、応答を処理する部分を書いています。

You are implementing a custom agent loop, writing the part that sends a request with tools and processes the response.

**設問 / Question:**

ループの構造として正しいものはどれですか？

Which loop structure is correct?

- A) 1 回リクエストを送り、応答をそのまま返して終了する / Send one request, return the response, and finish
- B) **応答の `stop_reason` を検査し、`tool_use` であれば要求されたツールを実行して結果を会話に追加し、再度リクエストを送る。これを `stop_reason` が `end_turn` になるまで繰り返す。1 つの応答に複数のツール要求が含まれる場合は、すべての結果を 1 つのメッセージにまとめて返す** / **Inspect the response's `stop_reason`: while it is `tool_use`, execute the requested tools, append the results to the conversation, and send again — repeating until `stop_reason` is `end_turn`. When one response contains several tool requests, return all their results in a single message**
- C) ツールの実行結果を無視して、次のリクエストを送る / Ignore the tool results and send the next request
- D) ツールが要求されたら、その時点でユーザーに確認を求めて終了する / Ask the user for confirmation and terminate whenever a tool is requested

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

エージェントループの基本構造は、**`stop_reason` を見て継続するかを判断する**ことです。`tool_use` はモデルがツールの実行結果を待っている状態を示し、アプリケーションが結果を会話に追加して再度呼び出すことで処理が進みます。重要な実装上の注意として、1 つの応答に複数のツール要求（並列ツール使用）が含まれる場合は、**すべての結果を 1 つのメッセージにまとめて返す**必要があります。結果を複数のメッセージに分けると、以後モデルが並列にツールを呼ばなくなります。

The loop's structure is driven by `stop_reason`: `tool_use` means the model is waiting on tool output, and the application appends the results and calls again. One implementation detail matters — when a single response contains several tool requests (parallel tool use), all their results must come back in one message. Splitting them across messages trains the model to stop making parallel calls.

- **A 不正解**: 1 回で終了すると、ツールを使う処理が完了しません。 / A single call never completes tool-using work.
- **C 不正解**: 結果を渡さなければ、モデルは同じ要求を繰り返すか誤った前提で進みます。 / Without the result, the model repeats or proceeds on a false premise.
- **D 不正解**: すべてのツール呼び出しに確認を求めるのは、リスクに応じた設計ではありません。 / Confirming every call is not a risk-proportionate design.

**参照 / Reference:** Agent Construction with Claude — エージェントループ、並列ツール使用
</details>

---

### 問題 9 / Question 9

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

開発支援エージェントが、リポジトリ内のファイルを編集できます。運用テストで、エージェントがリポジトリ外のパスや、CI 設定ファイルを書き換えようとする挙動が観測されました。システムプロンプトには「リポジトリ外を編集しないこと」と記載しています。

A development-assist agent can edit files in the repository. In testing it attempted to write outside the repository and to modify CI configuration. The system prompt states that files outside the repository must not be edited.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) システムプロンプトの記述をより強い表現にする / Strengthen the wording in the system prompt
- B) 編集後に差分を確認し、問題があれば戻す / Review the diff afterwards and revert if needed
- C) エージェントからファイル編集の機能を取り上げる / Remove file editing from the agent entirely
- D) **ツール実行の前に介入する決定的な仕組み（フック）で、許可された範囲外への書き込みを拒否する。パスを正規化したうえで境界を判定し、影響の大きいファイルは別途拒否対象にする。プロンプトによる制限は確率的で、不可逆な操作の統制にはならない** / **Intercept before the tool runs with a deterministic hook that refuses writes outside the permitted scope, canonicalizing the path before deciding and separately denying high-impact files. A prompt-level restriction is probabilistic and does not control an irreversible operation**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**ファイルへの書き込みのような不可逆な操作は、決定的な仕組みで統制します。**フックはツールの実行前に介入して、条件を満たさない呼び出しを拒否できるため、この用途に適合します。パスの判定では正規化が必須で、これがないと相対パスやシンボリックリンクで境界を越えられます。CI 設定のように影響の大きいファイルは、リポジトリ内であっても別扱いにする価値があります。プロンプトによる指示は、確率的であるため統制になりません。

Irreversible operations such as file writes are controlled deterministically. A hook intercepts before the tool runs and can refuse a call that fails the condition, which fits exactly. Path checks must canonicalize, or relative paths and symlinks escape the boundary, and high-impact files such as CI configuration deserve separate treatment even inside the repository. A prompt instruction is probabilistic and is not a control.

- **A 不正解**: 表現を強めても確率的な制限であることは変わりません。 / Emphasis does not make it deterministic.
- **B 不正解**: 事後確認では、既に書き換えが実行されています。 / By review time the write has happened.
- **C 不正解**: 編集機能はエージェントの用途に必要で、範囲を制限すれば両立できます。 / Scoping achieves both safety and capability.

**参照 / Reference:** Agent Construction with Claude — 決定的な動作のためのフック
</details>

---

### 問題 10 / Question 10

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

エージェントの構築で、フックをどこに使うべきかを整理しています。

You are working out where hooks belong in an agent's construction.

**設問 / Question:**

フックの用途として適切なものを **2 つ選択してください**。

Select **2** appropriate uses for hooks.

- A) **不可逆な操作や機微な操作の実行前に検証し、条件を満たさない呼び出しを決定的に拒否する** / **Validating before an irreversible or sensitive operation runs, and deterministically refusing calls that fail the condition**
- B) モデルの回答の文体を整える / Adjusting the writing style of the model's answers
- C) **ツールの呼び出しや実行結果を記録し、監査証跡や可観測性のための情報を残す** / **Recording tool invocations and their results, producing information for audit trails and observability**
- D) モデルに与える知識を増やす / Adding to the model's knowledge
- E) プロンプトの文言を自動的に改善する / Automatically improving prompt wording

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, C**

**解説 / Explanation:**

フックの価値は、**エージェントの動作に決定的な処理を差し込めること**にあります。実行前の検証は、確率的なモデルの判断に依存せずに危険な操作を止める手段で、不可逆な操作の統制に不可欠です。記録は、いつどのツールがどの引数で呼ばれ何が返ったかを残す仕組みで、監査証跡と障害調査の両方に使えます。いずれも「モデルに頼らず確実に行われる」ことが要点で、文体や知識といったモデル側の性質を変える用途ではありません。

Hooks matter because they insert deterministic processing into an agent's operation. Pre-execution validation stops dangerous calls without relying on probabilistic judgment, which is essential for irreversible operations. Recording captures which tool ran with which arguments and what came back, serving both audit and investigation. Both are valuable precisely because they happen reliably rather than at the model's discretion; they are not a way to change style or knowledge.

- **B 不正解**: 文体はプロンプトや出力形式の設計で扱う領域です。 / Handled through prompt and output-format design.
- **D 不正解**: 知識の追加は取得やコンテキストの設計で行います。 / Knowledge comes from retrieval and context design.
- **E 不正解**: プロンプトの改善は開発プロセスの活動で、実行時のフックの役割ではありません。 / A development activity, not a runtime hook's role.

**参照 / Reference:** Agent Construction with Claude — フックの用途
</details>

---

### 問題 11 / Question 11

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

エージェントを構築するにあたり、既存のエージェント構築の仕組みを使うか、独自のループを一から書くかを検討しています。要件は、独自に定義した 6 つのツールを使う多段タスクで、特殊な制御フローの要求はありません。

You are deciding between an existing agent-construction facility and writing your own loop from scratch. The requirement is a multi-step task using six custom tools, with no unusual control-flow needs.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) **標準的な制御フローで足りるなら、既存の仕組みを使う。ループの実装、ツール結果の受け渡し、エラー処理といった定型部分が提供されるため、自前で書くとこれらを実装・保守する負担を負うことになる。独自のループは、既存の仕組みでは表現できない制御フローが必要な場合に選ぶ** / **Where standard control flow suffices, use the existing facility: the loop, tool-result handling, and error handling come with it, and writing your own means implementing and maintaining all of that. Reserve a custom loop for control flow the existing facility cannot express**
- B) 常に独自のループを書くべきである / Always write your own loop
- C) 常に既存の仕組みを使うべきで、独自実装に理由はない / Always use the existing facility; a custom loop is never warranted
- D) ツールが 6 つなら独自実装が必要である / Six tools requires a custom implementation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

判断軸は**必要な制御フローが標準的な形で表現できるか**です。ツールを定義して多段で使うという要件は標準的なので、既存の仕組みで足ります。自前で書くと、ループの構造、ツール結果の受け渡し、並列ツール使用の扱い、エラー処理といった定型部分をすべて実装・保守することになり、そこに不具合が入る余地が生まれます。一方、既存の仕組みで表現できない制御フローが必要なら、独自実装が正当化されます。「常にどちらか」という判断ではありません。

The criterion is whether the required control flow is expressible in the standard shape. Defining tools and using them across steps is standard, so the existing facility suffices. Writing your own means implementing and maintaining the loop structure, tool-result handling, parallel tool use, and error handling — each a place for defects. A custom loop is justified when the control flow cannot be expressed otherwise. Neither answer is universal.

- **B 不正解**: 標準的な要件で自前実装を選ぶと、定型部分の実装と保守が負担になります。 / Takes on boilerplate for no gain.
- **C 不正解**: 独自実装が正当化される場面は存在します。 / Custom loops are sometimes warranted.
- **D 不正解**: ツールの数は判断基準になりません。 / Tool count is not a criterion.

**参照 / Reference:** Agent Construction with Claude — 既存の仕組みと独自ループの選択
</details>

---

### 問題 12 / Question 12

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

エージェントのデプロイ形態を検討しています。要件は、1 セッションが数時間続き、途中でファイル操作とコード実行を伴い、実行環境はセッションごとに分離されている必要がある、というものです。運用チームは実行環境の管理負担を避けたいと考えています。

You are choosing a deployment form for an agent. Sessions run for hours, involve file operations and code execution, and require an execution environment isolated per session. The operations team wants to avoid managing that environment.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) 実行環境の管理は必ず自社で行うべきである / The execution environment must always be self-managed
- B) デプロイ形態は品質に影響しないので、どちらでもよい / Deployment form does not affect quality, so either is fine
- C) **エージェントループとツール実行環境の両方をホストしてもらう形態が要件に適合する。セッションごとの分離された実行環境と長時間のセッションが提供され、運用チームは環境の管理を負わずに済む。自前でホストする形態は、実行環境を自社で持ちたい場合や独自の実行基盤が必要な場合に選ぶ** / **A form where both the agent loop and the tool-execution environment are hosted fits: it provides per-session isolated environments and long-running sessions without the operations team managing them. Self-hosting is the choice when you want to own the compute or need a custom execution substrate**
- D) 長時間のセッションはどの形態でも実現できないので、要件を変更する / Long sessions are impossible in any form, so the requirement must change

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

デプロイ形態の選択は、**エージェントループとツール実行環境を誰が持つか**で分かれます。本問の要件（セッションごとの分離された実行環境、長時間セッション、環境管理を負いたくない）は、両方をホストしてもらう形態に適合します。逆に、実行環境を自社で管理したい場合や、独自の実行基盤が必要な場合は、自前でホストする形態を選びます。この判断は品質ではなく、運用の責任範囲と要件で決まります。

Deployment form turns on who owns the agent loop and the tool-execution environment. The stated requirements — per-session isolation, long sessions, no environment management — fit a form where both are hosted. Self-hosting is the choice where you want to own the compute or need a custom substrate. The decision follows operational responsibility and requirements, not quality.

- **A 不正解**: 自社管理が常に必要とは限らず、本問では管理負担を避けたいと明示されています。 / Not always necessary, and explicitly unwanted here.
- **B 不正解**: 実行環境の提供有無や運用負担は形態によって大きく異なります。 / The forms differ substantially in what they provide.
- **D 不正解**: 長時間のセッションを前提とした形態は存在します。 / Long-running sessions are supported.

**参照 / Reference:** Agent Construction with Claude — マネージドと自己ホストのデプロイ形態
</details>

---

### 問題 13 / Question 13

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

社内ヘルプデスクエージェントに、業務効率のため人事・給与データベースへの広い読み取り権限を与えています。ある社員が質問の仕方を工夫して、同僚の給与情報を引き出そうと試み、一部成功しました。エージェントは技術的には正常に動作していました。

An internal help-desk agent holds broad read access to HR and payroll databases for efficiency. An employee phrased questions creatively to extract colleagues' salary information and partially succeeded. The agent functioned exactly as built.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) 該当社員を処分し、他の社員に注意喚起する / Discipline the employee and warn others
- B) **エージェントが利用者本人の権限を超えないようにする。エージェント自身の広い権限ではなく、呼び出したユーザーの権限で人事システムに問い合わせる構成にし、認可の判定は人事システム側の既存の権限モデルに委ねる** / **Ensure the agent never exceeds the calling user's own permissions: query HR under that user's identity rather than the agent's broad grant, leaving authorization to HR's existing permission model**
- C) システムプロンプトに「給与に関する質問には答えないこと」と記載する / State in the system prompt that salary questions must not be answered
- D) エージェントから人事データベースへの接続を削除する / Remove the agent's access to the HR database

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**エージェントが利用者の権限を超えた権限を持つ構成は、権限昇格の経路そのものです。**技術的に正常動作していたという事実がこれを裏付けており、問題は個人の行為ではなく設計にあります。呼び出したユーザーの権限で問い合わせる構成にすれば、どのように質問を工夫しても、そのユーザーが本来アクセスできない情報は返りません。認可の判定を人事システム側に委ねることで、異動や昇格による権限変更も自動的に反映されます。

An agent holding more privilege than its user *is* a privilege-escalation path, and the fact that it worked as built confirms the problem is design rather than conduct. Querying under the calling user's identity means no phrasing returns data that user cannot access, and delegating the authorization decision to the HR system keeps permissions correct as people transfer or are promoted.

- **A 不正解**: 処分は個別事案への対応で、同じ構成である限り他の社員も同じことができます。 / The capability remains for everyone else.
- **C 不正解**: プロンプトによる制限は言い換えで回避され、統制になりません。 / Defeated by rephrasing.
- **D 不正解**: 接続の削除は、本人の情報照会というヘルプデスク本来の機能を失わせます。 / Removes legitimate functionality.

**参照 / Reference:** Agent Construction with Claude — 権限設計、AI Application Security — 権限昇格の防止
</details>

---

### 問題 14 / Question 14

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

エージェントループの中で、ツールの実行が失敗することがあります。現在の実装は、ツールが例外を投げた場合、その結果を会話に追加せずにループを続けています。運用すると、モデルが同じツールを同じ引数で繰り返し呼ぶ挙動が観測されています。

Within an agent loop, tool execution sometimes fails. The current implementation continues the loop without appending anything to the conversation when a tool raises. In production, the model repeatedly calls the same tool with the same arguments.

**設問 / Question:**

最も適切な修正はどれですか？

What is the most appropriate fix?

- A) ツールが失敗したらループを即座に終了する / Terminate the loop immediately when a tool fails
- B) ツールの例外を無視して、成功したことにする / Ignore the exception and report success
- C) ツールの実行を必ず成功するように実装し直す / Reimplement the tools so they never fail
- D) **失敗した場合も、失敗である旨を示す結果を会話に追加する。何が起きたか（引数が不正、対象が存在しない、一時的な障害など）と、再試行して意味があるかが分かる内容を含めることで、モデルは自己修正するか別の方法を試せる** / **Append a result marking the failure to the conversation, including what happened — malformed argument, missing target, transient failure — and whether retrying can help, so the model can self-correct or try another approach**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

ツールの実行結果は、**成功でも失敗でも会話に追加する**必要があります。結果を追加しないと、モデルから見れば要求が処理されなかったのと同じ状態なので、同じ呼び出しを繰り返すのは自然な帰結です。失敗を示す結果を返し、原因が分かる内容を含めれば、モデルは引数を修正したり別の方法を試したりできます。エラーメッセージはモデルへの入力であり、次の行動を決める材料になる、という視点が重要です。

Tool results must be appended whether they succeeded or failed. Without a result, the model sees a request that was never processed, and repeating the call is the natural consequence. Returning a failure result with enough detail to identify the cause lets the model correct the arguments or try another route. The error message is model input that determines the next action.

- **A 不正解**: 回復可能な失敗（引数の形式ミスなど）でも終了することになり、過剰です。 / Disproportionate for recoverable failures.
- **B 不正解**: 失敗を成功として扱うのは最も危険で、モデルは誤った前提で進みます。 / The most dangerous option; the model proceeds on a false premise.
- **C 不正解**: 外部システムに依存する以上、失敗しないツールは実現できません。 / Not achievable with external dependencies.

**参照 / Reference:** Agent Construction with Claude — ツール結果の扱い、エラーの返却
</details>

---

### 問題 15 / Question 15

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

自前のハーネスでエージェントを本番運用します。エージェントはツールを使って外部システムを操作し、1 セッションで数十回の呼び出しを行います。

You are running an agent in production on your own harness. It operates external systems through tools, making dozens of calls per session.

**設問 / Question:**

自前のハーネスで運用する際に必要な配慮を **2 つ選択してください**。

Select **2** considerations when operating your own harness.

- A) ツールの数を 3 つ以下にする / Limit the tools to three or fewer
- B) モデルを毎回変更する / Change the model on every request
- C) **ループの終了条件を明示的に設ける。回数や時間の上限、目標達成の判定を持たせ、想定外の入力でループが止まらなくなる事態を防ぐ** / **Define explicit termination conditions: caps on iterations and elapsed time, plus a goal-completion check, so an unexpected input cannot leave the loop running indefinitely**
- D) すべてのツール呼び出しに人間の承認を求める / Require human approval for every tool call
- E) **各ステップの状態を記録し、障害で中断した場合に途中から再開できるようにする。数十回の呼び出しをやり直すことにならないようにする** / **Record state at each step so a run interrupted by a failure can resume rather than repeating dozens of calls**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, E**

**解説 / Explanation:**

自前のハーネスでは、**既存の仕組みが提供する安全装置を自分で用意する**必要があります。終了条件は、想定外の入力でループが止まらなくなる（同じ操作を繰り返す、目標に到達しないまま呼び出し続ける）事態を防ぐために不可欠です。状態の記録は、数十回の呼び出しを伴うセッションが途中で中断したときに、最初からやり直さずに済ませるために必要です。どちらも、実装していないことに気づくのは障害が起きた後になりがちな項目です。

Running your own harness means supplying the safety mechanisms an existing facility would provide. Termination conditions prevent an unexpected input from leaving the loop running — repeating an operation, or calling indefinitely without reaching the goal. State recording keeps an interrupted session with dozens of calls from restarting from scratch. Both are typically noticed as missing only after an incident.

- **A 不正解**: ツール数の制限は用途によって決まるもので、ハーネス固有の配慮ではありません。 / Determined by the use case, not by harness ownership.
- **B 不正解**: モデルを毎回変えると挙動が予測不能になります。 / Makes behavior unpredictable.
- **D 不正解**: すべての呼び出しに承認を求めるのは、リスクに応じた設計ではなく、自動化の目的を損ないます。 / Not risk-proportionate, and defeats the automation.

**参照 / Reference:** Agent Construction with Claude — 独自ハーネスの運用、終了条件と状態管理
</details>

---

### 問題 16 / Question 16

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

エージェントが 1 つの応答の中で 3 つのツールを同時に要求しました。3 つは相互に独立しています。開発者は、3 つのツールを順に実行し、それぞれの結果を別々のメッセージとして会話に追加する実装をしました。その後、モデルが複数のツールを同時に要求しなくなりました。

An agent requested three independent tools in a single response. The developer executed them in sequence and appended each result as a separate message. Subsequently the model stopped requesting multiple tools at once.

**設問 / Question:**

最も適切な修正はどれですか？

What is the most appropriate fix?

- A) **3 つの結果を 1 つのメッセージにまとめて返す。ツール結果を複数のメッセージに分けると会話の構造が崩れ、モデルが並列にツールを要求しなくなる。あわせて、独立したツールは並行して実行し、待ち時間を重ね合わせる** / **Return all three results in a single message: splitting tool results across messages breaks the conversation structure and trains the model out of parallel requests. Also execute independent tools concurrently so their waits overlap**
- B) 並列のツール要求を無効にする / Disable parallel tool requests
- C) ツールを 1 つずつしか定義しない / Define only one tool at a time
- D) 3 つの結果を結合した 1 つの文字列にして返す / Concatenate the three results into one string

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**1 つの応答に含まれる複数のツール要求に対しては、すべての結果を 1 つのメッセージにまとめて返します。**結果を別々のメッセージに分けると会話の構造が想定と異なるものになり、観測されているとおりモデルが並列の要求をしなくなります。あわせて、独立したツールは並行して実行するのが自然で、待ち時間が重なる分だけ所要時間が短くなります。逐次実行は結果の順序には影響しませんが、レイテンシの面で不利です。

All results for the tool requests in one response belong in a single message. Splitting them produces a conversation structure the model does not expect, and the observed consequence is that it stops requesting tools in parallel. Independent tools should also be executed concurrently so their waits overlap; sequential execution does not change the results but costs latency.

- **B 不正解**: 並列要求は複数の独立した作業を効率的に進める仕組みで、無効化は損失です。 / Disabling forfeits an efficiency mechanism.
- **C 不正解**: ツール定義を減らすと、エージェントができる作業が減ります。 / Reduces what the agent can do.
- **D 不正解**: 結果を 1 つの文字列に結合すると、どの結果がどのツールのものか対応が失われます。 / Loses the correspondence between results and calls.

**参照 / Reference:** Agent Construction with Claude — 並列ツール使用、ツール結果の返し方
</details>

---

### 問題 17 / Question 17

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

多段タスクを処理するエージェントで、モデルがツールを呼び、結果を受けて次のツールを呼ぶ、という流れを実装しています。この基本的な構造の名前と性質を整理しています。

You are implementing an agent for multi-step tasks, where the model calls a tool, receives the result, and calls the next. You are characterizing this basic structure.

**設問 / Question:**

このパターンについて最も適切な説明はどれですか？

Which best describes this pattern?

- A) このパターンでは、ツールの呼び出し回数が事前に決まっている / The number of tool calls is fixed in advance
- B) このパターンでは、モデルは 1 回しかツールを呼べない / The model can call a tool only once
- C) **ツール使用ループと呼ばれる基本パターンで、モデルが次に何をするかを結果を見ながら決める。回数は事前に決まらないため、終了条件（目標の達成、上限回数、上限時間）を実装側が持つ必要がある** / **This is the tool-use loop, in which the model decides what to do next from the results. The number of iterations is not fixed in advance, so the implementation must supply termination conditions: goal completion, an iteration cap, and a time cap**
- D) このパターンはコストが一定である / The cost of this pattern is constant

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

ツール使用ループは、エージェントの最も基本的なパターンです。特徴は**呼び出し回数が事前に決まらないこと**で、これはモデルが結果を見て次を判断するという性質から直接導かれます。この性質には 2 つの帰結があります。第一に、終了条件を実装側が持たないとループが止まらなくなる可能性があります。第二に、1 タスクあたりのコストとレイテンシが可変になるため、上限の設計が必要になります。

The tool-use loop is the most basic agent pattern, characterized by an iteration count that is not fixed in advance — a direct consequence of the model deciding the next step from results. Two things follow: without termination conditions in the implementation the loop may not stop, and per-task cost and latency become variable, so caps must be designed in.

- **A 不正解**: 回数が事前に決まるなら、それはワークフローです。 / A fixed count describes a workflow.
- **B 不正解**: 複数回の呼び出しがこのパターンの本質です。 / Multiple calls are the essence of the pattern.
- **D 不正解**: 呼び出し回数が可変なので、コストも可変です。 / Variable iterations means variable cost.

**参照 / Reference:** Agent Patterns and Frameworks — ツール使用ループ
</details>

---

### 問題 18 / Question 18

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

顧客対応エージェントで、同じ顧客が複数回にわたって相談する際に、前回までのやり取りの要点を引き継ぎたいと考えています。全会話履歴をそのまま保持すると、回数を重ねるにつれてコンテキストが肥大化します。

In a customer-facing agent, you want to carry forward the essentials of prior conversations when the same customer returns. Retaining the full history verbatim inflates context as the number of conversations grows.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) 全会話履歴をすべて保持し、毎回コンテキストに含める / Retain every conversation in full and include it all each time
- B) **引き継ぐべき情報を選別して構造化した形で保持する。顧客の属性、確定した事実（契約内容、過去の対応結果）、未解決の課題といった、次回に必要な要素だけを残し、会話の逐語的なやり取りは保持しない** / **Select what must carry forward and hold it in structured form: customer attributes, established facts such as contract terms and prior resolutions, and open issues — retaining what the next conversation needs rather than the verbatim exchange**
- C) 会話履歴は一切保持しない / Retain nothing between conversations
- D) 会話履歴を毎回自由記述で要約して保持する / Summarize each conversation as free text and retain that

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

セッションをまたぐメモリの設計で重要なのは、**何を引き継ぐかを選別すること**です。次回に必要なのは、顧客の属性・確定した事実・未解決の課題であり、やり取りの逐語的な記録ではありません。構造化して保持すると、必要な項目を確実に参照でき、量も一定に保てます。自由記述の要約は、金額や日付といった具体値が抽象化されて失われるため、確定した事実の保持には向きません。

Cross-session memory turns on selecting what carries forward: the next conversation needs customer attributes, established facts, and open issues — not the verbatim exchange. Structured retention makes those reliably referenceable and bounds the volume. Free-text summaries abstract away precise values such as amounts and dates and are therefore poorly suited to holding established facts.

- **A 不正解**: 全保持はコンテキストの肥大化という、まさに避けたい問題を招きます。 / Produces exactly the inflation you are avoiding.
- **C 不正解**: 何も引き継がないと、顧客は毎回同じ説明を繰り返すことになります。 / The customer repeats themselves every time.
- **D 不正解**: 自由記述の要約は具体値を失いやすく、確定した事実の保持には不向きです。 / Loses the precise values that established facts consist of.

**参照 / Reference:** Agent Patterns and Frameworks — メモリのパターン
</details>

---

### 問題 19 / Question 19

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

エージェントのループが長くなるにつれてコンテキストが肥大化し、後半でコストとレイテンシが増加します。ツールの結果は多くが一度参照された後は不要ですが、タスクの目標と各段階で確定した事実は最後まで必要です。

As an agent's loop lengthens, context inflates and cost and latency rise in the later steps. Most tool results are unnecessary after their first use, while the task goal and the facts established at each step are needed to the end.

**設問 / Question:**

コンテキストウィンドウ管理の手法として適切なものを **2 つ選択してください**。

Select **2** appropriate context-window management techniques.

- A) **不要になったツール結果をコンテキストから除去する。一度使えば役目を終える結果は、要約する必要すらなく除去できる** / **Clear tool results that are no longer needed: single-use results can simply be removed rather than summarized**
- B) ループの回数を 5 回に制限する / Cap the loop at five iterations
- C) すべての履歴を毎回そのまま送り続ける / Continue sending the entire history each time
- D) **保持すべき情報（タスクの目標、確定した事実）を構造化して明示的に残し、毎ターン参照できるようにする** / **Retain what must persist — the task goal and established facts — explicitly in structured form so it is referenceable each turn**
- E) コンテキストが肥大化したらセッションを破棄する / Discard the session when context inflates

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, D**

**解説 / Explanation:**

長いエージェントループでは、**何を消し、何を残すかを分けて考えます**。一度使えば役目を終えるツール結果は「除去」の対象で、要約する必要もありません。一方、タスクの目標と確定した事実は「保持」の対象で、構造化して残すことで毎ターン確実に参照できます。この 2 つを分けずに一律の要約で扱うと、消すべきものが残り、残すべき具体値が抽象化されて失われるという逆の結果になりがちです。

Long loops require separating what to clear from what to keep. Single-use tool results can be cleared outright, without summarizing. The task goal and established facts must be kept, and structured retention makes them reliably referenceable each turn. Treating both with one generic summarization tends to invert the outcome: the disposable survives and the precise values that mattered are abstracted away.

- **B 不正解**: 回数制限は、5 回以上必要なタスクを完遂できなくします。 / Prevents completing tasks needing more.
- **C 不正解**: 蓄積を放置する現状そのもので、コストとレイテンシが増え続けます。 / The status quo being fixed.
- **E 不正解**: セッションの破棄は進行中の作業文脈を失い、タスクが完結しません。 / Loses the working context mid-task.

**参照 / Reference:** Agent Patterns and Frameworks — コンテキストウィンドウ管理
</details>

---

### 問題 20 / Question 20

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

エージェントの構築にあたり、エージェント抽象化フレームワークを使うかどうかを検討しています。要件は、標準的なツール使用ループと、いくつかのサブエージェントへの委譲です。チームはフレームワークの経験がありません。

You are considering whether to use an agentic abstraction framework. The requirement is a standard tool-use loop plus delegation to a few subagents. The team has no experience with such frameworks.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) フレームワークは常に使うべきである / Frameworks should always be used
- B) フレームワークは常に避けるべきである / Frameworks should always be avoided
- C) 最も人気のあるフレームワークを選べばよい / Choose the most popular framework
- D) **フレームワークが提供する抽象（ループ、状態管理、サブエージェントの調整、可観測性）が要件に対して価値を持つかで判断する。あわせて、学習コスト、依存の追加、フレームワークの制約に合わない要件が出たときの回避の難しさも評価する。要件が標準的で、その抽象が実際に使われるなら採用の価値がある** / **Decide by whether the abstractions a framework provides — the loop, state management, subagent coordination, observability — are valuable for your requirement, while also weighing the learning cost, the added dependency, and how hard it is to work around the framework when a requirement does not fit its model. Where the requirement is standard and those abstractions are actually used, adoption is worth it**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

フレームワークの採用は、**提供される抽象が要件に対して価値を持つか**で判断します。標準的なループやサブエージェントの調整、状態管理、可観測性といった機能が実際に使われるなら、自前で実装・保守する負担を減らせます。一方で、学習コスト、依存の追加、そしてフレームワークの想定から外れる要件が出たときの回避の難しさは、判断に含めるべき費用です。「常に使う／常に避ける」「人気で選ぶ」はいずれも要件との適合を見ていません。

Framework adoption turns on whether the abstractions are valuable for your requirement: where the loop, subagent coordination, state management, and observability are genuinely used, they replace work you would otherwise implement and maintain. The costs to weigh are learning time, an added dependency, and the difficulty of working around the framework when a requirement falls outside its model. "Always," "never," and "most popular" all skip the fit question.

- **A 不正解**: 要件によっては抽象が使われず、依存だけが残ります。 / Some requirements leave the abstractions unused.
- **B 不正解**: 標準的な要件では、自前実装のほうが負担が大きくなります。 / For standard requirements, hand-rolling costs more.
- **C 不正解**: 人気は自社の要件との適合を示しません。 / Popularity does not indicate fit.

**参照 / Reference:** Agent Patterns and Frameworks — フレームワークの選択
</details>

---

### 問題 21 / Question 21

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

企業調査タスクで、対象会社について (1) 財務、(2) 訴訟、(3) 知的財産、(4) 労務 の 4 領域を調べ、最後に統合レポートを作ります。4 領域は互いに独立して調査でき、各領域の資料は数百ページ規模です。単一のエージェントで実施したところ、後半の領域の分析品質が明確に低下しました。

A company-research task investigates four areas — financials, litigation, intellectual property, labor — and then produces an integrated report. The areas can be investigated independently and each has hundreds of pages of material. A single-agent attempt showed markedly degraded analysis on the later areas.

**設問 / Question:**

最も適切なパターンはどれですか？

Which pattern is most appropriate?

- A) **領域ごとに独立したサブエージェントを並列に動かし、それぞれが自領域の資料のみをコンテキストに持つ。各サブエージェントは所見を構造化して返し、統合エージェントはその構造化された所見だけを受け取ってレポートを作る** / **Run one independent subagent per area in parallel, each holding only its own material, returning structured findings — and have a synthesis agent produce the report from those findings alone**
- B) 単一エージェントのまま、システムプロンプトで各領域を丁寧に扱うよう指示する / Keep the single agent and instruct it to treat each area carefully
- C) 4 領域を順番に処理し、各領域の後にコンテキストをクリアする / Process the areas sequentially, clearing context after each
- D) 4 領域の資料をすべて 1 つのコンテキストに入れる / Load all four areas' material into one context

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「独立して調査でき、各領域の資料が大きく、最後に統合が必要」という条件は、**オーケストレーター・ワーカーの典型的な適合条件**です。並列化により所要時間が短縮され、コンテキスト分離により後半の品質低下が解消されます。統合エージェントが受け取るのを構造化された所見だけに限定するのが要点で、一次資料をそのまま統合側に流すと、統合側のコンテキストが再び肥大化して同じ問題が起きます。

Independent investigation, large per-area material, and a final synthesis are the textbook fit for orchestrator-worker. Parallelism cuts elapsed time and context isolation removes the late-area degradation. Restricting the synthesizer's input to structured findings is what keeps the problem from reappearing at the synthesis step.

- **B 不正解**: 品質低下の原因はコンテキストの累積であり、指示では解消しません。 / The cause is accumulation, not instruction.
- **C 不正解**: 逐次＋クリアは汚染を減らしますが並列化の利得がなく、統合に必要な情報の受け渡しも未定義です。 / Forfeits parallelism and leaves handoff undefined.
- **D 不正解**: 全資料を 1 コンテキストに入れるのは、失敗した構成の拡大版です。 / A larger version of the configuration that failed.

**参照 / Reference:** Agent Patterns and Frameworks — オーケストレーター・ワーカー、サブエージェント
</details>

---

### 問題 22 / Question 22

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

サブエージェントへの分割の仕方を検討しています。分割の境界をどう引くかで、構成の良し悪しが決まります。

You are deciding how to divide work among subagents. Where the boundaries are drawn determines whether the design works.

**設問 / Question:**

良いサブエージェントの境界の特徴を **2 つ選択してください**。

Select **2** characteristics of a good subagent boundary.

- A) サブエージェントの数ができるだけ多い / As many subagents as possible
- B) **各サブエージェントが独立して完了でき、他のサブエージェントの中間状態に依存しない** / **Each subagent can complete independently, without depending on another's intermediate state**
- C) すべてのサブエージェントが同じツールを持つ / All subagents hold the same tools
- D) すべてのサブエージェントが同じコンテキストを共有する / All subagents share the same context
- E) **返却する結果が構造化された形で表現でき、呼び出し側がそれだけで次の判断を行える** / **The returned result is expressible in structured form and is sufficient on its own for the caller's next decision**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, E**

**解説 / Explanation:**

良い境界の条件は、**独立して完了できること**と**結果が構造化して受け渡せること**です。前者が満たされないと、サブエージェント間で中間状態をやり取りする必要が生じ、分割の利点（並列化とコンテキスト分離）が失われます。後者が満たされないと、呼び出し側が生の出力を解釈する負担を負い、統合側のコンテキストが肥大化します。数を増やすことや、ツールやコンテキストを共有することは、いずれも良い境界の指標ではなく、むしろ分割の意味を薄めます。

A good boundary makes each subagent independently completable and its result structurally expressible. Without the first, subagents must exchange intermediate state and the benefits — parallelism and context isolation — disappear. Without the second, the caller must interpret raw output and its context inflates. Maximizing count or sharing tools and context are not indicators of a good boundary; they dilute the point of dividing at all.

- **A 不正解**: 数を増やすこと自体は目的ではなく、調整のコストが増えます。 / Count is not the goal; coordination cost rises.
- **C 不正解**: 同じツールを持つなら、分ける必要性が薄いことを示唆します。 / Identical tools suggest the split is unnecessary.
- **D 不正解**: コンテキストの共有は、分離という分割の主要な利点を失わせます。 / Sharing context forfeits the main benefit.

**参照 / Reference:** Agent Patterns and Frameworks — サブエージェントの境界設計
</details>

---

### 問題 23 / Question 23

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

3 つのサブエージェントを使う構成で、それぞれの実行順序を検討しています。サブエージェント A は市場データを収集し、B は競合の動向を調べ、C は A と B の結果を踏まえて示唆を導きます。

In a three-subagent design, you are deciding execution order. Subagent A gathers market data, B researches competitor activity, and C derives implications from A's and B's results.

**設問 / Question:**

最も適切な実行方式はどれですか？

Which execution scheme is most appropriate?

- A) A → B → C の順に逐次実行する / Run A → B → C strictly in sequence
- B) 3 つすべてを並列に実行する / Run all three in parallel
- C) **A と B は相互に依存しないため並列に実行し、両方の完了後に C を実行する。依存関係のあるところだけを直列にすることで、所要時間を最小にしながら正しい順序を保てる** / **Run A and B in parallel since they do not depend on each other, then run C once both complete: serializing only where a dependency exists minimizes elapsed time while preserving correctness**
- D) C を最初に実行する / Run C first

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

実行順序は**依存関係から導きます**。A と B は相互に依存しないので並列に実行でき、C は両方の結果を必要とするので、それらの完了を待って実行します。全体を逐次にすると、依存のない A と B の待ち時間が積み上がって無駄です。逆に 3 つすべてを並列にすると、C が必要な入力を得られないまま実行されることになり、正しく動きません。依存グラフに沿って、並列にできるところは並列に、直列が必要なところだけ直列にするのが原則です。

Execution order follows from the dependency graph. A and B are independent and run in parallel; C requires both and waits for them. Full serialization stacks A's and B's waits needlessly, while running all three in parallel executes C without the input it needs. Parallelize where the graph allows and serialize only where it requires.

- **A 不正解**: A と B の間に依存がないため、逐次実行は待ち時間の無駄です。 / No dependency between A and B; sequencing wastes time.
- **B 不正解**: C は A と B の結果を必要とするため、並列実行では成立しません。 / C needs their results.
- **D 不正解**: C を最初に実行すると、必要な入力が存在しません。 / C's inputs do not exist yet.

**参照 / Reference:** Agent Patterns and Frameworks — サブエージェントの並列と直列
</details>

---

## 発展 / Advanced

### 問題 24 / Question 24

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

保険金請求の処理を設計しています。全体の流れは規制により「受付 → 契約確認 → 免責確認 → 損害額算定 → 支払判定」の順で固定されています。一方、損害額算定の内部では、修理見積書・写真・過去事例など複数の情報源を状況に応じて参照する必要があり、必要な参照の順序と回数は案件によって変わります。

You are designing claims processing. Regulation fixes the overall sequence: intake → policy validity → exclusions → loss quantification → payout decision. Within loss quantification, however, repair estimates, photographs, and precedent cases must be consulted adaptively, with the order and number of lookups varying per claim.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) 全体を 1 つのエージェントにする / Make the whole flow one agent
- B) **全体を固定のワークフローとしてコードで記述し、損害額算定のステップの内部だけをツールを持つエージェントにする。規制で固定された順序はコードが保証し、探索的な判断が必要な範囲だけをモデルに委ねる** / **Encode the overall sequence as a code-defined workflow and make only the loss-quantification step an agent with tools: code guarantees the regulated order while model judgment is confined to the range that genuinely needs exploration**
- C) 全体を固定のワークフローにし、損害額算定も固定の順序で情報源を参照する / Make everything a fixed workflow, including a fixed lookup order in loss quantification
- D) 5 つのステップそれぞれをエージェントにし、コーディネーターが順序を判断する / Make each of the five steps an agent with a coordinator deciding the order

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**ワークフローとエージェントは排他ではなく、階層的に組み合わせられます。**規制で順序が固定されている外側はコードで書き、順序違反が構造的に起こり得ない状態にします。一方、探索的な判断が必要な内側の 1 ステップだけをエージェントにすれば、柔軟性が必要な範囲に限定して適用できます。この「決定論が必要な部分はコード、判断が必要な部分はモデル」という境界設定が、両方の要件を同時に満たす鍵です。

Workflows and agents are not exclusive; they compose hierarchically. The regulated outer sequence goes in code, where an ordering violation is structurally impossible, while the one step that genuinely requires exploration becomes an agent. Drawing the boundary at "code where determinism is required, model where judgment is" is what satisfies both requirements at once.

- **A 不正解**: 全体をエージェントにすると、規制で固定された順序をモデルの判断に委ねることになります。 / Delegates a regulated order to model judgment.
- **C 不正解**: 参照の順序を固定すると、案件によって変わる要件に対応できません。 / Cannot accommodate per-claim variation.
- **D 不正解**: 順序が固定されている以上、コーディネーターに順序を判断させる余地はありません。 / With the order fixed, there is nothing to decide.

**参照 / Reference:** Agent Architecture — ワークフローとエージェントの組み合わせ
</details>

---

### 問題 25 / Question 25

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

規制申請文書を作成するエージェントを運用します。1 件の作成は数日にわたり、数百ステップを要し、途中で人間のレビューが入ります。現在は中間成果物をすべてエージェントのセッションのコンテキストに保持しており、ワーカーが再起動すると最初からやり直しになります。また、処理中にプロンプトの修正をデプロイしたい場面が生じます。

An agent drafts regulatory submissions. One submission spans days and hundreds of steps with human review along the way. All intermediate artifacts currently live in the agent session's context, so a worker restart forces a restart from scratch — and prompt fixes sometimes need to deploy mid-run.

**設問 / Question:**

最も適切な状態管理はどれですか？

Which state-management design is most appropriate?

- A) より大きなコンテキストウィンドウのモデルに変更する / Move to a model with a larger context window
- B) ワーカーの再起動を禁止し、デプロイを停止する / Forbid worker restarts and freeze deploys
- C) 数日の処理を 1 日ごとに分割し、人間が手動で再起動する / Split into daily runs restarted manually
- D) **中間成果物とプロセスの状態を外部の永続ストアに置き、セッションには現在のステップに必要な最小限のコンテキストだけを持たせる。各ステップの完了時にチェックポイントを書き、再開時はストアから必要な分だけ読み込む。ステップは冪等にし、レビューの介入も状態遷移として記録する** / **Externalize intermediate artifacts and process state to a durable store, keeping only the context the current step needs in the session: checkpoint at each step boundary, reload only what a resumed step requires, make steps idempotent, and record review interventions as state transitions**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**長時間プロセスの状態をセッションのコンテキストに置くのは設計上の誤り**です。コンテキストは揮発性で、容量に上限があり、デプロイやプロセスの再起動をまたげません。状態を外部ストアに移すと、再起動耐性・デプロイ可能性・レビュー介入の記録・監査証跡が同時に得られます。セッションは「今のステップに必要な分だけ」を持つ短命な計算資源として扱うのが正解の形です。規制申請では、どの中間成果物がどの根拠から生じたかの追跡も要求されます。

Session context is the wrong home for long-running process state: it is volatile, bounded, and cannot survive a deploy. Externalizing buys restart tolerance, deployability, an intervention record, and an audit trail at once, and the session becomes a short-lived compute resource holding only what the current step needs. Regulatory submissions additionally require tracing which artifact came from which basis.

- **A 不正解**: ウィンドウを広げても揮発性は変わらず、再起動で失われます。 / A larger window is still volatile.
- **B 不正解**: デプロイ凍結は緊急の修正を阻み、再起動禁止は実現不可能な前提です。 / Blocks urgent fixes; forbidding restarts is not achievable.
- **C 不正解**: 分割しても状態の受け渡し方法が未定義なら、同じ問題が繰り返されます。 / Splitting without defining handoff repeats the problem.

**参照 / Reference:** Agent Architecture — 長時間プロセスの状態管理、外部チェックポイント
</details>

---

### 問題 26 / Question 26

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

サブエージェントを導入することで何が改善されるのかを、チーム内で整理しています。

Your team is articulating what introducing subagents actually improves.

**設問 / Question:**

サブエージェントがタスク実行を改善する理由として適切なものを **2 つ選択してください**。

Select **2** reasons subagents improve task execution.

- A) **各サブエージェントのコンテキストが自分の担当範囲に限定されるため、無関係な情報による影響を受けにくくなる** / **Each subagent's context is confined to its own scope, so unrelated material cannot influence it**
- B) サブエージェントを使うとモデルの精度自体が上がる / Using subagents raises the model's inherent accuracy
- C) **相互に独立したタスクを並行して実行できるため、全体の所要時間が短縮される** / **Independent tasks can run concurrently, reducing overall elapsed time**
- D) サブエージェントを使うとコストが必ず下がる / Subagents always reduce cost
- E) サブエージェントを使うと非決定性がなくなる / Subagents eliminate nondeterminism

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, C**

**解説 / Explanation:**

サブエージェントが改善するのは、**コンテキストの分離**と**並列化**の 2 点です。前者により、各サブエージェントは自分の担当範囲に集中でき、無関係な指示や資料による影響を受けません。後者により、独立したタスクの待ち時間が重なり、全体の所要時間が短くなります。一方、モデルの精度そのものが上がるわけではなく、コストはむしろ調整のオーバーヘッドで増えることもあります。非決定性も残ります。この 2 点が実際に必要でない場合、サブエージェントは複雑さを増やすだけになります。

Subagents improve two things: context isolation and parallelism. Isolation lets each focus on its own scope without influence from unrelated instructions or material; parallelism overlaps the waits of independent tasks. The model's inherent accuracy does not change, cost can rise through coordination overhead, and nondeterminism remains. Where neither benefit is needed, subagents add only complexity.

- **B 不正解**: モデルの能力は変わりません。改善するのは与えるコンテキストの質です。 / Capability is unchanged; what improves is the context given.
- **D 不正解**: 調整のオーバーヘッドでコストが増えることもあります。 / Coordination overhead can increase cost.
- **E 不正解**: 非決定性はモデルの性質であり、構成では消えません。 / A property of the model, not the topology.

**参照 / Reference:** Agent Architecture — サブエージェントの役割
</details>

---

### 問題 27 / Question 27

> サブスキル / Sub-skill: Agent Architecture (4.5%)

**シナリオ / Scenario:**

エージェントが顧客の注文内容を確認し、条件を満たす場合に返金を実行します。返金の可否は「購入から 30 日以内」「未開封」「セール品は対象外」というルールで決まり、このルールは法務が管理しています。現在はルールをシステムプロンプトに記述し、モデルが判定しています。

An agent checks a customer's order and issues a refund when conditions are met. Eligibility is governed by rules owned by legal: within 30 days of purchase, unopened, sale items excluded. The rules currently live in the system prompt and the model decides.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) **可否判定を決定的な処理としてツール側に実装し、モデルは判定結果を受けて顧客への説明を組み立てる役割に限定する。法務が管理する規則を確率的に評価させず、判定の一貫性と監査可能性を確保する** / **Implement eligibility as deterministic logic inside the tool and limit the model to explaining the outcome to the customer, so rules owned by legal are not evaluated probabilistically and the decision stays consistent and auditable**
- B) ルールをシステムプロンプトに詳細に書き直す / Rewrite the rules in the system prompt in more detail
- C) 判定結果を人間が全件確認する / Have a human verify every decision
- D) ルールを Few-shot 例として示す / Present the rules as few-shot examples

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**決定的なビジネスルールの評価は、モデルではなくコードの仕事**です。法務が管理する規則を確率的に評価させると、境界的なケース（購入 30 日目、セール品かつ未開封）で判定がぶれ、説明も一貫しません。ツール側に実装すれば、全条件を単体テストで検証でき、規則の改定も 1 か所の修正で済み、判定と理由が監査可能な形で残ります。モデルの役割は判定ではなく、結果を顧客に分かりやすく説明することです。ここでも「モデルが得意なこと」と「コードが得意なこと」の切り分けが要点になります。

Evaluating deterministic business rules is code's job, not the model's. Rules owned by legal, evaluated probabilistically, produce inconsistent outcomes on borderline cases and inconsistent explanations. Implemented in the tool, every condition is unit-testable, a revision is one edit, and the decision and its reasons are auditable. The model's role is explaining the outcome — again a matter of separating what models are good at from what code is good at.

- **B 不正解**: 詳細化しても確率的な評価であることは変わりません。 / Detail does not make it deterministic.
- **C 不正解**: 全件確認は自動化の目的を損ない、判定の一貫性も担当者に依存します。 / Defeats the automation and depends on the reviewer.
- **D 不正解**: Few-shot は判断の手本を示す手段で、規則の確実な適用を保証しません。 / Demonstrates judgment; does not guarantee rule application.

**参照 / Reference:** Agent Architecture — 決定論とモデル判断の境界
</details>

---

### 問題 28 / Question 28

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

エージェントが本番システムに対して破壊的な操作（データの削除、設定の変更）を行える構成です。これらの操作には人間の承認を必須にしたいと考えています。開発者は、システムプロンプトに「破壊的な操作の前には必ずユーザーに確認すること」と記載する案を提示しました。

An agent can perform destructive operations against production — deleting data, changing configuration — and you want human approval to be mandatory for them. A developer proposes stating in the system prompt that the agent must always confirm with the user first.

**設問 / Question:**

最も適切な指摘はどれですか？

What is the most appropriate objection?

- A) システムプロンプトの表現が弱いので、より強い表現にすべきである / The wording is too weak and should be strengthened
- B) 破壊的な操作をエージェントから完全に取り上げるべきである / Destructive operations should be removed from the agent entirely
- C) **プロンプトによる確認は確率的で、承認ゲートとして機能しない。承認は、ツールの実行前に介入する決定的な仕組みで実装し、承認が得られるまで操作が実行されない状態を保証する必要がある。承認画面には、何が行われるかと影響範囲を提示する** / **A prompt-level confirmation is probabilistic and does not function as an approval gate. Approval must be implemented by a deterministic mechanism that intercepts before the tool runs and guarantees the operation cannot execute without it — with the approval screen showing what will happen and its scope**
- D) 破壊的な操作は夜間にのみ実行するようにすべきである / Destructive operations should run only overnight

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**承認ゲートは決定的な仕組みで実装する**必要があります。プロンプトによる指示は、モデルが確認を省略する可能性を排除できないため、「必ず承認を経る」という保証になりません。ツールの実行前に介入する仕組みを使えば、承認が得られるまで操作が実行されないことを構造的に保証できます。あわせて、承認画面に「何が行われるか」と「影響範囲」を提示することが重要で、これがないと承認が形式的なものになり、ゲートとしての実質を失います。

An approval gate must be implemented deterministically. A prompt instruction cannot exclude the possibility that the model skips the confirmation, so it does not guarantee anything. A mechanism intercepting before the tool runs structurally guarantees that the operation cannot execute without approval. Equally important is showing what will happen and its scope on the approval screen — without that, approval becomes nominal and the gate loses its substance.

- **A 不正解**: 表現を強めても確率的であることは変わりません。 / Emphasis does not change its probabilistic nature.
- **B 不正解**: 承認ゲートを設ければ機能を保てます。全面的な削除は過剰です。 / Gating preserves the capability.
- **D 不正解**: 実行時間の制限は、承認の必要性とは別の話です。 / A scheduling constraint, not an approval control.

**参照 / Reference:** Agent Construction with Claude — フックによる承認ゲート
</details>

---

### 問題 29 / Question 29

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

本番のエージェントで、特定の入力に対してループが終わらず、同じツールを数百回呼び続ける事象が発生しました。コストが急増し、下流のシステムにも負荷がかかりました。現在の実装には、ループの上限が設けられていません。

In production, a particular input caused an agent's loop not to terminate, calling the same tool hundreds of times. Cost spiked and downstream systems came under load. The implementation has no loop cap.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) 該当する入力を拒否するフィルタを追加する / Add a filter rejecting that specific input
- B) **ループに複数の終了条件を設ける。呼び出し回数、経過時間、消費トークンの上限に加えて、同じツールを同じ引数で繰り返し呼んでいる状態の検出を入れる。上限に達した場合は、途中経過を保持したうえで停止し、人間に引き継ぐ** / **Give the loop several termination conditions: caps on iterations, elapsed time, and token spend, plus detection of repeated calls to the same tool with the same arguments — and on hitting a cap, stop while preserving progress and escalate to a human**
- C) コストの上限アラートを設定する / Set a cost-threshold alert
- D) モデルを変更する / Change the model

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

エージェントループには、**複数の観点からの終了条件が必要**です。回数・時間・トークン消費の上限は、どの軸で暴走しても止まる保証になります。加えて、同じツールを同じ引数で繰り返している状態は「進捗していない」ことの明確な兆候なので、これを検出して止める価値があります。上限到達時の扱いも設計対象で、途中経過を捨てて終了するのではなく、保持して人間に引き継げば、それまでの作業が無駄になりません。

An agent loop needs termination conditions along several axes: caps on iterations, elapsed time, and token spend guarantee it stops however it runs away. Detecting repeated calls with identical arguments adds a direct signal of no progress. What happens at the cap is also a design decision: preserving progress and escalating to a human keeps the work done so far from being wasted.

- **A 不正解**: 特定の入力を塞いでも、別の入力で同じ事象が起き得ます。 / Another input can reproduce it.
- **C 不正解**: アラートは検知の手段で、暴走を止める仕組みではありません。 / Detection, not prevention.
- **D 不正解**: モデル変更は、終了条件がないという構造的な欠陥を解消しません。 / Does not supply the missing termination conditions.

**参照 / Reference:** Agent Construction with Claude — ループの終了条件
</details>

---

### 問題 30 / Question 30

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

数時間にわたるエージェントセッションが、途中でワーカーの障害により中断しました。それまでに 60 回のツール呼び出しを行い、外部システムへの書き込みも含まれていました。再開しようとしましたが、どこまで完了していたかを判別する手段がありません。

An agent session running for hours was interrupted by a worker failure after 60 tool calls, some of which wrote to external systems. On attempting to resume, there is no way to determine how far it got.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) 中断したら最初からやり直す / Restart from the beginning on interruption
- B) ワーカーが落ちないように冗長化する / Make workers redundant so they do not fail
- C) 中断したセッションは破棄して、人間が手動で対応する / Discard interrupted sessions and handle them manually
- D) **各ステップの完了を外部の永続ストアに記録し、再開時にどこまで完了したかを判別できるようにする。あわせて、外部システムへの書き込みを冪等にし、再開時に同じ操作が重複して実行されないようにする** / **Record each step's completion in a durable store so a resumed run knows where it left off, and make writes to external systems idempotent so a resumption cannot re-execute the same operation**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

再開可能にするには、**進捗の記録**と**冪等性**の 2 つが必要です。進捗の記録がないと、どこまで完了したかが分からず、再開のしようがありません。冪等性がないと、記録があっても境界付近（記録の直前に実行された書き込み）で二重実行が起こり得ます。この 2 つは補完的で、片方だけでは安全な再開になりません。外部システムへの書き込みを含む長時間セッションでは、とりわけこの組み合わせが重要です。

Resumability needs both progress records and idempotency. Without records there is no way to know where to resume. With records but without idempotency, a write executed just before its record was written can run twice. The two are complementary, and the combination matters most in long sessions that write to external systems.

- **A 不正解**: 60 回の呼び出しをやり直すのは無駄で、外部への書き込みが重複します。 / Wastes the work and duplicates the external writes.
- **B 不正解**: 冗長化しても障害はゼロにならず、再開の仕組みは依然として必要です。 / Failures are never zero; resumption is still needed.
- **C 不正解**: 手動対応はスケールせず、数時間分の作業を人間がやり直すことになります。 / Does not scale and repeats hours of work manually.

**参照 / Reference:** Agent Construction with Claude — チェックポイントと再開、冪等性
</details>

---

### 問題 31 / Question 31

> サブスキル / Sub-skill: Agent Construction with Claude (5.3%)

**シナリオ / Scenario:**

毎晩定時にエージェントを起動し、その日のデータを分析してレポートを生成する仕組みを構築します。処理は 1〜3 時間かかり、途中でファイル生成とコード実行を伴います。運用チームは、スケジューラや実行環境の管理を自社で持ちたくないと考えています。

You are building a nightly agent run that analyzes the day's data and produces a report. Processing takes one to three hours and involves file generation and code execution. The operations team does not want to own a scheduler or execution environment.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) **エージェントループ、実行環境、スケジュール実行のすべてを提供する形態が要件に適合する。自社でスケジューラや実行環境を持たずに済み、長時間の処理とコード実行も提供される。自前でホストする場合は、これらを自社で構築・運用することになる** / **A form that provides the agent loop, the execution environment, and scheduled firing fits: no self-managed scheduler or environment, with long-running processing and code execution supplied. Self-hosting means building and operating all of that yourself**
- B) スケジュール実行は必ず自社の仕組みで行うべきである / Scheduling must always be done with your own infrastructure
- C) 定時実行は実現できないので、手動起動にする / Scheduled firing is not achievable, so trigger it manually
- D) 処理時間を 10 分以内に収めるよう要件を変更する / Change the requirement so processing fits in ten minutes

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

要件は 3 つあり、**長時間の処理**、**ファイル生成とコード実行が可能な実行環境**、**定時のスケジュール実行**です。これらをすべて提供する形態を選べば、運用チームはスケジューラも実行環境も持たずに済みます。自前でホストする場合、この 3 つを自社で構築・運用することになり、それが望ましいのは実行環境を自社で管理したい理由がある場合です。デプロイ形態の選択は、運用の責任範囲をどこに置くかの判断です。

Three requirements apply: long-running processing, an execution environment capable of file generation and code execution, and scheduled firing. A form providing all three leaves the operations team owning neither a scheduler nor an environment. Self-hosting means building and running all three, which is preferable when there is a reason to own the compute. The choice of deployment form is a choice about where operational responsibility sits.

- **B 不正解**: 自社での実装が必須という根拠はなく、本問では管理負担を避けたいと明示されています。 / No such requirement, and management burden is explicitly unwanted.
- **C 不正解**: 定時実行を提供する形態は存在します。 / Scheduled firing is available.
- **D 不正解**: 処理時間は分析の内容から決まるもので、恣意的に短縮できません。 / Determined by the analysis, not arbitrarily compressible.

**参照 / Reference:** Agent Construction with Claude — デプロイ形態、スケジュール実行
</details>

---

### 問題 32 / Question 32

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

複数のサブエージェントを使う構成を本番に出したところ、単一エージェントのときにはなかった問題がいくつか現れました。

After taking a multi-subagent design to production, several problems appeared that the single-agent version did not have.

**設問 / Question:**

マルチエージェント構成に固有の失敗モードを **2 つ選択してください**。

Select **2** failure modes specific to multi-agent designs.

- A) モデルが非決定的な出力を返す / The model returns nondeterministic output
- B) コンテキストウィンドウに上限がある / The context window has a limit
- C) **サブエージェント間で情報が欠落する。呼び出し側が渡した情報が不足していたり、返却された結果が統合に必要な要素を含んでいなかったりして、全体として一貫性を欠く** / **Information is lost between subagents: the caller passed too little, or the returned result lacks what integration needs, leaving the whole inconsistent**
- D) **調整のオーバーヘッドが利得を上回る。分割が細かすぎると、委譲と統合のコストとレイテンシが、並列化やコンテキスト分離による改善を打ち消す** / **Coordination overhead exceeds the benefit: with too fine a division, the cost and latency of delegation and integration cancel the gains from parallelism and isolation**
- E) API にレート制限がある / The API has rate limits

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, D**

**解説 / Explanation:**

マルチエージェント構成に固有の失敗モードは、**分割したことによって生じるもの**です。情報の欠落は境界をまたぐ受け渡しで起きる問題で、単一エージェントでは存在しません。調整のオーバーヘッドも同様で、分割が細かすぎると委譲と統合のコストが並列化の利得を上回ります。一方、非決定性・コンテキスト上限・レート制限は、構成に関わらず存在する制約であり、マルチエージェント固有ではありません。この区別は、問題の原因を構成に帰すべきかどうかの判断に必要です。

The failure modes specific to multi-agent designs are those created by the division itself. Information loss happens at the boundaries and does not exist in a single agent, and coordination overhead can exceed the parallelism gains when the division is too fine. Nondeterminism, the context limit, and rate limits are constraints present regardless of topology. The distinction matters for deciding whether a problem should be attributed to the design.

- **A 不正解**: 非決定性は構成に関係なく存在します。 / Present regardless of topology.
- **B 不正解**: コンテキストの上限は単一エージェントにもあります。 / Also present in a single agent.
- **E 不正解**: レート制限は API 側の制約で、構成とは無関係です。 / An API-side constraint.

**参照 / Reference:** Agent Patterns and Frameworks — マルチエージェントの失敗モード
</details>

---

### 問題 33 / Question 33

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

エージェント抽象化フレームワークを採用して 1 年が経ちました。新しい要件（特殊な承認フローと、独自の状態管理）が、フレームワークの想定する構造に収まらないことが分かりました。回避策を書くと、フレームワークの内部構造に依存したコードになります。

A year after adopting an agentic abstraction framework, a new requirement — a bespoke approval flow and custom state management — does not fit the framework's assumed structure. Working around it produces code that depends on the framework's internals.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) フレームワークの内部構造に依存した回避策を書く / Write the workaround against the framework's internals
- B) **フレームワークが提供する抽象と要件の乖離が、どの程度恒久的かを評価する。一時的な不一致なら回避策も選択肢だが、内部構造への依存は将来の更新で壊れる。乖離が構造的で今後も広がるなら、その部分をフレームワークの外に出すか、移行を計画するほうが総コストは小さくなる** / **Assess how permanent the mismatch is: a temporary gap can justify a workaround, but depending on internals breaks on future updates. Where the divergence is structural and likely to widen, moving that part outside the framework or planning a migration has the lower total cost**
- C) 新しい要件を諦める / Abandon the new requirement
- D) 直ちにフレームワークを廃止して全面的に書き直す / Drop the framework immediately and rewrite everything

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

フレームワークと要件の乖離が生じたときの判断軸は、**その乖離が一時的か構造的か**です。一時的な不一致であれば、限定的な回避策で乗り切る選択もあり得ます。ただし内部構造に依存した回避策は、フレームワークの更新で壊れるため、恒久的な解にはなりません。乖離が構造的で今後も広がると見込まれるなら、その部分だけをフレームワークの外に出すか、移行を計画するほうが総コストは小さくなります。「回避策を書く」か「全面書き直し」かの二択ではありません。

The criterion is whether the mismatch is temporary or structural. A temporary gap can justify a bounded workaround, though one written against internals is not a durable solution — a framework update breaks it. Where the divergence is structural and likely to widen, extracting that part or planning a migration costs less in total. The choice is not between a workaround and a rewrite.

- **A 不正解**: 内部構造への依存は、更新のたびに壊れる負債になります。 / Becomes debt that breaks on every update.
- **C 不正解**: 要件の放棄は、フレームワークの都合で事業要件を切り捨てる判断です。 / Discards a business requirement for the framework's convenience.
- **D 不正解**: 1 つの要件の不一致で全面書き直しは過剰で、既存の資産も失います。 / Disproportionate to one mismatch.

**参照 / Reference:** Agent Patterns and Frameworks — フレームワークの制約と乖離
</details>

---

### 問題 34 / Question 34

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

調査エージェントの品質を評価しています。現在は最終的に生成されたレポートの品質のみを採点しています。運用すると、レポートの品質は基準を満たしているのに、途中で無効なツール呼び出しを繰り返す、同じ検索を何度も試すといった非効率が生じており、コストとレイテンシが想定の 3 倍になっています。

You are evaluating a research agent, scoring only the quality of the final report. In production, reports meet the bar while the agent repeats invalid tool calls and retries the same searches, driving cost and latency to three times the projection.

**設問 / Question:**

最も適切な評価設計はどれですか？

Which evaluation design is most appropriate?

- A) 最終レポートの品質基準を厳しくする / Raise the quality bar on the final report
- B) ツール呼び出し回数の上限を 10 回に制限する / Cap tool calls at ten
- C) **最終成果物の品質に加えて、エージェントの軌跡を評価対象に含める。ツール呼び出し回数、無効な呼び出しの割合、目標に寄与しないステップ、総トークン数、所要時間を記録し、品質を維持したまま効率が改善しているかを追跡する** / **Evaluate the agent's trajectory alongside the final artifact: record tool-call count, invalid-call rate, steps that do not advance the goal, total tokens, and elapsed time, tracking whether efficiency improves while quality holds**
- D) エージェントをやめて固定のワークフローにする / Replace the agent with a fixed workflow

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**エージェントの評価は最終出力だけでは不十分**です。同じ品質の成果物でも、そこに至る過程の効率は大きく異なり、コスト・レイテンシ・信頼性に直結します。軌跡の指標（呼び出し回数、無効な呼び出し、寄与しないステップ）を記録すると、この差が可視化され、改善の対象になります。測定されていないものは改善されないため、効率を要件とするならまず測る必要があります。上限の設定は症状を抑えるだけで、非効率の原因は分かりません。

Evaluating only the final artifact is insufficient for agents: identical outputs can be reached by very different processes, and the process determines cost, latency, and reliability. Instrumenting the trajectory makes those differences visible and therefore improvable — what is not measured does not improve. A cap suppresses the symptom without revealing the cause.

- **A 不正解**: 品質は既に基準を満たしており、問題は効率です。 / Quality already passes; efficiency is the problem.
- **B 不正解**: 上限は症状を抑えますが、複雑なタスクを完遂できなくする副作用があります。 / Caps symptoms and breaks complex tasks.
- **D 不正解**: 効率を測定せずに構成を変えるのは早計で、探索的なタスクへの適応性も失います。 / Changes architecture before measuring, and forfeits adaptivity.

**参照 / Reference:** Agent Patterns and Frameworks — 軌跡の評価
</details>

---

### 問題 35 / Question 35

> サブスキル / Sub-skill: Agent Patterns and Frameworks (4.9%)

**シナリオ / Scenario:**

同じ利用者が繰り返し使うエージェントで、セッションをまたいで情報を引き継ぐメモリを設計しています。引き継ぐ候補には、利用者の好み、過去に扱った案件の結果、進行中の作業の状態、過去の会話の全文があります。マルチテナントの環境で、複数の顧客企業が利用します。

You are designing cross-session memory for an agent used repeatedly by the same people. Candidates to carry forward include user preferences, outcomes of past matters, the state of work in progress, and the full text of past conversations. The environment is multi-tenant, serving several customer companies.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) 過去の会話の全文をすべて保持し、毎回コンテキストに含める / Retain every past conversation in full and include it each time
- B) メモリは持たず、毎回ゼロから始める / Hold no memory and start fresh every time
- C) 全利用者のメモリを共通の領域に保存して、知見を共有する / Store all users' memory in a shared space so knowledge is shared
- D) **引き継ぐ情報を用途で選別し、構造化して保持する。好み・過去の結果・進行中の状態は次回に必要だが、会話の全文は不要である。あわせて、メモリの保存領域を利用者とテナントの単位で分離し、他の利用者や他社の情報が混入しない構造にする** / **Select what carries forward by purpose and hold it in structured form: preferences, past outcomes, and in-progress state are needed next time; full conversation text is not. Separately, partition the memory store by user and tenant so no other user's or company's information can enter**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

メモリの設計には**選別**と**分離**の 2 つの観点が必要です。選別については、次回に必要なのは好み・過去の結果・進行中の状態であり、会話の全文は量が増えるだけで価値が低いという判断になります。分離については、マルチテナント環境である以上、メモリの保存領域を利用者とテナントの単位で分けることが必須で、これを怠ると他社の情報が別の顧客の応答に現れる事態を招きます。メモリは便利な機能であると同時に、情報が境界を越える経路にもなり得ます。

Memory design needs both selection and separation. On selection, what the next session needs is preferences, past outcomes, and in-progress state; full conversation text adds volume with little value. On separation, a multi-tenant environment requires partitioning the store by user and tenant, or one company's information surfaces in another's responses. Memory is a useful capability and simultaneously a path across boundaries.

- **A 不正解**: 全文保持は量が増え続け、必要な情報が埋没します。 / Volume grows without bound and buries what matters.
- **B 不正解**: メモリなしでは、繰り返し利用の利点が得られません。 / Forfeits the benefit of repeat usage.
- **C 不正解**: 共通領域への保存は、テナント間の情報漏洩に直結します。 / Directly causes cross-tenant disclosure.

**参照 / Reference:** Agent Patterns and Frameworks — メモリ、マルチテナントでの分離
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **前 / Previous:** [`domain5_model_selection_optimization.md`](./domain5_model_selection_optimization.md) | **次 / Next:** [`domain6_prompt_context_engineering.md`](./domain6_prompt_context_engineering.md)
