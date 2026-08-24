---
title: Agent Skills エコシステム
last_patrol: "2026-08-25"
standard: agentskills.io (46 platforms confirmed)
tracked_skills: 20  # recommended.md の行数（Tier A 11 / Tier B 9）
patrol_schedule: weekly (月曜)
---

# Agent Skills エコシステム管理

Claude Code / Codex CLI で活用できる公式・コミュニティスキルの推薦管理。段階的開示で必要時に詳細参照。

## 参照先一覧

詳細は `sources.md` 参照。

| サイト | 種別 | 役割 |
|--------|------|------|
| [claude.com/plugins](https://claude.com/plugins) | Anthropic公式 | プラグイン/スキルディレクトリ（Frontend Design 1.134M installs が首位、Superpowers 1.009M で 2 位。Anthropic 検証バッジ継続） |
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic公式 | 公式スキル実装例（**171.3K stars**、**19 スキル**で増減なし。2026-08-18 に `claude-academy-guide` → **`academy-guide`** へリネーム＋description 短縮（#1605）、2026-08-21 に claude-api スキルへ **Python SDK 0.x → 1.x アップグレードガイド**を追加（#1623）） |
| [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) | OpenAI公式 | Codex Skills仕様（learn.chatgpt.com/docs/build-skills へ 308 恒久リダイレクト） |
| [learn.chatgpt.com/docs/build-plugins](https://learn.chatgpt.com/docs/build-plugins) | OpenAI公式 | **skill-only プラグイン作成の一次情報**（`.codex-plugin/plugin.json` + `skills/`、`@plugin-creator`）。GitHub カタログ停止後の実質的な正典 |
| [openai/skills](https://github.com/openai/skills) | OpenAI公式 | **2026-06-22 deprecated**（25.0K stars）。後継として案内する openai/plugins も archive 済みで**二重の行き止まり** |
| [openai/plugins](https://github.com/openai/plugins) | OpenAI公式 | **2026-08-16 に archive（read-only）**（5.1K stars、最終 push 2026-07-14）。OpenAI 側に維持された公開サンプルカタログは無くなった |
| [skills.sh](https://skills.sh/) | コミュニティ | スキルディレクトリ兼リーダーボード（find-skills **3.1M** installs でトップ継続。mattpocock 系 6 本の top10 占有も継続で顔ぶれ変動なし。agent-browser 689.0K → **721.0K**） |
| [agentskills.io](https://agentskills.io/) | オープン標準 | Agent Skills仕様（SKILL.mdフォーマット）。仕様変更なし、採用プラットフォームは **46 で据え置き** |
| [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) | OpenAI公式 | **新規追跡（2026-08-25）**。Claude Code から Codex を呼ぶ公式プラグイン（32.2K stars）。2026-08-24 の公式 changelog が `codex mcp-server` 非推奨の移行先として明示的に案内 |

## 推薦スキル概要

詳細は `recommended.md` 参照。

**Tier A（管理対象）**: skill-creator, systematic-debugging, test-driven-development, frontend-design, security-scan, pdf/pptx/xlsx/docx, dispatching-parallel-agents, using-git-worktrees, postgres-best-practices, find-skills, **codex-plugin-cc**
**Tier B（ウォッチリスト）**: docker-optimize, data-pipeline, Context7, landing-page-guide, deploy-checklist, brainstorming, agent-browser, discernment-nudge, academy-guide

## 巡回時の注意（2026-08-18 追記）

OpenAI の公開スキル/プラグイン**サンプルカタログは GitHub 上で完全に停止**した（openai/skills は deprecated、その後継として案内されていた openai/plugins も 2026-08-16 に archive）。Codex 側のスキル・プラグイン作成手順を参照するときは GitHub リポジトリではなく `learn.chatgpt.com/docs/build-skills` / `build-plugins` と `@plugin-creator`（Codex では `$plugin-creator`）を一次情報とすること。仕様は `specs/codex/configuration.md` §6 に収載済み。

## 巡回時の注意（2026-08-25 追記）

Claude⇔Codex 連携の**公式の入口が変わった**。従来 harness-harness が Claude→Codex ブリッジ候補として見ていた `codex mcp-server` は 0.149.1 で非推奨になり、公式 changelog は移行先として (a) プロダクト組み込み向けの `codex app-server`、(b) Claude Code 利用者向けの `openai/codex-plugin-cc` の 2 つを案内している。**クロスレビュー運用は (b) を第一候補とする**（`/codex:review --background` → `/codex:status` → `/codex:result` で Claude 側をブロックしない）。詳細は `specs/codex/mcp.md` §5 参照。
