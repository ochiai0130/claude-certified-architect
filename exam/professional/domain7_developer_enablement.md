# Domain 7: 開発者生産性と運用イネーブルメント / Developer Productivity and Operational Enablement

> 配点比率 / Weight: **7%**（最も配点が小さい / the smallest domain）
> 問題数 / Questions: **30**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- 組織規模での CLAUDE.md・スキル・スラッシュコマンドの管理と配布 / Managing and distributing CLAUDE.md, skills, and slash commands at organizational scale
- CI/CD への組み込みと開発ワークフローのガードレール / CI/CD integration and guardrails in the development workflow
- 内部プラットフォーム・ゴールデンパス・支援体制 / Internal platforms, golden paths, and support models
- 可観測性・コスト配賦・容量の共有 / Observability, cost attribution, shared capacity
- 生産性への効果の測定と定着 / Measuring productivity impact and driving adoption
- 運用体制・ランブック・インシデントからの学習の展開 / Operating models, runbooks, and propagating incident learnings

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

全社 400 名のエンジニアが Claude Code を利用しています。各リポジトリの `CLAUDE.md` はチームが自由に記述しており、内容は「コーディング規約」「アーキテクチャの説明」「よくあるタスクの手順」「個人的な好み」まで様々です。全社共通の規約（セキュリティ要件、ログ形式、依存ライブラリの方針）を各リポジトリに反映したいのですが、120 のリポジトリを個別に更新するのは現実的ではありません。

400 engineers use Claude Code. Each repository's `CLAUDE.md` is written freely by its team and ranges from coding conventions to architecture notes, common task procedures, and personal preferences. You want organization-wide conventions (security requirements, log format, dependency policy) reflected everywhere, but updating 120 repositories individually is not practical.

**設問 / Question:**

最も適切な管理方法はどれですか？

Which management approach is most appropriate?

- A) 全社共通の規約を各チームに周知し、反映を各チームに任せる / Communicate the conventions and leave adoption to each team
- B) 120 リポジトリの `CLAUDE.md` を一括置換スクリプトで更新する / Update all 120 files with a bulk replacement script
- C) 全社共通の規約のみを記述した単一の `CLAUDE.md` に統一する / Standardize on one `CLAUDE.md` containing only the shared conventions
- D) **階層構造で管理し、共通部分と個別部分を分離する**。全社共通の規約は 1 か所で管理された参照先に置き、各リポジトリの `CLAUDE.md` からはそれを参照する形にする。リポジトリ固有の内容（アーキテクチャ、そのリポジトリでのタスク手順）はローカルに記述する。共通規約の更新は参照先を変えるだけで全体に反映され、各チームの自律性も保たれる / **Manage it hierarchically, separating shared content from local content.** Keep the organization-wide conventions in one maintained location that each repository's `CLAUDE.md` references, and write repository-specific material (architecture, local task procedures) locally. Updating the shared conventions then reaches everything by changing one place, while teams keep autonomy over their own content

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**共通部分と個別部分を分離するのは、設定管理の基本**です。全社共通の規約を各リポジトリに複製すると、更新のたびに 120 か所の同期が必要になり、必ず漏れが生じます。参照による構成にすれば、共通部分の更新は 1 か所で済み、各チームはリポジトリ固有の内容に集中できます。この構造は、共通規約の所有者（セキュリティ部門など）と各リポジトリの所有者（チーム）という責任分界とも一致します。

Separating shared from local content is basic configuration management. Copying shared conventions into 120 repositories means synchronizing 120 places on every change, and omissions are certain. Referencing one maintained source makes the update a single edit while teams focus on what is genuinely theirs — and the structure matches the ownership split between the convention owner and the repository owner.

- **A 不正解**: 周知に任せると反映率が上がらず、どのリポジトリが最新かも把握できません。 / Adoption stalls and compliance is unknowable.
- **B 不正解**: 一括置換はリポジトリごとの記述の違いにより失敗しやすく、ローカルな内容を壊す危険があります。 / Fragile across divergent files and risks destroying local content.
- **C 不正解**: 共通規約のみに統一すると、リポジトリ固有の有用な情報（アーキテクチャ、タスク手順）が失われます。 / Discards the repository-specific content that makes the file useful.

**参照 / Reference:** CLAUDE.md の階層管理、共通部分と個別部分の分離、参照による構成
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

あるチームが、リリース作業を支援する優れたスキル（手順書、チェック項目、必要なツール呼び出しをまとめたもの）を作成しました。他の 8 チームも同じリリース手順を使っており、このスキルを利用したいと考えています。現在は、各チームがファイルをコピーして自分のリポジトリに置く方法しかありません。元のスキルが改善されても、コピーには反映されません。

A team has built a strong skill for release work — procedure, checklist, and the tool calls involved. Eight other teams follow the same release process and want to use it. Today the only option is copying the files into each team's repository, so improvements to the original never reach the copies.

**設問 / Question:**

最も適切な配布方法はどれですか？

Which distribution approach is most appropriate?

- A) 各チームがコピーを維持し、必要に応じて手動で同期する / Have each team maintain a copy and sync manually when needed
- B) **共有可能な形式でパッケージ化し、バージョン管理された配布経路を用意する**。スキルを所有チームが管理する単一の場所に置き、各チームはバージョンを指定して取り込む。改善は所有チームが行い、利用チームはバージョンを上げることで取り込む。破壊的な変更はバージョンで区別し、利用状況を把握できるようにする。全社的に有用なスキルを登録・発見できる仕組みも用意する / **Package it for sharing and provide a versioned distribution path.** Keep the skill in one location owned by the authoring team, and have consuming teams pull it at a specified version; improvements are made once and adopted by bumping the version, with breaking changes distinguished by version and usage visible. Provide a way to register and discover skills that are broadly useful
- C) スキルの内容を文書化して、各チームが自分で実装する / Document the skill's content and have each team implement their own
- D) 元のチームが 8 チーム分のスキルを個別に保守する / Have the authoring team maintain eight separate copies

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**再利用されるべき資産には、バージョン管理された配布経路が必要**です。コピーによる共有は、改善が伝播せず、時間とともに 9 つの異なる亜種に分かれていきます。所有チームが 1 か所で管理し、利用チームがバージョンを指定して取り込む形にすれば、改善が届き、破壊的変更も制御できます。加えて、**発見できる仕組み**が重要で、有用なスキルの存在を知らなければ各チームが同じものを作り直します。

Assets meant for reuse need a versioned distribution path. Copying does not propagate improvements and diverges into nine variants over time. One owned location with version-pinned consumption delivers improvements and controls breaking changes — and discoverability matters too, because teams that do not know a skill exists will rebuild it.

- **A 不正解**: 手動同期は行われなくなり、コピーが分岐します。現状と実質的に変わりません。 / Manual sync stops happening; the copies diverge.
- **C 不正解**: 各チームによる再実装は、労力の重複と品質のばらつきを生みます。 / Duplicates effort and produces inconsistent quality.
- **D 不正解**: 8 つの個別保守は所有チームの負担を 8 倍にし、持続しません。 / Multiplies the owner's burden and is unsustainable.

**参照 / Reference:** スキルの共有と配布、バージョン管理、発見可能性
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

CI パイプラインに Claude Code を組み込んで、プルリクエストの自動レビューを実施したいと考えています。CI 環境は非対話型で、実行結果を後続のステップで機械的に処理する必要があります。また、CI 上のエージェントに与える権限は最小限にしたいと考えています（リポジトリの読み取りとコメント投稿のみ、書き込みや外部への通信は不可）。

You want to run Claude Code in CI for automated pull-request review. The CI environment is non-interactive and the result must be processed programmatically by later steps. You also want the CI agent's permissions minimized: repository read and comment posting only, with no writes and no external network access.

**設問 / Question:**

最も適切な構成はどれですか？

Which configuration is most appropriate?

- A) **非対話実行と機械可読な出力、および権限の明示的な制限を組み合わせる**。対話を伴わない実行モードで起動し、出力を構造化された形式（後続ステップがパースできる形）で受け取る。利用可能なツールを必要なものだけに明示的に限定し、それ以外は許可しない設定にする。認証情報は CI のシークレット管理から注入し、実行環境は使い捨てにする。失敗時の終了コードを定義して、パイプラインが結果を判定できるようにする / **Combine non-interactive execution, machine-readable output, and explicit permission limits**: run in a non-interactive mode, take output in a structured format later steps can parse, restrict the available tools explicitly to only those required and disallow the rest, inject credentials from the CI secret store, use a disposable execution environment, and define exit codes so the pipeline can act on the result
- B) 対話モードで起動し、応答を自動入力するスクリプトを書く / Launch interactively and script the responses
- C) すべてのツールを許可して、必要な操作が失敗しないようにする / Allow all tools so nothing fails for lack of permission
- D) 出力を自由記述のテキストで受け取り、正規表現で解析する / Take free-form text output and parse it with regular expressions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

CI での実行には 3 つの要件があります。**非対話**（人間の応答を待たない）、**機械可読な出力**（後続ステップが判定できる）、**最小権限**（CI 環境は認証情報を持つため、権限が広いと影響が大きい）。加えて、実行環境を使い捨てにすることで、実行間の汚染を防ぎます。終了コードの定義は、パイプラインが結果に応じて分岐するために必要です。これらは個別の工夫ではなく、CI 統合の標準的な構成要素です。

CI execution has three requirements: non-interactive operation, machine-readable output so later steps can act, and least privilege — a CI environment holds credentials, so a broad grant has wide consequences. A disposable environment prevents cross-run contamination, and defined exit codes let the pipeline branch on the outcome. These are the standard elements of CI integration, not ad hoc tweaks.

- **B 不正解**: 対話モードへの自動入力は脆く、出力形式の変化で容易に壊れます。 / Scripted interaction is brittle and breaks on any output change.
- **C 不正解**: CI 環境は認証情報を持つため、全ツール許可は爆発半径を最大化します。 / Maximizes blast radius in a credential-holding environment.
- **D 不正解**: 自由記述の正規表現解析は、表現の揺れで壊れます。構造化出力を使うべき場面です。 / Regex over free text breaks on phrasing variation.

**参照 / Reference:** CI での非対話実行、構造化出力、ツール権限の限定、使い捨て実行環境
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

全社で Claude Code を導入して 6 か月が経ちました。経営層から「実際に使われているのか」「効果は出ているのか」と問われましたが、把握できているのはライセンス数（400）だけです。誰がどの程度使っているか、どのような用途で使われているか、どのチームで定着しているかが分かりません。個々の開発者の詳細な操作を監視することには、現場から強い抵抗があります。

Six months after rolling out Claude Code across the company, leadership asks whether it is actually used and whether it is producing results. The only figure available is the license count (400). Nobody knows who uses it, how much, for what, or which teams have adopted it. Detailed monitoring of individual developers' actions faces strong resistance from engineers.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 個々の開発者の操作を詳細に記録して分析する / Record and analyze each developer's actions in detail
- B) ライセンス数を利用実績として報告する / Report license count as usage
- C) **集計レベルの利用テレメトリと、開発者からの定性的な情報を組み合わせる**。個人を特定しない粒度（チーム単位、用途カテゴリ単位）で利用頻度と規模を集計し、定着しているチームとそうでないチームの差を把握する。あわせて、開発者へのアンケートやインタビューで、どの作業に有効か・どこで使いにくいかを収集する。何を測り何を測らないかを事前に開発者に開示し、目的（個人評価ではなく導入改善）を明示する / **Combine aggregate usage telemetry with qualitative input from developers**: measure frequency and volume at a non-identifying granularity (per team, per use-case category) to see which teams have adopted it and which have not, and gather from surveys and interviews where it helps and where it gets in the way. Disclose in advance what is and is not measured, and state the purpose — improving the rollout, not evaluating individuals
- D) 利用状況の把握は諦め、定性的な感想のみを報告する / Give up on usage data and report impressions only

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**開発者の抵抗は、監視そのものより「個人評価に使われること」への懸念**であることが多く、集計レベルの計測であればこの懸念は大きく下がります。チーム単位・用途カテゴリ単位で見れば、「どこで定着しているか」という導入改善に必要な情報は十分得られます。定性的な情報を組み合わせるのは、数値だけでは「なぜ使われていないか」が分からないためです。**何を測り何を測らないかを事前に開示する**ことが、信頼を保ちながら計測を成立させる条件です。

