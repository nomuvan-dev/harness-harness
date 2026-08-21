# OpenAI Codex CLI 設定仕様

最終更新: 2026-08-20（巡回更新）

---

## 1. config.toml

Codex CLI の設定は TOML 形式で管理される。複数レベルの設定ファイルがマージされ、より近いスコープが優先される。

### 1.1 設定ファイルの配置場所

| レベル | パス | 用途 |
|--------|------|------|
| システム | `/etc/codex/config.toml` | 全ユーザー共通（任意） |
| ユーザー | `~/.codex/config.toml` | 個人のデフォルト設定 |
| プロジェクト | `.codex/config.toml`（リポジトリルートから CWD まで複数可） | プロジェクト固有の設定（信頼済みプロジェクトのみ読み込み） |

### 1.2 設定の優先順位（高い順）

1. CLI フラグ / `--config` (`-c`) による上書き
2. プロファイル値（`--profile <name>`）
3. プロジェクト設定（CWD に最も近いファイルが優先）
4. ユーザー設定（`~/.codex/config.toml`）
5. システム設定（`/etc/codex/config.toml`）
6. ビルトインデフォルト

### 1.3 主要キー一覧

#### コア設定

| キー | 型 | デフォルト | 説明 |
|------|------|-----------|------|
| `model` | string | `"o4-mini"` | 使用する AI モデル |
| `model_provider` | string | `"openai"` | モデルプロバイダー ID。組み込みプロバイダに **`amazon-bedrock-runtime`**（0.148.0+）が追加された（下記「Amazon Bedrock Runtime プロバイダ」参照） |
| `approval_policy` | string | `"on-request"` | 承認ポリシー（後述） |
| `sandbox_mode` | string | `"workspace-write"` | サンドボックスモード（後述） |
| `personality` | string | `"friendly"` | 応答スタイル（`none` / `friendly` / `pragmatic`） |
| `service_tier` | string | - | パフォーマンス層（`flex` / `fast`） |
| `web_search` | string | `"cached"` | Web 検索動作（`disabled` / `cached` / `live`） |
| `model_reasoning_effort` | string | `"high"` | 推論レベル（`minimal` 〜 `xhigh`。上位に `max` / `ultra` あり。0.149.0 で SDK からも `max` / `ultra` を選択可能に） |
| `model_reasoning_summary` | string | `"auto"` | 推論サマリー（`auto` / `concise` / `detailed` / `none`） |
| `log_dir` | string | - | ログ出力先ディレクトリ |
| `model_context_window` | integer | - | 利用可能なコンテキストトークン数 |
| `forced_login_method` | string | - | 認証方式の制限（ChatGPT / API） |
| `forced_chatgpt_workspace_id` | string | - | 特定ワークスペースへのログイン制限 |
| `cli_auth_credentials_store` | string | - | 認証情報ストア（`file` / OS keychain） |
| `default_permissions` | string | - | デフォルト権限プロファイル名 |
| `file_opener` | string | - | citation用URIスキーム（`vscode`, `cursor`, `windsurf` 等） |
| `check_for_update_on_startup` | bool | `true` | 起動時アップデートチェック |
| `allow_login_shell` | bool | `true` | ログインシェルセマンティクス許可 |

#### プロジェクトドキュメント設定

| キー | 型 | デフォルト | 説明 |
|------|------|-----------|------|
| `project_doc_max_bytes` | integer | `32768` | AGENTS.md の最大読み込みサイズ（バイト） |
| `project_doc_fallback_filenames` | array | - | フォールバックファイル名（例: `["TEAM_GUIDE.md", ".agents.md"]`） |
| `project_root_markers` | array | `[".git", ".hg"]` | プロジェクトルート検出マーカー |

#### 履歴設定

```toml
[history]
persistence = "default"   # "default" / "none"
max_bytes = 1048576       # 最大サイズ（バイト）、超過時は自動コンパクション
```

#### 機能フラグ

```toml
[features]
shell_snapshot = true                    # コマンド実行の高速化
smart_approvals = false                  # ガーディアンレビュアー経由の承認
multi_agent = true                       # サブエージェント機能
unified_exec = true                      # PTY実行（stable、Windows除く）
undo = false                             # Undo 機能
web_search = true                        # Web 検索
skill_mcp_dependency_install = true      # スキル依存MCPサーバーの自動インストール（stable）
codex_hooks = true                       # Hooks システム（0.124.0+ デフォルト有効）
plugin_hooks = true                      # プラグインバンドル hooks（0.131.0+ デフォルト有効）
network_proxy = false                    # ネットワークプロキシ（0.131.0+ 段階的ロールアウト）
```

