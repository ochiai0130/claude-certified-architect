# Domain 3: 統合アーキテクチャ / Integration

> 配点比率 / Weight: **19%**（全ドメイン中最大 / the largest domain）
> 問題数 / Questions: **30**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- MCP・ツール・直接 API の選択と接続方式の判断 / Choosing between MCP, tools, and direct APIs; transport selection
- 認証・認可・エンドユーザー権限の伝播・秘密情報の管理 / Authentication, authorization, end-user permission propagation, secrets
- ツール定義の粒度・スキーマ進化・バージョニング・非推奨化 / Tool granularity, schema evolution, versioning, deprecation
- 冪等性・部分障害・リトライ・サーキットブレーカー・サーガ / Idempotency, partial failure, retries, circuit breakers, sagas
- 同期／非同期・イベント駆動・長時間処理・ページネーション / Sync vs async, event-driven integration, long operations, pagination
- 可観測性・分散トレーシング・境界でのデータ加工 / Observability, distributed tracing, data shaping at the boundary

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

社内に 3 つのエージェントアプリケーション（サポート、営業支援、社内ヘルプデスク）があり、いずれも顧客マスタへのアクセスが必要です。現在は各アプリが独自に顧客マスタ API のクライアントコードを実装しており、認証方式・エラー処理・レート制限対応がそれぞれ異なります。顧客マスタ API に破壊的変更が入った際、3 か所を別々に修正する必要がありました。今後、外部パートナーのエージェントからのアクセス要望も見込まれています。

Three agent applications (support, sales assist, internal help desk) all need access to the customer master. Each implements its own client for the customer-master API with different authentication, error handling, and rate-limit behavior. A breaking change to that API required three separate fixes. Access requests from external partners' agents are anticipated.

**設問 / Question:**

最も適切な統合方式はどれですか？

Which integration approach is most appropriate?

- A) 3 つのアプリのクライアントコードを共通ライブラリに切り出す / Extract the three clients into a shared library
- B) **顧客マスタを MCP サーバーとして提供**し、3 つのエージェントはツールとして接続する。認証・認可・レート制限・エラー処理をサーバー側に集約し、ツール定義がインターフェース契約になる。バックエンド API の変更はサーバー内で吸収し、外部パートナーには同じツール定義を権限を絞って提供する / **Expose the customer master as an MCP server** and have the three agents connect to it as tools: authentication, authorization, rate limiting, and error handling live in the server, and the tool definitions form the interface contract. Backend API changes are absorbed inside the server, and external partners get the same tool definitions under narrower permissions
- C) 各アプリのクライアントコードをそのまま維持し、変更時の連絡体制を整える / Keep the three clients and establish a change-notification process
- D) 顧客マスタのデータを各アプリのローカル DB に複製する / Replicate the customer master into each application's local database

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

MCP サーバーが価値を持つのは、**複数のエージェントから同じ能力が必要とされ、その能力に横断的な関心事（認証・認可・レート制限・エラー処理）が伴う場合**です。共通ライブラリでも重複は減りますが、各アプリのプロセス内で実行されるため権限の集中管理ができず、外部パートナーへの提供もできません。MCP サーバーならツール定義が契約として機能し、バックエンドの変更をサーバー内に閉じ込められます。

An MCP server earns its place when several agents need the same capability and that capability carries cross-cutting concerns. A shared library removes duplication but runs inside each application, so permissions cannot be centrally enforced and external partners cannot be served. An MCP server makes the tool definition the contract and confines backend churn to one place.

- **A 不正解**: 重複は減りますが、権限の集中管理と外部提供という要件を満たせません。ライブラリ更新は各アプリの再デプロイを要します。 / Reduces duplication only; no central authorization and no external exposure.
- **C 不正解**: 連絡体制は組織的な回避策で、3 か所修正という構造的な問題を残します。 / A process workaround that leaves the structural duplication intact.
- **D 不正解**: 複製は鮮度と整合性の問題を生み、顧客マスタのような正となるデータでは特に危険です。 / Replication introduces staleness and consistency risk for a system of record.

**参照 / Reference:** MCP サーバーの適合条件、横断的関心事の集約、インターフェース契約
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

在庫管理システム用の MCP サーバーを構築します。利用者は、開発者のローカル Claude Code（12 名）、CI パイプライン上のエージェント、本番のサポートエージェント（3 サービス）です。サーバーは社内ネットワーク内の在庫 DB に接続し、認証情報を保持します。更新は月 2 回程度あり、全利用者に同時に反映したいと考えています。

You are building an MCP server for inventory management. Consumers are 12 developers' local Claude Code sessions, CI-pipeline agents, and three production support services. The server connects to an inventory database inside the corporate network and holds credentials. It is updated about twice a month and updates should reach all consumers at once.

**設問 / Question:**

最も適切な構成はどれですか？

Which configuration is most appropriate?

- A) 各利用者のマシン上で stdio 接続のローカルプロセスとして動かす / Run it as a local stdio process on each consumer's machine
- B) 開発者はローカル stdio、本番はリモート HTTP と、環境ごとに別実装にする / Use local stdio for developers and a separate remote HTTP implementation for production
- C) MCP を使わず、各利用者が在庫 DB に直接接続する / Skip MCP and have each consumer connect to the database directly
- D) **リモート HTTP でホストされる単一の MCP サーバーとして提供**する。認証情報はサーバー側にのみ置き、利用者には各自の ID で認証させて権限を分ける。更新はサーバーのデプロイで全利用者に即時反映され、監査ログも一元化される / **Host it as a single remote HTTP MCP server**: credentials live only on the server, consumers authenticate with their own identities under differentiated permissions, a server deploy reaches every consumer at once, and audit logging is centralized

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

判断材料は 3 つです。(1) **認証情報を配布したくない**（12 名のローカルマシンに DB 認証情報を置くのは統制上不可）、(2) **更新を一斉に反映したい**（ローカルプロセスは各自の更新が必要）、(3) **監査を一元化したい**。いずれもリモートホスト型を指しています。stdio ローカル接続が適するのは、利用者自身の環境のリソース（ローカルファイル、開発者個人の資格情報）を扱う場合です。

Three factors decide it: credentials must not be distributed to 12 laptops, updates must land everywhere at once, and audit must be centralized. All point to remote hosting. Local stdio is the right choice when a server operates on the consumer's own environment — local files, a developer's personal credentials.

- **A 不正解**: 12 台のローカルマシンに DB 認証情報を配布することになり、更新も各自任せになります。 / Distributes database credentials to 12 machines and makes updates per-machine.
- **B 不正解**: 2 実装は挙動の乖離を生み、開発者の環境で通ったものが本番で通らない事態を招きます。認証情報配布の問題も残ります。 / Two implementations drift, and the credential problem persists on the developer side.
- **C 不正解**: 直接接続は認可・監査・レート制限を各利用者に委ねることになり、MCP を導入する目的を失います。 / Pushes authorization, audit, and rate limiting onto every consumer.

**参照 / Reference:** MCP のトランスポート選択、リモートホスト型、認証情報の集約
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

人事システムに接続する MCP サーバーを運用しています。エージェントは単一のサービスアカウントで人事システムに接続しており、そのアカウントは全社員のレコードを読み書きできます。エンドユーザーは各部門のマネージャーで、本来は自部門の部下のレコードのみ閲覧可能です。監査で「エージェント経由なら誰でも全社員のデータを引ける」ことが指摘されました。

An MCP server fronts the HR system. The agent connects with a single service account that can read and write every employee record. End users are department managers who should only see their own reports. An audit found that "through the agent, anyone can pull any employee's data."

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **エンドユーザーの ID を人事システムまで伝播させ、そのユーザーの権限で操作を実行する**。MCP サーバーは呼び出し元ユーザーのトークンを受け取り、人事システムに対してそのユーザーとして問い合わせる。サービスアカウントの広い権限には依存せず、認可判定は人事システム側の既存の権限モデルに委ねる。監査ログには実行ユーザーを記録する / **Propagate the end user's identity through to the HR system and execute under that user's permissions**: the MCP server receives the calling user's token and queries HR as that user, relying on HR's existing authorization model rather than the service account's broad rights, and recording the acting user in the audit log
- B) システムプロンプトで「自部門の社員の情報のみ扱うこと」と指示する / Instruct the agent to handle only its own department's employees
- C) MCP サーバー内に部門判定のロジックを実装し、フィルタする / Implement department filtering logic inside the MCP server
- D) マネージャーごとに別々のサービスアカウントを発行する / Issue a separate service account per manager

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**エージェントはエンドユーザーの権限を超えてはならない**という原則が本問の核心です。ユーザー ID を伝播させて既存の権限モデルで判定させれば、認可のロジックを二重に持つ必要がなく、人事システム側の権限変更（異動、昇格）が自動的に反映されます。監査ログに実行ユーザーが残ることで、「誰の権限で何をしたか」が追跡可能になります。サービスアカウントの広い権限に依存する構成は、権限昇格の経路そのものです。

The governing principle is that an agent must never exceed the end user's own permissions. Propagating identity lets the existing authorization model decide, so permissions stay correct as people transfer or are promoted, and the audit log records who acted. A broad service account *is* the privilege-escalation path.

- **B 不正解**: プロンプトによる制限は認可の統制になりません。プロンプトを回避すれば全データにアクセスできます。 / A prompt is not an authorization control.
- **C 不正解**: 認可ロジックの二重実装は、人事システム側の権限モデルと乖離します。異動や例外的な権限付与に追随できません。 / Duplicated authorization drifts from the system of record and misses transfers and exceptions.
- **D 不正解**: アカウントを増やしても、各アカウントの権限を人事システムの権限モデルと同期させる問題が残り、運用負荷が人数に比例します。 / Multiplies accounts without solving synchronization with the real permission model.

**参照 / Reference:** エンドユーザー権限の伝播、最小権限、認可の単一の真実源
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

分析エージェント用に、データウェアハウスへのアクセスを提供する MCP サーバーを設計しています。案 1 は `run_query(sql: string)` という 1 つの汎用ツールで任意の SQL を実行させるもの、案 2 は `get_sales_by_region(region, period)`、`get_customer_ltv(customer_id)` など業務的な意味を持つ 20 個のツールを定義するものです。データウェアハウスには機微な個人情報を含むテーブルも存在します。

You are designing an MCP server exposing a data warehouse to an analytics agent. Option 1 is a single generic `run_query(sql: string)` tool; Option 2 defines 20 business-meaningful tools such as `get_sales_by_region(region, period)` and `get_customer_ltv(customer_id)`. The warehouse contains tables with sensitive personal data.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) 案 1。柔軟性が高く、新しい分析要求にツール追加なしで対応できる / Option 1: maximum flexibility, no new tools needed for new questions
- B) 案 1 に SQL の構文チェックを追加する / Option 1 plus SQL syntax validation
- C) **案 2 を基本とする**。業務的な意味を持つツールは、アクセス可能な範囲を定義上限定でき、機微テーブルへの到達を構造的に防げる。引数が型付けされるためモデルの誤用も減り、監査ログも「誰が何を照会したか」が業務語彙で残る。網羅できない探索的分析が必要なら、権限を絞った専用の経路を別途、承認を伴う形で用意する / **Option 2 as the default**: business-meaningful tools bound the reachable surface by definition, structurally preventing access to sensitive tables; typed arguments reduce misuse; and audit logs record intent in business vocabulary. Where genuinely exploratory analysis is needed, provide a separate, narrowly-permissioned path behind an approval step
- D) 案 2 の 20 ツールをすべて 1 つのツールに統合し、操作名を引数に取る / Merge Option 2's 20 tools into one taking an operation name as an argument

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

