# 共通概念対照表

最終更新: 2026-08-20

Claude Code と Codex CLI が共有する概念の対照表。harness-harness による抽象化レイヤー設計の基盤とする。

---

## 1. 指示ファイル

両プラットフォームとも、Markdown ベースの指示ファイルでエージェントの振る舞いを制御する。

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **指示ファイル名** | `CLAUDE.md` | `AGENTS.md` | 両方を生成。内容は共通テンプレートから派生 |
| **ファイル形式** | Markdown | Markdown | 共通 |
| **プロジェクトスコープ** | `./CLAUDE.md`, `.claude/CLAUDE.md` | `./AGENTS.md` | 各プラットフォームの規約に従い配置 |
| **ユーザースコープ** | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` | ホームディレクトリの違いを吸収 |
| **ディレクトリ走査** | ルートから CWD へ、各階層を走査 | Git ルートから CWD へ、各階層を走査 | 同等の動作 |
| **サイズ目安** | 200行以下 | 32 KiB 以下 | 制約の厳しい方（200行）に合わせる |
| **インポート構文** | `@path/to/file` | なし | Claude 向けのみインポートを使用 |
| **上書きファイル** | なし | `AGENTS.override.md` | Codex 向けのみ override を使用 |
| **無効化** | `claudeMdExcludes` 設定 | `--no-project-doc` / `CODEX_DISABLE_PROJECT_DOC=1` | プラットフォーム別の無効化方法を提供 |
| **自動生成** | `/init` | `/init` | 共通のベースから各形式を生成 |

### 1.1 指示ファイル共通テンプレート戦略

```
project-root/
├── CLAUDE.md          # Claude 向け（@docs/shared-instructions.md をインポート）
├── AGENTS.md          # Codex 向け（内容を直接記述）
└── docs/
    └── shared-instructions.md  # 共通指示（両方の元ネタ）
