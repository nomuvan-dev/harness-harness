# Claude Code Hooks 仕様書

最終更新: 2026-11-12（巡回更新）

公式ドキュメント: https://code.claude.com/docs/en/hooks

---

## 1. 概要

Hooks はユーザー定義のシェルコマンド・HTTPエンドポイント・LLMプロンプト・エージェントを、Claude Code のライフサイクルの特定ポイントで自動実行する仕組み。バリデーション、自動化、カスタムワークフロー制御に使用する。

CLAUDE.md の指示は助言的だが、Hooks は**決定論的**であり確実に実行される。

**v2.1.287 以降の関連機能**: プラグインは JavaScript / TypeScript の関数としてフックを登録することもできる（Claude Code プロセス内で実行され、UI 描画も可能）。これを持つプラグインは **mod** と呼ばれ、公式ドキュメントでは本仕様書の settings.json ベースのフックを "settings hook"、mod の関数フックを単に "hook" と区別する。詳細は skills-and-commands.md 4.9 参照。

---

## 2. イベント一覧

### 2.1 メインイベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `SessionStart` | セッション開始/再開 | No | `startup`, `resume`, `clear`, `compact`, `fork`（v2.1.214: フォーク開始時は `resume` ではなく `fork` を報告。`fork` の対象は `--fork-session`＋`--resume` / `--continue`、`/fork`、`/branch` に加え、**会話のバックグラウンド移行（`←` / `/background`）も含む**〈2026-10 ドキュメント改訂で明文化〉） |
| `UserPromptSubmit` | プロンプト送信後、処理前。**ユーザー入力以外でも発火**（2026-10 ドキュメント改訂で明文化）: スケジュールタスクの発火（`/loop` イテレーション含む）、バックグラウンドサブエージェントの起動元への報告、クロスセッションメッセージ受信。**v2.1.299 からスキル / カスタムスラッシュコマンドの展開前に実行**され、プロンプトをブロックすると埋め込みシェルコマンドも止まる（スキル自身の frontmatter `UserPromptSubmit` フックは起動プロンプトの次のプロンプトから発火） | Yes | - |
| `UserPromptExpansion` | ユーザーが打ったコマンドがプロンプトへ展開される時（Claude に届く前） | Yes | `command_name`（スキル名 / コマンド名）。matcher 省略で全 prompt 型コマンドに発火 |
| `PreToolUse` | ツール実行前 | Yes | **`EndConversation` 以外の全ツール名**（`Bash` / `PowerShell` / `Edit` / `Write` / `Read` / `Glob` / `Grep` / `Agent` / `Workflow` / `WebFetch` / `WebSearch` / `AskUserQuestion` / `ExitPlanMode` 等の組み込みツールと MCP ツール名） |
| `PermissionRequest` | ツール使用の権限ダイアログ表示時 | Yes | ツール名 |
| `PostToolUse` | ツール成功後（`EndConversation` の呼び出しは PreToolUse 同様スキップ） | No | ツール名 |
| `PostToolUseFailure` | ツール失敗後。**未知ツール名・スキーマ / ツール固有検証に落ちた入力は `tool_use_error` としてフック実行前に返されるため `PreToolUse` も本イベントも発火しない**。権限拒否は `PreToolUse` は発火するが本イベントは発火せず `PermissionDenied` が担当（2026-11-12 巡回で反映） | No | ツール名 |
| `PostToolBatch` | 並列ツール呼び出しのバッチ全体が解決した後、次のモデル呼び出し前 | Yes | matcher非サポート（全バッチで発火） |
| `Stop` | Claude の応答完了時（**ユーザー割り込みによる停止では発火しない**。API エラーは `StopFailure`） | Yes | - |
| `StopFailure` | APIエラー発生時 | No | `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `account_on_hold`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `cloud_credential_error`, `max_output_tokens`, `unknown`（`account_on_hold` は 2026-09-02 の公式ドキュメント改訂で追加。`cloud_credential_error` のマッチは **v2.1.267+** — それ以前は credential ロード失敗が `server_error` / `unknown` で報告されていた。2026-11-12 巡回で反映） |
| `PermissionDenied` | Auto Mode分類器が拒否した後（verdict なしの拒否を含む） | No | ツール名 |
| `SessionEnd` | セッション終了時 | No | `clear`, `resume`, `logout`, `prompt_input_exit`, `other`（**v2.1.234 で `bypass_permissions_disabled` を廃止**。Claude Code は送出しなくなったため matcher から削除すること） |

### 2.2 サブエージェントイベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `SubagentStart` | サブエージェント起動時。**加えて Claude がサブエージェントを resume したとき、およびインプロセスのエージェントチームのチームメイトが新しいメッセージを処理するたびにも発火する**（公式ドキュメント明文化） | No | エージェントタイプ名 |
| `SubagentStop` | サブエージェント完了時 | Yes | エージェントタイプ名 |
| `TeammateIdle` | チームメイトがアイドル状態になる直前 | Yes | matcherなし |
| `TaskCompleted` | タスク完了マーク時 | Yes | matcherなし |

### 2.3 通知・設定・環境イベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `Notification` | 通知送信時 | No | `permission_prompt`（ツール承認に加え、**サンドボックスコマンドのネットワークリクエスト承認**も対象。ターミナルセッションでは v2.1.246 以降）, `idle_prompt`（応答完了約60秒後、未入力のとき。**バックグラウンドエージェント〔バックグラウンドサブエージェント等〕が実行中の間は発火しない**〈2026-10 ドキュメント改訂で明文化〉）, `auth_success`, `elicitation_dialog`, `agent_needs_input`（バックグラウンドエージェントが入力待ちになったとき。**v2.1.248 以降、agent team のチームメイト用ターミナル設定の質問**、さらに **auto mode の分類器リクエスト課金通知**〈2026-10 ドキュメント改訂〉に対しても、一定時間ユーザーが入力しないと発火する）, `agent_completed`（バックグラウンドエージェント通知、v2.1.198）, `elicitation_url_dialog`, `elicitation_complete`, `elicitation_response`, `quota_auto_resume_fired` / `quota_auto_resume_stale` / `quota_auto_resume_disabled`（利用上限リセット後の自動継続、v2.1.234+。2026-11-12 巡回でバージョン訂正）, `auth_storage_failure`（サインインは完了したが新しい credential を credential storage に保存できなかったとき。v2.1.288+。2026-11-07 巡回で反映）, `plugin_notification`（mod が `$.ui.notify` で通知を出したとき。`title` は mod が設定しなければ mod 名。2026-11-05 巡回で反映）。`preferredNotifChannel`（`notifications_disabled` 含む）は通知の見せ方のみを変え、フックは常に発火する。canUseTool コールバックにプロンプトを送るセッション（Claude Desktop / VS Code 拡張）では `permission_prompt` は要求から約6秒後に発火し（タイプしても延期されない）、`CLAUDE_CODE_DISABLE_PERMISSION_PROMPT_NOTIFY_HOOKS=1` で無効化できる（2026-11-05 巡回で反映）。**ターミナルセッションでも `permission_prompt` / `elicitation_dialog` / `elicitation_url_dialog` は約6秒タイプしないと発火しない**（プロンプト表示でタイマー開始、打鍵ごとに延期。権限要求への即時反応が必要なら `PermissionRequest` を使う）。`idle_prompt` は利用上限リセット待機中は送られない（待機が自然終了すると代わりに `quota_auto_resume_*` が発火）。canUseTool セッションではユーザーまたは `PermissionRequest` フックが先に答えると `permission_prompt` は走らない（**v2.1.233 より前はこれらのセッションで発火しなかった**）（2026-11-12 巡回で反映） |
| `MessageDisplay` | アシスタントメッセージ表示時 | No（transform/hide可） | - | アシスタントメッセージのテキストを変換または非表示にできる（v2.1.152） |
| `ConfigChange` | 設定ファイル変更時 | Yes | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills` |
| `InstructionsLoaded` | CLAUDE.md/rules読み込み時 | No | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact` |
| `CwdChanged` | **メイン会話のシェルコマンド**がワーキングディレクトリを変更したとき | No | matcher非サポート（毎回発火）。`CLAUDE_ENV_FILE` に書いた変数は**次の CwdChanged イベントでクリアされる**（FileChanged で書いた変数も同様。2026-09-13 リファレンス改訂で明文化） |
| `DirectoryAdded` | `/add-dir` または SDK `register_repo_root` でセッション途中に作業ディレクトリが登録された後。`--add-dir` 起動フラグ・`/permissions` Workspace タブからの追加・既存作業ディレクトリ内の追加では発火しない。追加完了後にバックグラウンドで実行（追加はフックを待たない。ブロック不可） | No | `slash_command`（`/add-dir`）, `register_repo_root`（SDK control request）（matcher は 2026-11-05 巡回で反映。v2.1.219〜） |
| `FileChanged` | 監視対象ファイルのディスク変更時 | No | ファイル名（basename）例: `.envrc`, `.env` |
| `TaskCreated` | `TaskCreate` ツールでタスク作成時 | Yes | matcherなし |
| `Setup` | **`claude --init-only` 起動時、または `-p` モードでの `--init` / `--maintenance` 時のみ**（通常起動では発火しない。2026-10 ドキュメントで明確化）。CI / スクリプトから明示的に起動する一度きりの準備・定期クリーンアップ用。matcher は起動フラグに対応（`init` = `--init-only` / `-p --init`、`maintenance` = `-p --maintenance`）。`--init-only` は Setup フックと `startup` matcher の SessionStart フックを実行して会話を開始せず終了する。セッションごとの初期化には `SessionStart` を使う。**Setup フックも `CLAUDE_ENV_FILE` にアクセスでき、書いた変数はそのセッションの以後の Bash コマンドに引き継がれる**（SessionStart と同様。2026-11-05 巡回で反映。v2.1.318 でクラウドセッションにおいて Setup フックが書いた変数が Bash に届かないバグが修正された）。Setup で実行されるのは `type: "command"` フックのみ（`mcp_tool` は常にスキップ、`http` / `prompt` / `agent` は非サポート） | No | `init` / `maintenance` |

> **`Setup` フックの出力の扱い（ドキュメント改訂）**: `Setup` はブロックできず、終了コードに関わらず処理が続く。**どの終了コードでも `systemMessage` / `continue` / `hookSpecificOutput.additionalContext` などの JSON 出力フィールドは破棄される**（以前は `additionalContext` で Claude のコンテキストへ情報を渡せると記載されていたが、現在は渡せない）。`-p` 実行では `--output-format stream-json --verbose` で起動した場合に限り、stdout / stderr / 終了コードが `hook_response` イベントとして出力に現れる。決定制御の分類も「Context only」から `WorktreeRemove` / `Notification` / `SessionEnd` などと同じ**決定制御なし**へ移動した。

### 2.4 ワークツリー・コンパクションイベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `WorktreeCreate` | ワークツリー作成時 | Yes | - |
| `WorktreeRemove` | ワークツリー削除時 | **Yes（2026-09-13 リファレンス改訂）**: 非ゼロ終了コードで、削除後もディレクトリが残っている場合に削除を失敗させられる。worktree はディスクに残り、コマンドと stderr はデバッグログへ。バックグラウンドセッション削除中ならセッションも残り、agent view の拒否メッセージにフックの終了状態と stderr 冒頭が表示される。JSON 出力は破棄される（従来は「決定制御なし・失敗はデバッグログのみ」だった）。**クリーンアップ責務（2026-09-30 ドキュメント改訂で詳細化）**: WorktreeRemove フック未設定の場合、WorktreeCreate フックが返したパスに対し `git worktree remove --force` でフォールバック削除される（git が認識しない worktree はディスクに残る）。フックが exit 0 なら「削除済み」とみなされる（実際に削除したかは検証されないためフック側で確実に削除すること）。**WorktreeCreate フックが作ったブランチは Claude Code は削除しない**ため、ブランチ削除も WorktreeRemove フックで行う | - |
| `PreCompact` | コンパクション前 | **Yes**（exit 2 または `decision: "block"` でコンパクションをブロック。`manual` では stderr がユーザーに表示される。自動コンパクションのブロックは、コンテキスト上限前の先行実行なら未圧縮のまま継続、API がコンテキスト上限エラーを返した後のリカバリ実行ならエラーが表面化しリクエスト失敗。`auto` matcher は idle compaction でも発火。2026-11-05 巡回で修正。**stdout の扱い（2026-11-07 巡回で反映）**: exit 0 でブロックしない場合、フックの stdout は**コンパクション要約への指示**として使われる。フックあたり 10,000 文字以内に収めること（超過分はカットされマーカーが付く）。大出力をファイルに退避する通常の仕組みも PreCompact stdout には適用されない） | `manual`, `auto`（入力の `custom_instructions` は `manual` で `/compact` に引数が渡された場合のみ文字列。引数なし・`auto` では `null`。従来は空文字列と記載されていた） |
| `PostCompact` | コンパクション後 | No | `manual`, `auto` |

### 2.5 MCP Elicitation イベント

| イベント | 発火タイミング | ブロック可能 | matcher対象 |
|:--|:--|:--|:--|
| `Elicitation` | MCPサーバーがユーザー入力を要求 | Yes | MCPサーバー名 |
| `ElicitationResult` | ユーザーがMCP入力に回答 | Yes | MCPサーバー名 |

- `Elicitation` フックは `hookSpecificOutput` の `action`（`accept`＋`content` / `decline` / `cancel`）でダイアログを出さずに代理回答・拒否できる（出力なし exit 0 でダイアログに委ねる）。exit 2 またはトップレベル `decision: "block"` も decline として扱われ、同一出力内の `action` を上書きする。複数フックが競合した場合 decline が accept / cancel に優先。stderr や `reason` はユーザーにもサーバーにも表示されない
- `ElicitationResult` フックはユーザー回答の素通し / `content` の書き換え（`accept`、`content` は丸ごと置換） / ブロック（`decline`）/ `cancel`（ユーザーの送信値を伴う。値を渡したくなければ `decline`）を選べる。ユーザーが decline / cancel した場合も発火し、フックの `action` がユーザーの選択を置き換えるため、`accept` を返す前に入力の `action` を確認すること
- **v2.1.105〜v2.1.284 の間、Elicitation / ElicitationResult のトップレベル `decision` は無視されていた**（v2.1.284 で修正。2026-11-05 巡回で反映）

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

**managed プラグインの PreModelSwitch フックがロードに失敗した場合（v2.1.323 で挙動変更）**: 以前は組織の managed プラグインが届ける PreModelSwitch フックを確認できないと切替自体を拒否していた（`so organization-managed PreModelSwitch hooks could not be checked`）。v2.1.323 以降は**切替を許可**する — 失敗したプラグインのフックは走らないため、それがブロックしたはずの切替も通る（ロード済みの他の PreModelSwitch フックは引き続き実行・ブロック可能）。失敗原因は `/plugin` の **Errors** タブ、または `claude --debug` のデバッグログで確認し、修正後に `/reload-plugins`（2026-11-14 巡回で反映）。

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

**イベント別のハンドラタイプ対応（2026-11-05 巡回で反映）**:

- **全5タイプ（command / http / mcp_tool / prompt / agent）対応**: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`, `PermissionDenied`, `Stop`, `SubagentStop`, `UserPromptSubmit`, `UserPromptExpansion`, `TaskCreated`, `TaskCompleted`, `TeammateIdle`
- **`PermissionRequest`**: agent 以外の4タイプ（agent フックはスキップされ権限フローは変わらない）
- **command / http / mcp_tool のみ**: `ConfigChange`, `CwdChanged`, `DirectoryAdded`, `Elicitation`, `ElicitationResult`, `FileChanged`, `InstructionsLoaded`, `MessageDisplay`, `Notification`, `PreCompact`, `PostCompact`, `PreModelSwitch`, `PostModelSwitch`, `SessionEnd`, `StopFailure`, `SubagentStart`, `WorktreeCreate`, `WorktreeRemove`
- **`SessionStart` / `Setup`**: command と mcp_tool のみ（http / prompt / agent 非サポート。Setup の mcp_tool は常にスキップ、起動時 SessionStart の mcp_tool もスキップ — §3.5 参照）

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

