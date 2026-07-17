# harness-harness 更新履歴

## 2026-07-18 — 公式ドキュメント巡回（前回から1日、Claude Code新版あり）

### 巡回対象URL
- Claude Code: changelog — **v2.1.212**（2026-07-17）を検出。`/fork` 挙動変更・セッション上限系の新環境変数3件・MCP自動バックグラウンド化
- Claude Code: llms.txt — MD5一致で変更なし（174行、新規ドキュメントページなし）
- Codex CLI: GitHub Releases — 安定版は 0.144.5 のまま変更なし。プレリリースは 0.145.0-alpha.22（2026-07-17）まで進行（リリースノート本文なし）
- スキルエコシステム: last_patrol 2026-07-13（5日前）→ Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.212（2026-07-17）
- **specs/claude/changelog.md** — v2.1.212 のエントリを追加。主要: `/fork` がバックグラウンドセッションへのコピーに変更（旧挙動は `/subtask` に改名）、`claude auto-mode reset` 追加、WebSearch・サブエージェントスポーンのセッション上限（各デフォルト200）、MCPツール呼び出しの2分超自動バックグラウンド化、plan mode のファイル変更系Bash自動実行修正・worktree symlink 経由のリポジトリ外書き込み修正（セキュリティ）
- **specs/claude/configuration.md** — 新環境変数3件（`CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` / `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION` / `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS`）を追加
- **specs/claude/skills-and-commands.md** — `/fork` の挙動変更を追記、`/subtask` を新規追加
- **specs/claude/mcp.md** — MCP環境変数テーブルに `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` を追加

#### Codex CLI
- 安定版リリースなし → specs/codex/changelog.md 更新なし

### PR
- （後述）

## 2026-07-17 — 公式ドキュメント巡回（前回から1日、小規模差分）

### 巡回対象URL
- Claude Code: changelog — **v2.1.211**（2026-07-15）を検出。新フラグ1件＋セキュリティ・バグ修正中心
- Claude Code: llms.txt — 新規ドキュメントページなし（whats-new は w28 まで、前回確認済み範囲と同一）
- Codex CLI: GitHub Releases — **安定版 0.144.5**（2026-07-16）を検出。dangerous-command 検出改善のパッチ。プレリリースは 0.145.0-alpha.16 まで進行
- スキルエコシステム: last_patrol 2026-07-13（4日前）→ Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.211（2026-07-15）
- **specs/claude/changelog.md** — v2.1.211 のエントリを追加。主要: `--forward-subagent-text` フラグ / `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT` 環境変数（stream-json にサブエージェント出力を含める）、permission プレビューの bidi/ゼロ幅文字無害化、auto mode が hook の `ask` を上書きしない修正、"always allow" ルールのリポジトリルート保存化（worktree 間で承認持続）、Bedrock/Vertex 等の prompt-caching 課金回帰修正
- **specs/claude/configuration.md** — 新環境変数1件（`CLAUDE_CODE_FORWARD_SUBAGENT_TEXT`）を追加

#### Codex CLI 0.144.5（2026-07-16）
- **specs/codex/changelog.md** — 0.144.5 のエントリを追加（dangerous-command 検出改善: 強制 `rm` 変種の検出拡大、拒否理由の明確化）

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/97

## 2026-07-16 — 公式ドキュメント巡回（前回から1日、小規模差分）

### 巡回対象URL
- Claude Code: changelog — **v2.1.210**（2026-07-14）を検出。バグ修正中心のリリース
- Claude Code: llms.txt — 新規ドキュメントページなし（whats-new は w28 まで、前回確認済み範囲と同一）
- Codex CLI: GitHub Releases — 安定版は 0.144.4 のまま変更なし。プレリリースは 0.145.0-alpha.13（2026-07-15）まで進行（リリースノート本文なし）
- スキルエコシステム: last_patrol 2026-07-13（3日前）→ Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.210（2026-07-14）
- **specs/claude/changelog.md** — v2.1.210 のエントリを追加。主要: permission ルール `Write(path)`/`NotebookEdit(path)`/`Glob(path)` への起動時警告（`Edit(path)`/`Read(path)` を推奨）、auto mode 分類器の外部セッション Sonnet 5 デフォルト化、Agent ツールの間接プロンプトインジェクション対策強化、`isolation: 'worktree'` サブエージェントの隔離不備修正、hook タイムアウトのユーザー拒否誤報告修正、skills/commands の `$1`/`$2` プレースホルダ verbatim 保持化
- specs/claude/configuration.md — 更新なし（v2.1.210 に新設定キー・環境変数の追加なし）

#### Codex CLI
- 安定版リリースなし → specs/codex/changelog.md 更新なし

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/96

## 2026-07-15 — 公式ドキュメント巡回（前回から1日、中規模差分）

### 巡回対象URL
- Claude Code: changelog — **v2.1.208 / v2.1.209**（2026-07-14）を検出。v2.1.208 は大型メンテナンスリリース
- Claude Code: llms.txt — 新規ドキュメントページなし（whats-new は w28 まで、前回確認済み範囲と同一）
- Codex CLI: GitHub Releases — **安定版 0.144.4**（2026-07-14、ユーザー向け変更なしのパッチ）。最新プレリリースは 0.145.0-alpha.11
- スキルエコシステム: last_patrol 2026-07-13（2日前）→ Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.208 / v2.1.209（2026-07-14）
- **specs/claude/changelog.md** — 2バージョン分のエントリを追加。主要: スクリーンリーダーモード（`--ax-screen-reader`）、`vimInsertModeRemaps`、`CLAUDE_CODE_PROCESS_WRAPPER`（企業ラッパー経由スポーン）、`$(…)`/バッククォート内の破壊的削除も skip-permissions / auto mode で確認プロンプト化、長時間セッションのメモリリーク修正多数（トランスクリプト最大79分の1削減等）、permission ルール多数時の遅延解消
- **specs/claude/configuration.md** — 新設定キー2件（`axScreenReader`, `vimInsertModeRemaps`）、新環境変数2件（`CLAUDE_AX_SCREEN_READER`, `CLAUDE_CODE_PROCESS_WRAPPER`）を追加

#### Codex CLI 0.144.4（2026-07-14）
- **specs/codex/changelog.md** — 0.144.4 のエントリを追加（ユーザー向け変更なしのパッチリリース）

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/95

## 2026-07-14 — 公式ドキュメント巡回（前回から1日、小規模差分）

### 巡回対象URL
- Claude Code: changelog — **新バージョンなし**（最新は v2.1.207、2026-07-11 のまま）
- Claude Code: llms.txt — whats-new が w28 まで公開済みであることを確認（内容は前日反映済みの v2.1.169-207 と同範囲）。新規ドキュメントページなし
- Codex CLI: GitHub Releases — **安定版 0.144.2 / 0.144.3**（2026-07-13）を検出。最新プレリリースは 0.145.0-alpha.7
- スキルエコシステム: last_patrol 2026-07-13（1日前）→ Phase 3.5 スキップ

### 検出された変更と更新内容
- **specs/codex/changelog.md** — 0.144.2（Guardian auto-review のプロンプト回帰リバート）/ 0.144.3（バージョンのみ）の合同エントリを追加

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/94（マージ済み）

## 2026-07-13 — 公式ドキュメント巡回（前回から34日、大型差分）

### 巡回対象URL
- Claude Code: changelog（**v2.1.169 → v2.1.207、30バージョン分の新版**）
- Codex CLI: GitHub Releases（**安定版 0.138.0 → 0.144.1、8リリース**）
- スキルエコシステム: 前回 2026-06-09 から34日経過 → Phase 3.5 実行（全6ソース取得成功）

### 検出された変更と更新内容

#### Claude Code v2.1.169〜v2.1.207（2026-06-08〜07-11）
- **specs/claude/changelog.md** — 30バージョン分のエントリを追加。最重要: **Claude Fable 5 導入**（v2.1.170、Mythosクラス）、**Claude Sonnet 5 デフォルトモデル化**（v2.1.197、ネイティブ1Mコンテキスト・8/31までプロモ価格）、**サブエージェントのバックグラウンドデフォルト化・Claude in Chrome GA**（v2.1.198）、**agent teams 刷新**（v2.1.178、TeamCreate/TeamDelete 削除→暗黙チーム化）、**サブエージェントの5階層ネスト**（v2.1.172）、**hookマッチャーのハイフン識別子完全一致化（破壊的変更）**（v2.1.195）、auto mode の破壊的gitコマンドブロック（v2.1.183）と Bedrock/Vertex/Foundry GA（v2.1.207）、セキュリティ修正多数（MCP自己承認の穴 v2.1.196、plugin shell-injection v2.1.207）
- **specs/claude/configuration.md** — 新設定キー11件（`disableBundledSkills`, `enforceAvailableModels`, `sandbox.credentials`, `autoMode.classifyAllShell`, `respondToBashCommands`, `footerLinksRegexes` 等）、新環境変数11件（`CLAUDE_CODE_SAFE_MODE`, `OTEL_LOG_ASSISTANT_RESPONSES`, `CLAUDE_ENABLE_STREAM_WATCHDOG` 等）、`Tool(param:value)` 権限構文、Manual モード改名
- **specs/claude/hooks.md** — matcher のカンマ区切り修正・ハイフン完全一致化、`Notification` の `agent_needs_input`/`agent_completed`、plugin `${user_config.*}` シェルインジェクション対策
- **specs/claude/skills-and-commands.md** — 新コマンド（`/cd`, `/doctor`(`/checkup`), `/config key=value`, `/dataviz`, `/commit-push-pr`）、`/agents` ウィザード削除、ネストされた `.claude/skills`、スキルのスタック呼び出し、Explore のモデル継承、サブエージェントのバックグラウンドデフォルト・ネスト仕様
- **specs/claude/mcp.md** — `claude mcp login/logout`、ツールアイドルタイムアウト、`roots/list`、v2.1.196 セキュリティ修正
- **specs/claude/agent-teams.md** — v2.1.178 の暗黙チーム化を反映、`teammateMode: "iterm2"` 追加

#### Codex CLI 0.138.0〜0.144.1（2026-06-08〜07-09）
- **specs/codex/changelog.md** — 安定版8リリース分を追加。alpha.7 由来の暫定エントリは公式リリースノート準拠の 0.138.0 / 0.139.0 エントリに再統合。最重要: **/import（Claude Code からの設定・チャットインポート）**・`codex delete`・Bedrock APIキー認証＋認証情報暗号化保存（0.140）、Noise relay E2E 暗号化（0.141）、**rollout token budgets**・multi-agent delegation 3段階制御・indexed web-search（0.142）、**リモートプラグインデフォルト有効化**・システムプロキシ PAC/WPAD・Bedrock GPT-5.6（0.143）、**writes app-approval mode**・MCP対話型認証標準化（0.144）
- **specs/codex/commands.md** — `/delete`, `/import`, `/usage` を追加
- **specs/codex/configuration.md** — 0.140〜0.144 の注目デフォルト変更（MCP tool search / リモートプラグイン / システムプロキシ / token budgets 等）を追記

#### スキルエコシステム巡回（Phase 3.5）
- **kb/skills/_index.md** — `last_patrol` 2026-07-13。anthropics/skills 148K → 160.5K stars、claude.com/plugins 4ページ化（Frontend Design 1.05M installs 首位）、find-skills 2.5M installs、agentskills.io 44プラットフォーム
- **kb/skills/sources.md** — **openai/skills が 2026-06-22 に deprecated（#496）**。後継 openai/plugins（180件）を巡回対象に追加、旧リポジトリは優先度「低（凍結）」に降格。developers.openai.com/codex/skills の learn.chatgpt.com への308リダイレクトを注記
- **kb/skills/recommended.md** — インストール数更新のみ、**Tier 変動なし**（mattpocock 系 grill-me / grill-with-docs と Microsoft Azure 系の top10 定着を確認したが需要不明のため候補記録に留める）

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/93 — merged 2026-07-13

---

## 2026-06-09 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**変更なし**。最新は v2.1.168（2026-06-06）で前回巡回時点から進展なし）/ `whats-new/2026-w23` 未公開（404 継続）
- Codex CLI: GitHub Releases（**新版あり**。`rust-v0.138.0-alpha.7` が 2026-06-08 14:47 UTC 公開）
- スキルエコシステム: 前回 2026-06-01 から 8 日経過 → Phase 3.5 実行

### 検出された変更と更新内容

#### Codex 0.138.0-alpha.7（2026-06-08）
- **specs/codex/changelog.md** — 0.138.0-alpha.7 セクション新設。GitHub Releases 本文は空のため、`rust-v0.138.0-alpha.6` 以降の `main` コミット 14 件から主要変更点を整理。最終更新日を 2026-06-09 に更新
  - 主要変更: **MAv2 residency LRU**（アイドル v2 サブエージェントを `ThreadManager` から退去、論理識別と residency を分離）、**MAv2 並列実行カウント方式変更**（論理エージェント数 → アクティブ非 root v2 ターン数、`/goal` のアイドル継続は除外）、**MAv2 ツール `close_agent` → `interrupt_agent` リネーム（v2 のみ）**（`Op::Interrupt` を送出、対象は registered のまま維持、root/self ターゲット拒否、v1 サーフェスは変更なし）、**code mode で standalone web search 有効化**（`/v1/alpha/search` プレーンテキスト出力対応）、**unified exec で approval/sandbox 決定保持**（zsh-fork ランタイム経由のバグ修正、`WithAdditionalPermissions` は bounded 経路）、**TUI MCP startup をスレッドスコープ化**（`mcpServer/startupStatus/updated` に nullable `threadId` 追加、サブエージェント由来失敗を親トランスクリプトから隔離）、**TUI `resume --last` / `fork --last` でプロンプト位置引数を受理**、**remote-control 汎用 404 で enrollment 維持**、core-plugins SKU 送信、`rusty_v8` 149.2.0、Bazel BuildBuddy シークレット

#### スキルエコシステム巡回（Phase 3.5）
- **kb/skills/_index.md** — `last_patrol` を 2026-06-09 に更新。claude.com/plugins 件数 72+ → 147+、anthropics/skills stars 140K+ → 148K+、openai/skills stars 15K+ → 21.7K+、find-skills installs 1.8M → 1.9M に更新
- **kb/skills/recommended.md** — `last_checked` を 2026-06-09 に更新。frontend-design installs 334K → 516K、find-skills installs 1.8M → 1.9M
- **Tier 変動なし**: Vercel surge（vercel-react-best-practices #3、agent-browser #4 が skills.sh top 10 に新規ランクイン）は確認したが、harness-harness 内の Vercel 需要が明確化するまで Tier B 追加は保留。Microsoft Azure 系も同様

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/92 — merged 2026-06-09

---

## 2026-06-08 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.168 が 2026-06-06 公開、バグ修正・信頼性改善のみ）/ `whats-new/2026-w23` 未公開（404 継続。w23 は 2026-06-01〜05 範囲を想定）
- Codex CLI: GitHub Releases（**変更なし**。最新は 0.138.0-alpha.6（2026-06-06）で前回巡回時点から進展なし）
- スキルエコシステム: 前回 2026-06-01 から 7 日 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.168（2026-06-06）
- **specs/claude/changelog.md** — v2.1.168 セクション新設（バグ修正・信頼性改善のみ。ユーザー向け新機能なし）。最終更新日を 2026-06-08 に更新

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/91 — merged 2026-06-08

---

## 2026-06-07 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.166 が 2026-06-06 公開、v2.1.167 が同日公開）/ `whats-new/2026-w23` 未公開（404 継続）
- Codex CLI: GitHub Releases（**新版あり**。0.138.0-alpha.5 が 2026-06-05、0.138.0-alpha.6 が 2026-06-06 公開）
- スキルエコシステム: 前回 2026-06-01 から 6 日 → Phase 3.5 スキップ（7 日未満）

### 検出された変更と更新内容

#### Claude Code v2.1.166（2026-06-06）
- **specs/claude/changelog.md** — v2.1.166 セクション新設。最終更新日を 2026-06-07 に更新
  - 主要新機能: `fallbackModel` 設定（最大 3 つの fallback モデル）、`--fallback-model` のインタラクティブセッション適用、`permissions.deny` ツール名位置の glob サポート（`"*"` で全 deny、`permissions.allow` は MCP 以外の glob 拒否）、クロスセッション `SendMessage` のセキュリティ強化（中継メッセージはユーザー権限を持たない）、`MAX_THINKING_TOKENS=0` / `--thinking disabled` / per-model toggle が Claude API 経由の thinking 既定モデルを無効化、fallback モデルでのターン自動再試行、`claude update` のターゲットバージョン事前告知、`claude agents` の URL によるセッションフィルタ
  - 主要修正: 画像処理連続エラーと余分トークン消費、worker registration 中断時のリモートセッション復旧、JetBrains 2026.1+ ターミナルチラつき、Kitty keyboard protocol 上の `Shift+非ASCII`、Windows PowerShell コマンド検証ハング、macOS orphan `claude --bg-pty-host`、音声モード `/login` 修正、Managed settings の invalid エントリで全 policy silently 無効化、`allowedMcpServers` / `deniedMcpServers` の `${VAR}` 参照、`claude agents` 内 worktree 再オープン時のクラッシュループ、Ctrl+O トランスクリプトの thinking 二重描画
- **specs/claude/configuration.md** — `fallbackModel` を model 群に追加。`permissions.allow` / `permissions.deny` 行に v2.1.166 の glob ルール挙動を追記。env vars 表に `MAX_THINKING_TOKENS` の挙動変更を追加。settings 表末尾に Managed settings invalid エントリ堅牢化と `${VAR}` 参照修正の補足を追加。最終更新日を 2026-06-07 に更新
- **specs/claude/agent-teams.md** — `SendMessage` セクションにクロスセッション中継のセキュリティ強化（v2.1.166）を追記

#### Claude Code v2.1.167（2026-06-06）
- **specs/claude/changelog.md** — v2.1.167 セクション新設（バグ修正・信頼性改善のみ。ユーザー向け新機能なし）

