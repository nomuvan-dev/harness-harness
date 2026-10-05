---
title: Agent Skills エコシステム
last_patrol: "2026-10-06"
standard: agentskills.io (46 platforms confirmed)
tracked_skills: 22  # recommended.md の行数（Tier A 11 / Tier B 11）
patrol_schedule: weekly (月曜)
---

# Agent Skills エコシステム管理

Claude Code / Codex CLI で活用できる公式・コミュニティスキルの推薦管理。段階的開示で必要時に詳細参照。

## 参照先一覧

詳細は `sources.md` 参照。

| サイト | 種別 | 役割 |
|--------|------|------|
| [claude.com/plugins](https://claude.com/plugins) | Anthropic公式 | プラグイン/スキルディレクトリ（Frontend Design 1.134M installs が首位、Superpowers 1.009M で 2 位、Code Review 438.5K・Context7 417.8K が続く。**2026-10-06 確認: installs 数は前回から実質変化なし**。Anthropic 検証バッジ継続） |
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic公式 | 公式スキル実装例（**179.8K stars**（+2.4K）。**最終 push 2026-10-05 — 9/10 以来の更新再開**。差分内容はリポジトリルートのキャッシュからは特定できず、スキル数 19 の増減有無を含め**次回巡回で skills/ 配下を直接確認すること**。README は ./skills（4 カテゴリ）・./spec（Agent Skills 仕様）・./template の 3 構成を提示） |
| [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) | OpenAI公式 | Codex Skills仕様（learn.chatgpt.com/docs/build-skills へ 308 恒久リダイレクト） |
| [learn.chatgpt.com/docs/build-plugins](https://learn.chatgpt.com/docs/build-plugins) | OpenAI公式 | **skill-only プラグイン作成の一次情報**（`.codex-plugin/plugin.json` + `skills/`、`@plugin-creator`）。GitHub カタログ停止後の実質的な正典 |
| [openai/skills](https://github.com/openai/skills) | OpenAI公式 | **deprecated 継続**（27.9K stars）。ただし README が 2026-09-08 に更新され、後継案内が「**復活した openai/plugins**」＋ build-plugins ガイドに差し替わった（`$skill-installer` の手順は維持）。「二重の行き止まり」は解消 |
| [openai/plugins](https://github.com/openai/plugins) | OpenAI公式 | **archive 解除（復活）を 2026-10-06 に確認**（7.3K stars、最終更新 2026-09-28、`isArchived: false`）。Codex プラグイン実例の公式キュレーションカタログとして再始動: `plugins/<name>/` + `.codex-plugin/plugin.json`、figma / notion / build-ios-apps / build-macos-apps / build-web-apps / expo / netlify / remotion / google-slides 等。マーケットプレイスは `.agents/plugins/marketplace.json`（API キーログイン用は `api_marketplace.json` で別） |
| [skills.sh](https://skills.sh/) | コミュニティ | スキルディレクトリ兼リーダーボード（find-skills **3.7M** installs でトップ継続。grill-me **1.3M** で 2 位維持。**agent-browser 1.0M で 5 位に上昇**（前回 7 位）、tdd 1.0M（6 位）、frontend-design **956.4K**（7 位、前回 6 位から 1 つ後退）。**新顔: reddit-automation**（flowkit-labs/skills、**794.9K** で 10 位。週間 installs 114.3K はリーダーボード最速）。heygen-com/hyperframes クラスタが**表示分 20 スキル計 9.4M に急拡大**（hyperframes-cli 807.3K で 9 位）。**genmedia-labs/skills はリーダーボードから消滅**し、video-edit は prime-skills/runcomfy-agent-skills 名義（431.7K、クラスタ表示分 13 スキル計 5.0M）で掲載。vercel-react-best-practices 772.2K（12 位）、lark-doc 745.7K（15 位）。ベンダー公式クラスタ上位占有は継続: mattpocock/skills 表示分 19.9M / azure-skills 7.6M / open.feishu.cn 4.0M） |
| [agentskills.io](https://agentskills.io/) | オープン標準 | Agent Skills仕様（SKILL.mdフォーマット）。仕様変更なし、Client Showcase 掲載プラットフォームは **46 で据え置き**（2026-10-06 にエントリ実カウントで再確認） |
| [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) | OpenAI公式 | **新規追跡（2026-08-25）**。Claude Code から Codex を呼ぶ公式プラグイン（**33.4K stars**、archive されておらず現役。最終 push 2026-07-08 のまま）。2026-08-24 の公式 changelog が `codex mcp-server` 非推奨の移行先として明示的に案内 |

## 推薦スキル概要

詳細は `recommended.md` 参照。

**Tier A（管理対象）**: skill-creator, systematic-debugging, test-driven-development, frontend-design, security-scan, pdf/pptx/xlsx/docx, dispatching-parallel-agents, using-git-worktrees, postgres-best-practices, find-skills, **codex-plugin-cc**
**Tier B（ウォッチリスト）**: docker-optimize, data-pipeline, Context7, landing-page-guide, deploy-checklist, brainstorming, agent-browser, discernment-nudge, **microsoft/azure-skills**, academy-guide, **reddit-automation**（2026-10-06 追加）

## 巡回時の注意（2026-08-18 追記）※2026-10-06 追記により一部失効

OpenAI の公開スキル/プラグイン**サンプルカタログは GitHub 上で完全に停止**した（openai/skills は deprecated、その後継として案内されていた openai/plugins も 2026-08-16 に archive）。Codex 側のスキル・プラグイン作成手順を参照するときは GitHub リポジトリではなく `learn.chatgpt.com/docs/build-skills` / `build-plugins` と `@plugin-creator`（Codex では `$plugin-creator`）を一次情報とすること。仕様は `specs/codex/configuration.md` §6 に収載済み。

## 巡回時の注意（2026-08-25 追記）

Claude⇔Codex 連携の**公式の入口が変わった**。従来 harness-harness が Claude→Codex ブリッジ候補として見ていた `codex mcp-server` は 0.149.1 で非推奨になり、公式 changelog は移行先として (a) プロダクト組み込み向けの `codex app-server`、(b) Claude Code 利用者向けの `openai/codex-plugin-cc` の 2 つを案内している。**クロスレビュー運用は (b) を第一候補とする**（`/codex:review --background` → `/codex:status` → `/codex:result` で Claude 側をブロックしない）。詳細は `specs/codex/mcp.md` §5 参照。

## 巡回時の注意（2026-09-01 追記）

skills.sh のリーダーボードで**ベンダー公式スキルクラスタ**（open.feishu.cn / microsoft/azure-skills）が上位を大きく占めるようになった。個別スキルの installs では mattpocock 系が引き続き優勢だが、組織単位の合計では逆転している。推薦時は「単体スキルの人気」と「ベンダーがまとめて配布するクラスタ」を区別し、後者は**対象プロジェクトがそのベンダーを使っている場合のみ**提案する（多様性は善だが、ベンダーロックインを黙って持ち込まない）。

## 巡回時の注意（2026-09-08 追記）

anthropics/skills の **claude-api スキルが Anthropic API 側の「陳腐化しやすい前提」の一次情報源になっている**。2026-09-01 の #1704 で Fable 5.1 / **Mythos 5.1**（`claude-mythos-5-1`、Project Glasswing 参加組織のみ。Fable 5.1 と同一モデルで別提供枠）、Managed Agents の Vault 資格情報、cost-optimize が反映された。ハーネス生成時にモデルID・API パラメータを書く場合は、記憶ではなく `skills/claude-api/shared/models.md` を都度参照すること。特に **`thinking.budget_tokens` は Fable 5/5.1・Sonnet 5・Opus 5/4.8/4.7 で 400 エラー**（`{type: "adaptive"}` を使う）、**Files API / Skills は beta 卒業**（`client.files.*` / `client.skills.*`）という 2 点は、学習時点の知識で書くと壊れる。

なお **これは Anthropic API の話であって Claude Code CLI の仕様ではない**。`specs/claude/` は Claude Code CLI の仕様書として維持し、API 側の詳細は claude-api スキルへのポインタで留める（段階的開示）。

## 巡回時の注意（2026-10-06 追記）

**openai/plugins が archive 解除されて復活した**（最終更新 2026-09-28、`isArchived: false` を確認）。2026-08-18 追記の「OpenAI の GitHub サンプルカタログは完全停止・二重の行き止まり」は**この時点で失効**。現状の整理:

- **openai/plugins = Codex プラグイン実例の現役公式カタログ**（figma / notion / build-ios-apps / expo 等 + `.agents/plugins/marketplace.json`）。openai/skills（deprecated のまま）の README も 2026-09-08 更新でここを後継として案内
- 作成手順の一次情報は引き続き learn.chatgpt.com/docs/build-skills / build-plugins と `@plugin-creator`。**実例参照は openai/plugins を使ってよい**（specs/codex/configuration.md §6 の注意書きは 2026-10-06 巡回で反映済み）
- skills.sh では**動画生成ベンダークラスタの新陳代謝が激しい**（genmedia-labs 消滅 → runcomfy 名義に、heygen が 20 スキル 9.4M に急拡大）。クラスタ系は installs の絶対値より「継続性」を見て推薦判断すること
