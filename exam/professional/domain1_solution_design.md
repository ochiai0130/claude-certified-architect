# Domain 1: ソリューション設計とアーキテクチャ / Solution Design and Architecture

> 配点比率 / Weight: **17%**
> 問題数 / Questions: **30**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- ビジネス要件からアーキテクチャへの翻訳・ユースケース適合性の判断 / Translating business requirements into architecture; judging use-case fit
- ワークフロー / 単一エージェント / オーケストレーター・ワーカー の選択 / Choosing between fixed workflows, single agents, and orchestrator-worker topologies
- マルチテナント分離・データレジデンシー・マルチリージョン設計 / Multi-tenancy isolation, data residency, multi-region design
- レイテンシ・コスト・品質・SLA のトレードオフと容量計画 / Latency, cost, quality, SLA tradeoffs and capacity planning
- 障害ドメイン設計・縮退運転・災害復旧 / Failure-domain design, graceful degradation, disaster recovery
- 段階的ロールアウト（シャドー・カナリア）と既存システムへの漸進的統合 / Phased rollout and incremental integration with legacy systems

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

損害保険会社で自動車保険の支払査定を Claude で支援します。査定フローは法令とレギュレーターとの合意により **必ず** 「事故受付 → 契約有効性確認 → 免責事由チェック → 損害額算定 → 支払可否判定」の順で実施し、各ステップの入出力を保存する義務があります。ステップの順序を入れ替えたり省略したりすることは監督官庁への説明が不可能です。一方、損害額算定の内部では、修理見積書・写真・過去事例など複数の情報源を状況に応じて参照する必要があります。

A P&C insurer is using Claude to assist auto-insurance claim adjudication. By regulation and by agreement with the supervisor, the flow **must** execute in the fixed order: intake → policy validity → exclusion check → loss quantification → payout decision, with each step's inputs and outputs retained. Reordering or skipping steps cannot be defended to the regulator. Within loss quantification, however, the system must consult repair estimates, photos, and precedent cases adaptively.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) 全体を 1 つの自律エージェントにし、システムプロンプトで 5 ステップの順序を厳守するよう指示する / Implement the whole flow as one autonomous agent, instructing it in the system prompt to follow the five steps in order
- B) 全体をオーケストレーター・ワーカー構成にし、コーディネーターが状況に応じて 5 つのサブエージェントを呼び分ける / Use an orchestrator-worker topology where a coordinator dispatches the five subagents adaptively
- C) 5 ステップは**コードで記述した固定ワークフロー**として実装し、各ステップの境界で入出力を永続化する。損害額算定ステップの**内部のみ**をツールを持つエージェントにする / Implement the five steps as a **code-defined fixed workflow**, persisting inputs and outputs at each step boundary, and make **only** the loss-quantification step an agent with tools
- D) 5 ステップそれぞれを独立したマイクロサービスにし、各サービスが自分の次のステップを判断して呼び出す / Implement each step as an independent microservice that decides and invokes its own successor

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

規制で順序が固定されている制御フローは、**モデルの判断に委ねてはならない領域**です。順序保証をコード側の固定ワークフローに置けば、順序違反は構造的に起こり得ず、各境界での永続化が監査証跡になります。一方、損害額算定は本質的に探索的なので、そこだけをエージェントにするのが適合します。「決定論が必要な部分はコード、判断が必要な部分はモデル」という境界設定が本ドメインの中核です。

Control flow that regulation fixes must not be delegated to model judgment. Encoding the order in code makes violations structurally impossible and yields an audit trail at each boundary, while the genuinely exploratory step is where agentic behavior earns its cost.

- **A 不正解**: プロンプトによる順序保証は確率的で、監督官庁に「順序は必ず守られる」と説明できません。 / Prompt-level ordering is probabilistic and cannot be defended as a guarantee.
- **B 不正解**: 動的ディスパッチは順序固定要件と真っ向から矛盾します。柔軟性が要件ではなく違反要因になっています。 / Adaptive dispatch directly contradicts the fixed-order requirement.
- **D 不正解**: 各サービスが後続を判断する構成は分散した暗黙の制御フローになり、順序の証明が最も困難になります。 / Distributing control flow across services makes the ordering hardest to prove.

**参照 / Reference:** ワークフロー vs エージェントの境界設定、規制対応の決定論的統制
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

社内から「経費精算の承認判定を Claude エージェント化したい」という要望が来ました。調査したところ、判定ロジックは「金額 < 5 万円かつ勘定科目が交際費以外なら自動承認、それ以外は上長承認」という完全に決定的なルールで、例外は存在せず、過去 3 年で仕様変更は 1 回のみでした。現状は手作業で月 4,000 件処理しており、担当者はルールを機械的に適用しています。

An internal request asks to "agentify expense-report approval with Claude." On investigation, the logic is fully deterministic — auto-approve if amount < ¥50,000 and the account code is not entertainment; otherwise route to a manager — with no exceptions and one spec change in three years. 4,000 items/month are currently processed by hand, mechanically applying the rule.

**設問 / Question:**

アーキテクトとして最も適切な提案はどれですか？

As the architect, what is the most appropriate recommendation?

- A) Claude を使わず**決定的なルールエンジン**で自動化することを提案する。LLM は非決定性・コスト・監査説明の負担を増やすだけで、この要件には利得がない。ただし添付レシートの読み取り（非構造化 → 構造化）が課題なら、その部分に限って Claude を提案する / Recommend a **deterministic rules engine** with no LLM: an LLM only adds nondeterminism, cost, and audit burden here. If OCR of attached receipts (unstructured → structured) is the real pain, propose Claude only for that slice
- B) Claude Haiku で判定エージェントを構築し、コストを抑えつつ自動化する / Build the approval agent on Claude Haiku to automate cheaply
- C) Claude Opus で判定エージェントを構築し、将来の例外ルール追加に備える / Build it on Claude Opus so future exception rules are covered
- D) Claude に判定させたうえで、全件を人間がレビューする構成にする / Have Claude decide and route 100% of cases to human review

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

Professional で頻出の「**LLM を使わないことを提案できるか**」を問う問題です。要件が完全に決定的で例外がない領域に LLM を持ち込むと、正確性は下がり（100% → 99.x%）、コストと監査説明の負担は増えます。一方で、真のボトルネックが非構造化データの構造化にあるなら、そこは LLM の適合領域です。**要望をそのまま実装せず、要件を分解して適合部分だけに適用する**のがアーキテクトの仕事です。

A recurring Professional theme: recognizing when *not* to use an LLM. For a fully deterministic rule, an LLM lowers accuracy from 100% and raises cost and audit burden — but the unstructured-to-structured slice is genuine LLM territory. Decompose the request rather than implementing it literally.

- **B 不正解**: モデルを安くしても、決定的ルールに確率的判定を持ち込む設計上の誤りは解消しません。 / A cheaper model does not fix applying probabilistic judgment to a deterministic rule.
- **C 不正解**: 存在しない将来要件のために非決定性を導入するのは投機的過剰設計です。例外が生じた時点でルールを追加すれば済みます。 / Speculative over-engineering for exceptions that do not exist.
- **D 不正解**: 全件人間レビューでは自動化の目的（工数削減）が達成されず、レビュー工数が上乗せされるだけです。 / 100% human review defeats the automation goal and adds work.

**参照 / Reference:** ユースケース適合性の判断、LLM を使わない判断
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

競合関係にある複数の銀行に同一の与信分析 SaaS を提供しています。各行のデータは相互に絶対に漏れてはならず、契約上、単一のインシデントで複数テナントに影響が及ぶ構成は禁止されています。現在は 1 つの Claude API キーと 1 つのベクトルストアを全テナント共有し、リクエストごとに `tenant_id` でフィルタしています。

You provide the same credit-analysis SaaS to multiple competing banks. Cross-tenant leakage is contractually unacceptable, and a configuration in which a single incident can affect multiple tenants is prohibited. Today a single Claude API key and a single vector store are shared across tenants, filtered per request by `tenant_id`.

**設問 / Question:**

契約要件を満たすうえで最も適切な改善はどれですか？

Which change best satisfies the contractual requirement?

- A) `tenant_id` フィルタをシステムプロンプトにも明記し、モデルにテナント境界を守らせる / Also state the `tenant_id` boundary in the system prompt so the model respects it
- B) ベクトルストアのクエリに `tenant_id` フィルタが付いているかを単体テストで検証する / Add unit tests asserting that every vector-store query carries a `tenant_id` filter
- C) 全リクエストのプロンプトとレスポンスをログに保存し、事後に漏洩を検知できるようにする / Log all prompts and responses so leakage can be detected after the fact
- D) テナントごとに**データストアと認証情報を物理的に分離**し（テナント別のインデックス／スキーマと別々の資格情報）、アプリケーション層のフィルタはその上の二次防御として残す。分離の単位を障害ドメインの単位と一致させる / **Physically isolate the data store and credentials per tenant** (per-tenant index/schema, separate credentials), keeping the application-layer filter as a secondary defense, and align the isolation unit with the failure domain

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

「単一インシデントで複数テナントに影響が及ばないこと」という要件は、**論理分離では満たせません**。アプリケーション層のフィルタは 1 行のバグで全テナントに波及します。要件が求めているのは障害ドメインの分離であり、ストアと資格情報の物理分離がその実装です。フィルタは多層防御の 2 層目として残す価値がありますが、1 層目にはできません。

"No single incident may affect multiple tenants" cannot be met by logical isolation: one bug in a shared filter reaches every tenant. The requirement is about failure domains, and physical separation of store and credentials is its implementation. The filter remains valuable as a second layer, never the first.

- **A 不正解**: モデルへの指示はテナント分離の統制になりません。取得層で既に混ざったデータは指示では戻せません。 / Model instructions are not an isolation control; data already mixed at retrieval cannot be un-mixed by a prompt.
- **B 不正解**: テストは実装ミスを減らしますが、共有ストアである限り単一障害が全テナントに及ぶ構成自体は変わりません。 / Tests reduce defects but leave the shared-blast-radius topology intact.
- **C 不正解**: 事後検知は漏洩を防ぎません。競合他社にデータが渡った後の検知には契約上の価値がありません。 / Detection after disclosure does not prevent it.

**参照 / Reference:** マルチテナント分離、障害ドメイン設計、多層防御
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

EU と米国の両方で医療文書要約サービスを提供します。GDPR により EU 居住者の個人データは EU 域内で処理する必要があり、米国側は HIPAA 対応が必要です。現在は米国リージョンの単一エンドポイントに全トラフィックを集約しており、EU の顧客から DPA（データ処理契約）上の指摘を受けています。開発チームは 1 つのコードベースを維持したいと考えています。

