# 03. マージとコンフリクト解決：ブランチを統合する

このセクションでは、複数のブランチで行った変更を統合する方法と、コンフリクト（競合）の解決方法を学びます。

## 学習目標

- ブランチをマージする
- コンフリクトとは何かを理解する
- コンフリクトを解決する
- マージの戦略を学ぶ

---

## 準備：練習用のシナリオを作成

新しいディレクトリで始めましょう。

```bash
cd ~/Desktop
mkdir jj-merge-practice
cd jj-merge-practice
jj git init
```

---

## ⚠️ 重要：ブックマーク作成のタイミング

### よくある間違い

```bash
# ❌ 間違い：コミット後にブックマークを作成
vim file.txt
jj commit -m "変更"
jj bookmark create my-branch  # 空のworking copyに付いてしまう！
```

**問題：**
```
@  xyz... my-branch | (empty)        ← ブックマークがここに
◉  abc... 変更                       ← 本当はここに付けたい
```

### 正しい方法：`-r @-`で親コミットを指定

```bash
# ✅ 推奨：作業後にブックマークを作成
vim file.txt
jj commit -m "変更"
jj bookmark create my-branch -r @-  # 親コミットに付ける
```

**結果：**
```
@  xyz... (empty)
◉  abc... my-branch | 変更    ← ブックマークが正しい位置に
```

**💡 メリット：**
- 警告メッセージが出ない
- 作業内容が明確になってからブックマーク名を決められる
- 常に同じワークフロー（編集→コミット→ブックマーク）で作業できる
- 確実に正しい位置にブックマークが付く
- 本ドキュメントで一貫して使用している方法

---

### 🪄 補足：スムーズな運用パターン

実際の開発では、以下のようなワークフローを使うとスムーズです：

#### 基本の標準フロー

```bash
# 1. mainから新しい作業を開始
jj new main

# 2. 作業を進める
vim src/feature.rs
vim tests/feature_test.rs

# 3. コミット + ブックマークを一気に
jj commit -m "機能Xを実装"
jj bookmark create feature/x -r '@-'
```

**💡 メリット：**
- 作業内容を確認してからブックマーク名を決められる
- 警告メッセージが出ない
- 確実に正しい位置にブックマークが付く
- 本ドキュメントで一貫して使用している方法

#### エイリアスで効率化

頻繁に使うコマンドを短縮したい場合は、`~/.config/jj/config.toml`にエイリアスを追加：

```toml
[aliases]
nb = ["new", "main"]  # new bookmark用のショートカット
bc = ["bookmark", "create"]
```

使用例：

```bash
jj nb                          # mainから新しい作業開始
vim feature.rs

jj commit -m "実装"
jj bc my-feature -r '@-'       # 素早くブックマーク作成
```

**💡 メリット：** タイピング量が減り、作業が高速化

---

### なぜこうなるのか？

`jj commit`を実行すると：
1. 現在の変更がコミットとして確定
2. **自動的に新しい空のworking copyが作成される**
3. その空のworking copyに移動

この時点で`jj bookmark create`すると、空のworking copyにブックマークが付いてしまいます。

### 確認方法

```bash
# ブックマークが正しい位置にあるか確認
jj bookmark list

# 警告が出たら要注意
Warning: Target revision is empty.
```

この警告が出たら、空のコミットにブックマークを付けています！

### 修正方法

```bash
# 間違ったブックマークを削除
jj bookmark delete wrong-bookmark

# 正しい位置に作り直す
jj bookmark create correct-bookmark -r <正しいコミットID>
```

---

## 1. マージの基本概念

### マージとは？

- 複数のブランチの変更を統合すること
- 2つ以上の親を持つコミットを作成する

### jujutsuのマージの特徴

- マージは明示的に行う
- コンフリクトは「状態」として保持される
- 後からいつでもコンフリクトを解決できる

---

## 2. マージの準備：2つのブランチを作成

### ベースとなるファイルを作成

