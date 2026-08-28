# Claude Code ツール仕様書

最終更新: 2026-08-23（巡回で新規作成）

公式ドキュメント: https://code.claude.com/docs/en/tools-reference

---

## 1. 概要

Claude Code の組み込みツール一覧。**ここに書かれたツール名は文字列として完全一致で使われる**——
`permissions.allow` / `deny` / `ask` の権限ルール、サブエージェント定義の `tools` / `disallowedTools`、
フックの `matcher`、`--allowedTools` / `--tools` フラグはすべてこの名前を参照する。
ハーネス設計で権限ルールやフックを書く際の正である。

- カスタムツールの追加は MCP サーバー経由。**スキルは新しいツールを増やさず** 既存の `Skill` ツールを通して動く
- Pro / Max / Team プランでは既定で Auto Mode で開始するため、下表の「権限」列が Yes でも分類器が判断を代行することが多い
- 実セッションのツール構成はプロバイダ・プラットフォーム・設定で変わる。確認は Claude に「What tools do you have access to?」と聞く（MCP の正確なツール名は `/mcp`）
- **Advisor はサーバーツール**（API 側が実行）であり Claude Code が実装するツールではない。権限ルールで参照できる名前を持たない

---

## 2. ツール一覧

「権限」= 既定で権限プロンプトが出るか。

| ツール | 権限 | 説明 |
|:--|:--|:--|
| `Agent` | No | サブエージェントを独立コンテキストで起動。Agent Teams 有効時、`name` 付き呼び出しはチームメイトを起動しうる |
| `Artifact` | Yes | HTML / Markdown ファイルを claude.ai 上の非公開インタラクティブページ（Artifact）として公開 |
| `AskUserQuestion` | No | 選択式の質問。既定では回答するまで開いたまま（`askUserQuestionTimeout` で自動継続可） |
| `Bash` | Yes | シェルコマンド実行 |
| `PowerShell` | Yes | PowerShell をネイティブ実行 |
| `CronCreate` / `CronDelete` / `CronList` | No | セッションスコープの定期・単発プロンプトのスケジュール。`--resume` / `--continue` で未期限のものは復元される |
| `Edit` | Yes | ファイルの部分編集（完全一致文字列置換） |
| `Write` | Yes | ファイル新規作成・全上書き（追記・マージはしない） |
| `Read` | No | ファイル読み取り |
| `NotebookEdit` | Yes | Jupyter ノートブックのセル編集 |
| `Glob` | No | ファイル名パターン検索 |
| `Grep` | No | ファイル内容検索（ripgrep ベース） |
| `LSP` | No | 言語サーバー経由のコードインテリジェンス（定義ジャンプ・参照検索・型エラー報告等） |
| `Monitor` | Yes | バックグラウンドでコマンドを走らせ出力行ごとに Claude へ返す。WebSocket を開いて各メッセージをイベント扱いすることも可能 |
| `EnterPlanMode` | No | プランモードへ移行 |
| `ExitPlanMode` | Yes | プランを提示して承認を求め、プランモードを抜ける |
| `EnterWorktree` | Yes | 隔離 git worktree を作成して移動。`path` 指定で既存 worktree に入る |
| `ExitWorktree` | No | worktree セッションを抜けて元ディレクトリへ戻る。`isolation: worktree` 等で既に自前の作業ディレクトリを持つサブエージェントには提供されない |
| `Skill` | Yes | メイン会話内でスキルを実行 |
| `Workflow` | Yes | 動的ワークフロー（多数のサブエージェントをバックグラウンドで統率し統合結果を返すスクリプト）を実行。**v2.1.248 でツール説明が約 5.7k → 約 1k トークンに縮小**され、スクリプト作成リファレンスは同梱スキル `workflow-authoring` へ移動 |
| `TaskCreate` / `TaskGet` / `TaskList` / `TaskUpdate` | No | タスクリストの作成・取得・列挙・更新。**モデルによっては既定で非提供**（§4 参照） |
| `TodoWrite` | No | セッションのタスクチェックリスト。`TaskCreate` 系に置き換えられ既定で無効。`CLAUDE_CODE_ENABLE_TASKS=0` で復帰 |
| `TaskOutput` | No | バックグラウンドタスクの出力取得。**非推奨**（タスクの出力ファイルパスを `Read` する方式が推奨） |
| `TaskStop` | No | バックグラウンドタスクを ID で停止。v2.1.198 以降はチームメイトや名前付きバックグラウンドエージェントも受け付ける |
| `ListAgents` | No | `SendMessage` の宛先になりうるエージェント一覧（セッション内サブエージェント、チームメイト、他のローカルセッション、Remote Control 接続中は他マシン／クラウドのセッション） |
| `SendMessage` | No | 他エージェント（チームメイト、agent ID / 名前で再開するサブエージェント、自分の他セッション）へメッセージ送信 |
| `SendUserFile` | No | セッションからユーザーへファイル送信（任意のキャプション付き）。v2.1.196 以降 `display` パラメータあり |
| `PushNotification` | No | デスクトップ通知（Remote Control 接続時はスマートフォンへのプッシュも）を送る |
| `ScheduleWakeup` | No | 自己ペースの `/loop` の次イテレーション時刻を 1 分〜1 時間の範囲で再スケジュール |
| `RemoteTrigger` | No | claude.ai 上の Routines の作成・更新・実行・列挙。`/schedule` コマンドの実体 |
| `SendFeedback` | No | Claude Code へのフィードバックレポート（製品不具合／Claude 自身の挙動）を起草し、ローカルにキューする。送信はユーザーの操作を待つ。v2.1.238+（§6 参照） |
| `ReportFindings` | No | コードレビュー指摘を構造化リスト（file / summary / failure scenario）として報告し、Claude Code 側でレンダリングさせる |
| `ShareOnboardingGuide` | Yes | `ONBOARDING.md` をアップロードし、チームメイトが Claude Code で開ける共有リンクを返す。`/team-onboarding` から呼ばれる |
| `WebFetch` | Yes | URL の取得と抽出 |
| `WebSearch` | Yes | Web 検索 |
| `ToolSearch` | No | tool search 有効時にディファードツールを検索・ロード |
| `WaitForMcpServers` | No | バックグラウンドで接続中の MCP サーバーを待つ（セッション再起動なしにツールを使えるようにする） |
| `ListMcpResourcesTool` / `ReadMcpResourceTool` | No | MCP リソースの列挙 / URI 指定読み取り |
| `EndConversation` | No | セッションを終了（§5 参照） |

