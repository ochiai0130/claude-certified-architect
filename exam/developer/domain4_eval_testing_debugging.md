# Domain 4: 評価・テスト・デバッグ / Eval, Testing, and Debugging

> 配点比率 / Weight: **2.6%**
> 問題数 / Questions: **6**（基礎 4 / 発展 2）
> 形式 / Format: 単一選択（選択肢 A–D）＋複数選択（選択肢 A–E、選ぶ数を明示）

## サブスキル / Sub-skills

| サブスキル / Sub-skill | Weight | 本ファイル |
|---|---|---|
| Debugging and Error Handling | 2.6% | 6 |

## 出題範囲 / Scope（公式ガイドの記述に基づく）

- **Debugging and Error Handling** — Claude アプリケーションのデバッグとエラー処理の技法。エラー種別の特定、回復戦略の選択、失敗モードを特定するためのトレース分析、統合レイヤーとモデル出力のどちらに問題の原因があるかの切り分け / Debugging and error handling techniques for Claude applications, including error type identification, recovery strategy selection, trace analysis to identify failure modes, and problem origin isolation between the integration layer and model output

> **配点は 2.6%（本番 53 問中およそ 1〜2 問）** と最小のドメインですが、サブスキルが 1 つしかないため出題範囲は狭く、確実に取れます。なお「Eval」の語がドメイン名にありますが、公式のサブスキルは Debugging and Error Handling の 1 つだけです。
>
> At **2.6%** (roughly 1–2 of the 53 live items) this is the smallest domain, but with a single sub-skill its scope is narrow and reliably scoreable. Note that although "Eval" appears in the domain name, the only official sub-skill is Debugging and Error Handling.

---

## 基礎 / Foundations level

### 問題 1 / Question 1

> サブスキル / Sub-skill: Debugging and Error Handling (2.6%)

**シナリオ / Scenario:**

本番のアプリケーションで、Claude API の呼び出しが失敗するようになりました。ログには HTTP ステータス 429 と、レート制限に関するエラー種別が記録されています。

Your production application has started failing on Claude API calls. The logs show HTTP status 429 with a rate-limit error type.

**設問 / Question:**

このエラーの性質と、取るべき対応として正しいのはどれですか？

Which correctly describes this error and the appropriate response?

- A) **一時的なエラーで、時間をおけば成功しうる。指数バックオフとジッターを入れて再試行し、応答に再試行までの待ち時間が示されていればそれに従う。恒常的に発生するならリクエスト量そのものを見直す** / **A transient error that can succeed later. Retry with exponential backoff and jitter, honoring any indicated wait time in the response. If it persists, revisit the request volume itself**
- B) リクエストの内容に誤りがあるので、リクエストを修正する / The request content is invalid, so fix the request
- C) 認証に失敗しているので、API キーを再発行する / Authentication failed, so reissue the API key
- D) モデル側の不具合なので、別のモデルに切り替える / A model-side defect, so switch models

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

エラー種別の特定は、まずステータスコードで大きく分けます。4xx のうち 400（リクエストが不正）・401（認証）・403（権限）は、同じリクエストを繰り返しても必ず同じ結果になる決定論的なエラーなので、再試行は無意味です。一方 429（レート制限）と 5xx 系（サーバー側の一時的な問題や過負荷）は、時間をおけば成功しうる一時的なエラーなので再試行が有効です。再試行には指数バックオフを使い、ジッター（ランダムなずらし）を加えます。ジッターがないと、失敗した多数のクライアントが同じタイミングで一斉に再試行して輻輳を悪化させます。恒常的に 429 が出る場合は、再試行では解決せず、リクエスト量やバッチ処理への切り替えを検討します。

Start error identification with the status code. Among 4xx, 400 (invalid request), 401 (authentication), and 403 (permission) are deterministic: the same request always produces the same result, so retrying is pointless. By contrast 429 (rate limit) and the 5xx family (transient server issues and overload) can succeed later, so retrying helps. Use exponential backoff with jitter — without jitter, many failed clients retry in lockstep and worsen congestion. If 429s are constant, retrying will not fix it; revisit request volume or move work to batch.

- **B 不正解**: リクエストの誤りは 400 で返ります。429 はリクエストの内容ではなく頻度の問題です / An invalid request returns 400. A 429 concerns frequency, not content
- **C 不正解**: 認証の失敗は 401 で返ります。キーの再発行は無関係で、かえって稼働中の他の経路を壊しかねません / Authentication failure returns 401. Reissuing the key is unrelated and risks breaking other running paths
- **D 不正解**: レート制限はアカウントやモデルに対する利用量の制限であり、モデルの不具合ではありません / A rate limit is a usage limit on the account or model, not a model defect

