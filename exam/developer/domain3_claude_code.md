# Domain 3: Claude Code / Claude Code

> 配点比率 / Weight: **3.1%**
> 問題数 / Questions: **7**（基礎 5 / 発展 2）
> 形式 / Format: 単一選択（選択肢 A–D）＋複数選択（選択肢 A–E、選ぶ数を明示）

## サブスキル / Sub-skills

| サブスキル / Sub-skill | Weight | 本ファイル |
|---|---|---|
| Claude Code Operation | 3.1% | 7 |

## 出題範囲 / Scope（公式ガイドの記述に基づく）

- **Claude Code Operation** — Claude Code の中核構成要素（Rules、Skills、Commands、Agents、Agent Memory）、機能（セッション管理、組み込みおよびカスタムのスラッシュコマンド、ヘッドレスモード、ストリーミングモード、auto モード）、`CLAUDE.md` の階層、リポジトリの初期化、`settings.json` の設定 / Claude Code core components (Rules, Skills, Commands, Agents, Agent Memory), features (session management, built-in and custom slash commands, headless mode, streaming mode, auto-mode), the `CLAUDE.md` hierarchy, repository initialization, and `settings.json` configuration

> **配点は 3.1%（本番 53 問中およそ 2 問）** と小さい領域ですが、Claude Code を日常的に使っていれば確実に得点できる範囲です。取りこぼさないようにしてください。
>
> At **3.1%** (roughly 2 of the 53 live items) this is a small domain, but it is reliably scoreable for anyone who uses Claude Code daily. Do not leave these points on the table.

---

## 基礎 / Foundations level

### 問題 1 / Question 1

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

モノレポで Claude Code を使っています。リポジトリのルートに `CLAUDE.md` があり、共通のコーディング規約が書かれています。`services/payment/` 配下は他と異なるテストの流儀を採用しており、その旨を Claude に伝えたいと考えています。ルートの `CLAUDE.md` に「payment だけは例外で」と書き足す案が出ています。

You use Claude Code in a monorepo. The root `CLAUDE.md` holds shared coding conventions. Code under `services/payment/` follows a different testing convention and you want Claude to know. Someone proposes appending "except in payment" to the root `CLAUDE.md`.

**設問 / Question:**

より適切な方法はどれですか？

Which is the better approach?

- A) **`services/payment/CLAUDE.md` を作り、そのディレクトリ固有の流儀をそこに書く。`CLAUDE.md` は階層的に読み込まれ、そのディレクトリの内容を扱うときにより近い階層の記述が効く** / **Create `services/payment/CLAUDE.md` and put the directory-specific convention there. `CLAUDE.md` files load hierarchically, and the nearer file applies when working with that directory's contents**
- B) ルートの `CLAUDE.md` に全ディレクトリの例外を列挙する / Enumerate every directory's exceptions in the root `CLAUDE.md`
- C) payment 用に別のリポジトリを切り出す / Split payment into its own repository
- D) 作業のたびにプロンプトで payment の流儀を説明する / Explain the payment convention in the prompt each time

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

`CLAUDE.md` は階層的に扱われます。ユーザー全体の設定、リポジトリルート、そしてサブディレクトリと、それぞれの階層に置けます。サブディレクトリの `CLAUDE.md` は、そのディレクトリの内容を扱うときに読み込まれるため、そこにしか関係しない指示を全体のコンテキストに常駐させずに済みます。これは指示を「適用範囲と一致する場所」に置くという原則の、Claude Code における具体形です。ルートには全体に効く共通規約だけを残すことで、両方が読みやすく保守しやすくなります。

`CLAUDE.md` is hierarchical: there are files for the user overall, for the repository root, and for subdirectories. A subdirectory's `CLAUDE.md` loads when working with that directory's contents, so guidance that concerns only that directory does not sit permanently in the global context. This is the "place an instruction where its scope lives" principle in its Claude Code form. Leaving only genuinely shared conventions at the root keeps both files readable and maintainable.

