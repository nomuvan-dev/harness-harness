---
title: Agent Skills エコシステム
last_patrol: "2026-11-03"
standard: agentskills.io (46 platforms confirmed)
tracked_skills: 22  # recommended.md の行数（Tier A 12 / Tier B 10）
patrol_schedule: weekly (月曜)
---

# Agent Skills エコシステム管理

Claude Code / Codex CLI で活用できる公式・コミュニティスキルの推薦管理。段階的開示で必要時に詳細参照。

## 参照先一覧

詳細は `sources.md` 参照。

| サイト | 種別 | 役割 |
|--------|------|------|
| [claude.com/plugins](https://claude.com/plugins) | Anthropic公式 | プラグイン/スキルディレクトリ（**2026-11-03 確認: Claude Code 向け 528 プラグイン**（前回 549 から微減、掲載整理とみられる）。上位は Frontend Design・Superpowers・Code Review・Context7・Code Simplifier・Playwright の順で顔ぶれ変化なし。Anthropic 検証バッジ継続） |
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic公式 | 公式スキル実装例（**186.7K stars**（+3.0K）。最終 push 2026-10-28。**2026-11-03 巡回: スキル追加・削除なし（19 スキル据え置き）**。唯一の変更は 2026-10-28 の #2087 で **mcp-builder が MCP TypeScript SDK v2 対応に改訂**（v1/v2 選択ガイド、v2 移行 codemod、self-check スクリプト追加。v1 はバグ/セキュリティ修正のみ）— MCP サーバー構築系ハーネスに mcp-builder を組み込む際は改訂後版を取得すること） |
| [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) | OpenAI公式 | Codex Skills仕様（learn.chatgpt.com/docs/build-skills へ 308 恒久リダイレクト） |
| [learn.chatgpt.com/docs/build-plugins](https://learn.chatgpt.com/docs/build-plugins) | OpenAI公式 | **skill-only プラグイン作成の一次情報**（`.codex-plugin/plugin.json` + `skills/`、`@plugin-creator`）。GitHub カタログ停止後の実質的な正典 |
| [openai/skills](https://github.com/openai/skills) | OpenAI公式 | **deprecated 継続**（28.5K stars、2026-11-03 確認。デフォルトブランチへのコミットは増えておらず、カタログ変更なし）。README が 2026-09-08 に更新され、後継案内が「**復活した openai/plugins**」＋ build-plugins ガイドに差し替わった（`$skill-installer` の手順は維持）。「二重の行き止まり」は解消 |
| [openai/plugins](https://github.com/openai/plugins) | OpenAI公式 | **archive 解除（復活）を 2026-10-06 に確認**（7.3K stars、最終更新 2026-09-28、`isArchived: false`）。Codex プラグイン実例の公式キュレーションカタログとして再始動: `plugins/<name>/` + `.codex-plugin/plugin.json`、figma / notion / build-ios-apps / build-macos-apps / build-web-apps / expo / netlify / remotion / google-slides 等。マーケットプレイスは `.agents/plugins/marketplace.json`（API キーログイン用は `api_marketplace.json` で別） |
| [skills.sh](https://skills.sh/) | コミュニティ | スキルディレクトリ兼リーダーボード（**2026-11-03 確認**: find-skills **4.2M** installs でトップ継続。grill-me **1.5M** で 2 位維持。heygen-com/hyperframes クラスタ（hyperframes-cli 1.4M / hyperframes 1.3M / media-use 1.3M）が 3・4・7 位を占有、agent-browser **1.3M** で 5 位。mattpocock 系（grill-with-docs 1.3M / improve-codebase-architecture 1.2M / tdd 1.2M）が中位継続。reddit-automation **1.2M** で 9 位（継続性確認済み）。frontend-design はトップ 10 圏外のまま） |
| [agentskills.io](https://agentskills.io/) | オープン標準 | Agent Skills仕様（SKILL.mdフォーマット）。仕様変更なし、Client Showcase 掲載プラットフォームは **46 で据え置き**（2026-10-06 にエントリ実カウントで再確認） |
| [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) | OpenAI公式 | **新規追跡（2026-08-25）**。Claude Code から Codex を呼ぶ公式プラグイン（**34.6K stars**（2026-10-26 確認）、archive されておらず現役。最終 push 2026-07-08 のまま）。2026-08-24 の公式 changelog が `codex mcp-server` 非推奨の移行先として明示的に案内 |

## 推薦スキル概要

詳細は `recommended.md` 参照。

**Tier A（管理対象）**: skill-creator, systematic-debugging, test-driven-development, frontend-design, security-scan, pdf/pptx/xlsx/docx, dispatching-parallel-agents, using-git-worktrees, postgres-best-practices, find-skills, codex-plugin-cc, **agent-browser**（2026-11-03 昇格）
**Tier B（ウォッチリスト）**: docker-optimize, data-pipeline, Context7, landing-page-guide, deploy-checklist, brainstorming, discernment-nudge, microsoft/azure-skills, academy-guide, reddit-automation

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
