# Codex CLI 変更一覧

公式changelogを端的にまとめたもの。マイナーバグ修正は省略。
公式: https://developers.openai.com/codex/changelog

最終更新: 2026-06-04

---

## CLI 0.137.0-alpha.4 / .5 (2026-06-03, プレリリース)

GitHub Releases に `rust-v0.137.0-alpha.4`（2026-06-03 01:26 UTC）と `rust-v0.137.0-alpha.5`（2026-06-03 17:23 UTC）が公開。リリースノートは空のため、`rust-v0.136.0 stable` (2026-06-01) 以降の `main` ブランチコミットから主要なユーザー向け変更点を整理。安定版 0.137.0 リリース時に内容を再確認・統合する。

### Skills 拡張機能（新規）

- **v1 Skills 拡張プロンプト注入**: ターン入力コンテキストからスキルカタログを per-turn 解決し、関連スキルの SKILL.md をプロンプトに注入する仕組み（拡張機能ベース）
- **拡張機能 turn-input contributors**: 拡張機能がターン入力に文脈を寄与できる汎用フレームワーク
- **Skills 拡張スキャフォールド** および **拡張機能の context-fragments クレート分離**

### Multi-Agent v2（MAv2、dogfood デフォルト）

- **MAv2 を dogfood デフォルトに設定**
- **`assign_task` を `followup_task` に改名**
- **`close_agent` の自己ターゲット指定を拒否**
- **`hide_spawn_agent_metadata` デフォルトを `true` に変更**: spawn_agent のメタデータ表示を既定で隠す
- **MAv2 runtime metadata**: 種類、永続化、per-thread 解決、selector override 対応
- **Codex turn events に workspace kind を伝搬**

### Goal 拡張

- **idle continuation**: ゴール継続時に idle ターンを注入（Plan-mode ゲート撤廃）
- **GoalApi 追加**
- **goal progress accounting のシリアライズ修正**
- **/goal edit の複数行ペースト修正**
- **goal steering プロンプトをテンプレート化**

### Remote control / app-server / Cloud config bundle

- **remote-control に client management RPCs 追加**（app-server）
- **remote-control ペアリング開始（pairing start）追加**
- **request_permissions に environmentId 追加**、environment 単位での grant key 化
- **app-server: 環境パス参照（environment path refs）追加**
- **app-server: 実験的 `persist_extended_history` bool flag を削除**（汎用化に統合）
- **Cloud config bundle**: 設定をクラウド管理 layer に移行する大規模リファクタ。requirements layer の合成、bundle transport types、サービスモジュール分割、EDU アカウントでも bundle 取得可能
- **runtime を cloud config bundle に切替**
- **app-server: websocket 無効時の startup prewarm をスキップ**

### TUI / コマンド改善

- **検索可能セレクションメニューでペースト許可**
- **TUI: 関数キー F1〜F24 をキーマップで使用可能に**
- **スラッシュポップアップ: フィルタ変更時に選択をリセット**
- **TUI: 中断したプロンプトの出力なし表示を復元**
- **reasoning-only ステータスサーフェスアイテムを追加**
- **TUI: フッターショートカットオーバーレイヒントの文言を明確化**
- **standalone web search 並列実行を有効化**
- **standalone 画像生成を Code Mode で公開**（host finalization md 経由）

### Sandbox / セキュリティ / 認証

- **暗黙の sandbox デフォルトを permission profile から導出**: ビルトイン権限プロファイルを raw policies から派生
- **MITM CA trust を子環境に伝達**（管理 CA 対応）
- **dead profile sandbox fallback を削除**
- **exec-server: バインドされたファイルシステムパスを canonicalize**
- **MAv2 connector レベル Guardian reviewer オーバーライド**
- **EDU アカウントが cloud config bundle を取得可能に**
- **Codex async main を sized stack で実行**: セッション起動・config rebuild のスタック圧迫を軽減

### Plugins / 補助

- **プラグイン skill の base name を検証**
- **インストール済みローカル/リモートのキュレートプラグインを重複排除**
- **リモートプラグインカタログをサジェストのためにキャッシュ**
- **プラグインリストの JSON 出力対応**
- **無効なプラグイン skills manifest フィールドのハンドリング**
- **macOS で codex アプリのパスにディープリンクを使用**

### コールド rollout 圧縮（パフォーマンス）

- **コールドローカル rollout を圧縮**: append 前に materialize し読み取り、並列圧縮、繰り返し走行のスロットル化
- **rollout 圧縮 counters / histograms 追加**
- **圧縮済み rollout の検索スニペット再利用**

### 修正