- `shell`: 使用シェル。`"bash"`（既定。**Windows で Git Bash 未インストールなら既定は `"powershell"`**）または `"powershell"`。hooks は PowerShell を直接 spawn するため `CLAUDE_CODE_USE_POWERSHELL_TOOL` は不要。**`args` 設定時は `shell` は無視される**（2026-11-12 巡回で反映）
- `args: string[]`（exec form, v2.1.139+）: シェル解釈なしで直接コマンドを起動。`command` と組み合わせると引数を配列で安全に渡せる。パスプレースホルダーのクォート不要

```json
{
  "type": "command",
  "command": "/usr/bin/python3",
  "args": ["${CLAUDE_PROJECT_DIR}/scripts/validate.py", "$TOOL_NAME"]
}
```

**exec form / shell form（2026-11-12 巡回で反映）**: `args` の有無で形式が決まる。exec form（`args` あり）は `command` を PATH 上の実行ファイルとして解決し**シェル非経由**で直接 spawn する（各 `args` 要素はそのまま 1 引数、トークナイズなし）。shell form（`args` なし）はシェル（macOS/Linux は `sh -c`、Windows は Git Bash、不在なら PowerShell）に文字列を渡す。Windows の `.cmd` / `.bat` シム（npx / eslint 等が `node_modules/.bin` に置くもの）は実行ファイルでないため exec form では起動できず、`"command": "node", "args": ["…/eslint.js"]` のように node + スクリプトパスで呼ぶ。両形式とも `CLAUDE_PROJECT_DIR` / `CLAUDE_PLUGIN_ROOT` / `CLAUDE_PLUGIN_DATA` を環境変数としてエクスポートする。`${user_config.*}` 置換は exec form のみ。パスなしのコマンド名（bare name）に空白を含むと警告が出る。

**終了コード**:
- `0`: 成功（stdout の JSON をパース）
- `2`: ブロッキングエラー（stderr がエラーメッセージ）
- その他: 非ブロッキングエラー

**終了コード × stdout の判定マトリクス（2026-11-05 巡回で反映）**:

| stdout | exit 0 | exit 2 | その他 |
|:--|:--|:--|:--|
| スキーマ検証を通る JSON | 成功（フィールド適用） | ブロック（フィールドは読むがブロックを覆せない。`permissionDecision: "allow"` でも通らない） | **成功扱い**（exit code は無視されフィールドだけで決まる。`onFailure: "block"` のフックでは失敗扱い） |
| パース/検証失敗の JSON | 非ブロッキングエラー | ブロック（stderr が理由） | 非ブロッキングエラー |
| プレーンテキスト / 無出力 | 成功 | ブロック（stderr が理由） | 非ブロッキングエラー |

**例外的な終了コード挙動（2026-11-05 巡回で反映）**:
- `Stop` / `SubagentStop` / `TaskCompleted` / プラグインの `UserPromptSubmit`: exit 2 で stdout 無し・stderr が `No such file or directory` 等のファイル欠落メッセージの場合は**非ブロッキングエラー**として扱われる（存在しないスクリプトのフックが永遠にブロックし続けるのを防ぐ）
- `PermissionRequest`: exit 2 は無効（`decision` オブジェクトで deny する）
- `Elicitation` / `ElicitationResult`: `hookSpecificOutput` は exit 0 のときのみ適用
- **バックグラウンドプロセスを起動するフック**: 判定はバックグラウンド起動**前に** stdout へ出力し、バックグラウンドプロセスの stdout/stderr はリダイレクトすること。allow を出した後に未リダイレクトのバックグラウンドプロセスを残すと、Claude Code は `refusing to honor the suspect capture` エラーでツール呼び出しをブロックする。dev サーバー等の**長時間動くプログラムをフックから起動してセッション継続中も走らせ続けること自体は可能**（2026-11-11 巡回で公式ドキュメントに明文化を確認）: スクリプト / シェル形式コマンド内でバックグラウンド実行し、stdout / stderr をリダイレクトする（例 `npm run dev > dev-server.log 2>&1 &`）。リダイレクトしないと、`async` でないフックは自身のプロセス終了後も最大10秒完了扱いにならない
- **Windows（Git Bash 不在）**: シェル形式の `PreToolUse` / `PermissionRequest` command フックは 0 / 2 以外の exit code を verdict と見なさず、`onFailure` 未設定でも失敗がツール呼び出しをブロックしうる（エラーに `blocking because Git Bash wasn't found...` を含む。v2.1.309 でメッセージ改訂）

> **stdout を JSON と読むかプレーンテキストと読むかの判定（ドキュメント明文化）**: 前後の空白を無視して、
> - **`{` で始まり `}` で終わる**: JSON としてパースする。ただし出力が複数行で各行が単独で JSON としてパースでき、かつどの行も JSON 出力オブジェクトのフィールドを設定していない場合は、全体をプレーンテキストとして扱う。いずれかの行がフィールドを設定している場合は全体を JSON として扱う
> - **`{` で始まるが `}` で終わらない**: プレーンテキストとして扱う
>
> 標準の決定モデルを使うイベントで JSON パースに失敗した場合、**終了コード 2 以外のすべてで非ブロッキングエラー**を報告し、トランスクリプトに `<hook name> hook error` 通知（パースメッセージ付き）を表示する。
>
> **stdout がほとんどのイベントではデバッグログ行きでトランスクリプトに出ない**。例外は `UserPromptSubmit` / `UserPromptExpansion` / `SessionStart` / **`PostModelSwitch`（v2.1.251+）** で、これらはプレーンテキスト stdout を Claude が見られるコンテキストとして追加する。

> セキュリティ（v2.1.207）: プラグインの hooks / monitors / MCP headersHelper で、シェル形式コマンド内の `${user_config.*}` 展開はシェルインジェクション対策として拒否されるようになった。hooks は exec 形式（`args` 配列）または `$CLAUDE_PLUGIN_OPTION_<KEY>` 環境変数を使用する。また v2.1.199 から `SessionStart` / `Setup` / `SubagentStart` フックが exit code 2 で終了した際の stderr がトランスクリプトに表示される。

> **ハンドラの作業ディレクトリ（ドキュメント追記）**: ハンドラはカレントディレクトリで Claude Code の環境を引き継いで実行される。**カレントディレクトリが既に存在しない場合**（別シェルがセッション中に worktree や一時ディレクトリを削除した等）、Claude Code は「セッションを開始したディレクトリ → プロジェクトルート → ホームディレクトリ → システム一時ディレクトリ」の順に、存在する最初のものから command フックを実行し、警告を記録する。

