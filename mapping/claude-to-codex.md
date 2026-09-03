# Claude Code → Codex CLI 変換ルール

最終更新: 2026-03-24

本ドキュメントは Claude Code の各機能が Codex CLI でどのように対応するかを定義する。対応なしの場合は代替策を記載する。

---

## 1. 設定ファイル

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| `CLAUDE.md` | `AGENTS.md` | 指示ファイル名が異なる。内容は Markdown で共通 |
| `.claude/CLAUDE.md` | `AGENTS.md`（プロジェクトルート） | Codex は `.codex/` 配下ではなくディレクトリ直下に配置 |
| `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` | グローバルスコープの指示 |
| `settings.json` | `config.toml` | JSON → TOML 形式変換が必要 |
| `~/.claude/settings.json` | `~/.codex/config.toml` | ユーザーレベル設定 |
| `.claude/settings.json` | `.codex/config.toml` | プロジェクトレベル設定 |
| `.claude/settings.local.json` | **対応なし** | 代替: `.codex/config.toml` をプロファイルで分離し `.gitignore` に追加 |
| `~/.claude.json` | `~/.codex/config.toml` | Codex は全設定を `config.toml` に集約 |
| `.mcp.json` | `.codex/config.toml` の `[mcp_servers]` | MCP設定はメイン設定ファイルに統合 |

### 1.1 変換ガイド: CLAUDE.md → AGENTS.md

- ファイル名を `CLAUDE.md` から `AGENTS.md` に変更する
- `@path/to/import` インポート構文は Codex に存在しない。インポート先の内容を直接記述するか、AGENTS.md にまとめる
- `.claude/rules/` のパススコープルール（YAML フロントマター `paths:`）は対応なし。AGENTS.md 本文に条件付き指示として記述する
- サイズ制限: Claude は200行目安、Codex は32 KiB（`project_doc_max_bytes` で変更可能）

### 1.2 変換ガイド: settings.json → config.toml

```
# Claude Code (JSON)
{
  "model": "sonnet",
  "permissions.allow": ["Bash(npm test)"],
  "effortLevel": "high"
}

# Codex CLI (TOML)
model = "o4-mini"
approval_policy = "on-request"
model_reasoning_effort = "high"
```

---

## 2. 指示ファイルの階層構造

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| Managed Policy CLAUDE.md | **対応なし** | 代替: `/etc/codex/config.toml` のシステム設定で部分的に代替 |
| Project CLAUDE.md | `AGENTS.md`（Git ルート〜CWD の各階層） | 同等。Codex もディレクトリ走査で読み込む |
| User CLAUDE.md | `~/.codex/AGENTS.md` | 同等 |
| `.claude/rules/*.md` | **対応なし** | 代替: AGENTS.md 内にセクション分けして記述 |
| パススコープルール（`paths:` フロントマター） | **対応なし** | 代替: AGENTS.md 本文中に「このルールは `src/api/**` に適用」と記述 |
| `AGENTS.override.md` | Claude に対応なし → **Codex 固有** | Codex のみの機能。override が必要な場合は Codex のみに設定 |
| `claudeMdExcludes` | **対応なし** | 代替: ディレクトリ構成で回避 |

---

## 3. 権限・サンドボックス

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| `permissions.allow` | `approval_policy` + 個別設定 | Claude は細粒度（ツール単位）、Codex はポリシーベース |
| `permissions.deny` | **対応なし（ポリシーで制御）** | Codex は `approval_policy` で全体制御 |
| `permissions.ask` | `approval_policy = "on-request"` | デフォルトの承認モードで近い動作 |
| `permissions.defaultMode` | `approval_policy` | 直接対応 |
| `bypassPermissions` モード | `--yolo` フラグ | 両方とも非推奨の全バイパス |
| サンドボックス設定 (`sandbox.*`) | `sandbox_mode` | Claude は詳細設定、Codex は3段階モード |

### 3.1 権限モード対応表

