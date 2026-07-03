# claude-fable-5-skills 公式ドキュメント検証メモ(Qiita 記事用素材)

本リポジトリの 10 スキルが、Anthropic の公式ドキュメントに照らして妥当かを検証した記録。
各スキルについて「スキルがどう働くか」→「根拠となる公式ドキュメントの原文引用」→「日本語訳」の順でまとめる。

- 検証日: 2026-07-03
- 照合した公式ソース: 末尾の[参照元](#参照元)を参照

## 検証結果サマリ

| # | スキル | 判定 | 根拠となる公式セクション |
|---|--------|------|--------------------------|
| 1 | `skill-refactorer` | ✅ 妥当 | Prompting Claude Fable 5「Recommended scaffolding changes」 |
| 2 | `act-when-ready` | ✅ 妥当 | 同「Longer turns by default」 |
| 3 | `effort-calibrator` | ✅ 妥当(1点の食い違いを検証後に修正済み) | Effort ドキュメント / Claude Code「Model configuration」/ 移行ガイド |
| 4 | `no-gold-plating` | ✅ 妥当 | 同「Consider all effort levels」 |
| 5 | `grounded-progress` | ✅ 妥当 | 同「Ground progress claims during long runs」 |
| 6 | `scope-guard` | ✅ 妥当 | 同「State the boundaries」 |
| 7 | `subagent-orchestration` | ✅ 妥当 | 同「Parallel subagents」「Recommended scaffolding changes」 |
| 8 | `markdown-memory` | ✅ 妥当 | 同「Construct a memory system」 |
| 9 | `autonomous-continuation` | ✅ 妥当 | 同「Rare cases of early stopping」「Rare cases of context-budget concern」 |
| 10 | `regrounding-summary` | ✅ 妥当 | 同「Readability when communicating with the user」「Strong instruction following」 |

READMEの前提となる事実も確認済み:

- Fable 5 のリリース日「2026-06-09」→ 公式に "Claude Fable 5 and Claude Mythos 5 both become available on June 9, 2026" と一致。
- 「旧モデル向けスキルは Fable 5 には過剰に規範的で品質を下げうる」→ 公式ガイドに同趣旨の記述あり(スキル1参照)。

---

## 1. skill-refactorer — 旧モデル向けスキルの棚卸し

**どう働くか:** 旧モデル時代のスキル/プロンプトに含まれる「能力補償」(手取り足取りの手順、過剰な注意書き)を洗い出し、ガードレール(安全境界)だけ残して削除・書き直す監査ワークフローを与える。

**公式根拠:**

> **Refactor existing prompts and skills.** Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality. Review and consider removing older instructions if default performance is better.

*(訳)* **既存のプロンプトとスキルをリファクタリングする。** 旧モデル向けに開発されたスキルは Claude Fable 5 には過剰に規範的であることが多く、出力品質を劣化させうる。デフォルトの性能の方が良い場合は、古い指示の削除をレビュー・検討すること。

スキル内の「内部推論を応答に書き出させる指示は削除(拒否応答を誘発しうる)」という赤旗項目も公式記述と一致する:

> **Don't instruct Claude to reproduce its reasoning in the response.** Prompts, skills, or harness instructions that tell the model to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category on Claude Fable 5, causing elevated fallbacks to Claude Opus 4.8.

*(訳)* **Claude に推論内容を応答として再現させる指示を出さない。** 内部推論を応答テキストとして書き出す・転記する・説明するよう指示するプロンプトやスキル、ハーネス指示は、Claude Fable 5 で `reasoning_extraction` の拒否カテゴリを誘発し、Claude Opus 4.8 へのフォールバックを増加させうる。

---

## 2. act-when-ready — 過剰プランニングの抑制

**どう働くか:**「十分な情報が揃った瞬間に行動する」「会話内で確定済みの事実は再検証しない」「選ばない選択肢のカタログを提示しない」という判断閾値を明示し、高 effort 時の遅延・ノイズを削る。thinking ブロックには適用しない、という但し書きも持つ。

**公式根拠(タスクが曖昧なときの過剰プランニング対策として公式が提示するプロンプト):**

> When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.

*(訳)* 行動に足る情報が揃ったら、行動せよ。会話内で既に確定した事実を再導出したり、ユーザーが決定済みの事項を蒸し返したり、実行しない選択肢をユーザー向けメッセージで語ったりしないこと。選択を迷う場合は、網羅的な調査ではなく推奨案を1つ出すこと。これは thinking ブロックには適用されない。

背景となる挙動の記述:

> On routine work at higher effort, Claude Fable 5 can gather context and deliberate beyond what the task needs.

*(訳)* 高い effort 設定では、Claude Fable 5 はルーチン作業でもタスクに必要な範囲を超えてコンテキスト収集と熟考を行うことがある。

---

## 3. effort-calibrator — effort 設定の選択(検証で1点修正)

**どう働くか:** ワークロード種別ごとの effort 初期値の表と、上げ下げの判断シグナル、triage→escalate のパイプラインパターンを与える。

**公式根拠(Fable 5 向けの推奨):**

> Effort is the primary control for trading off intelligence, latency, and cost on Claude Fable 5. **Start with `high`, the default, for most tasks**, use `xhigh` for the most capability-sensitive workloads, and step down to `medium` or `low` for routine work. Lower effort settings on Claude Fable 5 still perform well and often exceed `xhigh` performance on prior models.

*(訳)* Fable 5 では effort が知能・レイテンシ・コストのトレードオフを制御する主要な手段である。**ほとんどのタスクはデフォルトの `high` から始め**、最も能力を要するワークロードには `xhigh` を、ルーチン作業には `medium` や `low` を使う。Fable 5 の低 effort 設定でも十分に性能が高く、旧モデルの `xhigh` を上回ることも多い。

`max` を安易に使わない、という記述の根拠(Opus 4.7 向け表の記述だが、Fable 5 のガイドでも `max` は推奨初期値に含まれない):

> Reserve for genuinely frontier problems. On most workloads `max` adds significant cost for relatively small quality gains, and on some structured-output or less intelligence-sensitive tasks it can lead to overthinking.

*(訳)* 真にフロンティアな問題のためにとっておくこと。ほとんどのワークロードでは `max` は比較的小さな品質向上のために大きなコストを追加し、構造化出力タスクなどでは考えすぎ(overthinking)につながることがある。

**検証で見つかった食い違い(修正済み):** スキルには当初「`xhigh` is the best setting for most coding/agentic work and is **Claude Code's default**」という記述があったが、これは現行の公式ドキュメントと食い違っていた。Claude Code のモデル設定ドキュメントには次のように明記されている:

> The default effort is `high` on Fable 5, Sonnet 5, Opus 4.8, Opus 4.6, and Sonnet 4.6, and `xhigh` on Opus 4.7.

*(訳)* デフォルトの effort は Fable 5・Sonnet 5・Opus 4.8・Opus 4.6・Sonnet 4.6 では `high`、Opus 4.7 では `xhigh` である。

「コーディング/エージェント作業は `xhigh` から」という推奨は Opus 4.7/4.8 向けのガイダンス("Start with `xhigh` for coding and agentic use cases")であり、Fable 5 向けの公式推奨は「`high` を既定とし、最も能力を要するワークロードで `xhigh`」。旧 Opus 向けガイダンスの持ち越しであることは、移行ガイド(Opus 4.8 → Fable 5)の次の記述で裏付けられる:

> **Start at `high` effort:** The effort parameter default remains `high`. On Claude Opus 4.8, the recommendation for coding and high-autonomy work is to set `xhigh` explicitly. On `claude-fable-5`, use `high` as the default for most tasks and reserve `xhigh` for the most capability-sensitive workloads.

*(訳)* **`high` effort から始める:** effort パラメータのデフォルトは `high` のまま。Claude Opus 4.8 ではコーディングや高自律性の作業に `xhigh` を明示設定するのが推奨だったが、`claude-fable-5` ではほとんどのタスクでデフォルトの `high` を使い、`xhigh` は最も能力を要するワークロードのためにとっておく。

移行チェックリストにも同趣旨の項目がある:

> Re-evaluate your `effort` setting. Start at `high` for most tasks, including workloads that ran at `xhigh` on Claude Opus 4.8.

*(訳)* `effort` 設定を再評価せよ。Claude Opus 4.8 で `xhigh` で動かしていたワークロードも含め、ほとんどのタスクは `high` から始めること。

これを受け、スキルの該当行は「`high`(API と Claude Code の Fable 5 デフォルト)から始め、最も能力を要するタスクで `xhigh` へ」に修正した。あわせて「旧モデルの最大設定(max)を上回る」としていた箇所も、公式表現("often exceed `xhigh` performance on prior models")に合わせて `xhigh` 比較に改めた(スキル本文・README・README.ja とも修正済み)。

---

## 4. no-gold-plating — 差分の最小化

**どう働くか:**「差分はリクエストと1対1対応」「単一呼び出し箇所へのヘルパー禁止」「仮想的な将来要件のための設計禁止」「検証はシステム境界のみ」というルールと、diff 確定前のセルフチェック3問を課す。

**公式根拠(高 effort 時の頼んでいない整頓・リファクタ対策として公式が提示するプロンプト):**

> Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well. Avoid premature abstraction and half-finished implementations. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.

*(訳)* タスクが要求する範囲を超えて機能追加・リファクタリング・抽象化を導入しないこと。バグ修正に周辺の掃除は不要で、一度きりの操作にヘルパー関数はたいてい不要。仮想的な将来要件のために設計せず、うまく動く最もシンプルな実装を選ぶこと。早すぎる抽象化と作りかけの実装を避けること。起こりえないシナリオへのエラーハンドリング・フォールバック・バリデーションを追加しないこと。内部コードとフレームワークの保証を信頼すること。検証はシステム境界(ユーザー入力、外部 API)でのみ行うこと。コードを直接変更できるなら、フィーチャーフラグや後方互換シムを使わないこと。

スキルの各ルールはこの公式プロンプトのほぼ全項目をカバーしており、忠実度が高い。

---

## 5. grounded-progress — 進捗報告の証拠拘束

**どう働くか:** 進捗報告の各主張を「このセッションのツール結果」に紐付けることを義務化する。完了→証明するコマンド/テストを名指し、失敗→出力をそのまま引用、未検証→明示ラベル。証拠のない主張は出荷させない。

**公式根拠:**

> On long autonomous runs, instruct Claude Fable 5 to audit progress against actual tool results. In Anthropic's testing, this nearly eliminated fabricated status reports even on tasks designed to elicit them:

*(訳)* 長時間の自律実行では、実際のツール結果に照らして進捗を監査するよう Claude Fable 5 に指示すること。Anthropic のテストでは、これにより捏造された進捗報告が——それを誘発するよう設計されたタスクでさえ——ほぼ根絶された。

公式が提示するプロンプト:

> Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.

*(訳)* 進捗を報告する前に、各主張をこのセッションのツール結果に照らして監査すること。証拠を指し示せる作業だけを報告し、未検証のものは未検証と明言すること。結果は忠実に報告すること: テストが失敗したなら出力とともにそう言い、ステップを飛ばしたならそう言い、完了して検証済みのものはためらいなく明言する。

---

## 6. scope-guard — 頼まれていない行動の禁止

**どう働くか:** リクエストを「問題の記述/質問/思考の垂れ流し」と「明示的な変更依頼」に分類させ、前者では調査・報告・推奨で止まる。状態変更コマンドの前には「その特定のアクション」を裏付ける証拠を要求し、頼まれていない副次成果物(メール、チケット、バックアップブランチ等)を禁止する。

**公式根拠:**

> Claude Fable 5 can occasionally take unrequested actions (drafting an email when none was asked for, creating defensive git-branch backups). Define explicit constraints on what Claude Fable 5 should and should not do:

*(訳)* Claude Fable 5 は時折、頼まれていない行動(依頼されていないメールの下書き、保険的な git ブランチのバックアップ作成など)を取ることがある。何をすべきで何をすべきでないか、明示的な制約を定義すること。

公式が提示するプロンプト:

> When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.

*(訳)* ユーザーが変更を依頼しているのではなく、問題を説明している・質問している・考えを声に出しているだけのときは、成果物はあなたの評価(アセスメント)である。所見を報告して止まること。求められるまで修正を適用しないこと。システム状態を変更するコマンド(再起動、削除、設定変更)を実行する前に、証拠がその特定のアクションを実際に裏付けているか確認すること。既知の障害にパターン一致するシグナルでも、原因は別のことがある。

---

## 7. subagent-orchestration — 並列サブエージェントの活用

**どう働くか:** 委譲の判断基準(独立性・規模・仕様化可能性)、同一ターンでの並列起動とノンブロッキング協調、コンテキストを持ち越す長寿命サブエージェント、そして「新鮮なコンテキストの検証者サブエージェント」パターンを与える。

**公式根拠(能力向上の記述):**

> **Delegation and collaboration.** Claude Fable 5 is significantly more dependable at dispatching and sustaining parallel subagents, and reliably manages ongoing communication with long-running subagents and peer agents.

*(訳)* **委譲と協調。** Claude Fable 5 は並列サブエージェントの派遣と維持において著しく信頼性が高く、長時間稼働するサブエージェントや対等なエージェントとの継続的な連携を確実にこなす。

協調パターンの根拠:

> Use subagents frequently, provide explicit guidance about when delegation is appropriate, and prefer asynchronous communication between orchestrator and subagents over blocking until each subagent returns. Long-lived subagents that keep their context across subtasks save time and cost through cache reads and avoid bottlenecking on the slowest subagent.

*(訳)* サブエージェントを積極的に使い、いつ委譲が適切かの明示的なガイダンスを与え、各サブエージェントの完了を待ってブロックするのではなく、オーケストレーターとサブエージェント間の非同期通信を優先すること。サブタスクをまたいでコンテキストを保持する長寿命サブエージェントは、キャッシュ読み込みで時間とコストを節約し、最も遅いサブエージェントがボトルネックになるのを防ぐ。

fresh-context verifier(新鮮なコンテキストの検証者)の根拠:

> **Make self-verification explicit in long-run prompts.** Separate, fresh-context verifier subagents tend to outperform self-critique. For long-running tasks, instruct: `Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.`

*(訳)* **長時間実行のプロンプトでは自己検証を明示すること。** 独立した、新鮮なコンテキストを持つ検証者サブエージェントは、自己批判(セルフクリティーク)を上回る傾向がある。長時間タスクでは次のように指示する:「構築しながら [X] の間隔で自分の作業をチェックする方法を確立せよ。これを [X] 間隔で実行し、サブエージェントを使って仕様に照らして作業を検証せよ。」

---

## 8. markdown-memory — ファイルベースの教訓メモリ

**どう働くか:** `memory/lessons/` に1教訓1ファイル+`INDEX.md` という構成で、修正(失敗)と確認(成功)の両方を「なぜ重要だったか」付きで記録・維持させる。重複禁止・既知情報の記録禁止・誤った教訓の削除という維持規律と、過去セッションからのブートストラップ手順を持つ。

**公式根拠:**

> Claude Fable 5 performs particularly well when it can record lessons from previous runs and reference them. Provide a place to write notes, as simple as a Markdown file:

*(訳)* Claude Fable 5 は、過去の実行から得た教訓を記録して参照できるとき、とりわけ良い性能を発揮する。メモを書く場所を用意すること——Markdown ファイル1つで十分である。

公式が提示するメモリ運用プロンプト(スキルの維持規律とほぼ1対1対応):

> Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.

*(訳)* 1ファイルに1教訓を、冒頭に1行サマリを付けて保存すること。修正事項と確認済みアプローチの両方を、なぜ重要だったかも含めて記録すること。リポジトリやチャット履歴に既にある情報は保存しない。重複を作らず既存のノートを更新する。誤りと判明したノートは削除する。

ブートストラップの根拠:

> Reflect on the previous sessions we've had together. Use subagents to identify core themes and lessons, and store them in [X]. Make sure you know to reference [X] for future use.

*(訳)* これまでのセッションを振り返れ。サブエージェントを使って中心的なテーマと教訓を特定し、[X] に保存せよ。今後 [X] を参照することを忘れないようにせよ。

---

## 9. autonomous-continuation — 無人実行の完走

**どう働くか:** 無人パイプライン向けの「自律性契約」(可逆なら進む、止まってよいのは破壊的操作・スコープ変更・ユーザーのみが持つ入力の3つ)と、ターン終了前に最終段落を読み返す「turn-ending check」、残りコンテキスト表示への動揺を防ぐ「context-budget composure」を課す。

**公式根拠(early stopping の記述):**

> Deep into a long session, Claude Fable 5 can occasionally end a turn with a text-only statement of intent ("I'll now run X") without issuing the corresponding tool call, or pause to ask permission when it already has enough to proceed.

*(訳)* 長いセッションの深部で、Claude Fable 5 は時折、対応するツール呼び出しを発行しないままテキストのみの意図表明(「これから X を実行します」)でターンを終えたり、続行に十分な情報があるのに許可を求めて停止したりすることがある。

公式が提示する自律パイプライン向けシステムリマインダー(抜粋):

> You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking "Want me to…?" or "Shall I…?" will block the work. For reversible actions that follow from the original request, proceed without asking. (…) Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ("I'll…", "let me know when…"), do that work now with tool calls. End your turn only when the task is complete or you are blocked on input only the user can provide.

*(訳)* あなたは自律的に動作している。ユーザーはリアルタイムで見ておらず、タスクの途中で質問に答えられないため、「〜しましょうか?」と尋ねると作業がブロックされる。元のリクエストから導かれる可逆的なアクションは、確認せずに進めること。(…)ターンを終える前に最終段落を確認せよ。それが計画・分析・質問・次のステップのリスト・まだやっていない作業への約束(「これから〜します」「〜したら教えてください」)であるなら、今すぐツール呼び出しでその作業を行うこと。タスクが完了したか、ユーザーのみが提供できる入力でブロックされたときにのみターンを終えること。

context-budget composure の根拠:

> In very long sessions, Claude Fable 5 can occasionally suggest a new session, offer to summarize and hand off, or trim its own work. This is most often triggered when the harness shows a remaining-token countdown to the model. Avoid surfacing explicit context-budget counts where possible. If the harness must show them, a reassurance helps:
>
> You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work.

*(訳)* 非常に長いセッションでは、Claude Fable 5 が新しいセッションを提案したり、要約して引き継ぐと申し出たり、自分の作業を切り詰めたりすることがある。これはハーネスが残りトークンのカウントダウンをモデルに見せている場合に最も起きやすい。可能であれば明示的なコンテキスト残量の表示を避けること。表示せざるを得ない場合は、次のような安心材料が有効:「コンテキストは十分に残っている。コンテキスト制限を理由に停止・要約・新セッションの提案をしないこと。作業を続けよ。」

チェックポイント配置(有人時)の根拠:

> Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise.

*(訳)* 作業が本当にユーザーを必要とするときだけ停止すること: 破壊的または不可逆なアクション、本物のスコープ変更、ユーザーのみが提供できる入力。これらに当たったら、約束でターンを終えるのではなく、質問してターンを終えること。

---

## 10. regrounding-summary — 読者を再接地させる最終報告

**どう働くか:** 最終メッセージを「作業を一切見ていない読者への初めての報告」として書かせる。結論から始める、完全な文で書く、矢印チェーンや自作略語を禁止、識別子には平易な説明句を付ける、圧縮ではなく取捨選択で短くする——というルール群。

**公式根拠:**

> In extended or agentic conversations (many tool calls, large working context), Claude Fable 5 can produce text that's hard to follow: dense arrow-chain shorthand, deep implementation detail, references to thinking the user never saw, or overly technical phrasing.

*(訳)* 長時間またはエージェント的な会話(大量のツール呼び出し、大きな作業コンテキスト)では、Claude Fable 5 は追いにくいテキストを生成することがある: 密度の高い矢印チェーンの速記、深すぎる実装詳細、ユーザーが見ていない思考への言及、過度に技術的な言い回しなど。

公式が提示するコミュニケーションスタイル補遺(抜粋):

> Your final summary is different: it's for a reader who didn't see any of that. (…) Write it as a re-grounding, not a continuation of your working thread: the outcome first, then the one or two things you need from them, each explained as if new. The vocabulary you built up while working is yours, not theirs; leave it behind unless you re-introduce it. (…) Don't use arrow chains, hyphen-stacked compounds, or labels you made up earlier. When you mention files, commits, flags, or other identifiers, give each one its own plain-language clause. Open with the outcome: one sentence on what happened or what you found. (…) If you have to choose between short and clear, choose clear.

*(訳)* 最終サマリは別物である: それは作業の過程を一切見ていない読者のためのものだ。(…)作業スレッドの続きではなく「再接地(re-grounding)」として書くこと: まず結果、次に読者に必要な1〜2点を、それぞれ初出のものとして説明する。作業中に築いた語彙はあなたのものであって読者のものではない。再導入しない限り置いていくこと。(…)矢印チェーン、ハイフンで連結した複合語、自分が途中で作ったラベルを使わないこと。ファイル・コミット・フラグなどの識別子に言及するときは、それぞれに平易な説明句を付けること。結果から始めよ: 何が起きたか・何が分かったかを1文で。(…)短さと明瞭さのどちらかを選ぶなら、明瞭さを選べ。

「選択で短くする(圧縮で短くしない)」の根拠(「Strong instruction following」セクションの簡潔さ指示):

> The way to keep output short is to be selective about what you include (drop details that don't change what the reader would do next), not to compress the writing into fragments, abbreviations, arrow chains like A → B → fails, or jargon.

*(訳)* 出力を短く保つ方法は、含める内容を取捨選択すること(読者の次の行動を変えない詳細を落とすこと)であって、文章を断片・略語・「A → B → fails」のような矢印チェーン・専門用語に圧縮することではない。

---

## 総評

- 10 スキルすべてが、公式ガイド「Prompting Claude Fable 5」の各セクションと明確に対応しており、多くはガイドが提示するプロンプト例の内容を(独自の文面で)忠実に再構成している。README の「Original text(指示文はすべて書き下ろし)」というポリシーどおり、丸写しではないが趣旨は一致する。
- 唯一の実質的な食い違いは **`effort-calibrator` にあった「xhigh は Claude Code のデフォルト」という記述**。現行公式ドキュメントでは Fable 5 の Claude Code デフォルト effort は `high` であり、`xhigh` がデフォルトなのは Opus 4.7。移行ガイドの記述から、Opus 4.7/4.8 向けガイダンス(コーディングは `xhigh` から)の持ち越しと確認できたため、検証後にスキル本文と README(英・日)を公式推奨に合わせて修正した(詳細はスキル3の項)。
- 補足: 公式の Fable 5 紹介ページでは [memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) がサポート機能として挙げられており、`markdown-memory` スキルは API のメモリツールを使わないハーネスでも同じ効果を得るための実装、と位置づけられる。

## 参照元

1. [Prompting Claude Fable 5 — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — スキル 1, 2, 4〜10 の主要根拠
2. [Effort — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/effort) — スキル 3(effort レベル定義、Fable 5 向け推奨、`output_config.effort` の API 形状)
3. [Introducing Claude Fable 5 and Claude Mythos 5 — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5) — リリース日(2026-06-09)、adaptive thinking、サポート機能一覧
4. [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — Claude Code における各モデルのデフォルト effort(Fable 5 = `high`)、`/effort` コマンド、ultracode
5. [Migration guide — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — Opus 4.8 → Fable 5 移行時の effort 再評価(`xhigh` → `high`)

(いずれも 2026-07-03 時点の取得内容に基づく)