- **Vim の複数行ペースト** が `/goal edit` で機能しない問題
- **named thread タイトル**が reconciliation で上書きされる問題
- **Windows**: thread resume パス正規化、PDB staging、BuildBuddy Bazel wrapper 実行、setup helper の UAC マニフェスト復元
- **Linux exec-server / app-server**: SQLite intrinsics を Windows x64 で無効化、git CLI 経由の Cargo fetch
- **/diff の git enrichment ガード**
- **codex exec で auto-review approval policy を保持**

### 補足

- 0.137.0-alpha.4 と alpha.5 は同日リリース。alpha.4 〜 alpha.5 間の差分は MAv2 final touches（`close_agent` self-target 拒否、v1 skills 拡張プロンプト注入、ゴール進捗シリアライズ、複数行ペースト、git enrichment ガード等）と内部リファクタが中心
- スキル拡張機能・MAv2 のデフォルト化・cloud config bundle が 0.137.0 系の 3 大テーマ

---

## CLI 0.136.0 (2026-06-01)

- **セッションアーカイブ**: TUI で `/archive`、CLI で `codex archive` / `codex unarchive`。アーカイブ済みセッションは復元するまで resume / fork から保護
- **TUI Markdown 強化**:
  - **OSC 8 web リンク**: rich content で外部リンクをクリック可能に保持
  - **詰まった markdown テーブルを key/value レコード化**: リンクターゲットを保持しつつ可読性を確保
- **App-server 機能拡張**:
  - **thread resume に初回ターンページを含める**: 復帰直後にターン履歴が見える
  - **MCP server status のリッチ化**: `MCP server info` を server status の一部として公開
  - **`codex app-server --stdio` モード**: stdio 起動エイリアス追加
- **リモート実行強化**:
  - **`CODEX_API_KEY` による API キー登録**: 承認済み OpenAI ホスト向けに remote exec-server 登録を許可
  - **Remote-control websocket を短命サーバートークンへ移行**: 旧 ChatGPT access token を廃止し短命 server tokens を使用
- **Windows 改善**:
  - **`codex sandbox setup --elevated`（alpha）**: Windows 管理者向けに sandbox provisioning パスを追加
  - **Windows sandbox requirements の制約強化**: 許可される Windows サンドボックス実装の requirements サポート
- **Bedrock 改善**:
  - **API キー リージョンフォールバック**: 認証時に `AWS_REGION` / `AWS_DEFAULT_REGION` をフォールバック
  - **GPT モデル を default service tier に制限**: サポート外の Bedrock GPT service tier を広告・送信しない
  - **カタログ更新**: GPT-5.5 追加、サポート外 OSS モデルを除去
- **画像生成エクステンション（feature-gated, standalone）**: ネイティブ Codex image artifact completion パイプライン経由で動作
- **`experimental_request_user_input` 設定トグル追加**
- **セキュリティ修正**:
  - **`/diff` でリポジトリ提供 Git ヘルパー / フック実行を防止**
  - **PowerShell パーサ実行を非 Windows ホストで回避**
  - **exec-server websocket: Origin ヘッダ付きリクエストを拒否**（ブラウザ起点を排除）
  - **Sandbox: deny-read 強制を safe-command / approval-bypass 経路でも維持**
- **認証**:
  - **ChatGPT access token を 5 分の expiry 前に refresh**
  - **`refresh_token_reused` 400 を relogin-required として扱う**（汎用エラーに潰さず再ログインを促す）
- **修正**:
  - **Vim normal mode 編集の修正**: 既知の編集挙動不正を解消
  - **複数行 hook 出力を TUI で別行レンダリング**
  - **Resume 時のプロンプト履歴をセッショントランスクリプトから seed**
  - **fs/watch debounce のバッチング修正**（後続バッチが正しく debounce）
  - **standalone web search のアクティビティ表示・復元**
  - **Linux sandbox: 中断時のシェルクリーンアップを保持**
  - **Windows sandbox: ネットワーク拒否時にキャンセル**
- **内部 / 開発**:
  - **MCP 依存を `rmcp` 1.7.0 に更新**
  - **Python SDK ベータの配布パスを `pip install openai-codex` に統一**（クイックスタート / API ref / FAQ / 例を刷新、独立リリース可能化）
  - **Built-in tool schema の説明改善**（shell / Code Mode / MCP / image / goal / plan / multi-agent でデフォルト・オプショナル・境界・enum を明示）
  - **debug-client（stale app-server デバッグクライアント）削除**

---

## CLI 0.136.0-alpha.1 (2026-05-29, プレリリース)

- プレリリース版（GitHub Releases に prerelease タグで公開）。安定版 0.136.0 が 2026-06-01 公開されたため、本項は履歴目的で残置。
- 安定版の主な変更点は 0.136.0 (2026-06-01) 参照。

---

