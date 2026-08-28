# Claude Code MCP 仕様書

最終更新: 2026-07-13（巡回更新）

公式ドキュメント: https://code.claude.com/docs/en/mcp

---

## 1. MCP の概要

**Model Context Protocol (MCP)** は AI ツールと外部データソースを接続するためのオープンソース標準。Claude Code は MCP サーバーを通じて数百の外部ツール・データベース・API に接続できる。

### 1.1 できること

- **イシュートラッカーからの機能実装**: 「JIRA の ENG-4521 に記載された機能を実装して PR を作成」
- **モニタリングデータの分析**: 「Sentry と Statsig を確認」
- **データベースクエリ**: 「PostgreSQL から該当ユーザーを検索」
- **デザイン統合**: 「Figma のデザインに基づいてテンプレートを更新」
- **ワークフロー自動化**: 「Gmail の下書きを作成」
- **外部イベントへの反応**: チャンネル機能で Telegram / Discord / webhook イベントを受信

---

## 2. トランスポートタイプ

| タイプ | 説明 | 推奨度 |
|:--|:--|:--|
| **HTTP** (streamable-http) | リモートHTTPサーバー。クラウドサービス向け推奨 | 推奨 |
| **SSE** (Server-Sent Events) | リモートSSEサーバー。非推奨 | 非推奨（HTTP を使用） |
| **stdio** | ローカルプロセス。直接システムアクセスが必要な場合 | ローカル用 |
| **WebSocket** (`ws`) | WebSocket接続 | 特定用途 |

---

## 3. 設定方法

### 3.1 CLI による追加

#### HTTP サーバー（推奨）

```bash
# 基本構文
claude mcp add --transport http <name> <url>

# 例: Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Bearer トークン付き
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

#### SSE サーバー（非推奨）

```bash
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 認証ヘッダー付き
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: your-key-here"
```

#### stdio サーバー

```bash
# 基本構文
claude mcp add [options] <name> -- <command> [args...]

# 例: Airtable
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server
```

**重要**: オプション（`--transport`, `--env`, `--scope`, `--header`）はサーバー名の**前**に配置。`--` でサーバー名とコマンド/引数を分離。

#### Windows での注意

Windows（WSL除く）で `npx` を使用するローカルサーバーは `cmd /c` ラッパーが必要:

```bash
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### 3.2 管理コマンド

```bash
# サーバー一覧
claude mcp list

# サーバー詳細
claude mcp get github

# サーバー削除
claude mcp remove github

# プロジェクトMCPの承認リセット
claude mcp reset-project-choices

# CLIからのMCP認証（v2.1.186。--no-browser でSSH越しのstdinリダイレクト完結も可能）
claude mcp login <name>
claude mcp logout <name>

# セッション内でステータス確認・OAuth認証
/mcp
```

v2.1.154: `claude mcp list` / `get` の出力がパイプされた場合、未承認の `.mcp.json` サーバーは自動承認・接続せず `⏸ Pending approval` 表示となる。Stdio MCP サーバーサブプロセスには `CLAUDE_CODE_SESSION_ID` と `CLAUDECODE=1` 環境変数が渡される。v2.1.163 で `--resume` パスでも hooks / Bash と同じ `CLAUDE_CODE_SESSION_ID` が stdio MCP サーバーに一貫して渡るよう修正。

**v2.1.161 のシークレット保護強化**: `claude mcp list` / `get` / `add` がサーバー定義を出力する際、`${VAR}` 環境変数参照は展開されずそのまま表示される。さらに認証ヘッダー（`Authorization` 等）や URL に埋め込まれたシークレットは redact 表示される。CI ログや支援用 paste でのシークレット漏洩を防止。

**v2.1.196 のセキュリティ修正**: `claude mcp list` / `get` は、リポジトリがコミット済み `.claude/settings.json` で自己承認した `.mcp.json` サーバーをスポーンしなくなった。未信頼ワークスペースでは `⏸ Pending approval` 表示。

**接続を試みずに表示される設定ステータス**（v2.1.247 時点）:

| 表示 | 意味 | どこに出るか |
|:--|:--|:--|
| `⏸ Pending approval (run `claude` to approve)` | 未承認の `.mcp.json` プロジェクトスコープサーバー。`claude` を対話起動して承認する | `claude mcp list` と `get` |
| `✘ Rejected (see disabledMcpjsonServers in settings)` | `disabledMcpjsonServers` で拒否されている | `claude mcp get <name>` のみ |
| `⊘ Disabled for this project (re-enable via /mcp)` | プロジェクトの `disabledMcpServers` に列挙されている。`/mcp` パネルから再有効化 | `claude mcp list` と `get` |

> v2.1.238 より前は、無効化済みサーバーにもヘルスチェックのため接続しに行き接続結果を報告していた。

**MCP 設定の警告（4 種）**:

- **前後の空白**: `command` / `url` / `args` 各要素 / `env`・`headers` の値とキー名に前後の空白があると警告（`Leading or trailing whitespace in: headers.Authorization` のように値は伏せてフィールド名だけ表示）。Claude Code はトリムせずそのまま使うので設定側を直す
- **複数スコープでの同名サーバー**: 異なるエンドポイントで同じサーバー名を複数スコープに定義すると警告。OAuth サインインはエンドポイント単位で保存されるため、別プロジェクトでは再サインインが必要になる。`claude mcp remove <name> --scope <scope>` で整理する。警告中のエンドポイントは `${VAR}` を展開せずそのまま引用される
- **予約名**: `workspace` / `claude-in-chrome` / `computer-use` / `Claude Preview` / `Claude Browser` は組み込みサーバー用に予約済み。定義するとロード時にスキップされリネームを促す警告。`claude mcp add` はエラーで拒否（`Claude Browser` の予約は v2.1.205 以降）
- **未設定の環境変数**: `${VAR}` 参照が未設定かつ `:-default` も無い場合、変数名を挙げて警告した上で `${VAR}` のテキストのまま**サーバーはロードされる**。変数を設定するか `${VAR:-default}` を書く

**`/mcp` からの認証後に接続が失敗した場合**: HTTP ステータスやトランスポートエラーコードに加え、試行した URL の**オリジン**（scheme + host、ポート指定があればポートまで）がメッセージに含まれる。パスとクエリは出ない。オリジンは `${VAR}` 展開後の値。ステータスもエラーコードも無い失敗ではオリジンなしでエラーテキストのみ表示。

**プロジェクトスコープサーバーの承認プロンプトが出せない実行形態**（`claude -p` / Agent SDK / クラウドセッション、および `skipDangerousModePermissionPrompt` 付きの `bypassPermissions` セッション）では、確認なしでロードされる。除外するには:

- `disabledMcpjsonServers` に追加（全権限モードでブロック）
- `--setting-sources` / SDK の `settingSources` でプロジェクト設定ごと除外
- `--strict-mcp-config` で起動し `--mcp-config` で渡したサーバーのみ使う（**ロードしないプロジェクトスコープサーバーの承認待ちをスキップするのは v2.1.246 以降**。それ以前は strict セッションでも承認待ちになりバックグラウンドセッションが起動時に止まっていた）