> **`$CLAUDE_MODEL` は存在しない**。フックからモデルを知るには `SessionStart` の `model` フィールド（常に含まれるとは限らない）か、`PreModelSwitch` / `PostModelSwitch` の `from_model` / `to_model` を使う。`$ANTHROPIC_MODEL` はシェルで設定した場合に読めるが、セッション中に `/model` で切り替えても値は変わらない。フックプロセスは親環境を継承するが、Claude Code が全サブプロセスから除去する `OTEL_*` エクスポータ変数と、`CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1` のときに除去される変数は除く。HIPAA 構成が適用されたセッションでは、Anthropic 資格情報もフック環境から除去される（2026-10-06 ドキュメント改訂）。


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
- `2xx` JSONボディ: 判定としてパース（スキーマ検証失敗は非ブロッキングエラー）
- `2xx` その他ボディ（プレーンテキスト等）: **非ブロッキングエラー**。テキストは Claude のコンテキストに追加されない（従来「成功 + コンテキスト」と記載していたが、現行 docs では非2xx と同じ扱い。2026-11-05 巡回で修正）
- 非2xx / 接続失敗: 非ブロッキングエラー

**HTTP フックの allowlist 設定（2026-11-05 巡回で反映）**: 設定の `allowedHttpHookUrls` が（どのレベルでも）定義されていると、URL がマージ済み allowlist にマッチする HTTP フックのみ実行される。`httpHookAllowedEnvVars` が定義されていると、ヘッダへ補間できる環境変数はそのリスト上のものに限定される。両者は managed policy settings を含む全ソースのフックに適用される。

### 3.3 Prompt ハンドラ

Claude モデルにプロンプトを送信して評価。`model` 省略時の既定は、従来「高速モデル（Haiku）」だったが、**2026-09-30 ドキュメント改訂で「Claude Code が[バックグラウンド機能](https://code.claude.com/docs/en/costs#background-token-usage)に使うモデル」に変更**された（agent ハンドラも同様）。

```json
{
  "type": "prompt",
  "prompt": "これを承認すべきか？ $ARGUMENTS",
  "model": "model-name",
  "timeout": 30
}
```

`$ARGUMENTS` はフック入力 JSON のプレースホルダ（無い場合は入力 JSON がプロンプト末尾に付加される）。リテラルの `$` はバックスラッシュでエスケープ（`\$1.00`）。

**応答スキーマ**: モデルは `{"ok": true|false, "reason": "...", "impossible": true|false}` を返す。`ok: false` 時の挙動はイベント依存（2026-11-05 巡回で反映）:

- `Stop` / `SubagentStop`: `reason` が次の指示として Claude に戻りターン継続。`impossible: true` 付きなら「条件は満たしえない」と判断し停止を許可
- `PreToolUse`: ツール呼び出しを拒否。既定ではターン終了＋警告表示、`continueOnBlock: true` で reason をツールエラーとして Claude に返しターン継続
- `PostToolUse`: 既定でターン終了＋警告、`continueOnBlock: true` でフィードバックとして継続
- `PostToolBatch` / `UserPromptSubmit` / `UserPromptExpansion`: ターン終了＋警告（`continueOnBlock` 無効）
- `PostToolUseFailure` / `TaskCreated`: 常にツールエラーとして返りターン継続
- `TaskCompleted`: TaskUpdate 起因なら常にツールエラーで継続、teammate 停止起因なら TeammateIdle と同様に既定で停止
- `TeammateIdle`: 既定で teammate 停止、`continueOnBlock: true` で働き続けさせる
- `PermissionRequest` / `PermissionDenied`: `ok: false` は無効（command フックを使うこと）

`impossible` は prompt フック専用（agent フックは非対応）。

### 3.4 Agent ハンドラ

サブエージェントを起動して条件を検証。

```json
{
  "type": "agent",
  "prompt": "この設定を検証してください。$ARGUMENTS",
  "timeout": 60
}
```

設定フィールドは prompt ハンドラと同じだが、**agent ハンドラは既定タイムアウトが 60 秒と長く、`continueOnBlock` を持たない**。`$ARGUMENTS` はフック入力 JSON のプレースホルダ。サブエージェントは Read / Grep / Glob 等のツールを使い**最大 50 ターン**で `{"ok": true/false}` を返す。`ok: false` 時は**そのイベントで prompt フックが `continueOnBlock: true` の場合と同じ扱い**になる（`impossible` フィールドは非サポート）。agent フックは実験的機能（2026-11-05 巡回で反映）。

**同時実行上限（v2.1.312+）**: prompt / agent フックはサブエージェント分も含めセッションあたり最大 24 並列。イベントのタイムアウトを超えて待機しているものは最大 12。また `timeout` の 1 秒未満指定は 1 秒として扱われる（※公式 hooks ページ未収載＝changelog 先行）。

### 3.5 MCP Tool ハンドラ（v2.1.118+）

MCP サーバーのツールを直接呼び出す。シェル経由のラッパースクリプトなしに MCP ツールを発火可能。

```json
{
  "type": "mcp_tool",
  "server": "my-server",
  "tool": "validate_change",
  "input": { "file_path": "${tool_input.file_path}" },
  "timeout": 30
}
```

- `input`（任意）: ツールに渡す引数。文字列値はフック入力 JSON からの `${path}` 置換をサポート（例 `"${tool_input.file_path}"`）（2026-11-05 巡回で反映）
- `server`: プラグイン同梱サーバーはスコープ付き名 `plugin:<plugin-name>:<server-name>` で指定する（素のサーバーキーではない）

**結果の読み取り**: ツールのテキスト出力は command フックの stdout と同じルール（exit code 0 のパースルール）で読まれる。ツールが `isError: true` を返した場合は非ブロッキングエラーとなり実行は継続する。

**巨大な結果で verdict が読めない場合（v2.1.321 / 2026-11-11 巡回で反映）**: MCP サーバーからのメッセージが大きすぎて答えを読めなかった場合（`gave no answer that could be read, because a message from the server was too large` / `answered without its text, which happens when a result is too large`）、フックは **verdict なし**となる。`PreToolUse` ではチェック対象のツール呼び出しが**拒否**され（v2.1.321 以前はフックがスキップされて素通りだった）、`UserPromptSubmit` / `PostToolUse` / `Stop` 等の他イベントでは非ブロッキングエラー。恒常的に起きる場合はツールの返すデータを減らすか、フックの `tool` / `input` を変えて答えを小さくする。なお stdio / HTTP MCP サーバー上の `PreToolUse` `mcp_tool` フックは、答えが 16 MiB 超のときスキップされていたのが v2.1.321 で拒否に修正された。

**サーバー接続待機（2026-09-28 リファレンス改訂で明文化）**: 以前は「サーバーが接続済みでなければならない」とされていたが、接続中のサーバーを待つ動作に改訂された。

- **ブロック可能イベント**（`PreToolUse` / `Stop` など、結果を変えられるイベント）では、接続中のサーバーを **`MCP_TIMEOUT` かつフック自身の `timeout` の範囲内で待ってから**ツールを呼ぶ
- **観測系イベント**（`Notification` / `SessionEnd` など）では待たない
- `cached` ステータスのサーバーはフックがツールを呼んだ時点で接続される
- 呼び出し時点で未接続ならフックは非ブロッキングエラーとなり実行継続。**OAuth フローは決して開始しない**ため、リモートサーバーは事前に `/mcp` で認証しておくこと

**発火タイミングの制約（2026-09-13 リファレンス改訂で明文化）**: `mcp_tool` フックはセッションの MCP サーバーがフックから利用可能になった後にのみ実行される。

- **起動時の `SessionStart`**（`--continue` / `--resume` 含む）はサーバー利用可能前に発火するため、`mcp_tool` フックは**スキップ**される（デバッグログに `mcp_tool hooks are not available for the 'SessionStart' hook event (no MCP client context)` が記録される）
- **`/clear` やコンパクション後の `SessionStart`** はサーバーが既に利用可能なので実行される
- **`Setup` は常にサーバー利用可能前**なので、`Setup` の `mcp_tool` フックは毎回スキップされる
- 初回ターンから必要な処理は `type: "command"` フックで行うこと

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

> **クラウドセッション / self-hosted 環境のフック（2026-11-05 巡回で反映)**: クラウドセッションはローカルの `~/.claude/settings.json` を読まない。self-hosted 環境では、オペレーターが runner ホストの `~/.claude/` からシードしたフックも実行され、runner イメージの managed settings ファイルのフックは「Claude Code が適用する managed ソース」に含まれる場合（既定ではサーバー管理設定も MDM 配布ポリシーも managed 層を供給していない場合のみ）に実行される。どの設定ファイル・プラグイン（＝どのフック）がクラウドセッションに届くかは cloud-environments ドキュメントの "what carries over from your setup" を参照。

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

- `disableAllHooks` は settings precedence 適用後の値が読まれる: プロジェクトの `.claude/settings.json` の `false` はユーザー設定の `true` を上書きする。1回の実行だけ無効化するには `--settings '{"disableAllHooks": true}'`（project / local 設定に優先）。個別フックだけを無効化する手段は無い（2026-11-05 巡回で反映）
- 管理者は managed settings の `allowManagedHooksOnly` で user / project / local / プラグインフックをブロックできる（managed settings の `enabledPlugins` で強制有効化されたプラグインのフックは例外。`statusLine` / `fileSuggestion` も managed 設定のものに絞られる。`subagentStatusLine` は v2.1.323 で設定ごと削除）（2026-11-05 巡回で反映、2026-11-14 更新）
- フック設定は settings レベル間で**マージ**される（置換ではない）。settings ファイルの直接編集はファイルウォッチャーで自動反映される
- managed 設定 `strictPluginOnlyCustomization` でフックを**プラグイン＋managed 設定のみ**にロックできる（スキル frontmatter フックはプラグインスキル / bundled skill / managed settings ディレクトリのスキルのみ存続。自分の settings ファイルのフックも走らない）。確認は `/status` の `Setting sources` 行（2026-11-12 巡回で反映）
- 未信頼環境では `command` ソースのプラグイン（managed settings の `enabledPlugins` で強制有効化された分も含む）とマーケットプレイスの `headersHelper` コマンドは、`disableCommandPluginSources: false` の明示設定まで無効（managed settings 自身が宣言するマーケットプレイスの headersHelper は例外）（2026-11-12 巡回で反映）

### 4.4 ワークスペース信頼（2026-11-12 巡回で反映）

設定ファイル由来のフックはワークスペース信頼チェックを通ってから実行される:

- **対話セッション**: 信頼ダイアログを受諾するまで、`~/.claude/settings.json` 分も含め**全 settings ファイルのフックが保留される**（親ディレクトリの信頼が及ぶ場合は除く）
- **`-p` / SDK セッション**: ダイアログを出さず**常に信頼扱い**。未信頼リポジトリの `.claude/settings.json` のフックが走る。見知らぬリポジトリで `claude -p` をスクリプト実行する前に `.claude/` の settings を確認するか、`--bare` 起動または `--settings '{"disableAllHooks": true}'` で防ぐ
- プロジェクトスキルの frontmatter フックは settings ファイルと同じ信頼ルール（`-p` の未信頼フォルダでも登録される）。プロジェクトサブエージェントの frontmatter フックはより厳しく、エージェントファイルのあるフォルダの信頼ダイアログ受諾後のみ実行される（`-p` セッションは受諾と見なされない）

---

## 5. Matcher

Matcher はフック発火条件をフィルタリングする文字列。省略・`""`・`"*"` で全対象。