---

## 3. ハーネス設計で押さえるべき挙動

### 3.1 Bash

- コマンドごとに別プロセス。**環境変数は持ち越されない**（`export` は次のコマンドに効かない）。永続化は `CLAUDE_ENV_FILE` か SessionStart フックで行う
- `cd` はプロジェクトディレクトリまたは追加作業ディレクトリの内側にいる限り持ち越される。外に出ると自動リセットされ結果に `Shell cwd was reset to <dir>` が付く。`CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1` で持ち越し自体を無効化
- シェル起動ファイル（`~/.zshrc` / `~/.bashrc` / `~/.profile`）のエイリアス・関数はセッション開始時にキャプチャされ利用可能
- タイムアウト: `BASH_DEFAULT_TIMEOUT_MS`（既定 2 分）と `BASH_MAX_TIMEOUT_MS`（既定 10 分の上限）。**per-command のタイムアウトは Claude が `timeout` 引数で指定する**ものでユーザーが個別設定するものではない
- 出力: 実行中は作業ファイルへストリーム。**5 GB 超で kill**。成功時はおよそ 30,000 文字までインライン、超過分はセッションディレクトリのファイルパス（64 MiB で切り詰め）+ 冒頭プレビュー。失敗時はおよそ 10,000 文字までインライン。`BASH_MAX_OUTPUT_LENGTH` で読み戻し量を変更可（既定 30,000、上限 150,000）
- **終了コード 1 が「正常な否定」として扱われるコマンド**: `grep`, `rg`, `egrep`, `fgrep`, `find`, `diff`, `test`, `[`, `git diff`, `git grep`
- バックグラウンド化: `run_in_background: true`。タイムアウトに達したコマンドも自動でバックグラウンドへ移される。ただし **`sleep` 始まり / `git` を含む / 完全にパースできない複合コマンド**の 3 種は自動バックグラウンド化されず停止される
- Linux / WSL では `CLAUDE_CODE_TOOL_MEMORY_LIMIT`（例 `4G`）で Bash / PowerShell、および **v2.1.246 以降は Monitor** ツールコマンド全体のメモリ上限を cgroup で設定できる。**セッション内の全コマンド合算**で、値変更の反映には再起動が必要（v2.1.246 より前は Monitor は上限外）
- v2.1.246 以降、`CLAUDE_CODE_TOOL_MEMORY_CGROUP_EXCLUDE` で Claude Code が起動する他種別のプロセス（`mcp` / `lsp` / `hooks` / `plugin` / `helper` / `agent`）も同じ上限の対象にできる。列挙した種別が**除外**され、それ以外は上限対象。`none` で全種別対象、`all-new` で Bash / PowerShell / Monitor のみ対象。未設定時の対象集合はサーバー配信の構成に従って変動する。権限ゲート系フック（およびそれが呼ぶ MCP サーバー）は常に除外される

