# Claude Code 変更一覧

公式changelogを端的にまとめたもの。マイナーバグ修正は省略。
公式: https://code.claude.com/docs/en/changelog

最終更新: 2026-08-15

---

## v2.1.232 (2026-08-13)

- **サブエージェント fork がデフォルト有効化**: `subagent_type: "fork"` のサブエージェントが会話全体とプロンプトキャッシュを継承。対話セッションでのチームメイト以外のエージェントスポーンは既定でバックグラウンド実行に
- **`@` メンションによるセッション間メッセージ**: プロンプトで `@` を入力して他の Claude セッションを名前で指名すると、Claude が `SendMessage` で直接そのセッションに到達
- **`SendMessage` の名前解決簡素化**: 生の名前が1つの稼働中セッションと完全一致する場合、ref による確認を求めず直接配送
- **セッション名の一意化**: 同一マシン上の対話セッションで既存の稼働中セッションと同名を指定すると `name-word-word` のバリアントが自動付与され通知される
- **`/config` の新規行**: 「Dialog expiry」と「Messages from your other sessions」（セッション間受信の accept/hold/refuse）
- **GitLab トークンのシークレット秘匿**: `glrt-` / `gloas-` / `glptt-` / `glagent-` / `glimt-` / `glsoat-` / `glcbt-` / `glft-` / `glffct-` 系を秘匿、routable な `glpat-` / `gldt-` は完全秘匿。`glab` CLI の config store も `gh` と同等のサンドボックス・認証情報パス保護対象に
- **プラグインマーケットプレースの GitLab 対応**: 素の `gitlab.com` リポジトリ URL（ネストしたサブグループ含む）を `github.com` と同様に clone。clone 認証失敗時のヒントは実際の git ホスト名を表示
- **設定エイリアス追加**: `additionalMarketplaces` / `allowedMarketplaces` が `extraKnownMarketplaces` / `strictKnownMarketplaces` の別名として受理される
- **Enterprise ポリシー**: 素のリポジトリ URL に対する url 型 `blockedMarketplaces` エントリが、CLI が git clone と分類した場合にもブロックを継続
- **ゲートウェイ**: `desktop:` オーバーレイがリリース済み Desktop 設定を全て受理（従来は手書き11キーのみ）。Desktop 自身のスキーマで起動時検証し、未知・不正キーは起動失敗に
- **ゲートウェイ**: `managed.policies[].match.groups` / `admin.admin_groups` の空エントリ、不正な `email_domain`（空・`@`・空白・カンマを含む）を起動時に失敗させる（従来は誰にもマッチしない、または admin 権限を黙って付与）
- **Fable 5 の `/advisor` 復帰**: Fable アクセスのある組織で advisor として再提供。利用クレジット同意は `/model fable` 経由
- **セキュリティ修正**: PowerShell の権限バイパス（変数書き込みパラメータによる `$PSDefaultParameterValues` 上書き）、Windows の Git Bash が Cygwin 形式シンボリックリンクを辿る権限バイパス、ネストした git リポジトリが親ディレクトリの信頼を継承する問題（各リポジトリで個別に信頼確認が必要に）を修正
- **`sandbox.ripgrep` のスコープ制限**: user / managed / `--settings` 由来のみ有効に。プロジェクト設定からサンドボックスの ripgrep バイナリを上書き不可
- **managed settings 承認ダイアログ改善**: エンドポイント URL 表示、テレメトリのみの変更の文言明確化、定型 OpenTelemetry オプションのスキップ、サーバー管理のサンドボックスバイナリ上書き（`sandbox.bwrapPath` / `sandbox.socatPath` / `sandbox.ripgrep`）に承認を必須化
- **`/code-review`**: high / xhigh / max エフォートも他レベル同様バックグラウンドエージェントで実行
- **`/plugin install plugin@marketplace`**: 先にマーケットプレースを更新するため、新規公開プラグインを手動更新なしでインストール可能
- **Remote Control 改善**: ネットワーク断後 約30分間再接続を継続、他の Claude Code から Remote Control を黙って奪わない（移すには当該側で `/remote-control`）、テイクオーバー/他アプリでの終了/削除の区別を端末に表示。クラウドセッション内ブリッジのトランスクリプト・認証情報継承、Desktop/IDE 由来セッションの重複表示、アイドル時の到達不能表示、ワーカー再起動時の履歴復元を修正
- **その他修正**: MCP のプロトコルバージョン探索で無応答・不正応答時に30秒タイムアウトまでハングする問題、mTLS クライアント証明書ローテーションに再起動が必要だった問題、不正な AWS / Vertex リージョン値、Bedrock / Vertex / ゲートウェイでのストリームアイドルタイムアウト、`/update`・`/tui` が再起動を拒否する問題
- **セキュリティ強化**: 共有 `/tmp` 上のセッション間メッセージング用ソケットディレクトリ（事前設置シンボリックリンク・他ユーザーのディレクトリを拒否）、Linux ファイルシステムサンドボックスの保護パスバイパス、Bash の入力リダイレクト（`< file`）を全プラットフォームで権限チェック対象に
- **Cowork セッション**: user スコープのメモリファイルから外部 `@`-import をインライン展開しないように
- **削除**: カスタムサブエージェント作成を勧める起動時 tip と `/powerup` ツアーの該当ナッジ

## v2.1.231 (2026-08-13)

- **修正**: 事前登録 OAuth クライアントを使う MCP サーバー（Slack 等）で redirect URI 不一致によりサインインが失敗する問題を修正

## v2.1.229 (2026-08-12)

- **プラグインマーケットプレイスの `command` ソース**: ローカルコマンド（IDE 等）がプラグインディレクトリを出力し、毎セッション再解決してリスタート不要で適用。`mode: "link"` でその場参照
- **`claude remote-control --continue`**: 直近の Remote Control セッションを再開するオプションを文書化
- **セルフホストランナーのサーバー供給フック**: マネージド環境同様、サーバーから Claude Code フックを供給可能に
- **ゲートウェイ SSE keepalive ping**: 長い thinking 中のアイドルタイムアウト切断を防止（Vertex/Bedrock upstream）
- **`ListAgents` のステータス表示**: 切断済み Remote Control セッションを `offline`、クラウドセッションを `cloud` として表示
- **`/commit-push-pr` の安全化**: `--force`/`--amend`/`--no-verify` 等の危険フラグを含む git/gh コマンドを自動承認しないよう変更
- **ワークフロー fan-out の prefix stagger**: 同一 prefix の兄弟エージェントを時間差起動し、後続がキャッシュ済み prompt prefix を再利用（`CLAUDE_CODE_WORKFLOW_PREFIX_STAGGER_MS=0` で無効化）
- **セルフホストランナー Windows**: 起動に明示的な `--base-dir` を必須化（Windows に既定チェックアウト先なし）
- サンドボックス: ネットワークドメインリストの IPv6 リテラルをブラケット表記（`[::1]:443`）、曖昧な綴りは fail-closed で `/doctor` が警告
- CPU 制限コンテナ内の動的ワークフローがホストのコア数を使う問題を修正

## v2.1.228 (2026-08-11)

- **Write ツールの挙動変更**: 新しいモデルは当該セッションで未読のファイルでも上書き可能に（Edit ツールと同じルール）。旧モデルは従来どおり事前読み取りが必要
- **claude.ai 同期スキルのハードニング**: ローカルコマンド/MCP プロンプトをシャドウしなくなり、説明文はサニタイズ・ラベル付与。ローカルマシン上ではスキル本体が `!` コマンド実行や `@` ファイル展開を行わない
- **クロスセッションメッセージ改善**: 送信者と本文をインライン表示（従来は折りたたみ行）。別マシンの Remote Control セッション宛では送信者に自分の Remote Control セッション名を表示
- **修正**: 内部レイアウトエラー後に対話セッションが再描画停止する問題、Windows で親フォルダから起動時に git/Git Bash が見つからない問題、セッションクリーンアップがプロジェクトの memory フォルダ内を削除する問題、`self-hosted-runner` 関連の複数の問題を修正
- Vertex AI の資格情報処理改善（期限切れ/欠落時に数分リトライせず数秒で失敗）、コンパクション進捗表示の改善

## v2.1.227 (2026-08-10)

- **スラッシュコマンドメニュー改善**: 選択行のみを青でマーク、マッチした文字を太字表示、絵文字・アクセント付き名も字形を保持
- **修正**: 期限切れログイントークンでセッション開始時にサブスクリプション階層なしで機能フラグが評価される問題、GitHub-hosted runner 上で `allowed_non_write_users` 指定時に `claude-code-action` 下の全 Bash コマンドが失敗する問題、`/tui` が最初のメッセージ以前まで巻き戻された会話を復元してしまう問題を修正
- パフォーマンス改善（file-not-found サジェストと at-mention サイズチェックのイベントループ停滞を軽減）

## v2.1.226 (2026-08-08)

- バグ修正・信頼性改善のみ

## v2.1.225 (2026-08-08)

- **ゲートウェイ支出上限**: 使用量警告にゲートウェイの支出上限（spend-limit）サポートを追加。上限到達メッセージに上限額・リセット時刻・オペレーターのメッセージを表示（ゲートウェイ 2.1.225 以降が必要）
- **`claude agents` の信頼プロンプト**: 未信頼ディレクトリに対してワークスペース信頼プロンプトを表示（`claude` と同じ挙動に）
- **SendMessage の Remote Control 対応**: 別マシンの Remote Control セッションに対し、名前を指定して会話を開始できるように。確認済みの相手は同名の別セッションと入れ替わらない
- **修正**: 短命な保存ログイントークンが長命な `CLAUDE_CODE_OAUTH_TOKEN` を置き換えヘッドレスセッションを壊す問題、macOS で MCP OAuth サーバーが keychain タイムアウト後に 401 を連発する問題、auto モードが自身の権限チェックに対する安全フィルタ拒否を連続ブロック数に数える問題、大規模会話コンパクション後の Remote Control 再開で会話履歴が壊れる問題を修正
- Remote Control を改善（Claude アプリ添付写真を直接提示、コンパクション進捗表示など）

## v2.1.224 (2026-08-07)

- **セルフホスト環境**: `claude self-hosted-runner` を追加（Team / Enterprise プラン向け）
- **`archive` プラグインソース**: HTTPS 経由の zip からプラグインをインストール。任意で SHA-256 ハッシュピン止めによる完全性検証に対応
- **セッション間メッセージ**: Claude Code セッション同士が `SendMessage` で相互にメッセージ送信可能に。`crossSessionInbound` / `dialogExpiry` 設定を追加
- **サンドボックス認証情報マスキング拡充**: `extract`（抽出範囲）・`decode: "jwt"`（JWT デコード）・`awsPairs` / `sigv4`（AWS 認証情報・SigV4 署名）オプションを追加
- **修正**: 長いプロジェクトパスが別プロジェクトのセッションディレクトリに解決される問題、`SendMessage` が書き込み失敗時に成功と報告する問題、末尾スラッシュ付きサンドボックス deny エントリのバイパス問題を修正
- フルスクリーンモードの履歴保持と Remote Control 機能を改善

## v2.1.223 (2026-08-06)

- **マーケットプレース managed settings**: オーナーワイルドカードエントリ（`"owner/*"`）を追加
- **ワークフローエージェント警告**: 制限された要求サブエージェントモデルを持つワークフローエージェントに警告を表示
- **`/teleport` ヒント**: クラウドセッションをローカルで継続する際のヒントを追加
- **セキュリティ修正**: 細工された Bash コマンドによる権限バイパス、隠し Unicode 文字を含む権限プロンプト、動的 `import()` を使うワークフロースクリプトがサンドボックス外で実行される問題を修正
- **修正**: `modelOverrides` が非 Anthropic ID を正規モデル ID として扱う問題を修正

## v2.1.222 (2026-08-04)

- **ultraplan 機能削除**: `/ultraplan` によるクラウドプランニング（リサーチプレビュー）を廃止
- **セキュリティ修正**: worktree 隔離セッションとそのサブエージェントがメインチェックアウトに対して破壊的 git コマンドを実行できた問題を修正（隔離が全セッション種別のファイル編集・Bash に適用）。バックグラウンドエージェントタスク（要約・コンパクション・リネーム）で PreToolUse 自動許可フックがツール制限をバイパスする問題も修正
- **auto モード安全性向上**: `SendMessage` で他エージェントセッションへ送るメッセージを、送信前に権限クラシファイアで評価
- **`disable-model-invocation` スキルの拒否改善**: Claude がスキルを呼べない場合、ワークフローを自前で再現せずユーザーに実行を依頼するよう指示
- **Remote Control 自動起動の変更**: リポジトリローカル設定（`.claude/settings.json` / `.claude/settings.local.json`）からは有効化不可に（無効化は可能）。有効化はユーザースコープの `/config` から
- **diff 表示の変更**: `/diff`・Remote Control のワークスペース diff・web セッションのファイル編集 diff が raw git blob を使用（diff driver / textconv を無視）
- **モデルフォールバック修正**: 組織制限下の `model: opus` 系ファミリーエイリアスが親モデルに落ちる代わりに、組織で許可された同ファミリー最新モデルへ正しくステップダウン
- 修正多数: HTTPS プロキシ背後の起動時接続チェックのハング、`/usage` の MCP サーバー使用量過大計上、push 後作成された PR とセッションのリンク切れ、`ANTHROPIC_BASE_URL` ゲートウェイでのストリームアイドルタイムアウト誤発火、`SendMessage` の長いサマリ拒否（切り詰めに変更）等

## v2.1.221 (2026-08-04)

- **[VSCode] Focus view 追加**: ツール実行の詳細を折りたたみ、ターン毎サマリと実行中ツールインジケータのみ表示するチャットメニュートグル。`Ctrl+Alt+F` または「Claude Code: Toggle Focus view」コマンドで切替
- **サンドボックス認証情報の `mode: "mask"` 追加**（Linux/WSL）: サンドボックスコマンドにはセンチネル値のコピーを読ませ、egress 時にサンドボックスプロキシが実値へ置換。ファイル全体または `extract` 正規表現でキャプチャした範囲のみマスク可。macOS では `deny` にフォールバック
- **`claude plugin validate` の警告追加**: marketplace / plugin 名が Claude Desktop の managed marketplace 同期で拒否される場合に警告
- **`claude-api` スキルに `prompt-audit` サブコマンド追加**: 旧モデル向けに書かれたプロンプト・ツール記述のパターンを監査
- **セキュリティ修正**: zsh の `[[ ]]` 正規表現条件内に隠しコマンドを仕込める Bash ツール権限チェックのバイパスを修正（該当コマンドは権限プロンプトを表示）。Windows で引用符を含むパスの PowerShell 権限チェック不備も修正
- **`/fork` の挙動変更**: fork したセッションは元セッションのチェックアウトを共有せず、自前の新しい worktree を作成
- **`/status` にセッション種別表示**: `interactive`、またはバックグラウンドジョブの `attached` / `unattended` を表示
- **プラグイン改善**: `/plugin install` は stale な marketplace カタログを更新してリトライ。`/plugin` からのインストールは安全な場合 `/reload-plugins` 不要で即時有効化。`skills` パスに `"."`（プラグインルート）を指定可能に
- **バックグラウンドセッションの挙動変更**: commit・push で作業を保全し、必要な場合のみ draft PR を作成。CLAUDE.md の git 指示に従い、最後に成果物の場所を必ず報告
- **Stats パネル改善**: キャッシュトークンを合計に含め、input / output / cache read / cache write の内訳を表示
- **auto mode 改善**: 並列ツールコールの権限チェックがキャッシュ効率化され、プロンプトキャッシュの会話プレフィックス再利用でコスト削減
- その他: 絵文字補完が `:thumbsup:` 等の別名ショートコードに対応、Claude in Chrome が不要になったタブを自動クローズ、fast mode のクレジット枯渇をストリーム上で通知、Vertex AI で Claude 4.5 世代以降のツール検索を再有効化

## v2.1.220 (2026-07-25)

- バグ修正・信頼性改善のみ

## v2.1.219 (2026-07-24)

- **Claude Opus 5 追加**: `claude-opus-5` がデフォルトの Opus モデルに。1M コンテキスト、fast mode は $10/$50 per Mtok
- **`sandbox.network.strictAllowlist` 設定**: サンドボックスコマンドで許可リスト外ホストをプロンプトなしで拒否
- **`DirectoryAdded` フック追加**: `/add-dir` または SDK の `register_repo_root` でセッション途中に作業ディレクトリが登録された後に発火
- **ネストサブエージェントのデフォルト復活**: サブエージェントは既定で深度3までネスト起動可能に（v2.1.217 の既定無効化から変更）。`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH=1` で無効化
- **`workflowSizeGuideline` 設定キー**: Dynamic workflow のサイズガイドラインを任意の settings ファイルから設定可能に。Dynamic workflows のデフォルトが medium ガイドライン（15エージェント未満目安）に変更
- **stream-json のネストサブエージェント転送**: `--forward-subagent-text` 時に深度2以上のサブエージェントも、スポーン元 Agent `tool_use` id をキーに出力される
- **headless の `mcp_server_errors`**: stream-json init イベントに、設定バリデーションでスキップされた `--mcp-config` エントリの一覧を追加。ターミナル実行では起動時警告
- **fast mode の対象変更**: Opus 4.7 が fast mode から除外され、`/fast` は Opus 5 と Opus 4.8 に適用
- **MCP 接続エラーの詳細化**: `claude mcp list` / `/mcp` に HTTP ステータスとエラーテキストを表示。設定値の前後空白警告も追加
- そのほか: `claude -p` がミッドストリーム API エラー時に生成済み回答を落とす問題の修正、managed MCP 許可/拒否リストの `${VAR}` が起動時環境変数から解決されるよう変更、`--teleport` のリポジトリ不一致表示改善

## v2.1.218 (2026-07-22)

- **`/code-review` がバックグラウンドサブエージェント化**: レビュー作業が会話コンテキストを消費しなくなった。スタックされたスラッシュコマンドをレビュー対象として保持
- **`context: fork` スキルのデフォルトバックグラウンド化**: fork コンテキストのスキルは既定でバックグラウンド実行。スキルごとに `background: false` でオプトアウト
- **フロントマター真偽値の許容値拡大**: スキル・プラグインのフロントマターで `yes`/`no`/`on`/`off`/`1`/`0`（大文字小文字不問）を `true`/`false` と同様に受理
- **エージェント名の `:` 禁止**: プラグイン名前空間予約のため、`:` を含むエージェント名を拒否
- **`/deep-research` の手動起動限定化**: Claude が自律的に起動しなくなった
- **auto モード・プランモードの権限判定改善**: dangerous-rm / バックグラウンド `&` / 疑わしい Windows パスのチェックが権限ダイアログではなく auto モード分類器で裁定されるように。プランモード + auto でも静的解析で読み取り専用と証明できない Bash コマンドを分類器が判定
- **エージェントフロントマターフックの信頼要件**: エージェントファイルのあるフォルダ自体のワークスペース信頼承認が必要に（未信頼フォルダからのフック実行を修正）
- そのほか: `/ultrareview` が「review my auth changes」のような記述的引数を受理、非対話セッションの `/code-review ultra` がクラウドレビューを正しく起動、server-managed settings の無害なトグルが承認プロンプトを出さないよう変更

## v2.1.217 (2026-07-21)

- **サブエージェント並列実行の上限**: 1メッセージからの無制限なバックグラウンドエージェント fan-out を防ぐため、同時実行サブエージェント数を上限化（デフォルト 20、`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` で変更可）
- **ネストサブエージェントのデフォルト無効化**: サブエージェントは既定でネスト起動しなくなった。`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` で深いネストを許可
- **`--max-budget-usd` がバックグラウンドサブエージェントにも適用**: 上限到達後は新規スポーン拒否＋実行中のバックグラウンドエージェントを停止
- **絵文字ショートコード補完**: プロンプト入力で `:heart:` → ❤️ の補完。`:hea` でサジェスト表示。`emojiCompletionEnabled` 設定で無効化可
- **トランスクリプト書き込み失敗の警告**: ディスクフル等で書き込みが失敗している場合や、継承環境変数によりセッション保存が無効の場合に、黙って失われる代わりに警告を表示
- **MCP 出力のメモリリーク修正**: 切り詰められた MCP ツール出力が、切り詰め前の全結果をセッション終了まで保持していた問題を修正
- **バックグラウンドセッション隔離の symlink 対応**: 作業ディレクトリの symlink を正規化せず、セッションがワークスペース外に逸脱できた問題を修正
- **フッター PR バッジのハイパーリンク改善**: ssh/tmux 等で端末サポートを検出できない場合もクリック可能に。`FORCE_HYPERLINK=0` でオプトアウト
- そのほか: Windows 自動更新失敗で `claude.exe` が消える問題の修正、Bedrock の Opus 4.8 で auto-compact が発動しない問題の修正、ログイン失効警告を5日前→3日前に変更、`CLAUDE.md`/`SKILL.md` の paths フロントマターの brace 展開を予算制限化（起動時 OOM/ストール対策）

## v2.1.216 (2026-07-20)

バグ修正中心のリリース。

- **`sandbox.filesystem.disabled` 追加**: ファイルシステム分離のみスキップし、ネットワーク egress 制御は維持する設定
- **セキュリティ修正（worktree/symlink 関連）**: worktree 隔離サブエージェントが `git -C` / `--git-dir` / `GIT_DIR` 環境変数で共有チェックアウトへ git 操作をリダイレクトできた問題、`.claude` の symlink 経由でワークフロー保存・スケジュールタスク書き込みがプロジェクト外へ逸脱できた問題を修正。`/rewind` も symlink / ハードリンク越しの復元・削除をしなくなり、スキップしたパス数を報告
- **長時間セッションの高速化**: メッセージ正規化コストがターン数に対し二次関数的に増大し、数秒のストールや resume 遅延を起こしていた問題を修正
- **auto mode の OAuth 失効対応**: トークン失効・ローテーション後に分類器が HTTP 401 でコマンドを拒否し続ける問題を修正
- **改善多数**: `/fork` 確認を1行化（セッション名と `claude attach` id 表示）、`/ultrareview` の diff 超過エラーに設定上限・実測サイズ・上位ファイルを表示、`/context` がコンテキストウィンドウ超過を明示警告、バックグラウンドセッションの `/mcp`・`/install-github-app` が「要入力」リクエストを agent view に退避

