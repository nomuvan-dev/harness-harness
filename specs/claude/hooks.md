# Claude Code Hooks 仕様書

最終更新: 2026-08-30（巡回更新）

公式ドキュメント: https://code.claude.com/docs/en/hooks

---

## 1. 概要

Hooks はユーザー定義のシェルコマンド・HTTPエンドポイント・LLMプロンプト・エージェントを、Claude Code のライフサイクルの特定ポイントで自動実行する仕組み。バリデーション、自動化、カスタムワークフロー制御に使用する。

CLAUDE.md の指示は助言的だが、Hooks は**決定論的**であり確実に実行される。

---

## 2. イベント一覧

### 2.1 メインイベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `SessionStart` | セッション開始/再開 | No | `startup`, `resume`, `clear`, `compact`, `fork`（v2.1.214: フォーク開始時は `resume` ではなく `fork` を報告） |
| `UserPromptSubmit` | ユーザープロンプト送信後、処理前 | Yes | - |
| `UserPromptExpansion` | ユーザーが打ったコマンドがプロンプトへ展開される時（Claude に届く前） | Yes | `command_name`（スキル名 / コマンド名）。matcher 省略で全 prompt 型コマンドに発火 |
| `PreToolUse` | ツール実行前 | Yes | **`EndConversation` 以外の全ツール名**（`Bash` / `PowerShell` / `Edit` / `Write` / `Read` / `Glob` / `Grep` / `Agent` / `Workflow` / `WebFetch` / `WebSearch` / `AskUserQuestion` / `ExitPlanMode` 等の組み込みツールと MCP ツール名） |
| `PermissionRequest` | ツール使用の権限ダイアログ表示時 | Yes | ツール名 |
| `PostToolUse` | ツール成功後 | No | ツール名 |
| `PostToolUseFailure` | ツール失敗後 | No | ツール名 |
| `PostToolBatch` | 並列ツール呼び出しのバッチ全体が解決した後、次のモデル呼び出し前 | Yes | matcher非サポート（全バッチで発火） |
| `Stop` | Claude の応答完了時 | Yes | - |
| `StopFailure` | APIエラー発生時 | No | `rate_limit`, `authentication_failed`, `billing_error`, `invalid_request`, `server_error`, `max_output_tokens`, `unknown` |
| `PermissionDenied` | Auto Mode分類器が拒否した後 | No | ツール名 |
| `SessionEnd` | セッション終了時 | No | `clear`, `resume`, `logout`, `prompt_input_exit`, `other`（**v2.1.234 で `bypass_permissions_disabled` を廃止**。Claude Code は送出しなくなったため matcher から削除すること） |

### 2.2 サブエージェントイベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `SubagentStart` | サブエージェント起動時 | No | エージェントタイプ名 |
| `SubagentStop` | サブエージェント完了時 | Yes | エージェントタイプ名 |
| `TeammateIdle` | チームメイトがアイドル状態になる直前 | Yes | matcherなし |
| `TaskCompleted` | タスク完了マーク時 | Yes | matcherなし |

### 2.3 通知・設定・環境イベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `Notification` | 通知送信時 | No | `permission_prompt`（ツール承認に加え、**サンドボックスコマンドのネットワークリクエスト承認**も対象。ターミナルセッションでは v2.1.246 以降）, `idle_prompt`, `auth_success`, `elicitation_dialog`, `agent_needs_input`, `agent_completed`（バックグラウンドエージェント通知、v2.1.198）, `elicitation_url_dialog`, `elicitation_complete`, `elicitation_response`, `quota_auto_resume_fired` / `quota_auto_resume_stale` / `quota_auto_resume_disabled`（利用上限リセット後の自動継続、v2.1.243 系） |
| `MessageDisplay` | アシスタントメッセージ表示時 | No（transform/hide可） | - | アシスタントメッセージのテキストを変換または非表示にできる（v2.1.152） |
| `ConfigChange` | 設定ファイル変更時 | Yes | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills` |
| `InstructionsLoaded` | CLAUDE.md/rules読み込み時 | No | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact` |
| `CwdChanged` | ワーキングディレクトリ変更時 | No | matcher非サポート（全変更で発火） |
| `DirectoryAdded` | `/add-dir` または SDK `register_repo_root` でセッション途中に作業ディレクトリが登録された後 | No | -（v2.1.219） |
| `FileChanged` | 監視対象ファイルのディスク変更時 | No | ファイル名（basename）例: `.envrc`, `.env` |
| `TaskCreated` | `TaskCreate` ツールでタスク作成時 | Yes | matcherなし |
| `Setup` | セッション開始時（`SessionStart` / `SubagentStart` と同じく最初のプロンプト前）。依存インストール等のセットアップ用 | No | - |

