---
title: Agent Skills エコシステム
last_patrol: "2026-08-03"
standard: agentskills.io (44 platforms confirmed)
tracked_skills: 10
patrol_schedule: weekly (月曜)
---

# Agent Skills エコシステム管理

Claude Code / Codex CLI で活用できる公式・コミュニティスキルの推薦管理。段階的開示で必要時に詳細参照。

## 参照先一覧

詳細は `sources.md` 参照。

| サイト | 種別 | 役割 |
|--------|------|------|
| [claude.com/plugins](https://claude.com/plugins) | Anthropic公式 | プラグイン/スキルディレクトリ（Frontend Design 1.13M installs が首位。言語別 LSP プラグイン群が台頭、Anthropic 検証バッジ導入） |
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic公式 | 公式スキル実装例（165K+ stars、17 スキル。2026-07-24 に claude-api スキルが Opus 5 対応、以降新規 push なし） |
| [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) | OpenAI公式 | Codex Skills仕様（learn.chatgpt.com/docs/build-skills へ 308 恒久リダイレクト） |
| [openai/skills](https://github.com/openai/skills) | OpenAI公式 | **2026-06-22 deprecated**（23.5K+ stars）。後継は [openai/plugins](https://github.com/openai/plugins)（180 プラグイン） |
| [skills.sh](https://skills.sh/) | コミュニティ | スキルディレクトリ兼リーダーボード（find-skills が 2.8M installs でトップ。mattpocock 系 4 本が top10 継続。agent-browser / vercel-react-best-practices / ai-video-generation が新規 top10 入り、microsoft-foundry は圏外へ） |
| [agentskills.io](https://agentskills.io/) | オープン標準 | Agent Skills仕様（SKILL.mdフォーマット）。仕様変更なし、44 プラットフォーム採用継続 |

## 推薦スキル概要

詳細は `recommended.md` 参照。

**Tier A（管理対象）**: skill-creator, systematic-debugging, test-driven-development, frontend-design, security-scan, pdf/pptx/xlsx/docx, dispatching-parallel-agents, using-git-worktrees, postgres-best-practices, find-skills
**Tier B（ウォッチリスト）**: docker-optimize, data-pipeline, Context7, landing-page-guide, deploy-checklist, brainstorming