- **B 不正解**: 例外の列挙はルートの `CLAUDE.md` を肥大化させ、無関係な作業でもすべての例外がコンテキストを占めます。階層の仕組みを使わない理由がありません / Enumerating exceptions bloats the root file so every unrelated task carries all of them in context. There is no reason to forgo the hierarchy
- **C 不正解**: テストの流儀が違うだけでリポジトリを分割するのは過剰で、モノレポの利点を失います / Splitting a repository over a differing test convention is disproportionate and gives up the monorepo's benefits
- **D 不正解**: 毎回の説明は手間がかかるうえ、伝え忘れが起こります。リポジトリに書いておけばチーム全員に共有されます / Explaining every time is laborious and will be forgotten. Committing it to the repository shares it with the whole team

**参照 / Reference:** Claude Code Operation — `CLAUDE.md` の階層
</details>

---

### 問題 2 / Question 2

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

CI のジョブから Claude Code を呼び出し、変更されたファイルに対する簡易レビュー結果をテキストで得たいと考えています。対話的なやり取りは不要で、結果を後続のステップで機械的に処理します。

You want to invoke Claude Code from a CI job to get a short review of the changed files as text. No interaction is needed, and the result will be processed mechanically by a later step.

**設問 / Question:**

適切な実行方法はどれですか？

Which is the appropriate way to run it?

- A) 対話セッションを起動し、期待する入力を自動で送り込む / Launch an interactive session and feed it the expected keystrokes
- B) **ヘッドレスモードで実行する。プロンプトを引数として渡し、対話 UI を介さずに結果を標準出力へ返させる。機械処理する場合は構造化された出力形式を指定する** / **Run in headless mode: pass the prompt as an argument so the result goes to standard output without the interactive UI, specifying a structured output format when the result will be machine-processed**
- C) CI では Claude Code を使わず、API を直接呼ぶしかない / Claude Code cannot be used in CI; you must call the API directly
- D) 対話セッションの画面出力をキャプチャして解析する / Capture and parse the interactive session's screen output

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

ヘッドレスモードは、まさにこの用途のための機能です。プロンプトを引数として渡すと、対話 UI を起動せずに実行し、結果を標準出力に返します。CI・スクリプト・バッチ処理から呼び出す場合の標準的な使い方です。結果を後続のステップで機械的に処理するなら、構造化された出力形式を指定しておくと、テキストの体裁変更に影響されずにパースできます。ストリーミング形式を選べば、長い処理の途中経過を逐次受け取ることもできます。

Headless mode exists for exactly this. Passing the prompt as an argument runs without the interactive UI and returns the result on standard output — the standard way to call it from CI, scripts, and batch jobs. When a later step processes the result mechanically, specifying a structured output format makes parsing immune to changes in text formatting. A streaming format is available when you want incremental progress from a long run.

- **A 不正解**: 対話 UI への自動入力は脆く、UI の変更で壊れます。専用の非対話モードがあるのに使わない理由がありません / Driving the interactive UI is brittle and breaks with UI changes. There is no reason to skip the purpose-built non-interactive mode
- **C 不正解**: ヘッドレスモードが用意されており、CI からの利用は想定された使い方です / Headless mode exists, and CI use is an intended one
- **D 不正解**: 画面出力の解析は A と同じ脆さを持ち、構造化出力という正しい手段があります / Parsing screen output has the same brittleness as A when a structured output format is available

**参照 / Reference:** Claude Code Operation — ヘッドレスモード、ストリーミングモード
</details>

---

### 問題 3 / Question 3

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

チームには「リリースノートを、決まった書式と手順で生成する」という定型作業があります。誰でも同じ結果が得られるようにしたく、実行のたびに手順を説明したくありません。作業は明示的に「これをやって」と起動する性質のものです。