> **`Setup` フックの出力の扱い（ドキュメント改訂）**: `Setup` はブロックできず、終了コードに関わらず処理が続く。**どの終了コードでも `systemMessage` / `continue` / `hookSpecificOutput.additionalContext` などの JSON 出力フィールドは破棄される**（以前は `additionalContext` で Claude のコンテキストへ情報を渡せると記載されていたが、現在は渡せない）。`-p` 実行では `--output-format stream-json --verbose` で起動した場合に限り、stdout / stderr / 終了コードが `hook_response` イベントとして出力に現れる。決定制御の分類も「Context only」から `WorktreeRemove` / `Notification` / `SessionEnd` などと同じ**決定制御なし**へ移動した。

### 2.4 ワークツリー・コンパクションイベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `WorktreeCreate` | ワークツリー作成時 | Yes | - |
| `WorktreeRemove` | ワークツリー削除時 | No | - |
| `PreCompact` | コンパクション前 | No | `manual`, `auto`（入力の `custom_instructions` は `manual` で `/compact` に引数が渡された場合のみ文字列。引数なし・`auto` では `null`。従来は空文字列と記載されていた） |
| `PostCompact` | コンパクション後 | No | `manual`, `auto` |

### 2.5 MCP Elicitation イベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `Elicitation` | MCPサーバーがユーザー入力を要求 | Yes | MCPサーバー名 |
| `ElicitationResult` | ユーザーがMCP入力に回答 | Yes | MCPサーバー名 |

### 2.6 モデル切替イベント（v2.1.251+）

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `PreModelSwitch` | ユーザー／クライアントが要求したモデル切替を適用する前 | **Yes**（切替をキャンセル） | **切替先モデルの正規名**（`[1m]` 接尾辞は無視） |
| `PostModelSwitch` | セッションのモデルが変わった後（Claude Code 自身による変更を含む） | No（既に切替済み） | 同上 |

**matcher の評価対象が特殊**: 他のイベントは stdin の JSON 入力のフィールド（ツールイベントなら `tool_name`）に対して matcher を評価するが、`PreModelSwitch` / `PostModelSwitch` は入力の `to_model` から導出した**正規名**に対して評価する。`opus` のようなエイリアス、日付付きモデルID、Bedrock 等のプロバイダ固有IDはすべて同一の正規名に解決されるため、`claude-opus-5` と書けばそのモデルの全表記をカバーできる。matcher は完全一致名・`|` 区切りリスト（`claude-opus-4-6|claude-opus-5`）・正規表現（`.*opus.*`）のいずれでも書ける。

**正規名が判定できない場合**（LLM ゲートウェイのみが知るカスタムモデルID等）は matcher に関わらず**全 `PreModelSwitch` フックが実行される**。ブロックするフックは matcher だけに頼らず入力の `to_model` を必ず確認すること。

**`PreModelSwitch` が発火する要求**:

- `/model <name>` および `/model` ピッカー
- `Option+P` / `Alt+P` のモデルピッカー
- `/config` の Model 設定
- fast モードのオン（それがセッションのモデルを変える場合）
- Agent SDK ホストまたは Remote Control からの `set_model` 要求、`apply_flag_settings` 要求内のモデル変更

**`PreModelSwitch` が発火しない**: 自動モデルフォールバック、セッション再開時のモデル復元など Claude Code 自身が行う切替。これらは `PostModelSwitch` にのみ届く。

**`PostModelSwitch` が発火する変更**: 上記の明示的切替に加え、自動モデルフォールバック、`opusplan` のような設定によるプランモード出入り、セッション再開時のモデル復元。**フォールバックモデルチェーンの代替モデルが1ターンだけ応答した場合は発火しない**（セッションのモデルが変わらないため）。

**入力フィールド**（`PreModelSwitch` / `PostModelSwitch` 共通、共通入力フィールドに加えて）:

| フィールド | 型 | 説明 |
|:--|:--|:--|
| `from_model` | string | 切替前のモデルID |
| `to_model` | string | 切替後のモデルID。matcher はこの正規名と比較される |
| `requested_model` | string / `null` | 要求が指定した名前（`opus` 等のエイリアス、完全なモデルID、既定モデル要求なら `null`） |
| `source` | string | `"command"`（`/model <name>` / `/config` の Model 設定 / fast モードのオン）、`"picker"`（モデルピッカー）、`"sdk"`（Agent SDK / Remote Control の `set_model` 等）。`PostModelSwitch` ではさらに `"auto"`（自動フォールバック等 Claude Code 自身の変更）と `"resume"`（再開時のモデル復元）が加わる |
| `context_tokens` | number | 次のリクエストがプロンプトとして再送するトークン数（メイン会話の直近応答の input / cache read / cache creation / output の合計）。最初の応答前は `0` |
| `prompt_cache_warm` | boolean | 現行モデルのプロンプトキャッシュがまだ温かい見込みか（＝切替でそれを捨てることになるか） |
| `cache_ttl` | string | このセッションが要求しているプロンプトキャッシュ TTL（`"5m"` / `"1h"`） |
| `estimated_cache_write_usd` | number | `to_model` に `cache_ttl` レートで `context_tokens` を書き込む推定コスト（USD、次の応答分は含まない）。サーバーが全コンテキストを再キャッシュするとは限らないため推定値 |
| `pricing` | string | 上記の算出根拠。`"configured"`（組織の設定単価）／ `"catalog"`（定価）／ `"default"`（`to_model` の価格が不明で既定レートを仮定） |

`PostModelSwitch` では `hook_event_name` が `"PostModelSwitch"` になり、`source` が `"auto"` のとき `requested_model` は `null`、`"resume"` のときは復元された保存済みモデル設定になる。

**`PreModelSwitch` の決定制御**:

- 終了コード 2、またはトップレベル `decision: "block"` で切替をキャンセル
- 細かい制御は `hookSpecificOutput` の `permissionDecision`（`"allow"` / `"deny"` / `"ask"`）と `permissionDecisionReason`。`"defer"` / `updatedInput` / `additionalContext` は**受け付けない**
  - `"allow"`: 切替を進め、キャッシュが温かいときに Claude Code が出す確認をスキップする
  - `"deny"`: 切替をキャンセル。理由がユーザーに表示され、`set_model` 要求ではエラーとして返る
  - `"ask"`: ユーザーに確認を求める。**`"ask"` プロンプトを出せるのは対話セッションの `/model` だけ**。それ以外（`-p` の非対話、`/config`、`set_model`）では `"ask"` は拒否として扱われる
- 複数フックが異なる決定を返した場合の優先順位は `deny` > `ask` > `allow`
- 決定に関わらず `systemMessage` はユーザーに表示されるため、コスト報告フックは `{"systemMessage": "..."}` を返して終了コード 0 にすればよい
- **タイムアウトで応答しないフックは切替をブロックする**（`PreToolUse` ではタイムアウトしたコマンドフックはツール実行を継続させるので挙動が逆）。既定タイムアウトは 30 秒
- `PreModelSwitch` は `command` / `http` / `mcp_tool` ハンドラのみ実行する（`prompt` / `agent` ハンドラは使えない）
- 0 でも 2 でもない終了コードで JSON 決定も出さない場合はブロックせず、stderr を表示して切替を適用する

**`PostModelSwitch` の決定制御**: ブロックできない。終了コード 0 のプレーンテキスト stdout、または JSON の `hookSpecificOutput.additionalContext` が、切替後の次のリクエストで Claude に届く。**次のプロンプト送信から 5 秒以内にフックが完了しない場合、そのリクエストには載らず次のリクエストへ回される**。次のリクエストまでにモデルが複数回変わった場合は、最後の切替先の出力のみが届く。

設定例（Opus 4.6 への切替を拒否）:

```json
{
  "hooks": {
    "PreModelSwitch": [
      {
        "matcher": "claude-opus-4-6",
        "hooks": [
          {
            "type": "command",
            "command": "jq -e '.to_model | test(\"opus-4-6\")' > /dev/null && { echo 'Opus 4.6 is retired for this project.' >&2; exit 2; }; exit 0"
          }
        ]
      }
    ]
  }
}
```

設定例（Opus 系へ切り替わったらモデル固有のガイダンスを注入）:

```json
{
  "hooks": {
    "PostModelSwitch": [
      {
        "matcher": ".*opus.*",
        "hooks": [
          { "type": "command", "command": "echo 'On Opus, delegate implementation work to subagents.'" }
        ]
      }
    ]
  }
}
```

