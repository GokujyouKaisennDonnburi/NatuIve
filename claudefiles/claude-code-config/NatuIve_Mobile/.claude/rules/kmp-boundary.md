# KMP 共有層の境界ルール

globs: ["sharedLogic/**/*.kt"]

## 許可する内容
- データモデル（data class）
- リポジトリインターフェースと実装
- ユースケース / ドメインロジック
- Ktor Client によるネットワーク通信
- バリデーション（UX 補助目的のみ）
- DI 設定

## 禁止する内容
- sharedLogic 内への ViewModel クラスの配置（ViewModel は各プラットフォームで実装する）
- UI コンポーネント（Compose Multiplatform は使わない）
- プラットフォーム固有の UI ロジック
- `sharedUI` モジュールの作成・復元