```bash
echo "# ウェブサイトのタイトル

## メインコンテンツ

ここにコンテンツが入ります。" > index.md

jj commit -m "初期ウェブサイトを作成"
jj bookmark create main -r @-
```

**💡 ポイント：** 初回のコミットなので、`-r @-`で親コミットにブックマークを付けています。

### ブランチ1：デザインの改善

```bash
jj new main
```

**💡 重要：** 作業を開始する前にブックマークを作成します！

```bash
echo "# 🎨 素敵なウェブサイトのタイトル

## メインコンテンツ

ここにコンテンツが入ります。

## スタイル

- モダンなデザイン
- レスポンシブ対応" > index.md

jj commit -m "デザインを改善"
jj bookmark create feature/design -r '@-'
```

**💡 ポイント：** コミット後に`-r @-`で親コミットにブックマークを付けています。

### ブランチ2：コンテンツの追加

```bash
jj new main
```

```bash
echo "# ウェブサイトのタイトル

## メインコンテンツ

ここにコンテンツが入ります。

## 新しいセクション

このセクションは新しく追加されました。

### サブセクション

詳細な情報がここに入ります。" > index.md

jj commit -m "コンテンツを追加"
jj bookmark create feature/content -r '@-'
```

**💡 ポイント：** こちらもコミット後に`-r @-`でブックマークを付けています。

### 現在の状態を確認

```bash
jj log
```

**期待される出力：**

```
@  xyzbcdef taro@example.com 2025-10-23 11:00:15 7h8i9j0k
│  (empty) (no description set)
│ ◉  abcdefgh taro@example.com 2025-10-23 10:58:22 4e5f6g7h feature/content
├─╯  コンテンツを追加
│ ◉  ijklmnop taro@example.com 2025-10-23 10:56:45 1b2c3d4e feature/design
├─╯  デザインを改善
◉  qrstuvwx taro@example.com 2025-10-23 10:55:10 8a9b0c1d main
│  初期ウェブサイトを作成
◉  zzzzzzzz root() 00000000
   (empty) (no description set)
```

2つのブランチが`main`から分岐しています！

---

## 3. コンフリクトなしのマージ

まずは簡単なケース：異なるファイルを扱っている場合を試しましょう。

### 新しいシナリオを作成

```bash
jj new main

echo "body { font-family: Arial; }" > style.css
jj commit -m "スタイルシートを追加"
jj bookmark create add-css -r '@-'

jj new main

echo "console.log('Hello');" > script.js
jj commit -m "JavaScriptを追加"
jj bookmark create add-js -r '@-'
```

**💡 ポイント：** 各ブランチでコミット後に`-r @-`でブックマークを作成しています。

### 2つのブランチをマージ

```bash
jj new add-css add-js -m "CSSとJSをマージ"
```

**💡 ポイント：** マージは`jj new`で複数の親を指定することで行います。`jj merge`コマンドは存在しません。

**期待される出力：**

```
Working copy now at: newcommit 3m4n5o6p CSSとJSをマージ
Parent commit      : mergeid1 2a3b4c5d スタイルシートを追加
Parent commit      : mergeid2 6d7e8f9g JavaScriptを追加
Added 2 files, modified 0 files, removed 0 files
```

### ファイルを確認

```bash
ls
```

**出力：**

```
index.md  script.js  style.css
```

両方のファイルが存在します！これは**クリーンなマージ**です。

---

## 4. コンフリクトが発生するマージ

今度は、同じファイルの同じ部分を異なる方法で変更した場合を見てみましょう。

### 最初のブランチに戻る

```bash
jj edit feature/design
```

`index.md`の内容を確認：

```bash
cat index.md
```

### もう一つのブランチを確認

```bash
jj edit feature/content
cat index.md
```

### マージを実行

```bash
jj new feature/design feature/content -m "デザインとコンテンツをマージ"
```

**期待される出力：**

