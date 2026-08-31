# Domain 8: ツールと MCP / Tools and MCPs

> 配点比率 / Weight: **10.6%**
> 問題数 / Questions: **25**（基礎 17 / 発展 8）
> 形式 / Format: 単一選択（選択肢 A–D）＋複数選択（選択肢 A–E、選ぶ数を明示）

## サブスキル / Sub-skills

| サブスキル / Sub-skill | Weight | 本ファイル |
|---|---|---|
| Tool Implementation | 4.4% | 10 |
| MCP Server Development | 2.1% | 5 |
| Agentic Customization | 4.1% | 10 |

## 出題範囲 / Scope（公式ガイドの記述に基づく）

- **Tool Implementation** — Claude アプリケーションにおけるツール実装の実践。ツール使用と関数呼び出し、外部システム連携のための設定、ツール説明文の記述、エラー処理、ツール使用パターン（エージェンティックハーネスによるディスパッチ、クライアント側とサーバー側のツール、承認パターン）、ツールセット構築のベストプラクティス / Tool implementation practices for Claude applications, including tool use and function calling, configuration for external system interaction, tool description writing, error handling, tool usage patterns (agentic harness dispatch, client-side vs. server-side tools, approval patterns), and tool set construction best practices
- **MCP Server Development** — MCP サーバー開発の実践。サーバーの作成、デプロイ、Claude アプリケーションとの統合、MCP のリソース・ツール・プロンプト、通信パターン（stdio、ソケット、クライアントとサーバー） / MCP server development practices, including server authoring, deployment, integration with Claude applications, MCP resources, tools, and prompts, and communication patterns (stdio, sockets, client vs. server)
- **Agentic Customization** — 組み込みツール、カスタムツール、Skills、MCP の間のトレードオフ。ユースケースに応じた適切なアプローチの選択と適用 / Tradeoffs among built-in Tools, custom Tools, Skills, and MCPs for selecting and applying the appropriate approach for a given use case

---

## 基礎 / Foundations level

### 問題 1 / Question 1

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

在庫照会ツールを 1 つ定義して Messages API を呼び出したところ、応答の `stop_reason` が `tool_use` になり、`content` に `tool_use` ブロックが 1 つ含まれていました。

You defined a single inventory-lookup tool and called the Messages API. The response came back with `stop_reason` of `tool_use` and a `tool_use` block in `content`.

**設問 / Question:**

この時点で API は何をしたのですか？

What has the API done at this point?

- A) **Claude はツールを呼ぶべきだと判断し、ツール名と入力引数を返した。実際にツールを実行するのは呼び出し側のアプリケーションで、API がツールのコードを動かしたわけではない** / **Claude decided a tool should be called and returned the tool name and input arguments. Executing the tool is the calling application's job; the API did not run your tool code**
- B) API がツールを実行し、その結果を `tool_use` ブロックに入れて返した / The API executed the tool and returned its result inside the `tool_use` block
- C) ツール定義にエラーがあり、実行が中断された / There was an error in the tool definition and execution was aborted
- D) Claude がツールの実行を待っている間、接続が保留された / The connection was held open while Claude waited for the tool to run

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

ツール使用（関数呼び出し）は、モデルが「このツールをこの引数で呼ぶべきだ」と判断した内容を返す仕組みです。API 側でコードが実行されるわけではありません。アプリケーションが `tool_use` ブロックの `name` と `input` を読み、自分の環境でその処理を行い、結果を `tool_result` として次のリクエストに含めて返します。この往復を回すのがエージェンティックループです。

Tool use (function calling) returns the model's decision about which tool to call with which arguments. No code runs on the API side. Your application reads `name` and `input` from the `tool_use` block, performs the work in its own environment, and returns the outcome as a `tool_result` in the next request. Cycling that round trip is the agentic loop.

- **B 不正解**: `tool_use` ブロックはモデルの要求であって結果ではありません。結果を運ぶのは、次のリクエストで送る `tool_result` ブロックです / A `tool_use` block is the model's request, not a result. Results travel in the `tool_result` block you send in the next request
- **C 不正解**: 定義エラーならリクエスト自体が検証エラーになります。`stop_reason: "tool_use"` は正常な流れです / A definition error would fail request validation. `stop_reason: "tool_use"` is the normal path
- **D 不正解**: Messages API はステートレスで、応答を返した時点でリクエストは完結しています。保留状態は存在しません / The Messages API is stateless; the request completes when the response is returned. There is no held state

**参照 / Reference:** Tool Implementation — ツール使用と関数呼び出しの基本フロー
</details>

---

### 問題 2 / Question 2

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

ツールを実行して結果が得られました。この結果を Claude に返して回答を続けさせたいと考えています。

You executed the tool and have a result. You want to return it to Claude so it can continue.

**設問 / Question:**

結果の返し方として正しいのはどれですか？

Which is the correct way to return the result?

- A) `assistant` ロールのメッセージに `tool_result` ブロックとして入れる / Put it in an `assistant`-role message as a `tool_result` block
- B) **直前の `assistant` メッセージを会話履歴に含めたうえで、新しい `user` ロールのメッセージに `tool_result` ブロックを入れる。ブロックの `tool_use_id` は対応する `tool_use` ブロックの `id` と一致させる** / **Include the preceding `assistant` message in the conversation history, then send a new `user`-role message containing a `tool_result` block whose `tool_use_id` matches the `id` of the corresponding `tool_use` block**
- C) `system` プロンプトに結果を追記する / Append the result to the `system` prompt
- D) ツール定義の `description` に結果を書き込む / Write the result into the tool definition's `description`

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

`tool_result` は `user` ロールのメッセージに入れます。直感に反しますが、会話の構造上「モデルが要求し（assistant）、環境が応答する（user）」という形になるためです。`tool_use_id` は必須で、どの要求に対する結果かを対応づけます。並列でツールが呼ばれた場合はこの対応づけが不可欠です。また Messages API はステートレスなので、直前の `assistant` メッセージ（`tool_use` ブロックを含むもの）を履歴に含めて送らないと、対応づける相手が存在せずエラーになります。

A `tool_result` goes in a `user`-role message. It reads oddly, but structurally the model requests (assistant) and the environment responds (user). The `tool_use_id` is required and binds the result to its request — essential when tools are called in parallel. And because the Messages API is stateless, you must include the preceding `assistant` message (the one carrying the `tool_use` block) in the history, or there is nothing to bind to and the request errors.

- **A 不正解**: `tool_result` は `user` メッセージに入れます。`assistant` メッセージはモデルの発話を表します / `tool_result` belongs in a `user` message; `assistant` messages represent the model's own turns
- **C 不正解**: system プロンプトは会話全体の方針を書く場所で、個別のツール結果を運ぶ経路ではありません / The system prompt carries overall guidance, not individual tool results
- **D 不正解**: `description` はツールが何をするかの説明で、実行結果とは無関係です / A `description` says what a tool does; it has nothing to do with an execution result

**参照 / Reference:** Tool Implementation — tool_result の返し方、tool_use_id の対応づけ
</details>

---

### 問題 3 / Question 3

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

1 回の応答で Claude が 3 つのツールを同時に呼び出しました（`tool_use` ブロックが 3 つ）。実装では、結果が得られたものから順に 1 件ずつ `user` メッセージにして送り返しています。3 件目を送る前にエラーが返るようになりました。

In one response Claude called three tools at once (three `tool_use` blocks). Your implementation sends each result back as its own `user` message as soon as it is ready. You now get an error before the third is sent.

**設問 / Question:**

正しい実装はどれですか？

Which is the correct implementation?

- A) ツールを 1 つずつ順番に呼ばせるよう、プロンプトで並列実行を禁止する / Forbid parallel calls in the prompt so tools are invoked one at a time
- B) 3 件を別々の `user` メッセージにしたまま、間に空の `assistant` メッセージを挟む / Keep three separate `user` messages but insert empty `assistant` messages between them
- C) **3 つの `tool_result` ブロックすべてを 1 つの `user` メッセージにまとめて送る。すべてのツールの完了を待ってから 1 回だけ送信する** / **Put all three `tool_result` blocks in a single `user` message, sent once after all the tools have completed**
- D) 最も早く終わったツールの結果だけを返し、残りは破棄する / Return only the fastest tool's result and discard the rest

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

並列ツール使用では、1 つの `assistant` メッセージに含まれるすべての `tool_use` ブロックに対する `tool_result` を、**1 つの `user` メッセージ**にまとめて返す必要があります。分割して送ると、会話の構造上、要求と結果の対応が崩れて API エラーになります。実装としては、3 つのツールを（できれば並行に）実行し、全部の完了を待ってから 1 回だけ送るのが正解です。並列実行できることが並列ツール使用の利点なので、直列に待つ必要はありません。

With parallel tool use, every `tool_use` block in one `assistant` message must be answered by its `tool_result` in a **single** `user` message. Splitting them breaks the request/result correspondence and the API rejects it. The right implementation runs the three tools (ideally concurrently), waits for all of them, and sends once. Running them concurrently is the whole point — you do not have to serialize.

- **A 不正解**: 並列実行はレイテンシ削減のための機能で、禁止するのは症状への対処です。原因は返し方の実装です / Parallelism exists to cut latency; forbidding it treats the symptom. The cause is how results are returned
- **B 不正解**: 空の `assistant` メッセージを挟んでも対応関係は回復せず、会話構造がさらに壊れます / Empty `assistant` messages do not restore the correspondence and further corrupt the conversation structure
- **D 不正解**: 未応答の `tool_use` が残るとエラーになり、そもそも破棄した結果の分だけタスクが不完全になります / Leaving `tool_use` blocks unanswered errors out, and discarding results leaves the task incomplete