Developer resistance is usually less about measurement than about being individually evaluated, and aggregate measurement largely removes that concern while still answering the rollout question: which teams adopted it and which did not. Qualitative input is needed because numbers do not explain non-adoption. Disclosing in advance what is and is not measured is the condition for measuring at all without losing trust.

- **A 不正解**: 個人の詳細監視は現場の信頼を損ない、抵抗を強めます。導入改善には集計で十分です。 / Damages trust for information aggregates already provide.
- **B 不正解**: ライセンス数は利用実績ではありません。配布と利用は別です。 / Licenses distributed is not usage.
- **D 不正解**: 集計レベルの計測は実施可能であり、諦める理由がありません。 / Aggregate measurement is achievable.

**参照 / Reference:** 利用テレメトリ、集計粒度とプライバシー、定量と定性の組み合わせ、計測目的の開示
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

社内の AI 利用コストが月 1,200 万円に達しました。内訳は、本番システムが 700 万円、開発者の Claude Code 利用が 400 万円、実験・検証が 100 万円です。財務からは「部門別の負担を明確にしてほしい」と求められています。現在は情報システム部門の予算から一括で支払っており、各部門はコストを意識していません。一部の部門では明らかに非効率な使い方（巨大なコンテキストの反復投入）が見られます。

Internal AI spend has reached ¥12M/month: ¥7M production, ¥4M developer Claude Code usage, ¥1M experimentation. Finance wants per-department cost visibility. Everything is currently paid from the IT budget and no department sees the cost. Some departments show clearly inefficient usage patterns, such as repeatedly submitting enormous contexts.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 各部門に一律の上限を設定する / Set a uniform cap per department
- B) 情報システム部門の予算のまま、非効率な部門に個別に注意する / Keep central funding and warn the inefficient departments individually
- C) 開発者の Claude Code 利用を停止する / Stop developer Claude Code usage
- D) **コストを部門・用途別に可視化し、責任の所在と改善の動機を揃える**。すべての利用にコスト配賦の識別子（部門、システム、用途）を付与して集計し、部門別のダッシュボードで見えるようにする。非効率な使い方については、単に費用を付け替えるだけでなく、なぜそうなっているか（プロンプトキャッシュの未使用、不要なコンテキストの投入）を分析して改善策を提示する。予算の帰属は財務と合意して決め、コスト意識と改善支援を同時に成立させる / **Make cost visible by department and use case, aligning accountability with the incentive to improve.** Tag every call with attribution identifiers (department, system, use case), aggregate them, and expose per-department dashboards. For inefficient usage, do more than reassign the charge — analyze why it happens (caching unused, unnecessary context) and propose fixes. Agree budget ownership with finance so cost awareness and improvement support arrive together

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

コストを一括で負担する構造では、**使う側にコストを意識する動機がありません。**部門別に可視化すると、非効率な使い方が当事者に見えるようになり、改善の動機が生まれます。ただし、可視化だけでは改善方法が分からないため、**なぜ非効率なのかを分析して具体的な改善策を提示する**ことがセットで必要です（プロンプトキャッシュの活用、不要なコンテキストの削減）。責任と支援を同時に提供することが、コスト配賦を機能させる条件です。

Central funding removes any incentive for users to consider cost. Per-department visibility puts inefficiency in front of the people who can change it — but visibility alone does not tell them how, so pairing it with analysis and concrete remedies (enable caching, trim context) is what produces improvement. Accountability plus support is what makes chargeback work.

- **A 不正解**: 一律上限は部門ごとの正当な利用量の違いを無視し、必要な利用まで止めます。 / Ignores legitimate differences in need.
- **B 不正解**: 個別の注意は非効率の原因に対処せず、他部門への波及もありません。 / Does not address causes and does not generalize.
- **C 不正解**: 開発者利用の停止は、生産性向上という目的そのものを捨てます。 / Discards the productivity benefit entirely.

**参照 / Reference:** コスト配賦、部門別可視化、非効率の原因分析、責任と支援の両立
</details>

---

## 問題 6 / Question 6

**シナリオ / Scenario:**

社内の 12 チームが、それぞれ独自にエージェントアプリケーションを構築しています。各チームが、認証、ログ、評価、監視、デプロイの仕組みを個別に実装しており、品質にばらつきがあります。あるチームは評価を全く行っておらず、別のチームは監視がありません。一方、先進的なチームは優れた仕組みを持っています。プラットフォームチームから「標準化すべきか」という相談を受けました。

Twelve teams each build agent applications independently, implementing authentication, logging, evaluation, monitoring, and deployment separately with uneven quality. One team does no evaluation at all; another has no monitoring. Meanwhile, the most advanced teams have built excellent tooling. The platform team asks whether to standardize.

**設問 / Question:**

最も適切なアプローチはどれですか？

Which approach is most appropriate?

- A) **ゴールデンパス（推奨される標準経路）を提供し、採用を選択制にする**。認証、ログ、評価、監視、デプロイを含む共通基盤を、そのまま使えば品質基準を満たす形で提供する。先進的なチームの実装を出発点として取り込み、実際に有用なものにする。強制はせず、標準経路を使う方が楽であることで採用を促す。標準から外れる場合は、その理由と代替の統制を示すことを求める。品質基準（評価と監視の有無など）は標準経路とは別に、全チームへの要求として定める / **Provide a golden path and make adoption optional but attractive.** Offer a shared foundation covering authentication, logging, evaluation, monitoring, and deployment that meets the quality bar out of the box, seeded from what the advanced teams already built so it is genuinely useful. Do not mandate it; drive adoption by making the standard path the easier one, and where a team departs from it, ask them to state why and what alternative controls apply. Define the quality requirements themselves (evaluation and monitoring must exist) separately, as an expectation of every team
- B) 全チームに同一の実装を強制する / Mandate an identical implementation for all teams
- C) 各チームの自由に任せ、標準化は行わない / Leave every team free and standardize nothing
- D) 最も進んだチームの実装をそのまま他 11 チームにコピーさせる / Copy the most advanced team's implementation into the other eleven

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**ゴールデンパスは、強制ではなく「標準経路を使う方が楽である」ことで採用を得る**手法です。強制すると、事情の異なるチームが無理に合わせるか、形だけ従って実質を伴わなくなります。一方、完全な自由放任では、評価も監視もないチームが残ります。この 2 つを両立させるのが、「品質要求は全チームに課し、それを満たす最も簡単な手段としてゴールデンパスを提供する」という構成です。先進チームの実装を出発点にすることで、実際に使えるものになり、既存の投資も活かせます。

A golden path earns adoption by being the easier route, not by mandate: mandates force ill-fitting teams to comply nominally. Pure autonomy, though, leaves teams with no evaluation and no monitoring. The combination that works sets the quality requirements for everyone and offers the golden path as the easiest way to meet them — seeded from the advanced teams' work so it is real and their investment is not discarded.

- **B 不正解**: 強制は事情の異なるチームに合わない構成を押し付け、形式的な準拠を招きます。 / Imposes a poor fit and produces nominal compliance.
- **C 不正解**: 自由放任は、評価も監視もないチームを放置することになります。 / Leaves teams without evaluation or monitoring.
- **D 不正解**: コピーは改善が伝播せず、11 チームの文脈に合うとも限りません。 / Improvements do not propagate and the fit is not guaranteed.

**参照 / Reference:** ゴールデンパス、内部プラットフォーム、品質要求と標準経路の分離
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

新しく入社した 15 名のエンジニアに、社内のエージェント開発ワークフローを習得してもらう必要があります。現在の導入は「ドキュメントのリンク集を渡す」だけで、3 か月経っても実際に使いこなせているのは 4 名です。他の 11 名は「何から始めればよいか分からない」「試したが期待どおりに動かず、そのままになっている」と述べています。

Fifteen new engineers need to learn the company's agent-development workflow. Onboarding today consists of handing over a list of documentation links; three months in, only four are working effectively. The other eleven report not knowing where to start, or having tried something that did not work as expected and left it there.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate approach?

- A) ドキュメントをより詳細に書き直す / Rewrite the documentation in more detail
- B) 全員に 1 日の座学研修を実施する / Run a one-day classroom session for everyone
- C) **実際の業務課題を題材にした段階的な習得経路を用意する**。小さく確実に成功する最初の課題（既存のスキルを使って実業務を 1 つ片付ける）から始め、段階的に難易度を上げる。各段階には期待される成果と、つまずいたときの相談先を明示する。よくあるつまずき（期待どおり動かない場面）については、原因と対処を事前に共有する。習得状況を追跡し、停滞している人に個別に働きかける / **Provide a staged learning path built on real work**: start with a first task that succeeds reliably and small (use an existing skill to complete one real piece of work), then increase difficulty in stages, each with a stated expected outcome and a named person to ask when stuck. Share the common failure modes — the "it didn't work as expected" moments — with their causes and fixes up front. Track progress and reach out individually to anyone who stalls
- D) 使いこなせている 4 名に、他の 11 名を教えるよう依頼する / Ask the four who succeeded to teach the other eleven

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

「リンク集を渡す」方式が失敗する理由は、**最初の一歩が定義されていないこと**と、**つまずいたときに止まってしまうこと**の 2 点です。11 名の証言はそのまま両方を指しています。有効な習得経路は、確実に成功する小さな課題から始めて成功体験を作り、段階的に難易度を上げるものです。とりわけ重要なのが、**よくあるつまずきを事前に共有すること**で、これがないと「動かない」時点で多くの人が離脱します。相談先の明示と停滞者への働きかけが、離脱を防ぎます。

Handing over links fails for two reasons the eleven describe directly: no defined first step, and nothing to do when stuck. An effective path starts with a small task that reliably succeeds, builds from there, and — critically — shares the common failure modes in advance, because "it didn't work" is where most people drop out. A named person to ask and outreach to stalled learners close the remaining gap.

- **A 不正解**: 詳細化しても、最初の一歩とつまずき時の支援という 2 つの原因は解消しません。 / More detail addresses neither cause.
- **B 不正解**: 座学のみでは実践での定着につながらず、つまずき時の支援もありません。 / Classroom time alone does not translate into practice.
- **D 不正解**: 4 名に負担を集中させる方式は持続せず、教える内容も体系化されません。 / Concentrates the burden and produces unsystematic teaching.

**参照 / Reference:** 段階的な習得経路、初回の成功体験、よくあるつまずきの共有、停滞者への働きかけ
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

開発者が Claude Code を使う際、ローカル環境には本番データベースへの接続情報、クラウドの管理者権限、社内 API のトークンが設定されています。開発者は「自分の作業に必要だから」と説明しています。エージェントはこの環境で任意のコマンドを実行できます。あるとき、エージェントが意図せず本番データベースに対する操作を実行しかけた事例が報告されました。