**参照 / Reference:** Debugging and Error Handling — エラー種別の特定、回復戦略の選択
</details>

---

### 問題 2 / Question 2

> サブスキル / Sub-skill: Debugging and Error Handling (2.6%)

**シナリオ / Scenario:**

構造化データを返させる機能で、「JSON のパースに失敗した」というエラーが 1 日に数件発生しています。開発者は「モデルが壊れた JSON を返している」と考え、プロンプトに「必ず有効な JSON を返すこと」という指示を追加しました。それでも発生し続けています。失敗したレスポンスを調べると、いずれも JSON が途中で切れていました。

A feature that returns structured data fails with "JSON parse error" a few times a day. A developer assumed "the model returns malformed JSON" and added "always return valid JSON" to the prompt. It keeps happening. Inspecting the failed responses shows the JSON is cut off mid-way in every case.

**設問 / Question:**

最も可能性が高い原因はどれですか？

What is the most likely cause?

- A) モデルが JSON の文法を理解していない / The model does not understand JSON syntax
- B) ネットワークがレスポンスを切断している / The network is truncating the response
- C) パーサーの実装に不具合がある / The parser implementation is defective
- D) **出力が `max_tokens` に達して打ち切られている。`stop_reason` が `max_tokens` になっているはずで、これはモデル出力の品質の問題ではなく、上限設定の問題である** / **The output hit `max_tokens` and was truncated. `stop_reason` should read `max_tokens` — a limit-configuration problem, not a model-output-quality problem**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

「JSON が途中で切れている」という症状は、`max_tokens` による打ち切りの典型的な現れ方です。確認する場所は `stop_reason` で、`max_tokens` になっていればこれが原因と確定します。ここで重要なのは切り分けの考え方です。開発者は原因をモデル出力の品質だと決めつけてプロンプトを直しましたが、実際には統合レイヤー（上限の設定）の問題でした。この誤診が起きたのは、`stop_reason` を見ずに症状だけで判断したためです。対処は `max_tokens` を出力に必要な長さまで引き上げること、そして打ち切りを検知したときにパースを試みずエラーとして扱うことです。

JSON cut off mid-way is the classic signature of `max_tokens` truncation. The place to look is `stop_reason`: if it reads `max_tokens`, the cause is settled. The important lesson is the isolation reasoning. The developer assumed model-output quality and edited the prompt, when the problem was in the integration layer — a limit setting. The misdiagnosis happened because the judgment was made from the symptom without checking `stop_reason`. The fix is to raise `max_tokens` to what the output requires and to treat a detected truncation as an error rather than attempting to parse it.

- **A 不正解**: 文法を理解していないなら、途中で切れるのではなく構文的に誤った JSON が最後まで返るはずです / A syntax misunderstanding would produce a complete but syntactically wrong document, not a truncation
- **B 不正解**: ネットワークによる切断なら接続エラーとして現れ、正常なレスポンスとして受け取れません / A network truncation surfaces as a connection error rather than a successfully received response
- **C 不正解**: パーサーの不具合なら、切れていない正常な JSON でも失敗するはずです。失敗が切れたものに限られている点が合いません / A defective parser would also fail on complete JSON. Failures confined to truncated payloads do not fit

**参照 / Reference:** Debugging and Error Handling — 統合レイヤーとモデル出力の切り分け、`stop_reason` の確認
</details>

---

### 問題 3 / Question 3

> サブスキル / Sub-skill: Debugging and Error Handling (2.6%)

**シナリオ / Scenario:**

アプリケーションに一律の再試行処理を入れました。「例外が発生したら 5 回まで再試行する」という実装です。導入後、一部のエラーでレスポンスが極端に遅くなり、コストも増えました。

You added blanket retry logic: "retry up to five times on any exception." Since then some errors produce extremely slow responses and cost has gone up.

**設問 / Question:**

最も適切な修正はどれですか？

Which is the most appropriate correction?

- A) 再試行の回数を 2 回に減らす / Reduce the retry count to two
- B) **エラー種別に応じて再試行の可否を分ける。一時的なエラー（レート制限、サーバー側の過負荷、接続の失敗）だけを指数バックオフで再試行し、決定論的なエラー（不正なリクエスト、認証・権限の失敗）は再試行せず即座に失敗させる** / **Decide retryability by error type: retry only transient errors — rate limits, server overload, connection failures — with exponential backoff, and fail deterministic ones (invalid request, authentication and permission failures) immediately without retrying**
- C) 再試行の間隔を固定の 10 秒にする / Use a fixed 10-second retry interval
- D) 再試行をやめ、すべてのエラーを即座に失敗させる / Stop retrying and fail immediately on every error

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