| Claude Code | Codex CLI | 説明 |
|:--|:--|:--|
| `permissions.allow: ["Bash(*)"]` | `approval_policy = "never"` | 全コマンド許可（危険） |
| `permissions.allow: ["Bash(npm test)"]` | `/permissions` で個別設定 | Codex も個別コマンド許可が可能 |
| デフォルト（確認あり） | `approval_policy = "on-request"` | 標準的な動作 |
| `permissions.deny: ["Bash(rm *)"]` | **対応なし** | 代替: AGENTS.md に「`rm` コマンドを使用しないこと」と記述 |

### 3.2 サンドボックスモード対応表

| Claude Code | Codex CLI |
|:--|:--|
| サンドボックス無効 | `sandbox_mode = "danger-full-access"` |
| 読み取り専用 | `sandbox_mode = "read-only"` |
| ワークスペース書き込み可 | `sandbox_mode = "workspace-write"` |

---

## 4. Skills / Commands

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| Skills (`SKILL.md`) | Skills (`SKILL.md`) | **直接対応**。SKILL.md フロントマター形式は同一 |
| `.claude/skills/<name>/SKILL.md` | `.agents/skills/<name>/SKILL.md` | ディレクトリ名が異なる（`.claude/skills/` → `.agents/skills/`）。**2026-03更新**: Codex は `.codex/skills/` から `.agents/skills/` に移行 |
| `~/.claude/skills/` | `$HOME/.agents/skills/` | グローバルスキルのパスも変更 |
| `/skills` コマンド | `/skills` コマンド | 同等。スキル一覧表示と選択 |
| `$` メンションでスキル参照 | `$` メンションでスキル参照 | 同等 |
| `$ARGUMENTS` 変数展開 | **対応なし** | 代替: プロンプトで直接指示を渡す |
| `context: fork` サブエージェント実行 | サブエージェント機能（実験的） | `[features] multi_agent = true` で有効化 |
| `allowed-tools` フロントマター | **対応なし** | 代替: `approval_policy` で全体制御 |
| 動的コンテキスト注入 (`` !`command` ``) | **対応なし** | 代替: シェルスクリプトで事前にコンテキストを生成し、プロンプトに含める |
| バンドルスキル多数 | バンドル3種（`skill-creator`, `skill-installer`, `openai-docs`） | Claude の方がバンドルスキルは豊富 |

---

## 5. コマンド（Slash Commands）

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| `/clear` | `/clear`, `/new` | 同等 |
| `/compact` | `/compact` | 同等 |
| `/resume` | `/resume` | 同等。Codex は CLI サブコマンドとしても利用可能 |
| `/branch` (`/fork`) | `/fork` | 同等 |
| `/diff` | `/diff` | 同等 |
| `/model` | `/model` | 同等 |
| `/config` | `/debug-config` | Codex は診断寄り |
| `/permissions` | `/permissions` | 同等 |
| `/mcp` | `/mcp` | 同等 |
| `/status` | `/status` | 同等 |
| `/init` | `/init` | Claude は CLAUDE.md 生成、Codex は AGENTS.md 生成 |
| `/plan` | `/plan` | 同等 |
| `/export` | **対応なし** | 代替: `codex exec --output-last-message` で最終出力を保存 |
| `/memory` | **対応なし** | Codex にオートメモリ機能なし |
| `/effort` | **対応なし** | 代替: `config.toml` の `model_reasoning_effort` で設定 |
| `/cost` | `/status` | Codex の `/status` にトークン使用量が含まれる |
| `/context` | `/status` | 同上 |
| `/pr-comments` | **対応なし** | 代替: GitHub MCP サーバー経由でPRコメントを取得 |
| `/security-review` | `/review` | Codex の `/review` はセキュリティ含む汎用レビュー |
| `/sandbox` | **対応なし** | 代替: `--sandbox` CLI フラグ |
| `/add-dir` | `--add-dir` CLI フラグ | Claude はコマンド、Codex はフラグのみ |
| `/agents` | `/agent` | 同等（名称が微妙に異なる） |
| `/skills` | `/skills` | 同等。両方ともスキル一覧を表示 |
| `/batch` | **対応なし** | 代替: `codex exec` をシェルスクリプトで並列実行 |
| `/btw` | **対応なし** | 代替: 通常のプロンプトで質問 |
| `/rewind` | **対応なし** | Codex にチェックポイント機能なし |
| `/voice` | **対応なし** | Codex に音声入力なし |

