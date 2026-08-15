# Claude Code 設定仕様書

最終更新: 2026-07-13（巡回更新）

公式ドキュメント: https://code.claude.com/docs/en/settings / https://code.claude.com/docs/en/memory

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
| `permissions.additionalDirectories` | 追加ワーキングディレクトリ |
| `permissions.disableBypassPermissionsMode` | `bypassPermissions` モード無効化 |
| `hooks` | ライフサイクルフック設定 |
| `disableAllHooks` | 全フック無効化 |
| `allowManagedHooksOnly` | Managed フックのみ許可（Managed設定のみ） |
| `allowedHttpHookUrls` | HTTP フック許可URL |
| `httpHookAllowedEnvVars` | HTTP フック許可環境変数 |
| `env` | 環境変数設定 |
| `model` | デフォルトモデル上書き |
| `availableModels` | 選択可能モデル制限 |
| `modelOverrides` | モデルIDマッピング |
| `fallbackModel` | プライマリモデルが overloaded / unavailable 時に順番に試行する fallback モデルを最大 3 つまで設定。`--fallback-model` フラグは v2.1.166 からインタラクティブセッションにも適用（v2.1.166） |
| `advisorModel` | Advisor ツール（実験的）のモデル設定。メインモデルより強力なモデルをアドバイザーとして併用し、方針決定・エラー行き詰まり・完了宣言前などの要所で Claude が相談する。`/advisor` コマンド / `--advisor` フラグでも設定可。Anthropic API 専用（Bedrock / Vertex / Foundry 不可）。アドバイザーはメインモデル以上の能力が必要（詳細: https://code.claude.com/docs/en/advisor ） |
| `effortLevel` | エフォートレベル (`low` / `medium` / `high`) |
| `autoMode` | Auto Modeの分類器設定。`environment`, `allow`, `soft_deny`, `hard_deny` 配列で構成。共有プロジェクト設定からは読み込まれない。v2.1.118 で `"$defaults"` を配列に含めることで組み込みルールを置換せず追加可能。v2.1.136 で `hard_deny` 追加: ユーザー意図や allow 例外に関わらず無条件にマッチアクションをブロック |
| `disableAutoMode` | `"disable"` で Auto Mode の有効化を阻止。`Shift+Tab` サイクルから除外し `--permission-mode auto` を拒否。v2.1.207 で Bedrock / Vertex / Foundry の Auto mode がオプトイン不要になったため、無効化はこの設定で行う。同版から `autoMode` はリポジトリ内 `.claude/settings.local.json` から読み込まれなくなった（`~/.claude/settings.json` を使用） |
| `useAutoModeDuringPlan` | プランモードで Auto Mode セマンティクスを使用（デフォルト: `true`）。共有プロジェクト設定からは読み込まれない |
| `defaultShell` | `!` コマンドのデフォルトシェル。`"bash"`（デフォルト）または `"powershell"`（Windows、`CLAUDE_CODE_USE_POWERSHELL_TOOL=1` 必要） |
| `otelHeadersHelper` | 動的OpenTelemetryヘッダー生成スクリプト。起動時と定期的に実行 |
| `apiKeyHelper` | カスタムAPIキー生成スクリプト |
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
| `outputStyle` | 出力スタイル設定 |
| `agent` | メインスレッドをサブエージェントとして実行 |
| `language` | 応答言語設定 |
| `sandbox.*` | サンドボックス設定 |
| `attribution` | git commit/PR 帰属表記設定（`commit`, `pr` キー） |
| `alwaysThinkingEnabled` | 拡張思考のデフォルト有効化 |
| `plansDirectory` | プランファイル保存先 |
| `spinnerVerbs` | スピナー動詞カスタマイズ |
| `emojiCompletionEnabled` | プロンプト入力の絵文字ショートコード補完（`:heart:` → ❤️）の有効/無効（v2.1.217） |
| `workflowSizeGuideline` | Dynamic workflow のサイズガイドライン（advisory）。設定時は `/config` の該当行が非表示（v2.1.219。デフォルトは medium = 15エージェント未満目安） |
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
| `spinnerTipsOverride` | カスタムスピナーヒント（`excludeDefault`, `tips` キー） |
| `prefersReducedMotion` | UIアニメーション削減 |
| `fastModePerSessionOptIn` | セッションごとのFastモードオプトイン要求 |
| `teammateMode` | Agent Teams表示モード（`auto` / `in-process` / `tmux` / `iterm2`〈v2.1.186〉） |
| `feedbackSurveyRate` | セッション品質アンケート確率（0-1） |
| `showClearContextOnPlanAccept` | プラン承認画面で「コンテキストクリア」オプション表示 |
| `worktree.symlinkDirectories` | ワークツリーシンボリックリンク対象 |
| `worktree.sparsePaths` | ワークツリースパースチェックアウト対象 |
| `sandbox.failIfUnavailable` | サンドボックス起動不可時にエラー終了（v2.1.83） |
| `sandbox.network.deniedDomains` | サンドボックスでブロックするドメイン一覧。`allowedDomains` と併用可（v2.1.113） |
| `sandbox.network.strictAllowlist` | サンドボックスコマンドで許可リスト外ホストをプロンプトなしで拒否（v2.1.219） |
| `disableDeepLinkRegistration` | `claude-cli://` プロトコルハンドラ登録の無効化（v2.1.83） |
| `allowedChannelPlugins` | （Managed のみ）チャンネルプラグイン許可リスト（v2.1.84） |
| `disableSkillShellExecution` | スキル・カスタムコマンド・プラグインコマンド内のインラインシェル実行（`` !`cmd` ``）を無効化（v2.1.91） |
| `forceRemoteSettingsRefresh` | （Managed のみ）リモート設定の取得をfail-closed化。取得失敗時にセッション起動をブロック（v2.1.92） |
| `prUrlTemplate` | フッターの PR バッジを github.com 以外のカスタムコードレビュー URL に向けるテンプレート（v2.1.119） |
| `mcpServers.<name>.alwaysLoad` | MCP サーバーのツールを tool-search のディファード化対象から外し常時ロード（v2.1.121） |
| `skillOverrides` | スキル単位の表示制御。`off`（モデルと `/` から非表示）/ `user-invocable-only`（モデルから非表示）/ `name-only`（説明を圧縮）。v2.1.129 で実装が修正され機能するように |
| `worktree.baseRef` | `--worktree` / `EnterWorktree` / エージェント隔離ワークツリーのベース選択（`fresh` = `origin/<default>`、`head` = ローカル `HEAD`）。v2.1.133 でデフォルト `fresh` に戻った |
| `worktree.bgIsolation` | バックグラウンドセッションの worktree 隔離挙動。`"none"` を指定すると `EnterWorktree` なしで作業コピーを直接編集可能。worktree が現実的でないリポジトリ向け（v2.1.143） |
| `sandbox.bwrapPath` | （Managed のみ、Linux/WSL）bubblewrap バイナリのカスタムパス指定（v2.1.133） |
| `sandbox.socatPath` | （Managed のみ、Linux/WSL）socat バイナリのカスタムパス指定（v2.1.133） |
| `parentSettingsBehavior` | （Managed のみ、admin tier）SDK `managedSettings`（parent tier）をポリシーマージに含めるか（`first-wins` / `merge`）。v2.1.133 |
| `allowAllClaudeAiMcps` | （Managed のみ、Enterprise）`managed-mcp.json` と並んで claude.ai クラウド MCP コネクタを一括ロード（v2.1.149） |
| `pluginSuggestionMarketplaces` | （Managed のみ）コンテキストアウェア tips 経由で suggest 対象とする組織マーケットプレースを allowlist 化（v2.1.152） |
| `<marketplace>.skipLfs` | プラグインマーケットプレース定義（`github` / `git` ソース）に `skipLfs: true` を指定すると Git LFS ダウンロードをスキップ（v2.1.153） |
| `archive` プラグインソース | HTTPS 経由の zip からプラグインをインストールするソースタイプ（v2.1.224）。任意で SHA-256 ハッシュピン止めによる完全性検証に対応。`owner/*` 形式のオーナーワイルドカードをマーケットプレース managed settings に指定可（v2.1.223） |
| `<plugin>.defaultEnabled` | プラグインの `plugin.json` またはマーケットプレースエントリで `defaultEnabled: false` を指定するとインストール後デフォルト無効。`/plugin` または `claude plugin enable` で有効化。依存関係は引き続き自動有効化（v2.1.154） |
| `requiredMinimumVersion` | （Managed のみ）Claude Code の最小許容バージョン。範囲外なら起動を拒否し承認済みバージョンへ案内（v2.1.163） |
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
| `crossSessionInbound` | セッション間メッセージ（`SendMessage`）の受信を制御（v2.1.224）。`accept`（配送） / `hold`（通知のみ・未配送、後で `accept` が適用されれば解放） / `refuse`（黙って破棄）。`/config` の「Messages from your other sessions」行からも設定可（v2.1.232、書き込み先は user 設定。`/config crossSessionInbound=value` ショートハンドは拒否される）。**未設定時は送受信両セッションの権限モードクラスから自動判定**（bypassPermissions 系 vs プロンプト系。詳細は agent-teams.md 参照） |
| `dialogExpiry` | セッション間メッセージのダイアログ有効期限を設定（v2.1.224）。v2.1.232 で `/config` に「Dialog expiry」「Messages from your other sessions」の行が追加され GUI から設定可能に |
| `extraKnownMarketplaces` | 既知プラグインマーケットプレースの追加登録。v2.1.232 で `additionalMarketplaces` が別名として受理される |
| `sandbox.ripgrep` | サンドボックスが使う ripgrep バイナリの指定。v2.1.232 で user / managed / `--settings` 由来のみ有効化（プロジェクト設定からの上書き不可）。サーバー管理設定からの上書きは承認必須 |
| `isolatePeerMachines` | `true` でマシン外のセッションへの `SendMessage` 送信前にユーザー承認を必須化。`bypassPermissions` モードでも承認を求める。**いずれかのスコープの `true` が優先**されるため、コミット済みプロジェクト設定で有効化はできても無効化はできない。同一マシン内のメッセージには適用されない |