## CLI 0.135.0 (2026-05-28)

- **`codex doctor` 拡張診断**: 環境・Git・ターミナル・app-server・thread inventory のリッチな診断情報を出力（サポートケース用）
- **`/status` がリモート接続情報を表示**: TUI がリモートトランスポートで接続されている場合、接続詳細とサーバーバージョンを表示
- **Vim mode 強化**:
  - text-object 編集対応（vim text object bindings）
  - 単語末尾 / 行末挙動を改善
  - ターン中断 keybind を設定可能化
- **`/permissions` で named permission profile に対応**: プロファイル名指定・カスタムプロファイル表示
- **macOS / Linux で bundled patched zsh helper を発見・利用可能**: パッケージ済み Codex ビルドが対応ターゲットで採用
- **Python SDK に `Sandbox` プリセット API**: thread / turn API でフレンドリーな sandbox preset を公開
- **TUI 表示改善**:
  - Markdown テーブルのカラム配分とアプリスタイル描画
  - 複数行 markdown リストの可読性
  - macOS / Zellij で composer / 生出力の崩れを防止
- **修正**:
  - スラッシュコマンド補完が引数を取るコマンドのドラフトテキストを保持
  - 古い tmux / iTerm control-mode で Ctrl-C が機能しない問題
  - アプリメンションがアクセス不可・無効アプリを除外
  - resume フローが非対話 exec セッションを含め、idle cached thread の cwd override を尊重
- **内部**: メモリランタイム状態を専用 SQLite DB に移動、レスポンス再試行ロジック中央集約、レガシー config-profile consumer 廃止、Rust toolchain ピン更新（1.95.0）

---

## CLI 0.134.0 (2026-05-26)

- **ローカル会話履歴の全文検索**: 大文字小文字非依存のコンテンツマッチ＋結果プレビュー付きで rollout-backed なスレッド検索を提供
- **`--profile` がプライマリプロファイルセレクタに昇格**: CLI / TUI permissions / sandbox フロー全体で正式採用。legacy プロファイル設定は migration guidance 付きで拒否（v1 plumbing 撤去、`--profile` で `codex sandbox` も指定可能、`mcp` 配下でも guidance を表示）
- **MCP セットアップ強化**:
  - per-server environment targeting（`mcp servers` を明示的な環境にルーティング）
  - streamable HTTP MCP サーバーの OAuth options を `codex mcp add` で指定可能
- **Connector tool schemas の信頼性向上**: ローカル `$ref` / `$defs` 構造を保持しつつ、過大スキーマは公開前に best-effort で圧縮
- **`readOnlyHint` 付き MCP ツールの並列実行**: read-only と annotate された MCP ツール呼び出しを並行実行
- **Extension / Hook コンテキスト拡張**:
  - 拡張ツールに会話履歴を公開
  - subagent identity を hook input に追加
- **Goal の予算制限ターン steering**: budget-limited goal extension turns を steering
- **Plugin Hooks 機能フラグ撤去**: opt-in plumbing が完全撤去され default に
- **Workspace usage limit メッセージ**: credit / spend-cap 失敗時にワークスペース固有の usage-limit メッセージをレスポンスヘッダから表示
- **Bedrock Mantle GovCloud リージョン追加**
- **enterprise requirement gate 追加**
- **Windows / リモート信頼性**:
  - Windows TUI レンダリング崩れ修正（描画前に virtual terminal モードを復元）
  - Windows サンドボックスログを rolling files 化
  - 切断した exec-server websocket クライアントを新セッションで再接続
  - auth 復旧直後の remote control 即時リトライ
  - remote compaction v2 stream のリトライ
- **`auto-review` の permission profile override 修正**: runtime 設定同期時にアクティブな permission profile メタデータを保持
- **Node ベースツールの managed network proxy 環境変数尊重**
- **App-server API 拡張**:
  - `TurnStartedEvent` に `trace_id` を追加
  - `codex-api` に typed Images クライアント追加
  - app-server のオプショナル bool アノテーション修正
- **TUI `codex-tui.log` を opt-in 化**
- **CLI が host sandbox backend を自動推論**
- **プラグインスキルが plugin-level の共有アイコン資産を再利用可能に**
- **リリースパッケージング**:
  - macOS x64 zsh アーティファクト追加
  - V8 アーティファクトを使ったリリースビルド
  - DotSlash 実行可能ファイル取得の共通化
  - npm package 旧 artifact synthesis 廃止
- **ドキュメント**: README に curl/PowerShell インストーラ追記、開発者ドキュメントは `cargo test` より `just test` を優先、profile error メッセージに migration ドキュメントへのリンク

## CLI 0.133.0 (2026-05-21)