### 3.2 PowerShell

- 有効化条件: **Windows で Git Bash 無し** → 自動有効。**Windows で Git Bash 有り** → claude.ai / Console アカウントは既定 ON、Bedrock / Vertex / Foundry は `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` が必要。**Linux / macOS / WSL** → オプトイン（PowerShell 7+ の `pwsh` が PATH に必要）
- **フックでシェルコマンドを検査する場合は `Bash|PowerShell` でマッチすること。`Bash` だけでは不十分**
- 関連設定は 3 系統: `defaultShell: "powershell"`（`!` コマンド）、コマンドフックの `shell: "powershell"`（ツール有効化に依存せず動く）、スキルフロントマターの `shell: powershell`（`` !`cmd` `` ブロック）
- プレビュー中の制限: PowerShell プロファイルは読み込まれない / Windows ではサンドボックス非対応

### 3.3 Read / Edit / Write

- Read は行番号付きで返す。トークン上限超過時は `PARTIAL view` 通知付きで先頭ページのみ返り、`offset` / `limit` で続きを読む。**`PARTIAL view` の読みは read-before-edit を満たさない**
- Read は画像（視覚コンテンツとして返る）・PDF（10 ページ超は `pages` で範囲指定、1 回 20 ページまで）・`.ipynb`（全セル + 出力、100 MB 超は拒否）に対応。**ディレクトリは読めない**
- Edit は正規表現でもファジーでもない**完全一致置換**。`old_string` はファイル内に**ちょうど 1 回**出現する必要がある（複数なら文脈を足すか `replace_all: true`）
- read-before-edit: Opus 4.6 / Haiku 4.5 以前は常に必須。新しいモデルは「読んでも権限プロンプトが不要」かつ「Read ツールが使える」場合に限り未読ファイルの上書きが可能（v2.1.228 以降。ノートブックと `PARTIAL view` は全モデルで必須）
- **Bash での閲覧も read-before-edit を満たす**: `cat`, `nl`, `bat`, `batcat`, `head`, `tail`, `sed -n 'X,Yp'`, `grep`, `egrep`, `fgrep`, `rg` を単一ファイルにパイプ・リダイレクトなしで使った場合
- **`Read` の deny ルールにマッチするパスは Edit / Write も拒否される**（そこへの新規ファイル作成も含む）。Read / Edit の deny ルールは Claude Code が認識する Bash のファイルコマンド（`cat`, `head`, `tail`, `sed` 等）にも適用される

### 3.4 Glob / Grep

- Glob は更新時刻順、**100 件で打ち切り**（切り詰めフラグが Claude に見える）。**既定で `.gitignore` を尊重しない**（`CLAUDE_CODE_GLOB_RESPECT_GITIGNORE` 系で変更）
- Grep は ripgrep ベースで **POSIX grep ではなく ripgrep の正規表現構文**。`.gitignore` を尊重する（gitignore されたファイルはパスを直接指定すれば読める）
- 出力モード: `files_with_matches`（既定） / `content` / `count`

### 3.5 WebFetch / WebSearch