Developers run Claude Code in local environments configured with production database credentials, cloud administrator rights, and internal API tokens, explaining that their work requires them. The agent can run arbitrary commands in that environment. An incident was reported in which the agent nearly executed an operation against the production database.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 開発者に注意を促し、慎重に使うよう指導する / Advise developers to be careful
- B) **開発者の作業環境と、危険な操作が可能な環境を分離する**。日常の開発作業には本番権限を必要としないため、既定の環境からは本番の認証情報を外し、開発・検証環境の権限のみを持たせる。本番に対する操作が必要な場合は、都度昇格する経路（申請、期間限定の権限、操作の記録）を用意する。あわせて、エージェントが実行できるコマンドの範囲を制限し、破壊的な操作には確認を挟む設定を標準として配布する / **Separate the everyday development environment from one where dangerous operations are possible.** Day-to-day work does not need production rights, so remove production credentials from the default environment and grant only development and staging access. Where production operations are genuinely needed, provide a per-occasion elevation path (request, time-limited grant, recorded actions). Additionally restrict the commands the agent may run and distribute a standard configuration that requires confirmation for destructive operations
- C) Claude Code の利用を禁止する / Prohibit Claude Code
- D) エージェントに「本番環境を操作しないこと」と指示する / Instruct the agent not to touch production

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

問題の本質は、**日常の作業環境に本番権限が常時存在すること**です。エージェントの有無にかかわらず、これは権限管理上の欠陥であり、エージェントはその欠陥を顕在化させたにすぎません。対策は、既定の環境から本番権限を外し、必要時に昇格する経路を用意することです。加えて、エージェントが実行できるコマンドの制限と破壊的操作への確認を標準設定として配布することで、個々の開発者の設定に依存せずに保護が働きます。

The core problem is that production rights sit permanently in the everyday environment — a privilege-management defect independent of agents, which merely exposed it. The fix removes production credentials from the default and provides per-occasion elevation. Restricting the commands the agent may run and requiring confirmation for destructive operations, shipped as a standard configuration, makes the protection independent of each developer's setup.

- **A 不正解**: 注意喚起は、権限が常時存在するという構造的な問題を解決しません。 / Does not change the standing privilege.
- **C 不正解**: 禁止は生産性の利得を捨て、権限管理の問題も解決しません。 / Forfeits the benefit and leaves the privilege problem.
- **D 不正解**: プロンプトによる制限は確率的で、不可逆な本番操作に対する統制になりません。 / A probabilistic control over irreversible production actions.

**参照 / Reference:** 開発環境の権限分離、都度昇格、コマンド範囲の制限、標準設定の配布
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

本番のエージェントシステムについて、24 時間のオンコール体制を敷くことになりました。オンコール担当者は、AI システムを開発していないインフラ運用チームのメンバーです。夜間に「エージェントの応答品質が低下している」というアラートが鳴った場合、担当者が何をすべきかが定義されていません。過去の夜間障害では、担当者が判断できず開発者を起こす結果になりました。

Twenty-four-hour on-call is being established for a production agent system. The on-call staff are from the infrastructure operations team and did not build it. There is no definition of what they should do when a "response quality degraded" alert fires overnight. In past night incidents, on-call could not decide and ended up waking a developer.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 夜間のアラートは翌朝まで対応しないことにする / Defer overnight alerts to the following morning
- B) すべての夜間障害で開発者を起こす運用にする / Wake a developer for every overnight incident
- C) アラートの閾値を上げて、夜間に鳴らないようにする / Raise the thresholds so alerts do not fire at night
- D) **アラートごとにランブックを整備し、判断と対処を担当者が実行できるようにする**。各アラートについて、何を意味するか、まず確認すべき指標、取り得る対処（縮退への切り替え、機能停止、再起動）とその判断基準、対処してよい範囲、開発者を呼ぶべき条件を記述する。縮退や停止のような重い操作についても、実行してよい条件と手順を明示する。ランブックは実際の障害を経て更新し、訓練で実行可能性を確認する / **Write a runbook per alert so on-call can decide and act**: what the alert means, the metrics to check first, the available actions (switch to degraded mode, disable the feature, restart) with the criteria for each, the boundary of what they may do unaided, and the conditions for escalating to a developer. Include the conditions and steps for heavy actions such as degradation and shutdown, update the runbook after real incidents, and rehearse to confirm it is executable

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

オンコール担当者が判断できない原因は、**アラートの意味と取り得る対処が定義されていないこと**です。ランブックの要点は、単に手順を書くことではなく、**判断基準と権限の範囲を明示すること**にあります。「縮退に切り替えてよいのはどういう場合か」「どこまでは自分で対処し、どこから開発者を呼ぶか」が定まっていないと、担当者は動けません。実際の障害を経て更新することと、訓練で実行可能性を確認することが、ランブックを実用的なものにします。

On-call cannot decide because neither the alert's meaning nor the available actions are defined. A runbook's value is less in the steps than in stating the decision criteria and the boundary of authority: without knowing when switching to degraded mode is permitted, or where their remit ends and escalation begins, on-call is immobilized. Updating after real incidents and rehearsing is what keeps it usable.

- **A 不正解**: 夜間に品質が低下したまま朝まで運用するのは、顧客影響を放置する対応です。 / Leaves customers exposed until morning.
- **B 不正解**: 全件呼び出しは開発者の疲弊を招き、オンコール体制を敷いた意味がありません。 / Burns out developers and defeats the purpose of on-call.
- **C 不正解**: 閾値を上げるのは、検知すべき問題を検知しない設定にする対応です。 / Configures the system not to detect what it should.

**参照 / Reference:** ランブック、判断基準と権限範囲の明示、インシデント後の更新、訓練
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

Claude Code 導入の効果を測定するよう求められました。あるチームが「コミット数が 40% 増えた」というデータを提出しました。別のチームは「体感で 2 倍速くなった」と報告しています。経営層はこれらを根拠に全社展開の拡大を検討していますが、あなたはこれらの指標が生産性を正しく表しているか疑問を持っています。

You are asked to measure the impact of Claude Code. One team submits "commits increased 40%." Another reports "it feels twice as fast." Leadership is considering expanding the rollout on this basis, and you doubt these indicators represent productivity.

**設問 / Question:**

最も適切な測定の考え方はどれですか？

Which measurement approach is most appropriate?

- A) **成果に近い指標を複数組み合わせ、単一の代理指標に依存しない**。コミット数のような活動量の指標は、成果ではなく作業量を測っており、細かいコミットに分割するだけで増やせる。より成果に近い指標（変更のリードタイム、変更失敗率、レビューの手戻り、機能提供までの期間）と、開発者の主観的な評価（どの作業で有効か、どこで負担が増えたか）を組み合わせて見る。あわせて、品質面の指標（不具合率、レビュー指摘数）が悪化していないかを併せて確認する / **Combine several outcome-proximate indicators rather than relying on a single proxy.** Activity counts such as commits measure work performed, not results, and can be inflated by splitting commits. Look instead at indicators closer to outcomes — change lead time, change failure rate, review rework, time to deliver a feature — alongside developers' own assessment of where it helps and where it adds burden, and check in parallel that quality indicators (defect rate, review findings) have not worsened
- B) コミット数を全社の標準指標として採用する / Adopt commit count as the company-wide standard metric
- C) 体感による報告を集計して効果とする / Aggregate subjective reports as the measure of impact
- D) 効果測定は困難なので行わない / Skip measurement as too difficult

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**活動量の指標（コミット数）は、成果と混同されやすい典型例**です。指標として採用すると、コミットを細かく分割するといった行動を誘発し、実質的な改善なしに数字だけが上がります。生産性の測定では、成果に近い指標（リードタイム、変更失敗率）を複数組み合わせ、**品質が犠牲になっていないかを同時に確認する**必要があります。開発者の主観的な評価も、どの作業で有効かという定性的な情報として価値があります。単一指標への依存が、測定を歪める最大の要因です。

Activity counts are the classic proxy mistaken for outcome: adopted as a target, they induce commit-splitting and rise without real improvement. Productivity measurement needs several outcome-proximate indicators together — lead time, change failure rate — and a parallel check that quality has not been traded away. Developers' own assessment adds qualitative signal about where it helps. Dependence on a single metric is the main source of distortion.

- **B 不正解**: コミット数を目標にすると、指標を上げる行動が誘発され、意味を失います。 / Targeting commits induces gaming and destroys the signal.
- **C 不正解**: 体感のみでは、期待や新規性による偏りを排除できません。 / Subjective reports carry expectancy and novelty bias.
- **D 不正解**: 完全な測定は困難でも、成果に近い指標の組み合わせで有用な把握は可能です。 / Imperfect measurement is still informative.

**参照 / Reference:** 生産性指標、活動量と成果の区別、複数指標の組み合わせ、品質との同時確認
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

社内で MCP サーバーが乱立しています。調査すると、41 個の MCP サーバーが稼働しており、そのうち 7 個は同じ社内 API に接続する重複したもの、12 個は作成者が異動して所有者不明、9 個は 6 か月以上呼び出されていません。新しくエージェントを作るチームは、既存のサーバーを見つけられず、また 1 つ作ることになります。

Internal MCP servers have proliferated: 41 are running, of which 7 duplicate each other by connecting to the same internal API, 12 have no owner after their authors transferred, and 9 have not been called in over six months. Teams building new agents cannot find the existing servers and add one more.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) すべての MCP サーバーを停止し、必要なものだけ作り直す / Shut down all of them and rebuild only what is needed
- B) 新規の MCP サーバー作成を全面的に禁止する / Prohibit new MCP servers entirely
- C) **登録簿（カタログ）を整備し、所有者・利用状況・棚卸しの運用を確立する**。全サーバーを登録して、接続先・提供する機能・所有チーム・利用元を記録し、検索できるようにする。所有者不明のものは所属を確定するか廃止し、6 か月未使用のものは利用者に確認したうえで廃止する。重複しているものは統合先を決めて移行する。新規作成時にはカタログの確認を手順に組み込み、既存のもので足りる場合は再利用させる / **Establish a catalog with ownership, usage data, and a recurring review.** Register every server with its backend, the capabilities it provides, its owning team, and its consumers, and make it searchable. Assign owners to the orphaned ones or retire them, confirm with consumers before retiring the unused ones, and pick a survivor among the duplicates and migrate. Make catalog lookup a step in creating a new server so existing ones get reused where they suffice
- D) 41 個すべてをプラットフォームチームが引き取って保守する / Have the platform team take over maintenance of all 41

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

乱立の原因は、**既存のものを発見できないこと**と**棚卸しの仕組みがないこと**の 2 点です。カタログを整備すると、新規作成前に既存の再利用を検討でき、重複の発生自体が減ります。同時に、所有者不明と長期未使用のサーバーは、セキュリティ上のリスク（更新されない、認証情報を保持している）でもあるため、棚卸しで整理する必要があります。**新規作成時にカタログ確認を手順に組み込む**ことが、再発防止の要点です。

Proliferation has two causes: existing servers cannot be found, and nothing retires anything. A catalog enables reuse before creation, reducing duplication at the source, while orphaned and long-unused servers are also a security concern — unpatched, holding credentials — and need clearing. Making catalog lookup a required step in creation is what prevents recurrence.

- **A 不正解**: 全停止は稼働中の業務を止めます。利用されているサーバーもあります。 / Stops live business; many are in use.
- **B 不正解**: 全面禁止は正当な新規需要を止め、統制外での実装を誘発します。 / Blocks legitimate need and drives implementations underground.
- **D 不正解**: 41 個の集中保守はプラットフォームチームの能力を超え、所有と責任も曖昧になります。 / Exceeds capacity and blurs ownership.