- **Goals がデフォルト有効化**: 専用ストレージにバックされ、アクティブターン横断で進捗を追跡。実験的フラグから卒業
- **`codex remote-control` がフォアグラウンドコマンド化**: ready 待機・マシンステータス報告を行う通常コマンドとして動作。明示的な daemon 風 `start` / `stop` サブコマンドは引き続き利用可能
- **Permission profiles の大幅強化**:
  - 一覧 API、継承（inheritance）対応
  - `requirements.toml` 経由の managed permission profile サポート
  - ランタイムでのアクティブプロファイル更新
  - Windows サンドボックスとのより強い統合（permission profile を elevated runner に渡す、profile-native elevated API 等）
- **プラグイン discovery の可視化強化**:
  - marketplace-aware な list 出力、インストール済みバージョン表示
  - discovery が考慮する marketplace ルート一覧表示
  - vertical remote plugin collection サポート
  - プラグイン MCP tool metadata / tool call items に plugin id を含める
- **拡張機能（extensions）が観測できるライフサイクルイベントを拡大**:
  - **SubagentStart / SubagentStop hook 追加**
  - **Compact SessionStart hook サポート**
  - ツール実行、ターンメタデータ（turn_id, truncation_policy 含む）の公開
  - 非同期 approval / turn item 処理 contributor 追加
- **MITM hook ランタイム enforcement**: MITM hook config モデルと named MITM permissions config を導入し、ランタイムに組み込み
- **`/btw` の side slash command alias 追加**
- **app-server API 拡張**: `thread/settings/update` API、`codex-app-server --version` フラグ、CUA requirements 用 locked computer use の docs / schema 拡張
- **`codex exec-server` に strict config モード追加**
- **デフォルトの安全側変更**: filesystem permission entry のデフォルトを `deny` 側に統一。承認無効時の read-only fallback を拒否
- **修正**: TUI の起動時カレントディレクトリ誤検出、plan-mode の Shift+Enter 等 modified Enter 誤送信、stale な background terminal poll events、raw code-mode exec 出力の保持、AGENTS.md 読み込みの信頼性向上（無効 UTF-8 警告等）、app-server 起動/シャットダウン競合、realtime v1 websocket 互換性
- **ビルド/配布**: Codex package archive パイプライン導入（installers, npm packages, DotSlash, SDK runtimes が共通レイアウトへ移行）。Linux Python runtime wheel の glibc tag を修正

## CLI 0.132.0 (2026-05-20)

- **Python SDK の認証一級化**: API キーログイン、ChatGPT ブラウザ/デバイスコードフロー、アカウント検査、ログアウト API を SDK 内蔵
- **Python turn API の使い勝手向上**: 入力に素の文字列を直接渡せる。handle-based runs は `TurnResult`（collected items、タイミング、usage data）を返却
- **`codex exec resume --output-schema` 対応**: resume 後の自動化でセッションコンテキストを維持しつつ構造化 JSON 出力を強制可能
- **TUI 起動高速化**: ターミナル capability 探査をバッチ化し、最初の対話フレーム表示までの待ち時間を短縮
- **Remote executor 登録に標準 Codex auth を使用可能**: 別の registry credential フロー不要に
- **App-server で画像詳細度の保持**: ローカル画像のオリジナル解像度をユーザー入力と image-producing tool を通じて維持
- **Goal 継続ループの停止条件追加**: usage limits や繰り返しブロッカーに当たった時点でループ継続を停止（トークン無駄消費の防止）。完了レスポンスの usage 表現も自然に
- **セッションピッカー改善**: リネームされたスレッドは `name (thread-id)` 形式で resume ヒントに表示。検索ボックスにペースト可能
- **マルチセッション TUI 信頼性向上**: 進行中の MCP 呼び出しが replay 中も active 表示維持、elicitation reply が要求元スレッドへ正しくルーティング
- **リモートセッション堅牢化**: websocket keepalive、diff パスを `/tmp/...` プレフィックスではなく repo 相対で表示
- **Windows インストール強化**: `codex doctor` が npm-managed install を正しく検出、MSVC リリースバイナリが別途 VC++ ランタイム DLL に依存しないように
- **TUI 仕上げ**: 終了時の即時シャットダウンフィードバック、非 OpenAI プロバイダで ChatGPT usage リンクを非表示、side-thread resume 後に Fast tier クリア状態が復活しないように
- **メモリ要約のバージョニング**: stored format が古い場合に再生成。長寿命メモリコンテキストをよりリーンに

## CLI 0.131.0 (2026-05-18)