---

## 3. ハンドラタイプ

### 3.1 Command ハンドラ

シェルコマンドを実行。JSON を stdin で受け取り、終了コードと stdout で結果を返す。

```json
{
  "type": "command",
  "command": ".claude/hooks/script.sh",
  "async": false,
  "timeout": 600,
  "shell": "bash"
}
```

- `shell`: 使用シェル。`"bash"`（デフォルト）または `"powershell"`
- `args: string[]`（exec form, v2.1.139+）: シェル解釈なしで直接コマンドを起動。`command` と組み合わせると引数を配列で安全に渡せる。パスプレースホルダーのクォート不要

```json
{
  "type": "command",
  "command": "/usr/bin/python3",
  "args": ["${CLAUDE_PROJECT_DIR}/scripts/validate.py", "$TOOL_NAME"]
}
```

**終了コード**:
- `0`: 成功（stdout の JSON をパース）
- `2`: ブロッキングエラー（stderr がエラーメッセージ）
- その他: 非ブロッキングエラー

> **stdout を JSON と読むかプレーンテキストと読むかの判定（ドキュメント明文化）**: 前後の空白を無視して、
> - **`{` で始まり `}` で終わる**: JSON としてパースする。ただし出力が複数行で各行が単独で JSON としてパースでき、かつどの行も JSON 出力オブジェクトのフィールドを設定していない場合は、全体をプレーンテキストとして扱う。いずれかの行がフィールドを設定している場合は全体を JSON として扱う
> - **`{` で始まるが `}` で終わらない**: プレーンテキストとして扱う
>
> 標準の決定モデルを使うイベントで JSON パースに失敗した場合、**終了コード 2 以外のすべてで非ブロッキングエラー**を報告し、トランスクリプトに `<hook name> hook error` 通知（パースメッセージ付き）を表示する。
>
> **stdout がほとんどのイベントではデバッグログ行きでトランスクリプトに出ない**。例外は `UserPromptSubmit` / `UserPromptExpansion` / `SessionStart` / **`PostModelSwitch`（v2.1.251+）** で、これらはプレーンテキスト stdout を Claude が見られるコンテキストとして追加する。

> セキュリティ（v2.1.207）: プラグインの hooks / monitors / MCP headersHelper で、シェル形式コマンド内の `${user_config.*}` 展開はシェルインジェクション対策として拒否されるようになった。hooks は exec 形式（`args` 配列）または `$CLAUDE_PLUGIN_OPTION_<KEY>` 環境変数を使用する。また v2.1.199 から `SessionStart` / `Setup` / `SubagentStart` フックが exit code 2 で終了した際の stderr がトランスクリプトに表示される。

> **ハンドラの作業ディレクトリ（ドキュメント追記）**: ハンドラはカレントディレクトリで Claude Code の環境を引き継いで実行される。**カレントディレクトリが既に存在しない場合**（別シェルがセッション中に worktree や一時ディレクトリを削除した等）、Claude Code は「セッションを開始したディレクトリ → プロジェクトルート → ホームディレクトリ → システム一時ディレクトリ」の順に、存在する最初のものから command フックを実行し、警告を記録する。

> **`$CLAUDE_MODEL` は存在しない**。フックからモデルを知るには `SessionStart` の `model` フィールド（常に含まれるとは限らない）か、`PreModelSwitch` / `PostModelSwitch` の `from_model` / `to_model` を使う。`$ANTHROPIC_MODEL` はシェルで設定した場合に読めるが、セッション中に `/model` で切り替えても値は変わらない。フックプロセスは親環境を継承するが、Claude Code が全サブプロセスから除去する `OTEL_*` エクスポータ変数は除く。


### 3.2 HTTP ハンドラ

JSON POST リクエストをURLエンドポイントに送信。

```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/validate",
  "timeout": 30,
  "headers": {
    "Authorization": "Bearer $MY_TOKEN"
  },
  "allowedEnvVars": ["MY_TOKEN"]
}
```

**レスポンス処理**:
- `2xx` 空ボディ: 成功
- `2xx` テキストボディ: 成功 + コンテキスト
- `2xx` JSONボディ: 判定としてパース
- 非2xx: 非ブロッキングエラー

### 3.3 Prompt ハンドラ

Claude モデルにプロンプトを送信して評価。

```json
{
  "type": "prompt",
  "prompt": "これを承認すべきか？ $ARGUMENTS",
  "model": "model-name",
  "timeout": 30
}
```