任意 SQL を実行できるツールは、**そのデータベースで可能なすべての操作を許可する**ことと同義です。機微テーブルが存在する環境では、到達可能な範囲をツール定義で限定することが最も強い統制になります。加えて、型付き引数はモデルの誤用を減らし、業務語彙の監査ログは事後分析を容易にします。柔軟性が本当に必要な場合は「例外的な経路を承認付きで用意する」という形にするのが、原則と例外の正しい分け方です。

An arbitrary-SQL tool grants everything the database can do. Where sensitive tables exist, bounding the reachable surface in the tool definition is the strongest available control, and typed arguments plus business-vocabulary audit logs are real secondary benefits. Genuine exploratory needs belong in a separate, approved path — the correct way to handle an exception without weakening the default.

- **A 不正解**: 柔軟性と引き換えに、機微テーブルを含む全データへの到達を許しています。統制不能な設計です。 / Trades away all containment, including sensitive tables.
- **B 不正解**: 構文チェックは実行可能な SQL かを見るだけで、何にアクセスしてよいかという認可の問題を扱いません。 / Syntax validation says nothing about authorization.
- **D 不正解**: 操作名を引数にすると型安全性が失われ、ツールごとの権限分離もできなくなります。案 2 の利点を捨てています。 / Discards typing and per-tool permission separation.

**参照 / Reference:** ツール粒度、最小権限、到達可能範囲の限定
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

サブスクリプション解約時の返金処理を行うエージェントで、`issue_refund(subscription_id, amount)` ツールを提供しています。ネットワークのタイムアウトでレスポンスが失われることがあり、エージェントのリトライにより同一の返金が二重に実行される事象が月に数件発生しています。返金は決済ゲートウェイに対する不可逆な操作です。

An agent issues subscription refunds through `issue_refund(subscription_id, amount)`. Network timeouts sometimes lose the response, and agent retries cause duplicate refunds a few times a month. A refund is an irreversible operation against the payment gateway.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) タイムアウトを 120 秒に延長してレスポンス欠落を減らす / Raise the timeout to 120 seconds to lose fewer responses
- B) **冪等キーを導入する**。呼び出し側（MCP サーバー）が返金要求ごとに一意なキーを生成して決済ゲートウェイに渡し、ゲートウェイ側で同一キーの重複要求を排除する。キーは業務的に同一の返金を指す情報（サブスクリプション ID ＋ 解約イベント ID）から決定的に導出し、リトライ時にも同じキーが再生成されるようにする / **Introduce an idempotency key**: the MCP server generates a unique key per refund request and passes it to the payment gateway, which deduplicates repeats of the same key. Derive the key deterministically from what identifies the refund in business terms (subscription ID + cancellation event ID) so a retry regenerates the same key
- C) エージェントにリトライさせない設定にする / Configure the agent never to retry
- D) 返金実行前に「この返金は既に実行済みですか」と確認するツールを追加する / Add a tool that asks "has this refund already been issued?" before executing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

不可逆な副作用を持つツールには**冪等性が必須**です。要点は、キーをリトライ時に**再生成できる形で決定的に導出する**ことで、ランダムな UUID をリトライごとに生成すると重複排除が機能しません。業務的に同一の操作を指す情報からキーを導けば、何度リトライしても決済ゲートウェイ側で 1 回だけ実行されます。冪等性はサーバー側（決済ゲートウェイ）で担保されて初めて保証になります。

Tools with irreversible side effects require idempotency, and the key must be *deterministically derivable* so a retry produces the same key — a fresh UUID per attempt defeats deduplication entirely. Deriving it from what identifies the operation in business terms makes any number of retries collapse to one execution, and the guarantee lives on the gateway side.

- **A 不正解**: タイムアウト延長は確率を下げるだけで、レスポンス欠落は依然として起こります。 / Lowers probability; loss still occurs.
- **C 不正解**: リトライを禁止すると、一時的な障害で返金が実行されないまま放置されます。信頼性が下がります。 / Refunds silently fail on transient errors.
- **D 不正解**: 確認と実行の間に競合が生じ得ます（Time-of-check to time-of-use）。確認ツールも呼ばれる保証がありません。 / A check-then-act race, and there is no guarantee the check is called.

**参照 / Reference:** 冪等キー、決定的なキー導出、不可逆操作の設計
</details>

---

## 問題 6 / Question 6

**シナリオ / Scenario:**

エージェントが基幹 ERP の API を呼び出します。ERP 側の制約は厳格で、「1 分あたり 60 リクエスト、超過した場合は 10 分間 IP 単位でブロック」というものです。ブロックされると ERP を使う他の社内システムも巻き込まれて業務が停止します。エージェントのトラフィックは変動が大きく、ピーク時には毎分 200 リクエスト相当の需要があります。

An agent calls a core ERP API with strict limits: 60 requests/minute, and exceeding it blocks the source IP for 10 minutes. A block also takes down other internal systems that share the ERP, halting operations. Agent traffic is bursty, with peak demand equivalent to 200 requests/minute.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) **MCP サーバー側でクライアント側レート制限（トークンバケット等）を実装し、ERP の上限を超える呼び出しを送信前にキューで待たせる**。超過分は拒否ではなく待機させ、待機が一定時間を超える場合のみエージェントにエラーを返す。ERP 側のブロックを発生させないことを設計目標とし、実際の送信レートをメトリクスで監視する / **Implement client-side rate limiting (e.g. a token bucket) in the MCP server and queue calls before they exceed the ERP's limit**: hold excess requests rather than rejecting them, returning an error to the agent only when the wait exceeds a threshold. Make "never trigger the ERP's block" the design goal and monitor the actual outbound rate
- B) ERP がブロックしたらエージェントを 10 分停止する / Pause the agent for 10 minutes whenever the ERP blocks
- C) ERP からブロックされたら別の IP から再接続する / Reconnect from a different IP when blocked
- D) ERP チームにレート上限の引き上げを依頼し、それまでは現状維持する / Ask the ERP team to raise the limit and keep the current behavior meanwhile

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**ブロックの影響が自分たちだけに閉じない**（他システムも巻き込む）ため、「起きてから対処する」設計は許容されません。上限が既知なのだから、その上限を超えないことを送信側で保証するのが正しい設計です。クライアント側レート制限は拒否ではなくキューイングを基本とし、バーストを時間方向に均します。実際の送信レートを監視することで、設定が正しく機能しているかを継続的に確認できます。

Because the block harms other systems too, a react-after-the-fact design is unacceptable. The limit is known, so the sender must guarantee it is never exceeded. Client-side limiting should queue rather than reject, smoothing bursts over time, with the actual outbound rate monitored to confirm the control works.

- **B 不正解**: 事後対応であり、ブロックは既に発生して他システムに影響しています。 / Reactive; the damage to other systems has already occurred.
- **C 不正解**: レート制限の回避行為であり、ERP チームとの信頼関係と統制を損ないます。技術的にも他システムへの影響は変わりません。 / Circumventing a control; harms trust and does not protect the shared system.
- **D 不正解**: 依頼中も現状の挙動が続き、ブロックが発生し続けます。上限が上がっても制御機構がなければ再発します。 / The blocks continue meanwhile, and recur at any limit without a control.

**参照 / Reference:** クライアント側レート制限、共有リソースの保護、バーストの平準化
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

顧客の注文ステータスが変化したとき（発送済み、遅延、キャンセル）に、エージェントが顧客へ通知を送る必要があります。現在は 30 秒ごとに注文システムの API をポーリングして変化を検出しています。注文は 1 日 8 万件あり、変化するのはそのうち約 12% です。ポーリングは注文システムに常時負荷をかけており、また最大 30 秒の検出遅延があります。注文システムは Webhook 送出に対応しています。

When an order's status changes (shipped, delayed, cancelled), an agent must notify the customer. Today the order system's API is polled every 30 seconds to detect changes. There are 80,000 orders/day, of which about 12% change status. Polling loads the order system continuously and adds up to 30 seconds of detection delay. The order system supports webhooks.

**設問 / Question:**

最も適切なアーキテクチャはどれですか？

Which architecture is most appropriate?

- A) ポーリング間隔を 5 秒に短縮して検出遅延を減らす / Poll every 5 seconds to reduce detection delay
- B) ポーリングを維持しつつ、変化しやすい注文だけ高頻度にする / Keep polling but poll change-prone orders more frequently
- C) ポーリングを 5 分間隔にして負荷を下げる / Poll every 5 minutes to reduce load
- D) **Webhook によるイベント駆動に切り替える**。注文システムからのステータス変更イベントを受信して処理し、無駄なポーリングをなくす。受信エンドポイントは署名検証で真正性を確認し、イベントには重複配信があり得るためイベント ID による冪等処理を行う。Webhook の欠落に備え、低頻度（例: 1 時間ごと）の差分照合をバックストップとして残す / **Move to webhook-driven events**: consume status-change events instead of polling, verify authenticity by signature on the receiving endpoint, deduplicate by event ID since delivery may repeat, and retain a low-frequency reconciliation sweep (say hourly) as a backstop against missed webhooks

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

変化率が 12% ということは、**ポーリングの 88% は無駄**です。イベント駆動にすれば負荷が実際の変化量に比例し、検出遅延もほぼゼロになります。実装上の要点は 3 つで、**署名検証**（偽イベントの防止）、**イベント ID による冪等処理**（重複配信は正常な挙動）、**低頻度の差分照合**（Webhook は配信保証が完全でないため、取りこぼしを検出するバックストップ）です。イベント駆動に切り替えても照合を完全に捨てないのが実務的な設計です。

A 12% change rate means 88% of polls are wasted. Events make load proportional to real change and remove detection delay. Three implementation points matter: signature verification, idempotent handling keyed on event ID (repeat delivery is normal), and a low-frequency reconciliation sweep, since webhook delivery is not perfectly guaranteed.

- **A 不正解**: 無駄なポーリングを 6 倍に増やし、注文システムの負荷を大幅に悪化させます。 / Multiplies wasted polling sixfold.
- **B 不正解**: 変化しやすさの予測は不確実で、複雑さの割に効果が限定的です。根本の無駄は残ります。 / Speculative and complex, with the fundamental waste intact.
- **C 不正解**: 負荷は下がりますが検出遅延が 5 分に伸び、通知の適時性が損なわれます。 / Trades the requirement (timeliness) for the load reduction.

**参照 / Reference:** イベント駆動統合、Webhook の署名検証、重複配信への冪等処理、差分照合
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

MCP サーバーのツールがエラーを返す際、現在はすべて `{"error": "request failed"}` という同一のメッセージを返しています。エージェントはこのエラーを受け取ると、同じ引数で無限にリトライするか、あるいは無関係な別のツールを試す挙動を示します。実際のエラー原因は、認可不足、引数の型不正、対象レコードの不存在、下流のタイムアウトなど多岐にわたります。

