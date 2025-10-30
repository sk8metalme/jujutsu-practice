# 06. 実運用ワークフロー：実際の開発で使いこなす

このセクションでは、実際の開発現場でjujutsuをどう使うかを学びます。個人開発からチーム開発まで、実践的なシナリオを網羅します。

## 学習目標

- 個人開発での効率的なワークフローを身につける
- チーム開発でのjujutsu活用法を学ぶ
- GitHubとの連携方法をマスターする
- よくあるトラブルへの対処法を知る

## 前提知識

このセクションを始める前に、以下を学習済みであることを前提とします：

- [ ] 01-basics.md - 基本操作
- [ ] 02-branches.md - ブランチ操作
- [ ] 03-merging.md - マージとコンフリクト解決
- [ ] 04-history-editing.md - 履歴編集
- [ ] 05-advanced.md - 中級機能

---

## このドキュメントで使用する設定

以下のエイリアスが設定されていることを前提とします（`~/.config/jj/config.toml`）：

```toml
[aliases]
# 基本操作
c = ["commit"]
n = ["new"]
st = ["status"]
d = ["diff"]

# ブックマーク
bcp = ["bookmark", "create", "-r", "@-"]
bl = ["bookmark", "list"]

# ログ
ll = ["log", "--limit", "10"]
lb = ["log", "-r", "bookmarks()"]

# Git連携
gp = ["git", "push"]
gpb = ["git", "push", "--branch"]
gpbn = ["git", "push", "--branch", "--allow-new"]
gf = ["git", "fetch"]

# 履歴編集
sq = ["squash"]
desc = ["describe"]
rb = ["rebase"]

# 操作履歴
opl = ["op", "log"]
opu = ["op", "undo"]
```

設定がまだの場合は、[00-setup.md](00-setup.md)を参照してください。

---

## 1. 個人開発ワークフロー

### 1.1 基本的な開発サイクル

最もシンプルな開発の流れです。

#### シナリオ：新機能を開発してプッシュ

```bash
# 1. mainから新しい作業を開始
jj n main

# 2. 機能を実装
vim src/feature.js
vim tests/feature.test.js

# 3. コミット
jj c -m "新機能Xを実装"

# 4. ブックマーク作成
jj bcp feature/new-feature

# 5. プッシュ
jj gpbn feature/new-feature
```

**💡 ポイント：**
- `jj bcp` で親コミットに確実にブックマークを付ける
- `jj gpbn` で新規ブランチをプッシュ（`--allow-new`不要）

---

### 1.2 試行錯誤しながらの開発

何度も修正を重ねて、最後に1つのコミットにまとめる方法です。

#### シナリオ：実装→テスト→修正を繰り返す

```bash
# 1. 作業開始
jj n main
jj desc -m "認証機能を実装"

# 2. 最初の実装
vim src/auth.js
jj n

# 3. テストしてバグ発見、修正
vim src/auth.js
jj sq  # 親コミットに統合

# 4. さらに改善
vim src/auth.js
jj sq  # また統合

# 5. テスト追加
vim tests/auth.test.js
jj sq  # 統合

# 6. 最終的に1つのクリーンなコミット
jj ll
```

**出力例：**
```
@  (empty)
◉  認証機能を実装  ← すべての変更がまとまっている
◉  main
```

**💡 ポイント：**
- `jj describe -m` で先に説明を付ける
- `jj new` → 編集 → `jj squash` を繰り返す
- PRに出す前に履歴がクリーン

---

### 1.3 複数の作業を並行

2つの機能を同時に進める場合です。

#### シナリオ：feature-aとfeature-bを並行開発

```bash
# 1. feature-a開始
jj n main
vim src/feature-a.js
jj c -m "機能Aを実装"
jj bcp feature/a

# 2. feature-b開始
jj n main
vim src/feature-b.js
jj c -m "機能Bを実装"
jj bcp feature/b

# 3. feature-aに戻って追加作業
jj edit feature/a
vim src/feature-a.js
jj desc -m "機能Aを実装（改善版）"

# 4. feature-bに戻る
jj edit feature/b
vim src/feature-b.js

# 5. ブックマーク一覧確認
jj bl
```

