# Domain 5: ガバナンス・安全性・リスク管理 / Governance, Safety and Risk Management

> 配点比率 / Weight: **14%**（Professional で新たに加わる領域 / new at the Professional tier）
> 問題数 / Questions: **30**
> 形式 / Format: 4択・単一選択 / Multiple choice (single answer)

## 出題範囲 / Scope

- AI ガバナンス体制・ユースケースのリスク分類・承認プロセス / AI governance bodies, use-case risk tiering, approval processes
- 規制適合（GDPR / EU AI Act / HIPAA / PCI DSS / SOX / DORA など）と説明義務 / Regulatory compliance and duties to explain
- データガバナンス（保持・目的制限・越境・サブプロセッサ開示） / Data governance: retention, purpose limitation, transfers, subprocessor disclosure
- 安全性（プロンプトインジェクション、有害出力、レッドチーミング、緊急停止） / Safety: prompt injection, harmful output, red teaming, kill switches
- アクセス統制・職務分離・内部不正・シャドー AI / Access control, segregation of duties, insider misuse, shadow AI
- 監査証跡・インシデント対応・当局対応・残存リスクの受容 / Audit trails, incident response, regulatory examinations, residual-risk acceptance

---

## 問題 1 / Question 1

**シナリオ / Scenario:**

社内で Claude を使ったアプリケーションが 1 年で 3 件から 27 件に増えました。中には、社内向けの議事録要約のように影響が小さいものから、与信判断の補助や採用選考の一次スクリーニングのように個人の権利に直接影響するものまであります。現在は全アプリケーションが同一の軽量な承認プロセス（所属部門長の承認のみ）を通っています。法務部門から「リスクに見合った統制になっていない」という指摘を受けました。

Claude-based applications grew from 3 to 27 in a year, ranging from low-impact internal meeting summaries to systems that directly affect individuals' rights, such as credit-decision support and first-pass recruiting screens. All of them pass through the same lightweight approval — sign-off by the owning department head. Legal has flagged that controls are not proportionate to risk.

**設問 / Question:**

最も適切なガバナンス設計はどれですか？

Which governance design is most appropriate?

- A) 全 27 件に対して、最も厳しい承認プロセス（法務・リスク・セキュリティの三部門承認）を一律に適用する / Apply the strictest approval — legal, risk, and security sign-off — uniformly to all 27
- B) **ユースケースをリスクで分類し、分類に応じた統制を適用する**。個人の権利・安全・財務に影響する用途は高リスクとして、事前の影響評価・人間による最終判断・独立したレビュー・定期的な再評価を必須にする。影響が限定的な社内用途は軽量な届出で足りる。分類基準を文書化し、新規申請は必ず分類を経てから該当する統制に進む / **Tier use cases by risk and apply proportionate controls**: treat uses affecting individuals' rights, safety, or finances as high risk, requiring a prior impact assessment, a human final decision, independent review, and periodic reassessment; low-impact internal uses need only lightweight registration. Document the classification criteria and route every new request through classification before the applicable controls
- C) 現行の部門長承認を維持し、問題が起きたケースのみ事後に強化する / Keep department-head approval and strengthen only where problems arise
- D) 高リスクな用途への Claude 利用を全面的に禁止する / Prohibit Claude entirely for high-risk uses

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

ガバナンスの基本は**リスクに比例した統制**です。すべてに最も厳しい統制を課すと、承認が滞留し、軽微な用途まで止まり、結果として統制を迂回する動き（シャドー AI）を生みます。逆に一律に軽い統制では、与信や採用のような高リスク用途に必要な保護が働きません。分類基準を文書化して**分類自体を必須のステップにする**ことが要で、これがないと分類が申請者の自己判断になり形骸化します。

Governance rests on proportionality. Applying the strictest control everywhere creates a backlog, blocks trivial uses, and drives people around the process into shadow AI. Applying the lightest everywhere leaves credit and hiring decisions unprotected. Making classification itself a mandatory documented step is what keeps the tiering real — otherwise applicants self-classify and the scheme hollows out.

- **A 不正解**: 過剰な統制は承認の滞留とシャドー AI を招き、実効的な統制をむしろ弱めます。 / Excessive control creates backlogs and shadow AI, weakening real control.
- **C 不正解**: 事後強化は、既に個人の権利に影響が生じた後の対応です。高リスク用途では事前統制が要求されます。 / Reactive control comes after rights have already been affected.
- **D 不正解**: 全面禁止は、適切な統制のもとで実現できる価値を放棄します。禁止すれば統制外での利用が増える傾向もあります。 / A ban forfeits value achievable under proper controls and tends to push usage underground.

**参照 / Reference:** リスクベースのガバナンス、ユースケース分類、比例原則
</details>

---

## 問題 2 / Question 2

**シナリオ / Scenario:**

EU で、Claude を用いた採用候補者の一次スクリーニングシステムを提供します。システムは応募書類から候補者を評価し、面接に進める候補者を推奨します。最終判断は人事担当者が行います。EU AI Act の観点で、この用途の位置づけと必要な対応を整理する必要があります。

You are deploying a Claude-based first-pass recruiting screen in the EU. It evaluates applications and recommends which candidates advance to interview; an HR staff member makes the final decision. You need to establish the system's position under the EU AI Act and the resulting obligations.

**設問 / Question:**

最も適切な理解はどれですか？

Which understanding is most appropriate?

- A) 最終判断を人間が行うため、規制上の特段の義務は生じない / Because a human makes the final decision, no particular obligations arise
- B) 汎用 AI を利用しているだけなので、義務はモデル提供者側にある / Since a general-purpose model is used, the obligations sit with the model provider
- C) 社内利用ではないので、義務は顧客企業側にある / As this is not internal use, the obligations sit with the customer
- D) **雇用・労働者管理に関わる用途は高リスクに分類され、提供者・利用者それぞれに義務が生じる**と理解する。リスク管理体制、データガバナンス、技術文書、ログ記録、透明性（候補者への通知）、**人間による実効的な監督**、正確性・堅牢性の要件が課される。人間が最終判断するという事実だけでは義務は免除されず、その監督が形式的でないこと（判断材料の提示、覆せる権限、監督者の訓練）を示せる必要がある / **Recognize that employment and worker-management uses are classified high-risk, creating obligations for both provider and deployer**: risk management, data governance, technical documentation, logging, transparency toward candidates, **meaningful human oversight**, and accuracy and robustness requirements. A human making the final call does not by itself remove the obligations — you must be able to show the oversight is not nominal (evidence presented, authority to override, trained overseers)

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

EU AI Act では、**雇用や労働者管理に関わる AI システムは高リスクに分類**され、リスク管理・データガバナンス・技術文書・ログ・透明性・人間による監督・正確性といった義務が課されます。最も誤解されやすいのが「人間が最終判断すれば規制対象外」という点で、実際には**監督が実効的であることを示す必要があります**。判断材料が提示されず、実質的に推奨をそのまま承認しているだけの運用は、形式的な監督とみなされ得ます。義務は提供者と利用者の双方に分配されます。

Under the EU AI Act, employment and worker-management systems are high-risk, carrying risk-management, data-governance, documentation, logging, transparency, human-oversight, and accuracy obligations. The common misconception is that a human in the loop exempts the system: what is required is *meaningful* oversight, demonstrably so. Rubber-stamping recommendations without evidence or authority to override does not qualify, and obligations are allocated across provider and deployer.

- **A 不正解**: 人間の関与は義務を免除しません。むしろ「実効的な監督」自体が義務の 1 つです。 / Human involvement does not exempt; meaningful oversight is itself an obligation.
- **B 不正解**: 汎用モデルの提供者にも義務はありますが、それを高リスク用途に組み込んだ側の義務は消えません。 / Provider obligations exist, but do not absorb those of the party deploying it in a high-risk use.
- **C 不正解**: 義務は一方に集約されず、提供者と利用者に分配されます。 / Obligations are shared, not transferred wholesale.

**参照 / Reference:** EU AI Act、高リスク分類、実効的な人間による監督、提供者と利用者の義務
</details>

---

## 問題 3 / Question 3

**シナリオ / Scenario:**

顧客から「当社のデータが AI モデルの学習に使われないことを保証してほしい」という要求を受けました。顧客は金融機関で、送信されるデータには顧客企業の取引先情報が含まれます。契約交渉の担当者から「モデル提供者の利用規約に学習利用しない旨の記載があるので、それを示せばよいか」と相談を受けました。

A customer — a financial institution whose submitted data includes information about their own clients — demands assurance that their data will not be used to train AI models. The contract negotiator asks whether pointing to the model provider's terms, which state that data is not used for training, is sufficient.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **契約上の裏付けと技術的・運用的な統制の両方で示す**。モデル提供者との契約条件（学習利用の有無、データ保持期間、サブプロセッサ）を確認して顧客に提示できる形にすると同時に、自社側で顧客データがどこに保存され、誰がアクセスでき、いつ削除されるかを文書化する。自社のログや評価データセットへの流用も統制対象に含め、実際の運用が文書どおりであることを内部監査で確認できるようにする / **Demonstrate it through both contractual and technical/operational controls**: verify and be able to present the provider's terms (training use, retention period, subprocessors), and separately document where the customer's data is stored on your side, who can access it, and when it is deleted — including controls over reuse in your own logs and evaluation datasets, with internal audit able to confirm that practice matches the documentation
- B) モデル提供者の利用規約を顧客に転送して回答とする / Forward the provider's terms to the customer as the answer
- C) 学習利用されないことは保証できないと回答し、要求を断る / Reply that this cannot be guaranteed and decline the requirement
- D) 顧客データを送信する前にすべて匿名化する / Anonymize all customer data before transmission

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

顧客の関心は「モデル提供者が学習に使わないこと」だけではなく、**自社に渡ったデータ全体がどう扱われるか**にあります。実務上、見落とされやすいのは**自社側の流用**で、本番ログや評価データセットへの取り込みは学習利用ではないものの、顧客データの二次利用として同様の懸念に触れます。したがって、提供者との契約条件と、自社の保存・アクセス・削除・二次利用の統制を合わせて示すのが正しい回答です。文書と実運用の一致を内部監査で確認できることが、保証の実質になります。

The customer's concern is not only the provider's training practices but how their data is handled end to end. The commonly missed piece is *your own* secondary use: pulling customer data into production logs and evaluation datasets is not training, yet raises the same concern. So the answer combines the provider's terms with your own storage, access, deletion, and reuse controls — and internal audit confirming practice matches documentation is what makes it an assurance rather than a claim.

- **B 不正解**: 提供者の規約だけでは、自社に渡ったデータの取り扱い（ログ、評価データ流用）を説明できません。 / Provider terms say nothing about how you handle the data on your side.
- **C 不正解**: 統制によって実際に保証可能な内容であり、断る根拠がありません。 / The assurance is achievable with controls; declining is unwarranted.
- **D 不正解**: 匿名化は有用な場合もありますが、取引先情報を含むデータでは分析価値を大きく損ない、要求（学習利用の禁止）への直接の答えでもありません。 / Often destroys the analytical value, and does not answer the requirement asked.

**参照 / Reference:** データガバナンス、学習利用の統制、二次利用の管理、内部監査
</details>

---

## 問題 4 / Question 4

**シナリオ / Scenario:**

保険引受の判断補助に Claude を使っています。3 年後、ある契約者から「当時の引受条件が不当だった」という申立てがあり、監督官庁からも照会が来ました。当時の判断根拠を示す必要がありますが、現在保存されているのは「Claude が生成した推奨文（テキスト）」と「最終的な引受条件」のみです。当時使用していたプロンプト、モデルバージョン、参照した社内基準の版は記録されていません。

Claude assists underwriting decisions. Three years later, a policyholder disputes the terms they were given and the regulator makes an inquiry. You must show the basis of the decision, but only two things were retained: the recommendation text Claude generated, and the final terms. The prompt in use, the model version, and the version of the internal criteria consulted were never recorded.

**設問 / Question:**

最も適切な改善はどれですか？

Which improvement is most appropriate?

- A) 推奨文のテキストがあれば判断根拠として十分と主張する / Argue that the recommendation text alone constitutes the basis
- B) 今後は推奨文をより詳細に生成させる / Have future recommendations generated in more detail
- C) **判断を再現・説明できる一式を監査証跡として保存する**。入力データのスナップショット、使用したプロンプトのバージョン、モデル識別子とバージョン、参照した社内基準・料率表の版、生成された推奨とその根拠、人間の最終判断と（推奨と異なる場合は）その理由を、判断単位で紐付けて保存する。保持期間は規制上の時効に合わせ、改竄されないことを担保する / **Retain, as an audit trail, everything needed to reconstruct and explain the decision**: a snapshot of the input data, the prompt version, the model identifier and version, the versions of the internal criteria and rate tables consulted, the recommendation and its stated basis, and the human's final decision with reasons where it differed — all linked per decision, retained for the regulatory limitation period, and protected against alteration
- D) 判断根拠の保存が難しいため、Claude の利用をやめる / Stop using Claude, since retaining the basis is difficult

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