**参照 / Reference:** Tool Implementation — 並列ツール使用、tool_result の一括返却
</details>

---

### 問題 4 / Question 4

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

`search_records` というツールを定義しましたが、Claude が使うべき場面で使わなかったり、別のツールと取り違えたりします。現在の `description` は「レコードを検索します」の 1 行です。

You defined a tool called `search_records`, but Claude sometimes fails to use it when it should and sometimes confuses it with another tool. The current `description` is a single line: "Searches records."

**設問 / Question:**

ツール説明文の改善として有効なものを **2 つ選択してください**。

Select **2** effective improvements to the tool description.

- A) 説明文を削除し、ツール名だけで判断させる / Delete the description and let the tool name speak for itself
- B) **このツールが何を検索し、どんなときに使い、どんなときに使わないのか（他のツールとの使い分け）を具体的に書く** / **Spell out what it searches, when to use it, and when *not* to — how it differs from the neighboring tools**
- C) 説明文に「必ずこのツールを使うこと」と書く / Write "always use this tool" in the description
- D) **返る内容と制約（検索対象の範囲、件数上限、日付形式などの入力の前提）を書き、`input_schema` の各プロパティにも説明を付ける** / **Describe what comes back and the constraints — the scope searched, result limits, input assumptions such as date format — and document each property in the `input_schema`**
- E) 説明文を英語ではなくアプリケーションの表示言語に合わせる / Match the description to the application's display language rather than English

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, D**

**解説 / Explanation:**

ツールの選択はモデルが `description` と `input_schema` を読んで行います。したがって選択の失敗はほぼ常に説明文の問題です。B の「使うとき・使わないとき」は取り違えに直接効きます。似たツールが並んでいるとき、境界を書かなければモデルは推測するしかありません。D の返り値と制約、そして各プロパティの説明は、引数の組み立てを正しくし、期待と実際のずれ（例: 日付形式の不一致）を減らします。ツール定義は、モデルに対する API ドキュメントだと考えるのが適切です。

The model selects tools by reading the `description` and `input_schema`, so selection failures are nearly always description failures. Stating when to use it and when not to (B) directly addresses confusion: with similar tools side by side and no stated boundary, the model can only guess. Documenting the return shape, the constraints, and each schema property (D) makes argument construction correct and removes mismatches such as date formats. Treat a tool definition as API documentation written for the model.

- **A 不正解**: 名前だけでは何を検索するか、いつ使うかが伝わりません。説明文の削除は選択精度を下げます / A name alone conveys neither what is searched nor when to use it; removing the description makes selection worse
- **C 不正解**: 「必ず使う」は無条件の指示で、使うべきでない場面でも呼ばれるようになります。必要なのは強制ではなく境界の明示です / "Always use this" is unconditional and produces calls where the tool does not belong. What is needed is a stated boundary, not a mandate
- **E 不正解**: 表示言語との一致は利用者向けの話で、ツール定義はモデルが読むものです。選択精度を左右するのは記述の具体性であって言語ではありません / Display language is a user-facing concern; a tool definition is read by the model. Specificity, not language, determines selection accuracy

**参照 / Reference:** Tool Implementation — ツール説明文の記述、input_schema の設計
</details>

---

### 問題 5 / Question 5

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

ツールの実行中に「指定された顧客 ID が存在しない」というエラーが発生しました。現在の実装は、エラー時に `tool_result` を返さず、ループを打ち切ってユーザーに「エラーが発生しました」と表示しています。

A tool execution failed with "the specified customer ID does not exist." Your implementation currently returns no `tool_result` on error; it breaks the loop and shows the user "an error occurred."

**設問 / Question:**

より適切なエラー処理はどれですか？

Which is the better error handling?

- A) エラーを無視して空の結果を返す / Ignore the error and return an empty result
- B) 同じ引数で自動的に 3 回再試行する / Automatically retry three times with the same arguments
- C) エラー内容をそのまま例外として上位に投げ、ユーザーに生のスタックトレースを表示する / Rethrow the raw exception and show the user the stack trace
- D) **`tool_result` を `is_error: true` として返し、内容に「顧客 ID が存在しない」という具体的で行動可能なメッセージを入れる。Claude はこれを読んで、ID を確認し直す・別のツールで検索する・ユーザーに聞き返す、といった回復行動を取れる** / **Return the `tool_result` with `is_error: true` and an actionable message saying the customer ID does not exist. Claude can then recover — re-check the ID, search with another tool, or ask the user**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

ツールのエラーはループを止める理由ではなく、モデルに伝えるべき情報です。`tool_result` に `is_error: true` を付けて具体的なメッセージを返せば、Claude はそれを新しい事実として扱い、回復を試みます。ここで重要なのはメッセージの具体性です。「エラー」だけでは何をすればよいか分からず、「顧客 ID `C-1234` が存在しません」であれば ID の確認や再検索という次の行動につながります。ツールのエラーメッセージは、モデルに対する回復のヒントとして書きます。

A tool error is information to hand back to the model, not a reason to stop the loop. Returning `tool_result` with `is_error: true` and a specific message lets Claude treat it as a new fact and attempt recovery. Specificity is what matters: "error" gives nothing to act on, while "customer ID `C-1234` does not exist" leads to re-checking the ID or searching again. Write tool error messages as recovery hints for the model.

- **A 不正解**: 空の結果は「該当なし」と区別がつかず、モデルは存在しない ID を有効なものとして扱い続けます / An empty result is indistinguishable from "no matches," and the model goes on treating an invalid ID as valid
- **B 不正解**: 存在しない ID は決定論的に失敗します。再試行はコストとレイテンシを増やすだけで結果は変わりません / A nonexistent ID fails deterministically. Retrying only adds cost and latency
- **C 不正解**: スタックトレースの露出は情報漏えいのリスクがあり、モデルにも回復の機会を与えません / Exposing a stack trace risks information disclosure and gives the model no chance to recover

**参照 / Reference:** Tool Implementation — ツールのエラー処理、is_error と行動可能なメッセージ
</details>

---

### 問題 6 / Question 6

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

アプリケーションでは、社内 DB を叩く自作のツールと、Anthropic が提供するサーバー側ツール（ウェブ検索など）の両方を使いたいと考えています。

Your application wants to use both a custom tool that queries your internal database and an Anthropic-provided server-side tool such as web search.

**設問 / Question:**

クライアント側ツールとサーバー側ツールの違いとして正しいのはどれですか？

Which correctly describes the difference between client-side and server-side tools?

- A) **クライアント側ツールは、モデルが要求し、アプリケーションが実行して結果を返す。サーバー側ツールは Anthropic 側で実行され、実行結果が応答の中に含まれて返るため、アプリケーションがツールを実装する必要がない** / **A client-side tool is requested by the model and executed by your application, which returns the result. A server-side tool executes on Anthropic's side and its result comes back inside the response, so your application implements nothing**
- B) サーバー側ツールもアプリケーションが実装して結果を返す必要がある / Server-side tools also require your application to implement them and return results
- C) クライアント側ツールはブラウザでのみ動作する / Client-side tools run only in a browser
- D) 両者は同じリクエストに混在させられない / The two cannot be mixed in the same request

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「クライアント側」「サーバー側」は実行場所の話です。カスタムツール（クライアント側）は、モデルが `tool_use` で要求し、アプリケーションが自分の環境で実行して `tool_result` を返します。社内 DB のように Anthropic からアクセスできないリソースは、必然的にこちらになります。一方、ウェブ検索のようなサーバー側ツールは Anthropic 側で実行され、結果が応答に含まれて返るので、実装も往復も不要です。両者は同じ `tools` 配列に並べられます。

Client-side and server-side describe *where execution happens*. A custom (client-side) tool is requested via `tool_use` and executed by your application, which returns a `tool_result` — the only option for a resource Anthropic cannot reach, such as your internal database. A server-side tool like web search executes on Anthropic's side and its result arrives inside the response, so there is nothing to implement and no round trip. Both can appear in the same `tools` array.

- **B 不正解**: サーバー側ツールを自分で実装する必要はありません。それが「サーバー側」である意味です / You do not implement server-side tools — that is what "server-side" means
- **C 不正解**: 「クライアント側」はアプリケーションのプロセスを指し、ブラウザに限りません。サーバー上のバックエンドで実行するのが一般的です / "Client-side" means your application process, not a browser; it is usually your backend
- **D 不正解**: 混在は可能で、実際に組み合わせるのが一般的な構成です / Mixing them is supported and is a common configuration

**参照 / Reference:** Tool Implementation — クライアント側ツールとサーバー側ツール
</details>

---

### 問題 7 / Question 7

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

エージェントに `refund_payment`（返金実行）ツールを持たせます。返金は取り消せません。

You are giving an agent a `refund_payment` tool. Refunds cannot be undone.

**設問 / Question:**

このツールの扱いとして最も適切なのはどれですか？

Which is the most appropriate way to handle this tool?

- A) 返金額に上限を設ければ、承認は不要である / A cap on the refund amount removes the need for approval
- B) **ツールの実行前に人間の承認を挟む承認パターンを実装する。Claude が `tool_use` を返した時点でいったん止め、対象と金額を提示して承認を得てから実行する。承認は Claude に約束させるのではなく、アプリケーション側のフローとして強制する** / **Implement an approval pattern in front of the tool: pause when Claude returns the `tool_use`, present the target and amount for human approval, and execute only after it is granted. Enforce approval in the application flow rather than asking Claude to promise it**
- C) system プロンプトに「返金前に必ず確認すること」と書けば十分である / Writing "always confirm before refunding" in the system prompt is sufficient
- D) 返金ツールは持たせず、Claude に手順を説明させて人間に手作業でやらせる / Do not give the tool at all; have Claude explain the steps and let a human do it manually

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