## v2.1.215 (2026-07-19)

- **`/verify` と `/code-review` の自動実行を廃止**: Claude が自己判断でこれらのスキルを起動しなくなった。実行したい場合はユーザーが `/verify` / `/code-review` を明示的に呼び出す

## v2.1.214 (2026-07-18)

permission チェックの堅牢化を中心としたリリース（v2.1.213 は欠番）。

- **EndConversation ツール追加**: 高度に虐待的なユーザーや jailbreak 試行に対して Claude がセッションを終了できる（claude.ai では 2025 年から提供済みの機能）
- **permission チェック堅牢化（多数）**: 単一セグメント `dir/**` 許可ルール（`Edit(src/**)` 等）がツリー内任意の同名ディレクトリに誤マッチする問題、Windows PowerShell 5.1 でのバイパス、ファイルディスクリプタリダイレクト形式、10,000 文字超コマンド、zsh 変数添字、`help`/`man` 経由の危険オプション実行などを修正（fail closed 化）
- **破壊的変更: hook `if:` 条件の単一セグメント `dir/**` は `<cwd>/dir` のみにマッチ**: 任意深度でマッチさせたい場合は `**/dir/**` と書く。`deny`/`ask` permission ルールは従来通り任意深度マッチを維持
- **`docker` コマンドのデーモンリダイレクトフラグに許可プロンプト追加**: `--url` / `--connection` / `--identity` / Podman リモートモードが対象（従来はプロンプトなしで実行）
- **長時間ツール呼び出しに定期進捗ハートビート追加**: 従来は無音になっていた
- **`CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH` 追加**: OpenTelemetry コンテンツ属性の 60KB 切り詰め上限を設定可能。OTel ログイベントに `message.uuid` / `client_request_id` / `tool_source` 属性も追加
- **`subagentStatusLine` ペイロードに reasoning effort 追加**: カスタムエージェント行でモデルと effort を表示可能
- **メモリファイルのフロントマターに ISO `modified` タイムスタンプ追加**
- **SessionStart hook の source に `"fork"` 追加**: フォークとして開始したセッションは `"resume"` ではなく `"fork"` を報告
- **`file` コマンドの `-m`/`--magic-file`・`-f`/`--files-from` が要許可に変更**（従来は読み取り専用として自動許可）
- バックグラウンドセッション関連の修正多数: 放置セッションによるデーモン常駐、`claude rm` で削除不能、非 git フォルダ発のセッション削除不能、停止セッション再開失敗など
- Windows PowerShell ツールの修正多数: stdin 待ちハング、非 UTF-8 データの UnicodeDecodeError、`>` リダイレクトの UTF-16LE 問題など
- スケジュールタスクが自身の設定プロンプトを untrusted input として拒否する問題を修正
- `--settings` 経由で有効化したプラグインが読み込まれない問題を修正（v2.1.181 からのリグレッション）
- hook の exit code 2 が stdout JSON のスキーマ検証失敗時にブロックしない問題を修正

---

## v2.1.212 (2026-07-17)

- **`/fork` の挙動変更**: 会話を新しいバックグラウンドセッション（`claude agents` に独立した行として表示）にコピーするようになった。従来のセッション内サブエージェント起動は `/subtask` に改名
- **`claude auto-mode reset` 追加**: auto-mode 設定をデフォルトに復元（確認プロンプトあり、`--yes` でスキップ）
- **WebSearch のセッション上限追加**: デフォルト 200 回。`CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` で調整可（暴走検索ループ対策）
- **サブエージェントスポーンのセッション上限追加**: デフォルト 200。`CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION` で上書き可、`/clear` でリセット（暴走委譲ループ対策）
- **MCP ツール呼び出しの自動バックグラウンド化**: 2 分超で自動的にバックグラウンドへ移行しセッションを維持。閾値変更・無効化は `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS`
- **agent view での `/resume`**: 過去セッション（一覧から削除済み含む）のピッカーを開き、選択したセッションをバックグラウンドセッションとして再開
- **セキュリティ修正**: plan mode がファイル変更系 Bash コマンド（`touch`, `rm` 等）を許可プロンプトなしで自動実行する問題、worktree 作成が `.claude/worktrees` のコミット済み symlink を辿ってリポジトリ外にファイルを作成し得る問題を修正
- print/SDK モードで Bash 実行中の SIGTERM がプロセスツリーを孤児化する問題を修正（ターン中断・ツリー kill・exit 143）
- hook の `continue:false` による停止がツール失敗・ストリーム途中完了時に落ちる問題、hook インフラエラーがユーザー拒否と誤報告される問題を修正

---

## v2.1.211 (2026-07-15)

- **`--forward-subagent-text` フラグ / `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT` 環境変数を追加**: stream-json 出力にサブエージェントのテキスト・思考を含められる
- **セキュリティ強化**: チャットチャネルに中継される permission プレビューで bidi 制御文字・ゼロ幅文字・紛らわしい引用符を無害化（ツール入力が承認メッセージを視覚的に偽装できる問題を修正）
- **auto mode が PreToolUse hook の `ask` 決定を上書きしないよう修正**: 非サンドボックス Bash に対する hook の `ask` は必ずプロンプト表示に floor される
- **"always allow" permission ルールの保存先をリポジトリルートに変更**: worktree で許可した承認がセッション・worktree 間で持続するように
- **prompt-caching 回帰を修正**: Bedrock / Vertex / Mantle / Foundry で末尾の system context ブロックが毎リクエスト新規入力トークンとして課金されていた
- **サブエージェントの model override が resume / フォローアップで親モデルに戻る問題を修正**
- **nested `.claude/rules/*.md` が設定ソースからプロジェクト設定を除外していても読み込まれる問題を修正**
- **バックグラウンドエージェントの結果報告を改善**: 実行中エージェントは状態を報告し実際の完了を待つ（結果の捏造をしない）。ユーザーが kill したエージェントの自動再スポーン・古いプロンプト再実行も修正
- **`/loop` 使用後にセッションが `/resume` から消える問題、`/clear` がコストカウンターをリセットしない問題を修正**
- **`/usage-credits` が組織管理者へのリクエスト送信前に確認を求めるように変更**
- 整数系環境変数（タイムアウト・トークンバジェット・リトライ回数）が `1e6` / `64_000` 形式の表記を受理

---

## v2.1.210 (2026-07-14)

バグ修正中心のリリース。主要な変更のみ記載。

- **permission ルールの起動時警告追加**: `Write(path)` / `NotebookEdit(path)` / `Glob(path)` 形式のルールに対して警告を表示。代わりに `Edit(path)` / `Read(path)` を使用すべき
- **auto mode 改善**: permission 分類器が外部セッションで Sonnet 5 をデフォルトに（セッション初回リクエストで検証しセッション中ピン留め）
- **安全性強化**: Agent ツールをサブエージェントが読んだコンテンツ経由の間接プロンプトインジェクションに対して強化。`ultracode` キーワードが webhook ペイロード・転送 PR コメント等の非人間入力で発火する問題も修正
- **`isolation: 'worktree'` サブエージェントの隔離不備を修正**: main リポジトリの checkout に対して git 変更コマンドを実行できてしまう問題
- **hook コールバックのタイムアウト誤報告を修正**: ユーザー拒否として誤ってモデルに伝わり、無人セッションが停止・待機してしまう問題
- **skills / commands の未マッチ `$1`/`$2` プレースホルダを verbatim 保持に修正**（従来は黙って除去）
- **MEMORY.md インデックスが読み取り上限を超過する書き込みは silent truncation ではなく明示エラーに**
- UI 改善: 折りたたみツールサマリ行に経過時間カウンター、agents フッターに入力待ちエージェント数表示、スクリーンリーダーモードで Shift+Tab の permission mode 変更を読み上げ
- タイムアウトで自動バックグラウンド化されたコマンドのツールメッセージを改善（ハングと明示的バックグラウンド要求を区別可能に）

---

## v2.1.209 (2026-07-14)

- `claude agents` バックグラウンドセッションで `/model` 等のダイアログがブロックされる問題を修正（v2.1.208 の過剰なガードのリバート）

---

## v2.1.208 (2026-07-14)

- **スクリーンリーダーモード追加**: プレーンテキスト描画へのオプトイン。`claude --ax-screen-reader` / 環境変数 `CLAUDE_AX_SCREEN_READER=1` / settings `"axScreenReader": true` のいずれかで有効化
- **`vimInsertModeRemaps` 設定追加**: vim モードのインサートモードで `jj` → Escape のような 2 キーシーケンスをマップ可能
- **`CLAUDE_CODE_PROCESS_WRAPPER` 追加**: agent view とバックグラウンドサービスの全自己スポーンを、指定した企業ランチャー（ラッパー実行ファイル）経由で起動
- **フルスクリーンモードのマルチセレクトメニューと「Other」入力行がマウスクリック対応**
- **安全性強化**: `$(…)`・バッククォート・`<(…)` を含むコマンド内の破壊的削除（`rm -rf ~` 等）が、プレーン形式と同様に `--dangerously-skip-permissions` / auto mode でも確認プロンプトを出すように
- **長時間セッションのメモリリーク・肥大化修正多数**: MCP stdio stderr のサーバー毎最大 64MB 蓄積、LSP ドキュメントの無期限保持（LRU 50 件上限に）、ファイル編集キャッシュを 16MB に制限、編集多用セッションのトランスクリプトを最大 79 分の 1 に削減＋チェックポイントのディスク使用量を有界化
- **パフォーマンス改善**: permission deny/ask ルール多数時のターン毎複数秒遅延を解消（ルールマッチャーのコンパイル1回化＋キャッシュ）、print/SDK セッションのツールプール組み立てキャッシュで MCP ツール多数時に最大 7 倍高速化、タスクリスト更新時の全 UI 再描画を廃止
- **`/release-notes` の「Show all」が changelog 全文を以後の全リクエストのコンテキストに注入していた問題を修正**
- 完了したバックグラウンドエージェントがクリーンアップまで `/tasks` に残るように（従来は完了直後に消滅）
- `/usage` がレート制限時にエラー画面ではなく最終既知の使用量バーを「as of」注記付きで表示
- fast mode 対応モデルへ切り替え戻した際、settings で有効なら fast mode が自動復元されるように修正
- CLI 自動更新後にコンテキストウィンドウ表示が 200k にリセットされ、長コンテキストセッション再開時に誤って「100% 使用」となる問題を修正
- `CLAUDE_CODE_MAX_OUTPUT_TOKENS` 等が科学記法値（`1e6`）の仮数部だけを読む問題を修正
- Bedrock: `sso_region` が Bedrock リージョンと異なる AWS SSO プロファイルの認証失敗を修正（v2.1.207 リグレッション）
- agent view: Ctrl+X がリネーム済みブランチの worktree を削除可能に。未 push コミットは破壊しない

---

## v2.1.207 (2026-07-11)

- **Bedrock / Vertex AI / Foundry で auto mode が GA**: `CLAUDE_CODE_ENABLE_AUTO_MODE` のオプトイン不要に。無効化は settings の `disableAutoMode`
- **Bedrock / Vertex / Claude Platform on AWS のデフォルトモデルが Claude Opus 4.8 に変更**
- **auto mode 設定の読み取り元変更**: リポジトリ内の `.claude/settings.local.json` の `autoMode` を読まなくなった。`~/.claude/settings.json` を使用すること
- **セキュリティ: plugin の shell-injection 修正**: hooks / monitors / MCP headersHelper のシェル形式コマンド中の `${user_config.*}` を拒否。hooks は exec 形式（`args` 配列）または `$CLAUDE_PLUGIN_OPTION_<KEY>` を使用
- **セキュリティ: plugin オプション値（`pluginConfigs`）をプロジェクトレベル `.claude/settings.json` から読まなくなった**: user / `--settings` / managed settings のみ有効
- **セキュリティ: 非対話実行（`claude -p` / SDK）でリモート managed settings がセキュリティ同意ダイアログなしに consented 記録される問題を修正**
- **`/usage-credits` の金額入力検証強化**: 不正値は digits へのサイレント切り詰めではなくエラーに。$1,000 超は typed confirmation 必須
- 長いリスト/テーブル/コードブロックのストリーミング中に端末がフリーズ・キー入力が遅延する問題を修正

---

## v2.1.206 (2026-07-09)

- **`EnterWorktree` の外部 worktree 確認**: プロジェクトの `.claude/worktrees/` 外の git worktree に入る前に確認を求めるように
- **`/doctor` に CLAUDE.md トリミング提案チェック追加**: コードベースから導出可能な内容の削減を提案
- **`/commit-push-pr` が push remote への `git push` を自動許可**: origin に加え `remote.pushDefault`（remote が 1 つだけならその remote）も対象
- **`/cd` にディレクトリパス補完追加**（`/add-dir` と同じ挙動）
- **Gateway: `/login` が Anthropic 運営のパブリックゲートウェイエンドポイントに対応**
- **バックグラウンドエージェントの即時バージョン更新**: Claude Code 更新直後にバックグラウンドで新バージョンへアップグレード。アタッチ時の遅い stale-session アップグレード待ちを解消

---

## v2.1.205 (2026-07-08)

- **`/doctor` が統合セットアップチェックアップに**: 問題の診断だけでなく修正まで実行。`/checkup` はそのエイリアス
- **auto mode 強化**: セッショントランスクリプトファイルの改ざんをブロックするルールを追加。コンテキストから解決できない変数への `rm -rf` は実行前に確認
- **バックグラウンドタスク通知の偽装承認対策**: 通知に「人間の入力は発生していない」ことを明記し、トランスクリプト内に捏造された承認が実行されるのを防止
- **agent view 改善**: 行に色付き状態ワード+分類器生成の見出しを表示。既存 PR への編集/マージ/コメント/push もリンク表示。blocked セッションの peek は質問内容を先頭表示
- **「Claude Browser」MCP サーバー名を予約**: 「Claude Preview」に加えて予約（Claude Desktop ペインのリネームに先行）。ユーザー設定 MCP は両名称で登録不可に
- auto-update のダウンロードをディスクストリーミング化（ピークメモリ約 400MB 削減）

---

## v2.1.204 (2026-07-08)

- バグ修正のみ（headless セッションの SessionStart hook 中に hook イベントがストリーミングされずリモートワーカーが idle-reap される問題の修正）

---

## v2.1.203 (2026-07-07)

- **ログイン期限切れ間近の警告追加**: バックグラウンドセッションが中断される前に再認証可能に
- **manual permission mode の常時表示**: フッターにグレーの ⏸ バッジを表示
- **MCP `roots/list` にセッションの追加作業ディレクトリを反映**: セット変更時は `notifications/roots/list_changed` を送出
- **サブエージェントの再委譲抑制**: タスク全体を別のサブエージェントに丸投げしにくくなった
- **左矢印でバックグラウンドタスク / diff / workflow 詳細ビューを閉じなくなった**: Esc を使用
- バックグラウンドエージェントの安定性修正多数（macOS の 15〜20 秒 stall、stale な `PATH` 継承、shell-exported `ANTHROPIC_BASE_URL` のドロップによる 401 等）
- バイナリサイズ約 7MB・起動メモリ約 7MB 削減、ストリーミング中の応答性改善

---

## v2.1.202 (2026-07-06)

- **「Dynamic workflow size」設定を `/config` に追加**: 動的ワークフローのエージェント数規模（small / medium / large）の目安を指定。強制上限ではなく助言的ガイドライン
- **workflow の OTel 属性追加**: `workflow.run_id` / `workflow.name` をワークフロー生成エージェントのテレメトリに付与。OTel データからワークフロー実行を再構成可能に
- **`/review <pr>` が高速シングルパスレビューに回帰**: 多エージェントレビューは `/code-review <level> <pr#>` を使用
- ロード済みスキルの再呼び出しで指示が重複追加される問題、多数の git worktree があるリポジトリでの resume が数分かかる問題を修正

---

## v2.1.201 (2026-07-03)

- **Claude Sonnet 5 セッションがハーネスリマインダーに mid-conversation system role を使用しなくなった**

---

## v2.1.200 (2026-07-03)

- **`AskUserQuestion` の自動継続を廃止**: ダイアログがデフォルトで auto-continue しなくなった。idle タイムアウトは `/config` でオプトイン
- **「default」パーミッションモードを「Manual」に改名**: CLI / `--help` / VS Code / JetBrains 全体で変更。`--permission-mode manual` / `"defaultMode": "manual"` を `default` と併せて受理
- バックグラウンドエージェントデーモンの安定性修正多数（stale `daemon.lock`、古いビルドによるデーモン乗っ取り防止、roster 破損等）
- スクリーンリーダー出力の改善（装飾グリフ非表示、ネストテーブルの読み上げ改善）

---

## v2.1.199 (2026-07-02)

- **スタック型スキル呼び出し対応**: `/skill-a /skill-b do XYZ` で先頭のスキルを最大 5 個まで全てロード（従来は最初の 1 個のみ）
- **一時的レート制限（429）の自動リトライ**: 使用量上限と無関係なサーバー側 429 をサブスクライバー向けにバックオフ付きで自動リトライ
- **`CLAUDE_CODE_RETRY_WATCHDOG` 強化**: 非容量系一時エラーのデフォルトリトライ回数を 300 に引き上げ、`CLAUDE_CODE_MAX_RETRIES` の上限 15 を撤廃
- **サブエージェントの部分結果保全**: レート制限/サーバーエラーで切られたサブエージェントが部分成果を親に返すように。API エラーを成功結果として誤報告する問題も修正
- SSL 証明書エラーはリトライを浪費せず即座に対処ヒント付きで失敗、mid-stream エラー時は部分レスポンスを保持

---

## v2.1.198 (2026-07-01)

- **サブエージェントがデフォルトでバックグラウンド実行に**: 実行中も Claude が作業を継続し、完了時に通知される（段階的ロールアウトが全面化）
- **Claude in Chrome が GA（一般提供）に**
- **`/dataviz` スキル追加**: チャート・ダッシュボード設計ガイダンス+実行可能なカラーパレットバリデータ
- **`claude agents` のバックグラウンドエージェント通知**: 入力待ち/完了時に `Notification` hook（`agent_needs_input` / `agent_completed`）が発火
- **worktree 完了時の自動 commit → push → draft PR**: `claude agents` から起動したバックグラウンドエージェントがコード作業完了時に確認なしで実行
- **Explore エージェントがメインセッションのモデルを継承**（opus 上限。従来は haiku 固定）
- **サブエージェント・コンパクションが extended thinking 設定を継承**: 委譲タスクの品質向上
- **`/agents` ウィザード削除**: Claude に依頼するか `.claude/agents/` を直接編集
- **Gateway: Claude Platform on AWS（anthropicAws）をアップストリームプロバイダに追加**: model-not-found でフェイルオーバーチェーンを進行
- サブエージェントは起動元エージェントのメッセージを通常のタスク指示として扱う（ただしユーザーの承認としては決して扱わない）

---

## v2.1.197 (2026-06-30)

- **Claude Sonnet 5 登場**: Claude Code のデフォルトモデルに。ネイティブ 1M トークンコンテキストウィンドウ、2026-08-31 までのプロモ価格 $2/$10 per Mtok

---

## v2.1.196 (2026-06-29)

- **組織デフォルトモデル対応**: 管理者が org console で設定。自分で未選択の場合 `/model` に「Org default」（または「Role default」）と表示
- **セキュリティ: MCP 自己承認の穴を修正**: `claude mcp list`/`get` が、コミット済み `.claude/settings.json` でリポジトリが自己承認した `.mcp.json` サーバーを起動しなくなった。未信頼ワークスペースは `⏸ Pending approval` 表示
- **ストリーミング idle watchdog が全プロバイダでデフォルト有効に**: 5 分間イベントのないストリームを中断してリトライ。`CLAUDE_ENABLE_STREAM_WATCHDOG=0` で無効化
- **Remote Control を非 Anthropic ホストの `ANTHROPIC_BASE_URL` で無効化**（Bedrock/Vertex/Foundry と同じ扱い）
- **セッション開始時に可読なデフォルト名を付与**: 識別・メッセージ送信が容易に
- `/code-review` のクリーンアップファインダー 5 個を統合しトークン約 25% 削減、バックグラウンドセッションの耐久性改善（プロセスの停止/再起動/更新をまたいで長時間コマンドが生存。Windows 含む）

---

## v2.1.195 (2026-06-26)

- **破壊的変更: hook マッチャーのハイフン付き識別子が完全一致に**: `code-reviewer` や `mcp__brave-search` 等が substring マッチしなくなった。ハイフン付き MCP サーバーの全ツールにマッチさせるには `mcp__brave-search__.*` を使用
- **`CLAUDE_CODE_DISABLE_MOUSE_CLICKS` 追加**: fullscreen でクリック/ドラッグ/ホバーを無効化しつつホイールスクロールは維持
- **音声入力の auto-submit がスペースなし言語（日本語・中国語・タイ語）で発火しない問題を修正**
- プロジェクト `.claude/settings.json` のみで有効化された外部プラグインに、全ロードパスで明示的なインストール同意を必須化

---

## v2.1.193 (2026-06-25)

- **`autoMode.classifyAllShell` 設定追加**: 全 Bash/PowerShell コマンドを auto-mode classifier に通す（従来は任意コード実行パターンのみ）
- **auto mode の拒否理由表示**: トランスクリプト、拒否トースト、`/permissions` の recent denials に理由を表示
- **OTEL `claude_code.assistant_response` ログイベント追加（プライバシー注意）**: モデルの応答テキストを含む。`OTEL_LOG_ASSISTANT_RESPONSES=1` で有効。未設定時は `OTEL_LOG_USER_PROMPTS` に従うため、プロンプト内容を既にログしているデプロイはアップグレード後に応答内容も受信し始める — prompts-only を維持するには `OTEL_LOG_ASSISTANT_RESPONSES=0` を設定
- **bash モード（`!`）にライブファイルパス補完追加**
- アイドルバックグラウンドシェルのメモリ圧力自動回収（`CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP=1` で無効化）
- MCP `headersHelper` 認証が 401/403 時に自動再実行・再接続、plugin の marketplace `renames` マップ自動追従