**💡 ポイント：**
- `jj edit <bookmark>` で作業を切り替え
- それぞれのブランチで独立して作業できる
- lazyjjで視覚的に確認すると便利

---

## 2. チーム開発ワークフロー

### 2.1 機能開発：feature branch → PR → merge

GitHub Flowを使った標準的な開発フローです。

#### ステップ1：ブランチ作成と開発

```bash
# 1. 最新のmainを取得
jj gf

# 2. mainから新機能の開発開始
jj n main
vim src/login.js
vim tests/login.test.js

# 3. コミット
jj c -m "ログイン機能を実装

- フォームバリデーション
- エラーハンドリング
- テストカバレッジ: 95%"

# 4. ブックマーク作成
jj bcp feature/login

# 5. プッシュ
jj gpbn feature/login
```

#### ステップ2：PR作成

```bash
# GitHub CLIでPR作成
gh pr create --base main --head feature/login \
  --title "ログイン機能を実装" \
  --body "## 変更内容
- ログインフォームの実装
- バリデーション機能
- エラーハンドリング

## テスト
- ユニットテスト追加
- カバレッジ: 95%

Fixes #123"
```

**💡 ポイント：**
- コミットメッセージに詳細を書く（PR descriptionにコピペできる）
- Issue番号を含める

---

### 2.2 コードレビュー対応

レビュー指摘を受けて修正する場合の対応方法です。

#### シナリオ：レビューで指摘を受けた

**レビューコメント：** 「エラーメッセージが英語のままです。日本語化してください」

```bash
# 1. feature/loginブランチに移動（すでにいる場合は不要）
jj edit feature/login

# 2. 修正
vim src/login.js

# 3. 変更は自動的に記録される
jj st

# 4. コミットメッセージも更新
jj desc -m "ログイン機能を実装

- フォームバリデーション
- エラーハンドリング（日本語化対応）
- テストカバレッジ: 95%"

# 5. プッシュ（force pushではない！）
jj gpb feature/login
```

**⚠️ Gitとの違い：**
- Gitでは `git push --force` が必要
- jujutsuでは普通にプッシュするだけ
- 履歴がクリーンに保たれる

#### なぜjujutsuでは`--force`が不要なのか？

**理由1：履歴編集が前提の設計**

jujutsuは「履歴編集は普通のこと」という哲学で設計されています。そのため：

```bash
# Gitの場合（面倒）
git commit --amend -m "修正版"
git push --force-with-lease  # ← 明示的にforceが必要

# jujutsuの場合（シンプル）
jj describe -m "修正版"
jj git push --branch feature/xxx  # ← 普通にプッシュするだけ
```

**理由2：ブックマーク（ブランチ）の仕組み**

jujutsuでは：
1. ブックマークは「特定のコミットを指すポインタ」
2. `jj git push --branch`は「このコミットに更新して」という意味
3. リモートブランチを上書きする動作がデフォルト

**理由3：安全性の担保**

jujutsuには `jj op undo` があるため：
- プッシュしても、ローカルでは操作履歴が残っている
- 間違えてもいつでも戻せる
- だから大胆にプッシュできる

#### 内部的には何が起きているのか？

```bash
# jujutsuの場合
jj git push --branch feature/login
# ↓ 内部的には類似の安全性チェックを実行
# git push --force-with-lease に近いが、完全に同等ではない

# つまり、類似した安全性チェックを実行している
```

#### ⚠️ 注意点：チーム作業の場合

**個人のfeatureブランチなら問題なし：**
```bash
# 自分だけが使っているブランチ → 安全
jj git push --branch feature/my-work
```

**共有ブランチは慎重に：**
```bash
# 他の人も使っているブランチ → 注意
jj git push --branch develop  # ← 他の人の作業を上書きする可能性
```

#### 💡 ベストプラクティス

1. **個人のfeatureブランチ**
   - 自由に履歴編集してOK
   - 普通にプッシュするだけ
   