不可逆な副作用を持つツールには承認ゲートを置きます。重要なのは、承認をどこで強制するかです。`tool_use` が返った時点でアプリケーションが実行を保留し、人間の判断を得てから初めて実際の処理を呼ぶ、という制御フローにすれば、モデルが指示に従わなかった場合でも返金は起こりません。これが決定論的な統制です。自動化の度合いは技術的な可能性ではなく、誤った場合の影響の大きさで決めます。

Tools with irreversible side effects get an approval gate, and where you enforce it is what matters. If the application holds execution when the `tool_use` arrives and only invokes the real operation after a human decides, no refund can happen even when the model ignores its instructions. That is a deterministic control. How much to automate is set by the cost of being wrong, not by what is technically possible.

- **A 不正解**: 上限は影響の大きさを抑えますが、誤った相手への返金という不可逆性は残ります。上限は承認の代替ではありません / A cap limits blast radius but not the irreversibility of refunding the wrong party. It is not a substitute for approval
- **C 不正解**: プロンプトによる約束は確率的な統制で、不可逆な操作の保証には足りません / A promise in the prompt is a probabilistic control, insufficient to guarantee anything irreversible
- **D 不正解**: 承認パターンを実装すれば自動化と統制は両立します。ツールを一切持たせないのは過剰で、エージェントの価値を失います / An approval pattern gets both automation and control. Withholding the tool entirely overshoots and gives up the agent's value

**参照 / Reference:** Tool Implementation — 承認パターン、不可逆操作の統制
</details>

---

### 問題 8 / Question 8

> サブスキル / Sub-skill: MCP Server Development (2.1%)

**シナリオ / Scenario:**

社内のドキュメント基盤を Claude アプリケーションから使えるようにするため、MCP サーバーを作ります。公開したいのは (1) 検索の実行、(2) 特定ドキュメントの本文の提供、(3) 「このドキュメントをレビューする」という定型の依頼テンプレート、の 3 種類です。

You are writing an MCP server so Claude applications can use your internal documentation platform. You want to expose three things: (1) running a search, (2) supplying the body of a specific document, (3) a canned "review this document" request template.

**設問 / Question:**

MCP のプリミティブへの割り当てとして正しいのはどれですか？

Which mapping onto MCP primitives is correct?

- A) 3 つとも tools として公開する / Expose all three as tools
- B) 3 つとも resources として公開する / Expose all three as resources
- C) **(1) は tools、(2) は resources、(3) は prompts として公開する** / **Expose (1) as a tool, (2) as a resource, and (3) as a prompt**
- D) (1) は prompts、(2) は tools、(3) は resources として公開する / Expose (1) as a prompt, (2) as a tool, and (3) as a resource

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

MCP の 3 つの主要プリミティブは役割が分かれています。**tools** はモデルが呼び出して副作用や計算を行うもの（検索の実行）。**resources** はクライアントがコンテキストに読み込むデータ（ドキュメント本文）。**prompts** は利用者が選んで起動する定型のテンプレート（レビュー依頼）。この区別は、誰が起動するかで整理すると覚えやすくなります。tools はモデルが、resources はアプリケーションが、prompts は利用者が起点になります。

MCP's three main primitives have distinct roles. **Tools** are invoked by the model to act or compute (running a search). **Resources** are data the client loads into context (a document body). **Prompts** are canned templates the user selects and triggers (the review request). The clearest way to keep them apart is by who initiates: the model for tools, the application for resources, the user for prompts.

- **A 不正解**: すべてを tools にすると、本文の取得までモデルの判断に依存し、テンプレートも利用者から選べません。プリミティブを分ける意味が失われます / Making everything a tool leaves even fetching a body to the model's discretion and makes the template unselectable by the user — the distinction exists for a reason
- **B 不正解**: resources は読み取り専用のデータで、検索の実行のような操作は表現できません / Resources are read-only data and cannot express an operation such as running a search
- **D 不正解**: 対応が入れ替わっています。検索は操作なので tool、ドキュメント本文はデータなので resource です / The mapping is swapped: a search is an operation (tool) and a document body is data (resource)

**参照 / Reference:** MCP Server Development — resources / tools / prompts の使い分け
</details>

---

### 問題 9 / Question 9

> サブスキル / Sub-skill: MCP Server Development (2.1%)

**シナリオ / Scenario:**

開発者のローカル環境で、そのマシンのファイルシステムと Git リポジトリを操作する MCP サーバーを動かします。サーバーは各開発者のマシン上でだけ動けばよく、ネットワーク越しに他者から使われる想定はありません。

You are running an MCP server on developers' local machines to operate on that machine's filesystem and Git repository. It only needs to run on each developer's own machine and is not meant to be reached over the network by anyone else.

**設問 / Question:**

通信パターンの選択として最も適切なのはどれですか？

Which communication pattern is most appropriate?

- A) HTTP サーバーとして公開し、`0.0.0.0` でリッスンする / Expose it as an HTTP server listening on `0.0.0.0`
- B) HTTP サーバーとして公開し、認証トークンで保護する / Expose it as an HTTP server protected by an auth token
- C) 各開発者が共有の中央サーバーに接続する / Have every developer connect to a shared central server
- D) **stdio トランスポートを使い、クライアントが子プロセスとしてサーバーを起動して標準入出力で通信する** / **Use the stdio transport: the client launches the server as a child process and communicates over standard input/output**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

ローカルで完結し、そのマシンのリソースだけを扱う MCP サーバーには stdio トランスポートが適しています。クライアントがサーバーを子プロセスとして起動し、標準入出力で JSON-RPC をやり取りするため、ネットワークポートを開かず、ネットワーク経由の攻撃面がそもそも存在しません。認証も、プロセスを起動したユーザーの権限で動くという形で自然に解決します。リモートから複数クライアントが使うサーバーであれば HTTP を選びますが、この要件ではその必要がありません。

For a server that is local by nature and touches only that machine's resources, stdio is the right transport. The client launches the server as a child process and exchanges JSON-RPC over standard input/output, so no network port is opened and there is no network attack surface to defend. Authentication resolves naturally: the server runs with the privileges of the user who launched it. HTTP is the choice when multiple remote clients must reach one server — not a requirement here.

- **A 不正解**: `0.0.0.0` でのリッスンは、ローカル用途のサーバーを同一ネットワーク上の全員に開放します。ファイルシステム操作ができるサーバーでこれは深刻なリスクです / Listening on `0.0.0.0` exposes a local-only server to everyone on the network — severe for a server that can operate on the filesystem
- **B 不正解**: 認証を足しても、不要なネットワーク面を作ってトークン管理という運用負担を追加しただけです / Adding auth still creates an unnecessary network surface plus the operational burden of token management
- **C 不正解**: 中央サーバーは各開発者のローカルファイルシステムにアクセスできません。要件を満たしません / A central server cannot reach each developer's local filesystem, so it does not meet the requirement

**参照 / Reference:** MCP Server Development — 通信パターン（stdio とネットワーク越しの接続）
</details>

---

### 問題 10 / Question 10

> サブスキル / Sub-skill: MCP Server Development (2.1%)

**シナリオ / Scenario:**

社内向けの MCP サーバーを、複数チームの Claude アプリケーションから使えるようにデプロイします。サーバーは基幹システムの読み取り API を呼び出します。

You are deploying an internal MCP server for use by several teams' Claude applications. The server calls read APIs on a core business system.

**設問 / Question:**

デプロイと統合の設計として適切なものを **2 つ選択してください**。

Select **2** appropriate deployment and integration design choices.

- A) **サーバーの認証情報を呼び出し元ごとに分け、MCP サーバー自身が基幹システムに対して最小権限で接続する。呼び出し元の識別と権限確認をサーバー側で行う** / **Separate credentials per caller and have the MCP server connect to the core system with least privilege, identifying the caller and checking authorization on the server side**
- B) 全チームが同じ管理者権限の認証情報を共有する / Have all teams share one administrator credential
- C) **公開するツールと resources のバージョンを管理し、後方互換を壊す変更は新しい名前で追加する。既存の呼び出し元が壊れないようにする** / **Version the exposed tools and resources, adding breaking changes under new names so existing callers keep working**
- D) 各チームがサーバーのコードをコピーして、それぞれ改造して使う / Have each team copy the server code and modify their own fork
- E) 基幹システムの API をそのまま 1 対 1 でツール化し、200 個のツールを公開する / Mirror the core system's API one-to-one and expose 200 tools

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, C**

**解説 / Explanation:**

共有される MCP サーバーは、実質的に社内 API と同じ性質を持ちます。したがって API と同じ規律が要ります。A は権限設計で、サーバーが基幹システムに対して持つ権限を最小限にし、呼び出し元の識別と権限確認をサーバーが担うことで、あるチームが他チームのデータに触れることを防ぎます。C はインターフェースの安定性で、複数の呼び出し元がいる以上、ツール名や引数の非互換な変更は他チームを壊します。名前を変えて追加し、旧版を段階的に廃止するのが安全な進め方です。

A shared MCP server is effectively an internal API and needs the same discipline. Choice A is the authorization design: keep the server's own privileges on the core system minimal and make the server responsible for identifying the caller and checking authorization, so one team cannot reach another team's data. Choice C is interface stability: with multiple callers, an incompatible change to a tool name or argument breaks other teams. Add under a new name and deprecate the old one gradually.

