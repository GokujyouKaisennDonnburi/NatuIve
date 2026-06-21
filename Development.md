# 開発者向け資料

## Git関係のルール

### コミット規則

- `feat:` 機能追加
- `update:` 機能変更
- `fix:` バグ修正
- `docs:` ドキュメント修正
- `refactor:` リファクタリング
- `test:` テスト

### Issue

コミットメッセージの最後に、対象の**Issue**番号を記載してください。
記載方法は`#番号`の形式です。

#### 例

```
docs: githubのドキュメント作成 #3
```


### ブランチ保護

有効になっている各ルールの具体的な状況は以下の通りです。

#### 簡単にいうと
- mainの削除禁止
- mainの直接変更禁止
- Approveと会話の解決必須

#### 該当ルール
- Restrict deletions（削除の制限）
対象ブランチの削除を禁止します。誤操作や悪意によるブランチの消失を防ぎます。

- Block force pushes（強制プッシュのブロック）
対象ブランチへのgit push --force等による履歴の書き換えや上書きを禁止します。

- Require a pull request before merging（マージ前のPR必須化）
対象ブランチへの直接プッシュを禁止し、必ずPull Request（PR）を経由させます。

    - Require approval of the most recent reviewable push
    最新のPushに対する他者の承認（Approve）を必須にします。承認後に修正コミットを追加した場合、以前の承認は無効化され、再度レビューと承認が必要です。

    - Require conversation resolution before merging
    PR内のすべてのコメント（レビューの指摘事項など）が「解決済み（Resolved）」になるまでマージをブロックします。