#### Codex 0.138.0-alpha.5 / .6（2026-06-05〜06）
- **specs/codex/changelog.md** — 0.138.0-alpha.5 / .6 セクション新設。GitHub Releases 本文は空のため、`rust-v0.138.0-alpha.4` 以降の `main` コミット 57 件から主要変更点を整理。最終更新日を 2026-06-07 に更新
  - 主要変更: Responses Lite の override / standalone ツール / 専用トランスポートヘッダ、プラグイン `plugin` JSON 出力 / `plugin read` のリモート MCP 公開 / `plugin detail` で利用不可テンプレ公開 / TUI 起動高速化、MAv2 ペイロード暗号化 / v2 PAT サポート / Goal 拡張のコア挙動アラインメント完遂 (1/2, 2/2)、マネージドパーミッションプロファイル allowlist 強制 / exec policy ファイルシステム由来化 / Linux サンドボックスでの `socketpair` 許可（proxy 経路）、AGENTS.md discovery の論理パス維持、`resume --last` の state DB 優先、TUI 細部修正（コメントディレクティブ描画、ストリーミング空行抑制、キャンセル時カーソル復元、Windows composer 背景）、remote-control ペアリング状態 RPC / transport、app-server account token 使用量公開 (1/2)、`/usr/bin/bash` シェルフォールバック、Bazel 起動オプション安定化

### PR
- https://github.com/nomuvan-dev/harness-harness/pull/90 — merged 2026-06-07

---

## 2026-06-06 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.163 が 2026-06-04 公開、v2.1.165 が 2026-06-05 公開。v2.1.164 は公式 changelog で欠番）/ `whats-new/2026-w23` 未公開（404 継続。w23 は 2026-06-01〜05 範囲）
- Codex CLI: GitHub Releases（**新版あり**。0.138.0-alpha.3 と alpha.4 が 2026-06-04 公開）
- スキルエコシステム: 前回 2026-06-01 から 5 日 → Phase 3.5 スキップ（7 日未満）

### 検出された変更と更新内容

#### Claude Code v2.1.163（2026-06-04）
- **specs/claude/changelog.md** — v2.1.163 セクション新設。最終更新日を 2026-06-06 に更新
  - 主要新機能: `requiredMinimumVersion` / `requiredMaximumVersion` managed settings（許容バージョン外なら起動拒否）、`/plugin list` コマンド（`--enabled`/`--disabled` フィルタ）、`/btw` の「c でコピー」ショートカット、Hooks の `Stop` / `SubagentStop` で `hookSpecificOutput.additionalContext` をサポート（hook エラー扱いではない通常応答）、Skills の `\$` エスケープ構文、`--resume` 時の stdio MCP サーバーへの `CLAUDE_CODE_SESSION_ID` 共有、バックグラウンドエージェントセッションのバックグラウンド更新（コールド再起動を待たない）
  - 主要修正: `claude -p` ハング修正（バックグラウンドコマンド未終了時 5 秒で SIGTERM）、Bedrock/Vertex/Foundry + `CI=true` の `claude -p` 認証エラー、v2.1.154 リグレッションの `$TMPDIR` 上書き範囲修正（サンドボックスコマンドのみに限定、Bazel/EDR Go 問題解消）、Windows 読み取り専用/OneDrive での session-env EEXIST、fresh config 起動中の組織 managed 権限ルール未適用、`claude agents` 再アタッチ時のバックグラウンドタスク喪失、`claude agents` Esc 不整列・数秒ハング、Hook `if: "Bash(...)"` 条件のサブシェル/`$()`誤発火、`Read(~/...)` deny の `$HOME` 経由 Bash 未ブロック
- **specs/claude/configuration.md** — settings table に `requiredMinimumVersion` / `requiredMaximumVersion` を追加。最終更新日を 2026-06-06 に更新
- **specs/claude/hooks.md** — section 6.4 Stop に `hookSpecificOutput.additionalContext` サポート（v2.1.163+）の説明と JSON 例を追加。最終更新日を 2026-06-06 に更新
- **specs/claude/skills-and-commands.md** — `/btw` 行に `c` キーでクリップボードコピー追記、`/plugin list` 行を新規追加。最終更新日を 2026-06-06 に更新
- **specs/claude/mcp.md** — v2.1.154 段落に `--resume` パスでも `CLAUDE_CODE_SESSION_ID` が stdio MCP サーバーに渡る旨を追記

#### Claude Code v2.1.165（2026-06-05）
- **specs/claude/changelog.md** — v2.1.165 セクション新設（バグ修正・信頼性改善のみ。ユーザー向け新機能なし）

#### Codex 0.138.0-alpha.3 / .4（2026-06-04）
- **specs/codex/changelog.md** — 0.138.0-alpha.3 / .4 セクション新設。GitHub Releases 本文は空のため、`rust-v0.138.0-alpha.2` 以降の `main` コミットから主要変更点を整理
  - 主要変更: Code mode からのツール名前空間除外、standalone 画像生成の保存パスヒント、モデル定義 reasoning effort のサポートと順序、`plugin list` JSON 出力へのマーケットプレースソース追加、`app-server -c` config 上書き、External agent セッション検知の境界整理、External agent migration の異種 MCP transport 混在ガード、AGENTS.md ロードを environment filesystem 経由に集約、`experimentalFeature/enablement/set` 整理、`response.processed` websocket リクエスト削除、`codex-analytics` 初期化時 forked thread id 発行、ThinLTO リリースバイナリ、Azure artifact 署名環境シークレット

### Phase 3.5（スキルエコシステム）スキップ
- `kb/skills/_index.md` の `last_patrol: 2026-06-01` から 5 日経過 → 7 日未満のためスキップ
- 次回スキル巡回は 2026-06-08 以降

### 結論
- Claude Code v2.1.163（2026-06-04）と v2.1.165（2026-06-05）、Codex 0.138.0-alpha.3/.4（2026-06-04）が前回巡回（2026-06-05）以降に公開されており、いずれも specs/ に反映済み
- v2.1.163 は managed settings の新キー 2 つ、`/plugin list` 新規コマンド、Hooks の `Stop`/`SubagentStop` の `additionalContext` 取り扱い拡張、Skills の `\$` エスケープ、stdio MCP への `--resume` 時 session ID 共有など、機能性変更が多くコマンド・hooks・settings の各 spec を横断更新
- v2.1.164 は公式 changelog で欠番（おそらく内部リリースのみ）。changelog.md にはそれが分かるよう v2.1.165 → v2.1.163 の順で記載
- mapping/ への影響なし

---

## 2026-06-05 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.162 が 2026-06-03 公開）/ `whats-new/2026-w23` 未公開（404 継続。w23 は 2026-06-01〜05 範囲）
- Codex CLI: GitHub Releases（**新版あり**。0.137.0 stable が 2026-06-04 公開、0.138.0-alpha.1 と alpha.2 が同 2026-06-04 公開）
- スキルエコシステム: 前回 2026-06-01 から 4 日 → Phase 3.5 スキップ（7 日未満）

### 検出された変更と更新内容

#### Claude Code v2.1.162（2026-06-03）
- **specs/claude/changelog.md** — v2.1.162 セクション新設。最終更新日を 2026-06-05 に更新
  - 主要新機能・改善: `claude agents --json` に `waitingFor` フィールド追加、ネイティブビルドで `--tools` 指定時に Grep / Glob 提供、`/effort` 永続化確認、スラッシュコマンド補完が即実行ではなくプロンプト挿入になる挙動変更、Remote Control が起動メッセージから永続フッターピルへ、`Windsurf` → `Devin Desktop` リネーム、起動出力削減、launch-prompt 警告のピン留め、失敗ターンのコンパクト警告化、`claude update` 起動検証強化
  - 主要セキュリティ修正: `WebFetch` パーミッションルールが事前承認ドメインに適用されない問題（明示ルールが事前承認に優先するように）、Windows でバックスラッシュ / 大文字小文字違いのパーミッションルールがマッチしない問題（`Read` deny ルールが Glob/Grep からもファイルを隠すように）
  - その他主要修正: ターン開始時の Esc 割り込みが stream-json / SDK セッションで silently drop される問題、絵文字含む classifier クエリでの API 400、MCP `timeout` <1000ms の watchdog floor 問題、LSP `workspaceSymbol` の空結果問題、`claude agents` の表示幅・画像ペースト・session 名切り詰め問題、バックグラウンドセッションのサービス起動失敗時会話喪失、深い `CLAUDE_CODE_TMPDIR` でクロスセッション `SendMessage` が壊れる問題、実行中バックグラウンドセッション 5 秒 stall

#### Codex 0.137.0 stable（2026-06-04）
- **specs/codex/changelog.md** — 0.137.0 stable セクション新設。リリースノートは空で内容は 0.137.0-alpha.5 と同等のため、既存 alpha.4/.5 節への参照誘導を追記

#### Codex 0.138.0-alpha.1 / .2（2026-06-04）
- **specs/codex/changelog.md** — 0.138.0-alpha.1 / .2 セクション新設。GitHub Releases 本文は空のため、`rust-v0.137.0` 以降の `main` コミットから主要変更点を整理
  - 主要変更: Code mode でローカル画像パスをモデルに公開（Windows カバレッジ復元）、Codex worktree への Bazel 設定コピー、MAv2 設定のカタログ化（`feat: catalog multi-agent v2 config`）と MAv2 プロンプト微調整、リモートプラグインのデフォルトプロンプト保持、`/goal edit` 複数行ペースト再修正、app-server account session protocol（profile-switcher の 1/2）、コア `exec` 経路から `SandboxPolicy` を排除、リモート圧縮時のオーバーサイズ tool output 書き換え、`/diff` 等の git enrichment ガード、`codex-pr-body` で confidential references 回避、forked スレッド名継承修正

### Phase 3.5（スキルエコシステム）スキップ
- `kb/skills/_index.md` の `last_patrol: 2026-06-01` から 4 日経過 → 7 日未満のためスキップ
- 次回スキル巡回は 2026-06-08 以降

### 結論
- Claude Code v2.1.162（2026-06-03）、Codex 0.137.0 stable + 0.138.0-alpha.1/.2（2026-06-04）が前回巡回（2026-06-04）以降に公開されており、いずれも specs/claude/changelog.md と specs/codex/changelog.md に反映済み
- v2.1.162 のセキュリティ修正（WebFetch ルール / Windows パーミッション）は configuration.md にバックスラッシュや事前承認ドメインに関する既存セクションが無いため、changelog の記述のみで完結
- mapping/ への影響なし

---

## 2026-06-04 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.161 が 2026-06-02 公開）/ `whats-new/2026-w23` 未公開（404 継続）
- Codex CLI: GitHub Releases（**新版あり**。0.137.0-alpha.4 と alpha.5 が 2026-06-03 公開、0.136.0 stable 以降のプレリリース系列）
- スキルエコシステム: 前回 2026-06-01 から 3 日 → Phase 3.5 スキップ（7 日未満）

### 検出された変更と更新内容

#### Claude Code v2.1.161（2026-06-02）
- **specs/claude/changelog.md** — v2.1.161 セクション新設。最終更新日を 2026-06-04 に更新
  - 主要新機能・変更: `OTEL_RESOURCE_ATTRIBUTES` をメトリクスのラベル化（Team/Repo スライス）、`claude agents` 行で `done/total` 表示・peek で最長実行中表示、`/mcp` 未使用 claude.ai connector 折りたたみ、並列ツール実行の独立化（Bash 失敗が他をキャンセルしない）、Linux fullscreen クリップボード `wl-copy`/`xclip`/`xsel` + PRIMARY、`/effort` ダイアログ・ワークフローアニメーション・プロンプトキーワード shimmer の Reduce motion 遵守
  - 主要セキュリティ修正: `claude mcp list`/`get`/`add` がシークレットを出力する問題を修正（`${VAR}` 非展開・認証ヘッダー/URL 内シークレット redact）
  - 主要バグ修正: `forceLoginOrgUUID`/`forceLoginMethod` のサードパーティプロバイダブロック（v2.1.146 リグレッション）、`/usage-credits` の Team/Enterprise admin 再ログイン誤動作、`/autofix-pr` の git worktree 内誤検知、`--resume` ピッカーの非 worktree 環境表示、Workflow `isolation: "worktree"` のバックグラウンドセッション編集ブロック、Windows hooks の `/usr/bin/bash` 明示失敗、OTEL ログイベント初期化前の silent drop、バックグラウンドセッションの stale モデル、`.claude/worktrees/` オーファンスイープ
- **specs/claude/configuration.md** — `OTEL_RESOURCE_ATTRIBUTES` をメトリクスラベル用途として行追加。`forceLoginOrgUUID` / `forceLoginMethod` 行に v2.1.146〜v2.1.160 リグレッション修正注記
- **specs/claude/mcp.md** — §3.2 に v2.1.161 シークレット redaction（`claude mcp list`/`get`/`add`）と `/mcp` connector 折りたたみを記録。最終更新日を 2026-06-04 に更新
- **specs/claude/skills-and-commands.md** — `/effort` 行に Reduce motion 遵守、`/mcp` 行に connector 折りたたみ、`/usage-credits` 行に Team/Enterprise admin 修正を追記。最終更新日を 2026-06-04 に更新

#### Codex CLI 0.137.0-alpha.4 / alpha.5（2026-06-03、プレリリース）
- **specs/codex/changelog.md** — 0.137.0-alpha.4 / alpha.5 セクションを新設（リリースノートが空のため `main` ブランチコミットから整理）。最終更新日を 2026-06-04 に更新
  - 主要新機能（3 大テーマ）: **v1 Skills 拡張**（per-turn カタログ解決、プロンプト注入、scaffold、manifest 検証）、**Multi-Agent v2（MAv2）dogfood デフォルト化**（`assign_task`→`followup_task` 改名、`close_agent` 自己ターゲット拒否、`hide_spawn_agent_metadata` デフォルト `true`、per-thread runtime metadata）、**Cloud config bundle 大規模リファクタ**（EDU 取得、MITM CA trust 子環境伝達、runtime 切替）
  - その他: Goal 拡張（idle continuation、GoalApi、`/goal edit` 複数行ペースト修正、ステアリングテンプレ化）、remote-control RPCs + pairing start + `environmentId`、TUI F1〜F24 キーマップ、検索可能セレクションメニューでペースト許可、standalone 画像生成を Code Mode で公開、並列 standalone web search、コールド rollout 圧縮 + counters/histograms + スニペット再利用、Windows BuildBuddy Bazel wrapper・UAC マニフェスト・PDB staging・thread resume パス修正
  - 補足: 公式リリースノートは空、安定版 0.137.0 リリース時に内容を再確認・統合する旨を specs に明記

#### スキルエコシステム（Phase 3.5）
- スキップ（`kb/skills/_index.md` の `last_patrol: 2026-06-01` から 3 日経過のみ、7 日未満）

#### キャッシュ
- **.patrol-cache/url-metadata.json** — `last_patrol` を 2026-06-04 に更新。Claude Code changelog / Codex Releases / settings.md / env-vars / commands / mcp.md の `status` を `changed` へ遷移し、v2.1.161 と 0.137.0-alpha 検出要点を `notes` に記録

### 注記
- v2.1.161 はテレメトリ強化（OTEL_RESOURCE_ATTRIBUTES）、アクセシビリティ向上（Reduce motion 遵守）、セキュリティ修正（mcp コマンドのシークレット redact）、リグレッション修正（v2.1.146 Bedrock/Vertex/Foundry/Mantle ブロック）が主軸の堅実なリリース
- 0.137.0-alpha 系は Codex CLI の構造的進化（Skills 拡張による per-turn カタログ注入、MAv2 dogfood デフォルト化、cloud config bundle への基盤移行）が同時進行する大型シリーズ。安定版 0.137.0 リリース時に specs/codex/changelog.md を再整理する余地あり
- mapping/ への影響は限定的: Codex の Skills 拡張は Claude Code の `.claude/skills/` plugin autoload（v2.1.157）と概念的に近接。MAv2 は Claude Code の Dynamic workflows（v2.1.154）と用途が重なる。両者の対応関係は安定版 0.137.0 確定後に整理予定

---

## 2026-06-03 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.160 が 2026-06-02 公開）/ `whats-new/2026-w23` 未公開（404 継続）
- Codex CLI: GitHub Releases（**新版なし**。0.136.0 stable 2026-06-01 が最新のまま）
- スキルエコシステム: 前回 2026-06-01 から 2 日 → Phase 3.5 スキップ（7 日未満）

### 検出された変更と更新内容

#### Claude Code v2.1.160（2026-06-02）
- **specs/claude/changelog.md** — v2.1.160 セクション新設。最終更新日を 2026-06-03 に更新
  - 主要新機能・変更: シェル起動ファイル (`.zshenv` / `.zlogin` / `.bash_login`) と `~/.config/git/` への書き込みプロンプト追加、`acceptEdits` モードでビルドツール設定ファイル (`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/` 等) プロンプト追加、Dynamic workflow トリガーキーワード `workflow` → `ultracode` リネーム、`CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` 環境変数削除、Edit の read-before-edit を単一ファイル `grep`/`egrep`/`fgrep` でも満たすよう緩和
  - 主要バグ修正: WSL の copy-on-select クリップボード（PowerShell interop 化）、`claude agents` セッション復元時の履歴喪失、バックグラウンドセッション夜間 retire 再アタッチ
- **specs/claude/configuration.md** — `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` を v2.1.160 削除（no-op）として注記。Workflow キーワードトリガー注記に `workflow` → `ultracode` リネーム追記
- **specs/claude/skills-and-commands.md** — `/workflows` 行にトリガーキーワード `ultracode` リネーム追記

#### Codex CLI
- 変更なし（0.136.0 stable 2026-06-01 が最新のまま、0.137.x の alpha 等はまだ未公開）

#### スキルエコシステム（Phase 3.5）
- スキップ（`kb/skills/_index.md` の `last_patrol: 2026-06-01` から 2 日経過のみ、7 日未満）

#### キャッシュ
- **.patrol-cache/url-metadata.json** — `last_patrol` を 2026-06-03 に更新。Claude Code changelog の `status` を `changed` へ遷移し、v2.1.160 検出要点を `notes` に記録。Codex Releases は `no_change` で notes 更新のみ

### 注記
- v2.1.160 はセキュリティ・体験改善が中心の比較的大きなリリース。`workflow` キーワードのリネームは既存ユーザーの筋肉記憶に影響するため、specs 3 箇所（changelog / configuration / skills-and-commands）で明記
- mapping/ への影響なし（Codex 側に対応概念がないため）

---

## 2026-06-02 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版あり**。v2.1.159 が 2026-05-31 公開）/ `whats-new/2026-w23` 未公開（404 継続）
- Codex CLI: GitHub Releases（**新版あり**。0.136.0 stable が 2026-06-01 公開、0.135.0 → 0.136.0）
- スキルエコシステム: 前回 2026-06-01 から 1 日 → Phase 3.5 スキップ（7 日未満）

### 検出された変更と更新内容

#### Claude Code v2.1.159（2026-05-31）
- **specs/claude/changelog.md** — v2.1.159 を 1 行で追記（内部インフラ改善のみ、ユーザー向け変更なし）。最終更新日を 2026-06-02 に更新