You run a clinical-document summarization service in both the EU and the US. GDPR requires EU residents' personal data to be processed within the EU; the US side needs HIPAA compliance. All traffic currently funnels to a single US-region endpoint, and EU customers have raised findings against the DPA. The team wants to keep one codebase.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) 米国リージョンに送る前に EU 居住者データを仮名化し、単一エンドポイント構成を維持する / Pseudonymize EU residents' data before sending it to the US region, keeping the single endpoint
- B) **同一のコードベースをリージョンごとにデプロイ**し、EU トラフィックは EU リージョンの Claude エンドポイント（例: Bedrock / Vertex の EU リージョンまたは対応する first-party エンドポイント）とEU 内のデータストアに閉じる。ルーティングは認証時に確定した居住地属性で決定し、リージョン間のデータ複製は行わない / **Deploy the same codebase per region**, keeping EU traffic on an EU-region Claude endpoint (e.g. an EU region of Bedrock/Vertex or the corresponding first-party endpoint) and EU-resident data stores, routing on a residency attribute resolved at authentication, with no cross-region replication
- C) EU 顧客には別会社を設立して法的に分離する / Establish a separate legal entity for EU customers
- D) EU からのリクエストのみ同期処理をやめ、バッチで夜間に米国側で処理する / Process EU requests in an overnight batch in the US instead of synchronously

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

データレジデンシー要件は「どこで処理されるか」の問題なので、**処理を要件のある地域に置く**のが直接的な解です。同一コードベースをリージョン別にデプロイすれば、開発チームの要求（単一コードベース）とレジデンシー要件を同時に満たせます。居住地の判定を認証時に確定させるのは重要で、リクエストごとの推測に任せると誤ルーティングが生じます。

Residency is a question of *where processing happens*, so the direct answer is to place processing in the required region. One codebase deployed per region satisfies both the team's constraint and the regulation; resolving residency at authentication avoids per-request misrouting.

- **A 不正解**: 仮名化データは GDPR 上なお個人データであり、越境移転の根拠にはなりません（匿名化とは異なります）。 / Pseudonymized data is still personal data under GDPR and does not legitimize the transfer.
- **C 不正解**: 法人を分けても、処理が米国リージョンで行われる限りデータ移転の問題は残ります。法的形態は技術的な処理場所を変えません。 / A separate entity does not change where processing physically occurs.
- **D 不正解**: バッチ化はタイミングを変えるだけで処理場所を変えず、加えて EU 顧客の体験を一方的に劣化させます。 / Batching changes timing, not location, and degrades the EU experience.

**参照 / Reference:** データレジデンシー、マルチリージョン設計、GDPR
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

コールセンターのオペレーター支援ツールで、通話中にリアルタイムで回答候補を提示します。SLA は「**p99 で初回トークン 800ms 以内**」。現在は Opus クラスのモデルに 40,000 トークンのナレッジベース全文を毎回渡して回答生成しており、p50 は 1.4 秒、p99 は 4.2 秒です。回答品質そのものには不満が出ていません。

An operator-assist tool for a call center surfaces answer candidates in real time during calls. The SLA is **p99 time-to-first-token ≤ 800 ms**. Today an Opus-class model receives the full 40,000-token knowledge base on every request; p50 is 1.4 s and p99 is 4.2 s. Answer quality itself is not a complaint.

**設問 / Question:**

SLA を満たすために最も効果的なアプローチはどれですか？

Which approach is most effective for meeting the SLA?

- A) タイムアウトを 800ms に設定し、超過したリクエストは破棄してオペレーターに「該当なし」と表示する / Set an 800 ms timeout and show "no match" to the operator when exceeded
- B) **ストリーミングを有効化**して初回トークンまでの時間を短縮し、あわせて固定のナレッジベース部分に**プロンプトキャッシュ**を適用、さらに検索で関連チャンクのみに絞ってプロンプトを縮小する。品質が保てる範囲で軽量モデルへの切り替えを評価する / **Enable streaming** to cut time-to-first-token, apply **prompt caching** to the static knowledge-base prefix, retrieve only relevant chunks to shrink the prompt, and evaluate a lighter model where quality holds
- C) サーバーを増設して並列度を上げる / Scale out the server fleet to increase concurrency
- D) ナレッジベースを 80,000 トークンに拡張してより良い回答を出す / Expand the knowledge base to 80,000 tokens for better answers

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**初回トークンまでの時間**が SLA なので、対策は「入力処理時間の削減」と「出力開始の前倒し」に向かいます。ストリーミングは初回トークンを直接改善し、プロンプトキャッシュは固定接頭辞の処理時間を削減し、検索による絞り込みは入力トークン数そのものを減らします。この 3 つは相互に補完的で、いずれも品質を大きく損なわずに効きます。モデル軽量化は最後に評価すべき選択肢です。

The SLA is on *time to first token*, so the levers are reducing input processing and starting output sooner. Streaming attacks TTFT directly, caching removes reprocessing of the static prefix, and retrieval shrinks the input. These compose; model downsizing is the last lever to evaluate, not the first.

- **A 不正解**: タイムアウトは SLA 違反を隠蔽するだけで、オペレーターは支援を受けられません。可用性を犠牲にしたメトリクス操作です。 / A timeout hides the violation and denies the operator help — gaming the metric.
- **C 不正解**: 並列度はスループットを改善しますが、単一リクエストのレイテンシは改善しません。負荷が原因でない限り無効です。 / Concurrency improves throughput, not single-request latency.
- **D 不正解**: 入力を倍増させればレイテンシは悪化します。品質は問題になっていません。 / Doubling input worsens latency, and quality was not the complaint.

**参照 / Reference:** レイテンシ最適化、ストリーミング、プロンプトキャッシュ
</details>

---

## 問題 6 / Question 6

**シナリオ / Scenario:**

マルチテナント SaaS で、1 つの大口テナントがバッチ処理を開始すると組織全体の Claude API レート制限を消費し尽くし、他の 200 テナントの対話型リクエストが 429 で失敗する障害が月に数回発生しています。大口テナントの処理自体は正当な利用で、契約上も許容されています。

In a multi-tenant SaaS, when one large tenant starts a batch job it exhausts the organization-wide Claude API rate limit, and interactive requests from the other 200 tenants fail with 429 several times a month. The large tenant's usage is legitimate and contractually permitted.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) 429 が発生したら指数バックオフでリトライする / Retry with exponential backoff on 429
- B) 大口テナントに夜間バッチへの移行を依頼する / Ask the large tenant to move to an overnight batch window
- C) 組織全体のレート制限の引き上げを申請する / Request a higher organization-wide rate limit
- D) **テナントごとのクォータと、対話型／バッチのワークロード分離**を自前のスケジューラで実装する。対話型トラフィックに予約容量を確保し、バッチは残余容量で動く低優先度キューに載せ、バッチ側にはバックオフとバッチ API の利用を適用する / Implement **per-tenant quotas and interactive/batch workload separation** in your own scheduler: reserve capacity for interactive traffic, run batch on a low-priority queue over residual capacity, and apply backoff and the Batch API on the batch path

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

問題の本質は「**共有資源に公平性の統制がない**」ことです。レート制限は組織単位で課されるので、テナント単位・ワークロード単位の配分はアプリケーション側で実装するしかありません。対話型に予約容量を確保することで、バッチの正当な利用が対話型の SLA を壊さなくなります。バッチ API は遅延許容ワークロードのスループットを別枠で確保する手段として適合します。

The root cause is the absence of fairness control over a shared resource. Rate limits are enforced per organization, so per-tenant and per-workload allocation must live in your scheduler. Reserving interactive capacity lets legitimate batch usage coexist with the interactive SLA.

- **A 不正解**: バックオフは失敗を遅らせるだけで、容量の奪い合い自体を解決しません。全テナントが同時にバックオフすると対話型の遅延が悪化します。 / Backoff defers failure without resolving contention, and worsens interactive latency.
- **B 不正解**: 運用上の依頼は契約で許容された利用を制限するもので、技術的統制の欠如を顧客に転嫁しています。再発も防げません。 / Pushing the constraint onto a customer whose usage is permitted; no structural fix.
- **C 不正解**: 上限を上げても配分の仕組みがなければ、より大きなバッチが再び使い切るだけです。 / Without allocation, a larger batch simply exhausts the larger limit.

**参照 / Reference:** 容量計画、公平配分、ワークロード分離、Batch API
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

EC サイトの商品検索を Claude ベースのセマンティック検索に置き換えました。移行後、Claude API 側の障害で 22 分間全リクエストが失敗し、その間サイトの検索機能が完全に停止して売上に直接影響が出ました。従来のキーワード検索エンジンはまだ稼働していますが、新実装からは呼ばれていません。

An e-commerce site replaced product search with Claude-based semantic search. After the migration, a 22-minute upstream outage took search down entirely and directly hit revenue. The legacy keyword search engine is still running but is no longer called by the new implementation.

**設問 / Question:**

最も適切なアーキテクチャ上の改善はどれですか？

Which architectural improvement is most appropriate?

- A) Claude API 呼び出しが失敗またはタイムアウトした場合に、**既存のキーワード検索へ自動フォールバック**する縮退経路を実装する。フォールバック中であることを内部メトリクスで可視化し、UI では機能低下を明示せず結果を返す / Implement a **graceful-degradation path that automatically falls back to the existing keyword search** on failure or timeout, expose fallback state in internal metrics, and still return results to the user
- B) リトライ回数を 5 回に増やす / Increase retries to five attempts
- C) 障害時にエラーページを表示し、ユーザーに再試行を促す / Show an error page asking the user to retry
- D) 検索結果を 24 時間キャッシュして障害時に配信する / Cache search results for 24 hours and serve them during outages

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

外部依存を持つコンポーネントには、**依存が落ちても機能が完全には停止しない縮退経路**が必要です。ここでは劣化した品質でも動作する既存資産（キーワード検索）が存在するのに使われていないことが設計上の欠陥です。「品質は落ちるが機能は生きている」状態は、売上を守るうえで「完全停止」よりはるかに良い結果になります。フォールバック発生率をメトリクス化しないと、劣化が常態化しても気づけません。

Any component with an external dependency needs a degradation path. Here a lower-quality but working asset already exists and simply is not wired in. "Degraded but alive" protects revenue far better than "down," and instrumenting fallback rate is what keeps degradation from becoming invisible and permanent.

- **B 不正解**: 22 分の上流障害中はリトライも失敗します。リトライ回数増は瞬断には効きますが持続的障害には無力で、負荷を増やします。 / Retries also fail during a 22-minute outage and add load.
- **C 不正解**: エラー表示は可用性の問題を解決せず、代替手段があるのに使っていない点が変わりません。 / Surfacing the error does not restore function when an alternative exists.
- **D 不正解**: 検索クエリは多様なのでキャッシュヒット率が低く、新規クエリは救えません。また 24 時間前の在庫・価格を返すのは EC では有害です。 / Low hit rate for diverse queries, and stale inventory/pricing is harmful in e-commerce.

**参照 / Reference:** 縮退運転、障害ドメイン設計、可用性設計
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

M&A デューデリジェンス支援ツールで、1 案件あたり平均 3,000 ファイル（契約書・財務諸表・訴訟記録）を分析し、リスク一覧を生成します。処理には平均 40 分かかります。現在は HTTP リクエスト・レスポンスの同期処理で実装しており、ロードバランサのタイムアウト（60 秒）、ブラウザのタイムアウト、処理中のデプロイによる中断が頻発しています。ユーザーは進捗を知りたがっています。