**参照 / Reference:** MCP サーバーの登録簿、所有者の明確化、棚卸し、再利用の促進
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

開発者が Claude Code のエージェントに調査を依頼したところ、エージェントがリポジトリ内の `.env` ファイルを読み込み、その内容（API キーを含む）を要約の一部として出力しました。この出力はターミナルに表示され、開発者がその内容をチケットに貼り付けたため、社内のチケットシステムに認証情報が記録されました。同様の構成のリポジトリが多数存在します。

A developer asked a Claude Code agent to investigate something; the agent read a `.env` file in the repository and included its contents — API keys among them — in its summary. The output appeared in the terminal, and the developer pasted it into a ticket, recording credentials in the internal ticketing system. Many repositories have the same setup.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 開発者に、出力を貼り付ける前に確認するよう指導する / Advise developers to check output before pasting it
- B) **露出した認証情報を無効化したうえで、構造的な対策を講じる**。まずチケットに記録されたキーを無効化・再発行し、チケットの記録も削除する。そのうえで、エージェントが読み取れる範囲から機微ファイルを除外する設定（`.env` や鍵ファイルへのアクセス制限）を標準として全リポジトリに配布し、ローカルの認証情報はファイルではなくシークレット管理から供給する構成に移す。あわせて、チケットやリポジトリに対する秘密情報の検出を継続的に実施する / **Revoke the exposed credentials, then address it structurally.** First invalidate and reissue the keys recorded in the ticket and purge the record. Then distribute a standard configuration excluding sensitive files from what the agent may read (`.env`, key files) across all repositories, and move local credentials out of files into a secret store. Additionally run continuous secret detection against tickets and repositories
- C) `.env` ファイルの名前を変更して、エージェントに見つからないようにする / Rename `.env` files so the agent does not find them
- D) エージェントに「機微なファイルを読まないこと」と指示する / Instruct the agent not to read sensitive files

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

対応は 2 層で、**露出への即時対処**（無効化と再発行、記録の削除）と**構造的な対策**です。構造面では、エージェントの読み取り範囲から機微ファイルを除外する設定を標準配布し、さらに一段深く、**認証情報をファイルとして置かない構成**（シークレット管理からの供給）へ移すのが根本的です。同様の構成が多数のリポジトリに存在する以上、個別対応ではなく標準設定の配布が必要です。秘密情報の継続的な検出は、すり抜けた場合の検出層になります。

The response has two layers: immediate handling of the exposure (revoke, reissue, purge the record) and a structural fix. Structurally, ship a standard configuration excluding sensitive files from the agent's reach, and go one level deeper by removing credentials from files entirely in favor of a secret store. Since many repositories share the setup, distribution of a standard is required rather than per-case fixes, with continuous secret detection as the backstop.

- **A 不正解**: 人的な注意は再発を防げず、既に露出したキーへの対処もありません。 / Does not prevent recurrence or address the live exposure.
- **C 不正解**: ファイル名の変更は隠蔽にすぎず、認証情報がファイルに平文で存在する問題は残ります。 / Obscurity, with plaintext credentials still on disk.
- **D 不正解**: プロンプトによる制限は確率的で、標準設定として全リポジトリに適用することもできません。 / Probabilistic and not distributable as a standard.

**参照 / Reference:** 開発環境の秘密情報管理、読み取り範囲の制限、標準設定の配布、秘密情報の検出
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

12 チームがそれぞれ独自に評価の仕組みを作っています。あるチームは Python スクリプト、別のチームは表計算ソフト、別のチームは目視確認です。指標の定義もばらばらで（あるチームの「精度」は完全一致、別のチームは人手判定）、チーム間で品質を比較できません。経営層から「全社の AI システムの品質水準を把握したい」という要求がありました。

Twelve teams each built their own evaluation tooling: a Python script here, a spreadsheet there, manual inspection elsewhere. Metric definitions differ too — one team's "accuracy" is exact match, another's is human judgment — so quality cannot be compared across teams. Leadership wants a view of AI system quality across the company.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 各チームの数値をそのまま集計して全社の平均を出す / Average the teams' numbers as reported
- B) 全チームに同一の指標定義と同一の閾値を強制する / Mandate identical metric definitions and thresholds for all teams
- C) 各チームに現在の数値を報告させ、比較はしない / Collect each team's number and make no comparisons
- D) **共通の評価基盤と指標の定義を提供し、チーム固有の指標は上に載せる形にする**。全社で比較可能にすべき指標（何をもって正解とするかの定義を含む）を定め、それを測る共通の仕組みを提供する。各システムの用途に固有の指標は、共通指標に加えてチームが定義する。共通指標があることで、全社の品質水準の把握とチーム間の学習が可能になり、チーム固有の指標によって各システムの実態も測れる / **Provide shared evaluation tooling and metric definitions, with team-specific metrics layered on top.** Define the metrics that must be comparable company-wide, including what counts as correct, and supply common tooling to measure them; teams then add metrics specific to their use case. The shared layer makes an organizational view and cross-team learning possible, while the team layer keeps each system honestly measured

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**定義が異なる数値を集計しても意味を持ちません。**完全一致で測った 90% と人手判定で測った 90% は、まったく異なる実態を指します。共通の指標定義と評価基盤を提供すれば、全社での比較とチーム間の学習が可能になります。ただし、共通指標だけではシステム固有の重要な品質次元が測れないため、**チーム固有の指標を上に載せる**構成が必要です。共通化すべきものと各チームに委ねるものを分けるのが要点です。

Aggregating numbers with different definitions is meaningless: 90% by exact match and 90% by human judgment describe different realities. Shared definitions and tooling make comparison and cross-team learning possible — but a shared layer alone cannot capture each system's important quality dimensions, so team-specific metrics layer on top. The discipline is separating what must be common from what belongs to each team.

- **A 不正解**: 定義の異なる数値の平均は、解釈できない数字になります。 / Averaging incomparable numbers produces an uninterpretable figure.
- **B 不正解**: 閾値まで統一すると、リスクも用途も異なるシステムに同じ基準を課すことになります。 / Uniform thresholds ignore differences in risk and use case.
- **C 不正解**: 比較しないままでは、経営層の要求（全社の品質水準の把握）に応えられません。 / Does not answer the question asked.

**参照 / Reference:** 評価基盤の共通化、指標定義の統一、共通指標とチーム固有指標の分離
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

エージェントが生成したコードのプルリクエストが増えています。レビュアーからは「量が多くて追いつかない」「一見正しそうだが、細部に問題があることがある」「テストは通るが、意図と違う実装になっていることがある」という声が上がっています。現在、エージェント生成のコードと人間が書いたコードは区別なくレビューされています。

Pull requests containing agent-generated code are increasing. Reviewers report being unable to keep up with the volume, that the code looks right but has problems in the details, and that it passes tests while implementing something other than what was intended. Agent-generated and human-written code are currently reviewed identically.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **レビューの負荷と焦点を、生成コードの性質に合わせて調整する**。機械的に検証できる部分（形式、静的解析、テストカバレッジ、既存 API の誤用）は自動チェックで先に潰し、レビュアーは「意図どおりか」「設計として妥当か」という機械では判定しにくい観点に集中する。プルリクエストには、何を意図した変更かと、どう検証したかを記述することを求める。変更の粒度が大きすぎる場合は分割を求め、レビュー可能な単位に保つ / **Adjust review load and focus to the nature of generated code.** Push what can be checked mechanically — formatting, static analysis, test coverage, misuse of existing APIs — into automated gates so reviewers concentrate on what machines cannot judge: whether it does what was intended and whether the design is sound. Require each pull request to state the intent of the change and how it was verified, and ask for splits when a change is too large to review meaningfully
- B) エージェント生成のコードはレビューを省略する / Skip review for agent-generated code
- C) エージェントによるコード生成を禁止する / Prohibit agent code generation
- D) レビュアーの人数を増やして量に対応する / Add reviewers to absorb the volume

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

レビュアーの 3 つの声は、それぞれ異なる対策を要します。**量**は自動チェックによる負荷軽減と変更の分割で、**細部の問題**は静的解析とテストで、**意図との乖離**は意図の明示とレビュアーの焦点の絞り込みで対処します。とりわけ「テストは通るが意図と違う」という指摘は重要で、これは機械的に検出できないため、**プルリクエストに意図を記述させる**ことが直接的な対策になります。レビューの総量を減らすのではなく、人間が見るべき部分に集中させるのが方針です。

The three complaints need different remedies: volume is addressed by automated gates and smaller changes, detail problems by static analysis and tests, and intent divergence by stating intent and refocusing reviewers. The last is the important one — "passes tests but implements something else" is not machine-detectable, so requiring the intent in the pull request is the direct fix. The aim is not less review but review concentrated where humans are needed.

- **B 不正解**: 生成コードこそ「意図と違う実装」が起きやすく、レビュー省略は最も危険な対応です。 / Generated code is where intent divergence occurs; skipping review is the worst option.
- **C 不正解**: 生産性の利得を捨てる対応で、レビュー体制の調整で対処可能です。 / Discards the benefit for a problem the review process can absorb.
- **D 不正解**: 人数を増やしても、細部の問題や意図との乖離という質の問題は解決しません。 / More reviewers do not address the quality issues.

**参照 / Reference:** 生成コードのレビュー、自動チェックとの役割分担、意図の明示、変更の分割
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

チーム内で頻繁に行う作業（新規マイクロサービスの雛形作成）を、Claude Code で効率化したいと考えています。この作業は、決まった手順（ディレクトリ構成の作成、設定ファイルの生成、CI 設定の追加、README の作成）からなり、いくつかのパラメータ（サービス名、所属ドメイン、依存関係）で変わります。チーム 8 名が週に数回実行します。実行時には、開発者が対話的に確認しながら進めたいと考えています。

Your team frequently scaffolds new microservices and wants to streamline it with Claude Code. The task follows a fixed procedure (directory structure, configuration files, CI setup, README) parameterized by service name, domain, and dependencies. Eight team members run it several times a week and want to work through it interactively, confirming as they go.

**設問 / Question:**

最も適切な実装方法はどれですか？

Which implementation is most appropriate?

- A) 各開発者が毎回、必要な手順を自分でプロンプトに書く / Have each developer write the steps in a prompt each time
- B) 完全に自動化されたシェルスクリプトに置き換える / Replace it with a fully automated shell script
- C) **チームで共有するスラッシュコマンドとして実装する**。決まった手順とパラメータの受け取り方を定義し、リポジトリに置いてチーム全員が同じ手順を呼び出せるようにする。対話的に確認しながら進めたいという要件に合致し、手順の改善は 1 か所の更新でチーム全体に反映される。パラメータで変わる部分は引数として受け取り、手順そのものはコマンドの定義に持たせる / **Implement it as a team-shared slash command**: define the fixed procedure and how parameters are supplied, keep it in the repository so everyone invokes the same procedure, and improve it in one place for the whole team. This fits the requirement to work through it interactively, with the varying parts taken as arguments and the procedure itself held in the command definition
- D) サブエージェントとして実装し、独立したコンテキストで実行させる / Implement it as a subagent running in an isolated context

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**繰り返し実行される定型的な手順を、チームで共有する形にまとめる**用途に適合します。要件は 3 つあり、(1) 手順が決まっている、(2) パラメータで変わる、(3) 対話的に確認しながら進めたい。共有されたコマンドとして定義すれば、全員が同じ手順を呼び出せ、改善も 1 か所で済みます。完全自動化のスクリプトは (3) の要件に合わず、また判断が必要な場面（既存サービスとの命名衝突、依存の選択）に対応しにくくなります。