- WebFetch は取得したページを小型・高速モデルで抽出プロンプトに対して処理する。**設計上ロッシー**——「ページに書かれていない」という結果は「プロンプトが聞かなかっただけ」の可能性がある
- HTTP は HTTPS に自動昇格。**別ホストへのリダイレクトは追わず**、元 URL とリダイレクト先を返すので Claude が 2 回目の呼び出しをする
- レスポンスは既定 15 分キャッシュ（v2.1.233 以降 `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` で変更可）
- Manual（`default`）/ `acceptEdits` モードでは取得前にプロンプトが出るが、権限ルールが既に allow / deny しているドメインと**組み込みの事前承認ドキュメントドメイン集合**はプロンプトなしで処理される。明示的な `WebFetch(domain:...)` ルール（deny / ask / allow）は事前承認集合より優先する。ルールが何を許可していても、取得は先に **WebFetch ドメイン安全性チェック**を通る
- プロンプトの選択肢は 3 つ: **Yes**（今回限り。同一ドメインでも次回また聞かれる）/ **Yes, and don't ask again for `<domain>`**（そのリポジトリの `.claude/settings.local.json` に `WebFetch(domain:...)` allow ルールを保存。`allowManagedPermissionRulesOnly` が有効な組織ではこの選択肢は非表示）/ **No, and tell Claude what to do differently**
- 事前に許可するには allow ルール `WebFetch(domain:example.com)`。`WebFetch(domain:*)` で全ドメイン許可。`auto` / `bypassPermissions` モードはプロンプトをスキップするが、明示的な `ask` ルールに合致するドメインは例外
- **サンドボックスは組み込みの事前承認ドキュメントドメイン集合を継承しない**。サンドボックスコマンドからプロンプトなしで到達させたいドメインは `sandbox.network.allowedDomains` に追加するか `WebFetch(domain:...)` ルールで許可する（サンドボックスは後者も尊重する）。逆方向は成立せず、WebFetch はサンドボックスの allowlist を読まない
- サンドボックスが解釈する `WebFetch(domain:...)` のワイルドカードは先頭 `*.`（`*.example.com`）と裸の `*`（v2.1.186 以降）の 2 形式のみ。`example.*` のような他の位置のワイルドカードは WebFetch 自体にはマッチするがサンドボックスには効かない
- WebSearch は結果のタイトルと URL のみ返し**ページ本文は取得しない**。1 回の呼び出しで最大 8 回のバックエンド検索を行いうる。`allowed_domains` / `blocked_domains` は併用不可
- **セッションあたり最大 200 回**（メイン会話と全サブエージェント合算）。`CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` で引き上げ可能だが**無効化はできない**。`/clear` でリセット
- WebSearch の権限ルールは specifier を取らない。`allow` / `deny` に裸の `WebSearch` を書く形のみ

### 3.6 Agent（サブエージェント）

- `tools` / `disallowedTools` の解決順: 両方未設定 → サブエージェントに提供されうる全ツールを継承 / `tools` のみ → 列挙されたものだけ / `disallowedTools` のみ → 親のツールから除外 / **両方設定 → `disallowedTools` が優先**
- いずれの場合も「サブエージェントに提供されうるツール」の範囲に制限される。`tools` に書いても提供対象外のツールは付与されない
- **Agent 呼び出し自体は権限プロンプトを出さない**。サブエージェント側の個々のツール呼び出しが権限ルールで判定される
- v2.1.186 以降、バックグラウンドサブエージェントの権限プロンプトもメインセッションに出る。どのサブエージェントの要求かが表示され、Esc はそのツール呼び出しだけを拒否する（サブエージェントは止まらない）
- ターン数上限はサブエージェント定義の `maxTurns`

### 3.7 LSP

- **言語用の code intelligence プラグインをインストールするまで非アクティブ**。設定は Claude Code ではなくプラグインから取得する
- ファイル編集ごとに型エラー・警告を自動報告するため、別途ビルドを走らせずに Claude が修正できる
- セッション中に一度でも言語サーバーが使えた場合、そのセッションの残りではツールはアクティブなまま。起動できないファイルへの LSP 呼び出しは個別にエラー結果を返す

---

## 4. Task 系ツールの提供条件

v2.1.233 以降、**Opus 4.8 / Sonnet 5 / Fable 5 / Mythos 5 およびそれ以降の同系統モデル**では
`TodoWrite`, `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList` が**既定で提供されない**。

オプトイン方法:

- 起動前に `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` を export（全モデル・全プロバイダで提供される）
- `--allowedTools` にツール名を挙げる（例 `claude --allowedTools TaskCreate`）
- `--tools` に列挙する（セッションの組み込みツールを列挙したものだけに制限する点に注意）
- Agent SDK では `allowedTools` / `tools` オプションが同様に働く

例外: **バックグラウンドセッションと Claude Code on the web では、モデルに関わらず常に提供される**。

サブエージェントには**親セッションが持っている場合にのみ**提供される（サブエージェントが別モデルでも同じ）。
インプロセスのチームメイトも親セッションに追随するが、独立ペインで動くチームメイトは自身のモデルの規則に従う。

---

## 4.4 動的ワークフロー（`Workflow`）の権限と再開（ドキュメント改訂）

