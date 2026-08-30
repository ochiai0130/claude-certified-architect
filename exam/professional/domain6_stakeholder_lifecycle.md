# Domain 6: ステークホルダー折衝とライフサイクル管理 / Stakeholder Communication and Lifecycle Management

> 配点比率 / Weight: **14%**（Professional で新たに加わる領域 / new at the Professional tier）
> 問題数 / Questions: **30**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- 要件の引き出しと問題の再定義・成功基準の合意 / Eliciting requirements, reframing the problem, agreeing success criteria
- 非技術者への説明（確率的挙動・精度・限界）と期待値の管理 / Explaining probabilistic behavior, accuracy, and limits to non-technical stakeholders; managing expectations
- ROI・TCO・投資判断の提示 / Presenting ROI, TCO, and the investment case
- パイロットから本番への移行・導入時の変更管理と現場のイネーブルメント / Pilot-to-production transition, change management, front-line enablement
- 運用引き継ぎ・意思決定の記録・スコープ管理 / Handover to operations, decision records, scope control
- 機能の終了・移行の告知と顧客への約束 / Deprecation and end-of-life communication, commitments to customers

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

営業部門の責任者から「営業日報を Claude で自動生成したい。日報作成に 1 人あたり毎日 40 分かかっていて負担が大きい」という依頼を受けました。ヒアリングを進めると、日報は上司が読むためのもので、実際には上司の 8 割は日報を読んでおらず、読む 2 割も「訪問件数と受注見込み」の 2 項目しか見ていないことが分かりました。日報の内容は CRM に入力済みのデータと大部分が重複しています。

A sales director asks you to auto-generate daily sales reports with Claude, citing 40 minutes per person per day. In discovery you learn the reports are written for managers, that 80% of managers do not read them, and that the 20% who do look only at visit count and expected bookings — data already entered in the CRM.

**設問 / Question:**

アーキテクトとして最も適切な対応はどれですか？

As the architect, what is the most appropriate response?

- A) 依頼どおり日報生成エージェントを構築する / Build the report-generation agent as requested
- B) 日報の廃止を提案し、それ以上は関与しない / Propose abolishing the report and disengage
- C) **調査結果を依頼者に共有し、解くべき問題を再定義したうえで選択肢を提示する**。真の課題は「日報作成の負担」であり、その負担の大半は既に CRM にあるデータの再入力から生じている。選択肢として (1) 日報を廃止して CRM から自動集計したダッシュボードに置き換える、(2) 読まれている 2 項目だけを CRM から自動生成する、(3) 依頼どおり全文を生成する、を効果とコストとともに示し、意思決定は依頼者に委ねる / **Share the findings, reframe the problem, and present options.** The real issue is the reporting burden, most of which comes from re-entering data already in the CRM. Offer (1) retire the report and replace it with a dashboard aggregated from the CRM, (2) auto-generate only the two fields anyone reads, or (3) generate the full text as requested — each with its effect and cost — and let the requester decide
- D) 上司に日報を読むよう働きかける / Encourage managers to read the reports

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

依頼は「日報を自動生成してほしい」ですが、**背後にある課題は「日報作成の負担」**です。調査で分かったのは、その負担の大部分が既存データの再入力であり、しかも成果物の大半が読まれていないという事実です。ここでアーキテクトがすべきなのは、依頼をそのまま実装することでも、独断で廃止を決めることでもなく、**発見した事実を共有して問題を再定義し、選択肢と効果を示して意思決定を依頼者に返すこと**です。業務の廃止は依頼者の権限に属する判断であり、技術者が単独で決めるものではありません。

The request is "generate the report"; the underlying problem is the reporting burden, most of which comes from re-entering existing data — for an artifact largely unread. The architect's job is neither to implement literally nor to unilaterally retire the report, but to share the findings, reframe, and hand the decision back with options and their consequences. Retiring a business process is the requester's call.

- **A 不正解**: 読まれない文書を効率よく生成する構成で、投資に対する価値が最も低い選択肢です。 / Efficiently produces an artifact nobody reads.
- **B 不正解**: 廃止の判断は依頼者の権限であり、提案だけして離脱するのは支援として不十分です。 / The decision is theirs, and disengaging leaves the problem unsolved.
- **D 不正解**: 読まれていないこと自体が価値の低さを示しており、読ませることが目的化しています。 / Treats readership as the goal rather than the outcome.

**参照 / Reference:** 要件の引き出し、問題の再定義、意思決定の返却
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

新しいエージェントの開発を開始するにあたり、事業部門から「とにかく良いものを作ってほしい」「使ってみて判断する」という要望を受けました。過去の類似プロジェクトでは、完成後に「思っていたものと違う」という評価になり、3 回の大きな作り直しが発生しました。開発期間は 4 か月の予定です。

Starting a new agent project, the business unit says "just build something good; we'll judge it once we try it." On a previous similar project, the finished system was judged "not what we expected" and went through three major rebuilds. The planned timeline is four months.

**設問 / Question:**

最も適切な進め方はどれですか？

What is the most appropriate way to proceed?

- A) 要望どおり、まず作って見せてから調整する / Build first and adjust after showing it, as requested
- B) **着手前に成功基準を具体的に合意する**。「どのような入力に対して、どの水準の出力が得られれば成功か」を、測定可能な形で定義する。既存業務の実例を 20〜30 件持ち寄って「この入力ならこの出力が期待される」を明示し、それを評価データセットの出発点にする。精度・レイテンシ・カバー範囲・対象外とする範囲を書き出し、双方が署名する。あわせて早期に動くものを見せる機会を設けて、基準自体を見直す場も計画に含める / **Agree concrete success criteria before starting**: define, measurably, which inputs must produce which quality of output. Bring 20–30 real cases and state the expected output for each, using them as the seed of the evaluation set. Write down accuracy, latency, coverage, and explicit out-of-scope areas, and have both sides sign off — while also scheduling early demonstrations so the criteria themselves can be revisited
- C) 開発期間を 8 か月に延長して、作り直しの余裕を持たせる / Extend the timeline to eight months to absorb rebuilds
- D) 事業部門に要件定義書の作成を依頼し、完成を待つ / Ask the business unit to produce a requirements document and wait

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

3 回の作り直しは、**成功基準が事前に合意されていなかったこと**の帰結です。「使ってみて判断する」は、判断基準が判断の時点まで存在しないことを意味し、評価が主観的な印象に依存します。有効な方法は、抽象的な要件定義ではなく**実例に基づく合意**で、「この入力ならこの出力」を 20〜30 件並べれば、期待値が具体的になり、そのまま評価データセットになります。同時に、基準を固定して終わりにせず、早期に動くものを見せて基準を見直す場を計画に含めるのが実務的です。

Three rebuilds are the consequence of criteria never agreed in advance: "we'll judge when we try it" means no standard exists until judgment time, so evaluation reduces to impression. What works is agreement grounded in examples rather than abstract requirements — 20–30 input/expected-output pairs make expectations concrete and double as the evaluation set — combined with early demonstrations so the criteria themselves can evolve.

- **A 不正解**: 過去 3 回失敗した進め方の反復です。 / Repeats the approach that failed three times.
- **C 不正解**: 期間を延ばしても基準がなければ作り直しは繰り返されます。原因に対処していません。 / More time without criteria means more rebuilds.
- **D 不正解**: 抽象的な要件定義書は、実例に基づく合意より期待値のずれを生みやすく、待つ間に時間も失われます。 / An abstract document diverges from expectations more easily than examples, and stalls the project.

**参照 / Reference:** 成功基準の事前合意、実例による期待値の明確化、評価データセットとの接続
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

CFO に対して、契約書レビュー支援エージェントの投資判断を仰ぐ資料を作成しています。開発費は 1,800 万円、API 費用は月 60 万円と見積もりました。削減効果は「法務担当者の作業時間が月 320 時間削減、時間単価 6,000 円換算で月 192 万円」としています。同僚から「これだけ示せば十分だ」と言われました。

You are preparing an investment case for a contract-review assist agent. Development is estimated at ¥18M and API costs at ¥600K/month. The benefit is stated as 320 hours/month of legal-team time saved, valued at ¥6,000/hour, or ¥1.92M/month. A colleague says this is sufficient.

**設問 / Question:**

最も適切な資料の構成はどれですか？

Which case structure is most appropriate?

- A) 開発費と API 費用、削減効果の 3 点で十分と判断する / Present development cost, API cost, and savings — nothing more
- B) 削減効果を実際より大きく見積もって、承認を得やすくする / Inflate the savings to improve the odds of approval
- C) API 費用を含めず、開発費のみを提示する / Present development cost only, excluding API costs
- D) **継続的に発生するコストと前提条件を含めた総所有コスト（TCO）として提示する**。API 費用に加えて、評価データセットの整備と維持、監視と運用、モデル更新時の移行作業、人手レビューの工数、障害対応の体制を運用コストとして見積もる。効果側も、削減時間の根拠（測定方法）と、削減された時間が実際に他の業務に振り向けられるのかという前提を明示する。前提が崩れる条件と、その場合の見直し時期も示す / **Present a total cost of ownership including recurring costs and stated assumptions**: beyond API spend, estimate building and maintaining the evaluation set, monitoring and operations, migration work at model updates, human-review effort, and incident response. On the benefit side, state how the saved hours were measured and whether that time is actually redeployed to other work. Name the conditions under which the assumptions fail and when the case would be revisited

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

AI システムの投資判断で最も見落とされるのが**運用に継続的にかかるコスト**です。評価データセットの維持、監視、モデル更新時の移行、人手レビューは、いずれも無視できない工数で、これらを含めないと実際の収支と乖離します。効果側についても、「削減された時間が実際に価値を生む活動に振り向けられるか」という前提を明示すべきで、単に時間単価を掛けただけの数字は CFO の検証に耐えません。**前提が崩れる条件を先に示す**ことは、信頼できる提案の条件でもあります。

The most commonly omitted element in an AI investment case is recurring operating cost: evaluation-set maintenance, monitoring, migration at model updates, and human review are all material. On the benefit side, multiplying hours by a rate does not survive CFO scrutiny without stating whether the freed time is actually redeployed. Naming the conditions under which the case fails is part of what makes it credible.

- **A 不正解**: 継続的な運用コストを含まないため、実際の総コストを大幅に過小評価します。 / Materially understates total cost.
- **B 不正解**: 過大な見積もりは、達成できなかった場合に信頼を失い、以後の提案も通らなくなります。 / Destroys credibility when the numbers do not materialize.
- **C 不正解**: 継続費用を隠す提示は不誠実であり、承認後に問題化します。 / Concealing recurring costs is dishonest and surfaces after approval.

**参照 / Reference:** TCO、継続コストの計上、効果の前提の明示、見直し条件
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

経営会議で、導入予定のエージェントについて役員から「このシステムは 100% 正確なのか」と質問を受けました。実際の評価結果は主要タスクで 94% です。同席していた開発リーダーは「ほぼ正確です」と回答しました。役員は「では従来の人手チェックは不要だな」と述べ、体制の縮小を検討する方向で議論が進みそうです。

At an executive meeting, a board member asks whether the planned agent is "100% accurate." The measured figure on the primary task is 94%. The development lead answers "essentially accurate." The board member responds that manual checking can therefore be removed, and the discussion turns toward reducing headcount.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **数値と、その数値が業務上何を意味するかを具体的に説明する**。「主要タスクで 94%、月間の処理件数に換算すると約 X 件の誤りが生じ得る」と示し、誤りの種類（重大なものと軽微なもの）と検知可能性を説明する。そのうえで、人手チェックを外す場合に残るリスクと、チェックを残す場合のコストを並べ、どこまでのリスクを受容するかの判断材料を提供する。「ほぼ正確」といった曖昧な表現は、この場では誤解を生む / **State the number and what it means operationally.** Say "94% on the primary task, which at current volume implies roughly X errors per month," describe the error types (serious versus minor) and their detectability, then lay out the residual risk of removing manual checks against the cost of keeping them — giving the board what it needs to decide how much risk to accept. Vague phrasing such as "essentially accurate" actively misleads in this setting
- B) 「ほぼ正確です」という説明を維持し、詳細は後日資料で示す / Stand by "essentially accurate" and provide details later in writing
- C) 100% 正確であると回答して、導入への支持を得る / Answer that it is 100% accurate to secure support
- D) 精度の話題を避け、コスト削減効果に議論を戻す / Steer the discussion away from accuracy and back to cost savings

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「ほぼ正確」という表現が、**「人手チェック不要」という重大な意思決定を誘発している**点が本問の核心です。非技術者に確率的な挙動を伝えるときは、パーセンテージだけでなく**業務上の意味に換算する**のが有効です（「94%」より「月あたり X 件の誤り」の方が判断できます）。さらに、誤りの重大度と検知可能性を分けて説明することで、「どのリスクを人手で受け止めるか」という具体的な議論に落ちます。曖昧な表現は、誠実さの問題であると同時に、誤った意思決定を招く実害があります。