An MCP server's tools currently return the same `{"error": "request failed"}` for every failure. On receiving it, the agent either retries indefinitely with identical arguments or tries unrelated tools. The underlying causes vary: insufficient authorization, malformed arguments, missing records, downstream timeouts.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) エラー時はツールの実行を無言で成功として返し、空の結果を渡す / Return success with an empty result instead of an error
- B) エラーメッセージに下流システムのスタックトレースをそのまま含める / Include the downstream system's raw stack trace in the error
- C) **エラーを種別ごとに区別し、エージェントが次の行動を選べる情報を返す**。「引数 `date` の形式が不正（期待: YYYY-MM-DD）」のように**修正可能な情報**を含め、リトライして意味があるか（一時的障害か恒久的な失敗か）を明示する。認可不足では権限不足である旨を返し、リトライさせない。内部実装の詳細は漏らさない / **Differentiate errors by type and return information the agent can act on**: include correctable detail such as "argument `date` has an invalid format (expected YYYY-MM-DD)", state explicitly whether retrying can help (transient vs permanent), and for authorization failures say so and signal not to retry — without leaking internal implementation detail
- D) エラー時は例外を送出してエージェントのセッションを終了させる / Raise an exception that terminates the agent session

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**ツールのエラーメッセージはモデルへの入力であり、次の行動を決める材料**です。区別のないエラーは、モデルに何をすべきか判断させる情報を与えないため、無限リトライや無関係なツールの試行という観察された挙動を招きます。修正可能な情報（期待される形式）を返せばモデルは自己修正でき、リトライの可否を明示すれば無駄な再試行を防げます。一方で、内部実装の詳細を漏らさないことはセキュリティ上の要件です。

A tool's error message is model input that determines the next action. An undifferentiated error gives the model nothing to decide with, which produces exactly the observed behavior. Correctable detail lets the model self-correct, and an explicit retryability signal stops futile retries — while withholding internal detail remains a security requirement.

- **A 不正解**: 失敗を成功として返すのは最も危険な設計で、エージェントは誤った前提で処理を進めます。 / Silently converting failure to success is the worst option.
- **B 不正解**: スタックトレースは内部構造を露出させるセキュリティリスクであり、モデルにとっても行動を決める情報になりません。 / Leaks internals without providing actionable guidance.
- **D 不正解**: セッション終了は回復可能なエラー（引数の形式ミス）に対して過剰で、自己修正の機会を奪います。 / Disproportionate for recoverable errors; removes the chance to self-correct.

**参照 / Reference:** ツールエラーの設計、行動可能なエラー情報、リトライ可否の明示
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

社内 MCP サーバーの `search_documents` ツールに、検索対象を絞る `department` 引数を追加したいと考えています。このツールは 6 つのエージェントアプリケーションから利用されており、うち 2 つは他部門が管理していて、変更の調整に時間がかかります。既存の呼び出しはすべて `query` 引数のみを渡しています。

You want to add a `department` argument to an internal MCP server's `search_documents` tool to narrow the search. Six agent applications use it; two are owned by other departments and coordination is slow. All existing calls pass only `query`.

**設問 / Question:**

最も適切な変更方法はどれですか？

Which change approach is most appropriate?

- A) `department` を必須引数として追加し、6 つのアプリに同時に対応を依頼する / Add `department` as a required argument and ask all six applications to adapt simultaneously
- B) **`department` を省略可能な引数として追加する**。省略時は従来どおり全部門を検索する後方互換の挙動とし、既存の 6 アプリは変更なしで動き続ける。ツール説明には省略時の挙動を明記し、絞り込みが有効なアプリから順次採用してもらう / **Add `department` as an optional argument** with a backward-compatible default (search all departments when omitted) so all six applications keep working unchanged, document the default behavior in the tool description, and let applications adopt it as it benefits them
- C) `search_documents_v2` という別のツールを新設し、旧ツールは即座に削除する / Add a `search_documents_v2` tool and remove the old one immediately
- D) 引数を追加せず、`query` 文字列の中に部門名を書く規約にする / Skip the argument and adopt a convention of writing the department into the `query` string

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**後方互換な追加（省略可能な引数）は、既存利用者に一切の変更を強いない**ため、調整コストがゼロで済みます。スキーマ進化の基本原則は「追加は省略可能に、削除と必須化は破壊的変更として扱う」です。省略時の挙動をツール説明に明記することが重要で、これがないとモデルが引数を渡すべきか判断できません。組織的な調整が難しい環境では、この原則の価値がとりわけ大きくなります。

Backward-compatible addition imposes zero change on existing consumers, so coordination cost disappears. The rule of schema evolution is: additions are optional; removals and new required fields are breaking. Documenting the default matters — without it, the model cannot judge whether to supply the argument.

- **A 不正解**: 必須化は破壊的変更で、6 アプリすべてを同時に壊します。調整が難しい環境では特に不適切です。 / A breaking change that stops all six at once.
- **C 不正解**: 旧ツールの即時削除は移行期間を与えず、破壊的です。バージョン分けするなら移行期間が必要です。 / Immediate removal gives no migration window.
- **D 不正解**: 構造化されるべき情報を自由文字列に埋め込む設計で、解釈が曖昧になり検証もできません。 / Encodes structured data in free text, defeating validation.

**参照 / Reference:** スキーマ進化、後方互換な追加、破壊的変更の定義
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

MCP サーバーに `generate_financial_report` ツールを実装しました。このツールは会計システムから 1 年分のデータを集計してレポートを生成し、完了までに 4〜11 分かかります。現在は同期的に処理しており、MCP のリクエストがタイムアウトして失敗します。まれに成功しても、その間エージェントは他の作業ができません。

An MCP server implements `generate_financial_report`, which aggregates a year of accounting data and takes 4–11 minutes. It is implemented synchronously and the MCP request times out. On the rare occasions it succeeds, the agent can do nothing else meanwhile.

**設問 / Question:**

最も適切なツール設計はどれですか？

Which tool design is most appropriate?

- A) タイムアウト値を 15 分に設定する / Set the timeout to 15 minutes
- B) レポートの集計範囲を 1 か月に縮小して処理時間を短くする / Reduce the aggregation window to one month to shorten processing
- C) **ツールを非同期の 2 段構成にする**。`start_financial_report` がジョブを登録してジョブ ID を即座に返し、`get_report_status(job_id)` が進行状況または完了した成果物を返す。エージェントは待機中に他の作業を進められ、長時間処理がリクエストのタイムアウトから切り離される。ジョブは永続化し、サーバー再起動をまたいで生存させる / **Split the tool into an asynchronous pair**: `start_financial_report` registers a job and returns a job ID immediately, and `get_report_status(job_id)` returns progress or the finished artifact. The agent can work on other things while waiting, the long operation is decoupled from request timeouts, and jobs are persisted so they survive a server restart
- D) レポート生成を毎晩バッチで全件事前生成しておく / Pre-generate every possible report in a nightly batch

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**長時間処理を同期ツールに載せてはいけません。**開始と状態取得の 2 段構成にすることで、リクエストのタイムアウトから処理を切り離し、エージェントに待機中の自由度を与えられます。ジョブの永続化が重要で、これがないとサーバー再起動で進行中のレポートが失われます。この「start / poll」パターンは、長時間の外部処理を MCP ツールとして提供する際の標準形です。

Long operations do not belong in a synchronous tool. A start/poll pair decouples the work from request timeouts and frees the agent while it runs. Persisting the job matters: without it, a server restart loses in-flight reports. This is the standard shape for exposing long-running work through MCP.

- **A 不正解**: 長いタイムアウトは接続を占有し、11 分を超えるケースでは依然失敗します。エージェントの待機問題も残ります。 / Ties up a connection, still fails past 11 minutes, and blocks the agent.
- **B 不正解**: 集計範囲の縮小は要件（年次レポート）の否定です。機能を削って問題を消しています。 / Removes the requirement rather than meeting it.
- **D 不正解**: パラメータの組み合わせが多く事前生成は現実的でなく、鮮度の問題も生じます。 / Combinatorially infeasible and introduces staleness.

**参照 / Reference:** 長時間処理のツール設計、start / poll パターン、ジョブの永続化
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

開発チームが、コミュニティで公開されている MCP サーバー（プロジェクト管理ツール連携）を本番のエージェントに組み込みたいと提案しています。このサーバーは npm パッケージとして配布され、実行時に本番のプロジェクト管理ツールの API トークンを渡す必要があります。サーバーはネットワークアクセスとファイルシステムアクセスを持ちます。メンテナは個人で、直近の更新は 4 か月前です。

A team proposes adopting a community MCP server (project-management integration) in production. It ships as an npm package and requires the production project-management API token at runtime. The server has network and filesystem access. It is maintained by an individual and was last updated four months ago.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **サードパーティ依存としてサプライチェーンのリスク評価を行う**。ソースを読んで実際に必要な権限を確認し、バージョンを固定して自社のレジストリにミラーする。実行はネットワーク到達先とファイルシステムを制限したサンドボックスで行い、渡すトークンは必要最小限のスコープに絞った専用トークンにする。更新は差分をレビューしてから取り込む / **Treat it as a supply-chain risk and assess it**: read the source to confirm the permissions actually required, pin the version and mirror it into your own registry, run it in a sandbox restricting network destinations and filesystem access, and supply a dedicated token scoped to the minimum required — reviewing diffs before adopting updates
- B) 公開されているので安全とみなし、そのまま本番に導入する / Treat public availability as sufficient assurance and adopt it as-is
- C) 同等の機能を全て自前で再実装する / Reimplement the equivalent functionality entirely in-house
- D) 本番では使わず、開発環境でのみ使用する / Use it only in development, never in production

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

サードパーティ MCP サーバーの導入は**サプライチェーンリスクの問題**です。本番トークンを渡し、ネットワークとファイルシステムにアクセスするコンポーネントは、依存ライブラリと同等以上の審査が必要です。対策は多層で、ソースレビュー（何をするか確認）、バージョン固定とミラー（更新による予期しない変更を防ぐ）、サンドボックス（万一の場合の影響限定）、最小スコープのトークン（漏洩時の被害限定）を組み合わせます。**採用するかしないかの二択ではなく、リスクを管理して採用する**のが実務的な答えです。

Adopting a third-party MCP server is a supply-chain decision. A component that receives a production token and holds network and filesystem access warrants at least the scrutiny of any dependency. The controls layer: source review, version pinning and mirroring, sandboxing to bound the impact, and a minimum-scope token to bound a leak. The realistic answer is managed adoption, not a binary accept/reject.

- **B 不正解**: 公開されていることは安全性の根拠になりません。本番トークンを渡す相手として審査が不足しています。 / Public availability is not assurance, least of all for a component holding a production token.
- **C 不正解**: 再実装は選択肢の 1 つですが、リスク評価もせずに自動的に選ぶべきものではなく、コストと保守負担が大きくなります。 / A valid option, but not the automatic one; it carries real cost.
- **D 不正解**: 開発環境でも認証情報や社内データに触れるため、無審査で使ってよいわけではありません。 / Development environments also hold credentials and internal data.

**参照 / Reference:** サプライチェーンリスク、サードパーティ MCP サーバー、サンドボックス、最小スコープの資格情報
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

開発支援エージェントに、リポジトリ内のファイルを編集する `write_file(path, content)` ツールを提供します。エージェントは自律的に動作し、人間は結果のみを確認します。運用テストで、エージェントが意図せずリポジトリ外のパス（`../../etc/`、`~/.ssh/`）や、CI 設定ファイル、依存関係のロックファイルを書き換えようとする挙動が観測されました。