---

## 6. Hooks

> **2026-03-24 更新**: Codex CLI が Hooks を実験的にサポート開始。
> **2026-05-21 更新（Codex 0.133.0）**: `SubagentStart` / `SubagentStop` / compact `SessionStart` / `MITM` を追加し対応イベントは 6 種に拡張。
> **2026-09-03 更新（Codex 0.150.0 / 0.153.0）**: `Interrupt` が加わり対応イベントは **12 種**。0.153.0 で公式 hooks ドキュメントに収載された。
> **2026-08-20 更新（Codex 0.148.0）**: 対応イベントが **11 種**に拡張され、`PreToolUse` / `PostToolUse` / `PermissionRequest` / `PreCompact` / `PostCompact` / `SessionEnd` が**直接対応**に昇格。ハンドラも `mcp_tool` が追加され、`command` は `async` 実行に対応。変換可能性が大きく改善したため、以下の代替パターンの多くは不要になった。

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| Hooks システム全体 | **概ね対応** | 0.124.0+ デフォルト有効。`~/.codex/hooks.json` / `<repo>/.codex/hooks.json` または `config.toml` の `[[hooks.<Event>]]` で設定 |
| `SessionStart` | `SessionStart` | **直接対応**。matcher で `startup` / `resume` / `clear` / `compact` を切り分け可能 |
| `SessionEnd` | `SessionEnd`（0.148.0+） | **直接対応**。既定タイムアウトが 1 秒と短く、`async` 指定と `mcp_tool` ハンドラは非対応 |
| `Stop` フック | `Stop` | **直接対応** |
| `UserPromptSubmit` | `UserPromptSubmit` | **直接対応**。終了コード 2 でブロック可能 |
| `PreToolUse` | `PreToolUse`（0.148.0+） | **直接対応**。matcher はツール名の正規表現。ツール名が両者で異なる点に注意（変換表は §3 参照） |
| `PostToolUse` | `PostToolUse`（0.148.0+） | **直接対応**。matcher はツール名の正規表現 |
| `PreCompact` | `PreCompact`（0.148.0+） | **直接対応**。matcher は `manual` / `auto` |
| （Claude は `SessionStart` の compact source で代替） | `PostCompact`（0.148.0+） | Codex 独自の明示イベント |
| `SubagentStop` | `SubagentStop`（0.133.0+） | **直接対応** |
| （Claude にはない） | `SubagentStart`（0.133.0+） | Codex 独自。サブエージェント起動契機での前処理に利用 |
| （Claude にはない） | `PermissionRequest`（0.148.0+） | Codex 独自。権限確認の発生を捕捉 |
| （Claude にはない） | `Interrupt`（0.150.0+、2026-09-03 に公式ドキュメント収載） | Codex 独自。メインスレッドのターン中断時に走る。**アイドルスレッドとサブエージェントでは走らず `matcher` は無視**。入力に `turn_id` / `permission_mode` を含む。既定タイムアウト 1 秒・上限 3 秒で、中断の阻止やターン再開はできない。出力は終了コード 0 の無出力か `systemMessage` を含む JSON のみ（プレーンテキストは不正）。Claude 側は Esc 中断を捕捉するイベントを持たないため、変換時は Codex 側限定の機能として残す |
| `Notification` 他の UI 系イベント | **対応なし** | Claude は 17+ イベント。残差は主に通知・UI 系 |
| HTTP ハンドラ | **対応なし** | 代替: `mcp_tool` ハンドラ（0.148.0+）で MCP サーバー経由の外部連携、または command ハンドラ内で `curl` |
| Prompt ハンドラ | **設定は受理されるが未実装** | `type = "prompt"` はパースされるがスキップ警告が出る。代替: AGENTS.md に判断基準を記述 |
| Agent ハンドラ | **設定は受理されるが未実装** | `type = "agent"` も同様にスキップ。代替: サブエージェント機能で部分的に代替 |
| （Claude にはない） | `mcp_tool` ハンドラ（0.148.0+） | Codex 独自。`server` / `tool` / `input` を指定してフックから MCP ツールを直接呼ぶ |
| （Claude にはない） | `async = true`（0.148.0+） | Codex 独自。バックグラウンド実行。**起動元の操作を制御できない**ため、ガード用途には使わない |

