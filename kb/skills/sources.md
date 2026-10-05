---
title: スキルエコシステム参照先
last_checked: "2026-10-06"
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
| openai/plugins (GitHub) | https://github.com/openai/plugins | **2026-10-06 に archive 解除（復活）を確認**。7,309 stars、最終更新 2026-09-28。Codex プラグイン実例の現役キュレーションカタログ（`plugins/<name>/` + `.codex-plugin/plugin.json`、figma / notion / build-ios-apps / build-macos-apps / build-web-apps / expo / netlify / remotion / google-slides）。マーケットプレイスは `.agents/plugins/marketplace.json`（API キーログイン用は `api_marketplace.json`） | 高（復活） |
| openai/skills (GitHub) | https://github.com/openai/skills | **deprecated 継続**（27,889 stars）。README が 2026-09-08 更新で後継として「復活した openai/plugins」＋ build-plugins ガイドを案内。`$skill-installer` の手順記載は維持 | 低（凍結） |
| openai/codex-plugin-cc (GitHub) | https://github.com/openai/codex-plugin-cc | **Claude Code から Codex を使う公式プラグイン**（32.2K stars）。2026-08-24 の公式 changelog が `codex mcp-server` 非推奨の移行先として案内。OpenAI が GitHub 上で現役維持している数少ないカタログ | 高 |
| Codex app server Docs | https://learn.chatgpt.com/docs/app-server | `codex app-server`（JSON-RPC 2.0）の一次情報。`codex mcp-server` の後継 | 中 |

## オープン標準

| サイト | URL | 役割 | 巡回優先度 |
|--------|-----|------|----------|
| agentskills.io | https://agentskills.io/ | Agent Skills仕様。**46プラットフォーム採用**（2026-10-06 再確認、据え置き）。SKILL.mdフォーマット、3段階の段階的開示（Discovery / Activation / Execution）。仕様バージョン変更なし | 中 |

## コミュニティ

| サイト | URL | 役割 | 巡回優先度 |
|--------|-----|------|----------|
| skills.sh | https://skills.sh/ | コミュニティディレクトリ。90K+スキル登録。リーダーボード | 中 |
| awesome-claude-skills | https://github.com/travisvn/awesome-claude-skills | キュレーション済みリスト | 低 |

## 重要な区別

- **skills.shはAnthropicの公式サイトではない**。コミュニティ運営のディレクトリとして扱う
- **agentskills.ioはディレクトリではなく仕様サイト**。スキル一覧はここにはない
- **OpenAI側: Skills = authoring format、Plugins = distribution unit**。ChatGPT と Codex は単一の共通プラグインディレクトリを共有する
- **openai/plugins は 2026-09-28 頃に archive 解除されて復活した**（2026-10-06 確認）。openai/skills は deprecated のままだが README が openai/plugins を現役後継として案内。**作成手順の一次情報は引き続き learn.chatgpt.com/docs/build-skills・build-plugins と `@plugin-creator`（Codex では `$plugin-creator`）**だが、**実例参照は openai/plugins を使ってよい**（「GitHub リポジトリを参照先に書かない」ルールは失効）
  - **openai/codex-plugin-cc** は Claude Code 向けの実プラグインで、2026-08-24 の公式 changelog が `codex mcp-server` の正式な移行先として案内しているため引き続き参照先として有効
- **Agent Skills標準（SKILL.md）は46プラットフォームで共通**。Claude/Codex間でスキルファイル自体は変換不要