```
Working copy now at: pqrstuv 9h8g7f6e デザインとコンテンツをマージ
Parent commit      : ijklmnop 1b2c3d4e feature/design デザインを改善
Parent commit      : abcdefgh 4e5f6g7h feature/content コンテンツを追加
Added 0 files, modified 1 files, removed 0 files
New conflicts appeared in these commits:
  pqrstuv 9h8g7f6e デザインとコンテンツをマージ
To resolve the conflicts, start by updating to it:
  jj new pqrstuv
Then use `jj resolve` to resolve the conflicts.
```

**コンフリクトが発生しました！** 😱

---

## 5. コンフリクトを確認する

### 状態を確認

```bash
jj status
```

**期待される出力：**

```
Working copy changes:
C index.md
Working copy : pqrstuv 9h8g7f6e (conflict) デザインとコンテンツをマージ
Parent commit: ijklmnop 1b2c3d4e feature/design デザインを改善
Parent commit: abcdefgh 4e5f6g7h feature/content コンテンツを追加
```

`C index.md`：Conflictを意味します。

### コンフリクトの内容を確認

```bash
cat index.md
```

**期待される出力：**

```
<<<<<<<
%%%%%%%
# ウェブサイトのタイトル
+++++++
# 🎨 素敵なウェブサイトのタイトル
>>>>>>>

## メインコンテンツ

ここにコンテンツが入ります。

<<<<<<<
%%%%%%%
+++++++
## スタイル

- モダンなデザイン
- レスポンシブ対応
>>>>>>>
<<<<<<<
%%%%%%%
+++++++
## 新しいセクション

このセクションは新しく追加されました。

### サブセクション

詳細な情報がここに入ります。
>>>>>>>
```

### 📖 コンフリクトマーカーの読み方

```
<<<<<<<  コンフリクト開始
%%%%%%%  ベース（共通の祖先）の内容
+++++++  1つ目のブランチの内容
>>>>>>>  コンフリクト終了
```

---

## 6. コンフリクトを解決する（方法1：手動編集）

### ファイルを直接編集

エディタで`index.md`を開き、コンフリクトマーカーを削除して、望ましい内容にします：

```bash
nano index.md
```

または

```bash
code index.md
```

**編集後の内容（例）：**

```markdown
# 🎨 素敵なウェブサイトのタイトル

## メインコンテンツ

ここにコンテンツが入ります。

## 新しいセクション

このセクションは新しく追加されました。

### サブセクション

詳細な情報がここに入ります。

## スタイル

- モダンなデザイン
- レスポンシブ対応
```

保存して閉じます。

### 状態を確認

```bash
jj status
```

コンフリクトが解決されているはずです！

### コミット（オプション）

working copyの状態をコミットします：

```bash
jj commit -m "コンフリクトを解決してマージ完了"
```

---

## 7. コンフリクトを解決する（方法2：`jj resolve`）

jujutsuには対話的にコンフリクトを解決するツールがあります。

### 別のコンフリクトを作成（練習用）

```bash
jj new main

echo "バージョン1の内容" > conflict-test.txt
jj commit -m "バージョン1"
jj bookmark create version1 -r '@-'

jj new main

echo "バージョン2の内容" > conflict-test.txt
jj commit -m "バージョン2"
jj bookmark create version2 -r '@-'

jj new version1 version2 -m "バージョンをマージ"
```

**💡 ポイント：** 各ブランチでコミット後に`-r @-`でブックマークを作成しています。

コンフリクトが発生します。

### `jj resolve`を使用

```bash
jj resolve
```

対話的なインターフェースが起動し、コンフリクトを解決できます：

- **左側（local）**：最初のブランチ
- **右側（other）**：2番目のブランチ
- **ベース**：共通の祖先

必要な変更を選択・編集して保存します。

---

## 8. マージの種類

### 2-wayマージ（既に実践済み）

```bash
jj new parent1 parent2
```

2つの親を持つコミットを作成。

### 3-way以上のマージ

複数のブランチを一度にマージできます：

