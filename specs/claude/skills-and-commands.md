# Claude Code Skills & コマンド仕様書

最終更新: 2026-08-30（巡回更新）

公式ドキュメント: https://code.claude.com/docs/en/skills / https://code.claude.com/docs/en/commands / https://code.claude.com/docs/en/sub-agents / https://code.claude.com/docs/en/scheduled-tasks / https://code.claude.com/docs/en/web-scheduled-tasks / https://code.claude.com/docs/en/discover-plugins

---

## 1. Skills

Skills は Claude の能力を拡張する仕組み。`SKILL.md` ファイルに指示を記述し、Claude が自動的に関連性を判断して適用するか、ユーザーが `/skill-name` で直接呼び出す。

[Agent Skills](https://agentskills.io) オープンスタンダードに準拠。

### 1.1 配置場所

| スコープ | パス | 適用範囲 |
|:--|:--|:--|
| Enterprise | Managed settings 経由 | 組織内全ユーザー |
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` | 全プロジェクト |
| Project | `.claude/skills/<skill-name>/SKILL.md` | 当該プロジェクトのみ |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` | プラグイン有効時 |
| Plugin (単一スキル) | `<plugin>/SKILL.md` （`skills/` サブディレクトリなし） | プラグイン全体が単一スキルとしてサーフェスされる（v2.1.142） |

> **プラグインスキルの frontmatter `name`**: `name` はコマンド名の**最終セグメント**（ディレクトリ名）を置き換える。`my-plugin/skills/review/SKILL.md` に `name: fancy` を書くと `/my-plugin:fancy` になり、他が使っていなければ素の `/fancy` でも呼べる。`name` が既にプラグイン接頭辞を含む場合（`name: my-plugin:fancy`）、**v2.1.246 以降は接頭辞を重ねない**（v2.1.216〜v2.1.245 は二重化していた）。v2.1.216 より前は `name` がコマンド名全体を置き換えていた。

名前が重複する場合の優先度: Enterprise > Personal > Project。Plugin スキルは `plugin-name:skill-name` 名前空間で衝突しない。

`.claude/commands/` も引き続き動作する。同名の場合は Skills が優先。

#### サブディレクトリの自動検出

サブディレクトリのファイルを操作中、そのディレクトリの `.claude/skills/` も自動検出される（モノレポ対応）。

v2.1.178 から、ネストされた `.claude/skills/` のスキルは配下のファイル操作時にロードされ、名前衝突時はネスト側が `<dir>:<name>` として表示され両方が利用可能。agent / workflow / output-style もワーキングディレクトリに最も近い `.claude/` が勝つ。

#### `--add-dir` からのスキル

`--add-dir` で追加したディレクトリの `.claude/skills/` は自動読み込みされ、ライブ変更検出も有効。同ディレクトリの `.claude/commands/` と `.claude/agents/` も読み込まれる（ただし**監視はされない**ため、追加・編集後はセッション再起動が必要）。

この例外は `--add-dir` / `/add-dir` と、**Agent SDK が追加するディレクトリ**（TypeScript の `additionalDirectories` / Python の `add_dirs`。SDK が `--add-dir` として渡す）に適用される。settings.json の `permissions.additionalDirectories` はファイルアクセス権のみを与え、skills / commands / subagents を読み込まない（TypeScript のオプション名が同じでも意味が異なる）。

**読み込みの前提条件（ドキュメント追記）**:

- 追加ディレクトリからの skills / commands / subagents の読み込みは、`project` [設定ソース](https://code.claude.com/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources)が有効な場合のみ（既定は有効）。CLI で `--setting-sources` を渡す場合、SDK で `settingSources` / `setting_sources` を明示する場合は `project` を含めること
- `--safe-mode` では 3 種とも読み込まれない
- `strictPluginOnlyCustomization` と bare モードでは 3 種の扱いが異なる:
  - **skills**（`.claude/skills/`）: skills をロックするポリシーでオフ。bare モードでは読み込まれる
  - **commands**（`.claude/commands/`）: 同じ skills ロックでオフ。bare モードでは読み込まれない
  - **subagents**（`.claude/agents/`）: ポリシーの `agents` エントリでオフ（`skills` エントリではない）。bare モードではプロジェクト自身のものを含め全ての `.claude/agents/` をスキップ
- bare モードではスキルディレクトリの監視自体を行わない
- 追加ディレクトリの `.claude/settings.json` / `.claude/settings.local.json` からは `enabledPlugins` と `extraKnownMarketplaces` のみ読まれる。output style 等その他の `.claude/` 設定は読み込まれない

**`/cd` によるセッション移動（v2.1.246）**: 移動先ディレクトリのプロジェクトスキルが追加される。プロジェクト設定（`.claude/settings.json` / `settings.local.json`）も移動先のものを読むようになる。

**frontmatter が壊れた `SKILL.md` の検出**: `claude plugin validate .claude/skills`（個人スキルなら `~/.claude/skills`）でパースできない `SKILL.md` を洗い出せる（v2.1.233 以降）。

### 1.2 SKILL.md 構造

```
my-skill/
├── SKILL.md           # メイン指示（必須）
├── template.md        # テンプレート（任意）
├── examples/
│   └── sample.md      # 出力例（任意）
└── scripts/
    └── validate.sh    # 実行スクリプト（任意）
```

`SKILL.md` は YAML フロントマター + Markdown 本文で構成される。

### 1.3 フロントマターフィールド

| フィールド | 必須 | 説明 |
|:--|:--|:--|
| `name` | No | スキル表示名。省略時はディレクトリ名。小文字英数字とハイフンのみ（最大64文字） |
| `description` | 推奨 | スキルの用途と使用タイミング。Claude が自動適用の判断に使用。**250文字上限**（v2.1.86） |
| `argument-hint` | No | 引数ヒント。例: `[issue-number]` |
| `disable-model-invocation` | No | `true` で Claude の自動呼び出しを禁止。手動 `/name` のみ |
| `user-invocable` | No | `false` で `/` メニューから非表示。バックグラウンド知識用 |
| `allowed-tools` | No | スキル有効時に許可なしで使えるツール |
| `disallowed-tools` | No | スキル有効時にモデルから取り除くツール（v2.1.152）。スラッシュコマンドのフロントマターでも有効 |
| `model` | No | スキル有効時のモデル指定 |
| `effort` | No | エフォートレベル (`low` / `medium` / `high` / `max`（Opus 4.6のみ）) |
| `context` | No | `fork` でフォークサブエージェントコンテキストで実行。v2.1.218 から fork スキルは既定でバックグラウンド実行 |
| `agent` | No | `context: fork` 時のサブエージェントタイプ指定 |
| `background` | No | `context: fork` スキルのバックグラウンド実行を `false` でオプトアウト（v2.1.218） |
| `hooks` | No | スキルライフサイクルにスコープされたフック |
| `paths` | No | スキル自動適用を限定するglobパターン。カンマ区切り文字列またはYAMLリスト。パスマッチ時のみ自動読み込み |
| `shell` | No | インライン `` !`command` `` のシェル。`bash`（デフォルト）または `powershell`（Windows、`CLAUDE_CODE_USE_POWERSHELL_TOOL=1` 必要） |

### 1.4 呼び出し制御

| フロントマター | ユーザー呼び出し | Claude 呼び出し | コンテキスト読み込み |
|:--|:--|:--|:--|
| (デフォルト) | Yes | Yes | 説明は常にコンテキスト内、全文は呼び出し時 |
| `disable-model-invocation: true` | Yes | No | 説明はコンテキスト外、全文はユーザー呼び出し時 |
| `user-invocable: false` | No | Yes | 説明は常にコンテキスト内、全文は呼び出し時 |

- **スタック呼び出し（v2.1.199）**: `/skill-a /skill-b do XYZ` のように先頭に並べたスキルは最大5つまで全てロードされる
- **フロントマターキーの表記ゆれ許容（v2.1.186）**: `display-name` / `default-enabled` / `fallback` / `metadata.*` は kebab-case / snake_case / camelCase を受理。YAML フロントマターが壊れている場合も silent fail せず空メタデータで本文をロード

### 1.5 変数展開

| 変数 | 説明 |
|:--|:--|
| `$ARGUMENTS` | スキル呼び出し時に渡された全引数 |
| `$ARGUMENTS[N]` | N番目の引数（0始まり） |
| `$N` | `$ARGUMENTS[N]` の短縮形 |
| `${CLAUDE_SESSION_ID}` | 現在のセッションID |
| `${CLAUDE_SKILL_DIR}` | スキルの `SKILL.md` があるディレクトリ |
| `${CLAUDE_EFFORT}` | 現在の effort level（low/medium/high/xhigh/max）。スキル本文に埋め込み可能（v2.1.120） |

**プレースホルダが 1 つも引数を受け取らなかった場合**、末尾に `ARGUMENTS: <value>` が追加される（従来記載は「`$ARGUMENTS` が含まれない場合」）。プレースホルダとは `$ARGUMENTS`・`$1` 等のインデックス形式・名前付き引数を指す。インデックス形式でその位置に引数が無い場合はリテラルテキストのまま残り「受け取った」とは数えない。名前付きはその位置に引数が無くても空文字列に展開されるため「受け取った」と数える。

**引数の値自体に `$1` / `$ARGUMENTS` のような文字列が含まれていても展開されない**（リテラル挿入）。例: 本文に `Summarize $0` があるスキルを `/summarize "$ARGUMENTS from yesterday"` で呼ぶと Claude には `Summarize $ARGUMENTS from yesterday` が届く。`${CLAUDE_*}` 変数（`${CLAUDE_SKILL_DIR}` 等）は引数を挿入した**後**に置換される。

### 1.6 動的コンテキスト注入

`` !`<command>` `` 構文でシェルコマンドをプリプロセッシングとして実行し、出力をスキルコンテンツに埋め込む:

```yaml
---
name: pr-summary
context: fork
agent: Explore
---

## PRコンテキスト
- PR diff: !`gh pr diff`
- 変更ファイル: !`gh pr diff --name-only`
```

### 1.7 サブエージェントでの実行

`context: fork` で隔離されたサブエージェントコンテキストで実行。会話履歴にアクセスしない:

```yaml
---
name: deep-research
context: fork
agent: Explore
---

$ARGUMENTS を徹底的に調査してください。
```

`agent` フィールドで実行環境を指定: `Explore` / `Plan` / `general-purpose` / カスタムサブエージェント名。

v2.1.218 から `context: fork` のスキルは既定でバックグラウンド実行される。フォアグラウンドで実行したい場合はフロントマターに `background: false` を指定する。

> フロントマターの真偽値は v2.1.218 から `true`/`false` に加え `yes`/`no`/`on`/`off`/`1`/`0`（大文字小文字不問）も受理される（スキル・プラグイン共通）。

### 1.8 ツールアクセス制限

```yaml
---
name: safe-reader
allowed-tools: Read, Grep, Glob
---
```

### 1.9 スキルのアクセス制御

- **全スキル無効化**: `/permissions` で `Skill` を deny
- **個別許可/拒否**: `Skill(commit)` / `Skill(deploy *)`
- **個別非表示**: `disable-model-invocation: true`

### 1.10 バンドルスキル

Claude Code に同梱されるスキル:

| スキル | 用途 |
|:--|:--|
| `/batch <instruction>` | コードベース全体の大規模変更を並列オーケストレーション。ワークツリーごとにエージェントを起動しPRを作成 |
| `/claude-api` | Claude API リファレンス素材の読み込み（Python/TS/Java等） |
| `/debug [description]` | セッションデバッグログの解析 |
| `/loop [interval] <prompt>` (`/proactive`) | プロンプトを定期的に繰り返し実行（v2.1.105 で `/proactive` エイリアス追加） |
| `/code-review [effort] [--fix]` | 変更ファイルのコード品質レビューと修正（3エージェント並列）。`/code-review high` のように effort level を指定可能。v2.1.147 で `/simplify` からリネーム。v2.1.152 で `--fix` フラグ追加（レビュー結果をワーキングツリーに直接適用）。v2.1.215 で Claude による自動起動が廃止され、ユーザーの明示的な呼び出しのみに |
| `/verify` | コード変更が実際に意図通り動くかをエンドツーエンドで検証（テストや型チェックだけでなく対象フローを実際に動かす）。v2.1.215 で Claude による自動起動が廃止され、ユーザーの明示的な呼び出しのみに |
| `/simplify` | v2.1.152 で `/code-review --fix` のエイリアスとして復活。v2.1.154 でクリーンアップ専用レビュー（reuse / simplification / efficiency / altitude）に変更され、`/code-review --fix` のバグハンティングは行わなくなった |
| `/workflows` | Dynamic workflows の実行状況表示（v2.1.154）。Claude にワークフロー作成を依頼するとバックグラウンドで数十〜数百のエージェントを跨いだ作業をオーケストレーション。v2.1.160 でトリガーキーワードが `workflow` → `ultracode` にリネーム |
| `/less-permission-prompts` | 読み取り専用 bash/MCP 呼び出しを検出し許可リスト追加を提案（v2.1.111） |
| `/team-onboarding` | 新メンバー向けのプロジェクトオンボーディング資料生成（v2.1.101） |
| `/ultrareview` | クラウドベースの包括的コードレビュー。並列チェック・diffstat 表示（v2.1.111、v2.1.113 で改善）。CLI でも `claude ultrareview [target]` 非インタラクティブサブコマンドで CI/スクリプトから実行可能（`--json` 対応、終了コード 0/1、v2.1.120） |

---

## 2. 組み込みコマンド（Slash Commands）

`/` を入力して一覧表示。主要なコマンド:

> **タイポ時の挙動（v2.1.236）**: 存在しないコマンド、またはそのセッションで利用できないコマンドで Enter を押しても、最も近いコマンドへのファジーマッチで勝手に実行されることはなくなり、その旨が報告される。プレフィックス入力とエイリアスは従来どおり実行される。

### 2.1 セッション管理

| コマンド | 説明 |
|:--|:--|
| `/clear` (`/reset`, `/new`) | 会話履歴クリア |
| `/compact [instructions]` | 会話コンパクション |
| `/resume [session]` (`/continue`) | セッション再開 |
| `/rename [name]` | セッション名変更 |
| `/rewind` (`/checkpoint`, `/undo`) | 会話/コードを前の状態に巻き戻し（v2.1.108 で `/undo` エイリアス追加。v2.1.191 で `/clear` 実行前からの会話再開に対応） |
| `/cd <path>` | 会話を保ったままセッションのワーキングディレクトリを移動（v2.1.169）。v2.1.206 でパス補完対応。**v2.1.246 以降、移動と同時に移動先ディレクトリの project / local 設定、プロジェクトスキル、そのディレクトリの設定が有効化するプラグインの MCP サーバーを読み込む**（`/reload-plugins` 不要）。ファイルアクセス権だけを足したい場合は `/add-dir` を使う。`Cd` 権限ルールで移動先を制限・禁止できる |
| `/branch [name]` (`/fork`) | 会話のブランチ作成。v2.1.212 で `/fork` は会話を新しいバックグラウンドセッション（`claude agents` に独立表示）へコピーする挙動に変更。v2.1.221 から fork セッションは元のチェックアウトを共有せず自前の worktree を作成 |
| `/subtask` | 会話をコピーしたセッション内サブエージェントを起動（v2.1.212 で旧 `/fork` の挙動から改名） |
| `/export [filename]` | 会話をテキストエクスポート |
| `/exit` (`/quit`) | CLI終了 |
| `/btw <question>` | コンテキストに残らないサイドクエスチョン。回答画面で `c` キーを押すと生 Markdown をクリップボードへコピー（v2.1.163） |

### 2.2 設定・情報

| コマンド | 説明 |
|:--|:--|
| `/config` (`/settings`) | 設定インターフェース表示。v2.1.181 で `/config key=value` 構文により任意設定をプロンプトから直接変更可能（インタラクティブ / `-p` / Remote Control 対応、例: `/config thinking=false`）。`/config --help` でショートハンドキー一覧（v2.1.183）。v2.1.202 で「Dynamic workflow size」設定（動的ワークフローのエージェント数目安 small/medium/large）追加 |
| `/doctor` (`/checkup`) | セットアップの総合チェックアップ。問題の診断と修正まで実行（v2.1.205 で強化・`/checkup` エイリアス追加）。CLAUDE.md の冗長部分の削減提案も（v2.1.206） |
| `/permissions` (`/allowed-tools`) | 権限設定。allow / ask / deny ルールをスコープ別に閲覧・追加・削除し、作業ディレクトリ管理と Auto mode の拒否履歴レビューを行う。**v2.1.246 で Auto mode タブが追加**され、Auto mode 分類器ルールの閲覧・編集もダイアログから可能に。応答中に実行するとダイアログが即座に開き、変更は同一ターンの次のツール呼び出しから適用される（v2.1.234 以降） |
| `/fast [on\|off]` | Fastモードトグル |
| `/model [model]` | モデル変更（v2.1.144 から現在セッションにのみ適用。新規セッションのデフォルト変更はピッカーで `d` キー、保存せず切り替えるだけなら `s` キー）。ピッカーの effort スライダを矢印キーで動かして effort を設定でき、`ultracode` まで動かすと当該セッションで ultracode が有効になる（そのモデルを既定として保存した場合でもセッション限定）。低〜`xhigh` を選ぶと `modelSettings` にモデル別で保存される |
| `/effort [level\|auto\|status]` | エフォートレベル設定。引数なしで対話スライダ、レベル名で直接設定、`auto` で**現在のモデルの保存済みレベルをクリア**（他モデルのエントリと最上位の `effortLevel` は残る）、`status` で現在値を表示。`max` と `ultracode` はセッション限定（設定ファイルには保存されない）。**v2.1.251 でモデルごとに既定 effort レベルを保存**するようになり、保存先は `effortLevel` ではなく設定キー [`modelSettings`](configuration.md)（user 設定）に変わった。モデルを切り替えても各モデルの設定が保持される。Claude の作業中に実行しても即時反映（キュー待ちしない）。v2.1.154 でスライダのラベルが「Speed」/「Intelligence」→「Faster」/「Smarter」に変更。v2.1.161 でダイアログが「Reduce motion」アクセシビリティ設定を尊重 |
| `/memory` | CLAUDE.md/オートメモリ管理 |
| `/hooks` | フック設定表示 |
| `/mcp` | MCPサーバー管理。v2.1.161 で未使用 claude.ai connector を「Show unused connectors」の下に折りたたみ表示に変更 |
| `/status` | ステータス表示。v2.1.221 からセッション種別（`interactive` / バックグラウンドの `attached` / `unattended`）も表示。セッション間メッセージが有効なセッションでは自身の受信箱アドレスを `Peer address` 行（`uds:` プレフィックス）に表示。v2.1.243 で `Skipped sources` 行（`managed-settings.json` 等、存在するが上位の managed ソースが有効なため適用されない managed 設定ソース）と、Claude Code on the web 向けの GitHub 接続状況行（未接続なら `/web-setup` を案内）を追加 |
| `/list-agents` | Claude が到達可能なエージェント一覧（サブエージェント / 同一マシンの他セッション / クラウドセッション / 他マシンの Remote Control セッション）。エイリアス `/peers`。セッション間メッセージ機能の有無を確認する手段でもある（コマンド自体が未認識ならその機能を持たない）。ローカルセッションは作業ディレクトリも表示され、同名セッションの区別が可能 |
| `/context` | コンテキスト使用量の可視化 |
| `/usage` | 使用量・統計を統合表示。**v2.1.251 で Spend limit バーを追加**（Claude apps gateway 配下で spend limit がある開発者向け。ステータスライン用の `rate_limits.spend_limit` フィールドも追加）。（v2.1.118 で `/cost` と `/stats` を `/usage` に統合。両コマンドはタイピングショートカットとして残存、対応タブを開く。v2.1.149 で制限使用量を駆動する要因（skills / subagents / plugins / MCPサーバー単位のコスト）のカテゴリ別内訳を表示。v2.1.243 で Loops 内訳（ループ単位の実行回数・総トークン・1 実行あたりトークン・最終実行時刻）を追加し、暴走した `/loop` を特定しやすくした） |
| `/cost` | `/usage` のトークン使用量タブを開くショートカット。**v2.1.251 でセッション単位のプロンプトキャッシュ行を追加**（ヒット率・ミス・再キャッシュしたトークン数・warm/cold）。同じ情報はステータスラインスクリプト向けに `prompt_cache` オブジェクトとしても提供される |
| `/stats` | `/usage` の日次使用量・セッション履歴タブを開くショートカット |
| `/insights` | セッション分析レポート（プロジェクト領域、操作パターン、摩擦点） |
| `/usage-credits` | 追加使用量情報（v2.1.113 で Remote Control クライアントからも利用可能に。v2.1.144 で `/extra-usage` から改名、旧名もエイリアスとして残存。v2.1.161 で Team / Enterprise admin に再ログインを開始する代わりに組織 usage settings ページへ案内するよう修正。v2.1.236 で `/usage` が Team / Enterprise メンバーにも usage-credits の消費行を表示するようになり、未消費時は 0% の行を表示） |
| `/tui` | フリッカーフリー・フルスクリーン描画の切り替え（v2.1.110） |
| `/focus` | フォーカスモード切替（v2.1.110。brief・focus モードは v2.1.101 で改善） |

### 2.3 開発ワークフロー

| コマンド | 説明 |
|:--|:--|
| `/init` | CLAUDE.md の自動生成 |
| `/diff` | 差分ビューア |
| `/plan` | プランモード開始 |
| `/powerup` | Claude Code機能のインタラクティブレッスン＋アニメーションデモ |
| `/security-review` | セキュリティ脆弱性分析 |
| `/review <pr>` | PRレビュー。v2.1.186 で `/code-review medium` と同じエンジンに変更されたが、v2.1.202 で高速な単一パスレビューに回帰（多段レビューは `/code-review <level> <pr#>` を使用） |
| `/dataviz` | チャート・ダッシュボード設計ガイダンススキル（カラーパレットバリデータ付き）（v2.1.198） |
| `/commit-push-pr` | commit→push→PR作成。v2.1.206 でリポジトリの push remote（`remote.pushDefault` または唯一のremote）への `git push` を自動許可。v2.1.229 で `--force`/`--amend`/`--no-verify` 等の危険フラグを含む git/gh コマンドは自動承認しないよう変更 |
| `/release-notes` | インタラクティブバージョンピッカー付き変更ログ表示 |
| `/sandbox` | サンドボックスモード切替 |
| `/schedule [description]` | クラウドスケジュールタスクの作成・管理 |
| `/autofix-pr [prompt]` | 現在のブランチのPRを監視するクラウドセッション（Claude Code on the web）を起動し、CI失敗・レビューコメントに自動で修正pushする Auto-fix を有効化。`gh pr view` でPRを検出。プロンプトで対応方針を指定可（例: `/autofix-pr only fix lint and type errors`）。`gh` CLI とweb版アクセスが必要 |

### 2.4 環境・インテグレーション

| コマンド | 説明 |
|:--|:--|
| `/add-dir <path>` | ワーキングディレクトリ追加 |
| `/agents` | ~~サブエージェント管理~~ **v2.1.198 でウィザード削除**。サブエージェントの作成・管理は Claude に依頼するか `.claude/agents/` を直接編集 |
| `/skills` | スキル一覧。type-to-filter 検索ボックスで即座に絞り込み（v2.1.121。v2.1.247 から name だけでなく description / source も検索対象）。`t` でトークン数ソート、`Space` または `Enter`（`Enter` は v2.1.247+）で Claude と `/` メニューに対する可視性をサイクル、`Esc` で保存して閉じる。プラグインスキル・frontmatter に `disable-model-invocation: true` を持つスキル・managed settings や `--settings` の `skillOverrides` 対象のスキルはサイクル不可 |
| `/plugin` | プラグイン管理（マーケットプレース、インストール、有効化/無効化）。`claude plugin prune` で孤立した自動インストール依存を削除、`plugin uninstall --prune` でカスケード削除（v2.1.121）。マーケットプレース browse ペインに projected context cost（ターン当たり・呼び出し当たりのトークン推定）を表示（v2.1.143）。Discover/Browse 画面でインストール前にプラグインが提供する commands / agents / skills / hooks / MCP/LSP サーバーをプレビュー（v2.1.145） |
| `/plugin list` | インストール済みプラグイン一覧表示。`--enabled` / `--disabled` フィルタ対応（v2.1.163） |
| `claude plugin enable/disable` | 依存関係を強制。`disable` は他の有効プラグインの依存先を拒否し disable-chain ヒントを表示。`enable` は推移的依存を強制有効化（v2.1.143） |
| `/reload-plugins` | プラグイン変更の即時反映（v2.1.221 から `/plugin` 経由のインストールは安全な場合、実行不要で即時有効化） |
| `/reload-skills` | スキル / コマンドディレクトリを再スキャン。セッション再起動不要。利用可能スキル数と増減数を報告する |
| `/desktop` (`/app`) | デスクトップアプリでセッション継続 |
| `/remote-control` (`/rc`) | リモートコントロール有効化 |
| `/teleport` (`/tp`) | Claude Code on the web のクラウドセッションをこの端末に引き込む（ピッカー→ブランチfetch＋会話履歴ロード）。claude.ai サブスクリプション認証が必要 |
| `/tasks` (`/bashes`) | 現在セッションのバックグラウンド作業（完了済みサブエージェント含む）の表示・管理。v2.1.242 以降、各サブエージェントが動いたモデルを行に表示し、サブエージェント定義（またはフォーク元スキル）が `effort` を設定している場合は effort レベルも併記する |
| `/web-setup` | ローカルの `gh` CLI 認証情報で GitHub アカウントを Claude Code on the web に接続。**v2.1.248**: `gh` のトークンに `workflow` スコープが無い場合、巨大リポジトリへの push が拒否されうる旨を警告する |
| `/remote-env` | クラウドセッションのデフォルト環境をピッカーで選択（ユーザー設定 `remote.defaultEnvironmentId` に保存） |
| `/ide` | IDE連携管理 |
| `/chrome` | Chrome設定。v2.1.154 で "Select browser…" を選択することで複数接続済みブラウザから使用ブラウザを指定可能（チャット内でも実行時に選択可） |
| `/voice` | 音声入力トグル |
| `/color` | プロンプトバー色変更 |
| `/copy [N]` | レスポンスのコピー（インタラクティブピッカー。Nで直接指定。`w`キーでファイル書き出し） |
| `/login` / `/logout` | 認証 |

### 2.5 MCP プロンプト

MCPサーバーが公開するプロンプトは `/mcp__<server>__<prompt>` 形式でコマンドとして表示される。

### 2.6 その他の組み込みコマンド

2026-08-21 巡回で公式 commands リファレンスと突合し、未収載だったコマンドを補完。

| コマンド | 説明 |
|:--|:--|
| `/goal [condition\|clear]` | ゴールを設定し、条件を満たす（または別の理由でクリアされる）までターンをまたいで作業を継続。引数なしで現在のゴールを表示。`CLAUDE_CODE_GOAL_CHECKIN_MINUTES` で停滞時の自動チェックイン間隔を制御 |

**`/goal` がクリアされる条件**: ①条件達成、②評価モデルが「条件は達成不能」と判断（トランスクリプトに理由付きで failed 記録）、③**ユーザーが直さない限り解消しないエラーでターンが失敗**、④`/goal clear`。③は `Goal cleared after an unrecoverable error` で始まり `Run /goal again to continue` で終わる警告を表示し、以下の4種のみが該当する:

- 認証失敗（Claude Code 自身が認証情報を管理している場合のみ。Desktop / VS Code 拡張 / クラウドセッションなどホスト側が管理する場合はホストが復旧するためゴールは維持される）
- クレジット残高の枯渇
- auto-compaction でも解消できないコンテキストあふれ
- 利用不可のモデル

**アイドルチェックインの上限（v2.1.246）**: バックグラウンド作業がゴール評価を待たせている間、アイドルセッションから自動的に開始されるチェックインは **ユーザーのプロンプト間で 1 ゴールあたり最大 3 回**。3 回目のチェックインで「次のプロンプトまでアイドルチェックインを停止する」旨を告知し、ユーザーが次のメッセージを送るとさらに 3 回分が許可される（v2.1.245 以前は上限なし）。

レート制限・サーバー過負荷などの一時的エラーではゴールはクリアされない。ハーネス設計上、`/goal` を長時間の自律ループに使う場合はこの4条件が実質的な停止点になる。
| `/recap` | 現在のセッションの1行サマリをオンデマンド生成（離席後に自動表示されるリキャップの手動版）。v2.1.236 で 400 文字・語境界で打ち切り |
| `/background [prompt]` | 現在のセッションをバックグラウンドエージェントとしてデタッチし端末を解放。プロンプトを渡すとデタッチ前に最後の指示を送る |
| `/stop` | バックグラウンドセッションを停止（アタッチ中のみ表示）。トランスクリプトと worktree は保持。停止せずデタッチするには `/exit` |
| `/artifacts` | 自分が所有する、または共有された Artifact 一覧を表示し、セッションへの添付・ブラウザで開く・リンクのコピーを行う。Artifact が利用可能な環境でのみ使用可（v2.1.208 以降。`Enter` での添付は v2.1.216 以降） |
| `/auto-mode-setup` | プロジェクトと直近セッションから `autoMode.environment` エントリのドラフトを生成し、レビューしてユーザー設定へ保存する。Pro / Max / Team プランかつ v2.1.228 以降（ネイティブ Windows は v2.1.233 以降） |
| `/advisor [model\|off]` | Advisor ツール（タスク中の要所で第2のモデルに助言を求める）の有効化 / 無効化。`fable` / `opus` / `sonnet` / 完全なモデル ID を受け付ける |
| `/autocompact [auto\|<tokens>]` | 自動コンパクション発動のコンテキスト使用量を設定（例 `500k`、`auto` でモデル既定に戻す） |
| `/deep-research <question>` | **バンドルワークフロー**。Web 検索を fan-out し、ソースを取得・相互検証して引用付きレポートを合成 |
| `/run` | **バンドルスキル**。テストだけでなく実アプリを起動・操作して変更の動作を確認 |
| `/run-skill-generator` | **バンドルスキル**。プロジェクト固有のビルド / 起動 / 操作手順をスキルとして生成し、`/run` と `/verify` に教える |
| `/design [brief]` | **バンドルスキル**（リサーチプレビュー、v2.1.234 以降）。Claude Design のアートボードワークフローを CLI / Claude Code Desktop に持ち込み、Artifacts 基盤で編集可能なアートボードのキャンバスを公開する。ブリーフを渡すと UI 案を複数生成し、ユーザーが選んだ案を Claude に実装させる流れ。Pro / Max / Team / Enterprise、かつ [Artifacts が利用可能なセッション](https://code.claude.com/docs/en/artifacts#availability)で利用可。アカウントで保存が有効ならキャンバス上でアートボードを編集して新バージョンを公開でき、無効ならドラフトの閲覧と PNG / PDF エクスポートのみ。2026-09-01 巡回で公式 commands リファレンスに収載されたことを確認 |
| `/design-sync [hint]` | **バンドルスキル**。リポジトリの React デザインシステムを変換して Claude Design にアップロードし、生成デザインに実コンポーネントを使わせる |
| `/design-login` | claude.ai アカウントで `/design-sync` のデザインシステムアクセスを認可 |
| `/fewer-permission-prompts` | **バンドルスキル**。トランスクリプトから頻出の読み取り専用 Bash / MCP 呼び出しを抽出し、プロジェクト `.claude/settings.json` に優先度付き allowlist を追加 |
| `/import [codex\|gemini] [--dry-run] [--yes]` | 他のコーディングエージェント（OpenAI Codex / Google Gemini CLI）の設定（指示ファイル・MCP サーバー・コマンド・サブエージェント・スキル）を Claude Code に取り込む |
| `/statusline` | ステータスラインの設定。引数なしでシェルプロンプトから自動構成 |
| `/keybindings` | キーボードショートカット設定ファイル（`~/.claude/keybindings.json`）を開く。キー名は**大文字小文字を区別しない**（`K` は `k` と同一。Shift 併用は `shift+k` と明記する必要がある）。`wheelup` / `wheeldown` がマウスホイールイベントとして指定でき、既定で `scroll:lineUp` / `scroll:lineDown` にバインドされている。未知のアクション名を指定したバインドは v2.1.246 からスキップされ（既定バインドが維持され `--debug` で警告）、以前はそのキーが無効化されていた。v2.1.247 で `chat:queueSubmit`（既定 `Ctrl+X Enter`。作業中でも割り込まずキューに積む送信。オートコンプリート表示中でも送信される）と `select:pageUp` / `select:pageDown` / `select:first` / `select:last`（PageUp / PageDown / Home / End。適用は `/skills` メニュー。`/model` 等では既定の PageUp / PageDown が使われ Home / End は無視）を追加。キー名に `pageup` / `pagedown` / `home` / `end` が使えるようになった。`ctrl+x` を単独キーとして奪うには `ctrl+x ctrl+k` / `ctrl+x ctrl+e` / `ctrl+x enter`（Chat）と `ctrl+x ctrl+b`（Task）を全て `null` にする。再バインド不可: `Ctrl+C` / `Ctrl+D` / `Ctrl+M`（Enter として届く）/ `Ctrl+[`（Escape として届く）/ `Ctrl+I`（Tab として届く）/ `Ctrl+H`（ASCII backspace）/ Caps Lock。非 Latin レイアウト（キリル等）では Kitty キーボードプロトコル対応端末なら US 配列の物理位置でマッチし、AZERTY 等の Latin 系再配置レイアウトでは「そのキーが打つ文字」でマッチする（v2.1.247 以前は Kitty プロトコル端末の非 Latin レイアウトで Ctrl ショートカットが発火しなかった） |
| `/theme` | カラーテーマ変更（`auto`・light/dark・色覚多様性対応（daltonized）・ANSI テーマ等） |
| `/scroll-speed` | マウスホイールのスクロール速度をインタラクティブ調整（フルスクリーン描画時） |
| `/terminal-setup` | Shift+Enter 等の端末キーバインド設定（VS Code / Cursor / Alacritty / Zed 等、必要な端末でのみ表示） |
| `/install-github-app` | リポジトリへの Claude GitHub App インストール（GitHub Actions ワークフロー・シークレット設定も任意で実施） |
| `/install-slack-app` | Claude Slack アプリのインストール（ブラウザで OAuth フローを完了） |
| `/setup-bedrock` | Amazon Bedrock の認証・リージョン・モデルピンをウィザードで設定（`CLAUDE_CODE_USE_BEDROCK=1` 時のみ表示） |
| `/setup-vertex` | Google Cloud Agent Platform の認証・プロジェクト・リージョン・モデルピンをウィザードで設定（`CLAUDE_CODE_USE_VERTEX=1` 時のみ表示） |
| `/bug [report]` | バグ報告 / 会話共有。送信履歴の範囲を選択し同意画面で確認してから送信 |
| `/feedback [report]` | Claude Code へのプロダクトフィードバック送信（`/bug` と同じダイアログ・同意フロー）。Claude 起草フィードバック（`SendFeedback` ツール）が有効なセッションでは、**引数なしの `/feedback` はドラフトキュー**を開き、Claude がキューしたドラフトのレビュー・編集・送信・破棄ができる（`w` で通常ダイアログ）。引数付き `/feedback` と `/bug` は常にダイアログを直接開く（v2.1.247） |
| `/privacy-settings` | プライバシー設定の確認・更新（Pro / Max プランのみ） |
| `/upgrade` | 上位プランへのアップグレードページをブラウザで開く |
| `/rate-limit-options` | claude.ai の利用上限でリクエストが止まった際の選択肢メニュー（リセットまで待って自動継続 / usage credits 追加 / プランのアップグレード）を開く。自分の端末で上限に達した場合は Claude Code 側から自動的に開くこともある。自動継続の ON/OFF は `autoContinueAtUsageLimit` 設定 |
| `/passes` | Claude Code の1週間無料パスを友人に共有（対象アカウントのみ表示） |
| `/mobile` | Claude モバイルアプリのダウンロード QR を表示。エイリアス `/ios`・`/android` |
| `/heapdump` | JavaScript ヒープスナップショットとメモリ内訳を `~/Desktop`（Linux では home）に書き出す。メモリ使用量診断用 |
| `/help` | ヘルプと利用可能コマンド一覧 |
| `/radio` | Claude FM（lo-fi ラジオ）をブラウザで開く。ブラウザが無い環境ではストリーム URL を表示。**v2.1.251 で Bedrock / Vertex AI / Microsoft Foundry / AWS 上の Claude Platform、およびテレメトリ無効時でも利用可能になった**（従来はこれらで利用不可） |
| `/stickers` | Claude Code ステッカーの注文 |

**削除済みコマンド**（公式リファレンスに履歴として残るもの）:

| コマンド | 状態 |
|:--|:--|
| `/pr-comments [PR]` | v2.1.91 で削除。PR コメントの参照は Claude に直接依頼する |
| `/vim` | v2.1.92 で削除。`/config` → Editor mode でトグルする |
| `/ultraplan <prompt>` | 削除。プランモードを使用する（以前は Claude Code on the web セッションへ計画タスクを送っていた） |

---

## 3. スケジュールタスク

公式ドキュメント: https://code.claude.com/docs/en/scheduled-tasks / https://code.claude.com/docs/en/web-scheduled-tasks

### 3.1 3つのスケジューリング方式

| | Cloud | Desktop | `/loop` |
|:--|:--|:--|:--|
| 実行環境 | Anthropicクラウド | ローカルマシン | ローカルマシン |
| マシン起動が必要 | No | Yes | Yes |
| セッション不要 | Yes | Yes | No（セッションスコープ） |
| 再起動後の永続性 | Yes | Yes | No |
| ローカルファイルアクセス | No（毎回clone） | Yes | Yes |
| MCPサーバー | タスクごとにコネクタ設定 | 設定ファイル＋コネクタ | セッション継承 |
| 最小間隔 | 1時間 | 1分 | 1分 |

### 3.2 `/loop`（セッション内スケジューリング）

```text
/loop 5m check if the deployment finished
/loop 20m /review-pr 1234
```

- 間隔構文: `30m`, `2h`, `every 2 hours`, 省略時10分
- 単位: `s`（秒→分に丸め）, `m`（分）, `h`（時間）, `d`（日）
- 内部ツール: `CronCreate`, `CronList`, `CronDelete`
- セッションあたり最大50タスク
- 繰り返しタスクは3日で自動期限切れ
- ジッター: 繰り返しタスクは周期の10%（最大15分）遅延、一発タスクは最大90秒早期発火
- 無効化: `CLAUDE_CODE_DISABLE_CRON=1`

### 3.3 クラウドスケジュールタスク

作成方法:
- Web: `claude.ai/code/scheduled` → New scheduled task
- デスクトップアプリ: Schedule → New task → New remote task
- CLI: `/schedule`

設定項目:
- **プロンプト**: 自律実行のため自己完結的に記述
- **リポジトリ**: 1つ以上。毎回デフォルトブランチからclone。`claude/` プレフィックスブランチにpush
- **環境**: ネットワークアクセス、環境変数、セットアップスクリプト
- **頻度**: Hourly / Daily（デフォルト9:00 AM） / Weekdays / Weekly。最小1時間
- **コネクタ**: claude.aiのMCPコネクタ

管理: `/schedule list`, `/schedule update`, `/schedule run`

---

## 4. プラグインマーケットプレース

公式ドキュメント: https://code.claude.com/docs/en/discover-plugins

### 4.1 公式マーケットプレース

`claude-plugins-official` は自動利用可能。`/plugin` → Discover タブで閲覧。

```bash
/plugin install github@claude-plugins-official
```

`/plugin` の引数は autocomplete に対応（v2.1.157）。サブコマンド、インストール済みプラグイン名、既知マーケットプレース上のプラグインが補完される。

### 4.2 マーケットプレースの追加

```bash
/plugin marketplace add anthropics/claude-code       # GitHub
/plugin marketplace add https://gitlab.com/company/plugins.git  # Git URL
/plugin marketplace add ./my-marketplace             # ローカル
/plugin marketplace add https://example.com/marketplace.json    # URL
```

**GitLab 対応（v2.1.232）**: 素の `gitlab.com` リポジトリ URL（ネストしたサブグループを含む）が `github.com` URL と同様に clone されるようになった。clone 認証に失敗した場合のヒントは実際の git ホスト名を表示する。

**`/plugin install plugin@marketplace`（v2.1.232）**: インストール前にマーケットプレースを自動更新するため、新規公開されたプラグインを手動の marketplace update なしでインストールできる。

### 4.2.1 `.claude/skills/` 配下のプラグイン自動ロード（v2.1.157）

`.claude/skills/` 配下に置かれたプラグインはマーケットプレース不要で自動ロードされる。雛形は `claude plugin init <name>` で生成可能。

```bash
claude plugin init my-plugin    # .claude/skills/my-plugin/ を生成
```

### 4.3 プラグインカテゴリ

| カテゴリ | 内容 |
|:--|:--|
| **コードインテリジェンス** | LSPプラグイン（11言語: C/C++, C#, Go, Java, Kotlin, Lua, PHP, Python, Rust, Swift, TypeScript）。自動診断＋コードナビゲーション |
| **外部インテグレーション** | github, gitlab, atlassian, asana, linear, notion, figma, vercel, firebase, supabase, slack, sentry |
| **開発ワークフロー** | commit-commands, pr-review-toolkit, agent-sdk-dev, plugin-dev |
| **出力スタイル** | explanatory-output-style, learning-output-style |

### 4.4 インストールスコープ

| スコープ | 対象 |
|:--|:--|
| User（デフォルト） | 全プロジェクトの自分 |
| Project | リポジトリの全コラボレーター |
| Local | このリポジトリの自分のみ |
| Managed | IT管理者による配布（変更不可） |

### 4.5 チームマーケットプレース

`.claude/settings.json` の `extraKnownMarketplaces` でチーム自動インストール設定可能。

### 4.6 自動更新

公式マーケットプレースはデフォルトで自動更新有効。`DISABLE_AUTOUPDATER=1` で全無効化。`FORCE_AUTOUPDATE_PLUGINS=1` でプラグインのみ自動更新維持。

### 4.7 security-guidance プラグイン（公式 / 2026-w22 featured）

公式ドキュメント: https://code.claude.com/docs/en/security-guidance

`claude-plugins-official` 配下の公式プラグイン。Claude のコード変更を 3 段階で自動レビューし、同一セッション内で修正させる。要 v2.1.144 以降 + Python 3.8 以降 + git リポジトリ（end-of-turn / commit レビューのみ git 必須、per-edit パターン照合は git 不要）。

**3 段階レビュー**:

| 段階 | 内容 | モデル呼出 |
|:--|:--|:--|
| **per-edit** | `Edit`/`Write`/`NotebookEdit` 後のパターン照合（`eval(`, `new Function`, `pickle`, `dangerouslySetInnerHTML`, `.github/workflows/` 等）| なし（決定的文字列照合）|
| **end-of-turn** | `Stop` フックで working-tree diff を別 Claude セッションでセキュリティレビュー（バックグラウンド実行）| あり（既定 Opus 4.7、`SECURITY_REVIEW_MODEL` で上書き）|
| **commit/push** | Claude が `git commit` / `git push` した際に周辺コードを読む agentic レビュー（rolling hour あたり 20 回まで）| あり（既定 Opus 4.7、`SG_AGENTIC_MODEL` で上書き）|

**インストール**:

```text
/plugin install security-guidance@claude-plugins-official
/reload-plugins
```

cloud / 共有レポでは `.claude/settings.json` の `enabledPlugins` で宣言可能。組織横断有効化は managed-settings 経由。

**カスタムルール**:

| ファイル | 用途 |
|:--|:--|
| `.claude/claude-security-guidance.md` | model-backed レビュー向けの追加ガイダンス（脅威モデル・チェックリストを自然言語で記述）。user/project/project local 配置可、合計 8 KB まで |
| `.claude/security-patterns.yaml` | per-edit パターン照合の追加ルール（`rule_name` / `reminder` / `regex` or `substrings` / `paths` / `exclude_paths`）。最大 50 ルール、`.json` 形式も可 |

**レイヤー別無効化（環境変数）**:

| 変数 | 効果 |
|:--|:--|
| `ENABLE_PATTERN_RULES=0` | per-edit 無効化 |
| `ENABLE_STOP_REVIEW=0` | end-of-turn レビュー無効化 |
| `ENABLE_COMMIT_REVIEW=0` | commit/push レビュー無効化 |
| `ENABLE_CODE_SECURITY_REVIEW=0` | model-backed レビュー全停止 |
| `SECURITY_GUIDANCE_DISABLE=1` | プラグイン全停止 |

**実装**: SessionStart / UserPromptSubmit / PostToolUse（Edit/Write/NotebookEdit, Bash filtered to git commit/push）/ Stop の hook 構成のみ。診断ログは `~/.claude/security/log.txt`。harness-harness 側でフック実装の参考になる。

**位置づけ**: 防御の多層化の最も早い層。PR 時の `/code-review`（Team/Enterprise）、CI の静的解析と組み合わせる。プラグインだけでセキュリティを保証するものではない。

### 4.8 claude-security プラグイン（公式 / 2026-07 新設ドキュメント）

公式ドキュメント: https://code.claude.com/docs/en/claude-security

`claude-plugins-official` 配下の公式プラグイン。セッション内でマルチエージェント脆弱性スキャンを実行するオンデマンド・ディープスキャン層。エージェントチームがアーキテクチャ把握→脅威モデル構築→脆弱性ハント→独立検証エージェントによる全 finding レビュー→レポート作成を行う。

**前提条件**: Claude Code v2.1.154 以降＋有料プラン（動的ワークフロー必須。Pro は `/config` の Dynamic workflows で有効化）、`python3` 3.9 以降（標準ライブラリのみ使用。2026-09-01 の公式ドキュメント改訂で 3.9.6 → 3.9 に修正）、変更スキャン・パッチ生成には git。

**インストール**:

```text
/plugin install claude-security@claude-plugins-official
/reload-plugins
```

**提供コマンド**: `/claude-security` の1つ。メニューから3ジョブを選択（引数・自然言語での直接指定も可）:

| ジョブ | 内容 |
|:--|:--|
| **Scan codebase** | リポジトリ全体 or 選択領域のフルスキャン。バージョン管理外ディレクトリも可 |
| **変更のみスキャン** | ブランチ diff / PR / 単一コミットを対象。コミット済みの変更のみ（要 git） |
| **Suggest patches** | finding を選んでパッチ生成。独立レビューエージェントが検証してから `patches/F<n>.patch` に出力 |

**結果出力**: `CLAUDE-SECURITY-<timestamp>/` ディレクトリに `RESULTS.md`（finding ID・severity・confidence 付き）、`RESULTS.jsonl`、`REVISION-<commit>.json`（スキャン対象コミットの記録）。ディレクトリ自身が `.gitignore` を持ち誤コミットを防止。

**パッチは自動適用されない**: `git apply` でユーザー自身が適用。パッチごとに独立 PR 推奨。

**注意点**: スキャンはセッション権限で動き独自の隔離なし。信頼できないリポジトリのスキャンは sandbox-runtime 等でセッションごとサンドボックス化する。スキャンは非決定的（同一コードでも finding が変わりうる）。

**位置づけ（多層防御での役割分担）**: security-guidance（書きながらレビュー）→ `/security-review`（ブランチの単発パス）→ **claude-security（オンデマンド・ディープスキャン）** → Code Review（PR 時）→ Claude Security 管理サービス（Enterprise）→ CI の静的解析。GitLab / Bitbucket などマネージド製品が届かないリポジトリにも使える。

---

## 5. Agents / Subagents

### 5.1 概要

サブエージェントは独自のコンテキストウィンドウ、システムプロンプト、ツールアクセスを持つ専門AIアシスタント。メインの会話コンテキストを消費せずにタスクを委譲できる。

### 5.2 ビルトインサブエージェント

| エージェント | モデル | 用途 |
|:--|:--|:--|
| **Explore** | メインセッションのモデルを継承（上限 Opus。v2.1.198 で Haiku 固定から変更） | 読み取り専用のコードベース探索。quick/medium/very thorough の3段階 |
| **Plan** | 継承 | プランモード時のリサーチ。読み取り専用 |
| **general-purpose** | 継承 | 探索と変更の両方が必要な複雑タスク |
| **Bash** | 継承 | 別コンテキストでのターミナルコマンド実行 |
| **statusline-setup** | Sonnet 固定 | ステータスライン設定用の専用エージェント |
| **Claude Code Guide** | Haiku 固定 | Claude Code の使い方ガイド |
| **claude** | 固定モデルを持たず、上記「サブエージェントのモデル優先順位」に従う | 専門エージェントに当てはまらないタスク向けの catch-all。サブエージェントが使える全ツールを持ち、dispatched バックグラウンドセッションの既定エージェントでもある |

### 5.3 カスタムサブエージェントの作成

Markdown ファイル + YAML フロントマターで定義:

```markdown
---
name: code-reviewer
description: コード品質とベストプラクティスのレビュー
tools: Read, Grep, Glob, Bash
model: sonnet
---

あなたはシニアコードレビュアーです。...
```

`description` はメイン会話のコンテキストを常時消費するため短く保つ。**組み込み以外のサブエージェントの `description` 合計が 15,000 トークンを超えると起動時に合計トークン数付きの警告が出る**。詳細は各サブエージェントのシステムプロンプト（そのサブエージェント実行時にのみ読み込まれる）へ移す。

`experimental.cacheTtl`（`"5m"` / `"1h"`、v2.1.248）: サブエージェント TTL 設定（`subagentPromptCacheTtl` / `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`）が未構成のときに使われる、エージェント単位のプロンプトキャッシュ TTL。

### 5.4 サブエージェントのスコープ

| 場所 | スコープ | 優先度 |
|:--|:--|:--|
| `--agents` CLIフラグ | 現在のセッションのみ | 1（最高） |
| `.claude/agents/` | プロジェクト | 2 |
| `~/.claude/agents/` | 全プロジェクト | 3 |
| プラグインの `agents/` | プラグイン有効時 | 4（最低） |

### 5.5 フロントマターフィールド

| フィールド | 必須 | 説明 |
|:--|:--|:--|
| `name` | Yes | 一意識別子（小文字+ハイフン） |
| `description` | Yes | Claude が委譲判断に使用する説明 |
| `tools` | No | 許可ツール。省略時は全ツール継承 |
| `disallowedTools` | No | 拒否ツール |
| `model` | No | `sonnet` / `opus` / `haiku` / `fable` / `inherit` / フルモデルID（`claude-opus-5` 等）。**省略時は下記のサブエージェントモデル優先順位に従う**（v2.1.251 以前は `inherit` が既定扱い） |
| `permissionMode` | No | `default` / `acceptEdits` / `dontAsk` / `bypassPermissions` / `plan` |
| `maxTurns` | No | 最大エージェンティックターン数。**上限に達した場合、Claude Code は出力を「部分的（partial）」とマークして返し、Claude は[サブエージェントの resume](https://code.claude.com/docs/en/sub-agents#resume-subagents) で継続できる**（partial マークは v2.1.246 以降。エージェントIDを返すサブエージェントでは「メッセージを送れば続きから再開できる」旨も結果に付く） |
| `skills` | No | 起動時にプリロードするスキル |
| `mcpServers` | No | スコープされたMCPサーバー |
| `hooks` | No | ライフサイクルフック |
| `memory` | No | 永続メモリスコープ (`user` / `project` / `local`) |
| `background` | No | `true` でバックグラウンドタスクとして実行 |
| `effort` | No | エフォートレベル |
| `isolation` | No | `worktree` で一時ワークツリーでの隔離実行 |
| `initialPrompt` | No | 最初のターンで自動送信するプロンプト（v2.1.83） |

#### サブエージェントのモデル優先順位（v2.1.251〜）

**最初に該当したもの**が使われる:

1. 呼び出しごとの `model` パラメータ（Agent ツール等でメインの Claude が指定した値）
2. サブエージェント定義の `model` frontmatter（`inherit` はメイン会話のモデルを選ぶ）
3. `CLAUDE_CODE_SUBAGENT_MODEL` 環境変数（モデルエイリアスまたはモデルID。`inherit` は未設定と同義）
4. メイン会話のモデル

- **v2.1.251 より前はこの順序の先頭が `CLAUDE_CODE_SUBAGENT_MODEL`** で、呼び出しごとの指定と frontmatter（`model: inherit` を含む）を上書きしていた
- `CLAUDE_CODE_SUBAGENT_MODEL` を設定するだけでは、ビルトインの Explore / Plan サブエージェントが動くモデルは変わらない
- 組織の `availableModels` 許可リストでブロックされた値は、ファミリーエイリアスなら許可リスト内の最新版に置換され、それ以外は継承モデルへフォールバックする（`CLAUDE_CODE_SUBAGENT_MODEL` を設定している場合は同じ規則の下でまずそのモデルが試される）。インタラクティブセッションでは置換時に要求モデルと置換モデルを示す警告が出る

### 5.6 呼び出し方法

- **自然言語**: エージェント名をプロンプトに含める
- **@メンション**: `@"code-reviewer (agent)"` でエージェント指定を保証
- **セッション全体**: `claude --agent code-reviewer` または設定 `"agent": "code-reviewer"`。v2.1.157 から `claude agents` の dispatched セッションでも `settings.json` の `agent` フィールドが尊重される（`--agent <name>` で上書き可能）

### 5.7 フォアグラウンド/バックグラウンド

- **v2.1.198 からサブエージェントはデフォルトでバックグラウンド実行**。Claude は実行中も作業を継続し、完了時に通知を受け取る
- **フォアグラウンド**: メイン会話をブロック。権限プロンプトと質問がパススルー
- **バックグラウンド**: 並行実行。`Ctrl+B` でバックグラウンド化。v2.1.186 から権限プロンプトは自動拒否ではなくメインセッションにサーフェスされる（どのエージェントの要求かを表示、Esc でそのツールのみ拒否）

### 5.7.1 ネスト実行（v2.1.172+）

サブエージェントは自身のサブエージェントをスポーン可能（最大5階層。v2.1.181 でフォアグラウンドにも同じ深度制限を適用）。サブエージェントとコンパクションはセッションの拡張思考設定を継承する（v2.1.198）。

### 5.8 サブエージェントの再開

完了したサブエージェントは `SendMessage` ツールで再開可能。会話履歴が保持される。

- 完了時に Claude はエージェントIDを受け取る。組み込みの Explore / Plan は one-shot でエージェントIDを返さないため再開できない（継続が必要なら `general-purpose` かカスタムサブエージェントを使う）
- **`maxTurns` の上限で停止した場合**、Claude Code は返却出力を「部分的（partial）」とマークする（v2.1.246 以降）。エージェントIDを返すサブエージェントでは「メッセージを送れば停止地点から継続できる」旨も結果に付く

**フォールバックモデルチェーンの適用（v2.1.247）**: `fallbackModel` チェーンを構成している場合、サブエージェントのリクエストがチェーンの対象となる障害（モデル利用不可等）に遭うと、Claude Code はチェーンの順にモデルを試し、受理したモデルでサブエージェントを継続させる。セッション自身のモデルは変わらない。**v2.1.247 より前は、チェーンが対象とする障害でもサブエージェントは終了していた。**

### 5.9 MCPサーバーのスコープ

```yaml
---
name: browser-tester
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  - github  # 既存サーバー参照
---
```

### 5.10 永続メモリ

| スコープ | 保存先 |
|:--|:--|
| `user` | `~/.claude/agent-memory/<name>/` |
| `project` | `.claude/agent-memory/<name>/` |
| `local` | `.claude/agent-memory-local/<name>/` |

---

## 6. ヘッドレスモード（非対話モード）

公式ドキュメント: https://code.claude.com/docs/en/headless

### 6.1 基本使用法

```bash
# 単発クエリ
claude -p "このプロジェクトが何をするか説明して"

# ベアモード（高速起動、自動検出スキップ）
claude -p --bare "質問"
```

### 6.2 ベアモードのコンテキスト読み込み

`--bare` は自動検出をスキップするが、以下のフラグで選択的にコンテキストを読み込める:

| フラグ | 説明 |
|:--|:--|
| `--append-system-prompt` | システムプロンプトへの追加指示 |
| `--settings` | 設定ファイルの明示的指定 |
| `--mcp-config` | MCP設定ファイルの指定 |
| `--agents` | エージェント定義ディレクトリの指定 |
| `--plugin-dir` | プラグインディレクトリの指定 |
| `--plugin-url <url>` | URL から `.zip` プラグインアーカイブを取得して当該セッションに読み込む（v2.1.129） |

### 6.3 出力フォーマット

| フォーマット | フラグ | 用途 |
|:--|:--|:--|
| テキスト | `--output-format text` | デフォルト。人間向け |
| JSON | `--output-format json` | 最終結果の構造化出力 |
| Stream JSON | `--output-format stream-json` | リアルタイムストリーミング |

#### 型付き出力

`--json-schema` で出力スキーマを指定して構造化データを取得:

```bash
claude -p "全APIエンドポイントを一覧" --output-format json --json-schema schema.json
```

#### ストリーミング

```bash
claude -p "ログを分析" --output-format stream-json --verbose --include-partial-messages
```

`--verbose` でシステムイベント（`system`, `api_retry` 等）も含める。

### 6.4 ツール自動承認

```bash
claude -p "テストを実行して修正" --allowedTools "Edit,Bash(npm test *)"
```

### 6.5 セッション継続

```bash
# 最後のセッションを継続
claude -p --continue "前回の続き"

# 特定セッションIDで再開
claude -p --resume SESSION_ID "追加作業"
```

---

## 7. スキル一覧のコンテキストバジェット（説明の切り詰め）

**ハーネス設計上の重要な制約。** Claude Code は毎ターン「スキル名＋説明」の一覧をコンテキストに載せる。名前は必ず全件載るが、スキルが増えると説明が文字数バジェットに収まるよう切り詰められ、Claude がリクエストとマッチさせるためのキーワードが落ちることがある。

- **バジェット**: モデルのコンテキスト窓の 1%（`skillListingBudgetFraction`、デフォルト `0.01`）
- **溢れたときの挙動**: 呼び出し頻度の低いスキルから説明が落とされ、よく使うスキルはフルテキストを維持する
- **1スキルあたりの上限**: `description` + `when_to_use` の合算で 1,536 文字（`skillListingMaxDescChars`）。バジェットに関係なく適用される
- **診断**: `/doctor` が一覧のコンテキストコストと寄与の大きいスキルを推定表示。バジェット超過時は `--debug` のデバッグログにも警告が出る
- **`/context` の Skills 行**: バジェット適用後のサイズを表示（モデルが実際に受け取る量と一致）。v2.1.196 より前は全説明のフルテキストを数えており、設定バジェットの数倍の値を表示することがあった

**対処の選択肢（多様性を残す）**:

| 方針 | 手段 | トレードオフ |
|:--|:--|:--|
| バジェットを増やす | `skillListingBudgetFraction` を `0.02`（2%）等に、または `SLASH_COMMAND_TOOL_CHAR_BUDGET` で固定文字数指定 | 毎ターンのコンテキスト消費が増える |
| 低優先スキルを名前のみに | `skillOverrides` で `"name-only"` を指定 | 該当スキルは呼び出せるが内容が見えない |
| 説明を元から絞る | `description` / `when_to_use` の冒頭に主要ユースケースを置く | 執筆コストがかかるが最も副作用が少ない |

> ハーネス作成時の指針: スキルを大量に同梱する構成では、`description` の冒頭にトリガー語を置くことが実質的な必須条件になる。段階的開示（詳細は SKILL.md 本体、一覧には要約のみ）と整合する。

---

## 参考リンク

- Skills: https://code.claude.com/docs/en/skills
- 組み込みコマンド: https://code.claude.com/docs/en/commands
- サブエージェント: https://code.claude.com/docs/en/sub-agents
- ヘッドレスモード: https://code.claude.com/docs/en/headless
- エージェントチーム: https://code.claude.com/docs/en/agent-teams
- プラグイン: https://code.claude.com/docs/en/plugins
- プラグイン発見: https://code.claude.com/docs/en/discover-plugins
- スケジュールタスク: https://code.claude.com/docs/en/scheduled-tasks
- クラウドスケジュールタスク: https://code.claude.com/docs/en/web-scheduled-tasks