#### Codex CLI 0.136.0 stable（2026-06-01、0.136.0-alpha.1 → stable）
- **specs/codex/changelog.md** — 0.136.0 stable セクションを新設。0.136.0-alpha.1 は履歴目的で残置し stable への参照リンクを追加。最終更新日を 2026-06-02 に更新
  - 主要新機能: セッションアーカイブ（`/archive` / `codex archive` / `codex unarchive`）、TUI Markdown 強化（OSC 8 リンク・key/value テーブル）、`codex app-server --stdio`、`CODEX_API_KEY` リモート登録、Windows `codex sandbox setup --elevated`（alpha）、feature-gated standalone 画像生成 ext、Bedrock GPT-5.5 + API キー region フォールバック、`experimental_request_user_input` トグル
  - 主要セキュリティ修正: `/diff` でリポジトリ提供 Git ヘルパー実行を防止、PowerShell パーサ実行を非 Windows で回避、exec-server websocket Origin ヘッダ拒否、sandbox deny-read を safe-command / approval-bypass 経路でも維持
  - 認証: ChatGPT access token を 5 分 expiry 前に refresh、`refresh_token_reused` を relogin-required として扱う
- **specs/codex/commands.md** — `/archive` を 1.4 セッション管理表に追加（CLI 等価 `codex archive` / `codex unarchive` を併記）。最終更新日を 2026-06-02 に更新

#### スキルエコシステム（Phase 3.5）
- スキップ（`kb/skills/_index.md` の `last_patrol: 2026-06-01` から 1 日経過のみ、7 日未満）

#### キャッシュ
- **.patrol-cache/url-metadata.json** — `last_patrol` を 2026-06-02 に更新。Claude Code changelog と Codex Releases の `status` を `changed` へ遷移し、検出した新版の要点を `notes` に記録

### 注記
- Codex 0.136.0 は alpha.1 から stable への昇格で、安定版変更点が公開された。`/archive` という新規ユーザー機能が増えたため commands.md へも反映
- Claude Code v2.1.159 は内部のみで仕様反映なし（changelog 1 行のみ）。w23 ダイジェスト未公開のためレビュー対象なし
- mapping/ への影響なし（Codex 側の `/archive` は Claude Code に直接対応する機能がなく、変換ルール変更不要）

---

## 2026-06-01 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**新版なし**。v2.1.158 が依然最新）/ `whats-new/2026-w21` / `whats-new/2026-w22` 公開（前回は 404）
- Codex CLI: GitHub Releases（**新版なし**。0.136.0-alpha.1 / 安定版 0.135.0 のまま）
- スキルエコシステム: 前回 2026-05-24 から 8 日 → Phase 3.5 実行

### 検出された変更と更新内容

#### Claude Code: security-guidance プラグインの追記（w22 featured）
- **specs/claude/skills-and-commands.md** — §4.7「security-guidance プラグイン（公式 / 2026-w22 featured）」を追加。3 段階レビュー（per-edit パターン / end-of-turn diff / commit·push agentic）、`.claude/claude-security-guidance.md` と `.claude/security-patterns.yaml` のカスタムルール、`ENABLE_*` 環境変数によるレイヤー別無効化、登録される hook 構成を記録。要 v2.1.144 以降、Python 3.8 以降。`SessionStart/UserPromptSubmit/PostToolUse(Edit·Write·NotebookEdit, Bash filtered to git commit/push)/Stop` のみで実装されており、harness-harness 側のフック実装の参考になる

#### Claude Code 週次ダイジェスト確認（情報のみ。仕様反映済）
- `2026-w21`（v2.1.143–149）: Auto mode の Pro / Sonnet 4.6 拡大、`/usage` のカテゴリ別内訳、`/code-review` 導入 — いずれも既に specs/claude/changelog.md 該当バージョン項に反映済み
- `2026-w22`（v2.1.150–157）: Opus 4.8、dynamic workflows、security-guidance、fast mode on Opus 4.8 — security-guidance のみ未反映だったため §4.7 として今回追記

#### スキルエコシステム（Phase 3.5）
- **kb/skills/_index.md** — `last_patrol: 2026-06-01` に更新。skills.sh のトップ install 数を 1.5M → 1.8M に更新
- **kb/skills/recommended.md** — `last_checked: 2026-06-01` に更新。`find-skills` の installs 表記を 1.4M → 1.8M に更新
- 観測: skills.sh トップ 10 の半数近くを Microsoft Azure 系（microsoft-foundry, azure-ai, azure-hosted-copilot-sdk, azure-compute）が占める。Tier B 入りは harness-harness の Azure 関連需要が出てから検討（今回は据え置き）
- 観測: `anthropics/skills` リポは 8 日で 140K → 145K stars（+5K）。トップレベル構成変更なし

#### キャッシュ
- **.patrol-cache/url-metadata.json** — w21 / w22 を `published` へ遷移、security-guidance ドキュメント URL を新規登録、スキルエコシステム URL を 2026-06-01 で更新

### 注記
- 今巡回は新バージョンなし。直近の週次ダイジェスト 2 本（w21 / w22）が遅れて公開されたことで、changelog 項目に対応する「公式が featured とした機能」を再確認できた。security-guidance だけが specs/未反映だったため §4.7 で補完
- security-guidance プラグインは hook-only 構成のため、harness-harness の create-harness / diagnose-harness フローでフックレビュー機構の reference 実装として参照可能

---

## 2026-05-31 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.156 / 2.1.157 / 2.1.158** リリース。前回比 3 版増。v2.1.155 は欠番）
- Codex CLI: GitHub Releases（**0.136.0-alpha.1** がプレリリース公開、安定版は 0.135.0 のまま）
- スキルエコシステム: 前回 2026-05-24 から 7 日 → 7日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.156 / v2.1.157 / v2.1.158 追記
- **specs/claude/changelog.md** — v2.1.158（Auto mode の Bedrock/Vertex/Foundry 対応）、v2.1.157（`.claude/skills/` 配下プラグイン自動ロード、`claude plugin init`、`/plugin` autocomplete、`agent` 設定参照、`EnterWorktree` 切替、`OTEL_LOG_TOOL_DETAILS` 拡張、Workflow キーワードトリガー設定、起動バナー整理ほか）、v2.1.156（Opus 4.8 thinking blocks API エラー修正）、v2.1.155 欠番注記
- **specs/claude/configuration.md** — `CLAUDE_CODE_ENABLE_AUTO_MODE` 環境変数追加（v2.1.158）、`OTEL_LOG_TOOL_DETAILS` の `tool_decision.tool_parameters` 拡張注記、Workflow キーワードトリガー設定注記
- **specs/claude/skills-and-commands.md** — `.claude/skills/` 配下のプラグイン自動ロードと `claude plugin init` を「4.2.1」として追記、`/plugin` 引数 autocomplete 注記、`claude agents` の `agent` 設定参照を v2.1.157 注記

#### Codex CLI 0.136.0-alpha.1 追記
- **specs/codex/changelog.md** — プレリリース版エントリ追加（変更点は安定版公開時に追記）

#### キャッシュ
- **.patrol-cache/url-metadata.json** — v2.1.158 観測、0.136.0-alpha.1 観測、関連 URL ハッシュ更新

### 注記
- `.claude/skills/` 配下のプラグイン自動ロード（v2.1.157）はマーケットプレースなしでローカルプラグインを実験できる重要パス。harness-harness 内のスキル開発フローと相性が良い
- `EnterWorktree` の管理 worktree 間切替（v2.1.157）は worktree 運用を簡素化する。docs/conventions.md の worktree 規約に追記の余地あり（別タスク）
- Workflow キーワードトリガー設定（v2.1.157）は v2.1.154 の Dynamic workflows 暴発抑止策。skill/コマンド名に "workflow" を含む場合の影響を要確認
- 0.136.0-alpha.1 はプレリリース。安定版公開まで harness 推奨は 0.135.0 のまま

---


### 巡回対象URL
- Claude Code: changelog（**v2.1.154 リリース 2026-05-28** — Opus 4.8 メジャーリリース）
- Claude Code: llms.txt / w21-w23 digest（依然未公開 — w21 は 12 日超遅延）
- Codex CLI: GitHub Releases（0.135.0 stable のまま、新版なし）
- スキルエコシステム: 前回 2026-05-24 から 6 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.154 追記（Opus 4.8 メジャーリリース）
- **specs/claude/changelog.md** — v2.1.154（2026-05-28）エントリ追加。Opus 4.8、Dynamic workflows、Lean system prompt デフォルト化、`/simplify` クリーンアップ専用化、`/effort` ラベル変更、`/chrome` ブラウザ選択、プラグイン `defaultEnabled` など
- **specs/claude/configuration.md** — `<plugin>.defaultEnabled` 設定追加、`CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` 非推奨注記（2026-06-01 削除予定）
- **specs/claude/skills-and-commands.md** — `/simplify` v2.1.154 でクリーンアップ専用に変更、`/workflows` 追加、`/effort` ラベル変更注記、`/chrome` ブラウザ選択注記
- **specs/claude/mcp.md** — `claude mcp list/get` のパイプ出力時 `⏸ Pending approval` 表示、Stdio MCP サーバーへの `CLAUDE_CODE_SESSION_ID` / `CLAUDECODE=1` 注記

#### キャッシュ
- **.patrol-cache/url-metadata.json** — v2.1.154 観測、関連ページのハッシュ更新

### 注記
- v2.1.154 は Opus 4.8 を伴うメジャーリリース。Lean system prompt のデフォルト化と多肢選択プロンプト抑制は CLAUDE.md / philosophy.md の前提に直接影響しないが、AI挙動への影響として認識
- Dynamic workflows（`/workflows`）は数十〜数百エージェント並列のメカニズム。ハーネス側の長時間ワークフロー設計に新たな選択肢
- `/simplify` の挙動変更で `simplify` スキル（ローカル）と公式 `/simplify` の意味が再び乖離。kb/skills/recommended.md 側のスキル説明見直しは別タスクで検討
- Opus 4.6 fast mode override 環境変数の削除予定（2026-06-01）→ 関連ハーネスで使用がないか確認推奨
- 公式 settings ドキュメントは v2.1.154 反映が遅延。changelog 元データから補完

---

## 2026-05-29 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.153 リリース 2026-05-28** — 小規模リリース）
- Claude Code: llms.txt / w21-w23 digest（依然未公開 — Fri 2026-05-29、w21 は 11 日超遅延）
- Codex CLI: GitHub Releases（**0.135.0 stable リリース 2026-05-28** — alpha.1〜2 を経て同日 stable 昇格）
- スキルエコシステム: 前回 2026-05-24 から 5 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.153 追記（minor feature + fix release）
- **specs/claude/changelog.md** — v2.1.153（2026-05-28）エントリ追加。最終更新日を 2026-05-29 に
- **specs/claude/configuration.md** — `statusLine` に `COLUMNS` / `LINES` 環境変数注記、プラグインマーケットプレース `skipLfs` オプション追加

#### Codex 0.135.0 追記（stable）
- **specs/codex/changelog.md** — 0.135.0（2026-05-28）エントリ追加。`codex doctor` 拡張診断、`/status` リモート接続情報、Vim text-object、`/permissions` named profile、bundled zsh helper、Python SDK Sandbox プリセットなど
- **specs/codex/commands.md** — `/permissions` named profile、`/status` リモート情報、`codex doctor` 拡張診断を該当行に追記

#### キャッシュ
- **.patrol-cache/url-metadata.json** — v2.1.153、Codex 0.135.0 stable 観測、w21/w22/w23 digest 未公開状態を反映

### 注記
- v2.1.153 は minor release（major feature なし）。`statusLine` に端末サイズ環境変数が渡るようになったのは status line スクリプト設計に影響あり
- Codex 0.135.0 で `/permissions` が named permission profile に対応 → ハーネス側で profile プリセット集約パターンを検討余地あり
- w21 digest 未公開が 11 日継続 → llms.txt 公開ペースの恒常的変化の可能性、要観察

---

## 2026-05-28 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.152 リリース 2026-05-27** — v2.1.151 は欠番、changelog 上 v2.1.150 → v2.1.152 に直接ジャンプ）
- Claude Code: llms.txt / w21-w22 digest（依然未公開 — Thu 2026-05-28、w21 は 9 日超遅延）
- Codex CLI: GitHub Releases（**0.134.0 stable リリース 2026-05-26** — alpha.1〜3 を経て stable 昇格）
- スキルエコシステム: 前回 2026-05-24 から 4 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.152 追記（major feature release）
- **specs/claude/changelog.md** — v2.1.152（2026-05-27）エントリ追加。v2.1.151 欠番の注記も付与。最終更新日を 2026-05-28 に
- **specs/claude/skills-and-commands.md** — `disallowed-tools` フロントマター追加、`/code-review --fix` 説明追加、`/simplify` 復活（`/code-review --fix` エイリアス）、`/reload-skills` 新規コマンド追加
- **specs/claude/hooks.md** — `MessageDisplay` フックイベント追加、`SessionStart` フックの `reloadSkills` / `sessionTitle` 機能追加
- **specs/claude/configuration.md** — Managed setting `pluginSuggestionMarketplaces` 追加、環境変数 `OTEL_METRICS_INCLUDE_ENTRYPOINT` 追加

#### Codex 0.134.0 追記（stable）
- **specs/codex/changelog.md** — 0.134.0（2026-05-26）エントリ追加。会話履歴検索、`--profile` 統一、MCP per-server env / OAuth、`readOnlyHint` 並列、Windows TUI VT 修正など
- **specs/codex/configuration.md** — `--profile-v2` → `--profile` への統合と legacy `[profiles]` 拒否を反映
- **specs/codex/mcp.md** — OAuth options、per-server env targeting、`readOnlyHint` 並列実行、connector schema 圧縮を追記
- **specs/codex/commands.md** — セッションピッカーでのローカル会話履歴検索（rollout-backed）を resume 節に追記

#### キャッシュ
- **.patrol-cache/url-metadata.json** — v2.1.152、Codex 0.134.0 stable 観測、w21/w22 digest 未公開状態を反映

### 注記
- v2.1.151 は欠番（公式 changelog で v2.1.150 → v2.1.152 に直接続く）
- v2.1.152 は major feature release。特に `MessageDisplay` フックと `SessionStart.reloadSkills` はハーネスのスキル動的注入パターンを大きく変える可能性あり
- Codex 0.134.0 で `--profile` がプライマリ昇格、legacy v1 `[profiles]` 拒否は **破壊的変更**。既存設定の移行が必要
- mapping/ への波及: `SessionStart.reloadSkills` は Codex 側の SessionStart hook で類似挙動が可能か要追加調査（次回タスク化候補）

---

## 2026-05-24 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.149 リリース 2026-05-22** + **v2.1.150 リリース 2026-05-23**。v2.1.149 は `/usage` カテゴリ別内訳・`/diff` キーボードスクロール・GFM タスクリスト・Enterprise `allowAllClaudeAiMcps` managed setting・`/feedback` の圧縮前文脈包含・複数の PowerShell 権限バイパス＆sandbox セキュリティ修正。v2.1.150 は内部インフラ改善のみ）
- Claude Code: llms.txt / w21 digest（依然未公開 — 巡回時点）
- Codex CLI: GitHub Releases（**0.133.0 stable のまま**。0.134.0 系は alpha.1〜3 が 2026-05-22〜23 にタグ付けされたが alpha 段階のため spec 未反映）
- スキルエコシステム: 前回 2026-05-16 から 8 日 → 7 日超のため Phase 3.5 実施

### 検出された変更と更新内容

#### Claude Code v2.1.149 / v2.1.150 追記
- **specs/claude/changelog.md** — v2.1.149（2026-05-22）と v2.1.150（2026-05-23）エントリ追加。最終更新日を 2026-05-24 に
- **specs/claude/configuration.md** — `allowAllClaudeAiMcps` Managed setting（Enterprise）を v2.1.149 として追加
- **specs/claude/skills-and-commands.md** — `/usage` の説明に v2.1.149 のカテゴリ別内訳（skills / subagents / plugins / MCPサーバー単位コスト）追記

#### スキルエコシステム巡回（Phase 3.5）
- **anthropics/skills** — 140K stars（前回 135K から増加）。`skills/` 配下のカテゴリ構造に変化なし。Tier 昇格対象なし
- **skills.sh** — find-skills 1.5M installs 横ばい。Top-10 構成不変（vercel-labs / anthropics / microsoft Azure 寡占）。新規昇格対象なし
- **kb/skills/_index.md** — `last_patrol: "2026-05-24"`、anthropics/skills の stars 表記を 140K+ に更新

#### キャッシュ
- **.patrol-cache/url-metadata.json** — v2.1.149/150、Codex 0.134.0-alpha 観測、skills.sh / anthropics/skills 巡回結果を反映

### 注記
- v2.1.149 にはセキュリティ修正が複数（PowerShell 権限バイパス、sandbox 書き込み許可リスト、PWD/OLDPWD/DIRSTACK 追跡）含まれる。本変更一覧でも明記
- Codex 0.134.0 系は alpha 段階のため spec/changelog 未反映。stable 昇格時に追記する
- mapping/ への波及なし（新機能はいずれも Claude Code 専有または既存対応の延長）

---

## 2026-05-23 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.147 リリース 2026-05-21** + **v2.1.148 リリース 2026-05-22** — 前回 2026-05-22 巡回時に v2.1.147 の内容を誤って v2.1.146 と記録していた。公式 changelog 上 v2.1.146 は存在せず、2.1.145 → 2.1.147 に直接ジャンプしている）
- Claude Code: llms.txt / w21 digest（依然未公開 — Sat 2026-05-23）
- Codex CLI: GitHub Releases（**0.133.0 stable のまま** — 新規リリースなし。alpha タグも 0.133.0 系のみ）
- スキルエコシステム: 前回 2026-05-16 から 7 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code バージョン番号訂正
- **specs/claude/changelog.md** — `## v2.1.146 (2026-05-21)` セクションを `## v2.1.147 (2026-05-21)` にリネーム。本文は変更なし（公式 v2.1.147 と一致）
- **specs/claude/skills-and-commands.md** — `/code-review` バンドルスキル説明のリネーム経緯注記を `v2.1.146` → `v2.1.147` に訂正

#### Claude Code v2.1.148 追記
- **specs/claude/changelog.md** — v2.1.148（2026-05-22）エントリ追加。内容は Bash ツール exit code 127 リグレッション修正のみ（v2.1.147 起因の hotfix）。最終更新日を 2026-05-23 に

#### キャッシュ
- **.patrol-cache/url-metadata.json** — 巡回キャッシュ更新（v2.1.147 / v2.1.148 / 0.133.0 / w21 依然未公開）

