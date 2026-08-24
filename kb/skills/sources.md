---
title: スキルエコシステム参照先
last_checked: "2026-08-25"
---

# スキルエコシステム参照先

巡回対象の公式・コミュニティサイト。信頼度と役割を整理。

## Anthropic公式

| サイト | URL | 役割 | 巡回優先度 |
|--------|-----|------|----------|
| claude.com/plugins | https://claude.com/plugins | 公式プラグインディレクトリ。72+件、24カテゴリ | 高 |
| anthropics/skills (GitHub) | https://github.com/anthropics/skills | 公式スキル実装例。source-available | 高 |
| anthropics/claude-plugins-official (GitHub) | https://github.com/anthropics/claude-plugins-official | 高品質プラグインディレクトリ | 中 |
| Claude Code Skills Docs | https://code.claude.com/docs/en/skills | スキル仕様・ベストプラクティス | 高 |

## OpenAI公式

| サイト | URL | 役割 | 巡回優先度 |
|--------|-----|------|----------|
| Codex Skills Docs | https://developers.openai.com/codex/skills | Skills = authoring format の仕様。**learn.chatgpt.com/docs/build-skills へ308恒久リダイレクト（2026-07確認）** | 高 |
| Codex Plugins Docs | https://developers.openai.com/codex/plugins | Plugins = installable distribution unit。作成手順の実体は learn.chatgpt.com/docs/build-plugins | 高 |
| openai/plugins (GitHub) | https://github.com/openai/plugins | **2026-08-16 に archive（read-only）**。5,110 stars、最終 push 2026-07-14。openai/skills の後継として案内されていたが更新停止 | 低（凍結） |
| openai/skills (GitHub) | https://github.com/openai/skills | **2026-06-22 に deprecated（#496）**。カタログは 5 system + 38 curated で凍結。README は後継として openai/plugins を案内するが、そちらも archive 済み | 低（凍結） |
| openai/codex-plugin-cc (GitHub) | https://github.com/openai/codex-plugin-cc | **Claude Code から Codex を使う公式プラグイン**（32.2K stars）。2026-08-24 の公式 changelog が `codex mcp-server` 非推奨の移行先として案内。OpenAI が GitHub 上で現役維持している数少ないカタログ | 高 |
| Codex app server Docs | https://learn.chatgpt.com/docs/app-server | `codex app-server`（JSON-RPC 2.0）の一次情報。`codex mcp-server` の後継 | 中 |

## オープン標準

| サイト | URL | 役割 | 巡回優先度 |
|--------|-----|------|----------|
| agentskills.io | https://agentskills.io/ | Agent Skills仕様。**46プラットフォーム採用**（2026-08-25 再確認、据え置き）。SKILL.mdフォーマット、3段階の段階的開示（Discovery / Activation / Execution） | 中 |

## コミュニティ

| サイト | URL | 役割 | 巡回優先度 |
|--------|-----|------|----------|
| skills.sh | https://skills.sh/ | コミュニティディレクトリ。90K+スキル登録。リーダーボード | 中 |
| awesome-claude-skills | https://github.com/travisvn/awesome-claude-skills | キュレーション済みリスト | 低 |

## 重要な区別

- **skills.shはAnthropicの公式サイトではない**。コミュニティ運営のディレクトリとして扱う
- **agentskills.ioはディレクトリではなく仕様サイト**。スキル一覧はここにはない
- **OpenAI側: Skills = authoring format、Plugins = distribution unit**。ChatGPT と Codex は単一の共通プラグインディレクトリを共有する
- **OpenAI の GitHub サンプルカタログは 2 つとも停止**（openai/skills = deprecated、openai/plugins = 2026-08-16 archive）。**Codex のスキル/プラグイン作成の一次情報は learn.chatgpt.com/docs/build-skills・build-plugins と `@plugin-creator`（Codex では `$plugin-creator`）**。GitHub リポジトリを参照先に書かないこと
  - ただし **openai/codex-plugin-cc は例外**。これはサンプルカタログではなく Claude Code 向けの実プラグインで、2026-08-24 の公式 changelog が正式な移行先として案内しているため参照先として有効
- **Agent Skills標準（SKILL.md）は46プラットフォームで共通**。Claude/Codex間でスキルファイル自体は変換不要