Your team has a recurring task: generating release notes in a fixed format by a fixed procedure. You want anyone to get the same result without explaining the procedure each time. The task is one you explicitly trigger — "do this now."

**設問 / Question:**

最も適切な仕組みはどれですか？

Which mechanism is most appropriate?

- A) `settings.json` に手順を書く / Write the procedure into `settings.json`
- B) ルートの `CLAUDE.md` に手順を書く / Write the procedure into the root `CLAUDE.md`
- C) **カスタムスラッシュコマンドとして手順をリポジトリに置き、`/コマンド名` で起動できるようにする。リポジトリに置けばチーム全員が同じ手順を使える** / **Put the procedure in the repository as a custom slash command, invoked as `/<name>`. Committing it to the repository gives the whole team the same procedure**
- D) 手順書を Wiki に置き、各自がコピーしてプロンプトに貼る / Keep the procedure in a wiki and have everyone paste it into a prompt

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

カスタムスラッシュコマンドは、「利用者が明示的に起動する定型手順」のための仕組みです。リポジトリに置けばチームで共有され、更新も 1 か所で済みます。起動が明示的であることがこの選択の決め手で、`CLAUDE.md` のように常時読み込まれる文書とは役割が違います。Claude Code の構成要素は、それぞれ起動のされ方が異なります。Rules（`CLAUDE.md`）は常に効く方針、Skills は関連する場面で読み込まれる専門知識、Commands は利用者が起動する定型手順、Agents は別コンテキストで動くサブエージェントです。この対応で選べば迷いません。

Custom slash commands are for a fixed procedure the user explicitly triggers. Committed to the repository they are shared across the team and updated in one place. The explicit trigger is what decides this choice, distinguishing them from a document like `CLAUDE.md` that is always loaded. Claude Code's components differ in how they are activated: Rules (`CLAUDE.md`) are always-on guidance, Skills are expertise loaded in relevant situations, Commands are user-triggered procedures, and Agents are subagents running in their own context. Match the activation and the choice is clear.

- **A 不正解**: `settings.json` は権限・フック・環境などの設定ファイルで、作業手順を書く場所ではありません / `settings.json` configures permissions, hooks, and environment; it is not where procedures go
- **B 不正解**: `CLAUDE.md` は常時読み込まれる方針を書く場所です。年に数回のリリースノート生成手順を常駐させると、無関係な作業でコンテキストを消費します / `CLAUDE.md` holds always-loaded guidance. Keeping an occasional release-notes procedure resident spends context on unrelated work
- **D 不正解**: 貼り付け運用は、コピー漏れやバージョンのずれを生みます。Claude Code に共有の仕組みがあるのに使わない理由がありません / Copy-and-paste invites omissions and version drift when Claude Code provides a sharing mechanism

**参照 / Reference:** Claude Code Operation — カスタムスラッシュコマンド、中核構成要素の使い分け
</details>

---

### 問題 4 / Question 4

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

チーム全員の Claude Code に、共通の権限設定（特定のコマンドの許可・拒否）を適用したいと考えています。同時に、各開発者が自分の環境でだけ使う一時的な許可設定も持てるようにしたいと考えています。

You want a common permission configuration — allowing and denying specific commands — applied to everyone's Claude Code, while still letting each developer keep temporary allowances that apply only in their own environment.

**設問 / Question:**

最も適切な構成はどれですか？

Which is the most appropriate setup?

- A) 全員が自分の設定を手で揃える / Have everyone align their settings by hand
- B) 共通設定も個人設定も、各自のホームディレクトリの設定ファイルに書く / Put both shared and personal settings in each person's home-directory settings file
- C) 共通設定をチャットで共有し、各自が必要に応じて反映する / Share the common settings in chat and have people apply them as needed
- D) **共有する設定はリポジトリの `.claude/settings.json` に置いてコミットし、個人だけの設定は同じディレクトリのローカル設定ファイル（バージョン管理の対象外）に置く。設定は階層的にマージされる** / **Commit shared settings to the repository's `.claude/settings.json`, and keep personal-only settings in the local settings file alongside it, excluded from version control. Settings merge hierarchically**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