A development-assist agent is given `write_file(path, content)` to edit repository files. It operates autonomously with humans reviewing only the results. In testing, it attempted to write outside the repository (`../../etc/`, `~/.ssh/`) and to modify CI configuration and dependency lockfiles.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) システムプロンプトに「リポジトリ外のファイルを編集しないこと」と明記する / State in the system prompt that files outside the repository must not be edited
- B) 編集後に差分を確認し、問題があれば戻す / Review the diff afterwards and revert if problematic
- C) エージェントに `write_file` を与えず、提案のみを出力させる / Withhold `write_file` and have the agent output suggestions only
- D) **ツールの実装側でパスを検証し、許可された範囲外への書き込みを拒否する**。リポジトリルートを基準に正規化したうえで境界外を拒否し（相対パスやシンボリックリンクによる脱出も塞ぐ）、CI 設定やロックファイルなど影響の大きいパスは別途拒否リストに載せる。実行自体をリポジトリのコピーに閉じたサンドボックスで行い、結果をレビュー経由で反映する / **Validate the path inside the tool implementation and refuse writes outside the permitted scope**: canonicalize against the repository root and reject anything beyond it (closing relative-path and symlink escapes), maintain a deny list for high-impact paths such as CI configuration and lockfiles, and run in a sandbox confined to a copy of the repository, landing results through review

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**ツールが実行できる範囲は、ツールの実装で決定的に制限します。**パスの正規化を伴う検証が必須で、これがないと `../` やシンボリックリンクで境界を越えられます。影響の大きいファイル（CI 設定は実行内容を変え、ロックファイルは依存関係を変える）は、リポジトリ内であっても別扱いが妥当です。サンドボックス実行を組み合わせると、検証をすり抜けた場合の影響も限定されます。多層防御の構成です。

What a tool can reach is bounded by the tool's implementation. Path validation must canonicalize, or `../` and symlinks escape the boundary. High-impact files deserve separate treatment even inside the repository — CI config changes what executes, lockfiles change what is installed. Sandboxed execution bounds whatever slips past validation.

- **A 不正解**: プロンプトによる制限は確率的で、ファイルシステムへの書き込みという不可逆な操作の統制になりません。 / Probabilistic control over an irreversible filesystem operation.
- **B 不正解**: 事後確認では、既に `~/.ssh/` が書き換えられている可能性があります。取り返しのつかない変更もあります。 / By review time the write has already happened.
- **C 不正解**: 書き込み能力を奪うのは安全ですが、エージェントの用途（開発支援の自動化）を大きく損ないます。範囲を制限すれば両立できます。 / Safe but forfeits the purpose; scoping achieves both.

**参照 / Reference:** ツール実装での境界検証、パス正規化、サンドボックス、多層防御
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

`list_transactions(account_id, from_date, to_date)` ツールが、指定期間の全取引を 1 つの配列で返します。大口顧客の 1 年分を照会すると 4 万件が返り、コンテキストを埋め尽くしてエージェントが機能不全になります。エージェントの実際の用途は「特定の取引を探す」「合計金額を知る」「異常な取引を見つける」の 3 つです。

`list_transactions(account_id, from_date, to_date)` returns every transaction in the period as one array. A year for a large customer returns 40,000 records, filling the context and leaving the agent unable to function. The agent's actual uses are: find a specific transaction, know the total, and spot anomalies.

**設問 / Question:**

最も適切なツール設計はどれですか？

Which tool design is most appropriate?

- A) 返却件数を先頭 100 件に固定で切り詰める / Truncate to the first 100 records
- B) **用途に合わせてツールを分割し、ページネーションを導入する**。合計は `get_transaction_summary`（件数・合計・期間別内訳を集計値で返す）、検索は絞り込み条件を引数に持つ `search_transactions`、一覧は `list_transactions(..., cursor, limit)` としてカーソルベースのページネーションを提供し、総件数と次カーソルを返す。大量データを生のまま返す設計を避ける / **Split the tools by use case and paginate**: `get_transaction_summary` returns counts, totals, and period breakdowns as aggregates; `search_transactions` takes filter arguments; and `list_transactions(..., cursor, limit)` provides cursor-based pagination returning the total count and next cursor — so raw bulk data is never dumped into context
- C) 返却データを Claude に要約させてから返す / Have Claude summarize the results before returning them
- D) 照会可能な期間を 1 週間に制限する / Limit queries to one week

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**ツールは、モデルが実際に必要とする形でデータを返すべき**です。「合計を知りたい」だけなら 4 万件は不要で、集計値を返すツールがあれば 1 行で済みます。用途に応じたツール分割は、コンテキスト消費を劇的に減らし、レイテンシとコストも改善します。一覧が必要な場合のためのカーソルベースのページネーションは、総件数を併せて返すことでモデルが「まだ続きがある」ことを認識できるようにします。

Tools should return data in the shape the model actually needs. "What is the total?" does not require 40,000 records — an aggregate tool answers it in one line. Splitting by use case cuts context, latency, and cost together, and cursor pagination with a total count lets the model know more remains.

- **A 不正解**: 固定の切り詰めは、残り 39,900 件の存在をモデルに伝えないため、誤った結論（「取引は 100 件しかない」）を招きます。 / Silent truncation invites the false conclusion that only 100 transactions exist.
- **C 不正解**: 要約のために 4 万件をモデルに渡す必要があり、問題が解決しません。集計はコードで行うべきです。 / Still requires passing 40,000 records; aggregation belongs in code.
- **D 不正解**: 期間制限は正当な照会要件（年次の合計）を満たせなくします。 / Blocks legitimate annual queries.

**参照 / Reference:** ツールの用途別分割、カーソルページネーション、集計値の返却
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

15 年前に構築された SOAP ベースの物流システムと統合します。WSDL は提供されていますが、レスポンスは深くネストした XML で、エラーは HTTP 200 のままボディ内のステータスコードで表現され、フィールド名は `FLD_TRK_STS_CD` のような略号です。物流システム側の変更は不可能です。エージェントには「荷物の追跡状況を調べる」機能を提供したいと考えています。

You must integrate a 15-year-old SOAP logistics system. A WSDL exists, but responses are deeply nested XML, errors return HTTP 200 with a status code in the body, and field names are abbreviations such as `FLD_TRK_STS_CD`. The logistics system cannot be modified. The agent needs a "check shipment tracking status" capability.

**設問 / Question:**

最も適切な統合方式はどれですか？

Which integration approach is most appropriate?

- A) SOAP のレスポンス XML をそのままツールの戻り値としてエージェントに渡す / Return the raw SOAP XML as the tool's result
- B) エージェントに SOAP リクエストの XML を組み立てさせて送信させる / Have the agent compose and send the SOAP request XML itself
- C) **MCP サーバーをアンチコラプションレイヤーとして実装する**。SOAP の呼び出し、XML のパース、ボディ内ステータスコードの解釈（エラーは MCP のエラーとして返す）、略号フィールドの意味のある名前への変換をサーバー内で行い、エージェントには `get_shipment_status(tracking_number)` という業務語彙のツールと、平坦で意味の明確な構造化レスポンスだけを見せる / **Implement the MCP server as an anti-corruption layer**: perform the SOAP call, parse the XML, interpret the in-body status code (surfacing failures as MCP errors), and rename abbreviated fields to meaningful ones inside the server — exposing only a business-vocabulary tool `get_shipment_status(tracking_number)` returning a flat, clearly named structure
- D) 物流システムのデータを毎晩エクスポートして、モダンな API で提供し直す / Export the logistics data nightly and re-serve it through a modern API

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

レガシー統合の要点は、**レガシー側の複雑さを境界の内側に封じ込める**ことです。MCP サーバーがアンチコラプションレイヤーとして機能し、プロトコル（SOAP）・データ形式（ネスト XML）・エラー表現（ボディ内コード）・語彙（略号）の 4 つの不整合をすべて吸収します。エージェントに見えるのは業務語彙のツールと明確な構造だけになり、モデルが略号の意味やエラー判定の慣習を推測する必要がなくなります。

Legacy integration is about confining legacy complexity inside a boundary. The MCP server acts as an anti-corruption layer absorbing four separate mismatches — protocol, data shape, error convention, and vocabulary — so the agent sees only business vocabulary and a clear structure, with nothing left to infer.

- **A 不正解**: 生の XML はコンテキストを浪費し、モデルが略号とエラー慣習を推測することになり誤りを招きます。 / Wastes context and forces the model to guess abbreviations and error conventions.
- **B 不正解**: モデルに XML を組み立てさせるのは脆弱で、SOAP の厳格な形式要件を確率的な生成に委ねることになります。 / Delegates strict formatting to probabilistic generation.
- **D 不正解**: 追跡状況は鮮度が本質的な情報で、夜間バッチでは用途を満たしません。 / Tracking status is inherently time-sensitive; nightly data does not serve it.

**参照 / Reference:** アンチコラプションレイヤー、レガシー統合、境界でのデータ整形
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

出張申請エージェントが 1 ターンで 3 つのツールを呼びます: `reserve_flight`、`reserve_hotel`、`register_expense`。ある実行で、航空券とホテルの予約は成功しましたが、経費登録が下流システムの障害で失敗しました。エージェントは「出張手配が完了しました」と応答し、経費登録が漏れたまま処理が終了しました。3 つのシステムは別々で、分散トランザクションは利用できません。

A travel-request agent calls three tools in one turn: `reserve_flight`, `reserve_hotel`, `register_expense`. In one run, the flight and hotel succeeded but expense registration failed due to a downstream outage. The agent responded "travel arrangements complete" and finished with the expense record missing. The three systems are separate and no distributed transaction is available.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) **部分的な成功を明示的に扱う設計にする**。3 つの操作を 1 つの業務トランザクションとして識別子で束ね、各操作の結果を状態として永続化する。失敗した操作は補償（予約の取り消し）または再試行のいずれかへ確実に回し、未完了のまま完了応答を返さない。どの操作が成功し、どれが未完了かをエージェントの応答と業務記録の両方に反映する / **Handle partial success explicitly**: bind the three operations into one business transaction identified by a key, persist each operation's outcome as state, and route any failure deterministically to either compensation (cancel the reservations) or retry — never returning a completion response while an operation is outstanding, and reflecting which steps succeeded in both the agent's answer and the business record
- B) 3 つのツールを 1 つのツールに統合して、内部で順に実行する / Merge the three tools into one that executes them in sequence internally
- C) 経費登録を省略可能な処理として扱い、失敗しても無視する / Treat expense registration as optional and ignore its failure
- D) エージェントに「すべてのツールが成功したか確認せよ」と指示する / Instruct the agent to verify that all tools succeeded

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

分散トランザクションが使えない環境での複数システム更新は、**サーガ（補償トランザクション）パターン**で扱います。要点は、業務トランザクションとしての識別、各操作結果の永続化、失敗時の補償または再試行への確実な誘導、そして**未完了状態で完了と応答しないこと**です。部分的な成功は「起こり得る正常な状態」として設計に織り込む必要があり、例外として扱うと本問のような静かな不整合が残ります。

Without distributed transactions, multi-system updates use the saga pattern: identify the business transaction, persist each step's outcome, route failures deterministically to compensation or retry, and never report completion while a step is outstanding. Partial success is a normal state to design for; treating it as an exception is what leaves the silent inconsistency seen here.

- **B 不正解**: 統合しても 3 つのシステムへの更新が原子的になるわけではなく、部分的失敗の問題は内部に移るだけです。 / Merging does not make three system updates atomic; the problem moves inside.
- **C 不正解**: 経費登録の欠落は精算漏れと会計上の不整合につながり、無視できる失敗ではありません。 / A missing expense record is an accounting inconsistency, not an ignorable failure.
- **D 不正解**: モデルへの確認指示は確率的で、状態の永続化も補償もありません。失敗時に何をするかが未定義のままです。 / Probabilistic, with no persisted state and no defined recovery.