**起動時の権限**:

| 権限モード | プロンプトのタイミング |
|:--|:--|
| Auto | 初回起動のみ。**Yes** はユーザー設定に同意を記録し以降はプロンプトなし。ultracode 有効時は完全にスキップ |
| Manual / accept edits | 毎回（そのプロジェクトのそのワークフローに **Yes, and don't ask again** を選んだ場合を除く） |
| Bypass permissions | プロンプトなしで即実行 |
| `claude -p` / Agent SDK | プロンプトを出さない |

- **Yes, and don't ask again for `<name>` in `<path>`** は、同梱 / 保存済み / プラグインのワークフローを名前で実行したときだけ提示される（Claude がその場で書いたスクリプトには出ない）
- `claude -p` / Agent SDK では Workflow ツール呼び出しがセッションの通常の権限評価を通るため、deny ルール・ask ルール・`dontAsk` モードがそのまま適用される。起動を許可する手段は次のいずれか: allow ルールの `Workflow`（全ワークフロー）または `Workflow(<name>)`（保存済みワークフローを名前指定） / auto モードの分類器 / bypass permissions モード / `allow` を返す `PreToolUse` フック / `--permission-prompt-tool`（SDK では `canUseTool` コールバックか `PermissionRequest` フック）
- **ワークフローが起動するサブエージェントは、ユーザーの権限ルールを使い、権限モードは[サブエージェントの権限モード決定規則](https://code.claude.com/docs/en/sub-agents#permission-modes)に従う**（従来記載の「常に `acceptEdits` で動く」は現行仕様ではない）。長時間実行でプロンプトを避けたい場合は、エージェントが必要とするツールを事前に allow ルールへ入れる

**停止したランの再開**:

- 一時停止したランは `/workflows` で選択して `p`。停止したランは同じスクリプトでの再起動を Claude に依頼する。エージェント開始順にリプレイされ、各エージェントは以下のいずれかになる:
  - **完了済み**: 保存結果を返す。ただし前回とプロンプトが異なる最初のエージェント（スクリプト編集や先行エージェントの結果変化による）は再実行され、**それ以降のエージェントは完了済みでも全て再実行**される
  - **停止時に実行中だったもの**: 最初からやり直し（ラン全体の停止は「失敗」扱いにならない）
  - **失敗したもの**: 再実行され、それ以降に開始したエージェントも完了済みを含め再実行。`/workflows` で個別エージェントを選んで `x` で止めた場合は「失敗」扱い
- fan-out の途中で失敗すると、完了済みの作業まで再実行される（A・B・C・D の順で開始し B が失敗すると、A はキャッシュ、B・C・D は再実行）
- 再開できるのは**同一 Claude Code セッション内**。セッションをバックグラウンド化した場合は背景セッションで同様にリプレイして継続する。ワークフロー実行中に Claude Code を終了する場合、agent view が有効なら終了ダイアログに `Move to background and exit` が出て同様に引き継がれる。`Exit and stop tasks` を選ぶ／選択肢が出ない場合はセッションと共に停止するが、保存結果は `~/.claude/projects/` 配下のセッションディレクトリに残るため、`claude --resume` で再開したセッションからワークフロー再起動を依頼すればリプレイできる（新規セッションはリプレイ対象がなく最初から実行）

---

## 4.5 制限モード（`--restricted` / `CLAUDE_CODE_RESTRICTED=1`）

v2.1.248 以降。共有マシン上で評価ハーネスが `claude` を駆動し、そのマシンのコマンド実行やユーザー／プロジェクト設定の読み取りをさせたくない用途向け。

制限内容:

- **コマンド・コードを実行する組み込みツールと `WebFetch` を削除**。`--tools` で**個別に名指し**した場合のみ復活し、`default` プリセット経由では復活しない
- 組み込みファイルツールを[ワーキングディレクトリ](https://code.claude.com/docs/en/permissions#working-directories)内に閉じ込める
- **managed 設定と `--settings` のみ**を読み、user / project / local の設定ファイルは読まない
- `bypassPermissions` を拒否する（`setMode` による `bypassPermissions` への切り替えも no-op になる）

`CLAUDE_CODE_RESTRICTED` は**起動環境からのみ**読まれ、設定ファイルの `env` ブロックでは無視される。

---

## 5. `EndConversation`

Claude がこのツールを使うのは 2 つの場合のみ:

- 継続的な暴言的入力に対する最終手段（会話の方向転換を試み、事前のメッセージで明確に警告した後）
- ユーザーが明示的にデモを求め、セッション終了に同意した場合

一般的な苛立ち・罵倒・タスクの難航は該当しない。有害コンテンツの要求も該当せず、その場合は Claude は**拒否する**のであって会話を終了しない。

終了後、対話セッションはロックされ、`/clear` / `/resume` / `/help` / `/exit` 以外は
`Claude ended this conversation. Start a new session (or /clear) to continue.` を返す。
`-p` の非対話モードで終了済みセッションを resume するとエラーで exit code 1 になる（スクリプトが成功と誤読しないため）。

**ハーネス設計上の注意**: このツールは権限プロンプトを出さず、`PreToolUse` フックも走らない。
他にツールが 1 つでも残っている限り **deny / ask ルールでブロックすることもできない**。
サブエージェントには決して提供されない。

提供条件（すべて満たす場合のみ出現）: v2.1.213 以降 / モデルが Opus 4.8・Sonnet 5・Fable 5 かそれ以降の同系統 /
対話ターミナルセッション（IDE 統合ターミナル内の `claude` を含む） / `--bare` でない /
Bedrock・Claude Platform on AWS・Vertex・Foundry ではない。
非対話 `-p`、Agent SDK、VS Code 拡張パネル、GitHub Actions、Claude Code on the web では提供されない。

---

## 6. `SendFeedback`（Claude 起草フィードバック）

v2.1.238 以降。Claude が Claude Code へのフィードバックレポートを起草し、
**ローカルの `~/.claude/feedback/drafts/`** に保存する。ユーザーが送信を選ぶまで Anthropic へは何も送られない。

Claude が起草する契機:

- ツールやコマンドが繰り返し失敗する
- 依頼に対応できなかった
- ユーザーがミスを指摘した、または Claude 自身がミスに気づいた
- ユーザーが明示的にフィードバック提出を求めた

### 6.1 ユーザー側の体験

- 起草されるとプロンプト上部にタイトル入りのカードが出る（`1` = レビュー、`2` を 2 回 = そのまま送信、`0` = 却下）。却下してもドラフトはキューに残る
- カードは既定で **1 セッションあたり最大 3 枚**（サーバー側で調整されうる）。上限後や `feedbackDrafts: "quiet"` ではプロンプトフッターに件数だけ出る
- `/feedback`（引数なし）で全セッション分のドラフトキューを開き、タイトル / area / details の編集、**Send transcript** の yes / no 切替、送信 / 破棄ができる。`w` で通常のフィードバックダイアログ。引数付き `/feedback` と `/bug` はダイアログを直接開く
- 送信内容: タイトル / area / details、環境情報（バージョン・OS・モデル）、直近の API リクエスト ID、**Send transcript = yes のときのみ**会話トランスクリプト。カードからの送信ではトランスクリプトは含まれない
- ドラフトは 30 日（`cleanupPeriodDays` がそれより短ければそちら）で失効。キューは全セッション合計 10 件で、11 件目で最古が削除される。キューにドラフトを残したまま `/exit` するとレビューか破棄かを尋ねられる

### 6.2 無効化

| 手段 | 効果 |
|:--|:--|
| `/config` の **Claude-drafted feedback** → `off`（= `feedbackDrafts: "off"`） | ツール自体を提供しない |
| `feedbackDrafts: "quiet"` | 起草は続けるがカードを出さない |
| `CLAUDE_CODE_SEND_FEEDBACK=0` | そのセッションのみ無効 |
| `DISABLE_FEEDBACK_COMMAND=1` / `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `/feedback` ごと無効 |

`feedbackDrafts` は **user または managed 設定のみ**（project / local は無視）。managed が優先する。

### 6.3 提供されない条件

- 非対話 `-p` 実行・Agent SDK（キューをレビューする画面がない）
- クラウドセッション（Claude Code on the web 等。ローカルのキューに書けない）
- Bedrock / Claude Platform on AWS / Google Cloud Agent Platform / Microsoft Foundry
- feature-flag fetching を切っているセッション
- 製品フィードバックを無効化した組織、ゼロデータ保持（ZDR）組織

---

## 参考リンク

- Tools reference: https://code.claude.com/docs/en/tools-reference
- 権限ルール: https://code.claude.com/docs/en/permissions
- サブエージェント: https://code.claude.com/docs/en/sub-agents
- Hooks: https://code.claude.com/docs/en/hooks