規制対象の判断では、**数年後に「当時どのような根拠で判断したか」を示せること**が要求されます。推奨文だけでは、その推奨がどの入力・どの基準・どのモデルから生じたかが分からず、説明として不十分です。監査証跡は判断単位で一式が紐付いている必要があり、保持期間は規制上の時効に合わせます。改竄防止も重要で、事後に書き換え可能な記録は証跡としての価値が下がります。人間が推奨と異なる判断をした場合の理由も、監督上重要な記録です。

Regulated decisions must be explainable years later. Recommendation text alone does not show which inputs, criteria, or model produced it. An audit trail links the full set per decision, is retained for the applicable limitation period, and is protected against alteration — a record that can be rewritten afterwards is worth much less. Where the human departed from the recommendation, that reasoning is itself a supervisory record.

- **A 不正解**: 推奨文だけでは、入力・基準・モデルの対応が示せず、当時の判断を再現できません。 / The text alone cannot reconstruct the decision.
- **B 不正解**: 詳細化は将来の推奨には効きますが、記録すべき要素（入力、バージョン）が欠けている構造は変わりません。 / More detail does not add the missing input and version records.
- **D 不正解**: 保存は実装可能な要件であり、利用を諦める理由になりません。 / Retention is implementable; abandoning the use case is unwarranted.

**参照 / Reference:** 監査証跡、判断の再現性、保持期間、改竄防止
</details>

---

## 問題 5 / Question 5

**シナリオ / Scenario:**

社内の 27 のエージェントアプリケーションのうち、19 が外部から取得したコンテンツ（Web ページ、受信メール、顧客が添付した文書、パートナーから届く発注書）をコンテキストに含めます。セキュリティ部門から「間接プロンプトインジェクションへの組織的な対策方針が必要」との要求がありました。各アプリの実装はチームごとに異なり、対策の有無もまちまちです。

Of 27 internal agent applications, 19 place externally sourced content in context — web pages, inbound email, customer-attached documents, partner purchase orders. Security requires an organization-wide policy on indirect prompt injection. Implementations differ per team and defenses are inconsistent.

**設問 / Question:**

最も適切な組織的対策はどれですか？

Which organizational measure is most appropriate?

- A) 各チームに「プロンプトインジェクションに注意すること」という通達を出す / Issue a notice asking teams to be careful about prompt injection
- B) 外部コンテンツを扱うアプリケーションの利用を禁止する / Prohibit applications that handle external content
- C) セキュリティ部門が全 27 アプリのプロンプトを個別にレビューする / Have security review all 27 applications' prompts individually
- D) **組織横断の統制標準を定め、実装ではなく成果を規定する**。(1) 外部由来コンテンツはデータとして扱い指示として解釈させない構造（区切り、役割の明示）、(2) 外部コンテンツを処理する経路のツール権限を最小化する、(3) 不可逆・機微な操作は決定的なフックまたは人間の承認で守る、(4) 影響の大きいアプリにはレッドチーミングを義務付ける、を標準として定義し、リスク分類に応じて適用範囲を決める。準拠状況を定期的に確認する / **Define a cross-cutting control standard specifying outcomes rather than implementations**: (1) externally sourced content is treated as data and not interpreted as instructions (delimiting, explicit roles); (2) tool permissions on paths that process external content are minimized; (3) irreversible or sensitive operations are protected by a deterministic hook or human approval; and (4) red teaming is mandatory for high-impact applications. Scope the standard by risk tier and verify compliance periodically

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

組織的対策の要点は、**個々の実装を統一するのではなく、満たすべき成果を標準として定めること**です。19 のアプリケーションは目的も構造も異なるため、実装を画一化するのは非現実的ですが、「外部コンテンツを指示として解釈させない」「その経路の権限を最小化する」「不可逆操作を決定的に守る」という成果は共通に要求できます。最も重要なのは (3) で、**注入が成功しても実害が出ない構造**を要求している点です。リスク分類に応じた適用と準拠確認が、標準を実効化します。

Organizational defense means standardizing outcomes, not implementations: 19 applications with different purposes cannot share one implementation, but they can share requirements. The most important is (3) — requiring that a successful injection cause no damage, rather than requiring that injection never succeed. Risk-tiered scoping and periodic compliance checks are what make a standard operative.

- **A 不正解**: 注意喚起は統制ではなく、準拠状況も検証できません。 / An advisory is not a control and cannot be verified.
- **B 不正解**: 外部コンテンツの処理は多くの業務の本質であり、禁止は事業要件の否定です。 / Processing external content is the point of most of these applications.
- **C 不正解**: プロンプトの個別レビューはスケールせず、変更のたびに繰り返す必要があり、権限設計という本質的な対策も見ていません。 / Unscalable, must repeat on every change, and misses the permission dimension.

**参照 / Reference:** 間接プロンプトインジェクション、成果ベースの統制標準、多層防御、リスク分類
</details>

---

## 問題 6 / Question 6

**シナリオ / Scenario:**

顧客向けの対話型エージェントをリリースする前に、安全性を確認したいと考えています。開発チームは「社内で 200 件のテストケースを試して問題がなかった」と報告しています。テストケースは開発者が想定した通常の利用シナリオに基づいています。このエージェントは決済情報の照会と返金処理のツールを持ち、一般消費者が利用します。

Before releasing a customer-facing conversational agent, you want safety assurance. The team reports "200 internal test cases with no issues," based on the usage scenarios the developers anticipated. The agent has tools for payment inquiry and refund processing and will be used by general consumers.

**設問 / Question:**

最も適切な追加の確認はどれですか？

Which additional assurance activity is most appropriate?

- A) テストケースを 200 件から 1,000 件に増やす / Expand from 200 to 1,000 test cases
- B) **敵対的な観点からのレッドチーミングを実施する**。想定利用シナリオではなく、「返金を不正に引き出せるか」「他人の決済情報を引き出せるか」「役割を偽って権限を超える操作をさせられるか」「有害な出力を引き出せるか」といった攻撃者の目的から試行する。実施者は開発チームと独立させ、発見された事項は修正して再試行する。結果と対処を記録し、リリース判定の材料とする / **Run adversarial red teaming**: instead of anticipated scenarios, probe from an attacker's goals — can a refund be extracted illegitimately, can another person's payment data be surfaced, can a false role induce operations beyond permission, can harmful output be elicited. Staff it independently of the development team, fix and retest findings, and record results and remediation as input to the release decision
- C) 利用規約に免責条項を追加する / Add disclaimers to the terms of service
- D) リリース後に問題が報告されたら対応する / Respond to problems if they are reported after release

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**開発者が想定したシナリオのテストは、開発者が想定しなかった攻撃を見つけられません。**返金処理という金銭的な副作用を持ち、一般消費者が利用するシステムでは、悪意ある利用が現実的な脅威です。レッドチーミングは「攻撃者の目的」から出発する点で通常のテストと本質的に異なり、実施者を開発チームから独立させることで想定の偏りを避けます。発見事項を修正して再試行し、記録を残すことで、リリース判定と事後の説明の両方に使えます。

Tests built from anticipated scenarios cannot find unanticipated attacks. A system with monetary side effects, exposed to the general public, faces adversarial use as a realistic threat. Red teaming differs fundamentally by starting from attacker goals, and staffing it independently avoids inheriting the builders' blind spots. Fix, retest, and record — the record serves both the release decision and later explanation.

- **A 不正解**: 同じ想定に基づくケースを 5 倍にしても、想定外の攻撃経路は見つかりません。 / Five times the same assumptions finds nothing new.
- **C 不正解**: 免責条項は法的な位置づけの話で、実際の不正利用や情報漏洩を防ぎません。 / Disclaimers do not prevent misuse or disclosure.
- **D 不正解**: 決済に関わる機能で事後対応に頼るのは、被害が発生してからの対処になります。 / For payment functionality, reactive handling means damage first.

**参照 / Reference:** レッドチーミング、敵対的評価、独立した実施、リリース判定への反映
</details>

---

## 問題 7 / Question 7

**シナリオ / Scenario:**

本番エージェントのシステムプロンプトは、リポジトリ上のテキストファイルで管理されており、開発者 24 名の誰でも編集してマージできます。プロンプトの変更はコードレビューを経ますが、レビュアーは同じチームの開発者です。このエージェントは融資審査の補助に使われており、プロンプトの内容が審査の判断傾向を左右します。内部監査から「本番の判断ロジックを変更できる権限が広すぎる」という指摘がありました。

The production system prompt lives in a repository text file that any of 24 developers can edit and merge. Prompt changes go through code review, but reviewers are developers on the same team. The agent assists loan underwriting, and prompt content shapes decision tendencies. Internal audit has flagged that the ability to change production decision logic is too broadly held.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 開発者全員にプロンプト変更の研修を実施する / Train all developers on prompt changes
- B) プロンプトを暗号化してリポジトリに保存する / Store the prompt encrypted in the repository
- C) **プロンプトを本番の判断ロジックとして扱い、変更統制を強化する**。融資判断に影響する部分の変更には、リスク管理または与信ポリシー所管部門の承認を必須とし、承認者を実装者と分離する（職務分離）。変更は評価データセットでの事前測定を伴い、承認記録・変更内容・適用日時を監査可能な形で残す。判断に影響しない文言修正とは変更区分を分けて、統制の重さを変える / **Treat the prompt as production decision logic and strengthen change control**: require approval from risk management or the credit-policy owner for changes affecting underwriting, with the approver separate from the implementer (segregation of duties). Require pre-change measurement on the evaluation set, and retain the approval record, the change content, and the effective time in auditable form — while classifying non-decision-affecting edits separately so control weight matches impact
- D) プロンプトの変更を月 1 回のリリース日にまとめる / Batch prompt changes into a monthly release day

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**プロンプトは実行される判断ロジックであり、コードと同等以上の変更統制が必要**です。融資判断に影響する変更を、与信ポリシーの所管外である開発者だけで完結できる構成は、統制上の欠陥です。要点は 2 つで、**職務分離**（実装者と承認者を分ける）と**所管部門の承認**（与信方針の変更は与信部門の権限）です。加えて、すべての変更に同じ重さの統制を課すと運用が滞るため、判断に影響しない修正とは区分を分けるのが実務的です。

A prompt is executed decision logic and needs at least the change control that code gets. Letting developers who do not own credit policy complete a change that shifts underwriting is a control deficiency. Two things matter: segregation of duties between implementer and approver, and approval by the function that owns the policy. Classifying non-decision-affecting edits separately keeps the control workable.

- **A 不正解**: 研修は能力の問題に対処するもので、権限の広さという統制上の問題を解決しません。 / Training addresses competence, not the breadth of authority.
- **B 不正解**: 暗号化は秘匿の手段であり、変更権限の統制とは無関係です。 / Encryption is confidentiality, not change control.
- **D 不正解**: 頻度をまとめても、誰が承認するかという職務分離の問題は残ります。 / Batching does not address who approves.

**参照 / Reference:** プロンプトの変更統制、職務分離、所管部門による承認、監査記録
</details>

---

## 問題 8 / Question 8

**シナリオ / Scenario:**

顧客向けエージェントが、ある顧客に対して事実と異なる補償内容を案内し、顧客がそれを根拠に行動して損害が生じました。社内では、誰が最初に対応すべきか、どこまでを止めるべきか、いつ誰に報告すべきかが定まっておらず、発覚から初動まで 9 時間かかりました。同種のエージェントは他に 6 つ稼働しています。

A customer-facing agent told a customer incorrect compensation terms; the customer acted on it and suffered a loss. Internally, no one knew who responds first, how much to shut down, or when and whom to notify — nine hours passed before any action. Six similar agents are also in production.

**設問 / Question:**

最も適切な再発防止策はどれですか？

Which prevention measure is most appropriate?

- A) **AI 固有のインシデント対応手順を整備する**。検知経路（顧客申告、監視アラート、社内報告）、初動の責任者、影響範囲の特定手順（同一プロンプト・同一ツールを使う他エージェントの点検を含む）、機能停止・縮退の判断基準と権限、顧客対応と社内外への報告基準、事後の原因分析と評価スイートへの反映までを定め、訓練で実行可能性を確認する / **Establish an AI-specific incident response procedure**: detection channels (customer report, monitoring alert, internal report), the first responder, a scope-assessment procedure including checks of other agents sharing the prompt or tools, the criteria and authority for disabling or degrading a feature, thresholds for customer and internal/external notification, and post-incident analysis feeding the evaluation suite — then exercise it to confirm it works
- B) エージェントの応答に免責文言を追加する / Add a disclaimer to agent responses
- C) 該当エージェントのみ停止し、他は様子を見る / Disable only the affected agent and watch the others
- D) 顧客対応部門に個別に対処してもらう / Delegate handling to the customer-service department

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