### 3.4 Agent ハンドラ

サブエージェントを起動して条件を検証。

```json
{
  "type": "agent",
  "prompt": "この設定を検証してください。$ARGUMENTS",
  "timeout": 60
}
```

### 3.5 MCP Tool ハンドラ（v2.1.118+）

MCP サーバーのツールを直接呼び出す。シェル経由のラッパースクリプトなしに MCP ツールを発火可能。

```json
{
  "type": "mcp_tool",
  "server": "my-server",
  "tool": "validate_change",
  "timeout": 30
}
```

---

## 4. 設定方法

### 4.1 設定場所

| 場所 | スコープ | 共有可能 |
|:--|:--|:--|
| `~/.claude/settings.json` | 全プロジェクト | No |
| `.claude/settings.json` | プロジェクト | Yes |
| `.claude/settings.local.json` | プロジェクト（個人） | No |
| プラグイン `hooks/hooks.json` | プラグイン有効時 | Yes |
| スキル/エージェントフロントマター | コンポーネントのライフタイム | Yes |
| Managed policy settings | 組織全体 | Yes |

### 4.2 設定構造

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "regex_pattern",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/script.sh",
            "timeout": 600,
            "statusMessage": "処理中..."
          }
        ]
      }
    ]
  }
}
```

### 4.3 フック無効化

```json
{
  "disableAllHooks": true
}
```

Managed フックはユーザー/プロジェクト/ローカル設定からは無効化できない。

---

## 5. Matcher

Matcher はフック発火条件をフィルタリングする正規表現文字列。省略または `"*"` で全対象。

### 5.1 MCPツールマッチング

MCPツールは `mcp__<server>__<tool>` パターンに従う:

```json
{
  "matcher": "mcp__memory__.*"
}
```

### 5.2 複数ツールのマッチング

```json
{
  "matcher": "Edit|Write"
}
```

### 5.3 マッチング挙動の変更（v2.1.191 / v2.1.195）

- **カンマ区切りmatcher修正（v2.1.191）**: `"Bash,PowerShell"` のようなカンマ区切りmatcherが silent に発火しないバグが修正された
- **ハイフン識別子の完全一致化（v2.1.195、破壊的変更）**: `code-reviewer` や `mcp__brave-search` のようなハイフンを含む識別子が部分文字列マッチしてしまう挙動を修正し、**完全一致**になった。ハイフン入りMCPサーバーの全ツールにマッチさせるには `mcp__brave-search__.*` のように明示的な正規表現を使う

---

## 6. 入出力フォーマット

### 6.1 共通入力フィールド

全フックが受け取る共通 JSON:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "EventName"
}
```

サブエージェント内では追加:

```json
{
  "agent_id": "unique-id",
  "agent_type": "AgentName",
  "worktree": "/path/to/worktree"
}
```

- `worktree`: ワークツリー内で実行中の場合、そのパス

v2.1.133 以降、すべてのイベントの入力 JSON に effort level も含まれる:

```json
{
  "effort": { "level": "low" | "medium" | "high" }
}
```

加えて、hooks のコマンドおよび Bash ツールサブプロセスから `$CLAUDE_EFFORT` 環境変数で同じ値が参照可能。

### 6.2 イベント固有の入力フィールド

| イベント | 追加入力フィールド |
|:--|:--|
| `SessionStart` | `source`, `model`, `agent_type`(opt)。**v2.1.251 以降、`source` が `resume` / `fork` かつトランスクリプトに Claude の応答が1件以上ある場合のみ** `seconds_since_last_response`（直近応答からの経過秒）, `context_tokens`（再開後の最初のリクエストが再送するトークン数）, `prompt_cache_likely_expired`（直近応答がプロンプトキャッシュ寿命より古い、または後続のコンパクションがキャッシュ済み会話を置換した場合 `true`）, `estimated_cache_write_usd`（`context_tokens` をセッションのモデルへ書き込む推定 USD、応答分は含まない）が加わる |
| `UserPromptSubmit` | `prompt` |
| `UserPromptExpansion` | `expansion_type` (`slash_command`/`mcp_prompt`), `command_name`, `command_args`, `command_source`, `prompt` |
| `PreToolUse` | `tool_name`, `tool_input`, `tool_use_id` |
| `PermissionRequest` | `tool_name`, `tool_input`, `permission_suggestions`(opt) |