This fits the case of a repeated, parameterized procedure shared across a team. Three requirements apply: a fixed procedure, parameter variation, and interactive confirmation. A shared command definition gives everyone the same procedure and one place to improve it. A fully automated script conflicts with the third requirement and handles poorly the points where judgment is needed, such as naming collisions or dependency choices.

- **A 不正解**: 毎回書くのは手順のばらつきを生み、改善も共有されません。 / Produces variation and shares no improvements.
- **B 不正解**: 完全自動化は対話的に確認したいという要件に反し、判断が必要な場面に対応できません。 / Conflicts with the interactive requirement and the judgment points.
- **D 不正解**: サブエージェントはコンテキスト分離が必要な独立したタスクに向いており、開発者が確認しながら進める定型手順には過剰です。 / Subagents suit isolated tasks needing context separation, not an interactive checklist.

**参照 / Reference:** スラッシュコマンドの適合条件、スキル・サブエージェントとの使い分け、チーム共有
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

エージェントが生成したコードに対して「テストカバレッジ 80% 以上」というマージ条件を設けました。導入後、カバレッジは常に 80% を超えるようになりましたが、本番の不具合率は改善していません。生成されたテストを確認すると、多くが「関数を呼び出すが、結果を検証していない」「例外が出ないことだけを確認している」というもので、カバレッジは上がるが欠陥を検出しない構造でした。

A merge gate requiring 80% test coverage was added for agent-generated code. Coverage now always exceeds 80%, but the production defect rate has not improved. Inspecting the generated tests shows many that call a function without asserting on the result, or assert only that no exception is raised — raising coverage without detecting defects.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) カバレッジの基準を 95% に引き上げる / Raise the coverage bar to 95%
- B) **カバレッジを唯一のゲートにするのをやめ、テストの有効性を測る仕組みを加える**。カバレッジは「実行された行」を測るだけで「検証されたか」は測らないため、単独の基準としては操作可能である。アサーションの存在と質を確認する静的チェック、意図的に欠陥を注入して検出できるかを見る手法（ミューテーションテスト）、レビューでのテスト内容の確認を組み合わせる。あわせて、何をテストすべきかをプロンプトやテンプレートで具体的に示す / **Stop using coverage as the sole gate and add measures of test effectiveness.** Coverage measures lines executed, not whether anything was verified, so alone it is gameable. Combine static checks for the presence and quality of assertions, a technique that injects defects deliberately and checks whether tests catch them (mutation testing), and review attention to test content — and specify concretely, in prompts or templates, what should be tested
- C) テストの自動生成をやめて、人間が書く / Stop generating tests and have humans write them
- D) カバレッジのゲートを撤廃する / Remove the coverage gate

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**カバレッジは「実行された行」の指標であり、「検証されたか」の指標ではありません。**これを唯一のゲートにすると、実行するだけのテストで基準を満たせてしまい、本問で観測されているとおり欠陥検出力は上がりません。有効な対策は、アサーションの有無と質を見る仕組みと、**意図的に欠陥を注入して検出できるかを確認する手法**を組み合わせることです。加えて、何をテストすべきかを具体的に指示すれば、生成されるテストの質そのものが上がります。

Coverage measures lines executed, not properties verified. As a sole gate it is satisfied by tests that merely execute code, which is exactly what happened. Effective measures check that assertions exist and are meaningful, and verify detection power directly by injecting defects. Specifying concretely what should be tested also improves the generated tests at the source.

- **A 不正解**: 基準を上げても、検証しないテストが増えるだけで欠陥検出力は上がりません。 / A higher bar produces more non-verifying tests.
- **C 不正解**: 人間が書けば質は上がり得ますが、量に対応できず、生成の利得も失います。 / Improves quality at the cost of throughput and the benefit.
- **D 不正解**: 撤廃すると、テストが全く書かれない状態に戻ります。 / Reverts to no tests at all.

**参照 / Reference:** カバレッジの限界、テスト有効性の測定、ミューテーションテスト、指標の操作可能性
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

開発者がエージェントに「このバグを直して」と依頼したところ、エージェントが修正を行い、そのままメインブランチにコミット・プッシュしました。修正内容は別の機能を壊すもので、CI が失敗する前に他の開発者 6 名がその変更を取り込み、それぞれの作業環境が壊れました。エージェントには Git 操作を含む広い権限が与えられていました。

A developer asked an agent to fix a bug; the agent made a change and committed and pushed it directly to the main branch. The change broke another feature, and before CI failed six other developers had pulled it and broken their working environments. The agent had broad permissions including Git operations.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) エージェントに「メインブランチには直接プッシュしないこと」と指示する / Instruct the agent not to push directly to main
- B) 開発者に、エージェントの操作を都度確認するよう指導する / Advise developers to confirm each agent action
- C) エージェントから Git 操作の権限をすべて外す / Remove all Git permissions from the agent
- D) **開発ワークフロー側のガードレールで、直接プッシュを構造的に不可能にする**。メインブランチを保護してレビューと CI 通過を必須にし、エージェントであれ人間であれ直接プッシュできない状態にする。エージェントの作業はブランチとプルリクエストを経由させ、CI が通るまでマージされないようにする。あわせて、エージェントに与える Git 操作の範囲をブランチ操作までに限定する。この統制は人間の誤操作にも同じく有効である / **Make direct pushes structurally impossible via workflow guardrails.** Protect the main branch to require review and passing CI so neither an agent nor a human can push to it directly, route the agent's work through a branch and pull request that cannot merge until CI passes, and limit the agent's Git permissions to branch-level operations. The same control protects against human mistakes equally

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**エージェントが直接プッシュできてしまう構成は、エージェント特有の問題ではなく、ワークフローの統制不備**です。ブランチ保護と CI 必須化があれば、この事故は構造的に起こり得ません。しかもこの統制は人間の誤操作にも同じく有効で、既に多くの組織で標準的な実践です。エージェントの導入は、既存のガードレールの不足を顕在化させたと捉えるのが適切です。プロンプトによる指示や人間の注意に頼る対策は、確率的でスケールしません。

An agent able to push directly is a workflow control gap, not an agent-specific problem: branch protection with required CI makes the incident structurally impossible, and the same control protects against human error — it is standard practice already. The agent's arrival simply exposed a missing guardrail. Prompt instructions and human vigilance are probabilistic and do not scale.

- **A 不正解**: プロンプトによる制限は確率的で、統制になりません。 / Probabilistic; not a control.
- **B 不正解**: 都度確認は人間の注意に依存し、疲労や見落としで破られます。 / Depends on attention and fails under fatigue.
- **C 不正解**: Git 操作を全面的に外すと、ブランチ作成やコミットといった正当な作業もできなくなり、有用性が大きく下がります。 / Removes legitimate branch and commit work too.

**参照 / Reference:** 開発ワークフローのガードレール、ブランチ保護、CI 必須化、権限の範囲
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

エンジニア以外の部門（経理、法務、人事）から「自分たちの業務でもエージェントを作りたい」という要望が増えています。これらの部門はプログラミングの経験がありませんが、業務知識は深く、どこを自動化すべきかを最もよく知っています。一方、無統制に構築されると、権限設定の誤りやデータの取り扱いに問題が生じる懸念があります。

Non-engineering functions (accounting, legal, HR) increasingly want to build their own agents. They have no programming experience but deep domain knowledge and know best what should be automated. Uncontrolled building, however, risks misconfigured permissions and mishandled data.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate approach?

- A) **安全な範囲があらかじめ定められた構築環境を提供する**。利用できるデータ範囲とツールが統制済みの環境を用意し、その中であれば業務部門が自由に構築できるようにする。機微データへのアクセスや外部への書き込みを伴う用途は、この環境の外として通常の承認プロセスに回す。あわせて、業務部門向けの研修（できることと、してはいけないこと、相談すべき場面）を用意し、構築されたものを定期的にレビューする経路を持つ / **Provide a build environment with safe boundaries built in.** Offer an environment where the accessible data and available tools are already governed, and let business functions build freely inside it. Uses that need sensitive data or external writes fall outside it and go through the normal approval process. Pair this with training for business users (what they can do, what they must not, when to ask) and a path for periodic review of what they build
- B) エンジニア以外による構築を禁止する / Prohibit building by non-engineers
- C) 要望をすべて情報システム部門が引き取って実装する / Have IT implement every request
- D) 制限を設けず、各部門の判断で自由に構築させる / Impose no limits and let each function build freely

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

業務部門は**どこを自動化すべきかを最もよく知っている**という強みを持ち、これを活かさない手はありません。同時に、権限設定やデータ取り扱いの判断は専門知識を要します。この 2 つを両立させるのが「安全な範囲を先に定めた環境」で、その中では自由に構築でき、範囲外の用途だけが通常の承認に回ります。**禁止すると需要はシャドー AI に向かう**ため、統制を働かせるうえでも安全な選択肢を提供する方が有効です。研修と定期レビューが、環境内の品質を保ちます。

Business functions know best what to automate, and that knowledge should be used. Permission and data-handling judgments, however, need expertise. A pre-bounded environment reconciles the two: free building inside, normal approval only for uses outside. Prohibition sends the demand into shadow AI, so offering a safe option is also the better control. Training and periodic review maintain quality within the boundary.

- **B 不正解**: 禁止は業務知識を活かす機会を捨て、需要をシャドー AI に向かわせます。 / Discards the domain knowledge and drives shadow AI.
- **C 不正解**: 全件を情報システム部門が引き取るのは能力的に成立せず、業務知識の伝達にも損失が生じます。 / Not feasible, and loses domain knowledge in translation.
- **D 不正解**: 無統制では、権限誤設定やデータ取り扱いの問題が現実に生じます。 / The stated risks materialize.

**参照 / Reference:** 統制された構築環境、業務部門のイネーブルメント、範囲外の承認経路、定期レビュー
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

あるリポジトリの `CLAUDE.md` は 4,000 行あり、コーディング規約、過去の議論の記録、個人の好み、既に廃止された仕組みの説明が混在しています。開発者からは「エージェントが古い方針に従って実装してしまう」「関係ない指示に引っ張られる」という報告があります。一方、別のリポジトリの `CLAUDE.md` は 3 行しかなく、エージェントがプロジェクトの構造を理解できずに毎回探索から始めています。

One repository's `CLAUDE.md` runs to 4,000 lines, mixing coding conventions, records of past discussions, personal preferences, and descriptions of retired mechanisms. Developers report the agent following obsolete policies and being pulled toward irrelevant instructions. Another repository's `CLAUDE.md` has three lines, and the agent cannot grasp the project structure, starting from exploration every time.

**設問 / Question:**

最も適切な指針はどれですか？

Which guidance is most appropriate?

- A) すべてのリポジトリで `CLAUDE.md` を 500 行前後に統一する / Standardize every repository's file at around 500 lines
- B) 4,000 行の方を正として、他のリポジトリにも同様に詳細に書かせる / Take the 4,000-line file as the model and expand the others
- C) **「エージェントが正しく作業するために必要な情報」という基準で内容を選別する**。プロジェクトの構造、使うべき／使ってはいけない仕組み、よくあるタスクの手順、注意すべき制約といった、作業の質を左右する情報を書く。過去の議論の記録や廃止済みの説明は、誤った実装を誘発するため削除するか別の場所に移す。3 行の方には、探索を減らす基本情報（構造、主要な入口、実行方法）を追加する。分量ではなく、記述が実際に作業の質を上げているかで判断する / **Select content by the test of what the agent needs to work correctly**: project structure, mechanisms to use and to avoid, procedures for common tasks, and constraints that matter. Records of past discussions and descriptions of retired mechanisms actively induce wrong implementations and should be deleted or moved elsewhere. For the three-line file, add the basics that reduce exploration (structure, main entry points, how to run things). Judge by whether the content improves the work, not by length
- D) `CLAUDE.md` を廃止し、すべてをコード内のコメントに移す / Abolish `CLAUDE.md` and move everything into code comments

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