一律の再試行が悪いのは、決定論的なエラーまで再試行するからです。不正なリクエストや認証の失敗は、何度繰り返しても同じ結果になります。5 回分の待ち時間とトークンを消費したうえで、結局同じエラーを返すことになり、これが遅延とコスト増の正体です。回復戦略はエラー種別ごとに選びます。一時的なエラーには指数バックオフ付きの再試行、決定論的なエラーには即座の失敗と、呼び出し側に対する明確なエラーの伝達です。加えて、再試行が有効な場合でも、副作用を持つ操作には冪等性の担保が別途必要になります。

Blanket retry is wrong because it retries deterministic errors too. An invalid request or an authentication failure returns the same result however many times you repeat it — so you spend five waits and five requests' worth of tokens only to return the same error, which is exactly where the latency and cost went. Choose the recovery strategy per error type: exponential-backoff retry for transient errors, immediate failure with a clear error to the caller for deterministic ones. And even where retry is appropriate, operations with side effects need idempotency established separately.

- **A 不正解**: 回数を減らしても、決定論的なエラーを再試行するという誤りは残ります。無駄が 5 回から 2 回に減るだけです / Fewer attempts leaves the same mistake of retrying deterministic errors. The waste goes from five to two
- **C 不正解**: 固定間隔は、輻輳時に多数のクライアントが同時に再試行して負荷を増幅します。バックオフとジッターが必要です / A fixed interval makes many clients retry simultaneously under congestion and amplifies load. Backoff with jitter is required
- **D 不正解**: 一時的なエラーまで即座に失敗させると、本来成功したはずのリクエストを落とし、可用性が下がります / Failing immediately on transient errors drops requests that would have succeeded and reduces availability

**参照 / Reference:** Debugging and Error Handling — 回復戦略の選択、エラー種別ごとの再試行
</details>

---

### 問題 4 / Question 4

> サブスキル / Sub-skill: Debugging and Error Handling (2.6%)

**シナリオ / Scenario:**

10 ステップ前後で完了するエージェントが、ときどき「答えが出ないまま停止する」という失敗をします。最終出力だけを見ても原因が分かりません。再現もしにくく、同じ入力でも成功することがあります。

An agent that normally completes in about ten steps sometimes fails by "stopping without producing an answer." The final output alone gives no clue, it is hard to reproduce, and the same input sometimes succeeds.

**設問 / Question:**

失敗モードを特定するために有効な取り組みを **2 つ選択してください**。

Select **2** effective ways to identify the failure mode.

- A) **各ステップの `stop_reason`、選択されたツール、渡した引数、返した結果、トークン使用量を、実行 ID で紐づけて記録し、失敗した実行を最初から追えるようにする** / **Record each step's `stop_reason`, the tool selected, the arguments passed, the result returned, and token usage, correlated by a run ID so a failed run can be traced from the start**
- B) 最終出力だけをより詳細にログに残す / Log the final output in more detail
- C) 失敗したら自動で最初からやり直す / Automatically restart from the beginning on failure
- D) **成功した実行と失敗した実行のトレースを突き合わせ、どのステップから分岐したかを特定する。ステップ数・ツール選択・コンテキスト長の分布も比較する** / **Diff traces of successful and failed runs to find the step where they diverge, and compare distributions of step count, tool selection, and context length**
- E) ログの保存期間を延ばす / Extend the log retention period

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A, D**

**解説 / Explanation:**

多段のエージェントでは、最終出力は失敗の結果であって原因ではありません。原因はどこかのステップにあり、それを見るにはステップ単位のトレースが要ります（A）。`stop_reason` は特に重要で、`max_tokens` で打ち切られたのか、`tool_use` で止まったまま結果が返されなかったのかで、原因はまったく異なります。そして、間欠的な失敗の分析には比較が有効です（D）。同じ入力で成功する実行があるなら、成功と失敗のトレースを突き合わせることで分岐点が特定でき、そこが調査の焦点になります。分布の比較は、たとえば「失敗する実行はステップ数が多くコンテキストが長い」といった傾向を明らかにします。