- **B 不正解**: 共有の管理者権限は最小権限の原則に反し、監査でどのチームが何をしたか分からなくなります / A shared admin credential violates least privilege and destroys the audit trail of which team did what
- **D 不正解**: フォークが増えると、セキュリティ修正がどこに適用されたか追えなくなります。共有サーバーにする意味が失われます / Proliferating forks makes it impossible to track where a security fix landed, defeating the point of a shared server
- **E 不正解**: ツール数が過大だとモデルの選択精度が落ちます。API の 1 対 1 の写像ではなく、ユースケースに沿って統合・抽象化したツールセットを設計します / Too many tools degrades selection accuracy. Design a tool set around use cases, consolidated and abstracted, rather than mirroring the API one-to-one

**参照 / Reference:** MCP Server Development — デプロイ、統合、権限設計、インターフェースのバージョン管理
</details>

---

### 問題 11 / Question 11

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

アプリケーションに「最新の公開情報を調べて回答する」機能を追加したいと考えています。Anthropic はサーバー側のウェブ検索ツールを提供しています。チームからは「検索エンジン API を契約して自作ツールを作るべきだ」という案も出ています。特殊な検索要件はありません。

You want to add "look up current public information and answer" to your application. Anthropic provides a server-side web search tool. A teammate proposes contracting a search-engine API and building a custom tool instead. There are no special search requirements.

**設問 / Question:**

最初に取るべき選択はどれですか？

Which choice should you make first?

- A) **組み込みのウェブ検索ツールをまず使う。実装・運用・課金契約が不要で、要件を満たすかを短期間で確認できる。組み込みで足りないと分かった時点で自作を検討する** / **Start with the built-in web search tool: nothing to implement, operate, or contract for, and you can confirm whether it meets the requirement quickly. Consider a custom tool once you find the built-in insufficient**
- B) 自作ツールを作る。将来の柔軟性が高いから / Build the custom tool, for future flexibility
- C) 両方を実装して Claude に選ばせる / Implement both and let Claude choose
- D) MCP サーバーとして検索機能を実装する / Implement search as an MCP server

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

組み込みツール・カスタムツール・Skills・MCP の使い分けは、「所有するコードが最も少ない選択肢から始める」が基本です。組み込みのサーバー側ツールは、実装も運用も課金契約も不要で、要件を満たすかどうかを最短で検証できます。特殊な要件がない現時点で自作を選ぶのは、根拠のない先行投資です。実際に組み込みでは足りない要件（特定ドメインへの限定、社内インデックスとの統合など）が判明したときに、その要件を根拠として自作に移ればよく、その判断は後からでも遅くありません。

Choosing among built-in tools, custom tools, Skills, and MCP starts from the option with the least code you own. A built-in server-side tool requires no implementation, no operations, and no billing contract, so it validates the requirement fastest. With no special requirement identified, building your own is unjustified up-front investment. If the built-in later proves insufficient — restricting to certain domains, integrating an internal index — you move then, with the requirement as evidence. That decision loses nothing by waiting.

- **B 不正解**: 「将来の柔軟性」は具体的な要件ではありません。使われるか分からない柔軟性のために、実装と運用の負担を今引き受けることになります / "Future flexibility" is not a requirement. You take on implementation and operational burden now for flexibility that may never be used
- **C 不正解**: 同じ目的のツールを 2 つ並べるとモデルの選択が不安定になり、評価も難しくなります / Two tools for the same purpose destabilizes selection and makes evaluation harder
- **D 不正解**: MCP は複数のクライアントから再利用する場合や既存システムを公開する場合に効きます。単一アプリの検索機能を MCP 化しても、組み込みで済む処理に層を足すだけです / MCP pays off for reuse across clients or exposing an existing system. Wrapping a single app's search in MCP just adds a layer over something the built-in already does

**参照 / Reference:** Agentic Customization — 組み込みツールとカスタムツールのトレードオフ
</details>

---

### 問題 12 / Question 12

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

社内には「経費精算の申請書を、経理部門の 12 ページの規程に従って書く」という定型業務があります。判断の大部分は規程の読み解きと文章作成で、外部システムの呼び出しは最後に 1 回、申請 API を叩くだけです。現在はこの規程全体を毎回 system プロンプトに貼り付けています。

Your organization has a recurring task: writing expense claims in accordance with a 12-page finance policy. Most of the work is interpreting the policy and drafting text; an external system is called exactly once at the end, to submit via the claims API. Today the entire policy is pasted into the system prompt on every request.

**設問 / Question:**

最も適切な構成はどれですか？

Which is the most appropriate structure?

- A) 規程全体をツールの `description` に移す / Move the whole policy into a tool `description`
- B) **規程の解釈と作成手順を Skill として切り出し、必要なときにだけ読み込まれるようにする。申請 API の呼び出しはツールとして定義する** / **Extract the policy interpretation and drafting procedure into a Skill that is loaded only when needed, and define the claims API call as a tool**
- C) 規程を MCP サーバーの prompts として公開する / Expose the policy as MCP prompts
- D) 規程を毎回コンテキストに入れたまま、プロンプトキャッシュだけを有効にする / Keep the policy in context every time and just enable prompt caching

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

この業務は「手順と知識」であって「外部システムの操作」ではありません。Skills は、まさにこうした手順・知識・専門性をパッケージ化し、関連する場面でだけ読み込ませる仕組みです。常時 system プロンプトに置く必要がなくなり、他の業務のリクエストでは 12 ページ分のコンテキストを消費しなくなります。一方、申請 API の呼び出しは副作用のある外部操作なので、これはツールとして定義するのが適切です。Skill とツールは競合せず、役割が違います。

This task is procedure and knowledge, not operating an external system. Skills exist to package exactly that — procedures, knowledge, expertise — and load it only in relevant situations. It no longer has to sit in the system prompt, so requests for other tasks stop paying for twelve pages of context. The claims API call, being an external operation with a side effect, is properly a tool. Skills and tools do not compete; they play different roles.

- **A 不正解**: `description` はツール選択のための短い説明で、12 ページの規程を置く場所ではありません。全ツールの説明は常にコンテキストに入るため、かえって悪化します / A `description` is short text for tool selection, not a home for a twelve-page policy. All tool descriptions are always in context, so this makes it worse
- **C 不正解**: MCP prompts は利用者が起動する定型テンプレートで、常時参照される手順知識の格納先としては目的が違います。単一アプリ内の手順なら MCP を挟む必要もありません / MCP prompts are user-triggered templates, a different purpose from housing procedural knowledge; and a procedure inside one application needs no MCP layer
- **D 不正解**: キャッシュはコストとレイテンシを下げますが、無関係なリクエストでも規程がコンテキストを占める構造は変わりません / Caching lowers cost and latency but does not change the fact that the policy occupies context on unrelated requests

**参照 / Reference:** Agentic Customization — Skills とツールの使い分け
</details>

---

### 問題 13 / Question 13

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

社内のチケット管理システムを、Claude Code・社内チャットボット・バッチ処理の 3 つのアプリケーションから操作できるようにしたいと考えています。3 つはそれぞれ別のチームが開発しており、必要な操作（チケット検索・作成・更新）は共通です。

You want three applications — Claude Code, an internal chatbot, and a batch job — to operate your ticketing system. Each is built by a different team, and all three need the same operations: search, create, and update tickets.

**設問 / Question:**

最も適切なアプローチはどれですか？

Which is the most appropriate approach?

- A) 3 チームがそれぞれカスタムツールを実装する / Have each of the three teams implement its own custom tools
- B) チケット管理システムの操作手順を Skill として配布する / Distribute the ticketing procedures as a Skill
- C) **チケット管理システムを公開する MCP サーバーを 1 つ作り、3 つのアプリケーションがそれぞれ MCP クライアントとして接続する** / **Build one MCP server that exposes the ticketing system and have all three applications connect to it as MCP clients**
- D) 組み込みツールで代替できないか検討する / See whether a built-in tool can substitute

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

MCP が最も効くのは、**同じ外部システムへの接続を複数のクライアントで再利用したい**ときです。この条件がまさに当てはまります。1 つのサーバーに実装を集約すれば、チケット API の仕様変更・認証方式の変更・セキュリティ修正をサーバー側で 1 回対応するだけで 3 つのアプリケーションに反映されます。標準プロトコルなので、将来 4 つ目のクライアントが増えても接続するだけで済みます。カスタムツールとの違いは、再利用と保守の集約にあります。

MCP pays off most when **one connection to an external system must be reused by multiple clients** — exactly the situation here. Centralizing the implementation in one server means an API change, an auth change, or a security fix is handled once and reaches all three applications. Because the protocol is standard, a fourth client later just connects. That reuse and consolidation of maintenance is precisely what distinguishes MCP from a custom tool.

- **A 不正解**: 3 重の実装は、仕様変更のたびに 3 か所の修正が必要で、必ず食い違います。認証情報の扱いも 3 通りに分散します / Three implementations means three fixes for every change, and they will diverge. Credential handling scatters three ways as well
- **B 不正解**: Skill は手順と知識のパッケージで、外部システムへの接続そのものを提供しません。API を呼ぶ実装は別に要ります / A Skill packages procedure and knowledge; it does not provide the connection. Something still has to call the API
- **D 不正解**: 社内独自のチケット管理システムに対応する組み込みツールは存在しません / No built-in tool exists for a proprietary internal ticketing system

**参照 / Reference:** Agentic Customization — MCP とカスタムツールのトレードオフ
</details>

---

### 問題 14 / Question 14

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

エージェントに接続した MCP サーバーとカスタムツールが増え、公開されているツールが合計 90 個になりました。ツールの選択ミスが目立ち始め、system プロンプトとツール定義だけで入力トークンの大部分を占めるようになっています。

The MCP servers and custom tools attached to your agent have grown to 90 exposed tools. Tool-selection mistakes are becoming noticeable, and the system prompt plus tool definitions now account for most of the input tokens.

**設問 / Question:**