> `PermissionRequest` は **サンドボックスコマンドのネットワークリクエスト**の権限プロンプトでは発火しない（ツール使用の権限要求のみ）。ネットワークリクエスト側のシグナルが必要な場合は `Notification` の `permission_prompt` タイプを使う（ただし約 6 秒の待機後に発火）。
| `PostToolUse` | `tool_name`, `tool_input`, `tool_response`, `tool_use_id`, `duration_ms`（v2.1.119+。権限プロンプトと PreToolUse 時間を除いたツール実行時間） |
| `PostToolUseFailure` | `tool_name`, `tool_input`, `tool_use_id`, `error`, `is_interrupt`, `duration_ms`（v2.1.119+） |
| `PostToolBatch` | `tool_calls`（各要素は `tool_name`, `tool_input`, `tool_use_id`, `tool_response`）。`tool_response` は `PostToolUse` と形が異なりモデルが見る `tool_result` のシリアライズ |
| `PermissionDenied` | `tool_name`, `tool_input`, `tool_use_id`, `reason` |
| `Stop` | `stop_hook_active`, `last_assistant_message`, `background_tasks`, `session_crons`（v2.1.145+） |
| `StopFailure` | `error`, `error_details`, `last_assistant_message` |
| `Notification` | `message`, `title`, `notification_type` |
| `SubagentStart` | `agent_id`, `agent_type` |
| `SubagentStop` | `stop_hook_active`, `agent_id`, `agent_type`, `agent_transcript_path`, `last_assistant_message`, `background_tasks`, `session_crons`（v2.1.145+） |
| `InstructionsLoaded` | `file_path`, `memory_type`, `load_reason`, `globs`(opt), `trigger_file_path`(opt), `parent_file_path`(opt) |
| `CwdChanged` | `cwd` |
| `FileChanged` | `file_path`, `change_type` (`created`/`modified`/`deleted`) |
| `TaskCreated` | `task_id`, `task_subject`, `task_description`(opt), `teammate_name`, `team_name` |
| `ConfigChange` | `source`, `file_path` |
| `WorktreeCreate` | `worktree_path`, `isolation_level` |
| `WorktreeRemove` | `worktree_path` |
| `PreCompact` | `trigger`, `custom_instructions` |
| `PostCompact` | `trigger`, `compact_summary` |
| `PreModelSwitch` / `PostModelSwitch` | `from_model`, `to_model`, `requested_model`, `source`, `context_tokens`, `prompt_cache_warm`, `cache_ttl`, `estimated_cache_write_usd`, `pricing`（§2.6 参照、v2.1.251+） |
| `Elicitation` | `mcp_server_name`, `message`, `mode`(opt), `url`(opt), `elicitation_id`, `requested_schema` |
| `TeammateIdle` | `teammate_name`, `team_name` |
| `TaskCompleted` | `task_id`, `task_subject`, `task_description`(opt), `teammate_name`, `team_name` |

### 6.3 JSON 出力フォーマット

終了コード 0 でパースされる JSON 構造:

```json
{
  "continue": true,
  "stopReason": "message",
  "suppressOutput": false,
  "systemMessage": "warning",
  "decision": "block|allow|deny",
  "reason": "explanation",
  "terminalSequence": "\u0007",
  "hookSpecificOutput": {
    "hookEventName": "EventName",
    "additionalContext": "string",
    "permissionDecision": "allow|deny|ask|defer",
    "permissionDecisionReason": "string",
    "updatedInput": {},
    "updatedPermissions": [],
    "action": "accept|decline|cancel",
    "content": {}
  }
}
```

`terminalSequence`（v2.1.141+）: 制御端末を持たない hook からデスクトップ通知・ウィンドウタイトル・ベル等の端末シーケンスを発行できる。例: `"\u0007"`（ベル）、`"\u001b]0;title\u0007"`（ウィンドウタイトル設定）。バックグラウンド hooks や HTTP hooks から端末 UX を制御する用途に有効。

### 6.4 主要イベントの出力詳細

#### PreToolUse

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow|deny|ask|defer",
    "permissionDecisionReason": "理由",
    "updatedInput": { "command": "npm run safe-lint" },
    "additionalContext": "追加コンテキスト"
  }
}
```

> フックが `"ask"` を返したときの権限プロンプトには、そのフックの出所ラベルが付く。ラベルは **`[settings]`**（settings ファイル由来またはエージェント frontmatter 由来） / **`[plugin:<name>]`**（プラグイン由来） / **`[skill]`**（スキル frontmatter 由来）の 3 種（従来ドキュメントの `[User]` / `[Project]` / `[Plugin]` / `[Local]` から変更）。

#### PostToolUse

ツール出力をフックで書き換える（v2.1.121 以降は MCP 以外の全ツールが対象）:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "updatedToolOutput": "整形済みの差し替え出力"
  }
}
```