- **`codex doctor` コマンド追加**: runtime, auth, terminal, network, config, ローカル状態を横断するサポート向け診断ツール
- **`codex remote-control` の daemon 化**: daemon ライフサイクル管理、ランタイム enable/disable API、ステータス読み取り、registry-backed 環境を追加
- **プラグインマーケットプレース CLI 追加**: `codex plugin` 配下にマーケットプレース操作コマンドが追加。バージョン対応の share、share checkout、共有ワークスペースの discoverability 区分も導入
- **プラグイン Hooks がデフォルト有効化**: 旧来の opt-in から default-enabled に変更
- **統一 `@` メンション**: ファイル / ディレクトリ / プラグイン / スキルを単一ピッカーで検索（app-server プラグインメタデータ駆動）
- **TUI 表示の刷新**:
  - サービスティアのスラッシュコマンドがデータ駆動に
  - ステータスラインに blended token count・権限・承認モードを表示
  - 有効ワークスペースルートを exec/summary 表示に追加
  - レスポンシブ Markdown テーブルレンダリング
- **Python SDK が `openai-codex` / `openai_codex` にリネーム**: ピン留め runtime-generated types、並行ターンルーティング、承認モード API、app-server 統合ハーネスを追加
- **`/goal edit` コマンド追加**: 既存ゴール内容を TUI から編集可能に
- **`--dangerously-bypass-hook-trust` CLI フラグ追加**: hook trust フローを意図的にバイパス（CI 等向け）
- **`--profile-v2` レイヤー化プロファイル設定**: 複数 TOML を重ねるプロファイル v2。旧 `[profiles]` 併用時は明示拒否
- **strict config parsing**: 設定スキーマ外フィールドの厳格チェック
- **Network proxy feature flag**: ネットワークプロキシ機能の段階的ロールアウト用フラグ
- **Windows hook command overrides**: Windows 上で hook コマンドをプラットフォーム別に差し替え可能
- **Multi-environment `apply_patch` 選択**: 複数環境に跨る apply_patch のターゲット指定
- **Windows サンドボックス強化**: deny-read parity、scoped write root SID、firewall policy が無効な場合の elevated setup 失敗
- **権限永続化の堅牢化**: escalation 中の managed deny-read 維持、workspace-roots と danger-full-access の正規化
- **SQLite/状態起動の安全化**: 破壊的バージョンバンプ廃止、state db オープン失敗時の fail-closed、復旧パス追加
- **`git` 周り改善**: 連結 worktree でルート repo hooks を使用、helper コマンドで repo hook/fsmonitor 設定を無視、login OAuth のローカルコールバック binding、再ログイン時の旧トークン失効
- **ambient terminal pets** (`tui.pets`): TUI にデコラティブなペットを表示（実験的）
- **削除・非推奨**:
  - `/collab` スラッシュコマンドを削除
  - 組み込み MCP を廃止（プラグイン経由で利用）
  - `experimental_use_freeform_apply_patch` / `windows_wsl_setup_acknowledged` / `tools.view_image` / `Feature::CodexGitCommit` 等の設定を削除
  - レガシー after-tool-use hooks を削除
  - Issue labeler を非推奨化
- **主要バグ修正**:
  - TUI: URL 周囲のテキスト wrap、ライトモード選択のコントラスト、tmux 内 Shift+Enter、`/review` MCP 起動表示、`/side` の Esc 抑止
  - exec-server: Windows での `taskkill` 出力抑制、transport timeout 延長
  - 設定: TUI keymap で `minus` を許容

## CLI 0.130.0 (2026-05-08)

- **`codex remote-control` コマンド追加**: ヘッドレス・リモート制御可能な app-server を起動するシンプルなエントリポイント
- **プラグイン共有・詳細表示の拡張**: プラグイン詳細にバンドル済み hooks を表示。プラグイン共有でリンクメタデータと discoverability コントロールを公開
- **App-server: 大規模スレッドのページング対応**: unloaded / summary / full ターン項目ビューで巨大スレッドを段階ロード
- **Bedrock auth が `aws login` プロファイル credentials を利用可能に**: AWS console-login credentials を直接使用
- **`view_image` がマルチ環境セッションで選択環境経由でファイルを解決**
- **重要バグ修正**:
  - ライブ app-server スレッドが再起動なしで設定変更を取り込むように
  - `apply_patch` の部分失敗（ファイル変更済みでも失敗）でターン差分が正確に保たれるように
  - `ThreadStore` 経由のスレッドサマリ・リネーム・resume・fork 経路改善（ローカル rollout パスがないスレッドを含む）
  - リモートコンパクション v2 ストリームで `response.processed` を発行、API キー compact リクエストでの `service_tier` 送信回避
  - Windows サンドボックスでサンドボックスユーザーがデスクトップランタイムバイナリキャッシュにアクセス可能に
  - `codex exec` 起動バナーから陳腐化した「research preview」文言を削除