2. **PRマージ前**
   - レビュー指摘の修正は直接コミットを編集
   - 新しい修正コミットを積まない
   - 履歴がクリーンに保たれる

3. **PRマージ後**
   - mainにマージされたら、そのブランチは編集しない
   - 新しいブランチで作業開始

**💡 ポイント：**
- 過去のコミットを直接編集できる
- 新しい修正コミットを作る必要がない
- PRの履歴がクリーン

---

### 2.3 mainの更新を取り込む

開発中にmainが更新された場合の対応です。

#### シナリオ：mainに新しいコミットが入った

```bash
# 1. 最新のmainを取得
jj gf

# 2. 現在のブランチをmainにリベース
jj rb -s feature/login -d main

# 3. コンフリクトが発生した場合
jj st  # コンフリクトを確認
vim src/login.js  # 手動で解決
jj st  # 解決されたか確認

# 4. プッシュ
jj gpb feature/login
```

**期待される出力（リベース成功時）：**
```
Rebased 1 commits
```

**期待される出力（コンフリクト発生時）：**
```
Working copy changes:
C src/login.js
Warning: There are unresolved conflicts at these paths:
src/login.js    2-sided conflict
```

**💡 ポイント：**
- `jj rebase -s <source> -d <destination>`
- コンフリクトは通常通り解決（03-mergingを参照）

---

## 3. よくあるシナリオと対処法

### 3.1 バグ修正（hotfix）

緊急のバグ修正が必要な場合です。

```bash
# 1. 最新のmainから開始
jj gf
jj n main

# 2. バグ修正
vim src/bug.js
jj c -m "修正: ログイン時のnullエラー

原因: ユーザー情報がnullの場合の処理不足
対応: nullチェックを追加

Fixes #456"

# 3. ブックマーク作成
jj bcp hotfix/login-null-error

# 4. すぐにプッシュ
jj gpbn hotfix/login-null-error

# 5. PR作成（緊急なので簡潔に）
gh pr create --base main --head hotfix/login-null-error \
  --title "🚨 修正: ログイン時のnullエラー" \
  --body "Fixes #456"
```

**💡 ポイント：**
- 素早く対応できる
- コミットメッセージに原因と対応を明記
- 🚨 絵文字で緊急性を示す（任意）

---

### 3.2 間違ったブランチで作業してしまった

mainで作業してしまった場合の対処法です。

#### シナリオ：mainで直接コミットしてしまった

```bash
# 現在の状態
jj ll
# @  (empty)
# ◉  間違って追加した機能  ← これをブランチに移動したい
# ◉  main
```

**解決方法1：ブックマークを作成**

```bash
# 1. 間違ったコミットにブックマークを付ける
jj edit @-
jj bookmark create feature/my-feature

# 2. mainを元の位置に戻す
jj bookmark set main -r @-

# 完了！
jj ll
# @  間違って追加した機能  ← feature/my-featureになった
# ◉  main  ← 元の位置
```

**解決方法2：リベース**

```bash
# 間違ったコミットを別のブランチに移動
jj bcp feature/my-feature
jj rb -s feature/my-feature -d main~1

# mainブックマークを前のコミットに移動
jj bookmark set main -r main~1
```

**💡 ポイント：**
- `jj op undo` でやり直すこともできる
- 慌てずに、operation logで元に戻せる

---

### 3.3 コミットメッセージを後から修正

PRを出す前に、コミットメッセージを整える場合です。

```bash
# 現在のコミット
jj ll
# @  (empty)
# ◉  add login feature  ← 雑なメッセージ

# 修正
jj desc @- -m "ログイン機能を実装

- フォームバリデーション
- パスワード強度チェック
- リメンバーミー機能

## テスト
- ユニットテスト追加
- E2Eテスト追加"

# 確認
jj ll
```

**💡 ポイント：**
- PRを出す前に整える習慣をつける
- マークダウン形式で構造化
- テスト内容も含める

---

### 3.4 複数のコミットを1つにまとめる

細かいコミットが多すぎる場合です。

#### シナリオ：5個のコミットを1つに