> 0.131.0+: 設定スキーマ外フィールドは **strict config parsing** により拒否される。旧来 unknown フィールドが警告だった挙動から変更されている点に注意。

#### Agent 管理

```toml
[agents]
max_threads = 6              # 最大同時エージェントスレッド数
max_depth = 3                # エージェントネスト最大深度

[agents.code-reviewer]
description = "コードレビュー担当"   # ロールガイダンス
```

#### UI 設定

```toml
[tui]
theme = "monokai"            # シンタックスハイライトテーマ
animations = true            # ターミナルアニメーション
notifications = true         # 通知の有効化
status_line = ["model", "context", "git"]  # フッターステータスライン項目
```

#### 承認粒度（Granular Approval）

```toml
[approval_policy]
granular.sandbox_approval = true        # サンドボックスエスカレーション表示
granular.rules = true                   # execpolicy プロンプトトリガー承認
granular.mcp_elicitations = true        # MCP elicitation プロンプト表示
granular.request_permissions = true     # request_permissions ツール承認
granular.skill_approval = true          # スキルスクリプト承認
```

#### Windows 固有設定

```toml
[windows]
sandbox = "elevated"       # "elevated"（推奨、管理者権限必要） / "unelevated"
```

#### 0.140〜0.144 の注目デフォルト変更・設定関連機能（2026-06〜07）

- **MCP tool search がデフォルト化（0.142.2 / 0.143.0）**: 対応モデルでは MCP ツールが tool search 経由でオンデマンド発見される（旧モデル・プロバイダとの互換性は維持）
- **リモートプラグインがデフォルト有効化（0.143.0）**: npm マーケットプレースソース、リモート/ローカルバージョン表示付きカタログ
- **システムプロキシ対応（0.142.1 / 0.142.2 / 0.143.0）**: 認証・Responses API トラフィックが macOS / Windows のシステムプロキシ（PAC / WPAD 含む）を経由可能（`respect_system_proxy`）
- **rollout token budgets（0.142.0）**: エージェントスレッド横断のトークン予算を設定可能。残量リマインダーと枯渇時のターン中断
- **multi-agent delegation の3段階制御（0.142.0）**: app-server クライアントがスレッド/ターン単位で disabled / explicit-request-only / proactive を設定
- **writes app-approval mode（0.144.0）**: 宣言済み read-only アクションは許可し書き込みのみ承認を求めるアプリ承認モード
- **MCP 対話型認証の標準化（0.144.0）**: MCP ツールが実験的オプトインなしに対話的な認証を要求可能
- **認証情報の暗号化ローカル保存（0.140.0）**: CLI / MCP OAuth 認証情報の暗号化保存、managed Amazon Bedrock APIキー認証
- **Ultra reasoning 選択時の警告（0.144.0）**: multi-agent 並列度が高い場合に使用量急増を警告
- **SDK の config オーバーライド（0.149.0）**: SDK から CLI の config オーバーライドをそのまま渡せるようになり、推論努力度 `max` / `ultra` も選択可能
- **スキルカタログのトークンバジェット（0.149.0）**: スキル一覧がコンテキストを圧迫しないよう、カタログのトークンバジェットを設定可能