初動に 9 時間かかった原因は、**手順・責任・権限が事前に定まっていないこと**です。AI 固有の要素として重要なのが、**影響範囲の特定**（同じプロンプトやツールを共有する他のエージェントに同じ欠陥がある可能性）と、**機能停止の判断権限**（誰が止めてよいかが決まっていないと止まらない）です。訓練で実行可能性を確認する点も要点で、文書があるだけでは初動は速くなりません。事後分析を評価スイートに反映する経路が、同じ誤りの再発を防ぎます。

Nine hours to first action means the procedure, ownership, and authority did not exist in advance. The AI-specific parts are scope assessment — other agents sharing the prompt or tools may carry the same defect — and explicit authority to disable, without which nothing gets disabled. Exercising the procedure matters too: a document alone does not shorten response time, and routing the analysis into the evaluation suite is what prevents recurrence.

- **B 不正解**: 免責文言は誤った案内そのものを防がず、既に生じた損害にも対応しません。 / Disclaimers neither prevent the misinformation nor address the loss.
- **C 不正解**: 他の 6 つに同じ欠陥がある可能性を確認しないのは、影響範囲の特定を怠る対応です。 / Skips scope assessment across six systems that may share the defect.
- **D 不正解**: 顧客対応は必要ですが、技術的な原因究明と他システムの点検を含む全体の手順が定まりません。 / Necessary but insufficient; it leaves technical scope and remediation undefined.

**参照 / Reference:** AI インシデント対応、影響範囲の特定、停止権限、訓練による検証
</details>

---

## 問題 9 / Question 9

**シナリオ / Scenario:**

事業部門から「業務効率化のため、外部ベンダーが提供する AI エージェント SaaS を導入したい」という申請がありました。この SaaS は社内の顧客データベースに接続し、顧客対応履歴を分析して提案を生成します。ベンダーは設立 2 年のスタートアップで、データの保存場所と再委託先（サブプロセッサ）については「クラウド上で安全に管理」としか説明していません。事業部門は「競合も使っているので問題ない」と主張しています。

A business unit requests adoption of a third-party AI agent SaaS. It connects to the internal customer database, analyzes support history, and generates recommendations. The vendor is a two-year-old startup and describes data location and subprocessors only as "securely managed in the cloud." The business unit argues that "competitors use it, so it must be fine."

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 競合も使っているので、リスクは業界標準の範囲内と判断して承認する / Approve, treating the risk as industry-standard since competitors use it
- B) 事業部門の判断に委ね、IT 部門は関与しない / Leave it to the business unit; IT should not be involved
- C) スタートアップとの取引は一律禁止とする / Prohibit engagements with startups as a class
- D) **サードパーティリスク評価を実施したうえで判断する**。データの保存場所・再委託先の一覧・保持期間・学習利用の有無・暗号化・アクセス統制・インシデント通知義務・事業継続性（ベンダー破綻時のデータ返還と移行）を契約と技術の両面で確認する。顧客データを扱う以上、GDPR 等の下では再委託先の開示は権利ではなく要件であり、「安全に管理」という説明は評価の代わりにならない。評価結果に基づき、条件付き承認・代替案・不承認を判断する / **Decide after a third-party risk assessment**: confirm data location, the subprocessor list, retention, training use, encryption, access controls, incident-notification duties, and business continuity (data return and migration on vendor failure) — contractually and technically. Where customer data is involved, subprocessor disclosure is a requirement rather than a courtesy under GDPR and similar regimes, and "securely managed" is not an assessment. Then approve with conditions, propose an alternative, or decline

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

顧客データを外部に渡す判断は、**サードパーティリスク評価を経て行う**べきものです。特に個人データを扱う場合、サブプロセッサの開示は規制上の要件であり、「安全に管理」という説明では委託先としての適格性を判断できません。設立 2 年という点は事業継続性のリスク要因として評価すべき事実であり、破綻時のデータ返還と移行可能性を契約で確保する必要があります。「競合も使っている」は、自社のリスク評価の代わりにはなりません。

Handing customer data to a third party is a decision that follows an assessment. Where personal data is involved, subprocessor disclosure is a regulatory requirement, and "securely managed" does not establish suitability. Two years of operating history is a genuine continuity factor to assess, with data return and migration secured contractually. "Competitors use it" substitutes someone else's risk appetite for your own assessment.

- **A 不正解**: 他社の判断は自社のリスク評価の代替になりません。他社が異なる条件で契約している可能性もあります。 / Another company's decision is not your assessment, and their terms may differ.
- **B 不正解**: 顧客データの外部提供は全社的なリスクであり、事業部門単独で判断してよい範囲を超えています。 / Exporting customer data is an enterprise risk beyond a single unit's authority.
- **C 不正解**: 企業規模による一律禁止は評価ではなく、有用なベンダーを不必要に排除します。 / A blanket rule is not an assessment and excludes useful vendors.

**参照 / Reference:** サードパーティリスク評価、サブプロセッサ開示、事業継続性、委託先管理
</details>

---

## 問題 10 / Question 10

**シナリオ / Scenario:**

監督官庁から、稼働中の AI システムについて「システムの目的、想定される利用条件、性能特性、既知の限界、評価方法とその結果」を記載した文書の提出を求められました。社内には設計時の技術仕様書とソースコードはありますが、これらの観点で整理された文書は存在しません。システムは 2 年前から稼働しており、その間にモデルとプロンプトは複数回更新されています。

A regulator requests documentation of a live AI system covering its purpose, intended conditions of use, performance characteristics, known limitations, and evaluation methodology and results. Internally, design specifications and source code exist, but nothing organized along those lines. The system has run for two years, with several model and prompt updates.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) ソースコードと技術仕様書をそのまま提出する / Submit the source code and design specifications as they are
- B) **システムの説明文書（モデルカード相当）を整備し、以後は更新のたびに維持する**。目的と適用範囲、想定利用条件と想定外の利用、対象データの性質、性能特性（全体および重要セグメント別）、既知の限界と失敗モード、評価方法・データセット・結果、人間による監督の仕組み、更新履歴（モデル・プロンプトのバージョンと変更理由）を記載する。文書は一度きりの提出物ではなく、変更管理と連動して維持する成果物として位置づける / **Produce a system documentation package (a model-card equivalent) and maintain it with every update**: purpose and scope, intended and out-of-scope conditions of use, the nature of the data, performance characteristics overall and by significant segment, known limitations and failure modes, evaluation methodology, datasets and results, the human-oversight mechanism, and a change history of model and prompt versions with reasons. Treat it as an artifact maintained alongside change management, not a one-off submission
- C) 提出を拒否し、企業秘密であると回答する / Decline to submit, citing trade secrets
- D) 現在のバージョンについてのみ記述した文書を作成する / Document only the current version

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

監督官庁が求めているのは、**システムを外部から理解・評価できる形の説明文書**です。ソースコードは実装の記述であって、目的・限界・性能特性の説明にはなりません。とりわけ重要なのが**既知の限界と失敗モードの明示**で、これを書けることが自らのシステムを理解している証拠になります。2 年間の更新履歴が求められる以上、文書は一度作って終わりではなく、変更管理と連動して維持される必要があります。この位置づけを最初に決めることが、次回の照会への備えになります。

Regulators want documentation that lets an outsider understand and assess the system. Source code describes implementation, not purpose, limitations, or performance. The most telling section is known limitations and failure modes: being able to write it is evidence you understand your own system. Because a two-year change history is in scope, the document must be maintained alongside change management rather than produced once.

- **A 不正解**: ソースコードは求められた観点（目的、限界、評価）に答えておらず、説明責任を果たしません。 / Code does not answer the questions asked.
- **C 不正解**: 監督官庁への提出拒否は、正当な根拠がない限り重大な問題を招きます。 / Refusing a supervisory request without proper grounds is a serious escalation.
- **D 不正解**: 2 年間の更新履歴が求められている以上、現行版のみでは不十分です。 / The request covers the period, not just the current state.

**参照 / Reference:** モデルカード／システム説明文書、既知の限界の開示、変更履歴の維持
</details>

---

## 問題 11 / Question 11

**シナリオ / Scenario:**

給付金申請の審査に Claude を使い、審査担当者が最終判断する構成です。運用データを見ると、担当者は Claude の推奨を 99.4% の割合でそのまま承認しており、1 件あたりの確認時間は平均 8 秒でした。画面には推奨結果（「承認」または「却下」）のみが表示され、判断根拠や該当する規定は表示されていません。規制上、この判断には人間による監督が必要とされています。

Claude assists benefit-claim adjudication with a human reviewer making the final decision. Operational data shows reviewers accept the recommendation 99.4% of the time, spending an average of 8 seconds per case. The screen shows only the recommendation — approve or deny — with no rationale or applicable rule. Regulation requires human oversight of these decisions.

**設問 / Question:**

最も適切な評価と対応はどれですか？

Which assessment and response is most appropriate?

- A) 承認率 99.4% は Claude の精度が高いことを示しており、問題ない / A 99.4% acceptance rate shows the model is accurate; no issue
- B) 担当者に「もっと慎重に確認するように」と指導する / Instruct reviewers to check more carefully
- C) **現状の監督は形式的であり、実効性を欠いていると評価する**。8 秒・99.4% という数字は、担当者が独立した判断を行える状態にないことを示す。判断根拠・該当規定・不確実性の高さ・過去の類似事例を画面に提示し、推奨と異なる判断を選ぶ経路を明確にする。あわせて、担当者の処理件数の目標を確認時間が確保できる水準に見直し、監督の実効性を示す指標（不一致率、確認時間の分布、覆した判断の内容）を継続的に測定する / **Assess the oversight as nominal rather than effective.** Eight seconds and 99.4% indicate reviewers are not in a position to judge independently. Present the rationale, the applicable rule, an uncertainty signal, and comparable past cases on screen, and make overriding an explicit path. Revise reviewer throughput targets so genuine review time exists, and continuously measure indicators of effective oversight — override rate, the distribution of review times, and the content of overridden decisions
- D) 人間による確認を廃止し、完全自動化する / Remove human review and fully automate

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**人間が介在していることと、実効的な監督が行われていることは別**です。8 秒で 99.4% を承認している状態は、判断ではなく追認であり、規制が求める監督を満たしているとは言えません。実効性の条件は 3 つで、(1) 判断材料が提示されていること、(2) 覆す権限と経路があること、(3) 覆すための時間があること。処理件数の目標が確認時間を圧迫している場合、担当者個人の姿勢の問題ではなく設計の問題です。実効性を測る指標を持つことが、この状態を再発させない仕組みになります。

A human being present is not the same as oversight being effective. Approving 99.4% in eight seconds is ratification, not judgment, and does not meet a regulatory oversight requirement. Effectiveness needs three things: evidence presented, authority and a path to override, and time to do it. Where throughput targets crowd out review time, this is a design problem, not an attitude problem — and measuring effectiveness is what keeps it from returning.

- **A 不正解**: 承認率の高さは監督が機能している証拠ではなく、機能していない可能性の方が高い兆候です。 / A high acceptance rate is a warning sign, not evidence of working oversight.
- **B 不正解**: 判断材料が提示されず時間もない状況で、個人の姿勢に帰する指導は効果がありません。 / Attributing a systemic condition to individual diligence does not work.
- **D 不正解**: 規制上、人間による監督が要求されているため、廃止は選択肢になりません。 / Regulation requires the oversight; removing it is not available.

**参照 / Reference:** 実効的な人間による監督、形式的監督の兆候、監督実効性の指標
</details>

---

## 問題 12 / Question 12

**シナリオ / Scenario:**

住宅ローンの事前審査に Claude を使っています。監査で、承認率を属性別に集計したところ、特定の郵便番号地域からの申込者の承認率が他地域より 14 ポイント低いことが判明しました。プロンプトには人種・性別・国籍を扱う指示は一切なく、入力データにもこれらの項目は含まれていません。郵便番号は「地域の不動産市況」を参照するために使われています。