An M&A due-diligence tool analyzes on average 3,000 files per deal (contracts, financials, litigation records) and produces a risk register. Processing takes ~40 minutes. It is implemented as a synchronous HTTP request/response, and load-balancer timeouts (60 s), browser timeouts, and mid-processing deploys interrupt it constantly. Users want progress visibility.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) ロードバランサとブラウザのタイムアウトを 60 分に延長する / Raise the load-balancer and browser timeouts to 60 minutes
- B) 3,000 ファイルを 50 ファイルずつに分けてユーザーに 60 回リクエストさせる / Have users issue 60 requests of 50 files each
- C) **非同期ジョブアーキテクチャ**に変更する。リクエスト受付時にジョブ ID を返し、処理はワーカーがキュー経由で実行、ファイル単位で結果を永続化して進捗を更新する。クライアントはジョブ ID でポーリングまたは通知を受け取り、ワーカー再起動時は未完了ファイルから再開する / Move to an **asynchronous job architecture**: return a job ID on submission, process via queued workers, persist results and progress per file, let clients poll or subscribe by job ID, and resume from unfinished files after a worker restart
- D) 処理を高速化して 60 秒以内に収まるようにモデルを Haiku に変更する / Switch to Haiku so processing fits within 60 seconds

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

40 分かかる処理を同期リクエストに載せているのが根本原因です。非同期ジョブ化すると、タイムアウト問題・進捗可視化・デプロイ耐性・再開可能性が**同時に**解決します。ファイル単位での永続化が再開の単位になり、ワーカーの再起動やデプロイが致命傷にならなくなります。長時間処理の設計として標準的な形です。

Putting a 40-minute job on a synchronous request is the root cause. Making it asynchronous resolves timeouts, progress visibility, deploy tolerance, and resumability *simultaneously*, with per-file persistence as the resumption unit.

- **A 不正解**: タイムアウト延長は接続を長時間占有し、デプロイ中断や進捗可視化の問題は残ります。60 分の HTTP 接続は運用上も脆弱です。 / Long-lived connections remain fragile and solve neither deploys nor progress.
- **B 不正解**: システムの設計上の問題をユーザーの手作業に転嫁しています。60 回の手動操作は現実的ではありません。 / Offloads a design problem onto the user.
- **D 不正解**: 3,000 ファイルの分析を 60 秒に収めることはモデル変更では不可能で、品質も犠牲になります。 / Not achievable by model choice, and sacrifices quality.

**参照 / Reference:** 非同期ジョブ設計、長時間処理、チェックポイントと再開
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

法務部門向けに、毎晩 12,000 件の契約書を分析して条項リスクを抽出するバッチ処理を運用しています。各リクエストは同一の 18,000 トークンの「審査基準書＋出力スキーマ＋Few-shot 例」を接頭辞として持ち、その後に契約書本文（平均 6,000 トークン）が続きます。結果は翌朝 9 時までに揃えばよく、リアルタイム性は不要です。月額の API コストが予算を超過しました。

A nightly batch analyzes 12,000 contracts for clause risk. Every request carries an identical 18,000-token prefix (review criteria + output schema + few-shot examples) followed by the contract body (~6,000 tokens). Results are only needed by 9 a.m.; there is no real-time requirement. Monthly API cost has exceeded budget.

**設問 / Question:**

コストを下げるために最も効果的な組み合わせはどれですか？

Which combination most effectively reduces cost?

- A) **プロンプトキャッシュで共通接頭辞 18,000 トークンを再利用し、あわせて Batch API を利用する**。さらに、契約書をタスクの難易度で振り分け、定型契約は軽量モデル、非定型のみ上位モデルに回す / **Cache the shared 18,000-token prefix and submit through the Batch API**, then tier the work: route standard contracts to a lighter model and only non-standard ones to a stronger model
- B) 契約書本文を 3,000 トークンに要約してから分析する / Summarize each contract to 3,000 tokens before analysis
- C) 分析対象を 12,000 件から 6,000 件にサンプリングする / Sample 6,000 of the 12,000 contracts
- D) 出力トークン数の上限を厳しく設定する / Set a tight cap on output tokens

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

このワークロードはコスト最適化の教科書的な条件を 3 つ揃えています。(1) **共通接頭辞が長い**（プロンプトキャッシュが効く）、(2) **遅延許容**（Batch API が使える）、(3) **難易度が不均一**（モデル階層化が効く）。いずれも**出力品質を犠牲にせずに**コストを下げる手段であり、組み合わせると効果が乗算的になります。品質を落とす選択肢を選ぶ前に、これらを尽くしたかを問うのが本問の主眼です。

This workload has all three textbook conditions: a long shared prefix (caching), latency tolerance (Batch API), and non-uniform difficulty (model tiering). All three reduce cost **without** sacrificing output quality, and they compose.

- **B 不正解**: 要約は情報の欠落を招き、条項リスク抽出という精度が要る用途で品質を直接損ないます。要約自体にもコストがかかります。 / Summarization drops the very details clause-risk extraction depends on, and costs tokens itself.
- **C 不正解**: 半分しか見ないのはコスト削減ではなくカバレッジの放棄です。法務レビューでは未検査の契約がリスクとして残ります。 / Halving coverage is not cost optimization; unreviewed contracts are unmanaged risk.
- **D 不正解**: 出力上限は結果を途中で切る危険があり、削減額も入力側に比べて小さいです（入力 24,000 対 出力数百トークン）。 / Truncation risk, and small savings relative to the input-dominated cost.

**参照 / Reference:** プロンプトキャッシュ、Batch API、モデル階層化、コスト最適化
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

産業機械の遠隔保守エージェントを設計しています。エージェントは診断ツールに加え、`restart_controller`（制御機の再起動）と `update_firmware`（ファームウェア更新）を呼べます。前者は数分のライン停止で済みますが、後者は失敗すると機械が起動不能になり、現地技術者の派遣（数日・数百万円）が必要です。顧客は「できるだけ自動化してダウンタイムを減らしたい」と要望しています。

You are designing a remote-maintenance agent for industrial machinery. Besides diagnostics, it can call `restart_controller` (a few minutes of line downtime) and `update_firmware` (which, if it fails, bricks the machine and requires an on-site engineer — days and millions of yen). The customer wants "as much automation as possible to cut downtime."

**設問 / Question:**

人間の承認ゲートの配置として最も適切なのはどれですか？

Where should the human approval gate be placed?

- A) 顧客の要望どおり両方を自動実行し、失敗時に通知する / Automate both as requested and notify on failure
- B) 両方に承認ゲートを置き、すべての操作を人間が承認する / Gate both, requiring human approval for every operation
- C) **`restart_controller` は自動実行を許可し、`update_firmware` には人間の承認ゲートを置く**。可逆性と失敗時の影響の大きさで自動化の度合いを分け、承認画面には診断根拠・対象機体・ロールバック可否を提示する / **Allow `restart_controller` to run automatically and gate `update_firmware` behind human approval**, deciding automation by reversibility and failure impact, and presenting the diagnosis, target machine, and rollback availability on the approval screen
- D) エージェントに「危険な操作の前は慎重に判断せよ」と指示し、モデルの判断に委ねる / Instruct the agent to "be careful before dangerous operations" and let the model decide

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

自動化の度合いは**技術的な可能性ではなくリスクで決めます**。判断軸は「可逆性」と「失敗時の影響の大きさ」の 2 つで、再起動は可逆かつ影響が小さいので自動化に適し、ファームウェア更新は不可逆かつ影響が甚大なので承認ゲートが必要です。顧客の「できるだけ自動化」という要望は、リスクの説明とセットで境界を合意し直すべき対象で、そのまま実装する要求ではありません。承認画面に判断材料を出すことが、ゲートを形骸化させないための条件です。

Automation level is set by risk, not by technical possibility. The axes are reversibility and blast radius: a restart is reversible and small, firmware update is neither. The customer's "automate as much as possible" is an input to renegotiate the boundary against risk, not a specification to implement literally — and the approval screen must carry the evidence, or the gate becomes a rubber stamp.

- **A 不正解**: 不可逆かつ高影響の操作を無人で実行する設計で、事後通知は文鎮化した機械を元に戻しません。 / Post-hoc notification does not un-brick a machine.
- **B 不正解**: 低リスク操作にまでゲートを課すと承認疲れを招き、承認が形骸化して高リスク操作の審査品質まで下がります。 / Gating low-risk actions causes approval fatigue that degrades scrutiny of the high-risk ones.
- **D 不正解**: 「慎重に」は確率的な指示で、不可逆操作に対する統制になりません。 / A probabilistic instruction is not a control over irreversible actions.

**参照 / Reference:** Human-in-the-loop の配置、可逆性による自動化度の決定
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

銀行のコールセンターで、既存のルールベース応答システムを Claude エージェントに置き換えます。現行システムは年間 400 万件を処理しており、誤応答は苦情と規制上の報告義務に直結します。経営層は 3 か月での全面切替を求めていますが、あなたは新システムの実運用データでの挙動を把握できていません。

A bank is replacing a rules-based call-center response system with a Claude agent. The current system handles 4 million interactions a year, and incorrect responses lead to complaints and mandatory regulatory reporting. Leadership wants a full cutover in three months; you have no production-behavior data for the new system.

**設問 / Question:**

最も適切な移行戦略はどれですか？

Which migration strategy is most appropriate?

- A) 3 か月後に全面切替し、問題があれば旧システムに戻す / Cut over fully in three months and roll back if problems appear
- B) **シャドーモードから始める**。本番トラフィックを新システムにも流すが応答は顧客に返さず、旧システムの応答と差分を比較して評価する。合意した品質基準を満たしたら、低リスクな問い合わせ種別から少量のカナリアに進め、種別と比率を段階的に広げる。各段階に明示的な後退条件を定義する / **Start in shadow mode**: mirror production traffic to the new system without serving its responses, and diff against the legacy responses. Once agreed quality thresholds are met, canary a small share of the lowest-risk intent categories, widening category and share in stages, with explicit rollback criteria at each stage
- C) 新規顧客のみ新システムに割り当て、既存顧客は旧システムのままにする / Route only new customers to the new system
- D) 社内スタッフによるテストを 3 か月実施してから全面切替する / Run three months of internal staff testing, then cut over fully

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

実運用データがない状態での高リスク切替に対する標準的な答えが**シャドー → カナリア → 段階拡大**です。シャドーは顧客影響ゼロで本番分布の挙動を測定でき、旧システムという比較基準があることがこの状況の強みです。**各段階の後退条件を事前に定義する**ことが要点で、これがないとカナリアは「問題が起きても押し切る」運用になります。経営層の期限は、この段階を短縮する理由ではなく、段階ごとの合格基準を合意する対象です。

Shadow → canary → staged expansion is the standard answer for a high-risk cutover with no production data. Shadow measures real-distribution behavior at zero customer risk, and the legacy system provides the comparison baseline. Pre-defining rollback criteria per stage is what keeps the canary honest.

- **A 不正解**: ロールバック可能でも、切替後に発生した誤応答と規制報告は取り消せません。 / Rollback does not retract incorrect responses already given or reports already triggered.
- **C 不正解**: 新規顧客は分布が偏っており、かつ最も関係が脆い層に未検証システムを当てることになります。 / New customers are a skewed, and the most fragile, population to experiment on.
- **D 不正解**: 社内テストは本番の入力分布を再現できず、400 万件規模のロングテールを見落とします。 / Internal testing cannot reproduce the production distribution or its long tail.

**参照 / Reference:** シャドーモード、カナリアリリース、段階的ロールアウト
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

