# Claude Code 設定ガイド — NatuPortal

## 概要

Opus 4.8 をオーケストレーター、Sonnet 4.6 をサブエージェントとする構成の設定ファイル一式。

---

## ファイル配置マップ

### ユーザーレベル（全プロジェクト共通）

```
~/.claude/
├── CLAUDE.md                    # 個人の好み（言語・モデル戦略）
└── agents/
    ├── implementer.md           # 実装サブエージェント
    └── reviewer.md              # レビューサブエージェント
```

### NatuIve_API（Go + Gin）

```
NatuIve_API/
├── CLAUDE.md                    # プロジェクト概要・ビルドコマンド・アーキテクチャルール
└── .claude/
    ├── settings.json            # パーミッション・環境変数
    ├── rules/
    │   ├── geofuzzing.md        # 座標情報保護（internal/**/*.go で発火）
    │   ├── docker.md            # Docker 規約（Dockerfile*, docker-compose* で発火）
    │   └── go-tests.md          # テスト規約（**/*_test.go で発火）
    └── skills/
        └── openapi-gen/
            └── SKILL.md         # OpenAPI 型生成ワークフロー
```

### NatuIve_Mobile（KMP）

```
NatuIve_Mobile/
├── CLAUDE.md                    # プロジェクト概要・ビルドコマンド・アーキテクチャルール
└── .claude/
    ├── settings.json            # パーミッション・環境変数
    ├── rules/
    │   ├── kmp-boundary.md      # KMP 共有層の境界（sharedLogic/**/*.kt で発火）
    │   ├── android-compose.md   # Android UI 規約（androidApp/**/*.kt で発火）
    │   ├── ios-swiftui.md       # iOS UI 規約（iosApp/**/*.swift で発火）
    │   └── gradle.md            # Gradle 規約（**/*.gradle.kts, gradle/** で発火）
    └── skills/
        └── openapi-gen/
            └── SKILL.md         # OpenAPI 型生成ワークフロー
```

### nachuibe-web（Next.js）

```
nachuibe-web/
├── CLAUDE.md                    # プロジェクト概要・ビルドコマンド・アーキテクチャルール
└── .claude/
    ├── settings.json            # パーミッション・環境変数
    ├── rules/
    │   ├── nextjs-app-router.md # App Router 規約（app/**/*.tsx で発火）
    │   └── bun-usage.md         # Bun 使用ルール（package.json 等で発火）
    └── skills/
        └── openapi-gen/
            └── SKILL.md         # OpenAPI 型生成ワークフロー
```

---

## セットアップ手順

### 1. ユーザーレベル設定を配置

```bash
# ~/.claude/ に CLAUDE.md と agents/ をコピー
cp user-level/.claude/CLAUDE.md ~/.claude/CLAUDE.md
cp -r user-level/.claude/agents ~/.claude/agents
```

### 2. 各リポジトリに設定を配置

```bash
# NatuIve_API
cp NatuIve_API/CLAUDE.md /path/to/NatuIve_API/CLAUDE.md
cp -r NatuIve_API/.claude /path/to/NatuIve_API/.claude
cp shared/settings.json /path/to/NatuIve_API/.claude/settings.json

# NatuIve_Mobile
cp NatuIve_Mobile/CLAUDE.md /path/to/NatuIve_Mobile/CLAUDE.md
cp -r NatuIve_Mobile/.claude /path/to/NatuIve_Mobile/.claude
cp shared/settings.json /path/to/NatuIve_Mobile/.claude/settings.json

# nachuibe-web
cp nachuibe-web/CLAUDE.md /path/to/nachuibe-web/CLAUDE.md
cp -r nachuibe-web/.claude /path/to/nachuibe-web/.claude
cp shared/settings.json /path/to/nachuibe-web/.claude/settings.json
```

### 3. Claude Code の起動

```bash
# Opus 4.8 をオーケストレーターとして起動
# settings.json の env で CLAUDE_CODE_SUBAGENT_MODEL=claude-sonnet-4-6 が設定済み
claude --model claude-opus-4-8
```

または、シェルプロファイルに追加:

```bash
# ~/.bashrc or ~/.zshrc
export CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-6"
alias cc="claude --model claude-opus-4-8"
```

---

## 設計判断の根拠

### CLAUDE.md vs rules/ vs skills/ の使い分け

| 置き場所 | 用途 | ロード条件 |
|---|---|---|
| `CLAUDE.md` | 毎セッション必要な情報（ビルドコマンド、プロジェクト構造、全体ルール） | セッション開始時に常にロード |
| `.claude/rules/` | 特定ファイルを触るときだけ必要なルール | `globs` にマッチするファイルを操作した時のみ |
| `.claude/skills/` | オンデマンドのワークフロー | 関連する作業を依頼された時のみ |
| `.claude/agents/` | サブエージェント定義 | Opus が委譲を判断した時のみ |

### なぜ rules/ にルールを分離するか

CLAUDE.md に全ルールを詰め込むと 200 行を超えやすく、Claude が重要なルールを見落とす原因になる。
path-scoped rules を使えば、Go のテストファイルを触っている時だけテスト規約がロードされ、
コンテキストウィンドウを節約しつつ遵守率が上がる。

### なぜ agents/ をユーザーレベルに置くか

implementer と reviewer は NatuPortal の全リポジトリで共通して使える汎用エージェント。
プロジェクト固有のサブエージェントが必要になった場合は、各リポジトリの `.claude/agents/` に追加する。

### geofuzzing ルールを rules/ に分離した理由

座標保護は NatuIve_API の最重要セキュリティ制約だが、CLAUDE.md の中に埋もれると
見落とされるリスクがある。path-scoped rule として `internal/**/*.go` に紐づけることで、
Go のコードを書くたびに必ずロードされ、強制力が高まる。

---

## Git 管理

### コミットすべきファイル（チーム共有）

- `CLAUDE.md`
- `.claude/rules/*.md`
- `.claude/skills/*/SKILL.md`
- `.claude/agents/*.md`（プロジェクト固有のもの）
- `.claude/settings.json`

### コミットしないファイル（個人設定）

- `CLAUDE.local.md`（個人用の上書き）
- `~/.claude/` 配下のすべて

`.gitignore` に追加:

```
CLAUDE.local.md
```

---

## 運用のポイント

### CLAUDE.md の定期メンテナンス

- Claude が同じミスを2回したら、ルールを追加する
- Claude が既にできていることのルールは削除する
- モデルのアップデート後に不要になったルールを見直す

### サブエージェントの使い方

Opus セッションで以下のように指示する:

```
implementer を使って、イベント一覧 API のハンドラを実装して。
仕様: GET /api/v1/events でページネーション付きのイベント一覧を返す。
対象ファイル: internal/handler/event.go, internal/service/event.go
```

実装後のレビュー:

```
reviewer を使って、さっき実装した internal/handler/event.go と
internal/service/event.go をレビューして。
要件: GET /api/v1/events のページネーション実装。
```