Claude assists mortgage pre-qualification. An audit found that applicants from certain postal codes are approved 14 points less often than others. The prompt contains no instruction concerning race, sex, or nationality, and none of those fields are in the input. Postal code is used to reference local property-market conditions.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **代理変数による間接的な差別の可能性を調査し、必要な是正を行う**。郵便番号は居住地域の人口構成と相関し得るため、保護属性に対する代理変数として機能している可能性がある。承認率の差が信用リスクの差で説明できるか（同一の信用条件下でも差が残るか）を統計的に検証し、説明できない差が残る場合は、郵便番号の使用方法を見直す（不動産市況を地域より細かい／異なる指標で表現するなど）。検証と是正の過程を記録する / **Investigate possible indirect discrimination through a proxy variable and remediate as needed.** Postal code can correlate with the demographic composition of an area and thus act as a proxy for a protected attribute. Test statistically whether the approval gap is explained by credit risk — that is, whether it persists at equivalent credit profiles — and if an unexplained gap remains, revise how postal code is used (representing property-market conditions by a different or finer-grained indicator). Document the analysis and the remediation
- B) プロンプトに保護属性への言及がないため、差別は生じていないと判断する / Conclude there is no discrimination, since the prompt never mentions protected attributes
- C) 郵便番号地域ごとに承認率が均等になるよう、承認基準を調整する / Adjust approval thresholds per postal code so approval rates equalize
- D) 監査結果を公表せず、内部で留める / Keep the audit finding internal and unpublished

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**保護属性を直接使っていないことは、差別が生じていないことを意味しません。**郵便番号のような地理的変数は人口構成と相関することが多く、代理変数として間接的な差別（disparate impact）を生み得ます。検証の要点は、**差が正当な要因（信用リスク）で説明できるかを統計的に確認する**ことで、同一の信用条件下でも差が残るなら、その差は正当化されません。是正は、承認率を機械的に揃えるのではなく、代理となっている変数の使い方を見直す方向で行うのが適切です。

Not using a protected attribute does not mean no discrimination occurred. Geographic variables frequently correlate with demographic composition and can produce disparate impact through proxying. The analytical question is whether the gap is explained by a legitimate factor — whether it persists at equivalent credit profiles — and a residual gap is not justified. Remediation targets how the proxy is used, not the output rates directly.

- **B 不正解**: 直接的な言及の不在は間接差別を排除しません。実際に 14 ポイントの差が観測されています。 / Absence of direct reference does not rule out indirect discrimination; the gap is observed.
- **C 不正解**: 地域ごとに基準を変えるのは、地域を理由とした異なる取り扱いそのもので、新たな法的問題を生みます。 / Applying different thresholds by area is itself differential treatment on that basis.
- **D 不正解**: 発見された可能性のある差別的影響を秘匿するのは、規制上も倫理上も重大な問題です。 / Concealing a potential disparate impact is a serious regulatory and ethical failure.

**参照 / Reference:** 代理変数、間接差別（disparate impact）、公平性検証、是正措置の記録
</details>

---

## 問題 13 / Question 13

**シナリオ / Scenario:**

情報システム部門の調査で、社内の複数の部署が承認を経ずに外部の AI サービスを業務利用していることが判明しました。用途は議事録の要約、顧客メールの下書き、契約書の確認など多岐にわたります。利用者に理由を聞くと、「正式な申請プロセスは 2 か月かかる」「そもそも申請先が分からなかった」「業務で必要だった」という回答でした。一部では顧客情報が入力されていました。

An IT survey found several departments using external AI services for work without approval: meeting-minute summarization, drafting customer emails, contract checking, and more. Asked why, users cited a two-month approval process, not knowing where to apply, and immediate business need. In some cases customer information had been entered.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 発見された利用をすべて停止させ、違反者を懲戒処分とする / Halt all discovered usage and discipline the users
- B) 外部 AI サービスへのネットワークアクセスを全面遮断する / Block network access to all external AI services
- C) 現状を黙認し、問題が起きてから対応する / Tolerate the situation and respond if problems occur
- D) **需要が実在することを前提に、承認された選択肢と迅速な経路を用意する**。まず顧客情報の入力があった事案については、対象データと影響範囲を特定して個別に対処する。並行して、承認済みの社内利用手段（統制下の環境）を提供し、リスクの低い用途については数日で完了する軽量な承認経路を設ける。あわせて、何が承認済みで何が禁止かを明確に周知し、検知の仕組みを継続する。**利用を止めるのではなく、安全な経路に誘導する**ことを設計目標とする / **Assume the demand is real and provide approved options with a fast path.** First, handle the cases where customer information was entered — identify the data and assess exposure. In parallel, offer an approved, governed internal option and a lightweight approval route that completes in days for low-risk uses, clearly communicate what is approved and what is prohibited, and keep detection running. Design for redirecting usage onto a safe path, not for stopping it

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**シャドー AI は、統制の失敗であると同時に需要の証拠**です。2 か月の承認プロセスと不明確な申請先が、迂回の直接的な原因になっています。抑圧的な対応は利用をより見えにくい形へ追いやるだけで、統制はかえって弱まります。正しい対応は 2 段階で、既に発生した情報流出への個別対処と、**承認された選択肢と迅速な経路の提供**です。統制が使いやすければ迂回する動機がなくなり、これが最も効果的な統制になります。検知の仕組みは、周知後も継続して残存を把握するために必要です。

Shadow AI is simultaneously a control failure and evidence of demand: a two-month process with an unclear entry point is the direct cause of circumvention. Suppression pushes usage further out of sight and weakens control. The response has two parts — handle the disclosures that already happened, and provide an approved option with a fast route. When the sanctioned path is easy, the incentive to bypass disappears, and continued detection tells you whether it worked.

- **A 不正解**: 懲戒中心の対応は報告を抑制し、シャドー AI をより発見しにくくします。原因（遅い承認）も解決しません。 / Punishment suppresses disclosure and leaves the cause untouched.
- **B 不正解**: 全面遮断は業務上の正当な需要を無視し、個人端末など統制外の経路への移行を促します。 / Ignores legitimate need and pushes usage onto uncontrolled devices.
- **C 不正解**: 顧客情報の入力が既に発生しており、黙認は容認できません。 / Customer information has already been exposed; tolerance is not available.

**参照 / Reference:** シャドー AI、承認プロセスの実効性、安全な代替経路の提供、検知の継続
</details>

---

## 問題 14 / Question 14

**シナリオ / Scenario:**

EU の顧客に SaaS を提供しており、Claude を組み込んだ機能を新たに追加します。既存の顧客との DPA（データ処理契約）には、サブプロセッサの一覧と、追加時の事前通知義務が定められています。開発チームは「モデル提供者は自社のインフラの一部なので、サブプロセッサには当たらない」と考えており、通知せずにリリースしようとしています。

You provide SaaS to EU customers and are adding a Claude-powered feature. Existing DPAs list subprocessors and require advance notice before adding one. The development team believes the model provider is "part of our infrastructure" rather than a subprocessor and plans to ship without notifying customers.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) インフラの一部なので通知は不要と判断してリリースする / Ship without notice, treating it as infrastructure
- B) **顧客の個人データを処理する第三者はサブプロセッサに当たると理解し、DPA の手続きに従う**。モデル提供者を含む処理の連鎖を整理し、サブプロセッサ一覧を更新して、契約に定められた事前通知期間を守って顧客に通知する。通知には処理の目的、対象データの種類、所在地、保持期間を含める。顧客が異議を申し立てる権利がある場合はその手続きも尊重し、リリース計画に通知期間を織り込む / **Recognize that a third party processing customers' personal data is a subprocessor, and follow the DPA process**: map the processing chain including the model provider, update the subprocessor list, and notify customers within the contractual advance-notice period, stating the purpose, categories of data, location, and retention. Where customers have a right to object, honor that procedure — and build the notice period into the release plan
- C) 顧客から質問があった場合にのみ回答する / Answer only if a customer asks
- D) DPA を改定して、サブプロセッサの通知義務を削除する / Amend the DPA to remove the subprocessor notice obligation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**顧客の個人データを処理する第三者は、自社インフラの一部という認識とは無関係にサブプロセッサに該当します。**GDPR の下では、処理者は再委託先の追加について管理者（顧客）に通知し、契約上の手続きに従う義務があります。「インフラの一部」という社内の位置づけは、法的な性質を変えません。実務上の要点は、**通知期間をリリース計画に織り込む**ことで、これを後から気づくとリリースが遅延するか、契約違反を犯すかの二択になります。

A third party processing your customers' personal data is a subprocessor regardless of how you categorize it internally. Under GDPR the processor must notify the controller and follow the contractual process; an internal label does not change the legal character. The practical point is building the notice period into the release plan — discovering it late leaves only a delay or a breach.

- **A 不正解**: 社内の位置づけは法的な該当性を左右しません。契約違反となります。 / Internal categorization does not change legal status; this breaches the DPA.
- **C 不正解**: 事前通知義務は受動的な開示では満たせません。 / An advance-notice obligation is not satisfied reactively.
- **D 不正解**: 一方的な改定はできず、顧客の同意なしに義務を消すことはできません。 / Obligations cannot be removed unilaterally.

**参照 / Reference:** GDPR、サブプロセッサ、DPA の事前通知義務、処理の連鎖
</details>

---

## 問題 15 / Question 15

**シナリオ / Scenario:**

エージェントのデバッグと品質分析のため、全リクエストのプロンプトとレスポンスを完全な形でログに保存しています。プロンプトには顧客の氏名・住所・健康状態に関する記述が含まれます。ログは開発者 30 名全員が検索可能なログ基盤に無期限で保存されており、アクセス記録は取っていません。開発チームは「デバッグに必要」と説明しています。

For debugging and quality analysis, complete prompts and responses are logged for every request. Prompts contain customers' names, addresses, and descriptions of health conditions. Logs are retained indefinitely in a platform searchable by all 30 developers, with no access records. The team explains that this is needed for debugging.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) デバッグに必要なので、現状を維持する / Keep it as is, since debugging requires it
- B) ログの保存を完全に停止する / Stop logging entirely
- C) **デバッグの必要性を認めたうえで、機微データの取り扱いを統制する**。ログに書き込む前に直接識別子と健康情報を除去または仮名化し、原データが必要な調査には権限を絞った別の経路（申請と承認、期間限定のアクセス、アクセス記録の取得）を設ける。保持期間を用途に見合う長さに設定して自動削除し、アクセスログを取得する。健康情報は多くの法域で特別な保護を受ける区分であることを前提に統制を設計する / **Accept the debugging need and control the handling of sensitive data**: strip or pseudonymize direct identifiers and health information before writing to logs, and provide a separate narrowly-permissioned path for investigations that genuinely need raw data (request and approval, time-limited access, access logging). Set retention to what the purpose requires with automatic deletion, and record access — designing the controls on the premise that health information is a specially protected category in most jurisdictions
- D) ログを暗号化して保存する / Store the logs encrypted

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

デバッグの必要性は正当ですが、**そのために健康情報を含む個人データを 30 名が無期限に検索できる状態にする必要はありません**。統制の要点は 4 つで、(1) 書き込み前の除去・仮名化（大半のデバッグはこれで足ります）、(2) 原データが必要な場合の限定された経路、(3) 用途に見合う保持期間と自動削除、(4) アクセス記録。健康情報は多くの法域で特別な保護区分に属するため、統制の水準を一般の個人データより上げる必要があります。**必要性を否定せず、必要な範囲に絞る**のが正しい設計です。

The debugging need is legitimate; making health data searchable by 30 people indefinitely is not what it requires. Four controls apply: redaction or pseudonymization before write (sufficient for most debugging), a narrow path for the cases needing raw data, purpose-bounded retention with automatic deletion, and access logging. Health information is a specially protected category in most jurisdictions and warrants a higher bar than ordinary personal data.

- **A 不正解**: 必要性は範囲を無制限にする根拠になりません。30 名・無期限・記録なしは必要性を大きく超えています。 / Necessity does not justify unlimited scope.
- **B 不正解**: 完全停止は障害調査と品質分析の能力を失わせます。統制すれば両立可能です。 / Eliminates a legitimate capability that controls can preserve.
- **D 不正解**: 暗号化は保存時の保護であり、権限を持つ 30 名が復号して閲覧できる状況は変わりません。 / Encryption at rest does not restrict the 30 authorized readers.

**参照 / Reference:** ログの機微データ統制、仮名化、保持期間、アクセス記録、特別カテゴリデータ
</details>

---

## 問題 16 / Question 16

**シナリオ / Scenario:**

EU 圏の顧客に対し、Claude を用いた与信限度額の自動設定を行っています。申込から限度額決定まで人間は介在せず、システムが自動で決定します。ある顧客から「なぜ自分の限度額が低く設定されたのか説明してほしい」「人間による再審査を求めたい」という申し出がありました。社内には「自動処理なので個別の説明はできない」という認識があります。

For EU customers, credit limits are set automatically using Claude, with no human involvement between application and decision. A customer asks why their limit was set low and requests human re-review. Internally, the assumption has been that "it's automated, so individual explanations aren't possible."