`CLAUDE.md` の基準は**分量ではなく、記述がエージェントの作業の質を上げるか**です。4,000 行の問題は長さそのものではなく、**廃止済みの説明が誤った実装を誘発している**ことと、無関係な内容が指示を希釈していることです。3 行の問題は、探索に無駄なコストがかかっていることです。どちらも「必要な情報が過不足なくあるか」という同じ基準で判断でき、一律の行数を定めることには意味がありません。廃止済みの記述の削除は、とりわけ効果が大きい改善です。

The standard for `CLAUDE.md` is whether its content improves the work, not its length. The 4,000-line problem is not size but that obsolete descriptions induce wrong implementations while unrelated content dilutes the rest; the three-line problem is wasted exploration. Both are judged by the same test, and a uniform line count means nothing. Removing descriptions of retired mechanisms is the highest-value single fix.

- **A 不正解**: 行数の統一は、内容の質と無関係な基準です。 / Line count is unrelated to content quality.
- **B 不正解**: 4,000 行の方は既に問題を起こしており、模範にはなりません。 / The 4,000-line file is the one causing problems.
- **D 不正解**: コード内コメントは局所的な情報には向きますが、プロジェクト全体の方針や手順を伝える手段としては不十分です。 / Comments suit local detail, not project-level policy and procedure.

**参照 / Reference:** CLAUDE.md の内容基準、廃止済み記述の害、探索コストの削減
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

チーム内で、開発者ごとに異なるバージョンのツールとスキルを使っています。ある開発者の環境では動くコマンドが、別の開発者の環境では動きません。また、あるスキルの挙動が開発者によって異なり、「自分の環境では正しく動く」という応酬が発生しています。CI 環境のバージョンもローカルと異なり、ローカルで通ったものが CI で失敗します。

Within a team, developers run different versions of tools and skills. A command that works in one developer's environment fails in another's, a skill behaves differently per developer, and arguments about "it works on my machine" follow. The CI environment's versions differ from local ones too, so what passes locally fails in CI.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 各開発者が最新版に更新するよう周知する / Ask everyone to update to the latest
- B) **バージョンをリポジトリで固定し、ローカルと CI で同一にする**。使用するツールとスキルのバージョンを設定としてリポジトリに記述し、各環境がその指定に従うようにする。CI もローカルも同じ指定を参照することで、環境差による挙動の違いをなくす。更新はリポジトリの変更として行い、変更の影響を CI で検証してからチーム全体に反映する。これにより「自分の環境では動く」問題が構造的に解消される / **Pin versions in the repository and make local and CI identical.** Declare the tool and skill versions as repository configuration that every environment follows, so CI and local resolve to the same thing and behavioral differences from environment drift disappear. Upgrades happen as repository changes, verified by CI before reaching the team — which structurally eliminates "works on my machine"
- C) 動かない場合は動く開発者に代わりに実行してもらう / Have whoever it works for run it instead
- D) CI 環境をローカルに合わせて、開発者ごとに用意する / Give each developer a CI environment matching their local setup

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

「自分の環境では動く」問題の原因は、**環境ごとにバージョンが異なること**です。バージョンをリポジトリで固定して全環境が同じ指定を参照すれば、この問題は構造的に消えます。CI とローカルの一致は特に重要で、これがないとローカルでの検証が CI の結果を予測できません。更新をリポジトリの変更として行うことで、更新の影響を CI で検証してからチームに反映でき、更新そのものも管理された作業になります。

"Works on my machine" is caused by per-environment version drift, and pinning versions in the repository so every environment resolves the same way removes it structurally. Local/CI parity matters most: without it, local verification does not predict CI. Making upgrades repository changes means their effect is verified by CI before reaching the team, so upgrading becomes a managed activity too.

- **A 不正解**: 周知に依存する更新は徹底されず、更新のタイミングもずれます。 / Advisory updates are neither complete nor synchronized.
- **C 不正解**: 特定の開発者への依存を生み、根本原因も残ります。 / Creates a dependency on one person and fixes nothing.
- **D 不正解**: CI をローカルに合わせると、環境ごとに CI が分裂し、検証の意味が失われます。 / Fragments CI and destroys its value as a common check.

**参照 / Reference:** バージョン固定、ローカルと CI の一致、管理された更新
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

組織全体の API レート制限を、本番システムと開発者の Claude Code 利用が共有しています。ある日、複数の開発者が同時に大規模なリファクタリング作業を実行したところ、レート制限を消費し尽くし、本番の顧客向けエージェントが応答不能になりました。開発作業自体は正当なもので、業務時間内に行われたものです。

Production systems and developers' Claude Code usage share the organization-wide API rate limit. One day several developers ran large refactoring tasks simultaneously, exhausted the limit, and the production customer-facing agent stopped responding. The development work was legitimate and done during business hours.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) 開発者に、大規模な作業は業務時間外に行うよう依頼する / Ask developers to run large tasks outside business hours
- B) 開発者の利用に一律の上限を設ける / Impose a uniform cap on developer usage
- C) 本番システムのリトライを強化する / Strengthen retries in the production systems
- D) **本番と開発の容量を分離し、本番に優先権を与える**。組織のレート制限を用途別に配分し、本番向けには予約容量を確保して開発作業が侵食できないようにする。開発向けは残余容量で動かし、上限に近づいた場合は開発側が待機または低優先度で処理されるようにする。可能であれば、契約上または構成上、本番と開発を別の枠として分離する。あわせて、消費状況を可視化して開発者が影響を認識できるようにする / **Separate production and development capacity and give production priority.** Allocate the organization's limit by purpose, reserving capacity for production that development cannot encroach on, and run development on the remainder so it queues or degrades to low priority near the ceiling. Where possible, separate production and development into distinct allocations contractually or structurally, and make consumption visible so developers can see their impact

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**顧客向けの本番システムと開発作業が同じ容量を奪い合う構成は、優先順位が設計に反映されていない**ことを意味します。正当な開発作業が本番を止めてしまうのは、開発者の問題ではなく容量配分の問題です。本番に予約容量を確保すれば、開発作業がどれだけ集中しても本番の可用性が守られます。開発側は残余容量で待機や低優先度処理になり、影響を受けるのは開発の待ち時間だけになります。可視化は、開発者が自分の消費を認識するための補助的な仕組みです。

Production and development competing for the same capacity means priority is not expressed in the design. Legitimate development work stopping production is a capacity-allocation problem, not a developer problem. Reserved production capacity protects availability regardless of development load, with development queuing on the remainder so the only casualty is developer wait time. Visibility helps developers see their own consumption.

- **A 不正解**: 運用上の依頼は徹底されず、本番停止のリスクが残ります。開発の生産性も下がります。 / Not enforceable, leaves the risk, and slows development.
- **B 不正解**: 一律上限は正当な大規模作業を阻害し、本番への予約がなければ保護にもなりません。 / Blocks legitimate work without protecting production.
- **C 不正解**: リトライは容量が枯渇している状況では効かず、むしろ消費を増やします。 / Ineffective when capacity is exhausted, and adds load.

**参照 / Reference:** 容量の分離、本番への予約容量、優先度制御、消費の可視化
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

内部の AI プラットフォームを提供して 4 か月が経ちました。利用チームからの質問がチャットで直接プラットフォームチームのメンバー個人に届き、1 日 30 件を超えています。同じ質問が繰り返し来ることも多く、メンバーは開発時間の 6 割を回答に費やしています。回答は個人のチャットに散在し、蓄積されていません。

Four months into providing an internal AI platform, questions from consuming teams arrive as direct chat messages to individual platform-team members, exceeding 30 per day. Many repeat, members spend 60% of their time answering, and the answers scatter across private chats without accumulating anywhere.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **支援の窓口と知識の蓄積を仕組みとして整える**。質問の受付を個人宛から公開のチャネルに移し、回答が他の利用者にも見える状態にする。繰り返される質問はドキュメントまたは FAQ に反映し、回答時にはその参照先を示す。当番制で一次対応を持ち回り、開発時間を確保する。質問の内容を分類して、頻出する領域についてはプラットフォーム側の改善（分かりにくい部分の修正、既定値の変更）につなげる / **Establish a support channel and a way for knowledge to accumulate.** Move intake from individual DMs to a public channel so answers are visible to other consumers, fold recurring questions into documentation or an FAQ and answer by pointing to it, rotate first-line duty so development time is protected, and categorize questions so recurring areas drive platform improvements (clarifying confusing parts, changing defaults)
- B) 質問への回答をやめ、ドキュメントを読むよう案内する / Stop answering and direct people to the documentation
- C) プラットフォームチームの人員を増やして対応する / Add headcount to absorb the volume
- D) 質問の多いチームには個別に担当者を割り当てる / Assign a dedicated person to the teams that ask most

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

問題は質問の量そのものより、**回答が蓄積されず、同じ質問が繰り返されること**です。公開チャネルへの移行だけで、他の利用者が同じ質問をする前に答えを見つけられるようになります。当番制は個人への集中を防ぎ、開発時間を確保します。最も重要なのが、**質問の分類をプラットフォームの改善につなげる**ことで、頻出する質問は多くの場合「プラットフォームが分かりにくい」ことの証拠です。質問に答え続けるのではなく、質問が発生しない状態を目指すのが本質的な対策です。

The problem is less the volume than that answers do not accumulate, so questions repeat. Moving to a public channel alone lets others find the answer before asking. Rotation prevents concentration and protects development time. Most importantly, categorizing questions feeds platform improvement: recurring questions are usually evidence that something is confusing. The goal is not answering faster but making the question unnecessary.

- **B 不正解**: 回答を止めるとプラットフォームの利用が停滞し、シャドー実装を招きます。 / Stalls adoption and drives shadow implementations.
- **C 不正解**: 人員増は同じ質問の繰り返しという構造を残したまま、コストだけを増やします。 / Adds cost without addressing repetition.
- **D 不正解**: 個別担当は蓄積を生まず、担当者が不在の場合に対応できません。 / No accumulation, and a single point of failure.

**参照 / Reference:** 内部プラットフォームの支援体制、公開チャネル、知識の蓄積、質問の改善への還流
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

開発環境ではエージェントが期待どおりに動作するのに、本番では異なる挙動を示すという報告が繰り返されています。調査すると、開発環境と本番でモデルのバージョンが異なり、ツールの一部がモックに置き換えられ、コンテキストに入る社内文書のインデックスも別物でした。開発者は「ローカルで確認した」という前提で変更をリリースしています。

Reports recur that the agent behaves as expected in development but differently in production. Investigation shows the environments use different model versions, some tools are replaced by mocks in development, and the internal-document index differs. Developers ship changes on the premise that "it was verified locally."

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 本番でのみ検証を行うことにする / Verify only in production
- B) 開発者に、本番との差異を意識するよう指導する / Advise developers to be mindful of the differences
- C) **環境間の差異を減らし、残る差異を明示する**。モデルバージョンは開発と本番で同一に固定する。ツールのモックは、挙動の差が問題になる箇所については実物に近い振る舞いをする代替に置き換えるか、本番相当のツールに接続できる検証環境を用意する。文書インデックスは、本番の構造とデータ分布を反映したものにする。それでも残る差異（データ量、実トラフィック）は文書化し、その差異が影響する変更については本番相当環境での検証を必須にする / **Reduce the differences and document what remains.** Pin the same model version in development and production; replace mocks with substitutes that behave realistically where the difference matters, or provide a staging environment connected to production-equivalent tools; and make the document index reflect production's structure and data distribution. Document the differences that remain (data volume, real traffic), and require verification in a production-equivalent environment for changes those differences affect