Claude Code の設定は階層的にマージされます。リポジトリにコミットする `.claude/settings.json` はチーム共通の設定で、プロジェクトの一部としてレビューやバージョン管理の対象になります。個人だけの設定は、同じディレクトリのローカル用ファイルに書き、バージョン管理から除外します。この分離により、共通の権限設定はチーム全員に確実に適用され、各自の一時的な許可は他人に影響しません。ユーザー全体（ホームディレクトリ）の設定は、リポジトリをまたいで効かせたい個人設定に使います。

Claude Code settings merge hierarchically. `.claude/settings.json` committed to the repository is the team's shared configuration, reviewed and versioned as part of the project. Personal-only settings go in the local file alongside it, excluded from version control. That separation applies shared permissions reliably to everyone while keeping one person's temporary allowance from affecting others. The user-level file in the home directory is for personal settings you want across repositories.

- **A 不正解**: 手作業での同期は必ずずれます。設定の共有はリポジトリで行うべきです / Manual synchronization always drifts. Sharing settings belongs in the repository
- **B 不正解**: ホームディレクトリの設定は個人のものなので、チーム共通の設定を確実に配る手段になりません / Home-directory settings are personal and are not a reliable way to distribute a team-wide configuration
- **C 不正解**: チャットでの共有は適用漏れが起こり、誰が適用済みかも分かりません / Sharing in chat leads to missed applications with no visibility into who has applied what

**参照 / Reference:** Claude Code Operation — `settings.json` の設定、共有設定とローカル設定
</details>

---

### 問題 5 / Question 5

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

長時間のセッションで複数のタスクを続けて依頼していたところ、Claude Code が前のタスクの文脈を引きずった提案をするようになりました。また、応答が遅くなってきています。

Over a long session in which you asked for several tasks in a row, Claude Code started making suggestions colored by the previous task's context, and responses have become slower.

**設問 / Question:**

セッション管理として適切な対応を **2 つ選択してください**。

Select **2** appropriate session-management responses.

- A) Claude Code を終了して再インストールする / Quit and reinstall Claude Code
- B) **関連しない新しいタスクを始めるときは、コンテキストをクリアして新しいセッションとして始める** / **When starting an unrelated new task, clear the context and begin a fresh session**
- C) より高性能なモデルに切り替える / Switch to a more capable model
- D) 1 つのセッションで扱うタスクを増やし、まとめて依頼する / Put more tasks into one session and ask for them together
- E) **タスクをまたいで引き継ぎたい前提（プロジェクトの規約、環境の癖）は、その場のセッションに残すのではなく `CLAUDE.md` などの永続的な場所に書いておく** / **Record assumptions you want carried across tasks — project conventions, environment quirks — in a persistent place such as `CLAUDE.md` rather than leaving them in the live session**

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B, E**

**解説 / Explanation:**

症状は、コンテキストの肥大化と混入です。長いセッションには前のタスクの試行錯誤がすべて残り、無関係な新しいタスクにも影響します。応答が遅くなるのも入力トークンが増えているためです。B はこれに直接対処し、タスクの区切りでコンテキストをリセットします。ただし、毎回リセットすると引き継ぎたい前提まで失われるので、E がそれを補います。恒常的に必要な情報はセッションではなく永続的な場所に置く、という切り分けです。この 2 つを組み合わせると、セッションは短く保ちつつ、必要な文脈は毎回自動的に入ります。

The symptoms are context bloat and contamination. A long session retains every prior task's exploration and colors an unrelated new one, and responses slow because input tokens have grown. Choice B addresses this directly by resetting at task boundaries. But resetting every time would also discard assumptions worth carrying, which is what E supplies: information that is permanently needed belongs in a persistent place, not in a session. Together they keep sessions short while the necessary context loads automatically each time.