最も適切な対処はどれですか？

Which is the most appropriate response?

- A) より大きなコンテキストウィンドウのモデルに変える / Move to a model with a larger context window
- B) 各ツールの `description` を 1 行に短縮する / Shorten every tool `description` to one line
- C) system プロンプトに「ツールは慎重に選ぶこと」と書く / Add "choose tools carefully" to the system prompt
- D) **タスクに実際に必要なツールだけを露出させる。ユースケースごとにツールセットを絞り込み、重複するツールを統合し、使われていないツールを外す。必要なら役割ごとにサブエージェントを分けて、それぞれが扱うツールを限定する** / **Expose only the tools a task actually needs: scope the tool set per use case, consolidate overlapping tools, and remove unused ones. Where warranted, split roles into subagents, each with a limited tool set**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

ツールが増えるほど選択は難しくなります。似た機能のツールが並ぶと境界が曖昧になり、選択ミスが増えます。同時に、全ツールの定義は毎リクエストのコンテキストを占め続けます。対処は数を減らすことです。ユースケース単位で必要なツールだけを渡し、重複を統合し、使われていないものを外します。役割が明確に分かれるなら、サブエージェントごとにツールセットを分離するのが有効で、これは各エージェントのコンテキストを小さく保つことにもなります。ツールセットの構築は、足し算ではなく設計です。

More tools make selection harder: similar tools blur the boundaries between them and mistakes rise, while every definition occupies context on every request. The fix is to reduce the number. Pass only what a use case needs, consolidate overlapping tools, and drop unused ones. Where roles separate cleanly, giving each subagent its own tool set works well and keeps each agent's context small. Tool-set construction is a design activity, not accumulation.

- **A 不正解**: ウィンドウを広げても選択の難しさは変わりません。90 個から 1 つを選ぶ問題は、コンテキスト長の問題ではありません / A larger window does not make selection easier. Choosing one of 90 is not a context-length problem
- **B 不正解**: 説明を削ると選択に必要な情報が失われ、選択ミスが増えます。トークンは減っても品質は悪化します / Cutting descriptions removes the information selection depends on, so mistakes increase. Fewer tokens, worse quality
- **C 不正解**: 一般的な注意喚起は、90 個の中から正しい 1 つを選ぶ助けになりません / A generic exhortation does not help pick the right one out of 90

**参照 / Reference:** Agentic Customization — ツールセット構築、露出するツールの絞り込み
</details>

---

### 問題 15 / Question 15

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

「問い合わせ文面を、社内の敬語ガイドに沿った表現に書き換える」という機能の要望が来ました。必要な情報はすべて入力テキストと 1 ページのガイドに含まれており、外部システムへのアクセスも、複数ステップの判断もありません。担当者は MCP サーバーを立てる設計を提案しています。

A request came in for a feature that rewrites inquiry text to match the company's honorific-language guide. Everything needed is in the input text and a one-page guide; there is no external system access and no multi-step decision-making. The assignee has proposed standing up an MCP server.

**設問 / Question:**

最も適切な判断はどれですか？

Which is the most appropriate judgment?

- A) **MCP もツールも不要で、ガイドを含めた単一の API 呼び出しで実現する。外部システムへのアクセスも多段の判断もない以上、ツールや MCP を挟む理由がない** / **Neither MCP nor tools are needed: a single API call including the guide does it. With no external system access and no multi-step decisions, there is no reason to introduce a tool or MCP layer**
- B) MCP サーバーを立てる。将来他のアプリからも使えるから / Stand up the MCP server, since other applications might use it later
- C) カスタムツールとして実装する。機能は独立させるべきだから / Implement it as a custom tool, since features should be isolated
- D) Skill として実装する。ガイドは知識だから / Implement it as a Skill, since the guide is knowledge

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

適切なアプローチを選ぶことには、「どれも使わない」という選択肢も含まれます。この要件は入力を変換して出力するだけの単一 LLM 呼び出しで、外部システムも多段の判断もありません。ツールも MCP も、実装・デプロイ・監視の対象が増えるコストを伴います。そのコストを払う理由が要件の側にないなら、払うべきではありません。トレードオフの評価とは、常に何かを導入することではなく、導入しない判断も含めて要件に照らすことです。

Selecting the appropriate approach includes selecting none of them. This requirement is a single LLM call that transforms input into output — no external system, no multi-step decisions. Tools and MCP each add something to implement, deploy, and monitor. If nothing in the requirement justifies that cost, do not pay it. Evaluating tradeoffs means testing every option against the requirement, including the option of adding nothing.

- **B 不正解**: 「将来使うかもしれない」は要件ではありません。実際に 2 つ目のクライアントが現れた時点で MCP 化を検討すれば十分です / "Might be used later" is not a requirement. Consider MCP when a second client actually appears
- **C 不正解**: ツールは外部システムの操作や決定論的な計算のための仕組みです。テキスト変換はモデル自身の仕事で、ツールを挟む必然性がありません / Tools exist for operating external systems or deterministic computation. Text transformation is the model's own work and needs no tool
- **D 不正解**: Skill は、常時は不要だが特定の場面で必要になる手順や知識をパッケージ化するものです。この機能専用のアプリケーションで、1 ページのガイドを常に使うのであれば、切り出す利点がありません / A Skill packages procedure or knowledge needed only in specific situations. In an application dedicated to this feature, where the one-page guide is always used, there is nothing to gain by extracting it

**参照 / Reference:** Agentic Customization — アプローチ選択、導入しない判断
</details>

---

### 問題 16 / Question 16

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

新機能ごとに「ツールにするか、Skill にするか、MCP にするか」の議論が毎回一から始まり、結論が担当者によってばらつきます。判断の指針をチームで整理することにしました。

Every new feature restarts the same debate — tool, Skill, or MCP — and the conclusion varies by who is deciding. The team decides to write down a decision guideline.

**設問 / Question:**

指針として妥当なものを **2 つ選択してください**。

Select **2** sound guidelines.

- A) 迷ったら常に MCP にする。最も汎用的だから / When in doubt, always choose MCP, as it is the most general
- B) 迷ったら常にカスタムツールにする。最も単純だから / When in doubt, always choose a custom tool, as it is the simplest
- C) **外部システムに対する操作や決定論的な計算が必要ならツール。手順・知識・専門性をモデルに与えたいなら Skill。同じ接続を複数のクライアントで再利用するなら MCP、と要件の性質で分ける** / **Split by the nature of the requirement: a tool when an external system must be operated or a deterministic computation performed; a Skill to give the model procedure, knowledge, or expertise; MCP when one connection is reused across multiple clients**
- D) 実装コストが最も低いものを常に選ぶ / Always pick whichever has the lowest implementation cost
- E) **まず組み込みで足りるかを確認し、足りない場合にのみ自作に進む。所有するコードが少ない選択肢から検討する** / **Check first whether a built-in suffices and move to building your own only when it does not — work up from the option with the least code you own**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C, E**

**解説 / Explanation:**

判断がばらつくのは、選択の基準が要件の性質に紐づいていないからです。C は 3 つのアプローチを、それぞれが解く問題の種類で切り分けます。この切り分けなら、要件を見れば答えが決まるので担当者による差が出ません。E は検討の順序で、所有するコードが少ない選択肢から確認していくことで、不要な実装を早い段階で排除できます。C が「どれか」を、E が「どこから検討するか」を決め、両者は補完的です。

Decisions vary because the criterion is not tied to the nature of the requirement. Guideline C separates the three approaches by the kind of problem each solves, so the requirement determines the answer and the decider stops mattering. Guideline E sets the order of evaluation: working up from the option with the least code you own eliminates unnecessary implementation early. C decides *which*; E decides *where to start*. They are complementary.

- **A 不正解**: MCP は再利用が要件のときに効く仕組みで、単一クライアントでは実装・デプロイ・運用の層を足すだけです / MCP earns its place when reuse is a requirement; for a single client it only adds layers to implement, deploy, and operate
- **B 不正解**: カスタムツールが常に最も単純とは限りません。手順知識の付与や複数クライアントでの再利用には向きません / A custom tool is not always simplest, and it fits neither delivering procedural knowledge nor reuse across clients
- **D 不正解**: 実装コストは判断材料の 1 つにすぎません。最も安い選択が要件を満たさなければ、後から作り直すコストのほうが大きくなります / Implementation cost is only one input. If the cheapest option does not meet the requirement, rebuilding later costs more

**参照 / Reference:** Agentic Customization — アプローチ選択の判断基準
</details>

---

### 問題 17 / Question 17

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

あるチームが作った MCP サーバーを、別のチームのアプリケーションでも使いたいと考えています。そのサーバーは 30 個のツールを公開しており、そのうち利用したいのは 4 個だけです。

Your team wants to use an MCP server built by another team. It exposes 30 tools, of which you need only four.

**設問 / Question:**

最も適切な対応はどれですか？

Which is the most appropriate approach?

- A) 30 個すべてを露出させる。使わないツールがあっても害はない / Expose all 30 — unused tools do no harm
- B) **クライアント側で、使用するツールを 4 個に絞って露出させる。MCP サーバーが公開しているツールと、自分のエージェントに渡すツールセットは別の判断である** / **Filter client-side and expose only the four you use. What the MCP server publishes and what you hand your agent are separate decisions**
- C) サーバーを作ったチームに、ツールを 4 個に減らしてもらう / Ask the owning team to reduce the server to four tools
- D) サーバーをフォークして、不要なツールを削除した版を自分たちで運用する / Fork the server and operate your own copy with the unneeded tools deleted

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