**設問 / Question:**

最も適切な理解と対応はどれですか？

Which understanding and response is most appropriate?

- A) **人間の介在なしに法的効果を及ぼす自動化された決定には、GDPR 上の特別な規律が及ぶと理解する**。データ主体には、当該決定に関する意味のある情報の提供、**人的介入を求める権利**、意見表明と異議申立ての権利が認められる。したがって、決定の主要因を説明できる形（構造化された理由）で保持し、人間による再審査の経路を整備する必要がある。「自動処理だから説明できない」は、要件を満たさない構成であることの表明にほかならない / **Recognize that an automated decision producing legal effects without human involvement is subject to specific GDPR provisions**: the data subject is entitled to meaningful information about the decision, **the right to obtain human intervention**, and rights to express a view and contest it. The principal factors must therefore be retained in explainable form (structured reasons) and a human re-review path must exist. "It's automated, so we can't explain" is a statement that the design does not meet the requirement
- B) 自動処理であるため、説明義務も再審査義務も生じない / Being automated, neither explanation nor re-review is owed
- C) 顧客に対して、モデルの内部動作は企業秘密であると回答する / Reply that the model's internals are a trade secret
- D) 当該顧客のみ手作業で再計算し、結果が同じなら説明不要とする / Recompute manually for this customer and, if the result matches, offer no explanation

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

**人間の介在なしに法的効果や重大な影響を及ぼす自動化された決定**は、GDPR で特別に規律される類型です。データ主体には、決定に関する意味のある情報を得る権利、人的介入を求める権利、異議を申し立てる権利が認められます。実装上の含意は明確で、(1) 主要因を構造化された形で保持しておくこと（自由記述では説明として弱く、後から再構成もできない）、(2) 人間による再審査の経路を業務プロセスとして持つこと、の 2 つです。「自動だから説明できない」という状態は、設計段階でこの要件を織り込まなかった結果です。

Automated decisions with legal or similarly significant effects and no human involvement are specifically regulated: the data subject can obtain meaningful information, human intervention, and the ability to contest. The design implications are concrete — retain principal factors in structured form (free text is both weaker as an explanation and unreconstructable later), and maintain a human re-review path as a business process. "Automated, therefore inexplicable" is the signature of a design that never accounted for this.

- **B 不正解**: 自動処理であることは義務の免除事由ではなく、むしろ特別な規律が及ぶ根拠です。 / Automation is the trigger for the provisions, not an exemption from them.
- **C 不正解**: 企業秘密は、決定の主要因についての説明を一切拒む根拠にはなりません。 / Trade secrecy does not license refusing any explanation of the principal factors.
- **D 不正解**: 同じ結果になったことは説明義務を消しません。また 1 件限りの対処では制度的な要件を満たしません。 / An identical outcome does not discharge the duty, and a one-off does not satisfy a systemic requirement.

**参照 / Reference:** GDPR の自動化された意思決定、人的介入を求める権利、説明可能な理由の保持
</details>

---

## 問題 17 / Question 17

**シナリオ / Scenario:**

コスト削減のため、エージェントの応答をキャッシュする仕組みを導入しました。キャッシュキーは「ユーザーの質問文のハッシュ」です。運用開始後、ある企業ユーザーから「他社の社名と契約金額が回答に含まれていた」という重大な報告がありました。調査すると、質問文が同一（「今月の契約状況を教えて」）であれば、別テナントのユーザーにも同じキャッシュが返っていました。

To reduce cost, agent responses are cached with a key derived from a hash of the user's question. After launch, a corporate customer reported that another company's name and contract value appeared in an answer. Investigation showed that identical question text ("show me this month's contract status") returned the same cached response across tenants.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) キャッシュの有効期間を 5 分に短縮する / Reduce the cache TTL to five minutes
- B) 質問文に社名が含まれる場合のみキャッシュを無効化する / Disable caching only when the question mentions a company name
- C) **キャッシュを直ちに無効化し、影響範囲を特定したうえで、キャッシュキーの設計を是正する**。キャッシュキーには、応答内容を左右するすべての要素（テナント ID、ユーザー ID、参照するデータ範囲、権限スコープ）を含めなければならない。ユーザー固有のデータを含む応答は、そもそもテナント境界を越えて共有され得ない設計にする。あわせて、どのテナントの情報がどのテナントに開示されたかを特定し、契約と規制に基づく通知義務を履行する / **Disable the cache immediately, determine the exposure, and correct the key design.** A cache key must include every factor that determines the response — tenant ID, user ID, the data scope consulted, the permission scope — and responses containing user-specific data must be structurally incapable of crossing a tenant boundary. In parallel, establish which tenants' information was disclosed to which, and discharge the resulting contractual and regulatory notification duties
- D) 該当ユーザーに謝罪し、キャッシュはそのまま継続する / Apologize to the reporting customer and continue caching

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

これは**テナント間の情報漏洩**であり、最も重大な種類のインシデントです。原因は明確で、キャッシュキーが応答を左右する要素（テナント、ユーザー、権限スコープ）を含んでいないことです。対応は 3 段階で、(1) 即座の停止（漏洩の継続を止める）、(2) 影響範囲の特定（誰の情報が誰に開示されたか — 通知義務の前提）、(3) 設計の是正。特に (2) は、契約上・規制上の通知義務を判断するために不可欠で、技術的修正だけで終わらせてはいけません。

This is cross-tenant disclosure, the most serious class of incident here. The cause is unambiguous: the cache key omits the factors that determine the response. The response has three parts — stop the ongoing disclosure immediately, determine who was exposed to whom (the prerequisite for notification duties), and fix the design. The second part is what makes this more than a technical fix.

- **A 不正解**: TTL を短くしても、5 分間はテナント境界を越えた共有が続きます。漏洩の原因が期間ではありません。 / A shorter window still leaks; duration is not the cause.
- **B 不正解**: 質問文の内容で判定する方法は網羅性がなく、社名が含まれない質問でも機微な情報が漏れます。 / Content-based heuristics are incomplete; leakage occurs without company names too.
- **D 不正解**: 原因を放置したままの謝罪は、他テナントへの漏洩が継続することを意味します。 / Leaves the ongoing disclosure in place.

**参照 / Reference:** テナント間漏洩、キャッシュキー設計、影響範囲の特定、通知義務
</details>

---

## 問題 18 / Question 18

**シナリオ / Scenario:**

決算数値の集計と開示文書の草案作成に Claude を使っています。この処理は財務報告に直接影響するため、SOX の内部統制の範囲に含まれます。開発チームは他システムと同様に、プロンプトとモデルの変更を週次でリリースしており、変更記録は Git のコミット履歴のみです。監査法人から内部統制の有効性について質問を受けました。

Claude is used to aggregate financial figures and draft disclosure documents. Because this feeds financial reporting, it falls within SOX internal-control scope. The team ships prompt and model changes weekly like any other system, with Git commit history as the only change record. The external auditor has asked about the effectiveness of internal controls.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) Git のコミット履歴を変更記録として提出する / Submit the Git commit history as the change record
- B) **SOX 範囲のシステムとして、統制を設計・文書化・運用証跡の保存まで整える**。変更管理（要求、影響評価、テスト、承認者と実装者の分離、本番反映の記録）、アクセス統制（誰が本番のプロンプトとモデル設定を変更できるか）、処理の完全性（集計結果と原データの突合、例外の検出と解消）を統制として定義し、各統制が期中を通じて機能した証跡を保存する。統制の設計と運用の両方が監査対象であることを前提にする / **Treat it as an in-scope system and build controls with design, documentation, and retained operating evidence**: change management (request, impact assessment, testing, separation of approver and implementer, production-deployment record), access control (who can change production prompts and model configuration), and processing integrity (reconciliation of aggregates against source data, detection and resolution of exceptions) — retaining evidence that each control operated throughout the period, on the premise that both control design and operating effectiveness are audited
- C) Claude の利用を決算処理から外し、手作業に戻す / Remove Claude from the close process and revert to manual work
- D) 監査法人に対し、AI の処理は統制の対象外であると説明する / Explain to the auditor that AI processing is out of control scope

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

財務報告に影響する処理は、**技術がどうであれ内部統制の対象**です。SOX の監査では、統制が適切に設計されていることと、**期中を通じて有効に運用されたことの証跡**の両方が問われます。Git のコミット履歴は変更の事実を示しますが、影響評価・テスト・承認・職務分離の証跡にはなりません。加えて、AI 特有の論点として**処理の完全性**（集計結果が原データと一致するかの突合）が重要で、確率的な処理を含む以上、決定的な突合による検証が統制の中核になります。

Anything affecting financial reporting is in control scope regardless of technology. A SOX audit asks both whether controls are designed appropriately and whether evidence shows they operated throughout the period. Commit history establishes that a change happened, not that impact was assessed, tested, approved, or separated by duty. The AI-specific control is processing integrity: because the processing is probabilistic, deterministic reconciliation against source data is the core assurance.

- **A 不正解**: コミット履歴は変更の記録にすぎず、承認・テスト・職務分離といった統制の証跡になりません。 / Commit history is not evidence of approval, testing, or segregation.
- **C 不正解**: 統制を整備すれば利用可能であり、手作業への回帰は必要ありません。 / The use case is viable under proper controls.
- **D 不正解**: 財務報告に影響する以上、対象外という説明は成立しません。 / Financial impact determines scope; the explanation is untenable.

**参照 / Reference:** SOX 内部統制、変更管理と職務分離、処理の完全性、運用証跡
</details>

---

## 問題 19 / Question 19

**シナリオ / Scenario:**

顧客対応エージェントが誤った案内を大量に出力していることが判明しました。停止しようとしたところ、停止する手段が「アプリケーションをデプロイし直す」しかなく、承認とビルドとデプロイに 50 分を要しました。その間に 1,400 件の誤った案内が顧客に届きました。同種のエージェントは他に 8 つあり、いずれも同じ状況です。

An agent was found emitting large volumes of incorrect guidance. The only way to stop it was to redeploy the application, which took 50 minutes for approval, build, and deploy. In that window, 1,400 incorrect responses reached customers. Eight similar agents exist, all in the same position.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) デプロイパイプラインを高速化して 10 分に短縮する / Speed the deploy pipeline to ten minutes
- B) 誤った案内を検出したら自動でエージェントを停止する仕組みを作る / Auto-disable the agent when incorrect guidance is detected
- C) 監視を強化して、より早く異常に気づけるようにする / Strengthen monitoring so anomalies are noticed sooner
- D) **デプロイを伴わずに即座に機能を停止・縮退できる仕組みを全エージェントに用意する**。機能フラグ等により、承認された運用者が数十秒で当該機能を停止するか、安全な縮退動作（定型応答、有人窓口への転送）に切り替えられるようにする。停止権限を持つ役割と発動基準を事前に定め、定期的に発動訓練を行って実際に動くことを確認する。停止は可逆な操作として設計し、復旧手順も定める / **Give every agent the ability to stop or degrade immediately without a deploy**: a feature flag or equivalent so an authorized operator can disable the capability, or switch it to a safe degraded mode (canned response, transfer to a human desk), within seconds. Pre-define who holds that authority and the criteria for using it, exercise it periodically to confirm it works, and design the stop as a reversible action with a documented restore procedure

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**顧客に影響を及ぼす機能には、デプロイを伴わずに即座に止める手段が必要**です。50 分という時間は、デプロイパイプラインの速度の問題ではなく、停止手段がデプロイに依存しているという設計の問題です。機能フラグによる停止・縮退は数十秒で発動でき、しかも可逆です。要点は仕組みだけでなく運用にもあり、**誰が止めてよいかを事前に決め、訓練で実際に動くことを確認する**ことが必要です。権限が曖昧だと、技術的に止められても組織的に止まりません。

Anything customer-facing needs a stop that does not require a deploy. Fifty minutes is not a pipeline-speed problem; it is a design problem in which stopping depends on deployment. A feature flag stops or degrades in seconds and is reversible. The operational half matters as much: pre-assigned authority and periodic exercises, because an ambiguous mandate means the technically-possible stop does not actually happen.

- **A 不正解**: 10 分でも 280 件相当の誤案内が届きます。デプロイに依存する構造自体が問題です。 / Ten minutes still means hundreds of incorrect responses; the dependency is the defect.
- **B 不正解**: 自動停止は有用な補助ですが、検出できない異常には効かず、誤検知による不要な停止のリスクもあります。手動の停止手段は依然として必要です。 / A useful complement, but it cannot cover undetected anomalies, and a manual stop is still required.
- **C 不正解**: 早く気づいても止める手段がなければ、被害の継続は止まりません。 / Faster detection without a stop does not shorten the exposure.