**参照 / Reference:** サーガパターン、補償トランザクション、部分的失敗の設計
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

CRM 連携の `get_customer(customer_id)` ツールが、CRM API のレスポンスをそのまま返しています。レスポンスには 140 のフィールドが含まれ、その多くは内部管理用（`sfdc_record_type_id`、`last_sync_batch_no`、監査用タイムスタンプ 12 種）です。エージェントが実際に使うのは、氏名・契約プラン・契約日・担当営業・未解決チケット数の 5 項目です。1 回の呼び出しで約 3,800 トークンを消費しています。

A CRM-integration tool `get_customer(customer_id)` returns the CRM API's response verbatim: 140 fields, most of them internal (`sfdc_record_type_id`, `last_sync_batch_no`, 12 audit timestamps). The agent actually uses five: name, plan, contract date, account owner, open ticket count. Each call consumes about 3,800 tokens.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) レスポンスをそのまま返し、エージェントに必要な項目だけ読ませる / Keep returning everything and let the agent read only what it needs
- B) レスポンスを Claude に要約させてから返す / Summarize the response with Claude before returning it
- C) フィールド名を短縮してトークン数を減らす / Shorten the field names to reduce tokens
- D) **境界でレスポンスを整形し、用途に必要なフィールドだけを返す**。ツールのスキーマで返却フィールドを明示的に定義し、内部管理用フィールドは公開しない。追加のフィールドが必要になった時点でスキーマに加える。必要ならフィールド選択を引数で指定できるようにするが、既定値は最小セットにする / **Shape the response at the boundary and return only the fields the use case needs**: declare the returned fields explicitly in the tool schema, do not expose internal ones, and add fields to the schema when a genuine need appears. Optionally allow field selection by argument, but keep the default at the minimal set

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**バックエンドのレスポンス形状を、そのままモデルへのインターフェースにしてはいけません。**不要な 135 フィールドは、トークンコストを浪費するだけでなく、モデルが誤ったフィールドを参照するリスクを生み、内部実装の詳細を露出させます。境界でフィールドを絞ることは、コスト・精度・情報開示の 3 つを同時に改善します。ツールスキーマで返却形状を明示することが、後方互換性の管理にもつながります。

A backend's response shape should not become the model's interface. The 135 unused fields waste tokens, invite the model to reference the wrong field, and expose internal implementation. Shaping at the boundary improves cost, accuracy, and disclosure simultaneously, and declaring the returned shape in the schema is what makes compatibility manageable later.

- **A 不正解**: 「必要な項目だけ読む」ためにも全 140 フィールドがコンテキストに入るため、コストは削減されません。 / All 140 fields still enter context, so nothing is saved.
- **B 不正解**: 要約のために追加の呼び出しコストとレイテンシが発生し、決定的に選べるものを確率的手段で選んでいます。 / Adds a call to do probabilistically what a schema does deterministically.
- **C 不正解**: 短縮は微小な削減にとどまり、フィールド名の可読性が下がってモデルの解釈精度を損ないます。 / Marginal savings at the cost of interpretability.

**参照 / Reference:** 境界でのデータ整形、ツールスキーマによる返却形状の定義、情報の最小化
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

社内 MCP サーバーで、次の 2 種類の機能を提供したいと考えています。(1) 全社共通の「業務マニュアル」「用語集」「組織図」— 内容は静的で、エージェントが文脈として参照する。(2) 「経費申請の登録」「休暇申請の承認」— 副作用を伴う操作。現在は両方をツールとして実装しています。

An internal MCP server needs to provide two kinds of capability: (1) company-wide reference material — operations manual, glossary, org chart — which is static and consulted as context; and (2) actions with side effects such as filing an expense claim and approving leave. Both are currently implemented as tools.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) 両方ともツールとして実装し続ける / Keep implementing both as tools
- B) **(1) は MCP リソースとして、(2) はツールとして提供する**。リソースは参照可能なコンテンツとしてクライアントが取得・提示でき、副作用がないことが型として表現される。ツールはモデルが実行を判断する副作用を伴う操作に限定され、承認や監査の対象を明確にできる。両者を分けることで、権限設計とレビュー範囲が整理される / **Expose (1) as MCP resources and (2) as tools**: resources are retrievable content the client can surface, with the absence of side effects expressed by the type, while tools are reserved for side-effecting operations the model chooses to invoke — which clarifies what needs approval and audit, and cleans up permission design
- C) (1) を大量のツール（`get_manual_section_1` など）に分割する / Split (1) into many tools such as `get_manual_section_1`
- D) (2) もリソースとして実装し、統一する / Implement (2) as resources too, for uniformity

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

MCP のリソースとツールは、**副作用の有無という本質的な違い**で使い分けます。リソースは「参照されるコンテンツ」で、クライアントが取得して文脈に含めることを想定します。ツールは「モデルが実行を選ぶ操作」で、副作用を伴い得ます。この区別を守ると、承認ゲートや監査の対象がツールに限定され、権限設計が明快になります。すべてをツールにすると、副作用のない参照まで実行判断とレビューの対象になり、統制が薄まります。

MCP resources and tools differ on side effects. Resources are content a client retrieves and surfaces; tools are operations the model elects to invoke, potentially with effects. Honoring the distinction confines approval gates and audit to tools and makes permissions legible — whereas making everything a tool drags side-effect-free reads into the same review surface and dilutes the control.

- **A 不正解**: 副作用のない参照と副作用のある操作が同じ扱いになり、権限設計と監査範囲が不必要に広がります。 / Conflates reads with effects, widening audit scope needlessly.
- **C 不正解**: 静的コンテンツを大量のツールに分割するとツール一覧が肥大化し、ツール選択の精度が落ちます。 / Bloats the tool list and degrades tool selection.
- **D 不正解**: 副作用のある操作をリソースとして表現するのは意味論に反し、実行の意図と承認の所在が不明確になります。 / Misrepresents side-effecting operations and obscures where approval belongs.

**参照 / Reference:** MCP リソースとツールの使い分け、副作用の有無による分類
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

MCP サーバーをマネージドなコンテナ基盤にデプロイしました。このサーバーは社内ネットワーク上のデータベース（プライベート IP）と、外部 SaaS の API（インターネット経由）の両方にアクセスする必要があります。セキュリティポリシーでは「社内 DB への到達経路はインターネットを経由しないこと」「外部通信は許可された宛先のみ」が定められています。現在は全通信がパブリックなインターネットゲートウェイ経由で、社内 DB へは VPN を張って到達しています。

An MCP server is deployed on a managed container platform. It needs both an internal database on a private IP and an external SaaS API over the internet. Security policy requires that paths to the internal database not traverse the internet and that outbound traffic reach only approved destinations. Today all traffic exits through a public internet gateway, with a VPN used to reach the internal database.

**設問 / Question:**

最も適切なネットワーク構成はどれですか？

Which network configuration is most appropriate?

- A) 全通信をインターネット経由にし、DB 側で IP 制限をかける / Send everything over the internet and restrict by source IP at the database
- B) MCP サーバーを 2 つに分け、社内用と外部用で別々にデプロイする / Split into two MCP servers, one internal and one external
- C) **社内 DB へはプライベートな接続経路（プライベートネットワーク接続・プライベートエンドポイント）で到達させ、外部 SaaS への通信は許可宛先を限定した egress 制御を通す**。2 系統の経路をネットワーク層で分離し、意図しない宛先への通信は既定で遮断する。到達先の一覧を構成として管理し、変更をレビュー対象にする / **Reach the internal database over private connectivity (private networking / private endpoints) and route external SaaS traffic through egress control restricted to approved destinations**: separate the two paths at the network layer, deny unintended destinations by default, and manage the destination allowlist as reviewed configuration
- D) 社内 DB のデータを外部 SaaS 側に複製し、外部通信だけにする / Replicate the internal database into the external SaaS so only external traffic remains

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

要件は 2 つで、**社内到達経路のインターネット非経由**と**外部宛先の限定**です。前者はプライベート接続で、後者は egress 制御で満たします。両者はネットワーク層の異なる仕組みなので、片方だけでは要件を満たしません。既定で遮断し、許可宛先を構成として管理してレビュー対象にすることで、後から追加される通信も統制下に置けます。MCP サーバーは社内資産に到達する経路そのものなので、ネットワーク境界の設計は統合設計の一部です。

There are two requirements: private paths inward and restricted destinations outward, satisfied by private connectivity and egress control respectively — different mechanisms, so neither alone suffices. Denying by default and managing the allowlist as reviewed configuration keeps later additions under control. Because the MCP server *is* the path to internal assets, its network boundary is part of the integration design.

- **A 不正解**: 社内 DB への到達がインターネット経由になり、ポリシー違反です。IP 制限は経路の要件を満たしません。 / Violates the policy on internal paths; source-IP filtering does not change the route.
- **B 不正解**: 分割は運用を複雑にするだけで、それぞれの経路制御は結局必要です。要件は 1 サーバーでも満たせます。 / Adds operational complexity while the same controls are still required.
- **D 不正解**: 社内データを外部 SaaS に複製するのは、統制を弱める方向でセキュリティポリシーに反します。 / Moves internal data outside the perimeter — the opposite of the policy's intent.

**参照 / Reference:** プライベート接続、egress 制御、ネットワーク境界設計
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

MCP サーバーのコードレビューで、下流システムの API キーがソースコードに直接記述されており、リポジトリにコミットされていることが判明しました。開発者は「プライベートリポジトリなので問題ない」と説明しています。このキーは本番の在庫システムへの読み書き権限を持ち、過去 2 年間ローテーションされていません。

A code review of an MCP server finds a downstream API key hard-coded in source and committed to the repository. The developer says "it's a private repository, so it's fine." The key carries read/write access to the production inventory system and has not been rotated in two years.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **キーを直ちに無効化して新しいキーを発行し、シークレット管理の仕組みから実行時に注入する構成に変える**。コミット履歴に残ったキーは無効化済みであることを前提に扱い、リポジトリの履歴に秘密情報が入らないよう検出の仕組み（コミット前スキャン）を導入する。あわせて定期ローテーションを設定し、キーの権限も必要最小限に絞り直す / **Revoke the key immediately, issue a new one, and inject it at runtime from a secrets manager.** Treat the key in commit history as compromised and already revoked, add pre-commit secret scanning so history stays clean, establish periodic rotation, and re-scope the key's permissions to the minimum required
- B) リポジトリをプライベートのまま維持し、アクセスできる開発者を制限する / Keep the repository private and limit which developers can access it
- C) キーを Base64 エンコードしてコードに記述する / Base64-encode the key in the source
- D) キーを環境変数に移し、コミット履歴はそのままにする / Move the key to an environment variable and leave the history as is

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**コミット履歴に入った秘密情報は漏洩したものとして扱う**のが原則です。プライベートリポジトリであっても、アクセス権を持つ全員・過去のフォーク・バックアップ・CI のログに残り得ます。したがって最初の行動は無効化と再発行で、その後に実行時注入への移行、履歴汚染の再発防止（コミット前スキャン）、定期ローテーション、権限の最小化と続きます。2 年間ローテーションされていない広い権限のキーは、それ自体が独立したリスクです。

A secret committed to history is treated as disclosed: private or not, it persists across everyone with access, past forks, backups, and CI logs. So the first action is revoke and reissue, followed by runtime injection, pre-commit scanning to prevent recurrence, rotation, and re-scoping. A two-year-old key with broad write access is a separate risk on its own.