The crux is that "essentially accurate" is driving a consequential decision to remove human checks. Communicating probabilistic behavior to non-technical stakeholders works best translated into operational terms — "X errors per month" is decidable in a way "94%" is not — and separating error severity from detectability turns it into a concrete discussion of which risks humans absorb. Vagueness here is not just imprecise; it causes the wrong decision.

- **B 不正解**: その場で誤解に基づく意思決定が進む可能性があり、後日の資料では間に合いません。 / A decision based on the misunderstanding may be made in the room.
- **C 不正解**: 事実に反する説明であり、後に判明したときに信頼を失います。 / False, and destroys credibility when discovered.
- **D 不正解**: 精度は人手チェックの要否を左右する中心的な論点で、避けることはできません。 / Accuracy is precisely the question that determines the decision at hand.

**参照 / Reference:** 非技術者への精度の説明、業務上の意味への換算、期待値の管理
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

3 か月のパイロットが終了しました。参加した 12 名のユーザーからは「便利だった」という好意的な反応が得られ、事業部門は全社 800 名への展開を求めています。パイロット参加者は AI に前向きな有志で、業務内容も比較的定型的な部署に偏っていました。パイロット期間中、開発チームが週次で個別に問い合わせ対応と設定調整を行っていました。

A three-month pilot has ended. Its 12 users found it useful and the business unit wants to roll out to all 800 staff. The pilot participants were AI-positive volunteers from departments with relatively routine work, and the development team handled inquiries and tuned configuration for them individually every week.

**設問 / Question:**

最も適切な判断はどれですか？

Which judgment is most appropriate?

- A) パイロットが成功したので、そのまま全社展開する / Roll out to everyone, since the pilot succeeded
- B) 全社展開を中止し、パイロットを継続する / Cancel the rollout and continue the pilot
- C) **パイロットの結果が全社に一般化できるかを検討したうえで、段階的に拡大する**。参加者が有志で業務も定型的だったため、この結果は AI に懐疑的な層や非定型業務での成否を示していない。次段階では、業務内容と AI への態度の両面で異なる部署を含む集団に広げて検証する。また、週次の個別対応は 800 名には成立しないため、自己解決できるドキュメント・研修・問い合わせ窓口を用意し、その運用体制も含めて次段階で検証する / **Assess whether the pilot generalizes, then expand in stages.** Volunteers doing routine work say nothing about skeptical users or non-routine work, so the next stage should include departments differing on both dimensions. Weekly individual support also does not scale to 800, so prepare self-service documentation, training, and a support desk — and validate that operating model in the next stage as well
- D) 全社展開し、問題が出た部署から個別に対応する / Roll out to all 800 and handle problem departments as they arise

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

パイロットの結果を読むときは、**その母集団が展開先を代表しているか**を必ず確認します。有志・定型業務という条件は、最も成功しやすい条件であり、800 名の中には AI に懐疑的な層や非定型業務の担当者が含まれます。もう 1 つの見落としが**支援体制のスケール**で、週次の個別対応という前提が結果に寄与している可能性が高く、800 名では同じ支援は提供できません。次段階では、条件の異なる集団と、実際に運用可能な支援体制の両方を検証する必要があります。

Reading a pilot means asking whether its population represents the rollout target. Volunteers doing routine work are the most favorable conditions available, while 800 people include skeptics and non-routine roles. The second, easily missed factor is support scaling: weekly hands-on help likely contributed to the result and cannot be reproduced at 800. The next stage must validate both a different population and a support model that actually scales.

- **A 不正解**: 母集団の偏りと支援体制の非スケーラビリティを見落としています。 / Ignores both population bias and the unscalable support model.
- **B 不正解**: 好意的な結果が出ている以上、中止は過剰な判断です。検証すべきは一般化可能性です。 / Overreacts to a favorable result; the question is generalizability.
- **D 不正解**: 800 名に一斉展開して個別対応するのは、支援体制が破綻して定着に失敗する典型的な形です。 / Overwhelms support and typically ends in failed adoption.

**参照 / Reference:** パイロットの一般化可能性、母集団の偏り、支援体制のスケール、段階的拡大
</details>

---

## 問題 6 / Question 6

**シナリオ / Scenario:**

経営層向けのデモで、エージェントが 8 件の質問すべてに的確に回答し、大きな期待が生まれました。デモで使った質問は開発チームが選んだもので、いずれも社内文書に明確な答えがあるものでした。実際の業務では、文書に答えがない質問、複数文書にまたがる質問、前提が曖昧な質問が半分以上を占めます。経営層は「来月から全社で使える」と発言しています。

In an executive demo, the agent answered all eight questions well and expectations rose sharply. The questions were chosen by the development team and all had clear answers in internal documents. In real usage, more than half of questions have no documented answer, span multiple documents, or rest on ambiguous premises. Leadership has said it can go company-wide next month.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 期待に応えるため、来月の全社展開に向けて開発を加速する / Accelerate to meet the expectation of a company-wide launch next month
- B) **デモと実業務の差を早い段階で明示し、期待値を調整する**。デモの質問が「文書に明確な答えがある」類型に偏っていたことを説明し、実業務の質問分布（文書に答えがない、複数文書横断、前提が曖昧）での性能を測って示す。可能であれば、実業務から抽出した質問でデモをやり直す。そのうえで、確実に価値を出せる範囲から段階的に展開する計画を提示し、来月の全社展開が現実的でない理由を根拠とともに伝える / **Surface the gap between the demo and real work early and reset expectations**: explain that the demo questions were skewed toward the documented-answer case, and measure and present performance on the real distribution (no documented answer, cross-document, ambiguous premises). Where possible, re-run the demo with questions drawn from real usage. Then propose a staged rollout beginning where value is assured, and explain with evidence why next month is not realistic
- C) 期待値の調整はせず、展開後に実態を知ってもらう / Say nothing and let reality become apparent after launch
- D) デモの内容は適切だったと説明し、実業務の難しさは伝えない / Defend the demo as appropriate and not mention the difficulty of real work

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**成功しやすい事例だけを選んだデモは、期待値を実力から乖離させます。**この乖離を放置すると、展開後に「期待外れ」という評価が下り、システム全体の信頼を失います。適切な対応は、乖離が小さいうちに事実を示すことで、**実業務の分布での測定結果**を持って説明するのが最も説得力を持ちます。実業務から抽出した質問でデモをやり直せば、期待値の調整と現状の共有が同時にできます。悪い知らせを早く伝えることは、信頼を損なうのではなく守る行為です。

A demo built from favorable cases decouples expectations from capability, and leaving that gap produces a "disappointing" verdict after launch that costs the system its credibility. The right move is closing the gap while it is small, and measured performance on the real distribution is the most persuasive way to do it. Re-running the demo with real questions resets expectations and shares the current state in one step. Delivering bad news early protects trust rather than spending it.

- **A 不正解**: 実業務での性能が未確認のまま展開すると、期待外れの評価と信頼喪失を招きます。 / Ships without knowing real-distribution performance.
- **C 不正解**: 展開後に判明する乖離は、最も大きな失望と信頼喪失を生みます。 / Discovery after launch maximizes the disappointment.
- **D 不正解**: 既知の限界を伝えないことは、後に判明したときに誠実性の問題になります。 / Withholding known limitations becomes an integrity issue later.

**参照 / Reference:** デモと実業務の乖離、期待値の調整、実分布での測定
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

請求書処理を自動化するエージェントを導入しました。技術的には正常に動作し、精度も基準を満たしています。しかし導入 2 か月後、経理部門での利用率は 18% にとどまり、多くの担当者が従来の手作業を続けています。ヒアリングすると「自分の仕事がなくなるのではないか」「間違っていた場合に責任を問われるのは自分だ」「使い方が分からない」という声が挙がりました。

An invoice-processing agent is live, works correctly, and meets its accuracy bar. Two months in, adoption in the accounting team is 18% and most staff continue working manually. Interviews surface three themes: fear that their jobs will disappear, concern that they will be held responsible for the system's errors, and not knowing how to use it.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **技術ではなく変更管理の問題として扱い、3 つの懸念それぞれに対応する**。役割の変化について、削減ではなく業務の再設計であることを経営として明示し、担当者が担う新しい役割（例外処理、判断、品質確認）を具体的に示す。責任については、エージェントの誤りに対する責任の所在と、担当者が承認した場合の扱いを明文化する。使い方については、実業務に即した研修と、失敗しても安全に試せる環境を用意する。利用率を追跡し、部署ごとの阻害要因を継続的に把握する / **Treat it as change management, not technology, and address all three concerns.** On roles, have leadership state explicitly that this is work redesign rather than reduction, and name the new responsibilities staff will hold (exceptions, judgment, quality assurance). On accountability, write down where responsibility sits for the system's errors and what changes when a person approves an output. On skills, provide training grounded in real work plus a safe environment to experiment in. Track adoption and keep identifying blockers by team
- B) 手作業を禁止して、エージェントの利用を義務化する / Prohibit manual processing and mandate use of the agent
- C) 精度をさらに高めれば利用されると判断し、モデルを改善する / Improve the model, assuming higher accuracy will drive adoption
- D) 利用率の低い担当者を評価上不利に扱う / Penalize low-adoption staff in performance reviews

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**技術的に動作することと、使われることは別**です。ヒアリングで挙がった 3 つの声は、いずれも技術的な問題ではなく、**役割の不安・責任の不明確さ・習熟の欠如**という変更管理の課題です。特に「間違っていた場合の責任」は重要で、責任の所在が曖昧なまま新しい道具を使えというのは、担当者にとって合理的なリスクです。役割の再設計を経営として明示し、責任を明文化し、安全に試せる環境を用意することが、利用率という結果に直接効きます。

Working and being used are different things. All three themes are change-management issues — role anxiety, unclear accountability, and unfamiliarity — not technical ones. Accountability matters most: asking someone to adopt a new tool while leaving liability undefined is a rational risk for them to decline. Leadership stating the role redesign, writing down where responsibility sits, and providing a safe place to practice is what moves adoption.

- **B 不正解**: 義務化は懸念に答えず、形式的な利用と回避行動を生みます。責任の不安はむしろ強まります。 / Mandates do not address the concerns and intensify the accountability fear.
- **C 不正解**: 精度は既に基準を満たしており、利用されない原因ではありません。 / Accuracy already meets the bar and is not the blocker.
- **D 不正解**: 評価上の不利益は、正当な懸念を表明した担当者を罰する対応で、信頼をさらに損ないます。 / Punishes people for raising legitimate concerns.

**参照 / Reference:** 変更管理、役割の再設計、責任の明確化、イネーブルメント
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

顧客に提供しているエージェントが、3 日間にわたり一部の照会に対して古いデータを返していたことが判明しました。原因はデータ同期の設定不備で、既に修正済みです。影響を受けた顧客は 40 社、うち 6 社はその情報を基に発注判断を行っていました。営業部門は「顧客に伝えると信頼を損なうので、静かに修正して様子を見たい」と主張しています。

