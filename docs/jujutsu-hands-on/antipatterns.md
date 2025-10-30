# jujutsuアンチパターン集：やってはいけないこと

このドキュメントでは、jujutsuを使う上で**避けるべき行動**と**よくある間違い**をまとめます。実運用で問題を起こさないために、これらのパターンを理解しておきましょう。

## 目次

1. [チーム開発でのアンチパターン](#1-チーム開発でのアンチパターン)
2. [履歴編集の落とし穴](#2-履歴編集の落とし穴)
3. [Git連携での間違い](#3-git連携での間違い)
4. [operation logの誤解](#4-operation-logの誤解)
5. [ブックマーク管理の問題](#5-ブックマーク管理の問題)
6. [コンフリクト解決の失敗](#6-コンフリクト解決の失敗)
7. [パフォーマンスの問題](#7-パフォーマンスの問題)

---

## 1. チーム開発でのアンチパターン

### ❌ アンチパターン1.1：共有ブランチの履歴を勝手に編集

**問題のあるコード：**

```bash
# developブランチ（チーム全員が使っている）
jj edit develop
vim src/shared.js
jj describe -m "修正"
jj git push --branch develop  # ← 他の人の作業を破壊！
```

**何が問題か：**
- 他の開発者がpullできなくなる
- チームメンバーのローカル履歴と食い違う
- 混乱とトラブルの原因

**✅ 正しい方法：**

```bash
# 新しいブランチで作業
jj new develop
vim src/shared.js
jj commit -m "修正"
jj bookmark create feature/fix-shared
jj git push --branch feature/fix-shared

# PR経由でマージ
gh pr create --base develop --head feature/fix-shared
```

**💡 ルール：**
- **共有ブランチ（main, develop など）は直接編集しない**
- 必ずfeatureブランチ経由で変更
- PRレビューを通す

---

### ❌ アンチパターン1.2：PRマージ後にブランチの履歴を編集

**問題のあるコード：**

```bash
# PR がマージされた後
jj edit feature/old-feature
vim src/file.js
jj git push --branch feature/old-feature  # ← 意味がない！
```

**何が問題か：**
- すでにmainにマージされているので、ブランチを更新しても意味がない
- 混乱を招く
- 無駄な作業

**✅ 正しい方法：**

```bash
# マージ後は新しいブランチで作業
jj new main
vim src/file.js
jj commit -m "追加修正"
jj bookmark create feature/additional-fix
jj git push --branch feature/additional-fix
```

**💡 ルール：**
- **マージ済みのブランチは編集しない**
- 新しい修正は新しいブランチで

---

### ❌ アンチパターン1.3：operation undoでチームの作業を巻き戻す

**問題のあるコード：**

```bash
# チームメンバーがプッシュした変更を取得
jj git fetch

# 気に入らないからと言って...
jj op undo  # ← fetchを取り消し！
```

**何が問題か：**
- ローカルとリモートの状態が不一致
- チームの作業を無視している
- 後で大きな問題になる

**✅ 正しい方法：**

```bash
# 変更が気に入らない場合
# 1. 問題を特定
jj log -r 'remote_branches()'

# 2. チームと相談
# 3. 必要なら revert コミットを作成
jj new main
jj restore --from @-- .  # 一つ前の状態に戻す
jj commit -m "Revert: 問題のある変更を取り消し"
```

**💡 ルール：**
- **operation undoは個人の作業のみに使う**
- リモートから取得した変更は巻き戻さない
- 問題があればrevertコミットで対応

---

## 2. 履歴編集の落とし穴

### ❌ アンチパターン2.1：過度なsquash

**問題のあるコード：**

```bash
# すべてのコミットを1つにまとめる
jj edit main
jj squash --from 'main..@'  # ← 数十個のコミットを全部まとめる
```

**何が問題か：**
- レビューが困難になる
- どこで何が変わったか追跡できない
- バグの原因を特定できない
- ロールバックが困難

**✅ 正しい方法：**

```bash
# 論理的な単位でまとめる

# ログイン機能の細かいコミット → 1つに
jj squash  # ログイン関連のみ

# API変更の細かいコミット → 1つに  
jj squash  # API関連のみ

# 結果：2つのコミット
# 1. ログイン機能実装
# 2. API変更
```

**💡 ルール：**
- **1コミット = 1つの論理的な変更**
- 関連する変更はまとめる
- 無関係な変更は分ける
- PRレビューしやすい単位を心がける

---

### ❌ アンチパターン2.2：意味のないコミットメッセージでsquash

**問題のあるコード：**

```bash
jj commit -m "fix"
jj commit -m "update"
jj commit -m "change"

# 全部squashして...
jj squash
jj describe -m "update"  # ← 何も伝わらない！
```

**何が問題か：**
- 何を変更したのか不明
- 将来見返した時に理解できない
- チームメンバーが困る

**✅ 正しい方法：**

```bash
jj commit -m "ログイン機能を実装"
jj commit -m "バリデーションを追加"
jj commit -m "エラーハンドリングを改善"

# squashして意味のあるメッセージに
jj squash
jj describe -m "ログイン機能を実装

- フォームバリデーション
- エラーハンドリング
- パスワード強度チェック

Fixes #123"
```

**💡 ルール：**
- **コミットメッセージは将来の自分への手紙**
- 何を、なぜ、どのように変更したかを書く
- Issue番号を含める

---

### ❌ アンチパターン2.3：change IDを直接編集しようとする

**問題のあるコード：**

```bash
# これは無意味
jj edit qpvuntsm  # change IDを指定
vim src/file.js
# change IDは変わらないが、混乱を招く
```

**何が問題か：**
- change IDは自動生成される識別子
- 直接編集するものではない
- ブックマークやrevsetを使うべき

**✅ 正しい方法：**

```bash
# ブックマークで管理
jj edit feature/login

# または revset で指定
jj edit @-
jj edit '@---'
jj edit 'description("ログイン")'
```

**💡 ルール：**
- **change IDは参照用**
- 編集にはブックマークやrevsetを使う

---

## 3. Git連携での間違い

### ❌ アンチパターン3.1：`jj git pull`を使おうとする

**問題のあるコード：**

```bash
jj git pull  # ← このコマンドは存在しない！
```

**何が問題か：**
- jujutsuには`pull`コマンドがない
- Gitの感覚で使うと失敗する

**✅ 正しい方法：**

```bash
# fetch + rebase を個別に実行
jj git fetch
jj rebase -s feature/mybranch -d main
```

**💡 ルール：**
- **`jj git fetch` のみ存在する**
- 必要に応じて手動で rebase

---

### ❌ アンチパターン3.2：リモートブランチを直接編集

**問題のあるコード：**

```bash
jj edit origin/main  # ← リモートブランチを編集！？
vim src/file.js
jj commit -m "修正"
```

**何が問題か：**
- リモートブランチは読み取り専用の参照
- ローカルとリモートの状態が不一致になる
- プッシュできない

**✅ 正しい方法：**

```bash
# ローカルブランチで作業
jj new main  # ローカルの main
vim src/file.js
jj commit -m "修正"
jj bookmark create feature/fix
jj git push --branch feature/fix
```

**💡 ルール：**
- **リモートブランチ（`origin/*`）は編集しない**
- ローカルブランチで作業

---

### ❌ アンチパターン3.3：複数のGitリモートを正しく理解していない

**問題のあるコード：**

```bash
# upstream と origin を混同
jj git fetch  # どのリモート？
jj rebase -d upstream/main  # ← 存在しないかも
```

**何が問題か：**
- デフォルトは`origin`のみ
- `upstream`は手動で追加が必要
- 混乱を招く

#### upstreamを追加する必要があるのはどんな時？

**シナリオ1：OSSにコントリビュートする場合**

```
元のリポジトリ（upstream）： https://github.com/original-author/awesome-project
あなたのフォーク（origin）： https://github.com/yourname/awesome-project
```

この場合の構成：

```bash
# 1. 自分のフォークをクローン
jj git clone https://github.com/yourname/awesome-project.git
cd awesome-project

# 2. リモートを確認（originのみ）
jj git remote list
# origin: https://github.com/yourname/awesome-project.git

# 3. upstreamを追加（元のリポジトリ）
jj git remote add upstream https://github.com/original-author/awesome-project.git

# 4. リモートを確認（originとupstream）
jj git remote list
# origin: https://github.com/yourname/awesome-project.git
# upstream: https://github.com/original-author/awesome-project.git
```

**なぜ2つ必要？**
- `origin`：自分の作業をプッシュする場所
- `upstream`：元のプロジェクトの最新変更を取得する場所

**典型的なワークフロー：**

```bash
# 元のプロジェクトの最新版を取得
jj git fetch --remote upstream

# mainを最新に更新
jj bookmark set main -r upstream/main

# 新機能を開発
jj new main
vim src/feature.js
jj commit -m "新機能を追加"
jj bookmark create feature/my-contribution

# 自分のフォークにプッシュ
jj git push --branch feature/my-contribution --remote origin

# 元のリポジトリにPR作成
gh pr create --repo original-author/awesome-project \
  --base main --head yourname:feature/my-contribution
```

**シナリオ2：チームで作業するが、個人フォークを使う場合**

```
チームのリポジトリ（upstream）： https://github.com/company/product
あなたのフォーク（origin）：     https://github.com/yourname/product
```

企業によっては、メインリポジトリへの直接pushを制限し、個人フォーク経由でのPRを義務付ける場合があります。

**シナリオ3：複数のリモートでテスト環境が分かれている場合**

```
本番環境（production）： git@github.com:company/app-production.git
開発環境（staging）：    git@github.com:company/app-staging.git
あなたのフォーク（origin）： git@github.com:yourname/app.git
```

この場合は、`upstream`だけでなく複数のリモートを追加することもあります。

**upstreamが不要な場合：**
- ✅ チームの共有リポジトリに直接pushする権限がある
- ✅ フォークせずに直接作業している
- ✅ 個人プロジェクト

**upstreamが必要な場合：**
- ⚠️ OSSにコントリビュートする
- ⚠️ 他の人のプロジェクトをフォークして作業
- ⚠️ 会社がフォークベースのワークフローを採用している

**✅ 正しい方法：**

```bash
# リモートを確認
jj git remote list

# 必要なら upstream を追加
jj git remote add upstream https://github.com/original/repo.git

# 明示的に指定
jj git fetch --remote upstream
jj rebase -d upstream/main
```

**💡 ルール：**
- **リモートを明示的に管理**
- `jj git remote list`で確認

---

## 4. operation logの誤解

### ❌ アンチパターン4.1：operation logが永遠に残ると思う

**問題のあるコード：**

```bash
# 3ヶ月前の操作に戻そうとする
jj op log  # ← 古い操作は削除されている！
jj op restore <3ヶ月前のID>  # ← エラー
```

**何が問題か：**
- operation logは定期的にガベージコレクションされる
- デフォルトでは90日程度で削除される
- 永続的なバックアップではない

**✅ 正しい方法：**

```bash
# 重要な状態はブックマークで保存
jj bookmark create backup/before-big-change

# または Git でバックアップ
jj git push --branch backup/state-2024-10-30
```

**💡 ルール：**
- **operation logは一時的なもの**
- 重要な状態はブックマークやGitで管理

---

### ❌ アンチパターン4.2：operation undoの連打

**問題のあるコード：**

```bash
jj commit -m "A"
jj op undo
jj commit -m "B"
jj op undo
jj commit -m "C"
jj op undo
jj commit -m "D"
jj op undo
# ← どこまで戻ったかわからない！
```

**何が問題か：**
- 現在の状態が不明
- operation logが複雑になる
- 混乱を招く

**✅ 正しい方法：**

```bash
# 現在の状態を確認してから戻る
jj op log --limit 10
jj log

# 特定の状態に戻る
jj op restore <operation-id>

# または、最初からやり直す
jj new main
```

**💡 ルール：**
- **やみくもにundoしない**
- `jj op log`で状態を確認
- 必要なら`op restore`で特定の状態に

---

## 5. ブックマーク管理の問題

### ❌ アンチパターン5.1：プッシュする作業にブックマークを付けない

**問題のあるコード：**

```bash
# PRを出す作業なのにブックマークなし
jj new main
vim src/feature.js
jj commit -m "新機能"
jj git push  # ← どこにプッシュ？エラー！

# 別の作業
jj new main
vim src/another.js
jj commit -m "別の機能"
# どれがどのPRかわからない！
```

**何が問題か：**
- プッシュする時にブックマークが必要
- PRを管理できない
- チームメンバーが混乱

**✅ 正しい方法：**

```bash
# PR用の作業にはブックマークを付ける
jj new main
vim src/feature.js
jj commit -m "新機能"
jj bookmark create -r @- feature/new-feature  # ← PR用
jj git push --branch feature/new-feature
gh pr create  # PR作成

# ローカルだけの試行錯誤にはブックマーク不要
jj new main
vim test.js  # ローカルでテスト
jj commit -m "テスト"
# プッシュしないのでブックマーク不要
```

**💡 ルール：**
- **1つのPR = 1つのブックマーク**
- プッシュする作業には必ずブックマーク
- ローカルだけの試行錯誤にはブックマーク不要
- 細かい作業ごとにブックマークを付けない

---

### ❌ アンチパターン5.2：ブックマークを付けすぎる

**問題のあるコード：**

```bash
# すべての小さな作業にブックマーク
jj new main
echo "step1" > file.txt
jj commit -m "ステップ1"
jj bookmark create feature/step1  # ← 不要

jj new main
echo "step2" > file.txt
jj commit -m "ステップ2"
jj bookmark create feature/step2  # ← 不要

jj new main
echo "step3" > file.txt
jj commit -m "ステップ3"
jj bookmark create feature/step3  # ← 不要

# 結果：大量のブックマークで管理が困難
jj bookmark list
# feature/step1
# feature/step2
# feature/step3
# feature/step4
# ...
```

**何が問題か：**
- ブックマークが多すぎて管理できない
- どれが重要かわからない
- リモートにも不要なブランチが増える

**✅ 正しい方法：**

```bash
# 1つのPRにまとめる作業は、1つのブックマーク
jj new main

echo "step1" > file.txt
jj commit -m "ステップ1"

echo "step2" > file.txt
jj commit -m "ステップ2"

echo "step3" > file.txt
jj commit -m "ステップ3"

# まとめてsquash
jj edit @---
jj squash
jj squash
jj describe -m "機能Xを実装

- ステップ1
- ステップ2
- ステップ3"

# 最後に1つのブックマーク
jj bookmark create feature/feature-x
jj git push --branch feature/feature-x

# 結果：1つのPR、1つのブックマーク
```

**💡 ルール：**
- **1つのPR = 1つのブックマーク**
- 細かい作業ごとにブックマークを作らない
- ローカルで試行錯誤してからブックマーク作成

---

### ❌ アンチパターン5.3：ブックマーク名の命名規則がない

**問題のあるコード：**

```bash
jj bookmark create mywork
jj bookmark create temp
jj bookmark create test123
jj bookmark create aaaaa
jj bookmark create new-new-new
```

**何が問題か：**
- 何の作業か不明
- チーム内で混乱
- 後で見返した時に理解できない

**✅ 正しい方法：**

```bash
# 命名規則を決める
jj bookmark create feature/user-authentication
jj bookmark create bugfix/login-crash
jj bookmark create hotfix/security-cve-2024
jj bookmark create refactor/api-client
jj bookmark create docs/setup-guide
```

**💡 ルール：**
- **チームで命名規則を統一**
- `<type>/<description>` の形式を推奨

---

### ❌ アンチパターン5.4：古いブックマークを削除しない

**問題のあるコード：**

```bash
jj bookmark list
# feature/old-1 (3ヶ月前)
# feature/old-2 (2ヶ月前)
# feature/old-3 (1ヶ月前)
# ... 数十個のブックマーク
```

**何が問題か：**
- リポジトリが肥大化
- どれが現在の作業か不明
- リモートにも不要なブランチが残る

**✅ 正しい方法：**

```bash
# マージ済みブックマークを定期的に削除
jj bookmark list

# ローカルで削除
jj bookmark delete feature/old-merged

# リモートでも削除
jj git push --branch feature/old-merged --delete

# または GitHub CLI で
gh pr list --state merged
gh pr view 123  # マージ済みか確認
```

**💡 ルール：**
- **週に1回は古いブックマークを整理**
- マージ済みブランチは削除

---

## 6. コンフリクト解決の失敗

### ❌ アンチパターン6.1：コンフリクトを無視してcommit

**問題のあるコード：**

```bash
jj rebase -s feature/mine -d main
# コンフリクト発生

jj status
# Working copy changes:
# C src/app.js

# コンフリクトを無視してプッシュ！
jj git push --branch feature/mine  # ← 壊れたコードをプッシュ！
```

**何が問題か：**
- 壊れたコードがリモートに
- CIが失敗
- チームメンバーが使えない

**✅ 正しい方法：**

```bash
jj rebase -s feature/mine -d main
# コンフリクト発生

# 必ず解決する
jj status  # Cマークを確認
vim src/app.js  # 手動で解決

# 解決されたか確認
jj status  # Cマークが消える

# テストを実行
npm test  # ← 必須！

# プッシュ
jj git push --branch feature/mine
```

**💡 ルール：**
- **コンフリクトは必ず解決**
- `jj status`でCマークが消えるまで確認
- テストを実行してから push

---

### ❌ アンチパターン6.2：コンフリクトマーカーを残したまま

**問題のあるコード：**

```javascript
// src/app.js
function login() {
<<<<<<< local
    return authenticate();
=======
    return newAuthenticate();
>>>>>>> remote
}
```

**何が問題か：**
- コードが動かない
- 構文エラー
- CIが失敗

**✅ 正しい方法：**

```javascript
// src/app.js
function login() {
    return newAuthenticate();  // ← マーカーを削除して選択
}
```

**または jj resolve を使う：**

```bash
jj resolve  # 対話的に解決
```

**💡 ルール：**
- **コンフリクトマーカーを必ず削除**
- エディタで確認してから commit

---

## 7. パフォーマンスの問題

### ❌ アンチパターン7.1：巨大なバイナリファイルをコミット

**問題のあるコード：**

```bash
# 大きなファイルをコミット
cp ~/Downloads/large-video.mp4 .
jj commit -m "ビデオを追加"  # ← 500MB！
```

**何が問題か：**
- リポジトリが肥大化
- クローン/fetchが遅くなる
- GitHub/GitLabの容量制限に引っかかる

**✅ 正しい方法：**

```bash
# .gitignoreに追加
echo "*.mp4" >> .gitignore
echo "*.zip" >> .gitignore
echo "*.tar.gz" >> .gitignore

# Git LFS を使う（大きなファイルが必要な場合）
git lfs track "*.mp4"
```

**💡 ルール：**
- **バイナリファイルは原則コミットしない**
- 必要ならGit LFSを使う
- ビルド成果物はコミットしない

---

### ❌ アンチパターン7.2：operation logが肥大化するまで放置

**問題のあるコード：**

```bash
# 何千回もの操作
jj new main
jj commit -m "test"
jj op undo
# ... これを数百回繰り返す

jj op log  # ← 膨大な操作履歴
```

**何が問題か：**
- operation logが肥大化
- `jj op log` が遅くなる
- ディスク容量を圧迫

**✅ 正しい方法：**

```bash
# 定期的にガベージコレクション
jj util gc

# または、古い操作履歴を手動で削除（慎重に）
jj config set --user operation.gc.max-age "30 days"
```

**💡 ルール：**
- **定期的に `jj util gc` を実行**
- 不要な操作履歴は削除

---

## 8. アンチパターンを仕組みで防ぐ

人間はミスをするものです。ここでは、設定やツールを使ってアンチパターンを**自動的に防ぐ方法**を紹介します。

### 8.1 Git Hooks で防ぐ

**重要：** jujutsuはGit hooksをネイティブサポートしていません。Git-backed/colocated形式のリポジトリでGit hooksを使用する場合は、Gitの`core.hooksPath`設定を使用して、ユーザーが管理するディレクトリにhooksファイルを配置する必要があります。

#### Hooksディレクトリの設定

```bash
# 1. プロジェクトルートにhooksディレクトリを作成
mkdir -p .githooks

# 2. Gitのhooks pathを設定
git config core.hooksPath .githooks

# 3. 設定を確認
git config --get core.hooksPath
```

#### pre-push hook：共有ブランチへの直接プッシュを防ぐ

**`.githooks/pre-push`**

```bash
#!/bin/bash

# プッシュしようとしているブランチ名を取得
while read local_ref local_sha remote_ref remote_sha; do
    branch_name=$(echo "$remote_ref" | sed 's/refs\/heads\///')
    
    # 保護対象のブランチリスト
    protected_branches=("main" "master" "develop" "production")
    
    for protected in "${protected_branches[@]}"; do
        if [ "$branch_name" = "$protected" ]; then
            echo "❌ エラー: '$protected' への直接プッシュは禁止されています"
            echo "💡 ヒント: feature ブランチを作成してPRを出してください"
            echo ""
            echo "  jj bookmark create feature/your-feature"
            echo "  jj git push --branch feature/your-feature"
            exit 1
        fi
    done
done

exit 0
```

**有効化：**

```bash
# 実行権限を付与
chmod +x .githooks/pre-push
```

---

#### pre-push hook：ブランチ名の命名規則をチェック

**`.githooks/pre-push`**

```bash
#!/bin/bash

while read local_ref local_sha remote_ref remote_sha; do
    branch_name=$(echo "$remote_ref" | sed 's/refs\/heads\///')
    
    # ブランチ名のパターンチェック
    if [[ ! "$branch_name" =~ ^(feature|bugfix|hotfix|refactor|docs)/ ]]; then
        echo "❌ エラー: ブランチ名が命名規則に従っていません"
        echo "💡 正しい形式: <type>/<description>"
        echo "   例: feature/user-auth, bugfix/login-error"
        echo ""
        echo "許可されているプレフィックス:"
        echo "  - feature/  : 新機能"
        echo "  - bugfix/   : バグ修正"
        echo "  - hotfix/   : 緊急修正"
        echo "  - refactor/ : リファクタリング"
        echo "  - docs/     : ドキュメント"
        exit 1
    fi
done

exit 0
```

---

#### pre-push hook：コンフリクトが解決されているかチェック（警告のみ）

**重要：** コンフリクトマーカーのチェックは **pre-push** で行います。**pre-commit では行いません**。

**理由：**
- jujutsuの利点：コンフリクト状態でもローカルで作業を継続できる
- ローカルではコンフリクトがあってもコミットしてOK
- プッシュ前には警告を出すが、最終判断は開発者に委ねる

**`.githooks/pre-push`（上記に追加）**

```bash
#!/bin/bash

has_error=0

# 共有ブランチチェック（前述）
while read local_ref local_sha remote_ref remote_sha; do
    branch_name=$(echo "$remote_ref" | sed 's/refs\/heads\///')
    
    # 保護対象のブランチリスト
    protected_branches=("main" "master" "develop" "production")
    
    for protected in "${protected_branches[@]}"; do
        if [ "$branch_name" = "$protected" ]; then
            echo "❌ エラー: '$protected' への直接プッシュは禁止されています"
            echo "💡 ヒント: feature ブランチを作成してPRを出してください"
            exit 1
        fi
    done
    
    # ブランチ名のパターンチェック
    if [[ ! "$branch_name" =~ ^(feature|bugfix|hotfix|refactor|docs)/ ]]; then
        echo "❌ エラー: ブランチ名が命名規則に従っていません"
        echo "💡 正しい形式: <type>/<description>"
        exit 1
    fi
    
    # コンフリクトマーカーを検出（警告のみ、プッシュは止めない）
    # jujutsuのrevsetでコンフリクトを検出
    if jj log -r "conflict()" --limit 1 2>/dev/null | grep -q "change"; then
        echo ""
        echo "⚠️  警告: コンフリクトマーカーが残っています"
        echo ""
        echo "以下を確認してください："
        echo "  jj status  # コンフリクトを確認"
        echo "  jj log -r 'conflict()'  # コンフリクト中のコミット一覧"
        echo ""
        read -p "このままプッシュしますか？ (y/N): " confirm
        if [ "$confirm" != "y" ] && [ "$confirm" != "Y" ]; then
            echo "キャンセルしました"
            echo ""
            echo "💡 解決方法："
            echo "  vim <file>     # ファイルを編集して解決"
            echo "  jj resolve     # 対話的に解決"
            exit 1
        fi
        echo "⚠️  コンフリクトマーカー付きでプッシュします..."
    fi
done

exit 0
```

**💡 jujutsuの哲学：**
- **ローカル**：コンフリクトがあっても自由に作業を継続
- **リモート**：警告は出すが、最終判断は開発者に委ねる
- 柔軟性とチーム安全性のバランス

---

#### コンフリクトをコマンドでチェックする仕組み

**エイリアスを追加（`~/.config/jj/config.toml`）**

```toml
[aliases]
# コンフリクトチェック
check-conflicts = ["log", "-r", "conflict()"]
conflicts-count = ["log", "-r", "conflict()", "--no-graph", "-T", "description"]
has-conflicts = ["log", "-r", "conflict()", "--limit", "1"]

# プッシュ前チェック（安全確認）
pre-push-check = ["status"]
```

**使い方：**

```bash
# コンフリクト中のコミット一覧
jj check-conflicts

# コンフリクトの数を確認
jj conflicts-count

# コンフリクトがあるかどうか（空なら問題なし）
jj has-conflicts

# プッシュ前の確認
jj pre-push-check
jj git push --branch my-feature
```

**シェルスクリプトで自動チェック（`~/bin/jj-push-safe`）**

```bash
#!/bin/bash

# 安全なプッシュスクリプト

branch=$1

if [ -z "$branch" ]; then
    echo "❌ エラー: ブランチ名を指定してください"
    echo "使い方: jj-push-safe <branch-name>"
    exit 1
fi

echo "🔍 プッシュ前チェックを実行中..."
echo ""

# 1. コンフリクトをチェック
if jj log -r "conflict()" --limit 1 2>/dev/null | grep -q "conflict"; then
    echo "⚠️  警告: コンフリクトが残っています"
    jj log -r "conflict()" --limit 5
    echo ""
    read -p "このままプッシュしますか？ (y/N): " confirm
    if [ "$confirm" != "y" ] && [ "$confirm" != "Y" ]; then
        echo "キャンセルしました"
        exit 1
    fi
else
    echo "✅ コンフリクトなし"
fi

# 2. 未コミットの変更をチェック
if jj status | grep -q "Working copy changes:"; then
    echo "⚠️  警告: 未コミットの変更があります"
    jj status
    echo ""
    read -p "このままプッシュしますか？ (y/N): " confirm
    if [ "$confirm" != "y" ] && [ "$confirm" != "Y" ]; then
        echo "キャンセルしました"
        exit 1
    fi
else
    echo "✅ 未コミットの変更なし"
fi

# 3. プッシュ実行
echo ""
echo "🚀 プッシュ中..."
jj git push --branch "$branch"

if [ $? -eq 0 ]; then
    echo "✅ プッシュ成功！"
else
    echo "❌ プッシュ失敗"
    exit 1
fi
```

**セットアップ：**

```bash
chmod +x ~/bin/jj-push-safe

# エイリアス追加（~/.zshrc または ~/.bashrc）
alias jjp='jj-push-safe'

# 使い方
jjp feature/my-feature
```

---

#### ターミナルプロンプトでコンフリクトを表示

##### Zsh用（Powerlevel10k/Starship/カスタム）

**1. Starship を使う場合（推奨）**

https://starship.rs/

**`~/.config/starship.toml`**

```toml
# Jujutsuサポート
[jj]
disabled = false
format = "on [$symbol$state( $repo_name)]($style) "
symbol = "🦀 "
style = "bold purple"

# コンフリクトがある場合は赤色で表示
conflicted = "🔥"
conflicted_style = "bold red"
```

**表示例：**
```
~/project 🦀 main         # 正常
~/project 🦀 🔥 main      # コンフリクトあり
```

---

**2. カスタムZshプロンプト**

**`~/.zshrc` に追加**

```bash
# Jujutsuのコンフリクトをプロンプトに表示

function jj_conflict_status() {
    # jujutsuリポジトリかチェック
    if [ ! -d .jj ]; then
        return
    fi
    
    # コンフリクトがあるかチェック
    local conflicts=$(jj log -r "conflict()" --limit 1 --no-graph --color=never 2>/dev/null)
    
    if [ -n "$conflicts" ]; then
        echo "%{$fg_bold[red]%}⚠️ CONFLICT%{$reset_color%} "
    fi
}

function jj_bookmark_status() {
    if [ ! -d .jj ]; then
        return
    fi
    
    # 現在のブックマークを取得
    local bookmark=$(jj log -r @ --no-graph -T 'bookmarks' --color=never 2>/dev/null | tr -d '\n')
    
    if [ -n "$bookmark" ]; then
        echo "%{$fg[cyan]%}$bookmark%{$reset_color%}"
    else
        echo "%{$fg[yellow]%}(no bookmark)%{$reset_color%}"
    fi
}

# プロンプトに統合
setopt PROMPT_SUBST
PROMPT='%{$fg[green]%}%n@%m%{$reset_color%}:%{$fg[blue]%}%~%{$reset_color%} $(jj_conflict_status)$(jj_bookmark_status)
%# '
```

**表示例：**
```bash
user@host:~/project feature/my-work
%

user@host:~/project ⚠️ CONFLICT feature/my-work
%
```

---

**3. Oh My Zsh用カスタムテーマ**

**`~/.oh-my-zsh/custom/themes/jujutsu.zsh-theme`**

```bash
# Jujutsuサポート付きテーマ

function jj_prompt_info() {
    if [ ! -d .jj ]; then
        return
    fi
    
    local bookmark=$(jj log -r @ --no-graph -T 'bookmarks' --color=never 2>/dev/null | tr -d '\n')
    local conflicts=$(jj log -r "conflict()" --limit 1 --no-graph --color=never 2>/dev/null)
    
    local status_line=""
    
    if [ -n "$conflicts" ]; then
        status_line="%{$fg_bold[red]%}🔥 "
    fi
    
    if [ -n "$bookmark" ]; then
        status_line="${status_line}%{$fg[cyan]%}⎇ ${bookmark}%{$reset_color%}"
    else
        status_line="${status_line}%{$fg[yellow]%}(no bookmark)%{$reset_color%}"
    fi
    
    echo "$status_line"
}

PROMPT='%{$fg[green]%}%n@%m%{$reset_color%} %{$fg[blue]%}%~%{$reset_color%} $(jj_prompt_info)
%# '
```

**有効化：**

```bash
# ~/.zshrc
ZSH_THEME="jujutsu"
```

**表示例：**

```bash
# 通常の状態（コンフリクトなし、ブックマークあり）
user@host ~/project ⎇ feature/login
%

# コンフリクトあり（赤い🔥で警告）
user@host ~/project 🔥 ⎇ feature/login
%

# ブックマークなし（黄色で警告）
user@host ~/project (no bookmark)
%

# コンフリクトあり＆ブックマークなし（両方警告）
user@host ~/project 🔥 (no bookmark)
%

# mainブランチにいる場合
user@host ~/project ⎇ main
%

# 複数のブックマークがある場合
user@host ~/project ⎇ feature/login feature/signup
%
```

**色の説明：**
- `user@host` - 緑色
- `~/project` - 青色
- `🔥` - 赤色（コンフリクト警告）
- `⎇ feature/login` - シアン色（ブックマーク）
- `(no bookmark)` - 黄色（警告）

---

##### Bash用

**`~/.bashrc` に追加**

```bash
# Jujutsuのコンフリクトをプロンプトに表示

function __jj_ps1() {
    if [ ! -d .jj ]; then
        return
    fi
    
    local bookmark=$(jj log -r @ --no-graph -T 'bookmarks' --color=never 2>/dev/null | tr -d '\n')
    local conflicts=$(jj log -r "conflict()" --limit 1 --no-graph --color=never 2>/dev/null)
    
    local status=""
    
    if [ -n "$conflicts" ]; then
        status="\[\033[1;31m\]⚠️ CONFLICT\[\033[0m\] "
    fi
    
    if [ -n "$bookmark" ]; then
        status="${status}\[\033[0;36m\]⎇ ${bookmark}\[\033[0m\]"
    else
        status="${status}\[\033[0;33m\](no bookmark)\[\033[0m\]"
    fi
    
    echo -e " ${status}"
}

PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]$(__jj_ps1)\n\$ '
```

---

#### プロンプトの表示パターン

```bash
# 通常状態
user@host:~/project ⎇ feature/login
$

# コンフリクトあり（赤で警告）
user@host:~/project ⚠️ CONFLICT ⎇ feature/login
$

# ブックマークなし（黄色で警告）
user@host:~/project (no bookmark)
$

# コンフリクトあり＆ブックマークなし
user@host:~/project ⚠️ CONFLICT (no bookmark)
$
```

---

#### まとめ：多層防御（柔軟版）

```
1層目：ターミナルプロンプト
  ↓ 視覚的に常時警告（⚠️ CONFLICT）
  
2層目：コマンドでチェック
  ↓ jj check-conflicts, jj-push-safe
  
3層目：pre-push hook
  ↓ 警告＋確認（止めない）
  
4層目：CI/CD
  ↓ リモートで検証（エラー）
  
5層目：Branch Protection
  ↓ マージ時に最終確認
  
= 柔軟性と安全性のバランス
```

**💡 重要な哲学：**

1. **ローカルでは警告のみ** - 開発者の判断を尊重
2. **視覚的なフィードバック** - プロンプトで常に状態を確認
3. **リモートでは厳格に** - CI/CDで品質を保証
4. **最終判断は人間** - ツールは補助、決定は開発者

---

### 8.2 チーム向け：Branch Protection Rules（GitHub/GitLab）

#### GitHub Branch Protection

GitHubのリポジトリ設定で、強制的にルールを適用：

**Settings → Branches → Add rule**

```
Branch name pattern: main

☑ Require a pull request before merging
  ☑ Require approvals (最低1人)
  ☑ Dismiss stale pull request approvals when new commits are pushed
  
☑ Require status checks to pass before merging
  ☑ Require branches to be up to date before merging
  
☑ Require conversation resolution before merging

☑ Do not allow bypassing the above settings

☑ Restrict who can push to matching branches
  (管理者のみ許可)
```

これにより：
- mainへの直接プッシュが**物理的に不可能**になる
- 必ずPR経由になる
- レビューが必須になる

---

### 8.3 実装の優先順位

#### レベル1：すぐに導入すべき（必須）

1. **Git Hooks（pre-push）**
   - 共有ブランチへのプッシュを防ぐ
   - 実装コスト：低
   - 効果：高

2. **GitHub Branch Protection**
   - リモートで強制
   - 実装コスト：ゼロ（設定のみ）
   - 効果：最高

3. **ターミナルプロンプト表示**
   - コンフリクトを視覚的に警告
   - 実装コスト：低
   - 効果：中

#### レベル2：チームで検討（推奨）

4. **コンフリクトチェックエイリアス**
   - jj check-conflicts
   - 実装コスト：低
   - 効果：中

5. **安全なプッシュスクリプト**
   - jj-push-safe
   - 実装コスト：中
   - 効果：中

---

### 8.4 チーム導入のステップ

#### Week 1：ルール策定

```bash
# チームで合意
1. ブランチ命名規則を決める
2. コミットメッセージの最低要件を決める
3. 保護するブランチを決める
```

#### Week 2：GitHub設定

```bash
# リモートで強制
1. Branch Protection Rules を設定
2. 必須レビュアーを設定
3. ステータスチェックを設定
```

#### Week 3：ローカル防御

```bash
# 各メンバーのローカル
1. pre-push hookを配布
2. .jj/repo/store/git/hooks に配置
3. 実行権限を付与
4. ターミナルプロンプトを設定
```

#### Week 4：エイリアスとスクリプト

```bash
# 便利なツール導入
1. check-conflicts エイリアス追加
2. jj-push-safe スクリプト配布
3. 使い方を共有
```

---

### 8.5 トラブルシューティング

#### Hooksが動作しない場合

```bash
# core.hooksPathの設定を確認
git config --get core.hooksPath

# 実行権限を確認
ls -la .githooks/pre-push

# 権限がない場合
chmod +x .githooks/pre-push

# hookをテスト
.githooks/pre-push
```

#### プロンプトが表示されない場合

```bash
# シェルをリロード
source ~/.zshrc
# または
source ~/.bashrc

# jujutsuリポジトリかチェック
ls -la .jj
```

#### Hooksをスキップしたい場合（緊急時のみ）

```bash
# Git環境変数でスキップ（非推奨）
SKIP_HOOKS=1 jj git push --branch emergency-fix

# または
git push --no-verify
```

**⚠️ 注意：** 緊急時以外はhooksをスキップしないこと

---

### 8.6 まとめ：防御の多重化

```
1層目：ターミナルプロンプト
  ↓ 視覚的に常時警告（⚠️ CONFLICT）
  
2層目：コマンドでチェック
  ↓ jj check-conflicts, jj-push-safe
  
3層目：pre-push hook
  ↓ 警告＋確認（止めない）
  
4層目：CI/CD
  ↓ リモートで検証（エラー）
  
5層目：Branch Protection
  ↓ マージ時に最終確認
  
= 柔軟性と安全性のバランス
```

**💡 重要な原則：**

1. **完璧な防御は存在しない**
   - 複数の層で防御する
   - 教育も重要

2. **バランスが大事**
   - 厳しすぎると生産性が落ちる
   - 緩すぎると事故が起きる

3. **チームで合意**
   - ルールはチーム全員で決める
   - 定期的に見直す

4. **緊急時の逃げ道**
   - hooksをスキップできる方法も用意
   - ただし記録を残す

5. **ローカルは柔軟、リモートは厳格**
   - jujutsuの哲学を活かす
   - 開発者の判断を尊重

---

## まとめ：避けるべき行動チェックリスト

### チーム開発

- [ ] 共有ブランチ（main, develop）を直接編集しない
- [ ] PRマージ後のブランチは編集しない
- [ ] operation undoでチームの作業を巻き戻さない

### 履歴編集

- [ ] 過度なsquashをしない（論理的な単位で）
- [ ] 意味のないコミットメッセージを避ける
- [ ] change IDを直接編集しようとしない

### Git連携

- [ ] `jj git pull` を使おうとしない（存在しない）
- [ ] リモートブランチを直接編集しない
- [ ] リモートを明示的に管理する

### operation log

- [ ] operation logが永遠に残ると思わない
- [ ] operation undoを連打しない
- [ ] 重要な状態はブックマークで保存

### ブックマーク管理

- [ ] プッシュする作業には必ずブックマークを付ける
- [ ] 1つのPR = 1つのブックマーク
- [ ] ローカルだけの試行錯誤にはブックマーク不要
- [ ] 細かい作業ごとにブックマークを作らない
- [ ] 命名規則を統一する
- [ ] 古いブックマークを定期的に削除

### コンフリクト解決

- [ ] コンフリクトを無視してcommitしない
- [ ] コンフリクトマーカーを残さない
- [ ] 解決後は必ずテストを実行

### パフォーマンス

- [ ] 巨大なバイナリファイルをコミットしない
- [ ] 定期的に `jj util gc` を実行
- [ ] operation logを肥大化させない

---

## 困った時の対処法

### パニックになった時

1. **落ち着く**：すべて元に戻せる
2. **確認する**：`jj op log` と `jj log` で状態確認
3. **戻す**：`jj op undo` または `jj op restore`
4. **相談する**：チームメンバーやコミュニティに聞く

### operation logで戻せない時

```bash
# Gitのreflogを使う（最後の手段）
cd .jj/repo/store/git
git reflog
git checkout <commit-hash>
```

### すべてを失った気がする時

**安心してください！**
- jujutsuは操作履歴を保持している
- Gitバックエンドなら、Git側にもデータが残っている
- ほぼすべての操作は復元可能

---

## 参考リンク

- [06-workflows.md](06-workflows.md) - 推奨されるワークフロー
- [04-history-editing.md](04-history-editing.md) - 安全な履歴編集方法
- [05-advanced.md](05-advanced.md) - operation logの活用

**💡 心に刻むべき原則：**

1. **共有ブランチは神聖**：編集しない
2. **operation logは一時的**：重要な状態はブックマーク
3. **コンフリクトは必ず解決**：無視しない
4. **意味のあるメッセージ**：将来の自分への手紙
5. **チームとコミュニケーション**：独断で危険な操作をしない

これらを守れば、jujutsuを安全に、そして効果的に使えます！🎯