> 公式ドキュメント: [Config Basics](https://developers.openai.com/codex/config-basic) / [Config Reference](https://developers.openai.com/codex/config-reference) / [Sample Config](https://developers.openai.com/codex/config-sample)

---

#### Amazon Bedrock Runtime プロバイダ（0.148.0+）

`model_provider = "amazon-bedrock-runtime"` で、リージョン別 `bedrock-runtime` の OpenAI 互換エンドポイントを組み込みプロバイダとして利用できる。

- 認証はベアラートークンを維持しつつ、エンドポイント固有の SigV4 サービス設定を使用
- AWS プロファイル / リージョン / トランスポートはプロバイダ単位で上書き可能
- GPT-5.6 は **global 版**と **US クロスリージョン版**を提供。フォールバックおよびバックグラウンドタスクは global ルーティングを優先
- 当プロバイダでは **Web 検索は非対応**（`web_search` 設定に関わらず送信されない）

> 0.147.0 で入った Bedrock 向けのキャッシュ済み Web 検索・リモート会話コンパクションは、managed Bedrock APIキー認証（0.140.0）経路の話であり、本組み込みプロバイダとは別枠。

---

## 2. AGENTS.md

Codex CLI に対するカスタム指示を Markdown 形式で記述するファイル。

### 2.1 3レベル階層と検索順序

1. **グローバルスコープ**（`~/.codex/`）
   - `AGENTS.override.md` → `AGENTS.md` の順に検索
2. **プロジェクトスコープ**（Git ルート → CWD まで各ディレクトリ）
   - 各ディレクトリで `AGENTS.override.md` → `AGENTS.md` → フォールバックファイル名の順に検索
3. **マージ順序**: ルートから CWD に向かって連結。より近いファイルが先の指示を上書き

### 2.2 override の動作

- 同一ディレクトリでは `AGENTS.override.md` が `AGENTS.md` より優先
- ネストされた override はより広いスコープのルールを置換
- グローバル override はグローバルベースファイルを完全に抑制
- `CODEX_HOME` 環境変数でホームディレクトリをリダイレクト可能

### 2.3 サイズ制限

- **デフォルト最大サイズ**: 32 KiB（`project_doc_max_bytes` で変更可能）
- 上限に達した時点でファイルの追加読み込みを停止
- **1ディレクトリにつき最大1ファイル**
- 空ファイルは自動スキップ

### 2.4 無効化

```bash
codex --no-project-doc "タスク"
# または
CODEX_DISABLE_PROJECT_DOC=1 codex "タスク"
```

### 2.5 確認コマンド

```bash
# 現在読み込まれている指示を確認
codex --ask-for-approval never "現在の指示をまとめてください"

# サブディレクトリで有効なファイルを確認
codex --cd subdir --ask-for-approval never "有効な指示ファイルを表示"
```

> 公式ドキュメント: [AGENTS.md Guide](https://developers.openai.com/codex/guides/agents-md)

---

## 3. プロファイル

名前付きの設定セットを定義し、ワークフローに応じて切り替え可能。

### 3.1 定義方法

```toml
# ~/.codex/config.toml

# デフォルトプロファイルの指定（任意）
profile = "default"

[profiles.default]
model = "o4-mini"
approval_policy = "on-request"

[profiles.fast]
model = "gpt-5.4"
service_tier = "fast"
approval_policy = "on-request"

[profiles.autonomous]
model = "gpt-5.4"
approval_policy = "never"
sandbox_mode = "workspace-write"
```

### 3.3 profile-v2（レイヤー化プロファイル, 0.131.0+ / 0.134.0 で `--profile` に統一）

複数の TOML ファイルを重ね合わせて適用するレイヤー化プロファイル。

**0.134.0 以降**: `--profile <name>` がプライマリプロファイルセレクタとして CLI / TUI permissions / sandbox / mcp / app-server 全体で正式採用。`codex sandbox --profile`、`codex mcp --profile` も指定可能。legacy `[profiles]` テーブル（v1）は migration guidance 付きで拒否される。

```
~/.codex/profiles.d/
├── 10-base.toml
├── 20-fast.toml
└── 30-autonomous.toml
```

- 数値プレフィックスで重ね順を制御（小さい順に適用、大きい順が優先）
- 0.131.0〜0.133.0 では `--profile-v2 <name>` フラグだったが、0.134.0 で `--profile` に統合
- 旧来の `[profiles]` テーブル（v1）は 0.134.0 で plumbing が撤去され、設定すると拒否される
- 主用途: チーム共有プロファイル（ベース）+ 個人オーバーレイ（チューニング）の階層管理

### 3.2 使用方法

```bash
codex --profile fast "コードをリファクタリングして"
codex -p autonomous "テストを全部実行して修正して"
```

---

## 4. 環境変数

### 4.1 認証関連

| 環境変数 | 説明 |
|----------|------|
| `OPENAI_API_KEY` | OpenAI API キー |
| `AZURE_OPENAI_API_KEY` | Azure OpenAI API キー |
| `AZURE_OPENAI_API_VERSION` | Azure API バージョン |
| `OPENROUTER_API_KEY` | OpenRouter API キー |
| `GEMINI_API_KEY` | Gemini API キー |
| `MISTRAL_API_KEY` | Mistral API キー |
| `DEEPSEEK_API_KEY` | DeepSeek API キー |
| `XAI_API_KEY` | xAI API キー |
| `GROQ_API_KEY` | Groq API キー |

### 4.2 動作制御

| 環境変数 | 説明 |
|----------|------|
| `CODEX_HOME` | Codex ホームディレクトリの変更（デフォルト: `~/.codex`） |
| `CODEX_DISABLE_PROJECT_DOC` | `1` に設定すると AGENTS.md を無効化 |
| `CODEX_QUIET_MODE` | `1` に設定するとパイプライン向けサイレントモード |
| `DEBUG` | `true` で詳細ログを有効化 |

### 4.3 シェル環境ポリシー

サブプロセスに渡す環境変数を制御する。

```toml
[shell_environment_policy]
inherit = "core"             # "none" / "core"
include_only = ["PATH", "HOME", "LANG"]
exclude = ["SECRET_*"]       # glob パターン対応
set = { NODE_ENV = "development" }
```

---

## 5. サンドボックスと承認ポリシー

Codex CLI は **OS レベルのサンドボックス** と **承認ポリシー** の2層で安全性を確保する。

### 5.1 サンドボックスモード (`sandbox_mode`)

| モード | ファイルシステム | ネットワーク |
|--------|-----------------|-------------|
| `read-only` | 読み取りのみ | 無効 |
| `workspace-write` | CWD + `/tmp` に書き込み可。`.git`, `.agents/`, `.codex/` は読み取り専用のまま保護 | デフォルト無効 |
| `danger-full-access` | 制限なし | 制限なし |

OS ごとの実装:
- **macOS**: Apple Seatbelt (`sandbox-exec`)
- **Linux**: bubblewrap + seccomp
- **Windows**: WSL またはネイティブサンドボックス

ネットワークアクセスを選択的に有効化:

```toml
[sandbox_workspace_write]
network_access = true
```

### 5.2 承認ポリシー (`approval_policy`)

| ポリシー | 動作 |
|----------|------|
| `untrusted` | 安全な操作は自動実行。状態変更コマンドには承認が必要 |
| `on-request` | ファイル書き込みとシェルコマンドに承認が必要（デフォルト） |
| `never` | 承認プロンプトなし（危険） |

CLI フラグでの指定:

```bash
codex --ask-for-approval untrusted "タスク"
codex -a never "タスク"
codex --full-auto "タスク"                # on-request + 低摩擦モード
codex --yolo "タスク"                     # 承認・サンドボックス完全バイパス（非推奨）
```

### 5.3 権限プロファイル

名前付きのファイルシステム・ネットワークアクセスプロファイルを定義可能。

```toml
default_permissions = "restricted"

[permissions.restricted]
# ファイルシステムとネットワークのアクセスルールを定義
# deny-read グロブで秘密情報を含むパスを読み取り禁止に指定可能（0.122.0+）
# 管理対象 deny-read は強制要件として設定可能
```

隔離 `codex exec` 実行時はユーザー設定の permission プロファイルをバイパスし、ジョブ固有のサンドボックス境界のみ適用される（0.122.0+）。

0.125.0 以降、permission プロファイルは TUI セッション、ユーザーターン、MCP サンドボックス状態、シェルエスカレーション、app-server API を横断して永続化される（プロセスや接続をまたいで設定が保持される）。

### 5.4 OpenTelemetry 監視（オプトイン）

```toml
[otel]
environment = "staging"
exporter = "otlp-http"       # "otlp-http" / "otlp-grpc" / "none"
log_user_prompt = false       # プロンプトのログ記録（デフォルト無効）
```

追跡対象イベント: API リクエスト、ツール呼び出し、ユーザープロンプト（リダクト済み）、承認操作

> 公式ドキュメント: [Agent Approvals & Security](https://developers.openai.com/codex/agent-approvals-security) / [Config Advanced](https://developers.openai.com/codex/config-advanced)

---

## 6. Skills

Codex CLI は Skills システムをサポートする（stable）。Claude Code の Skills と同じ SKILL.md フォーマットを採用している。

### 6.1 Skills の配置場所

> **重要**: Skills のパスが `.codex/skills/` から `.agents/skills/` に変更された（2026-03時点）。

| レベル | パス | 用途 |
|--------|------|------|
| プロジェクト (CWD) | `$CWD/.agents/skills/` | ワーキングフォルダのチームスキル |
| プロジェクト (親) | `$CWD/../.agents/skills/` | リポジトリ内ネストフォルダスキル |
| プロジェクト (ルート) | `$REPO_ROOT/.agents/skills/` | リポジトリルートの組織スキル |
| ユーザー | `$HOME/.agents/skills/` | 全リポジトリ横断の個人スキル |
| 管理者 | `/etc/codex/skills/` | マシン/コンテナ用システムスキル |
| システム | バンドル | デフォルトOpenAIスキル |

CWD からリポジトリルートまで走査。同名スキルは両方表示（マージなし）。シンボリックリンク対応。

### 6.2 SKILL.md フォーマット

Claude Code と同一の SKILL.md フロントマター形式を使用する。YAML フロントマターでメタデータを定義し、本文に指示を記述する。

### 6.3 バンドルスキル

Codex CLI には以下のスキルが組み込まれている:

| スキル名 | 説明 |
|----------|------|
| `skill-creator` | 新しいスキルの作成を支援 |
| `skill-installer` | 外部スキルのインストール |
| `openai-docs` | OpenAI ドキュメント参照 |

### 6.4 スキルの起動方法

- `/skills` コマンドでスキル一覧を表示・選択
- `$` メンションで直接スキルを参照
- 暗黙マッチング（プロンプト内容に基づき自動選択）

### 6.5 `agents/openai.yaml` メタデータ（任意）

スキルディレクトリに `agents/openai.yaml` を配置して UI 表示やポリシーを設定可能:

```yaml
interface:
  display_name: "表示名"
  short_description: "ユーザー向け説明"
  icon_small: "./assets/small-logo.svg"
  icon_large: "./assets/large-logo.png"
  brand_color: "#3B82F6"
  default_prompt: "デフォルトプロンプト"

policy:
  allow_implicit_invocation: false  # false で明示的 $skill 呼び出しのみに制限

dependencies:
  tools:
    - type: "mcp"
      value: "openaiDeveloperDocs"
      description: "OpenAI Docs MCP server"
      transport: "streamable_http"
      url: "https://developers.openai.com/mcp"
```

### 6.6 スキルインストーラー

```
$skill-installer linear
```

外部リポジトリからスキルをダウンロード。自動検出されるが、反映されない場合は再起動。

### 6.7 スキルの有効化/無効化

```toml
[[skills.config]]
path = "/path/to/skill/SKILL.md"
enabled = false
```

### 6.8 関連する機能フラグ

```toml
[features]
skill_mcp_dependency_install = true   # stable — スキルが依存するMCPサーバーの自動インストール
```

### 6.9 スキルのみのプラグイン化（skill-only plugin）

スキルを個人利用を超えて配布したくなった段階で、**プラグイン**にパッケージする。プラグインは skill / MCP サーバー / その両方を含む配布単位で、ChatGPT と Codex は**単一の共通プラグインディレクトリ**を共有する（公開すれば両製品から発見可能）。

使い分けの目安（公式ガイダンス）:

| 状況 | 選択 |
|------|------|
| 個人ワークフローを試行錯誤中 | 素のスキル（`.agents/skills/`）のまま |
| チーム共有・関連スキルの束ね・外部サービス接続・安定した機能の配布 | プラグイン化 |

#### `@plugin-creator` を使う方法（推奨）

ChatGPT Work モードでは `@plugin-creator`、Codex では `$plugin-creator` を呼ぶ。`.codex-plugin/plugin.json` マニフェストの生成、フォルダ構成、ローカルマーケットプレースへの登録までを代行する。

生成後の確認手順:
1. `.codex-plugin/plugin.json` をレビュー
2. `skills/` 配下の各スキルを確認
3. ChatGPT / Codex を再読み込みし、ローカルマーケットプレースソースからインストール
4. 新規会話で代表的なリクエストを投げてテスト

MCP サーバーを含む場合は、先にサーバーを構築・テストし、登録済みの接続情報を `@plugin-creator` に渡す。

#### 手動で作る最小構成

```text
meeting-follow-up/
├── .codex-plugin/
│   └── plugin.json
└── skills/
    └── meeting-follow-up/
        └── SKILL.md
```

`.codex-plugin/plugin.json`:

```json
{
  "name": "meeting-follow-up",
  "version": "1.0.0",
  "description": "Turn meeting notes into decisions and next steps",
  "skills": "./skills/"
}
```

- プラグイン名は**安定した kebab-case** を使う
- スキルの `description` は、ChatGPT / Codex がワークフロー該当を判定できる程度に具体的に書く
- 開発中は**ローカルマーケットプレース**でテストしてから共通ディレクトリへ提出する

> 公式: [Build plugins](https://learn.chatgpt.com/docs/build-plugins) / 完全な builder ドキュメントは [developers.openai.com/plugins](https://developers.openai.com/plugins)
>
> 注意: GitHub の公開サンプルカタログ [openai/skills](https://github.com/openai/skills)（deprecated）と [openai/plugins](https://github.com/openai/plugins)（**2026-08-16 に archive、read-only**）はいずれも更新停止。作成手順の一次情報は上記ドキュメントと `@plugin-creator` を参照する。

---

## 7. Hooks

Codex CLI の Hooks は **0.124.0 で正式化（stable）** され、0.148.0 で**非同期実行**と **MCP ツールハンドラ**に対応した。イベント数・ハンドラ種別ともに Claude Code へ大きく近づいている。

### 7.1 有効化

0.124.0 以降はデフォルトで有効。旧バージョン（0.123.0 以前）では以下のフィーチャーフラグが必要:

```bash
# 0.123.0 以前のみ必要
codex --enable codex_hooks
# または
[features]
codex_hooks = true
```

### 7.2 定義場所

以下の各レイヤーが**合成**される（上位が下位を置き換えるのではなく足し合わせ）:

| レイヤー | ファイル |
|----------|----------|
| ユーザー | `~/.codex/hooks.json` または `~/.codex/config.toml` |
| プロジェクト | `<repo>/.codex/hooks.json` または `<repo>/.codex/config.toml` |
| プラグイン | プラグイン同梱の `hooks/hooks.json` またはマニフェストのエントリ |
| Managed | `requirements.toml` / managed config レイヤー（`managed_dir` / `windows_managed_dir` で配置先指定可） |

> 同一ディレクトリに `hooks.json` と `config.toml` の両方の hooks 定義があると警告が出る。どちらか一方に寄せること。
>
> Managed のみ: `requirements.toml` に `allow_managed_hooks_only = true` を置くと、user / project / session の hook 設定を無視し managed hooks のみを実行する。**`config.toml` に書いても効かない**。

### 7.3 設定形式

イベント名 → マッチャーグループ配列 → ハンドラ配列、という3階層構造。JSON 形式:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "shell",
        "hooks": [
          { "type": "command", "command": "python3 scripts/guard.py", "timeout": 30 }
        ]
      }
    ]
  }
}
```

TOML 形式（`config.toml`）では同じ構造を array-of-tables で書く:

```toml
[[hooks.PreToolUse]]
matcher = "shell"

  [[hooks.PreToolUse.hooks]]
  type = "command"
  command = "python3 scripts/guard.py"
  timeout = 30

[[hooks.SessionStart]]

  [[hooks.SessionStart.hooks]]
  type = "command"
  command = "bash scripts/preflight.sh"
  command_windows = "powershell -ExecutionPolicy Bypass -File scripts/preflight.ps1"
```

`matcher` は正規表現文字列。省略または `"*"` で全件マッチ。マッチ対象はイベントによって異なる:

- ツール系イベント（`PreToolUse` / `PostToolUse` / `PermissionRequest`）: ツール名
- `PreCompact` / `PostCompact`: 発動契機（`manual` / `auto`）
- `SessionStart`: セッションの起点（`startup` / `resume` / `clear` / `compact`）

### 7.4 対応イベント（0.148.0 時点で 11 種）

| イベント | スコープ | 説明 |
|----------|----------|------|
| `PreToolUse` | ターン | ツール実行前。ブロック可能 |
| `PermissionRequest` | ターン | 権限確認の発生時 |
| `PostToolUse` | ターン | ツール実行後 |
| `PreCompact` | ターン | コンテキスト要約の前 |
| `PostCompact` | ターン | コンテキスト要約の後 |
| `UserPromptSubmit` | ターン | ユーザーのプロンプト送信時。ブロック可能 |
| `SubagentStop` | ターン | サブエージェント終了時（0.133.0+） |
| `Stop` | ターン | ターン終了時 |
| `SessionStart` | セッション | セッション開始時 |
| `SessionEnd` | セッション | セッション終了時 |
| `SubagentStart` | セッション | サブエージェント起動時（0.133.0+） |

> **比較**: Claude Code は 17 以上のイベントをサポート。Codex は 0.133.0 で 6 → 0.148.0 時点で 11 イベント。`Notification` 系や `PreCompact` 以外の UI 系イベントは引き続き未対応。
>
> 別枠として `MITM` hook（0.133.0+）がランタイム enforcement 用に存在する（named MITM permissions config と組み合わせて使用）。

### 7.5 ハンドラ種別

| `type` | Codex CLI | Claude Code |
|--------|-----------|-------------|
| `command` | サポート | サポート |
| `mcp_tool` | **0.148.0 でサポート**（`SessionEnd` と MCP 呼び出し非対応ランタイムは起動時警告付きでスキップ） | 相当機能なし |
| `prompt` | 設定は受理されるが**未実装**（スキップ警告） | サポート |
| `agent` | 設定は受理されるが**未実装**（スキップ警告） | サポート |
| HTTP | **非サポート** | サポート |

**`command` ハンドラのフィールド**:

| フィールド | 説明 |
|-----------|------|
| `command` | 実行コマンド（必須） |
| `commandWindows` / `command_windows` | Windows でのみ使う代替コマンド（0.131.0+） |
| `timeout` | タイムアウト秒（既定 600。`SessionEnd` のみ既定 1） |
| `async` | `true` でバックグラウンド実行（**0.148.0+**）。詳細は 7.6 |
| `statusMessage` | 実行中に UI へ出す文言 |
| `additionalContextLimit` | `additionalContext` をディスクへ退避する概算トークン閾値（既定 2,500。`0` で退避無効） |

**`mcp_tool` ハンドラのフィールド**（0.148.0+）:

| フィールド | 説明 |
|-----------|------|
| `server` | 呼び出す MCP サーバー名（必須） |
| `tool` | 呼び出すツール名（必須） |
| `input` | ツールへ渡す JSON オブジェクト。フックイベントのプレースホルダがネストしたまま JSON 型を保って展開される。**TOML で表現できない値（`null` 等）は拒否**される（trust hash のため） |
| `timeout` | タイムアウト秒（既定 600） |
| `statusMessage` | 実行中に UI へ出す文言 |

`hooks/list` およびハンドラ名は TUI の hooks ブラウザに `server` / `tool` 付きで表示される。

### 7.6 非同期 command hook（0.148.0+）

`async = true` を指定した `command` ハンドラはバックグラウンドで走る。

- セッション単位の同時実行上限あり
- **起動元の操作をブロック・停止・書き換え・制御できない**（同期フックのような decision 制御は効かない）
- 警告や `additionalContext` は安全なターン境界で配送される: 実行中のターンにはサンプリング後に注入、アイドル時は次のユーザープロンプトの直前までバッファされる
- 設定リロードをまたいで実行中のフックは維持され、セッション終了時に中断される
- **`SessionEnd` は常に同期**（`async` 指定は無効）

> ハーネス設計上の使い分け: ガード（実行を止めたい）は同期、通知・計測・ログ送信など**結果でセッションを制御しない**用途は `async = true`。

### 7.7 終了コード

| 終了コード | 動作 |
|------------|------|
| `0` | 成功。処理を続行 |
| `2` | ブロック（`UserPromptSubmit` / `PreToolUse` 等のブロック可能イベント）。当該操作をキャンセル |
| その他 | エラーとして扱われる |

### 7.8 Hook trust の意図的バイパス（0.131.0+）

CI 等で hook trust 確認をスキップしたい場合は CLI フラグ `--dangerously-bypass-hook-trust` を使用。インタラクティブな信頼確認をスキップするため、信頼された CI 環境以外では使用しないこと。フックごとの信頼状態は `[hooks.state.<key>]` の `enabled` / `trusted_hash` で保持される。

### 7.9 プラグイン Hooks（0.131.0+ デフォルト有効）

プラグインがバンドルする hooks が、明示的に opt-out しない限りデフォルトで有効化される。プラグインインストール時に hook 一覧が UI に表示され、信頼判断に利用される。