- **A 不正解**: 再インストールはコンテキストの問題と無関係で、症状の原因はソフトウェアの状態ではありません / Reinstalling is unrelated to a context problem; the cause is not the software's state
- **C 不正解**: モデルを変えても、蓄積したコンテキストは同じように渡されます。混入も遅さも解消しません / A different model still receives the same accumulated context. Neither the contamination nor the slowness goes away
- **D 不正解**: 1 セッションに詰め込むことは、まさに現在起きている問題を悪化させる方向です / Packing more into one session is precisely the direction that worsens the current problem

**参照 / Reference:** Claude Code Operation — セッション管理、コンテキストの分離
</details>

---

## 発展 / Advanced

### 問題 6 / Question 6

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

50 人規模の開発組織で Claude Code を本格導入します。既存のリポジトリは 30 以上あり、それぞれ規約もビルド手順も異なります。「まず全リポジトリに `CLAUDE.md` を用意しよう」という方針が決まりましたが、内容をどう作るかで議論になっています。網羅的な文書を書こうとして着手が進んでいません。

A 50-person engineering organization is rolling out Claude Code across more than 30 repositories with differing conventions and build procedures. The decision is to give every repository a `CLAUDE.md`, but the team is stuck debating the content, attempting an exhaustive document before starting.

**設問 / Question:**

最も適切な進め方はどれですか？

Which is the most appropriate approach?

- A) 全リポジトリで共通の `CLAUDE.md` テンプレートを作り、全項目を埋めることを必須とする / Create one template for all repositories and require every field to be filled in
- B) **各リポジトリで初期化コマンドを使って `CLAUDE.md` の草案を生成し、実際に Claude Code を使う中で「毎回説明している内容」を追記していく。網羅を目指さず、繰り返し必要になった知識だけを足す** / **Generate a draft `CLAUDE.md` in each repository with the initialization command, then add what you find yourself explaining repeatedly as you actually use Claude Code. Do not aim for exhaustiveness — add only knowledge that recurs**
- C) `CLAUDE.md` は用意せず、必要な情報は毎回プロンプトで渡す / Skip `CLAUDE.md` and pass what is needed in each prompt
- D) 各リポジトリの README をそのまま `CLAUDE.md` にコピーする / Copy each repository's README to `CLAUDE.md` unchanged

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

着手が進まない原因は、網羅を目指していることです。`CLAUDE.md` に何を書くべきかは、実際に使ってみるまで分かりません。初期化コマンドでリポジトリを解析した草案を作れば出発点ができ、そこから「毎回同じことを説明している」という実体験を根拠に追記していけば、実際に効く内容だけが残ります。この進め方には副次的な利点もあり、書いた内容が本当に必要だったかを使いながら検証できます。網羅的な文書は、書くコストが高いわりに大半が使われず、しかも常時コンテキストを占めます。

The team is stuck because it is aiming for exhaustiveness. What belongs in a `CLAUDE.md` is not knowable until you use it. The initialization command produces a draft from an analysis of the repository, giving you a starting point; adding to it on the evidence of "I keep explaining this" leaves only content that demonstrably helps. There is a secondary benefit: you validate each addition against real use. An exhaustive document costs a lot to write, goes mostly unused, and occupies context permanently.

- **A 不正解**: 全項目必須のテンプレートは、リポジトリによっては該当しない項目を埋めさせ、内容の薄い文書を量産します / A template with mandatory fields forces entries where they do not apply and mass-produces thin documents
- **C 不正解**: 30 以上のリポジトリで 50 人が毎回説明するのは、まさに `CLAUDE.md` が解決する無駄です / Fifty people explaining the same things across 30-plus repositories is exactly the waste `CLAUDE.md` removes
- **D 不正解**: README は人間の読者向けで、導入手順や機能紹介が中心です。Claude が必要とする規約やビルド手順の情報とは重なりが限られます / A README targets human readers with setup and feature overviews. Its overlap with the conventions and build procedures Claude needs is limited