**参照 / Reference:** 緊急停止（キルスイッチ）、機能フラグによる縮退、停止権限、発動訓練
</details>

---

## 問題 20 / Question 20

**シナリオ / Scenario:**

マーケティング部門が、Claude を使って広告コピーとブログ記事を大量生成し、自社サイトと広告に掲載しています。法務部門から「生成物に第三者の著作物との類似が生じるリスク」「生成物の権利帰属」「他社の商標に抵触する表現」について整理するよう求められました。現在、生成物は担当者の確認のみで公開されており、確認は内容の正確性に限られています。

Marketing generates advertising copy and blog posts with Claude at volume and publishes them on the company site and in ads. Legal has asked for an assessment of the risk of similarity to third-party works, ownership of the output, and expressions that may infringe others' trademarks. Today output is published after a single staff check limited to factual accuracy.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **公開前の確認項目に知的財産の観点を加え、リスクに応じた手続きを定める**。既存著作物との類似（特に特徴的な表現の再現）、他社商標の使用、第三者の権利を示唆する記述について確認手順を設ける。商標については既存のクリアランス手続きに乗せ、露出の大きい素材（主要広告、ブランドサイト）は法務レビューを必須とする。生成物の権利帰属と利用条件についてはモデル提供者の規約と自社の取引条件を確認し、方針として文書化する / **Add intellectual-property checks to pre-publication review, with procedures scaled to risk**: check for similarity to existing works (especially reproduction of distinctive expression), use of others' trademarks, and statements implying third-party rights. Route trademark questions through the existing clearance process, and require legal review for high-exposure material (major campaigns, brand pages). Confirm ownership and permitted use of output against the provider's terms and your own commercial terms, and document the position as policy
- B) 生成物であることを明示すれば、権利上の問題は生じない / Labeling output as AI-generated resolves the rights questions
- C) 生成物の公開を全面的に停止する / Stop publishing generated content entirely
- D) 法務部門に全生成物のレビューを依頼する / Ask legal to review every generated item

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

生成物の公開に伴う知的財産上のリスクは、**既存の出版・広告のリスク管理に組み込むのが実務的**です。確認項目を明示し（類似、商標、第三者の権利への言及）、**露出の大きさに応じて手続きの重さを変える**ことで、量をこなしながら重要な素材には十分なレビューを確保できます。権利帰属と利用条件は、提供者の規約と自社の取引条件の両方を確認して方針として文書化しておくべき事項で、個別案件ごとに判断すると一貫性が失われます。

The IP risk of publishing generated content is best folded into existing publication and advertising risk management: name the checks, and scale the procedure to exposure so volume remains feasible while significant material gets real review. Ownership and permitted use should be settled once as documented policy against the provider's terms and your commercial terms — deciding case by case produces inconsistency.

- **B 不正解**: 生成物である旨の表示は透明性の問題であり、著作権や商標の問題を解消しません。 / Disclosure is a transparency measure; it does not resolve copyright or trademark exposure.
- **C 不正解**: 統制のもとで実施可能な活動であり、全面停止は過剰です。 / Disproportionate for an activity that is manageable under controls.
- **D 不正解**: 全件法務レビューは量的に成立せず、露出の小さい素材にまで同じコストをかけることになります。 / Does not scale, and applies the same cost to low-exposure material.

**参照 / Reference:** 生成物の知的財産リスク、露出に応じた審査、権利帰属の方針化
</details>

---

## 問題 21 / Question 21

**シナリオ / Scenario:**

購買業務のエージェントに、次のツールを付与しています: `create_purchase_order`（発注書の作成）、`approve_purchase_order`（発注の承認）、`register_vendor`（新規取引先の登録）、`issue_payment`（支払の実行）。エージェントは購買担当者の指示に基づいて動作し、承認は「1 件 50 万円未満なら自動承認」というルールで実行されます。内部監査から統制上の指摘を受けました。

A procurement agent holds these tools: `create_purchase_order`, `approve_purchase_order`, `register_vendor`, and `issue_payment`. It acts on a buyer's instructions, and approval runs on a rule that items under ¥500,000 are auto-approved. Internal audit has raised a control finding.

**設問 / Question:**

指摘される可能性が最も高い統制上の問題はどれですか？

Which control issue is most likely being raised?

- A) ツールの数が多すぎるため、エージェントが誤ったツールを選ぶ可能性がある / Too many tools, so the agent may choose the wrong one
- B) 自動承認の閾値が 50 万円では高すぎる / The ¥500,000 auto-approval threshold is too high
- C) **単一のエージェントが、取引先の登録・発注・承認・支払を一貫して実行できる構成になっており、職務分離が成立していない**。この組み合わせは、架空の取引先を登録して発注・承認・支払まで完結させることを技術的に可能にする。閾値以下であれば人間の関与なしに資金が外部へ流出し得るため、権限の分離（発注と承認、取引先登録と支払を別の主体に）と、取引先登録のような重要操作への人間の承認が必要になる / **A single agent can register a vendor, raise a purchase order, approve it, and issue payment end to end — segregation of duties does not hold.** That combination technically enables registering a fictitious vendor and completing the entire cycle, moving funds outside the company with no human involvement below the threshold. Permissions must be separated (ordering from approval, vendor registration from payment) and high-impact operations such as vendor registration gated behind human approval
- D) 購買担当者がエージェントに指示を出す形式が記録されていない / The buyer's instructions to the agent are not recorded

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

**職務分離は不正防止の中核的な統制**であり、人間の組織では「発注する人」「承認する人」「支払う人」「取引先を登録する人」を分けるのが原則です。エージェントに 4 つすべての権限を与えると、この統制が技術的に無効化されます。重要なのは、これが「エージェントが不正を働く」という話ではなく、**エージェントを操作できる者（内部者、あるいはプロンプトインジェクションによる外部者）が一連の流れを完結できてしまう**という点です。権限の分離と、取引先登録のような重要操作への人間の承認が必要です。

Segregation of duties is a core anti-fraud control: in a human organization, ordering, approving, paying, and vendor registration are held by different people. Granting one agent all four voids the control technically. The point is not that the agent commits fraud but that anyone able to direct it — an insider, or an outsider via prompt injection — can complete the whole cycle. Separate the permissions and gate vendor registration behind human approval.

- **A 不正解**: ツール選択の精度は運用上の問題であり、監査が指摘する統制上の欠陥とは性質が異なります。 / Tool-selection accuracy is an operational concern, not the control deficiency.
- **B 不正解**: 閾値の水準は議論の余地がありますが、職務分離が成立していないことの方が根本的な問題です。 / Debatable, but subordinate to the absence of segregation.
- **D 不正解**: 記録は重要ですが、権限が分離されていないという根本的な統制欠陥は記録では補えません。 / Records do not compensate for undivided authority.

**参照 / Reference:** 職務分離、エージェントの権限設計、不正防止統制、重要操作の承認
</details>

---

## 問題 22 / Question 22

**シナリオ / Scenario:**

新しいエージェントのリスク評価で、「一定の割合で誤った要約が生成され、それが下流の判断に影響する可能性がある」という残存リスクが特定されました。技術的な対策（評価、検証、人間のレビュー）を実施してもこのリスクをゼロにはできません。事業部門は「リスクは理解しているのでリリースしたい」と主張しています。プロジェクトマネージャーは口頭での合意でリリースを進めようとしています。

A risk assessment identified a residual risk: a proportion of summaries will be incorrect and may influence downstream decisions. Technical mitigations (evaluation, verification, human review) cannot reduce it to zero. The business unit says it understands the risk and wants to launch. The project manager intends to proceed on a verbal agreement.

**設問 / Question:**

最も適切な進め方はどれですか？

What is the most appropriate way to proceed?

- A) リスクがゼロにならないため、リリースを中止する / Cancel the launch, since the risk cannot be eliminated
- B) 事業部門が理解しているので、口頭の合意でリリースする / Proceed on the verbal agreement, since the business understands
- C) 残存リスクを社内文書に記載するが、承認は求めない / Record the residual risk internally without seeking approval
- D) **残存リスクを定量化し、受容する権限を持つ者による文書化された承認を得る**。誤りの発生率と影響の大きさ、実施済みの緩和策、残る影響と検知方法、発生時の対応手順を明記し、リスクの大きさに見合う職位（リスク所管部門または事業責任者）が署名する。受容には有効期限と再評価の条件（発生率が想定を超えた場合など）を付し、リリース後に実測値を追跡して前提が成立しているかを確認する / **Quantify the residual risk and obtain documented acceptance from someone with the authority to accept it**: state the error rate and impact, the mitigations already applied, the residual exposure and how it will be detected, and the response procedure if it materializes, signed at a level commensurate with the risk (the risk function or the accountable business owner). Give the acceptance an expiry and re-assessment triggers (for instance, the rate exceeding the assumption), and track actuals after launch to confirm the premises hold

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**リスクをゼロにできないことは、リリースできないことを意味しません。**適切なのは、残存リスクを明示して、それを受容する権限を持つ者が文書で承認することです。口頭の合意は、問題が起きた際に誰が何を了解していたかを再現できず、責任の所在も曖昧になります。**有効期限と再評価条件を付す**のが実務上の要点で、これがないと「一度受容したから永久に問題ない」という状態が生じます。リリース後に実測値を追跡し、受容の前提が成立しているかを確認する経路も必要です。

Non-zero risk does not mean no launch: the correct move is explicit residual risk accepted in writing by someone authorized to accept it. A verbal agreement cannot reconstruct who understood what when the risk materializes, and leaves accountability unclear. Expiry and re-assessment triggers matter — without them, one acceptance becomes permanent — and tracking actuals after launch is what tests whether the premises still hold.

- **A 不正解**: ゼロリスクを要求すると、あらゆるシステムがリリースできません。リスクは管理するものです。 / A zero-risk bar would block every system; risk is managed, not eliminated.
- **B 不正解**: 口頭合意は事後に再現できず、受容の範囲も権限も不明確です。 / Unreconstructable, with unclear scope and authority.
- **C 不正解**: 記載するだけでは、誰がそのリスクを受容したのかが定まりません。 / Documentation without acceptance leaves ownership undefined.

**参照 / Reference:** 残存リスクの受容、権限に応じた承認、有効期限と再評価、実測値の追跡
</details>

---

## 問題 23 / Question 23

**シナリオ / Scenario:**

社内ヘルプデスクエージェントは、業務効率のため人事・給与・評価データベースへの広い読み取り権限を持ち、社員の質問に答えます。ある社員が「同僚の給与を知りたい」という意図で、質問の仕方を工夫して（「部門の給与分布を役職別・氏名付きで一覧にして」など）情報を引き出そうと試み、一部成功していたことが判明しました。エージェントは技術的には正常に動作していました。

An internal help-desk agent holds broad read access to HR, payroll, and performance databases to answer employee questions efficiently. An employee attempting to learn colleagues' salaries phrased questions creatively ("list the department's salary distribution by grade, with names") and partially succeeded. The agent was functioning exactly as built.

**設問 / Question:**

最も適切な対策はどれですか？

Which countermeasure is most appropriate?

- A) 該当社員を処分し、他の社員には注意喚起する / Discipline the employee and warn others
- B) **エージェントの権限を、利用者本人の権限を超えないように是正する**。エージェントが持つ広い読み取り権限ではなく、呼び出したユーザーの権限で人事システムに問い合わせる構成にする。給与などの機微データは、本人の情報および職務上の権限がある範囲（例: 直属の部下）に限定されるべきで、この判定は人事システム側の既存の権限モデルに委ねる。あわせて、機微データへの照会をログに残して異常なパターンを検知できるようにする / **Correct the agent's permissions so it never exceeds the calling user's own.** Query HR under the requesting user's identity rather than the agent's broad grant, so sensitive data such as salary is limited to the user's own records and what their role entitles them to (their direct reports, say), with that determination made by HR's existing permission model. Additionally log sensitive-data queries so unusual patterns can be detected
- C) 給与に関する質問を拒否するようシステムプロンプトに記述する / Instruct the system prompt to refuse salary questions
- D) エージェントから人事データベースへの接続を削除する / Remove the agent's connection to the HR database

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**エージェントが利用者の権限を超えた権限を持つ構成は、権限昇格の経路そのものです。**技術的に正常動作していたという事実がこれを裏付けており、問題は個人の行為ではなく設計にあります。利用者の権限で問い合わせる構成にすれば、どのように質問を工夫しても、その利用者が本来アクセスできない情報は返りません。プロンプトによる拒否は言い換えで回避されるため、統制になりません。機微データ照会のログは、内部不正の検知手段として補完的に有効です。