> 補足: `allowedMcpServers` / `deniedMcpServers` 述語の `${VAR}` 参照は v2.1.166 で正しくマッチするようになった。Managed settings は invalid なエントリが含まれても、v2.1.166 から残りの valid policy は enforce される（従来は silently 全無効化）。

#### セルフホスト環境（self-hosted-runner）— v2.1.224 で公開ベータ

Team / Enterprise 向けに、クラウドセッションを自社ホスト上で実行する `claude self-hosted-runner` コマンドが追加された（Linux / macOS）。Owner/admin が管理画面で「Allow self-hosted environments」を有効化して利用する。詳細な全フラグは `claude self-hosted-runner --help` および [reference](https://code.claude.com/docs/en/self-hosted-environments-reference) 参照。デプロイインフラ寄りのためここでは**ハーネス設計に直接関わるガードのみ**を段階的開示で記載する:

- `--confine-repo-settings <warn|enforce|off>`（既定 `warn`）: リポジトリにコミットされた `settings.json` がセッション自身のワークスペース外への読み書き付与・環境変数設定・オペレータの sandbox/hooks ポスチャ上書き（例 `sandbox.enabled: false`、`disableAllHooks`）を試みた場合にフラグ立て。`enforce` で拒否
- `--trust-workspace [bool]`（既定 on）: リポジトリコミット済みの `permissions.allow` / `additionalDirectories` を信頼するか。`false` でリポジトリ由来の権限付与を破棄しホスト側 `settings.json` の allow ルールを使用（`sandbox.*` は常に適用され `--confine-repo-settings` の走査対象）
- 追加サブコマンド `self-hosted-runner orchestrator`（オンデマンド runner の自動起動）、環境変数は `SELF_HOSTED_RUNNER_*` / `CLAUDE_RUNNER_*` 系

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

## 6. 環境変数

主要な環境変数（`settings.json` の `env` キーまたはシェルで設定）:

| 変数名 | 用途 |
|:--|:--|
| `ANTHROPIC_API_KEY` | APIキー |
| `ANTHROPIC_MODEL` | モデル指定 |
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
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | `1` でサブプロセス環境から認証情報を除去（v2.1.83） |
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
| `ENABLE_PROMPT_CACHING_1H` | `1` でプロンプトキャッシュ TTL を 1 時間化（v2.1.108） |
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
| `CLAUDE_CODE_TOOL_MEMORY_LIMIT` | （Linux、オプトイン）Bash ツールコマンドに memory cgroup を適用し、暴走ビルドによるセッション停止を防ぐ（v2.1.233） |
| `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` | WebFetch のセッション内 URL キャッシュ TTL を設定（デフォルトは従来通り 15 分）（v2.1.233） |
| `CLAUDE_CODE_ENABLE_TODO_TOOLS` | `1` で Todo / タスク追跡ツール（`TaskCreate` / `TaskGet` / `TaskUpdate` / `TaskList` / `TodoWrite`）を復活させる。**v2.1.233 で Opus 4.8 / Sonnet 5 / Fable 5 / Mythos 5 以降のモデルでは既定で提供されなくなった**ため、これらに依存するハーネスは明示設定が必要 |

> 補足: `OTEL_LOG_TOOL_DETAILS=1` は v2.1.157 で `tool_decision` イベントに `tool_parameters`（bash コマンド、MCP/skill 名等）を追加する効果も併せ持つようになった。

> Workflow キーワードトリガー: v2.1.157 から `/config` の「Workflow keyword trigger」設定で、プロンプト中のトリガーキーワードがワークフロー要求を発動するかをユーザー設定できる。**v2.1.160 でトリガーキーワードが `workflow` → `ultracode` にリネーム**。

完全な環境変数リファレンス: https://code.claude.com/docs/en/env-vars

---

## 参考リンク

- 設定: https://code.claude.com/docs/en/settings
- メモリ: https://code.claude.com/docs/en/memory
- 環境変数: https://code.claude.com/docs/en/env-vars
- 権限: https://code.claude.com/docs/en/permissions
- サンドボックス: https://code.claude.com/docs/en/sandboxing
