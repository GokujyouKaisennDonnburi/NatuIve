# Gradle ルール

> 適用対象: `**/*.gradle.kts`, `gradle/**`。
> このファイルは CLAUDE.md から `@import` で常時ロードされる。

## Version Catalog
- すべての依存バージョンは `gradle/libs.versions.toml` で管理する
- `build.gradle.kts` にバージョン番号を直接書かない
- 新しい依存を追加する場合は Version Catalog に追加してから参照する

## 注意
- `settings.gradle.kts` に `include(":sharedUI")` を追加しないこと（削除済み）
- `sharedUI` への依存を `androidApp/build.gradle.kts` に追加しないこと
