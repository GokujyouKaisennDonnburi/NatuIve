# iOS UI ルール

globs: ["iosApp/**/*.swift"]

## 規約
- UI は SwiftUI で実装する
- ViewModel は `@Observable` マクロ（iOS 17+）または `ObservableObject`
- ViewModel から KMP の sharedLogic を呼び出す
- Swift Package Manager で iOS 固有の依存を管理
- KMP framework は Gradle で生成し Xcode にリンク
