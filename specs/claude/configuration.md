# Claude Code 設定仕様書

最終更新: 2026-08-30（巡回更新）

公式ドキュメント: https://code.claude.com/docs/en/settings / https://code.claude.com/docs/en/memory

> **2026-08-22 の公式ドキュメント再編**: `settings.md` が「設定の変え方・スコープの選び方・値の解決順」を扱うガイドに絞られ、全キーの網羅リファレンスが [`settings-reference.md`](https://code.claude.com/docs/en/settings-reference)、貼り付け用サンプルが [`settings-example.md`](https://code.claude.com/docs/en/settings-example)、組織配布手順が [`managed-settings.md`](https://code.claude.com/docs/en/managed-settings) に分離された。以後キーの網羅性は `settings-reference.md` を正とする（本仕様書はその 210 キーを突合済み）。

---

## 1. CLAUDE.md

CLAUDE.md はセッション開始時にコンテキストウィンドウへ読み込まれるMarkdownファイル。Claude の振る舞いに対する持続的な指示を提供する。

### 1.1 配置場所とスコープ

| スコープ | パス | 用途 | 共有範囲 |
|:--|:--|:--|:--|
| **Managed Policy** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux/WSL: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 組織全体の指示（IT/DevOps管理） | 組織内全ユーザー |
| **Project** | `./CLAUDE.md` または `./.claude/CLAUDE.md` | チーム共有のプロジェクト指示 | バージョン管理経由でチーム |
| **User** | `~/.claude/CLAUDE.md` | 全プロジェクト共通の個人設定 | 自分のみ |

### 1.2 階層構造と読み込み順

- カレントディレクトリからルートへ向かってディレクトリツリーを走査し、各階層の CLAUDE.md を読み込む
- サブディレクトリの CLAUDE.md は起動時には読み込まれず、Claude がそのディレクトリのファイルを読むときにオンデマンドで読み込まれる
- より具体的なスコープが優先される

### 1.3 インポート構文

`@path/to/import` 構文で外部ファイルをインポートできる。

```markdown
See @README.md for project overview and @package.json for available npm commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
- Personal overrides: @~/.claude/my-project-instructions.md
```

- 相対パスはインポート元ファイルからの相対
- 絶対パスも使用可能
- 再帰的インポートは最大5階層まで
- 初回のプロジェクト外インポート時に承認ダイアログが表示される

### 1.4 CLAUDE.md の書き方ガイドライン

- **サイズ**: 1ファイルあたり200行以下を目標
- **構造**: Markdownの見出しと箇条書きでグループ化
- **具体性**: 検証可能な具体的指示を書く（例: 「2スペースインデント」）
- **一貫性**: 矛盾する指示がないか定期的に見直す

### 1.5 CLAUDE.md の除外

大規模モノレポで不要な CLAUDE.md を除外するには `claudeMdExcludes` 設定を使用する。

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

Managed Policy の CLAUDE.md は除外不可。

### 1.6 `--add-dir` からの読み込み

デフォルトでは `--add-dir` で追加したディレクトリの CLAUDE.md は読み込まれない。環境変数で有効化する:

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

---

## 2. settings.json

### 2.1 スコープ

| スコープ | ファイルパス | 対象 | チーム共有 |
|:--|:--|:--|:--|
| **Managed** | サーバー管理 / plist / レジストリ / `managed-settings.json` / `managed-settings.d/*.json` | マシン上の全ユーザー | Yes（IT配布） |
| **User** | `~/.claude/settings.json` | 全プロジェクトの自分 | No |
| **Project** | `.claude/settings.json` | リポジトリの全コラボレーター | Yes（gitコミット） |
| **Local** | `.claude/settings.local.json` | このリポジトリの自分のみ | No（gitignored） |

### 2.2 優先順位（高い順）

1. **Managed** -- 他の設定で上書き不可
2. **コマンドライン引数** -- セッション単位の一時的上書き
3. **Local** -- Project/User を上書き
4. **Project** -- User を上書き
5. **User** -- 最低優先

配列設定（`permissions.allow` 等）は各スコープから**マージ**（結合・重複排除）される。
ただしモデル一覧を持つ 3 キーは独自ルールに従う:
- `fallbackModel`: 順序に意味がある連鎖のため、定義している最上位ファイルの値を丸ごと採用
- `modelPicker`: 行のマージを一切行わず、managed / `--settings` / user のうち最上位のものを丸ごと採用。project / local では無視（v2.1.242 以降）
- `availableModels`: managed 階層の最上位が定義していればそれをそのまま適用し user / project / local の追加を無視（Claude Code を組み込むアプリが独自リストを供給する場合を除く）。managed 以外のスコープ同士では配列をマージ

また `autoContinueAtUsageLimit` は `User or managed` スコープでありながら、user / `--settings` / managed のいずれも指定していない場合に限り、project / local 設定が指定するとオフとして解釈される。

#### Managed ソース同士の合成（v2.1.242 で `managedSourcesBehavior` 追加）

managed 階層は単一ではなく、上から **server-managed settings → MDM / OS ポリシー → managed settings ファイル → Windows HKCU レジストリ** の順にランクされる。既定（`managedSourcesBehavior: "first-wins"`）では、**ポリシーキーを 1 つでも持つ最上位のソースだけ**が採用され、残りは「全 admin ソースから読むキー」（サンドボックスのロックと、そのロック対象 allowlist の union、`forceRemoteSettingsRefresh`、`env` の変数単位マージ、`enableArtifact` の `false` 等）を除いて無視される。スキップされたソースは `/status` の `Skipped sources` 行に出る（v2.1.242 以降）。

`managedSourcesBehavior: "merge"` を最上位ソースに置くと、配信された全 admin ソースがキーの種類別に合成される（詳細は §2.3 の当該キーを参照）。HKCU と埋め込みホストの親設定は合成に参加しない。合成が起きた場合 `/status` の `Setting sources` 行は `(remote + file, merged)` のように表示される。

- 「ポリシーキー」とは `wslInheritsWindowsSettings` と `managedSourcesBehavior` 以外の全設定キー。この 2 つだけを含む managed ファイル / MDM ポリシーは「ポリシーを配信していない」と見なされ、次のソースへ進む
- self-hosted 環境のランナーイメージ内 managed settings ファイルも、この規則に従って適用可否が決まる（従来記載の「server-managed settings が何も配信しない場合のみ読む」は `"first-wins"` 時の帰結）

#### Managed ドロップインディレクトリ

`managed-settings.d/` ディレクトリで複数チームが独立したポリシーフラグメントをデプロイ可能（v2.1.83）:

```
/Library/Application Support/ClaudeCode/     # macOS
├── managed-settings.json                     # メインポリシー
└── managed-settings.d/
    ├── security-team.json                    # セキュリティチームのポリシー
    └── platform-team.json                    # プラットフォームチームのポリシー
```

### 2.3 主要設定キー一覧

| キー | 説明 |
|:--|:--|
| `permissions.allow` | 許可するツール使用ルール配列。**v2.1.166** で MCP 以外の glob 使用は拒否されるようになった。**v2.1.178** で `Tool(param:value)` 構文（`*` ワイルドカード可）によるツール入力パラメータマッチをサポート（例: `Agent(model:opus)`）。**v2.1.214** で単一セグメント `dir/**`（`Edit(src/**)` 等）の allow ルールは `<cwd>/dir` のみにマッチするよう修正（従来はツリー内任意の同名ディレクトリに誤マッチ）。`deny`/`ask` は従来通り任意深度マッチ |
| `permissions.deny` | 拒否するツール使用ルール配列。**v2.1.166** でツール名位置の glob パターン（`"*"` で全ツール deny 等）をサポート。未知ツール名は起動時に警告 |
| `permissions.ask` | 確認を求めるツール使用ルール配列 |
| `permissions.defaultMode` | デフォルト権限モード。v2.1.200 で `default` モードの表示名が「Manual」に変更（`"manual"` も `default` と同義で受理） |
| `permissions.additionalDirectories` | 追加ワーキングディレクトリ。**サンドボックス既定の書き込み可能パスにも含まれる**（2026-08-31 時点の公式ドキュメント改訂で明文化）: 既定では作業ディレクトリ・セッション一時ディレクトリ（`$TMPDIR`）・`--add-dir` / `/add-dir` で追加したディレクトリに加え、本キーのディレクトリにもサンドボックスコマンドが書き込める。ファイルアクセス権のみを与え skills / commands / subagents は読み込まない点は従来通り（`skills-and-commands.md` 参照） |
| `permissions.disableBypassPermissionsMode` | `bypassPermissions` モード無効化 |
| `hooks` | ライフサイクルフック設定 |
| `disableAllHooks` | 全フック無効化 |
| `allowManagedHooksOnly` | Managed フックのみ許可（Managed設定のみ） |
| `allowedHttpHookUrls` | HTTP フック許可URL |
| `httpHookAllowedEnvVars` | HTTP フック許可環境変数 |
| `env` | 環境変数設定。**v2.1.251 以降、project / local 設定の `env` からは「チェックアウトしたリポジトリに委ねるべきでない変数」を設定できない**（該当変数は破棄され `claude --debug` で確認できる警告が出る。シェル・user 設定・managed 設定で設定する）。対象は ①Claude Code が自分のファイルを置く場所を決める変数（`CLAUDE_CONFIG_DIR` / `CLAUDE_CODE_TMPDIR` および `HOME` / `TMPDIR` / `TMP` / `TEMP` / `XDG_*` 系）、②セッション内容を外部へ書き出す変数（`OTEL_LOG_RAW_API_BODIES`、詳細ベータトレーシングの `ENABLE_BETA_TRACING_DETAILED` / `BETA_TRACING_ENDPOINT`）、③起動・同期の挙動を変える変数（`CLAUDE_CODE_PROCESS_WRAPPER` / `CLAUDE_CODE_SYNC_SKILLS` / `CLAUDE_CODE_SYNC_PLUGINS` / `CLAUDE_CODE_PLUGIN_CACHE_DIR` / `CLAUDE_CODE_PLUGIN_SEED_DIR`）。v2.1.251 より前は、`HOME` / `XDG_CONFIG_HOME` と ③ を除きすべて project / local 設定から設定できた。`CLAUDE_CODE_RESTRICTED` は起動環境からのみ読まれ、どの設定ファイルの `env` でも無視される。managed / project 設定由来の `ANTHROPIC_CUSTOM_HEADERS` は、認証・組織/テナント・ルーティング・API 挙動系ヘッダ（`Authorization` / `Host` 等）を設定する場合、v2.1.251 以降は適用前にユーザー承認を要求する |
| `model` | デフォルトモデル上書き |
| `availableModels` | 選択可能モデル制限。エントリはモデルファミリー（`sonnet`）・バージョン接頭辞（`claude-sonnet-4-5`）・完全なモデルID のいずれでも一致する。**バージョン接頭辞は「もう1セグメント伸ばした後続モデルID」にも一致する**ため、`claude-fable-5` は Fable 5 と Fable 5.1 の両方を許可し、Fable 5.1 だけに絞るには `claude-fable-5-1` と書く（2026-09-02 の公式ドキュメント改訂で明文化）。ファミリーエイリアス（`opus` / `sonnet` / `haiku` / `fable`）は**許可リストが通常の解決先を許すならそのモデルに解決**し、ブロックされている場合のみ許可リスト内の最新版に置換されて要求モデルと置換モデルを示す通知が出る |
| （組織 effort 上限） | 設定キーではなく Enterprise の admin コンソール側の機能（v2.1.195 以降）。カスタムロール単位・モデル単位に最大 effort レベルを設定でき、上限超のレベルは `/effort` ピッカーに出ず、`--effort` / `/effort` で指定しても上限に丸められる。対話セッションとプレーンテキスト `--print` では警告が出るが、`json` / `stream-json` 出力やバックグラウンドエージェントでは無警告で丸められる。複数ロールが同一モデルを許可する場合は最も緩い上限が適用。組織のモデル制限と同じ経路で配信される |
| `modelOverrides` | モデルIDマッピング |
| `modelPicker` | `/model` ピッカーに並べるモデルを自前の順序・ラベルで指定（v2.1.242 以降）。`options` 配列（各行に必須 `model` と任意 `label` / `description`）と任意の `replaceBuiltInOptions`（既定 `false`）を持つ。`model` は `--model` と同じ表記を受理（エイリアス・Anthropic モデルID・Bedrock / Vertex / Foundry 形式ID）。`replaceBuiltInOptions: true` で組み込みラインナップ・`availableModels` 由来行・ゲートウェイ探索結果・`ANTHROPIC_CUSTOM_MODEL_OPTION` を全て隠し、**Default** と現在使用中モデルの行だけを残す。`false` なら組み込みの後ろに追記。提供不可の行は落とされ、選択不可の行は理由付きでグレーアウト、全行が残らなければ組み込みラインナップに戻る。`availableModels` の許可リストは引き続き適用される。**スコープは User or managed**（managed / `--settings` / user のうち最上位のものが丸ごと採用され、project / local では無視。ラインナップのマージは行われない） |
| `promptCacheTtl` | メイン会話（対話 / `-p` / Agent SDK ターンと、それらにインラインで走るヘルパー）のプロンプトキャッシュ TTL。`"5m"` / `"1h"`。未設定時は各リクエストの既定 TTL。1h はキャッシュ書き込み単価が高くなる。優先順位は `FORCE_PROMPT_CACHING_5M` > `CLAUDE_CODE_PROMPT_CACHE_TTL` > 本キー > `ENABLE_PROMPT_CACHING_1H`。v2.1.242 以降 |
| `subagentPromptCacheTtl` | メイン会話**以外**（サブエージェント、ワークフロー、コンパクションやセッションタイトル等のバックグラウンド / ヘルパーリクエスト）のプロンプトキャッシュ TTL。`"5m"` / `"1h"`。サブエージェント・ワークフローエージェント・エージェントチームのイン・プロセスメイトはメイン会話と別の TTL バケットに入るため、Claude サブスクリプションでも既定は 5 分。`"1h"` で 1 時間保持。優先順位は `FORCE_PROMPT_CACHING_5M` > `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` > 本キー > `ENABLE_PROMPT_CACHING_1H`。v2.1.242 以降 |
| `autoContinueAtUsageLimit` | claude.ai の利用上限で停止した後、セッションを開いたまま待機しリセット後に自動で作業を継続する（既定 `true`、v2.1.234 以降）。**スコープは User or managed**（user / `--settings` / managed のみ読む）だが、それらのいずれも設定していない場合に限り project / local 設定が指定するとオフ扱いになる（無視されない唯一の例外）。`/config` の **Continue automatically at usage limit** はユーザー設定に書き込み、managed / `--settings` が指定している間は行自体が隠れる |
| `fallbackModel` | プライマリモデルが overloaded / unavailable 時に順番に試行する fallback モデルを最大 3 つまで設定。`--fallback-model` フラグは v2.1.166 からインタラクティブセッションにも適用（v2.1.166） |
| `advisorModel` | Advisor ツール（実験的）のモデル設定。メインモデルより強力なモデルをアドバイザーとして併用し、方針決定・エラー行き詰まり・完了宣言前などの要所で Claude が相談する。`/advisor` コマンド / `--advisor` フラグでも設定可。Anthropic API 専用（Bedrock / Vertex / Foundry 不可）。アドバイザーはメインモデル以上の能力が必要（詳細: https://code.claude.com/docs/en/advisor ） |
| `effortLevel` | エフォートレベル。設定ファイルで指定できるのは `low` / `medium` / `high` / `xhigh` の4段階（`max` は設定ファイル不可でセッション限定、`ultracode` は別キー `ultracode` を使う）。**v2.1.251 以降は「保存済みレベルを持たないモデルに対する既定値」という位置づけ**に変わり、`/effort` は本キーではなく `modelSettings` に書き込む。同一設定ファイル内ではモデル別 `modelSettings` エントリが本キーより優先される。スキル / サブエージェントの frontmatter `effort` は当該実行中のみセッションレベルを上書きする（環境変数は上書きしない）。Managed 設定に置いても**強制ではなく既定値**で、`/effort` や `--effort` でセッション単位に変更可能。モデル別対応レベル: Fable 5 / Opus 5 / Sonnet 5 / Opus 4.8 / Opus 4.7 は `low`〜`max`、Opus 4.6 / Sonnet 4.6 は `xhigh` を除く4段階（未対応レベル指定時は直下の対応レベルにフォールバック）。既定は `high`（Opus 4.7 のみ `xhigh`。組織が[組織既定モデル](https://code.claude.com/docs/en/model-config#organization-default-model)に既定 effort を設定している場合はそのモデルでその値が既定になる） |
| `modelSettings` | **v2.1.251 で追加**。モデルごとに effort レベルを保存するオブジェクト。型は「モデル名 → `{ "effortLevel": "low" \| "medium" \| "high" \| "xhigh" }`」。既定は未設定。インタラクティブセッションで `/effort low|medium|high|xhigh` を実行するか `/model` ピッカーの effort スライダを動かすと、Claude Code が**使用中モデルのエントリとして user 設定に自動で書き込む**ため手で編集する機会は少ない。キーは `claude-opus-5` のような正規名で書かれ、そのモデルのエイリアス・日付サフィックス付き ID・`[1m]` 付き ID・認識済みプロバイダ固有 ID が同じエントリにマッチする。`/effort auto` で使用中モデルのエントリのみクリアされる。**優先順位**: 同一設定ファイル内ではモデル別エントリ > `effortLevel`。ファイル間ではモデルごとに独立して解決され、「そのモデルのエントリまたは `effortLevel` を設定している最上位の設定ファイル」が決める（したがって managed の `effortLevel` は user 設定の保存済みレベルに勝つ）。`ultracode` はこれらすべてに優先する。スコープ: Any file |
| `autoMode` | Auto Modeの分類器設定。`environment`, `allow`, `soft_deny`, `hard_deny` 配列で構成。共有プロジェクト設定からは読み込まれない。v2.1.118 で `"$defaults"` を配列に含めることで組み込みルールを置換せず追加可能。v2.1.136 で `hard_deny` 追加: ユーザー意図や allow 例外に関わらず無条件にマッチアクションをブロック |
| `disableAutoMode` | `"disable"` で Auto Mode の有効化を阻止。`Shift+Tab` サイクルから除外し `--permission-mode auto` を拒否。v2.1.207 で Bedrock / Vertex / Foundry の Auto mode がオプトイン不要になったため、無効化はこの設定で行う。同版から `autoMode` はリポジトリ内 `.claude/settings.local.json` から読み込まれなくなった（`~/.claude/settings.json` を使用） |
| `useAutoModeDuringPlan` | プランモードで Auto Mode セマンティクスを使用（デフォルト: `true`）。共有プロジェクト設定からは読み込まれない |
| `defaultShell` | `!` コマンドのデフォルトシェル。`"bash"`（デフォルト）または `"powershell"`（Windows、`CLAUDE_CODE_USE_POWERSHELL_TOOL=1` 必要） |
| `otelHeadersHelper` | 動的OpenTelemetryヘッダー生成スクリプト。起動時と定期的に実行 |
| `apiKeyHelper` | カスタムAPIキー生成スクリプト。値はキャッシュされ、**①キャッシュ寿命（既定 5 分。`CLAUDE_CODE_API_KEY_HELPER_TTL_MS` で変更）経過後、②Anthropic API（LLM ゲートウェイ経由を含む）へのリクエストが `401` / `403` で失敗したとき**に再実行される。インタラクティブセッションで project / local 設定由来のコマンドは、ワークスペース信頼プロンプトを受け入れるまで実行されない |
| `autoMemoryEnabled` | オートメモリ有効/無効（デフォルト: true） |
| `autoMemoryDirectory` | オートメモリ保存先ディレクトリ |
| `cleanupPeriodDays` | セッション保持日数（デフォルト: 30）。`0` は全トランスクリプト削除+永続化無効。検証エラーで拒否されなくなった（v2.1.89で `0` の動作変更）。v2.1.117 で `~/.claude/tasks/`、`~/.claude/shell-snapshots/`、`~/.claude/backups/` もスイープ対象に拡張 |
| `showThinkingSummaries` | thinking summariesの表示（v2.1.89でデフォルト `false` に変更。`true` で復元） |
| `companyAnnouncements` | 起動時通知メッセージ |
| `forceLoginMethod` | ログイン方式強制（`claudeai` / `console`）。v2.1.146〜v2.1.160 で Bedrock / Vertex / Foundry / Mantle のサードパーティプロバイダセッションが組織 pin と並んでブロックされるリグレッションあり。**v2.1.161 で修正済** |
| `forceLoginOrgUUID` | 組織UUID強制選択。同上のリグレッションが v2.1.146〜v2.1.160 にあり、**v2.1.161 で修正済** |
| `enableAllProjectMcpServers` | プロジェクトMCPサーバー全自動承認 |
| `enabledMcpjsonServers` | 承認するMCPサーバーリスト |
| `disabledMcpjsonServers` | 拒否するMCPサーバーリスト |
| `allowManagedMcpServersOnly` | Managed MCPサーバーのみ許可 |
| `allowedMcpServers` | MCPサーバー許可リスト |
| `deniedMcpServers` | MCPサーバー拒否リスト |
| `statusLine` | カスタムステータスライン設定。v2.1.153 でステータスラインコマンドに `COLUMNS` / `LINES` 環境変数（端末サイズ）が渡されるようになった |
| `fileSuggestion` | `@` ファイル補完カスタムコマンド |
| `outputStyle` | 出力スタイル設定。組み込みは Default / Proactive / Explanatory / Learning に加え **Concise**（前置き・ナレーションを省き結果から書き出す。作業の徹底度は変えない。v2.1.237 追加）。`/config` の Output style 行から選択すると `.claude/settings.local.json` に保存される |
| `agent` | メインスレッドをサブエージェントとして実行 |
| `language` | 応答言語設定 |
| `sandbox.*` | サンドボックス設定。**v2.1.246 以降、`--setting-sources` / SDK `settingSources` で除外したソースの `sandbox.filesystem` エントリ・`Edit` 権限ルール・`Read` deny ルールはサンドボックス構成の組み立て時に無視される**。認証情報系は除外ソースの種類で扱いが分かれる: project / local を除外するとその `sandbox.credentials` エントリは一切適用されない。user 設定を除外した場合は `~/.claude/settings.json` の `deny` エントリとファイル `mask` エントリは「制限」として残る（ただし `mask` が実値への置換を許可する効果は失われる）が、**環境変数の `mask` エントリは落とされる** |
| `attribution` | git commit/PR 帰属表記設定（`commit`, `pr` キー） |
| `alwaysThinkingEnabled` | 拡張思考のデフォルト有効化 |
| `plansDirectory` | プランファイル保存先 |
| `spinnerVerbs` | スピナー動詞カスタマイズ |
| `spellcheck` | プロンプト入力欄のスペルチェック（v2.1.235）。既定は無効。**user 設定 / `--settings` / Managed 設定のみ**から読み込まれ、プロジェクトの `.claude/settings.json` / `settings.local.json` は無視される。複数箇所にある場合は Managed → `--settings` → user の順で**1つだけ**採用しフィールドのマージはしない。フィールド: `enabled` / `checker`（`aspell` / `hunspell` / `ispell`、指定時はフォールバックしない）/ `language`（辞書名。パスや空白入りは無視）/ `color`（色名・`#rrggbb`・`rgb()`・`ansi256(n)`・`ansi:<name>`。既定はテーマのエラー色） |
| `emojiCompletionEnabled` | プロンプト入力の絵文字ショートコード補完（`:heart:` → ❤️）の有効/無効（v2.1.217） |
| `workflowSizeGuideline` | Dynamic workflow のサイズガイドライン（advisory）。設定時は `/config` の該当行が非表示（v2.1.219。デフォルトは medium = 15エージェント未満目安） |
| `disableDesktopLocalSessions` | （Managed のみ、Desktop アプリのみ、v2.1.246）デバイス上で動く Code セッションを無効化し、SSH 越しのリモートホスト / クラウド環境のみに限定する。`true`（JSON Boolean のみ有効）で Code タブの **Local** 環境がグレーアウトし選択不可（Windows の WSL エントリも同様。WSL セッション自体の可否は別途 admin-setup で制御）。既存のローカルセッションは一覧に残るが継続できない。文字列 `"true"` や `1` など Boolean 以外は無視され警告ログを出す。`sshConfigs` / `sshHostAllowlist` と併用するのが前提。**注意**: Claude Desktop 由来のポリシー（egress allowlist / ファイルシステムサンドボックス / MCP 制限）は、admin ソース（server-managed 設定・MDM/OS ポリシー・managed 設定ファイル）が存在すると無視されるため、本キーを admin ソース経由で新規配布すると Desktop 由来ポリシーが効かなくなる |
| `disableMobileSimulatorTools` | Claude の iOS Simulator ツールをブロック（Managed 設定、Claude Code Desktop / macOS）。Simulator ペイン自体は手動操作用に残る（詳細: https://code.claude.com/docs/en/desktop-ios-simulator ） |
| `remote.defaultEnvironmentId` | CLI から作成するクラウドセッション（`claude --cloud` 等）のデフォルトクラウド環境。`/remote-env` コマンドのピッカーでユーザー設定に保存される（詳細: https://code.claude.com/docs/en/cloud-environments ） |
| `autoUpdatesChannel` | 更新チャンネル (`stable` / `latest`) |
| `respectGitignore` | `@` ファイルピッカーで `.gitignore` を尊重（デフォルト: `true`） |
| `includeGitInstructions` | 組み込みcommit/PRワークフロー指示の有効化（デフォルト: `true`） |
| `includeCoAuthoredBy` | **非推奨**: `attribution` を使用 |
| `channelsEnabled` | （Managed のみ）Team/Enterprise ユーザーのチャンネル機能 |
| `allowManagedPermissionRulesOnly` | （Managed のみ）ユーザー/プロジェクトの権限ルール定義を禁止 |
| `strictKnownMarketplaces` | プラグインマーケットプレース許可リスト。v2.1.232 で `allowedMarketplaces` が別名として受理される |
| `wslInheritsWindowsSettings` | （Managed のみ）WSL on Windows が Windows 側の managed settings を継承（v2.1.118） |
| `blockedMarketplaces` | （Managed のみ）マーケットプレースブロックリスト。v2.1.232 で素のリポジトリ URL に対する url 型エントリが、CLI が git clone と分類した場合もブロックを継続 |
| `pluginTrustMessage` | （Managed のみ）プラグイン信頼警告のカスタムメッセージ |
| `awsAuthRefresh` | AWS認証リフレッシュカスタムスクリプト |
| `awsCredentialExport` | AWS認証情報JSON出力カスタムスクリプト |
| `voiceEnabled` | プッシュトゥトーク音声入力の有効化 |
| `spinnerTipsEnabled` | スピナーヒント表示（デフォルト: `true`） |
| `spinnerTipsOverride` | カスタムスピナーヒント。`tips`（文字列または `{id, text, cooldownSessions, priority}` オブジェクトの配列）／`tipsFile`（絶対 or `~/` パスの JSON、最大 256KB。server-managed settings では不可）／`label`（接頭辞、既定 `Tip`、40 文字まで）／`excludeDefault`（組み込みTipsを隠す）。オブジェクト形式・`tipsFile`・`label` は **v2.1.247+**。project / local 設定からは**プレーン文字列の tips のみ**読まれる（それ以外のフィールドは user / `--settings` / managed のみ）。合計 200 件まで。組み込みTipsと同じローテーション（最も長く未表示のもの優先、cooldown 中はスキップ、同点は priority 順） |
| `prefersReducedMotion` | UIアニメーション削減 |
| `fastModePerSessionOptIn` | セッションごとのFastモードオプトイン要求 |
| `teammateMode` | Agent Teams表示モード（`auto` / `in-process` / `tmux` / `iterm2`〈v2.1.186〉） |
| `feedbackSurveyRate` | セッション品質アンケート確率（0-1） |
| `feedbackDrafts` | Claude 起草フィードバック（`SendFeedback` ツール）の制御。`"notify"`（既定・カード表示）/ `"quiet"`（カードなし、フッターに件数）/ `"off"`（ツール自体を提供しない）。**User または Managed のみ**（project / local は無視）。managed が優先。`/config` の **Claude-drafted feedback**。セッション単位の無効化は `CLAUDE_CODE_SEND_FEEDBACK=0`（v2.1.247） |
| `showClearContextOnPlanAccept` | プラン承認画面で「コンテキストクリア」オプション表示 |
| `teammateDefaultModel` | **v2.1.234 で削除**。残存値は無視される。チームメイトのモデルはプロンプトで明示するか `CLAUDE_CODE_SUBAGENT_MODEL` で指定する |
| `autoCompactEnabled` | 自動コンパクション（デフォルト: `true`）。`/config` の **Auto-compact**。環境変数で無効化する場合は `env` に `DISABLE_AUTO_COMPACT` |
| `autoCompactWindow` | 自動コンパクション発動時のコンテキスト充填量（トークン、`100000`〜`1000000`）。未設定時はモデルごとの調整値。`/autocompact` コマンドがユーザー設定に書き込み、`--autocompact` フラグと `CLAUDE_CODE_AUTO_COMPACT_WINDOW` が上書き可。**実効値はモデルのコンテキストウィンドウでも頭打ちになる**（2026-09-02 の改訂で明文化） |
| `skillListingBudgetFraction` | 毎ターンClaudeに提示するスキル一覧に割り当てるコンテキスト窓の割合（デフォルト: `0.01` = 1%）。超過時は利用頻度の低いスキルの説明が落とされ名前のみになる（呼び出しは可能だが内容が見えない）。`/doctor` が一覧コストを推定表示 |
| `skillListingMaxDescChars` | スキル一覧における1スキルあたりの `description` + `when_to_use` の文字数上限（デフォルト: `1536`）。超過分は切り詰め |
| `workflowKeywordTriggerEnabled` | プロンプト中の `ultracode` キーワードで動的ワークフローを発動するか（デフォルト: `true`）。`/config` の **Ultracode keyword trigger**。v2.1.157 追加、v2.1.160 でキーワードが `workflow` → `ultracode` にリネーム |
| `disableWorkflows` | 動的ワークフローと同梱ワークフローコマンドを無効化（デフォルト: `false`）。`CLAUDE_CODE_DISABLE_WORKFLOWS=1` と同等 |
| `askUserQuestionTimeout` | 未回答の `AskUserQuestion` ダイアログが選択済みの内容で自動続行するまでのアイドル時間（デフォルト: `"never"`。`"60s"` / `"5m"` / `"10m"` / `"never"`）。`/config` の **Question auto-continue timeout** がユーザー設定に書き込む。project / local 設定からは読まれない（v2.1.200 以降） |
| `promptSuggestionEnabled` | プロンプト入力欄のグレー表示予測（プロンプトサジェスト）の表示（デフォルト: `true`）。`CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` が優先 |
| `enableArtifact` | Artifact ツール（セッション成果を claude.ai の非公開Webページとして公開）の**オフ専用スイッチ**。**v2.1.242 でスコープが `Any file` に変更**され、どのファイルの `false` でも当該セッションのツールをオフにでき、`true` はどこからもオンに戻せない（未設定と同義）。未設定時はアカウントの提供状況に従う。`/config` の **Artifacts** 行をオフにすると user 設定に本キーを書き込み `disableArtifact` を消す。自分の user 設定以外がオフにしている間は `/config` の行自体が隠れる。v2.1.242 より前は project / local では無視され、上位ファイルが下位の off を覆せた |
| `disableArtifact` | **非推奨**（`enableArtifact` に置き換え）。`disableArtifact: true` は `enableArtifact: false` と等価として引き続き尊重されるが、`disableArtifact: false` は無視される。`CLAUDE_CODE_DISABLE_ARTIFACT=1` は一度設定するとどの設定ファイルからもオンに戻せない |
| `subagentStatusLine` | サブエージェントタスク表示の行を書き換えるカスタムコマンド。`statusLine` / `fileSuggestion` と同様に `disableAllHooks` / `allowManagedHooksOnly` / ワークスペース信頼のゲートが適用される |
| `worktree.symlinkDirectories` | ワークツリーシンボリックリンク対象 |
| `worktree.sparsePaths` | ワークツリースパースチェックアウト対象 |
| `sandbox.failIfUnavailable` | サンドボックス起動不可時にエラー終了（v2.1.83） |
| `sandbox.network.deniedDomains` | サンドボックスでブロックするドメイン一覧。`allowedDomains` と併用可（v2.1.113） |
| `sandbox.network.strictAllowlist` | サンドボックスコマンドで許可リスト外ホストをプロンプトなしで拒否（v2.1.219） |
| `disableDeepLinkRegistration` | `claude-cli://` プロトコルハンドラ登録の無効化（v2.1.83） |
| `allowedChannelPlugins` | （Managed のみ）チャンネルプラグイン許可リスト（v2.1.84） |
| `disableSkillShellExecution` | スキル・カスタムコマンド・プラグインコマンド内のインラインシェル実行（`` !`cmd` ``）を無効化（v2.1.91） |
| `forceRemoteSettingsRefresh` | （Managed のみ）リモート設定の取得をfail-closed化。取得失敗時にセッション起動をブロック（v2.1.92） |
| `prUrlTemplate` | フッターの PR バッジを github.com 以外のカスタムコードレビュー URL に向けるテンプレート（v2.1.119）。`{host}` / `{owner}` / `{repo}` / `{number}` / `{url}` を置換。Claude の文中の `#123` 自動リンクと GitLab MR バッジには影響しない |
| （フッター GitLab MR バッジ） | v2.1.234 以降、GitLab リモートかつ `glab` CLI 認証済みのリポジトリでオープンな MR があると、GitHub PR リンクと同じフッター枠に `MR !N` バッジを表示。下線色は緑（マージ可能）/ 黄（その他のオープン状態）/ グレー（draft）。MR のマージ・クローズで消える。`glab mr create` や `git push` 成功時に即時更新 |
| `mcpServers.<name>.alwaysLoad` | MCP サーバーのツールを tool-search のディファード化対象から外し常時ロード（v2.1.121） |
| `skillOverrides` | スキル単位の表示制御。`off`（モデルと `/` から非表示）/ `user-invocable-only`（モデルから非表示）/ `name-only`（説明を圧縮）。v2.1.129 で実装が修正され機能するように |
| `worktree.baseRef` | `--worktree` / `EnterWorktree` / エージェント隔離ワークツリーのベース選択（`fresh` = `origin/<default>`、`head` = ローカル `HEAD`）。v2.1.133 でデフォルト `fresh` に戻った |
| `worktree.bgIsolation` | バックグラウンドセッションの worktree 隔離挙動。`"none"` を指定すると `EnterWorktree` なしで作業コピーを直接編集可能。worktree が現実的でないリポジトリ向け（v2.1.143） |
| `sandbox.bwrapPath` | （Managed のみ、Linux/WSL）bubblewrap バイナリのカスタムパス指定（v2.1.133） |
| `sandbox.socatPath` | （Managed のみ、Linux/WSL）socat バイナリのカスタムパス指定（v2.1.133） |
| `parentSettingsBehavior` | （Managed のみ、admin tier）SDK `managedSettings`（parent tier）をポリシーマージに含めるか（`first-wins` / `merge`）。v2.1.133 |
| `allowAllClaudeAiMcps` | （Managed のみ、Enterprise）`managed-mcp.json` と並んで claude.ai クラウド MCP コネクタを一括ロード（v2.1.149） |
| `pluginSuggestionMarketplaces` | （Managed のみ）コンテキストアウェア tips 経由で suggest 対象とする組織マーケットプレースを allowlist 化（v2.1.152） |
| `keybindingFlavor` | プロンプト入力のキーバインド流儀。`"readline"` で Ctrl+W が Bash 同様に「直前の空白まで削除」になる。既定は `"classic"`（従来動作）。v2.1.238 |
| `<marketplace>.headersHelper` | url マーケットプレース定義またはカタログエントリに指定すると、カタログ取得・同一オリジンのアーカイブ取得用 HTTP ヘッダ（短命トークン等）をコマンドで動的生成する。カタログエントリ側の `headersHelper` は当該プラグインの install / update 時のみ実行され、実行前にコマンド内容が表示され `[y/N]` 確認が入る（`-y` で省略）。v2.1.238 |
| `<marketplace>.skipLfs` | プラグインマーケットプレース定義（`github` / `git` ソース）に `skipLfs: true` を指定すると Git LFS ダウンロードをスキップ（v2.1.153） |
| `archive` プラグインソース | HTTPS 経由の zip からプラグインをインストールするソースタイプ（v2.1.224）。任意で SHA-256 ハッシュピン止めによる完全性検証に対応。`owner/*` 形式のオーナーワイルドカードをマーケットプレース managed settings に指定可（v2.1.223） |
| `<plugin>.defaultEnabled` | プラグインの `plugin.json` またはマーケットプレースエントリで `defaultEnabled: false` を指定するとインストール後デフォルト無効。`/plugin` または `claude plugin enable` で有効化。依存関係は引き続き自動有効化（v2.1.154） |
| `requiredMinimumVersion` | （Managed のみ）Claude Code の最小許容バージョン。範囲外なら起動を拒否し承認済みバージョンへ案内（v2.1.163）。`requiredMaximumVersion` ともども **fail open** 設計で、不正な値は強制されず破棄される |
| `requiredMaximumVersion` | （Managed のみ）Claude Code の最大許容バージョン。範囲外なら起動を拒否し承認済みバージョンへ案内（v2.1.163） |
| `disableBundledSkills` | バンドルスキル・ワークフロー・組み込みスラッシュコマンドをモデルから隠す（v2.1.169）。環境変数 `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` でも可 |
| `wheelScrollAccelerationEnabled` | フルスクリーンモードのマウスホイールスクロール加速を無効化（v2.1.174） |
| `enforceAvailableModels` | （Managed のみ）`availableModels` 許可リストを Default モデルにも適用。user/project 設定による managed リストの拡張も禁止（v2.1.175） |
| `footerLinksRegexes` | フッター行に正規表現マッチのリンクバッジを表示（user / managed 設定）（v2.1.176） |
| `sandbox.allowAppleEvents` | サンドボックスコマンドに macOS Apple Events 送信を許可（オプトイン）（v2.1.181） |
| `attribution.sessionUrl` | Web / Remote Control セッションで commit・PR への claude.ai セッションリンク付与を制御（v2.1.183） |
| `respondToBashCommands` | `false` で `!` bashコマンド出力への Claude 自動応答を無効化（v2.1.186 から自動応答がデフォルト） |
| `sandbox.credentials` | サンドボックスコマンドによる認証情報ファイル・シークレット環境変数の読み取りをブロック（v2.1.187）。v2.1.221 で `mode: "mask"` 追加（Linux/WSL）: センチネル値を読ませ egress 時にプロキシが実値へ置換。`extract` 正規表現で範囲指定可。macOS は `deny` にフォールバック。v2.1.224 でマスキングオプション拡充: `extract`（抽出範囲指定）・`decode: "jwt"`（JWT デコード）・`awsPairs` / `sigv4`（AWS 認証情報ペア・SigV4 署名対応） |
| `autoMode.classifyAllShell` | 全 Bash/PowerShell コマンドを auto-mode 分類器に通す（デフォルトは任意コード実行パターンのみ）（v2.1.193） |
| `axScreenReader` | スクリーンリーダー向けプレーンテキスト描画にオプトイン。`claude --ax-screen-reader` / `CLAUDE_AX_SCREEN_READER=1` でも可（v2.1.208） |
| `vimInsertModeRemaps` | vim モードのインサートモードで `jj` → Escape のような 2 キーシーケンスをマップ（v2.1.208） |
| `sandbox.filesystem.disabled` | ファイルシステム分離のみスキップし、ネットワーク egress 制御は維持（v2.1.216） |
| `sandbox.filesystem.denyRead` | 読み取り拒否パス。ワイルドカードは全プラットフォームで有効（Linux/WSL2 では実パスに展開）。**v2.1.236（macOS）**: `**/.env` のようなワイルドカード read-deny が許可済み read 領域の内部でも優先され、マッチしたディレクトリ配下も対象に含まれ、拒否対象ファイルのリネームによる回避もできなくなった |
| `crossSessionInbound` | セッション間メッセージ（`SendMessage`）の受信を制御（v2.1.224）。`accept`（配送） / `hold`（通知のみ・未配送、後で `accept` が適用されれば解放） / `refuse`（黙って破棄）。`/config` の「Messages from your other sessions」行からも設定可（v2.1.232、書き込み先は user 設定。`/config crossSessionInbound=value` ショートハンドは拒否される）。**未設定時は送受信両セッションの権限モードクラスから自動判定**（bypassPermissions 系 vs プロンプト系。詳細は agent-teams.md 参照）。**v2.1.248**: 認識できない値を設定すると警告が出る。user / project / local / `--settings` にある間は上位ソースが `accept` でも受信を `hold` し（他ソースの `refuse` は引き続き有効）、managed 設定にある場合は最も厳しい `refuse` として扱われる（v2.1.248 より前は黙って無視） |
| `dialogExpiry` | セッション間メッセージのダイアログ有効期限を設定（v2.1.224）。v2.1.232 で `/config` に「Dialog expiry」「Messages from your other sessions」の行が追加され GUI から設定可能に |
| `extraKnownMarketplaces` | 既知プラグインマーケットプレースの追加登録。v2.1.232 で `additionalMarketplaces` が別名として受理される |
| `sandbox.ripgrep` | サンドボックスが使う ripgrep バイナリの指定。v2.1.232 で user / managed / `--settings` 由来のみ有効化（プロジェクト設定からの上書き不可）。サーバー管理設定からの上書きは承認必須 |
| `isolatePeerMachines` | `true` でマシン外のセッションへの `SendMessage` 送信前にユーザー承認を必須化。`bypassPermissions` モードでも承認を求める。**いずれかのスコープの `true` が優先**されるため、コミット済みプロジェクト設定で有効化はできても無効化はできない。同一マシン内のメッセージには適用されない |
| `claudeMd` | （Managed のみ）別ファイルを配布せずに CLAUDE.md 相当の指示を組織管理メモリとして注入する。文字列（Markdown、改行は `\n`）で書き、user / project の CLAUDE.md より先に読み込まれる |
| `claudeMdExcludes` | メモリ読み込み時にスキップする CLAUDE.md のパターン配列（glob または絶対パス。**絶対ファイルパスに対してマッチ**）。大規模モノレポで他チームの CLAUDE.md を除外する用途 |
| `verbose` | トランスクリプトでツール呼び出しの入出力を常に全文表示する（既定 `false` = 要約表示 + `Ctrl+O` で展開）。フック・MCP サーバー・長いシェルコマンドのデバッグ向け。`/config` の **Verbose output**。`--verbose` がセッション単位で優先。`viewMode` を設定するとこのキーは上書きされる |
| `voice` | 音声ディクテーションの有効化と入力キーの挙動。`enabled`（Boolean）/ `autoSubmit`（Boolean、hold モードのみ）/ `mode`（`"hold"` = 押している間だけ録音、`"tap"` = 1 回目で録音開始・2 回目で送信）。`/voice` が自動で書き込む。`enabled: true` で `mode` 未指定なら `"hold"` |
| `fastMode` | 利用可能なセッションで Fast モードを有効化（トークン単価は上がるが応答が速い。反復開発・ライブデバッグ向け）。`/fast` が `~/.claude/settings.json` に `fastMode: true` を書き込み、再度実行すると削除する。**Opus 5 / Opus 4.8 でのみ動作**し、他モデルから ON にすると Opus に切り替わり、非対応モデルに切り替えると OFF になる。`CLAUDE_CODE_DISABLE_FAST_MODE` が優先 |
| `theme` | インターフェースの配色テーマ。`/config` の **Theme**。既定 `"dark"`。旧版が `~/.claude.json` に残した値は、どの settings ファイルも指定していない場合に適用される |
| `tui` | 端末 UI レンダラの選択。`"fullscreen"` = 代替画面バッファのちらつきのないレンダラ（仮想スクロールバック付き）、`"default"` = 従来のメイン画面レンダラ。`/tui fullscreen` / `/tui default` が書き込む。未設定時は Claude Code が自動選択。`CLAUDE_CODE_NO_FLICKER` / `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` がセッション単位で優先 |
| `viewMode` | 起動時のトランスクリプト表示（`"default"` / `"verbose"` / `"focus"`）。設定すると `/focus` の記憶と `verbose` 設定の両方を上書きする。`--verbose` がセッション単位で優先 |
| `autoScrollEnabled` | フルスクリーンレンダリングで新規出力を追って最下部へ自動スクロール（既定 `true`）。`false` にするとスクロール位置を保持したまま作業が進む（権限プロンプトは引き続き視野に入る） |
| `syntaxHighlightingDisabled` | 端末に表示する diff・コードブロック・ファイルプレビューの言語別シンタックスハイライトを無効化しプレーンテキスト表示にする（既定 `false`）。組み込みハイライタのみを使用し、プラグインや LSP は関与しない |
| `awaySummaryEnabled` | 数分離席して端末に戻った際の 1 行セッションリキャップ表示。`/config` の **Session recap**。未設定時は有効。`CLAUDE_CODE_ENABLE_AWAY_SUMMARY` がセッション単位で双方向に優先 |
| `terminalTitleFromRename` | 端末タブのタイトルに `/rename` / `--name` で付けたセッション名を使うか（既定 `true`）。`false` にすると命名後も自動生成タイトルを表示し続ける。名前自体は有効なままで `/resume <name>` から引ける |
| `externalEditorContext` | `Ctrl+G` で外部エディタを開く際、直前の Claude の応答を `#` コメント行としてバッファ冒頭に入れる（既定 `false`。保存時に自動除去）。`/config` の **Show last response in external editor**。スコープは `~/.claude.json`（Global config） |
| `diffTool` | VS Code / JetBrains 接続時に `Edit` / `Write` の差分をどこに出すか。`"auto"` = IDE の diff ビューア、`"terminal"` = 端末内（既定 `"auto"`）。スコープは Global config。IDE 接続中のみ `/config` の **Diff tool** に出現 |
| `permissionExplainerEnabled` | Bash / PowerShell の権限プロンプトで `Ctrl+E` を押すとモデルがコマンド解説（何をするか・なぜ実行するか・何が起きうるか、**Low/Med/High risk** ラベル付き）を生成する機能（既定 `true`）。押した時だけモデルに問い合わせ、表示してもコマンドは実行されない。スコープは Global config |
| `switchModelsOnFlag` | 安全性分類器がリクエストをフラグした際の挙動。`true`（既定）= フォールバックモデルへ自動切替して継続、`false` = 対話セッションでは一時停止して切替か編集かを選ばせる（`-p` などダイアログを出せない場合はエラー終了）。`/config` の **Switch models when a message is flagged** |
| `enableWorkflows` | 動的ワークフローをユーザー単位で ON/OFF（プランの既定と違う挙動にしたい場合）。`/config` の **Dynamic workflows** がユーザー設定に書き込み、既定へ戻すとキーを削除する。未設定時は Pro プランのみ無効・その他は有効。組織全体で止める場合は `disableWorkflows`（Managed）を使う。`CLAUDE_CODE_DISABLE_WORKFLOWS` が優先 |
| `enabledPlugins` | プラグインの個別有効/無効。キーは `plugin-name@marketplace-name`、値は Boolean。どのスコープにもエントリが無いプラグインは `defaultEnabled` に従う。`/plugin` や `claude plugin enable` が自動で書き込む |
| `pluginConfigs` | プラグインの `userConfig` ダイアログで入力した**非機微**な設定値をプラグイン ID をキーに保存。`options`（オプション名 → string / number / boolean / string 配列）と任意の `mcpServers`（サーバー別ユーザー設定）を持つ。機微なオプションは macOS Keychain（非対応環境は `~/.claude/.credentials.json`）に保存される。スコープは user / managed |
| `fileCheckpointingEnabled` | 編集前にファイルスナップショットを取り `/rewind` で復元可能にする（既定 `true`）。`/config` の **Rewind code (checkpoints)**。`CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` と「どちらかが OFF なら OFF」 |
| `minimumVersion` | バックグラウンド自動更新と `claude update` がこのバージョン未満をインストールしないようにする。`stable` チャンネルへ切り替える際に新しい `latest` ビルドからダウングレードされるのを防ぐ。`/config` でチャンネル切替時に「現在のバージョンに留まる」を選ぶと自動で書き込まれ、`latest` に戻すと削除される。Managed に置けば組織全体の下限を user/project から下げられなくできる |
| `preferredNotifChannel` | タスク完了・権限待ちのローカル通知方法。`/config` の **Local notifications**。既定 `"auto"`。旧版が `~/.claude.json` に残した値も読む |
| `inputNeededNotifEnabled` | 権限プロンプト・質問待ちの際にスマートフォンへプッシュ通知（既定 `false`）。Remote Control 接続中のみ送信。`/config` の **Push when actions required** |
| `agentPushNotifEnabled` | 長時間タスク完了時など、Claude が送る価値ありと判断した際のスマートフォンへのプッシュ通知（既定 `false`）。アカウントに同期され、Remote Control 接続中に届く。`/config` の **Push when Claude decides** |
| `remoteControlAtStartup` | 対話セッション開始時に Remote Control へ自動接続（`/remote-control` を待たない）。未設定時は組織の admin 既定 → Claude Code の既定の順。`--remote-control` はこのキーが `false` でも 1 セッションだけ有効化する |
| `disableRemoteControl` | Remote Control を無効化（既定 `false`）。`claude remote-control`・`--remote-control`・自動起動・セッション内トグルを全て拒否し、組織ポリシーによる無効化である旨を表示。Managed に置けば MDM でデバイス単位に強制できる |
| `disableAgentView` | バックグラウンドエージェントとエージェントビュー（`claude agents`, `--bg`, `/background`, オンデマンドスーパーバイザ）を無効化。`CLAUDE_CODE_DISABLE_AGENT_VIEW` と「どちらかが OFF なら OFF」 |
| `syncClaudeAiSkills` | claude.ai で有効化したスキルのダウンロードを止める。`-p`（非対話）実行かつ `CLAUDE_CODE_SYNC_SKILLS` 設定時に `~/.claude/skills/synced/` へ落ちてくるものが対象。**`false` のみ有効**で `true` は未設定と同義（同期を ON にはできない）。スコープは user / local / managed（リポジトリからは無効化できない） |
| `disableClaudeAiConnectors` | claude.ai の MCP コネクタの取得・接続を停止（既定 `false`）。**いずれかのスコープの `true` が優先**されるため、コミット済みプロジェクト設定でリポジトリ単位のオプトアウトはできても、project の `false` で user/managed の `true` は覆せない。v2.1.182 以降。`ENABLE_CLAUDEAI_MCP_SERVERS=false` と「どちらかが OFF なら OFF」 |
| `skipWebFetchPreflight` | WebFetch のドメイン安全性チェック（取得前にホスト名を `api.anthropic.com` へ送る）をスキップ。Bedrock / Vertex / Foundry など Anthropic 宛通信が塞がれた環境向け |
| `skipAutoPermissionPrompt` | 自分で Auto Mode に入った際（設定やモードセレクタ経由）に一度だけ出る Auto Mode 説明の告知をスキップ。スコープは user / managed（リポジトリからは設定できない） |
| `skipDangerousModePermissionPrompt` | `--dangerously-skip-permissions` または `defaultMode: "bypassPermissions"` で `bypassPermissions` に入る前の確認ダイアログをスキップ。一度承認するとユーザー設定に `true` が自動で書き込まれる。スコープは user / local / managed（信頼されていないリポジトリからは設定できない） |
| `gcpAuthRefresh` | Google Cloud の Application Default Credentials が期限切れ・読み込み不能になった際に実行する自前のリフレッシュコマンド（`awsAuthRefresh` の GCP 版） |
| `processWrapper` | （macOS / Linux、スコープ user / managed）Claude Code が起動するバックグラウンドプロセスの前段に企業ランチャーコマンドを挟む。ランチャーは自身のコマンドラインに Claude Code のものが追記された形で呼ばれるため、最後に exec する必要がある（[corporate-launcher](https://code.claude.com/docs/en/corporate-launcher) 参照）。v2.1.210 以降。`CLAUDE_CODE_PROCESS_WRAPPER` が優先 |
| `forceLoginGatewayUrl` | （Managed のみ）`/login` の Cloud gateway 画面が接続するゲートウェイ URL を指定。画面には URL 入力欄が無く、未設定だと「IT 管理者に問い合わせ」と表示される。`forceLoginMethod` 未設定時はこのキー単独で Cloud gateway 画面が開く。`forceLoginMethod: "gateway"` はログイン方式ピッカーも消す。`claudeai` / `console` を指定した場合はそちらが優先されるため、両方を整合させて設定する |
| `managedSourcesBehavior` | （Managed のみ、v2.1.242 以降）複数の managed ソースが同一マシンに配信されたときの合成方法。`"first-wins"`（既定）は**ポリシーキーを1つでも持つ最上位のソースだけを採用**し、残りは「全 admin ソースから読むキー」を除き無視する。`"merge"` は配信された全 admin ソースを種類別に合成する: リスト（`permissions.allow`・`hooks`・`sandbox.network.allowedDomains`・`deniedMcpServers` 等）は全ソースの要素を結合、ロック（`allowManagedHooksOnly`・`permissions.disableBypassPermissionsMode`・`crossSessionInbound` 等）は最も厳しい値、制限リスト（`availableModels`・`allowedMcpServers`・`strictKnownMarketplaces`・`allowedChannelPlugins`・`fallbackModel` チェーン）はそれを設定する最上位ソースの内容を丸ごと採用、最上位ソース限定キー（`apiKeyHelper`・`awsAuthRefresh`・`awsCredentialExport`・`gcpAuthRefresh`・`otelHeadersHelper`・`proxyAuthHelper`・`forceLoginOrgUUID`・`forceLoginMethod`・`forceLoginGatewayUrl`・`parentSettingsBehavior`・`modelPicker`・`permissions.defaultMode`）は最上位ソースのみ、`env` は変数単位でマージ（両モード共通）、その他は最上位ソースの値。**本キー自身は「本キーかポリシーキーを持つ最上位ソース」からのみ読まれる**ため、下位ソースが自分を merge 対象に引き上げることはできず、server-managed settings が届かないマシンでは MDM プロファイル側にも書く必要がある。Windows HKCU と埋め込みホストの親設定は merge に参加しない。`managed-settings.json` は最下位の admin ソースなので、そこに `"merge"` を書いても合成相手がいない。`"merge"` は最上位より下の全ソースが管理者の統制下にある場合のみ使う（下位ソースの allow ルールが加算されるため）。`/status` の `Setting sources` 行に `(remote + file, merged)` のように表示される |

> **managed 設定ファイル / drop-in ファイルが読めない・パースできない場合**: 他の admin ソースがポリシーを供給していなければ、claude.ai または Claude Console の認証情報でサインインしたセッションは起動時に「管理者に連絡してください」というメッセージを出して終了する。

> **承認が必要になった managed 設定（v2.1.251）**: サンドボックスの TLS を終端する、サンドボックストラフィックを自前のプロキシへ流す、認証情報を注入する、サンドボックス隔離を弱める、といった server-managed 設定は適用前にユーザー承認を要求するようになった。`ANTHROPIC_CUSTOM_HEADERS`（managed / project 設定由来）も同様。あわせて managed 設定の承認ダイアログは、**前回承認時からの差分だけ**を列挙するようになった。同一の Claude apps gateway へ再サインインしても設定が変わっていなければ承認プロンプトは再表示されない。

| `desktopSessionCleanupPeriodDays` | Claude Desktop / Cowork が書き込んだセッションのトランスクリプト保持除外に上限日数を設ける（v2.1.248）。従来 30 日でクリーンアップされ Desktop / Cowork のセッションが消えていた問題の修正に伴い追加。アプリ内に残っている限り除外されるが、本キーで上限を設定できる（組織ポリシーが保持期間を管理している場合を除く） |
| `policyHelper` | （Managed のみ）起動時に managed settings を計算する実行ファイルを走らせ、その出力をそのセッションの managed settings として扱う。デバイスポスチャ・ID・リモートサービスから動的にポリシーを導出する用途。**macOS plist / Windows HKLM レジストリ / managed settings ファイルのいずれか**（最優先の managed ソース）に置かれた場合のみ実行され、サーバー管理設定・HKCU・親設定由来のものは無視される。`path` / `refreshIntervalMs` / `timeoutMs` を持つ。`path` は正規化済み絶対パス（`.` / `..` セグメント不可）で、Windows ではドライブレター付きパスか UNC パスかつ `.exe` 終端であること。**ヘルパー実行が失敗する条件（2026-08-31 時点で明文化）**: `path` が規則違反 / `path` に通常ファイルが無い（存在確認も `timeoutMs` の予算内で行うため応答しないネットワークマウントでも失敗する） / 非ゼロ終了・`timeoutMs` 超過・実行権限なしで起動しない / stdout または stderr へ 1 MiB 超を出力 / stdout が単一 JSON オブジェクトでない・`managedSettings` に修復不能なスキーマ違反がある。**起動時の実行に失敗すると Claude Code は理由を表示して起動を拒否する**（対話セッション・`claude -p`・Agent SDK・バックグラウンドセッション・大半のサブコマンドが対象。非ゼロ終了／タイムアウト時は stderr も表示）。この拒否は意図的な設計なので、障害耐性が必要なヘルパーは自前キャッシュから応答して `0` で終了させる。バックグラウンド更新の失敗時は直前に成功したポリシーを維持する（`--debug` で毎回の stderr がデバッグログに残る）。`policyHelper` の値自体が不正（裸のパス文字列、`timeoutMs` が最小値未満など）な場合はドロップされたエントリとして報告し、ヘルパーを実行せず残りの managed 設定でセッションを開始する。起動時の `forceRemoteSettingsRefresh` チェックはヘルパーより先に全 admin ソースを読む。`managedSettings` キーを含まない envelope で `0` 終了した場合は managed 設定に何も寄与せず、他ソースが通常どおり適用される。無効化はキーを削除して行う（stdout 上限は 1 MB ではなく 1 MiB） |
| `strictPluginOnlyCustomization` | （Managed のみ）skills / agents / hooks / MCP サーバーを user・project ソースから読み込ませず、プラグインと managed settings 由来のみに限定する。`true` で 4 種すべて、または `["skills","agents","hooks","mcp"]` の配列で個別指定。`strictKnownMarketplaces` と組み合わせるとカスタマイズのサプライチェーン全体を統制できる |
| `disableCommandPluginSources` | （Managed のみ）マーケットプレースが宣言したコマンドをユーザーのマシンで実行してインストールする `command` プラグインソースをブロック。`true` でコマンドを実行せず、command ソースのプラグインを install/update せず、既にインストール済みのものもロードしなくなる。未設定時は `allowManagedHooksOnly` に追随する。マーケットプレースの `headersHelper` コマンド（アーカイブ取得時の認証ヘッダ生成）も、本キーが明示的に `false` でない限りブロックされる（managed 設定自身が宣言したマーケットプレースは例外）。v2.1.229 以降 |
| `modelPricing` | （Managed のみ、v2.1.242 以降）組織の契約単価・割引倍率をコスト表示に反映させ、定価の代わりに使う。適用先は `/usage`・ステータスライン・Agent SDK の `total_cost_usd`・`--max-budget-usd`・OpenTelemetry のコストメトリクス／イベント。単価は管理者が自分で書く（契約や Console からは読まない）。任意の `multiplier`（0 超 1 以下、全コストに乗算）と任意の `overrides`（モデルID → `input` / `output` / `cacheRead` / `cacheWrite` の百万トークンあたり USD、4 つとも必須、各 0〜10000）を持つ。fast モード加算や US-only 推論単価は加算されない。行のキーが組み込みモデルIDなら、そのモデルの日付付きスナップショットIDやプロバイダ固有IDすべてに適用され、それ以外のキー（ゲートウェイのエイリアス等）はその ID 限定。Bedrock のアプリケーション推論プロファイルは解決後のモデルの行が適用される。パースできない行・`multiplier` は落として残りを使う。user / project / local / `--settings` / Windows HKCU では無視。server-managed settings 経由の場合、そのセッションの設定取得が完了するまでは定価表示 |
| `disableSideloadFlags` | （Managed のみ）`--plugin-dir` / `--plugin-url` / `--agents` / `--mcp-config` を起動時に拒否する（これらは `strictKnownMarketplaces` を単発で回避できてしまうため）。拒否されたフラグ名を挙げてエラー終了し、内部的に CLI をこれらのフラグ付きで起動する経路（現状は Desktop の Cowork ローカルセッション）にも同じチェックを適用する。クラウドセッションではサーバーが `--mcp-config` で渡した MCP サーバー（`type: "sdk"` のインプロセス以外）を落としてセッションを開始する。v2.1.193 以降 |
| `sshConfigs` | （user / managed、Desktop アプリのみ）Desktop の環境ドロップダウンに SSH 接続を追加する。各要素は必須の `id` / `name` / `sshHost` と任意の `sshPort` / `sshIdentityFile`。managed で定義したものは managed 表示となり、ユーザーは選択のみ可能で編集・削除できない |
| `sshHostAllowlist` | （Managed のみ、Desktop アプリのみ）Desktop の SSH セッションが接続できるホストを制限。大文字小文字を無視し、`*` は任意ホスト、`*.example.com` は `example.com` とその全サブドメイン、それ以外は `~/.ssh/config` 解決後のホスト名との完全一致。空配列で SSH セッションを無効化 |
| `browserExternalPageTools` | （Managed のみ、Desktop アプリのみ）Desktop の Browser ペインで Claude のツールが外部ページを読んだり操作したりするのを止める（`"disabled"`、Desktop は `"disable"` も受理）。人が外部サイトを開くことは可能で、ローカル開発サーバーのプレビューに対するツール利用も継続する。CLI はこのキーを無視 |
| `disableBrowserExternalNavigation` | （Managed のみ、Desktop アプリのみ）Desktop の Browser ペインの外部ブラウジングを人・Claude 双方に対して無効化（JSON の `true` のみ有効）。localhost の開発サーバープレビューは動作し続ける。CLI はこのキーを無視 |

> 補足: `allowedMcpServers` / `deniedMcpServers` 述語の `${VAR}` 参照は v2.1.166 で正しくマッチするようになった。Managed settings は invalid なエントリが含まれても、v2.1.166 から残りの valid policy は enforce される（従来は silently 全無効化）。

#### スペルチェック（`spellcheck`）— v2.1.235

プロンプト入力欄のテキストのみを対象とし、Claude の応答やファイルは検査しない。シェルモード（`!`）・`Ctrl+R` 履歴検索・音声入力中、およびスクリーンリーダーモードでは何も検査しない。

- **前提**: `aspell` / `hunspell` / `ispell` のいずれかを PATH に用意する。Claude Code はこの順で最初に見つかった1つを実行する（Windows のパッケージマネージャが置く `.cmd` シムも対象）
- **スキップ対象**: `/help` 等のコマンド、`@` メンション、URL、ファイルパス、`--verbose` 等のフラグ、数字・アンダースコア・2文字目以降の大文字を含む語、バッククォート内テキスト、中国語・日本語・韓国語・タイ語・ラオ語・クメール語・ミャンマー語
- **単語の除外**: Claude Code 自身の単語リストは持たない。チェッカーのパーソナル辞書に追加し、Claude Code を再起動する
- **停止条件**: チェッカー未インストール / `checker` 指定のものが不在 / 連続2回の失敗 / 15秒超のタイムアウトが3回、のいずれかで当該セッションのチェックを停止。原因は `claude --debug` 実行時の `~/.claude/debug/<session-id>.txt` の `[spellcheck]` 行で確認できる

> ハーネス設計上の注意: プロジェクト設定に書いても効かないため、**リポジトリにコミットして配布することはできない**。組織で強制したい場合は Managed 設定を使う。

#### セルフホスト環境（self-hosted-runner）— v2.1.224 で公開ベータ

Team / Enterprise 向けに、クラウドセッションを自社ホスト上で実行する `claude self-hosted-runner` コマンドが追加された（Linux / macOS）。Owner/admin が管理画面で「Allow self-hosted environments」を有効化して利用する。詳細な全フラグは `claude self-hosted-runner --help` および [reference](https://code.claude.com/docs/en/self-hosted-environments-reference) 参照。デプロイインフラ寄りのためここでは**ハーネス設計に直接関わるガードのみ**を段階的開示で記載する:

- `--confine-repo-settings <warn|enforce|off>`（既定 `warn`）: リポジトリにコミットされた `settings.json` がセッション自身のワークスペース外への読み書き付与・環境変数設定・オペレータの sandbox/hooks ポスチャ上書き（例 `sandbox.enabled: false`、`disableAllHooks`）を試みた場合にフラグ立て。`enforce` で拒否
- `--trust-workspace [bool]`（既定 on）: リポジトリコミット済みの `permissions.allow` / `additionalDirectories` を信頼するか。`false` でリポジトリ由来の権限付与を破棄しホスト側 `settings.json` の allow ルールを使用（`sandbox.*` は常に適用され `--confine-repo-settings` の走査対象）
- `--defer-shutdown-max-min <minutes>`（v2.1.238）: SIGTERM 受信後もアタッチ中のセッションを指定分数まで処理し続け、残ったものを park してから終了する（ローリング更新時のセッション断を緩和）
- `--proxy-authorization-command` / `--proxy-authorization-file`（v2.1.238）: 接続ごとに新規発行の `Proxy-Authorization` ヘッダを要求する egress プロキシに対応
- 追加サブコマンド `self-hosted-runner orchestrator`（オンデマンド runner の自動起動）、環境変数は `SELF_HOSTED_RUNNER_*` / `CLAUDE_RUNNER_*` 系

#### クラウド環境の API クレデンシャル（2026-08-31 時点で公式ドキュメントに追加）

Anthropic ホスト型のクラウド環境（Claude Code on the web）に **API クレデンシャル**を保存し、Claude が鍵の値を見ないまま外部 API を呼べるようにする仕組みが追加された。Anthropic の agent proxy が、セッションの VM を出た後のリクエストに対して、環境に登録されたホスト向けだけヘッダを付与する。鍵はセッションの環境変数にもファイルにも現れない。

- **前提条件**: claude.ai 組織の admin ロール（**Team / Enterprise では Owner のみが保有し Admin は持たない**。Pro / Max は自分の組織で保有）／**既存の** Anthropic ホスト型クラウド環境（セルフホスト環境は非対応、新規作成ダイアログにも項目は出ない）／API がインターネットから到達可能／組織が顧客管理暗号鍵（CMEK）を使っていないこと
- **登録方法**: [claude.ai/code](https://claude.ai/code) の環境編集ダイアログ「Update cloud environment」の **Environment variables** 直下にある **API credentials** から1件ずつ追加する。編集は不可（変更するには削除して再追加）。既定の Credential type は `Bearer`（`Authorization: Bearer <値>` ヘッダ。`X-Api-Key` のようなヘッダ名・prefix なしにも変更可）。**Allowed websites** に対象ホストを列挙し、先頭 `*.` で全サブドメインにマッチ。保存後は値を再表示できない
- **ネットワーク許可との関係**: クレデンシャルに列挙したホストは、環境の [network access level] が本来許可しない場合でもセッションから到達できる（agent proxy がスキップするホストを除く）。GitHub（専用プロキシ）・MCP コネクタ（Anthropic サーバー経由）も従来どおりセッションの allowlist を通らない
- **クレデンシャルが付かないリクエスト**: GitHub（GitHub プロキシが認証）／`api.anthropic.com` と公開パッケージレジストリ（`registry.npmjs.org`, `jsr.io`, `npm.jsr.io`, `pypi.org`, `files.pythonhosted.org`, `index.crates.io`, `proxy.golang.org`）／**セットアップスクリプトからのリクエスト**（Claude Code は setup script の実行後に agent proxy へ接続するため）
- **共有環境**: 組織共有環境も環境セレクタから開けるようになり、**Owner** はそこで編集・API クレデンシャル追加ができる（他メンバーは読み取り専用）。環境変数とセットアップスクリプトは利用者全員が読めるため、秘密情報は環境変数ではなく API クレデンシャルに置く
- **アーカイブ時**: 環境をアーカイブしても実行中セッションにはクレデンシャルが付いたままになるため、不要なものは事前に削除する

> ハーネス設計上の注意: 「クラウド環境には専用のシークレットストアが無い」という従来の前提が変わった。ただし**鍵をコマンドラインやスクリプトで参照する用途には使えない**（Claude に値は渡らず、proxy が付与するだけ）。`curl` 等で当該ホストを叩く形のハーネスに限って有効。

### 2.4 `~/.claude.json` のグローバル設定

`settings.json` ではなく `~/.claude.json` に格納される設定:

| キー | 説明 |
|:--|:--|
| `autoConnectIde` | IDE自動接続 |
| `autoInstallIdeExtension` | IDE拡張自動インストール |
| `editorMode` | キーバインドモード (`normal` / `vim`) |
| `showTurnDuration` | ターン所要時間表示 |
| `terminalProgressBarEnabled` | ターミナルプログレスバー |

---

## 3. settings.local.json

`.claude/settings.local.json` はプロジェクト固有の個人設定ファイル。

- 作成時に Claude Code が自動で git ignore 設定を行う
- Project設定やUser設定を上書きできる
- チームに共有されない個人的なオーバーライドや実験的設定に使用

**配置場所（ドキュメント改訂）**:

- git リポジトリのサブディレクトリで起動した場合、Claude Code は**リポジトリルート**の `.claude/settings.local.json` を読み書きし、承認をリポジトリ全体に適用する
- **worktree では main チェックアウトのルートのファイル**を使う
- 次の場合はルートではなく `.claude/settings.json` と同じ場所に置かれる: git リポジトリ外、リポジトリルートがホームディレクトリ、Windows、リポジトリルート／その `.git`／`.claude` が自ユーザー所有でない
- **ファイル内のパスはリポジトリルートを基準にしない**。`/` 始まりの権限ルールや相対サンドボックスパスは[セッションのプライマリワーキングディレクトリ](https://code.claude.com/docs/en/permissions#read-and-edit)基準で解決される

**共有 `.claude/settings.json` の読み込み元**: セッションのプライマリワーキングディレクトリ。リポジトリルートにコミットしたファイルを使うにはそこで起動する。**`/cd` でセッションを移動すると（v2.1.246 以降）、移動先ディレクトリの project / local 双方を読むようになる**（local ファイルの配置は上記規則に従う）。

---

## 4. .claude/rules/

プロジェクトの指示を複数ファイルに分割して管理するディレクトリ。

### 4.1 基本構造

```
.claude/
├── CLAUDE.md
└── rules/
    ├── code-style.md
    ├── testing.md
    └── security.md
```

- `.md` ファイルは再帰的に検出される（サブディレクトリ対応）
- `paths` フロントマターなしのルールは起動時に `.claude/CLAUDE.md` と同じ優先度で読み込まれる

### 4.2 パススコープルール

YAML フロントマターの `paths` フィールドで特定ファイルにスコープを限定できる:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API開発ルール
- 全APIエンドポイントに入力バリデーション必須
```

| パターン例 | マッチ対象 |
|:--|:--|
| `**/*.ts` | 全ディレクトリのTypeScriptファイル |
| `src/**/*` | `src/` 配下の全ファイル |
| `*.md` | プロジェクトルートのMarkdownファイル |
| `src/components/*.tsx` | 特定ディレクトリのReactコンポーネント |

ブレース展開もサポート: `"src/**/*.{ts,tsx}"`

### 4.3 ユーザーレベルルール

`~/.claude/rules/` に配置した個人ルールは全プロジェクトに適用される。プロジェクトルールより低い優先度。

### 4.4 シンボリックリンク

`.claude/rules/` はシンボリックリンクをサポートする:

```bash
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

---

## 5. メモリシステム

### 5.1 CLAUDE.md（手動メモリ）

ユーザーが手動で記述・管理する指示ファイル。詳細は上記セクション1を参照。

### 5.2 オートメモリ

Claude が自動的にセッション間の学習を蓄積する仕組み。v2.1.59以降で利用可能。

#### 保存場所

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # エントリポイント（毎セッション先頭200行を読み込み）
├── debugging.md       # トピック別ノート
├── api-conventions.md
└── ...
```

`<project>` パスは git リポジトリから導出。同一リポジトリの全ワークツリー/サブディレクトリで共有。

#### 仕組み

- `MEMORY.md` の先頭200行が毎セッション開始時に読み込まれる
- 200行超のコンテンツは読み込まれない（Claude が自動的に詳細をトピックファイルへ分離）
- トピックファイル（`debugging.md` 等）は起動時に読み込まれず、必要時にオンデマンドで読む
- セッション中に Claude がメモリファイルを読み書きする

#### 有効化/無効化

- `/memory` コマンドでトグル
- 設定: `"autoMemoryEnabled": false`
- 環境変数: `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

#### カスタムディレクトリ

```json
{
  "autoMemoryDirectory": "~/my-custom-memory-dir"
}
```

`autoMemoryDirectory` は policy / local / user 設定から受け付ける。Project 設定（`.claude/settings.json`）からは受け付けない（セキュリティ上の理由）。

### 5.3 `/memory` コマンド

- 現在のセッションで読み込まれている CLAUDE.md とルールファイルを一覧表示
- オートメモリの有効/無効トグル
- オートメモリフォルダへのリンク
- ファイル選択でエディタを開く

### 5.4 サブエージェントメモリ

サブエージェントも `memory` フロントマターフィールドで独自のオートメモリを保持できる。スコープ: `user` / `project` / `local`。

---

> **プラン別の既定モデル（2026-09-01 時点）**: Max / Team Premium / Enterprise / Anthropic API は **Opus 5**、Pro / Team Standard は **Sonnet 5**。v2.1.251 でシート課金の Enterprise サブスクリプションも Opus 5 になり、従量課金（pay-as-you-go）との区別が公式ドキュメントから消えた。

## 6. 環境変数

主要な環境変数（`settings.json` の `env` キーまたはシェルで設定）:

| 変数名 | 用途 |
|:--|:--|
| `ANTHROPIC_API_KEY` | APIキー |
| `ANTHROPIC_MODEL` | モデル指定 |
| `ANTHROPIC_DEFAULT_MODEL` | 新規セッションが開始するモデルの既定値（v2.1.236）。`--model` / `ANTHROPIC_MODEL` / 設定ファイルの `model`（`/model` の保存選択を含む） / 組織既定モデルのいずれも無い場合のみ適用。`/model` の保存選択は次回以降も本変数より優先（`ANTHROPIC_MODEL` は毎回変数側に戻る点が異なる）。`default` / `inherit` / `opusplan` / `haiku` 指定時、`enforceAvailableModels` 有効時、`availableModels`・組織制限で除外される場合、アカウント未提供の場合は無視される。`/model` ピッカーの Default 行に「Set by ANTHROPIC_DEFAULT_MODEL」と表示 |
| `CLAUDE_CODE_USE_BEDROCK` | Amazon Bedrock使用 |
| `CLAUDE_CODE_USE_VERTEX` | Google Vertex AI使用 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | オートメモリ無効化 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | `--add-dir` からCLAUDE.md読み込み |
| `CLAUDE_CODE_NEW_INIT` | `/init` の新しいインタラクティブフロー有効化 |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | バックグラウンドタスク無効化 |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | git指示無効化 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 自動コンパクション閾値（%） |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEndフックタイムアウト |
| `MCP_TIMEOUT` | MCPサーバー起動タイムアウト（ms） |
| `MAX_MCP_OUTPUT_TOKENS` | MCPツール出力トークン上限 |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | スキル説明の文字数バジェット |
| `CLAUDE_CODE_REMOTE` | Webリモート環境で `"true"` |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | `1` でサブプロセス環境（Bash ツール・hooks・MCP stdio サーバー）から認証情報を除去（v2.1.83）。除去対象は Anthropic / クラウドプロバイダの認証情報に限らず、**Claude Code が認証情報と認識する任意の変数と、パッケージレジストリ URL に埋め込まれた認証情報**まで広がる（2026-09-02 の改訂で明文化）。**hooks プロセスも対象**で、常に除去される `OTEL_*` エクスポータ変数に加えて、本変数が `1` のときは除去対象の変数が hook からも見えなくなる |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 非ストリーミングフォールバック無効化（v2.1.83） |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | ストリーミングアイドルウォッチドッグ閾値（デフォルト90秒）（v2.1.84） |
| `ANTHROPIC_DEFAULT_{OPUS,SONNET,HAIKU}_MODEL_SUPPORTS` | ピンモデルのeffort/thinking検出オーバーライド（v2.1.84） |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP `headersHelper` スクリプトに渡されるサーバー名（v2.1.85） |
| `CLAUDE_CODE_MCP_SERVER_URL` | MCP `headersHelper` スクリプトに渡されるサーバーURL（v2.1.85） |
| `CLAUDE_CODE_NO_FLICKER` | `1` でフリッカーフリーのalt-screen描画有効化（リサーチプレビュー）（v2.1.89） |
| `MCP_CONNECTION_NONBLOCKING` | `true` で `-p` モードのMCP接続待機スキップ。`--mcp-config` サーバー接続は5秒上限（v2.1.89） |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | `1` で `git pull` 失敗時にマーケットプレースキャッシュを保持。オフライン/エアギャップ環境向け（v2.1.90） |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | `1` でFastモードを完全無効化（v2.1.92） |
| `CLAUDE_CODE_PERFORCE_MODE` | Perforce VCS 連携モード（v2.1.98） |
| `ENABLE_PROMPT_CACHING_1H` | `1` でプロンプトキャッシュ TTL を既定の 5 分から 1 時間に要求（v2.1.108）。API キー / Bedrock / Vertex（Google Cloud Agent Platform） / Microsoft Foundry / Claude Platform on AWS 利用者向け。優先順位は最下位で、`CLAUDE_CODE_PROMPT_CACHE_TTL` / `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` / `promptCacheTtl` / `subagentPromptCacheTtl` に負ける |
| `CLAUDE_CODE_PROMPT_CACHE_TTL` | `5m` / `1h` のみ受理。メイン会話（対話 / `-p` / SDK ターン＋インラインヘルパー）のプロンプトキャッシュ TTL を指定。`promptCacheTtl` 設定と `ENABLE_PROMPT_CACHING_1H` より優先（v2.1.242 系） |
| `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` | `5m` / `1h` のみ受理。サブエージェント・ワークフロー・バックグラウンド処理などメイン会話外のリクエストの TTL を指定。`subagentPromptCacheTtl` 設定と `ENABLE_PROMPT_CACHING_1H` より優先（v2.1.242 系） |
| `FORCE_PROMPT_CACHING_5M` | `1` で 1 時間 TTL が適用される状況でも 5 分 TTL を強制。上記 2 変数・`ENABLE_PROMPT_CACHING_1H`・`promptCacheTtl` / `subagentPromptCacheTtl` の全てに優先する最上位（v2.1.242 系） |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | Remote Controlセッション名の自動生成プレフィックス（デフォルト: ホスト名）（v2.1.92） |
| `CLAUDE_CODE_FORK_SUBAGENT` | `1` で外部ビルド（サードパーティ）でもフォークサブエージェントを有効化（v2.1.117） |
| `OTEL_LOG_TOOL_DETAILS` | `1` で OpenTelemetry のカスタム/MCP コマンド名の redact を解除（v2.1.117） |
| `DISABLE_UPDATES` | 手動 `claude update` 含む全更新パスをブロック（`DISABLE_AUTOUPDATER` より厳格）（v2.1.118） |
| `CLAUDE_CODE_HIDE_CWD` | 起動ロゴでの作業ディレクトリ表示を隠す（v2.1.119） |
| `AI_AGENT` | Claude Code がサブプロセスに自動設定。`gh` などのツールが Claude Code 由来トラフィックを識別可能に（v2.1.120） |
| `OTEL_LOG_USER_PROMPTS` | `1` で OpenTelemetry の `user_system_prompt` 属性を出力（v2.1.121） |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock サービスティア（`default` / `flex` / `priority`）を選択し `X-Amzn-Bedrock-Service-Tier` ヘッダで送信（v2.1.122） |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | `1` で同期出力を強制有効化。自動検出が外す端末（Emacs `eat` 等）向け（v2.1.129） |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | Homebrew/WinGet インストールで設定時、バックグラウンドでアップグレードを実行し再起動プロンプトを表示（v2.1.129） |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | `1` で `ANTHROPIC_BASE_URL` ゲートウェイの `/v1/models` 探索を有効化。v2.1.129 でオプトイン化（v2.1.126〜v2.1.128 は自動） |
| `CLAUDE_CODE_SESSION_ID` | Claude Code が Bash ツールサブプロセスに自動設定する現在のセッション ID。hooks に渡される `session_id` と一致（v2.1.132） |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | `1` でフルスクリーン alternate-screen レンダラーをオプトアウトし、会話を端末のネイティブスクロールバックに残す（v2.1.132） |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | OpenTelemetry 経由でセッション品質サーベイ応答を収集する企業向けに、無効化されているセッションフィードバックサーベイを再有効化（v2.1.136） |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | `1` で GitHub プラグインソースを SSH ではなく HTTPS でクローン。SSH 鍵未設定環境向け（v2.1.141） |
| `ANTHROPIC_WORKSPACE_ID` | workload identity federation で発行トークンを特定ワークスペースにスコープ。federation ルールが複数ワークスペースをカバーする場合に必須（v2.1.141） |
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **v2.1.160 で削除（no-op）**。v2.1.142 以降、Fast mode デフォルトは Opus 4.7、v2.1.154 で非推奨化。Opus 4.6 で fast mode を使うには `/model claude-opus-4-6[1m]` の後に `/fast on` |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | `1` で PowerShell ツールが `-ExecutionPolicy Bypass` を渡さなくなる（v2.1.143） |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | `0` で Windows の Bedrock/Vertex/Foundry ユーザーでも PowerShell ツールを無効化（v2.1.143 でデフォルト有効化） |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | stop hook の連続ブロック上限を変更。デフォルト 8 回（v2.1.143） |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | `true` でセッションエントリポイントを OpenTelemetry メトリクスに `app.entrypoint` 属性として追加（v2.1.152） |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | `1` で Bedrock / Vertex / Foundry の Opus 4.7 / 4.8 ユーザーが Auto mode に opt-in（v2.1.158） |
| `OTEL_RESOURCE_ATTRIBUTES` | （標準 OTEL 変数）v2.1.161 から、ここで指定した属性（`team=foo,repo=bar` 等）が OpenTelemetry メトリクスデータポイントのラベルとして添付され、Team / Repo 別カスタムディメンションでのスライス分析が可能 |
| `MAX_THINKING_TOKENS` | 拡張思考のトークン上限。**v2.1.166** で `0` を指定すると Claude API 経由の thinking 既定モデルでも thinking を無効化できるようになった（3P プロバイダは挙動変更なし） |
| `CLAUDE_CODE_SAFE_MODE` | `--safe-mode` フラグ相当。CLAUDE.md・プラグイン・スキル・hooks・MCP など全カスタマイズを無効化して起動（トラブルシュート用）（v2.1.169） |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | バンドルスキル・組み込みコマンドをモデルから隠す（`disableBundledSkills` 設定と同等）（v2.1.169） |
| `API_FORCE_IDLE_TIMEOUT` | `0` で Vertex / Foundry のストリームアイドルタイムアウト（デフォルト5分）をオプトアウト（v2.1.169） |
| `CLAUDE_CLIENT_PRESENCE_FILE` | マーカーファイルを指定し、マシン在席中のモバイルプッシュ通知を抑制（v2.1.181） |
| `CLAUDE_CODE_MAX_RETRIES` | リトライ回数上限。v2.1.186 で上限 15 にキャップ（無人セッションは `CLAUDE_CODE_RETRY_WATCHDOG` を使用）。v2.1.199 の `CLAUDE_CODE_RETRY_WATCHDOG` 有効時はキャップ解除・非キャパシティ系一時エラーのデフォルトリトライ 300 回 |
| `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` | リモート MCP ツール呼び出しの無応答タイムアウト（デフォルト5分で中断）のオーバーライド（v2.1.187） |
| `OTEL_LOG_ASSISTANT_RESPONSES` | `1` で `claude_code.assistant_response` OTELログイベント（モデル応答テキスト）を出力。未設定時は `OTEL_LOG_USER_PROMPTS` に追従するため、プロンプト収集済みデプロイは `0` 明示でプロンプトのみ維持（v2.1.193） |
| `CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP` | `1` でアイドルなバックグラウンドシェルのメモリ圧迫時自動回収を無効化（v2.1.193） |
| `CLAUDE_CODE_DISABLE_MOUSE_CLICKS` | フルスクリーンモードのクリック/ドラッグ/ホバーを無効化（ホイールスクロールは維持）（v2.1.195） |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | `0` でストリーミングアイドルウォッチドッグ（5分無イベントで中断・リトライ、v2.1.196 から全プロバイダでデフォルト有効）を無効化 |
| `CLAUDE_AX_SCREEN_READER` | `1` でスクリーンリーダーモード（プレーンテキスト描画）を有効化。settings `axScreenReader` / `--ax-screen-reader` と同等（v2.1.208） |
| `CLAUDE_CODE_PROCESS_WRAPPER` | agent view とバックグラウンドサービスの全 Claude Code 自己スポーンを、指定した企業ランチャー（ラッパー実行ファイル）経由で起動（v2.1.208） |
| `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT` | stream-json 出力にサブエージェントのテキスト・思考を含める。`--forward-subagent-text` フラグと同等（v2.1.211） |
| `CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` | WebSearch ツール呼び出しのセッション上限（デフォルト 200）。暴走検索ループ対策（v2.1.212） |
| `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION` | サブエージェントスポーンのセッション上限（デフォルト 200）。`/clear` でリセット（v2.1.212） |
| `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` | MCP ツール呼び出しの自動バックグラウンド化閾値（デフォルト 2 分）の変更・無効化（v2.1.212） |
| `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH` | OpenTelemetry コンテンツ属性の切り詰め上限（デフォルト 60KB）を設定（v2.1.214） |
| `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` | 同時実行サブエージェント数の上限（デフォルト 20）。1メッセージからの無制限 fan-out 対策（v2.1.217） |
| `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` | サブエージェントのネスト起動深度。v2.1.217 で既定無効化 → v2.1.219 で既定深度3に変更。`=1` でネスト無効化 |
| `FORCE_HYPERLINK` | `0` でフッター PR バッジ等のハイパーリンク強制をオプトアウト（ssh/tmux 等の端末サポート未検出時もリンク化される挙動、v2.1.217） |
| `CLAUDE_CODE_WORKFLOW_PREFIX_STAGGER_MS` | ワークフロー fan-out で同一 prompt prefix の兄弟エージェントを時間差起動し、後続がキャッシュ済み prefix を再利用してコストを削減。`0` で無効化（v2.1.229） |
| `CLAUDE_CODE_MESSAGING_SOCKET` | Claude Code が各セッションの受信箱 Unix ドメインソケットのパスを hooks / Bash コマンドに自動エクスポート。`SessionStart` を含む全 hook より前に設定される（機能フラグ取得前に起動したセッションでは取得完了後に設定されるため、それ以前に起動したプロセスでは未設定）。各セッションは自身のソケットのみをエクスポートし、親セッションから継承しない |
| `CLAUDE_CODE_MESSAGING_TOKEN` | 上記ソケットへ投函するスクリプト用のセッション単位トークン。接続の最初の行に `{"type":"auth","token":"<token>"}` を送ることで「自セッションの子プロセスからの投函」として検証される（macOS でプロセス終了後、および Claude Code が PID 1 のコンテナでは、プロセス証跡が取れないためこのトークンが唯一の検証手段） |
| `CLAUDE_CODE_TOOL_MEMORY_LIMIT` | （Linux / WSL、オプトイン）Bash / PowerShell、および **v2.1.246 以降は Monitor** ツールコマンドに memory cgroup を適用し、暴走ビルドによるセッション停止を防ぐ（v2.1.233）。`4G` のようにプレーンな数値＋`K`/`M`/`G`/`T` 接尾辞、`0` / `off` で無効。**Claude Code が最初に起動したプロセス**が有効／無効を確定した後は、値を変えても次回 `claude` 起動時まで反映されない |
| `CLAUDE_CODE_TOOL_MEMORY_CGROUP_EXCLUDE` | （Linux / WSL、v2.1.246 以降）上記メモリ上限の対象から外すプロセス種別をカンマ区切りで指定。指定できるのは `mcp`（ローカル MCP サーバー） / `lsp`（言語サーバー） / `hooks`（フックコマンド） / `plugin`（プラグインが実行するコマンド） / `helper`（`git` 等 Claude Code 自身のヘルパー） / `agent`（エージェントチームメイト等の子 Claude Code プロセス）。`none` で全種別を上限対象、`all-new` で Bash / PowerShell / Monitor のみを対象。Bash / PowerShell / Monitor は何を指定しても常に上限対象。未知の名前は無視。**未設定時の対象集合は Anthropic がサーバーから配信する構成に従い変動する**ため、固定したい場合は明示設定する。アクションをブロック・変更しうる権限ゲート系フックと、そのフックが呼ぶ MCP サーバーは、全種別を上限対象にしても除外される（カーネルによる kill がブロックを解除してしまうのを防ぐため） |
| `CLAUDE_CODE_RESTRICTED` | `1` で制限モード（`--restricted` と同等）で起動（v2.1.248）。**起動環境からのみ読まれ、設定ファイルの `env` ブロックでは無視される** |
| `CLAUDE_CODE_GOAL_CHECKIN_MINUTES` | `/goal` 実行中、バックグラウンド作業がゴール評価を待たせ続けたときに Claude へ状況確認を促すまでの分数（デフォルト `30`、`0` で無効、最大 `10080`＝1週間。プレーンな整数以外は未設定扱い）。Claude Code は実行中タスクを列挙し、出力の確認・進捗中なら待機継続・停滞中なら修正か停止を依頼する（v2.1.234）。v2.1.236 でアイドルセッションのゴールがバックグラウンド作業の背後で停滞している場合、ユーザーの復帰を待たず 30 分後（以降 1h・2h）に自動チェックインするよう拡張 |
| `CLAUDE_CODE_PROJECT_DIR_NAME` | セッションごとに独自の config ディレクトリを与えるホスト向けに、プロジェクト単位のトランスクリプトディレクトリ名を短い名前で指定（v2.1.234。changelog 記載、公式 env-vars リファレンス未収載） |
| `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` | WebFetch のセッション内 URL キャッシュ TTL を設定（デフォルトは従来通り 15 分）（v2.1.233） |
| `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_DEFAULT` | `1` で、`Content-Type` ヘッダが欠落 / 空の Amazon Bedrock ストリーミング応答をバイナリ `application/vnd.amazon.eventstream` 形式として扱う既定挙動を止める。Bedrock は常に当該ヘッダを送るため、欠落＝プロキシによる除去とみなして既定ではバイナリ解釈するが、プロキシがヘッダを除去した上で本文を SSE として再送出する場合のみ本変数を設定する（v2.1.246 系） |
| `CLAUDE_CODE_SEND_FEEDBACK` | `0` でそのセッションの Claude 起草フィードバック（`SendFeedback` ツール）を無効化。`1` は「アカウントが既に利用可能な場合に有効化」であって提供権限を付与するものではない。`DISABLE_FEEDBACK_COMMAND` や `feedbackDrafts: "off"` 等、他の無効化スイッチは引き続き効く（v2.1.247） |
| `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` | 非対話モード（`-p`）で最終ターン後、出力に含まれるバックグラウンドサブエージェント / ワークフローの完了を待つ際の**連続アイドル待機時間の上限**（ms、デフォルト `600000`＝10分）。合計待機時間の上限ではなく「何も進捗が無いまま待ち続けられる時間」の上限である点に注意（2026-08-31 の公式ドキュメント改訂で明確化） |
| `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_GUARD` | `1` で、Amazon Bedrock ストリーミング応答が `application/vnd.amazon.eventstream` content-type を持つかの検査をスキップする。未設定時、異なる content-type の応答はエラーになる |
| `CLAUDE_CODE_ENABLE_TODO_TOOLS` | `1` で Todo / タスク追跡ツール（`TaskCreate` / `TaskGet` / `TaskUpdate` / `TaskList` / `TodoWrite`）を復活させる。**v2.1.233 で Opus 4.8 / Sonnet 5 / Fable 5 / Mythos 5 以降のモデルでは既定で提供されなくなった**ため、これらに依存するハーネスは明示設定が必要 |
| `CLAUDE_CODE_AUTO_BACKGROUND_WORKER_CHECKIN_SECONDS` | `CLAUDE_AUTO_BACKGROUND_TASKS` 有効時に、実行中のバックグラウンドサブエージェントを確認するよう Claude へ促すリマインダーの間隔（秒）。**`1`〜`86400` の素の整数のみ受理**し、それ以外の値・表記は未設定として読まれる。未設定時はリマインダーなし（v2.1.248 以降） |
| `BETA_TRACING_ENDPOINT` | [詳細ベータトレーシング](https://code.claude.com/docs/en/monitoring-usage#traces-beta)の OTLP エンドポイント。`ENABLE_BETA_TRACING_DETAILED=1` と併用するとログ・トレースが構成済みエクスポータではなくこちらへ送られる。シェル・user 設定・managed 設定でのみ設定可（project / local 設定では無視） |
| `ENABLE_BETA_TRACING_DETAILED` | `1` かつ `BETA_TRACING_ENDPOINT` 設定時に詳細ベータトレーシングを有効化。内容を含むスパン属性と `claude_code.hook` スパンが追加される。インタラクティブ CLI セッションでは組織がベータの許可リストに入っている必要がある。project / local 設定では無視 |

> 補足: `OTEL_LOG_TOOL_DETAILS=1` は v2.1.157 で `tool_decision` イベントに `tool_parameters`（bash コマンド、MCP/skill 名等）を追加する効果も併せ持つようになった。

> Workflow キーワードトリガー: v2.1.157 から `/config` の「Workflow keyword trigger」設定で、プロンプト中のトリガーキーワードがワークフロー要求を発動するかをユーザー設定できる。**v2.1.160 でトリガーキーワードが `workflow` → `ultracode` にリネーム**。

> 機能フラグ取得（feature flag fetching）: `DISABLE_GROWTHBOOK` / `DISABLE_TELEMETRY` / `DO_NOT_TRACK` / `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` のいずれかを設定するとオフになるほか、**Claude apps gateway 経由の接続では常にオフ**、**Bedrock / Claude Platform on AWS / Google Cloud Agent Platform / Microsoft Foundry でも既定でオフ**（Claude Code を埋め込むホストプラットフォームが `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` を設定した場合のみオン）。取得オフ時はフラグ制御下の機能が使えない。なお v2.1.246 で `/code-review` は Claude 自身の自律起動を含めこの制約から外れた。

> Claude 起草フィードバック（`SendFeedback`）も機能フラグ取得下の機能。`DISABLE_FEEDBACK_COMMAND=1` / `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`（非空値）でも一括で無効化される。

> インストール / アップグレード直後の初回セッション: フラグ取得が有効でも、フラグ制御下の機能が初回セッションでは欠けることがあり、通常は auto mode で始まるプランでも Manual モードで開始することがある（次回セッションからは正常）。新規インストール直後の非対話セッション（`claude -p` / Agent SDK / VS Code 拡張）は、開始権限モードの決定前にフラグを取得できる場合がある。

完全な環境変数リファレンス: https://code.claude.com/docs/en/env-vars

---

## 参考リンク

- 設定: https://code.claude.com/docs/en/settings
- メモリ: https://code.claude.com/docs/en/memory
- 環境変数: https://code.claude.com/docs/en/env-vars
- 権限: https://code.claude.com/docs/en/permissions
- サンドボックス: https://code.claude.com/docs/en/sandboxing