In a multi-step agent the final output is the consequence of the failure, not its cause. The cause lies at some step, and seeing it requires per-step traces (A). `stop_reason` matters especially: truncation at `max_tokens` and halting at `tool_use` with no result returned are entirely different causes. For intermittent failures, comparison is the productive technique (D). When the same input sometimes succeeds, diffing a successful trace against a failed one locates the divergence and focuses the investigation. Comparing distributions surfaces patterns such as "failing runs take more steps and carry longer context."

- **B 不正解**: 最終出力の詳細化では、途中のどのステップで何が起きたかは分かりません。停止している場合はそもそも出力がありません / More detail in the final output does not reveal what happened at intermediate steps — and when the run halts there is no output at all
- **C 不正解**: 自動再実行は症状を隠すだけで、原因の特定を遠ざけます。間欠的な失敗が見えなくなり、悪化しても気づけません / Automatic restart hides the symptom and moves you away from the cause. The intermittent failure becomes invisible and can worsen unnoticed
- **E 不正解**: 保存期間を延ばしても、記録されていない情報は増えません。問題は期間ではなく粒度です / A longer retention period does not add information that was never recorded. The problem is granularity, not duration

**参照 / Reference:** Debugging and Error Handling — トレース分析、失敗モードの特定
</details>

---

## 発展 / Advanced

### 問題 5 / Question 5

> サブスキル / Sub-skill: Debugging and Error Handling (2.6%)

**シナリオ / Scenario:**

分類アプリの精度が、先週から本番でだけ低下しています。評価データセットに対する精度は変わっていません。この 1 週間で行われた変更は、(1) プロンプトの軽微な文言修正、(2) モデルの新バージョンへの切り替え、(3) 入力前処理ライブラリのアップデート、の 3 つです。

A classifier's accuracy has dropped in production since last week, while accuracy on the evaluation dataset is unchanged. Three changes shipped in that week: (1) a minor prompt wording edit, (2) a switch to a newer model version, (3) an update to the input-preprocessing library.

**設問 / Question:**

最も適切な進め方はどれですか？

Which is the most appropriate approach?

- A) 3 つの変更をすべて元に戻し、精度が回復したら 1 つずつ戻していく / Revert all three changes and, if accuracy recovers, re-apply them one at a time
- B) 最も影響が大きそうなモデルの切り替えを元に戻す / Revert the model switch as the likeliest culprit
- C) **評価データセットで再現しないという事実に注目し、本番と評価の入力の違いを先に調べる。本番で失敗している実際の入力を収集して評価データセットに追加し、そのうえで 3 つの変更を 1 つずつ切り分ける** / **Focus on the fact that it does not reproduce on the evaluation dataset and first investigate how production input differs from it. Collect the real failing production inputs, add them to the evaluation dataset, and only then isolate the three changes one at a time**
- D) プロンプトの文言を元に戻す。最も安全な変更だから / Revert the prompt wording, as the safest change to undo

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

この問題の最も重要な手がかりは、「評価データセットでは再現しない」という事実です。これは、評価データセットが本番の入力分布を代表していないことを意味します。まずここに向き合わないと、変更を 1 つずつ戻しても、戻した効果を測る手段がありません。本番で失敗している実際の入力を収集して評価データセットに加えれば、再現可能な状態が作れ、そこで初めて 3 つの変更の切り分けが意味を持ちます。この順序は重要で、再現手段のないまま本番で試行錯誤するのは、利用者に影響を与えながら推測を繰り返すことになります。なお、前処理ライブラリのアップデートは見落とされがちですが、入力の正規化が変われば同じ文書でもモデルに届く内容が変わるため、モデル出力の問題に見える現象を統合レイヤーが起こしている典型例です。

The most important clue is that it does not reproduce on the evaluation dataset — which means that dataset does not represent production input. Until you face that, reverting changes one at a time gives you no way to measure the effect of each revert. Collecting the real failing production inputs into the evaluation dataset creates a reproducible case, and only then does isolating the three changes mean anything. The order matters: experimenting in production without a means of reproduction is repeated guessing while users are affected. Note too that the preprocessing library update is easy to overlook — a change in input normalization alters what actually reaches the model for the same document, a classic case of the integration layer producing what looks like a model-output problem.

- **A 不正解**: 全戻しは一時的に精度を回復させるかもしれませんが、再現手段がないままなので、1 つずつ戻したときに何が効いたかを測れません / Reverting everything may restore accuracy temporarily, but with no means of reproduction you cannot measure which re-application breaks it again
- **B 不正解**: 「影響が大きそう」は推測です。前処理ライブラリの更新が入力を変えている可能性は同程度にあり、根拠なく 1 つを選ぶべきではありません / "Likeliest" is a guess. The preprocessing update changing the input is just as plausible; picking one without evidence is not warranted
- **D 不正解**: 戻しやすさは原因の可能性とは無関係です。安全に戻せることを理由に選ぶのは、切り分けではありません / Ease of reverting is unrelated to likelihood of causation. Choosing by reversibility is not isolation