```bash
# 現在の状態
jj ll
# @  (empty)
# ◉  typo修正
# ◉  テスト追加
# ◉  バグ修正
# ◉  機能追加
# ◉  ファイル作成
# ◉  main
```

**方法1：squashを繰り返す**

```bash
# 最新のコミットから順に統合
jj edit @-
jj sq  # typo修正 → テスト追加に統合

jj edit @-
jj sq  # テスト追加 → バグ修正に統合

jj edit @-
jj sq  # バグ修正 → 機能追加に統合

jj edit @-
jj sq  # 機能追加 → ファイル作成に統合

# 結果
jj ll
# @  (empty)
# ◉  ファイル作成  ← すべてまとまった
# ◉  main
```

**方法2：範囲を指定してsquash（高度）**

```bash
jj sq --from @---- --to @-
```

**💡 ポイント：**
- PRレビュー前に整理
- 1コミット = 1論理的な変更

---

## 4. GitHubとの連携

### 4.1 リモートリポジトリのクローン

```bash
# HTTPSでクローン
jj git clone https://github.com/username/repo.git

# SSHでクローン
jj git clone git@github.com:username/repo.git

# ディレクトリ名を指定
jj git clone https://github.com/username/repo.git my-project
cd my-project
```

---

### 4.2 プッシュの基本

```bash
# 現在のブックマークをプッシュ
jj gp

# 特定のブックマークをプッシュ
jj gpb feature/my-feature

# 新規ブランチをプッシュ
jj gpbn feature/new-feature

# すべてのブックマークをプッシュ
jj git push --all
```

---

### 4.3 最新の変更を取得

```bash
# リモートの変更を取得（fetch）
jj gf

# mainブックマークが自動的に更新される
jj ll

# 特定のリモートから取得
jj git fetch origin
```

**💡 ポイント：**
- `jj git pull` はない（fetchのみ）
- 取得後、必要に応じて`jj rebase`

---

### 4.4 PR作成とマージ

#### PR作成

```bash
# GitHub CLIでPR作成
gh pr create --base main --head feature/my-feature \
  --title "機能Xを実装" \
  --body "詳細な説明"

# ブラウザで開く
gh pr view --web
```

#### マージ後の後処理

```bash
# 1. mainを更新
jj gf

# 2. マージされたブランチを削除
jj bookmark delete feature/my-feature

# 3. ローカルを整理
jj ll
```

---

## 5. ベストプラクティス

### 5.1 頻繁にコミット、プッシュ前に整理

```bash
# 開発中は頻繁にコミット
jj c -m "WIP: ログイン機能"
jj c -m "WIP: バリデーション追加"
jj c -m "WIP: テスト追加"

# PRを出す前に整理
jj edit feature/login
jj sq  # すべてをまとめる
jj desc -m "ログイン機能を実装"  # きれいなメッセージに

# プッシュ
jj gpbn feature/login
```

---

### 5.2 ブックマーク命名規則

チームで統一すると便利です：

```
feature/機能名     - 新機能
bugfix/バグ内容    - バグ修正
hotfix/緊急修正    - 緊急修正
refactor/対象     - リファクタリング
docs/対象         - ドキュメント更新
```

**例：**
```bash
jj bcp feature/user-authentication
jj bcp bugfix/login-null-error
jj bcp hotfix/security-vulnerability
jj bcp refactor/api-client
jj bcp docs/readme-update
```

---

### 5.3 コミットメッセージの書き方

良いコミットメッセージの例：

```
短い要約（50文字以内）

## 変更内容
- 何を変更したか
- なぜ変更したか
- どう変更したか

## テスト
- 追加したテスト
- 確認した動作

## 関連Issue
Fixes #123
Refs #456
```

---

### 5.4 いつsquashするか？

#### squashすべき時

- 「typo修正」「リント修正」などの細かいコミット
- 同じファイルへの複数の修正
- 試行錯誤のコミット

#### squashすべきでない時

- 独立した機能追加
- 異なるファイルへの変更
- レビューしやすい単位

---

### 5.5 operation logの活用