**Anthropic ホスト型コネクタ**: Microsoft 365 / Gmail / Google Calendar 等は claude.ai が登録したリダイレクト URL しか受け付けないため、Claude Code からのローカル OAuth に非対応。`claude mcp add` や `.mcp.json` で追加したサーバーがこれらを指す場合、`/mcp` や `claude mcp login` でのサインインは `is Anthropic-hosted and doesn't support local OAuth` を返す。`claude mcp remove <name>` で自前エントリを消し、[claude.ai/customize/connectors](https://claude.ai/customize/connectors) で接続すると Claude Code に自動的に現れる。

セッション内 `/mcp` コマンド: v2.1.161 から、未使用の claude.ai connector は「Show unused connectors」行の下に折りたたまれ、有効な接続のみが既定で展開表示される。

### 3.3 JSON ファイルによる設定

#### `.mcp.json`（プロジェクトスコープ）

プロジェクトルートに配置、バージョン管理にコミット:

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

#### 環境変数展開

`.mcp.json` 内で環境変数を展開可能:

- `${VAR}` -- 環境変数 VAR の値
- `${VAR:-default}` -- VAR が未設定時はデフォルト値

展開可能な箇所: `command`, `args`, `env`, `url`, `headers`

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

### 3.4 settings.json 関連キー

| キー | 説明 |
|:--|:--|
| `enableAllProjectMcpServers` | プロジェクト `.mcp.json` の全サーバーを自動承認 |
| `enabledMcpjsonServers` | 承認するサーバーリスト |
| `disabledMcpjsonServers` | 拒否するサーバーリスト |

### 3.5 サーバーごとの追加オプション

| キー | 説明 |
|:--|:--|
| `alwaysLoad` | `true` でそのサーバーのツールを tool-search のディファード化対象から外し常時ロード（v2.1.121） |

### 3.6 接続の堅牢性

- v2.1.121 以降、MCP サーバー起動時に transient error が発生しても最大 3 回まで自動リトライする（従来は接続不能のまま停止）。
- v2.1.121 で SDK の `mcp_authenticate` が `redirectUri` をサポート（カスタムスキーム / claude.ai connector 用）。
- v2.1.187: リモート MCP ツール呼び出しが5分間無応答の場合、無期限ブロックせずエラーで中断（`CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` でオーバーライド可）。
- v2.1.191: capability discovery（`tools/list` 等）と OAuth の discovery / token リクエストが transient エラーで短いバックオフ付きリトライ。ヘッドレス環境の OAuth はブラウザポップアップをスキップし URL 貼り付けプロンプトへ直行。
- v2.1.193: `headersHelper` 認証はツール呼び出しが 401/403 を返すと自動で再実行・再接続。
- v2.1.203: セッションの追加ワーキングディレクトリが MCP `roots/list` に含まれ、変更時は `notifications/roots/list_changed` を送信。
- v2.1.206: `--mcp-config` / `.mcp.json` サーバーの per-server `request_timeout_ms` が新規セッションでも尊重されるよう修正。OAuth トークンリフレッシュ1回失敗での手動再認証要求も解消。

- **再接続・リトライの整理（ドキュメント改訂）**:
  - **セッション中に切断したリモートサーバー**: 指数バックオフで最大 5 回再接続（初回 1 秒、以降倍増）。対話セッションでは `/mcp` に pending 表示、5 回失敗で failed（再認証が必要な場合は「要認証」）となり `/mcp` から手動再試行できる。`claude -p` / Agent SDK でも同じスケジュールで再接続するが `/mcp` パネルは無い
  - **初回接続の失敗**: HTTP / SSE サーバーが transient エラー（5xx・connection refused・タイムアウト）で初回接続に失敗した場合は最大 3 回リトライ。起動時とセッション途中の追加（クラウドセッションが構成から追加するサーバー、Agent SDK の `setMcpServers()` を含む）に適用。**WebSocket サーバーの初回接続**と**認証エラー / not-found** はリトライしない
  - **discovery リクエストの失敗**: 接続成功後の `tools/list` / `prompts/list` / `resources/list` は transient なネットワーク／サーバーエラーで最大 3 回・短いバックオフでリトライ。認証エラー・4xx・リクエストタイムアウトはリトライしない
  - **Claude への通知**: tool search が有効（既定）なら、接続に失敗したサーバー名と接続エラーが Claude に伝えられ（該当ツールが見つからない `ToolSearch` 結果にも含まれる）、Claude が応答内で接続失敗を報告する。tool search 無しの構成では Claude に伝わらない
- **`/cd` によるセッション移動（v2.1.246）**: 移動先ディレクトリの設定が有効化するプラグインの MCP サーバーが接続され、有効でなくなったプラグインのサーバーは切断される（移動後に `/reload-plugins` を実行する必要はない）
- **`headersHelper` の作業ディレクトリ（改訂）**: プロジェクト `.mcp.json` / local スコープのサーバーは**そのサーバーを宣言しているプロジェクトディレクトリ**が基準（従来記載は「Claude Code を起動したディレクトリ」）。プロジェクト内のエージェントファイル / SDK の `mcpServers`・`setMcpServers()` / `--mcp-config` 由来はセッションのプライマリワーキングディレクトリが基準。`/cd` はプライマリワーキングディレクトリから動くサーバーについてのみ作業ディレクトリを移動させる。信頼ダイアログの判定も「宣言元のプロジェクトディレクトリ」基準に変わった

### 3.7 入力スキーマが不正なツールの除外

Claude API はリクエスト中の全ツールの入力スキーマを検査し、1 つでも不正だとリクエスト全体を 400 で拒否する。そのため Claude Code はサーバーのツール読み込み時に API の検査のうち 2 つを自前で実行し、**当該除外を有効化するリモート構成を受け取っているデプロイでは**、失格するツールだけを除外して他のツールを生かす。

- トップレベルのプロパティ名は 1〜64 文字で、ASCII 英数字・`_`・`.`・`-` のみ
- JSON Schema draft 2020-12 のメタスキーマに対して妥当であること（`$schema` 未宣言か draft 2020-12 宣言のスキーマに適用。他方言を宣言したスキーマはこの検査をスキップするがプロパティ名検査は適用）

検査はルートレベル combinator の書き換え後、実際に送信するスキーマに対して行う。除外したツールは理由をサーバーログに記録し、Claude にも「どのツールをなぜ除外したか」が伝えられる。サーバー側でスキーマを直せば次回のツール読み込みで復帰する。

当該リモート構成を受け取っていないデプロイでは、検査自体は行ってログに残すが**ツールはそのまま送信される**ため、含まれるリクエストが「ツールを位置で名指しする 400 エラー」で失敗する。v2.1.216 より前はどのデプロイでも検査していなかった。

---

## 4. MCP インストールスコープ

| スコープ | 保存先 | 用途 | コマンド |
|:--|:--|:--|:--|
| **Local**（デフォルト） | `~/.claude.json` 内のプロジェクトパス配下 | 個人・プロジェクト固有 | `claude mcp add --scope local` |
| **Project** | `.mcp.json`（プロジェクトルート） | チーム共有 | `claude mcp add --scope project` |
| **User** | `~/.claude.json` | 全プロジェクト横断 | `claude mcp add --scope user` |
| **Managed** | `managed-mcp.json`（システムディレクトリ） | 組織全体 | IT配布 |

### 4.1 スコープ優先順位

同名サーバーが複数スコープに存在する場合: Local > Project > User

---

## 5. Managed MCP 設定

組織向けの管理設定:

```json
{
  "allowManagedMcpServersOnly": true,
  "allowedMcpServers": [
    { "serverName": "github" }
  ],
  "deniedMcpServers": [
    { "serverName": "filesystem" }
  ]
}
```

- `allowManagedMcpServersOnly`: Managed 設定のみから許可リストを適用
- `allowedMcpServers`: 許可するMCPサーバーのホワイトリスト
- `deniedMcpServers`: 拒否するMCPサーバーのブラックリスト（許可リストより優先）

`managed-mcp.json` の配置先:
- macOS: `/Library/Application Support/ClaudeCode/`
- Linux/WSL: `/etc/claude-code/`
- Windows: `C:\Program Files\ClaudeCode\`

---

## 6. MCPレジストリ

Anthropic が公開する MCP サーバーレジストリ:

- エンドポイント: `https://api.anthropic.com/mcp-registry/v0/servers`
- ドキュメント: `https://api.anthropic.com/mcp-registry/docs`
- 各サーバーには `worksWith` フィールドで対応プラットフォーム（`claude-code`, `claude-api`, `claude-desktop`）が記載

---

## 7. 主要 MCP サーバー例

> サードパーティ MCP サーバーは Anthropic が正確性やセキュリティを全て検証していない点に注意。

### 7.1 よく使われるサーバー

| サーバー | 用途 | 追加コマンド例 |
|:--|:--|:--|
| **GitHub** | コードレビュー、PR管理 | `claude mcp add --transport http github https://api.githubcopilot.com/mcp/` |
| **Sentry** | エラーモニタリング | `claude mcp add --transport http sentry https://mcp.sentry.dev/mcp` |
| **Notion** | ドキュメント管理 | `claude mcp add --transport http notion https://mcp.notion.com/mcp` |
| **Stripe** | 決済 | `claude mcp add --transport http stripe https://mcp.stripe.com` |
| **PayPal** | 決済 | `claude mcp add --transport http paypal https://mcp.paypal.com/mcp` |
| **HubSpot** | CRM | `claude mcp add --transport http hubspot https://mcp.hubspot.com/anthropic` |
| **Playwright** | ブラウザテスト | `claude mcp add --transport stdio playwright -- npx -y @playwright/mcp@latest` |

### 7.2 その他のサーバー

GitHub で数百以上の MCP サーバーが公開されている: https://github.com/modelcontextprotocol/servers

MCP SDK で独自サーバーを構築可能: https://modelcontextprotocol.io/quickstart/server

---

## 8. 高度な機能

### 8.1 動的ツール更新

MCP `list_changed` 通知により、サーバーが利用可能なツール・プロンプト・リソースを動的に更新可能。再接続不要。

### 8.2 チャンネル（プッシュメッセージ）

MCP サーバーが `claude/channel` ケーパビリティを宣言し、`--channels` フラグでオプトインすると、外部イベント（CI結果、モニタリングアラート、チャットメッセージ等）をセッションにプッシュ可能。

### 8.3 MCP プロンプト

MCPサーバーが公開するプロンプトはコマンドとして表示: `/mcp__<server>__<prompt>`

### 8.4 OAuth 認証

リモートサーバーが OAuth 2.0 認証を要求する場合、`/mcp` コマンドで認証フローを開始できる。

### 8.5 プラグイン提供 MCP サーバー

プラグインが MCP サーバーをバンドル可能:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": { "DB_URL": "${DB_URL}" }
  }
}
```

### 8.6 サブエージェントへの MCP スコープ

サブエージェントのフロントマターで MCP サーバーをスコープ可能（メイン会話のコンテキストを消費しない）:

```yaml
---
name: browser-tester
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
---
```

### 8.7 Claude Code を MCP サーバーとして使用

`claude mcp serve` コマンドで Claude Code 自体を MCP サーバーとして起動可能。他のエージェントやツールから Claude Code の機能を利用できる。

### 8.8 ツール結果サイズのサーバー側オーバーライド（v2.1.91）

MCP サーバーはツール結果の `_meta` フィールドに `anthropic/maxResultSizeChars` アノテーションを設定することで、結果の永続化上限を最大 **500K文字** まで引き上げ可能。DBスキーマや大規模データセットなど、切り詰めると情報が失われるケースで有用。

```json
{
  "content": [{ "type": "text", "text": "..." }],
  "_meta": {
    "anthropic/maxResultSizeChars": 500000
  }
}
```

### 8.9 ツール説明とサーバー指示の上限

MCP ツール説明およびサーバー指示は **2KB** に制限される（v2.1.84）。超過分は切り詰められる。

### 8.9.1 claude.ai コネクタが Claude Code に届く経路（ドキュメント新設）

コネクタを支配する設定はセッションの実行場所によって異なる。claude.ai から自分でコネクタを取得するのは 1 行目だけである点に注意（Desktop の WSL セッションはコネクタ未提供のため表に無い）。

| セッションの実行場所 | コネクタの届き方 | 効く制御 |
|:--|:--|:--|
| ターミナル / VS Code / JetBrains / Agent SDK | **Claude Code 自身が claude.ai から取得** | `disableClaudeAiConnectors`・`ENABLE_CLAUDEAI_MCP_SERVERS`・`allowAllClaudeAiMcps` と managed MCP 設定 |
| クラウドセッション（Claude Code on the web） | リモートホストが渡す | claude.ai の組織設定＋セッションに届く `allowedMcpServers` / `deniedMcpServers`＋実行ホスト上の `managed-mcp.json` |
| Desktop アプリのローカル / SSH セッション | Desktop アプリがインプロセスで配る（`type: "sdk"` サーバーとして登録） | 組織の[コネクタツール制御](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools)の `blocked` エントリのみ |

- `disableClaudeAiConnectors` / `ENABLE_CLAUDEAI_MCP_SERVERS` / `allowAllClaudeAiMcps` は **1 行目にのみ**作用する
- クラウドセッションではセッションプロキシがコネクタ URL を書き換えるため、コネクタ本来の URL 向けに書いた `serverUrl` パターンはマッチしない。実行ホストに `managed-mcp.json` があると（self-hosted runner ホスト等）`allowAllClaudeAiMcps` の有無に関わらず配布コネクタは落とされる
- Desktop のローカル / SSH セッションには MCP 設定も `managed-mcp.json` も届かない。ユーザーは [claude.ai/customize/connectors](https://claude.ai/customize/connectors) で切断する、組織はツールを `blocked` にするか Desktop の Claude Code 自体を止める。`ask` 設定は Claude Code に届かないため通常の権限ルールが適用される。`blocked` は Desktop と claude.ai チャットで共通のため「Desktop のセッションだけ隠してチャットでは使う」はできない
- `allowedMcpServers` / `deniedMcpServers` は宣言場所を問わずブロックする（プラグインのサーバー、`--mcp-config` 渡し、`managed-mcp.json` を含む）。組み込みサーバー（Claude in Chrome、VS Code / JetBrains 起動中の `ide` サーバー、CLI 自身が構成するサーバー）は allowlist の対象外で denylist は適用される。**セッションを起動したアプリが登録するインプロセスの `type: "sdk"` サーバーは両リストの対象外**

### 8.10 claude.ai コネクタの重複排除

ローカル設定と claude.ai コネクタの両方で同じ MCP サーバーが設定されている場合、自動的に重複排除される（v2.1.84）。プラグイン提供の MCP サーバーが組織管理コネクタと重複する場合は、プラグイン側が抑制される。

### 8.11 MCP Tool Search

多数の MCP ツールがある環境で、Claude が最も関連性の高いツールを自動的に見つける機能。

- **仕組み**: 全ツールのメタデータをインデックス化し、タスクに応じて関連ツールを検索
- **設定**: `ENABLE_TOOL_SEARCH` 環境変数で有効化

### 8.12 MCP リソース

MCP サーバーが公開するリソース（ドキュメント、データ等）を `@` メンションで参照可能。

### 8.13 MCP Elicitation（ユーザー入力要求）

MCP サーバーがセッション中にユーザー入力を要求する仕組み:

- **Form モード**: フォームフィールドで構造化入力を要求
- **URL モード**: URLを開いて認証等を完了させる

### 8.14 OAuth 認証の詳細

- **固定コールバックポート**: `--oauth-port` で OAuth コールバックポートを固定
- **事前設定 OAuth 認証情報**: `.mcp.json` にクライアントID/シークレットを記載可能
- **OAuth メタデータディスカバリのオーバーライド**: カスタムメタデータURLの指定
- **RFC 9728 対応**: Protected Resource Metadata ディスカバリに対応（v2.1.85）

---

## 9. Managed MCP の Policy-based Control

サーバー名だけでなく、コマンドやURLでのマッチングも可能:

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": "npx @company/*" },
    { "serverUrl": "https://*.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "filesystem" }
  ]
}
```