---

## v2.1.191 (2026-06-24)

- **`/rewind` が `/clear` 実行前からの会話再開に対応**
- **サンドボックスのネットワーク許可をセッション内で記憶**: 一度「Yes」で許可したホストは接続のたびに再プロンプトされない
- **カンマ区切り hook マッチャー（例 `"Bash,PowerShell"`）が silently 発火しない問題を修正**
- MCP 信頼性改善: capability discovery と OAuth の一時エラーリトライ、headless 環境では browser popup をスキップして URL ペーストプロンプトへ
- ストリーミング中の CPU 使用率約 37% 削減（テキスト更新を 100ms に coalesce）

---

## v2.1.190 (2026-06-24)

- バグ修正・信頼性改善のみ（公式 changelog 記載）。ユーザー向け新機能なし

---

## v2.1.187 (2026-06-23)

- **`sandbox.credentials` 設定追加**: サンドボックス内コマンドによる認証情報ファイル・シークレット環境変数の読み取りをブロック
- **組織設定のモデル制限に対応**: model picker / `--model` / `/model` / `ANTHROPIC_MODEL` に適用され、制限モデル選択時は「restricted by your organization's settings」と表示
- **fullscreen の選択メニュー（パーミッションプロンプト、`/model`、`/config` 等）でマウスクリック対応**
- **リモート MCP ツール呼び出しの 5 分アイドルタイムアウト**: 無応答ハングを中断（`CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` で変更可）
- サブエージェント深度トラッキング修正: 再開時に元の spawn 深度を復元、fork したサブエージェントも深度上限にカウント
- `/install-github-app` で GitHub App のみのインストールが可能に（workflow/secret 手順のスキップ可）

---

## v2.1.186 (2026-06-22)

- **`claude mcp login <name>` / `claude mcp logout <name>` 追加**: 対話的 `/mcp` メニューを開かず CLI から MCP サーバー認証。SSH 越しの完了用に `--no-browser` の stdin リダイレクト対応
- **`!` bash コマンドの出力に Claude が自動応答するように**: 従来のコンテキスト追加のみの挙動に戻すには settings.json で `"respondToBashCommands": false`
- **バックグラウンドサブエージェントのパーミッションプロンプトをメインセッションに表示**: 自動 deny から変更。どのエージェントの要求かを表示し、Esc でそのツールのみ拒否
- **`/review <pr>` が `/code-review medium` と同じレビューエンジンを使用するように**
- **`CLAUDE_CODE_MAX_RETRIES` に上限 15 を設定**: 無人セッションは `CLAUDE_CODE_RETRY_WATCHDOG` を使用
- **`Agent(type)` deny ルール / `Agent(x,y)` 許可型制限が named サブエージェント spawn にも強制されるように修正**
- `teammateMode: "iterm2"` 設定追加、`/plugin` Installed タブに Skills セクション追加、skill frontmatter キーが kebab-case / snake_case / camelCase を受理

---

## v2.1.185 (2026-06-20)

- ストリーム停滞ヒントの文言・タイミング調整のみ（「Waiting for API response · will retry in …」表記に変更、発火を 10 秒→20 秒に）

---

## v2.1.183 (2026-06-19)

- **auto mode の破壊的 git コマンドブロック**: ユーザーがローカル作業の破棄を求めていない場合の `git reset --hard` / `git checkout -- .` / `git clean -fd` / `git stash drop` をブロック。このセッションでエージェントが作成していないコミットへの `git commit --amend`、特定スタックの指示がない `terraform destroy` / `pulumi destroy` / `cdk destroy` もブロック
- **非推奨/自動更新モデルの警告追加**: print モード（`-p`）では stderr に表示。agent frontmatter で設定されたモデルもカバー
- **`attribution.sessionUrl` 設定追加**: web / Remote Control セッションのコミット・PR から claude.ai セッションリンクを除外可能
- **`/config --help` 追加**: `/config key=value` で使える shorthand キーの一覧表示
- WebSearch がサブエージェントで空結果を返す問題を修正、`/config` トグルは Esc で保存して閉じる挙動に変更

---

## v2.1.181 (2026-06-17)

- **`/config key=value` 構文追加**: 任意の設定をプロンプトから直接変更可能（例 `/config thinking=false`）。インタラクティブ / `-p` / Remote Control で動作
- **`sandbox.allowAppleEvents` 設定追加**: サンドボックス内コマンドの Apple Events 送信をオプトイン許可（macOS）
- **`CLAUDE_CLIENT_PRESENCE_FILE` 環境変数追加**: マーカーファイルを指定すると在席中のモバイルプッシュ通知を抑制
- **バンドル Bun ランタイムを 1.4 にアップグレード**
- **foreground サブエージェントにも 5 階層の深度制限を適用**: 無制限のネストチェーン spawn を修正
- fullscreen の URL オープンが Cmd+クリック（macOS）/ Ctrl+クリック必須に、長い段落のストリーミングを行単位表示に改善
- 起動リグレッション修正（2.1.169 起因の約 120ms）、ネットワークドライブ/クラウド同期フォルダでの Write/Edit 0 バイト生成修正

---

## v2.1.179 (2026-06-16)

- バグ修正・信頼性改善のみ（mid-stream 切断時の部分レスポンス保持、WSL2 のマウスホイールスクロール修正等）

---

## v2.1.178 (2026-06-15)

- **agent teams 刷新: `TeamCreate` / `TeamDelete` ツールを削除**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 設定下では全セッションが暗黙の 1 チームを持つように。Agent ツールの `name` パラメータで直接チームメイトを spawn（セットアップ手順不要）。`team_name` パラメータは受理されるが無視される
- **`Tool(param:value)` パーミッションルール構文追加**: ツールの入力パラメータにマッチ（`*` ワイルドカード対応）。例: `Agent(model:opus)` で Opus サブエージェントをブロック
- **ネストされた `.claude/skills` ディレクトリのロード対応**: 配下ファイルの作業時にロードされ、名前衝突時は `<dir>:<name>` として両方利用可能
- **ネストされた `.claude/` の優先順位**: agent / workflow / output-style は名前衝突時に作業ディレクトリに最も近いものが優先
- **auto mode がサブエージェント spawn を起動前に classifier で評価**: サブエージェントがレビューなしでブロック対象アクションを要求できるギャップを閉鎖
- workflow プロンプトキーワードは「run a workflow」等の明示的フレーズのみでトリガーするよう変更

---

## v2.1.176 (2026-06-12)

- **セッションタイトルが会話の言語で生成されるように**（`language` 設定で特定言語に固定可）
- **`footerLinksRegexes` 設定追加**: フッター行に正規表現マッチのリンクバッジを表示（user / managed settings で設定）
- **`availableModels` 強制の強化**: `ANTHROPIC_DEFAULT_*_MODEL` 環境変数経由でブロック済みモデルにリダイレクト不可に。`/fast` も allowlist 外モデルへの切替を拒否
- Bedrock: `awsCredentialExport` の認証情報を固定 1 時間ではなく `Expiration` までキャッシュ
- hook `if` 条件の Read/Edit/Write パスパターン（`Edit(src/**)`、`Read(.env)` 等）が正しくマッチするよう修正

---

## v2.1.175 (2026-06-12)

- **`enforceAvailableModels` managed setting 追加**: 有効時、`availableModels` allowlist が Default モデルも制約（不許可モデルに解決される Default は最初の許可モデルへフォールバック）。user / project 設定で managed の `availableModels` リストを広げることも不可に

---

## v2.1.174 (2026-06-12)

- **`wheelScrollAccelerationEnabled` 設定追加**: fullscreen のマウスホイールスクロール加速を無効化可能
- `/model` ピッカーが Default の解決先モデルファミリを独立行で表示するよう修正
- [VSCode] `/usage` に使用量アトリビューション追加: cache miss、long context、サブエージェント、skill/agent/plugin/MCP 別の直近 24h/7d 内訳

---

## v2.1.173 (2026-06-11)

- バグ修正のみ（Fable 5 のモデル名 `[1m]` サフィックス正規化 — Fable 5 は 1M コンテキストが標準のため自動除去 — 等）

---

## v2.1.172 (2026-06-10)

- **サブエージェントのネスト対応**: サブエージェントが自身のサブエージェントを spawn 可能に（最大 5 階層）
- **Bedrock が `~/.aws` 設定ファイルから AWS リージョンを読むように**: `AWS_REGION` 未設定時。AWS SDK と同じ優先順位で、`/status` にリージョンの出所を表示
- **`/plugin` のマーケットプレイス閲覧に検索バー追加**
- 1M コンテキスト使用中にクレジット切れでセッションが恒久スタックする問題を修正（標準コンテキスト上限まで自動コンパクト）
- `availableModels` 制限のサブエージェントモデルオーバーライド / dispatch ピッカー / advisor への適用漏れを修正
- `WebFetch(domain:*.example.com)` ワイルドカードがサブドメインにマッチしない問題、途中ワイルドカードのファイル権限ルール拒否を修正
- 長い会話・並列サブエージェント時のパフォーマンスとアイドル CPU 使用率を改善

---

## v2.1.170 (2026-06-09)

- **Claude Fable 5 導入**: Mythos クラスのモデルを一般利用向けに安全化したもので、一般提供されたモデルとして過去最高の能力。v2.1.170 以降で利用可能
- VS Code 統合ターミナル等（Claude Code 環境変数を継承したシェル）からの起動でトランスクリプトが保存されず `--resume` に出ない問題を修正

---

## v2.1.169 (2026-06-08)

- **`--safe-mode` フラグ / `CLAUDE_CODE_SAFE_MODE` 追加**: 全カスタマイズ（CLAUDE.md、plugins、skills、hooks、MCP サーバー）を無効化して起動。トラブルシューティング用
- **`/cd` コマンド追加**: プロンプトキャッシュを壊さずにセッションの作業ディレクトリを移動
- **`disableBundledSkills` 設定 / `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` 追加**: バンドルスキル・ワークフロー・組み込みスラッシュコマンドをモデルから隠す
- **Self-hosted runner: `post-session` ライフサイクル hook 追加**: セッション終了後・ワークスペース削除前に実行（未コミット作業のスナップショットやログエクスポート用）。子プロセスの SIGTERM→SIGKILL 猶予も設定可能に（デフォルト 5 秒）
- **セキュリティ: 未信頼プロジェクト設定が信頼確認なしに OTEL クライアント証明書パスを設定できる問題を修正**
- enterprise managed MCP ポリシー（`allowedMcpServers`/`deniedMcpServers`）の強制漏れ（reconnect / IDE 設定 / インストール直後の `--mcp-config` 等）を修正
- `claude agents --json` に `--all` オプションと `id` / `state` フィールド追加、「CLAUDE.md is too long」警告閾値がモデルのコンテキストウィンドウに応じてスケール

---

## v2.1.168 (2026-06-06)

- バグ修正・信頼性改善のみ（公式 changelog 記載）。ユーザー向け新機能なし

---

## v2.1.167 (2026-06-06)

- バグ修正・信頼性改善のみ（公式 changelog 記載）。ユーザー向け新機能なし

---

## v2.1.166 (2026-06-06)

- **`fallbackModel` 設定の追加**: プライマリモデルが overloaded / unavailable のときに順番に試行する fallback モデルを最大 3 つまで設定可能。`--fallback-model` フラグもインタラクティブセッションに適用されるようになった
- **deny ルールのツール名位置で glob パターンをサポート**: `permissions.deny` のツール名位置で `"*"` 等の glob 指定が可能（`"*"` は全ツールを deny）。allow ルール側は MCP 以外の glob を拒否、deny ルール内の未知ツール名は起動時に警告
- **クロスセッション `SendMessage` のセキュリティ強化**: 他の Claude セッションから `SendMessage` で中継されたメッセージはユーザー権限を持たないものとして扱われる。受信側は中継経由の権限リクエストを拒否し、auto mode は該当をブロック
- **`MAX_THINKING_TOKENS=0` / `--thinking disabled` / per-model thinking トグルが thinking 既定モデルを無効化**: Claude API 経由でデフォルト thinking のモデルでも上記指定で thinking を無効化可能に（3P プロバイダの挙動は変更なし）
- **fallback モデルでのターン自動再試行**: API が予期せぬ非リトライエラーを返した場合、ターンを fallback モデルで一度だけ自動再試行。auth / rate-limit / request-size / transport エラーは即時返却
- **`claude update` のターゲットバージョン事前告知**: ダウンロード開始前に更新先バージョンを表示。サイレント実行が解消
- **`claude agents`: URL によるセッションフィルタ**: 一覧で URL を入力すると、最初のプロンプトにその URL を含むセッションに絞り込み
- **画像処理エラー修正**: 処理不能な画像を含むセッションでの「image could not be processed」連続エラーと余分なトークン消費を修正
- **起動時 worker registration 中断時のリモートセッション復旧**: バックエンド一時切断中に worker registration が失敗するとリモートセッションが永久に詰まる問題を修正
- **JetBrains IDE ターミナル (2026.1+) のチラつき解消**: synchronized output を有効化（IntelliJ / PyCharm / WebStorm 等）
- **Kitty keyboard protocol 上での Shift+非 ASCII キー修正**: WezTerm / Ghostty / kitty で `Shift+ä → Ä` 等が drop される問題を修正
- **Windows PowerShell コマンド検証のハング修正**: 子プロセスが出力パイプを保持したまま親プロセスが kill されると検証が時間予算を超えてハングする問題を修正
- **macOS の orphan `claude --bg-pty-host` プロセス修正**: デーモン死亡時に 100% CPU で回り続ける問題を修正
- **音声モード `/login` 修正**: `/voice` トグル後に stale な auth check を `/login` 無しでクリア
- **Managed settings の堅牢化**: invalid なエントリ一つで残りの valid policy 全てが silently 無効化される問題を修正
- **`allowedMcpServers` / `deniedMcpServers` の `${VAR}` 参照修正**: 述語に変数参照を使うとマッチしない問題を修正
- **`claude agents` worktree 再オープン修正**: git worktree に入ったバックグラウンドエージェントセッションが "No conversation found" でクラッシュループする問題を修正
- **Ctrl+O トランスクリプトでの thinking 二重描画修正**
- **`/doctor` のリモートセッション内表示修正**: リモートセッション内で「Not inside a remote session」のような矛盾チェックが表示される問題を修正
- **`claude agents` dispatch/reply 入力のマルチライン修正**: 複数行プロンプト入力時にカーソルが一行目末尾に貼り付く問題を修正
- **Unicode 非対応端末でのタスク一覧表示修正**

---

## v2.1.165 (2026-06-05)

- バグ修正・信頼性改善のみ（公式 changelog 記載）。ユーザー向け新機能なし

---

## v2.1.163 (2026-06-04)

- **`requiredMinimumVersion` / `requiredMaximumVersion` managed setting 追加**: Claude Code のバージョンが許容範囲外なら起動を拒否し、承認済みバージョンへ案内する。Managed Policy でのバージョン pin が可能に
- **`/plugin list` コマンド追加**: インストール済みプラグイン一覧を表示。`--enabled` / `--disabled` フィルタ対応
- **`/btw` に「c でコピー」ショートカット追加**: 生 Markdown 回答をクリップボードへコピーし、書式を保ったまま貼り付け可能
- **Hooks: `Stop` / `SubagentStop` で `hookSpecificOutput.additionalContext` をサポート**: Claude にフィードバックを返してターンを継続させられる。hook エラー扱いではない通常応答として返るように
- **Skills: `\$` エスケープ構文**: コマンド本文中で数字の前にリテラル `$` を書く場合の新エスケープ
- **`--resume` 時の stdio MCP サーバーへの `CLAUDE_CODE_SESSION_ID` 共有**: hooks / Bash と同じ session ID を `--resume` でも stdio MCP サーバーに渡すように（v2.1.154 で導入されたサブプロセス環境変数が `--resume` パスでも一貫して渡る）
- **Background agent の version 更新挙動改善**: バックグラウンドエージェントセッションが新バージョンへバックグラウンド更新するようになり、Claude Code 更新後にセッションを開いてもコールド再起動を待たされない
- **`/` メニューの説明文整理**: 組み込みコマンド・スキルの説明をより明確化
- **`claude agents` のディレクトリ継承**: state-grouped view から dispatch するとエージェントビューを開いたディレクトリでセッションを開始する
- **サブスクリプション切替の案内位置変更**: トースト通知から起動アナウンス枠に移動
- **`claude -p` ハング修正**: バックグラウンドで起動したコマンドが終了しない場合に `claude -p` が最終結果後にハングする問題を修正（stdin クローズ後 5 秒でバックグラウンドシェルを停止）
- **`claude -p` の Bedrock/Vertex/Foundry 認証修正**: `CI=true` で Anthropic API キー未設定の場合に「ANTHROPIC_API_KEY required」で失敗する問題を修正
- **`$TMPDIR` 上書き範囲修正**: v2.1.154 のリグレッションで全コマンドが `/tmp/claude-{uid}` に上書きされていた問題を修正（サンドボックスコマンドのみに限定）。Bazel / EDR 保護 Go 環境での Bash 失敗が解消
- **Windows OneDrive / 読み取り専用属性での session-env 修正**: Windows でセッション環境ディレクトリが read-only 属性 or OneDrive 配下にあると「EEXIST: file already exists」で Bash が失敗する問題を修正
- **組織 managed 権限ルールの初期セッション修正**: fresh config ディレクトリで起動中に managed settings 取得が完了すると、組織 managed 権限ルールが全セッションに適用されない問題を修正
- **`claude agents` バックグラウンドタスク継続性**: 再アタッチ時にバックグラウンドタスクが失われる問題を修正
- **`claude agents` 終了時のターミナル整列修正**: Esc でエージェントビューを抜けた際の表示崩れと数秒のハングを修正
- **デスクトップアプリ Stop ボタン修正**: バックグラウンドタスクチップで Stop を押した際、プロセスが既に消えていてもチップが残る問題を修正
- **ペースト直後のキー応答不能修正**: ペースト終端マーカーがターミナルでドロップされるとキー入力が永久に止まる問題を修正
- **Hook `if: "Bash(...)"` 条件の改善**: `$()` や `$VAR` を含む全 Bash コマンドで誤発火していた問題を修正。サブシェル/バッククォート内のコマンドにもマッチするように
- **`Read(~/...)` deny ルールの Bash 連携修正**: ホームディレクトリパスの deny ルール（例 `Read(~/Desktop/**)`）が `$HOME` 経由で参照する Bash コマンドをブロックしない問題を修正
- **トランスクリプトの「(no content)」削除**: `/mcp` / `/plugins` 等のパネル閉じた後に残る空行を除去

---

## v2.1.162 (2026-06-03)

- **`claude agents --json` に `waitingFor` フィールド追加**: 待機中セッションが何を待っているかを構造化出力に含める
- **ネイティブビルドでの Grep/Glob 提供**: `--tools` で明示的に列挙された場合、ネイティブ build (embedded search) でも Grep / Glob を提供
- **`/effort` 永続化確認**: 選択したレベルが新規セッションのデフォルトとして保存される旨を確認表示
- **スラッシュコマンドのオートコンプリート挙動変更**: 補完候補は即実行せずプロンプトに挿入。Enter で実行
- **Remote Control 表示形式変更**: 起動時メッセージではなく永続フッターピルとして表示
- **`Windsurf` → `Devin Desktop` リネーム**: CLI 全体で名称統一
- **起動時の出力削減**: 通知をグループ化、セッション情報を圧縮表示。「Claude in Chrome enabled」「marketplace installed」起動メッセージを削除
- **Launch-prompt 警告の挙動変更**: 入力欄下部に対応されるまでピン留め
- **失敗ターン表示**: 複数行エラーブロックではなくコンパクトな警告表示
- **`claude update` の起動検証強化**: バックグラウンドサービス開始フローを改善
- **セキュリティ**: `WebFetch` パーミッションルールが事前承認ドメイン (preapproved domains) に対して適用されない問題を修正。明示的ルールが事前承認に優先するように
- **セキュリティ**: Windows でバックスラッシュ / 大文字小文字違いを含むパーミッションルールがマッチしない問題を修正。`Read` deny ルールが Glob/Grep からも該当ファイルを隠すように
- **割り込み修正**: ターン開始時の Esc 割り込みが `stream-json` / SDK セッションで silently drop される問題を修正
- **API エラー修正**: 切り詰め境界付近の絵文字を含む classifier クエリで `no low surrogate` API 400 エラーを修正
- **MCP timeout 修正**: 1000ms 未満の MCP `timeout` が 1 秒の watchdog に floor される問題を修正（無視して env/default にフォールバック）
- **LSP `workspaceSymbol` 修正**: 結果が空になる問題を修正。`query` パラメータを受け付けるように
- **`claude agents` 表示修正**: 60〜120 columns でステータステキストが切られる問題を修正（フル幅利用）。40 columns でセッション名が切られる問題を修正。Ctrl+V 画像ペーストが反応しない問題を修正
- **バックグラウンドセッション修正**: サービス起動失敗時に会話を無音で失う問題を修正。エージェントビューで失敗した返信を queue するように修正。深い `CLAUDE_CODE_TMPDIR` でクロスセッション `SendMessage` が壊れる問題を修正。実行中のバックグラウンドセッションを開く際の 5 秒 stall を修正

---

## v2.1.161 (2026-06-02)