An agent provided to customers returned stale data for some queries over three days. The cause was a data-synchronization misconfiguration, now fixed. Forty customers were affected, six of whom made ordering decisions based on the information. Sales argues for fixing it quietly to avoid damaging trust.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 修正済みなので、顧客への連絡は行わない / Do not contact customers, since the issue is fixed
- B) 顧客から問い合わせがあった場合のみ説明する / Explain only if a customer asks
- C) 全顧客に一斉メールで軽く触れる / Mention it lightly in a mass email to all customers
- D) **影響の内容に応じて、顧客に事実を伝える**。とりわけ発注判断に影響した可能性のある 6 社には個別に連絡し、影響を受けた期間・対象データ・生じ得た影響・原因と実施した対策を具体的に伝える。残る 34 社にも影響の範囲を通知する。あわせて、顧客側で必要な確認や是正（発注内容の見直し）を支援する。契約上の通知義務や補償条項があれば、それに従って対応する / **Tell customers, with the depth matched to the impact.** Contact the six whose ordering decisions may have been affected individually, stating the affected period, the data involved, the possible consequences, and the cause and remediation; notify the remaining 34 of the scope. Offer to help customers verify and correct anything on their side, such as revisiting orders placed, and follow any contractual notification or remedy provisions

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**顧客が誤った情報に基づいて意思決定を行った可能性がある場合、それを伝えないことは信頼の保護ではなく毀損**です。6 社は既に発注判断を行っており、その判断が誤った前提に基づいていた可能性を知る必要があります。伝えないという選択は、顧客が後から自力で気づいた場合に、はるかに深刻な信頼喪失を招きます。伝え方の要点は、影響の大きさに応じて対応を分けること（個別連絡と全体通知）と、**顧客側で必要な是正を支援する**ことで、これが誠実な対応と単なる報告の違いになります。

When customers may have decided on false information, withholding it damages trust rather than protecting it. Six have already placed orders on a possibly wrong premise and need to know. Silence turns into a far worse breach if they discover it themselves. Two things distinguish a good disclosure: matching depth to impact, and helping customers remediate on their side.

- **A 不正解**: 顧客側で行われた意思決定に影響が残っており、修正だけでは対応が完結しません。 / The fix does not address decisions already made on the customer side.
- **B 不正解**: 顧客は誤りに気づく手段がないため、受動的な説明は実質的に非開示です。 / Customers cannot know to ask; this is non-disclosure in practice.
- **C 不正解**: 影響の大きい 6 社に個別の説明が届かず、必要な是正支援も行われません。 / The six most affected receive neither individual notice nor help.

**参照 / Reference:** インシデントの顧客コミュニケーション、影響に応じた対応、是正の支援
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

AI プラットフォームチームに対して、5 つの事業部門から同時に要望が来ています。いずれも「自部門が最優先」と主張し、それぞれ異なる期限を提示しています。チームの開発能力は同時に 2 件が限界です。過去には、声の大きい部門の要望が優先され、後から「なぜあの案件が先なのか」という不満が繰り返し出ていました。

Five business units are simultaneously requesting work from the AI platform team, each claiming top priority with a different deadline. The team can handle two concurrently. Historically, the loudest unit's request went first, with recurring complaints afterwards about why that project was chosen.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate approach?

- A) 要望が来た順に着手する / Work in the order requests arrived
- B) 最も早い期限を提示した部門から着手する / Start with the unit citing the earliest deadline
- C) **優先順位の判断基準を公開し、意思決定の場を設ける**。事業インパクト（金額または影響人数）、緊急性の根拠（規制期限など外部要因の有無）、実現の確からしさ、他案件への波及効果といった軸で各案件を評価し、その結果を全部門に見える形で示す。優先順位の決定は、プラットフォームチームではなく事業横断のガバナンス体制（各部門の責任者が参加する場）で行う。着手できない案件については、待ち行列の位置と見込み時期を伝える / **Publish the prioritization criteria and create a decision forum.** Score each request on business impact (value or people affected), the basis of its urgency (external drivers such as regulatory deadlines), feasibility, and knock-on value for other work, and make the scoring visible to all units. Have a cross-functional governance forum with unit leaders — not the platform team — make the call, and tell unqueued requests where they sit and when they are likely to start
- D) 5 件すべてに同時に着手して、全部門に対応する / Start all five at once to serve everyone

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

繰り返される不満の原因は、優先順位そのものより**判断の根拠と決定の主体が不透明であること**です。基準を公開すれば、選ばれなかった部門も「なぜ」を理解でき、次回に向けて自分の案件の位置づけを改善できます。さらに重要なのが**決定の主体**で、プラットフォームチームが事業部門間の優先順位を決めると、技術チームが事業判断を代行する構図になり、どの決定にも正当性が伴いません。事業横断のガバナンス体制で決めることが、決定に権威を与えます。

The recurring complaint is caused less by the ordering than by opaque criteria and an unclear decision-maker. Published criteria let unselected units understand why and improve their case next time. More importantly, when the platform team sets priorities between business units, a technical team is making business decisions and no outcome carries legitimacy — a cross-functional forum is what gives the decision authority.

- **A 不正解**: 到着順は事業価値と無関係で、緊急性の高い案件が後回しになります。 / Arrival order is unrelated to value or urgency.
- **B 不正解**: 提示された期限は自己申告であり、早い期限を出した方が優先される仕組みは操作可能です。 / Self-declared deadlines are gameable.
- **D 不正解**: 能力の 2.5 倍の案件に着手すれば、全案件が遅延し、どの部門も満足しません。 / Overcommitting delays everything and satisfies no one.

**参照 / Reference:** 優先順位付けの透明性、事業横断ガバナンス、意思決定主体の明確化
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

社内向けエージェントの機能のうち、「週次レポート自動生成」を廃止したいと考えています。この機能は保守コストが高く、利用者は全社で 23 名です。しかしその 23 名は、この機能を業務プロセスに組み込んでおり、週次の部門会議の資料として使っています。開発チームは「利用者が少ないので、来週のリリースで削除する」と計画しています。

You want to retire the "weekly report auto-generation" feature of an internal agent. It is expensive to maintain and has 23 users company-wide. Those 23 have built it into their process and use its output as material for weekly departmental meetings. The team plans to remove it in next week's release.

**設問 / Question:**

最も適切な進め方はどれですか？

What is the most appropriate way to proceed?

- A) 利用者が少ないので、予定どおり来週削除する / Remove it next week as planned, given the small user base
- B) **利用者への影響を確認し、移行の道筋を示したうえで段階的に廃止する**。23 名がこの機能を何に使い、廃止後に何が困るのかを確認する。代替手段（既存のダッシュボード、簡易な集計、他機能での代替）を提示し、移行に必要な期間を確保する。廃止予定日を事前に告知し、移行状況を確認してから削除する。代替が用意できない場合は、廃止の判断自体を見直すか、影響を受ける部門の責任者と合意する / **Confirm the impact, offer a migration path, and retire in stages.** Establish what the 23 use it for and what breaks without it, present alternatives (an existing dashboard, a simpler aggregation, another feature), and allow time to migrate. Announce the removal date in advance and confirm migration before deleting. Where no alternative exists, either revisit the decision or agree it explicitly with the affected departments' leaders
- C) 削除後に問い合わせがあった部門にのみ対応する / Handle only the departments that complain after removal
- D) 機能を残したまま、動作を徐々に遅くして利用を減らす / Keep it but progressively slow it down to discourage use

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**利用者数の少なさは、その利用者にとっての重要度とは無関係**です。23 名は業務プロセスに組み込んでおり、週次会議という定例業務が依存しています。廃止の判断自体は妥当であり得ますが、進め方には移行の道筋と告知期間が必要です。要点は、代替手段を示すこと（何も示さずに廃止すると業務が止まります）と、代替が用意できない場合に**廃止の判断を見直すか、影響を受ける側と合意する**という選択肢を持つことです。

A small user count says nothing about how much those users depend on it: 23 people have wired it into a recurring meeting. The decision to retire may well be right, but the execution needs a migration path and notice. Two things matter — presenting an alternative, since removal without one simply stops work, and being willing either to revisit the decision or to agree it explicitly with the affected leaders when no alternative exists.

- **A 不正解**: 移行期間も代替手段もない削除は、23 名の定例業務を直接止めます。 / Halts a recurring process with no notice or alternative.
- **C 不正解**: 事後対応では、既に会議資料が用意できない状態が生じています。 / By then the meetings have already lost their material.
- **D 不正解**: 意図的な劣化は不誠実であり、原因不明の不具合として無駄な調査を招きます。 / Dishonest, and generates wasted investigation of a phantom defect.

**参照 / Reference:** 機能の廃止手順、移行の道筋、事前告知、影響を受ける側との合意
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

使用モデルの更新に伴い、出力の文体と詳細度が変わることが評価で分かりました。品質指標は改善していますが、出力を日常的に読んでいる 60 名の業務担当者から見ると「書き方が変わった」と感じられる程度の違いがあります。過去に別システムで無告知の変更を行った際、「システムが壊れた」という問い合わせが 40 件寄せられ、対応に 2 日を要しました。

A model update changes output style and level of detail. Quality metrics improve, but to the 60 staff who read this output daily the writing will noticeably differ. On a previous system, an unannounced change generated 40 "the system is broken" tickets and two days of response effort.

**設問 / Question:**

最も適切な進め方はどれですか？

What is the most appropriate way to proceed?

- A) 品質が改善しているので、告知せずに更新する / Update without notice, since quality improves
- B) 出力の文体を旧モデルに合わせるよう調整してから更新する / Tune the output to match the old style before updating
- C) 更新を見送り、旧モデルを使い続ける / Defer the update and stay on the old model
- D) **変更内容を事前に告知し、利用者が変化を予期できるようにする**。何が変わるか（文体と詳細度）、なぜ変えるのか（品質指標の改善）、いつ変わるかを事前に伝え、可能であれば変更前後の出力例を示す。問い合わせ窓口を明示し、変更直後は問い合わせが増えることを前提に体制を用意する。変更後にフィードバックを集め、想定外の影響があれば対応する / **Announce the change in advance so users can anticipate it**: state what changes (style and detail), why (measured quality improvement), and when, showing before-and-after examples where possible. Name a support channel, staff for an initial spike in questions, and collect feedback after the change to catch unanticipated effects

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**利用者にとっての「変化」は、それ自体が不安の源**です。品質が改善していても、予期しない変化は「壊れた」と受け取られます。過去に 40 件の問い合わせが発生したという経験は、告知の欠如が実際にコストを生むことの証拠です。事前告知の要点は、**何が・なぜ・いつ変わるかを具体的に示すこと**と、変更前後の例を見せることで、これにより利用者は変化を「予期された改善」として受け取れます。問い合わせ増を織り込んだ体制も、実務上必要な準備です。

Change itself is what unsettles users: an unanticipated shift reads as breakage even when quality improved, and the 40 previous tickets prove the cost is real. Effective notice states what, why, and when concretely and shows before-and-after examples, so users receive the change as an anticipated improvement. Staffing for the initial question spike is the practical other half.

- **A 不正解**: 過去に同じ判断で 40 件の問い合わせが発生しており、同じ結果が予測されます。 / The same decision previously produced 40 tickets.
- **B 不正解**: 文体を旧モデルに合わせる調整は、品質改善の一部を打ち消し、恒久的な負債にもなります。 / Cancels part of the gain and creates lasting coupling to an old style.
- **C 不正解**: 更新の見送りは品質改善を捨て、いずれ非推奨化で移行が必要になります。 / Forgoes the improvement and merely defers a forced migration.

**参照 / Reference:** 変更の事前告知、利用者への影響の説明、問い合わせ体制の準備
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

顧客の医療情報を扱うエージェントの開発が 3 か月進み、実装が 8 割完了した段階で、法務部門に確認を取りました。その結果、「同意の取得方法が現行の同意書ではカバーされていない」「データの保存場所が委託契約の範囲外」という指摘を受け、アーキテクチャの大幅な変更が必要になりました。開発チームは「早く相談すればよかった」と振り返っています。