社内ナレッジ検索エージェントが、人事規程・技術文書・営業資料・法務文書の 4 領域を横断して回答します。運用開始後、「人事規程の質問なのに技術文書の用語で回答する」「法務の免責文言が営業回答に混入する」といった問題が報告されています。1 つのシステムプロンプトに 4 領域すべての指示・用語定義・注意事項を詰め込んだ結果、9,000 トークンに肥大化しています。

An internal knowledge agent answers across four areas: HR policy, engineering docs, sales collateral, and legal. In production, users report answers to HR questions phrased in engineering terminology, and legal disclaimers bleeding into sales answers. A single system prompt now carries instructions, glossaries, and caveats for all four areas and has grown to 9,000 tokens.

**設問 / Question:**

最も適切なアーキテクチャ上の改善はどれですか？

Which architectural improvement is most appropriate?

- A) システムプロンプトに「領域を混同しないこと」という指示を追加する / Add an instruction not to mix areas
- B) システムプロンプトを 15,000 トークンに拡張し、各領域の区別をより詳細に説明する / Expand the system prompt to 15,000 tokens with more detailed distinctions
- C) 4 領域の文書をすべて 1 つのベクトルストアに統合する / Merge all four areas' documents into a single vector store
- D) **入口で領域をルーティングし、領域ごとに専用のシステムプロンプト・ツール・データソースを持つ構成に分割**する。領域をまたぐ質問はコーディネーターが該当領域それぞれに委譲し、結果を統合する / **Route by area at the entry point and split into per-area configurations** — each with its own system prompt, tools, and data sources — with a coordinator delegating cross-area questions to the relevant areas and merging the results

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

症状は典型的な**コンテキスト汚染**です。無関係な領域の指示と用語が同一コンテキストに同居すると、モデルはそれらを区別しきれません。指示を足して解決しようとするのは、原因（同居そのもの）ではなく症状に対処する試みです。領域ごとに分離すれば各コンテキストは短く一貫し、精度とコストの両方が改善します。領域横断の質問はコーディネーター経由で扱うことで、分離を保ったまま対応できます。

This is classic context contamination: unrelated instructions and glossaries sharing one context cannot be reliably separated by the model. Splitting per area makes each context short and coherent, improving both accuracy and cost, while a coordinator preserves cross-area coverage.

- **A 不正解**: 汚染された同一コンテキスト内での指示追加は、区別の負荷をモデルに押し付けるだけで根本原因が残ります。 / Adding instructions inside the contaminated context leaves the cause intact.
- **B 不正解**: 肥大化が原因なのでプロンプトをさらに拡大するのは逆効果で、コストとレイテンシも悪化します。 / Growing the prompt worsens the very cause, plus cost and latency.
- **C 不正解**: データソースの統合は領域境界をさらに曖昧にし、混入を悪化させます。 / Merging sources blurs the boundaries further.

**参照 / Reference:** コンテキスト分離、ルーティング、サブエージェント分割
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

技術文書 Q&A システムで、参照対象は 4,000 ページ（約 300 万トークン）のマニュアル群です。更新は四半期に 1 回。質問の 90% は特定の 1〜2 セクションで答えられますが、残り 10% は複数章を横断した理解を要します。現在は毎回 RAG で上位 8 チャンクを取得していますが、横断的な質問への回答品質が低いという指摘があります。

A technical-docs Q&A system references 4,000 pages (~3M tokens) of manuals, updated quarterly. 90% of questions are answerable from one or two sections; the remaining 10% require understanding across chapters. Today it always retrieves the top 8 RAG chunks, and cross-chapter answers are reported as poor.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) 300 万トークン全文を毎回コンテキストに入れる / Put all 3M tokens in context on every request
- B) 取得チャンク数を 8 から 40 に増やす / Increase retrieval from 8 to 40 chunks
- C) RAG をやめて全文検索に戻す / Abandon RAG and return to full-text search
- D) **質問の性質でルーティングする**。局所的な質問は従来どおり RAG、横断的な質問と判定されたものは該当章の全文（数万トークン規模）をコンテキストに投入する経路に回す。章単位の要約インデックスを用意して、どの章が必要かを判定できるようにする / **Route by question type**: keep RAG for local questions, and send questions classified as cross-cutting down a path that loads the full text of the relevant chapters (tens of thousands of tokens) into context, using a chapter-level summary index to decide which chapters are needed

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

RAG と長コンテキストは排他ではなく、**質問の性質に応じて使い分ける**のが本問の要点です。局所的な質問（90%）にはチャンク取得が最も効率的で、横断的な質問（10%）にはチャンクの断片では文脈がつながらないため章単位の投入が有効です。章単位の要約インデックスは、どの章を丸ごと入れるかを決めるためのルーティング材料になります。90/10 の分布があるので、全件を高コスト経路に載せる必要はありません。

RAG and long context are not mutually exclusive; the point is routing by question type. Chunks are efficient for the 90% local case and structurally insufficient for the 10% cross-cutting case, where whole chapters restore the continuity that chunking destroys. A chapter-level summary index is the routing signal.

- **A 不正解**: 全件で 300 万トークンを投入するとコストとレイテンシが実用外になり、90% の局所質問には過剰です。 / Prohibitive cost and latency, and unnecessary for 90% of traffic.
- **B 不正解**: チャンク数を増やしても断片の集合であることは変わらず、章をまたぐ論理のつながりは復元されません。ノイズも増えます。 / More chunks are still fragments; cross-chapter continuity is not restored, and noise rises.
- **C 不正解**: 全文検索はセマンティックな一致に弱く、横断的理解の問題はさらに悪化します。 / Full-text search is weaker on semantics and worsens the failing case.

**参照 / Reference:** RAG vs 長コンテキスト、ハイブリッド検索、質問ルーティング
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

決済事業者で、不正取引検知の説明文生成に Claude を使っています。金融当局からの要求で、システムは **DORA（デジタル・オペレーショナル・レジリエンス法）** の観点から「重要な ICT サードパーティへの依存が単一障害点にならないこと」を示す必要があります。現在は単一のプロバイダー経由でのみ Claude を呼び出しています。

A payments provider uses Claude to generate explanations for fraud-detection decisions. Under **DORA**, the regulator requires evidence that dependence on a critical ICT third party is not a single point of failure. Today Claude is called through exactly one provider path.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) プロバイダーの SLA 文書を当局に提出して可用性を説明する / Submit the provider's SLA document to the regulator as evidence of availability
- B) **複数の到達経路を用意する**。同一モデルファミリーに対して first-party API と Bedrock / Vertex など複数の経路を構成し、抽象化レイヤーで切替可能にする。あわせて、いずれの経路も使えない場合の**機能縮退の手順**（説明文なしでの検知継続、事後生成）を定義し、切替と縮退を定期的に演習する / **Provision multiple access paths** to the same model family — first-party API plus Bedrock/Vertex — behind an abstraction layer, and additionally define a **degradation procedure** for when no path is available (continue detection without explanations, generate them later), exercising both switchover and degradation on a schedule
- C) 内製の小規模モデルに全面的に置き換えて外部依存をなくす / Replace everything with a small in-house model to remove the external dependency
- D) 障害時にはシステムを停止し、全取引を手動審査に回す / Halt the system during outages and route all transactions to manual review

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

DORA が求めるのは**集中リスクの緩和と、それを実証できること**です。同一モデルへの複数到達経路は切替可能性を与え、抽象化レイヤーがそれを実装可能にします。しかし経路の冗長化だけでは全経路障害を説明できないため、**縮退手順**が対になって必要です。さらに「演習していること」が規制対応では決定的で、設計上可能なだけの切替は実証になりません。

DORA asks for concentration-risk mitigation that can be *evidenced*. Multiple paths to the same model give switchability, an abstraction layer makes it implementable, and a degradation procedure covers the all-paths-down case. Crucially, regulators credit exercised capability — a switchover that is only theoretically possible is not evidence.

- **A 不正解**: SLA は可用性の約束であって、依存の集中を緩和する統制ではありません。単一障害点の存在自体は変わりません。 / An SLA is a promise, not a control; the single point of failure remains.
- **C 不正解**: 品質要件を満たせない可能性が高く、内製化は集中リスクを別の場所（自社の運用能力）に移すだけです。 / Likely fails quality requirements and merely relocates the concentration risk.
- **D 不正解**: 全停止は事業継続性の観点でむしろ DORA が避けさせたい結果です。手動審査は数量的に成立しません。 / A full stop is the outcome DORA seeks to avoid, and manual review does not scale.

**参照 / Reference:** DORA、集中リスク、マルチプロバイダー構成、縮退手順
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

消費者向け融資の審査で、Claude が申込内容の要約と推奨を生成し、人間の審査担当者が最終判断します。米国の公正信用機会法（ECOA / Regulation B）により、否認時には**具体的な主要因**を記載した adverse action notice を交付する義務があります。現在の実装では、Claude の推奨理由が自由記述テキストとしてのみ残り、どの入力項目が判断に効いたかを追跡できません。

In consumer lending, Claude summarizes applications and produces a recommendation; a human underwriter decides. Under ECOA / Regulation B, a denial requires an adverse action notice stating the **specific principal reasons**. Today Claude's rationale is retained only as free-form text, and there is no traceability from decision to input fields.

**設問 / Question:**

最も適切なアーキテクチャ上の改善はどれですか？

Which architectural improvement is most appropriate?

- A) 自由記述の理由文をそのまま adverse action notice に転記する / Copy the free-form rationale directly into the adverse action notice
- B) 審査担当者に理由を手書きさせ、Claude の出力は参考情報にとどめる / Have underwriters hand-write reasons and treat Claude's output as reference only
- C) **理由を構造化出力にする**。規制で認められた理由コードの列挙から選ばせ、各コードに対応する入力項目値と閾値を併記させる。生成物と入力スナップショット・モデルバージョン・プロンプトバージョンを紐付けて保存し、通知文はコードから決定的にレンダリングする / **Make the rationale structured output**: constrain it to an enumeration of permissible reason codes, require the contributing input field values and thresholds per code, store the output alongside an input snapshot with model and prompt versions, and render the notice deterministically from the codes
- D) 否認された申込のみ人間が理由を再分析する / Re-analyze reasons manually only for denied applications

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

規制が要求するのは「具体的な主要因」であり、これは**列挙可能な理由コードと、それを裏付ける入力値**として表現できます。自由記述を構造化出力に変えることで、通知文の生成が決定的になり（モデルが通知文そのものを書かない）、監査時に「この判断はこの入力に基づく」と再現できます。入力スナップショットとバージョンの保存は、数年後の監査で当時の判断を再現するために不可欠です。

The regulation demands specific principal reasons, which is expressible as an enumeration plus supporting input values. Structured output makes notice generation deterministic — the model does not author the notice — and versioned input snapshots are what let an audit years later reproduce the decision.

- **A 不正解**: 自由記述をそのまま交付すると、規制上要求される特定性を満たさない文言や不適切な理由が通知に混入し得ます。 / Free-form text risks non-compliant or impermissible wording in a regulated notice.
- **B 不正解**: 手書きは追跡可能性を担保しますがスケールせず、Claude を使う意義が失われます。担当者間の一貫性も保証されません。 / Traceable but unscalable and inconsistent across underwriters.
- **D 不正解**: 事後の再分析は当時の判断の再現ではなく、監査上は別の分析にすぎません。 / Post-hoc re-analysis is a different analysis, not a reproduction of the decision.