**評価ルール（2026-11-05 巡回で反映）**: matcher に含まれる文字で評価方法が決まる。

- **英数字・`_`・`-`・スペース・`,`・`|` のみ**: 完全一致文字列、または `|` / `,` 区切りの完全一致リスト（`Edit|Write`、`Edit, Write`）。`code-reviewer` はそのエージェントタイプのみに一致
- **それ以外の文字を含む**: JavaScript 正規表現（アンカーなし、部分一致）。`Edit.*` は `NotebookEdit` にも一致するため、完全一致には `^Edit$` のように `^` `$` で囲む
- **`FileChanged` / `StopFailure`** は完全一致文字セットが英数字・`_`・`|` のみとさらに狭く、区切りも `|` のみ（ハイフン・スペース・カンマは正規表現パスに乗る）

### 5.1 MCPツールマッチング

MCPツールは `mcp__<server>__<tool>` パターンに従う:

```json
{
  "matcher": "mcp__memory__.*"
}
```

サーバーの全ツールに一致させるには `.*` が**必須**: `mcp__memory` や `mcp__brave-search` は完全一致文字のみで構成されるため完全一致文字列として比較され、どのツールにも一致しない。**プラグイン同梱 MCP サーバーのツール名はスコープ付き** `mcp__plugin_<plugin-name>_<server-name>__<tool>` となり、素のサーバーキーで書いた matcher は発火しない（例: プラグイン `my-plugin` のサーバー `db` の全ツールは `mcp__plugin_my-plugin_db__.*`。`if` フィールドでも同じスコープ付きツール名を使う）（2026-11-05 巡回で反映）。

### 5.2 複数ツールのマッチング

```json
{
  "matcher": "Edit|Write"
}
```

### 5.3 マッチング挙動の変更（v2.1.191 / v2.1.195）

- **カンマ区切りmatcher修正（v2.1.191）**: `"Bash,PowerShell"` のようなカンマ区切りmatcherが silent に発火しないバグが修正された
- **ハイフン識別子の完全一致化（v2.1.195、破壊的変更）**: `code-reviewer` や `mcp__brave-search` のようなハイフンを含む識別子が部分文字列マッチしてしまう挙動を修正し、**完全一致**になった。ハイフン入りMCPサーバーの全ツールにマッチさせるには `mcp__brave-search__.*` のように明示的な正規表現を使う

### 5.4 Monitor コマンドのマッチング（v2.1.314+、2026-11-07 巡回で反映）

`Bash` matcher の `PreToolUse` / `PermissionRequest` フックは、**Monitor ツールが起動するシェルコマンドでも発火する**（`Bash|PowerShell`・`^Bash$`・`Bash.*` も「Bash matcher」扱い）。ただしこの経路では `allow` と `updatedInput` は無効（deny / ask は可能）。WebSocket watch では発火せず、`PostToolUse` の Bash matcher は Monitor では発火しない。

Monitor コマンドを allow / 書き換えしたい場合は matcher を Monitor を含むものに切り替える:

- **Monitor を含む matcher**（`Bash|Monitor`・`.*`・matcher なし）: フックは Monitor 呼び出し自体の入力を受け取り、`allow` / `updatedInput` が効く。`if` では `Monitor(tail *)` のように Monitor を名指しする（マッチ規則は `Bash(tail *)` と同じ）。`if` で Bash を名指しすると入力は Monitor のものだが `allow` / `updatedInput` は無効
- **`Bash` matcher**: フックは Bash 呼び出しとして入力を受け取り、`if` では `Bash(tail *)` と書く。`if` で Monitor を名指しするとフック自体が発火しない

---

## 6. 入出力フォーマット

### 6.1 共通入力フィールド

全フックが受け取る共通 JSON:

```json
{
  "session_id": "abc123",
  "prompt_id": "550e8400-e29b-41d4-a716-446655440000",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "EventName",
  "scratchpad_dir": "/tmp/claude-1000/-home-user-my-project/abc123/scratchpad"
}
```

- `prompt_id`: 処理中のユーザープロンプトを識別する UUID。OpenTelemetry イベントの `prompt.id` 属性と一致し、フック出力とテレメトリを単一プロンプト単位で突合できる。最初のユーザー入力までは存在しない（2026-11-05 巡回で反映）
- `scratchpad_dir`: セッションのスクラッチパッドディレクトリ（Claude が一時作業ファイルを置く場所）へのパス。**v2.1.257 以降**。セッションがスクラッチパッドを持たない場合や temp ディレクトリが使えない場合は**キー自体が存在しない**
- `permission_mode`: `"default"` / `"plan"` / `"acceptEdits"` / `"auto"` / `"dontAsk"` / `"bypassPermissions"`。UI で **Manual** と表示されるモードは常に `"default"` として届く（`"manual"` にはならない）。全イベントが受け取るわけではない
- `transcript_path`: トランスクリプトファイルは非同期書き込みのため、フック発火時点では現在ターンの最新メッセージを含まないことがある。現在ターンの最終応答テキストが必要なら `Stop` / `SubagentStop` の `last_assistant_message` を使う

サブエージェント内では追加:

```json
{
  "agent_id": "unique-id",
  "agent_type": "AgentName",
  "worktree": "/path/to/worktree"
}
```

- `worktree`: ワークツリー内で実行中の場合、そのパス

v2.1.133 以降、ツール実行コンテキストを持つイベント（`PreToolUse` / `PostToolUse` / `Stop` / `SubagentStop` 等。effort 対応モデルのみ）の入力 JSON に effort level も含まれる:

```json
{
  "effort": { "level": "low" | "medium" | "high" | "xhigh" | "max" }
}
```

`level` はフック実行時点の effort **設定**で、スキル / サブエージェントの frontmatter `effort` や `ultrathink` によるレベルも反映される。モデルが未対応のレベルを指定した場合は実際に実行されたレベルが入る。**変更したレベルは適用開始前からここに現れうる**（2026-10-30 ドキュメント改訂。新レベルの適用開始タイミングは model-config の "When a new effort level starts" 参照）。加えて、hooks のコマンドおよび Bash ツールサブプロセスから `$CLAUDE_EFFORT` 環境変数で同じ値が参照可能。

### 6.2 イベント固有の入力フィールド

| イベント | 追加入力フィールド |
|:--|:--|
| `SessionStart` | `source`, `model`, `agent_type`(opt)。**v2.1.251 以降、`source` が `resume` / `fork` かつトランスクリプトに Claude の応答が1件以上ある場合のみ** `seconds_since_last_response`（直近応答からの経過秒）, `context_tokens`（再開後の最初のリクエストが再送するトークン数）, `prompt_cache_likely_expired`（直近応答がプロンプトキャッシュ寿命より古い、または後続のコンパクションがキャッシュ済み会話を置換した場合 `true`）, `estimated_cache_write_usd`（`context_tokens` をセッションのモデルへ書き込む推定 USD、応答分は含まない）が加わる。**`session_title`**（2026-10 ドキュメント改訂で明文化）: カスタムタイトルが設定済みの場合のみ含まれる。`--name` / `/rename` / フックの `sessionTitle` 出力 / SDK `renameSession()` 由来のタイトルが対象で、**自動生成タイトルは含まれない**。`sessionTitle` を出すフックは先に本フィールドを確認すると既存タイトルの上書きを避けられる |
| `UserPromptSubmit` | `prompt`。`[Pasted text #N]` に折りたたまれたペースト内容は**展開されて届く**。ペーストマーキング有効セッションでは展開内容が `<pasted_content id="…">` 〜 `</pasted_content id="…">` 行に挟まれるため、プロンプトをパースするフックはこの行を考慮する（2026-09-22 ドキュメント改訂で明文化）。カスタムタイトル設定済みセッションでは `session_title` も受け取る（意味は `SessionStart` の同フィールドと同じ。2026-10 ドキュメント改訂） |
| `UserPromptExpansion` | `expansion_type` (`slash_command`/`mcp_prompt`), `command_name`, `command_args`, `command_source`, `prompt` |
| `PreToolUse` | `tool_name`, `tool_input`, `tool_use_id` |
| `PermissionRequest` | `tool_name`, `tool_input`, `permission_suggestions`(opt) |