- **`OTEL_RESOURCE_ATTRIBUTES` をメトリクスのラベル化**: `OTEL_RESOURCE_ATTRIBUTES` で渡した属性（team, repo 等）をメトリクスデータポイントのラベルとして添付。Team / Repo 別カスタムディメンションでのスライス分析が可能
- **`claude agents` ビュー改善**: 行表示で `done/total`（fan-out された作業の進捗）を詳細より前に表示。peek は最長実行中の項目を表示
- **`/mcp` で未使用 connector 折りたたみ**: 未使用 claude.ai connectors を「Show unused connectors」の下に折りたたみ、表示を整理
- **並列ツール実行の独立化**: 同一バッチ内の Bash コマンドが失敗しても他の並列ツール呼び出しがキャンセルされず、それぞれ独立して結果を返すように
- **Linux fullscreen クリップボード強化**: `wl-copy` / `xclip` / `xsel` が利用可能なら使用。クリップボードと PRIMARY selection の両方にコピーし、ミドルクリック貼り付けに対応。「hold `{key}` for native selection」ヒントをターミナル別に正しく表示
- **アクセシビリティ「Reduce motion」遵守**: `/effort` ダイアログ、ワークフロー アニメーション、プロンプトキーワード shimmer がいずれも「Reduce motion」設定を尊重するよう修正
- **セキュリティ**: `claude mcp list` / `get` / `add` がシークレットを出力する問題を修正 — `${VAR}` 参照は展開せず、認証ヘッダーや URL 中のシークレットを redact
- **Managed settings 修正**: `forceLoginOrgUUID` / `forceLoginMethod` が組織 pin と並んで Bedrock / Vertex / Foundry / Mantle のサードパーティプロバイダセッションをブロックしていた問題（v2.1.146 のリグレッション）を修正
- **`/usage-credits` 修正**: Team / Enterprise admin に対して再ログインを開始する代わりに、組織 usage settings ページへ案内するように
- **`/autofix-pr` worktree 対応**: git worktree 内で「cannot run on default branch」が誤報告される問題を修正
- **`--resume` ピッカー修正**: カレントディレクトリが git worktree でない場合にセッションが表示されなかった問題を修正
- **Workflow worktree 隔離修正**: `isolation: "worktree"` 指定の Workflow エージェントが、バックグラウンドセッションで worktree ファイルの編集をブロックされていた問題を修正
- **バックグラウンドセッション**: デーモン環境のモデルではなく `settings.json` のモデルを使うように修正。`.claude/worktrees/` 配下の 30 日保持スイープで残るオーファン worktree を修正。`--output-format text` / `json` 利用時に `claude -p` の stdout がサブエージェント出力で汚染される問題を修正
- **Windows hooks 修正**: `/usr/bin/bash script.sh` のように bash を明示する Windows hooks が「command not found」で失敗する問題を修正
- **OpenTelemetry 修正**: テレメトリ初期化前の `user_prompt` / `api_request` / `tool_result` / `tool_decision` ログイベントが silently drop される問題を修正
- **パフォーマンス**: ターミナルレンダリングのレイアウトエンジン JIT プロファイル安定化、大きなファイル書き込み時の描画性能改善
- **VS Code 統合**: garbled glyph 対策としてターミナル GPU acceleration 無効化のヒントを追加

---

## v2.1.160 (2026-06-02)

- **シェル起動ファイル書き込み時のセキュリティプロンプト追加**: `.zshenv` / `.zlogin` / `.bash_login` および `~/.config/git/` への書き込み前に確認を求める
- **`acceptEdits` モードのビルドツール設定ファイル保護拡張**: コード実行を許可しうる設定ファイル (`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/` 等) への書き込み前にプロンプト
- **Dynamic workflow トリガーキーワード変更**: `workflow` → `ultracode` にリネーム。`/config` の Workflow keyword trigger 設定も `ultracode` ベースに
- **`CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` 環境変数を削除**: no-op 化（v2.1.154 で非推奨化済み）。Opus 4.6 で fast mode を使うには `/model claude-opus-4-6[1m]` → `/fast on`
- **Edit の read-before-edit 制約を緩和**: 単一ファイルに対する `grep` / `egrep` / `fgrep` コマンドが read 済みとみなされ、別途 `Read` が不要に
- **WSL クリップボード修正**: copy-on-select が Windows クリップボードへ書き込まれない問題を修正（OSC 52 ではなく PowerShell interop を使用）
- **`claude agents` セッション復元修正**: 完了済みセッションの復元時にチャット履歴が失われ元プロンプトが再実行される問題を修正
- **バックグラウンドセッション再アタッチ修正**: 夜間 retire 後の再アタッチで会話が失われる問題を修正
- バックグラウンドデーモン、agents 一覧、ターミナル応答性の各種安定化修正

---

## v2.1.159 (2026-05-31)

- 内部インフラ改善のみ（ユーザー向け変更なし）

---

## v2.1.158 (2026-05-30)

- **Auto mode が Bedrock / Vertex / Foundry 対応**: Opus 4.7 / 4.8 で `CLAUDE_CODE_ENABLE_AUTO_MODE=1` を設定して opt-in

---

## v2.1.157 (2026-05-29)

- **`.claude/skills/` 配下のプラグイン自動ロード**: マーケットプレース不要で `.claude/skills` 内プラグインを自動認識
- **`claude plugin init <name>`**: `.claude/skills` 配下に新規プラグインの雛形を生成
- **`/plugin` 引数の autocomplete 強化**: サブコマンド、インストール済みプラグイン名、既知マーケットプレース上のプラグインを補完
- **`claude agents` の `agent` 設定参照**: `settings.json` の `agent` フィールドが dispatched セッションで尊重される。`--agent <name>` で上書き可能
- **`EnterWorktree` で Claude 管理 worktree 間を切替**: セッション途中で別の管理 worktree に切り替え可能
- **`tool_decision` テレメトリ拡張**: `OTEL_LOG_TOOL_DETAILS=1` で `tool_parameters`（bash コマンド、MCP/skill 名等）を含める
- **Claude 管理 worktree のロック解除**: エージェント終了時に worktree をアンロックし、`git worktree remove`/`prune` でクリーンアップ可能に
- **Workflow キーワードトリガー設定**: `/config` から「Workflow keyword trigger」を OFF にすると、プロンプト中の "workflow" がワークフロー要求を発動しなくなる
- **Backspace でワークフロー要求を解除**: トリガーキーワード直後の Backspace で文字削除ではなくワークフロー要求を解除（`alt+w` と同等）
- **`/terminal-setup` で VS Code/Cursor/Windsurf の GPU acceleration 無効化**: ターミナル garbled text を防止
- **`claude agents` の slash command autocomplete が substring 一致**: 部分一致でマッチ
- **長尺・再開会話のレンダリング性能改善**: 冗長な再計算を排除
- **Feature of the Week 通知**: クレジット請求ステータスをプロンプト上の行ではなく status area の通知として表示
- **起動バナーの整理**: sandbox banner と `/ide for …` ヒントを除去（sandbox 状態は `/status` とブロック時に確認可能）
- 多数のバグ修正: 画像処理クラッシュ、`claude agents` のセッションリタイヤ/ESC/Tmuxクリップボード、`--resume` でのバックグラウンドサブエージェント表示、`--worktree` での canonical root 復帰、`/model` ピッカーの "Newer version" 誤表示、fullscreen での markdown 文字残留、managed-settings ダイアログ承認後フリーズ、WSL 画像貼り付け、VS Code/Cursor/Windsurf 統合ターミナルでの右クリック paste 重複、background session の日時通知、サブエージェントのバックグラウンドシェルリーク、起動時の sandbox network 権限プロンプト等

---

## v2.1.156 (2026-05-29)

- **Opus 4.8 の thinking blocks 修正**: thinking block 改変による API エラーを修正

> v2.1.155 は欠番（公式 changelog で v2.1.154 から v2.1.156 へ）

---

## v2.1.154 (2026-05-28)

- **Opus 4.8 リリース**: モデルID `claude-opus-4-8`。デフォルトで high effort。最難タスク向けは `/effort xhigh`
- **Dynamic workflows**: 「ワークフローを作って」と依頼すると、バックグラウンドで数十〜数百のエージェントを跨いだ作業をオーケストレーション。`/workflows` で実行状況を表示
- **Opus 4.8 の Fast mode が大幅値下げ**: 標準レートの2倍で2.5倍の速度
- **Lean system prompt がデフォルト化**: Haiku、Sonnet、Opus 4.7以前を除く全モデルでデフォルト
- **多肢選択プロンプトの濫用抑制**: 既にコンテキストで判断可能な場面では選択肢を出さず、本当に判断できないときだけ表示
- **`/simplify` の挙動変更**: 旧来の `/code-review --fix`（バグハンティング含むレビュー）ではなく、クリーンアップ専用レビュー（reuse / simplification / efficiency / altitude）を実行して fix を適用
- **`/effort` スライダのラベル変更**: 「Speed」/「Intelligence」→「Faster」/「Smarter」
- **`claude agents` でバックグラウンドシェルセッション**: `! <command>` で起動・アタッチ・デタッチ可能。`claude --bg --exec '<command>'` でも同等
- **`claude agents` の `/logout` 修正**: バックグラウンドセッション送信ではなく実際にサインアウト
- **`←←` で agents view を開く動作の拡張**: Bedrock / Vertex / Foundry / テレメトリ無効化環境でも動作
- **Claude in Chrome: ブラウザ選択**: `/chrome` → "Select browser…" または複数ブラウザ接続時のチャット内選択でブラウザを指定可能
- **プラグインの `defaultEnabled: false`**: `plugin.json` またはマーケットプレースエントリで宣言可能。`/plugin` または `claude plugin enable` で有効化。有効化済みプラグインの依存関係は引き続き自動有効化
- **`/plugin` Discover タブのディレクトリ別レコメンド**: 現在のディレクトリにマッチするレレバンスシグナルを持つプラグインを "suggested for this directory" 注釈付きでピン留め
- **ストリーミングツール実行が常時有効化**: テレメトリ無効環境や Bedrock / Vertex / Foundry でも有効（旧フィーチャーフラグ廃止）
- **Stdio MCP サーバーサブプロセスに `CLAUDE_CODE_SESSION_ID` と `CLAUDECODE=1` を渡す**
- **`claude mcp list` / `get` で `⏸ Pending approval` 表示**: パイプ出力時に未承認 `.mcp.json` サーバーを自動承認・接続せず保留状態として表示
- **`/remote-control` の disconnect 補完**: Remote Control 有効時に "Disconnect Remote Control" を補完表示
- **`/claude-api` スキルが Opus 4.8 対応 + 4.7 → 4.8 移行ガイド追加**
- **非推奨**: `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` 環境変数（2026-06-01 削除予定）。Opus 4.6 で fast mode を使うには `/model claude-opus-4-6[1m]` の後に `/fast on`
- **Auto mode 分類器の改善**: データ流出（特にリポジトリ全体のバルク転送）検知強化
- セキュリティ修正: `HOME` に末尾スラッシュがある場合の `rm -rf $HOME` 危険パスブロック漏れ、サンドボックス内外で `$TMPDIR` が別ディレクトリに解決される問題
- 各種修正: `claude agents` ハイライト行可読性、background-agent 完了通知での1Mコンテキストモデルにおける "out of context" 早期発動、background-session 分類器の `/command` 起動時ゴール喪失、Claude Code 更新後のピン留め background session 毎分再生成、background session の "blocked"/"running"/"working" stuck、background subagent の worktree-isolation 迂回、macOS daemon 終了後の `claude --bg-pty-host` プロセス 100% CPU、option dialog のディバイダ下選択肢への数字キーショートカット不動作、`worktree.baseRef: "head"` がワークツリー内 HEAD ではなくメインチェックアウト HEAD に解決される問題、ターミナル幅と一致する行末以降の折り返し先頭スペース、VS Code でのターミナルレンダリング破損、plan ファイル名に `[Image #N]` / `[Pasted text #N]` プレースホルダ混入、短いANSIカラー行への幻の "ctrl+o to expand" ヒント、`allowedMcpServers` / `deniedMcpServers` の1件無効化で managed-settings ポリシー全廃棄、`CLAUDE_CODE_ALWAYS_ENABLE_EFFORT` 設定モデル非対応時の API 400、Windows `claude.exe` 使用中の更新失敗エラーメッセージ、shortcuts help panel の stale "& for background" ヒント、VSCode Auto mode と bypass-permissions、task panel の余分な "main" 行、`/mcp` tools 一覧のレンダリング、`/model` ピッカーの API（pay-as-you-go）ユーザー fast mode pricing 表示、Auto mode の safety classifier 出力トークン枯渇時の誤ブロック

---

## v2.1.153 (2026-05-28)

- **`skipLfs` プラグインマーケットプレースオプション**: `github`/`git` ソースに `skipLfs: true` を指定すると Git LFS ダウンロードをスキップ可能
- **npm global install 自動更新の警告**: 自動更新できない環境で1回限りの通知を表示
- **ステータスラインに `COLUMNS` / `LINES` 環境変数**: status line スクリプトがターミナルの幅・行数を環境変数経由で参照可能
- **`claude agents` 補完にネイティブスラッシュコマンドとバンドルスキルを含める**: 補完候補が拡張
- **`claude agents` の PR カラム表示形式**: `PR #N` または `N PRs` で表示
- **`claude doctor` が最終アップデート試行結果を表示**: 自動更新の成否がdoctor出力に含まれる
- **MCP サーバー/コネクタの認証通知を統合**: 「認証が必要」通知を1本化
- **macOS バックグラウンドエージェントが "Claude Code" として Privacy & Security に表示**
- 各種修正: MCP サーバー関連、API gateway 認証情報処理、プラグイン MCP サーバー、UI/UX 細部

---

## v2.1.152 (2026-05-27)

- **`/code-review --fix`**: レビュー結果（再利用・簡素化・効率改善）をワーキングツリーに直接適用する `--fix` フラグ追加。旧 `/simplify` は `/code-review --fix` を呼び出すエイリアスに復活
- **`disallowed-tools` フロントマター**: スキル・スラッシュコマンドのフロントマターで指定したツールをスキル有効中の間モデルから取り除ける
- **`/reload-skills` コマンド追加**: セッションを再起動せずにスキルディレクトリを再スキャン
- **`SessionStart` フックが `reloadSkills: true` を返却可能**: フックでインストールしたスキルを同一セッション内で利用可能に
- **`SessionStart` フックがセッションタイトルを設定可能**: `hookSpecificOutput.sessionTitle` で起動・再開時にタイトルを指定
- **`MessageDisplay` フックイベント追加**: アシスタントメッセージ表示時にテキストを変換または非表示にできる
- **`pluginSuggestionMarketplaces` managed setting 追加**: 管理者がコンテキストアウェア tips 経由で suggest 対象とする組織マーケットプレースを allowlist 化
- **`claude plugin marketplace remove` に `--scope user|project|local`**: `marketplace add` / `install` / `uninstall` と対称化
- **`--fallback-model` の自動切替**: プライマリモデルが見つからない場合、リクエスト毎に失敗させるのではなく、設定済み `--fallback-model` でセッション残りを継続
- **Auto mode の opt-in 不要化**: 同意ダイアログなしで Auto mode が利用可能に
- **Vim mode の `/` NORMAL モード**: bash/zsh vi-mode と同様に逆方向履歴検索（Ctrl+R 相当）を開く
- **`/usage` 内訳の large session files 対応**: ストリーム読み込みでメモリ使用量はフラットを維持しつつ大規模セッションファイルも内訳に含める
- **OpenTelemetry: `app.entrypoint` 属性**: セッションエントリポイントをメトリクス属性として記録（`OTEL_METRICS_INCLUDE_ENTRYPOINT=true` で opt-in）
- **Thinking summaries 改善**: 折り畳みグループで最低3秒表示、markdown レンダリング、10行で打ち切り（`Ctrl+O` で完全表示）
- **Fullscreen の "Thinking for Ns" インジケータ**: thinking 中ライブで秒数カウントアップ、途中中断時も値を保持
- **Workflow tool のインライン進捗表示簡素化**: ライブエージェント数はプロンプト下の永続 workflow status 行のみに表示
- **post-response timer**: バックグラウンドエージェント/ワークフロー実行中は "Waiting for N background agents/workflows to finish" を表示、結果処理後に累積時間をレポート
- 各種修正: 長時間セッションでのターミナルスタイル劣化（renderer style プール再利用）、condensed startup mode で sandbox-enabled warning 不表示、loading spinner の "still thinking"/"almost done thinking" 誤表示、focus mode の "N messages hidden" 誤カウント、展開ツール結果内のリンクをクリックすると展開がクローズ、markdown table セル枠線のインラインコード色継承、プラグイン MCP サーバーの環境変数違いでの誤 dedup、stale `enabledPlugins` での `/doctor` 誤レポート、Claude Code Remote セッション + egress proxy 環境での remote MCP 接続失敗、effort 変更確認ダイアログの誤発火、`--bare`/attachments disabled での Agent ツール description、`claude agents` の background worker クラッシュ、`cache_creation_input_tokens` の 0 報告問題、PushNotification の "Mobile push not sent (Remote Control inactive)" 誤レポート、モデル/login 切替後の stale thinking-block signature でセッションが固まる問題、等

> v2.1.151 は欠番（公式 changelog で v2.1.152 が直接続く）

---

## v2.1.150 (2026-05-23)

- 内部インフラ改善のみ（ユーザー向け変更なし）

---

## v2.1.149 (2026-05-22)