Three months and 80% into implementing an agent handling customers' medical information, you consult legal. They find that the existing consent form does not cover the intended use and that the planned storage location falls outside the processing agreement, requiring substantial architectural change. The team reflects that they should have asked sooner.

**設問 / Question:**

今後のプロジェクトに向けた最も適切な改善はどれですか？

What is the most appropriate improvement for future projects?

- A) **法務・コンプライアンスを設計段階から関与させる仕組みを作る**。データの種類（個人情報、機微情報の有無）、処理の目的、保存場所、第三者提供の有無といった制約は、実装ではなく設計を左右する要素であるため、着手時のチェックリストとして確認し、該当する場合は設計レビューに法務が参加する。プロジェクト開始のゲートに「データと法的制約の確認」を含め、確認結果を設計文書に記録する / **Bring legal and compliance in at design time as a process.** Constraints such as the categories of data (personal, sensitive), the processing purpose, storage location, and third-party disclosure determine the architecture rather than the implementation, so make them a start-of-project checklist and have legal join the design review where they apply. Add a "data and legal constraints" gate to project initiation and record the findings in the design documentation
- B) 法務への確認を実装完了後に行うルールにする / Standardize on consulting legal after implementation is complete
- C) 法務部門に開発チームの一員として常駐してもらう / Embed a legal staff member full-time in every development team
- D) 医療情報を扱うプロジェクトを今後行わない / Stop taking on projects involving medical information

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

法務上の制約は、**実装の細部ではなく設計の前提を決める要素**です。同意の範囲やデータの保存場所は、アーキテクチャそのものを規定するため、8 割完成してから判明すると手戻りが大きくなります。有効な仕組みは、着手時のチェックリスト（データの種類、目的、保存場所、第三者提供）で該当性を判定し、該当する場合に設計レビューへ法務を招くという**軽量だが確実に発火する**運用です。すべてのプロジェクトに法務が常駐する必要はなく、発火条件を明確にすることが要点です。

Legal constraints determine design premises, not implementation details: consent scope and storage location shape the architecture, so discovering them at 80% is expensive. What works is a lightweight but reliably triggered process — an initiation checklist (data categories, purpose, storage, disclosure) that determines applicability, and legal joining the design review when it applies. Full-time embedding is unnecessary; a clear trigger is what matters.

- **B 不正解**: 実装完了後の確認は、まさに今回の失敗を制度化するものです。 / Institutionalizes exactly the failure that occurred.
- **C 不正解**: 全プロジェクトへの常駐は法務のリソース的に非現実的で、大半のプロジェクトでは過剰です。 / Not feasible and disproportionate for most projects.
- **D 不正解**: 適切な設計で実現可能な領域を放棄する判断です。 / Abandons a domain that proper design can serve.

**参照 / Reference:** 法務・コンプライアンスの早期関与、着手時ゲート、設計を左右する制約
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

保険金支払査定を支援するエージェントを現場に導入しました。査定担当者への説明は 30 分の操作研修のみで、「Claude の推奨を参考に判断してください」と伝えました。導入後、担当者が推奨をそのまま採用する傾向が強く、また推奨が誤っていた事例で「システムがそう言ったから」という説明が査定記録に残るようになりました。担当者からは「どういうときに推奨を疑うべきか分からない」という声が出ています。

An agent assisting claims adjudication was rolled out with a 30-minute operational training and the instruction to "use Claude's recommendation as a reference." Adjusters now largely adopt recommendations as given, and in cases where a recommendation was wrong, adjudication records cite "the system said so." Adjusters report not knowing when they should doubt a recommendation.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 推奨の採用率を評価指標にして、担当者に慎重な判断を促す / Make adoption rate a performance metric to encourage caution
- B) 推奨の表示をやめて、担当者が自力で判断するようにする / Stop showing recommendations so adjusters decide unaided
- C) **システムの限界と、疑うべき状況を具体的に伝えるイネーブルメントを行う**。どのような入力で誤りやすいか（評価で判明している弱い類型）、推奨に付随する不確実性の読み方、推奨と異なる判断をしてよい／すべき場面を、実際の誤答事例を用いて研修する。あわせて、画面に判断根拠と確信度を提示して疑う手がかりを与え、推奨と異なる判断をした場合の記録方法を明確にする。責任の所在は担当者にあることも改めて明示する / **Provide enablement on the system's limits and when to doubt it**: train on which input types it gets wrong (the weak categories identified in evaluation), how to read the uncertainty attached to a recommendation, and when departing from it is permitted or expected — using real error cases. Complement this by showing the rationale and confidence on screen so there is something to doubt, define how a departure is recorded, and restate that accountability rests with the adjuster
- D) 誤った推奨が出た事例について、担当者を指導する / Coach adjusters individually on cases where the recommendation was wrong

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

「参考にしてください」という指示だけでは、**どう参考にすればよいかが分かりません。**担当者が推奨をそのまま採用するのは、疑う手がかりも、疑ってよいという許可も与えられていないためです。有効なイネーブルメントは、**システムが誤りやすい類型を具体的に共有すること**で、これは評価から既に分かっている情報です。加えて、画面に判断根拠と確信度を出して疑う材料を与え、推奨と異なる判断をした場合の記録方法を定めることで、独立した判断が実際に可能な状態になります。

"Use it as a reference" does not say how. Adjusters adopt recommendations because they have neither cues for doubt nor permission to doubt. Effective enablement shares the specific input types the system gets wrong — information evaluation already provides — and pairs it with on-screen rationale and confidence so there is something concrete to question, plus a defined way to record a departure so independent judgment is actually available.

- **A 不正解**: 採用率を指標にすると、内容と無関係に推奨を退ける動機が生まれ、判断の質はむしろ下がります。 / Incentivizes rejecting recommendations irrespective of merit.
- **B 不正解**: 支援機能を取り上げるのは、導入目的そのものの否定です。 / Removes the capability the project exists to provide.
- **D 不正解**: 個別指導は事後的で、疑うべき状況を体系的に伝える手段になりません。 / Reactive, and does not convey the pattern systematically.

**参照 / Reference:** 現場のイネーブルメント、限界の共有、疑う手がかりの提示、責任の明確化
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

経営層から「競合他社が AI を導入して話題になっている。当社も 3 か月以内に何らかの AI 機能をリリースせよ」という指示が下りました。具体的な業務課題や対象ユーザーの指定はありません。事業部門に聞いても「経営指示なので」という回答で、解決したい問題が特定できていません。

Leadership has directed that "competitors are getting attention for AI; we must ship some AI feature within three months." No specific business problem or target user is specified. Asked, the business unit says only that it is a leadership directive; no problem to solve has been identified.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 指示どおり、3 か月で作れそうな AI 機能を選んでリリースする / Pick whatever AI feature is buildable in three months and ship it
- B) **指示の背後にある目的を確認し、具体的な課題に接続する**。「AI を入れること」ではなく、経営が達成したいこと（対外的な訴求、業務効率、顧客体験の改善など）を確認する。そのうえで、既存の業務課題のうち AI が実際に価値を出せるものを短期間で洗い出し（現場ヒアリング、コスト構造の分析）、3 か月で成果を示せる範囲の候補を、期待できる効果とともに提示する。指示を拒否するのでも盲従するのでもなく、目的に沿った具体案に翻訳する / **Clarify the objective behind the directive and connect it to a concrete problem.** Establish what leadership actually wants — external positioning, operating efficiency, customer experience — then run a short discovery across existing pain points (front-line interviews, cost analysis) to find where AI genuinely creates value, and present candidates achievable in three months with their expected effect. Neither refuse nor comply blindly: translate the directive into a specific proposal aligned to its purpose
- C) 経営指示は目的が不明確であるとして、実施を保留する / Put the work on hold, citing an unclear objective
- D) 競合他社と同じ機能を調査して、同等のものを実装する / Study the competitor's feature and implement an equivalent

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

「AI を入れよ」という指示は**手段の指定であって目的ではありません。**背後には対外的な訴求や業務改善といった目的があるはずで、それを確認しないまま実装すると、目的を達成しない機能ができます。アーキテクトの役割は、指示を拒否することでも盲従することでもなく、**目的を確認して具体的な課題に接続し、実現可能な選択肢に翻訳すること**です。3 か月という制約は、この作業を省略する理由ではなく、候補を絞り込む条件として扱います。短期間の探索で候補を出せば、指示にも応えつつ実質的な価値を狙えます。

"Add AI" specifies a means, not an end. There is an objective behind it — positioning, efficiency, experience — and building without establishing it produces a feature that does not serve it. The architect neither refuses nor complies blindly but clarifies the objective, connects it to a real problem, and translates it into feasible options. The three-month constraint is a filter on candidates, not a reason to skip the step.

- **A 不正解**: 作れるものを作る進め方では、目的を達成する保証がなく、投資が無駄になる可能性が高いです。 / Building what is buildable does not serve any objective.
- **C 不正解**: 保留は経営の関心に応えておらず、目的の確認は保留せずとも実施できます。 / The clarification can be done without stalling.
- **D 不正解**: 競合の模倣は、自社の課題と顧客に適合する保証がありません。 / Imitation carries no guarantee of fit to your own problems.

**参照 / Reference:** 指示の目的の確認、課題への接続、制約下での候補提示
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

エージェント導入から 8 か月が経過しました。導入時には「年間 4,200 万円のコスト削減」を見込んでいましたが、実際に削減が達成されたかを検証していません。経営層から「投資対効果はどうなっているのか」と質問がありました。社内には、システムの稼働状況（処理件数、稼働率）のデータはありますが、業務工数の変化を示すデータはありません。

Eight months after deployment, the projected ¥42M annual saving has never been verified. Leadership asks about return on investment. Internally, system operational data exists (volume processed, availability), but nothing measures the change in business effort.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 処理件数のデータをもって、効果が出ていると説明する / Present volume processed as evidence of benefit
- B) 効果測定は困難であると説明し、定性的な評価に留める / Explain that measurement is difficult and offer a qualitative assessment
- C) 導入時の見込み額をそのまま実績として報告する / Report the projected figure as the realized result
- D) **効果の実測を行い、以後は継続的に追跡する仕組みを作る**。削減効果の根拠となった業務（対象工程、担当者数、1 件あたりの所要時間）について現状を測定し、導入前の値と比較する。差が見込みと乖離している場合は、その理由（想定より対象範囲が狭かった、人手確認が残っている、稼働率が低いなど）を分析して示す。あわせて、以後は効果指標を定期的に測定する運用を組み込み、次回以降の投資判断の材料にする / **Measure the realized benefit and build continuous tracking.** Measure the current state of the processes the saving was based on — steps covered, headcount, time per item — and compare against the pre-deployment baseline. Where the gap differs from the projection, analyze and present why (narrower scope than assumed, residual manual checking, low utilization). Then institutionalize periodic measurement of the benefit metrics so future investment decisions have evidence

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**投資対効果は、見込みではなく実測で示すもの**です。処理件数や稼働率はシステムが動いていることを示すだけで、業務工数が減ったかどうかは示しません。実測すると見込みと乖離していることが多く、その理由（対象範囲が狭かった、人手確認が残った、利用率が低い）を分析して示すことに価値があります。乖離を隠さず示すことは、次の投資判断の精度を上げ、提案の信頼性を高めます。効果測定を継続的な運用に組み込むことが、同じ問いに毎回答えられる状態を作ります。

Return is demonstrated by measurement, not projection. Volume and availability show the system runs; they say nothing about effort. Measured results often diverge from the projection, and analyzing why — narrower scope, residual manual work, low utilization — is where the value lies. Presenting the gap honestly improves the next investment decision and the credibility of future proposals, and institutionalizing the measurement means the question can be answered whenever it is asked.

- **A 不正解**: 処理件数は稼働の指標であって、コスト削減の証拠になりません。 / Volume evidences operation, not saving.
- **B 不正解**: 測定は可能であり（対象工程の工数を測る）、困難という説明は成立しません。 / Measurement is feasible; the difficulty claim does not hold.
- **C 不正解**: 見込みを実績として報告するのは事実に反し、次回以降の見込みの信頼性も失わせます。 / Reporting a projection as a result is false and destroys future credibility.

