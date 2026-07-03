# claude-fable-5-skills

**Claude Fable 5 のための実戦的 Agent Skills 10選** — 旧世代モデルの「お世話」前提ではなく、Claude Fable 5 が実際にどう振る舞うかを起点に設計した、最初のスキルコレクションです。

[English README →](README.md)

> Anthropic 自身の Fable 5 向けガイドは、旧モデル向けに書かれたスキルは Fable 5 には過剰に規範的で、出力品質をむしろ**劣化**させうると指摘しています。古いスキルは手順を細かく管理しようとしますが、Fable 5 に必要なのは「意図・境界・検証フック」です。ここにある全スキルは Fable 5 ファーストで書かれています。

**Claude Code / Claude Cowork / Cursor / Copilot / Gemini CLI** など、`SKILL.md`(YAMLフロントマター+Markdown本文、agentskills.io 形式)を読めるあらゆるハーネスで動作します。

## 収録スキル一覧

| # | スキル | 解決する問題 |
|---|--------|--------------|
| 1 | [`skill-refactorer`](skills/skill-refactorer/SKILL.md) | **目玉のメタスキル。** Fable 5 以前のスキル/プロンプトを監査し、旧モデルの能力不足を補うための足場(過剰指示)を削除、本物のガードレールだけを残す。 |
| 2 | [`act-when-ready`](skills/act-when-ready/SKILL.md) | 高effort時の過剰計画。確定済み事実の再導出、選ばない選択肢の列挙をやめさせる。 |
| 3 | [`effort-calibrator`](skills/effort-calibrator/SKILL.md) | ワークロード別の `effort` 設定指針。Fable 5 は低めの effort でも旧モデルの `xhigh` を上回ることが多い。 |
| 4 | [`no-gold-plating`](skills/no-gold-plating/SKILL.md) | 依頼より大きい差分。頼んでいないリファクタ、投機的な抽象化、起こり得ない状態へのエラー処理を抑止。 |
| 5 | [`grounded-progress`](skills/grounded-progress/SKILL.md) | 長時間実行の進捗報告をツール実行結果という証拠に紐付ける。「テスト通りました(実行してない)」を根絶。 |
| 6 | [`scope-guard`](skills/scope-guard/SKILL.md) | 「診断して」≠「直して」。未依頼アクションと、パターンマッチだけを根拠にした状態変更を防ぐ。 |
| 7 | [`subagent-orchestration`](skills/subagent-orchestration/SKILL.md) | 並列委譲・長寿命ワーカー・自己批判より強い「クリーンな文脈の検証サブエージェント」の型。 |
| 8 | [`markdown-memory`](skills/markdown-memory/SKILL.md) | Fable 5 が特に活用の上手いファイルベース教訓メモリと、腐らせないための運用規律。 |
| 9 | [`autonomous-continuation`](skills/autonomous-continuation/SKILL.md) | 「これからXを実行します」で止まる・実行中に許可を求める無人パイプラインの停止対策。コンテキスト残量への動揺対策も収録。 |
| 10 | [`regrounding-summary`](skills/regrounding-summary/SKILL.md) | 作業を一切見ていない読者に伝わる最終報告。矢印チェーンや独自略語を禁止し、結論から書かせる。 |

## インストール

**Claude Code(推奨)** — プラグインとしてインストール(更新も自動追従):

```
/plugin marketplace add kpab/claude-fable-5-skills
/plugin install fable5-skills@claude-fable-5-skills
```

**手動** — スキルフォルダをスキルディレクトリへコピー:

```bash
git clone https://github.com/kpab/claude-fable-5-skills
cp -r claude-fable-5-skills/skills/act-when-ready ~/.claude/skills/
```

プロジェクト単位なら、リポジトリ内の `.claude/skills/` 配下に必要なフォルダを置いてください。他のハーネスでは `skills/` ディレクトリをスキルローダーに読ませれば動きます(各スキルは `SKILL.md` 1枚で自己完結)。

## スキルの発動経路について

`skill-refactorer` / `effort-calibrator` / `subagent-orchestration` / `markdown-memory` の4つはタスク型で、該当するリクエストが来れば自動で発動します。

残り6つは「常時適用したい振る舞いルール」です。過剰計画の真っ最中のモデルが自発的に `act-when-ready` を呼ぶことは期待できないため、自動発動を当てにしないでください。確実な経路は2つ:

- **対話セッション** — セッション冒頭、または症状が出た時点で明示的に起動(例: `/act-when-ready`)。
- **無人パイプライン・常時適用** — スキル本文をシステムプロンプトや `CLAUDE.md` に転記。Anthropic 公式の Fable 5 ガイド自体がこの形でパターンを提供しています。`autonomous-continuation` はまさにこの用途向けに書かれています。

## 設計原則

1. **手順ではなく意図。** Fable 5 は指示に厳密に従うため、ステップバイステップのレシピではなく、達成すべき結果と境界を書く。
2. **構造的に短い。** 全スキルが1画面に収まる。挙動を変えない行は削除。
3. **精神論ではなく検証フック。** 正しさが重要な箇所には「気をつける」ではなく具体的なチェック(証拠ルール、ターン終了チェック、差分セルフチェック)を定義。
4. **本文はすべてオリジナル。** パターンは Anthropic 公開の [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) と独自検証に基づきますが、指示文は全てゼロから書き起こしています。

## ステータスと免責

Fable 5 のリリースは **2026年6月9日**。モデルの理解が進むにつれてスキルも更新されます。再現可能な before/after 付きの Issue・PR を歓迎します。

本プロジェクトは非公式のコミュニティプロジェクトであり、Anthropic とは無関係です。本番利用の前に最新のAPIパラメータを[公式ドキュメント](https://platform.claude.com/docs)で確認してください。

## ライセンス

MIT — [LICENSE](LICENSE) を参照。
