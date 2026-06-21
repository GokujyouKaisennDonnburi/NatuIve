# Claude Code 設定ガイド — NatuPortal

## 概要

Opus 4.8 をオーケストレーター、Sonnet 4.6 をサブエージェントとする構成の設定ファイル一式。
対象リポジトリ: `NatuIve_API` / `NatuIve_Mobile` / `NatuIve_Web_Frontend`。

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
├── CLAUDE.md                    # プロジェクト概要・ビルドコマンド・全体ルール（rules を @import）
└── .claude/
    ├── settings.json            # パーミッション・環境変数
    ├── rules/
    │   ├── geofuzzing.md        # 座標情報保護
    │   ├── docker.md            # Docker 規約
    │   └── go-tests.md          # テスト規約
    └── skills/
        └── openapi-gen/
            └── SKILL.md         # OpenAPI 型生成ワークフロー（api/openapi.yaml から全言語生成）
```

### NatuIve_Mobile（KMP）

```
NatuIve_Mobile/
├── CLAUDE.md                    # プロジェクト概要・ビルドコマンド・全体ルール（rules を @import）
└── .claude/
    ├── settings.json            # パーミッション・環境変数
    └── rules/
        ├── kmp-boundary.md      # KMP 共有層の境界
        ├── android-compose.md   # Android UI 規約
        ├── ios-swiftui.md       # iOS UI 規約
        └── gradle.md            # Gradle 規約
```

### NatuIve_Web_Frontend（Next.js）

```
NatuIve_Web_Frontend/
├── CLAUDE.md                    # プロジェクト概要・ビルドコマンド・全体ルール（rules を @import）
└── .claude/
    ├── settings.json            # パーミッション・環境変数
    └── rules/
        ├── nextjs-app-router.md # App Router 規約
        └── bun-usage.md         # Bun 使用ルール
```

> **OpenAPI 型生成 skill は `NatuIve_API` にのみ配置する。**
> 型生成は `api/openapi.yaml` を起点に Go / TypeScript / Kotlin の3言語をまとめて生成するワークフローのため、
> 定義ファイルが存在する API リポジトリに置く。Mobile / Web は生成済みの型を取り込む側。

---

## セットアップ手順

### 1. ユーザーレベル設定を配置

```bash
cp claude-code-config/user-level/.claude/CLAUDE.md ~/.claude/CLAUDE.md
cp -r claude-code-config/user-level/.claude/agents ~/.claude/agents
```

### 2. 各リポジトリに設定を配置

各リポジトリ用の `.claude/`（settings.json・rules・skills を含む）と `CLAUDE.md` を、
そのままターゲットリポジトリのルートへコピーする。

```bash
# NatuIve_API
cp claude-code-config/NatuIve_API/CLAUDE.md   /path/to/NatuIve_API/CLAUDE.md
cp -r claude-code-config/NatuIve_API/.claude  /path/to/NatuIve_API/.claude

# NatuIve_Mobile
cp claude-code-config/NatuIve_Mobile/CLAUDE.md   /path/to/NatuIve_Mobile/CLAUDE.md
cp -r claude-code-config/NatuIve_Mobile/.claude  /path/to/NatuIve_Mobile/.claude

# NatuIve_Web_Frontend
cp claude-code-config/NatuIve_Web_Frontend/CLAUDE.md   /path/to/NatuIve_Web_Frontend/CLAUDE.md
cp -r claude-code-config/NatuIve_Web_Frontend/.claude  /path/to/NatuIve_Web_Frontend/.claude
```

`settings.json` は各リポジトリの `.claude/` 配下に既に含まれている（リポジトリごとに
許可コマンドが異なるため共通化していない）。

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

### CLAUDE.md / rules / skills / agents の使い分け

| 置き場所 | 用途 | ロード条件 |
|---|---|---|
| `CLAUDE.md` | 毎セッション必要な情報（ビルドコマンド、構造、全体ルール） | セッション開始時に常にロード |
| `.claude/rules/` | 関心事ごとに分割した詳細ルール | CLAUDE.md の `@import` 経由で常にロード |
| `.claude/skills/` | オンデマンドのワークフロー | description に関連する作業を依頼された時のみ |
| `.claude/agents/` | サブエージェント定義 | Opus が委譲を判断した時のみ |

### なぜ rules/ にルールを分離するか

CLAUDE.md に全ルールを詰め込むと肥大化し、重要なルールを見落としやすい。
関心事ごとにファイルを分け、CLAUDE.md から `@import`（`@.claude/rules/xxx.md`）で取り込むことで、
**可読性と保守性**を上げている。

> **重要（仕様の前提）:** Claude Code には「glob にマッチするファイルを触った時だけ rule を自動ロードする」
> という機能は無い（それは Cursor 等の別ツールの仕組み）。本構成の rules は `@import` により
> **常時ロード**される。各 rules ファイル冒頭の「適用対象」はあくまで人間向けの適用範囲メモ。
> ルールが短いためコンテキストコストは小さい。真にパスローカルにしたい場合は、
> 対象ディレクトリ直下にネストした `CLAUDE.md` を置く方法もある（例: `internal/CLAUDE.md`）。

### なぜ agents/ をユーザーレベルに置くか

implementer と reviewer は NatuPortal の全リポジトリで共通して使える汎用エージェント。
プロジェクト固有のサブエージェントが必要になった場合は、各リポジトリの `.claude/agents/` に追加する。

### geofuzzing を最優先で扱う理由

座標保護は NatuIve_API の最重要セキュリティ制約。CLAUDE.md の CRITICAL セクションに要約を置きつつ、
`rules/geofuzzing.md` に詳細チェックリストを置いて `@import` で常時ロードし、見落としを防ぐ。

### settings.json をリポジトリごとに分ける理由

許可する Bash コマンドがリポジトリで異なる（Go の `go test`、KMP の `./gradlew`、Web の `bun run` など）。
共通化せず、各リポジトリの安全なビルド/テストコマンドだけを `allow` し、承認プロンプトの摩擦を減らす。
`deny` は `rm -rf` / force push などを**プレフィックスパターン**でブロックする安全ネット
（完全一致だと容易にすり抜けるため）。

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
- `.claude/settings.local.json`（個人のパーミッション上書き）
- `~/.claude/` 配下のすべて

`.gitignore` に追加:

```
CLAUDE.local.md
.claude/settings.local.json
```

---

## 運用のポイント

### CLAUDE.md / rules の定期メンテナンス

- Claude が同じミスを2回したら、該当 rules にルールを追加する
- Claude が既にできていることのルールは削除する
- モデルのアップデート後に不要になったルールを見直す

### サブエージェントの使い方

Opus セッションで以下のように指示する（サブエージェントは文脈ゼロで始まるため、
対象ファイルと要件を必ず明示する）:

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