**参照 / Reference:** 効果の実測、ベースラインとの比較、乖離の分析、継続的な効果追跡
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

新しいエージェントの導入にあたり、事業部門から「精度はどのくらいですか」と質問されました。評価結果は、主要タスクで 91%、そのうち誤りの 7 割は軽微なもの（表現の不適切さ）、3 割は業務上意味のある誤り（金額や日付の誤り）です。処理件数は月 5,000 件です。「91% です」と回答したところ、「では 9% は使えないということですね」と受け取られました。

Asked "what's the accuracy?" for a new agent, you have measured 91% on the primary task, with 70% of errors being minor (awkward phrasing) and 30% being operationally meaningful (wrong amounts or dates). Volume is 5,000 items/month. You answered "91%," and it was heard as "so 9% is unusable."

**設問 / Question:**

最も適切な説明の仕方はどれですか？

Which way of explaining is most appropriate?

- A) **業務上の意味に換算し、誤りの種類ごとに示す**。「月 5,000 件のうち、業務上意味のある誤りは約 135 件（2.7%）、表現上の軽微な誤りが約 315 件（6.3%）」と分けて示す。そのうえで、意味のある誤りがどのように検知でき、どの工程で吸収されるのか（人手確認、下流の突合）を説明し、「残るリスクはどれか」を具体的に示す。数字そのものより、業務にとって何を意味するかが判断材料になる / **Translate into operational terms and separate by error type**: "of 5,000 monthly items, about 135 (2.7%) carry an operationally meaningful error and about 315 (6.3%) are minor phrasing issues." Then explain how the meaningful errors are detected and where they are absorbed (human check, downstream reconciliation), and state what risk remains. What the number means for the business is the decidable part, not the number itself
- B) 「91%」とだけ回答し、解釈は相手に委ねる / Answer "91%" and leave the interpretation to them
- C) 精度を「ほぼ問題ないレベル」と表現する / Describe it as "essentially fine"
- D) 誤りの 7 割は軽微なので、実質的な精度は 97.3% であると説明する / Say that since 70% of errors are minor, effective accuracy is 97.3%

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

「91%」という数字だけでは、事業部門は判断できません。**件数に換算し、誤りの種類で分けて示す**ことで、はじめて「何を心配すべきか」が具体的になります。さらに重要なのが、**誤りが検知・吸収される仕組みの説明**です。月 135 件の意味のある誤りのうち、下流の突合で捕捉されるものと、そのまま通るものとでは、残るリスクがまったく異なります。数字を業務の言葉に翻訳し、残るリスクを明示することが、期待値の管理の実務です。

A bare "91%" is not decidable. Converting to counts and separating by error type makes the concern concrete, and explaining where errors are detected and absorbed matters more still: of 135 meaningful errors a month, those caught by downstream reconciliation and those that pass through represent very different residual risk. Translating into business language and naming what remains is the substance of expectation management.

- **B 不正解**: 解釈を委ねた結果が、本問で起きている誤解（「9% は使えない」）です。 / Leaving interpretation open is what produced the misunderstanding.
- **C 不正解**: 曖昧な表現は、過大な期待か過小な評価のいずれかを生みます。 / Vagueness produces either inflated or deflated expectations.
- **D 不正解**: 軽微な誤りを分母から除いて精度を高く見せるのは、都合の良い定義変更です。軽微な誤りも利用者には見えます。 / Redefining the metric to look better; minor errors are still visible to users.

**参照 / Reference:** 精度の伝え方、件数への換算、誤りの類型化、残存リスクの明示
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

プロジェクト開始から 2 か月、当初の前提が成立しないことが判明しました。「社内文書の 9 割が構造化されて検索可能」という前提でしたが、実際には 3 割で、残りはスキャン画像と個人フォルダに散在していました。この前提はプロジェクトの成立要件そのもので、成果の見込みが大きく変わります。プロジェクトマネージャーは「まだ挽回できるかもしれないので、次の定例まで様子を見よう」と言っています。

Two months in, a founding assumption has proven false: 90% of internal documents were believed to be structured and searchable, but the real figure is 30%, with the rest in scanned images and personal folders. The assumption was the project's basis, and expected outcomes change substantially. The project manager suggests waiting until the next regular meeting in case it can be recovered.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 次の定例まで待って、状況が改善するか見る / Wait for the next regular meeting to see whether it improves
- B) 前提が崩れたことは伏せて、達成できる範囲で成果を出す / Say nothing and deliver whatever is achievable
- C) **前提の崩壊を速やかにステークホルダーに報告し、選択肢を提示する**。判明した事実（構造化率 3 割）、それが成果見込みに与える影響、取り得る選択肢（文書の構造化に投資して当初計画を継続する、対象を構造化済み文書に絞って範囲を縮小する、中止する）を、それぞれの追加コストと期待効果とともに示す。判断はステークホルダーに委ね、決定を待つ間の作業方針も合わせて提案する / **Report the failed assumption promptly and present options**: the finding (30% structured), its effect on expected outcomes, and the available paths — invest in structuring the documents and continue as planned, narrow scope to the already-structured corpus, or stop — each with its added cost and expected benefit. Leave the decision to the stakeholders and propose how work proceeds while they decide
- D) 前提を満たすため、独断で文書の構造化プロジェクトを開始する / Unilaterally start a document-structuring project to restore the assumption

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**プロジェクトの成立要件が崩れたことは、ステークホルダーの意思決定事項**です。継続・縮小・中止のいずれを選ぶかは投資判断であり、技術チームが単独で決めるものではありません。報告を遅らせるほど選択肢は狭まり、投じたコストも増えます。報告の要点は、事実だけでなく**選択肢とそれぞれのコスト・効果を示すこと**で、これがないとステークホルダーも判断できません。悪い知らせは早く、選択肢とともに伝えるのが原則です。

A failed founding assumption is a stakeholder decision: continue, narrow, or stop is an investment call, not a technical team's to make alone. Every week of delay narrows the options and increases sunk cost. The report must carry not just the finding but the options with their costs and benefits, or the stakeholders cannot decide either. Bad news early, with choices attached.

- **A 不正解**: 待っても構造化率は変わりません。時間の経過は選択肢を狭めるだけです。 / Waiting does not change the figure and only narrows the options.
- **B 不正解**: 前提の崩壊を伏せることは、ステークホルダーから投資判断の機会を奪います。 / Deprives stakeholders of the decision that is theirs to make.
- **D 不正解**: 別プロジェクトの開始は大きな投資判断であり、独断で行える範囲を超えています。 / Launching a second project is an investment decision beyond your authority.

**参照 / Reference:** 前提の崩壊の報告、選択肢の提示、投資判断の返却
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

問い合わせ対応の自動化プロジェクトで、事業責任者が「最終的には人間のオペレーターをゼロにしたい」と繰り返し述べています。分析すると、問い合わせの 62% は定型的で自動化に適していますが、23% は判断を要し、15% は感情的な対応や例外的な状況（苦情、事故、契約解除の交渉）を含みます。後者の類型は自動化の適合性が低く、誤対応のコストも高い領域です。

In a support-automation project, the business owner repeatedly states the goal of eliminating human operators entirely. Analysis shows 62% of inquiries are routine and suitable for automation, 23% require judgment, and 15% involve emotional handling or exceptional situations (complaints, accidents, cancellation negotiations) — a category poorly suited to automation where mishandling is costly.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 目標に合わせて、全類型の自動化を目指す計画を立てる / Plan for full automation of all categories, matching the stated goal
- B) **分析結果を示し、自動化の目標を類型ごとに再設定する**。62% の定型対応を高い水準で自動化することが最大の価値であり、そこに集中すれば人員を判断業務と例外対応に振り向けられることを示す。15% の類型については、誤対応のコスト（顧客離反、紛争、規制上の問題）を具体的に示して、自動化が適さない理由を根拠づける。「オペレーターゼロ」ではなく「オペレーターが高付加価値な対応に集中できる状態」を目標として再合意する / **Present the analysis and reset the automation target per category.** The largest value is automating the routine 62% to a high standard, which frees staff for judgment work and exceptions. For the 15%, quantify the cost of mishandling — churn, disputes, regulatory exposure — as the basis for why automation does not fit. Re-agree the goal as "operators concentrating on high-value interactions" rather than "zero operators"
- C) 事業責任者の目標は非現実的であると伝え、それ以上は議論しない / Tell the owner the goal is unrealistic and end the discussion
- D) 全類型を自動化し、問題が起きた類型のみ後から人間に戻す / Automate everything and return categories to humans where problems arise

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

「オペレーターゼロ」という目標は、**手段（自動化率）を目的化したもの**です。背後にある事業目的（コスト削減、対応品質の向上）は正当なので、それを達成する現実的な形に再定義するのがアーキテクトの役割です。有効なのは、分析に基づいて**類型ごとに自動化の適否とコストを示すこと**で、特に 15% の類型については誤対応のコストを具体的に示すと、自動化しない判断が事業的にも合理的であることが伝わります。目標の再合意は、拒否ではなく、より達成可能で価値の高い形への翻訳です。

"Zero operators" elevates a means into an end. The underlying objective — cost and quality — is legitimate, so the architect's job is redefining it into an achievable form. What works is category-level analysis of suitability and cost, and for the 15%, quantifying the cost of mishandling shows that *not* automating is the commercially rational choice. Re-agreeing the goal is translation, not refusal.

- **A 不正解**: 適合性の低い領域まで自動化すると、誤対応のコストが自動化の利得を上回る可能性が高いです。 / Automating the ill-suited categories likely costs more than it saves.
- **C 不正解**: 目標を否定するだけでは、事業責任者の正当な関心に応えていません。 / Rejecting the goal does not address the legitimate interest behind it.
- **D 不正解**: 苦情や事故対応で失敗してから戻すのは、顧客関係に回復困難な損害を与えます。 / Failing first on complaints and accidents causes irreversible relationship damage.

**参照 / Reference:** 目標の再定義、類型ごとの自動化適合性、誤対応コストの提示
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

3 か月の開発を経て、エージェントが本番稼働しました。開発チームは次のプロジェクトに移る予定です。しかし、本番稼働後に必要となる作業（監視、障害対応、評価データセットの更新、モデル更新への追随、利用者からの問い合わせ対応、プロンプトの改善）について、誰が担当するかが決まっていません。運用部門は「AI システムの運用経験がない」と述べています。

After three months of development the agent is live and the team is moving to the next project. No one owns the post-launch work: monitoring, incident response, evaluation-set updates, keeping up with model updates, user inquiries, and prompt improvements. The operations department says it has no experience running AI systems.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 開発チームが次のプロジェクトと兼務で運用も担当する / Have the development team run it alongside the next project
- B) 運用部門に引き継ぎ、経験がない点は問題視しない / Hand it to operations and disregard the experience gap
- C) 運用は発生した時点で誰かが対応することにする / Deal with operational work ad hoc as it arises
- D) **運用に必要な作業を明示し、担当と手順を定めたうえで引き継ぐ**。監視すべき指標と閾値、障害時の対応手順とエスカレーション先、評価データセットの更新頻度と手順、モデル更新への対応手順、問い合わせの一次受けと開発への切り分け基準を文書化する。運用部門に対しては、AI 特有の運用（品質劣化の検知、評価の実行、プロンプト変更の手続き）について研修を行い、移行期間は開発チームが並走する。引き継ぎ完了の条件を定義する / **Define the operational work, assign owners and procedures, and then hand over.** Document the metrics and thresholds to monitor, the incident procedure and escalation path, the cadence and steps for updating the evaluation set, the procedure for model updates, and the triage criteria between first-line support and development. Train the operations team on the AI-specific parts (detecting quality degradation, running evaluations, the prompt-change process), have development run alongside during a transition period, and define what completed handover means

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**本番稼働はプロジェクトの終わりではなく、運用の始まりです。**AI システムの運用には、通常のシステム運用にない要素（品質劣化の検知、評価の実行、モデル更新への追随、プロンプト変更の手続き）が含まれるため、「運用部門に渡す」だけでは機能しません。必要なのは、作業の明示・手順の文書化・研修・並走期間・引き継ぎ完了条件の 5 点です。引き継ぎ完了の条件を定義しないと、開発チームが実質的に運用を担い続けるか、誰も担当しない状態が続きます。