### 注記
- 前回パトロール（PR #77）で v2.1.146 と記録した内容は実体としては v2.1.147 だった。公式 changelog ページが当時のドラフト状態だった可能性が高いが、再取得で v2.1.146 が存在しないことを確認。バージョン番号のみ訂正
- v2.1.148 は v2.1.147 の Bash exit code 127 リグレッションを修正する hotfix のみ。spec 構造に影響なし
- Codex 側は引き続き 0.133.0 が最新。MITM hook 等の新機構の周辺ドキュメントは前回更新で十分

---

## 2026-05-22 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.146 リリース 2026-05-21** — `/simplify` を `/code-review` にリネーム + effort level 引数化 + MCP リスト系ページネーション修正 + `forceLoginOrgUUID`/`forceLoginMethod` enforcement 修正 + Windows PowerShell（winget/Store）の v2.1.124 リグレッション修正）
- Claude Code: llms.txt（w21 digest 依然未公開 — Fri 2026-05-22）
- Codex CLI: GitHub Releases（**0.133.0 stable リリース 2026-05-21** — Goals デフォルト有効化 + `codex remote-control` フォアグラウンド化 + permission profile の inheritance/managed `requirements.toml` + SubagentStart/SubagentStop/MITM hook + compact SessionStart + プラグイン discovery 強化）
- スキルエコシステム: 前回 2026-05-16 から 6 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.146
- **specs/claude/changelog.md** — v2.1.146 エントリ追記、最終更新日を 2026-05-22 に
- **specs/claude/skills-and-commands.md** — バンドルスキル表の `/simplify` を `/code-review [effort]` に置換（リネーム経緯を注記）

#### Codex CLI 0.133.0
- **specs/codex/changelog.md** — 0.133.0 エントリ追記、最終更新日を 2026-05-22 に
- **specs/codex/configuration.md** — 対応イベント表に `SubagentStart` / `SubagentStop` / `MITM` を追加。`SessionStart` に compact 後発火対応を注記。Claude との比較も「6 イベント」に更新

#### mapping/
- **mapping/claude-to-codex.md** — Hooks 比較表を 0.133.0 反映。`SubagentStop`/`SubagentStart` の直接対応、`PreToolUse` の MITM hook による部分代替、未対応イベント数を 14+ → 11+ に修正

#### キャッシュ
- **.patrol-cache/url-metadata.json** — 巡回キャッシュ更新（v2.1.146 / 0.133.0 / w21 依然未公開）

### 注記
- ハーネス内 `.claude/skills/simplify/` などローカルにバンドルスキル参照がある場合は名称変更（Claude Code 内バンドルの `/simplify` → `/code-review`）の影響を受けないが、ドキュメント・チュートリアル類は表現更新を検討
- Codex 0.133.0 の MITM hook は Claude の `PreToolUse` 相当の介入を可能にする初の機構。クロスプラットフォーム対応スキル/フックの設計時に再評価する価値あり
- Codex 0.133.0 でファイルシステム permission entry のデフォルトが `deny` 側へ統一された点は、既存設定の挙動に影響しうる（明示 `allow` していない箇所が拒否される可能性）

---

## 2026-05-21 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.145 リリース 2026-05-19** — 前回巡回時にはまだ追記されていなかった。`claude agents --json` + OTEL agent_id/parent_agent_id + Stop/SubagentStop に `background_tasks`/`session_crons` 追加 + 権限プロンプトバイパスのセキュリティ修正 等）
- Claude Code: llms.txt（w21 digest 依然未公開）
- Codex CLI: GitHub Releases（**0.132.0 stable リリース 2026-05-20** — Python SDK 認証一級化 + `codex exec resume --output-schema` + Goal 継続ループ停止条件 + Windows インストール強化）
- スキルエコシステム: 前回 2026-05-16 から 5 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.145
- **specs/claude/changelog.md** — v2.1.145 エントリ追記、最終更新日を 2026-05-21 に
- **specs/claude/hooks.md** — Stop / SubagentStop の JSON 入力フィールドに `background_tasks` / `session_crons`（v2.1.145+）を追記
- **specs/claude/skills-and-commands.md** — `/plugin` Discover/Browse の事前プレビュー機能（v2.1.145）を追記

#### Codex CLI 0.132.0
- **specs/codex/changelog.md** — 0.132.0 エントリ追記、最終更新日を 2026-05-21 に

#### キャッシュ
- **.patrol-cache/url-metadata.json** — 巡回キャッシュ更新（v2.1.145 / 0.132.0 / w21 依然未公開）

### 注記
- v2.1.145 のセキュリティ修正（Bash の裸の変数代入による権限プロンプトバイパス）は重要。ハーネス側で `PreToolUse` フックを使ったコマンド検査を行っている場合は、新挙動と整合しているか確認推奨
- Codex 0.132.0 は Python SDK 中心のリリースで、CLI コマンド/設定スキーマの変更は最小限。configuration.md / commands.md は変更不要
- mapping/ への影響なし（互換性に影響するシンタクス変更は今回なし）

---

## 2026-05-20 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.144 リリース 2026-05-19** — `/resume` bg対応 + 現セッション専用 `/model` + `/usage-credits` リネーム + 多数のバグ修正）
- Claude Code: whats-new/2026-w20 が公開（v2.1.139-v2.1.142 カバー）
- Codex CLI: GitHub Releases（**変更なし** — 0.131.0 が依然最新）
- スキルエコシステム: 前回 2026-05-16 から 4 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容
- `specs/claude/changelog.md` — v2.1.144 エントリ追加、最終更新日を 2026-05-20 に更新
- `specs/claude/skills-and-commands.md` — `/extra-usage` → `/usage-credits` リネーム反映、`/model` の現セッション専用化（v2.1.144）を注記
- `.patrol-cache/url-metadata.json` — 巡回キャッシュ更新（v2.1.144、w20 公開、w21 未公開）

### 注記
- v2.1.144 は主にバグ修正リリース。設定（settings）・フック（hooks）まわりに新項目はなく、configuration.md / hooks.md は変更不要
- w20 digest は v2.1.139-v2.1.142 を扱うため、v2.1.143 と並行して 5/19 に公開された形。本巡回では digest 内容は changelog 既存記載で十分カバー済み
- mapping/ への影響なし（v2.1.144 の機能は Codex 互換性に影響しないバグ修正中心）

---

## 2026-05-19 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**変更なし** — v2.1.143 が依然最新。w20 digest 未公開）
- Codex CLI: GitHub Releases（**0.131.0 stable リリース 2026-05-18** — 大型機能追加多数）
- スキルエコシステム: 前回 2026-05-16 から 3 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Codex CLI 0.131.0（2026-05-18）— 大型リリース

- **specs/codex/changelog.md** — 0.131.0 エントリ追記、最終更新日を 2026-05-19 に
- **specs/codex/commands.md**:
  - `/goal` に `edit` サブコマンド追加
  - `@` 統合メンション（ファイル / ディレクトリ / プラグイン / スキル）追加
  - グローバルフラグに `--dangerously-bypass-hook-trust` / `--profile-v2` 追加
  - その他サブコマンド表に `codex doctor` / `codex remote-control`（漏れていた）/ `codex plugin marketplace ...` を追加
- **specs/codex/configuration.md**:
  - `[features]` 表を更新: `codex_hooks = true` (default)、新規 `plugin_hooks` / `network_proxy` 追加
  - strict config parsing への変更を注記
  - 新セクション「3.3 profile-v2（レイヤー化プロファイル）」追加
  - Hooks セクションに「7.6 Windows hook command overrides」「7.7 Hook trust の意図的バイパス」「7.8 プラグイン Hooks（デフォルト有効）」追加
- 主な機能:
  - **`codex doctor`**: 診断 CLI（runtime / auth / terminal / network / config / state を横断）
  - **`codex remote-control` の daemon 化**: ライフサイクル管理、ランタイム enable/disable API、registry-backed 環境
  - **プラグインマーケットプレース CLI**: share / share checkout / version 対応
  - **プラグイン Hooks デフォルト有効化**
  - **統一 `@` メンション**: ファイル / ディレクトリ / プラグイン / スキルを単一ピッカーで検索
  - **TUI 強化**: blended token count、権限/承認モード表示、レスポンシブ Markdown table、effective workspace roots
  - **Python SDK リネーム** `openai-codex` / `openai_codex` + 承認モード API
  - **`--profile-v2`** レイヤー化プロファイル
  - **strict config parsing**
  - **`--dangerously-bypass-hook-trust`**
  - **Windows hook command overrides**
  - **削除**: `/collab`、組み込み MCP、`experimental_use_freeform_apply_patch`、`windows_wsl_setup_acknowledged`、`tools.view_image`、レガシー after-tool-use hooks、Issue labeler 非推奨化

#### Claude Code（変更なし）

- v2.1.143 が依然最新。次バージョン未公開
- whats-new w20 digest 未公開（Tue 2026-05-19 時点でも 404）

#### スキルエコシステム

- 前回巡回 2026-05-16 から 7 日以内のため Phase 3.5 スキップ

### 影響なし

- mapping/: `codex doctor` / `codex remote-control` は Claude 側に対応コマンドなし。マッピング対象外
- templates/: profile-v2 はテンプレート化対象ではない（個別プロジェクトの判断に委ねる）

---

## 2026-05-18 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.143 / 2026-05-15 検出** — プラグイン依存強制 + worktree.bgIsolation + PowerShell 改善 + 多数のバグ修正）
- Codex CLI: GitHub Releases（0.130.0 stable のまま。alpha トラックは 0.131.0-alpha.16 → alpha.22 と進行、2026-05-15）
- whats-new w20: **未公開**（Mon 2026-05-18 時点でも 404）
- スキルエコシステム: 前回 2026-05-16 から 2 日 → 7 日以内のため Phase 3.5 スキップ

### 検出された変更と更新内容

#### Claude Code v2.1.143（2026-05-15）— プラグイン依存強制 + worktree.bgIsolation + PowerShell

- **specs/claude/changelog.md** — v2.1.143 エントリ追記、最終更新日を 2026-05-18 に
- **specs/claude/configuration.md**:
  - 設定キー表に `worktree.bgIsolation` を追加（`"none"` でバックグラウンドセッションが作業コピー直接編集）
  - 環境変数表に 3 件追加: `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` / `CLAUDE_CODE_USE_POWERSHELL_TOOL` / `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`
- **specs/claude/skills-and-commands.md** — `/plugin` 行に projected context cost 表示を追記、`claude plugin enable/disable` の依存関係強制を新規追加
- **specs/claude/hooks.md** — Stop hook セクションに「8 連続ブロックで自動終了 + `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` 上限調整可」を追記
- 主な変更:
  - **プラグイン依存関係の強制**: `claude plugin disable` は依存先を拒否 + disable-chain ヒント。`enable` は推移的依存を強制有効化
  - **`worktree.bgIsolation: "none"`**: バックグラウンドセッションが `EnterWorktree` なしで作業コピー直接編集
  - **PowerShell ツール改善**: `-ExecutionPolicy Bypass` がデフォルトで付与、Windows の Bedrock/Vertex/Foundry でデフォルト有効化
  - **`/plugin` projected context cost**: マーケットプレース browse ペインにトークン推定表示
  - **stop hook 安全装置**: 8 連続ブロックでターン終了
  - **`/bg` / ← detach の引数永続化**: `--mcp-config`, `--settings`, `--add-dir`, `--plugin-dir`, `--strict-mcp-config`, `--fallback-model`, `--allow-dangerously-skip-permissions` がバックグラウンド化後も保持されるように
  - **macOS Documents/Desktop/Downloads アクセス修正**: Full Disk Access 付与済みでも「Operation not permitted」になる問題を解消
  - **Shift+Tab に auto モード追加**: アタッチ中のエージェントセッションでのサイクル対象に
  - Worktree クリーンアップが `git worktree remove` 失敗時に `rm -rf` フォールバックを廃止（ファイル喪失防止）

#### Codex CLI（stable 変更なし）

- 0.130.0 stable のまま。alpha は alpha.16 → alpha.22（2026-05-15）と進行。stable 昇格は次回以降フォロー

#### スキルエコシステム

- 前回巡回 2026-05-16 から 7 日以内のため Phase 3.5 スキップ

### 影響なし
- mapping/: Claude/Codex 互換性に影響する変更なし
- agent-teams.md: `claude agents` view（CLI ダッシュボード）は Agent Teams 機能と別物。changelog エントリで吸収済み

---

## 2026-05-16 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.142 / 2026-05-14 検出** — 機能追加 + バグ修正）
- Codex CLI: GitHub Releases（0.130.0 stable のまま。0.131.0-alpha.19 が 2026-05-15 公開、alpha 継続）
- whats-new w20: 未公開（404、Sat 2026-05-16 時点）
- スキルエコシステム: 前回 2026-05-09 から 7 日経過 → 軽巡回実施

### 検出された変更と更新内容

#### Claude Code v2.1.142（2026-05-14）— `claude agents` 拡張 + Fast mode = Opus 4.7

- **specs/claude/changelog.md** — v2.1.142 エントリ追記
- **specs/claude/configuration.md** — 環境変数表に `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` を追加（Fast mode を Opus 4.6 に固定）
- **specs/claude/skills-and-commands.md** — プラグイン配置表に「単一スキル形式」(`<plugin>/SKILL.md`) 追記
- 主な変更:
  - **`claude agents` 新フラグ**: `--add-dir`, `--settings`, `--mcp-config`, `--plugin-dir`, `--permission-mode`, `--model`, `--effort`, `--dangerously-skip-permissions` で dispatch 設定可能
  - **Fast mode デフォルト**: Opus 4.6 → **Opus 4.7**（`CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE=1` で旧版固定）
  - **プラグイン単一スキル化**: ルートレベル `SKILL.md` のみのプラグインを単一スキルとして直接サーフェス
  - `/plugin` 詳細ペインに LSP サーバー表示
  - `/web-setup` が既存 GitHub App 接続を置き換える前に警告
  - 多数のバグ修正（`MCP_TOOL_TIMEOUT` リモート反映、macOS sleep/wake デーモン、`brew upgrade` 後 crash-loop、Windows ネットワークドライブデッドロック、リアクティブコンパクション改善等）

#### Codex CLI（stable 変更なし）

- 0.130.0 stable のまま。0.131.0-alpha.19 が 2026-05-15 公開（alpha.16 → alpha.19 と進行）。stable 昇格は次回以降フォロー

#### スキルエコシステム（軽巡回）

- **kb/skills/_index.md** — last_patrol を 2026-05-16 に更新
  - anthropics/skills: 130K → **135K** stars
  - skills.sh: find-skills が 1.4M → **1.5M** installs
  - Tier 昇格・降格に値する新規スキルなし

### 影響なし
- mapping/: Claude/Codex 互換性に影響する変更なし
- スキルカテゴリ構造、agentskills.io 仕様: 変更なし

---

## 2026-05-15 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.141 / 2026-05-13 検出** — 機能追加 + バグ修正）
- Codex CLI: GitHub Releases（0.130.0 stable のまま。0.131.0-alpha.16 が 2026-05-14 公開、alpha 継続）
- whats-new w20: 未公開（404、週末公開待ち）
- スキルエコシステム: 前回 2026-05-09 から 6 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.141（2026-05-13）— Hook 機能拡張 + 環境変数追加

- **specs/claude/changelog.md** — v2.1.141 エントリ追記
- **specs/claude/hooks.md** — JSON 出力フォーマットに `terminalSequence` フィールド追加（v2.1.141+）
- **specs/claude/configuration.md** — 環境変数表に以下を追加:
  - `CLAUDE_CODE_PLUGIN_PREFER_HTTPS`: GitHub プラグインソースを HTTPS クローン
  - `ANTHROPIC_WORKSPACE_ID`: workload identity federation のワークスペーススコープ
- 主な変更:
  - **Hook `terminalSequence` 出力**: 制御端末なし hook からデスクトップ通知・ウィンドウタイトル・ベル発行
  - **`claude agents --cwd <path>`**: セッションリストを指定ディレクトリにスコープ
  - **`/feedback` 拡張**: 24h/7d セッションを含められる
  - **Rewind「Summarize up to here」**: 過去コンテキスト圧縮
  - **`/bg` バックグラウンドエージェント**: permission mode 維持
  - 多数のバグ修正（権限ダイアログ、autocompact 閾値、vim Ctrl+C、Bedrock auth、MCP 403 表示、Remote Control 401 ループ、light-ansi テーマ等）

#### Codex CLI（stable 変更なし）

- 0.130.0 stable のまま。0.131.0-alpha.16 が 2026-05-14 公開（alpha.9 → alpha.16 と進行）。stable 昇格は次回以降フォロー

---

## 2026-05-14 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.140 / 2026-05-12 検出** — バグ修正中心のマイナーリリース）
- Codex CLI: GitHub Releases（0.130.0 stable のまま。0.131.0-alpha.9 は alpha 継続、stable 昇格なし）
- whats-new w20: 未公開（404、週末公開待ち）
- スキルエコシステム: 前回 2026-05-09 から 5 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.140（2026-05-12）— バグ修正中心
- **specs/claude/changelog.md** — v2.1.140 エントリ追記
- 主な変更:
  - **Agent `subagent_type` マッチが大文字小文字・区切り非依存**: `"Code Reviewer"` → `code-reviewer` 自動解決
  - **`/goal` ハング修正**: `disableAllHooks` / `allowManagedHooksOnly` 有効時のメッセージ表示改善
  - **設定ホットリロード**: シンボリックリンク設定の変更イベント誤帰属修正（spurious `ConfigChange` hook 抑制）
  - **`claude --bg` 安定化**: connection dropped 修正・起動猶予延長
  - **Loop スケジューリング**: バックグラウンドタスクの冗長 wakeup 削減
  - **プラグイン警告**: `plugin.json` 設定で暗黙的に skills/ 等が無視される際の警告
  - **Read ツール**: `offset` 文字列バリデーション緩和
  - **Windows**: `where.exe` 再起動による停止修正
- 大半は内部修正・UI 微調整。specs の hooks / mcp / agent-teams 等への構造的変更なし

#### Codex CLI（変更なし）
- 0.130.0 stable のまま。0.131.0-alpha.9 が 2026-05-12 公開済みだが alpha 継続。次回巡回でフォロー

---

## 2026-05-13 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.139 / 2026-05-11 検出** — 大型リリース）
- Codex CLI: GitHub Releases（0.130.0 stable のまま、0.131.0-alpha.9 まで出ているが stable 昇格なし）
- whats-new w18: 公開済み（前回 404 だった — Apr 27 – May 1 / v2.1.120–126 をカバー）
- whats-new w20: 未公開（404）
- スキルエコシステム: 前回 2026-05-09 から 4 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.139（2026-05-11）— 大型機能追加リリース
- **specs/claude/changelog.md** — v2.1.139 エントリ追記
- **specs/claude/hooks.md**:
  - 3.1 Command ハンドラに `args: string[]`（exec form）追記
  - 6.4 PostToolUse に `continueOnBlock` を追記