- D) 環境差異の一覧を作成して開発者に配布し、確認するかどうかは各自の判断に任せる / Publish a list of the environment differences to developers and leave it to each of them whether to verify

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**「ローカルで確認した」が意味を持つのは、ローカルが本番を十分に近似している場合だけ**です。モデルバージョン、ツールの実装、参照データのすべてが異なる環境での確認は、本番の挙動をほとんど予測しません。対策は、差異を減らすこと（特にモデルバージョンは容易に揃えられます）と、**残る差異を明示して、その差異が影響する変更には本番相当環境での検証を求めること**です。すべての差異をなくすことは現実的でないため、どの差異が何に影響するかを把握することが要点になります。

"Verified locally" means something only when local approximates production. Verification in an environment differing in model version, tool implementations, and reference data predicts almost nothing. The remedy is reducing the differences — model version is easily aligned — and documenting those that remain, with staging verification required for changes those differences affect. Eliminating every difference is unrealistic, so knowing which difference affects what is the real objective.

- **A 不正解**: 本番のみでの検証は、顧客に影響する形で問題を発見することになります。 / Discovers problems in front of customers.
- **B 不正解**: 意識するよう求めても、差異が何に影響するかを判断する材料がありません。 / Mindfulness without knowing which differences matter changes nothing.
- **D 不正解**: 一覧を配布しても確認が任意であれば実施されず、差異が影響する変更でも検証が省かれます。 / An optional list goes unread, and verification is skipped precisely where it matters.

**参照 / Reference:** 環境間の差異、本番相当環境、残存差異の文書化
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

あるチームで、エージェントが外部から取得したコンテンツに含まれる指示に従ってしまうインシデントが発生しました。原因分析と対策は当該チーム内で完了しています。社内には同様の構成（外部コンテンツをコンテキストに含める）のエージェントが他に 14 あり、いずれも同じ脆弱性を持つ可能性があります。現在、インシデントの記録はそのチームのドキュメントにのみ存在します。

A team suffered an incident in which the agent followed instructions embedded in externally retrieved content. Root-cause analysis and remediation are complete within that team. Fourteen other agents in the company share the same pattern of ingesting external content and may carry the same vulnerability. The incident record exists only in that team's documentation.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 当該チームの対策で十分とし、他チームには共有しない / Treat the team's fix as sufficient and not share it
- B) 全チームに向けてインシデントの概要をメールで送る / Email a summary of the incident to all teams
- C) 他の 14 チームに、自分たちで同様の問題がないか確認するよう依頼する / Ask the other fourteen teams to check themselves
- D) **インシデントの学びを組織的に展開し、同種の構成に対して確認と対策を行う**。原因と対策を全社で参照できる形にまとめ、同じ構成を持つ 14 のエージェントを特定して、それぞれについて同じ脆弱性の有無を確認する。対策を標準（外部コンテンツの扱い、権限の最小化、不可逆操作の保護）として文書化し、以後の新規開発でも適用されるようにする。確認結果を追跡し、未対応のものが残らないようにする / **Propagate the learning organizationally and verify the same pattern elsewhere.** Publish the cause and remediation for company-wide reference, identify the fourteen agents sharing the pattern, and check each for the same vulnerability. Codify the remediation as a standard (handling of external content, minimized permissions, protection of irreversible operations) so future development inherits it, and track the checks so nothing is left unaddressed

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**1 チームで起きたインシデントは、同じ構成を持つ他のシステムでも起き得ます。**14 の同種エージェントが存在すると分かっている以上、それぞれについて確認するのが当然の対応です。要点は 3 つで、(1) 学びを全社で参照できる形にすること、(2) 同種構成を特定して個別に確認すること（依頼ではなく追跡すること）、(3) 対策を標準として文書化して新規開発に適用することです。追跡がないと、確認が行われないまま忘れられます。

An incident in one team can occur in every system sharing its pattern, and with fourteen such agents known, checking each is the obvious response. Three elements matter: publishing the learning for company-wide reference, identifying and verifying each instance (tracked, not merely requested), and codifying the remediation as a standard so new development inherits it. Without tracking, the checks quietly do not happen.

- **A 不正解**: 同じ脆弱性が 14 のシステムに残る可能性を放置します。 / Leaves the same vulnerability potentially live in fourteen systems.
- **B 不正解**: メール共有は認知に留まり、実際の確認と対策が行われる保証がありません。 / Awareness without verification or remediation.
- **C 不正解**: 依頼だけでは実施状況が把握できず、未対応が残ります。 / Unverified requests leave gaps.

**参照 / Reference:** インシデントの水平展開、同種構成の特定、標準への反映、実施の追跡
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

プラットフォームチームが、共通化すべき機能の範囲を検討しています。候補として、認証、ログ形式、評価基盤、監視、プロンプトのテンプレート、ドメイン固有のツール、UI コンポーネントが挙がっています。チーム内では「全部共通化すべき」という意見と「共通化しすぎると各チームが動けなくなる」という意見が対立しています。12 の利用チームは、業務領域も技術スタックも異なります。

The platform team is deciding what to centralize. Candidates include authentication, log format, evaluation tooling, monitoring, prompt templates, domain-specific tools, and UI components. Opinions split between centralizing everything and centralizing too much leaving teams unable to move. The twelve consuming teams differ in both business domain and technology stack.

**設問 / Question:**

最も適切な判断基準はどれですか？

Which decision criterion is most appropriate?

- A) **変化しにくく、全チームに共通で、誤ると影響が大きいものを共通化する**。認証、ログ形式、監視、評価基盤は、要件が全チームで共通で、各チームが独自に実装する利得がなく、誤ると統制上の問題を生むため共通化に適する。一方、プロンプトのテンプレートやドメイン固有のツールは、業務領域ごとに要件が異なり、変化も速いため各チームに委ねるべきである。判断軸を「共通性・変化の速さ・誤りの影響」として明示し、候補ごとに評価する / **Centralize what changes slowly, is common to all teams, and is costly to get wrong.** Authentication, log format, monitoring, and evaluation tooling meet all three: requirements are shared, per-team implementation adds nothing, and errors create control problems. Prompt templates and domain-specific tools differ by business area and change quickly, so they belong to the teams. State the axes — commonality, rate of change, cost of error — and evaluate each candidate against them
- B) すべての候補を共通化する / Centralize all the candidates
- C) 共通化は行わず、すべて各チームに委ねる / Centralize nothing and leave everything to teams
- D) 利用チームの多数決で決める / Decide by majority vote among consuming teams

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

共通化の判断は、**「共通性・変化の速さ・誤りの影響」という 3 つの軸**で行うと整理できます。認証やログ形式は、全チームで要件が同じで、変化が遅く、誤ると統制上の問題が生じるため、共通化の利得が明確です。逆に、プロンプトのテンプレートは業務領域ごとに要件が異なり、頻繁に改善されるため、共通化すると各チームの改善速度を落とします。「全部」か「なし」かの対立は、判断軸が明示されていないことから生じています。

Centralization decisions resolve on three axes: commonality, rate of change, and cost of error. Authentication and log format score on all three — same requirements everywhere, slow-changing, control problems when wrong — so the gain is clear. Prompt templates differ by domain and improve frequently, so centralizing them slows every team down. The all-or-nothing argument exists because the axes were never stated.

- **B 不正解**: ドメイン固有のツールまで共通化すると、各チームの業務要件に合わず、変更のたびにプラットフォームチームがボトルネックになります。 / Centralizing domain-specific tools makes the platform a bottleneck.
- **C 不正解**: 認証やログ形式を各チームが独自実装すると、統制上の問題と重複投資が生じます。 / Per-team authentication and logging create control problems and duplicated effort.
- **D 不正解**: 多数決は、影響の大きさや変化の速さといった判断すべき性質を反映しません。 / A vote does not reflect the properties that should decide it.

**参照 / Reference:** 共通化の判断軸、プラットフォームの範囲、チームの自律性とのバランス
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

社内のスキル共有の仕組みを整えたところ、6 か月で 140 個のスキルが登録されました。しかし品質にばらつきがあり、動作しないもの、説明が不正確で誤って使われるもの、他のスキルと重複するものが混在しています。利用者からは「どれを使えばよいか分からない」「使ってみたら動かなかった」という声が上がっています。登録は誰でも自由に行えます。

Six months after establishing internal skill sharing, 140 skills are registered. Quality varies: some do not work, some have inaccurate descriptions leading to misuse, and some duplicate each other. Users report not knowing which to use and finding that a skill did not work. Anyone can register a skill.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 登録を停止し、既存の 140 個を精査してから再開する / Suspend registration and review all 140 before reopening
- B) **貢献の仕組みに、品質を担保する要素を組み込む**。登録時に満たすべき要件（動作確認の記録、用途と適用範囲の記述、所有者の明示）を定め、それを満たさないものは登録できないようにする。既存のスキルには利用実績と評価を表示して、選択の手がかりを与える。長期間使われていないものや、動作しない報告があるものは所有者に確認したうえで整理する。重複しているものは統合先を決める。登録の自由度は保ちつつ、品質の可視化と最低要件で全体の質を上げる / **Build quality assurance into the contribution process.** Define registration requirements — evidence it works, a description of purpose and scope, a named owner — and block registration without them. Show usage and ratings on existing skills so users have a basis for choosing, and clear out the long-unused and the reported-broken after confirming with their owners. Pick a survivor among duplicates. Keep contribution open while raising overall quality through visible quality signals and a minimum bar
- C) プラットフォームチームが全スキルを審査してから登録を許可する / Have the platform team review every skill before registration
- D) スキル共有の仕組みを廃止する / Abolish skill sharing

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

140 個という数は、**仕組みが機能して貢献が集まっている証拠**でもあります。問題は貢献の量ではなく、品質を担保する要素が仕組みに組み込まれていないことです。登録時の最低要件（動作確認、用途の記述、所有者）は、貢献の障壁を大きく上げずに質の下限を引き上げます。利用実績と評価の表示は、利用者が選ぶための手がかりになり、質の高いものが自然に選ばれる循環を作ります。中央審査は、貢献の速度を落とし、審査側がボトルネックになります。

The 140 skills are also evidence the mechanism works and contributions are flowing. The problem is not volume but the absence of quality assurance in the process. Minimum registration requirements raise the floor without significantly raising the barrier to contribute, and visible usage and ratings give users a basis for choosing, creating a cycle where good skills are found. Central review would slow contribution and make the reviewers the bottleneck.

- **A 不正解**: 登録停止は貢献の流れを止め、140 個の一括精査も現実的ではありません。 / Halts contribution, and reviewing 140 at once is impractical.
- **C 不正解**: 全件の中央審査は、プラットフォームチームがボトルネックになり、貢献の速度を大きく落とします。 / Makes the platform team the bottleneck.
- **D 不正解**: 有用なスキルの共有という価値を捨てる対応です。 / Discards the value of sharing entirely.

**参照 / Reference:** 貢献モデルの設計、登録時の最低要件、品質の可視化、棚卸し
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

開発ツールの新しいバージョンがリリースされ、有用な機能が追加されました。しかし全社 400 名の環境を更新する手段がなく、各開発者が任意のタイミングで更新しています。3 か月後の調査では、最新版の利用者は 22%、1 年以上前のバージョンを使っている開発者が 18% いました。古いバージョンでは、新しく配布したスキルが動作しません。