Go-live is the start of operations, not the end of the project. Running an AI system includes work ordinary operations does not cover — detecting quality degradation, running evaluations, tracking model updates, governing prompt changes — so "hand it to operations" does not work by itself. Five things are needed: named work, documented procedures, training, a period running in parallel, and a definition of completed handover. Without the last, development keeps running it informally or nobody does.

- **A 不正解**: 兼務は次のプロジェクトと運用の両方を圧迫し、障害時の対応も遅れます。恒久的な体制になりません。 / Squeezes both, delays incident response, and is not a durable structure.
- **B 不正解**: 経験のない部門に手順も研修もなく渡すのは、引き継ぎではなく放棄です。 / Handing over without procedures or training is abandonment.
- **C 不正解**: 担当が未定のままでは、障害時に誰も動かず、定期的な作業（評価の更新）は行われません。 / Undefined ownership means no response and no recurring work.

**参照 / Reference:** 運用引き継ぎ、AI 特有の運用作業、並走期間、引き継ぎ完了条件
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

リリース予定日の 3 週間前、評価で重要な問題が見つかりました。修正には 4 週間かかる見込みです。関係部門はリリース日に合わせて研修と告知を準備しており、営業は顧客に日程を伝えています。プロジェクトマネージャーは「まだ 3 週間あるので、直前に判断すればよい」と考えています。

Three weeks before the scheduled release, evaluation surfaces a significant defect requiring an estimated four weeks to fix. Other departments have prepared training and communications for the release date, and sales has told customers the schedule. The project manager thinks the call can wait until closer to the date.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 予定どおりリリースし、問題は次のバージョンで修正する / Release on schedule and fix in a later version
- B) リリース直前に判断する / Decide close to the date
- C) **直ちに関係部門に共有し、選択肢とともに判断を仰ぐ**。判明した問題の内容と影響、修正に要する期間、取り得る選択肢（延期する、問題のある機能を除いて範囲を絞ってリリースする、暫定的な回避策を入れてリリースする）を、それぞれのリスクとともに示す。関係部門は研修・告知・顧客への連絡の準備を進めており、判断が遅れるほど手戻りが大きくなるため、早期の共有そのものが価値を持つ / **Share it immediately and put the decision to the stakeholders with options**: the defect and its impact, the time to fix, and the available paths — postpone, ship a reduced scope excluding the affected capability, or ship with an interim workaround — each with its risks. Because other departments are actively preparing training, communications, and customer messaging, every day of delay increases their rework, so early disclosure has value in itself
- D) 修正を 3 週間で終わらせるよう開発チームに指示する / Direct the team to complete the fix in three weeks

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**判断を遅らせても選択肢は増えず、関係部門の手戻りだけが増えます。**研修資料の作成、告知の準備、顧客への連絡はいずれも進行中で、3 週間分の準備が無駄になるか、直前の変更で混乱が生じます。早期共有の価値は、判断の質だけでなく、**関係部門が対応する時間を確保できること**にあります。選択肢を示すのも重要で、「延期」以外に範囲を絞る案や暫定策があれば、ステークホルダーはより良い判断ができます。

Delaying the decision adds no options and multiplies other teams' rework: training materials, announcements, and customer messaging are all in flight, and three more weeks of that is either wasted or thrown into last-minute confusion. Early disclosure buys those teams time to adjust, which is value independent of decision quality. Presenting options beyond "postpone" — reduced scope, interim workaround — lets stakeholders choose better.

- **A 不正解**: 重要な問題を抱えたままのリリースは、利用者の信頼を最初の接触で損ないます。 / Shipping a significant defect spends user trust at first contact.
- **B 不正解**: 直前の判断は、関係部門の準備を無駄にし、対応の余地も奪います。 / Wastes the preparation and removes room to adapt.
- **D 不正解**: 見積もりを指示で短縮することはできず、品質を犠牲にした修正を招きます。 / Directives do not shorten estimates; they produce rushed, low-quality fixes.

**参照 / Reference:** 悪い知らせの早期共有、選択肢の提示、関係部門への影響の考慮
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

2 年間運用してきたエージェントを終了することが決まりました。後継の別システムに機能が統合されるためです。このエージェントには、2 年分の会話ログ、業務判断の記録、利用者が作成した設定やテンプレートが蓄積されています。一部の記録は規制上の保持義務の対象です。開発チームは「サービスを停止してインフラを削除する」ことを計画しています。

An agent that has run for two years is being retired, its functionality folded into a successor system. It holds two years of conversation logs, records of business decisions, and user-created configurations and templates. Some records fall under regulatory retention obligations. The team plans to shut down the service and delete the infrastructure.

**設問 / Question:**

最も適切な終了手順はどれですか？

Which decommissioning procedure is most appropriate?

- A) **保持すべきデータと利用者への影響を整理したうえで、計画的に終了する**。規制上の保持義務がある記録を特定し、保持期間を満たす形で移管または保管する（検索・提示できる状態を維持する）。利用者が作成した設定やテンプレートについては、後継システムへの移行手段を提供するか、取り出せる形でエクスポートを可能にする。終了日を事前に告知し、移行期間を設ける。終了後の問い合わせ先も明示する / **Establish what must be retained and how users are affected, then decommission on a plan.** Identify records under regulatory retention and migrate or archive them so the obligation is met and they remain retrievable and producible. For user-created configurations and templates, provide migration into the successor or an export path. Announce the end date in advance with a migration window, and name where inquiries go afterwards
- B) サービスを停止し、インフラを削除する / Shut down the service and delete the infrastructure
- C) データはすべて削除して、後継システムで新規に開始する / Delete all data and start fresh in the successor
- D) インフラを停止せず、そのまま放置する / Leave the infrastructure running untouched

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

システムの終了は、**データの保持義務と利用者への影響という 2 つの観点で計画する**必要があります。規制上の保持義務がある記録は、システムが消えても義務が消えるわけではなく、保持期間を満たす形で移管・保管し、**検索と提示ができる状態を維持する**必要があります（保存されていても取り出せなければ義務を果たせません）。利用者が作成した資産は、移行手段かエクスポートを提供しないと、2 年分の蓄積が失われます。終了日の告知と移行期間は、他の廃止と同様に必要です。

Decommissioning is planned along two axes: retention obligations and user impact. Regulatory retention survives the system, so records must be migrated or archived in a form that remains retrievable and producible — data preserved but unretrievable does not satisfy the obligation. User-created assets need migration or export, or two years of accumulation is lost. Advance notice and a migration window apply as they would to any retirement.

- **B 不正解**: 保持義務のある記録が失われ、利用者の資産も回収できなくなります。 / Destroys records under retention obligations and users' assets.
- **C 不正解**: 全削除は保持義務違反に直結します。 / Directly violates the retention obligations.
- **D 不正解**: 放置は運用コストと攻撃対象を残し、統制外のシステムになります。 / Leaves cost, attack surface, and an ungoverned system.

**参照 / Reference:** システムの終了計画、保持義務のあるデータの移管、利用者資産の移行、終了告知
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

社内の AI 活用が拡大し、技術、法務、リスク管理、事業部門、情報セキュリティがそれぞれ独自に判断や指針を出しています。結果として、ある部門が承認した構成が別の部門の指針に反する、同じ質問に部門ごとに異なる回答が返る、といった混乱が生じています。各部門は自分の領域では正しい判断をしています。

As AI use has grown, technology, legal, risk management, business units, and information security each issue their own decisions and guidance. The result is confusion: a configuration one function approves conflicts with another's guidance, and the same question gets different answers depending on whom you ask. Each function is making correct decisions within its own domain.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 最も厳しい部門の指針に全社を統一する / Standardize on the strictest function's guidance
- B) **事業横断のガバナンス体制を設けて、判断の場と責任を整理する**。各領域の代表が参加する意思決定の場を設け、全社共通の指針（リスク分類、承認基準、統制標準）をそこで定める。個別案件については、どの水準の判断を誰が行うかを明確にし、部門間で判断が競合する場合の調整経路を定める。既に出ている指針を突き合わせて矛盾を解消し、一本化した参照先を用意する / **Establish a cross-functional governance body that owns where decisions are made and by whom.** Create a forum with representatives from each domain to set common guidance (risk tiering, approval criteria, control standards); for individual cases, define which level of decision belongs to whom and the escalation path when functions conflict. Reconcile the guidance already issued, resolve contradictions, and publish a single reference
- C) 技術部門が全社の判断を一元的に行う / Have the technology function decide everything centrally
- D) 各部門の独立性を尊重し、現状のまま運用する / Respect each function's independence and continue as is

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

問題は個々の判断の質ではなく、**判断の場と責任が整理されていないこと**です。各部門が自領域で正しく判断していても、それらが調整されなければ全体として矛盾します。必要なのは、共通の指針を定める場と、**案件ごとにどの水準の判断を誰が行うかの明確化**、そして競合時の調整経路です。既存の指針を突き合わせて矛盾を解消し、一本化した参照先を用意することが、現場の混乱を実際に減らします。

The problem is not decision quality but the absence of a defined venue and ownership: correct decisions within silos still contradict each other when nothing reconciles them. What is needed is a forum that sets common guidance, clarity on which level of decision belongs to whom, and an escalation path for conflicts — plus reconciling the existing guidance into a single published reference, which is what actually reduces confusion on the ground.

- **A 不正解**: 最も厳しい指針への統一は、他領域の正当な考慮（実現可能性、事業価値）を捨てることになります。 / Discards other domains' legitimate considerations.
- **C 不正解**: 技術部門は法務やリスクの判断を行う権限も専門性も持ちません。 / Technology holds neither the authority nor the expertise for legal and risk judgments.
- **D 不正解**: 現状維持は、既に生じている混乱と矛盾を放置します。 / Leaves the observed contradictions in place.

**参照 / Reference:** 事業横断ガバナンス、判断権限の整理、指針の一本化
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

住宅ローン審査の一次判定にエージェントを使っています。窓口担当者は、申込者から「なぜ承認されなかったのか」と聞かれます。現在、担当者に提供されている情報は判定結果（承認・保留・否認）のみで、担当者は「システムの判定です」としか答えられません。申込者からの不満が増え、担当者も対応に苦慮しています。

An agent performs first-pass mortgage screening. Branch staff are asked by applicants why they were not approved. Staff currently see only the outcome — approve, hold, deny — and can say only "it's the system's decision." Applicant dissatisfaction is rising and staff find the situation difficult.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 窓口担当者に「システムの判定です」と説明する研修を行う / Train staff to deliver the "it's the system's decision" line
- B) 判定結果を申込者に伝えず、後日書面で通知する / Withhold the outcome at the counter and notify by letter later
- C) **判定の理由を担当者が説明できる形で提供する**。判定に寄与した主要因（返済比率、勤続年数、既存借入など）を構造化された形で出力させ、窓口の画面に表示する。担当者向けには、各要因の意味と、申込者に伝えてよい範囲・伝え方を含めた研修を行う。あわせて、申込者が改善できる要因については、その旨を伝えられるようにする。判定に納得できない場合の再審査の申し出方法も案内できるようにする / **Give staff the reasons in a form they can explain.** Have the system output the principal contributing factors (debt-to-income, tenure, existing borrowing) in structured form and display them at the counter. Train staff on what each factor means and what may be conveyed and how, enable them to point out factors an applicant could improve, and equip them to explain how to request a review of a decision the applicant disputes
- D) 窓口での説明をやめ、コールセンターに問い合わせてもらう / Stop explaining at the counter and refer applicants to a call center

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**「システムの判定です」は説明ではありません。**申込者にとって不透明であるだけでなく、窓口担当者を説明できない立場に置いています。判定の主要因を構造化された形で出力し、窓口に提示すれば、担当者は根拠を持って説明でき、申込者は次に何ができるかを理解できます。**担当者への研修**が不可欠で、要因の意味と伝えてよい範囲を理解していないと、誤った説明や不適切な開示が生じます。再審査の申し出方法を案内できることも、多くの法域で求められる要素です。