> `PermissionRequest` は **agent フック非対応**（v2.1.280 で明確化）: サポートするフックタイプは `command` / `http` / `mcp_tool` / `prompt` のみ。agent フックを設定してもスキップされ（エラー表示あり）権限フローは変わらない。フックから allow / deny するには command / HTTP フックの decision オブジェクトを返す。agent フックは他のイベントでは prompt フックと同じイベントをサポートする。
>
> **権限ホストとの並走（2026-10 ドキュメント改訂で明文化）**: `--permission-prompt-tool` や Agent SDK の `canUseTool` コールバックに届く呼び出しでは、`PermissionRequest` フックはホストと**並行して**実行され、**先に決定した側が適用される**。
>
> `PermissionRequest` は **サンドボックスコマンドのネットワークリクエスト**の権限プロンプトでは発火しない（ツール使用の権限要求のみ）。ネットワークリクエスト側のシグナルが必要な場合は `Notification` の `permission_prompt` タイプを使う（ただし約 6 秒の待機後に発火）。
>
> **フック決定後の権限チェック失敗時の挙動（v2.1.323 で変更）**: `PreToolUse` フックが `allow` を返した後にツールの権限チェックがエラーで失敗した場合、Claude Code は**フックが答えなかったものとして**その呼び出しを判定する。`PermissionRequest` フックが書き換えた入力で権限チェックが失敗した場合は**再度ユーザーに確認**する（誰にも確認できない環境では呼び出しを拒否）（2026-11-14 巡回で反映）。
>
> **`mcp_server` オブジェクト（v2.1.274+、2026-09-18 ドキュメント改訂で明文化）**: MCP ツールの場合、`PreToolUse` / `PermissionRequest` / `PostToolUse` / `PostToolUseFailure` / `PermissionDenied` の入力に `mcp_server` オブジェクトが加わる。サーバーの `name` と、定義の提供元を示す `source`（`plugin`, `sdk`, `user` / `project` 等の設定スコープ。全列挙は Agent SDK リファレンスの `McpServerProvenance`）を持つ。**信頼判断は `name` や `mcp__<server>__` プレフィックスではなく `source` に基づくこと**（名前は偽装可能なため）。
>
> `permission_suggestions`（2026-11-09 巡回で改訂）: `PreToolUse` と同じ `tool_name` / `tool_input` を受け取るが `tool_use_id` は無い。`permission_suggestions` には Claude Code がその要求に対して提案する**権限更新（allow ルールの追加、権限モードの変更など）**が入る。**各権限ダイアログは自前で選択肢を構築する**: ファイル編集のダイアログなどは配列を一切読まず要求自体から選択肢を導出し、suggestion エントリを持たない選択肢（「Yes, and switch to auto mode」等、permission update を介さずモードを直接変えるもの）も提示されうる。フックは受け取った `permission_suggestions` の 1 つをそのまま `updatedPermissions` として出力できる。
>
> **プロンプトを出せないセッションでの発火とデフォルト deny（2026-11-09 巡回で反映）**: プロンプトを表示できないセッション（非対話モードのバックグラウンドサブエージェント等）でも `PermissionRequest` フックは実行される。発火タイミングは「権限を尋ねる直前、またはプロンプトを出せず auto-deny する直前」で、**どのフックも decision を返さなければそのツール呼び出しは deny される**。
>
> **decision オブジェクトの仕様（PermissionRequest decision control、2026-11-09 巡回で反映）**: `hookSpecificOutput.decision` のフィールドは以下。decision なしの exit 2 は権限フローを変えず **stderr も破棄される**。
>
> | フィールド | 制約 | 説明 |
> |:--|:--|:--|
> | `behavior` | 必須 | `allow` / `deny`。**allow でも deny / ask ルールは評価され、マッチする deny ルールを上書きできない** |
> | `updatedInput` | allow 専用 | ツール入力を丸ごと置換。**変更後の入力は deny / ask ルールで再評価される**。`AskUserQuestion` / `ExitPlanMode` では必須 |
> | `updatedPermissions` | allow 専用 | 権限更新（下記エントリ型参照） |
> | `message` | deny 専用 | Claude に渡す拒否理由 |
> | `interrupt` | deny 専用 | `true` で Claude を停止する |
>
> **Permission update エントリ型（`updatedPermissions` / `permission_suggestions` 共通、2026-11-09 巡回で反映）**: `type` は `addRules` / `replaceRules` / `removeRules`（`rules` 配列＋`behavior`）、`setMode`（`mode`）、`addDirectories` / `removeDirectories`（`directories` 配列）の6種。各エントリの `destination` は `session`（メモリのみ・セッション限り）/ `localSettings`（`.claude/settings.local.json`）/ `projectSettings`（`.claude/settings.json`）/ `userSettings`（`~/.claude/settings.json`）の4値。`rules` は **`{toolName, ruleContent?}` オブジェクトの配列**: `Bash(npm test)` なら `{"toolName": "Bash", "ruleContent": "npm test"}`、ツール全体（`Bash` / `Bash(*)`）は `ruleContent` 省略。**MCP ツールの allow ルールに `ruleContent` を付けると一切マッチしなくなる**ため省略すること（2026-11-12 巡回で反映）。
| `PostToolUse` | `tool_name`, `tool_input`, `tool_response`, `tool_use_id`, `duration_ms`（v2.1.119+。権限プロンプトと PreToolUse 時間を除いたツール実行時間）。Bash では条件付きで `tool_response.bashEditDiff`（下記注参照） |
| `PostToolUseFailure` | `tool_name`, `tool_input`, `tool_use_id`, `error`, `is_interrupt`, `duration_ms`（v2.1.119+） |
| （注: Bash の `bashEditDiff`） | **v2.1.269+、パブリックベータ**。Bash コマンドが Git リポジトリ内のファイルを変更した場合、`tool_response.bashEditDiff` に変更ファイル一覧が入る。記録条件: [`bashEditDiffEnabled`](configuration.md) が `true` なら全権限モードで記録、未設定なら auto モード / `bypassPermissions` で Claude Code が Bash 経由編集を指示した場合のみ。`false` で無効化（環境変数 `CLAUDE_CODE_BASH_EDIT_DIFF` がセッション単位で優先）。フィールド: `changedFiles`（絶対パス最大200件）/ `files`（表示用 diff 最大5件、`created` / `deleted` フラグ付き）/ `moreFiles`（diff なし変更ファイル数）/ `unavailable`（diff 不完全）/ `skipped`（`git checkout` / `git stash` 等ワーキングツリーを動かす Git コマンド）/ `shared`（同一リポジトリで別の Bash 呼び出しが並走し混入の可能性あり）。gitignore 対象とサブモジュール内は載らない。**ベストエフォート**であり、レビュー対象の発見用でポリシー強制には使わないこと |
| （注: `SubagentHandback` 経由のレポート） | **v2.1.271+**。auto モードでローカル実行される非フォークのサブエージェントは、最終レポートをテキストではなく `SubagentHandback` ツールで納品する。この場合フォアグラウンド Agent 完了時の `tool_response.content` はハンドバック済みを示す短い注記になり、`SubagentStop` の `last_assistant_message` もレポートではない。**レポート本文は `SubagentHandback` にマッチした `PreToolUse` / `PostToolUse` フックの `tool_input.message` で読む** |
| （注: Agent 完了時のテレメトリ） | **2026-11-12 巡回で反映**。フォアグラウンド `Agent` ツール完了時の `tool_response` は run テレメトリを含む: `status`（`completed` / `async_launched`）/ `agentId` / `content`（最終テキストブロック。SubagentHandback 経由なら短い注記）/ `resolvedModel`（起動時モデル。`availableModels` 等の上書きで要求モデルと異なりうる）/ `modelsUsed`（途中でモデル切替があった場合のみ。使用順・連続重複は畳む）/ `totalTokens`（**最終 API リクエストのみ**の input+output+cache 合計で run 全体ではない）/ `totalDurationMs` / `totalToolUseCount` / `usage`（最終リクエストの `input_tokens` / `output_tokens` / `cache_creation_input_tokens` / `cache_read_input_tokens`）。バックグラウンド起動（途中バックグラウンド化含む）では usage 系なしで `status: "async_launched"` + `agentId` / `description` / `prompt` / `outputFile` / `resolvedModel` のみ。サブエージェント横断のトークン・コスト集計は `query_source: "subagent"` でフィルタしたトークンカウンタを使う |
| `PostToolBatch` | `tool_calls`（各要素は `tool_name`, `tool_input`, `tool_use_id`, `tool_response`）。`tool_response` は `PostToolUse` と形が異なりモデルが見る `tool_result` のシリアライズ |
| `PermissionDenied` | `tool_name`, `tool_input`, `tool_use_id`, `reason`。`reason` の形式が改訂され、**分類器の判定はマッチしたルール名を角括弧で示す**（例 `[Data Exfiltration]`、`[Irreversible Local Destruction]`）。従来の固定文言 `Blocked by classifier` ではなくなった。no-verdict 拒否は従来どおり `Auto mode could not evaluate this action and is blocking it for safety` で始まる。**分類器モデルが利用不能だった拒否は固定文言 `Classifier unavailable`**（2026-11-05 巡回で反映） |
| `Stop` | `stop_hook_active`, `last_assistant_message`, `background_tasks`, `session_crons`（v2.1.145+）。`background_tasks` の各要素は `id` / `type`（`shell`, `subagent`, `monitor`, `workflow`, `teammate`, `cloud session`, `MCP task` 等のラベル） / `status` / `description`（1000字上限） / `command`（shell のみ） / `agent_type`（subagent のみ） / `server`・`tool`（monitor / MCP task のみ） / `name`（workflow のみ）。`session_crons` の各要素は `id` / `schedule`（cron 式） / `recurring` / `prompt`（1000字上限）。「セッション完了」と「バックグラウンド作業の起床待ち」を区別できる（2026-11-05 巡回でフィールド詳細を反映）。`session_crons` の発生源は `CronCreate` / `ScheduleWakeup` / `/loop`（2026-11-12 巡回で反映） |
| `StopFailure` | `error`（`rate_limit` / `overloaded` / `authentication_failed` / `oauth_org_not_allowed` / `account_on_hold` / `billing_error` / `invalid_request` / `model_not_found` / `server_error` / `cloud_credential_error` / `max_output_tokens` / `unknown`）, `error_details`, `last_assistant_message` |
| `Notification` | `message`, `title`, `notification_type` |
| `SubagentStart` | `agent_id`, `agent_type` |
| `SubagentStop` | `stop_hook_active`, `agent_id`, `agent_type`, `agent_transcript_path`, `last_assistant_message`, `background_tasks`, `session_crons`（v2.1.145+）。**内部エージェントでも発火**（2026-09-22 ドキュメント改訂で明文化）: prompt suggestions や `/btw` 等の Claude Code 内部機能のエージェント完了時にも発火し、その場合 `agent_type` はセッション自身のエージェント名（`--agent` / `agent` 設定。未設定なら空文字列）。エージェントタイプを名指しする matcher は空 `agent_type` にマッチしない（matcher 省略 / `""` / `"*"` / 空文字にマッチする正規表現は発火する） |
| `InstructionsLoaded` | `file_path`, `memory_type`, `load_reason`, `globs`(opt), `trigger_file_path`(opt), `parent_file_path`(opt)。**Project instructions 設定経由で AGENTS.md を直接読む場合（v2.1.277+）は発火しない**。CLAUDE.md が AGENTS.md をインポートする場合は `load_reason: include` で、symlink の場合は通常の CLAUDE.md ロードとして発火する |
| `CwdChanged` | `old_cwd`, `new_cwd`（従来記載の `cwd` のみではなく変更前後のパスを受け取る。2026-11-05 巡回で修正）。出力で `watchPaths`（絶対パス配列）を返すと FileChanged の動的監視リストを**置換**できる（空配列でクリア。matcher 由来のパスは常に監視） |
| `DirectoryAdded` | `directory`（追加されたディレクトリの絶対パス）, `source`（`"slash_command"` / `"register_repo_root"`）。`slash_command` では `systemMessage` が次ターンのコンテキストとして Claude に届く（ユーザーには出ない）、`register_repo_root` ではデバッグログのみ（2026-11-05 巡回で反映） |
| `FileChanged` | `file_path`, `event`（`"change"`=変更 / `"add"`=作成 / `"unlink"`=削除。従来記載の `change_type` (`created`/`modified`/`deleted`) は現行 docs に無く、フィールド名・値とも `event` 形式が正（2026-11-05 巡回で修正））。出力で `watchPaths` を返し監視リストを動的更新可能。matcher は「監視リストの構築（`\|` 区切りのリテラルファイル名）」と「発火フィルタ（basename に対する標準 matcher 規則）」の二役を持つ。ウォッチャーは何かがファイルを指名して初めて起動するため、動的パス用グループは matcher 省略にする（`"*"` はリテラル `*` というファイルを監視登録してしまう） |
| `TaskCreated` | `task_id`, `task_subject`, `task_description`(opt), `teammate_name`(opt), `team_name`（**非推奨: 将来のリリースで削除予定**。2026-11-05 巡回で反映）, `agent_id`(opt, v2.1.290+ タスクを作成したサブエージェント / in-process teammate を識別) |
| `ConfigChange` | `source`, `file_path`。発火するのは設定ファイル・managed ポリシーファイル（`managed-settings.json` / `managed-settings.d/` 内）・スキルファイルの変更時。**発火しない変更**（フックなしで適用のみ）: 「Yes, and don't ask again」/`/permissions`/`/model` で保存した当のセッション（同じファイルを読む別セッションでは発火する）、server-managed settings / macOS managed preferences / Windows レジストリポリシーの更新、WSL の `wslInheritsWindowsSettings` 経由で取り込む Windows 側 managed settings（2026-11-14 巡回で反映） |
| `WorktreeCreate` | `name`（新ワークツリーのスラグ識別子。ユーザー指定または自動生成）。従来記載の `worktree_path` / `isolation_level` は現行 docs に無い（2026-11-05 巡回で修正） |
| `WorktreeRemove` | `worktree_path` |
| `PreCompact` | `trigger`, `custom_instructions` |
| `PostCompact` | `trigger`, `compact_summary` |
| `PreModelSwitch` / `PostModelSwitch` | `from_model`, `to_model`, `requested_model`, `source`, `context_tokens`, `prompt_cache_warm`, `cache_ttl`, `estimated_cache_write_usd`, `pricing`（§2.6 参照、v2.1.251+） |
| `Elicitation` | `mcp_server_name`, `message`, `mode`(opt), `url`(opt), `elicitation_id`, `requested_schema` |
| `TeammateIdle` | `teammate_name`, `team_name`（**非推奨: 将来のリリースで削除予定**）, `agent_id`(opt, v2.1.290+) |
| `TaskCompleted` | `task_id`, `task_subject`, `task_description`(opt), `teammate_name`(opt), `team_name`（**非推奨: 将来のリリースで削除予定**）, `agent_id`(opt, v2.1.290+) |