- **B 不正解**: アクセス制限は既に履歴に残ったキーを回収できません。無効化しない限りキーは有効なままです。 / Access limits do not recover a key already in history.
- **C 不正解**: Base64 は暗号化ではなく単なる符号化で、秘密情報の保護になりません。 / Base64 is encoding, not protection.
- **D 不正解**: 環境変数化は正しい方向ですが、履歴に残った有効なキーを放置しており、最も重要な対応（無効化）が欠けています。 / Right direction, but leaves a live key in history — the critical step is missing.

**参照 / Reference:** シークレット管理、資格情報のローテーション、コミット履歴の汚染、最小権限
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

外部パートナー 40 社のエージェントが利用する MCP サーバーを提供しています。ツール `submit_order` の引数に、規制対応のため新たな必須項目 `delivery_country_code` を追加する必要が生じました。パートナーの対応時期はまちまちで、一部は数か月かかります。規制の適用日は 90 日後です。

You operate an MCP server used by 40 external partners' agents. For regulatory reasons, the `submit_order` tool needs a new required argument, `delivery_country_code`. Partner readiness varies, with some needing several months. The regulation takes effect in 90 days.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 90 日後に必須引数を追加し、対応できていないパートナーは利用停止とする / Add the required argument at 90 days and cut off unprepared partners
- B) **バージョン付きのツールを並行提供する**。新引数を持つ `submit_order`（v2 相当）を追加し、既存版は移行期間中も動作させる。パートナーには適用日・移行期限・旧版の停止日を明示して通知し、移行状況をパートナー単位で追跡する。規制適用日以降、旧版経由の注文は規制上受け付けられないため、その扱い（拒否するのか、別途情報を補うのか）を法務と事前に確定させる / **Offer versioned tools in parallel**: add the new-argument version alongside the existing one during a migration window, notify partners of the effective date, migration deadline, and end-of-life date, and track adoption per partner. Because orders through the old version cannot be accepted after the effective date, settle with legal in advance how they will be handled — rejected, or supplemented from another source
- C) 新引数を省略可能にして、省略時は既定値（自国コード）を設定する / Make the argument optional with a default of the home country code
- D) パートナーに通知せず、サーバー側で配送先住所から国コードを推定する / Infer the country code from the delivery address server-side without telling partners

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

外部パートナーが関わる破壊的変更では、**移行期間の提供が実務上必須**ですが、規制の適用日は動きません。したがって、バージョン並行提供で技術的な移行猶予を作りつつ、**規制適用日以降に旧版経由で来た注文をどう扱うか**を事前に決めておく必要があります。この後段の判断を先送りすると、適用日当日に法務判断を迫られることになります。移行状況をパートナー単位で追跡することが、期限管理の実務です。

Breaking changes affecting external partners require a migration window, but the regulatory date does not move. So parallel versions buy technical time while the substantive question — what happens to old-version orders after the effective date — must be settled with legal beforehand, or it lands as an emergency on the day. Per-partner adoption tracking is how the deadline is actually managed.

- **A 不正解**: 移行期間なしの必須化は 40 社の統合を同時に壊し、商業的にも受け入れられません。 / No migration window breaks 40 integrations at once.
- **C 不正解**: 誤った既定値を設定すると、規制上要求される情報が不正確なまま記録されます。空欄より有害です。 / A wrong default records inaccurate regulated data — worse than an omission.
- **D 不正解**: 推定値を規制報告に使うのはリスクが高く、パートナーに通知しない変更はさらに問題です。 / Inferred values in regulated reporting, plus an undisclosed behavior change.

**参照 / Reference:** ツールのバージョニング、外部パートナーへの移行計画、規制期限との整合
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

本番で「エージェントの応答が遅い」という報告があります。1 リクエストは、エージェント → MCP サーバー A（顧客照会）→ 内部 API → DB、および MCP サーバー B（在庫照会）→ 外部 SaaS という経路を通ります。各コンポーネントは個別にログを出していますが、リクエストを横断して紐付ける手段がなく、どこで時間を消費しているのか特定できません。調査に毎回数時間かかっています。

Production reports that "the agent is slow." One request traverses agent → MCP server A (customer lookup) → internal API → database, and MCP server B (inventory) → external SaaS. Each component logs independently, but there is no way to correlate a single request across them, so no one can tell where the time goes. Each investigation takes hours.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 各コンポーネントのログ出力を増やす / Increase log verbosity in each component
- B) 各コンポーネントの平均レイテンシをダッシュボードに表示する / Chart each component's average latency on a dashboard
- C) 最も遅そうなコンポーネントから順に個別に計測する / Measure components one at a time, starting with the likeliest suspect
- D) **分散トレーシングを導入する**。エージェントの入口でトレース ID を発行し、MCP サーバー・内部 API・下流呼び出しへ伝播させて、1 リクエストの全区間を 1 つのトレースとして可視化する。各区間のスパンに所要時間を記録し、外部 SaaS 呼び出しなど自社外の区間も計測対象に含める。既存のログにもトレース ID を付与して相関できるようにする / **Introduce distributed tracing**: issue a trace ID at the agent's entry point, propagate it through MCP servers, internal APIs, and downstream calls, and render one request as a single trace with a timed span per segment — including third-party segments such as the external SaaS. Stamp existing logs with the trace ID so they correlate

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

複数コンポーネントをまたぐレイテンシ問題は、**リクエスト単位の相関がなければ原因を特定できません**。分散トレーシングはトレース ID の伝播により 1 リクエストの全区間を 1 本の時系列として可視化し、どの区間が支配的かを即座に示します。平均値のダッシュボードでは、遅いリクエストが平均に埋もれて見えなくなります。既存ログにトレース ID を付けることで、トレースから該当ログへ辿る導線もできます。

Cross-component latency cannot be attributed without per-request correlation. Distributed tracing propagates a trace ID so one request renders as a single timeline showing which segment dominates. Averages hide slow requests, and stamping existing logs with the trace ID gives you a path from a trace to its logs.

- **A 不正解**: ログ量を増やしても、リクエスト単位で紐付ける手段がなければ突き合わせは手作業のままです。 / More logs without correlation means the same manual reconciliation.
- **B 不正解**: 平均値は個別の遅いリクエストを覆い隠し、区間ごとの因果関係も示しません。 / Averages mask slow requests and show no per-request causality.
- **C 不正解**: 推測に基づく逐次調査は時間がかかり、複数区間が絡む場合に原因を見誤ります。 / Guess-driven serial investigation is slow and misleading with multiple segments.

**参照 / Reference:** 分散トレーシング、トレース ID の伝播、スパンによる区間計測
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

MCP サーバーが下流 API を呼び出す際、現在はすべてのエラーに対して一律に「3 回まで指数バックオフでリトライ」しています。運用すると、(1) 引数の検証エラー（HTTP 400）でも 3 回リトライして無駄に時間を消費する、(2) 認証エラー（401）も同様、(3) 一方、下流の過負荷（503、`Retry-After` ヘッダ付き）では 3 回では足りず失敗する、という問題が出ています。

An MCP server currently retries every downstream error identically: three attempts with exponential backoff. In practice: (1) validation errors (HTTP 400) burn three attempts pointlessly, (2) so do authentication errors (401), and (3) meanwhile downstream overload (503 with a `Retry-After` header) needs more than three and fails.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) リトライ回数を一律 10 回に増やす / Raise the uniform retry count to ten
- B) リトライを廃止してすべて即座に失敗させる / Remove retries and fail fast on everything
- C) **エラーの種別に応じてリトライ方針を分ける**。4xx のうち恒久的な失敗（400 引数不正、401 認証、403 認可、404 不存在）はリトライせず即座に返す。429 と 5xx は再試行対象とし、`Retry-After` ヘッダがあればその値を尊重する。バックオフにはジッタを入れて同時再試行の集中を避け、全体の試行時間に上限を設ける / **Differentiate the retry policy by error class**: do not retry permanent 4xx failures (400 malformed, 401 unauthenticated, 403 unauthorized, 404 missing) and return them immediately; retry 429 and 5xx, honoring `Retry-After` when present; add jitter to the backoff to avoid synchronized retries, and bound the total time spent attempting
- D) すべてのエラーを 429 として扱い、統一的にリトライする / Treat every error as a 429 and retry uniformly

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**リトライの可否はエラーの性質で決まります。**引数が不正なリクエストは何度送っても不正なので、リトライは時間とコストの純損失です。一方、一時的な過負荷は再試行で回復し得るため、下流が示す `Retry-After` を尊重するのが協調的な振る舞いです。ジッタは複数クライアントの再試行が同期して下流をさらに圧迫するのを防ぎます。全体の試行時間に上限を設けるのは、呼び出し元を無限に待たせないためです。

Retryability is determined by the nature of the error. A malformed request stays malformed, so retrying is pure loss; transient overload may recover, and honoring `Retry-After` is the cooperative behavior. Jitter prevents synchronized retries from compounding the overload, and an overall time bound keeps the caller from waiting indefinitely.

- **A 不正解**: 恒久的な失敗に対する無駄なリトライが 10 回に増え、レイテンシがさらに悪化します。 / Multiplies the waste on permanent failures.
- **B 不正解**: 一時的な障害でも即座に失敗するため、回復可能な状況で可用性を落とします。 / Sacrifices availability on recoverable failures.
- **D 不正解**: エラーの意味を潰す扱いで、認証エラーなど対処が必要な問題を検知できなくなります。 / Erases error semantics and hides problems needing action.

**参照 / Reference:** リトライ方針、エラー分類、`Retry-After`、ジッタ付きバックオフ
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

在庫調整エージェントが、棚卸の結果に基づいて多数の在庫レコードを更新します。現在は `update_stock(sku, quantity)` を SKU ごとに呼び出しており、1 回の棚卸で 3,000 回のツール呼び出しが発生します。処理に 50 分かかり、途中で失敗するとどこまで完了したか分かりません。エージェントのコンテキストも 3,000 回分のツール呼び出しと結果で埋まります。

An inventory-reconciliation agent updates many stock records after a physical count. It calls `update_stock(sku, quantity)` per SKU — 3,000 tool calls per count. It takes 50 minutes, and on failure there is no record of how far it got. The agent's context also fills with 3,000 call/result pairs.

**設問 / Question:**

最も適切なツール設計はどれですか？

Which tool design is most appropriate?

- A) `update_stock` の呼び出しを並列化して時間を短縮する / Parallelize the `update_stock` calls to shorten the run
- B) **一括更新ツールを追加する**。`update_stock_batch(updates: [{sku, quantity}, ...])` として複数件を 1 回で受け付け、サーバー側でトランザクションまたはバッチ処理として実行する。結果は成功件数と失敗した SKU の一覧（理由付き）を集約して返し、バッチ全体に冪等キーを付与して再実行時の二重適用を防ぐ / **Add a bulk tool**: `update_stock_batch(updates: [{sku, quantity}, ...])` accepting many records per call and executing them as a transaction or batch server-side, returning an aggregate result — success count plus the failed SKUs with reasons — and carrying a batch-level idempotency key so a re-run does not double-apply
- C) 1 回の棚卸で更新する SKU 数を 100 件に制限する / Limit each reconciliation to 100 SKUs
- D) 更新をエージェントではなく人間が手作業で行う / Have humans perform the updates manually

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**大量の同種操作は、1 件ずつのツール呼び出しには向きません。**一括ツールにすると、往復回数が激減してレイテンシとコンテキスト消費が改善し、サーバー側でトランザクションまたはバッチとして扱えるため部分失敗の扱いも明確になります。結果を集約して返す設計（成功件数＋失敗 SKU と理由）が重要で、3,000 件の個別結果をコンテキストに戻す必要がなくなります。バッチ単位の冪等キーは、再実行時の二重適用を防ぎます。