"It's the system's decision" is not an explanation: it is opaque to the applicant and leaves front-line staff unable to do their job. Structured principal factors displayed at the counter let staff explain with grounds and let applicants understand what they could change. Training is essential — without understanding what each factor means and what may be disclosed, staff will explain wrongly or over-disclose — and pointing to a review path is required in many jurisdictions.

- **A 不正解**: 説明できない状態を制度化するもので、申込者の不満も担当者の困難も解消しません。 / Institutionalizes the inability to explain.
- **B 不正解**: 通知を遅らせても説明できないことは変わらず、申込者の体験はさらに悪化します。 / Delays without explaining, and worsens the experience.
- **D 不正解**: たらい回しは不満を増幅させ、コールセンターも同じ情報しか持ちません。 / Deflection amplifies frustration, and the call center has the same information.

**参照 / Reference:** 判断理由の提示、現場担当者のイネーブルメント、再審査経路の案内
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

エージェント基盤の構築にあたり、ツールとサービスの選定を進めています。情報セキュリティは「オンプレミス完結」を、事業部門は「3 か月以内の稼働」を、財務は「初期投資 500 万円以内」を、開発チームは「既存のスキルセットで運用可能なこと」を、それぞれ譲れない条件として主張しています。すべてを同時に満たす選択肢は見つかっていません。

Selecting tooling for an agent platform, information security insists on fully on-premises, the business unit on going live within three months, finance on capital expenditure under ¥5M, and the development team on operability with existing skills — each stated as non-negotiable. No option satisfies all four.

**設問 / Question:**

最も適切な進め方はどれですか？

What is the most appropriate way to proceed?

- A) 最も強く主張している部門の条件を優先する / Prioritize whichever function argues most forcefully
- B) すべての条件を満たす選択肢が見つかるまで選定を続ける / Keep searching until an option satisfies all four
- C) 開発チームの判断で決定し、事後に各部門へ通知する / Decide within the development team and inform the others afterwards
- D) **条件の背後にある要件とその根拠を確認し、トレードオフを可視化して意思決定の場に上げる**。各条件が「絶対に譲れない制約」なのか「望ましい条件」なのかを、根拠とともに確認する（オンプレミス要件が規制由来なのか慣行なのか、3 か月が外部要因由来なのかなど）。そのうえで、実現可能な組み合わせを 2〜3 案に整理し、それぞれが何を満たし何を諦めるかを示して、事業横断の意思決定の場で決定する / **Establish the requirement and rationale behind each condition, make the tradeoffs explicit, and escalate to a decision forum.** Determine with evidence whether each is a hard constraint or a preference — is the on-premises requirement regulatory or customary, is the three months driven by an external commitment? Then reduce the feasible space to two or three options, state what each satisfies and what it gives up, and have a cross-functional forum decide

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

4 つの条件が同時に満たせない場合、**まず確認すべきは各条件が本当に絶対的な制約なのか**です。「オンプレミス完結」が規制由来なら動かせませんが、慣行や漠然とした懸念に基づくものなら、統制を伴うクラウド利用で要件を満たせる場合があります。根拠を確認すると、多くの場合いくつかの条件には幅があることが分かります。そのうえで、実現可能な選択肢を絞ってトレードオフを明示し、**部門間の優先順位を決める権限を持つ場で決定する**のが正しい進め方です。技術チームが部門間の優先順位を決めるべきではありません。

When four conditions cannot be met together, the first question is which are genuinely hard. If the on-premises requirement is regulatory, it does not move; if it rests on custom or unspecific concern, a governed cloud deployment may satisfy the underlying need. Examining the rationale usually reveals slack in some conditions. Then narrow to feasible options, make the tradeoffs explicit, and let a forum with the authority to prioritize between functions decide — that is not the technical team's call.

- **A 不正解**: 主張の強さは要件の重要度と無関係で、正当性のない決定になります。 / Volume is unrelated to importance and yields an unjustifiable decision.
- **B 不正解**: 探し続けても存在しない可能性が高く、その間プロジェクトが停止します。 / Likely searches for something that does not exist while the project stalls.
- **C 不正解**: セキュリティや財務の制約を技術チームが単独で覆すことはできません。 / Security and finance constraints are not the technical team's to override.

**参照 / Reference:** 制約と選好の区別、トレードオフの可視化、意思決定の場への escalation
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

エージェント基盤の主要な設計判断（オーケストレーション方式、MCP サーバーの粒度、モデル階層化の方針など）を行ったメンバー 2 名が、相次いで異動しました。新しく参加したメンバーから「なぜこの構成になっているのか」という質問が繰り返し出ますが、答えられる人がいません。過去の判断の背景が分からないため、変更してよいのか判断できず、結果として誰も触れない領域が増えています。

The two people who made the platform's key design decisions — orchestration approach, MCP server granularity, model-tiering policy — have both transferred out. New members repeatedly ask why the system is built this way and no one can answer. Because the rationale is unknown, no one can judge whether a change is safe, and the set of untouchable areas keeps growing.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) **重要な設計判断を、背景・検討した選択肢・決定理由とともに記録する運用を始める**。既存の構成については、分かる範囲で経緯を再構成し、不明な部分は「理由不明」と明記したうえで、現時点で妥当かを再評価する。以後の判断については、決定時に「何を解決するための判断か」「他にどの選択肢を検討したか」「なぜこれを選んだか」「どの前提に依存しているか」を記録し、前提が変わったときに見直せるようにする / **Start recording significant design decisions with their context, the options considered, and the reasoning.** For the existing system, reconstruct what can be reconstructed, explicitly mark the rest as "rationale unknown," and re-assess whether those choices are still appropriate today. Going forward, record at decision time what problem the decision solves, which alternatives were weighed, why this one was chosen, and which assumptions it depends on — so it can be revisited when those assumptions change
- B) 異動したメンバーに個別に問い合わせて、口頭で確認する / Ask the transferred members individually and confirm verbally
- C) 現在の構成をすべて作り直して、新チームが理解できる形にする / Rebuild everything so the new team understands it
- D) 分からない部分には触れない方針を明文化する / Formalize a policy of not touching the parts nobody understands

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

設計判断の背景が失われると、**変更の可否を判断できなくなり、システムが硬直化します。**本問で「誰も触れない領域が増えている」のはその症状です。有効なのは、判断の記録を運用として持つことで、特に重要なのが**依存している前提を書き残すこと**です。前提が明示されていれば、状況が変わったときに「この判断はもう妥当でない」と判断できます。既存部分については、再構成できない箇所を「理由不明」と正直に記し、現時点での妥当性を評価し直すのが実務的です。

Losing decision rationale makes change unjudgeable and the system rigid — the growing set of untouchable areas is that symptom. The remedy is a decision-record practice, and the most valuable field is the assumptions a decision depends on: with those written down, a changed situation makes it clear when a decision no longer holds. For the existing system, honestly marking what cannot be reconstructed and re-assessing it today is the practical path.

- **B 不正解**: 口頭確認は記録に残らず、次に必要になったときに再び失われます。異動先の負担にもなります。 / Verbal answers are not retained and the loss recurs.
- **C 不正解**: 理解のための作り直しは膨大なコストがかかり、記録がなければ新しい構成でも同じことが起きます。 / Enormous cost, and without records the new system repeats the problem.
- **D 不正解**: 触れない領域を制度化すると、必要な変更もできなくなり硬直化が進みます。 / Institutionalizes the rigidity.

**参照 / Reference:** 設計判断の記録、依存する前提の明示、知識の継承
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

顧客との契約交渉で、顧客側から「AI 機能について、応答精度 99% を SLA として保証してほしい」という要求が出ました。社内の評価では、主要タスクの精度は 93% です。営業は「契約を取りたいので、なんとか合意できないか」と相談してきました。この機能は顧客の業務判断に使われます。

In a customer negotiation, the customer asks for a 99% response-accuracy SLA on the AI feature. Internal evaluation shows 93% on the primary task. Sales asks whether some form of agreement is possible in order to win the contract. The feature will inform the customer's business decisions.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 99% を保証する契約を締結し、達成に向けて改善を続ける / Sign the 99% commitment and work toward achieving it
- B) **達成できない水準は約束せず、測定可能で意味のある約束に置き換える**。精度 99% は現在の実力から乖離しており、そもそも「精度」の定義（何を正解とし、誰がどう測るか）を契約で共有できなければ紛争の種になる。代わりに、可用性、応答時間、インシデント時の対応時間、品質の継続的な測定と報告、品質が定義した水準を下回った場合の是正手順といった、**測定方法が明確で履行可能な内容**を提示する。精度については現在の実測値と測定方法を開示し、用途に応じた人手確認の設計を顧客と合意する / **Do not commit to a level you cannot meet; substitute commitments that are measurable and meaningful.** 99% is far from current capability, and without a contractual definition of accuracy — what counts as correct, measured by whom and how — the clause is a dispute waiting to happen. Offer instead availability, response time, incident response times, continuous measurement and reporting of quality, and a defined remediation procedure if quality falls below an agreed level. Disclose the measured accuracy and the measurement method, and agree with the customer on the human-verification design their use case needs
- C) 精度の定義を曖昧にしたまま 99% と契約する / Sign at 99% while leaving "accuracy" undefined
- D) 顧客の要求は非現実的であると伝え、交渉を打ち切る / Tell the customer the demand is unrealistic and end the negotiation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**達成できない水準を契約で約束することは、将来の紛争と信頼喪失を確定させる行為**です。加えて、AI の「精度」は測定方法によって値が変わるため、定義を共有せずに数値だけを契約に書くと、何をもって違反とするかで争いになります。実務的な解は、**測定方法が明確で履行可能な項目**（可用性、応答時間、インシデント対応、品質の測定と報告、是正手順）に置き換えることです。顧客の本当の関心は「業務判断に使えるか」なので、実測値の開示と人手確認の設計を合意する方が、顧客の目的にも適います。

Committing to an unachievable level guarantees a future dispute and lost trust. Worse, AI "accuracy" varies with measurement method, so a bare number without a shared definition becomes an argument about what counts as breach. The workable answer substitutes commitments that are unambiguously measurable and deliverable, and since the customer's real interest is whether they can rely on it for decisions, disclosing measured performance and agreeing the human-verification design serves them better.

- **A 不正解**: 現状 93% で 99% を約束するのは、違反を前提とした契約です。 / Commits to a breach.
- **C 不正解**: 定義を曖昧にするのは、紛争時に不利になるだけでなく、意図的な誤導に当たります。 / Deliberately misleading, and disadvantageous in a dispute.
- **D 不正解**: 代替案を示さずに打ち切るのは、顧客の正当な関心（信頼性の担保）に応えていません。 / Ends the conversation without addressing a legitimate concern.

**参照 / Reference:** SLA の設計、測定可能な約束、精度の定義、履行可能性
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

四半期の報告資料を作成しています。エージェントの品質評価は、全体で 87%、うち主要な 3 業務領域では 91%、92%、94% ですが、新しく追加した 1 領域は 58% にとどまっています。この領域は件数が少なく（全体の 6%）、まだ改善中です。上司から「良い数字を前面に出すように」と指示されました。

Preparing a quarterly report: overall quality is 87%, with the three main business areas at 91%, 92%, and 94%, while a newly added area sits at 58%. That area is small (6% of volume) and still being improved. Your manager has asked you to "lead with the good numbers."

**設問 / Question:**

最も適切な報告の仕方はどれですか？

Which reporting approach is most appropriate?