- **`/usage` カテゴリ別内訳**: 制限使用量を駆動している要因（skills / subagents / plugins / MCPサーバー単位のコスト）を内訳表示
- **`/diff` 詳細ビューのキーボードスクロール**: 矢印キー / `j` / `k` / `PgUp` / `PgDn` / `Space` / `Home` / `End` で詳細ビューをスクロール可能
- **Markdown 出力で GFM タスクリストのチェックボックスをレンダリング**: `- [ ] todo` / `- [x] done` をプレーン箇条書きではなくチェックボックスとして描画
- **Enterprise: `allowAllClaudeAiMcps` managed setting 追加**: `managed-mcp.json` に並んで claude.ai クラウド MCP コネクタを一括ロード
- **`/feedback` レポート改善**: コンテキスト圧縮以前の会話も含めるようになり、長時間セッション序盤に発生した問題のトリアージが容易に
- **セキュリティ修正（PowerShell 権限バイパス）**: ビルトイン `cd` 関数（`cd..`, `cd\`, `cd~`, `X:`）が作業ディレクトリを検出されずに変更し、後続コマンドがワークスペース外を読めていた問題を修正
- **セキュリティ修正（sandbox 書き込み許可リスト）**: git worktree でメインリポルート全体が書き込み許可になっていた問題を修正（共有 `.git` ディレクトリのみ許可、`hooks/` と `config` は拒否）
- **PowerShell の prefix/wildcard ルール修正**: `PowerShell(dotnet.exe build *)` 等のルールがネイティブ実行可能ファイルやスクリプトを事前承認しなかった問題
- **PowerShell 権限解析の `cd`/`pushd`/`popd` 追跡修正**: パーサが `PWD`/`OLDPWD`/`DIRSTACK` の古い変数追跡値を信用していた問題
- **`find` の macOS vnode テーブル枯渇修正**: Bash ツールで大規模ディレクトリツリーに `find` を実行するとシステムの file/vnode テーブルを使い切ってホストごとクラッシュする問題
- **managed-settings 承認ダイアログ修正**: 起動時に承認した直後にターミナルがフリーズする問題
- **`/ultraplan` とリモートセッション作成修正**: 作業ツリーに実質的な変更がない場合の `Could not capture uncommitted changes` 失敗
- **`otelHeadersHelper` 修正**: スクリプトパスに空白が含まれる場合にサイレント失敗していた問題。ヘルパー失敗は `/doctor` とデバッグログに報告されるようになった
- 各種 UI/UX 修正（thinking スピナーの色遷移、Ctrl+O トランスクリプトのテイリング、`/config` 退出サマリーの誤検知、`/insights` のキャッシュ欠損クラッシュ、リコールしたプロンプト編集の喪失、`/effort` 適用レベル表示、引数ヒントの末尾クリッピング、貼り付けテキストのプレースホルダ化、Remote Control セッション名同期、Jump to bottom ピル等）

---

## v2.1.148 (2026-05-22)

- **Bash ツール exit code 127 リグレッション修正**: v2.1.147 で導入されたリグレッションにより、一部ユーザーで全コマンドが exit code 127 を返す問題

---

## v2.1.147 (2026-05-21)

- **`/simplify` → `/code-review` にリネーム**: オプションの effort level 引数を受け付ける（例: `/code-review high`）。バンドルスキルの命名を「コード品質レビュー＋修正」という実態に合わせた。既存の `/simplify` 呼び出し箇所は移行が必要
- **Auto mode が `AskUserQuestion` を不要に抑制しなくなった**: ユーザーまたはスキルが明示的に依拠する場合は質問を発火させる
- **MCP リスト系のページネーション修正**: `resources/list` / `resources/templates/list` / `prompts/list` がページ 1 以降の項目を取りこぼしていた問題
- **`/background` の入力判定修正**: タイプ済み入力がスキルまたはカスタムスラッシュコマンドのみだったセッションを拒否していた問題
- **バックグラウンドセッションの権限再プロンプト修正**: 「もう聞かない」で許可済みのツール権限が再プロンプトされる問題
- **`forceLoginOrgUUID` / `forceLoginMethod` managed-settings の強制適用修正**: third-party-provider セッション・API キーセッションに対して enforcement されていなかった
- **`CLAUDE_CODE_SUBAGENT_MODEL` の子プロセス継承修正**: multi-agent セッションで子プロセスに転送されていなかった
- **Windows PowerShell ツール修正**: `pwsh` を winget / Microsoft Store 経由で導入した場合に "command line is invalid" で失敗していた問題（v2.1.124 のリグレッション）
- **Windows: バックグラウンドジョブ worktree 削除で NTFS junction を本リポに follow しない**
- **`/theme` の Esc 対応**: カラーエディタと「New custom theme」ダイアログが Esc に反応しない問題
- **Agent SDK 経由のストリーミング終端で uncaught exception 修正**
- **GNOME Terminal の右クリック / ミドルクリック貼り付け修正**
- **Windows Terminal でバックグラウンドセッション attach 中のフルスクリーン strobing 修正**
- **auto-updater 改善**: ネイティブバージョン確認・ダウンロードが即時失敗ではなく一過性ネットワーク障害をリトライ。更新失敗時もステータスラインに現バージョンを表示
- **大規模ファイル編集の diff 描画性能向上**

---

## v2.1.145 (2026-05-19)

- **`claude agents --json` 追加**: ライブセッション一覧を JSON で出力。tmux-resurrect / ステータスバー / セッションピッカー等の自動化向け
- **OTEL: `agent_id` / `parent_agent_id` 属性追加**: `claude_code.tool` スパンにエージェント識別子を付加。バックグラウンドサブエージェントスパンが dispatch 元の Agent ツールスパン配下にネストするようトレース親子関係を修正
- **ステータスライン JSON 入力に GitHub repo/PR 情報**: 検出時に GitHub リポジトリと PR 情報を含めて渡す
- **`/plugin` Discover/Browse 画面の事前プレビュー**: インストール前にプラグインが提供する commands / agents / skills / hooks / MCP/LSP サーバーを一覧表示
- **`claude agents` ターミナルタブタイトルに awaiting-input 件数**: alt-tab 中の別ウィンドウでもエージェントの注意要求が分かる
- **スラッシュコマンド/@-メンション候補のマウス対応**: フルスクリーンモードでホバー＆クリックに応答
- **Stop / SubagentStop hook 入力拡張**: `background_tasks` と `session_crons` フィールドを追加
- **権限プロンプトバイパスのセキュリティ修正**: Bash コマンド内で許可リストにない環境変数への裸の代入が自動承認されていた問題を修正
- **MCP prompt スラッシュコマンドのエラー改善**: 必須引数欠落時に raw な server validation エラーではなく、不足引数名と期待される使い方を表示
- **ターミナル描画修正**: リサイズや再フォーカス後にキー入力までスピナーと経過時間表示がフリーズする問題
- **Windows PowerShell 5.1 修正**: クロスプロジェクト resume ヒントが `;` をコマンドセパレータとして使うように
- **agent view リペインの voice push-to-talk 修正**
- **タスクリストの並び順修正**: 複数タスクを同時作成すると順序がランダムになっていた問題
- **`/review` の GraphQL 修正**: deprecated な `projectCards` クエリを使用しており Classic Projects のリポで失敗していた問題
- **`claude plugin validate` の検出強化**: `skills:` エントリがディレクトリではなくファイルを指している場合にフラグし、親ディレクトリを示すヒントを表示
- **`context: fork` スキルの無限ループ修正**: 実行されず自己再呼び出しを繰り返す問題
- **Read tool の堅牢化**: 全体読み取りがトークン上限を超えた場合に hard error ではなく "PARTIAL view" 通知付きで先頭ページを返す
- **その他**: Anthropic マーケットプレース二重インストール banner の修正、`gh pr create` 等実行後のフッター PR バッジ即時更新、Agent Teams で非ASCII名チームメイトの API ヘッダエンコーディング修正

## v2.1.144 (2026-05-19)

- **`/resume` がバックグラウンドセッションに対応**: `claude --bg` や agent view から起動したセッションも、インタラクティブセッションと並んで `bg` タグ付きで `/resume` ピッカーに表示される
- **`/model` のスコープが現在セッション限定に**: `/model` で選んだモデルは現セッションにだけ適用される。新規セッションのデフォルトを変更するにはモデルピッカーで `d` を押す
- **「extra usage」→「usage credits」リネーム**: CLI コピーが usage credits に統一。`/extra-usage` は `/usage-credits` に改名（旧名もエイリアスとして残存）
- **`/plugin` browse/discover ペインに最終更新日表示**: マーケットプレイス画面で各プラグインの last-updated 日付が確認できる
- **バックグラウンドサブエージェント通知に経過時間**: 完了通知に "Agent completed · 3h 2m 5s" のように所要時間を表示
- **起動ハング修正**: `api.anthropic.com` が到達不能（captive portal / firewall / VPN）なときに最大 75 秒固まる問題を修正。side-channel API 呼び出しに 15 秒タイムアウトを導入
- **MCP 重要バグ修正**:
  - paginated な `tools/list` レスポンスが先頭ページのみ返されてツールが silently 欠落する問題
  - SVG など未対応 MIME タイプの画像が会話を破壊する問題（ディスク保存してツール結果から参照する形式に）
- **プラグイン関連バグ修正**:
  - 自分の settings で有効化したプラグインが新規マシンで初回ロード後に "not cached" エラーを返す問題
  - プロジェクト `.claude/settings.json` のみで有効化されたプラグインに `claude plugin install` ヒントが表示されるように
  - `claude mcp list` が `.mcp.json` をパースできないとき（例：VS Code 形式の `"servers"` キー）silently 0 サーバーと報告していた問題（設定エラーを表示するように）
- **ターミナル描画修正**:
  - window-resize イベントを取りこぼした後の garbled output。Ctrl+L 不要で次フレームで自己修復
  - 超長セッションでの progressive display corruption（stale/garbled glyphs。リサイズ/再起動でしか直らなかった）
  - VS Code 上でのスピナーアニメーション色数を削減し描画グリッチ低減
- **ファイル操作の堅牢化**:
  - 画像拡張子だが実体が画像でないファイル（HTML を .png 保存等）読み込み時に会話が unrecoverable になる問題（テキストにフォールバック）
  - `head` / `tail` のファイル閲覧が read-before-edit チェックを満たすように
  - `egrep`/`fgrep`/`git grep`/`git diff` の "no matches"（exit 1）をコマンド失敗と報告しないように
- **バックグラウンドセッション関連の多数の修正**:
  - macOS で Full Disk Access 保護フォルダ配下のプロジェクトを使うと「exit 1 before init」でクラッシュ（v2.1.143 リグレッション）
  - `/branch` が worktree 投入後やバックグラウンドセッションで "No conversation to branch" 失敗
  - スクロール（PgUp/PgDn・マウスホイール・Ctrl+O トランスクリプト）が Windows でアタッチ中に効かない問題
  - アタッチ中のバックグラウンドセッション ターミナル close 時のクラッシュ
  - `! <cmd>` 実行セッションでアタッチ中に Ctrl+C が効かない問題
  - agent view のシェルコマンド行が完了後も Working に残り、Enter で再実行されてしまう問題
  - Windows で `claude agents` の ← 後にリストが入力非応答になる問題
  - `/bg` と ← detach で `/add-dir` 追加ディレクトリが保持されない問題
  - `claude respawn <id>` が停止中バックグラウンドセッションを「running」と表示する問題
  - `claude logs <id>` や agent view からのセッション再開がバックグラウンドサービス無反応時にハング（10s タイムアウト＋復旧ヒント）
- **モデル選択関連バグ修正**:
  - IDE モデルピッカーや `applyFlagSettings` 経由でのモデル変更が起動後に適用されない問題
  - resume 後のセッションが他セッションの `/model` 選択を引き継いでしまう問題
  - Bedrock/Vertex ユーザーが `/model` ピッカーから "Opus (1M context)" を選べない問題（v2.1.129 リグレッション）
- **AskUserQuestion**: notes フィールドで Esc を押すとターンが中断されていた問題（回答選択に戻るように）
- **`/doctor`**: コマンド hook の `command` 欄欠落時に exec 形式の例を表示。スキル一覧の truncation を起動時通知ではなく `/doctor` 内に集約
- **SDK / Headless**: MCP pre-wait が startup と並行実行されるようになり、遅い MCP サーバーで最大 2 秒高速化
- **その他細かい修正**: `/feedback` survey 後の follow-up ヒント文面改善、background side-query が ANTHROPIC_BASE_URL/Bedrock Mantle で Haiku 未使用問題、background daemon spawn の launcher 欠落フォールバック、攻撃的な stream stall 時の一度限り再試行、リモートセッションログイン時の `forceLoginMethod`/`forceLoginOrgUUID` 関連修正、`spinnerVerbs` がポストターン文言を上書きしないように

## v2.1.143 (2026-05-15)

- **プラグイン依存関係の強制**: `claude plugin disable` が他の有効プラグインの依存先を無効化しようとすると拒否し、コピペ可能な disable-chain ヒントを表示。`claude plugin enable` は推移的依存を強制有効化
- **`/plugin` マーケットプレイス: 推定コンテキストコスト表示**: マーケットプレイス browse ペインに projected context cost（ターン当たり・呼び出し当たりのトークン推定）を追加
- **`worktree.bgIsolation: "none"` 設定追加**: バックグラウンドセッションが `EnterWorktree` なしで作業コピーを直接編集可能に。worktree が現実的でないリポジトリ向け
- **PowerShell ツール改善**:
  - PowerShell ツール実行時に `-ExecutionPolicy Bypass` を渡すように。`CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY=1` でオプトアウト
  - Windows の Bedrock/Vertex/Foundry ユーザーでも PowerShell ツールがデフォルト有効に。`CLAUDE_CODE_USE_POWERSHELL_TOOL=0` でオプトアウト
- **バックグラウンドセッション**: アイドルからのウェイク後にモデルと effort レベルを維持
- **`claude agents` 新フラグ拡張**:
  - `--add-dir`, `--settings`, `--mcp-config`, `--plugin-dir` が view 自体および view から dispatch されるバックグラウンドセッションに適用
  - `--permission-mode`, `--model`, `--effort`, `--dangerously-skip-permissions` で view から dispatch されるセッションのデフォルトを指定可能
  - view から起動されるバックグラウンドセッションが `permissions.defaultMode` を尊重（従来は auto モードに上書きされていた）
- **Shift+Tab で auto モードもサイクル対象に**: アタッチ中のエージェントセッションで Shift+Tab のサイクルに auto モードが含まれるように
- **重要バグ修正**:
  - `.credentials.json` の `scopes` 値が非配列で破損していると CLI 起動がハング/OAuth トークン更新がサイレント失敗する問題
  - Windows Terminal/WSL で `claude agents` の右クリックペーストが効かない問題
  - ブロックを繰り返す stop hook が無限ループする問題（8 連続ブロックでターン終了。`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` で上限調整可）
  - Claude がイテレーション間で idle のとき Esc/Ctrl+C が保留中の `/loop` ウェイクをキャンセルしない問題
  - バックグラウンドシェルや delegated subagent 実行中に `/goal` evaluator が発火する問題
  - settings.json の `env` で `NO_COLOR`/`FORCE_COLOR` 指定すると Claude Code 自身の UI 色まで剥がされる問題（サブプロセスにのみ適用されるように修正）
  - エージェント view がセッション一覧時に Windows で PowerShell プロセスを繰り返し spawn する問題
  - `/bg` を引数なしで実行すると fork セッションに "continue" が送られる問題（fork は入力待ちに）
  - `--agent <name>` がプラグイン提供エージェントを `plugin:` プレフィックスなしでは見つけられない問題
  - エージェント view からセッションを削除しても transcript ファイルが残る問題
  - Windows Terminal アタッチ中のバックグラウンドセッションでスクロール時の stale fragment レンダリング
  - host sleep / macOS App Nap 後のバックグラウンドエージェントで worker-stall 誤検知ストーム
  - 5xx エラーメッセージが設定済みゲートウェイ/クラウドプロバイダ名ではなく `status.claude.com` を指す問題
  - macOS で `~/Documents`, `~/Desktop`, `~/Downloads` 配下のファイルがバックグラウンドジョブセッションから「Operation not permitted」になる問題（Full Disk Access 付与済みでも）
  - `/bg` および ← detach で `--mcp-config`, `--settings`, `--add-dir`, `--plugin-dir`, `--strict-mcp-config` が引き継がれず、再 spawn 後に MCP/設定を失う問題
  - 同様に `--fallback-model` が引き継がれず、オーバーロード時にフォールバックできず hard-fail する問題
  - 同様に `--allow-dangerously-skip-permissions` が引き継がれず、fork worker の Shift+Tab サイクルから bypass mode が消える問題
  - `~/.local/bin/claude` launcher が欠落/非実行可能のとき、バックグラウンドデーモンの spawn が現在実行中のバイナリにフォールバックしない問題
  - `claude agents --allow-dangerously-skip-permissions` が dispatch セッションを bypass モードにデフォルト設定してしまう問題（permission サイクルに available にするだけのはずだった）
  - `claude agents` でレスポンスストリーミング中に ← を押すとエージェントリストが入力に応答しなくなる問題
  - `claude --bg --dangerously-skip-permissions` が retire→wake を跨いで永続化しない問題
  - Bedrock/Vertex/Foundry/gateway で `ANTHROPIC_SMALL_FAST_MODEL` 未設定時にバックグラウンドサイドクエリが Haiku ID 送信で失敗する問題（メインループモデルにフォールバック。v2.1.141 の修正の追加対応）
  - Worktree クリーンアップが `git worktree remove` 失敗時に `rm -rf` フォールバックしなくなった（gitignored / in-progress ファイルの喪失を防止）

## v2.1.142 (2026-05-14)

- **`claude agents` 新フラグ追加**: dispatch されるバックグラウンドセッションの設定を制御可能に
  - `--add-dir`, `--settings`, `--mcp-config`, `--plugin-dir`, `--permission-mode`, `--model`, `--effort`, `--dangerously-skip-permissions`
- **Fast mode デフォルトモデル変更**: Opus 4.6 → **Opus 4.7**。Opus 4.6 に固定したい場合は `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE=1` を設定
- **プラグインの単一スキル化対応**: ルートレベルに `SKILL.md` を置き `skills/` サブディレクトリがないプラグインは、単一のスキルとして直接サーフェスされるように
- **`/plugin` 詳細ペインに LSP サーバー表示**: `claude plugin details` でもプラグインが提供する LSP サーバー一覧を確認可能
- **`/web-setup` 安全装置**: 既存の GitHub App 接続を置き換える前に警告
- **重要バグ修正**:
  - `MCP_TOOL_TIMEOUT` がリモート HTTP/SSE MCP サーバーの per-request fetch timeout に反映されず、設定値に関わらず 60 秒で打ち切られる問題
  - バックグラウンドセッションが既存 git worktree を認識せず Edit がブロックされ EnterWorktree も重複作成を拒否する問題
  - macOS スリープ/ウェイク後にバックグラウンドセッションが消失しデーモン再接続が失敗する問題（clock jump 検出に変更）
  - バイナリアップグレード（`brew upgrade` 等）後にデーモンがクリーン終了せず、dispatch されたエージェントが crash-loop する問題
  - Claude-in-Chrome 拡張接続時に共有タブがないとバックグラウンドエージェントが crash-loop する問題
  - `claude agents` セッションのリンククリックがアタッチ中もヘッドレスブラウザシムを使ってしまう問題
  - `claude agents` の "v to open in editor" がシェルの `$EDITOR`/`$VISUAL` ではなくデーモンのデフォルトエディタを使う問題
  - Windows でネットワークドライブを作業ディレクトリにすると `claude agents` がデッドロックする問題（起動中 Ctrl+C も効くように）
  - Apple Terminal 等 256色のみの端末から `claude agents` セッションにアタッチ時の背景色ブリード
  - `claude --bg --dangerously-skip-permissions` が retire/wake を跨いで永続化しない問題
  - 最初のメッセージがリンクの場合にセッションタイトルが URL から導出される問題
  - リモートクライアントからの冗長な `set_model` リクエストが transcript に重複 `/model` を注入する問題
  - `skills: ["./"]` を使うプラグインで誤った "path escapes plugin directory" エラー
  - プラグインキャッシュクリーンアップがインストールメタデータ不在時にアクティブプラグインバージョンを削除する問題
  - `/plugin` browse ペインで新規公開プラグインが "0 installs" 表示
  - プラグインアドバイザリが `plugin.json` のデフォルトフォルダを shadow するキーを全て表示しない問題
- **改善**:
  - リアクティブコンパクション: 初回 summarize がオリジナルリクエストのオーバーフローサイズから始まり、ニアフルコンテキストでの無駄なリトライを回避
  - Hook 設定エラー改善: `SessionStart`/`Setup`/`SubagentStart` に prompt/agent タイプの hook を設定すると "use a command-type hook instead" エラーを明示
- 利用ポリシー違反メッセージから古い `/model claude-sonnet-4-20250514` 提案を削除

## v2.1.141 (2026-05-13)

- **Hook JSON 出力に `terminalSequence` フィールド追加**: 制御端末なしでデスクトップ通知・ウィンドウタイトル・ベル等の端末シーケンスを発行可能に（バックグラウンド hooks 等で活用）
- **`CLAUDE_CODE_PLUGIN_PREFER_HTTPS` 環境変数追加**: GitHub プラグインソースを SSH ではなく HTTPS でクローン。SSH 鍵未設定環境向け
- **`ANTHROPIC_WORKSPACE_ID` 環境変数追加**: workload identity federation で発行トークンを特定ワークスペースにスコープ。federation ルールが複数ワークスペースをカバーする場合に有効
- **`claude agents --cwd <path>` 追加**: セッションリストをディレクトリでスコープ
- **`/feedback` 拡張**: 直近 24 時間または 7 日間のセッションを含められるように（現セッションを跨ぐ問題向け）
- **Rewind メニュー「Summarize up to here」追加**: 直近ターンを保持しつつ過去コンテキストを圧縮
- **Auto モード権限ダイアログ改善**: `permissions.ask` ルールが原因でプロンプトが出た場合に理由を明示
- **IDE 連携復元**: ファイル編集 permission プロンプトの「IDE で diff を見る」オプションが復活
- **バックグラウンドエージェント**: `/bg` / `←←` で起動したバックグラウンドエージェントが現在の permission mode を維持（従来は default に戻っていた）
- **`claude agents` 改善**: バックグラウンドシェルが残っているエージェントが Working ではなく Completed に移動
- **スピナーフィードバック改善**: 長い思考期間中、10 秒後にスピナーがアンバー色に温まり継続を示唆
- **重要バグ修正**:
  - Bedrock/Vertex/Foundry/gateway で `ANTHROPIC_SMALL_FAST_MODEL` 未設定時にバックグラウンドサイドクエリが Haiku ID 送信で失敗する問題（メインループモデルにフォールバック）
  - Windows で `claude daemon status` / `/doctor` がデーモンパイプキーファイルロック時に opaque failure する問題
  - `EnterWorktree` で作業ディレクトリ切り替え後、hooks が存在しない `transcript_path` を受け取る問題
  - `/model` 変更が他並列セッションの autocompact 閾値を黙って変更する問題
  - 権限プロンプト中のモード変更が新設定で許可されたツールでもプロンプトを自動 dismiss しない問題
  - 権限プロンプト中の Enter が入力ボックスにも送信される問題
  - markdown テーブルのセルラッピング時に縦型 key-value フォールバックする問題（v2.1.136 リグレッション）
  - vim INSERT/VISUAL モード中の Ctrl+C が turn を中断しない問題
  - 代替 `chat:submit` キーバインド（`meta+enter` 等）が `enter` 再バインド時に効かない問題
  - `spinnerVerbs` 設定が turn-completion メッセージで尊重されない問題
  - light-ansi テーマで diff コンテキスト行が白で不可視になる問題
  - Bedrock の `awsCredentialExport` が ambient AWS credentials 解決時にスキップされ、クロスアカウントアクセス auth が失敗する問題
  - SDK で Linux glibc/musl 両プラットフォームパッケージインストール時に "native binary not found" エラー
  - MCP 関連: HTTP/SSE サーバー 403 が "failed" 表示（"needs auth" が正しい）、POSIX shell パラメータ展開（`${var%pattern}`）が未設定変数として誤検出、`.mcp.json` 経由の MCP サーバーが 0 表示
  - Remote Control: 401 後のループ再 enroll を停止、worker session token rotation 時の 401 修正
- VSCode 拡張: マイクが silence しか拾わない時の "No audio detected" 表示、WSLg 向け `sox libsox-fmt-pulse` 案内
- daemon: `←` から残った空 idle バックグラウンドセッションを 5 分後に自動退避

## v2.1.140 (2026-05-12)

- **Agent ツール `subagent_type` マッチが大文字小文字・区切り文字非依存に**: `"Code Reviewer"` が `code-reviewer` に正しく解決されるようになり、メイン会話とサブエージェント仕様間の表記揺れを許容
- **`/goal` 改善**: `disableAllHooks` / `allowManagedHooksOnly` が有効な時にハングする問題を修正し、明確なメッセージを表示
- **設定ホットリロード**: シンボリックリンクされた設定ファイル編集の変更イベント誤帰属を修正（spurious な `ConfigChange` hook 発火を抑制）
- **`claude --bg` 安定化**: "connection dropped mid-request" で失敗する問題を修正。エンタープライズエンドポイントセキュリティ環境での起動猶予を延長
- **リモート管理設定**: 401 応答時にトークン強制リフレッシュで一度リトライ
- **Managed Settings 永続化**: `extraKnownMarketplaces` の auto-update ポリシーを `known_marketplaces.json` に永続化
- **プラグイン警告**: `plugin.json` 設定でデフォルトのコンポーネントフォルダ（skills/ 等）が暗黙的に無視される際に警告を出力
- **Loop スケジューリング最適化**: バックグラウンドタスクの冗長な wakeup を削減
- **Read ツール**: `offset` が空白パディング・`+` 接頭辞付き文字列の場合のバリデーション失敗を修正
- **Windows イベントループ**: 不在実行ファイルチェックが同期 `where.exe` 再起動を発火させて停止する問題を修正
- **エージェントカラーパレット更新**
- ターミナルカーソル位置・プラグイン更新・サブエージェントカラー等のマイナーUI修正

## v2.1.139 (2026-05-11)

- **Agent View (Research Preview) 追加**: `claude agents` で実行中・ユーザー入力待ち・完了済みの全 Claude Code セッションを一画面で確認。公式: https://code.claude.com/docs/en/agent-view
- **`/goal` コマンド追加**: 完了条件を設定すると、達成まで複数ターンに渡り Claude が継続動作。インタラクティブ / `-p` / Remote Control で利用可。経過時間・ターン数・消費トークンをライブオーバーレイ表示
- **`/scroll-speed` コマンド追加**: マウスホイールスクロール速度をライブプレビュー付きで調整
- **`claude plugin details <name>` 追加**: プラグインのコンポーネント一覧とセッション毎の予測トークンコストを表示
- **Transcript View ナビゲーション拡張**: `?` でショートカット一覧、`{`/`}` でユーザープロンプト間ジャンプ、`v` でパネルトグル
- **Hook `args: string[]` 配列フォーム（exec form）追加**: shell 解釈なしで直接コマンドを起動。パスプレースホルダーのクォート不要に
- **`PostToolUse` Hook `continueOnBlock` オプション追加**: 拒否理由を Claude に戻してターン継続（従来はブロック）
- **MCP stdio サーバーに `CLAUDE_PROJECT_DIR` を環境変数として渡す**: hooks と同じ挙動。プラグイン設定の commands で `${CLAUDE_PROJECT_DIR}` を参照可能に
- **Compaction プロンプト強化**: 会話圧縮中にユーザーのセンシティブな指示を保持するようモデルへ依頼
- **`/mcp` 再接続改善**: 再起動なしで `.mcp.json` 編集を取り込み、再接続失敗時に HTTP ステータスと URL を表示
- **`/context all` トークン推定**: スキル単位の推定値がモデルのトークナイザーを考慮し、丸めて表示
- **`claude plugin install <name>@<marketplace>` 強化**: マーケットプレース自動リフレッシュとリトライを行い、見つからないと報告する前に再試行
- **API キー認証時の機能制限**: `ANTHROPIC_API_KEY` / `apiKeyHelper` / `ANTHROPIC_AUTH_TOKEN` 設定時、Remote Control・`/schedule`・claude.ai MCP コネクタ・通知設定を無効化（Claude.ai ログイン併存でも API キーを優先）
- **設定ホットリロード**: シンボリックリンクされた `~/.claude/settings.json` の編集を検出可能に
- **Skill 権限ワイルドカード修正**: `Skill(name *)` がプレフィックス一致として正しく動作（`Bash(ls *)` と同じ挙動）
- **OTEL 拡張**: サブエージェントの API リクエストに `x-claude-code-agent-id` / `x-claude-code-parent-agent-id` ヘッダーを追加。`claude_code.llm_request` スパンに `agent_id` / `parent_agent_id` 属性を追加
- **VSCode 拡張**: Cmd/Ctrl+Shift+T で直近クローズしたセッションタブを再オープン（`claudeCode.enableReopenClosedSessionShortcut` で設定）
- **重要バグ修正**:
  - 期限切れクレデンシャル × `forceRemoteSettingsRefresh` ポリシーで `claude auth login/logout/status` がデッドロックする問題
  - `autoAllowBashIfSandboxed` が `$VAR`/`$(cmd)` を含むコマンドを自動承認しない問題
  - Hook が端末書き込みでインタラクティブプロンプトを破壊する問題（hooks は端末アクセスなしで実行に変更）
  - HTTP/SSE MCP サーバーがプロトコル外データをストリームした際の無制限メモリ増（SSE フレームあたり 16MB に制限）
  - `/model` ピッカーの "Default" 行が `ANTHROPIC_DEFAULT_OPUS_MODEL` / `ANTHROPIC_DEFAULT_SONNET_MODEL` オーバーライドを反映しない問題
  - ストリーム終了 5 分後に発生する偽の "stream idle timeout" エラー
  - MCP サーバー 10 個以上かつキャッシュディレクトリ書き込み不可時の silent `exit 1`
  - 複数画像のペースト/ドロップで最後の 1 つしか挿入されない問題
  - ダークテーマでハイパーリンクが読めない濃紺になる問題
  - Cursor / VS Code 1.92–1.104 でマウスホイールスクロール速度が乱れる問題
  - 切断済み MCP サーバーのリソースが `@server:` オートコンプリートに残る問題
  - Grep 結果が Windows ドライブレターパスを相対化しない問題、count モードが単一ファイルパスで誤った総数を報告する問題
  - スキル引数名に正規表現メタ文字を含むと引数置換が壊れる問題
  - `--print` モードで `claude_code.active_time.total` OTEL メトリックが発信されない問題
  - `claude plugin update` がマーケットプレース内のクロスプラグインシンボリックリンクを保持しない問題

## v2.1.138 (2026-05-09)

- 内部修正のみ

## v2.1.137 (2026-05-09)

- [VSCode] Windows で拡張がアクティベートに失敗する問題を修正

## v2.1.136 (2026-05-08)

- **`CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` 環境変数追加**: OpenTelemetry 経由で応答収集する企業向けに、セッション品質サーベイを再有効化
- **`settings.autoMode.hard_deny` 追加**: Auto Mode の分類器ルールで、ユーザー意図や allow 例外に関わらず無条件にマッチアクションをブロックする規則を定義可能。広範な allow があっても自動実行すべきでないアクション向け
- スラッシュコマンドダイアログのフッターヒント、ダイアログ間隔、矢印キースタイルを統一して視覚的一貫性を改善
- 新しい `--worktree` が既存/陳腐化したワークツリーと衝突した時のエラーメッセージを改善
- プラグインマーケットプレース削除キーを `r`（リトライと衝突）から他箇所と一致する `d` に変更
- **重要バグ修正**:
  - `.mcp.json`、プラグイン、claude.ai connector で構成された MCP サーバーが VS Code 拡張・JetBrains プラグイン・Agent SDK で `/clear` 後に静かに消失する問題
  - 並列クレデンシャル書き込みが新規ローテートされた OAuth トークンを上書きし再ログインを強制する稀なログインループ
  - 複数 MCP サーバーが同時にリフレッシュした際、MCP OAuth リフレッシュトークンが消失する問題
  - 拡張思考がツール呼び出し後に redacted thinking ブロックを発行した場合の API エラー (400)
  - プロジェクトパスにアンダースコアを含む場合、`--resume` / `--continue` がセッションを発見できない問題
  - マッチする `Edit(...)` allow ルールが存在する時、プランモードが書き込みをブロックしない問題
  - WSL2: xclip/wl-paste が画像データを読めない場合に PowerShell フォールバックで Windows クリップボードからの画像ペーストが動作するように
  - キャッシュクリーンアップが実行中セッションが使用するバージョンを削除した時、プラグインの `Stop`/`UserPromptSubmit` フックが失敗する問題
  - ツールエラー truncation マーカーが surrogate-pair 文字列で負のカウントを表示する問題
  - SessionStart フックの `CLAUDE_ENV_FILE` から取得した env vars が `/resume` や `/clear` 後に陳腐化する問題
  - `plugin.json` の `skills` エントリがプラグインのデフォルト `skills/` ディレクトリを隠す問題
  - `AskUserQuestion` が複数選択回答を配列で受けた時にドロップする問題
  - `CronList` 出力に修飾子とスケジュールされたプロンプトが欠落する問題
  - `@` ファイルピッカーがディレクトリ内 100 件超エントリでファイルを発見できない問題、およびセッション中盤に作成されたファイルを小規模 non-git ディレクトリでマッチしない問題
  - サブエージェントの `Skill` ツール経由のスキル発見不全（v2.1.133 で部分修正の続報）
  - `keybindings.json` でリバインドしたキーがキーボードショートカットヒントに反映されない問題
  - フルスクリーンモードでツールエラー出力が truncate された場合、失敗したツール呼び出しが click-to-expand されない問題
  - Indic 連結子・ZWJ 絵文字・CJK 端末でのカラム計算・bracketed paste・wide markdown table 等のレンダリング不整合
  - `/insights` がツール呼び出しに malformed input を含むセッション履歴でクラッシュする問題

## v2.1.133 (2026-05-07)

- **`worktree.baseRef` 設定追加** (`fresh` | `head`): `--worktree`、`EnterWorktree`、エージェント隔離ワークツリーが `origin/<default>` から切るか、ローカル `HEAD` から切るかを選択可能に
  - **重要**: デフォルト `fresh` により `EnterWorktree` のベースが `origin/<default>` に戻る（v2.1.128 以降は `HEAD` だった）。未push コミットを新ワークツリーに引き継ぎたい場合は `worktree.baseRef: "head"` を設定
- **`sandbox.bwrapPath` / `sandbox.socatPath` Managed 設定追加**（Linux/WSL）: bubblewrap と socat バイナリのカスタムパス指定
- **`parentSettingsBehavior` 設定（admin tier）追加** (`'first-wins' | 'merge'`): 管理者が SDK `managedSettings`（parent tier）をポリシーマージ対象に含めるかを制御
- **Hooks で effort level 取得可能に**: JSON 入力に `effort.level` フィールドが追加され、`$CLAUDE_EFFORT` 環境変数も hooks と Bash ツールサブプロセスから参照可能
- フォーカスモードの挙動改善
- メモリ圧迫時に warm-spare バックグラウンドワーカーを解放してメモリ使用量を改善
- **重要バグ修正**:
  - リフレッシュトークン競合で共有資格情報が消去され、並列セッションが全て 401 で死ぬ問題
  - ドライブルート（`C:\`）/ POSIX `/` をスコープにした `Edit`/`Write` 許可ルールが誤マッチして毎回プロンプトが出る問題
  - 履歴/セッションログのファイルロックがクロックスキューや遅いディスクで侵害された際の uncaught rejection (`ECOMPROMISED`)
  - 会話 compaction 中に Esc を押すと「Error compacting conversation」誤通知が出る問題
  - MCP OAuth フロー全体（discovery、dynamic client registration、token exchange、token refresh）で `HTTP(S)_PROXY` / `NO_PROXY` / mTLS が尊重されない問題
  - `--add-dir` / SDK `additionalDirectories` で渡したマップ済みネットワークドライブで Read/Write/Edit が拒否される問題
  - claude.ai からの Remote Control 停止/中断がローカル Esc と同じ完全なキャンセルにならず、スタックしたツール/プロンプトを中断後にキューメッセージが進まない問題
  - 単一セッションの `/effort` 変更が他の並列セッションに波及する問題、および IDE からの effort 変更が無音でドロップされる関連問題
  - サブエージェントが Skill ツール経由でプロジェクト/ユーザー/プラグインスキルを発見できない問題
  - VSCode 拡張: 拡張ビルドが Claude バイナリをバンドルしない場合に `claudeCode.claudeProcessWrapper` が「Unsupported platform」で失敗する問題
- `claude --help` に `--remote-control` を表示（既存の `--remote-control-session-name-prefix` と並列）

## v2.1.132 (2026-05-06)

- **`CLAUDE_CODE_SESSION_ID` 環境変数を Bash ツールサブプロセスに追加**: hooks に渡される `session_id` と一致
- **`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1` 環境変数追加**: フルスクリーン alternate-screen レンダラーをオプトアウトし、会話を端末のネイティブスクロールバックに残す
- **Ctrl+V 画像ペースト中に「Pasting…」フッターヒント**: クリップボードから読み取り中の状態表示
- **重要バグ修正**:
  - 外部 SIGINT（IDE 停止ボタン、`kill -INT`）で graceful shutdown が実行されない問題（端末モード復元と `--resume` ヒント表示が動作）
  - ネイティブビルドでセッション中に端末を閉じる/SSH 切断時の uncaught exception
  - `--resume` がツールエラー truncation で絵文字が分割された際 `no low surrogate in string` で失敗する問題（破損済みセッションは読込時に sanitize）
  - `-p --continue`/`--resume` でプランモードセッション再開時に `--permission-mode` フラグが無視される問題、および同一セッション内 `ExitPlanMode` 後にプランモードが再適用されない問題
  - フルスクリーンモードでラップトップスリープ復帰や Ctrl+Z/`fg` 後に次のキー入力/ストリーム出力まで画面が空白化する問題
  - Indic 連結子や ZWJ 絵文字が行を跨いでラップする際、Ctrl+E/A/K/U/矢印キーでカーソルが grapheme 中間に着地する問題
  - vim オペレーターが decomposed (NFD) アクセント文字を含むテキストを破損する問題
  - `/` で始まるテキストペーストで入力が無音で飲まれる/未知コマンド応答が発火する問題
  - フォーカスイベント/マウストラッキングレポートが bracketed paste に混ざるとプロンプトに stray escape sequence がダンプされる問題
  - Cursor / VS Code 1.92–1.104 でマウスホイールスクロールが速すぎる問題（上流 xterm.js バグ）
  - JetBrains IDE 2025.2 端末でスクロールホイール処理（誤った矢印キー、逆方向、暴走加速）の問題
  - `/usage` の Ctrl+S スクリーンショットコピーが Linux/X11 でハングする問題
  - `/terminal-setup` が Windows Terminal で矛盾するエラーを表示する問題（Shift+Enter はネイティブサポート）
  - `/effort` ピッカーが `CLAUDE_CODE_EFFORT_LEVEL` 環境変数オーバーライドを反映しない問題
  - `/status` が一部ユーザーで誤ったデフォルトモデルを表示する問題
  - スラッシュコマンド autocomplete ポップアップが約 3〜5 件で頭打ちになり端末高さに合わせてスケールしない問題
  - ステータスライン `context_window` トークン数が現在のコンテキスト使用量ではなくセッション累計を反映していた問題
  - macOS 端末（iTerm2、Terminal.app デフォルト）で「Option as Meta」未有効時に Alt+T（thinking トグル）が動作しない問題
  - Windows で `claude agents` バックグラウンドセッション再開後にキーボード入力が反応しない問題
  - stdio MCP サーバーが非プロトコルデータを stdout に書き込むと unbounded メモリ成長（10GB+ RSS）する問題
  - 接続成功するが `tools/list` で失敗する MCP サーバーが無音で 0 ツール表示される問題（1 回リトライ後 `/mcp` で「connected · tools fetch failed」表示）
  - 認証されていない claude.ai MCP コネクタが「failed」と表示される問題（「needs auth」表示に修正）、および headless `-p` モードが非一時的 4xx 接続失敗をリトライする問題
  - Bedrock / Vertex で `ENABLE_PROMPT_CACHING_1H` 設定時に 400 エラーが発生する問題

## v2.1.131 (2026-05-06)

- **VS Code 拡張: Windows でアクティブ化失敗を修正**: バンドル SDK のハードコードされたビルドパス（`createRequire` polyfill バグ）が原因
- **Mantle エンドポイント認証修正**: `x-api-key` ヘッダ欠落の問題を修正

## v2.1.129 (2026-05-06)

- **`--plugin-url <url>` 追加**: URL から `.zip` プラグインアーカイブを取得して当該セッションに読み込む
- **`CLAUDE_CODE_FORCE_SYNC_OUTPUT=1` 追加**: 自動検出が外す端末（例: Emacs `eat`）で同期出力を強制有効化
- **`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` 追加**: Homebrew/WinGet インストールでバックグラウンドアップグレード→再起動プロンプトを実行
- **プラグインマニフェスト**: `themes` と `monitors` は `"experimental": { ... }` 配下宣言が推奨に変更。トップレベル宣言は `claude plugin validate` で警告
- **ゲートウェイ `/v1/models` 探索がオプトイン化**: `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1` で有効化（v2.1.126〜v2.1.128 は自動）
- **`Ctrl+R` 履歴ピッカーのデフォルトを全プロジェクト横断検索に戻す**: pre-2.1.124 の挙動。`Ctrl+S` で現プロジェクト/セッションに絞り込み
- **3P デプロイメントでスピナーチップから 1P 案内を除去**: Bedrock/Vertex/Foundry/`ANTHROPIC_BASE_URL` ゲートウェイ利用時
- **`skillOverrides` 設定が機能するように修正**: `off` でモデルと `/` から非表示、`user-invocable-only` でモデルから非表示、`name-only` で説明を圧縮
- **`claude_code.pull_request.count` OTel メトリック**: シェルコマンドだけでなく MCP ツール経由 PR/MR 作成もカウント
- **ポリシー拒否エラーメッセージに API Request ID 付与**: サポートデバッグ容易化
- **重要バグ修正**:
  - 認識できない 400 ステータスコードで生 JSON が表示される問題（実エラーメッセージを表示）
  - `/clear` 後に端末タブタイトルがリセットされない問題
  - `/rename` のセッションタイトルチップが権限ダイアログ表示中に消える問題
  - サブエージェント実行中にプロンプト下のエージェントパネルが隠れる問題（v2.1.122 リグレッション）
  - 外部エディタハンドオフ（Ctrl+G）でプロンプト上の会話履歴が空白化する問題
  - `/context` がレンダリング ASCII グリッドを会話に投入し約 1.6k トークン浪費する問題
  - `/agents` Library のリスト矢印キー操作で選択項目がビューポート外に出る問題
  - `/branch` 成功メッセージに新ブランチセッション ID（`/resume` 用）が含まれていない問題
  - 太字ヘッダー内の keycap/ZWJ/skin-tone 絵文字が末尾文字を欠落する問題（フルスクリーンモード）
  - エンタープライズ/チームユーザーの保存 OAuth 資格情報に `user:inference` スコープが欠落していると server-managed settings ポリシーが適用されない問題
  - スリープ復帰後の OAuth リフレッシュ競合で全実行中セッションがログアウトされ得る問題
  - 1 時間プロンプトキャッシュ TTL が黙って 5 分にダウングレードされる問題
  - `/clear` やコンパクション後に `/effort`/`/model` を変更するとキャッシュミス警告が誤発出する問題
  - `Bash(mkdir *)`、`Bash(touch *)` 等の許可ルールがプロジェクト内パスで尊重されない問題
  - `deniedMcpServers` の `*://` スキームワイルドカードが大文字小文字混在ホスト名にマッチしない問題
  - voice モード `--debug` で無害な WebSocket 警告がエラーログ化する問題
  - [VSCode] `/clear` が会話コンテキストとトランスクリプト表示をクリアしない問題