- **OpenTelemetry 拡張**: トレースメタデータの設定可能化、レビュー/フィードバックアナリティクス強化
- **依存関係衛生**: GitHub Action の完全修飾ピン、Dependabot 7日クールダウン、`cargo-shear` 1.11.2 アップグレード

> 注: 0.126.0 / 0.127.0 / 0.130.0 はステーブル前段で多数の alpha が出たが、`-alpha` を含まないタグが付与されたのは 0.128.0 → 0.130.0 のジャンプで、stable ラインは 0.128.0 → 0.129.0 → 0.130.0 と進行している。

## CLI 0.129.0 (2026-05-07)

- **TUI コンポーザーの Vim モーダル編集対応**: `/vim` コマンドで Vim キーバインドを有効化
- **ワークフロー再開ピッカーを刷新**: より明確なバリデーションとセッション間の再開操作を改善
- **Raw scrollback モード**: 端末ネイティブスクロールバックを活用するレンダリングモード
- **ワークスペース対応の `/diff` コマンド**: 複数ワークスペースを跨いだ差分表示
- **ステータスラインがテーマ対応化**: PR/ブランチ変更サマリ表示のオプション追加
- **プラグイン管理拡張**: ワークスペース共有、アクセス制御、ソースフィルタ、ローカルパストラッキング、マーケットプレース操作を強化
- **実験的 Goals 機能**: 検出可能化、一時停止状態の永続化、再開セッション横断のバリデーション明確化
- **Linux サンドボックス強化**:
  - スタンドアロン `bwrap` フォールバックを Linux リリースに同梱
  - Bubblewrap vendoring を 0.11.2 に更新（上流のセキュリティ変更を取り込み）
  - 古い `bwrap` バージョンと symlink 保護パスでの信頼性向上
- **その他バグ修正**: tmux 互換 `/copy`、Alt+Enter 挙動修正、Windows タイピングレイテンシ低減

## CLI 0.128.0 (2026-04-30)

- **`/goal` 永続化ワークフロー導入**: app-server API・モデルツール・ランタイム継続・TUI コントロール（create / pause / resume / clear）を備えた目標駆動ワークフロー
- **`codex update` コマンド追加**: CLI 自身を最新版にアップグレード
- **TUI キーマップを設定可能に**: ユーザー定義のキーマップ対応
- **プランモードのナッジ強化**: ユーザーへの誘導表示改善
- **Action-required ターミナルタイトル**: 承認待ち等の状態をターミナルタイトルに反映
- **アクティブターン中の `/statusline` / `/title` 編集**: ターン進行中でも変更可能
- **権限プロファイル拡張**: ビルトインデフォルト・サンドボックス CLI プロファイル選択・cwd 制御・アクティブプロファイルメタデータをクライアントに公開
- **プラグインワークフロー強化**: マーケットプレースインストール、リモートバンドルキャッシュ、リモートアンインストール、プラグインバンドル hooks、hook 有効化状態、外部エージェント設定インポート
- **外部エージェントセッションインポート**: バックグラウンドインポートとインポート済みセッションタイトル処理
- **MultiAgentV2 設定の明示化**: スレッド上限、待機時間制御、root/subagent ヒント、v2 固有の depth 処理
- 主要バグ修正:
  - resume / interruption 関連（stale interrupt ハング、永続化プロバイダー復元、巨大リモート resume レスポンス、フィルター済み resume 一覧の遅延）
  - TUI 安定性（リサイズリフロー、Markdown リスト間隔、スラッシュコマンドポップアップレイアウト、シェルモード Esc）
  - Managed network 強化（deferred denials、proxy bypass デフォルト、IPv6 ホスト一致、`git -C` 承認）
  - Windows サンドボックス / PTY エッジケース（pseudoconsole 起動、elevated runner、コアシェル環境継承、名前付きパイプ検証）
  - Bedrock の `apply_patch` / GPT-5.4 reasoning level / GPT-5.4 エンドポイント・モデルメタデータ
  - MCP / プラグイン（stdio サーバークリーンアップ、プラグイン MCP 承認永続化、カスタム MCP メタデータ分離）
- ドキュメント更新: バンドル OpenAI Docs スキルが GPT-5.5 / `gpt-image-2` 対応、アップグレードガイダンス明確化

> 注: 0.126.0 / 0.127.0 はステーブルとしてリリースされず（alpha のみ）、0.125.0 → 0.128.0 にバージョンスキップ。

## CLI 0.125.0 (2026-04-24)