```bash
jj new branch1 branch2 branch3
```

---

## 9. マージの取り消し

### 間違ったマージをした場合

マージコミットを削除：

```bash
jj abandon @
```

これでマージコミットが削除され、元の状態に戻ります。

---

## やってみよう！

以下の練習問題に挑戦してください：

### 練習1：クリーンなマージ

1. 新しいリポジトリを作成
2. `feature-a`ブックマークを作成して、`file-a.txt`を作成
3. `feature-b`ブックマークを作成して、`file-b.txt`を作成
4. 2つをマージ
5. 両方のファイルが存在することを確認

**正しい手順：**
```bash
jj git init

echo "feature A" > file-a.txt
jj commit -m "feature-aを追加"
jj bookmark create feature-a -r '@-'

jj new 'root()'

echo "feature B" > file-b.txt
jj commit -m "feature-bを追加"
jj bookmark create feature-b -r '@-'

jj new feature-a feature-b -m "マージ"
ls  # file-a.txt と file-b.txt の両方を確認
```

### 練習2：コンフリクトの作成と解決

1. `README.md`に「タイトル」と書く
2. ブランチ1で「タイトル A」に変更
3. ブランチ2で「タイトル B」に変更
4. マージしてコンフリクトを発生させる
5. 手動で「タイトル C」に解決

**正しい手順：**
```bash
echo "タイトル" > README.md
jj commit -m "READMEを作成"
jj bookmark create base -r '@-'

jj new base

echo "タイトル A" > README.md
jj commit -m "タイトルをAに変更"
jj bookmark create branch-a -r '@-'

jj new base

echo "タイトル B" > README.md
jj commit -m "タイトルをBに変更"
jj bookmark create branch-b -r '@-'

jj new branch-a branch-b -m "マージ"
vim README.md  # コンフリクトを解決して「タイトル C」に
jj status  # 確認
```

### 練習3：複雑なコンフリクト

1. 3つのブランチで同じファイルの異なる部分を編集
2. すべてをマージ
3. コンフリクトを確認
4. 3つの変更すべてを含む形で解決

---

## トラブルシューティング

### コンフリクトが解決されない

- コンフリクトマーカー（`<<<<<<<`など）が完全に削除されているか確認
- `jj status`で状態を確認

### マージを元に戻したい

```bash
jj abandon @
```

または

```bash
jj op undo
```

### コンフリクト解決中に混乱した

```bash
jj new parent1
```

で片方の親に戻り、改めてマージをやり直す。

---

## 重要な概念のまとめ

### マージコミット

- 2つ以上の親を持つコミット
- 複数のブランチの変更を統合

### コンフリクト

- 同じファイルの同じ部分が異なる方法で変更された状態
- jujutsuではコンフリクトを「状態」として保持
- 後からいつでも解決可能

### コンフリクト解決の方法

1. **手動編集**：エディタでマーカーを削除
2. **`jj resolve`**：対話的ツールを使用
3. **片方を選択**：一方のブランチの内容を全て採用

### jujutsuの利点

- コンフリクトを保持したままコミット可能
- 後から何度でもやり直せる
- 操作履歴から元に戻せる

---

## チェックリスト

次のセクションに進む前に、以下を確認してください：

- [ ] `jj new parent1 parent2`でマージコミットを作成できた
- [ ] クリーンなマージを実行できた
- [ ] コンフリクトを発生させることができた
- [ ] コンフリクトマーカーの意味を理解した
- [ ] 手動でコンフリクトを解決できた
- [ ] `jj status`でコンフリクトを確認できた
- [ ] `jj abandon`でマージを取り消せた

---

## 次のステップ

マージとコンフリクト解決をマスターしました！次は [04-history-editing.md](04-history-editing.md) で、履歴を編集する強力な機能を学びましょう。

- コミットの修正
- コミットの統合（squash）
- リベース操作
- コミットの分割

jujutsuの真価を発揮するセクションです！お楽しみに！🎯