## v2.1.128 (2026-05-04)

- **`/mcp` がツール数を表示**: 接続済みサーバーのツール数を表示し、0 ツールで接続したサーバーを警告マーク
- **`--plugin-dir` が `.zip` 受理**: ディレクトリに加えて zip プラグインアーカイブも読み込み可能
- **`--channels` が console (API キー) 認証で利用可能に**: 管理設定を持つ console org は `channelsEnabled: true` で有効化
- **`/model` ピッカー整理**: Opus 4.7 重複エントリを統合、現行 Opus を「Opus」と表示
- **サブプロセスへの `OTEL_*` 継承を停止**: Bash/hooks/MCP/LSP サブプロセスが CLI の OTLP エンドポイントを誤って継承しないように
- **MCP `workspace` が予約サーバー名に**: 同名の既存サーバーは警告とともにスキップ
- **MCP 再接続時のツール一覧再公開を要約化**: 再接続毎に全ツール名を吐かず、サーバープレフィックスごとに集約
- **SDK ホスト向け `localSettings` サジェスト**: Bash 権限プロンプトで「Always allow」が `.claude/settings.local.json` に書き込まれるよう永続提案
- **`EnterWorktree` が local HEAD 起点に**: ドキュメント通り local HEAD から新ブランチを作成（従来は `origin/<default-branch>` 起点で未 push コミットが落ちていた）
- **オートモード: 分類器エラーにヒント追加**: 評価不能時に retry / `/compact` / `--debug` 起動の提案を表示
- **`/color` 引数なしでランダム色**: セッション色をランダム選択
- **重要バグ修正**:
  - 大入力（>10MB）を `claude -p` に stdin パイプするとクラッシュループする問題
  - 1M コンテキストモデルで autocompact ウィンドウが小さい場合に実 API 制限到達前に「Prompt is too long」で誤ブロックされる問題
  - 並列シェルツール呼び出し: read-only コマンド（grep/git diff/ls）が失敗すると兄弟呼び出しまでキャンセルされる問題
  - サブエージェントの進捗サマリがプロンプトキャッシュを取りこぼし `cache_creation` が約3倍になっていた問題
  - サブエージェントサマリがトランスクリプト静止中も繰り返し発火し、idle サブエージェントの最悪トークンコストが青天井だった問題
  - MCP stdio サーバー: `CLAUDE_CODE_SHELL_PREFIX` 設定時に空白/シェルメタ文字入り引数が破損する問題
  - MCP ツール結果: サーバーが structured content と content blocks の両方を返すと画像が落ちる問題
  - `/plugin update` が npm ソースプラグインの新バージョンを検出しない問題
  - `/plugin` Components パネルが `--plugin-dir` 経由ロードのプラグインで「Marketplace 'inline' not found」を表示する問題
  - `installed_plugins.json` の死んだキャッシュディレクトリエントリが PATH を汚染する問題
  - Bedrock デフォルトモデルがリージョン適切なプレフィックスではなく `global.*` に解決される問題
  - 3P プロバイダーで `/fast` が無関係スキルにファジーマッチする問題（「利用不可」を表示するように）
  - Remote Control: rate limit 時に空の "Opening your options…" 表示（実行可能なアップセル選択肢を表示）
  - 古い「remote-control is active」ステータスラインが `--resume`/`--continue` 後も残る問題
  - Kitty 等の OSC 9 通知解釈ターミナルで `/exit` 毎に "4;0;" デスクトップ通知が出る問題
  - 画像ドラッグ＆ドロップでファイル読み込み失敗時に「Pasting text…」でハングする問題
  - フルスクリーンモードで折り返し表示の長い URL が各行クリック不可だった問題
  - フォーカスモードで新プロンプト送信時に直前応答が一瞬暗くなる問題
  - OSC 8 非対応ターミナルで markdown リンクラベルが失われる問題（`label (url)` 形式で表示）
  - リスト項目内 fenced code block コピー時の先頭空白混入
  - `/config` タブナビゲーションがフォーカスを失う問題
  - vim モード NORMAL: `Space` でカーソル右移動（標準 vi/vim 互換）
  - ターミナル進捗インジケータ（OSC 9;4）がツール呼び出し間で点滅消失する問題
  - `/rename` 引数なしで compact 境界終端の resume セッションが失敗する問題
  - エフォート非対応モデルでバナーに「with X effort」と誤表示する問題
  - Headless `--output-format stream-json`: `init.plugin_errors` に `--plugin-dir` ロード失敗が含まれるように