MCP サーバーが何を公開するかと、自分のエージェントに何を渡すかは別の判断です。サーバーは利用者全体に対して機能を提供するので 30 個あってよく、各クライアントは自分のユースケースに必要な分だけを選んで露出させます。これにより、使わないツールがコンテキストを占めることも、選択の候補に紛れ込むこともなくなります。問題 14 のツールセット構築の原則を、MCP 経由のツールにも同じように適用する、ということです。

What a server publishes and what you hand your agent are separate decisions. The server serves all its consumers, so 30 tools is fine; each client selects the subset its use case needs. That keeps unused tools from occupying context and from muddying selection. It is the tool-set construction principle from Question 14, applied identically to tools that arrive via MCP.

- **A 不正解**: 使わないツールも定義がコンテキストを占め、選択の候補にも入ります。無害ではありません / Unused tools still cost context for their definitions and still enter the selection pool. They are not harmless
- **C 不正解**: 他のチームが使っているツールを削れば、そのチームが壊れます。共有サーバーに自分の都合を反映させるのは不適切です / Removing tools other teams use breaks them. Bending a shared server to one consumer's convenience is inappropriate
- **D 不正解**: フォークすると、本家のセキュリティ修正や機能追加が自動では入らなくなります。露出の絞り込みだけならクライアント側で済む話です / A fork stops receiving upstream security fixes and features automatically, when filtering is a client-side matter

**参照 / Reference:** Agentic Customization — MCP 由来ツールの露出制御、ツールセットの絞り込み
</details>

---

## 発展 / Advanced

### 問題 18 / Question 18

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

社内エージェントのツールセットを 90 個から絞り込む作業を始めます。「どのツールが使われているか分からない」「消したら誰かが困るかもしれない」という理由で議論が止まっています。エージェントの呼び出しログは保存されています。

You are starting to trim your internal agent's 90-tool set. The discussion is stuck on "we don't know which tools are used" and "someone might depend on the ones we remove." Agent invocation logs are retained.

**設問 / Question:**

最も適切な進め方はどれですか？

Which is the most appropriate approach?

- A) 直感で不要そうなツールから削除し、苦情が出たら戻す / Delete the ones that feel unnecessary and restore them if anyone complains
- B) 削除はせず、ツールを増やさないルールだけを設ける / Delete nothing and simply institute a rule against adding more
- C) **ログからツールごとの呼び出し頻度・成功率・選択ミスの発生状況を集計し、事実に基づいて統廃合の候補を選ぶ。評価データセットで統廃合前後の選択精度とタスク成功率を比較し、劣化がないことを確認してから段階的に外す** / **Aggregate per-tool invocation frequency, success rate, and misselection incidents from the logs, choose consolidation candidates on that evidence, compare selection accuracy and task success rate before and after on an evaluation dataset, and remove in stages once no regression is observed**
- D) 全ツールを一度に削除し、エラーが出たものだけ戻す / Delete every tool at once and restore whatever errors

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

議論が止まっている原因は情報がないことなので、まず事実を集めます。呼び出し頻度は「使われているか」を、成功率は「役に立っているか」を、選択ミスの発生状況は「他のツールと紛らわしいか」を示します。呼び出しが多くても成功率が低いツールは、統合や説明文の見直しの対象です。そして削除の影響は評価データセットで測ってから、段階的に適用します。これは問題 14 の「ツールセットの構築は設計である」を、既存の 90 個に対して実行する手順にしたものです。

The discussion is stuck for want of information, so gather facts first. Invocation frequency shows whether a tool is used, success rate whether it helps, and misselection incidents whether it is confusable with its neighbors. A tool called often but succeeding rarely is a candidate for consolidation or a rewritten description. Then measure the impact of removal on an evaluation dataset and apply in stages. This is the "tool-set construction is design" principle from Question 14, turned into a procedure for an existing set of 90.

- **A 不正解**: 直感に基づく削除は、低頻度でも重要なツール（四半期末の締め処理など）を落とします。苦情が出るまで気づかないのは統制として不十分です / Intuition-driven deletion drops low-frequency but critical tools — a quarter-end close, say. Finding out via complaints is not adequate control
- **B 不正解**: 増やさないだけでは 90 個の現状が固定されます。選択ミスとコンテキスト消費という現在の問題は解決しません / Freezing at 90 leaves the present problems — misselection and context consumption — unsolved
- **D 不正解**: 全削除は本番のエージェントを機能停止させます。エラーが出ないだけの静かな劣化（選択肢が減ったことで別の不適切なツールを選ぶ）も検出できません / Deleting everything takes the production agent down, and it cannot detect silent degradation — picking a different, wrong tool because the right one is gone raises no error

**参照 / Reference:** Tool Implementation — ツールセット構築、利用実態に基づく統廃合と検証
</details>

---

### 問題 19 / Question 19

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

`create_shipment`（出荷指示の作成）ツールで、ネットワークのタイムアウトが時々発生します。現在はタイムアウト時に同じ引数で自動再試行しています。運用チームから、同一注文に対する出荷指示が二重に作成されている事例が報告されました。

The `create_shipment` tool occasionally hits a network timeout. Today you automatically retry with the same arguments. Operations reports cases where the same order got two shipment instructions.

**設問 / Question:**

最も適切な対処はどれですか？

Which is the most appropriate remedy?

- A) タイムアウト時間を延ばして、タイムアウト自体を減らす / Increase the timeout so timeouts happen less often
- B) 再試行をやめ、タイムアウト時はエラーとしてユーザーに返す / Stop retrying and surface timeouts to the user as errors
- C) 再試行の前に、直近の出荷指示を検索して重複がないか Claude に確認させる / Before retrying, have Claude search recent shipments and judge whether it is a duplicate
- D) **ツールに冪等性キーを導入する。呼び出しごとに注文単位の一意なキーを生成して送り、下流のシステムが同じキーの要求を 1 回だけ処理するようにする。これにより再試行が安全になり、タイムアウト時に再試行を続けられる** / **Introduce an idempotency key: generate a unique per-order key on each call and have the downstream system process a given key exactly once. Retries then become safe and can continue on timeout**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

タイムアウトは「処理されなかった」ことを意味しません。要求は届いて処理され、応答だけが失われたという状態がありえます。この曖昧さがある限り、副作用を持つ操作の再試行は必ず重複を生みます。冪等性キーは、この曖昧さを下流側で解消する標準的な手段です。同じキーの要求は 1 回しか処理されないため、再試行しても結果は 1 件のままです。エージェントが自律的に再試行する構成では、副作用を持つツールに冪等性を持たせることが前提条件になります。

A timeout does not mean the request was not processed. The request may have arrived and completed with only the response lost. While that ambiguity exists, retrying an operation with side effects will produce duplicates. An idempotency key is the standard way to resolve the ambiguity downstream: a repeated key is processed once, so retries leave exactly one shipment. Where an agent retries autonomously, idempotency on side-effecting tools is a precondition, not an optimization.

- **A 不正解**: タイムアウトの頻度が下がっても、発生したときの重複はそのまま残ります。確率を下げるだけで、正しさは保証されません / Fewer timeouts still duplicate when one occurs. Lowering the probability does not establish correctness
- **B 不正解**: 一時的なネットワーク障害で出荷指示が失敗するのは可用性の低下です。再試行を安全にする方法があるのに諦めています / Losing a shipment to a transient network fault is an availability regression, and it gives up when a safe retry is available
- **C 不正解**: 検索と判断をモデルに委ねると、確率的な統制になります。加えて、直前の要求が処理中でまだ検索に現れない競合状態を防げません / Delegating the check to the model is a probabilistic control, and it cannot prevent the race where the prior request is still in flight and not yet visible to a search

**参照 / Reference:** Tool Implementation — 副作用のあるツールの冪等性、再試行の安全性
</details>

---

### 問題 20 / Question 20

> サブスキル / Sub-skill: Tool Implementation (4.4%)

**シナリオ / Scenario:**

金融機関で、顧客口座の照会・取引の下書き作成・取引の実行を行うエージェントを構築します。監査部門からは「エージェントが何をどの権限で実行したかを、後から完全に再現できること」を要求されています。エージェントは複数の担当者が異なる権限で利用します。

At a financial institution you are building an agent that looks up customer accounts, drafts transactions, and executes them. Audit requires that what the agent did, and under whose authority, be fully reconstructable after the fact. Multiple staff with different privilege levels use the agent.

**設問 / Question:**

エージェンティックハーネスの設計として適切なものを **2 つ選択してください**。

Select **2** appropriate designs for the agentic harness.

- A) **ツールのディスパッチ層で、呼び出し元の担当者の権限を毎回確認してから実行する。モデルが要求したツールでも、その担当者の権限で許可されていなければ実行せず、拒否を `tool_result` として返す** / **Check the invoking staff member's privileges in the tool dispatch layer on every call. Even when the model requests a tool, do not execute it unless that person's privileges permit it — return the denial as a `tool_result`**
- B) 権限に応じて system プロンプトを書き分け、許可されていない操作を依頼しないよう指示する / Vary the system prompt by privilege level and instruct the model not to request unpermitted operations
- C) 監査ログはモデルの最終回答だけを記録する / Log only the model's final answer for audit
- D) 取引実行ツールは、権限のある担当者の場合のみツール定義に含める / Include the transaction-execution tool in the tool definitions only for privileged staff
- E) **すべての `tool_use` 要求、実行の可否判断とその根拠、実際に渡した引数、返した `tool_result`、承認者を、相関 ID とともに追記専用のログに記録する** / **Record every `tool_use` request, the allow/deny decision and its basis, the arguments actually passed, the returned `tool_result`, and the approver — with a correlation ID, to an append-only log**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, E**

**解説 / Explanation:**