**参照 / Reference:** 構造化出力、ECOA / Regulation B、判断の再現性と監査証跡
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

EC サイトの商品説明生成機能で、通常時は毎分 200 リクエストですが、セール開始直後の 15 分間に毎分 6,000 リクエストまで跳ね上がります。この瞬間的な負荷に合わせて容量を確保すると平常時のコストが 30 倍になります。商品説明はページ表示時に生成されますが、実際には**生成から表示まで数秒の遅れは許容**され、生成済みなら再利用できます。

A product-description feature normally sees 200 requests/min but spikes to 6,000/min for the 15 minutes after a sale opens. Provisioning for the peak would cost 30× the steady-state. Descriptions are generated on page view, but **a few seconds of delay before display is acceptable** and generated descriptions are reusable.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) **キューによる負荷平準化**を導入する。リクエストをキューに投入し、ワーカーが一定レートで処理する。生成済み説明はキャッシュから即時返し、未生成のものは暫定表示（テンプレート）を返してから非同期に差し替える。セール対象商品は開始前に事前生成しておく / Introduce **queue-based load leveling**: enqueue requests and drain them at a fixed worker rate, serve already-generated descriptions from cache immediately, return a templated placeholder for misses and swap it asynchronously, and pre-generate descriptions for sale items before the sale opens
- B) ピークに合わせて常時 6,000 リクエスト/分の容量を確保する / Provision permanently for 6,000 requests/min
- C) セール中は商品説明機能を無効化する / Disable the feature during sales
- D) レート制限を超えたリクエストは 429 を返してクライアントにリトライさせる / Return 429 above the rate limit and let clients retry

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「数秒の遅延が許容される」「生成物が再利用可能」という要件の 2 点が、キューとキャッシュを適合させます。キューは瞬間的なピークを時間方向に均し、キャッシュはそもそもの生成回数を削減し、事前生成は予測可能なピーク（セール対象は事前に分かる）を平常時に移します。3 つを組み合わせると、ピーク容量の確保なしにピークを吸収できます。**予測可能なバーストは事前生成で消せる**という点が本問の要点です。

Two requirements — tolerable delay and reusable output — make queueing and caching the right fit. The queue spreads the spike over time, the cache removes repeat generation, and pre-generation moves a *predictable* peak into off-peak hours. Combined, they absorb the burst without provisioning for it.

- **B 不正解**: 15 分のピークのために 30 倍のコストを常時払う構成で、要件（遅延許容）を活用していません。 / Pays 30× continuously for a 15-minute peak and ignores the latency tolerance.
- **C 不正解**: 最も収益機会の大きいタイミングで機能を落とすのは、ビジネス目的に反します。 / Disables the feature exactly when it matters most commercially.
- **D 不正解**: クライアントリトライは負荷をさらに増幅させ（リトライストーム）、ユーザー体験も悪化します。 / Client retries amplify load and degrade UX.

**参照 / Reference:** キューによる負荷平準化、キャッシュ、事前生成
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

越境 EC の価格表示システムで、Claude が商品情報から関税・消費税・送料を含む着地価格を算出して表示しています。監査で「表示価格の 0.4% に計算誤りがある」と指摘されました。誤りはすべて算術（税率の適用順序、端数処理、複数税率の合算）に起因しており、税率テーブル自体は正確でした。

In a cross-border commerce pricing system, Claude computes landed prices (duties, VAT, shipping) from product data for display. An audit found errors in 0.4% of displayed prices. All errors were arithmetic — order of rate application, rounding, combining multiple rates — while the rate tables themselves were correct.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) システムプロンプトに計算手順を段階的に記述し、途中式を出力させる / Spell out the calculation steps in the system prompt and require intermediate work
- B) より高性能なモデルに変更して計算精度を上げる / Move to a stronger model for better arithmetic
- C) **計算をモデルから取り上げ、決定的な価格計算ツールに委譲**する。Claude の役割は商品情報から計算に必要なパラメータ（原産国・HS コード・重量・仕向地）を抽出することに限定し、算術はコードが行う。抽出結果はスキーマで検証する / **Take the arithmetic away from the model and delegate it to a deterministic pricing tool**: limit Claude to extracting the parameters the calculation needs (origin, HS code, weight, destination), validate that extraction against a schema, and let code do the math
- D) 算出結果を人間が全件確認する / Have a human verify every computed price

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**LLM の適合領域は非構造化情報からの抽出であり、算術ではありません。**表示価格の正確性は 99.6% では足りない性質の要件（消費者保護・景品表示上の問題に直結）なので、決定的なコードに任せるべきです。役割分担を「Claude = 抽出、コード = 計算」と切り直せば、誤りの原因そのものが消えます。抽出結果のスキーマ検証は、誤ったパラメータが正しい計算式に流れ込むのを防ぐ層です。

LLMs are suited to extraction from unstructured input, not arithmetic. Displayed-price accuracy is a requirement where 99.6% is not acceptable, so the math belongs in deterministic code. Re-cutting the boundary to "Claude extracts, code computes" removes the error class entirely.

- **A 不正解**: 途中式の出力は誤りを見えやすくしますが、算術が確率的であることは変わりません。0.4% が 0.1% になっても要件は満たしません。 / Showing work exposes errors without making arithmetic deterministic.
- **B 不正解**: モデル性能の向上は誤り率を下げますが、ゼロにはできません。決定的な要件に確率的手段を当て続けています。 / A stronger model lowers but does not eliminate a probabilistic error rate.
- **D 不正解**: 全件人手確認は表示価格のリアルタイム性と両立せず、スケールもしません。 / Incompatible with real-time display and does not scale.

**参照 / Reference:** 決定論とモデル判断の境界、ツール委譲、構造化抽出
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

B2B SaaS で 60 社のテナントに文書要約機能を提供しています。各社から「自社の用語集を使ってほしい」「要約の粒度を変えたい」「特定のセクションを必ず含めてほしい」といった個別要望が上がり、現在はテナントごとにシステムプロンプトをコピーして手で編集した 60 個の亜種が存在します。共通部分の改善を全社に反映するのに 60 ファイルの手作業が必要で、反映漏れが起きています。

A B2B SaaS offers document summarization to 60 tenants. Each asks for tenant-specific behavior — their own glossary, a different summary granularity, mandatory sections — and today there are 60 hand-edited copies of the system prompt. Propagating a shared improvement means editing 60 files by hand, and omissions occur.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) **共通のプロンプトテンプレートと、テナント設定（用語集・粒度・必須セクション）の分離**を行う。テンプレートはバージョン管理された 1 か所に置き、テナント固有部分は構造化された設定データとして注入する。設定のスキーマを定義し、許容される変更点を明示的に限定する / **Separate a shared prompt template from tenant configuration** (glossary, granularity, mandatory sections): keep one version-controlled template and inject tenant specifics as structured configuration, with a schema that explicitly bounds what a tenant may vary
- B) 60 個の亜種を維持したまま、変更時に一括置換スクリプトを使う / Keep the 60 variants but apply changes with a bulk find-and-replace script
- C) テナント固有の要望は受け付けず、単一のプロンプトに統一する / Stop accepting tenant-specific requests and standardize on one prompt
- D) テナントごとにモデルをファインチューニングする / Fine-tune a model per tenant

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

問題は「テナント差分の存在」ではなく「**差分と共通部分が分離されていない**」ことです。テンプレートと設定を分けると、共通改善は 1 か所の変更で 60 社に届き、テナント差分は設定データとして検証・監査できます。設定スキーマで「変えられる範囲」を限定するのが重要で、これがないと設定という形の亜種が再び増殖します。多テナント SaaS のカスタマイズにおける標準的な形です。

The problem is not that tenants differ, but that the differences are not separated from the shared part. Template-plus-configuration lets one edit reach all 60 while keeping tenant deltas inspectable and auditable, and a configuration schema is what prevents variants from re-emerging in configuration form.

- **B 不正解**: 一括置換は亜種間の差異により失敗しやすく、根本原因（重複）が残ります。 / Fragile across divergent copies, and the duplication remains.
- **C 不正解**: 正当な顧客要件を切り捨てる選択で、B2B SaaS では解約要因になります。技術的問題を商業的譲歩で解決しています。 / Discards legitimate requirements and trades a technical problem for a commercial loss.
- **D 不正解**: 60 個のファインチューンは運用・コスト・評価の負担が桁違いで、用語集程度の差分には過剰です。 / Wildly disproportionate operational cost for glossary-level differences.

**参照 / Reference:** マルチテナントのカスタマイズ、設定駆動設計、プロンプトのバージョン管理
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

高頻度取引部門から「マイクロ秒単位で発注判断を行う既存アルゴリズムを Claude に置き換えたい」という要望が来ました。現行システムの意思決定レイテンシは 40 マイクロ秒、1 日あたり 800 万回の判断を行っています。要望の背景は「ニュース記事の内容を発注判断に取り込みたい」というもので、現行アルゴリズムは数値時系列しか扱えません。

A high-frequency trading desk asks to replace its order-decision algorithm with Claude. The current system decides in 40 microseconds and makes 8 million decisions a day. The motivation is to "incorporate news article content into order decisions"; the existing algorithm handles only numeric time series.

**設問 / Question:**

アーキテクトとして最も適切な提案はどれですか？

As the architect, what is the most appropriate recommendation?

- A) Claude Haiku を使い、レイテンシを最小化して要望に応える / Use Claude Haiku to minimize latency and meet the request
- B) **発注判断そのものの置き換えは不適合であると説明**したうえで、真の要件（ニュース内容の取り込み）を満たす構成を提案する。Claude はニュース記事を**事前に**構造化シグナル（銘柄・事象種別・方向・確信度）に変換して低レイテンシのシグナルストアに書き込み、発注判断は従来どおりマイクロ秒級のアルゴリズムがそのシグナルを読んで行う / **Explain that replacing the decision path is a poor fit**, then propose an architecture that meets the real requirement: Claude converts news into structured signals (instrument, event type, direction, confidence) **ahead of** the decision, writing them to a low-latency signal store that the existing microsecond algorithm reads
- C) 発注判断を Claude に任せ、レイテンシ要件を 40 マイクロ秒から 200 ミリ秒に緩和するよう部門に交渉する / Ask the desk to relax the latency requirement from 40 µs to 200 ms
- D) Claude の判断結果を事前に全パターン計算してテーブル化しておく / Precompute Claude's decisions for all patterns into a lookup table

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

要望をそのまま受けると不適合ですが、**要望の背後にある要件は正当で、かつ Claude に適合します**。鍵は「判断のどこに LLM を挟むか」の設計で、クリティカルパス（マイクロ秒）ではなく、その手前の非同期な前処理（ニュース → 構造化シグナル）に置けば、レイテンシ制約を犯さずに新しい情報源を取り込めます。**適合しない要望を断るだけでなく、適合する形に翻訳して返す**のが Professional で問われる振る舞いです。

The literal request is a misfit, but the underlying requirement is legitimate and well-suited to Claude. The design question is *where* in the decision the LLM sits: off the microsecond critical path, in asynchronous preprocessing. Translating an infeasible request into a feasible architecture — rather than merely refusing it — is the expected behavior.