> 注: v2.1.127 はステーブル未リリース（バージョンスキップ）。v2.1.126 → v2.1.128 へ。

## v2.1.126 (2026-05-01)

- **`/model` ピッカーがゲートウェイの `/v1/models` から取得**: `ANTHROPIC_BASE_URL` が Anthropic 互換ゲートウェイを指す場合、`/model` がそのゲートウェイの利用可能モデル一覧を表示
- **`claude project purge [path]` 追加**: プロジェクトの Claude Code 状態（トランスクリプト、タスク、ファイル履歴、設定エントリ）を一括削除。`--dry-run`、`-y/--yes`、`-i/--interactive`、`--all` オプション対応
- **`--dangerously-skip-permissions` の保護解除拡張**: `.claude/`、`.git/`、`.vscode/`、シェル設定ファイル等の従来保護パスへの書き込みプロンプトを抑制（破滅的削除コマンドは引き続き安全網として確認）
- **`claude auth login` の OAuth コードペースト対応**: ブラウザコールバックが localhost に到達できない環境（WSL2、SSH、コンテナ）で、ターミナルにペーストしたコードを受理
- **OpenTelemetry**: `claude_code.skill_activated` イベントがユーザー入力スラッシュコマンドでも発火するように。新属性 `invocation_trigger`（`"user-slash"` / `"claude-proactive"` / `"nested-skill"`）を追加
- **オートモードのスピナー赤色化**: 権限チェックが停滞しているときスピナーが赤くなり、ツール実行中と区別可能に
- **ホスト管理デプロイ**: `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` 設定時でも Bedrock/Vertex/Foundry のアナリティクスが自動無効化されないように
- **Windows: PowerShell 7 検出範囲拡大**: Microsoft Store からのインストール、PATH 未設定 MSI、`.NET global tool` 経由の PowerShell も検出
- **Windows: PowerShell をプライマリシェルに**: PowerShell ツール有効時、Bash ではなく PowerShell が主シェルとして扱われる
- **Read ツール**: ファイル毎のマルウェア評価リマインダーを削除（誤拒否や「これはマルウェアではない」コメンタリーの原因を解消）
- **重要セキュリティ修正**: `allowManagedDomainsOnly` / `allowManagedReadPathsOnly` が、優先度の高い managed-settings ソースに `sandbox` ブロックがないとき無視される問題
- **重要バグ修正**:
  - 2000px 超の画像ペーストでセッションが破壊される問題（ペースト時に縮小、履歴内の超過画像は自動削除しリトライ）
  - 「OAuth not allowed for organization」エラーで誤ってログイン画面が表示される問題（管理者連絡を案内）
  - 低速/プロキシ接続、IPv6-only devcontainer、ブラウザコールバックが localhost に到達不能なケースでの OAuth ログインタイムアウト
  - 並行する認証情報書き込みが有効な OAuth refresh token を稀に消失させる競合
  - API リトライカウントダウンが「0s」で固まる問題
  - Mac スリープ復帰直後の「Stream idle timeout」エラー
  - 長時間モデル思考中にバックグラウンド/リモートセッションが「Stream idle timeout」で誤って中止される問題
  - 空ターン連発後にアシスタントが思考完了しても出力が表示されないハング
  - Cursor / VS Code 1.92–1.104 統合ターミナルでのトラックパッド過剰スクロール
  - needs-auth で停滞した手動 MCP サーバーが claude.ai MCP コネクタを抑制する問題
  - Windows no-flicker モードで日本語/韓国語/中国語が文字化けする問題
  - `Ctrl+L` がプロンプト入力をクリアしてしまう問題（readline 同様、画面再描画のみに）
  - `context: fork` スキルや他サブエージェントの初回ターンで deferred ツール（WebSearch、WebFetch 等）が利用不可
  - `--channels` 起動の対話セッションでプランモードツールが利用不可
  - `/plugin` Uninstall 後に "Enabled" と誤表示される問題
  - リンタが多数ファイルを変更したときのファイル変更リマインダーの総サイズ制限
  - `/remote-control` リトライが「connecting…」で停滞表示される問題（各リトライ結果を表示）
  - リモートコントロール初期接続失敗時に通知にエラー理由が含まれない問題
  - Windows: クリップボード書き込みでコピー内容がプロセスコマンドライン引数経由で EDR/SIEM テレメトリに露出する問題（22KB 超の選択もクリップボードに到達）
  - PowerShell ツール: 単独 `--`（例 `git diff -- file`）が `--%` パース停止トークンと誤認される問題
  - Agent SDK: 並列ツール呼び出しバッチでモデルが不正なツール名を出した際のハング

> 注: v2.1.124 / v2.1.125 はステーブル未リリース（バージョンスキップ）。v2.1.123 → v2.1.126 へ。

## v2.1.123 (2026-04-29)

- `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1` 設定時の OAuth 認証 401 リトライループを修正

## v2.1.122 (2026-04-28)

- **`ANTHROPIC_BEDROCK_SERVICE_TIER` 環境変数追加**: Bedrock サービスティア（`default` / `flex` / `priority`）を選択し、`X-Amzn-Bedrock-Service-Tier` ヘッダで送信
- **`/resume` 検索ボックスに PR URL ペースト対応**: その PR を作成したセッションを検索（GitHub / GitHub Enterprise / GitLab / Bitbucket）
- **`/mcp`**: 同一 URL の手動追加サーバーで隠れた claude.ai connector を表示し、重複削除のヒントを出す
- **OAuth サインイン後も未認可の MCP サーバー**に表示するメッセージを明確化
- **OpenTelemetry**: `api_request` / `api_error` の数値属性が文字列ではなく数値として送出されるように
- **OpenTelemetry**: `@`-mention 解決用に `claude_code.at_mention` ログイベントを追加
- **重要バグ修正**:
  - rewound タイムラインのエントリを含むセッションから `/branch` でフォークすると `tool_use ids were found without tool_result blocks` で失敗する問題
  - Bedrock application inference profile ARN で `/model` の Effort オプションが表示されない、`output_config.effort` が送られない問題
  - Vertex AI / Bedrock のセッションタイトル生成等の構造化出力で `invalid_request_error: output_config: Extra inputs are not permitted` が返る問題
  - プロキシゲートウェイ越しの Vertex AI `count_tokens` エンドポイントで 400 エラー
  - `spinnerTipsOverride.excludeDefault` で時間ベースのスピナーチップが抑制されない問題
  - nonblocking モードでセッション開始後に接続した MCP サーバーのツールを ToolSearch が見つけられない問題
  - bash モードでの `!exit` / `!quit` が CLI を終了させてしまう問題（シェルコマンドとして実行されるよう修正）
  - 新モデル送信画像が 2576px ではなく正しい 2000px 上限にリサイズされるように修正
  - Remote Control セッションの idle status が秒2回再描画され `tmux -CC` 制御パイプを溢れさせる問題
  - 古いビュー設定によりアシスタントメッセージが空表示になる問題
  - `settings.json` 内の hooks エントリが不正な形式でも全体が無効化されないように
  - Voice mode: Caps Lock にバインドしたキーバインドはターミナルがキーイベントを送らないためエラー表示

## v2.1.121 (2026-04-28)

- **MCP `alwaysLoad` オプション**: `true` を指定するとそのサーバーのツールが tool-search のディファード化対象から外れ常時ロードされる
- **`claude plugin prune` 追加**: 孤立した自動インストール済みプラグイン依存を削除。`plugin uninstall --prune` でカスケード削除
- **`/skills` に type-to-filter 検索ボックス**: 長いスキル一覧をスクロールせず即座に絞り込み可能
- **`PostToolUse` フックが全ツールの出力を置換可能に**: `hookSpecificOutput.updatedToolOutput`（従来は MCP 限定）
- **フルスクリーンモード**: スクロール上方で読書中のプロンプト入力で最下部へ戻らないように
- **オーバーフローダイアログがスクロール可能に**: 矢印・PgUp/PgDn・Home/End・マウスホイール対応（フル/非フル両対応）
- **SDK / `claude -p`**: `CLAUDE_CODE_FORK_SUBAGENT=1` が非インタラクティブセッションでも有効
- **`--dangerously-skip-permissions`**: `.claude/skills/`, `.claude/agents/`, `.claude/commands/` への書き込みでプロンプトを出さない
- **`/terminal-setup`**: iTerm2 の "Applications in terminal may access clipboard" を有効化（tmux 経由の `/copy` 対応）
- **MCP サーバー起動時の transient error は最大3回自動リトライ**（従来は接続失敗で停止）
- **ターミナルタブのセッションタイトルが `language` 設定に従って生成される**
- **Claude.ai connector の同一上流URLは重複排除**（重複表示の解消）
- **Vertex AI**: X.509 証明書ベースの Workload Identity Federation（mTLS ADC）対応
- 起動高速化: リリースノートスプラッシュから Recent Activity パネル削除
- LSP 診断サマリーがクリック / Ctrl+O で展開可能、展開ヒントを表示
- **SDK**: `mcp_authenticate` が `redirectUri` 対応（カスタムスキーム / claude.ai connector）
- **OpenTelemetry**: LLM リクエストスパンに `stop_reason`、`gen_ai.response.finish_reasons`、`user_system_prompt`（`OTEL_LOG_USER_PROMPTS` 有効時のみ）追加
- VSCode: 音声ディクテーションが Claude Code 言語未設定時 `accessibility.voice.speechLanguage` を尊重
- VSCode: `/context` がネイティブのトークン使用量ダイアログを開く
- **重要バグ修正**:
  - 多数の画像処理時の RSS 数 GB のメモリ無制限増大
  - 大規模トランスクリプト履歴での `/usage` の最大 ~2GB メモリリーク
  - 進捗イベントを発行しない長時間ツールでのメモリリーク
  - セッション中に開始ディレクトリが削除/移動された場合の Bash ツール永続不能
  - 外部ビルドでの `--resume` 起動時クラッシュ
  - 大規模セッションで unclean shutdown による破損行のスキップ機能（従来は `--resume` 失敗）
  - Bedrock application inference profile ARN での `thinking.type.enabled is not supported` エラー
  - Microsoft 365 MCP OAuth の重複/未対応 `prompt` パラメータ問題
  - tmux/GNOME Terminal/Windows Terminal/Konsole で Ctrl+L 時のスクロールバック重複
  - 起動時 connector-list fetch のtransient auth error で claude.ai MCP connector が消失
  - リモートセッションのビルトインツール "Always allow" がワーカー再起動で失われる
  - native build で `managed-settings.json` 経由 `NO_PROXY` が一部 HTTP クライアントで尊重されない
  - managed settings 承認プロンプトが受諾でもセッション終了する問題
  - stale OAuth token 後の `/usage` rate limit エラー（自動リフレッシュに）
  - レガシーenum値 1 つで `settings.json` 全体が無効化される問題
  - no-flicker 無効時の `/usage` ダイアログのクリッピング
  - フルスクリーンレンダラー無効時の `/focus` "Unknown command"（有効化方法を案内）
  - 実行中バイナリ削除中の grep/find/rg ラッパー失敗（インストール済みツールへフォールバック）
- 大規模ディレクトリツリーでの `find` のピーク FD 使用量削減

## v2.1.120 (2026-04-28)

- **Windows: Git Bash 不要に**: 未インストール時は PowerShell をシェルツールにフォールバック
- **`claude ultrareview [target]` 非インタラクティブサブコマンド**: CI / スクリプトから `/ultrareview` を実行。stdout に findings 出力（`--json` で raw）、終了コード 0/1
- **スキルが `${CLAUDE_EFFORT}` 参照可能**: スキル本文で現在の effort level を埋め込める
- **`AI_AGENT` 環境変数をサブプロセスに伝播**: `gh` などが Claude Code トラフィックを識別可能に
- スピナーのおすすめ表示はインストール済み機能（デスクトップアプリ・スキル・エージェント）に対しては非表示
- 矢印キーがスクロールイベント未発行のターミナルで「PgUp/PgDn でスクロール」ヒントを表示
- 多数の claude.ai connector を未認可で持つ場合のセッション開始高速化
- auto モードの拒否メッセージが設定ドキュメントへリンク
- **`claude plugin validate`**: `marketplace.json` トップレベルの `$schema`/`version`/`description` と `plugin.json` の `$schema` を受理
- auto モードの auto-compact 表示を `auto`（小文字、トークン数なし）に変更（誤解を招くトークン値の表示廃止）
- VSCode: `/usage` がネイティブの Account & Usage ダイアログを開く
- VSCode: 音声ディクテーションが `~/.claude/settings.json` の `language` 設定を尊重
- **重要バグ修正**:
  - Esc キーで stdio MCP ツール呼び出し中のサーバー接続全体が閉じる回帰（v2.1.105 起因）
  - `claude --resume` 起動後の `/rewind` など対話オーバーレイがキー入力に反応しない問題
  - 非フルスクリーンモードでのターミナルスクロールバック重複（リサイズ・ダイアログ・長セッション）
  - `DISABLE_TELEMETRY` / `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` が API/Enterprise ユーザーで使用量メトリクステレメトリを抑制しない問題
  - auto モードでパイプ＋リダイレクト含むマルチラインbashコマンドの "Dangerous rm operation" 誤検知
  - フルスクリーンモードで長い選択メニューがターミナル下部にクリップされる問題（フォーカス項目を画面に保持）
  - `find` ツールが大規模ディレクトリツリーで FD 枯渇しホスト全体クラッシュ（macOS/Linux ネイティブビルド）

## v2.1.119 (2026-04-23)

- **`/config` 設定の永続化**: theme、editor mode、verbose 等が `~/.claude/settings.json` に保存され、project/local/policy のオーバーライド優先順位に従う
- **`prUrlTemplate` 設定追加**: フッターの PR バッジを github.com 以外のカスタムコードレビュー URL に向けられる
- **`CLAUDE_CODE_HIDE_CWD` 環境変数**: 起動ロゴでの作業ディレクトリ表示を隠す
- **`--from-pr` 拡張**: GitLab merge-request、Bitbucket pull-request、GitHub Enterprise PR URL を受け付ける
- **`--print` モードがエージェントの `tools:` / `disallowedTools:` フロントマターを尊重**（インタラクティブモードと一致）
- **`--agent <name>` がビルトインエージェントの `permissionMode` を尊重**
- **PowerShell ツールの permission モード自動承認**: Bash と同じ扱いに
- **Hooks: `PostToolUse` / `PostToolUseFailure` に `duration_ms` 追加**（ツール実行時間。権限プロンプトと PreToolUse フックの時間は除く）
- サブエージェントと SDK MCP サーバーの再設定が並列接続に
- 別プラグインのバージョン制約でピン止めされたプラグインが、最上位の満たす git タグへ自動更新
- **Vim モード**: INSERT 中の Esc がキューされたメッセージを入力に戻さず、再度 Esc で中断
- `owner/repo#N` 省略リンクが github.com 固定ではなく git remote のホストを使用
- **セキュリティ**: `blockedMarketplaces` の `hostPattern` / `pathPattern` が正しく強制適用されるように
- **OpenTelemetry**: `tool_result` / `tool_decision` に `tool_use_id` 追加、`tool_result` に `tool_input_size_bytes` 追加
- ステータスライン stdin JSON に `effort.level` と `thinking.enabled` 追加
- **重要バグ修正**:
  - ネイティブ macOS/Linux ビルドで Bash が permissions で拒否されたときに Glob/Grep ツールが消える問題
  - フルスクリーンモードで上スクロール中にツール完了毎に最下部にスナップ戻りする問題
  - 非 JSON OAuth discovery レスポンスによる MCP HTTP 接続の "Invalid OAuth error response" 失敗
  - `async PostToolUse` フックが応答ペイロード無しの時にセッショントランスクリプトへ空エントリを書き込む問題
  - auto モードがプランモードを "Execute immediately" 指示で上書きする問題
  - Vertex AI でのツール検索デフォルト無効化（`ENABLE_TOOL_SEARCH` でオプトイン）
  - HTTP/SSE/WebSocket MCP サーバーの `headers` 内 `${ENV_VAR}` プレースホルダ未置換
  - `/skills` Enter キーがダイアログ閉じる代わりに `/<skill-name>` をプロンプトに pre-fill
  - `Agent` ツール `isolation: "worktree"` が前セッションの stale worktree を再利用
  - `/export` が会話で実際に使ったモデルではなく現在のデフォルトモデルを表示
  - verbose 出力設定が再起動後に永続化されない
  - `/plan` と `/plan open` がプランモード入時に既存プランに作用しない
  - auto-compaction 前に起動されたスキルが次のユーザーメッセージに対して再実行される
  - git worktree で作業中に PR がセッションに紐付かない

## v2.1.118 (2026-04-23)

- **vim ビジュアルモード**: `v`（visual）/`V`（visual-line）で選択、オペレータ適用、ビジュアルフィードバックを備えたビジュアルモード追加
- **`/cost` と `/stats` が `/usage` に統合**: 両コマンドはタイピングショートカットとして残り、対応タブを開く
- **カスタムテーマ機能**: `/theme` から名前付きカスタムテーマの作成・切替可能に。`~/.claude/themes/` で JSON 直接編集も可。プラグインも `themes/` ディレクトリでテーマを提供可能
- **フックが MCP ツールを直接呼び出し可能**: `type: "mcp_tool"` 新設
- **`DISABLE_UPDATES` 環境変数**: 手動 `claude update` 含む全更新パスをブロック（`DISABLE_AUTOUPDATER` より厳格）
- **WSL 管理設定継承**: `wslInheritsWindowsSettings` ポリシーキーで WSL on Windows が Windows 側 managed settings を継承可能
- **Auto mode `"$defaults"`**: `autoMode.allow` / `soft_deny` / `environment` に `"$defaults"` を含めることで組み込みルールを置換せずカスタムルールを追加
- Auto mode オプトインプロンプトに "Don't ask again" オプション追加
- **`claude plugin tag` 追加**: バージョンバリデーション付きでプラグインのリリース git タグを作成
- `--continue` / `--resume` が `/add-dir` で追加されたディレクトリを持つセッションも検索対象に
- `/color` がリモートコントロール接続時にセッションのアクセント色を claude.ai/code に同期
- `/model` ピッカーが `ANTHROPIC_BASE_URL` ゲートウェイ使用時の `ANTHROPIC_DEFAULT_*_MODEL_NAME` / `_DESCRIPTION` オーバーライドを尊重
- プラグイン自動更新が別プラグインのバージョン制約でスキップされた場合、`/doctor` と `/plugin` Errors タブに表示
- **重要バグ修正**:
  - `/mcp` メニューが `headersHelper` 設定サーバーの OAuth Authenticate/Re-authenticate アクションを隠す問題、HTTP/SSE MCP サーバーがカスタムヘッダーで 401 後に "needs authentication" 状態に固まる問題
  - MCP サーバーの OAuth トークンレスポンスが `expires_in` を省略すると毎時再認証を要求される問題
  - macOS キーチェーンレースで並行 MCP トークンリフレッシュが新鮮な OAuth トークンを上書きし "Please run /login" を誘発する問題
  - Linux/Windows でクレデンシャル保存クラッシュが `~/.claude/.credentials.json` を破損させる問題
  - `/login` が `CLAUDE_CODE_OAUTH_TOKEN` 起動セッションで無効化される問題（env トークンをクリアしディスククレデンシャルを有効化）
  - エージェントタイプフックが `Stop` / `SubagentStop` 以外のイベントで "Messages are required for agent hooks" で失敗する問題
  - `/fork` が fork 毎に親会話全体をディスクに書き込む問題（ポインタ方式に変更）
  - リモートセッション接続時に `~/.claude/settings.json` の `model` 設定が上書きされる問題
  - `plugin install` が不正バージョンインストール済み依存の再解決に失敗する問題

## v2.1.117 (2026-04-22)

- **ネイティブビルド Glob/Grep ツール廃止**: macOS/Linux ネイティブビルドで `Glob`/`Grep` ツールが廃止され、組み込み `bfs`/`ugrep` を Bash ツール経由で使用（ツール呼び出しラウンドトリップ削減で高速化）。Windows・npm インストール版は従来通り
- **外部ビルドでのフォークサブエージェント有効化**: `CLAUDE_CODE_FORK_SUBAGENT=1` でサードパーティビルドでもフォークサブエージェント利用可能に
- **エージェント `mcpServers` メインスレッド対応**: フロントマターの `mcpServers:` が `--agent` 経由のメインスレッド起動でも読み込まれる
- **`/model` 永続化**: 選択モデルが再起動後も維持される（プロジェクトが別モデルをピン止めしていても）。起動ヘッダにプロジェクト/managed-settings ピン止めの出典を表示
- **`/resume` 大規模セッション要約**: stale な大規模セッション再読み込み前に要約を提案（`--resume` 既存動作に整合）
- **MCP 起動高速化**: ローカル + claude.ai MCP サーバー両方設定時の並列接続がデフォルトに
- **Pro/Max デフォルト effort 変更**: Opus 4.6 / Sonnet 4.6 で `high` がデフォルトに（`medium` から昇格）
- **プラグイン依存自動解決**: `plugin install` 済みプラグインに対しても不足依存をインストール。`marketplace add` が設定済みマーケットプレースから不足依存を自動解決
- **Managed-settings 強制**: `blockedMarketplaces` と `strictKnownMarketplaces` がプラグイン install/update/refresh/autoupdate でも強制適用
- **Advisor Tool (experimental)**: "experimental" ラベル、learn-more リンク、起動時通知を追加。プロンプト毎に "Advisor tool result content could not be processed" で固まる問題を修正
- **`cleanupPeriodDays` 対象拡張**: `~/.claude/tasks/`、`~/.claude/shell-snapshots/`、`~/.claude/backups/` も保持期間スイープ対象に
- **OpenTelemetry 強化**:
  - `user_prompt` イベントに `command_name` / `command_source` 追加（スラッシュコマンド用）
  - `cost.usage` / `token.usage` / `api_request` / `api_error` に `effort` 属性追加（effort レベル対応モデル）
  - カスタム/MCP コマンド名は `OTEL_LOG_TOOL_DETAILS=1` 設定がない限り redact