- **specs/claude/mcp.md**:
  - 10. 環境変数表に `CLAUDE_PROJECT_DIR`（stdio MCP サーバー向け）を追記
- 主要新機能:
  - **Agent View (Research Preview)**: `claude agents` で全セッション統合表示
  - **`/goal` コマンド**: 完了条件達成までターン継続。`-p`/Remote Control 対応
  - **`/scroll-speed`**: マウスホイール速度ライブ調整
  - **`claude plugin details <name>`**: コンポーネント一覧とトークンコスト推定
  - **Transcript View**: `?`/`{`/`}`/`v` でナビゲーション
  - **Hook `args: string[]` exec form**: シェル解釈なしでコマンド起動
  - **PostToolUse `continueOnBlock`**: ブロック後もターン継続可
  - **MCP stdio に `CLAUDE_PROJECT_DIR`**: hooks と同じ環境変数を渡す
  - **API キー認証時の機能制限**: Remote Control / `/schedule` / claude.ai MCP コネクタ / 通知を無効化
  - **設定ホットリロード**: シンボリックリンク `~/.claude/settings.json` 編集検出
  - **Skill 権限ワイルドカード**: `Skill(name *)` がプレフィックス一致に修正
- 重要バグ修正: クレデンシャル × `forceRemoteSettingsRefresh` デッドロック、`autoAllowBashIfSandboxed` のシェル展開、Hook 端末書き込みによるプロンプト破壊、HTTP/SSE MCP メモリ無制限増、`/model` ピッカーのデフォルト表示、stream idle timeout 偽エラー、複数画像ペースト等

#### Codex CLI（変更なし）
- 0.130.0 stable のまま。0.131.0 は alpha.9 まで進行中だが stable 昇格なし。次回巡回でフォロー

#### whats-new w18 公開
- 前回 404 だった Week 18（Apr 27 – May 1）が公開（出版が w17 → w19 と飛んだ後にバックフィル）
- カバー範囲は v2.1.120-126（既に changelog 反映済み）。仕様更新不要

#### スキルエコシステム（Phase 3.5 スキップ）
- 前回 2026-05-09 から 4 日のため今回はスキップ

---

## 2026-05-10 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.136 / 2026-05-08、v2.1.137 / 2026-05-09、v2.1.138 / 2026-05-09 検出**）
- Codex CLI: GitHub Releases（**0.130.0 / 2026-05-08 ステーブル昇格** — 前回 alpha のみだったが今回 stable 公開）
- whats-new w19: 公開済み（v2.1.128–v2.1.136 をカバー、5月4–8日の週次ダイジェスト）
- スキルエコシステム: 前回 2026-05-09 から 1 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.136（2026-05-08）— 主要変更
- **specs/claude/changelog.md** — v2.1.136 / v2.1.137 / v2.1.138 を追記
- **specs/claude/configuration.md**:
  - `autoMode.hard_deny` 追加: ユーザー意図や allow 例外に関わらず無条件にマッチアクションをブロック
  - `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` 環境変数を追加（OpenTelemetry 経由のセッション品質サーベイ再有効化）
- 重要バグ修正多数: MCP サーバーが `/clear` で消失する問題、並列クレデンシャル書き込みのログインループ、複数 MCP OAuth リフレッシュトークンの消失、`@` ファイルピッカーの 100 件超対応、`AskUserQuestion` 配列回答ドロップ、`CronList` 出力欠落、wide markdown table・CJK・bracketed paste のレンダリング不整合等
- プラグインマーケットプレース削除キー: `r`（リトライと衝突）→ `d` に変更（破壊的変更）

#### Claude Code v2.1.137 / v2.1.138（2026-05-09）
- v2.1.137: VSCode Windows でのアクティベート失敗修正
- v2.1.138: 内部修正のみ

#### Codex CLI 0.130.0（2026-05-08）— ステーブル昇格
- **specs/codex/changelog.md** — 0.130.0 を追記（前回 alpha のみだった）
- 主要変更:
  - `codex remote-control` コマンド追加（ヘッドレス・リモート制御 app-server エントリポイント）
  - プラグイン詳細にバンドル hooks 表示、共有でリンクメタデータと discoverability コントロール公開
  - App-server: 大規模スレッドの unloaded/summary/full ターン項目ビューでページング対応
  - Bedrock auth が `aws login` profile credentials を利用可能
  - `view_image` がマルチ環境で選択環境経由でファイル解決
- 重要バグ修正: ライブスレッドの設定再読込、`apply_patch` 部分失敗のターン差分保持、ThreadStore 経由のサマリ・resume・fork 改善、Windows サンドボックスでデスクトップランタイムバイナリキャッシュアクセス

#### スキルエコシステム（Phase 3.5 スキップ）
- 前回巡回 2026-05-09 から 1 日のため今回はスキップ

---

## 2026-05-09 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.133 / 2026-05-07 検出**）
- Codex CLI: GitHub Releases（0.130.0-alpha.7 / 2026-05-08 — alpha のみのため見送り）
- whats-new w18: 未公開（404 継続。w13-w17 のみ llms.txt に掲載）
- スキルエコシステム: 前回 2026-05-02 から 7 日経過 → Phase 3.5 実施

### 検出された変更と更新内容

#### Claude Code v2.1.133（2026-05-07）
- **specs/claude/changelog.md** — v2.1.133 を追記
- **specs/claude/configuration.md** — 設定キー 4 件を追加
  - `worktree.baseRef`（`fresh` | `head`）: ワークツリーのベース参照を選択。デフォルトが `fresh`（=`origin/<default>`）に戻り、v2.1.128 以降の `HEAD` 挙動から変更
  - `sandbox.bwrapPath` / `sandbox.socatPath`（Linux/WSL Managed 設定）: バイナリのカスタムパス指定
  - `parentSettingsBehavior`（admin tier）: SDK `managedSettings` のポリシーマージ制御
- **specs/claude/hooks.md** — hooks 入力 JSON に `effort.level` フィールドを追加。`$CLAUDE_EFFORT` 環境変数も hooks コマンドと Bash サブプロセスから参照可能と明記
- 重要バグ修正: 並列セッション 401 デッドエンド、MCP OAuth フローの proxy 尊重、Remote Control 中断、サブエージェントのスキル発見等

#### Codex CLI（変更なし）
- 0.130.0-alpha.7 が 2026-05-08 公開されたが alpha のため specs 反映は見送り

#### スキルエコシステム（Phase 3.5 実施）
- **kb/skills/_index.md** — `last_patrol` を 2026-05-09 に更新。anthropics/skills を 127K→130K stars、agentskills.io を 37+→38+ platforms、find-skills 1.3M→1.4M installs へ追随
- **kb/skills/recommended.md** — find-skills の installs 表記を 1.4M に更新
- 推薦スキル昇格/降格は今回なし。leaderboard 上位構成（vercel-labs、anthropics、microsoft Azure）は不変

---

## 2026-05-08 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.132 / 2026-05-06 検出**。前回巡回時点で未確認）
- Codex CLI: GitHub Releases（**0.129.0 / 2026-05-07 ステーブル昇格** — 前回 alpha のみだったが今日 stable 公開）
- whats-new w18: 未公開（404 継続）
- スキルエコシステム: 前回 2026-05-02 から 6 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.132（2026-05-06）
- **specs/claude/changelog.md** — v2.1.132 を追記（新規エントリ）
  - **`CLAUDE_CODE_SESSION_ID`**: Bash サブプロセスに自動設定。hooks の `session_id` と一致
  - **`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1`**: フルスクリーンレンダラーをオプトアウト
  - **「Pasting…」フッターヒント**: Ctrl+V 画像ペースト中の状態表示
  - 重要バグ修正多数（外部 SIGINT graceful shutdown、`--resume` 絵文字 truncation、プランモード再開時の `--permission-mode` 無視、Cursor/VS Code/JetBrains マウススクロール、stdio MCP メモリリーク 10GB+、Bedrock/Vertex 1h プロンプトキャッシュ 400 エラー等）
- **specs/claude/configuration.md** — 環境変数 2 件を追加（`CLAUDE_CODE_SESSION_ID`、`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN`）

#### Codex CLI 0.129.0（2026-05-07）
- **specs/codex/changelog.md** — 0.129.0 を追記（新規エントリ）
  - **TUI Vim モーダル編集**: `/vim` コマンド対応
  - **ワークフロー再開ピッカー刷新**
  - **Raw scrollback モード** / **ワークスペース対応 `/diff`** / **ステータスラインのテーマ対応**
  - **プラグイン管理拡張**（ワークスペース共有、アクセス制御等）
  - **実験的 Goals 機能**: 検出可能化と一時停止状態の永続化
  - **Linux サンドボックス強化**: スタンドアロン `bwrap` フォールバック、Bubblewrap 0.11.2 vendoring

---

## 2026-05-07 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.129 / 2026-05-06、v2.1.131 / 2026-05-06 検出**）
- Codex CLI: GitHub Releases（0.129.0-alpha.9 / 2026-05-06、ステーブル未公開のため見送り）
- whats-new w18: 未公開（404 継続）
- スキルエコシステム: 前回 2026-05-02 から 5 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.129（2026-05-06）
- **specs/claude/changelog.md** — v2.1.129 を追記
  - **`--plugin-url <url>` 追加**: URL から `.zip` プラグインアーカイブを取得
  - **`CLAUDE_CODE_FORCE_SYNC_OUTPUT=1` 追加**: 同期出力を強制有効化（Emacs `eat` 等向け）
  - **`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` 追加**: Homebrew/WinGet で BG アップグレード→再起動プロンプト
  - **プラグインマニフェスト**: `themes`/`monitors` は `experimental` 配下が推奨に
  - **ゲートウェイ `/v1/models` 探索オプトイン化**: `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`（v2.1.126〜128 は自動）
  - **`Ctrl+R` 履歴ピッカー全プロジェクト検索デフォルトに復帰**（pre-2.1.124 挙動）
  - **3P デプロイメントでスピナーチップから 1P 案内除去**
  - **`skillOverrides` 設定の修正**: 実装が機能するように（`off`/`user-invocable-only`/`name-only`）
  - **`claude_code.pull_request.count` OTel メトリック**: MCP ツール経由 PR 作成もカウント
  - **ポリシー拒否エラーに API Request ID 付与**
  - 重要バグ修正多数（400 エラー誤表示、`/clear` 後タブタイトル、`/agents` 矢印キー、`/branch` セッション ID 欠落、OAuth リフレッシュ競合、1h プロンプトキャッシュ TTL ダウングレード、`Bash(mkdir *)` 等の許可ルール、VSCode `/clear` 等）
- **specs/claude/configuration.md** — `skillOverrides` 設定キーと新環境変数 3 件を追加
- **specs/claude/skills-and-commands.md** — `--plugin-url` フラグを追加

#### Claude Code v2.1.131（2026-05-06）
- **specs/claude/changelog.md** — v2.1.131 を追記
  - VS Code 拡張: Windows でアクティブ化失敗を修正（`createRequire` polyfill バグ）
  - Mantle エンドポイント認証修正（`x-api-key` ヘッダ欠落）
- 注: v2.1.130 はステーブル未リリース（バージョンスキップ）

#### Codex CLI（変更なし、ステーブル未公開）
- 0.129.0-alpha.9（2026-05-06）が GitHub Releases に出ているが、alpha のため specs/codex/changelog.md には未反映
- ステーブル昇格時に再評価

---

## 2026-05-06 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.128 検出 / 2026-05-04**）
- Codex CLI: GitHub Releases（0.129.0-alpha.6 / 2026-05-05、ステーブル未公開のため見送り）
- whats-new w18: 未公開（404 継続、v2.1.120-128 期間がカバーされていない状態が続く）
- スキルエコシステム: 前回 2026-05-02 から 4 日経過 → Phase 3.5 スキップ（7 日以内）

### 検出された変更と更新内容

#### Claude Code v2.1.128（2026-05-04）
- **specs/claude/changelog.md** — v2.1.128 を追記。最終更新日を 2026-05-06 に更新
  - **`/mcp` がツール数表示**（0 ツール接続のサーバーを警告マーク）
  - **`--plugin-dir` が `.zip` 受理**（プラグインアーカイブ直接ロード）
  - **`--channels` が console API キー認証で動作**（managed settings の `channelsEnabled: true`）
  - **`/model` ピッカー整理**（Opus 4.7 重複統合、表示を「Opus」に）
  - **サブプロセスへの `OTEL_*` 継承を停止**（OTLP エンドポイント誤継承を解消）
  - **MCP `workspace` が予約サーバー名に**
  - **MCP 再接続のツール再公開を要約化**（サーバープレフィックスごとに集約）
  - **SDK ホスト向け `localSettings` サジェスト永続化**（Bash 権限の Always allow が `.claude/settings.local.json` へ）
  - **`EnterWorktree` が local HEAD 起点**（未 push コミット欠落を修正）
  - **オートモード分類器エラーにヒント追加**
  - 重要バグ修正多数（>10MB 入力 stdin クラッシュループ、1M コンテキスト autocompact 誤ブロック、並列シェルツールのキャンセル波及、サブエージェントサマリのキャッシュ取りこぼし＆idle 連発、MCP stdio 引数破損 等）
- 注: v2.1.127 はステーブル未リリース（バージョンスキップ）

#### Codex CLI（変更なし、ステーブル未公開）
- 0.129.0-alpha.6（2026-05-05）が GitHub Releases に出ているが、alpha のため specs/codex/changelog.md には未反映
- ステーブル昇格時に再評価

#### スキルエコシステム（Phase 3.5 スキップ）
- 前回 2026-05-02 から 4 日経過、7 日以内のため Phase 3.5 全体をスキップ
- 次回巡回（2026-05-09 以降）で再評価

