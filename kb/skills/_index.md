---
title: Agent Skills エコシステム
last_patrol: "2026-09-08"
standard: agentskills.io (46 platforms confirmed)
tracked_skills: 21  # recommended.md の行数（Tier A 11 / Tier B 10）
patrol_schedule: weekly (月曜)
---

# Agent Skills エコシステム管理

Claude Code / Codex CLI で活用できる公式・コミュニティスキルの推薦管理。段階的開示で必要時に詳細参照。

## 参照先一覧

詳細は `sources.md` 参照。

| サイト | 種別 | 役割 |
|--------|------|------|
| [claude.com/plugins](https://claude.com/plugins) | Anthropic公式 | プラグイン/スキルディレクトリ（Frontend Design 1.134M installs が首位、Superpowers 1.009M で 2 位。Anthropic 検証バッジ継続） |
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic公式 | 公式スキル実装例（**175.0K stars**、**19 スキルで増減なし**。最終 push が 2026-08-21 → **2026-09-03** に更新。2026-09-01 に **claude-api スキルへ Claude Fable 5.1 / Mythos 5.1、Managed Agents、cost-optimize を反映**（#1704）、2026-09-03 に **frontend-design スキルを「汎用的なデザイン既定値に流れない」方向へ改訂**（#1713）） |
| [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) | OpenAI公式 | Codex Skills仕様（learn.chatgpt.com/docs/build-skills へ 308 恒久リダイレクト） |
| [learn.chatgpt.com/docs/build-plugins](https://learn.chatgpt.com/docs/build-plugins) | OpenAI公式 | **skill-only プラグイン作成の一次情報**（`.codex-plugin/plugin.json` + `skills/`、`@plugin-creator`）。GitHub カタログ停止後の実質的な正典 |
| [openai/skills](https://github.com/openai/skills) | OpenAI公式 | **2026-06-22 deprecated**（25.3K stars）。後継として案内する openai/plugins も archive 済みで**二重の行き止まり** |
| [openai/plugins](https://github.com/openai/plugins) | OpenAI公式 | **2026-08-16 に archive（read-only）**（5.3K stars、最終 push 2026-07-14）。OpenAI 側に維持された公開サンプルカタログは無くなった |
| [skills.sh](https://skills.sh/) | コミュニティ | スキルディレクトリ兼リーダーボード（find-skills **3.3M** installs でトップ継続。frontend-design 838.9K → **863.4K** で 5 位、agent-browser 760.2K → **804.0K** で 7 位。top10 に **vercel-react-best-practices**（695.3K、9 位）と **lark-doc**（open.feishu.cn、666.3K、10 位）が新規ランクイン。ベンダー公式クラスタ: open.feishu.cn 15.3M 据え置き、microsoft/azure-skills 7.8M → **7.4M**、**larksuite/cli 6.0M を新規確認**、mattpocock/skills 合計 3.0M） |
| [agentskills.io](https://agentskills.io/) | オープン標準 | Agent Skills仕様（SKILL.mdフォーマット）。仕様変更なし、Client Showcase 掲載プラットフォームは **46 で据え置き**（2026-09-08 に実カウントで再確認） |
| [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) | OpenAI公式 | **新規追跡（2026-08-25）**。Claude Code から Codex を呼ぶ公式プラグイン（**32.9K stars**、archive されておらず現役）。2026-08-24 の公式 changelog が `codex mcp-server` 非推奨の移行先として明示的に案内 |

## 推薦スキル概要

詳細は `recommended.md` 参照。

**Tier A（管理対象）**: skill-creator, systematic-debugging, test-driven-development, frontend-design, security-scan, pdf/pptx/xlsx/docx, dispatching-parallel-agents, using-git-worktrees, postgres-best-practices, find-skills, **codex-plugin-cc**
**Tier B（ウォッチリスト）**: docker-optimize, data-pipeline, Context7, landing-page-guide, deploy-checklist, brainstorming, agent-browser, discernment-nudge, **microsoft/azure-skills**, academy-guide

## 巡回時の注意（2026-08-18 追記）

OpenAI の公開スキル/プラグイン**サンプルカタログは GitHub 上で完全に停止**した（openai/skills は deprecated、その後継として案内されていた openai/plugins も 2026-08-16 に archive）。Codex 側のスキル・プラグイン作成手順を参照するときは GitHub リポジトリではなく `learn.chatgpt.com/docs/build-skills` / `build-plugins` と `@plugin-creator`（Codex では `$plugin-creator`）を一次情報とすること。仕様は `specs/codex/configuration.md` §6 に収載済み。

## 巡回時の注意（2026-08-25 追記）

Claude⇔Codex 連携の**公式の入口が変わった**。従来 harness-harness が Claude→Codex ブリッジ候補として見ていた `codex mcp-server` は 0.149.1 で非推奨になり、公式 changelog は移行先として (a) プロダクト組み込み向けの `codex app-server`、(b) Claude Code 利用者向けの `openai/codex-plugin-cc` の 2 つを案内している。**クロスレビュー運用は (b) を第一候補とする**（`/codex:review --background` → `/codex:status` → `/codex:result` で Claude 側をブロックしない）。詳細は `specs/codex/mcp.md` §5 参照。

## 巡回時の注意（2026-09-01 追記）

skills.sh のリーダーボードで**ベンダー公式スキルクラスタ**（open.feishu.cn / microsoft/azure-skills）が上位を大きく占めるようになった。個別スキルの installs では mattpocock 系が引き続き優勢だが、組織単位の合計では逆転している。推薦時は「単体スキルの人気」と「ベンダーがまとめて配布するクラスタ」を区別し、後者は**対象プロジェクトがそのベンダーを使っている場合のみ**提案する（多様性は善だが、ベンダーロックインを黙って持ち込まない）。

## 巡回時の注意（2026-09-08 追記）

anthropics/skills の **claude-api スキルが Anthropic API 側の「陳腐化しやすい前提」の一次情報源になっている**。2026-09-01 の #1704 で Fable 5.1 / **Mythos 5.1**（`claude-mythos-5-1`、Project Glasswing 参加組織のみ。Fable 5.1 と同一モデルで別提供枠）、Managed Agents の Vault 資格情報、cost-optimize が反映された。ハーネス生成時にモデルID・API パラメータを書く場合は、記憶ではなく `skills/claude-api/shared/models.md` を都度参照すること。特に **`thinking.budget_tokens` は Fable 5/5.1・Sonnet 5・Opus 5/4.8/4.7 で 400 エラー**（`{type: "adaptive"}` を使う）、**Files API / Skills は beta 卒業**（`client.files.*` / `client.skills.*`）という 2 点は、学習時点の知識で書くと壊れる。

なお **これは Anthropic API の話であって Claude Code CLI の仕様ではない**。`specs/claude/` は Claude Code CLI の仕様書として維持し、API 側の詳細は claude-api スキルへのポインタで留める（段階的開示）。