**参照 / Reference:** Debugging and Error Handling — 問題の原因の切り分け、本番と評価環境の差異
</details>

---

### 問題 6 / Question 6

> サブスキル / Sub-skill: Debugging and Error Handling (2.6%)

**シナリオ / Scenario:**

本番のエージェントで、ごくまれに「ツールを呼んだまま応答が返らず、タイムアウトする」という事象が起きます。発生率は 0.05% 程度で、再現手順が分かりません。監視は API のエラー率とレイテンシの平均値だけを見ており、この事象はエラーとして計上されていません。運用チームからは「発生率が低いので様子見でよい」という意見が出ています。

Very occasionally a production agent hangs after calling a tool and times out. The rate is about 0.05%, and there is no known reproduction. Monitoring covers only API error rate and mean latency, and these events are not counted as errors. Operations suggests "the rate is low; let's watch it."

**設問 / Question:**

最も適切な対応はどれですか？

Which is the most appropriate response?

- A) 発生率が低いので、監視のみを継続する / Continue monitoring only, given the low rate
- B) タイムアウト時間を延ばし、完了を待つようにする / Increase the timeout and wait for completion
- C) エージェントのループに再試行を追加し、タイムアウトしたら最初からやり直す / Add a retry to the agent loop and restart from the beginning on timeout
- D) **タイムアウトを失敗として計上し、平均値ではなく分布（パーセンタイル）で監視する。あわせて、タイムアウトした実行のトレースを保全してツール呼び出しの往復を追えるようにし、原因の候補（ツール側の応答なし、`tool_result` の返却漏れ、外部依存の停止）を切り分ける** / **Count timeouts as failures and monitor the distribution (percentiles) rather than the mean. Preserve the traces of timed-out runs so the tool-call round trip can be followed, and isolate the candidate causes: the tool never responding, a missing `tool_result`, or a stalled external dependency**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

この事象が見えていない理由は 2 つあります。1 つはタイムアウトがエラーとして計上されていないこと、もう 1 つは平均値での監視です。0.05% の事象は平均レイテンシをほとんど動かさないので、パーセンタイル（p99 など）で見ないと検知できません。まず計測を直すのが先で、見えない問題は調査もできません。そのうえで、タイムアウトした実行のトレースを保全します。エージェントがツールを呼んだまま止まる事象では、原因がツール側にあるのか（応答が返っていない）、統合レイヤーにあるのか（`tool_result` を返す処理が例外で抜けている）、外部依存にあるのかで対処がまったく異なります。低い発生率は無視してよい理由になりません。0.05% でも件数によっては日々一定数の利用者が影響を受けており、しかも原因が分からないまま放置すると、負荷や依存先の変化で急に増えることがあります。

Two things hide this event: timeouts are not counted as failures, and monitoring is on the mean. A 0.05% event barely moves mean latency, so it is only visible in the distribution — p99 and similar. Fix the measurement first; a problem you cannot see is a problem you cannot investigate. Then preserve the traces of timed-out runs. When an agent hangs after calling a tool, the remedy differs entirely depending on whether the cause is the tool (no response), the integration layer (the code path returning `tool_result` exited on an exception), or an external dependency. A low rate is not a reason to ignore it: at volume, 0.05% still affects a steady stream of users daily, and an unexplained failure can grow abruptly when load or a dependency changes.

- **A 不正解**: 現在の監視ではこの事象がそもそも計上されていないため、「様子見」は実質的に見ないことと同じです。増加しても気づけません / The current monitoring does not count these events, so "watch it" is indistinguishable from not looking. An increase would go unnoticed
- **B 不正解**: タイムアウトを延ばすと、失敗を検知するまでの時間が延びるだけで、応答が返らない原因は変わりません。利用者の待ち時間が悪化します / A longer timeout only delays detection without changing why no response comes, and it worsens the user's wait
- **C 不正解**: 原因不明のまま最初からやり直すのは、副作用のあるツールを再実行するリスクがあり、原因の特定も遠ざけます / Restarting without knowing the cause risks re-executing side-effecting tools and moves you further from the diagnosis

**参照 / Reference:** Debugging and Error Handling — トレース分析、統合レイヤーと外部依存の切り分け、計測の設計
</details>

---