### 6.1 設定形式の違い

Claude Code の JSON 構造（イベント → matcher グループ → ハンドラ配列）は、Codex 0.148.0 でほぼそのまま写せるようになった。`~/.codex/hooks.json` を使えば構造は同型で、トップレベルキーが `hooks` である点も同じ。

```jsonc
// Claude Code (.claude/settings.json)
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "python3 guard.py" }] }
    ]
  }
}

// Codex CLI (.codex/hooks.json) — ほぼ同型
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "shell", "hooks": [{ "type": "command", "command": "python3 guard.py" }] }
    ]
  }
}
```

`config.toml` にインラインで書く場合は array-of-tables を使う:

```toml
[[hooks.PreToolUse]]
matcher = "shell"

  [[hooks.PreToolUse.hooks]]
  type = "command"
  command = "python3 guard.py"
  timeout = 30
```

変換時の主な注意点:

- **matcher のツール名を変換する**: Claude の `Bash` / `Edit` / `Write` などは Codex のツール名に読み替える必要がある
- **Windows 分岐**: Codex は `commandWindows`（`command_windows`）でプラットフォーム別コマンドを指定できる。Claude にはこの機構がないため、逆変換ではラッパースクリプトに寄せる
- **タイムアウト既定値**: Codex は 600 秒、`SessionEnd` は 1 秒、`Interrupt` は 1 秒（上限 3 秒）
- **Managed 制約**: Codex は `requirements.toml` の `allow_managed_hooks_only = true` が Claude の `allowManagedHooksOnly` に対応する（`config.toml` に書いても効かない点に注意）

### 6.2 Hooks の代替パターン（残る非対応イベント向け）

0.148.0 で直接対応が大幅に増えたため、代替が必要なのは主に通知系・UI 系イベントとハンドラ種別:

1. **Notification 系の外部通知** → `mcp_tool` ハンドラで MCP サーバーを呼ぶ、または command ハンドラ内で Webhook 送信
2. **Prompt / Agent ハンドラ相当の判断** → AGENTS.md に判断基準を記述、またはサブエージェントで代替
3. **ブロック用途と非同期用途の使い分け** → 実行を止めたいガードは同期（`async` を付けない）、通知・計測・ログ送信は `async = true`

---

## 7. MCP

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| `.mcp.json` | `.codex/config.toml` の `[mcp_servers]` | JSON → TOML 変換が必要 |
| `claude mcp add` | `codex mcp add` | ほぼ同等の CLI |
| `--transport http` | `--url` | HTTP サーバーの追加方法が異なる |
| `--transport sse` | **対応なし** | Codex は SSE 非サポート。Streamable HTTP に移行 |
| `--transport stdio` | デフォルト | Codex のデフォルトは stdio |
| `--scope local/project/user` | グローバル or プロジェクト | Codex は2スコープのみ |
| `enableAllProjectMcpServers` | プロジェクト信頼設定 | Codex は信頼済みプロジェクトのみ読み込み |
| `allowManagedMcpServersOnly` | **対応なし** | Codex に Managed MCP 概念なし |
| Managed MCP (`managed-mcp.json`) | **対応なし** | 代替: `/etc/codex/config.toml` のシステム設定 |
| MCPチャンネル（プッシュメッセージ） | **対応なし** | Codex はプッシュ通知非対応 |
| プラグイン提供 MCP | **対応なし** | Codex にプラグインシステムなし |