- A) 主要 3 領域の数値のみを報告する / Report only the three main areas
- B) 全体の 87% のみを報告し、内訳は示さない / Report only the overall 87% without a breakdown
- C) **全領域の数値を示したうえで、低い領域の状況と対応を説明する**。主要領域の成果を報告するとともに、新領域が 58% であること、その理由（データが少ない、業務の性質が異なるなど）、現在進めている改善と見込み、そして改善が完了するまでの運用上の扱い（人手確認を厚くしている等）を示す。低い数値を隠すと、後に判明したときに報告全体の信頼性が失われ、また支援が必要な領域に資源が配分されない / **Show all areas and explain the low one's situation and remediation.** Report the main areas' results alongside the new area at 58%, why (limited data, different task characteristics), the improvement work in progress with its expected trajectory, and how the area is handled operationally meanwhile (heavier human verification). Concealing the low number costs the credibility of the whole report when it surfaces, and prevents resources being directed where they are needed
- D) 新領域は試験運用中として、報告対象から除外する / Exclude the new area as still being piloted

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

報告の目的は**印象を良くすることではなく、意思決定に必要な情報を提供すること**です。58% という数値は、その領域に資源を配分すべきか、運用上どう扱うべきかを判断するために必要な情報です。低い数値を隠すと、後に判明したときに報告全体の信頼性が失われ、以後の良い数値も疑われるようになります。適切な報告は、低い数値を示すと同時に、**理由・対応・見込み・当面の運用上の扱い**を添えることで、これにより数値が問題ではなく管理された状況として伝わります。

The purpose of a report is to inform decisions, not to create an impression. The 58% is exactly what a decision-maker needs to allocate resources and set operational handling. Concealing it costs the credibility of the entire report when it emerges, and casts doubt on the favorable numbers too. A good report pairs the low number with cause, remediation, trajectory, and interim handling — which presents it as a managed situation rather than a problem hidden.

- **A 不正解**: 都合の良い領域のみの報告は、意思決定に必要な情報を欠いた選択的開示です。 / Selective disclosure that omits what decisions require.
- **B 不正解**: 全体値のみでは、特定領域の深刻な低さが平均に埋もれます。 / The aggregate buries a serious per-area gap.
- **D 不正解**: 稼働している以上、除外は実態の隠蔽です。 / The area is live; excluding it conceals reality.

**参照 / Reference:** 誠実な報告、内訳の開示、対応と見込みの併記
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

別部門から、既に稼働している大規模なエージェント基盤の保守を引き継ぐことになりました。引き継ぎ資料は、システム構成図 1 枚と、主要なファイル一覧です。前任チームは 2 週間後に別プロジェクトへ完全に移ります。あなたのチームはこの基盤を触ったことがありません。基盤は 4 つの事業部門の業務で日常的に使われています。

Your team is taking over maintenance of a large, live agent platform from another department. The handover materials are one architecture diagram and a list of key files. The outgoing team moves fully to another project in two weeks. Your team has never worked on this platform, which is used daily by four business units.

**設問 / Question:**

引き継ぎ期間に最も優先すべきことはどれですか？

What should the handover period prioritize most?

- A) **運用を継続できる状態を作ることを最優先する**。2 週間で全体を理解することは不可能なので、まず「動かし続ける」ために必要な知識に集中する。障害時に何が起き、どう対処するか（過去のインシデントとその対応）、監視している指標と閾値の意味、日常的に発生する運用作業とその手順、変更を加える際に壊れやすい箇所、エスカレーション先を、前任チームと一緒に実際に手を動かしながら確認する。設計の詳細な理解は引き継ぎ後に段階的に進める / **Prioritize being able to keep it running.** Understanding the whole platform in two weeks is impossible, so concentrate on what operation requires: what failures look like and how they were handled (past incidents and their resolutions), what the monitored metrics and thresholds mean, the recurring operational tasks and their procedures, which areas break easily when changed, and who to escalate to — working through these hands-on with the outgoing team. Deep design understanding comes gradually afterwards
- B) ソースコード全体を読んで、設計を完全に理解する / Read the entire codebase and fully understand the design
- C) 基盤を作り直す計画を立てる / Plan a rebuild of the platform
- D) 引き継ぎ資料の充実を前任チームに依頼し、完成を待つ / Ask the outgoing team for better documentation and wait for it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

2 週間という制約下では、**何を優先するかの判断が引き継ぎの成否を決めます。**4 事業部門が日常的に使っている以上、最優先は「止めないこと」であり、そのために必要なのは設計の全体理解ではなく**運用知識**です。とりわけ価値が高いのは、過去のインシデントとその対応、壊れやすい箇所、エスカレーション先といった、**文書に残りにくく前任者の頭の中にしかない情報**です。実際に手を動かしながら確認することで、文書を読むだけでは得られない知識が移転します。設計理解は引き継ぎ後に時間をかけて進められます。

Under a two-week constraint, prioritization decides whether the handover works. With four business units depending on it daily, the priority is not stopping — which needs operational knowledge, not complete design understanding. The highest-value items are precisely those that rarely reach documentation and live in the outgoing team's heads: past incidents and their resolutions, fragile areas, escalation paths. Doing it hands-on transfers what reading cannot.

- **B 不正解**: 大規模な基盤を 2 週間で完全に理解することは不可能で、運用に必要な知識が優先されません。 / Not achievable, and deprioritizes what operation actually needs.
- **C 不正解**: 理解していないシステムの作り直し計画は根拠を持てず、稼働中の業務も守れません。 / A rebuild plan without understanding is baseless and does not protect live operations.
- **D 不正解**: 資料を待つ間に前任チームは去り、対話でしか得られない知識が失われます。 / The team leaves while you wait, taking the tacit knowledge with them.

**参照 / Reference:** 引き継ぎの優先順位、運用知識の移転、暗黙知の引き出し
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

エージェントのパイロットが好評で、事業部門から追加の要望が次々と寄せられています。「この業務にも対応してほしい」「あの部門でも使いたい」「この機能も追加してほしい」といった要望が 3 週間で 24 件に達しました。開発チームは要望に応えようとして、当初の範囲（1 業務、1 部門）を大きく超えた作業を抱え、本来のリリース目標が 2 か月遅延する見込みです。

A well-received pilot has drawn a stream of additional requests: other processes, other departments, extra features — 24 requests in three weeks. Trying to accommodate them, the team is now working far beyond the original scope (one process, one department) and the original release target will slip by two months.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate response?

- A) 要望はすべて価値があるので、優先順位を付けずに順次対応する / Work through all of them in order, since each has value
- B) すべての要望を断り、当初範囲のみに集中する / Decline everything and focus on the original scope
- C) 事業部門に要望を減らすよう依頼する / Ask the business units to submit fewer requests
- D) **要望を受け止める仕組みを作り、当初範囲の完了を守る**。24 件を記録して可視化し、当初のリリース目標を守ることを優先すると明示したうえで、追加要望は次期以降の候補として整理する。優先順位は事業横断の場で決め、要望を出した部門にはその位置づけと見込み時期を伝える。当初範囲を完了させることが、後続の要望に応える最短経路であることを説明する。要望の多さは需要の証拠なので、拒否ではなく順序の問題として扱う / **Create a place for the requests and protect completion of the original scope.** Log and publish all 24, state explicitly that the original release target takes precedence, and treat the additions as candidates for subsequent phases. Prioritize them in a cross-functional forum and tell each requesting unit where their item sits and when it is likely to come. Explain that finishing the original scope is the fastest route to serving the rest. The volume of requests is evidence of demand, so treat it as a sequencing question rather than a refusal

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

好評による要望の集中は**歓迎すべき状況であり、断るべきものではありません。**問題は要望そのものではなく、それを受け止める仕組みがなく、当初範囲が守られていないことです。要望を記録して可視化すれば、要望を出した側は「無視された」と感じずに済み、優先順位を事業横断の場で決めれば決定に正当性が生まれます。**当初範囲を完了させることが後続に応える最短経路である**という説明は、断りではなく順序の話として受け入れられやすく、実際にも正しい判断です。

A flood of requests after a well-received pilot is a good problem and should not be refused. The issue is the absence of a place to put them and the erosion of the original scope. Logging and publishing them means requesters do not feel ignored, and deciding priority in a cross-functional forum gives the ordering legitimacy. Framing "finish the original scope first" as the fastest route to everything else is both easier to accept and actually true.

- **A 不正解**: 優先順位なしの逐次対応は、当初範囲も追加要望もどちらも完了しない状態を招きます。 / Delivers neither the original scope nor the additions.
- **B 不正解**: 一律に断ると、需要という貴重な情報を捨て、事業部門との関係も損ないます。 / Discards the demand signal and damages the relationship.
- **C 不正解**: 要望を減らすよう依頼するのは、需要の存在という良い状況を抑制する対応です。 / Suppresses the very signal that indicates value.

**参照 / Reference:** スコープ管理、要望の可視化と待ち行列、優先順位の場、当初範囲の完了
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

顧客に提供している AI 機能を、6 か月後に終了することが決まりました。この機能は 1,200 社が利用しており、うち 180 社は自社の業務プロセスに組み込んで日常的に使っています。契約上は 3 か月前の通知で終了できると定められています。営業部門は「解約を誘発するので、通知は 3 か月前ぎりぎりにしたい」と主張しています。

An AI feature provided to customers will be discontinued in six months. It is used by 1,200 companies, 180 of which have embedded it in their daily operations. The contract permits termination with three months' notice. Sales wants to give notice at the last possible moment to avoid triggering cancellations.

**設問 / Question:**

最も適切な対応はどれですか？

What is the most appropriate approach?

- A) 契約上の最低限である 3 か月前に通知する / Give notice at the contractual minimum of three months
- B) **契約上の最低要件より早く通知し、移行を支援する**。特に業務に組み込んでいる 180 社は移行に時間を要するため、6 か月前の時点で終了予定と理由、代替手段、移行支援の内容を伝える。代替（後継機能、他社サービス、データのエクスポート）を具体的に示し、業務組み込み顧客には個別に移行計画を相談する。通知を遅らせるほど顧客の準備時間が減り、不満と解約リスクはむしろ高まる。誠実な通知は関係維持の手段でもある / **Give notice earlier than the contract requires and support the migration.** The 180 customers with operational dependencies need time, so tell them now, at six months, with the reason, the alternatives, and the migration support available. Name concrete alternatives (a successor feature, third-party options, data export) and work through migration plans individually with the embedded customers. Late notice reduces their preparation time and raises dissatisfaction and cancellation risk rather than lowering it; honest notice is itself a way to preserve the relationship
- C) 通知せず、機能を段階的に劣化させて利用を減らす / Give no notice and degrade the feature until usage falls
- D) 終了を延期し、無期限に提供を続ける / Postpone indefinitely and keep providing it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**通知を遅らせることは、解約リスクを下げるどころか高めます。**業務に組み込んでいる 180 社にとって、3 か月は移行に十分でない可能性が高く、準備が間に合わなかった顧客は「不誠実な扱いを受けた」と感じます。早期通知と移行支援は、顧客が自社の計画を立てられるようにするもので、これが関係維持につながります。要点は通知だけでなく、**代替手段の提示と個別の移行支援**で、これがあると終了が「見捨てられた」ではなく「支援された移行」として受け取られます。

Delaying notice raises cancellation risk rather than lowering it: for 180 customers with operational dependencies, three months may not be enough, and those who fail to migrate in time experience it as being treated badly. Early notice with migration support lets them plan, which is what preserves the relationship — and the substance is not the notice alone but concrete alternatives and individual migration help, which turns an ending into a supported transition rather than abandonment.

- **A 不正解**: 契約上の最低限は法的な下限であって、業務に組み込んだ顧客への適切な対応の基準ではありません。 / A contractual floor is not a standard of appropriate treatment.
- **C 不正解**: 意図的な劣化は不誠実であり、判明したときに信頼を決定的に損ないます。 / Deliberate degradation is dishonest and decisively destroys trust when discovered.
- **D 不正解**: 終了の判断には理由があり、無期限延期は判断を先送りするだけです。 / Indefinite postponement merely defers a decision made for reasons.

**参照 / Reference:** 機能終了の顧客コミュニケーション、早期通知、移行支援、代替手段の提示
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **前 / Previous:** [`domain5_governance_risk.md`](./domain5_governance_risk.md) | **次 / Next:** [`domain7_developer_enablement.md`](./domain7_developer_enablement.md)