Many homogeneous operations do not belong in per-item tool calls. A bulk tool collapses round trips, cutting latency and context, and lets the server handle the set as a transaction or batch with well-defined partial-failure semantics. Returning an aggregate — count plus failures with reasons — keeps 3,000 individual results out of context, and a batch-level idempotency key makes re-runs safe.

- **A 不正解**: 並列化は時間を短縮しますが、コンテキスト消費と再開可能性の問題は残ります。下流への同時負荷も増えます。 / Helps latency only; context and resumability problems remain.
- **C 不正解**: 件数制限は業務要件（全 SKU の棚卸）を満たせません。 / Fails the business requirement to reconcile all SKUs.
- **D 不正解**: 3,000 件の手作業は自動化の目的を否定します。 / Negates the purpose of automation.

**参照 / Reference:** 一括操作ツールの設計、集約結果の返却、バッチ冪等性
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

MCP サーバーと 4 つのエージェントアプリケーションを別々のチームが開発しています。MCP サーバー側で `get_invoice` ツールの返却フィールド `status` の値集合を変更（`paid`/`unpaid` → `paid`/`pending`/`overdue`）したところ、2 つのエージェント側で分岐処理が壊れ、本番で誤った案内が顧客に送られました。変更は MCP サーバーのユニットテストをすべて通過していました。

Separate teams develop an MCP server and four agent applications. When the server changed the value set of `get_invoice`'s `status` field (`paid`/`unpaid` → `paid`/`pending`/`overdue`), branching logic in two agents broke and incorrect guidance reached customers in production. The change passed all of the server's unit tests.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) **コンシューマ駆動の契約テストを導入する**。各エージェントが依存するツールの入出力（フィールド、値集合、必須性）を契約として宣言し、MCP サーバーの CI でその契約全件を検証する。契約を破る変更は CI で失敗し、本番に到達しない。値集合の追加は破壊的変更として扱い、コンシューマ側の対応を確認してから反映する / **Introduce consumer-driven contract tests**: each agent declares the tool inputs and outputs it depends on — fields, value sets, required-ness — as a contract, and the MCP server's CI verifies every contract, so a contract-breaking change fails CI and never reaches production. Treat value-set additions as breaking, and land them only after consumers confirm readiness
- B) MCP サーバーのユニットテストのカバレッジを上げる / Raise the MCP server's unit-test coverage
- C) 変更時に 4 チームへメールで通知する運用を定める / Establish a process of emailing the four teams before changes
- D) エージェント側で未知の `status` 値を受け取ったら無視する実装にする / Have agents ignore unknown `status` values

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**サーバー側のユニットテストは、コンシューマの期待を知りません。**サーバーは自分の仕様どおりに動いており、壊れたのはコンシューマとの暗黙の契約です。コンシューマ駆動の契約テストは、その期待を明示的な成果物にして CI で検証可能にします。これにより、破壊的変更がマージ前に検出され、組織的な通知運用に頼らずに済みます。値集合の追加が破壊的変更であるという認識も重要で、フィールドの追加とは影響が異なります。

Server-side unit tests cannot know consumer expectations: the server behaved to its own spec, and what broke was an implicit contract. Consumer-driven contract tests turn those expectations into an artifact CI can verify, catching breaking changes before merge without relying on notification processes. Recognizing that widening a value set *is* breaking — unlike adding a field — matters too.

- **B 不正解**: サーバー側のカバレッジをいくら上げても、コンシューマ側の分岐条件は検証対象に入りません。 / Server coverage never exercises consumer branching.
- **C 不正解**: 人的な通知は見落とされ、通知されても対応漏れを機械的に防げません。 / Human notification is missable and not enforcing.
- **D 不正解**: 未知の値を無視すると、`overdue` の請求書が正常扱いされるなど静かな誤動作を招きます。 / Silently ignoring `overdue` produces a quiet, worse failure.

**参照 / Reference:** コンシューマ駆動契約テスト、破壊的変更の定義、CI での契約検証
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

サポートエージェント用の `get_customer_profile` ツールが、顧客の氏名・住所・電話番号・生年月日・マイナンバー・クレジットカード下 4 桁・過去の問い合わせ履歴を返します。エージェントの用途は「過去の問い合わせ履歴を踏まえて回答すること」と「本人確認（氏名と電話番号の照合）」の 2 つです。返却値はすべてコンテキストに入り、会話ログとして 3 年間保存されます。

A support agent's `get_customer_profile` returns name, address, phone, date of birth, national ID number, last four digits of a credit card, and past inquiry history. The agent's uses are answering in light of past inquiries and verifying identity by matching name and phone. Everything returned enters context and is retained in conversation logs for three years.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 全項目を返し続け、会話ログを暗号化する / Keep returning everything and encrypt the conversation logs
- B) 全項目を返し、エージェントに不要な項目を使わないよう指示する / Keep returning everything and instruct the agent not to use unneeded fields
- C) **用途に必要な項目だけを返すようツールを再設計する**。問い合わせ履歴と、本人確認に必要な氏名・電話番号のみを返し、マイナンバー・生年月日・住所・カード情報は返却対象から除く。本人確認は、生の値をコンテキストに載せずに済むよう、照合結果（一致・不一致）だけを返す専用ツールに分ける / **Redesign the tool to return only what the use cases need**: inquiry history plus the name and phone required for verification, excluding national ID, date of birth, address, and card data. Split identity verification into a dedicated tool that returns only a match/no-match result, so the raw values never enter context at all
- D) マイナンバーとカード情報をマスクした文字列として返す / Return the national ID and card data as masked strings

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**コンテキストに入った個人情報は、会話ログとして 3 年間残ります。**したがって、そもそも境界を越えさせないのが最も強い統制です。用途を分析すると、マイナンバー・生年月日・住所・カード情報はいずれの用途にも不要で、返却対象から外せます。さらに本人確認は「照合の結果」だけあれば足りるので、専用ツールで照合を行い真偽値を返す設計にすれば、氏名と電話番号すらコンテキストに載せずに済みます。**データ最小化を用途の分析から導く**のが本問の要点です。

Personal data that enters context persists in three years of logs, so the strongest control is never letting it cross the boundary. Analyzing the use cases shows national ID, date of birth, address, and card data serve neither. Verification needs only the *outcome*, so a dedicated match tool returning a boolean keeps even name and phone out of context — data minimization derived from use-case analysis.

- **A 不正解**: 暗号化は保存時の保護ですが、不要な個人情報がコンテキストとログに 3 年間存在する事実は変わりません。 / Encryption at rest does not change what is retained.
- **B 不正解**: 「使わない」よう指示しても、データは既にコンテキストとログに存在します。指示は保持の統制になりません。 / The data is already present regardless of use.
- **D 不正解**: マスクは改善ですが、そもそも不要な項目を返す設計自体を見直していません。 / An improvement that still returns fields no use case needs.

**参照 / Reference:** データ最小化、境界での項目制限、照合結果のみを返す設計
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

エージェントが呼び出す外部の与信照会 API が、月に数回、数十分にわたって応答しなくなります（タイムアウトのみを返す）。その間、エージェントは毎リクエストで 30 秒のタイムアウトを待ち、リトライも行うため 1 リクエストあたり 2 分以上を消費します。結果としてワーカーが枯渇し、与信照会を必要としない他の機能まで応答不能になります。

An external credit-check API that the agent calls becomes unresponsive for tens of minutes several times a month, returning only timeouts. During those windows the agent waits the full 30-second timeout per request and retries, spending over two minutes per request. Workers exhaust and even features that do not need credit checks stop responding.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) タイムアウトを 5 秒に短縮する / Reduce the timeout to five seconds
- B) ワーカー数を 3 倍に増やす / Triple the worker count
- C) 与信照会が失敗したらエージェント全体を停止する / Stop the entire agent when a credit check fails
- D) **サーキットブレーカーを導入する**。連続失敗が閾値を超えたら回路を開き、一定時間は与信照会を即座に失敗させて待機時間を消費しない。回路が開いている間は縮退した挙動（与信照会なしで処理を進める、または人間にエスカレーション）に切り替える。一定時間後に少数のリクエストを通して回復を確認し、成功すれば回路を閉じる。あわせて与信照会用のワーカープールを他機能から隔離する / **Introduce a circuit breaker**: open the circuit after consecutive failures exceed a threshold so credit checks fail immediately instead of consuming wait time, switch to degraded behavior while open (proceed without the check, or escalate to a human), probe with a few requests after a cooldown and close on success — and isolate the credit-check worker pool from other features

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**障害中の下流を叩き続けることが、自分たちのリソースを枯渇させる**という典型的な障害連鎖です。サーキットブレーカーは、失敗が続く相手への呼び出しを即座に失敗させることで待機時間の消費を止め、リソース枯渇を防ぎます。回復確認（半開状態）により自動復旧も可能です。加えて、ワーカープールの隔離（バルクヘッド）により、与信照会の障害が他機能に波及しない構造にします。この 2 つは補完的な対策です。

Continuing to hammer a failed dependency is what exhausts your own resources. A circuit breaker stops the wait-time consumption by failing fast while the dependency is down, with a half-open probe for automatic recovery. Pool isolation (bulkheading) additionally prevents the credit-check failure from reaching unrelated features — the two controls are complementary.

- **A 不正解**: タイムアウト短縮は消費時間を減らしますが、障害中も呼び出し続けるためリソース枯渇は緩和にとどまります。 / Reduces but does not stop the resource drain.
- **B 不正解**: ワーカー増設は枯渇までの時間を延ばすだけで、より長い障害では同じ結果になります。コストも増えます。 / Only delays exhaustion.
- **C 不正解**: 与信照会を必要としない機能まで止めるのは、影響範囲を自ら広げる対応です。 / Widens the blast radius deliberately.

**参照 / Reference:** サーキットブレーカー、バルクヘッド（プール隔離）、障害連鎖の遮断
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

返品受付エージェントを開発しています。返品可否のルールは「購入から 30 日以内」「未開封」「セール品は対象外」「プレミアム会員は 60 日以内」で、法務が管理しており年に数回改定されます。実装案は 2 つあります。案 1: ルールをシステムプロンプトに記述し、モデルが `get_order` の結果を見て判定する。案 2: MCP サーバーに `check_return_eligibility(order_id)` を実装し、サーバー内でルールを評価して可否と理由を返す。

You are building a returns-intake agent. Eligibility rules — within 30 days of purchase, unopened, sale items excluded, 60 days for premium members — are owned by legal and revised a few times a year. Two options: (1) state the rules in the system prompt and let the model decide from `get_order` output, or (2) implement `check_return_eligibility(order_id)` in the MCP server, evaluating the rules server-side and returning a decision with reasons.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) 案 1。プロンプトの変更はデプロイ不要で、ルール改定に素早く追随できる / Option 1: prompt changes need no deploy and track rule revisions quickly
- B) **案 2**。可否判定は決定的なビジネスルールであり、法務が管理する規則を確率的に評価させるべきではない。サーバー側に置けば単体テストで全条件を検証でき、改定時の変更箇所が 1 か所に収まり、判定結果と理由が監査可能な形で残る。エージェントは判定結果を受けて顧客への説明を組み立てる役割に専念する / **Option 2**: eligibility is a deterministic business rule, and rules owned by legal should not be evaluated probabilistically. Server-side, every condition is unit-testable, revisions land in one place, and decisions with reasons are auditable — leaving the agent to do what it is good at: explaining the outcome to the customer
- C) 案 1 と案 2 の両方を実装し、結果が食い違ったら人間に回す / Implement both and escalate to a human when they disagree
- D) 判定を顧客に自己申告させる / Have the customer self-declare eligibility

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**ビジネスルールの評価は決定的な処理であり、統合レイヤー（MCP サーバー）に置くのが適切**です。法務が管理する規則を確率的に評価すると、境界的なケース（購入 30 日目、プレミアム会員かつセール品）で判定がぶれ、説明も一貫しません。サーバー側に置けば単体テストで全組み合わせを検証でき、改定の影響範囲が明確になります。モデルの役割は判定ではなく、判定結果を顧客に分かりやすく説明することです。**「モデルが得意なこと」と「コードが得意なこと」の切り分け**が要点です。