- **App-server 統合の拡張**: Unix socket トランスポート対応、ページネーション対応の resume/fork、固定環境（sticky environments）、リモートスレッド設定/ストア機能を追加
- **権限プロファイルの永続化**: TUI セッション・ユーザーターン・MCP サンドボックス状態・シェルエスカレーション・app-server API を横断して `permission profiles` が保持される
- **リモートプラグインのインストール / マーケットプレースアップグレード**: ローカル外のプラグインを直接インストール・更新可能
- **モデル検出強化**: AWS / Bedrock アカウント状態を含むモデルカタログ検出
- **`codex exec --json` に reasoning-token 報告追加**: プログラマティック利用者向けに reasoning-token の使用量が出力に含まれる
- **ロールアウトトレーシング**: ツール、コードモード、セッション、マルチエージェント関係を記録するデバッグリデューサー機能
- App-server: 明示的に信頼されていないプロジェクト設定の自動永続化を回避
- **設定スキーマ**: MultiAgentV2 のスレッド制限競合検出、相対エージェント設定パスの解決、MCP bearer-token の非対応フィールド隠蔽、`js_repl` 無効 MIME 型の拒否
- TUI: `/review` 中断と終了時にインターフェースが固まる問題を修正
- Exec-server: プロセス終了後のバッファ出力保持、ストリーム完全クローズ待機を修正
- WebSocket: ターンとツール出力通知バースト時の切断問題を軽減
- Windows サンドボックス: 複数 CLI バージョンと設定ディレクトリの互換性向上、PowerShell 表示ウィンドウの非表示化

## CLI 0.124.0 (2026-04-23)

- **TUI 推論コントロールのクイック操作**: `Alt+,` で推論レベル下げ、`Alt+.` で上げ
- **app-server が複数環境を管理**: ターンごとに環境/作業ディレクトリを選択可能
- **Amazon Bedrock ファーストクラス対応**: OpenAI 互換プロバイダーとして AWS SigV4 署名込みで組み込み
- **リモートプラグインマーケットプレース**: 直接一覧・読み取り可能、より大きなページサイズ対応
- **Hooks が正式化（stable）**: `config.toml` にインライン設定可能、managed `requirements.toml` で管理
- **Fast サービスティアがデフォルト**: 対象 ChatGPT プランでは明示オプトアウトしない限り Fast を利用
- Cloudflare Cookie が承認済み ChatGPT ホスト間で保持
- リモート app-server の websocket イベント排出信頼性向上
- `/permissions` 変更時の permission モードドリフト修正
- `wait_agent` がメールボックスに作業がキューされている際に即返却
- ローカル stdio MCP 起動で相対コマンドが動作
- 管理対象 config エッジケースでの起動失敗削減

## CLI 0.123.0 (2026-04-23)

- **Amazon Bedrock プロバイダー対応**: モデルプロバイダーとして Bedrock が利用可能に
- **`/mcp verbose` 診断**: MCP 接続状態とツール一覧の詳細出力でトラブルシュート容易化
- **バックグラウンドエージェント Realtime ハンドオフ強化**: 進捗転送とセッション復帰が改善
- `/copy` がロールバック後も正常動作
- VS Code WSL ターミナルでの Unicode 入力修正

## CLI 0.122.0 (2026-04-20)

- **スタンドアロンインストール改善**: Windows/Intel Mac で `codex app` が Desktop を正しく開く/インストール
- **`/side` コマンド追加**: TUI サイド会話で作業中でもクイック質問可能。キュー入力はスラッシュコマンド・シェルプロンプトに対応
- **プランモード強化**: フレッシュコンテキストで実装開始可能。継続判断前にコンテキスト使用量を表示
- **プラグインワークフロー刷新**: タブブラウジング、インラインの有効/無効トグル、マーケットプレース削除、リモート・クロスリポジトリ・ローカルマーケットプレースソース対応
- **ファイルシステム権限拡張**: deny-read グロブポリシー、管理対象 deny-read 要件、プラットフォームサンドボックス強制、ユーザー設定をバイパスする隔離 `codex exec` 実行
- ツール検出・画像生成がデフォルト有効、MCP と `js_repl` 出力向けの original-detail メタデータ対応
- App-server 承認と MCP elicitation が別クライアント解決時に TUI から消える（stale プロンプト防止）
- リモートコントロール起動が ChatGPT 認証欠如に耐性、MCP 起動キャンセルが app-server セッション経由で動作
- resume/fork された app-server スレッドがトークン使用量を即時リプレイ
- **セキュリティ**: logout が管理対象 ChatGPT トークンを取り消し、プロジェクトフックは信頼済みワークスペースを要求
- sandboxed `apply_patch` が split filesystem ポリシー下で正常動作
- `SECURITY.md` にセキュリティ境界リファレンス追加（サンドボックス、承認、ネットワーク制御）

## CLI 0.121.0 (2026-04-15)

