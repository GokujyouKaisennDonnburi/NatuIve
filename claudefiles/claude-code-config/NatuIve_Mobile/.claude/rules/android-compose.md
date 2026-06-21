# Android UI ルール

globs: ["androidApp/**/*.kt"]

## 規約
- UI は Jetpack Compose で実装する
- ViewModel は Android の `androidx.lifecycle.ViewModel` を使用
- ViewModel から `sharedLogic` のユースケース/リポジトリを呼び出す
- Navigation は Jetpack Navigation Compose
- テーマは Material 3