```

共通指示を単一ソースに保持し、各プラットフォーム向けファイルを生成または同期するワークフローを推奨する。

---

## 2. 設定ファイル

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **形式** | JSON (`settings.json`) | TOML (`config.toml`) | 抽象設定スキーマから各形式を生成 |
| **ユーザースコープ** | `~/.claude/settings.json` | `~/.codex/config.toml` | パスとフォーマットを変換 |
| **プロジェクトスコープ** | `.claude/settings.json` | `.codex/config.toml` | パスとフォーマットを変換 |
| **ローカルスコープ** | `.claude/settings.local.json` | なし | Claude 固有 |
| **システムスコープ** | Managed Policy | `/etc/codex/config.toml` | 各プラットフォームの管理方式に従う |
| **優先順位** | Managed > CLI > Local > Project > User | CLI > Profile > Project > User > System | 概念は同等。マージ戦略は異なる |
| **配列マージ** | 結合・重複排除 | 上書き | 挙動が異なる点に注意 |

---

## 3. MCP（Model Context Protocol）

両プラットフォームとも MCP をサポートし、外部ツール統合の主要手段として位置づけている。

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **プロトコルバージョン** | MCP 標準 | MCP 標準 | 共通 |
| **設定形式** | JSON (`.mcp.json` / `settings.json`) | TOML (`config.toml` の `[mcp_servers]`) | 同じサーバー定義を各形式に変換 |
| **STDIO トランスポート** | サポート | サポート | 共通。`command` + `args` は互換 |
| **HTTP トランスポート** | `--transport http` | `--url` / `url =` | 構文が異なるが概念は同等 |
| **SSE トランスポート** | サポート（非推奨） | 非サポート | HTTP に統一を推奨 |
| **WebSocket トランスポート** | サポート (`ws`) | 非サポート | Claude 固有 |
| **環境変数展開** | `${VAR}`, `${VAR:-default}` | TOML 内の `env` セクション | 記法が異なる |
| **CLI 管理** | `claude mcp add/list/get/remove` | `codex mcp add/list/get/remove` | コマンド構造が同等 |
| **TUI 確認** | `/mcp` | `/mcp` | 同等 |
| **OAuth 認証** | `/mcp` 経由 | `codex mcp login` | UI が異なるが機能は同等 |
| **サーバーモード** | 非サポート | `codex mcp-server`（0.149.1 で非推奨）→ `codex app-server` | Codex 固有。Claude Code から Codex を呼ぶ場合は公式プラグイン [codex-plugin-cc](https://github.com/openai/codex-plugin-cc) |

### 3.1 MCP サーバー定義の共通スキーマ

抽象化レイヤーで以下の共通スキーマを定義し、各プラットフォーム向けに変換する:

```yaml
# 共通 MCP サーバー定義（harness-harness 抽象スキーマ）
servers:
  github:
    transport: http
    url: "https://api.githubcopilot.com/mcp/"
    auth:
      type: bearer
      env_var: GITHUB_TOKEN
  playwright:
    transport: stdio
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
```

---

## 4. 権限管理

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **権限モデル** | ツール単位の細粒度制御 | ポリシーベースの段階制御 | 抽象レイヤーでは「安全度レベル」として統一 |
| **デフォルト動作** | 確認あり | `on-request`（確認あり） | 同等 |
| **全許可** | `permissions.allow: ["Bash(*)"]` 等 | `approval_policy = "never"` | 概念は同等。非推奨 |
| **全バイパス** | `bypassPermissions` | `--yolo` | 両方とも非推奨 |
| **読み取り専用** | `permissions.allow: ["Read", "Grep", "Glob"]` | `approval_policy = "untrusted"` | 近い動作 |
| **個別許可** | `permissions.allow: ["Bash(npm test)"]` | `/permissions` で設定 | 粒度が異なる |
| **個別拒否** | `permissions.deny: [...]` | なし | Claude 固有 |
| **TUI 設定** | `/permissions` | `/permissions` | 同等 |

### 4.1 安全度レベルの統一マッピング

| 抽象レベル | Claude Code | Codex CLI | 説明 |
|:--|:--|:--|:--|
| `strict` | `permissions.allow: ["Read", "Grep", "Glob"]` | `approval_policy = "untrusted"` | 読み取りのみ自動、他は全て確認 |
| `standard` | デフォルト | `approval_policy = "on-request"` | 書き込み・コマンド実行に確認 |
| `permissive` | 主要ツールを `allow` に追加 | `approval_policy = "never"` + サンドボックス | 確認なし（サンドボックスで保護） |
| `unrestricted` | `bypassPermissions` | `--yolo` | 全制限解除（非推奨） |

---

## 5. サンドボックス

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **読み取り専用** | サンドボックス設定 | `sandbox_mode = "read-only"` | 同等 |
| **ワークスペース書き込み** | デフォルト動作 | `sandbox_mode = "workspace-write"` | 同等 |
| **無制限** | サンドボックス無効化 | `sandbox_mode = "danger-full-access"` | 同等（非推奨） |
| **macOS 実装** | Apple Seatbelt | Apple Seatbelt | 同じ OS 機構 |
| **Linux 実装** | サンドボックス | bubblewrap + seccomp | OS レベルは異なる可能性あり |
| **ネットワーク制御** | なし | `network_access = true/false` | Codex の方が細かい |
| **保護対象** | プロジェクト外ファイル | `.git`, `.agents/`, `.codex/` + プロジェクト外 | 保護範囲が若干異なる |

---

## 6. モデル設定

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **モデル指定** | `model` 設定 / `ANTHROPIC_MODEL` / `/model` | `model` 設定 / `--model` / `/model` | キー名は同じ。値は異なる |
| **推論レベル** | `effortLevel` (`low`/`medium`/`high`) | `model_reasoning_effort` (`minimal`〜`xhigh`) | レベル名の変換が必要 |
| **プロバイダー** | Anthropic / Bedrock / Vertex | OpenAI / Azure / OpenRouter / Gemini / 他 | プロバイダーごとの認証設定が異なる |
| **拡張思考** | `alwaysThinkingEnabled` | `model_reasoning_summary` | 概念は近いが制御方法が異なる |

### 6.1 推論レベル対応表

| 抽象レベル | Claude Code | Codex CLI |
|:--|:--|:--|
| 最小 | `effortLevel: "low"` | `model_reasoning_effort = "minimal"` |
| 低 | `effortLevel: "low"` | `model_reasoning_effort = "low"` |
| 中 | `effortLevel: "medium"` | `model_reasoning_effort = "medium"` |
| 高 | `effortLevel: "high"` | `model_reasoning_effort = "high"` |
| 最大 | `effortLevel: "high"` + `alwaysThinkingEnabled: true` | `model_reasoning_effort = "xhigh"` |

---

## 7. セッション管理

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **セッション再開** | `/resume`, `claude --resume` | `/resume`, `codex resume` | 同等 |
| **セッション分岐** | `/branch` (`/fork`) | `/fork`, `codex fork` | 同等 |
| **新規セッション** | `/clear` (`/new`) | `/new`, `/clear` | 同等 |
| **コンパクション** | `/compact` | `/compact` | 同等 |
| **非対話実行** | `claude -p "タスク"` | `codex exec "タスク"` | コマンド構文が異なる |
| **履歴永続化** | デフォルト有効（`cleanupPeriodDays` で保持期間制御） | `[history] persistence` (`default`/`none`) | Codex の方が明示的に無効化可能 |
| **エフェメラル実行** | なし | `--ephemeral` | Codex 固有 |
| **差分表示** | `/diff` | `/diff` | 同等 |

---

## 8. サブエージェント / マルチエージェント

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **サポート状況** | 正式機能 | 実験的機能 (`multi_agent = true`) | 成熟度が大きく異なる |
| **ビルトインエージェント** | Explore, Plan, general-purpose, Bash | なし（実験的） | Claude が大幅にリード |
| **カスタム定義** | `.claude/agents/*.md` | なし | Claude 固有 |
| **モデル指定** | エージェントごとに設定可能 | 不可 | Claude 固有 |
| **ツール制限** | `tools` / `disallowedTools` フロントマター | 不可 | Claude 固有 |
| **バックグラウンド実行** | `background: true` / `Ctrl+B` | 実験的 (`/ps`) | 両方ある程度サポート |
| **永続メモリ** | `memory` フロントマター | なし | Claude 固有 |

---

## 9. Skills

> **2026-03-24 追加**: Codex CLI が Skills を正式サポート（stable）。両プラットフォームで SKILL.md フォーマットが共通となり、Skills は完全な共通概念となった。

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **スキルシステム** | 正式機能 | 正式機能（stable） | **共通** |
| **フォーマット** | SKILL.md（YAML フロントマター） | SKILL.md（YAML フロントマター） | 同一フォーマット。変換不要 |
| **プロジェクトスコープ** | `.claude/skills/` | `.agents/skills/`（**2026-03更新**: `.codex/skills/` から移行） | ディレクトリ名が異なる |
| **ユーザースコープ** | `~/.claude/skills/` | `$HOME/.agents/skills/`（**2026-03更新**: `~/.codex/skills/` から移行） | ディレクトリ名が異なる |
| **起動: コマンド** | `/skills` | `/skills` | 同等 |
| **起動: メンション** | `$` メンション | `$` メンション | 同等 |
| **起動: 暗黙マッチング** | サポート | サポート | 同等 |
| **`$ARGUMENTS` 変数** | サポート | **未確認** | Claude では確実にサポート |
| **`allowed-tools` フロントマター** | サポート | **未確認** | Claude 固有の可能性 |
| **`context: fork` フロントマター** | サポート | **未確認** | Claude 固有の可能性 |
| **動的コンテキスト** | `` !`command` `` | **未確認** | Claude 固有の可能性 |
| **MCP 依存自動インストール** | デフォルト有効 | `skill_mcp_dependency_install = true`（stable） | 同等 |

### 9.1 Skills の抽象化方針

Skills は両プラットフォームで最も互換性の高い概念の一つ:

1. **SKILL.md の共有**: 同一の SKILL.md ファイルを `.claude/skills/` と `.agents/skills/` の両方に配置可能
2. **ディレクトリ名変換のみ**: 唯一の違いは親ディレクトリ名（`.claude/skills/` vs `.agents/skills/`）
3. **共通テンプレート**: `templates/shared/skills/` にスキルテンプレートを配置し、両プラットフォーム向けにコピーするだけで利用可能
4. **Codex のメタデータ拡張**: Codex は `agents/openai.yaml` で UI 表示設定・暗黙呼び出しポリシー・ツール依存を追加定義可能（Claude 側には対応なし）

---

## 10. Hooks / ライフサイクル制御

> **2026-03-24 更新**: Codex CLI が Hooks を実験的サポート開始。3 イベント・command ハンドラのみだが、共通概念として扱えるようになった。
> **2026-08-20 更新（Codex 0.148.0）**: Codex が 11 イベント・`mcp_tool` ハンドラ・非同期実行に対応。イベント構造（イベント → matcher グループ → ハンドラ配列）も Claude と同型になり、**Hooks は「Claude 固有」から「ほぼ共通概念」へ格上げ**された。

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **フックシステム** | 包括的なイベント駆動フック（17+ イベント） | 11 イベント（0.148.0+） | 共通イベントを抽象化。差分は通知系・UI 系のみ |
| **設定形式** | JSON (`"hooks"` オブジェクト) | JSON (`.codex/hooks.json`) または TOML (`[[hooks.<Event>]]`) | `hooks.json` 同士なら構造は同型。TOML 側のみ形式変換 |
| **構造** | イベント → matcher グループ → ハンドラ配列 | 同一（0.148.0+） | **共通**。3階層構造をそのまま抽象スキーマにできる |
| **有効化** | デフォルト有効 | デフォルト有効（0.124.0+） | **共通**（0.123.0 以前は `codex_hooks` フラグが必要） |
| **SessionStart / Stop / UserPromptSubmit** | サポート | サポート | **共通** |
| **SessionEnd** | サポート | サポート（0.148.0+） | **共通**。Codex は既定タイムアウト 1 秒 |
| **PreToolUse / PostToolUse** | サポート | サポート（0.148.0+） | **共通**。matcher のツール名変換が必要 |
| **PreCompact** | サポート | サポート（0.148.0+） | **共通**。matcher は `manual` / `auto` |
| **PostCompact** | なし（`SessionStart` の compact source） | サポート（0.148.0+） | Codex 固有。Claude 側は compact 後の `SessionStart` に写す |
| **SubagentStart** | なし | サポート（0.133.0+） | Codex 固有 |
| **SubagentStop** | サポート | サポート（0.133.0+） | **共通** |
| **PermissionRequest** | なし | サポート（0.148.0+） | Codex 固有 |
| **Notification 等の UI 系** | サポート | なし | Claude 固有 |
| **ハンドラ: Command** | サポート | サポート | **共通**。Codex は `commandWindows` で OS 分岐可 |
| **ハンドラ: mcp_tool** | なし | サポート（0.148.0+） | Codex 固有。Claude は command / Agent で代替 |
| **ハンドラ: HTTP** | サポート | なし | Claude 固有。Codex は `mcp_tool` または command + `curl` |
| **ハンドラ: Prompt / Agent** | サポート | 設定は受理されるが未実装 | Claude 固有 |
| **非同期実行** | なし（同期のみ） | `async = true`（0.148.0+） | Codex 固有。起動元操作を制御できないためガード用途は不可 |
| **ブロッキング制御** | 終了コード 2 でブロック | 終了コード 2 でブロック | **共通** |
| **Managed 限定化** | `allowManagedHooksOnly` | `requirements.toml` の `allow_managed_hooks_only` | **共通**。Codex は `config.toml` では効かない |

### 9.1 Hooks の抽象化方針

Hooks は Claude が先行し Codex が部分的に追随した領域。harness-harness の抽象化では:

1. **共通イベント活用**: `SessionStart` / `SessionEnd` / `Stop` / `UserPromptSubmit` / `PreToolUse` / `PostToolUse` / `PreCompact` / `SubagentStop` の 8 イベントが両プラットフォームで利用可能。抽象スキーマから各形式（JSON / TOML）に変換する
2. **ツール名の変換表を持つ**: `PreToolUse` / `PostToolUse` の `matcher` は両者でツール名が異なるため、抽象スキーマ側は論理名で持ち、変換時に実名へ展開する
3. **MCP 連携パターン**: 外部通知やバリデーションは MCP サーバーとして実装し、両プラットフォームから利用
4. **段階的移行**: Codex の Hooks が成熟するにつれ、ラッパースクリプトから Hooks ネイティブに移行する計画を持つ

### 9.2 Hooks 共通スキーマ例

```yaml
# harness-harness 抽象スキーマ
hooks:
  - event: SessionStart
    handler: command
    command: "python3 scripts/preflight.py"
  - event: UserPromptSubmit
    handler: command
    command: "python3 scripts/validate-prompt.py"
  - event: Stop
    handler: command
    command: "python3 scripts/postflight.py"
```

---

## 11. 非対話実行（CI/CD 統合）

| 概念 | Claude Code | Codex CLI | 抽象化の方針 |
|:--|:--|:--|:--|
| **基本コマンド** | `claude -p "タスク"` | `codex exec "タスク"` | 統一ラッパーで吸収 |
| **JSON 出力** | `--output-format json` | `--json` / `--experimental-json` | フラグ名が異なる |
| **stdin 入力** | `claude -p -` | `codex exec -` | 同等 |
| **セッションなし実行** | なし | `--ephemeral` | Codex 固有 |
| **出力保存** | リダイレクト | `--output-last-message <path>` | Codex の方が組み込みサポート |
| **Git リポジトリ外実行** | 可能 | `--skip-git-repo-check` | Codex はフラグが必要 |
| **画像添付** | サポート | `--image <path>` | 両方サポート |

### 10.1 CI/CD 統一パターン

```bash
#!/bin/bash
# harness-harness 統一実行ラッパー例
PLATFORM="${HARNESS_PLATFORM:-claude}"  # "claude" or "codex"
TASK="$1"

if [ "$PLATFORM" = "claude" ]; then
    claude -p "$TASK"
elif [ "$PLATFORM" = "codex" ]; then
    codex exec -a never -s workspace-write "$TASK"
fi
```

---

## 12. 機能対応サマリー

各概念の対応状況を一覧する。

| 共通概念 | Claude Code | Codex CLI | 互換性 |
|:--|:--|:--|:--|
| 指示ファイル | CLAUDE.md | AGENTS.md | 高（内容互換、名前変換のみ） |
| 設定ファイル | settings.json (JSON) | config.toml (TOML) | 中（形式変換 + キー名マッピング必要） |
| MCP | 包括的サポート | 基本サポート | 高（STDIO/HTTP は互換、スコープが異なる） |
| 権限管理 | 細粒度 | ポリシーベース | 低（抽象レベルでの統一が必要） |
| サンドボックス | サポート | サポート | 高（3段階モデルで統一可能） |
| セッション管理 | 包括的 | 包括的 | 高（コマンド名は異なるが機能は同等） |
| Hooks | 包括的（17+ イベント） | 11 イベント（0.148.0+） | **高**（構造が同型。共通 8 イベント。差分は通知系と Prompt/Agent ハンドラ） |
| サブエージェント | 正式機能 | 実験的 | 低（機能差が大きい） |
| Skills | 包括的 | 正式サポート（stable） | 高（SKILL.md フォーマット共通、ディレクトリ名変換のみ） |
| プロファイル | なし | サポート | なし（Codex 固有、Claude はエイリアスで代替） |
| 非対話実行 | サポート | 包括的 | 高（フラグ名変換で対応可能） |
| Web 検索 | なし | サポート | なし（Codex 固有） |
| メモリシステム | オートメモリ | なし | なし（Claude 固有） |
| プラグイン | サポート | なし | なし（Claude 固有） |