**参照 / Reference:** Claude Code Operation — リポジトリの初期化、`CLAUDE.md` の育て方
</details>

---

### 問題 7 / Question 7

> サブスキル / Sub-skill: Claude Code Operation (3.1%)

**シナリオ / Scenario:**

CI パイプラインで Claude Code をヘッドレスモードで動かし、失敗したテストの修正を自動で試みる仕組みを検討しています。CI 環境には本番デプロイ用の資格情報が環境変数として存在し、リポジトリへの書き込み権限もあります。プルリクエストは外部のコントリビューターからも来ます。

You are considering running Claude Code headless in CI to attempt automatic fixes for failing tests. The CI environment holds production deployment credentials in environment variables and has write access to the repository. Pull requests also come from outside contributors.

**設問 / Question:**

最も重要な設計上の考慮はどれですか？

Which is the most important design consideration?

- A) ヘッドレスモードの出力形式を JSON にすること / Making the headless output format JSON
- B) 実行時間の上限を設けること / Setting a wall-clock limit on the run
- C) **外部コントリビューターのプルリクエストの内容が、そのまま Claude Code への入力になる点。信頼できない入力・本番資格情報・書き込み権限が同じ実行環境に揃っているため、この経路は最小権限で分離する必要がある。外部 PR に対しては資格情報を持たない隔離環境で実行し、権限は `settings.json` の拒否設定とフックで絞り、変更は直接コミットせず提案として出す** / **That an outside contributor's pull request becomes input to Claude Code directly. Untrusted input, production credentials, and write access coincide in one execution environment, so this path must be isolated under least privilege: run outside PRs in an environment holding no credentials, constrain capability with `settings.json` deny rules and hooks, and emit changes as suggestions rather than direct commits**
- D) 修正の成功率を事前に測定すること / Measuring the fix success rate up front

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

CI でエージェントを動かす構成では、実行環境が持っている権限がそのままエージェントの権限になります。ここでは、外部から誰でも送れるプルリクエストの内容（テストコード、コメント、ファイル名を含む）がエージェントの入力になり、その実行環境に本番の資格情報と書き込み権限があります。これはドメイン 7 で見た「信頼できない入力・機微な資産・外部への出口」が揃った状態そのものです。対処は経路の分離で、外部 PR に対する実行からは資格情報を外し、フックと拒否設定で実行できる操作を絞り、結果は直接コミットせず提案として人間のレビューに載せます。

When an agent runs in CI, the execution environment's privileges are the agent's privileges. Here the content of a pull request anyone can open — test code, comments, filenames — becomes the agent's input, in an environment holding production credentials and write access. This is precisely the coincidence of untrusted input, sensitive assets, and egress seen in Domain 7. The remedy is to separate the path: strip credentials from runs triggered by outside PRs, narrow what can execute with hooks and deny rules, and emit results as proposals for human review rather than direct commits.

- **A 不正解**: 出力形式は実装の詳細で、セキュリティ上の考慮とは重大度が比較になりません / Output format is an implementation detail, not comparable in severity to the security consideration
- **B 不正解**: 実行時間の上限はコスト管理として有用ですが、資格情報の露出を防ぎません / A time limit helps manage cost but does nothing about credential exposure
- **D 不正解**: 成功率の測定は導入の価値判断に必要ですが、この構成のリスクは成功率が高くても変わりません。むしろ成功率が高いほど自動適用が増え、リスクは大きくなります / Success rate matters for the value case, but the risk here does not shrink with a high rate — a higher rate means more automatic application and more exposure

**参照 / Reference:** Claude Code Operation — ヘッドレスモードの運用、権限と `settings.json`、フックによる統制
</details>

---