- **A 不正解**: どのモデルもネットワーク往復だけでマイクロ秒要件を満たせません。モデル選択で解決する問題ではありません。 / No model meets a microsecond budget; network round-trip alone exceeds it.
- **C 不正解**: HFT のレイテンシ要件は事業の前提であり、交渉対象ではありません。5,000 倍の緩和は事業の否定です。 / Latency is the business premise in HFT, not a negotiable parameter.
- **D 不正解**: 市場状態の組み合わせは事実上無限で、事前計算は不可能です。ニュースは新規テキストなので網羅もできません。 / The state space is unbounded and news is novel text; precomputation is impossible.

**参照 / Reference:** ユースケース適合性、クリティカルパス外への配置、要件の翻訳
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

新薬の規制申請文書を作成するエージェントを運用しています。1 件の申請は平均 3 日間・数百ステップにわたり、途中で臨床データの再取得や社内レビュアーの介入が入ります。現在はエージェントセッションのコンテキストに全ての中間成果物を保持しており、プロセス中にワーカーが再起動すると最初からやり直しになります。また 3 日間の間にプロンプトのバグ修正をデプロイしたい場面が生じます。

An agent drafts regulatory submissions for new drugs. A single submission spans ~3 days and hundreds of steps, with clinical-data refetches and internal reviewer interventions along the way. All intermediate artifacts currently live in the agent session's context, so a worker restart forces a restart from scratch — and prompt bug fixes sometimes need to deploy mid-run.

**設問 / Question:**

最も適切な状態管理アーキテクチャはどれですか？

Which state-management architecture is most appropriate?

- A) セッションのコンテキストウィンドウをより大きなモデルに変更して 3 日分を保持する / Move to a larger context window so three days fit in the session
- B) ワーカーの再起動を禁止し、デプロイを申請完了後まで停止する / Forbid worker restarts and freeze deploys until the submission completes
- C) 3 日間の処理を 1 日ごとに分割し、3 回に分けて人間が手動で起動する / Split into three one-day runs started manually by a human
- D) **中間成果物とプロセス状態を外部の永続ストアに置き、セッションは「現在のステップを実行するための最小限のコンテキスト」だけを持つ**設計に変える。各ステップ完了時にチェックポイントを書き、再開時はストアから必要な成果物だけを読み込む。ステップは冪等にし、レビュアー介入は状態遷移として記録する / **Externalize intermediate artifacts and process state to a durable store**, keeping in the session only the minimum context needed for the current step: checkpoint at each step boundary, reload only the artifacts a resumed step needs, make steps idempotent, and record reviewer interventions as state transitions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**長時間プロセスの状態をセッションコンテキストに置くのは設計上の誤り**です。コンテキストは揮発性で、容量に上限があり、デプロイやプロセス再起動をまたげません。状態を外部ストアに移すと、再起動耐性・デプロイ可能性・レビュアー介入の記録・監査証跡が同時に得られます。セッションは「今のステップに必要な分だけ」を持つ短命な計算資源として扱うのが正解の形です。規制申請では、どの中間成果物がどの根拠から生じたかの追跡も要求されます。

Session context is the wrong home for long-running process state: it is volatile, bounded, and cannot survive a deploy. Externalizing state buys restart tolerance, deployability, an intervention record, and an audit trail at once — and the session becomes a short-lived compute resource holding only what the current step needs.

- **A 不正解**: コンテキストを大きくしても揮発性は変わらず、再起動で失われる問題は解決しません。 / A larger context is still volatile; a restart still loses everything.
- **B 不正解**: デプロイ凍結は緊急のバグ修正を阻み、運用上受け入れられません。再起動禁止は実現不可能な前提です。 / Freezing deploys blocks urgent fixes, and forbidding restarts is not achievable.
- **C 不正解**: 分割しても状態の受け渡し方法を定義していないため、同じ問題が 3 回起きます。手動起動は運用負荷も増やします。 / Splitting without defining state handoff reproduces the problem three times.

**参照 / Reference:** 外部チェックポイント、長時間ジョブの状態管理、冪等性
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

企業買収のデューデリジェンスで、対象会社の (1) 財務諸表、(2) 係争中の訴訟、(3) 主要顧客契約、(4) 知的財産、(5) 労務コンプライアンス を並行して調査し、最後に統合レポートを作ります。5 領域は互いに独立して調査でき、各領域の一次資料は数百ページ規模です。過去に単一エージェントで実施したところ、後半の領域の分析品質が明確に低下しました。

For an acquisition due diligence, five areas must be investigated — financials, pending litigation, key customer contracts, IP, and labor compliance — and then synthesized into one report. The areas can be investigated independently, and each has hundreds of pages of primary sources. A previous single-agent attempt showed markedly degraded analysis quality on the later areas.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) 単一エージェントのまま、システムプロンプトで各領域を丁寧に扱うよう指示する / Keep the single agent and instruct it to treat each area carefully
- B) **領域ごとに独立したサブエージェントを並列実行**し、それぞれが自領域の一次資料のみをコンテキストに持つ構成にする。各サブエージェントは所見を構造化形式（発見事項・根拠文書・重大度）で返し、統合エージェントは 5 つの構造化所見のみを受け取ってレポートを生成する / **Run one independent subagent per area in parallel**, each holding only its own primary sources in context; each returns findings in a structured form (finding, source document, severity), and a synthesis agent receives only those five structured result sets to produce the report
- C) 5 領域を順番に処理し、各領域の完了後にコンテキストをクリアして次に進む / Process the five areas sequentially, clearing context between each
- D) 5 領域すべての一次資料を 1 つのコンテキストに入れ、大きなモデルで一度に分析する / Load all five areas' primary sources into one context and analyze in a single call with a large model

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

「独立して調査でき、各領域の資料が大きく、最後に統合が必要」という条件は、**オーケストレーター・ワーカー構成の典型的な適合条件**です。並列化で所要時間が短縮され、コンテキスト分離で領域間の汚染と後半の品質低下が解消されます。統合エージェントが受け取るのを**構造化所見だけ**に限定する点が重要で、一次資料をそのまま統合側に流すと統合側のコンテキストが再び肥大化して同じ問題が起きます。

Independent investigation, large per-area sources, and a final synthesis are the textbook fit for orchestrator-worker. Parallelism cuts wall-clock time and context isolation removes both contamination and the late-area degradation. Restricting the synthesizer's input to *structured findings* is what keeps the problem from reappearing at the synthesis step.

- **A 不正解**: 品質低下の原因はコンテキストの累積であり、指示では解消しません。 / The cause is context accumulation, which instructions do not address.
- **C 不正解**: 逐次＋クリアは汚染を減らしますが並列化の利得がなく、5 領域分の時間がかかります。統合に必要な情報の受け渡し方法も未定義です。 / Reduces contamination but forfeits parallelism and leaves handoff undefined.
- **D 不正解**: 全資料を 1 コンテキストに入れる構成は、まさに品質低下を起こした構成の拡大版です。 / A larger version of the configuration that already failed.

**参照 / Reference:** オーケストレーター・ワーカー、コンテキスト分離、構造化された結果の受け渡し
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

人材紹介会社で、候補者の職務経歴書を Claude が解析して求人とマッチングします。経歴書には氏名・住所・生年月日・現職の会社名などの個人情報が含まれます。個人情報保護の観点から、外部の推論サービスに送るデータを最小化する方針が定められましたが、マッチング精度のために職歴の内容そのものは必要です。また、マッチング結果は候補者の実名で人材コンサルタントに提示する必要があります。

A recruiting firm has Claude parse candidate CVs and match them to openings. CVs contain names, addresses, dates of birth, and current employer names. Policy now requires minimizing personal data sent to external inference services, but match quality depends on the work-history content itself, and results must be presented to consultants under the candidate's real name.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) 個人情報を含むため Claude の利用を断念する / Abandon Claude because personal data is involved
- B) 経歴書全体をそのまま送り、契約でデータの取り扱いを縛る / Send the whole CV and rely on contractual data-handling terms
- C) **境界にトークナイズ層を置く**。送信前に直接識別子（氏名・住所・生年月日・連絡先）を可逆トークンに置換し、職歴の内容は保持したまま推論に回す。結果を受け取った後、アプリケーション側でトークンを実データに復元して表示する。トークンと実データの対応表は自社の管理下に置く / **Put a tokenization layer at the boundary**: replace direct identifiers (name, address, date of birth, contact details) with reversible tokens before sending, preserve the work-history content for inference, and re-substitute the real values in your application when rendering results, keeping the token map under your own control
- D) 経歴書を要約してから送ることで送信データ量を減らす / Summarize the CV before sending to reduce the volume of data

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

「**外部に出す個人データは最小化したいが、内容は必要**」という制約に対する標準的な解が境界でのトークナイズです。直接識別子は推論品質にほぼ寄与しないため置換しても精度を損なわず、可逆であるため実名表示の要件も満たせます。対応表を自社管理下に置くことで、再識別の能力が外部に渡らない構造になります。**要件を「送るデータの種類」の粒度で分解する**のが要点です。

Tokenization at the boundary is the standard answer to "minimize personal data leaving the perimeter, but keep the content." Direct identifiers contribute little to match quality, so replacing them costs no accuracy, and reversibility satisfies the real-name display requirement while the mapping stays under your control.

- **A 不正解**: 要件を分解せずに用途全体を諦めており、実現可能な構成を検討していません。 / Abandons a feasible use case without decomposing the requirement.
- **B 不正解**: 契約は法的な保護であって、技術的なデータ最小化の統制ではありません。方針が求めているのは後者です。 / Contracts are legal protection, not the technical minimization the policy requires.
- **D 不正解**: 要約は識別子を確実には除去せず（氏名が要約に残り得る）、同時に精度に必要な情報も落とします。 / Summarization neither reliably removes identifiers nor preserves matching signal.

**参照 / Reference:** データ最小化、トークナイズ、境界での個人情報処理
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

マルチテナント SaaS の Claude 利用コストが月間で予算の 2.4 倍に達しました。CFO から「どのテナントがいくら使っているのか」「どの機能が高コストなのか」を示すよう求められていますが、現在は組織全体の合計請求額しか把握できておらず、テナント別・機能別の内訳が出せません。値上げか機能制限かの判断ができない状態です。

Claude spend in a multi-tenant SaaS has reached 2.4× budget. The CFO wants to know which tenants and which features drive the cost, but only the organization-wide total is visible today. Without a breakdown, the company cannot decide between repricing and feature limits.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 全テナント一律にリクエスト数の上限を設ける / Impose a uniform per-tenant request cap
- B) 最も安いモデルに全機能を切り替える / Switch every feature to the cheapest model
- C) 翌月の請求書を待って傾向を見る / Wait for next month's invoice to observe the trend
- D) **すべての Claude 呼び出しにテナント ID・機能名・リクエスト ID をメタデータとして付与し、入出力トークン数とキャッシュヒットを含む利用記録を自前で永続化**する。これを集計してテナント別・機能別のコストを可視化し、単価と利用量の両面から施策を判断する / **Tag every Claude call with tenant ID, feature name, and request ID, and persist your own usage records including input/output tokens and cache hits**, then aggregate to expose per-tenant and per-feature cost and decide on both unit price and volume

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**測定できないものは最適化も課金もできません**。コスト配賦のための計装が欠けているのが根本問題で、施策の選定はその後です。呼び出し単位でテナントと機能を紐付けた利用記録を持てば、値上げ（誰に）・機能制限（どれを）・最適化（どこから）のすべてが根拠を持って判断できます。CFO への説明責任という観点でも、内訳のない数字では意思決定できません。