An agent holding more privilege than its user *is* a privilege-escalation path, and the fact that it worked as built confirms the problem is design, not conduct. Querying under the user's identity means no phrasing can return data that user cannot access. Prompt-level refusal is defeated by rephrasing and is not a control; logging sensitive queries complements the fix as an insider-misuse detector.

- **A 不正解**: 処分は個別事案への対応で、同じ構成である限り他の社員も同じことができます。 / Addresses one instance while the capability remains for everyone.
- **C 不正解**: 言い換えで回避されるため統制になりません。「給与」という語を使わない照会も可能です。 / Defeated by rephrasing; the word "salary" need not appear.
- **D 不正解**: 接続の削除はヘルプデスクとしての機能（本人の情報照会など）を失わせます。権限を絞れば両立します。 / Removes legitimate functionality that scoping preserves.

**参照 / Reference:** 権限昇格の防止、利用者権限での実行、内部不正の検知、機微データの照会ログ
</details>

---

## 問題 24 / Question 24

**シナリオ / Scenario:**

金融庁による検査の事前通知を受けました。検査対象には、Claude を用いた顧客適合性判定システムが含まれます。検査では、システムの設計・運用・統制の有効性について、記録に基づく説明が求められます。社内では、開発チーム・リスク管理・コンプライアンス・内部監査がそれぞれ異なる資料を保有しており、統合された説明資料は存在しません。検査まで 4 週間です。

You have received advance notice of a regulatory examination that includes a Claude-based customer-suitability system. The examination will require records-based explanation of the system's design, operation, and control effectiveness. Internally, development, risk management, compliance, and internal audit each hold different materials; no consolidated package exists. The examination is four weeks away.

**設問 / Question:**

最も適切な準備はどれですか？

What is the most appropriate preparation?

- A) **検査で問われる観点に沿った証拠一式を整理し、記録と実態の一致を事前に確認する**。システムの目的と適用範囲、判断ロジック（プロンプト・モデル・参照基準）とそのバージョン管理、変更管理の承認記録、評価の方法と結果、人間による監督の仕組みとその実効性を示す運用データ、インシデントとその対処、残存リスクの受容記録を体系立てて揃える。あわせて、記録が実際の運用と一致しているかを内部で検証し、差異があれば検査前に是正または説明を用意する / **Assemble an evidence package organized around what the examination will ask, and verify in advance that the records match reality**: purpose and scope, the decision logic (prompt, model, referenced criteria) and its version control, change-management approvals, evaluation methodology and results, the human-oversight mechanism with operational data showing it is effective, incidents and their handling, and residual-risk acceptances. Separately, verify internally that the records reflect actual practice, and remediate or prepare an explanation for any divergence before the examination
- B) 検査当日に各部門の担当者を集めて、その場で説明する / Gather each department's staff on the day and explain in person
- C) 開発チームの技術資料をそのまま提出する / Submit the development team's technical materials as they are
- D) 検査までの 4 週間で、指摘されそうな箇所を修正する / Spend the four weeks fixing whatever seems likely to be criticized

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

検査対応の要点は、**問われる観点に沿って証拠を体系立てること**と、**記録と実態が一致していること**の 2 つです。後者はとりわけ重要で、文書上は統制があるのに実際には運用されていないという状態は、統制の不備そのものより重く扱われることがあります。したがって、資料を揃えるだけでなく、内部で実態を検証し、差異があれば検査前に是正するか、経緯を説明できるようにしておく必要があります。部門ごとに散在した資料は、検査官から見て統制の一貫性の欠如と映ります。

Examination readiness has two parts: organizing evidence around the questions that will be asked, and ensuring records match practice. The second matters most — a documented control that does not actually operate is often treated more seriously than an absent one. So verify internally, and remediate or be ready to explain divergences before the examination. Materials scattered across functions read to an examiner as inconsistent control.

- **B 不正解**: 口頭説明は記録に基づく説明の代わりにならず、部門間の不整合が露呈します。 / Verbal explanation does not substitute for records and exposes inconsistencies.
- **C 不正解**: 技術資料は統制の有効性を示す証拠として構成されておらず、問われる観点に答えません。 / Not organized as control evidence and does not answer the questions.
- **D 不正解**: 場当たり的な修正は体系的な準備にならず、記録と実態の不一致という最大のリスクにも対処しません。 / Ad hoc fixes are not preparation and miss the record-versus-practice risk.

**参照 / Reference:** 検査対応、証拠一式の整備、記録と実態の一致、統制の有効性の立証
</details>

---

## 問題 25 / Question 25

**シナリオ / Scenario:**

既に稼働している顧客対応エージェントに、新しいツール `update_contract_terms`（契約条件の変更）を追加したいという要望があります。既存のツールは参照系のみでした。開発チームは「ツールを 1 つ追加するだけなので、通常のリリースフローで進める」と考えています。このエージェントは外部から届く顧客メールの内容をコンテキストに含めます。

A request has come in to add a new tool, `update_contract_terms`, to a running customer-facing agent whose existing tools are read-only. The team considers this "just one more tool" and plans to ship it through the normal release flow. The agent places inbound customer email content into its context.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) ツールの追加は小さな変更なので、通常のリリースフローで進める / Proceed via the normal flow, as adding a tool is a small change

- B) ツールを追加したうえで、事後に監視を強化する / Add the tool and strengthen monitoring afterwards
- C) ツールの引数を減らして影響範囲を小さくする / Reduce the tool's arguments to limit its scope
- D) **参照系から更新系への変更はリスク区分の変更であると認識し、再評価を行う**。外部由来のコンテンツをコンテキストに含めるエージェントに、契約条件を変更する権限を与えることは、間接プロンプトインジェクションによる不正な契約変更を可能にし得る。リスク評価をやり直し、人間の承認ゲート、変更可能な範囲の限定（変更前後の値の制約）、決定的な検証、監査ログ、緊急停止手段を設計に含め、レッドチーミングで確認したうえでリリースする / **Recognize that moving from read-only to write changes the risk tier, and re-assess.** Granting contract-modification authority to an agent that ingests externally authored content creates a path to fraudulent contract changes via indirect prompt injection. Redo the risk assessment and build in a human approval gate, bounds on what may change (constraints on before/after values), deterministic validation, audit logging, and an emergency stop — verifying with red teaming before release

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

**参照系のみのエージェントに更新系のツールを追加することは、「ツールが 1 つ増える」以上の変化**です。システムの取り得る影響が、情報の開示から、契約という法的効果を持つ変更へと質的に変わります。とりわけ危険なのは、このエージェントが**外部から届くメールをコンテキストに含める**点で、攻撃者が送ったメールの内容が契約変更を誘発し得る構造になります。リスク区分の変更として再評価し、承認ゲート・範囲の限定・検証・監査・停止手段を揃えるのが正しい手順です。

Adding a write tool to a read-only agent is more than "one more tool": the system's possible impact changes in kind, from disclosing information to making legally effective changes. What makes it acute is that the agent ingests externally authored email, so attacker-supplied text sits on the path to a contract change. Re-assess as a tier change and build the gate, bounds, validation, audit, and stop before release.

- **A 不正解**: 変更の大きさをコード量で測っており、リスクの変化を見ていません。 / Measures change by code size rather than by risk.
- **B 不正解**: 事後監視では、既に実行された契約変更を防げません。 / Monitoring after the fact does not prevent an executed change.
- **C 不正解**: 引数を減らしても、外部コンテンツが契約変更を誘発し得る構造は変わりません。 / Fewer arguments do not remove the injection path.

**参照 / Reference:** リスク区分の変更、更新系ツールの追加審査、間接プロンプトインジェクション、多層の統制
</details>

---

## 問題 26 / Question 26

**シナリオ / Scenario:**

サポート業務のために顧客から収集した問い合わせ内容とチャット履歴を、マーケティング部門が「顧客の関心分析とターゲティング広告の最適化に使いたい」と提案しています。プライバシーポリシーには「サポート品質の向上のために利用する」と記載されています。データには顧客の氏名と連絡先が紐付いています。マーケティング部門は「既に保有しているデータを活用するだけ」と説明しています。

Marketing proposes using support inquiries and chat transcripts — collected for support purposes — to analyze customer interests and optimize targeted advertising. The privacy policy states the data is used "to improve support quality." The data is linked to customers' names and contact details. Marketing argues it is "just making use of data we already hold."

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 既に保有しているデータなので、追加の手続きなく利用してよい / No further steps are needed for data already held
- B) **収集目的と異なる利用は目的外利用に当たると整理し、必要な手続きを踏む**。当初の利用目的（サポート品質の向上）とターゲティング広告は目的が異なるため、そのままでは利用できない。取り得る選択肢は、プライバシーポリシーの改定と適切な告知・同意の取得、または個人を識別しない形への加工（統計的な傾向分析にとどめる）である。どちらを取るかを法務と決め、選んだ方法に応じた技術的な統制（識別子の除去、再識別の防止）を実装する / **Treat use for a different purpose as beyond the collection purpose and follow the required process.** Targeted advertising is a different purpose from improving support quality, so the data cannot simply be repurposed. The available paths are amending the privacy policy with proper notice and consent, or processing the data into a non-identifying form (limiting the work to statistical trends). Decide with legal which path to take, and implement the technical controls that path requires — identifier removal, re-identification prevention
- C) 顧客に個別に確認を取り、拒否しなかった顧客のデータを使う / Ask customers individually and use the data of those who do not object
- D) マーケティング部門にデータを渡さず、サポート部門が分析結果のみを共有する / Withhold the data and have support share only the analysis results

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

**目的制限は個人データ保護の基本原則**であり、「既に保有している」ことは別目的での利用を正当化しません。サポート品質の向上のために提供された情報を広告ターゲティングに使うのは、顧客の合理的な期待からも外れます。取り得る道は 2 つで、**適切な告知と同意による目的の追加**か、**個人を識別しない形への加工**です。後者を選ぶ場合、識別子を消すだけでは不十分なことがあり、再識別のリスクまで評価する必要があります。どちらの道でも、技術的な統制が伴って初めて実効性を持ちます。

Purpose limitation is a foundational principle: already holding data does not authorize a new purpose. Repurposing support conversations for ad targeting also falls outside what customers reasonably expect. Two paths exist — extend the purpose with proper notice and consent, or process into a genuinely non-identifying form — and for the latter, removing identifiers is often insufficient without assessing re-identification risk. Either path is real only with the technical controls attached.

- **A 不正解**: 保有していることは目的外利用の根拠になりません。目的制限の原則に反します。 / Possession does not authorize a new purpose.
- **C 不正解**: 拒否しなかったことをもって同意とみなす方式は、多くの法域で有効な同意と認められません。 / Opt-out silence is not valid consent in many jurisdictions.
- **D 不正解**: 分析結果のみの共有は運用上の工夫ですが、サポート部門が目的外の分析を行う点は変わりません。 / The out-of-purpose analysis still occurs, merely in a different department.

**参照 / Reference:** 目的制限、同意と告知、非識別化と再識別リスク、二次利用の統制
</details>

---

## 問題 27 / Question 27

**シナリオ / Scenario:**

一般消費者向けの保険相談エージェントで、ある利用者が会話の中で「もう生きていても仕方がない」という趣旨の記述を含めました。エージェントは保険商品の説明を続け、その発言には触れませんでした。この会話は事後のログレビューで発見されました。現在、エージェントにはこの種の状況に関する取り扱いが定義されておらず、人間へ引き継ぐ経路もありません。

In a consumer-facing insurance-advice agent, a user included a statement to the effect that there is no point in going on living. The agent continued explaining insurance products and did not acknowledge it. The exchange was found later in a log review. No handling for this kind of situation is defined, and there is no path to hand off to a human.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) エージェントは保険の相談窓口なので、対応範囲外として扱う / Treat it as out of scope, since the agent handles insurance inquiries
- B) システムプロンプトに「深刻な発言があれば適切に対応せよ」と追記する / Add "respond appropriately to serious statements" to the system prompt
- C) **危害の兆候に対する明確な取り扱いを設計し、人間への引き継ぎ経路を整備する**。当該類型の発言を検知した場合の応答方針（会話を打ち切らず、適切な相談先の情報を提示する）を定め、有人窓口への引き継ぎまたは通知の経路を用意する。方針は専門知識を持つ部門（産業保健、リスク管理、必要に応じて外部専門機関）の助言を得て策定し、担当者への訓練とあわせて運用する。検知と対応の記録を残し、定期的に見直す / **Design explicit handling for harm signals and build a human handoff path**: define the response policy when such a statement is detected (do not terminate the conversation; surface appropriate support resources), and provide a route to a staffed desk or a notification. Develop the policy with advice from functions holding the relevant expertise (occupational health, risk management, and external specialists where appropriate), pair it with training for the staff involved, retain records of detection and response, and review it periodically
- D) 該当する発言があった場合は会話を即座に終了する / Immediately terminate the conversation when such a statement appears

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: C**