- **Windows 起動高速化**: `where.exe` 実行ファイル検索をプロセス毎にキャッシュ
- **重要バグ修正**:
  - Plain-CLI OAuth セッションがアクセストークン期限切れで "Please run /login" で終了する問題を修正（401 で reactive refresh）
  - `WebFetch` が超大規模 HTML ページでハングする問題（HTML→markdown 変換前に truncate）
  - プロキシが HTTP 204 No Content を返す際の `TypeError` クラッシュ
  - `CLAUDE_CODE_OAUTH_TOKEN` で起動後トークン期限切れ時に `/login` が無効化される問題
  - Opus 4.7 セッションの `/context` 値が膨張し早期 autocompact される問題（200K コンテキスト前提の計算を 1M ネイティブに修正）
  - メイン/サブエージェントで異モデル実行時にファイル読み取りが malware 警告される問題
  - Bedrock application-inference-profile が Opus 4.7 + thinking 無効で 400 エラーを返す問題
  - プロンプト入力 undo (`Ctrl+_`) が入力直後に動作せず状態をスキップする問題
  - `NO_PROXY` が Bun 実行時に remote API リクエストで無視される問題
  - バックグラウンドタスク存在時の idle 再描画ループ（Linux でのメモリ増加）
  - MCP `elicitation/create` が print/SDK モードで接続完了と同時に自動キャンセルされる問題

## v2.1.116 (2026-04-20)

- **`/resume` 高速化**: 40MB 超の大規模セッションで最大67%高速化、dead-fork エントリを含むセッションの処理効率改善
- **MCP 起動高速化**: 複数 stdio サーバー設定時の起動が高速化。`resources/templates/list` は最初の `@` メンションまで遅延実行
- **フルスクリーンスクロール改善**: VS Code/Cursor/Windsurf ターミナルでスムーズに。`/terminal-setup` がエディタのスクロール感度を設定
- Thinking スピナーが進捗をインライン表示（"still thinking" → "thinking more" → "almost done thinking"）
- `/config` 検索がオプション値にもマッチ（例: "vim" で Editor mode 設定がヒット）
- `/doctor` が応答中でも起動可能に
- `/reload-plugins` とバックグラウンドプラグイン自動更新が、追加済みマーケットプレースから不足プラグイン依存を自動インストール
- Bash ツールが `gh` コマンドの GitHub API レート制限ヒット時にヒントを表示（エージェントがバックオフ可能）
- Settings の Usage タブが 5時間・週次使用量を即時表示、レート制限時もフェイル回避
- Agent フロントマターの `hooks:` が `--agent` 経由のメインスレッド実行でも発火
- **セキュリティ**: サンドボックス自動許可が `rm`/`rmdir` の `/`, `$HOME` 等重要ディレクトリ対象時の危険パスチェックをバイパスしないよう修正
- Devanagari 等インド系文字のターミナル列整列修正、Kitty キーボードプロトコル下での `Ctrl+-`（undo）・`Cmd+←/→` 修正
- npx/bun run 等ラッパー経由起動時の `Ctrl+Z` ハング修正、インラインモードのスクロールバック重複修正
- `/branch` が 50MB 超トランスクリプトを拒否する問題、`/resume` が大規模セッションで空会話を表示する問題修正
- `/update` と `/tui` が worktree 投入後に動作しない問題修正

## v2.1.114 (2026-04-18)

- エージェントチームのチームメイトがツール権限リクエスト時の権限ダイアログクラッシュを修正

## v2.1.113 (2026-04-17)

- **ネイティブバイナリ化**: bundled JavaScript ではなくネイティブ Claude Code バイナリを直接起動（起動高速化）
- `sandbox.network.deniedDomains` 設定追加（特定ドメインのブロック。allow/deny の併用可）
- フルスクリーンモード改善: `Shift+↑/↓` でスクロール、`Ctrl+A/E` で行頭・行末移動
- Windows: `Ctrl+Backspace` で直前単語削除
- `/loop` 改善: `Esc` で pending wakeup キャンセル
- `/extra-usage` が Remote Control クライアントから利用可能に
- `/ultrareview` 改善: 起動高速化、並列チェック、diffstat 表示
- Bash ツール・権限ルールのセキュリティハードニング
- MCP・UI/UX 各種改善

## v2.1.112 (2026-04-16)

- "claude-opus-4-7 is temporarily unavailable" エラー（Auto mode）修正

## v2.1.111 (2026-04-16)

- **Claude Opus 4.7 xhigh 利用可能** （`/effort` で段階調整）
- Max サブスクライバー向け Auto mode が Opus 4.7 対応
- `/effort` が矢印キーのインタラクティブスライダーに
- "Auto (match terminal)" テーマ追加
- `/less-permission-prompts` スキル追加
- `/ultrareview` コマンド追加（クラウドベースの包括的コードレビュー）
- Windows: PowerShell ツール段階展開（opt-in）
- Auto mode に `--enable-auto-mode` フラグ不要化
- プランファイル名がプロンプト由来に（例: `fix-auth-race-snug-otter.md`）
- グロブ付き読み取り専用 bash コマンドが権限プロンプトをトリガーしないように
- `/skills` メニューにトークン数ソート追加

## v2.1.110 (2026-04-15)

- `/tui` コマンド追加（フリッカーフリー・フルスクリーン描画）
- Remote Control 向け push notification ツール追加
- `Ctrl+O` を normal/verbose トランスクリプトビュー切り替えに変更
- `/focus` コマンド追加
- `/plugin` Installed タブ改善（favorites、注意インジケーター）
- `/doctor` の MCP サーバー警告改善
- `--resume`/`--continue` が未失効のスケジュールタスクを復元
- MCP・権限各種修正

## v2.1.109 (2026-04-15)

- 拡張思考インジケーター改善（回転式プログレスヒント）

## v2.1.108 (2026-04-14)

- `ENABLE_PROMPT_CACHING_1H` 環境変数追加（プロンプトキャッシュTTL 1時間化）
- recap 機能追加（セッション復帰時のおさらい。`/config` で設定可）
- モデルが Skill ツール経由で組み込みスラッシュコマンドを発見・実行可能に
- `/undo` を `/rewind` のエイリアスとして追加
- `/model` がセッション中の切り替え前に警告
- `/resume` ピッカーが現在ディレクトリのセッション優先に
- オンデマンド文法ロードでメモリフットプリント削減

## v2.1.107 (2026-04-14)

- 長時間処理中に thinking ヒントを早めに表示

## v2.1.105 (2026-04-13)

- `EnterWorktree` ツールに `path` パラメータ追加
- **PreCompact フック対応**（コンパクト前のフック実行）
- プラグインのバックグラウンドモニター対応
- `/proactive` を `/loop` のエイリアスとして追加
- API ストリーム停滞処理改善（5分タイムアウト）
- WebFetch 改善（`<style>`・`<script>` タグを除去）
- プラグイン・スキル・MCP 各種修正

## v2.1.101 (2026-04-10)

- `/team-onboarding` コマンド追加
- OS の CA 証明書ストアをデフォルトで信頼
- `/ultraplan` がクラウド環境を自動作成
- brief・focus モード改善
- Bash ツール権限バイパス修正（セキュリティ）
- レート制限リトライメッセージ修正
- 設定回復性改善

## v2.1.98 (2026-04-09)

- Google Vertex AI インタラクティブセットアップウィザード追加
- `CLAUDE_CODE_PERFORCE_MODE` 環境変数追加
- **Monitor ツール追加**（バックグラウンドイベントのストリーミング）
- Linux: サブプロセスサンドボックスで PID namespace 分離
- Bash ツール権限バイパス修正（セキュリティ）
- ストリーミング応答停滞時のフォールバックモード修正

## v2.1.92 (2026-04-04)

- `forceRemoteSettingsRefresh` ポリシー設定追加（リモート設定取得をfail-closed化。取得失敗時にセッション起動をブロック）
- Bedrock インタラクティブセットアップウィザード追加（ログイン画面から直接起動可能）
- `/cost` にモデル別・キャッシュヒット内訳を追加（サブスクリプションユーザー向け）
- `/release-notes` がインタラクティブバージョンピッカーに変更
- Remote Control セッション名のデフォルトプレフィックスがホスト名に変更（`CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` でカスタマイズ）
- Pro ユーザーにプロンプトキャッシュ有効期限のフッターヒント表示
- `Write` ツールの差分計算が大規模ファイルで60%高速化
- `/tag` コマンド削除
- `/vim` コマンド削除（`/config` → Editor mode に統合）
- Linux サンドボックスに `apply-seccomp` ヘルパー追加（unix-socketブロッキング）
- サブエージェント起動時の "Could not determine pane count" エラー修正
- prompt-type Stop フック、ツール入力バリデーション、拡張思考ホワイトスペースAPI 400等のバグ修正
- プラグインMCPサーバーが "connecting" で停止する問題修正

## v2.1.91 (2026-04-02)

- MCP ツール結果の永続化上限をサーバー側から指定可能に（`_meta["anthropic/maxResultSizeChars"]` アノテーション、最大500K。DBスキーマ等の大規模結果の切り詰め防止）
- `disableSkillShellExecution` 設定追加（スキル・カスタムコマンド・プラグインコマンド内のインラインシェル実行 `` !`cmd` `` を無効化）
- `claude-cli://open?q=` ディープリンクで複数行プロンプト対応（`%0A` エンコード改行）
- プラグインが `bin/` 配下に実行ファイルを同梱し、Bash ツールから直接呼び出し可能に
- `--resume` でのトランスクリプトチェーン断裂修正（非同期書き込み失敗時の会話履歴喪失）
- リモートセッションでのプランモード修正（コンテナ再起動後にプランファイルを見失い、権限プロンプトが表示される問題）
- `permissions.defaultMode: "auto"` の JSON スキーマ検証修正
- `/feedback` が利用不可時に理由を表示（メニューから消えるのではなく）
- `/claude-api` スキルのエージェント設計パターンガイダンス改善（ツール選定・コンテキスト管理・キャッシュ戦略）
- Edit ツールが短い `old_string` アンカーを使用し出力トークン削減
- パフォーマンス改善: Bun環境で `stripAnsi` を `Bun.stripANSI` にルーティング

## v2.1.90 (2026-04-01)

- `/powerup` コマンド追加（Claude Code機能のインタラクティブレッスン＋アニメーションデモ）
- `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` 環境変数追加（`git pull` 失敗時にマーケットプレースキャッシュを保持。オフライン環境向け）
- `.husky` を保護ディレクトリに追加（acceptEditsモード）
- レート制限オプションダイアログの無限ループ修正（自動再表示→クラッシュの問題）
- `--resume` のプロンプトキャッシュミス修正（deferred tools/MCP/カスタムエージェント使用時。v2.1.69からのリグレッション）
- Edit/Write ツールの "File content has changed" エラー修正（PostToolUse format-on-save フックによるファイル書き換え時）
- `PreToolUse` フックの exit code 2 + JSON stdout が正しくツールコールをブロックするように修正
- Auto Mode がユーザーの明示的境界（"don't push", "wait for X before Y"）を尊重するように修正
- PowerShell ツールの脆弱性修正（`&` バックグラウンドジョブバイパス、デバッガーハング、アーカイブ展開TOCTOU、パース失敗フォールバック）
- パフォーマンス改善（MCPツールスキーマのper-turn JSON.stringify廃止、SSEトランスポート線形時間化、SDK長会話最適化）
- `/resume` が全プロジェクトセッションを並列ロードに改善
- DNSキャッシュコマンドを自動許可リストから削除（プライバシー）

## v2.1.89 (2026-04-01)

- `PreToolUse` フックに `"defer"` permission decision 追加（ヘッドレスセッションがツールコールで一時停止、`-p --resume` で再評価）
- `PermissionDenied` フックイベント追加（Auto Mode分類器の拒否後に発火。`{retry: true}` で再試行指示可能）
- `CLAUDE_CODE_NO_FLICKER=1` 環境変数追加（フリッカーフリーのalt-screen描画、仮想スクロールバック）
- `MCP_CONNECTION_NONBLOCKING=true` 環境変数（`-p` モードでMCP接続待機スキップ。`--mcp-config` サーバー接続は5秒上限）
- 名前付きサブエージェントが `@` メンションタイプアヘッドに表示
- Auto Mode: 拒否コマンドが通知表示、`/permissions` → Recent タブで `r` キーでリトライ可能
- `Edit` ツールが `Bash` の `sed -n` / `cat` で閲覧したファイルに対しても `Read` 不要で動作
- フック出力50K文字超がディスク保存（ファイルパス+プレビューをコンテキストに注入）
- `cleanupPeriodDays: 0` が検証エラーで拒否されるように変更（以前はトランスクリプト永続化が無効化）
- thinking summaries がインタラクティブセッションでデフォルト非生成に変更（`showThinkingSummaries: true` で復元）
- `Edit(//path/**)`/`Read(//path/**)` の allow ルールがシンボリックリンクの解決先をチェックするよう修正
- autocompact スラッシュループ検出（3回連続で即座にリフィルした場合にエラー停止）
- プロンプトキャッシュミス修正（ツールスキーマバイト変更によるセッション中のミス）
- ネストされた CLAUDE.md の重複再注入修正（長セッションで数十回再注入される問題）
- `StructuredOutput` スキーマキャッシュバグ修正（複数スキーマ使用時の50%失敗率）
- フック `if` 条件が複合コマンド（`ls && git push`）や環境変数プレフィックス付きコマンドにマッチするよう修正
- `-p --resume` のハング修正（64KB超ツール入力、deferred マーカー不在時）
- CJK・絵文字・デーヴァナーガリーテキストの切り詰め/ドロップ修正
- Windows: Edit/Write の CRLF 二重化修正、PowerShell stderr 偽エラー修正
- macOS: 音声モードマイク権限修正（Apple Silicon）
- `/stats` がサブエージェント使用量を含むよう修正
- `/env` が PowerShell ツールコマンドにも適用
- `/buddy` エイプリルフール機能

## v2.1.87 (2026-03-29)

- Cowork Dispatch のメッセージ未配信バグ修正

## v2.1.86 (2026-03-27)

- `X-Claude-Code-Session-Id` ヘッダーをAPIリクエストに追加（プロキシのセッション集約用）
- `.jj`（Jujutsu）、`.sl`（Sapling）をVCSディレクトリ除外リストに追加
- `@` メンションファイル内容のJSON エスケープ廃止（トークンオーバーヘッド削減）
- スキル説明文の上限を250文字に制限（コンテキスト使用量削減）
- `/skills` メニューがアルファベット順ソートに
- Read ツールがコンパクトな行番号形式に変更、未変更の再読み込みを重複排除
- Bedrock/Vertex/Foundry のプロンプトキャッシュヒット率改善

## v2.1.85 (2026-03-26)

- Hooks に `if` 条件フィールド追加（permission rule構文でツールイベントをフィルタリング）
- `CLAUDE_CODE_MCP_SERVER_NAME`, `CLAUDE_CODE_MCP_SERVER_URL` 環境変数（MCP `headersHelper` スクリプト用）
- MCP OAuth が RFC 9728 Protected Resource Metadata ディスカバリに対応
- `PreToolUse` フックで `AskUserQuestion` の `updatedInput` 返却が可能に
- スケジュールタスク（`/loop`, `CronCreate`）のトランスクリプトにタイムスタンプマーカー
- deep link クエリが5,000文字まで対応
- 組織ポリシーでブロックされたプラグインがマーケットプレースから非表示に
- `@` メンションファイル補完のパフォーマンス改善
- WASM yoga-layout を Pure TypeScript に置換（スクロールパフォーマンス改善）

## v2.1.84 (2026-03-26)

- PowerShellツール（Windows、opt-inプレビュー）
- `TaskCreated` hookイベント追加
- `WorktreeCreate` hookが `type: "http"` 対応（`hookSpecificOutput.worktreePath`）
- `allowedChannelPlugins` managed設定
- `CLAUDE_STREAM_IDLE_TIMEOUT_MS` 環境変数
- `ANTHROPIC_DEFAULT_{OPUS,SONNET,HAIKU}_MODEL_SUPPORTS` 環境変数
- `paths:` フロントマターがYAMLリスト形式のglobを受け付けるように
- MCPツール説明文・サーバー指示の上限2KB
- ローカル/claude.aiコネクタ間のMCPサーバー重複排除
- 75分以上アイドル後のプロンプト復帰機能
- deep link (`claude-cli://`) が優先ターミナルで開くように
- トークン表示 1M以上は「1.5m」形式に

## v2.1.83 (2026-03-25)

- `managed-settings.d/` ドロップインディレクトリでポリシー分割管理
- `CwdChanged`, `FileChanged` hookイベント追加
- `sandbox.failIfUnavailable` 設定
- `disableDeepLinkRegistration` 設定
- `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` 環境変数
- `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` 環境変数
- トランスクリプト検索（`/`キー、`Ctrl+O` でトランスクリプトモード内）
- エージェントが `initialPrompt` フロントマターを宣言可能に
- 画像ペースト時に `[Image #N]` チップ挿入
- `chat:killAgents`, `chat:fastMode` キーバインド再設定可能
- `TaskOutput` 非推奨化（`Read` でタスク出力ファイルを直接読む方式へ）
- MEMORY.md インデックスが25KB/200行で切り詰め

## v2.1.81 (2026-03-20)

- `--bare` フラグ（スクリプト用最小モード。hooks/LSP/pluginsスキップ）
- `--channels` パーミッションリレー（リサーチプレビュー）

## v2.1.80 (2026-03-19)

- ステータスラインにレート制限情報（5時間/7日ウィンドウ）
- skills/slashコマンドに `effort` フロントマター対応
- `--channels` リサーチプレビュー

## v2.1.79 (2026-03-18)

- `claude auth login --console` でAnthropic Console認証
- ターン所要時間表示トグル（`/config`）
- `SessionEnd` フックが対話型 `/resume` で発火するように修正

## v2.1.78 (2026-03-17)

- `StopFailure` hookイベント（APIエラー時）
- `${CLAUDE_PLUGIN_DATA}` 変数（プラグイン永続ステート）
- `effort`, `maxTurns`, `disallowedTools` フロントマター対応

## v2.1.77 (2026-03-17)

- Opus 4.6の出力トークン上限拡張（デフォルト64k、上限128k）
- `allowRead` サンドボックスファイルシステム設定

## v2.1.76 (2026-03-14)

- MCP elicitation（構造化入力のインタラクティブダイアログ）
- `Elicitation`, `ElicitationResult` hook
- `-n` / `--name` フラグでセッション表示名
- `worktree.sparsePaths` 設定（大規模monorepo向け）
- `PostCompact` hook
- `/effort` slashコマンド

## v2.1.75 (2026-03-13)

- **Opus 4.6で1Mコンテキストウィンドウ**（Max/Team/Enterprise）
- `/color` コマンド（プロンプトバーの色変更）
- メモリファイルに最終更新タイムスタンプ

## v2.1.74 (2026-03-12)

- `autoMemoryDirectory` 設定（オートメモリ保存先変更）
- `/context` コマンドに改善提案表示

## v2.1.73 (2026-03-11)

- `modelOverrides` 設定（カスタムモデルIDマッピング）
- Bedrock/Vertex/Foundryのデフォルト Opus を 4.6 に変更
- `/output-style` コマンド非推奨化

## v2.1.72 (2026-03-10)

- `/copy` で `w` キーでファイル直接書き出し
- `/plan` にオプション説明引数
- `ExitWorktree` ツール
- `CLAUDE_CODE_DISABLE_CRON` 環境変数

## v2.1.71 (2026-03-07)

- **`/loop` コマンド**（定期実行プロンプト/コマンド）
- cronスケジューリングツール

## v2.1.69 (2026-03-05)

- `/claude-api` スキル（Claude API開発支援）
- 音声STT 10言語追加（計20言語）

## v2.1.68 (2026-03-04)

- **Opus 4.6のデフォルト effort が medium に変更**
- "ultrathink" キーワードでhigh effort

## v2.1.63 (2026-02-28)

- **`/simplify` と `/batch` バンドルslashコマンド**
- HTTP hooks（URLにPOST JSON）
- claude.aiからMCPサーバー利用可能

## v2.1.59 (2026-02-26)

- **auto-memory機能**（Claudeが自動でコンテキストを `/memory` に保存）
- `/copy` コマンド（インタラクティブピッカー）

## v2.1.51 (2026-02-24)

- `claude remote-control` サブコマンド
- ツール結果50K超をディスクに永続化

## v2.1.50 (2026-02-20)

- `WorktreeCreate`, `WorktreeRemove` hookイベント
- エージェント `isolation: worktree` サポート
- `claude agents` CLIコマンド
- `--worktree` (`-w`) フラグ

## v2.1.49 (2026-02-19)

- MCP OAuth step-up auth
- `--worktree` (`-w`) フラグ（git worktree分離）
- エージェント `background: true` サポート
- プラグインが `settings.json` を同梱可能

## v2.1.45 (2026-02-17)

- **Claude Sonnet 4.6 サポート**

## v2.1.33 (2026-02-06)

- `TeammateIdle`, `TaskCompleted` hook
- エージェント `tools`, `memory` フロントマター

## v2.1.32 (2026-02-05)

- **Claude Opus 4.6 リリース**
- **Agent Teams機能**（マルチエージェント協調）
- auto-memory記録・呼び出し
- スキル自動発見（`--add-dir` から）

## v2.1.30 (2026-02-03)

- ReadツールのPDF `pages` パラメータ
- MCP OAuth事前設定クライアント
- `/debug` コマンド