You cannot optimize or charge for what you cannot measure. The missing capability is cost-attribution instrumentation; choosing a remedy comes after. Per-call records tied to tenant and feature make repricing, limiting, and optimization all defensible — and the CFO cannot decide on an undifferentiated total.

- **A 不正解**: 一律上限は重いテナントも軽いテナントも同じに扱い、正当な大口利用を阻害しつつ原因の特定にも寄与しません。 / A uniform cap penalizes legitimate heavy use and identifies nothing.
- **B 不正解**: 品質影響を評価せずに全機能を最安モデルに落とすのは、コスト以外の要件を無視した対応です。 / Ignores quality requirements across every feature at once.
- **C 不正解**: 待っても内訳は得られません。計装を入れない限り翌月も同じ状態です。 / Waiting yields the same undifferentiated total.

**参照 / Reference:** コスト配賦、利用計装、マルチテナントの可観測性
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

顧客に対して「サービス全体で月間 99.9% の可用性」を約束しようとしています。サービスは 4 つの直列依存で構成されます: 自社 API ゲートウェイ（99.99%）、認証サービス（99.95%）、Claude 経由の推論（99.9%）、社内データベース（99.95%）。いずれか 1 つでも落ちるとリクエストは失敗します。

You intend to commit to 99.9% monthly availability for the whole service. It consists of four serial dependencies: your API gateway (99.99%), auth service (99.95%), Claude-based inference (99.9%), and an internal database (99.95%). A failure in any one fails the request.

**設問 / Question:**

この約束について最も適切な判断はどれですか？

Which assessment of that commitment is most appropriate?

- A) **直列依存の可用性は各要素の積になるため、合成可用性は約 99.79% となり 99.9% を約束できない**。約束するには、最も弱い依存（推論）に縮退経路を設けて実質的な可用性を引き上げるか、SLA を合成可用性に見合う水準に設定し直す必要がある / **Serial dependencies multiply, so composite availability is ≈ 99.79% and 99.9% cannot be promised.** To commit to it, add a degradation path around the weakest dependency (inference) to raise effective availability, or restate the SLA at a level the composition supports
- B) 最も低い依存が 99.9% なので、全体も 99.9% を約束できる / The lowest dependency is 99.9%, so 99.9% can be promised for the whole
- C) 平均を取ると 99.95% になるので、99.9% には余裕がある / The average is 99.95%, so there is margin above 99.9%
- D) 各依存が独立しているため、可用性は最も高い依存に近づく / Because the dependencies are independent, availability approaches that of the highest one

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

直列依存の合成可用性は積で求めます。0.9999 × 0.9995 × 0.999 × 0.9995 ≈ 0.99790、すなわち約 99.79%（月間ダウンタイム約 91 分）で、99.9%（約 43 分）には届きません。**依存を足すほど可用性は下がる**という基本を踏まえ、約束を下げるか、弱い依存の影響を縮退経路で切り離すかの二択になります。実現不可能な SLA を約束することは、Professional では技術的誤りであると同時に対外的なリスクです。

Serial availability multiplies: 0.9999 × 0.9995 × 0.999 × 0.9995 ≈ 99.79% (≈ 91 minutes/month) against 99.9% (≈ 43 minutes). Every added dependency lowers the composite, leaving two honest options: lower the promise, or decouple the weakest dependency with a degradation path. Committing to an unachievable SLA is both a technical error and an external risk.

- **B 不正解**: 最小値ではなく積になります。直列依存の基本を誤っています。 / Serial composition is a product, not a minimum.
- **C 不正解**: 平均は可用性の合成に対して意味を持ちません。 / Averaging is meaningless for composing availability.
- **D 不正解**: 独立した直列依存は可用性を下げます。上げるのは冗長化（並列）構成です。 / Independent *serial* dependencies reduce availability; only redundancy raises it.

**参照 / Reference:** 合成可用性、SLA 設計、エラーバジェット
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

アーキテクチャレビューで次の構成が提示されました。全社の 12 のエージェントアプリケーションが、単一の中央 MCP ゲートウェイを経由して 30 のバックエンドツール（DB、CRM、決済、社内 API 等）にアクセスします。ゲートウェイは認証・認可・監査ログ・レート制限を一元的に担い、単一のプロセスとして稼働しています。提案者は「一元管理でガバナンスが効く」と説明しています。

An architecture review presents the following: all 12 agent applications in the company reach 30 backend tools (DB, CRM, payments, internal APIs) through a single central MCP gateway. The gateway centralizes authentication, authorization, audit logging, and rate limiting, and runs as a single process. The proposer argues it "centralizes governance."

**設問 / Question:**

レビュアーとして指摘すべき最も重要な問題はどれですか？

As the reviewer, what is the most important issue to raise?

- A) **ゲートウェイが全社共通の単一障害点かつ単一の攻撃面になっている**。1 つのプロセスの障害が 12 アプリすべてを停止させ、1 つの認可バグが 30 ツールすべてに波及する。ガバナンスの一元化は方針の一元化で達成すべきで、実行経路の一元化と分けるべきである / **The gateway is a company-wide single point of failure and a single attack surface**: one process failure stops all 12 applications, and one authorization bug reaches all 30 tools. Centralized governance should be achieved by centralizing *policy*, not by funneling every execution path through one process
- B) MCP ではなく REST API を使うべきである / REST APIs should be used instead of MCP
- C) ツールが 30 個では多すぎるので 10 個に減らすべきである / 30 tools is too many; reduce to 10
- D) エージェントアプリケーションが 12 個もあるのは統合すべきである / Having 12 agent applications is too many; consolidate them

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「一元管理」は魅力的に聞こえますが、**ポリシーの一元化と実行経路の一元化は別物**です。この構成では可用性（全社停止）とセキュリティ（爆発半径）の両面で単一障害点を作っています。望ましい形は、認可ポリシーと監査要件を中央で定義・配布しつつ、実行経路はドメインや重要度ごとに分割することです。レビューで最も重要な指摘は、提案の利点そのものではなく、その実装方法が生む構造的リスクです。

"Centralization" conflates two separable things: centralizing *policy* and centralizing the *execution path*. This design creates a single point of failure for availability and a single blast radius for security. The better shape defines and distributes policy centrally while splitting execution paths by domain and criticality.

- **B 不正解**: プロトコルの選択はこの構成の中心的な問題ではありません。REST に変えても単一障害点は残ります。 / Protocol choice is not the issue; the SPOF survives the swap.
- **C 不正解**: ツール数は必要性で決まるもので、数そのものが問題ではありません。 / Tool count follows need; the number itself is not the defect.
- **D 不正解**: アプリケーションの統合はむしろ爆発半径を広げる方向で、指摘として逆行しています。 / Consolidating applications widens the blast radius further.

**参照 / Reference:** 単一障害点、爆発半径、ポリシーと実行経路の分離
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

社内で MCP サーバーを整備します。対象は 6 つの基幹システム（人事、会計、在庫、CRM、調達、勤怠）で、それぞれ所管部署・変更頻度・セキュリティ要件が異なります。人事と勤怠は個人情報を扱い、会計は SOX 統制の対象です。設計案として「1 つの巨大な MCP サーバーに 6 システム分のツールをすべて実装する」案と「システムごとに MCP サーバーを分ける」案が出ています。

You are standardizing internal MCP servers across six core systems (HR, accounting, inventory, CRM, procurement, time tracking), each with a different owning department, change cadence, and security posture. HR and time tracking handle personal data; accounting is in SOX scope. Two proposals exist: one monolithic MCP server implementing all six systems' tools, or one MCP server per system.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) 1 つの巨大な MCP サーバーにまとめる。管理対象が 1 つで済み運用が単純になる / One monolithic server: a single artifact is simpler to operate
- B) 使用頻度の高い 3 システムだけ MCP サーバーを作り、残りは直接 API を叩く / Build MCP servers only for the three most-used systems and call the rest directly
- C) ツールの数で分割する（15 ツールごとに 1 サーバー） / Split by tool count (one server per 15 tools)
- D) **システムごとに分割する**。所管部署が異なればリリースサイクルと責任分界も異なり、SOX 対象の会計と個人情報を扱う人事・勤怠は異なるアクセス統制と監査要件を持つ。分割の境界を所有権・変更頻度・セキュリティ要件の境界に一致させ、共通の認証・ログ規約は横断的な標準として定める / **Split per system**: different owning departments mean different release cycles and accountability, and SOX-scoped accounting and personal-data HR/time tracking need different access controls and audit requirements. Align the split with the boundaries of ownership, change cadence, and security posture, and define shared auth and logging conventions as a cross-cutting standard

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

分割の境界は技術的な都合ではなく、**所有権・変更頻度・セキュリティ要件の境界**に合わせます。所管部署が異なるものを 1 つの成果物に同居させると、リリースのたびに無関係な部署の調整が必要になり、SOX 対象コードと非対象コードが混在して監査範囲が不必要に広がります。共通化すべきは実装ではなく規約（認証方式・ログ形式・エラー規約）です。

Split boundaries should follow ownership, change cadence, and security posture, not technical convenience. Co-locating systems with different owners forces unrelated coordination on every release and drags non-SOX code into SOX audit scope. What should be shared is convention — auth, log format, error semantics — not the deployable.

- **A 不正解**: 単一成果物は運用が単純に見えて、実際には調整コストと監査範囲を増やし、爆発半径も広げます。 / Apparent simplicity at the cost of coordination, audit scope, and blast radius.
- **B 不正解**: 一貫性のない二重の統合方式を残す構成で、直接呼び出し側は MCP の認可・監査の恩恵を受けられません。 / Leaves an inconsistent second integration path outside MCP's authorization and audit.
- **C 不正解**: ツール数は分割の基準として意味を持たず、所有権や規制境界を横断してしまいます。 / Tool count is arbitrary and cuts across ownership and regulatory boundaries.

**参照 / Reference:** MCP サーバーの粒度、境界設計、監査範囲の限定
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

金融機関で、毎月末に監督官庁へ提出する取引報告書を Claude が生成します。対象は約 18 万件の取引で、提出期限は月初 5 営業日以内、遅延は行政処分の対象です。処理には現状 6 時間かかります。リアルタイム性の要求はなく、月末の 1 回だけ実行されます。品質要件は極めて高く、誤りは是正報告の対象になります。

A financial institution generates a monthly regulatory transaction report with Claude, covering ~180,000 transactions, due within five business days of month-end; lateness is a sanctionable offense. Processing currently takes six hours. There is no real-time requirement — it runs once, at month-end — and quality requirements are severe, with errors triggering corrective filings.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) リアルタイム API で 18 万件を並列に投げ、最速で完了させる / Fire all 180,000 through the real-time API in parallel for the fastest completion
- B) 提出期限直前に実行して、最新データを反映させる / Run immediately before the deadline to capture the freshest data
- C) **Batch API で処理**し、期限に対して十分なバッファ（複数営業日）を確保した実行計画にする。処理は再実行可能な単位に分割し、失敗分だけを再投入できるようにする。生成結果は提出前に決定的な検証（件数照合、合計値照合、スキーマ検証）を通し、検証不合格分のみ人間がレビューする / **Process through the Batch API** on a schedule with several business days of buffer against the deadline, split into re-runnable units so only failures are resubmitted, and gate submission behind deterministic validation (record-count reconciliation, control-total checks, schema validation) with human review only for validation failures
- D) 18 万件を 1 つの巨大なリクエストにまとめて 1 回で処理する / Combine all 180,000 into a single large request

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