要求は 2 つあります。「権限どおりに実行されること」と「後から再現できること」です。A はディスパッチ層での権限確認で、モデルの判断とは独立した決定論的な統制です。モデルがプロンプトインジェクションを受けても、あるいは単に誤ったツールを選んでも、権限のない実行は起こりません。拒否を `tool_result` として返すのも重要で、これによりモデルは状況を理解して代替行動を取れます。E は監査要求そのもので、要求・判断・引数・結果・承認者が揃って初めて「何がどの権限で起きたか」を再現できます。追記専用にするのは、ログ自体の改ざんを防ぐためです。

Two requirements: execution must match privileges, and the whole thing must be reconstructable. Choice A puts the privilege check in the dispatch layer — a deterministic control independent of the model's judgment. Whether the model is subverted by prompt injection or simply picks the wrong tool, an unauthorized execution cannot occur, and returning the denial as a `tool_result` lets the model understand the situation and take an alternative path. Choice E is the audit requirement itself: only request, decision, arguments, result, and approver together reconstruct what happened under whose authority. Append-only storage protects the log from being rewritten.

- **B 不正解**: プロンプトでの指示は確率的な統制で、権限の強制にはなりません。監査部門に提示できる統制でもありません / A prompt instruction is a probabilistic control, not privilege enforcement, and not a control you can present to audit
- **C 不正解**: 最終回答だけでは、どのツールがどの引数で実行されたかが残りません。再現要求を満たしません / A final answer alone does not record which tool ran with which arguments and does not satisfy the reconstruction requirement
- **D 不正解**: 露出の制御は有用ですが、単独では不十分です。ツール定義の組み立てを間違えれば権限のない実行が通ってしまうため、実行時の権限確認が別に要ります / Filtering exposure helps but is not sufficient alone: a mistake assembling the tool definitions lets an unauthorized execution through, so a runtime check is still required

**参照 / Reference:** Tool Implementation — エージェンティックハーネスのディスパッチ、権限確認、監査証跡
</details>

---

### 問題 21 / Question 21

> サブスキル / Sub-skill: MCP Server Development (2.1%)

**シナリオ / Scenario:**

社内の MCP サーバーを、SaaS 製品の一部として社外の顧客にも提供することになりました。顧客ごとにデータは完全に分離されている必要があり、顧客は自分の Claude アプリケーションからこのサーバーに接続します。現在はローカル開発用に stdio で動いており、認証は環境変数の API キー 1 本です。

Your internal MCP server will now be offered to external customers as part of a SaaS product. Customer data must be fully isolated, and customers connect from their own Claude applications. Today it runs over stdio for local development, authenticated by a single API key in an environment variable.

**設問 / Question:**

移行にあたって最も重要な設計変更はどれですか？

Which is the most important design change for this transition?

- A) **ネットワーク越しの接続を受けるトランスポートに変更したうえで、接続ごとに顧客を認証し、その認証結果に紐づくテナントのデータだけを返すようサーバー内部でスコープを強制する。単一の共有キーではなく顧客ごとの資格情報にし、ツールの実行結果もテナント境界で絞り込む** / **Move to a network-capable transport, authenticate the customer per connection, and enforce inside the server that only that tenant's data is returned. Replace the single shared key with per-customer credentials and scope tool results to the tenant boundary**
- B) stdio のまま、顧客のマシンでサーバーを動かしてもらう / Keep stdio and have customers run the server on their own machines
- C) 顧客ごとにサーバーのインスタンスを分けるだけでよい / Simply run a separate server instance per customer
- D) 公開するツールの `description` に「他の顧客のデータを返さないこと」と明記する / State in each tool `description` that data from other customers must not be returned

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

ローカル用サーバーをマルチテナントの SaaS にするときに変わるのは、トランスポートだけではなく信頼境界です。stdio では「プロセスを起動したユーザー = 利用者」だったので認証が暗黙に成立していましたが、ネットワーク越しでは接続ごとに誰なのかを確認しなければなりません。そして最も重要なのは、認証した顧客に紐づくテナントスコープを、サーバー内部のデータアクセス経路で強制することです。ツールの引数に顧客 ID を含めて信じる設計にすると、引数を書き換えるだけで他テナントのデータが取れてしまいます。スコープは認証結果から導出し、要求からは受け取りません。

Turning a local server into multi-tenant SaaS changes more than the transport — it changes the trust boundary. Under stdio, "whoever launched the process" was the user and authentication was implicit; over the network you must establish identity per connection. Most important, the tenant scope derived from that identity must be enforced inside the server's data-access path. A design that takes a customer ID in the tool arguments and trusts it lets an attacker read another tenant by editing an argument. Derive the scope from the authenticated identity, never from the request.

- **B 不正解**: 顧客のマシンで動かせば、サーバーが持つ基幹システムへの資格情報を顧客に渡すことになります。テナント分離どころか、全テナントへのアクセスを配布することになりかねません / Running it on the customer's machine hands them the server's credentials to the core system — not tenant isolation but potentially distributing access to every tenant
- **C 不正解**: インスタンス分離は分離の一手段ですが、認証と認可の設計を置き換えるものではありません。どのインスタンスに誰が接続してよいかは依然として確認が必要です / Instance separation is one isolation mechanism, not a replacement for authentication and authorization. Who may connect to which instance still has to be established
- **D 不正解**: `description` はモデルへの説明であり、サーバーのアクセス制御ではありません。データ分離をモデルの遵守に委ねることはできません / A `description` is guidance to the model, not server-side access control. Data isolation cannot rest on the model complying

**参照 / Reference:** MCP Server Development — デプロイ、認証、マルチテナントの分離
</details>

---

### 問題 22 / Question 22

> サブスキル / Sub-skill: MCP Server Development (2.1%)

**シナリオ / Scenario:**

データ分析基盤を公開する MCP サーバーを設計しています。基盤には 400 のテーブルがあり、それぞれにスキーマ定義があります。「エージェントがテーブル構造を理解してクエリを書けるようにしたい」という要件があります。全テーブルのスキーマを合計すると数十万トークンになります。

You are designing an MCP server over a data platform with 400 tables, each with a schema definition. The requirement is that an agent should understand table structure well enough to write queries. All schemas together run to hundreds of thousands of tokens.

**設問 / Question:**

サーバー設計として適切なものを **2 つ選択してください**。

Select **2** appropriate server designs.

- A) 400 テーブル分のスキーマをすべて resources として一度にコンテキストへ読み込ませる / Load all 400 schemas into context as resources at once
- B) **テーブルの一覧と概要だけを軽量に提供し、特定テーブルの詳細スキーマは必要になった時点で取得するツールを用意する（段階的な探索）** / **Provide a lightweight list of tables with summaries, plus a tool that fetches a specific table's detailed schema when it is actually needed (progressive discovery)**
- C) スキーマを 400 個のツールとして公開する（テーブルごとに 1 ツール） / Expose the schemas as 400 tools, one per table
- D) スキーマは提供せず、エージェントに試行錯誤でクエリを書かせる / Provide no schemas and let the agent discover the structure by trial and error
- E) **よく使われる少数のテーブルとその関連（結合キーなど）を、あらかじめまとめた「よく使うデータモデル」として提供し、そこから外れる場合にのみ全体探索に降りる** / **Ship a curated "common data model" covering the handful of frequently used tables and their relationships, such as join keys, and fall back to full discovery only outside it**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, E**

**解説 / Explanation:**

大規模なスキーマをエージェントに扱わせる際の原則は、必要になったものだけをコンテキストに入れることです。B の段階的な探索は、一覧から目星をつけて詳細を取りに行くという人間のアナリストと同じ動き方をさせるもので、コンテキストの消費を実際に使うテーブル分に抑えます。E は現実の利用が偏っていることを利用した最適化で、大半のクエリが少数のテーブルに集中するなら、その範囲を整理して先に渡すほうが探索の往復を減らせます。B が汎用の経路を、E が高頻度経路の近道を用意する形で、両者は組み合わせて機能します。

The principle for large schemas is to put only what is needed into context. Progressive discovery (B) has the agent work the way a human analyst does — scan a list, then fetch detail — keeping context consumption proportional to the tables actually used. The curated common model (E) exploits the skew in real usage: if most queries hit a handful of tables, shipping that subset organized up front removes discovery round trips. B provides the general path and E a shortcut for the hot path; they compose.

- **A 不正解**: 数十万トークンを常時コンテキストに入れるのは、コスト・レイテンシの面で成立せず、大半のリクエストで無関係な情報が大部分を占めます / Hundreds of thousands of tokens resident in context is untenable on cost and latency, and on most requests the bulk of it is irrelevant
- **C 不正解**: 400 個のツールはモデルの選択精度を大きく損ないます。データの取得手段はツール数を増やす方向ではなく、1 つのツールの引数で表現します / 400 tools badly degrades selection accuracy. A retrieval mechanism belongs in the arguments of one tool, not in the number of tools
- **D 不正解**: 試行錯誤はクエリの失敗を繰り返すだけでコストとレイテンシを浪費し、しかも本番データに対して意図しないクエリを走らせるリスクがあります / Trial and error burns cost and latency on repeated query failures and risks running unintended queries against production data

**参照 / Reference:** MCP Server Development — resources と tools の設計、段階的な探索
</details>

---

### 問題 23 / Question 23

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

法務部門が、契約書レビューの判断基準（80 ページ、社内の判例集を含む）をエージェントに反映させたいと考えています。この知識は外部システムへのアクセスを伴わず、法務部門が四半期ごとに更新します。利用するアプリケーションは、Claude Code と社内ポータルの 2 つです。

Legal wants an agent to apply its contract-review criteria — 80 pages including an internal casebook. The knowledge involves no external system access and is updated quarterly by Legal. Two applications will use it: Claude Code and the internal portal.

**設問 / Question:**

