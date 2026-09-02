# claude-fable-5-skills 公式ドキュメント検証メモ(Qiita 記事用素材)

本リポジトリの 10 スキルが、Anthropic の公式ドキュメントに照らして妥当かを検証した記録。
各スキルについて「スキルがどう働くか」→「根拠となる公式ドキュメントの原文引用」→「日本語訳」の順でまとめる。

- 検証日: 2026-07-03(追補: 2026-08-29 — [追補セクション](#追補2026-08-29)、2026-09-02 — [Fable 5.1 追補](#追補2026-09-02-fable-51-対応)を参照)
- 照合した公式ソース: 末尾の[参照元](#参照元)を参照

## 検証結果サマリ

| # | スキル | 判定 | 根拠となる公式セクション(Fable 5 ガイド等) | 5.1 ガイドでの対応セクション |
|---|--------|------|--------------------------|------------------------------|
| 1 | `skill-refactorer` | ✅ 妥当 | Prompting Claude Fable 5「Recommended scaffolding changes」(スキルのオンザフライ更新の記述を含む) | 「Formatting in chat」「Ask for user-facing progress updates」(赤旗 2 点) |
| 2 | `act-when-ready` | ✅ 妥当 | 同「Longer turns by default」 | 該当セクションなし(過剰プランニングの記述は Fable 5 ガイドのみ) |
| 3 | `effort-calibrator` | ✅ 妥当(1点の食い違いを検証後に修正済み) | Effort ドキュメント(「Changing effort mid-conversation」「Best practices」を含む) / Claude Code「Model configuration」/ 移行ガイド | 「Consider all effort levels」「Search triggering at low effort」「Leave room for long outputs at xhigh and max effort」/ What's new「Change effort mid-conversation (beta)」「Pricing」 |
| 4 | `no-gold-plating` | ✅ 妥当 | 同「Consider all effort levels」 | 「Keep changes and tests to what the task asks for」 |
| 5 | `grounded-progress` | ✅ 妥当 | 同「Ground progress claims during long runs」 | 証拠ルールの該当セクションなし。ハーネス向け注記のみ「Ask for user-facing progress updates」 |
| 6 | `scope-guard` | ✅ 妥当 | 同「State the boundaries」 | 「Finish the whole task」(例外節に同趣旨の文) |
| 7 | `subagent-orchestration` | ✅ 妥当 | 同「Parallel subagents」「Recommended scaffolding changes」「Give the reason, not only the request」 | 「Let the lead agent keep working while subagents run」「Batch independent tool calls in agent loops」 |
| 8 | `markdown-memory` | ✅ 妥当 | 同「Construct a memory system」 | 該当セクションなし(memory の記述は Fable 5 ガイドのみ) |
| 9 | `autonomous-continuation` | ✅ 妥当 | 同「Rare cases of early stopping」「Rare cases of context-budget concern」「Create a send-to-user tool」 | 「Finish the whole task」「Keep the conversation history append-only」 |
| 10 | `regrounding-summary` | ✅ 妥当 | 同「Readability when communicating with the user」「Strong instruction following」 | 「Ask for user-facing progress updates」「Writing density」「Quoting retrieved sources」 |

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

**どう働くか:** ワークロード種別ごとの effort 初期値の表と、上げ下げの判断シグナル、triage→escalate のパイプラインパターンを与える。エスカレーションを別リクエストとして行う理由(プロンプトキャッシュ)と、それがバッチパイプライン固有の論点であって対話セッションでは判断シグナルに従ってよいことも含む([追補1](#追補1-会話途中の-effort-変更とプロンプトキャッシュ--effort-calibrator))。

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

**どう働くか:** 委譲の判断基準(独立性・規模・仕様化可能性)、同一ターンでの並列起動とノンブロッキング協調、コンテキストを持ち越す長寿命サブエージェント、そして「新鮮なコンテキストの検証者サブエージェント」パターンを与える。ハンドオフテンプレートは目標・依頼理由・入力・完了条件・制約・出力先を各1文で埋めさせる([追補2](#追補2-依頼の理由を渡す--subagent-orchestration))。

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

**どう働くか:** 無人パイプライン向けの「自律性契約」(可逆なら進む、止まってよいのは破壊的操作・スコープ変更・ユーザーのみが持つ入力の3つ)と、ターン終了前に最終段落を読み返す「turn-ending check」、残りコンテキスト表示への動揺を防ぐ「context-budget composure」を課す。加えて、ターンを終えずにユーザーへ逐語のメッセージを届ける `send_to_user` ツールをハーネス作者向けに案内する([追補3](#追補3-send-to-user-ツール--autonomous-continuation))。

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

## 追補(2026-08-29)

現行の公式ドキュメント(Prompting Claude Fable 5 / Effort)に対して再照合を実施した。7/3 検証時に参照していなかった記述を 5 点スキルに反映しており、以下では本文と同じく「原文引用」→「日本語訳」→「反映内容」の順に記録する。

### 追補1. 会話途中の effort 変更とプロンプトキャッシュ → `effort-calibrator`

**公式根拠(Effort ドキュメント「Changing effort mid-conversation」):**

> `output_config.effort` is a request-level setting: each request carries its own value, so to run a later part of a conversation at a different effort level, set the new value on the next request. The effort level applies to the whole request. Because effort shapes the rendered prompt, changing it between requests does not preserve cached prefixes from earlier turns; if you rely on prompt caching across a long session, pick an effort level at the start and keep it constant.

*(訳)* `output_config.effort` はリクエスト単位の設定である。各リクエストが自分の値を持つため、会話の後半を別の effort で動かしたければ次のリクエストに新しい値を設定する。effort レベルはリクエスト全体に適用される。effort はレンダリングされるプロンプトを変えるため、リクエスト間で変更すると以前のターンのキャッシュ済みプレフィックスは保たれない。長いセッションでプロンプトキャッシュに依存しているなら、最初に effort レベルを決めて一定に保つこと。

同ドキュメントの Best practices は、この「固定」が動的な調整と両立することを明示している:

> 4. **Consider dynamic effort:** Adjust effort based on task complexity. Simple queries may warrant low effort while agentic coding and complex reasoning benefit from high effort. See the next item before varying it within one conversation.
> 5. **Hold effort constant within cached conversations:** Changing the effort value between requests invalidates prompt caching, so vary effort across workloads rather than within a conversation that relies on cache hits.

*(訳)* 4. **動的な effort を検討する:** タスクの複雑さに応じて effort を調整する。単純なクエリは低 effort で足り、エージェント的コーディングや複雑な推論は高 effort が有効。ただし1つの会話の中で変える前に次項を読むこと。 5. **キャッシュを使う会話の中では effort を固定する:** リクエスト間で effort 値を変えるとプロンプトキャッシュが無効化されるため、キャッシュヒットに依存する会話の内部ではなく、ワークロードをまたいで effort を変えること。

**反映:** `effort-calibrator` のパイプラインパターンに、エスカレーションは別リクエストとして行う旨と、その理由(triage 側の会話のキャッシュ済みプレフィックスが自分の後続ターン用に保たれる)を追記。あわせて、これがバッチパイプラインの論点であり、対話セッションでは上段の Adjustment signals に従って effort を変えてよいことを明記した——公式の 4 と 5 の並びがこの区別に対応する。なお、エスカレート先のリクエスト自体は effort が変わる以上キャッシュミスになる点も本文に含めている。

### 追補2. 依頼の理由を渡す → `subagent-orchestration`

**公式根拠(「Give the reason, not only the request」):**

> Claude Fable 5 tends to perform better when it understands the intent behind a request: context lets it connect the task to relevant information rather than inferring intent on its own. Provide context about why you're asking, especially for long-running agents drawing on multiple workstreams:
>
> I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].

*(訳)* Claude Fable 5 は依頼の背後にある意図を理解しているとき、より良い性能を発揮する傾向がある。コンテキストがあれば、自力で意図を推測するのではなく、タスクを関連情報に結びつけられる。なぜ依頼しているのかの背景を与えること。特に複数のワークストリームにまたがる長時間稼働エージェントでは重要である:「私は[より大きなタスク]を[誰のため]に進めている。相手は[この成果物で何ができるようになるか]を必要としている。それを踏まえて: [依頼]」

**反映:** ハンドオフテンプレートの goal 項目に「その背後にある理由」を追加。テンプレートは簡潔なブリーフを定義するものなので、goal と理由それぞれ1文という長さ制約は維持している。

### 追補3. send-to-user ツール → `autonomous-continuation`

**公式根拠(「Create a send-to-user tool」):**

> When running long, asynchronous agents, give the agent a way to surface a message the user must see exactly as written, without ending its turn: a deliverable (a generated code snippet or a drafted message), a progress update with specific numbers, or a direct reply to a question the user asked mid-loop. The tool's input is the message to display; when Claude calls it, render the input directly in your UI and return a simple acknowledgement as the tool result. Tool inputs are never summarized, so the content arrives intact.

*(訳)* 長時間の非同期エージェントを動かすときは、ターンを終えずに、ユーザーが書かれたとおりに見なければならないメッセージを表に出す手段をエージェントに与えること——成果物(生成したコード片や下書きしたメッセージ)、具体的な数値を含む進捗更新、ループの途中でユーザーが投げた質問への直接の回答など。ツールの入力が表示すべきメッセージであり、Claude がそれを呼んだら入力をそのまま UI に描画し、ツール結果としては単純な受領応答を返す。ツール入力は決して要約されないため、内容は無傷で届く。

> Defining the tool is not sufficient on its own; without an instruction in the system prompt, Claude Fable 5 rarely calls it.

*(訳)* ツールを定義するだけでは不十分である。システムプロンプトでの指示がなければ、Claude Fable 5 がそれを呼ぶことはめったにない。

**反映:** `autonomous-continuation` にハーネス向けセクションとして追記。この文書全体がシステムプロンプトへ貼り付けて使われうる(README がそう案内している)ため、見出しを「for the harness author, not the model」とし、モデル自身への指示ではないことを明示した。

### 追補4. スキルのオンザフライ更新 → `skill-refactorer`

**公式根拠(「Recommended scaffolding changes」):**

> **Refactor existing prompts and skills.** Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality. Review and consider removing older instructions if default performance is better. Claude Fable 5 also does a good job of updating skills on the fly based on what it learns from the task at hand.

*(訳)* **既存のプロンプトとスキルをリファクタリングする。** 旧モデル向けに開発されたスキルは Claude Fable 5 には過剰に規範的であることが多く、出力品質を劣化させうる。デフォルトの性能の方が良い場合は、古い指示の削除をレビュー・検討すること。また Claude Fable 5 は、手元のタスクから学んだことに基づいてスキルをその場で更新することも得意である。

(スキル1の項で引用したのと同じ箇条書きの最終文。7/3 時点では引用範囲に含めていなかった。)

**反映:** A/B テストの項に追記。ただし公式のこの一文は能力の記述であって、同スキルが課すガードレール(「ユーザーと出力を比較してから完了とする」「ガードレールを黙って落とさない」)を解除するものではないため、同じ規律の下に置く形——編集は提案として出す、ガードレールを黙って落とさない——で書いている。

### 追補5. 完了後のフォローアップ提案 → `autonomous-continuation`

**公式根拠(「Rare cases of early stopping」の自律パイプライン向けシステムリマインダー。スキル9の項では `(…)` で省略していた箇所):**

> For reversible actions that follow from the original request, proceed without asking. Offering follow-ups after the task is done is fine; asking permission after already discussing with the user before doing the work is not.

*(訳)* 元のリクエストから導かれる可逆的なアクションは、確認せずに進めること。タスク完了後にフォローアップを提案するのは構わない。作業前に、ユーザーと既に議論した内容について許可を求めるのは駄目である。

**反映:** 自律性契約に反映。ただし同じリマインダーの turn-ending check は「次のステップのリスト」で終わるターンを未完了と見なすため、そのままでは両者が衝突する。スキル側では、契約側を「完了後に、スコープ外のフォローアップを提案するのは可」、turn-ending check 側を「スコープ内の次ステップのリスト」で終わるならターンは終わっていない、と切り分けて整合させた。

### API 形状の再確認(`effort-calibrator`)

> Setting `effort` to `"high"` produces exactly the same behavior as omitting the `effort` parameter entirely.

*(訳)* `effort` に `"high"` を設定することは、`effort` パラメータを完全に省略するのとまったく同じ挙動になる。

> At `high` and `xhigh`, set a large `max_tokens`: it is a hard limit on total output, thinking plus response text.

*(訳)* `high` と `xhigh` では `max_tokens` を大きく設定すること。これは thinking と応答テキストを合わせた総出力に対する上限である。

`output_config.effort` という API 形状を含め、スキル3の記述は現行ドキュメントと一致していることを再確認した。

## 追補(2026-09-02)— Fable 5.1 対応

2026-09-01 に Claude Fable 5.1(API ID `claude-fable-5-1`)がリリースされ、公式ドキュメント上 Fable 5 は "Legacy models (still available)" に移った。公式は既存の Fable 5 向けプロンプトが 5.1 でも無変更で動くとしつつ、いくつかの挙動差を公開している。ここでは各スキルへの反映を、本文と同じく「原文引用」→「日本語訳」→「反映内容」の順に記録する。8/29 追補までの根拠(Fable 5 ガイド)は 5.1 にも引き続き適用される前提で残す。

前提となる事実の確認:

> Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes, but a handful of behavioral differences are worth knowing about.

*(訳)* 既存の Claude Fable 5 向けプロンプトは、変更なしで Claude Fable 5.1 でも良好に動作するはずだが、知っておく価値のある挙動差がいくつかある。(Prompting Claude Fable 5.1 冒頭)

> Legacy models (still available): Claude Fable 5, Claude Opus 4.8, (…)

*(訳)* レガシーモデル(引き続き利用可能): Claude Fable 5、Claude Opus 4.8、(…)(Models overview 末尾。同ページの比較表では Fable 5.1 の Retirement が "Not sooner than September 1, 2027" とされ、リリース日 2026-09-01 の 1 年後に当たる)

### 追補6. effort の再スイープと世代間の比較 → `effort-calibrator`

**公式根拠(「Consider all effort levels」):**

> Start at the default effort level, `high`, then test the other levels (`low`, `medium`, `xhigh`, and `max`) against your own evals. Effort is the primary control for trading off intelligence, latency, and cost on Claude Fable 5.1. Re-run the sweep even if you already ran one on Claude Fable 5: effort level names don't correspond to the same amount of thinking across models.
>
> Claude Fable 5.1's capability gains over Claude Fable 5 show up across effort levels and are largest at the higher settings. At `medium`, results roughly match Claude Fable 5 at lower cost, so step down to `medium` or `low` where your evals show quality holds. At `low`, Claude Fable 5.1 is often competitive with Claude Opus and Claude Sonnet models on cost per task while scoring higher, so include it in the comparison wherever you'd otherwise run a smaller model at a higher effort level.

*(訳)* デフォルトの `high` から始め、他のレベル(`low`、`medium`、`xhigh`、`max`)を自分の評価セットで試すこと。Claude Fable 5.1 でも effort は知能・レイテンシ・コストのトレードオフを制御する主要な手段である。Claude Fable 5 でスイープ済みでも、もう一度実行すること: effort レベル名はモデル間で同じ思考量に対応しない。 / Claude Fable 5.1 の Fable 5 に対する能力向上はすべての effort レベルで現れ、高い設定で最も大きい。`medium` では Fable 5 とほぼ同等の結果をより低いコストで得られるので、評価で品質が保てる範囲で `medium` や `low` に下げること。`low` では、タスクあたりコストで Claude Opus や Claude Sonnet と競合しつつ高いスコアを出すことが多いので、小さいモデルを高 effort で動かしていた箇所は比較対象に含めること。

**反映:** 冒頭段落を書き換え、「Fable 5 は低 effort で旧モデルの `xhigh` を超える」に加えて 5.1 向けの参照点(`medium` で Fable 5 相当、`low` で Opus/Sonnet より高スコア)と「世代をまたいで effort 名は同じ思考量を意味しないので 5.1 で再スイープ」を明記。description にも「Fable 5 から 5.1 への effort 移行」をトリガーとして追加。

### 追補7. 会話途中の effort 変更(per-message effort)→ `effort-calibrator`

**公式根拠(What's new「Change effort mid-conversation (beta)」):**

> On Claude Fable 5.1 you can change the effort level mid-conversation without invalidating the prompt cache. Raise it for a hard step and lower it for routine ones. Per-message effort is in beta: include the `mid-conversation-output-config-2026-07-01` beta header. Claude Fable 5.1, Claude Mythos 5.1, and Claude Opus 5 support it on the Claude API.

*(訳)* Claude Fable 5.1 では、プロンプトキャッシュを無効化せずに会話の途中で effort レベルを変更できる。難しいステップでは上げ、ルーチンのステップでは下げること。per-message effort はベータで、`mid-conversation-output-config-2026-07-01` ベータヘッダを含めること。Claude API 上では Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5 が対応する。

Effort ドキュメントは形式と、トップレベル変更との優劣を示す:

> Add a `role: "system"` message with empty `content` and the new level in `output_config.effort`. The new level takes effect from the next `user` turn and holds until a later message changes it. Everything before that message is unchanged, so the cached prefix still matches.
>
> On Claude Fable 5.1, prefer this form over changing the top-level value between requests. A top-level change restarts the cache and also steers the model less reliably: its earlier replies were written at the previous level, and it tends to stay consistent with them.

*(訳)* `content` が空で `output_config.effort` に新しいレベルを持つ `role: "system"` メッセージを追加する。新しいレベルは次の `user` ターンから効き、後続のメッセージで変えるまで持続する。そのメッセージより前は何も変わらないため、キャッシュ済みプレフィックスは引き続き一致する。 / Claude Fable 5.1 では、リクエスト間でトップレベルの値を変えるよりこの形を優先すること。トップレベルの変更はキャッシュを最初からやり直すうえ、モデルの誘導も効きにくい: 以前の返答は前のレベルで書かれており、モデルはそれと一貫しようとするからだ。

移行ガイドのチェックリストも方針転換を明示している:

> Re-tune effort from the default (`high`), and consider changing it mid-conversation instead of holding one level for the whole session.

*(訳)* effort をデフォルト(`high`)から再調整し、セッション全体で 1 つのレベルを維持するのではなく、会話の途中で変更することを検討せよ。

なお Fable 5(無印)では per-message effort は使えない:

> Models without per-message effort, including Claude Fable 5, return a 400 error (…)

*(訳)* Claude Fable 5 を含む per-message effort 非対応モデルは 400 エラーを返す(…)

**反映:** パイプラインパターンを世代別に分けた。5.1 は会話内で per-message effort により「ステップ単位で上げ下げ」、Fable 5 は従来どおり「別リクエストで escalate(キャッシュ論拠)」。8/29 追補1 の記述は Fable 5 向けとして残る。

### 追補8. low effort での検索抑制 → `effort-calibrator`

**公式根拠(「Search triggering at low effort」):**

> At `low` effort, Claude Fable 5.1 is less likely than Claude Fable 5 to call a search or retrieval tool, and more likely to answer from memory. In some cases the simplest fix is to raise effort for the affected turns rather than the whole conversation. (…) In other cases, a prompt nudge toward verification helps. In the system prompt, say that recognizing a name isn't the same as knowing its current state, and that such names should be searched as the user wrote them (…)

*(訳)* `low` effort では、Claude Fable 5.1 は Fable 5 より検索・取得ツールを呼びにくく、記憶から答えやすい。最も単純な対処は、会話全体ではなく該当ターンだけ effort を上げること。(…)別のケースでは、検証を促すプロンプトのナッジが効く。システムプロンプトで、名前を認識していることはその現在の状態を知っていることと同じではない、そうした名前はユーザーが書いたとおりに検索すべきだ、と述べる(…)

**反映:** Adjustment signals の「Raise effort when」に 1 項目追加(該当ターンの effort を上げるか、「名前を認識している ≠ 現在の状態を知っている」旨のプロンプト行を足す)。

### 追補9. xhigh / max での長い成果物 → `effort-calibrator`

**公式根拠(「Leave room for long outputs at xhigh and max effort」):**

> At `xhigh` and especially `max` effort, Claude Fable 5.1 can think for longer before it starts writing its reply. When a single request asks for a long deliverable, such as a full rewrite of a long document, it may draft much of that deliverable in its thinking and then write it out again as the reply, which means a longer wait and more output tokens. The simplest approach is to run requests like these at `high` (…). If you do run them at `xhigh` or `max`:
>
> * Set `max_tokens` to leave room for the thinking and the reply, not just the reply length you expect.
> * Append the following note to the end of the user message. It makes the thinking much shorter on prose and code requests.

*(訳)* `xhigh`、とりわけ `max` では、Claude Fable 5.1 は返答を書き始める前に長く考えることがある。長い文書の全面書き直しのような長い成果物を 1 リクエストで求めると、成果物の大部分を思考内で下書きしてから返答として再度書き出すことがあり、待ち時間と出力トークンが増える。最も単純な対処はこうしたリクエストを `high` で動かすこと(…)。`xhigh` や `max` で動かすなら: `max_tokens` は期待する返答の長さではなく、思考と返答の両方が入る余裕を持たせること。ユーザーメッセージ末尾に注記を付けること(散文・コードのリクエストで思考が大幅に短くなる)。

**反映:** Adjustment signals 末尾に「`xhigh`/`max` で長い成果物を作るときの注意」を 1 段落追加(まず `high`、上げるなら `max_tokens` に思考分の余裕、推論は判断に・出力は本文にという注記)。公式の注記文はそのまま貼らず趣旨のみ記載。

### 追補10. キャッシュ読み価格と早期 compaction の前提 → `effort-calibrator`

**公式根拠(What's new「Pricing」):**

> Cache reads (hits and refreshes) cost 0.025 times the base input price on these models, compared with 0.1 on other Claude models. Long agentic sessions that re-read a cached prefix pay a quarter of the Claude Fable 5 rate.

*(訳)* これらのモデルではキャッシュ読み(ヒットと更新)は base input 価格の 0.025 倍で、他の Claude モデルの 0.1 倍と対照的である。キャッシュ済みプレフィックスを読み直す長いエージェントセッションは、Claude Fable 5 の 4 分の 1 のレートで済む。

同「Keep the conversation history append-only」末尾:

> Because cache reads are now cheaper (…), compacting early to save cost may no longer be the right cost-intelligence tradeoff on Claude Fable 5.1, so experiment with later compaction points.

*(訳)* キャッシュ読みが安くなったため(…)、コスト削減のための早期 compaction は Claude Fable 5.1 ではもはや正しいコスト対知能のトレードオフでない可能性があるので、より遅い compaction ポイントを試すこと。

**反映:** パイプラインパターン末尾にコスト前提の変化を 1 文追記。デフォルト effort(API・Claude Code は `high`、claude.ai / Cowork は Medium)は発表記事および Models overview の表と一致することを確認し、表の記述は維持した。

### 追補11. テストの肥大化とスコープ外の修正 → `no-gold-plating`

**公式根拠(「Keep changes and tests to what the task asks for」):**

> When asked to implement an open-ended feature, Claude Fable 5.1 delivers what's asked for and sometimes more: it may fix nearby code, extend behavior the task didn't mention, or commit more test files than the change warrants. It responds well to explicit instructions about what to leave out. With the following instruction, unrequested additions and committed test code drop substantially with no measurable change in task success:
>
> If, while working or testing, you find a pre-existing bug, a performance concern, or behavior the task doesn't mention, don't fix, optimize or extend it in this change unless the requested behavior cannot work without it; report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption in your summary, and don't build for the other readings as well. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or this repository already keeps tests for this kind of change, sized like the neighboring test files — roughly one focused test per stated behavior — and don't turn scratch checks into additional permanent test files. This is about extras only: implement every behavior the task asks for, completely.

*(訳)* 自由度の高い機能実装を頼まれると、Claude Fable 5.1 は求められたものに加えて余計なことをすることがある: 近隣コードの修正、タスクで言及されていない挙動の拡張、変更に見合わない数のテストファイルのコミットなど。何を省くべきかの明示的な指示にはよく応じる。次の指示により、頼まれていない追加とコミットされたテストコードは大幅に減り、タスク成功率に測定可能な変化はなかった: / 作業中やテスト中に既存のバグ、性能上の懸念、タスクが言及していない挙動を見つけても、要求された挙動がそれなしでは成立しない場合を除き、この変更で修正・最適化・拡張しないこと。サマリでフォローアップとして報告すること。タスクが曖昧なら、文言と周辺コードが最も直接的に支持する読みで実装し、その仮定をサマリに書き、他の読みのぶんまで作らないこと。検証は好きな方法でよく、使い捨てスクリプトや簡易チェックは残さなくてよい。テストをコミットするのは、タスクが求める場合かこのリポジトリが同種の変更に対して既にテストを持っている場合のみとし、隣接するテストファイルと同程度の規模——明示された挙動 1 つにつき集中したテスト 1 つ程度——にし、使い捨てチェックを恒久的なテストファイルに増やさないこと。これは余分なものだけの話であり、タスクが求める挙動はすべて完全に実装すること。

**反映:** Rules に 3 項目(既存バグは触らず follow-up 報告、曖昧なら最も直接的な読みで実装し仮定を明記、テストの慣習・規模・使い捨てチェックの扱い)と「余分なものだけの話」の但し書きを追加。Self-check に「コミットするテストファイルの数は変更の規模に見合っているか」を追加。README の "What it fixes" 欄に公式の実測(余計な追加が大幅減、成功率不変)を一言足した。

### 追補12. 旧モデル向けアンチフォーマット指示と進捗抑制行 → `skill-refactorer`

**公式根拠(「Formatting in chat」):**

> Earlier models overused bullets and bold in chat, and many prompts carry anti-formatting rules written to hold that down. Claude Fable 5.1 leans the other way: it uses bold less and is less likely to reach for headers, lists, or quotation marks. If your prompt contains anti-formatting language, remove it or replace it with a rule that says when specific formatting is appropriate (…)

*(訳)* 旧モデルはチャットで箇条書きと太字を使いすぎ、それを抑えるためのアンチフォーマット規則を抱えたプロンプトが多い。Claude Fable 5.1 は逆方向に寄っており、太字を使うことが減り、見出し・リスト・引用符にも手を伸ばしにくい。プロンプトにアンチフォーマットの文言があるなら、削除するか、どのフォーマットがいつ適切かを述べる規則に置き換えること(…)

(「Ask for user-facing progress updates」):

> Second, audit your prompt for instructions that suppress narration. Some earlier models were eager to give updates while working, which led to system prompt lines such as "hold all findings for the final response." Remove lines like that before adding anything.

*(訳)* 次に、ナレーションを抑制する指示がプロンプトにないか監査すること。旧モデルの一部は作業中に更新を出したがり、その結果「所見はすべて最終応答まで保持せよ」のようなシステムプロンプト行が生まれた。何かを足す前に、そうした行を削除すること。

**反映:** Red flags に 2 点追加。description のトリガーに「Fable 5 向けに書かれ、いま 5.1 で動いているスキル」を追加。既存の「内部推論を応答に書き出させる指示」の赤旗は、5.1 が Fable 5 と同じ `stop_details` カテゴリを持つため維持:

> Claude Fable 5.1 includes safety classifiers covering the same `stop_details` categories as Claude Fable 5, and everything in Refusals and fallback applies.

*(訳)* Claude Fable 5.1 は Claude Fable 5 と同じ `stop_details` カテゴリを対象とする安全分類器を含み、「Refusals and fallback」の内容がすべて適用される。

### 追補13. 「Delivering work」ブロック → `autonomous-continuation`

**公式根拠(「Finish the whole task」。5.1 ガイドでは 2 段構成となり、2 段目が新規):**

> Two system prompt additions together mitigate this. Apply both. If you need to limit prompt length, use only the first, which keeps most of the effect.

*(訳)* 2 つのシステムプロンプト追加を組み合わせるとこれを緩和できる。両方を適用すること。プロンプト長を抑える必要があるなら 1 つ目だけを使えばよく、効果の大半は保たれる。

1 段目は 8/29 時点の Autonomy contract と同趣旨(スキル9の項の引用に "Do not stop because the context or session is long." が加わっている)。2 段目:

> The user's request — or the plan they approved — sets the scope, and the scope is the deliverable: don't quietly narrow, widen, or swap it. (…) If a question comes up partway, first do everything that doesn't depend on the answer; then state the assumption you made, or — when going ahead on a wrong guess would be unsafe or would make the work useless — put the question at the end of a turn that also delivers that progress. If one part turns out to be blocked, complete every other part in full and say exactly what you left out and why — the whole task is the deliverable, and scaling it down is the user's call, not yours. A step you have decided on is something to run, not to announce: describing the next step and ending the turn leaves it undone until the user replies.

*(訳)* ユーザーの依頼——あるいは承認された計画——がスコープを定め、スコープがそのまま成果物である: 黙って狭めたり広げたり差し替えたりしないこと。(…)途中で疑問が出たら、まず答えに依存しない部分をすべて進め、置いた仮定を述べること。誤った推測で進むと危険か作業が無意味になる場合は、その進捗も届けるターンの末尾に質問を置くこと。一部がブロックされたら、残りをすべて完全に仕上げ、何を残したか・なぜかを正確に述べること——タスク全体が成果物であり、縮小するかはあなたではなくユーザーの判断である。決めたステップは実行するものであって告知するものではない: 次のステップを説明してターンを終えれば、ユーザーが返答するまで未完のままだ。

**反映:** 新セクション「Delivering the whole task(pair with the contract)」を追加し、Autonomy contract 末尾に「コンテキストやセッションが長いことを理由に止まらない」を 1 文追加。Context-budget composure は独立セクションとして維持したが、公式の根拠は 5.1 ガイドでは本体ブロックに統合された(独立セクション「Rare cases of context-budget concern」は Fable 5 ガイドのみ)。2 段目末尾の「変更は依頼が必要とする範囲に留める」段落は `no-gold-plating` / `scope-guard` の既存ルールと重複するため、ここには取り込んでいない。

### 追補14. 履歴の append-only とターン限定システムメッセージ → `autonomous-continuation`

**公式根拠(「Keep the conversation history append-only」):**

> Append each assistant turn to the history exactly as the API returned it, thinking blocks included, and don't edit earlier turns between requests. For new accounts created on or after August 31, 2026, Claude Fable 5.1's thinking blocks are valid only in the exact conversation that produced them: a request that replays a thinking block after its prefix (the system prompt, the tool list, or any earlier message) has changed returns a 400, or drops the affected blocks if you set `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (…)
>
> The history edits that trip the check are the same ones that restart the prompt cache: injecting and removing per-turn reminders, summarizing older turns in place, or changing the system prompt mid-session. Send per-turn reminders as turn-scoped system messages (…)

*(訳)* 各 assistant ターンは API が返したとおり、thinking ブロックも含めて履歴に追加し、リクエスト間で過去ターンを編集しないこと。2026 年 8 月 31 日以降に作成された新規アカウントでは、Claude Fable 5.1 の thinking ブロックはそれを生成した会話の中でのみ有効である: プレフィックス(システムプロンプト、ツール一覧、それ以前のメッセージ)が変わった後に thinking ブロックを再送するリクエストは 400 を返すか、`thinking.block_binding.prefix_mismatch_behavior: "drop_block"` を設定していれば該当ブロックが落とされる(…) / このチェックに引っかかる履歴編集は、プロンプトキャッシュを最初からやり直させる編集と同じである: ターンごとのリマインダーの注入と削除、古いターンのその場での要約、セッション途中のシステムプロンプト変更。ターンごとのリマインダーはターン限定システムメッセージとして送ること(…)

What's new「Editing earlier turns invalidates thinking blocks」は既存アカウントの扱いも明示する:

> The check is enforced for new accounts created on or after August 31, 2026. For accounts created earlier, the API records the mismatch but acts on it only when the request sets `thinking.block_binding.prefix_mismatch_behavior`.

*(訳)* このチェックは 2026 年 8 月 31 日以降に作成された新規アカウントで強制される。それ以前に作成されたアカウントでは、API は不一致を記録するが、リクエストが `thinking.block_binding.prefix_mismatch_behavior` を設定したときにのみ作用する。

**反映:** ハーネス作者向けセクション「Per-turn reminders」を追加(注入して削除する方式を禁止、ターン限定システムメッセージ `clear_at: "next_user_message"` とベータヘッダ `mid-conversation-system-clear-at-2026-08-21` を案内、履歴は append-only)。

### 追補15. 進捗更新の受信設定 → `grounded-progress`

**公式根拠(「Ask for user-facing progress updates」):**

> Claude Fable 5.1's default behavior is to write fewer user-facing updates during long tool-calling turns than Claude Fable 5 does. This becomes more pronounced at higher effort and in longer tool chains. (…)
>
> First, check that your client receives progress updates at all. The model's short notes between tool calls (…) come back as progress-update `thinking` blocks, and those blocks are empty under the default `thinking.display` of `"omitted"`. Set `display: "updates"` (beta, `thinking-display-updates-2026-08-18` header) and render each non-empty `thinking` block as a status line (…)
>
> If your product collapses or hides tool output, tell the model. Otherwise it may run commands to "show" the user output that your UI never displays.

*(訳)* Claude Fable 5.1 のデフォルト挙動は、長いツール呼び出しターン中のユーザー向け更新が Claude Fable 5 より少ない。これは高い effort と長いツールチェーンでより顕著になる。(…) / まず、クライアントが進捗更新をそもそも受け取っているか確認すること。ツール呼び出し間のモデルの短いメモは(…)進捗更新の `thinking` ブロックとして返り、デフォルトの `thinking.display` が `"omitted"` だとそれらは空になる。`display: "updates"`(ベータ、`thinking-display-updates-2026-08-18` ヘッダ)を設定し、空でない各 `thinking` ブロックをステータス行として描画すること(…) / 製品がツール出力を畳む・隠すなら、モデルにそう伝えること。さもないと、UI が決して表示しない出力をユーザーに「見せる」ためにコマンドを実行することがある。

**反映:** ハーネス作者向けセクション「Receiving the updates」を追加。証拠ルール本体は変更なし(根拠は Fable 5 ガイド「Ground progress claims during long runs」のみで、5.1 ガイドに対応セクションはない)。

### 追補16. 冒頭宣言・文の密度・引用の明示 → `regrounding-summary`

**公式根拠(「Ask for user-facing progress updates」の推奨パターン):**

> (…) add a short system prompt line that says when you want user-facing text from the model and what each update should contain: (…) say in a line what you're about to do; brief updates while you work (…) Close with a short recap that stands on its own — what you found, what you did, and what's next — so a reader who only sees the last message has the full picture.

*(訳)* (…)いつユーザー向けテキストが欲しいか、各更新に何を含めるかを述べる短いシステムプロンプト行を足す: (…)これから何をするかを 1 行で言う。作業中は短い更新(…)。締めは単独で読めるレキャップ——何を見つけ、何をし、次は何か——にし、最後のメッセージしか見ない読者にも全体像が伝わるようにする。

(「Writing density」):

> In some cases, though, its prose is denser than Claude Fable 5's: sentences run longer and there are fewer paragraph breaks. An instruction that defines the anti-pattern, mannered prose, helps. (…) The fix is to say what you mean. When a literal phrase is available, use it.

*(訳)* ただし場合によっては、その散文は Claude Fable 5 より密度が高い: 文が長くなり、段落の区切りが減る。アンチパターン(気取った散文)を定義する指示が効く。(…)対処は言いたいことをそのまま言うこと。文字どおりの言い回しがあるならそれを使うこと。

(「Quoting retrieved sources」):

> When summarizing documents, Claude Fable 5.1 is more likely than Claude Fable 5 to reproduce passages of the source text without marking them as quotations.

*(訳)* 文書を要約するとき、Claude Fable 5.1 は Claude Fable 5 より、出典の一節を引用と明示せずに再現しやすい。

**反映:** 「Before the final message」節に冒頭 1 行宣言と作業中の短い更新を追加。Rules に「短い文・短い段落、文字どおりの表現があればそれを使う」と「出典の文言を使うときは引用と明示」を各 1 行追加。公式の mannered prose 定義文と example ブロックは貼らず趣旨のみ。

### 追補17. lead の非ブロッキングとツール呼び出しのバッチ化 → `subagent-orchestration`

**公式根拠(「Let the lead agent keep working while subagents run」):**

> If your coding agent lets Claude Fable 5.1 delegate work to subagents, don't force the lead agent to stop and wait for each one. On coding tasks, letting the lead continue while subagents run lowers average time to completion at similar quality, token usage, and cost. To set this up:
>
> * Have the tool that starts a subagent return immediately.
> * Pass each subagent's result back to the lead in a later `user` message once it's ready.
> * Give the lead a separate tool it can call when it wants to wait for a result.
>
> The model still often chooses to wait. The time savings come from the runs where it carries on with other work.

*(訳)* コーディングエージェントが Claude Fable 5.1 にサブエージェントへの委譲を許すなら、lead エージェントに各サブエージェントの完了を待たせないこと。コーディングタスクでは、サブエージェント実行中も lead を続行させると、品質・トークン使用量・コストが同程度のまま平均完了時間が下がる。設定は: サブエージェントを起動するツールは即座に return する。各サブエージェントの結果は準備でき次第、後続の `user` メッセージで lead に返す。lead が結果を待ちたいときに呼べる別のツールを用意する。 / それでもモデルは待つことを選ぶことが多い。時間短縮は、他の作業を続行した実行から生まれる。

(「Batch independent tool calls in agent loops」):

> The exception is coding and computer-use loops where the next independent calls are implied by the task rather than explicitly requested (custom coding agents, bash-and-editor harnesses, computer use): there it may issue them one per turn instead. This doesn't affect answer quality, but each extra turn costs tokens, a round trip, and wall-clock time. A one-sentence nudge at the end of the current request addresses it (…) Each time you send tool results back, append it after that user message as a turn-scoped system message (…) Append a fresh copy each turn and leave the earlier copies where they are, byte-for-byte.

*(訳)* 例外は、次の独立した呼び出しが明示的に依頼されるのではなくタスクから暗黙に導かれるコーディング・computer use のループ(カスタムコーディングエージェント、bash+エディタのハーネス、computer use)で、そこではターンごとに 1 つずつ発行することがある。回答品質には影響しないが、余分なターンごとにトークン・往復・実時間がかかる。現在のリクエスト末尾の 1 文のナッジで対処できる(…)ツール結果を返すたびに、その user メッセージの後にターン限定システムメッセージとして追加する(…)毎ターン新しいコピーを追加し、以前のコピーはバイト単位でそのまま残すこと。

**反映:** Coordination にハーネス形(即 return、後続 user メッセージで結果返却、待機用の別ツール)とバッチ化のナッジ(ターン限定システムメッセージで毎ターン送り、過去ターンは編集しない)を各 1 項目追加。description にもトリガー(1 ターン 1 呼び出しのループ)を追加。

### 変更しなかったスキルと、その根拠の所在

- `act-when-ready`、`scope-guard`、`markdown-memory` は 5.1 ガイドと矛盾しない。`scope-guard` の「問題の記述/質問/思考の垂れ流しなら成果物は評価」は 5.1 ガイド「Finish the whole task」の 1 段目に例外節としてそのまま含まれている。
- ただし 5.1 ガイドには過剰プランニング・memory・進捗捏造に相当するセクション自体が存在しない。`act-when-ready`、`markdown-memory`、`grounded-progress` の証拠ルールの根拠は引き続き Fable 5 ガイドのみである(サマリ表の「5.1 ガイドでの対応セクション」列を参照)。
- `act-when-ready` には、5.1 の「更新が少なめ」という傾向と「計画テキストは数行まで」が合わさって無言化しやすい点を考慮し、「これはプランニングの抑制であって進捗更新の抑制ではない」旨の 1 文を任意追加した。

### README・メタ情報

- 全スキルの frontmatter `description` を「Claude Fable 5 / 5.1」併記に変更(description はトリガー判定に使われるため)。
- README(英・日)の Status に 5.1 のリリース日、公式の「無変更で動く」声明、Fable 5 の legacy 扱いを追記。Design principles 4 に 5.1 ガイドへのリンクを追加。Disclaimer に API 側の破壊的変更(`tool_choice` の `any`/`tool` が 400、旧モデルは 5.1 の thinking ブロックを読めない、過去ターン編集で thinking ブロック無効化)がスキル範囲外である旨を追記(根拠: What's new「Breaking changes」)。
- `plugin.json` / `marketplace.json` を 1.2.0 → 1.3.0 に更新。

## 参照元

1. [Prompting Claude Fable 5 — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — スキル 1, 2, 4〜10 の主要根拠(5.1 にも引き続き適用)
2. [Effort — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/effort) — スキル 3(effort レベル定義、Fable 5 向け推奨、`output_config.effort` の API 形状)
3. [Introducing Claude Fable 5 and Claude Mythos 5 — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5) — リリース日(2026-06-09)、adaptive thinking、サポート機能一覧
4. [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — Claude Code における各モデルのデフォルト effort(Fable 5 = `high`)、`/effort` コマンド、ultracode
5. [Migration guide — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — Opus 4.8 → Fable 5 移行時の effort 再評価(`xhigh` → `high`)

6. [Prompting Claude Fable 5.1 — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) — 追補6〜17 の主要根拠
7. [What's new in Claude Fable 5.1 — Claude Platform Docs](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1) — per-message effort、ターン限定システムメッセージ、進捗更新の `display: "updates"`、価格、破壊的変更、`stop_details`
8. [Migrating to Claude Fable 5.1 and Claude Mythos 5.1 — Claude Platform Docs](https://platform.claude.com/docs/en/models/fable-5-1/migration-guide) — effort の再スイープと会話途中の変更、履歴編集チェック
9. [Models overview — Claude Platform Docs](https://platform.claude.com/docs/en/models/overview) — Fable 5 の legacy 表記、5.1 のデフォルト effort(`high`)、Retirement 日
10. [Claude Fable 5.1 and Claude Mythos 5.1 — Anthropic](https://www.anthropic.com/claude-fable-and-mythos-5-1) — 発表記事。Claude Code は High、claude.ai / Cowork は Medium がデフォルト。「Low / Medium で Fable 5 と同等以上の結果をはるかに低コストで」

(参照元 1・2 は 2026-08-29、6〜10 は 2026-09-02 に取得した内容に基づく。3〜5 は 2026-07-03 時点の取得内容。参照元 1 の Fable 5 ガイドは 5.1 にも引き続き適用される前提で扱っている。)
