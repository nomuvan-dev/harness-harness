---
title: Agent Skills エコシステム
last_patrol: "2026-07-20"
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
| [claude.com/plugins](https://claude.com/plugins) | Anthropic公式 | プラグイン/スキルディレクトリ（4 ページ構成、Frontend Design 1.08M installs が首位） |
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic公式 | 公式スキル実装例（162K+ stars、17 スキル。2026-07-17 に docx/pptx/xlsx 更新） |
| [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) | OpenAI公式 | Codex Skills仕様（learn.chatgpt.com/docs/build-skills へ 308 恒久リダイレクト） |
| [openai/skills](https://github.com/openai/skills) | OpenAI公式 | **2026-06-22 deprecated**（23.5K+ stars）。後継は [openai/plugins](https://github.com/openai/plugins)（180 プラグイン） |
| [skills.sh](https://skills.sh/) | コミュニティ | スキルディレクトリ兼リーダーボード（find-skills が 2.6M installs でトップ。mattpocock 系が top10 に 4 本と台頭） |
| [agentskills.io](https://agentskills.io/) | オープン標準 | Agent Skills仕様（SKILL.mdフォーマット）。仕様変更なし |

## 推薦スキル概要

詳細は `recommended.md` 参照。

**Tier A（管理対象）**: skill-creator, writing-skills, systematic-debugging, writing-plans, using-git-worktrees, dispatching-parallel-agents
**Tier B（ウォッチリスト）**: frontend-design, OWASP-security, Context7, find-skills
