# NatuIve_Mobile

なちゅいべのモバイルアプリ。KMP（Kotlin Multiplatform）でロジックを共有し、
UI は Android（Jetpack Compose）と iOS（SwiftUI）で個別実装する。

## Tech Stack
- KMP（Kotlin Multiplatform）
- Android: Jetpack Compose
- iOS: SwiftUI
- ネットワーク: Ktor Client
- Project ID: `org.natuportal.natuive`

## Module Structure
```
sharedLogic/          # KMP 共有ロジック
  network/            # Ktor Client, API 通信
  model/              # データモデル（OpenAPI 生成）
  data/               # リポジトリ実装
  domain/             # ユースケース
  auth/               # Supabase Auth 連携
  di/                 # DI 設定
androidApp/           # Jetpack Compose UI
iosApp/               # SwiftUI UI
```

## Commands
```bash
./gradlew :androidApp:assembleDebug     # Android ビルド
./gradlew :sharedLogic:build            # 共有ロジックビルド
./gradlew :sharedLogic:allTests         # 共有ロジックテスト
```

## Architecture Rules

### CRITICAL: KMP の境界
- **CMP（Compose Multiplatform）は使わない** — UI は各プラットフォームでネイティブ実装
- KMP 共有層は「データの取得・整形・検証まで」に限定
- **ViewModel は共有しない** — 各プラットフォームの ViewModel で sharedLogic を呼び出す
- `sharedUI` フォルダは削除済み。再生成しないこと

### 認証
- Supabase Auth を使用（DB/Storage は使わない）
- Google OAuth は「ウェブ アプリケーション」クライアントタイプ
- 将来的にネイティブ OS ログイン UX（iOS/Android 別クライアント ID）が必要になる可能性

### 型共有
- OpenAPI 定義から Kotlin の型を自動生成する
- 手書きの API モデルクラスは作らない

### バリデーション
- クライアント側バリデーションは UX 補助のみ
- 信頼の境界は API サーバー側

## Conventions
- 依存バージョン管理: `gradle/libs.versions.toml`（Version Catalog）
- マルチモジュール化（`features/` 分割）は MVP 段階では行わない
- IDE: Android Studio + KMP プラグイン（iOS は Xcode も必要）