- **マーケットプレースインストール**: GitHub、git URL、ローカルディレクトリからインストール可能に
- TUI 履歴改善: `Ctrl+R` で逆方向検索
- メモリモード制御と削除エンドポイント追加
- MCP Apps のツールコール対応と並列コール対応
- Realtime API 拡張（output modality、transcript events）
- bubblewrap 対応のセキュア devcontainer プロファイル
- macOS sandbox/proxy、Windows パスマッチング、レート制限、Guardian タイムアウトの修正

## CLI 0.120.0 (2026-04-11)

- Realtime V2 のバックグラウンドエージェント進捗ストリーミング
- TUI でのフック実行可視化改善
- MCP `outputSchema` 対応（構造化ツール結果）
- `/clear` セッション向け SessionStart フック区別
- Windows elevated サンドボックスの split policy 処理、symlink writable root、TLS websocket、ツール検索順序、live stop-hook、MCP クリーンアップの修正

## CLI 0.119.0 (2026-04-10)

- **Realtime 音声セッションが v2 WebRTC デフォルト**（transport 設定可）
- MCP Apps のリソース読み取り、ツールコールメタデータ、カスタムサーバーツール検索
- Remote workflow: egress websocket、remote `--cd` 転送、ランタイムリモートコントロール
- TUI `Ctrl+O` で最新エージェント応答をコピー
- `/resume` で ID・名前によるセッションジャンプ
- TUI 起動高速化（async レート制限取得）、resume ピッカー安定化、composer 挙動改善、MCP ノイズ削減

## CLI 0.118.0 (2026-03-31)

- Windows サンドボックスがプロキシ限定ネットワーキング対応（OSレベルのegressルール）
- アプリサーバークライアントがChatGPTデバイスコードフローでサインイン可能に
- `codex exec` がプロンプト+stdin ワークフローに対応
- カスタムモデルプロバイダーが動的ベアラートークン取得に対応

## CLI 0.117.0 (2026-03-26)

- プラグインをファーストクラスワークフローに昇格
- サブエージェントがパスベースアドレスに対応
- ターミナルタイトルピッカー機能
- アプリサーバーのシェルコマンド・ファイルシステム監視機能強化
- 推論サマリー重複、ChatGPTログイン、ターミナル状態復元、サンドボックス信頼性のバグ修正

## Plugin Support Released (2026-03-25)

- プラグインをインストール可能なバンドル（skills + アプリ統合 + MCP設定）として導入
- マニフェストファイル + オプションディレクトリ構造
- ユーザー/リポジトリ単位のインストール対応（キュレーション済みリスト + ローカル開発）

## CLI 0.116.0 (2026-03-19)

- アプリサーバーTUIでChatGPTデバイスコードサインイン対応
- プラグインセットアップの改善（インストールプロンプト、同期機能）
- `userpromptsubmit` hook追加（実行前のプロンプトインターセプト）
- リアルタイムセッションが直近スレッドコンテキストで開始

## アプリ 26.323 (2026-03-24)

- 過去スレッドの検索機能（サイドバーショートカット）
- ローカルプロジェクトスレッドのワンクリックアーカイブ
- アプリとVS Code拡張間の設定同期

## アプリ 26.318 (2026-03-19)

- Skillsが `@` メニューに表示（他のメンションと並列）
- `Cmd/Ctrl+F` 検索が選択テキストから開始

## アプリ 26.317 (2026-03-17)

- **GPT-5.4 mini 利用可能** — 軽量コーディングタスク向け高速モデル
- GPT-5.4の2倍速、利用量上限の30%で動作
- 会話フォーキング（最新ターンだけでなく過去のメッセージから分岐）
- スラッシュコマンドでモデル・推論レベル切り替え

## アプリ 26.316 (2026-03-16)

- 戻る/進むナビゲーションボタン
- スレッドメニューからファイルエクスプローラショートカット

## CLI 0.115.0 (2026-03-16)

- フル解像度画像検査（`view_image`, `codex.emitImage()`）
- リアルタイムWebSocket転写モード（v2ハンドオフ対応）
- ファイルシステムRPC（Python SDK連携）
- Smart Approvals（ガーディアンサブエージェントルーティング）

## アプリ 26.312 (2026-03-12)

- カスタムテーマシステム（色・フォントカスタマイズ）
- オートメーション刷新（ローカル/worktree実行オプション）

## アプリ 26.311 (2026-03-11)

- 統合ターミナル読み取り機能（開発サーバーのステータス確認）

## CLI 0.114.0 (2026-03-11)

- 実験的コードモード（分離ワークフロー）
- **Hooksエンジン**（SessionStart, Stopイベント）
- WebSocketヘルスチェックエンドポイント