> **TaskCreated / TeammateIdle / TaskCompleted の決定制御（2026-11-12 巡回で反映）**: `TaskCreated` は exit 2 または `decision: "block"` でタスクをキャンセルしメッセージを Claude に返す（`continue: false` は無視される）。`TeammateIdle` / `TaskCompleted` は exit 2（stderr がフィードバック）に加え、`{"continue": false, "stopReason": "..."}` でも `Stop` フック相当としてチームメイトを停止できる（ただし `TaskCompleted` は `TaskUpdate` ツール起因の発火では `continue: false` を無視する）。

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

`updatedPermissions` に `permission_suggestions` の要素をそのまま返すと、その権限更新が適用される（従来ドキュメントにあった「ダイアログでその always-allow 選択肢を選んだのと等価」という言い回しは、選択肢と提案が一対一でないため 2026-09-05 時点の記述から削除された）。

`updatedPermissions` の `setMode`（`mode` / `destination`）による権限モード変更の制約:

- 有効な `mode` は `default` / `auto` / `acceptEdits` / `dontAsk` / `bypassPermissions` / `plan`、および `default` の別名 `manual`（`manual` は v2.1.200+）
- `bypassPermissions` への変更は、**セッション起動時点で bypass モードが使える状態だった場合のみ**有効。すなわち `--dangerously-skip-permissions` / `--permission-mode bypassPermissions` / `--allow-dangerously-skip-permissions`、または **user / `--settings` / managed 設定**の `permissions.defaultMode: "bypassPermissions"` のいずれか（v2.1.257 以降、project / local 設定の同値は効かない）。それ以外は no-op
- `permissions.disableBypassPermissionsMode` で無効化されている場合、および restricted モードで起動したセッションでも no-op
- **`bypassPermissions` は `destination` の指定に関わらず `defaultMode` として設定ファイルに永続化されない**

`terminalSequence`（v2.1.141+）: 制御端末を持たない hook からデスクトップ通知・ウィンドウタイトル・ベル等の端末シーケンスを発行できる。例: `"\u0007"`（ベル）、`"\u001b]0;title\u0007"`（ウィンドウタイトル設定）。バックグラウンド hooks や HTTP hooks から端末 UX を制御する用途に有効。`StopFailure` / `Notification` など他の出力を破棄するイベントでも機能する。

**terminalSequence の allowlist（2026-11-05 巡回で反映）**: 許可されるのは OSC `0`/`1`/`2`（ウィンドウ・アイコンタイトル）、OSC `9`（iTerm2 / ConEmu / Windows Terminal / WezTerm 通知。`9;4` タスクバー進捗含む）、OSC `99`（Kitty）、OSC `777`（urxvt / Ghostty / Warp）、裸の BEL のみ（終端は BEL または ST）。CSI カーソル/色、OSC パレット、OSC 8 ハイパーリンク、OSC 52 クリップボード、OSC 1337 など allowlist 外を含む値は**フィールドごと無視される**。対話セッションで UI が画面に出ている間のみ書き込まれる（`-p` / Agent SDK では無視）。`WorktreeCreate` の command フックは stdout がパスとして読まれるため使えない（HTTP フックなら可）。**`async` / `asyncRewake` のフックからは機能しない**。tmux 3.3+ では `set -g allow-passthrough on` が必要（別ペインの Claude Code へ届かせるには 3.4+ の `all`）、GNU screen では Claude Code のウィンドウが表示状態である必要がある（2026-11-12 巡回で反映）。

**`suppressOutput` は効果なし（2026-11-05 巡回で修正）**: Claude Code はフィールドを受理するが動作しない。成功したフックの stdout はもともとトランスクリプトに表示されずデバッグログに記録される。

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