遅延許容かつ大量という条件は Batch API の適合条件そのもので、コスト面でも有利です。しかし本問の核心は**期限に対するバッファと、提出前の決定的な検証**です。規制報告では「速く終わること」より「期限内に、誤りなく提出できること」が要件なので、再実行可能な分割と、件数・合計値の照合という機械的に判定できる検証を挟むのが正解の形になります。人間のレビューは検証不合格分に集中させるのが現実的です。

Latency tolerance plus volume is the Batch API's fit, and it is cheaper. The core of the question, though, is buffer against the deadline and deterministic pre-submission validation: in regulatory reporting the requirement is on-time and correct, not fast. Re-runnable units plus count and control-total reconciliation give a mechanical pass/fail, concentrating human review where it is needed.

- **A 不正解**: 最速化は要件ではなく、大量並列はレート制限に当たり、対話型ワークロードも圧迫します。 / Speed is not the requirement, and mass parallelism hits rate limits and starves interactive traffic.
- **B 不正解**: 期限直前実行は障害時に再実行の余地がなく、行政処分リスクを最大化します。 / No room to re-run on failure; maximizes sanction risk.
- **D 不正解**: 単一巨大リクエストは失敗時に全件やり直しとなり、粒度の細かい再実行ができません。 / A single request forfeits partial re-runs entirely.

**参照 / Reference:** Batch API、規制報告のスケジューリング、決定的な事前検証
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

チームが設計したシステムのレビューを依頼されました。要件は「問い合わせメールを 5 カテゴリに分類し、担当チームのキューに振り分ける」というものです。提案された構成は、意図分類エージェント、感情分析エージェント、緊急度判定エージェント、担当決定エージェント、通知エージェントの 5 つをコーディネーターが束ねるマルチエージェント構成で、それぞれが専用の MCP サーバーを持ちます。現行の手作業では担当者が件名と本文を読んで 10 秒で振り分けています。

You are asked to review a design. The requirement: classify inbound support email into five categories and route it to the owning team's queue. The proposal is a multi-agent system — intent classification, sentiment analysis, urgency scoring, assignment, and notification agents behind a coordinator — each with its own MCP server. Today a human reads the subject and body and routes it in ten seconds.

**設問 / Question:**

レビュアーとして最も適切な指摘はどれですか？

What is the most appropriate review comment?

- A) 感情分析エージェントを追加して 6 エージェント構成にすべき / Add a sixth agent for sentiment
- B) **要件に対して構成が過剰である**。分類と振り分けは単一の構造化出力呼び出し（カテゴリ・緊急度を 1 回で返す）とキュー投入のコードで実現でき、5 エージェントとコーディネーターはレイテンシ・コスト・障害点・運用負荷をいずれも増やすだけで、精度上の利得の根拠が示されていない。まず単純な構成を評価用データセットで測定し、不足が実証された部分にだけ複雑さを足すべきである / **The design is disproportionate to the requirement.** Classification and routing are achievable with one structured-output call returning category and urgency together, plus code to enqueue. Five agents and a coordinator add latency, cost, failure points, and operational burden with no demonstrated accuracy benefit. Measure the simple design against an evaluation set first, and add complexity only where a gap is proven
- C) コーディネーターを 2 段構成にして拡張性を確保すべき / Make the coordinator two-tiered for extensibility
- D) 各エージェントを別々のリポジトリに分けて独立デプロイすべき / Split each agent into its own repository for independent deployment

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**複雑さは要件が要求したときにだけ足す**というのがアーキテクチャの基本原則です。ここでは人間が 10 秒で行える単純な分類タスクに対し、6 コンポーネント・5 MCP サーバーの構成が提案されています。マルチエージェントが正当化されるのは、コンテキスト分離や並列化に実際の必要がある場合で、この要件はいずれにも当たりません。まず最も単純な構成をベースラインとして測定し、**実証された不足に対してのみ**複雑さを追加するのが正しい順序です。

Complexity is added when a requirement demands it. Here a task a human completes in ten seconds is met with six components and five MCP servers. Multi-agent designs earn their cost when context isolation or parallelism is genuinely needed — neither applies. Measure the simplest design as a baseline and add complexity only against a demonstrated gap.

- **A 不正解**: 過剰な構成にさらに要素を足す提案で、問題を悪化させます。 / Adds to an already disproportionate design.
- **C 不正解**: 存在しない拡張要件のために階層を増やすのは投機的過剰設計です。 / Speculative complexity for a requirement that does not exist.
- **D 不正解**: リポジトリ分割は独立性を増しますが、そもそも不要なコンポーネントの運用負荷を確定させるだけです。 / Cements the operational burden of components that should not exist.

**参照 / Reference:** 適正な複雑さ、ベースライン測定、マルチエージェントの正当化条件
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

保険会社の契約管理は 30 年前のメインフレーム（COBOL、バッチ中心、日次のファイル連携）で稼働しており、当面の刷新予定はありません。ここに Claude ベースの契約内容問い合わせ機能を追加したいという要望があります。メインフレーム側のチームは変更に極めて慎重で、リアルタイム API の新設には 18 か月を要すると回答しています。問い合わせは参照のみで、更新は行いません。

An insurer's policy administration runs on a 30-year-old mainframe (COBOL, batch-oriented, daily file interchange) with no near-term replacement. A Claude-based policy-inquiry feature is requested. The mainframe team is highly change-averse and estimates 18 months to expose a new real-time API. Inquiries are read-only; no updates are performed.

**設問 / Question:**

最も適切なアプローチはどれですか？

Which approach is most appropriate?

- A) メインフレームのリアルタイム API 新設を 18 か月待つ / Wait 18 months for the real-time API
- B) メインフレームの刷新プロジェクトを立ち上げ、その中で機能を実現する / Launch a mainframe replacement program and deliver the feature within it
- C) **既存の日次ファイル連携を活用し、参照専用の読み取りレプリカを外部に構築**する。Claude はレプリカ側のツールだけを参照し、メインフレームには一切触れない。データの鮮度（最大 1 営業日遅れ）を機能仕様として明示し、鮮度が許容されない照会種別は対象外とするか有人窓口に回す / **Build a read-only replica outside the mainframe fed by the existing daily file interchange.** Claude reads only replica-backed tools and never touches the mainframe. Make the staleness bound (up to one business day) an explicit part of the feature's contract, and exclude — or route to a human desk — inquiry types where that staleness is unacceptable
- D) Claude からメインフレームの画面をエミュレートして直接操作させる / Have Claude drive the mainframe by emulating its terminal screens

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

レガシー統合では**既存の連携経路を活かし、レガシー側の変更を要求しない設計**が最も早く安全に価値を出せます。参照専用という要件がこれを可能にしており、読み取りレプリカなら整合性の問題も限定的です。重要なのは**鮮度の制約を隠さず機能仕様として明示する**ことで、これを曖昧にすると「昨日の情報で回答した」という信頼性の問題になります。鮮度が致命的な照会種別を対象外にする判断も設計の一部です。

Legacy integration delivers fastest and safest when it uses the interchange that already exists and demands no change from the legacy side, which the read-only requirement makes possible. The essential discipline is making the staleness bound an explicit part of the feature contract rather than hiding it — and scoping out the inquiry types where it is unacceptable.

- **A 不正解**: 18 か月の待機は、既存資産で実現可能な価値提供を不必要に遅らせます。 / Defers achievable value by 18 months without need.
- **B 不正解**: 一機能の追加のために基幹刷新を起こすのは規模が桁違いで、リスクも期間も比較になりません。 / Wildly disproportionate scope and risk for one feature.
- **D 不正解**: 画面エミュレーションは極めて脆く、メインフレームに予期しない負荷と操作リスクを持ち込みます。参照専用要件に対しても過剰です。 / Screen scraping is brittle and introduces unexpected load and operational risk on the system of record.

**参照 / Reference:** レガシー統合、読み取りレプリカ、鮮度制約の明示
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

旅行予約サイトで、Claude が「〇〇への 3 泊 4 日の家族旅行プラン」といった自由文の要望からプラン提案を生成します。分析すると、上位 500 パターンの要望が全体の 38% を占め、同一文面の要望が繰り返し届いています。コスト削減のため生成結果をキャッシュする案が出ました。ただしプランには航空券価格・ホテル空室状況・季節イベント情報が含まれます。

A travel site uses Claude to turn free-text requests such as "four-day family trip to X" into itinerary proposals. Analysis shows the top 500 request patterns account for 38% of traffic, with identical wording recurring. Caching generated results is proposed for cost reduction. Itineraries include airfare prices, hotel availability, and seasonal event information.

**設問 / Question:**

最も適切なキャッシュ設計はどれですか？

Which caching design is most appropriate?

- A) 生成結果を要望文をキーとして 30 日間キャッシュする / Cache the generated itinerary keyed by request text for 30 days
- B) **生成結果を鮮度の性質で分離**する。季節イベントや観光地の説明など変動の遅い部分はキャッシュし、価格・空室状況は毎回ライブで取得して合成する。キャッシュキーには要望文に加えて旅行日程や人数など結果を左右する要素を含め、TTL は各要素の変動速度に合わせる / **Separate the output by freshness**: cache slow-moving parts (seasonal events, destination descriptions) while fetching prices and availability live on every request and composing them, key the cache on the trip dates and party size as well as the request text, and set TTLs per element's rate of change
- C) キャッシュは一切使わず、常に再生成する / Never cache; always regenerate
- D) 全リクエストの結果を無期限にキャッシュする / Cache all results indefinitely

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

キャッシュ設計の要点は「**何が、どれくらいの速さで陳腐化するか**」を要素ごとに分けることです。観光地の説明は数か月変わりませんが、航空券価格と空室は分単位で変わり、古い値を表示すると予約失敗と信頼失墜に直結します。要素を分けてキャッシュすれば、38% の重複から得られるコスト削減を、鮮度が致命的な部分を犠牲にせずに取れます。**キャッシュキーに結果を左右する変数（日程・人数）を含める**ことも見落とされやすい要点です。

Cache design turns on decomposing output by how fast each part goes stale. Destination descriptions hold for months; fares and availability change by the minute, and showing stale ones causes failed bookings. Splitting by freshness captures the savings from the 38% overlap without sacrificing the parts where staleness is fatal — and the key must include every variable that changes the result.

- **A 不正解**: 30 日前の価格と空室を提示することになり、予約時点で成立しない提案を生みます。 / Presents 30-day-old fares and availability, producing unbookable itineraries.
- **C 不正解**: 38% の重複という明確な機会を捨てており、鮮度が問題にならない部分まで再生成しています。 / Discards a clear 38% opportunity, regenerating even the parts that never change.
- **D 不正解**: 無期限キャッシュは最も鮮度に敏感な情報を最も長く陳腐化させます。 / Indefinite caching maximizes staleness exactly where it hurts most.

**参照 / Reference:** キャッシュ戦略、鮮度による分離、キャッシュキー設計
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **次のドメイン / Next:** [`domain2_models_prompting_context.md`](./domain2_models_prompting_context.md)