```bash
# 定期的に確認
jj opl --limit 10

# 大きな操作の前にマイルストーン
# （特別な操作は不要、自動的に記録される）

# 間違えたらすぐに
jj opu
```

---

## 6. トラブルシューティング

### 6.1 コンフリクトが起きた

```bash
# 状態確認
jj st

# コンフリクトファイルを確認
cat conflicted-file.txt

# 手動で解決
vim conflicted-file.txt

# 解決されたか確認
jj st  # Cマークが消えればOK
```

**💡 詳細：** [03-merging.md](03-merging.md)を参照

---

### 6.2 間違ったコミットをプッシュした

```bash
# 1. ローカルで修正
jj edit feature/my-feature
vim wrong-file.txt
jj desc -m "修正版"

# 2. プッシュ（上書き）
jj gpb feature/my-feature
```

**⚠️ 注意：**
- PRマージ前なら問題なし
- マージ済みの場合は、新しいPRで修正

---

### 6.3 履歴が複雑になった

```bash
# lazyjjで視覚的に確認
lazyjj

# 複雑な部分を特定
jj ll -r 'all()'

# 必要に応じて
# - abandon で不要なコミット削除
# - rebase で整理
# - squash でまとめる
```

---

### 6.4 操作を間違えた

```bash
# 直前の操作を取り消し
jj opu

# 操作履歴を確認
jj opl

# 特定の時点に戻る
jj op restore <operation-id>
```

**💡 安心：** すべての操作は記録されているので、いつでも戻せる！

---

## 7. チートシート

### よく使うコマンドの組み合わせ

#### 新機能開発の流れ

```bash
jj n main                    # mainから開始
vim src/feature.js           # 実装
jj c -m "機能を実装"          # コミット
jj bcp feature/name          # ブックマーク作成
jj gpbn feature/name         # プッシュ
gh pr create                 # PR作成
```

#### レビュー指摘の対応

```bash
jj edit feature/name         # ブランチに移動
vim src/feature.js           # 修正
jj st                        # 確認
jj gpb feature/name          # プッシュ
```

#### mainの更新を取り込む

```bash
jj gf                        # fetch
jj rb -s feature/name -d main  # rebase
jj gpb feature/name          # プッシュ
```

#### コミットを整理

```bash
jj edit feature/name         # ブランチに移動
jj sq                        # 子コミットを統合
jj desc -m "新しいメッセージ" # メッセージ更新
jj gpb feature/name          # プッシュ
```

---

### シナリオ別早見表

| やりたいこと | コマンド |
|------------|---------|
| 新しい作業を始める | `jj n main` |
| コミットしてブックマーク | `jj c -m "..." && jj bcp name` |
| ブランチ切り替え | `jj edit <bookmark>` |
| 最新取得 | `jj gf` |
| プッシュ | `jj gpbn <bookmark>` |
| mainを取り込む | `jj rb -s <branch> -d main` |
| コミットをまとめる | `jj sq` |
| メッセージ修正 | `jj desc @- -m "..."` |
| やり直し | `jj opu` |
| 履歴確認 | `jj ll` または `lazyjj` |

---

## チェックリスト

このセクションを完了したら、以下を確認してください：

- [ ] 個人開発のワークフローを理解した
- [ ] チーム開発でのjujutsu活用法を学んだ
- [ ] GitHubとの連携方法をマスターした
- [ ] PR作成からマージまでの流れを理解した
- [ ] よくあるトラブルへの対処法を知った
- [ ] 実際のプロジェクトで試してみた

---

## 次のステップ

実運用ワークフローをマスターしました！

### さらに学ぶには

- [jujutsu公式ドキュメント](https://github.com/martinvonz/jj)
- [公式チュートリアル](https://github.com/martinvonz/jj/blob/main/docs/tutorial.md)
- [lazyjj](https://github.com/Cretezy/lazyjj) - TUIツール

### 実践

1. 実際のプロジェクトで使ってみる
2. チームメンバーに共有する
3. 自分なりのワークフローを確立する

おめでとうございます！これでjujutsuを実務で使いこなせるようになりました 🎉