A new version of the development tooling adds useful capabilities, but there is no way to update 400 environments and developers update whenever they choose. Three months later, 22% are on the latest version and 18% are on a version more than a year old. Newly distributed skills do not work on the old versions.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 最新版を使うよう繰り返し周知する / Repeatedly remind people to update
- B) 全開発者に強制的に最新版を配信する / Force the latest version onto everyone
- C) 古いバージョンでも動作するようスキルを作る / Author skills that work on the old versions too
- D) **更新を管理された運用として設計する**。サポートするバージョンの範囲（例: 最新から 2 世代）を定めて周知し、範囲外のバージョンでは動作を保証しないことを明示する。更新を容易にする手段（自動更新の仕組み、更新手順の簡素化、更新による変化の告知）を提供し、更新率を継続的に把握する。破壊的変更を含む更新については、移行期間と告知を伴う計画的な展開とする / **Make updating a managed process.** Define and publish a supported version range (for example, the latest two releases) and state that behavior outside it is not guaranteed. Provide the means to update easily (an automatic update mechanism, simplified steps, notice of what changes), and track the update rate continuously. For releases with breaking changes, roll out on a plan with a migration window and advance notice

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**更新が各自の判断に委ねられている構成では、バージョンの分散は必然です。**対策の中心は 3 つで、(1) サポート範囲を明示すること（無制限に古いバージョンを支える負担をなくす）、(2) 更新を容易にすること（自動更新、手順の簡素化）、(3) 更新率を把握すること（対策が効いているかを知る）。強制配信は、開発者の作業中の環境を壊す可能性があり、破壊的変更を含む場合はとりわけ問題になります。計画的な展開と移行期間が現実的な形です。

When updating is left to individual choice, version spread is inevitable. Three elements address it: a published support range that ends the burden of supporting arbitrarily old versions, mechanisms that make updating easy, and measurement of the update rate to know whether it is working. Forced deployment risks breaking a developer's in-flight work, especially where the release contains breaking changes — a planned rollout with a migration window is the workable form.

- **A 不正解**: 周知に依存する方法は 3 か月試して 22% という結果が出ています。 / Three months of this produced 22%.
- **B 不正解**: 強制配信は作業中の環境を壊す可能性があり、破壊的変更を含む場合は特に危険です。 / Risks breaking in-flight work, especially with breaking changes.
- **C 不正解**: 1 年以上前のバージョンまで対応し続けると、スキルの実装が複雑化し、新機能も使えません。 / Supporting year-old versions complicates every skill and blocks new capability.

**参照 / Reference:** サポートバージョン範囲、更新の容易化、更新率の把握、計画的な展開
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

エージェントが期待どおりに動作しなかったとき、開発者が原因を調べる手段がありません。「なぜこのツールを呼んだのか」「どのコンテキストを見て判断したのか」「どのプロンプトが適用されていたのか」を後から確認できず、再現も困難です。結果として、問題の報告が「なんか変な動きをした」という粒度に留まり、修正につながりません。

When an agent does not behave as expected, developers have no way to investigate. They cannot see afterwards why a tool was called, what context informed the decision, or which prompt version applied, and reproduction is difficult. As a result, problem reports stay at the level of "it did something odd" and do not lead to fixes.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **エージェントの実行を後から追跡・再現できる仕組みを提供する**。各実行について、適用されたプロンプトのバージョン、モデル識別子、コンテキストに含まれた内容、ツール呼び出しの系列とその引数・結果、最終出力を記録し、開発者が実行単位で参照できるようにする。可能であれば、記録した入力から同じ実行を再現できる手段を用意する。問題報告時にはこの記録の識別子を添えることを標準とし、報告の粒度を上げる / **Provide the ability to inspect and reproduce an execution after the fact.** For each run, record the prompt version applied, the model identifier, what entered the context, the sequence of tool calls with their arguments and results, and the final output, and let developers browse it per execution. Where possible, provide a way to replay a run from the recorded inputs. Make attaching the record's identifier the standard for problem reports, which raises the resolution of what gets reported
- B) 問題が起きたら、その場で開発者が手動で再現を試みる / Have developers try to reproduce manually when problems occur
- C) すべての実行のログ出力を増やす / Increase log verbosity across all executions
- D) 問題報告の書式を定めて、詳細な記述を求める / Define a report template requiring detailed descriptions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**報告が「なんか変」に留まる原因は、開発者に調べる手段がないこと**です。エージェントの挙動は、プロンプト・モデル・コンテキスト・ツール呼び出しの組み合わせで決まるため、これらを実行単位で記録しなければ原因を特定できません。再現手段があれば、修正の検証もできます。記録の識別子を報告に添える運用にすると、報告の粒度が自動的に上がり、開発者は推測ではなく事実から調査を始められます。これは AI システムのデバッグにおける基本的な基盤です。

Reports stay vague because developers have nothing to investigate with. Agent behavior is determined by the combination of prompt, model, context, and tool calls, so without recording those per execution the cause cannot be located — and replay additionally allows a fix to be verified. Making the record identifier part of every report raises reporting resolution automatically and lets investigation start from facts rather than guesses.

- **B 不正解**: 手動再現は、同じコンテキストと同じ条件を揃えられないため成功しにくいです。 / Manual reproduction rarely reconstructs the same context and conditions.
- **C 不正解**: ログ量を増やしても、実行単位で紐付いていなければ調査は困難なままです。 / Volume without per-execution correlation does not help.
- **D 不正解**: 書式を定めても、開発者が観測できない情報（適用プロンプト、コンテキスト）は書けません。 / A template cannot elicit information developers cannot observe.

**参照 / Reference:** 実行の記録と再現、エージェントのデバッグ基盤、報告の粒度向上
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

内部の AI プラットフォームを提供して 1 年が経ちました。12 チームのうち利用しているのは 5 チームで、残り 7 チームは独自に実装しています。プラットフォームチームは「機能は十分に揃っているのに使われない」と考えており、機能追加を続けています。利用していないチームに理由を聞いたことはありません。

One year in, only 5 of 12 teams use the internal AI platform; the other 7 build their own. The platform team believes the capabilities are sufficient and that teams simply do not use them, and continues adding features. No one has asked the non-adopting teams why.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 機能追加を続けて、魅力を高める / Keep adding features to increase appeal
- B) 全チームにプラットフォームの利用を義務化する / Mandate platform use for all teams
- C) **利用していない 7 チームに理由を聞き、採用の障害を特定する**。想定される理由は多岐にわたる（技術スタックが合わない、既存実装からの移行コストが高い、ドキュメントが不足している、必要な機能が欠けている、そもそも存在を知らない）。理由が分からないまま機能を追加しても、採用の障害が機能不足でなければ効果はない。障害を特定したうえで、移行支援、ドキュメント整備、機能追加のいずれが有効かを判断する。採用率を継続的に追跡する / **Ask the seven non-adopting teams why and identify the barriers.** The reasons could be many — a mismatched technology stack, high migration cost from an existing implementation, insufficient documentation, a missing capability, or simply not knowing it exists — and adding features cannot help if the barrier is not missing features. Identify the barrier first, then decide whether migration support, documentation, or new capability is the answer, and track adoption over time
- D) 利用していない 7 チームの実装を停止させる / Force the seven teams to stop their own implementations

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

「機能は十分なのに使われない」という認識は、**採用の障害を機能不足だと仮定している**点で危ういものです。実際には、移行コスト、ドキュメント不足、認知の欠如といった機能とは無関係な障害であることが多く、その場合は機能を追加しても採用率は上がりません。1 年間一度も理由を聞いていないという事実が、この問題の本質を示しています。障害を特定してから対策を選ぶという順序が、無駄な投資を避けます。

"The capabilities are sufficient but nobody uses it" assumes the barrier is missing capability. In practice the barrier is usually unrelated to features — migration cost, documentation, awareness — in which case adding features does not move adoption. That no one has asked in a year is the core of the problem. Identify the barrier, then choose the remedy.

- **A 不正解**: 障害が機能不足でない場合、機能追加は採用率に効きません。1 年間の実績がそれを示唆しています。 / If the barrier is not features, features do not help.
- **B 不正解**: 義務化は障害を解消せず、形式的な採用と不満を生みます。 / Mandates do not remove barriers.
- **D 不正解**: 既存実装の停止は、代替が使えるようになる前に業務を止めます。 / Stops work before a usable alternative exists.

**参照 / Reference:** 採用率の分析、障害の特定、プラットフォームの価値検証
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

半年前に全社で 2 日間の集中研修を実施し、エンジニア 400 名がエージェント開発の基礎を学びました。しかし現在、実際にエージェントを開発できているのは約 60 名です。研修後に新しく入社した 40 名は研修を受けておらず、ツールやプラクティスも半年で大きく変わりました。研修担当者からは「また大規模研修を実施すべきか」という相談がありました。

Six months ago, 400 engineers attended a two-day intensive training on agent development. Today about 60 can actually build agents. Forty engineers who joined since have had no training, and both the tooling and the practices have changed substantially in six months. The training owner asks whether to run another large session.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate approach?

- A) 同じ内容の集中研修を再度実施する / Run the same intensive session again
- B) **一度きりの研修ではなく、継続的に機能するイネーブルメントの仕組みを作る**。新規参加者がいつでも辿れる習得経路（実業務を題材にした段階的な課題）を用意し、入社時の導入に組み込む。ツールやプラクティスの変化は、更新される社内ドキュメントと変更の告知で追随させる。実務での定着を支えるため、質問できる場と、身近に相談できる推進役をチームごとに置く。習得状況と実際の構築数を追跡し、どこで止まっているかを把握して手を打つ / **Replace one-off training with continuous enablement.** Provide a self-serve learning path new joiners can start at any time (staged tasks based on real work) and fold it into onboarding. Keep pace with tooling and practice changes through maintained internal documentation and change announcements. Support adoption in daily work with a place to ask questions and a nominated champion in each team, and track both learning progress and how much is actually being built so you can see where people stall and intervene
- C) 研修は行わず、各自が自習することにする / Drop training and leave people to self-study
- D) 開発できている 60 名だけでエージェント開発を担当する / Concentrate all agent development in the 60 who can do it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

一度きりの集中研修が定着しにくいのは、**実務での実践が伴わないこと**と、**新規参加者と変化に追随できないこと**の 2 点によります。400 名中 60 名という結果はその帰結です。継続的な仕組みは、いつでも始められる習得経路（新規参加者に対応）、更新されるドキュメント（変化に対応）、質問できる場と身近な推進役（実務での定着を支援）から構成されます。習得状況と実際の構築数を追跡することで、どこで止まっているかを把握して手を打てます。

A one-off intensive does not stick for two reasons: it is not accompanied by practice in real work, and it cannot serve new joiners or keep pace with change. Sixty of 400 is the result. Continuous enablement combines a self-serve path available at any time, maintained documentation, and a place to ask plus a nearby champion supporting practice — with progress and actual output tracked so stalls are visible.

- **A 不正解**: 同じ方法を繰り返せば同じ結果になります。定着しなかった原因に対処していません。 / Repeating the method reproduces the result.
- **C 不正解**: 自習に委ねると、最初の一歩とつまずき時の支援が欠け、定着率はさらに下がります。 / Removes the first step and the support at the point of failure.
- **D 不正解**: 60 名への集中は、ボトルネックを作り、業務知識を持つチームが自分で作れる利点を捨てます。 / Creates a bottleneck and forfeits domain-embedded building.

**参照 / Reference:** 継続的なイネーブルメント、習得経路、推進役の配置、定着の追跡
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **前 / Previous:** [`domain6_stakeholder_lifecycle.md`](./domain6_stakeholder_lifecycle.md)