> `updatedInput` はツール入力を**丸ごと置換**するため、変更しないフィールドも含めて返す必要がある。**Claude Code は権限ルールの評価と Bash コマンドの[自動バックグラウンド化の可否判定](https://code.claude.com/docs/en/tools-reference#background-commands)を、Claude が送った入力ではなくフックが返した入力に対して行う**（`"defer"` の場合は無視される）。
>
> v2.1.300 から、PreToolUse / PermissionRequest フックの `updatedInput` はモデル入力と同様に検証される: 未知のキーは除去され、ツールが拒否する入力はその呼び出しを deny する。

**auto モードでの `updatedInput`（2026-11-05 巡回で反映）**: auto モードで `permissionDecision` なしに `updatedInput` だけを返すと、分類器サーバーは Claude が書いた入力をレビューしておりフック変更後の入力を見ていないため、Claude Code がその呼び出しを拒否しうる（対話ターミナル / VS Code の Anthropic API 直結セッションでは拒否文言に `a hook changed this call's input after auto mode reviewed it` を含む）。拒否後に Claude が変更済み入力をそのまま再発行するとサーバーがレビューした上でフックが再実行される（この再実行で返した `updatedInput` は破棄される）。この拒否が出ないツールもある: 当該セッションでは settings / プラグイン hooks ファイルの command フックは **Bash / PowerShell / Monitor / Edit / Write / NotebookEdit** の呼び出しを拒否なしに変更できる（同一応答内で先行呼び出しをフックが変更していた場合やサンドボックス外実行要求では拒否されうる）。

**`@` 参照はフックを通らない（2026-11-05 巡回で反映）**: プロンプトで `@` 参照したファイルはツール呼び出しなしでプロンプト構築時に挿入されるため、`Read` matcher を含むどの PreToolUse フックも発火しない。特定パスは `Read` の deny ルールでブロックでき、`CLAUDE_CODE_DISABLE_FILE_MENTIONS=1`（v2.1.299+。シェル / user settings / managed settings のみ有効）で挿入自体を無効化するとパスのみが渡され、Claude がファイルツールで読む際に PreToolUse フックが走る。

**ファイルパスの正規化（2026-11-05 巡回で反映）**: `Write` / `Edit` / `Read` の `tool_input.file_path` は常に絶対パス（`~` や相対パスはフック実行前に展開済みのためパスマッチングを回避できない）。Windows ではバックスラッシュ区切りで届く（Git Bash 下でも）ため、`/src/` のようなスラッシュ前提の比較は一致しない。比較前に区切りを正規化すること。
>
> `permissionDecisionReason` の扱い（2026-09-30 ドキュメント改訂）: `"ask"` はユーザーに表示（Claude には非表示）、`"deny"` は Claude に表示、**`"allow"` と `"defer"` はデバッグログ（`--debug`）にのみ書かれる**（従来は `"allow"` もユーザーに表示されるとされていた）。誰もプロンプトに答えられない `-p` 実行で Claude Code が呼び出しを自動拒否した場合、`"ask"` の理由は代わりにツール結果として Claude に渡る（2026-10 ドキュメント改訂で明文化）。

> フックが `"ask"` を返したときの権限プロンプトには、そのフックの出所ラベルが付く。ラベルは **`[settings]`**（settings ファイル由来またはエージェント frontmatter 由来） / **`[plugin:<name>]`**（プラグイン由来） / **`[skill]`**（スキル frontmatter 由来）の 3 種（従来ドキュメントの `[User]` / `[Project]` / `[Plugin]` / `[Local]` から変更）。

**auto モードでの PreToolUse `"ask"`（2026-10-30 ドキュメント改訂で節新設）**: auto モードでは PreToolUse フックの `"ask"` は通常そのまま権限プロンプトに送られ、分類器は判定しない。例外が 2 つ — ①**Claude Code 自身のチェックにも引っかかる呼び出し**（保護パスへの書き込み、サンドボックス外実行の要求など）は、プロンプト表示前に分類器が**拒否だけは**できる（承認された場合は通常どおりプロンプトへ）。②**`SubagentHandback`**（サブエージェントのレポート納品ツール）はプロンプトを出さず、分類器が反対しても実行される。`SubagentHandback` を止めるには `"deny"` を返す — サブエージェントは最大 3 回「レポート未納品、再呼び出しせよ」と告げられ、フックが拒否し続けるとレポート未納品のままサブエージェントが終了し、待っていた会話にその旨の注記が入る。

**`permissionDecision: "defer"` の動作（2026-10 ドキュメント改訂で詳細化）**: `"defer"` は `claude -p` をサブプロセスとして動かし JSON 出力を読む統合（Agent SDK アプリ、Claude Code 上のカスタム UI 等）向け。ツール呼び出しで Claude を一時停止し、呼び出し側が自前の UI で入力を集めてから再開させる。**`-p` の非対話モードでのみ有効**で、対話セッションでは警告をログして無視される。典型例は `AskUserQuestion`（答えるターミナルが無い場合）。

- フックが `"defer"` を返すとツールは実行されず、ターンが `stop_reason: "tool_deferred"` の結果で終了する（pending のツール呼び出しはトランスクリプトに残る）。結果の `deferred_tool_use` オブジェクトにツールの `id` / `name` / `input`（実行前に Claude が生成したパラメータ）が入る
- 呼び出し側は `claude -p --resume <session-id>` で再開する。同じツール呼び出しで `PreToolUse` が**再発火**するので、`"allow"`＋`updatedInput` で回答を渡してツールを実行させる（答えがまだ無ければ再度 `"defer"` も可。プロンプト無しで再開し続け、`"allow"` / `"deny"` でループを抜ける）
- **再開時にツールが利用不能**（提供元 MCP サーバーが再開セッションで未接続等）の場合、再開の最初の結果は `stop_reason: "tool_deferred_unavailable"`＋`is_error: true` となり、**フックは発火しない**（`deferred_tool_use` は含まれるのでどのツールかは特定できる）。**v2.1.319 以降**、呼び出しが pending の間はどのプロンプトも Claude に届かず、run は exit code 1 で終了する（`result` は `<tool> isn't available, so its deferred call didn't run.` で始まる）。解消は①ツールを利用可能にして再開、②`CLAUDE_CODE_DROP_UNAVAILABLE_DEFERRED_TOOL=1` を付けてプロンプト付きで再開（呼び出しにはエラーが結果として渡り Claude がプロンプトに回答。空 `result` の `tool_deferred_unavailable` 結果→回答結果の2件が出力され、回答がエラーでなければ exit 0。変数のみでプロンプト無しなら drop して exit 1）。v2.1.319 より前はプロンプト付き再開でツール結果なしに回答し呼び出しを破棄していた
- **ターン内のツール呼び出しが 1 つの場合のみ**有効。複数同時呼び出し（バッチ）では警告付きで無視され、通常の権限フローへ流れる（resume が再実行できるのは 1 ツールだけのため）
- **タイムアウト・リトライ上限は無い**。セッションは再開までディスクに残るが、`cleanupPeriodDays`（既定 30 日）の保持スイープの対象
- プランモードの deferred セッションを再開するには `--resume` に加えて `--permission-prompt-tool` が必要（v2.1.246+）。また `-p` での再開は保存済み権限モードを復元しない（新規 `claude -p` と同じモードで開始）ため、必要なら `--permission-mode` / `--dangerously-skip-permissions` を再指定する
- 複数の PreToolUse フックが異なる決定を返した場合の優先順位は `deny` > `defer` > `ask` > `allow`。`"defer"` では `updatedInput` / `additionalContext` は無視され、`permissionDecisionReason` はデバッグログにのみ書かれる

**ユーザー操作が必要なツール（AskUserQuestion / ExitPlanMode、2026-11-09 巡回で反映）**: `-p` 非対話モードでは両ツールは permission host（`--permission-prompt-tool` / SDK の `canUseTool`）がある場合のみ提供される。PreToolUse フックがユーザーの代わりに回答するには `permissionDecision: "allow"` に加えて**回答を入れた `updatedInput` が必須**（`allow` 単独では不十分）。`AskUserQuestion` では元の `questions` 配列をそのまま返しつつ `answers`（質問文→回答のマップ）を追加する。なお **`_meta["anthropic/requiresUserInteraction"]` 付きの MCP ツールは、フックの `allow` では承認プロンプトをスキップできない**（`updatedInput` の有無を問わず）。

**ExitPlanMode / AskUserQuestion の入力詳細（2026-11-12 巡回で反映）**: `ExitPlanMode` は Claude がプランをファイルに書いてから呼ぶため**モデル由来の `tool_input` は通常空**で、Claude Code が `plan`（Markdown 本文）と `planFilePath` を注入してからフックに渡す。`allowedPrompts` は非推奨（受理されるが無視。v2.1.205 より前の prompt-based permissions の名残）。`PostToolUse` の `tool_response` は承認済みプランを `plan` / `filePath` に持つので、**ファイルを再読込せず `tool_response.plan` を読む**こと。`AskUserQuestion` の `questions[]` は各要素が `question` / `header` / `options[]`（`label` / `description`）/ 任意の `multiSelect` を持ち、`answers` は質問文→選択ラベルのマップ（複数選択はラベルをカンマ連結。Claude は設定しないため `updatedInput` で与える）。

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

`updatedMCPToolOutput` は MCP ツール専用の出力差し替えフィールド（全ツール対応の `updatedToolOutput` を推奨）。組み込みツールでは出力スキーマに合わない差し替え値は無視され元の出力が使われるが、MCP ツール出力はスキーマ検証なしで通る（2026-11-05 巡回で反映）。例えば `Bash` の `tool_response` は `stdout` / `stderr` / `interrupted` / `isImage` を持つオブジェクトで、**差し替え値もこの形に一致させる必要がある**（2026-11-12 巡回で反映）。また **Cowork クラウドセッションでは `AskUserQuestion` / `ExitPlanMode` への `updatedToolOutput` は無視され**、Claude は元の回答・プランを読む。

`classifierContext`（v2.1.236+）: `hookSpecificOutput.classifierContext` で auto mode 分類器宛ての注記をツール結果に添付できる（**1ツール呼び出しあたり 2,000 文字上限**）。設定元により重み付けが異なり、Claude Code 設定由来は未検証コンテキスト扱い、in-process Agent SDK コールバック由来はユーザー意図として扱われうる。async フックでは無視、読み取り専用呼び出しでは破棄。`updatedToolOutput` と同時返却可。

**割り込み後の PostToolUse（v2.1.319+）**: ツール実行中にメッセージ送信・割り込みをした場合の挙動がキー別に規定された。`Ctrl+Enter`（ターン終了）ではツールが完走すれば PostToolUse フックはその結果に対して実行され、**10秒以内に完了する猶予**を持つ（ただし `!` コマンドをキュー済み・`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` でバックグラウンドタスク無効の場合は Esc と同じ扱い）。`Enter`（キュー送信）はターンを中断しないためフックは通常の `timeout` のまま。`Esc` ではフックは即座にキャンセルされる。10秒を過ぎると実行中の command フックは **`timeout` 設定に関わらず kill** される。長いフックは `async: true` で実行する（ただしブロック・出力書き換えは不可）。v2.1.319 より前は、メッセージでターンを中断するとフックはそもそも開始されず、実行中のフックも停止されていた。

**matcher 選択の注意（2026-11-09 巡回で反映）**: `Edit|Write` matcher の PostToolUse フックは **Bash コマンドや外部プロセスが同じファイルを書き換えた場合は発火しない**（その用途は `FileChanged`）。全ツール対象なら matcher 省略＋`git status --porcelain` で自力検出する。

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

**v2.1.143+ のブロック上限**: ブロックを繰り返す stop hook が無限ループする問題への安全装置として、**8 連続ブロックでターンが警告とともに自動終了** するように。連続カウントは Claude がツールを呼び出すたびにリセットされる（2026-10-06 ドキュメント改訂で明文化）。上限は環境変数 `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` で変更可能。

**v2.1.163+ の `additionalContext` サポート**: `Stop` および `SubagentStop` フックは `hookSpecificOutput.additionalContext` を返すことで、Claude にフィードバックを与えてターンを継続させられる。従来は同等の動作が「hook エラー」扱いだったが、通常応答として扱われるようになった。

```json
{
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "additionalContext": "テストが未通過なので修正を続けてください"
  }
}
```

#### UserPromptSubmit

出力: 全 JSON 出力フィールドが利用可能。`hookSpecificOutput` で `additionalContext`（コンテキスト追加）、`sessionTitle`（プロンプト内容からのセッション命名）、`suppressOriginalPrompt` を返せる。

**タイムアウト時の挙動（2026-10 ドキュメント改訂で明文化）**: `async: true` の command フックを除き、command / http / mcp_tool の `UserPromptSubmit` フックがタイムアウトに達すると**キャンセルされ、`additionalContext` を含む出力は破棄される**。プロンプトはそのコンテキストなしで Claude に届く（トランスクリプトにフック名・発火したタイムアウト・出力破棄を示す通知が出る）。タイムアウトでプロンプトをブロックしたい場合は command / HTTP フックに `onFailure: "block"` を指定する（§8 参照）。**例外**: Agent SDK のコールバックフックが `UserPromptSubmit` でタイムアウトした場合は、フック名とタイムアウトを示すメッセージとともに**プロンプトをブロックする**（ポリシーゲートとして動くコールバックを fail-open にしないため。セッション自体は継続）。

**プロンプトチェック中の `Esc`（2026-10 ドキュメント改訂で明文化）**: フックがプロンプトをチェックしている間に `Esc` やセッション割り込みを行うとチェックは停止し、プロンプトは Claude に届かない。入力したテキストは入力欄に戻り、再送時に改めてチェックされる。**例外**: 割り込みと同時にフックがブロックした場合、プロンプトはブロックされたまま（そのフックのブロックメッセージが表示される）でテキストは入力欄に戻らない。

**ブロックしたプロンプトの残留（2026-10 ドキュメント改訂で明文化）**: `decision: "block"`（または終了コード 2）でブロックしたプロンプトは Claude には届かないが、テキストが全消去されるわけではない:

- 既定では、ユーザーに表示されるブロックメッセージの末尾に `Original prompt:` ＋送信テキストが付き、**そのメッセージはトランスクリプトファイルにも書き込まれる**
- `hookSpecificOutput.suppressOriginalPrompt: true` を返すとブロックメッセージから原文を省略できる（`decision: "block"` でも exit 2 でも有効）
- ただし `suppressOriginalPrompt` が変えるのは**ブロックメッセージだけ**。送信テキストはトランスクリプトやプロンプト履歴などローカルファイルに残りうるため、**ブロックフックはシークレットをディスクに残さない手段にはならない**（制限するには公式ドキュメントの Plaintext storage / Clear local data を参照）

```json
{
  "decision": "block",
  "reason": "このプロンプトは送信できません",
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "suppressOriginalPrompt": true
  }
}
```

#### SessionStart

> **起動時・`--continue` / `--resume` 起動時・`/clear` 時はバックグラウンド実行**（2026-09-17 ドキュメント改訂）: マッチする `SessionStart` フックはバックグラウンドで走り、プロンプトはすぐに入力を受け付ける（再開した会話もフックを待たずに表示される）。Claude の最初の応答はフック完了を待つのでコンテキストは届く。**セッション内の `/resume` での会話切り替えは逆にフック完了を待つ**。バックグラウンドフック実行中に `/clear` や別会話への切り替えを行うと、そのフックの返す内容はセッションに適用されない。
>
> **起動時（resume 含む）も同じ待機が適用される（v2.1.271 で明文化）**: SessionStart フックの実行中に送ったプロンプトはフック完了まで Claude に届かない。待機中に `Esc` を押すとプロンプトを送信せず入力欄に取り戻せる（フックは走り続ける）。フック実行中はスピナーに経過時間付きで表示される（SessionStart / UserPromptSubmit / PreToolUse / SessionEnd 共通）。

環境変数の永続化に `CLAUDE_ENV_FILE` を使用:

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

**SessionStart の出力フィールド（2026-11-05 巡回で整理）**:

- `additionalContext`: 会話冒頭（最初のプロンプトの前）に追加されるコンテキスト。プレーンテキスト stdout でも同じ効果
- `initialUserMessage`: **`-p` 非対話モードで最初のユーザーメッセージとして使われる文字列**。プロンプトを渡さなくても最初のターンになる（渡したプロンプトは次のターン）（2026-11-05 巡回で反映）
- `sessionTitle`: セッションタイトルを設定（`/rename` と同じ効果）。`source` が `startup` / `resume` / `fork` のとき適用（v2.1.152）
- `watchPaths`: このセッションで `FileChanged` が監視する絶対パスの配列（2026-11-05 巡回で反映）
- `reloadSkills: true`: フックでインストールしたスキルディレクトリを同一セッション内で再スキャンしてサーフェスできる（v2.1.152）

プラグインの SessionStart フックが `initialUserMessage` / `sessionTitle` を返す場合、SessionStart フック実行後にインストールが完了したプラグインの両フィールドは無視される。

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

Command フック: stdout にワークツリーの絶対パスを出力（**最後の非空行**が読まれる。ANSI エスケープは除去されるため起動バナーは無視される。その他の出力は stderr へ）。
HTTP フック (`type: "http"`): レスポンスJSON の `hookSpecificOutput.worktreePath` でパスを返す。

**パス検証と発火条件の注意（2026-11-05 巡回で反映）**:

- 相対パスはフック実行ディレクトリ基準で解決（`.` / `..` は折りたたみ）。**`.` / `..` セグメントを含む絶対パス、およびリポジトリルート以下で symlink を通過するパスは拒否される**（リポジトリにコミットされた symlink によるワークツリー外リダイレクト対策）
- Bash 経由の `git worktree add` では発火しない（検査・ブロックは `Bash` matcher の PreToolUse で行う）
- `--worktree` と `--tmux` の併用時はプラグイン読み込み前にワークツリーが作られるため、**プラグイン提供の WorktreeCreate フックは走らない**（settings ファイルに定義すれば走る）。v2.1.300 より前は `--worktree` 単独でもプラグインのフックは走らなかった
- 入力は `worktree_path` ではなく **`name`**（ユーザー指定または自動生成のスラグ、例 `bold-oak-a3f2`）
- **`EnterWorktree` / `ExitWorktree` ツール経由でも発火する（2026-11-12 巡回で反映）**: Claude がセッション中に `EnterWorktree` で worktree を作ると WorktreeCreate フックが走り、`ExitWorktree` や `claude remote-control` のクリーンアップによる削除では WorktreeRemove フックが**確認プロンプトなしで**走る。`ExitWorktree` はフック作成の worktree を `discard_changes: true` なしでは削除拒否するため、この経路ではフックが最後のチェックになる

```bash
#!/bin/bash
NAME=$(jq -r .name)
DIR="$HOME/.claude/worktrees/$NAME"
svn checkout https://svn.example.com/repo/trunk "$DIR" >&2 && echo "$DIR"
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

> **複数フックと長大な値の扱い**: 同一イベントで複数のフックが `additionalContext` を返した場合、Claude はそのすべてを受け取る。**1 つの値が 10,000 文字を超えると、Claude Code は全文をセッションディレクトリのファイルに書き出し、Claude にはファイルパスと先頭最大 2,000 文字のプレビューを渡す**（2026-09-18 ドキュメント改訂で詳細化）。上限の対象は `additionalContext` / `systemMessage` / `initialUserMessage` の各文字列とプレーン stdout で、**同一イベントで複数フックが走っても文字列ごとに個別に計測**される（JSON 出力はフィールド単位、プレーン stdout は全体）。Bash の出力上限と違い**この上限を引き上げる設定・環境変数は無い**。また Claude Code はファイルを読むよう Claude に指示しないため、必ず見せたい内容は上限内に収めること。

> **additionalContext の取り扱い（2026-11-12 巡回で反映）**: 文字列に `<system-reminder>` / `</system-reminder>` タグが含まれる場合、そのタグの `<` は `&lt;` に置換されて Claude に渡る。注入テキストはセッショントランスクリプトに保存され、`--continue` / `--resume` では **mid-session イベント（`PostToolUse` / `UserPromptSubmit` 等）の過去ターン分はフックを再実行せず保存済みテキストを replay** する（タイムスタンプや commit SHA は陳腐化する。`SessionStart` のみ `source: "resume"` / `"fork"` で再実行されるためコンテキストを更新できる）。文面は命令調でなく**事実の記述**で書くこと（out-of-band のシステム命令風の文はプロンプトインジェクション防御に引っかかり、コンテキストとして扱われずユーザーへの表示に回される）。

> **`SubagentStart` の再発火時の注入**: 同一サブエージェントに対してフックが再び走った場合、Claude Code は**そのサブエージェントのコンテキストに前回分のコピーがまだ残っていないときだけ**返却コンテキストを注入する。起動時に注入したコピーはそのまま残り、サブエージェントの[プロンプトキャッシュ](https://code.claude.com/docs/en/prompt-caching#subagents-and-the-cache)を壊さない。auto-compaction がそのコピーを破棄した後は、次回実行分のコンテキストが改めて注入される。

`decision: "block"` または `continue: false` で次のモデル呼び出し前にエージェントループを停止できる（メッセージは `reason` / `stopReason`、または終了コード 2 の stderr から）。トランスクリプトには警告として残り会話にも残るため、会話継続時に Claude から見える。

#### MessageDisplay（2026-11-12 巡回で詳細反映）

入力（共通フィールドに加えて）:

| フィールド | 説明 |
|:--|:--|
| `turn_id` | 現在のターンの UUID |
| `message_id` | 表示中アシスタントメッセージの UUID（同一メッセージの全バッチで不変）。**API の `msg_…` ID とは別物**でトランスクリプトのメッセージ ID と紐付けできない |
| `index` | メッセージ内のバッチ番号（0 起点） |
| `final` | メッセージ最後のバッチで `true`（final バッチは各メッセージに正確に 1 つ） |
| `delta` | 前バッチ以降に確定した行（終端改行込み）。常に行単位だが、最終バッチのみ行途中で終わりうる。**対話実行ではメッセージが改行で終わると最終バッチの `delta` が空になるため、終端判定は delta の有無でなく `final` で行う** |

出力: `hookSpecificOutput.displayContent` で画面表示を `delta` 単位で置換できる（省略で原文表示）。**表示のみ**の変換で、トランスクリプトと Claude に渡る内容は変わらない。決定制御はなく、`systemMessage` / `continue` は破棄される。

- 非対話実行（Agent SDK / `claude -p`）ではバッチ単位でなく**メッセージ毎に 1 回**発火し、`index: 0`, `final: true`, `delta` = メッセージ全文で届く
- テキストを含まないメッセージでは発火しない
- 各バッチの描画はフック応答まで保留される（遅いフックは表示を遅らせる。既定タイムアウト 10 秒）

## 7. スクリプトパス参照

`command` フィールドで使用可能な変数:

| 変数 | 説明 |
|:--|:--|
| `$CLAUDE_PROJECT_DIR` | プロジェクトルート |
| `${CLAUDE_PLUGIN_ROOT}` | プラグインインストールディレクトリ |
| `${CLAUDE_PLUGIN_DATA}` | プラグイン永続データディレクトリ |
| `$CLAUDE_CODE_REMOTE` | Web環境で `"true"` |

**PowerShell フックでの書き換え（2026-11-12 巡回で反映）**: PowerShell の shell form コマンドでは、`${CLAUDE_PROJECT_DIR}` / `${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_PLUGIN_DATA}` プレースホルダは PowerShell の `${env:NAME}` 形へ書き換えられる（settings / プラグイン / スキル定義を問わず）。値は PowerShell のパース後に環境変数から解決されるため、ダブルクォート内では展開されるが**シングルクォート内では展開されない**。裸の `$CLAUDE_PROJECT_DIR` 表記は書き換え対象外で、未定義ローカル変数として `$null` に解決される（デバッグログに警告が出る）ため使わないこと。

---

## 8. 共通フィールド

| フィールド | 説明 | デフォルト |
|:--|:--|:--|
| `timeout` | タイムアウト（秒）。**`async: true` の command フックには適用されない**（バックグラウンドに入った後は Claude Code が強制終了しない）。`asyncRewake` のフックには引き続き適用される。`UserPromptSubmit` / **`PreModelSwitch`（v2.1.251+）/ `PostModelSwitch`** では command / http / mcp_tool の既定が 30 に、`MessageDisplay` では 10 に下がる。**`UserPromptSubmit` のタイムアウト時はフックがキャンセルされ出力は破棄、プロンプトは素通りする（Agent SDK コールバックフックのみブロック。§6.4 参照）**。`SessionEnd` は全フックで 1.5 秒の共通予算（予算は settings 内の最大 per-hook `timeout` まで自動で引き上がる・上限60秒。プラグイン提供フックの `timeout` は予算を上げない。`CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` で明示上書き可能で、**この値は `timeout` 未指定の各フックのタイムアウトにもなる**。v2.1.268 より前は予算のみ上がり個別フックは1.5秒のままだった） | command / http / mcp_tool: 600, prompt: 30, agent: 60 |
| `async` | バックグラウンド実行（command のみ）。決定系フィールド（`decision` / `permissionDecision` / `continue`）は無効。完了後の `additionalContext` / `systemMessage` は次の会話ターンで Claude に届く（ユーザーには表示されない）。JSON 応答は stdout 単独（複数行可、v2.1.295+）か単独の1行で出力し、型が合わないフィールドは破棄される。`-p` セッション終了時に実行中の async フックは kill され `cancelled` 扱い（2026-11-05 巡回で詳細反映） | `false` |
| `asyncRewake` | バックグラウンド実行し、**exit 2 で Claude を起床させる**（セッションがアイドルでも即時）。stderr（空なら stdout）が system reminder として Claude に表示される。長時間バックグラウンド処理の失敗に反応させる用途。`timeout` は適用される（2026-11-05 巡回で反映） | `false` |
| `once` | **初回の成功実行後**にフックを取り除く。失敗・exit code 2 によるブロック・タイムアウトの場合はフックが残り、次の一致イベントで再実行される。スキル frontmatter で宣言したフックのみ有効（settings ファイルとエージェント frontmatter では無視） | `false` |
| `statusMessage` | スピナーメッセージ | - |
| `if` | ツールイベント専用の条件フィルタ（permission rule構文） | - |
| `onFailure` | **`"block"` を指定すると、フックが起動できない・タイムアウトした・想定外の exit code で終わった場合にアクションをブロックする**（command / HTTP フック。v2.1.295+）。既定ではこれらは非ブロッキングエラー扱いで素通りするため、ポリシーフックを fail-closed にしたい場合に指定する（2026-10-11 巡回で公式 hooks ページに「Block the action when a hook fails」節として正式収載を確認。PreToolUse のタイムアウトは既定で通常の権限フローに流れて素通りするため、タイムアウト時もブロックしたい場合に有効） | 非ブロッキング（fail-open） |

### 8.0 実行上限・fail-closed 化（2026-10 の変更）

- **prompt / agent フックは同一イベント最大 12 並列**で実行され、残りは順番待ちになる（v2.1.302）
- **スキルフックの上限**: 1 ソース（リポジトリ / 自分のフォルダ / プラグイン）のスキル群はセッションあたり最大 1,024 フック・16,384 matcher 文字まで登録可能（v2.1.304）
- **プラグインフックがブロック可能なアクションは、フックにイベントを渡せない / 回答を受け取れない場合に素通りでなく拒否される**（実行済みツールは結果が差し止められる）（v2.1.299）
- プラグインフック・ワークフロースクリプトが返す値は 28,000 レベル超のネストを拒否（v2.1.299）
- **観測性（v2.1.296+、2026-11-05 巡回で反映）**: command フックは完了時にデバッグログへ `Hooks: PostToolUse:Write [.claude/hooks/log-write.sh] finished with status 0 (31ms)` 形式の行を残す（`timed out after <N>ms` / `cancelled` / `moved to the background` / `failed to start` もあり）。遅いフックの特定には `Hooks:` 行を検索する。`CLAUDE_CODE_DEBUG_LOG_LEVEL=verbose` で matcher マッチングの詳細も出力される。スキルフックが上限超過で登録されなかった場合はデバッグログに `registers none of its hooks` が出る

### 8.1 `if` 条件フィールド（v2.1.85）

`if` フィールドはツールイベント（`PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, **`PermissionDenied`**）でのみ評価される。他のイベントでは `if` を持つフックは実行されない。`if` は**単一ルールのみ**（`&&` / `||` / リスト構文は不可。複数条件は別ハンドラに分ける）。Bash に対するマッチングは best-effort: 先頭の `VAR=value` は除去して評価、複合コマンドはサブコマンドごとに評価（`$()` / バックティック内も対象）、コマンド名以上を指定したパターンは `$()` / `$VAR` が相手だと無条件実行扱い、判定不能時は常に実行される（強制が必要なら permission システムを使う）。

> **`PermissionDenied` の `retry`**: JSON 出力の `hookSpecificOutput.retry: true` で「拒否されたツール呼び出しを再試行してよい」とモデルに伝えられる。分類器 verdict の無い拒否では無視される。終了コードと stderr は無視（拒否は既に発生済み）。

matcher と組み合わせた2段階フィルタリング:
1. `matcher` でツール名をフィルタ
2. `if` でツール引数をさらに絞り込み

**構文**: permission rule構文を使用
- `"Bash(rm *)"` — `rm` で始まるBashコマンド
- `"Edit(*.ts)"` — TypeScriptファイルへの編集
- `"Bash(git push *)"` — git push コマンド
- `"Write(**/config.json)"` — config.json への書き込み

ファイルツールの `if` で `"Edit(src/**)"` のような**単一セグメントのディレクトリパターンはワーキングディレクトリ直下の `src` のみ**に一致する。任意の深さの `src` に一致させるには `"Edit(**/src/**)"` と書く（2026-11-09 巡回で反映）。

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