Business-rule evaluation is deterministic work that belongs in the integration layer. Evaluating legally owned rules probabilistically makes borderline cases — day 30, a premium member with a sale item — inconsistent and inconsistently explained. Server-side, every combination is unit-testable and revisions have a clear blast radius, leaving the model to do what it does well: explain the outcome.

- **A 不正解**: デプロイ不要という利便性は、判定の一貫性と監査可能性を犠牲にしています。法務管理のルールでは不適切なトレードオフです。 / Trades consistency and auditability for convenience, on rules owned by legal.
- **C 不正解**: 二重実装は保守コストが 2 倍になり、食い違いの検出は決定的な実装が既にあることを前提にしています。 / Doubles maintenance, and presupposes the deterministic implementation anyway.
- **D 不正解**: 自己申告はルールの適用そのものを放棄しており、不正利用に対して無防備です。 / Abandons rule enforcement entirely.

**参照 / Reference:** ビジネスロジックの配置、決定的評価、モデルとコードの役割分担
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

MCP サーバーの `search_legacy_orders` ツールを廃止したいと考えています。後継の `search_orders` は 8 か月前に提供済みで機能的に上位互換ですが、テレメトリを見ると `search_legacy_orders` は今も 1 日 400 回呼ばれています。呼び出し元は 3 つのエージェントアプリケーションで、うち 1 つは呼び出し箇所を特定できていません。

You want to retire the `search_legacy_orders` tool. Its successor `search_orders` shipped eight months ago and is a functional superset, but telemetry shows `search_legacy_orders` is still called 400 times a day from three agent applications, one of which has not located its call site.

**設問 / Question:**

最も適切な廃止手順はどれですか？

Which deprecation procedure is most appropriate?

- A) **段階的な廃止手順を踏む**。まずツール説明に非推奨である旨と後継を明記し、呼び出し元をテレメトリで特定して各チームに移行を依頼する。次に呼び出しごとに警告をログに出し、呼び出し元別の残存件数を可視化する。移行完了後、一定期間ツールを「呼び出すと明示的なエラーを返す」状態にしてから削除する。削除日は事前に周知する / **Deprecate in stages**: mark the tool deprecated in its description with a pointer to the successor, identify callers from telemetry and ask each team to migrate, log a warning per call and track remaining volume per caller, then — once migration completes — leave the tool returning an explicit error for a defined period before removing it, with the removal date announced in advance
- B) 即座に削除し、壊れたアプリケーションのチームに連絡してもらう / Remove it immediately and let the affected teams report breakage
- C) ツールを残したまま、内部実装を空にして何も返さないようにする / Keep the tool but empty its implementation so it returns nothing
- D) 呼び出し元が特定できないので、廃止せず永久に維持する / Since one caller is unidentified, keep it forever

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**利用中のインターフェースの廃止は段階的に行います。**要点は、(1) 非推奨の明示（ツール説明はモデルにも読まれるため、新規利用を抑制できる）、(2) テレメトリによる呼び出し元の特定（特定できていない 1 つを見つける唯一の手段）、(3) 明示的なエラーを返す期間（黙って消すより、残存呼び出しを確実に顕在化させる）、(4) 削除日の事前周知、です。削除前にエラーを返す段階を置くことで、見落とされた呼び出し元が本番障害ではなく明確なエラーとして検出されます。

Retiring a live interface is staged. Marking it deprecated in the description discourages new use (the model reads it too); telemetry is the only way to find the unidentified caller; an explicit-error period surfaces stragglers deliberately rather than silently; and announcing the removal date sets expectations. The error stage is what turns a missed caller into a clear signal instead of a production incident.

- **B 不正解**: 400 回/日の呼び出しを予告なく壊すのは、共有インターフェースの提供者として不適切です。 / Breaking 400 calls/day without notice is not acceptable for a shared interface.
- **C 不正解**: 空の結果を返すのは最悪の廃止方法で、呼び出し元は「該当データなし」と解釈して静かに誤動作します。 / Empty results are read as "no data," producing silent wrong behavior.
- **D 不正解**: 特定できないことは維持の理由になりません。テレメトリと段階的手順で特定できます。 / An unidentified caller is findable; it is not grounds for permanent maintenance.

**参照 / Reference:** ツールの非推奨化と削除、テレメトリによる利用状況把握、段階的廃止
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

文書処理エージェント用に、スキャンした PDF（1 件あたり 5〜80 MB）を扱うツールを設計しています。案として、`get_document(document_id)` が PDF の内容を Base64 エンコードして返す実装が提案されました。エージェントは受け取った文書を分析ツールに渡したり、要約したりします。1 日 2,000 件処理する見込みです。

You are designing a tool for scanned PDFs (5–80 MB each) for a document-processing agent. A proposal has `get_document(document_id)` return the PDF Base64-encoded. The agent passes the document to analysis tools and summarizes it. Expected volume is 2,000 documents/day.

**設問 / Question:**

最も適切な設計はどれですか？

Which design is most appropriate?

- A) **大容量のバイナリをツールの戻り値として流さず、参照で受け渡す設計にする**。`get_document` は文書の識別子・メタデータ（ページ数、種別、作成日）と、必要に応じて署名付きの一時 URL を返す。実際の解析は文書 ID を引数に取るツール（`extract_document_text(document_id)` など）がサーバー側で行い、モデルには解析結果だけを返す / **Do not stream large binaries through tool results; pass references instead.** `get_document` returns the identifier, metadata (page count, type, date), and where needed a signed temporary URL, while analysis happens server-side in tools that take the document ID (`extract_document_text(document_id)`), returning only results to the model
- B) Base64 で返すが、10 MB を超える文書は先頭 10 MB のみ返す / Return Base64 but truncate documents over 10 MB
- C) Base64 で返し、コンテキスト上限に達したら会話をリセットする / Return Base64 and reset the conversation when the context fills
- D) 全文書を事前にテキスト化してデータベースに保存し、ツールは常に全文テキストを返す / Pre-extract all documents to text and always return the full text

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**大容量バイナリをツールの戻り値としてモデルのコンテキストに流すのは設計上の誤り**です。80 MB の Base64 は数千万トークン相当で、コンテキストに収まらず、収まったとしてもコストが非現実的です。正しい形は、**モデルには参照（ID・メタデータ）だけを渡し、重い処理はサーバー側で行って結果だけを返す**ことです。モデルが扱うべきは解析結果であって、バイト列ではありません。

Streaming large binaries into model context is a design error: 80 MB of Base64 is tens of millions of tokens — it does not fit, and would be economically absurd if it did. The correct shape gives the model references and metadata while heavy processing happens server-side and only results return. The model's business is the analysis, not the bytes.

- **B 不正解**: 先頭のみの切り詰めは文書の大半を失い、静かに不完全な分析を生みます。 / Silently discards most of the document.
- **C 不正解**: コンテキストリセットは作業文脈を失い、根本的にサイズの問題を解決しません。 / Loses working context without addressing the size problem.
- **D 不正解**: 常に全文テキストを返すのは、80 MB のスキャン文書では依然として巨大で、必要な部分だけを扱う設計になっていません。 / Full text of a large scan is still enormous, and returns far more than needed.

**参照 / Reference:** 大容量データの参照渡し、サーバー側処理、コンテキストに載せるべき情報
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

社内の 1 つのエージェントアプリケーションが、そのアプリ専用に構築された社内マイクロサービス（そのアプリ以外から呼ばれる予定はない）を呼び出す必要があります。呼び出しは 1 種類のみで、認証は同一クラスタ内のサービス間通信として既に確立しています。チームから「統合には必ず MCP サーバーを立てるべきか」という質問が来ました。他チームからの利用要望も、外部公開の予定もありません。

One internal agent application needs to call an internal microservice built specifically for it, with no other planned consumers. There is a single call, and authentication is already established as in-cluster service-to-service communication. The team asks whether integration must always go through an MCP server. No other team has requested access and there are no plans to expose it externally.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) 社内標準として、すべての統合に MCP サーバーを立てる / As an internal standard, stand up an MCP server for every integration
- B) MCP サーバーを立てたうえで、将来の利用者に備えて汎用的なツールを 20 個定義する / Stand up an MCP server and define 20 general-purpose tools for future consumers
- C) マイクロサービスを廃止して、エージェント内にロジックを取り込む / Retire the microservice and absorb its logic into the agent
- D) **MCP サーバーを立てず、アプリケーション内のツール定義として直接統合する**。MCP が価値を持つのは、複数のクライアントからの再利用、プロセス境界をまたぐ配布、横断的関心事の集約が必要な場合であり、ここではいずれも当てはまらない。将来、他チームからの利用要望が生じた時点で MCP サーバーへ切り出せばよく、そのための抽象化は今は不要である / **Do not stand up an MCP server; integrate directly as a tool defined in the application.** MCP earns its cost when a capability is reused by multiple clients, distributed across process boundaries, or carries cross-cutting concerns — none of which applies here. If another team asks for it later, extract it into an MCP server then; the abstraction is not needed now

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**MCP はすべての統合に必要なわけではありません。**単一のクライアントが専用のサービスを 1 種類だけ呼ぶケースでは、MCP サーバーを立てることで得られるもの（再利用、集中管理、外部提供）が存在せず、運用対象のコンポーネントが 1 つ増えるだけです。適切な判断は、直接のツール定義として実装し、**再利用の要求が実際に生じた時点で切り出す**ことです。「標準だから」という理由での一律適用は、複雑さを要件なしに増やします。

MCP is not required for every integration. Where a single client makes a single call to a dedicated service, none of MCP's benefits — reuse, centralized control, external exposure — are present, and the result is one more component to operate. Implement it as a direct tool definition and extract it when reuse actually materializes. Applying a standard uniformly, absent the conditions that motivate it, adds complexity for nothing.

- **A 不正解**: 標準の一律適用は、適合条件を確認せずに複雑さを増やします。標準は判断を置き換えるものではありません。 / Uniform application adds complexity without checking the conditions that justify it.
- **B 不正解**: 存在しない将来の利用者のために 20 ツールを定義するのは、典型的な投機的過剰設計です。 / Speculative over-engineering for consumers who do not exist.
- **C 不正解**: 動作しているマイクロサービスを廃止する理由がなく、統合方式の質問に対する答えになっていません。 / No reason to retire a working service, and it does not answer the question asked.

**参照 / Reference:** MCP の適合条件、適正な複雑さ、後からの切り出し
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **前 / Previous:** [`domain2_models_prompting_context.md`](./domain2_models_prompting_context.md) | **次 / Next:** [`domain4_evaluation.md`](./domain4_evaluation.md)