最も適切なアプローチはどれですか？

Which is the most appropriate approach?

- A) 80 ページを両アプリの system プロンプトに常時入れ、プロンプトキャッシュで対応する / Keep all 80 pages in both applications' system prompts, relying on prompt caching
- B) **判断基準を Skill としてパッケージ化し、両方のアプリケーションから利用できるように配布する。契約書レビューに関連する場面でのみ読み込まれ、法務部門は Skill を更新するだけで両アプリに反映される** / **Package the criteria as a Skill and distribute it to both applications. It loads only in contract-review situations, and Legal updates one artifact to reach both**
- C) MCP サーバーを立て、判断基準を resources として公開する / Stand up an MCP server and expose the criteria as resources
- D) 判断基準を要約して 2 ページにし、system プロンプトに入れる / Summarize the criteria to two pages and put that in the system prompt

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

この要件の性質は「手順と専門知識をモデルに与えること」であり、外部システムの操作ではありません。Skills はまさにこの用途のもので、関連する場面でだけ読み込まれるため、契約書レビュー以外のリクエストが 80 ページ分のコンテキストを負担しません。更新の観点でも、法務部門が Skill という 1 つの成果物を更新すれば両アプリに反映され、二重管理による食い違いが起こりません。複数アプリで使うことは MCP を示唆する条件のように見えますが、共有したいのが「接続」ではなく「知識」である以上、MCP を挟む必然性はありません。

The nature of this requirement is delivering procedure and expertise to the model, not operating an external system — precisely what Skills are for. Because a Skill loads only in relevant situations, requests unrelated to contract review do not pay for 80 pages. On the update path, Legal maintains one artifact that reaches both applications, so the two cannot drift apart. Use by multiple applications looks like a signal for MCP, but what is being shared is knowledge, not a connection, so an MCP layer is not warranted.

- **A 不正解**: キャッシュはコストを下げますが、契約書レビュー以外の全リクエストで 80 ページがコンテキストを占める構造は変わりません。両アプリでの二重管理も残ります / Caching lowers cost but leaves 80 pages resident on every non-review request, and the duplicate maintenance in two applications remains
- **C 不正解**: MCP は接続の再利用に効く仕組みです。外部システムへのアクセスがない静的な知識を MCP で配るのは、デプロイと運用の層を足すだけで、Skills が提供する場面依存の読み込みも得られません / MCP earns its place for reusing a connection. Distributing static knowledge with no external access through MCP adds deployment and operations while forgoing the situational loading Skills provide
- **D 不正解**: 80 ページを 2 ページに要約すれば判断基準が失われます。契約書レビューは細部の判断が本質なので、要約は品質を直接損ないます / Compressing 80 pages to two destroys the criteria. Contract review turns on fine distinctions, so summarizing degrades quality directly

**参照 / Reference:** Agentic Customization — Skills と MCP のトレードオフ
</details>

---

### 問題 24 / Question 24

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

既存のアプリケーションには、社内 CRM を操作するカスタムツールが 12 個あります。別チームが同じ CRM を使いたいと言ってきたため、これを MCP サーバーに移行する提案が出ました。現行のツールは 2 年間安定稼働しており、評価データセットもあります。

Your existing application has 12 custom tools that operate the internal CRM. Another team now wants the same CRM access, prompting a proposal to migrate them to an MCP server. The current tools have run stably for two years and have an evaluation dataset.

**設問 / Question:**

移行の進め方として最も適切なのはどれですか？

Which is the most appropriate way to run this migration?

- A) 移行せず、別チームにもカスタムツールを実装してもらう / Skip the migration and have the other team implement its own custom tools
- B) 一度に切り替える。両方を並行させると混乱するから / Cut over all at once, since running both invites confusion
- C) **MCP サーバーを新設し、既存の 12 ツールと同じインターフェースを公開する。既存アプリを MCP 経由に切り替える前に、評価データセットで MCP 経由と直接呼び出しの結果が一致することを確認する。確認後に既存アプリを切り替え、その後に別チームを接続する** / **Stand up the MCP server exposing the same interface as the 12 existing tools. Before switching the existing application over, confirm on the evaluation dataset that results through MCP match direct invocation. Switch the existing application after that confirmation, then connect the other team**
- D) 別チームには読み取り専用の MCP サーバーを新規に作り、既存アプリは現状維持とする / Build a separate read-only MCP server for the other team and leave the existing application unchanged

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

複数クライアントでの再利用は MCP が適する条件なので、移行の方向自体は正しい判断です。問題は進め方で、2 年間安定稼働している本番機能を壊さないことが優先されます。同じインターフェースを公開すればエージェント側から見た振る舞いは変わらないはずですが、「変わらないはず」を評価データセットで確認してから切り替えます。既存アプリを先に切り替えるのは、問題があれば検知できる体制（評価データセットと 2 年分の運用知見）が既存側にあるからで、新しいチームを未検証の経路に最初に載せるべきではありません。

Reuse across clients is the condition MCP fits, so the direction is right. The question is execution, and the priority is not breaking a production capability that has run for two years. Exposing the same interface should leave agent-visible behavior unchanged — but "should" is confirmed against the evaluation dataset before cutting over. Switch the existing application first because that is where the means to detect a problem live (the evaluation dataset and two years of operational knowledge); the new team should not be the first traffic on an unverified path.

- **A 不正解**: 2 重実装は、CRM の仕様変更やセキュリティ修正のたびに 2 か所の対応を必要とし、必ず食い違います / A second implementation requires two fixes for every CRM change or security patch, and the two will diverge
- **B 不正解**: 検証なしの一斉切り替えは、2 年間安定していた本番機能を賭けの対象にします。評価データセットがあるのに使わない理由がありません / An unverified big-bang cutover gambles a capability that has been stable for two years, and there is no reason to leave an available evaluation dataset unused
- **D 不正解**: 読み取り専用に限定しても実装は 2 系統に分かれ、しかも別チームが更新操作を必要とした時点で行き詰まります / Restricting to read-only still leaves two implementations, and it dead-ends the moment the other team needs a write operation

**参照 / Reference:** Agentic Customization — カスタムツールから MCP への移行、検証を伴う段階的切り替え
</details>

---

### 問題 25 / Question 25

> サブスキル / Sub-skill: Agentic Customization (4.1%)

**シナリオ / Scenario:**

組織全体で Claude アプリケーションが増え、各チームが独自にカスタムツール・Skills・MCP サーバーを作っています。同じ外部システムに対するツールが 3 チームで別々に実装されており、あるチームが作った MCP サーバーが本番の書き込み権限を持っていることを誰も把握していませんでした。

Claude applications have proliferated across the organization, with each team building its own custom tools, Skills, and MCP servers. Three teams have separately implemented tools against the same external system, and nobody was aware that one team's MCP server holds production write access.

**設問 / Question:**

組織としての最も適切な対応はどれですか？

Which is the most appropriate organizational response?

- A) 新規のツール・MCP サーバーの作成を全面的に禁止し、中央チームがすべて作る / Ban all new tools and MCP servers and have a central team build everything
- B) 各チームの自主性に任せ、問題が起きたチームが個別に対応する / Leave it to each team's discretion and let whoever has a problem deal with it
- C) すべてのツールを 1 つの巨大な MCP サーバーに統合する / Consolidate every tool into one giant MCP server
- D) **ツール・Skills・MCP サーバーの登録簿を設け、何が存在し、どの外部システムにどの権限で接続しているかを可視化する。書き込み権限や不可逆な操作を持つものはレビュー必須にし、重複しているものは所有チームを決めて統合する。新規作成は禁止せず、登録と権限申告を要件にする** / **Establish a registry of tools, Skills, and MCP servers that makes visible what exists and which external systems each reaches with which privileges. Require review for anything holding write access or irreversible operations, and consolidate duplicates under a designated owning team. Do not ban new creation — require registration and privilege declaration**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

問題は 2 つあります。可視性の欠如（誰も権限を把握していない）と重複です。登録簿はまず可視性を回復させ、そこから初めてリスクの高いもの（書き込み権限・不可逆な操作）を選別してレビュー対象にできます。重複の統合は、把握できて初めて可能になります。重要なのは新規作成を禁止しないことです。禁止すれば開発は中央チームで詰まり、チームは登録簿を迂回する動機を持ちます。登録と権限申告を要件にすれば、開発の速度を保ったまま可視性が維持されます。統制は、迂回されない程度に軽いことが条件です。

Two problems: no visibility (nobody knows what privileges exist) and duplication. A registry restores visibility first, which is what makes it possible to single out the high-risk items — write access, irreversible operations — for mandatory review. Consolidating duplicates only becomes possible once you can see them. The important part is not banning new creation: a ban bottlenecks development on the central team and gives teams a motive to route around the registry. Requiring registration and privilege declaration preserves velocity while keeping visibility. A control only works if it is light enough not to be bypassed.

- **A 不正解**: 中央集権はボトルネックを作り、チームは登録されない「影のツール」を作るようになります。可視性はかえって悪化します / Centralization creates a bottleneck and pushes teams toward unregistered shadow tools, making visibility worse
- **B 不正解**: 現状そのもので、本番の書き込み権限が把握されていない状態が続きます。問題が起きてからの対応では、不可逆な操作には間に合いません / This is the status quo, leaving production write access unaccounted for. Reacting after the fact is too late for irreversible operations
- **C 不正解**: 1 つに統合するとツール数が過大になり選択精度が落ちるうえ、全チームの変更が 1 つのサーバーに集中して結合度が上がります / One giant server makes the tool count unmanageable for selection and couples every team's changes into a single artifact

**参照 / Reference:** Agentic Customization — 組織横断のツール・MCP のガバナンス
</details>

---