### スキップ理由
- **specs/claude/configuration.md** — v2.1.128 で MCP `workspace` 予約サーバー名や `--plugin-dir` の zip 受理など個別仕様変更はあるが、いずれも changelog に集約。設定キー一覧自体に新キーの追加はないため見送り
- **specs/claude/mcp.md** — `workspace` 予約名・再接続時ツール要約は MCP の実装挙動変更で、specs としては「予約サーバー名」「再接続挙動」を別途まとめる価値はあるが、今回は changelog で十分。次回大幅変更があれば統合検討
- **specs/codex/*** — 0.128.0 以降のステーブル未リリース
- **kb/skills/*** — 7 日以内のため Phase 3.5 スキップ

---

## 2026-05-02 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.126 検出 / 2026-05-01**）
- Codex CLI: GitHub Releases（0.129.0-alpha.2 のみ、ステーブル未公開のため見送り）
- whats-new w18: 未公開（404 継続）
- スキルエコシステム: 前回 2026-04-25 から 7 日経過 → Phase 3.5 実施

### 検出された変更と更新内容

#### Claude Code v2.1.126（2026-05-01）
- **specs/claude/changelog.md** — v2.1.126 を追記。最終更新日を 2026-05-02 に更新
  - **`/model` ピッカーがゲートウェイの `/v1/models` から取得**（`ANTHROPIC_BASE_URL` が Anthropic 互換ゲートウェイ時）
  - **`claude project purge [path]` 追加**（プロジェクト状態の一括削除、`--dry-run` / `-y` / `-i` / `--all` 対応）
  - **`--dangerously-skip-permissions` の保護解除拡張**（`.claude/`、`.git/`、`.vscode/`、シェル設定ファイル等）
  - **`claude auth login` の OAuth コードペースト対応**（WSL2/SSH/コンテナ向け）
  - **OpenTelemetry**: `claude_code.skill_activated` がユーザー入力スラッシュコマンドでも発火、新属性 `invocation_trigger`
  - **オートモードのスピナー赤色化**（権限チェック停滞の可視化）
  - **Windows: PowerShell 7 検出範囲拡大**（Microsoft Store / PATH 未設定 MSI / .NET global tool）
  - **Windows: PowerShell をプライマリシェルに**（PowerShell ツール有効時、Bash ではなく PowerShell 主シェル）
  - **重要セキュリティ修正**: `allowManagedDomainsOnly` / `allowManagedReadPathsOnly` が高優先度 managed-settings ソースで無視される問題
- 注: v2.1.124 / v2.1.125 はステーブル未リリース（バージョンスキップ）

#### Codex CLI（変更なし、ステーブル未公開）
- 0.129.0-alpha.2（2026-05-01）が GitHub Releases に出ているが、alpha のため specs/codex/changelog.md には未反映
- ステーブル昇格時に再評価

#### スキルエコシステム（Phase 3.5）
- **kb/skills/_index.md** — `last_patrol` を 2026-05-02 に、anthropics/skills の Stars を 127K+、skills.sh トップ find-skills を 1.3M installs に更新
- 顕著な新興スキルなし（find-skills が引き続き圧倒的トップ、Tier 変更不要）
- agentskills.io 確認プラットフォーム数は 37+ で横ばい

### スキップ理由
- **specs/claude/configuration.md** — v2.1.126 で OpenTelemetry イベント仕様変更（新属性 `invocation_trigger`）はあるが、本ファイルは設定キー一覧のため詳細イベント仕様は changelog に集約。今回は更新見送り
- **specs/codex/*** — 0.128.0 以降のステーブル未リリース
- **kb/skills/recommended.md** — Tier 変更を伴う動きなし

---

## 2026-05-01 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（v2.1.123 のまま変更なし）
- Codex CLI: GitHub Releases（**0.128.0 検出 / 2026-04-30**）
- whats-new w18: 未公開（404）
- スキルエコシステム: 前回 2026-04-25 から 6 日のため Phase 3.5 はスキップ（次回 2026-05-02 以降）

### 検出された変更と更新内容

#### Codex CLI 0.128.0（2026-04-30）
- **specs/codex/changelog.md** — 0.128.0 を追記。最終更新日を 2026-05-01 に更新
  - **`/goal` 永続化ワークフロー**（create/pause/resume/clear、app-server API・モデルツール統合）
  - **`codex update` コマンド**（CLI 自身のアップグレード）
  - **TUI キーマップ設定可能化**、プランモードナッジ強化、Action-required ターミナルタイトル
  - **アクティブターン中の `/statusline` / `/title` 編集**
  - **権限プロファイル拡張**（ビルトインデフォルト・サンドボックス CLI プロファイル選択・cwd 制御）
  - **プラグインワークフロー強化**（マーケットプレースインストール、リモートバンドルキャッシュ、プラグインバンドル hooks）
  - **外部エージェントセッションインポート**、MultiAgentV2 設定明示化
- **specs/codex/commands.md** — `/goal` と `/title` を 1.4 セッション管理セクションに追加、`codex update` をセクション 4 に追加
- 注: 0.126.0 / 0.127.0 はステーブル未リリース（alpha のみ）。0.125.0 → 0.128.0 にバージョンスキップ

### スキップ理由
- **specs/codex/configuration.md** — 0.128.0 で言及された権限プロファイルの「ビルトインデフォルト」は既存の `default_permissions` キーで概ねカバーされており、新規キーの明確化情報がリリースノートにないため今回は更新見送り
- **specs/claude/*** — v2.1.123 から進展なし（前回巡回でカバー済み）
- **kb/skills/*** — 7日ルール内のためスキップ

---

## 2026-04-30 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.122 / v2.1.123 検出**）
- Codex CLI: changelog（0.125.0 のまま変更なし）
- whats-new w18: 未公開（404）
- スキルエコシステム: 前回 2026-04-25 から 5 日のため Phase 3.5 はスキップ（次回 2026-05-02 以降）

### 検出された変更と更新内容

#### Claude Code v2.1.123（2026-04-29）
- **specs/claude/changelog.md** — v2.1.123 を追記
  - `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1` 設定時の OAuth 401 リトライループを修正（単発バグ修正）

#### Claude Code v2.1.122（2026-04-28）
- **specs/claude/changelog.md** — v2.1.122 を追記
  - **`ANTHROPIC_BEDROCK_SERVICE_TIER` 環境変数追加**（Bedrock のサービスティア `default` / `flex` / `priority` を選択し `X-Amzn-Bedrock-Service-Tier` ヘッダ送信）
  - **`/resume` の検索ボックスで PR URL ペースト → 該当セッション検索**（GitHub / GitHub Enterprise / GitLab / Bitbucket）
  - **`/mcp`**: 同一 URL 手動登録サーバーで隠れた claude.ai connector を表示し、重複削除のヒントを出す
  - **OpenTelemetry**: `api_request` / `api_error` の数値属性を文字列ではなく数値で出力／`@`-mention 解決の `claude_code.at_mention` イベント追加
  - rewound タイムライン由来の `/branch` フォーク失敗、Bedrock ARN の `/model` Effort、Vertex AI 構造化出力エラー、`count_tokens` 400、画像リサイズの 2000px 制限不具合などを修正
- **specs/claude/configuration.md** — 環境変数表に `ANTHROPIC_BEDROCK_SERVICE_TIER` を追加

### スキップ理由
- **specs/claude/hooks.md / mcp.md / skills-and-commands.md** — v2.1.122 / v2.1.123 では仕様面で言及すべき新規追加なし（バグ修正・OpenTelemetry 微調整中心）
- **Codex CLI specs** — 0.125.0 から進展なし
- **kb/skills/*** — 7日ルール内のためスキップ

### 引き続きウォッチ対象
- whats-new w18 ダイジェスト（v2.1.120-123 をカバー予定、現時点 404）
- Codex 0.126.0 のリリース
- スキルエコシステム週次巡回（次回 2026-05-02 以降）

---

## 2026-04-29 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（**v2.1.120 / v2.1.121 検出**）、llms.txt（**w16・w17 ダイジェスト公開済み**）
- Codex CLI: changelog（0.125.0 のまま変更なし）
- スキルエコシステム: 前回 2026-04-25 から 4 日のため Phase 3.5 はスキップ（次回 2026-05-02 以降）

### 検出された変更と更新内容

#### Claude Code v2.1.121（2026-04-28）
- **specs/claude/changelog.md** — v2.1.121 を追記
  - **MCP `alwaysLoad` オプション**（tool-search ディファード化を回避し常時ロード）
  - **`claude plugin prune` / `plugin uninstall --prune`**（孤立した自動インストール依存の削除）
  - **`/skills` の type-to-filter 検索ボックス**（長いスキル一覧の絞り込み）
  - **`PostToolUse` フックの `hookSpecificOutput.updatedToolOutput` が全ツール対応**（従来は MCP のみ）
  - フルスクリーンモードのスクロール戻り防止、オーバーフローダイアログのスクロール対応
  - `CLAUDE_CODE_FORK_SUBAGENT=1` が SDK / `claude -p` の非インタラクティブセッションでも有効
  - `--dangerously-skip-permissions` で `.claude/skills/`, `.claude/agents/`, `.claude/commands/` を承認スキップ
  - **MCP サーバー起動時の transient error は最大 3 回自動リトライ**（従来は接続不能で停止）
  - `mcp_authenticate` SDK が `redirectUri` 対応
  - **OpenTelemetry**: LLM スパンに `stop_reason` / `gen_ai.response.finish_reasons` / `user_system_prompt`（`OTEL_LOG_USER_PROMPTS` 有効時）追加
  - 多数の重要メモリリーク・回帰修正（画像処理・`/usage`・`--resume`・Bash ツールの worktree 削除耐性 等）
- **specs/claude/configuration.md** — `mcpServers.<name>.alwaysLoad` と `OTEL_LOG_USER_PROMPTS` 環境変数を追加
- **specs/claude/hooks.md** — `PostToolUse` 出力例（`updatedToolOutput`）を追加
- **specs/claude/mcp.md** — 「3.5 サーバーごとの追加オプション」（`alwaysLoad`）と「3.6 接続の堅牢性」（自動リトライ・`redirectUri`）を追加
- **specs/claude/skills-and-commands.md** — `/skills` と `/plugin` の説明に v2.1.121 機能を追記

#### Claude Code v2.1.120（2026-04-28）
- **specs/claude/changelog.md** — v2.1.120 を追記
  - **Windows: Git Bash 不要**（PowerShell フォールバック）
  - **`claude ultrareview [target]` 非インタラクティブサブコマンド**（CI / スクリプトから `/ultrareview` 実行、`--json` 対応）
  - **スキルの `${CLAUDE_EFFORT}` 変数展開**（現在の effort level を埋め込み）
  - **`AI_AGENT` 環境変数**（サブプロセスへ伝播。`gh` などが Claude Code を識別可能に）
  - `claude plugin validate` が marketplace.json/plugin.json の `$schema`/`version`/`description` を受理
  - auto モードの auto-compact 表示を `auto` に変更（誤解を招くトークン値廃止）
  - VSCode: `/usage` がネイティブ Account & Usage ダイアログを開く
  - 非フルスクリーンモードのスクロールバック重複や Esc / `/rewind` 系の対話オーバーレイ修正
- **specs/claude/configuration.md** — `AI_AGENT` 環境変数を追加
- **specs/claude/skills-and-commands.md** — `${CLAUDE_EFFORT}` を変数展開表に、`/ultrareview` に非インタラクティブモードを追記

#### Codex CLI（変更なし）
- 0.125.0（2026-04-24）が依然最新。specs/codex/* の更新は不要

### llms.txt の追従
- whats-new w16（4/13–17）、w17（4/20–24）が公開
- w16: Opus 4.7 / xhigh effort、Routines、`/usage` breakdown、`/ultrareview`、ネイティブバイナリ
- w17: `/ultrareview` 研究プレビュー、自動セッションリキャップ、カスタムカラーテーマ、claude.ai/code リデザイン
- いずれも v2.1.105〜v2.1.119 の範囲で既に specs に反映済み（追加更新は不要と判断）

## 2026-04-26 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（v2.1.119 のまま変更なし）、llms.txt（whats-new は依然 w13/w14/w15、w16 はまだ未公開）
- Codex CLI: changelog（**0.125.0 検出**）
- スキルエコシステム: 前回 2026-04-25 から 1 日のため Phase 3.5 はスキップ

### 検出された変更と更新内容

#### Codex CLI（0.125.0、4月24日）
- **specs/codex/changelog.md** — 0.125.0 を追記
  - App-server 統合の拡張（Unix socket、ページネーション対応 resume/fork、固定環境、リモートスレッド設定）
  - **権限プロファイルの永続化**（TUI セッション・ユーザーターン・MCP サンドボックス・shell エスカレーション・app-server API を横断）
  - リモートプラグインインストール / マーケットプレースアップグレード対応
  - AWS / Bedrock アカウント状態を含むモデル検出
  - `codex exec --json` に reasoning-token 報告追加
  - ロールアウトトレーシング（マルチエージェント関係を記録するデバッグリデューサー）
  - 設定スキーマ強化: MultiAgentV2 スレッド制限競合検出、相対エージェント設定パス解決、MCP bearer-token 非対応フィールド隠蔽、`js_repl` 無効 MIME 型拒否
  - WebSocket 切断軽減、TUI `/review` 中断時のフリーズ修正、Windows サンドボックス互換性向上
- **specs/codex/configuration.md** — 5.3 権限プロファイルセクションに 0.125.0 での永続化挙動を追記

#### Claude Code（変更なし）
- v2.1.119 が依然最新。w16 weekly digest はまだ公開されていない
- specs/claude/* の更新は不要

## 2026-04-25 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（v2.1.119 検出）、llms.txt（whats-new は依然 w13-w15）
- Codex CLI: changelog（0.124.0 検出）
- スキルエコシステム: 前回2026-04-18から7日経過のため巡回実施

### 検出された変更と更新内容

#### Claude Code（v2.1.119、4月23日）
- **specs/claude/changelog.md** — v2.1.119 を追記
  - `/config` 設定（theme, editor mode, verbose 等）が `~/.claude/settings.json` に永続化
  - `prUrlTemplate` 設定追加（github.com 以外のカスタムPRレビューURL）
  - `CLAUDE_CODE_HIDE_CWD` 環境変数追加（起動ロゴの cwd 非表示）
  - `--from-pr` が GitLab/Bitbucket/GitHub Enterprise URL を受付
  - `--print` がエージェントの `tools:` / `disallowedTools:` フロントマターを尊重
  - PowerShell ツールの permission モード自動承認
  - Hooks: `PostToolUse`/`PostToolUseFailure` に `duration_ms` 追加
  - Vim INSERT 中の Esc 挙動改善、`owner/repo#N` ショートハンドの動的ホスト対応
  - OpenTelemetry: `tool_result`/`tool_decision` に `tool_use_id`、`tool_result` に `tool_input_size_bytes`
  - MCP OAuth discovery の非JSON応答、fullscreen スクロールスナップ戻り、Vertex AI ツール検索デフォルト無効化等の修正
- **specs/claude/configuration.md** — `prUrlTemplate` と `CLAUDE_CODE_HIDE_CWD` を追加
- **specs/claude/hooks.md** — `PostToolUse` / `PostToolUseFailure` 入力に `duration_ms` 追記

#### Codex CLI（0.124.0、4月23日）
- **specs/codex/changelog.md** — 0.124.0 を追記
  - TUI 推論コントロールのクイック操作（`Alt+,` / `Alt+.`）
  - app-server が複数環境を管理（ターンごとに環境選択）
  - **Amazon Bedrock ファーストクラス対応**（OpenAI互換プロバイダーとして AWS SigV4 署名込み）
  - リモートプラグインマーケットプレースの一覧・読み取り対応
  - **Hooks が正式化（stable）**、`config.toml` インライン設定と managed `requirements.toml` 対応
  - 対象 ChatGPT プランで Fast サービスティアがデフォルト
- **specs/codex/configuration.md** — セクション 7 を「Hooks（実験的機能）」から「Hooks」に変更、0.124.0 での正式化と `requirements.toml` 対応を記載

#### スキルエコシステム
- **kb/skills/_index.md** — last_patrol を 2026-04-25 に更新、anthropics/skills stars を 123K+ に、find-skills installs を 1.2M に更新
- **kb/skills/recommended.md** — last_checked を 2026-04-25 に、frontend-design installs を 334K に、find-skills installs を 1.2M に更新
- skills.sh トレンド観測: vercel-react-best-practices（346K）、remotion-best-practices（264K）、microsoft-foundry（247K）が台頭。いずれもベンダー/フレームワーク特化のため Tier A/B 昇格は見送り
- agentskills.io: 37 プラットフォーム（fast-agent, Google AI Edge Gallery, nanobot 等を確認）

#### その他
- `whats-new/2026-w16` は依然未公開

---

## 2026-04-24 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（v2.1.118 検出）、llms.txt（whats-new は w13-w15 のまま）
- Codex CLI: changelog（0.123.0 検出）
- スキルエコシステム: 前回巡回(2026-04-18)から6日経過のためスキップ（7日ルール）

### 検出された変更と更新内容

#### Claude Code（v2.1.118、4月23日）
- **specs/claude/changelog.md** — v2.1.118 を追記
  - vim ビジュアルモード（`v`/`V`）追加
  - `/cost` と `/stats` が `/usage` に統合（両コマンドはタイピングショートカットとして残存）
  - カスタムテーマ機能（`/theme` で作成・切替、`~/.claude/themes/` 直接編集、プラグインの `themes/` 提供）
  - **フックが MCP ツールを直接呼び出し可能**（`type: "mcp_tool"` 新設）
  - `DISABLE_UPDATES` 環境変数追加（`DISABLE_AUTOUPDATER` より厳格）
  - `wslInheritsWindowsSettings` ポリシーキーで WSL が Windows 側 managed settings を継承
  - Auto mode `"$defaults"` で組み込みルールを置換せずカスタム追加可能
  - `claude plugin tag` コマンド追加（バージョンバリデーション付きリリースタグ作成）
  - `--continue`/`--resume` が `/add-dir` で追加したディレクトリを持つセッションも検索
  - OAuth（`expires_in` 省略、keychain レース、`.credentials.json` 破損）等の重要修正
- **specs/claude/configuration.md** — `DISABLE_UPDATES` と `wslInheritsWindowsSettings` を追加、`autoMode` の `$defaults` 記法を注記
- **specs/claude/hooks.md** — セクション 3.5 に MCP Tool ハンドラ（`type: "mcp_tool"`）を追加
- **specs/claude/skills-and-commands.md** — `/cost`/`/stats` を `/usage` のショートカットに変更

#### Codex CLI（0.123.0、4月23日）
- **specs/codex/changelog.md** — 0.123.0 を追記
  - Amazon Bedrock プロバイダー対応
  - `/mcp verbose` 診断コマンドで MCP 接続状態とツール一覧を詳細出力
  - バックグラウンドエージェントの Realtime ハンドオフ強化
  - `/copy` のロールバック後動作、VS Code WSL の Unicode 入力修正

#### その他
- `whats-new/2026-w16` は依然未公開

---

## 2026-04-23 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog（v2.1.117 検出）、llms.txt（whats-new は w13-w15 のまま）
- Codex CLI: changelog（0.122.0 のまま、変更なし）
- スキルエコシステム: 前回巡回(2026-04-18)から5日経過のためスキップ（7日ルール）

### 検出された変更と更新内容

#### Claude Code（v2.1.117、4月22日）
- **specs/claude/changelog.md** — v2.1.117 を追記
  - ネイティブビルドで `Glob`/`Grep` ツール廃止、`bfs`/`ugrep` を Bash 経由で使用（macOS/Linux のみ）
  - `CLAUDE_CODE_FORK_SUBAGENT=1` で外部ビルドのフォークサブエージェント解禁
  - エージェント `mcpServers:` フロントマターがメインスレッド `--agent` でも有効
  - `/model` 選択が再起動後も永続化、`/resume` が stale 大規模セッションを要約提案
  - Pro/Max の Opus 4.6 / Sonnet 4.6 デフォルト effort が `high` に昇格
  - `blockedMarketplaces` / `strictKnownMarketplaces` がプラグイン操作全般で強制適用
  - `cleanupPeriodDays` 対象に `tasks/`、`shell-snapshots/`、`backups/` を追加
  - OpenTelemetry に `command_name` / `command_source` / `effort` 属性追加（`OTEL_LOG_TOOL_DETAILS=1` で redact 解除）
  - Opus 4.7 `/context` を 1M コンテキスト前提に修正（早期 autocompact 解消）
  - OAuth/WebFetch/プロキシ/Bedrock 等の重要バグ修正
- **specs/claude/configuration.md** — `CLAUDE_CODE_FORK_SUBAGENT`、`OTEL_LOG_TOOL_DETAILS` を環境変数表に追加、`cleanupPeriodDays` の対象拡張を注記

#### Codex CLI
- 新バージョンなし（0.122.0 のまま）

#### その他
- `whats-new/2026-w16` は依然未公開

### 更新ファイル
- specs/claude/changelog.md
- specs/claude/configuration.md

---

## 2026-04-22 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog, llms.txt（settings/hooks/skills/mcp/agent-teams/best-practices はキャッシュ比較で変更なし）
- Codex CLI: changelog
- スキルエコシステム: 前回巡回(2026-04-18)から4日経過のためスキップ（7日ルール）

### 検出された変更と更新内容

#### Claude Code（v2.1.116、4月20日）
- **specs/claude/changelog.md** — v2.1.116 を追記
  - `/resume` 大規模セッション高速化（最大67%）、MCP 起動高速化、フルスクリーンスクロール改善
  - Thinking スピナーのインライン進捗表示
  - Agent フロントマター `hooks:` がメインスレッド `--agent` 実行で発火
  - サンドボックス危険パスチェックの修正（`rm`/`rmdir` `/`, `$HOME` 対象時）
  - ターミナル互換性の多数の修正（Kitty プロトコル、インラインモードスクロールバック等）
- v2.1.115 は公開changelogに記載なし（スキップ）

#### Codex CLI（0.122.0、4月20日）
- **specs/codex/changelog.md** — 0.122.0 を追記
- **specs/codex/commands.md** — `/side` コマンド（サイド会話）を1.4 セッション管理に追加
- **specs/codex/configuration.md** — 5.3 権限プロファイルに deny-read グロブ、隔離 exec の注記追加

#### その他
- `whats-new/2026-w16` は未公開
- agent-teams/best-practices/hooks/mcp/settings/skills いずれも差分なし

### 更新ファイル
- specs/claude/changelog.md
- specs/codex/changelog.md
- specs/codex/commands.md
- specs/codex/configuration.md
- .patrol-cache/url-metadata.json

---

## 2026-04-19 — 公式ドキュメント巡回（差分のみ）

### 巡回対象URL
- Claude Code: changelog, settings, hooks, llms.txt
- Codex CLI: changelog
- スキルエコシステム: 前回巡回(2026-04-18)から1日経過のためスキップ（7日ルール）

### 検出された変更と更新内容

#### Claude Code（v2.1.114、4月18日）
- **specs/claude/changelog.md** — v2.1.114 を追記
  - エージェントチームのチームメイトがツール権限リクエスト時の権限ダイアログクラッシュを修正（バグ修正のみ）

#### Codex CLI
- 変更なし（0.121.0 が引き続き最新）

#### その他
- `whats-new/2026-w16` は未公開
- settings/hooks/skills/mcp/agent-teams/best-practices いずれも変更なし

### 更新ファイル
- specs/claude/changelog.md

---

## 2026-04-18 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（96ページ、16増）、changelog、settings、commands、hooks
- Codex CLI: changelog
- スキルエコシステム: anthropics/skills, openai/skills, agentskills.io, skills.sh

### 検出された変更と更新内容

#### Claude Code（v2.1.98〜v2.1.113 の10バージョン、4月9〜17日）
- **specs/claude/changelog.md** — 新バージョン10件を追記
  - v2.1.113: ネイティブバイナリ化、`sandbox.network.deniedDomains`、フルスクリーン操作拡張、`/ultrareview` 改善
  - v2.1.111: **Claude Opus 4.7 xhigh**、`/effort` スライダー、`/ultrareview` 追加、`/less-permission-prompts` スキル
  - v2.1.110: `/tui`、`/focus`、push notification ツール
  - v2.1.108: `ENABLE_PROMPT_CACHING_1H`、recap 機能、`/undo` エイリアス、Skill ツール経由の組み込みコマンド呼び出し
  - v2.1.105: **PreCompact フック対応**、`EnterWorktree.path` パラメータ、`/proactive` エイリアス
  - v2.1.101: `/team-onboarding`、OS CA 証明書信頼、`/ultraplan` クラウド環境自動作成
  - v2.1.98: Vertex AI セットアップウィザード、`CLAUDE_CODE_PERFORCE_MODE`、**Monitor ツール**
- **specs/claude/configuration.md** — `sandbox.network.deniedDomains`、`CLAUDE_CODE_PERFORCE_MODE`、`ENABLE_PROMPT_CACHING_1H` 追加
- **specs/claude/skills-and-commands.md** — `/ultrareview`、`/tui`、`/focus`、`/team-onboarding`、`/less-permission-prompts`、`/undo`、`/proactive`、`/extra-usage` 追加

#### 新規ドキュメントページ検出
- `ultrareview.md`（新規コマンド）、`whats-new/2026-w13.md`、`whats-new/2026-w15.md`（週次ダイジェスト）

#### Codex CLI（3バージョン、4月10〜15日）
- **specs/codex/changelog.md** — 新バージョン3件を追記
  - CLI 0.121.0: マーケットプレースインストール（GitHub/git URL/ローカル）、`Ctrl+R` 逆検索、MCP Apps 並列コール、bubblewrap devcontainer
  - CLI 0.120.0: Realtime V2 進捗ストリーミング、MCP `outputSchema`、`/clear` 用 SessionStart フック区別
  - CLI 0.119.0: Realtime 音声 v2 WebRTC デフォルト化、Remote workflow、`/resume` ID ジャンプ

### スキルエコシステム巡回
- anthropics/skills: **120K+ stars**（115K→120K）
- agentskills.io: 対応プラットフォーム **37+** に増加（33→37）。新規: TRAE, Spring AI, VT Code, Qodo, Emdash, Snowflake Cortex Code, nanobot
- skills.sh: find-skills が **1.1M installs** でトップ（788K→1.1M）、frontend-design 310K、vercel-react-best-practices 328K
- 推薦スキル: Tier A/B 据え置き。find-skills と frontend-design のインストール数更新

### 更新ファイル
- `specs/claude/changelog.md`
- `specs/claude/configuration.md`
- `specs/claude/skills-and-commands.md`
- `specs/codex/changelog.md`
- `kb/skills/_index.md`
- `kb/skills/sources.md`
- `kb/skills/recommended.md`
- `kb/update-history.md`

---

## 2026-04-06 — 公式ドキュメント巡回（04-05巡回を統合）

### 巡回対象URL
- Claude Code: llms.txt（80ページ）、changelog, settings, commands, env-vars, whats-new/2026-w14, ultraplan, fast-mode, checkpointing, remote-control
- Codex CLI: changelog
- スキルエコシステム: anthropics/skills, openai/skills, skills.sh, agentskills.io

### 検出された変更と更新内容

#### Claude Code
- **specs/claude/changelog.md** — v2.1.92 追加（`forceRemoteSettingsRefresh`、Bedrockウィザード、`/cost` モデル別内訳、`/release-notes` ピッカー化、Write差分60%高速化、`/tag`・`/vim` 削除、Linux seccomp）、v2.1.90 追加（`/powerup` コマンド）
- **specs/claude/configuration.md** — `forceRemoteSettingsRefresh` 設定追加、`CLAUDE_CODE_DISABLE_FAST_MODE`・`CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` 環境変数追加
- **specs/claude/skills-and-commands.md** — `/ultraplan` コマンド追加、`/release-notes` 説明更新、`/pr-comments` 廃止反映

#### 新規ドキュメントページ検出（specs未反映・参考情報）
- `ultraplan.md` — クラウドプランニング機能（リサーチプレビュー）
- `fast-mode.md` — Fastモード詳細（$30/150 MTok、2.5x高速）
- `checkpointing.md` — チェックポイント/巻き戻し機能詳細
- `remote-control.md` — Remote Control詳細（server mode、spawn、capacity）
- `computer-use.md` — CLI内Computer Use（リサーチプレビュー）

#### Codex CLI
- 変更なし（CLI 0.118.0 が最新のまま）

### スキルエコシステム巡回
- anthropics/skills: 110K+ stars。新規コミットなし
- openai/skills: 16K+ stars、45スキル。新規コミットなし
- skills.sh: 91.5K件（+1.5K）。find-skills 774K installs
- agentskills.io: 対応プラットフォーム33に増加。新規: Kiro, Mistral Vibe, Snowflake Cortex Code, Databricks Genie Code等
- 推薦スキル変更なし（Tier A/B据え置き）

---

## 2026-04-04 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（75ページ）、changelog, hooks, settings, env-vars, commands, mcp
- Codex CLI: changelog
- スキルエコシステム: anthropics/skills, skills.sh

### 検出された変更と更新内容

#### Claude Code
- **specs/claude/changelog.md** — v2.1.91 追加（MCP `_meta["anthropic/maxResultSizeChars"]` 500K上限オーバーライド、`disableSkillShellExecution` 設定、プラグイン `bin/` 実行ファイル同梱、ディープリンク複数行対応、Edit ツール短アンカー化、各種修正）
- **specs/claude/configuration.md** — `disableSkillShellExecution` 設定追加、`CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` 環境変数追加
- **specs/claude/hooks.md** — `PermissionDenied` の matcher 修正（matcherなし→ツール名）
- **specs/claude/mcp.md** — §8.8 ツール結果サイズのサーバー側オーバーライド機能を追加、セクション番号再整理

#### Codex CLI
- 変更なし（CLI 0.118.0 が最新のまま）

### スキルエコシステム巡回
- anthropics/skills: 105K→110K stars。構造的変更なし
- skills.sh: find-skills 750K→788K installs、frontend-design 211K→222K installs
- 推薦スキル変更なし（Tier A/B据え置き）
- kb/skills/_index.md, recommended.md のインストール数・stars数更新

---

## 2026-04-03 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（75ページ）、changelog, hooks, settings, env-vars, commands
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code
- **specs/claude/changelog.md** — v2.1.90 追加（/powerup コマンド、PLUGIN_KEEP_MARKETPLACE_ON_FAILURE env var、.husky保護、Auto Mode境界尊重、PostToolUse format-on-save修正、PowerShell脆弱性修正、パフォーマンス改善）
- **specs/claude/skills-and-commands.md** — `/powerup`, `/fast`, `/release-notes`, `/stats`, `/insights` コマンド追加
- **specs/claude/configuration.md** — `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` 環境変数追加
- **specs/claude/hooks.md** — `PermissionDenied` 入力フィールド修正（`tool_use_id`, `reason`）

#### Codex CLI
- 変更なし（CLI 0.118.0 が最新のまま）

### スキルエコシステム巡回
- スキップ（last_patrol: 2026-03-28、7日以内→4月5日以降に実施予定）

---

## 2026-04-02 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（81ページ、前回75→6ページ増）、changelog, hooks, settings, mcp, scheduled-tasks, web-scheduled-tasks, discover-plugins
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code
- **specs/claude/hooks.md** — `CwdChanged`, `FileChanged`, `TaskCreated`, `WorktreeCreate` の入力フィールド修正（公式ドキュメントとの不整合解消）
- **specs/claude/skills-and-commands.md** — スケジュールタスク（セクション3: Cloud/Desktop/`/loop`比較、CronCreate/CronList/CronDelete）、プラグインマーケットプレース（セクション4: 公式マーケット、コードインテリジェンス11言語LSP、外部インテグレーション）追加。`/schedule`, `/reload-plugins` コマンド追加
- **specs/claude/mcp.md** — `MCP_CONNECTION_NONBLOCKING` 環境変数をMCPセクションに追記

#### Codex CLI
- 新バージョンなし（0.118.0 が最新、前回キャプチャ済み）

### スキルエコシステム巡回
- スキップ（last_patrol: 2026-03-28、5日前→7日以内のためスキップ。4月4日以降に実施予定）

### llms.txt ページ数変化（75→81）
- 6ページ増。新規ドキュメント: scheduled-tasks, web-scheduled-tasks, discover-plugins, plugin-marketplaces, plugins-reference, channels-reference

---

## 2026-04-01 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（75ページ、前回73→2ページ増）、changelog, hooks, settings, env-vars, skills
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code
- **specs/claude/changelog.md** — v2.1.89 追加（大規模アップデート: defer permission, PermissionDenied hook, NO_FLICKER env, autocompact thrash検出, thinking summaries デフォルト無効化 他多数）
- **specs/claude/hooks.md** — `PermissionDenied` イベント追加、`PreToolUse` に `defer` permission decision 追加
- **specs/claude/configuration.md** — `showThinkingSummaries`, `cleanupPeriodDays` 動作変更注記追加。`CLAUDE_CODE_NO_FLICKER`, `MCP_CONNECTION_NONBLOCKING` 環境変数追加

#### Codex CLI
- **specs/codex/changelog.md** — CLI 0.118.0 追加（Windows sandbox proxy networking, device code flow, codex exec stdin, dynamic bearer tokens）

### スキルエコシステム巡回
- スキップ（last_patrol: 2026-03-28、7日以内→4月4日以降に実施予定）

### llms.txt ページ数変化（73→75）
- 2ページ増。新規ドキュメントの追加を示唆

---

## 2026-03-31 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（73ページ、前回76→3ページ減）、settings, hooks, skills, mcp, changelog, env-vars
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code
- **specs/claude/changelog.md** — v2.1.87 追加（Cowork Dispatch メッセージ未配信修正のみ）
- **specs/claude/skills-and-commands.md** — `effort` フロントマターの `max` オプションに「Opus 4.6のみ」注記追加

#### Codex CLI
- 変更なし（CLI 0.117.0、App 26.323 が最新のまま）

### スキルエコシステム巡回
- スキップ（last_patrol: 2026-03-28、7日以内）

### llms.txt ページ数変化（76→73）
- 3ページ減。ページ統合の可能性（前回巡回でも79→76の減少傾向あり）

---

## 2026-03-30 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（76ページ、前回79→3ページ減、再編成の可能性）、settings, hooks, skills, mcp, changelog
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code
- 新バージョンなし（v2.1.86が最新のまま）

1. **specs/claude/configuration.md** — 設定キー5件補完（前回以前の巡回で取りこぼし）
   - `autoMode`: Auto Mode分類器設定（`environment`, `allow`, `soft_deny`）
   - `disableAutoMode`: Auto Mode有効化の阻止（`"disable"`）
   - `useAutoModeDuringPlan`: プランモードでのAuto Modeセマンティクス（デフォルト`true`）
   - `defaultShell`: `!`コマンドのデフォルトシェル（`bash` / `powershell`）
   - `otelHeadersHelper`: 動的OpenTelemetryヘッダー生成スクリプト
   - `attribution` の重複エントリ修正

#### Codex CLI
- 変更なし（CLI 0.117.0、App 26.323 が最新のまま）

### スキルエコシステム巡回
- スキップ（last_patrol: 2026-03-28、7日以内）

### llms.txt ページ数変化（79→76）
- 3ページ減。ページ統合・リネームの可能性（詳細は次回以降に追跡）

---

## 2026-03-29 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（79ページ、前回75→4ページ増）、changelog
- Codex CLI: changelog（変更なし）

### 検出された変更と更新内容

#### Claude Code (v2.1.86)

1. **specs/claude/changelog.md** — v2.1.86 追加
   - `X-Claude-Code-Session-Id` APIヘッダー
   - `.jj`/`.sl` VCS除外
   - `@` メンションのトークンオーバーヘッド削減
   - スキル説明文250文字上限
   - Read ツールのコンパクト行番号形式

2. **specs/claude/skills-and-commands.md** — 1件更新
   - `description` フロントマターに250文字上限の注記追加

#### Codex CLI
- 変更なし（CLI 0.117.0、App 26.323 が最新のまま）

### スキルエコシステム巡回
- スキップ（last_patrol: 2026-03-28、7日以内）

### llms.txt ページ数変化（75→79）
- 新規ページ: extended-thinking 等（4ページ追加確認）

---

## 2026-03-28 — スキルエコシステム巡回（初回動作確認）

### 巡回結果
- skills.shトレンド上位10件を確認（find-skills 750K, frontend-design 211K等）
- anthropics/skills: 104,564 stars（安定）
- openai/skills: 15,530 stars（安定）
- agentskills.io: 32プラットフォーム確認（Claude Code, Codex, Gemini CLI, Cursor, VS Code, GitHub Copilot, Kiro, Roo Code, JetBrains Junie, Databricks, Snowflake等）

### 更新ファイル
- kb/skills/_index.md: プラットフォーム数を修正（33→32）

### 知見
- skills.shトレンドはVercel系（find-skills, react-best-practices）とMicrosoft Azure系が上位を占める
- anthropics/skillsの公式スキルは安定。新規追加なし
- Tier A推薦スキル（obra/superpowers系）は引き続き有効

---

## 2026-03-28 — 公式ドキュメント巡回

### 巡回対象URL
- Claude Code: llms.txt（75ページ一覧取得、前回67→8ページ増）、settings, hooks, skills, mcp, plugins, channels, changelog
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code (v2.1.85)

1. **specs/claude/changelog.md** — v2.1.85 追加
   - Hooks `if` 条件フィールド（permission rule構文）
   - MCP OAuth RFC 9728 対応
   - `CLAUDE_CODE_MCP_SERVER_NAME`, `CLAUDE_CODE_MCP_SERVER_URL` 環境変数
   - deep link 5,000文字対応
   - スケジュールタスクのタイムスタンプマーカー

2. **specs/claude/hooks.md** — 2件更新
   - `if` 条件フィールドの仕様追加（v2.1.85）: ツールイベント専用、permission rule構文
   - `TaskCreated` がブロック可能（Yes）に修正

3. **specs/claude/configuration.md** — 環境変数2件追加
   - `CLAUDE_CODE_MCP_SERVER_NAME`, `CLAUDE_CODE_MCP_SERVER_URL`

4. **specs/claude/skills-and-commands.md** — フロントマター1件追加
   - `paths` フィールド: glob パターンでスキル自動適用を限定

5. **specs/claude/mcp.md** — OAuth 1件追加
   - RFC 9728 Protected Resource Metadata ディスカバリ対応

#### Codex CLI

6. **specs/codex/changelog.md** — 2エントリ追加
   - CLI 0.117.0（2026-03-26）: プラグインのファーストクラスワークフロー化、サブエージェントパスベースアドレス等
   - Plugin Support Released（2026-03-25）: プラグインバンドル導入

### llms.txt ページ数変化（67→75）
新規ドキュメントページ検出: channels, channels-reference, checkpointing, chrome, claude-code-on-the-web, discover-plugins, fast-mode, features-overview, keybindings, microsoft-foundry, output-styles, plugin-marketplaces, plugins, plugins-reference, remote-control, scheduled-tasks, server-managed-settings, statusline, voice-dictation, web-scheduled-tasks 等（一部は既存ページのリネーム/リオーガナイズの可能性あり）

### 変更なし（差分なし確認済み）
- specs/claude/agent-teams.md — 公式ドキュメントとの大きな差分なし
- specs/claude/best-practices.md — 公式ドキュメントとの大きな差分なし

---

## 2026-03-27 — AiToEarn 再調査（業務ドメイン深掘り込み）

### 書き直しファイル
- `kb/external/aitoearn/analysis.md` — AiToEarn徹底調査レポート（再調査版）。frontmatter追加、libs/16モジュール詳細分析、CLAUDE.md未成熟事例評価、Docker Compose構成分析、業界トレンド対比
- `kb/external/aitoearn/takeaways.md` — 採用判断（再調査版）。frontmatter追加、法規制リスク（日本/EU/米国）追加、前回調査との差分表追加

### 新規ファイル
- `kb/domains/content-monetization/overview.md` — コンテンツ収益化業務ドメイン深掘り（新規ドメイン）。市場動向（$43.5億→$128.5億）、CPS/CPE/CPM収益モデル、8ステップパイプライン、プラットフォーム別戦略（YouTube/TikTok/Instagram/X/LinkedIn/Threads/ブログ/Substack）、法規制（日本ステマ規制/EU AI Act/米国各州法）、AIエージェント活用指針、ハーネス設計への示唆

### 更新ファイル
- `kb/external/_index.md` — AiToEarnの最終確認日を2026-03-27に更新、説明を拡充
- `kb/domains/_index.md` — content-monetizationドメインを追加

### 主な知見（前回からの追加分）
- AiToEarnのCLAUDE.mdはNx公式テンプレートのまま（未成熟事例）→ 二層構造（公式テンプレート+カスタマイズ層）の推奨に変更
- libs/nest-mcpモジュールが「既存Web APIのMCP化」の具体実装として最も有用なパターン
- コンテンツ収益化市場は2025年$43.5億→2029年$128.5億（CAGR 31.4%）の急成長
- 2026年のトレンド: ハイブリッド（AI+人間）がAI onlyより信頼度+33%、エンゲージメント+23%上回る
- 法規制が急速に整備中: EU AI Act Article 50（2026年8月）、カリフォルニアAI透明性法（2026年1月）、日本ステマ規制（2023年10月施行済み）
- Hub-and-Spokeモデル（1本のハブコンテンツ→10+バリエーション）でリーチ+35%
- 総合評価をC+からB-に上方修正（業務ドメイン知見の価値を考慮）

---

## 2026-03-26 — claude-peers-mcp 調査

### 追加ファイル
- `kb/external/claude-peers/analysis.md` — claude-peers-mcp徹底調査レポート（基本情報、アーキテクチャ、MCPツール4種、セキュリティ評価、公式Agent Teams比較、類似ツール5件比較）
- `kb/external/claude-peers/takeaways.md` — 採用判断（Brokerデーモンパターン、スコープ付きピア発見、自動コンテキストサマリーの3パターン抽出）

### 更新ファイル
- `kb/external/_index.md` — claude-peers-mcpをレジストリに追加（ステータス: reference）
- `kb/update-history.md` — 本エントリを追加

### 主な知見
- Louis Arge作のClaude Code P2Pメッセージング MCPサーバー。GitHub 1,249スター、TypeScript/Bun、ライセンス未明示
- Broker (localhost:7899) + SQLite + MCPチャネルプロトコルによるリアルタイム配信アーキテクチャ
- 公式Agent Teams（実験的）と補完関係にあるが、機能重複が大きい。ライセンス未明示・成熟度不足（Issue 16件中0件クローズ、メッセージロスト問題複数）により統合は不採用
- 設計パターン3件を抽出: (1) Brokerデーモン自動起動パターン、(2) スコープ付きピア発見（machine/directory/repo）、(3) 自動コンテキストサマリー
- マルチエージェント協調の3層分類を整理: ビルトイン層（Agent Teams）/ MCP拡張層（claude-peers等）/ 外部管理層（claude-squad等）。harness-harnessはビルトイン層基本+パターン抽出方針

---

## 2026-03-26 — AiToEarn 調査

### 追加ファイル
- `kb/external/aitoearn/analysis.md` — AiToEarn徹底調査レポート（基本情報、技術アーキテクチャ、MCP対応、収益化モデル、類似ツール比較、harness-harness適用評価）
- `kb/external/aitoearn/takeaways.md` — 採用判断（MCP HTTP公開パターン、Nxモノレポ向けCLAUDE.md等）

### 更新ファイル
- `kb/external/_index.md` — AiToEarnをレジストリに追加（ステータス: reference）

### 主な知見
- AiToEarnはAI活用SNSコンテンツマーケティングの全自動化プラットフォーム（Monetize/Publish/Engage/Createの4Agent構成）
- GitHub 12,431スター、MIT License、TypeScript 92.6%、NestJS+Nx+Electron構成
- MCP HTTP公開パターン（`"type": "http"`でWeb APIをMCPサーバー化）がharness-harnessテンプレートとして有用
- NxモノレポのCLAUDE.md配置パターンが参考になる
- ツール自体の統合・定期監視は不要（ドメイン特化度が高いため）

---

## 2026-03-26 — DeerFlow 2.0 調査（research-kbスキル実行）

### 追加ファイル
- `kb/external/deerflow/analysis.md` — ByteDance製SuperAgentハーネスの深掘り分析
- `kb/external/deerflow/takeaways.md` — 採用判断（Harness/App分離、Progressive Skill Loading等）

### 主な知見
- Harness/App分離（`Harness must never import App`）がharness-harnessの設計原則と完全合致
- Progressive Skill Loading = 段階的開示の具体実装
- skills/public + skills/custom の二層構造 = templates/ + プロジェクト固有の対応
- メモリの信頼度スコア+タイムスタンプはkb/知見管理に応用可能
- LangGraph依存やServer-first構成は不採用（パターンのみ抽出）

---

## 2026-03-26 — マルチハーネスベストプラクティス調査

### 追加ファイル
- `kb/research/2026-03-26-multi-harness-best-practices.md` — 1プロジェクト複数AIハーネスのベストプラクティス調査

### 調査トピック（5分野）
1. Claude Code の目的別ハーネス設計（CLAUDE.md スコープ、rules paths、agents、skills）
2. Codex CLI のプロファイル活用（config.toml profiles、AGENTS.md 階層マージ）
3. モノレポでの AI 設定パターン（Cursor .mdc、Copilot instructions、Windsurf rules）
4. マルチエージェント協調パターン（サブエージェント、Agent Teams、ペルソナ設計）
5. 非技術ドメインのハーネス設計（マーケティング、分析、自律学習）

### 主な知見
- Claude Code は rules(paths指定) + agents/ + skills(paths指定) の3層で目的別ハーネスを最も細かく制御可能
- Codex CLI は profiles で設定セット切替、AGENTS.md 階層マージでディレクトリ別指示を実現
- AGENTS.md がクロスツール標準化（Codex, Copilot, Cursor, Windsurf, Amp, Devin が読み取り）
- Agent Teams（実験的）で複数エージェントが共有タスクリスト+メールボックスで協調可能
- 非技術ドメインもエージェント定義 + 永続メモリで対応可能（data-scientist 公式例あり）

---

## 2026-03-26 — autoresearch調査（research-kbスキル初回実行）

### 追加ファイル
- `kb/external/autoresearch/analysis.md` — Karpathy autoresearchの深掘り分析
- `kb/external/autoresearch/takeaways.md` — 採用判断（Claude/Codexクロスレビュー統合版）

### 調査方法
- Phase 1: Claude徹底調査（GitHub, Web, ソースコード）
- Phase 2: Codex独自調査（codex exec, 325kトークン使用）
- Phase 3: クロスレビュー統合

### 主な知見
- 「bounded autoresearch pattern」: 固定評価器+単一可変面+予算+台帳の6要素をテンプレート化候補
- ML訓練基盤そのものは不採用（GPU依存）。設計パターンのみ取り込む
- 派生プロジェクト`autoimprove-cc`がCLAUDE.md自動改善を既に実装

## 2026-03-26 — 公式ドキュメント巡回（自律巡回）

### 巡回対象URL
- Claude Code: llms.txt（67ページ一覧取得）、settings, hooks, skills, mcp, agent-teams, best-practices, changelog
- Codex CLI: changelog

### 検出された変更と更新内容

#### Claude Code (v2.1.83〜v2.1.84)

1. **specs/claude/hooks.md** — 新hookイベント3件、フィールド追加
   - `CwdChanged` イベント（v2.1.83）: ワーキングディレクトリ変更時に発火
   - `FileChanged` イベント（v2.1.83）: 監視ファイルのディスク変更時に発火（matcher: ファイル名）
   - `TaskCreated` イベント（v2.1.84）: TaskCreate でタスク作成時に発火
   - Command ハンドラに `shell` フィールド追加（`"bash"` / `"powershell"`）
   - WorktreeCreate の HTTP フック対応（`hookSpecificOutput.worktreePath`）
   - SessionEnd matcher に `bypass_permissions_disabled` 追加
   - StopFailure matcher に `invalid_request` 追加
   - InstructionsLoaded 入力に `parent_file_path` 追加
   - 共通入力に `worktree` フィールド追加

2. **specs/claude/configuration.md** — 新設定キー・環境変数
   - `managed-settings.d/` ドロップインディレクトリ（v2.1.83）
   - `sandbox.failIfUnavailable`, `disableDeepLinkRegistration` 設定（v2.1.83）
   - `allowedChannelPlugins` managed設定（v2.1.84）
   - 環境変数4件: `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`, `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK`, `CLAUDE_STREAM_IDLE_TIMEOUT_MS`, `ANTHROPIC_DEFAULT_{OPUS,SONNET,HAIKU}_MODEL_SUPPORTS`

3. **specs/claude/skills-and-commands.md** — フロントマター・コマンド追加
   - スキル `shell` フロントマター（`powershell` 対応）
   - エージェント `initialPrompt` フロントマター（v2.1.83）
   - `/color`, `/copy [N]` コマンド追加

4. **specs/claude/mcp.md** — 上限・重複排除
   - MCPツール説明・サーバー指示の2KB上限（v2.1.84）
   - ローカル/claude.aiコネクタ間のMCPサーバー重複排除（v2.1.84）

5. **specs/claude/changelog.md** — 不足バージョン追記
   - v2.1.84, v2.1.83 の詳細追記
   - v2.1.79, v2.1.74, v2.1.73 を新規追加

#### Codex CLI

6. **specs/codex/changelog.md** — 不足エントリ追記
   - アプリ 26.318, 26.317拡充, 26.316, 26.311 を追加

### 変更なし（差分なし確認済み）
- specs/claude/agent-teams.md — 公式ドキュメントとの大きな差分なし
- specs/claude/best-practices.md — 公式ドキュメントとの大きな差分なし
- specs/codex/configuration.md — 新規公式変更なし

---

## 2026-03-23 — 公式ドキュメント巡回（自律巡回）

### 巡回対象URL
- Claude Code: settings, skills, hooks, mcp, agent-teams, best-practices, sub-agents, headless（全8ページ）
- Codex CLI: config-reference, slash-commands, skills, changelog（全4ページ）

### 検出された変更と更新内容

#### Claude Code

1. **specs/claude/configuration.md** — 新設定キー15件追加
   - `respectGitignore`, `includeGitInstructions`, `channelsEnabled`, `allowManagedPermissionRulesOnly`
   - `strictKnownMarketplaces`, `blockedMarketplaces`, `pluginTrustMessage`
   - `awsAuthRefresh`, `awsCredentialExport`, `voiceEnabled`
   - `spinnerTipsEnabled`, `spinnerTipsOverride`, `prefersReducedMotion`
   - `fastModePerSessionOptIn`, `teammateMode`, `feedbackSurveyRate`, `showClearContextOnPlanAccept`
   - `includeCoAuthoredBy` の非推奨化を記録

2. **specs/claude/mcp.md** — 新機能5件追加
   - MCP Tool Search（`ENABLE_TOOL_SEARCH`）
   - MCP Resources（`@` メンション参照）
   - MCP Elicitation（Form/URL モード）
   - `claude mcp serve`（Claude Code を MCP サーバーとして使用）
   - OAuth 詳細（固定コールバックポート、事前設定認証情報、メタデータディスカバリ）
   - Managed MCP の Policy-based Control（`serverName`/`serverCommand`/`serverUrl` マッチング）

3. **specs/claude/skills-and-commands.md** — 3件更新
   - ビルトインサブエージェント追加: `statusline-setup`, `Claude Code Guide`
   - ヘッドレスモード（非対話モード）セクション新規追加: `--bare`, `--output-format`, `--json-schema`, `--allowedTools`, `--continue`/`--resume`

#### Codex CLI

4. **specs/codex/configuration.md** — 大規模更新
   - 新設定キー9件追加: `model_context_window`, `forced_login_method`, `forced_chatgpt_workspace_id`, `cli_auth_credentials_store`, `default_permissions`, `file_opener`, `check_for_update_on_startup`, `allow_login_shell`
   - 新機能フラグ2件: `unified_exec`（stable）, `undo`
   - Agent管理セクション新規追加: `agents.max_threads`, `agents.max_depth`, `agents.<name>.description`
   - UI設定セクション新規追加: `tui.theme`, `tui.animations`, `tui.notifications`, `tui.status_line`
   - 承認粒度セクション新規追加: `approval_policy.granular.*`
   - **Skills パス変更**: `.codex/skills/` → `.agents/skills/`（CWD/親/ルート/ユーザー/管理者/システムの6レベル）
   - `agents/openai.yaml` メタデータ追加: UI設定、`allow_implicit_invocation`ポリシー、ツール依存定義
   - スキルインストーラー（`$skill-installer`）の追加
   - スキル有効化/無効化設定の追加

5. **specs/codex/commands.md** — `/send-feedback` コマンド追加

6. **Codex Changelog 主要項目**:
   - CLI 0.116.0（2026-03-19）: `userpromptsubmit` フック、Realtime セッション改善
   - GPT-5.4 mini（2026-03-17）: 新モデル追加
   - CLI 0.115.0（2026-03-16）: `view_image`, Smart Approvals, Python SDK
   - CLI 0.114.0（2026-03-11）: 実験的Hooks（SessionStart, Stop）、Code Mode

#### Mapping 更新

7. **mapping/claude-to-codex.md** — Skills パスを `.codex/skills/` → `.agents/skills/` に更新
8. **mapping/codex-to-claude.md** — 同上 + 移行ガイド更新
9. **mapping/shared-concepts.md** — Skills パス更新 + `agents/openai.yaml` メタデータ拡張を追記

### 変更なし（差分なし確認済み）
- specs/claude/hooks.md — 既存仕様書が最新の公式ドキュメントと一致
- specs/claude/agent-teams.md — 既存仕様書が最新の公式ドキュメントと一致
- specs/claude/best-practices.md — 軽微な追加（Chrome拡張、プラグイン言及）のみ。仕様影響なし

### 要確認事項
- Codex Skills の `.codex/skills/` → `.agents/skills/` パス変更が正式な移行なのか、並行サポートなのかは公式アナウンスで確認が必要
- Codex CLI 0.116.0 の `userpromptsubmit` フックが Hooks 仕様の正式イベント追加なのか要確認

## 2026-03-23 — 外部プロジェクト初期調査

### 追加ファイル
- `kb/external/_index.md` — 外部プロジェクト調査レジストリ
- `kb/external/openclaw/analysis.md` — OpenClaw深掘り分析
- `kb/external/openclaw/takeaways.md` — OpenClawからの知見と採用判断
- `kb/external/gstack/analysis.md` — gstack深掘り分析
- `kb/external/gstack/takeaways.md` — gstackからの知見と採用判断
- `kb/external/superpowers/analysis.md` — superpowers深掘り分析
- `kb/external/superpowers/takeaways.md` — superpowersからの知見と採用判断

### 削除ファイル
- `kb/external/openclaw/.gitkeep`
- `kb/external/gstack/.gitkeep`
- `kb/external/superpowers/.gitkeep`

### 調査概要

3つの主要な外部プロジェクトを調査:

1. **OpenClaw** (Peter Steinberger) — ローカルファーストの自律AIエージェント。20+チャネル統合、53+バンドルスキル、180,000+ GitHub Stars。パーソナルAIアシスタントとしての方向性はharness-harnessと異なるが、SKILL.mdフォーマット標準・3層スキル優先順位・ゲーティング機構を採用。

2. **gstack** (Garry Tan) — ロールベースの仮想開発チームワークフロー。28スキル、構造化スプリントプロセス。安全ガードレール（/careful, /freeze, /guard）・2層テスティング・グローバル/ローカルインストールモデルを採用。

3. **superpowers** (Jesse Vincent) — マルチプラットフォームスキルフレームワーク。Claude Code/Codex/Cursor/OpenCode対応。SKILL.mdフロントマター標準の策定者。マルチプラットフォームディレクトリ構造・Hookベース初期化・CSO概念を採用。

### 横断的な主要判断

| 判断 | 項目 |
|------|------|
| 採用（高優先度） | SKILL.mdフロントマター標準、マルチプラットフォームディレクトリ構造、安全ガードレール、構造化プロセステンプレート、Hookベース初期化、グローバル/ローカルインストール |
| 採用（中優先度） | 3層スキル優先順位、ゲーティング機構、CSO、2層テスティング、クロスモデル対応 |
| 検討 | シンボリックリンク戦略、ロールベース設計、ワークフロー選択肢、Git Worktree、自己修正型エージェント |
| 不採用 | マルチチャネル統合（範囲外）、常駐デーモンモデル（過剰） |

## 2026-03-23 — project-alpha初回dogfoodingフィードバック

### 追加ファイル
- `logs/evaluations/2026-03-23-project-alpha-diagnosis.md` — project-alphaハーネス診断レポート
- `logs/evaluations/2026-03-23-project-alpha-improvement-proposal.md` — 改善提案書
- `logs/evaluations/2026-03-23-project-alpha-feedback.md` — 改善実施後のフィードバック
- `docs/decisions/ADR-001-harness-improvement-process.md` — 作業プロセス標準化

### 主な学び
- セキュリティhookに `|| true` は禁物（テンプレートに反映要）
- hookのJSONパースはjq必須（テンプレートの前提条件に追加要）
- 診断は推測ではなくファイル内容を実際にパースして判定すべき
- gitignored設定の改善はPR対応不可（手動推奨として別セクション化要）
- claude-pr-reviewの指摘品質が高い（ワークフローテンプレート化候補）

## 2026-03-24 — Codex CLI Skills/Hooks サポート発見・仕様反映

### 発見内容

Codex CLI が以下の機能をサポートしていることを確認:

1. **Skills（stable）**: SKILL.md フロントマター形式で Claude Code と完全互換。`.codex/skills/`（プロジェクト）、`~/.codex/skills/`（グローバル）に配置。バンドルスキル 3 種（`skill-creator`, `skill-installer`, `openai-docs`）。`/skills` コマンド、`$` メンション、暗黙マッチングで起動。`skill_mcp_dependency_install` フラグは stable。

2. **Hooks（実験的）**: `codex_hooks` フラグで有効化（デフォルト無効）。対応イベントは `SessionStart`, `Stop`, `UserPromptSubmit` の 3 種のみ（Claude Code は 17+）。ハンドラは `command` のみ（Claude は command/HTTP/Prompt/Agent の 4 種）。`config.toml` の `[[hooks]]` テーブル配列で設定。終了コード 2 でブロック可能（`UserPromptSubmit`）。

### 更新ファイル

- `specs/codex/configuration.md` — Skills（セクション 6）と Hooks（セクション 7）を追加
- `mapping/claude-to-codex.md` — Skills セクションを「対応なし」→「直接対応」に更新。Hooks セクションを「対応なし」→「部分対応（実験的）」に更新
- `mapping/codex-to-claude.md` — Skills（セクション 5）と Hooks（セクション 6）を新規セクションとして追加。以降のセクション番号を繰り下げ
- `mapping/shared-concepts.md` — Skills（セクション 9）を共通概念として追加。Hooks（セクション 10）を「なし」→「実験的サポート」に更新。機能対応サマリー表を更新
- `docs/codex-plan.md` — 「Hooks 不在はラッパースクリプトで埋める」→「Hooks はネイティブ + ラッパースクリプトの併用で埋める」に更新。Claude 先行領域の記述を修正
- `kb/changelog.md` — 本エントリを追加

### 影響と方針変更

- **Skills は完全な共通概念に昇格**: `.claude/skills/` と `.codex/skills/` で SKILL.md を共有可能。harness-harness のスキルテンプレート戦略を共通化できる
- **Hooks は部分的共通概念**: 3 イベント（SessionStart, Stop, UserPromptSubmit）は両プラットフォームで利用可能。残りは引き続き Claude 固有
- **codex-plan.md の方針修正**: Hooks 代替のラッパースクリプトは引き続き必要だが、共通 3 イベントについてはネイティブ Hooks を優先利用する方針に変更