### 7.1 MCP 設定変換例

```
# Claude Code (.mcp.json)
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}

# Codex CLI (config.toml)
[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_TOKEN"

[mcp_servers.playwright]
command = "npx"
args = ["-y", "@playwright/mcp@latest"]
```

---

## 8. サブエージェント

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| ビルトインサブエージェント（Explore, Plan, Bash等） | **対応なし（実験的）** | `[features] multi_agent = true` で実験的サポート |
| カスタムサブエージェント (`.claude/agents/`) | **対応なし** | 代替: AGENTS.md に役割別の指示セクションを記述 |
| サブエージェントのモデル指定 | **対応なし** | Codex はセッション全体で単一モデル |
| サブエージェントの永続メモリ | **対応なし** | Codex にメモリシステムなし |
| `background: true` バックグラウンド実行 | `/ps` で確認可能な実験的機能 | Codex もバックグラウンドターミナルを実験的にサポート |
| ワークツリー隔離 (`isolation: worktree`) | **対応なし** | 代替: 別ディレクトリで `codex exec` を実行 |

---

## 9. メモリシステム

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| オートメモリ | **対応なし** | Codex にセッション間学習機能なし |
| `MEMORY.md` | **対応なし** | 代替: AGENTS.md にプロジェクト知識を手動記述 |
| `/memory` コマンド | **対応なし** | - |
| サブエージェントメモリ | **対応なし** | - |

### 9.1 メモリの代替策

Codex にはオートメモリがないため、以下のアプローチで代替する:

1. `AGENTS.md` にプロジェクト固有の知識・慣習を詳細に記述
2. セッション履歴の `resume` 機能でコンテキストを引き継ぐ
3. `/compact` でコンテキストを圧縮しつつ重要情報を保持

---

## 10. 環境変数

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| `ANTHROPIC_API_KEY` | `OPENAI_API_KEY` | プロバイダーが異なる |
| `ANTHROPIC_MODEL` | `--model` フラグ or `config.toml` | Codex は環境変数でのモデル指定なし |
| `ANTHROPIC_DEFAULT_MODEL` | `config.toml` の `model` | Claude 側は「ユーザーの `/model` 選択を上書きしない既定値」（v2.1.236）。Codex には同等の階層がなく `config.toml` の `model` が最も近い（`--model` フラグで都度上書き） |
| `CLAUDE_CODE_USE_BEDROCK` | `model_provider` 設定 | Codex は `model_provider` で統一管理 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | **対応なし** | Codex にメモリなし |
| `MCP_TIMEOUT` | `startup_timeout_sec` / `tool_timeout_sec` | Codex は TOML 設定で個別制御 |
| `MAX_MCP_OUTPUT_TOKENS` | **対応なし** | Codex に相当設定なし |

---

## 11. その他の機能

| Claude Code | Codex CLI | 備考 |
|:--|:--|:--|
| プロファイル | プロファイル (`[profiles.*]`) | 同等。Codex は `--profile` フラグで切替 |
| プラグインシステム | **対応なし** | Codex にプラグイン概念なし |
| デスクトップアプリ連携 (`/desktop`) | `codex app`（macOS のみ） | 部分的対応 |
| IDE連携 (`/ide`) | IDE拡張（設定共有） | Codex は IDE と設定ファイルを共有 |
| Web検索 | `web_search` 設定 | Codex は `cached`/`live`/`disabled` の3段階 |
| `codex exec`（非対話モード） | Claude は `claude -p` | 両方ともパイプライン実行をサポート |
| Codex Cloud | Claude にはなし → **Codex 固有** | `codex cloud` でリモートタスク実行 |
| OpenTelemetry 監視 | Claude にはなし → **Codex 固有** | `[otel]` セクションで設定 |
| シェル環境ポリシー | Claude にはなし → **Codex 固有** | `[shell_environment_policy]` で環境変数フィルタリング |