マッチングフィールド: `serverName`, `serverCommand`, `serverUrl`

---

## 10. 環境変数

| 変数 | 説明 |
|:--|:--|
| `MCP_TIMEOUT` | MCPサーバー起動タイムアウト（ms）。例: `MCP_TIMEOUT=10000` |
| `MAX_MCP_OUTPUT_TOKENS` | MCPツール出力の警告閾値。デフォルト10,000トークン |
| `ENABLE_TOOL_SEARCH` | MCP Tool Search 機能の有効化 |
| `MCP_CONNECTION_NONBLOCKING` | `true` で `-p` モードのMCP接続待機スキップ。`--mcp-config` サーバー接続は5秒上限（v2.1.89） |
| `CLAUDE_PROJECT_DIR` | stdio MCP サーバーに渡される環境変数（v2.1.139+）。プロジェクトルートの絶対パス。プラグイン設定の `command` 内で `${CLAUDE_PROJECT_DIR}` として参照可能 |
| `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` | 2 分超の MCP ツール呼び出しは自動でバックグラウンドに移行しセッションを維持（v2.1.212）。この変数で閾値変更・無効化 |

---

## 参考リンク

- MCP: https://code.claude.com/docs/en/mcp
- MCP プロトコル仕様: https://modelcontextprotocol.io/introduction
- MCP サーバー一覧: https://github.com/modelcontextprotocol/servers
- チャンネル: https://code.claude.com/docs/en/channels
- Managed MCP: https://code.claude.com/docs/en/permissions#managed-only-settings