**解説 / Explanation:**

一般消費者向けのシステムでは、**サービスの対象範囲外であっても、利用者の安全に関わる兆候への取り扱いを事前に設計しておく必要があります**。「対応範囲外」として何もしないことは、事業者としての姿勢の問題であると同時に、レピュテーションと法的なリスクでもあります。設計上の要点は 3 つで、(1) 応答方針（会話を打ち切らず、適切な相談先を示す）、(2) 人間への引き継ぎ経路、(3) 専門知識を持つ部門の関与です。プロンプトに一文足すだけでは、引き継ぎ先も担当者の訓練もないため機能しません。

Consumer-facing systems need pre-designed handling for signals bearing on user safety even when they fall outside the service's subject matter. Doing nothing on "out of scope" grounds is both a stance and a reputational and legal exposure. Three elements matter: a response policy that does not end the conversation and surfaces support resources, a human handoff, and involvement from functions with the relevant expertise. A prompt line alone has nowhere to hand off to and no trained recipient.

- **A 不正解**: 対象範囲の議論は、利用者の安全に関わる兆候を無視してよい理由になりません。 / Subject-matter scope does not justify ignoring a safety signal.
- **B 不正解**: 「適切に対応せよ」では具体的な行動が定まらず、引き継ぎ先も訓練もないため実効性がありません。 / Too vague to determine behavior, with no handoff or trained recipient behind it.
- **D 不正解**: 会話の即時終了は、利用者を突き放す対応であり、支援につながる経路も断ちます。 / Abruptly ending the conversation cuts off any route to support.

**参照 / Reference:** 危害の兆候への取り扱い、人間への引き継ぎ、専門部門の関与、記録と見直し
</details>

---

## 問題 28 / Question 28

**シナリオ / Scenario:**

基幹業務に組み込んだエージェントが、特定のモデルバージョンに強く依存しています（プロンプトと出力形式がそのバージョンに合わせて調整されている）。モデル提供者から、そのバージョンの提供終了予定が 9 か月後であるとの通知を受けました。事業継続計画（BCP）には AI サービスに関する記載がなく、経営層は「代替が効かないなら重大なリスクではないか」と懸念しています。

An agent embedded in core operations depends heavily on a specific model version, with prompts and output formats tuned to it. The provider has announced end of availability for that version in nine months. The business continuity plan says nothing about AI services, and leadership is asking whether an irreplaceable dependency constitutes a serious risk.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) **モデル依存を事業継続リスクとして管理対象に加え、移行計画と依存の低減を並行して進める**。9 か月の期限に対して、後継バージョンでの評価・差分分析・段階移行の計画を立てて実行する。同時に、特定バージョンへの依存を下げる構造改善（出力形式をスキーマで固定する、プロンプトをモデル固有の癖に依存しない形に整理する、評価スイートで移行判断を機械的に行えるようにする）を進める。BCP に AI サービスの依存と移行手順を記載し、以後の提供終了通知に対応できる体制にする / **Add model dependency to business-continuity management and pursue migration and dependency reduction in parallel.** Against the nine-month deadline, plan and execute evaluation on the successor, difference analysis, and staged migration. Concurrently reduce version-specific coupling — pin output format by schema, remove reliance on model-specific quirks from the prompt, and make the evaluation suite capable of deciding migration mechanically. Record the AI dependency and migration procedure in the BCP so future end-of-life notices are routine
- B) 9 か月あるので、期限が近づいてから対応する / There are nine months; act closer to the deadline
- C) モデル提供者に提供継続を要請する / Ask the provider to extend availability
- D) 自社でモデルを保有する方式に切り替える / Move to self-hosting a model

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: A**

**解説 / Explanation:**

対応には**短期（今回の移行を完了させる）と構造（次回以降の移行を容易にする）の 2 つの層**があります。短期だけを行うと、次のバージョン終了時に同じ規模の作業が再発します。依存を下げる構造改善（スキーマによる出力形式の固定、モデル固有の癖への依存排除、機械的に移行判断できる評価スイート）が、移行を定常的な運用作業に変えます。BCP への記載は、経営層の懸念に対する制度的な答えであり、AI サービスへの依存を他のベンダー依存と同じ枠組みで管理することを意味します。

The response has two layers: complete this migration, and make the next one cheap. Doing only the first guarantees a repeat at the next end-of-life. Structural work — schema-pinned output, prompts free of model-specific quirks, an evaluation suite that can decide migration mechanically — converts migration into routine operations. Recording it in the BCP answers leadership institutionally, managing AI dependency in the same frame as other vendor dependencies.

- **B 不正解**: 評価・差分分析・段階移行には時間が必要で、期限直前の着手は失敗時の余地を残しません。 / Evaluation, analysis, and staged migration need time; a late start leaves no room to recover.
- **C 不正解**: 提供終了は通常は提供者側の判断であり、要請は計画の代わりになりません。 / A request is not a plan, and the decision is generally not yours.
- **D 不正解**: 自社保有は運用・品質・コストの負担が大きく、依存先を変えるだけで事業継続リスクが消えるわけではありません。 / Shifts the dependency to your own operational capability at substantial cost.

**参照 / Reference:** モデル非推奨化への対応、事業継続計画、ベンダー依存の低減、移行の定常化
</details>

---

## 問題 29 / Question 29

**シナリオ / Scenario:**

社内で稼働する 27 のエージェントについて、それぞれが「人間の承認なしにどこまで実行してよいか」の基準が統一されていません。あるエージェントは 100 万円の支払を無人で実行でき、別のエージェントは社内文書の閲覧にも承認を求めています。判断は各チームが個別に行ってきました。リスク管理部門から、全社的な整理を求められています。

Across 27 internal agents, there is no consistent basis for how much each may do without human approval. One can issue a ¥1,000,000 payment unattended; another requires approval even to read an internal document. Each team decided independently. Risk management has asked for an organization-wide reconciliation.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) すべてのエージェントに人間の承認を必須とする / Require human approval for every agent action
- B) すべてのエージェントの承認を廃止して効率を優先する / Remove approvals everywhere in favor of efficiency
- C) 各チームの判断を尊重し、現状を追認する / Respect each team's judgment and ratify the status quo
- D) **自律性の水準を全社共通の基準で定義し、各エージェントの権限を分類に沿って見直す**。判断軸は操作の可逆性、影響の大きさ（金額・対象人数・法的効果）、誤りの検知可能性の 3 つとし、水準ごとに必要な統制（無人実行可、事後通知、事前承認、二重承認）を定める。各エージェントの保有ツールを棚卸しして水準に割り当て、逸脱があれば是正する。台帳として維持し、新規ツールの追加時に必ず分類を経る運用にする / **Define autonomy levels as a common standard and re-scope each agent's permissions accordingly.** Use three axes — reversibility, magnitude of impact (amount, people affected, legal effect), and detectability of errors — and specify the control required at each level (unattended, notify after the fact, prior approval, dual approval). Inventory every agent's tools, assign them to levels, remediate deviations, maintain it as a register, and require classification whenever a new tool is added

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: D**

**解説 / Explanation:**

自律性の水準がチームごとにばらばらである状態は、**高リスクな操作が無人で実行される一方、低リスクな操作に不要な承認がかかる**という二重の非効率を生みます。共通の判断軸（可逆性、影響の大きさ、検知可能性）を定めれば、各エージェントの権限を一貫した根拠で決められます。台帳として維持し、**新規ツール追加時に分類を必須のステップにする**ことが重要で、これがないと時間とともに再びばらつきます。全社基準は個別チームの判断を否定するものではなく、判断の枠組みを与えるものです。

Inconsistent autonomy produces two failures at once: high-risk actions running unattended while low-risk actions carry needless approvals. Common axes — reversibility, impact magnitude, detectability — let every agent's permissions be set on consistent grounds. Maintaining a register and making classification mandatory when tools are added is what prevents drift back to inconsistency. The standard supplies a framework for team judgment rather than replacing it.

- **A 不正解**: 一律の承認は承認疲れを招き、重要な承認の審査品質まで下げます。効率も大きく損ないます。 / Universal approval causes fatigue that degrades scrutiny where it matters.
- **B 不正解**: 100 万円の支払を無人で実行する構成を全社に広げることになり、リスク管理の要求と正反対です。 / Generalizes the riskiest configuration across the organization.
- **C 不正解**: 現状追認は、ばらつきそのものが問題であるという指摘に答えていません。 / Does not respond to the finding, which is the inconsistency itself.

**参照 / Reference:** 自律性水準の定義、可逆性と影響による分類、権限台帳、新規ツールの分類義務
</details>

---

## 問題 30 / Question 30

**シナリオ / Scenario:**

エージェントの設定不備により、ある顧客の回答に別の顧客 220 名分の氏名・メールアドレス・契約内容が含まれて送信されたことが判明しました。発覚は事象発生から 18 時間後です。社内では、技術チームが原因調査と修正を進めていますが、監督当局や本人への通知については「調査が完了してから判断する」という方針で進んでいます。対象者には EU 居住者が含まれます。

A misconfiguration caused one customer's response to include the names, email addresses, and contract details of 220 other customers, and it was sent. Discovery came 18 hours after the event. The technical team is investigating and remediating, while the internal position on notifying the supervisory authority and the individuals is to "decide once the investigation is complete." EU residents are among those affected.

**設問 / Question:**

最も適切な対応はどれですか？

Which response is most appropriate?

- A) 調査が完了して原因が確定してから、通知の要否を判断する / Decide on notification after the investigation establishes the cause
- B) **通知義務の時限は調査完了を待たないことを前提に、並行して対応を進める**。EU 居住者の個人データ侵害については、認識から 72 時間以内に監督当局へ通知する義務が生じ得るため、その時点で判明している事実（発生の性質、対象者のおおよその数と種類、想定される影響、講じた措置）で通知を行い、続報で補完する。本人への通知が必要となる要件を評価し、該当すれば遅滞なく行う。技術的な原因究明は並行して進め、通知判断を待たせない / **Proceed in parallel, on the premise that notification deadlines do not wait for the investigation to finish.** For a personal-data breach affecting EU residents, notification to the supervisory authority may be required within 72 hours of becoming aware, so notify with the facts known at that point — the nature of the breach, the approximate number and categories of people affected, likely consequences, and measures taken — and supplement later. Assess separately whether notification to the individuals is required and, if so, do it without undue delay. Technical root-cause work continues in parallel and does not gate the notification
- C) 影響を受けた 220 名にのみ通知し、当局への通知は行わない / Notify only the 220 individuals and not the authority
- D) 原因が設定不備であり悪意がないため、通知は不要と判断する / Conclude that no notification is needed, since the cause was a misconfiguration rather than malice

<details>
<summary>正解と解説 / Answer & Explanation</summary>

**正解 / Answer: B**

**解説 / Explanation:**

個人データ侵害の通知義務には**時限があり、調査の完了を待つことは許されません。**EU 居住者が含まれる以上、認識から 72 時間という時限を前提に、その時点で判明している事実で通知し、続報で補完するのが正しい進め方です。「調査完了後に判断する」という方針は、時限を徒過するリスクを直接生みます。本人への通知は別の要件で判断され、権利と自由への高いリスクが認められる場合に必要となります。技術対応と法務対応は並行して進めるべきもので、前者が後者を待たせてはなりません。

Breach-notification duties are time-bound and do not wait for a completed investigation. With EU residents affected, work from a 72-hour clock starting at awareness: notify with what is known and supplement afterwards. "Decide after the investigation" directly risks missing the deadline. Notification to individuals is assessed separately, required where there is high risk to their rights and freedoms. Technical and legal tracks run in parallel; the former must not gate the latter.

- **A 不正解**: 時限は調査の進捗と無関係に進みます。調査完了を待つ方針は義務違反のリスクを生みます。 / The clock runs regardless of investigative progress.
- **C 不正解**: 当局への通知と本人への通知は別個の要件であり、一方を行えば他方が免除されるわけではありません。 / The two duties are separate; satisfying one does not discharge the other.
- **D 不正解**: 通知義務の判断基準は悪意の有無ではなく、侵害の発生と権利・自由へのリスクです。 / The trigger is the breach and the risk to rights, not intent.

**参照 / Reference:** 個人データ侵害の通知義務、72 時間の時限、本人への通知要件、並行対応
</details>

---

> **目次 / Index:** [`README.md`](./README.md) | **前 / Previous:** [`domain4_evaluation.md`](./domain4_evaluation.md) | **次 / Next:** [`domain6_stakeholder_lifecycle.md`](./domain6_stakeholder_lifecycle.md)