`continueOnBlock`（v2.1.139+）: PostToolUse フックがブロック判定を出した際、拒否理由を Claude に戻してターンを継続させる（従来はターン停止）。

```json
{
  "decision": "block",
  "reason": "Lint エラー: 未使用変数あり",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "continueOnBlock": true
  }
}
```

#### Stop

`stop_hook_active` を確認して無限ループを防止:

```json
{
  "decision": "block",
  "reason": "テストが未通過です"
}
```

**v2.1.143+ のブロック上限**: ブロックを繰り返す stop hook が無限ループする問題への安全装置として、**8 連続ブロックでターンが警告とともに自動終了** するように。上限は環境変数 `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` で変更可能。

**v2.1.163+ の `additionalContext` サポート**: `Stop` および `SubagentStop` フックは `hookSpecificOutput.additionalContext` を返すことで、Claude にフィードバックを与えてターンを継続させられる。従来は同等の動作が「hook エラー」扱いだったが、通常応答として扱われるようになった。

```json
{
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "additionalContext": "テストが未通過なので修正を続けてください"
  }
}
```

#### SessionStart

環境変数の永続化に `CLAUDE_ENV_FILE` を使用:

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

**v2.1.152 追加機能**:

- `hookSpecificOutput.reloadSkills: true` を返すと、フックでインストールしたスキルディレクトリを同一セッション内で再スキャンしてサーフェスできる
- `hookSpecificOutput.sessionTitle` で起動・再開時のセッションタイトルを設定できる

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "reloadSkills": true,
    "sessionTitle": "Patrol Docs - 2026-05-28"
  }
}
```

#### WorktreeCreate

Command フック: stdout にワークツリーの絶対パスを出力。
HTTP フック (`type: "http"`): レスポンスJSON の `hookSpecificOutput.worktreePath` でパスを返す。

```bash
#!/bin/bash
WORKTREE_PATH=$(jq -r .worktree_path)
echo "$WORKTREE_PATH"
```

---

#### UserPromptExpansion

ユーザーが `/skillname` のように**直接タイプしたコマンド**がプロンプトへ展開される直前に発火する。`PreToolUse` の `Skill` matcher は Claude が `Skill` ツールを呼んだ時にしか発火しないため、直接入力の経路はこのイベントでしか捕まえられない。特定コマンドの直接起動をブロックする、特定スキルにコンテキストを注入する、どのコマンドが使われたかを記録する、といった用途に使う。

入力（共通フィールドに加えて）:

| フィールド | 説明 |
|:--|:--|
| `expansion_type` | `slash_command`（スキル / カスタムコマンド）または `mcp_prompt`（MCP サーバーのプロンプト） |
| `command_name` | コマンド名（matcher の対象） |
| `command_args` | コマンド引数の文字列 |
| `command_source` | コマンドの出所（`plugin` 等） |
| `prompt` | 元のプロンプト文字列（例 `/example-skill arg1 arg2`） |

出力: 全 JSON 出力フィールドが利用可能。`decision: "block"` で展開を阻止（`reason` がユーザーに表示される。終了コード 2 + stderr でも同じ経路）。`hookSpecificOutput.additionalContext` で展開後プロンプトと併せてコンテキストを追加できる。stdout のプレーンテキストは `UserPromptSubmit` / `SessionStart` と同様に Claude が読めるコンテキストとして追加される。

```json
{
  "decision": "block",
  "reason": "このスラッシュコマンドは利用できません",
  "hookSpecificOutput": {
    "hookEventName": "UserPromptExpansion",
    "additionalContext": "展開に付随する追加コンテキスト"
  }
}
```

#### PostToolBatch

バッチ内の**全ツール呼び出しが解決した後**、次のモデルリクエスト送信前に 1 回だけ発火する。`PostToolUse` はツール毎に発火するため並列呼び出しでは同時多発するのに対し、`PostToolBatch` はバッチ全体で正確に 1 回。バッチ単位で 1 度だけ検証やコンテキスト注入をしたい場合はこちらを使う。

入力（共通フィールドに加えて）: `tool_calls` 配列。各要素は `tool_name` / `tool_input` / `tool_use_id` / `tool_response`。

> **注意**: `tool_response` の形が `PostToolUse` と異なる。`PostToolUse` はツールの構造化 `Output` オブジェクト（`Write` なら `{filePath: "...", success: true}` 等）を渡すが、`PostToolBatch` は**モデルが見る `tool_result` の内容をシリアライズした文字列またはコンテンツブロック配列**を渡す。`Read` なら行番号プレフィックス付きテキストであり生のファイル内容ではない。レスポンスは大きくなりうる。

出力:

| フィールド | 説明 |
|:--|:--|
| `hookSpecificOutput.additionalContext` | 次のモデル呼び出し前に 1 度だけ注入されるコンテキスト文字列 |

`decision: "block"` または `continue: false` で次のモデル呼び出し前にエージェントループを停止できる（メッセージは `reason` / `stopReason`、または終了コード 2 の stderr から）。トランスクリプトには警告として残り会話にも残るため、会話継続時に Claude から見える。


## 7. スクリプトパス参照

`command` フィールドで使用可能な変数:

| 変数 | 説明 |
|:--|:--|
| `$CLAUDE_PROJECT_DIR` | プロジェクトルート |
| `${CLAUDE_PLUGIN_ROOT}` | プラグインインストールディレクトリ |
| `${CLAUDE_PLUGIN_DATA}` | プラグイン永続データディレクトリ |
| `$CLAUDE_CODE_REMOTE` | Web環境で `"true"` |

---

## 8. 共通フィールド

| フィールド | 説明 | デフォルト |
|:--|:--|:--|
| `timeout` | タイムアウト（秒）。**`async: true` の command フックには適用されない**（バックグラウンドに入った後は Claude Code が強制終了しない）。`asyncRewake` のフックには引き続き適用される。`UserPromptSubmit` / **`PreModelSwitch`（v2.1.251+）** では command / http / mcp_tool の既定が 30 に、`MessageDisplay` では 10 に下がる。`SessionEnd` は全フックで 1.5 秒の共通予算 | command / http / mcp_tool: 600, prompt: 30, agent: 60 |
| `async` | バックグラウンド実行（command のみ） | `false` |
| `once` | **初回の成功実行後**にフックを取り除く。失敗・exit code 2 によるブロック・タイムアウトの場合はフックが残り、次の一致イベントで再実行される。スキル frontmatter で宣言したフックのみ有効（settings ファイルとエージェント frontmatter では無視） | `false` |
| `statusMessage` | スピナーメッセージ | - |
| `if` | ツールイベント専用の条件フィルタ（permission rule構文） | - |

### 8.1 `if` 条件フィールド（v2.1.85）

`if` フィールドはツールイベント（`PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`）でのみ評価される。他のイベントでは `if` を持つフックは実行されない。

matcher と組み合わせた2段階フィルタリング:
1. `matcher` でツール名をフィルタ
2. `if` でツール引数をさらに絞り込み

**構文**: permission rule構文を使用
- `"Bash(rm *)"` — `rm` で始まるBashコマンド
- `"Edit(*.ts)"` — TypeScriptファイルへの編集
- `"Bash(git push *)"` — git push コマンド
- `"Write(**/config.json)"` — config.json への書き込み

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": ".claude/hooks/block-rm.sh"
          },
          {
            "type": "command",
            "if": "Bash(git push *)",
            "command": ".claude/hooks/validate-push.sh"
          }
        ]
      }
    ]
  }
}
```

プロセス起動のオーバーヘッドを削減するため、フック実行前にプリフィルタリングされる。

---

## 9. フックの重複排除

- Command フック: command 文字列で重複排除
- HTTP フック: URL で重複排除
- 複数設定箇所にまたがる同一ハンドラは1回のみ実行

---

## 10. 実用例

### 10.1 破壊的コマンドのブロック

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

### 10.2 ファイル編集後のリンティング

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix"
          }
        ]
      }
    ]
  }
}
```

### 10.3 セッション開始時のコンテキスト追加

```bash
#!/bin/bash
jq -n '{
  hookSpecificOutput: {
    hookEventName: "SessionStart",
    additionalContext: "本番モードで動作中"
  }
}'
```

---

## 参考リンク

- Hooks: https://code.claude.com/docs/en/hooks
- Hooks ガイド: https://code.claude.com/docs/en/hooks-guide
- 権限: https://code.claude.com/docs/en/permissions
