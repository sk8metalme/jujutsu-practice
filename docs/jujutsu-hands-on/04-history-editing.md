# 04. 履歴編集：過去を変更する力

このセクションでは、jujutsuの最も強力な機能の一つである履歴編集を学びます。コミット後でも、自由に履歴を整理・修正できます。

## 学習目標

- コミットメッセージを修正する
- コミットの内容を変更する
- 複数のコミットを統合する（squash）
- コミットを分割する
- リベース操作を行う
- コミットを削除する

---

## 準備：練習用のリポジトリ

```bash
cd ~/Desktop
mkdir jj-history-practice
cd jj-history-practice
jj git init
```

---

## 1. コミットメッセージの修正（`jj describe`）

### シナリオ：typoのあるコミットを作成

```bash
echo "def hello():
    print('Hello, World!')" > hello.py

jj commit -m "Helo function を追加"  # typo!
```

### メッセージを修正

```bash
jj describe -m "Hello関数を追加"
```

**期待される出力：**

```
Updated commit qpvuntsm f5e7d6c8 Hello関数を追加
```

### 履歴を確認

```bash
jj log
```

コミットメッセージが修正されています！

### 💡 ポイント

- `jj describe`は現在のコミット（`@`）のメッセージを変更
- 特定のコミットを指定することも可能：`jj describe REVISION -m "新しいメッセージ"`
- change IDは変わりません！

---

## 2. コミットの内容を変更する

### シナリオ：コミット後に修正が必要

```bash
jj new
echo "def goodbye():
    print('Goodby!')" > goodbye.py  # typo!

jj commit -m "goodbye関数を追加"
```

typoに気づきました！

### 過去のコミットを編集

```bash
jj edit @-
```

`@-`は「一つ前のコミット」を意味します。

### ファイルを修正

```bash
echo "def goodbye():
    print('Goodbye!')" > goodbye.py  # 修正
```

### 状態を確認

```bash
jj status
```

**出力：**

```
Working copy changes:
M goodbye.py
Working copy : rlvkpnrz 8e9f0a1b goodbye関数を追加
Parent commit: qpvuntsm f5e7d6c8 Hello関数を追加
```

### 変更を確定

ここで2つの選択肢があります：

#### 方法1：コミットを修正（amend）

```bash
jj commit --amend -m "goodbye関数を追加（修正版）"
```

元のコミットが上書きされます。

#### 方法2：新しいコミットとして作成

```bash
jj commit -m "typoを修正"
```

修正が新しいコミットとして記録されます。

### 元の位置に戻る

```bash
jj edit @
```

---

## 3. コミットの統合（`jj squash`）

複数の小さなコミットを1つにまとめます。

### シナリオ：細かいコミットが多数

```bash
jj new
echo "# 数学ライブラリ" > math_lib.py
jj commit -m "数学ライブラリのファイルを作成"

jj new
echo "# 数学ライブラリ

def add(a, b):
    return a + b" >> math_lib.py
jj commit -m "add関数を追加"

jj new
echo "
def subtract(a, b):
    return a - b" >> math_lib.py
jj commit -m "subtract関数を追加"

jj new
echo "
def multiply(a, b):
    return a * b" >> math_lib.py
jj commit -m "multiply関数を追加"
```

### 履歴を確認

```bash
jj log --limit 5
```

4つの細かいコミットがあります。

### 3つのコミットを1つに統合

```bash
jj squash --from @--- --to @-
```

**引数の説明：**
- `@---`：3つ前のコミット
- `@-`：1つ前のコミット
- これらの範囲のコミットを統合

### より簡単な方法

現在のコミットを親コミットに統合：

```bash
jj edit @-
jj squash
```

これで現在のコミットの変更が親コミットに統合されます。

### 履歴を確認

```bash
jj log --limit 5
```

コミットが統合されています！

---

## 4. 対話的なsquash

### 部分的に変更を統合

```bash
jj new
echo "test line 1" > test1.txt
echo "test line 2" > test2.txt
jj commit -m "テストファイルを追加"

jj new
echo "test line 1 modified" > test1.txt
echo "test line 2 modified" > test2.txt
jj commit -m "テストファイルを修正"
```

`test1.txt`の変更だけを親コミットに統合したい場合：

```bash
jj squash --interactive
```

対話的なインターフェースで、統合したい変更を選択できます。

---

## 5. コミットの分割（`jj split`）

1つのコミットを複数のコミットに分割します。

### シナリオ：複数の変更を1つのコミットにしてしまった

```bash
jj new
echo "frontend code" > frontend.js
echo "backend code" > backend.py
echo "database schema" > schema.sql
jj commit -m "色々追加"  # 良くない！
```

### コミットを分割

```bash
jj edit @-
jj split
```

対話的なインターフェースが開きます：

1. 最初のコミットに含めるファイルを選択
2. 残りは自動的に2つ目のコミットになる

または、手動で分割：

```bash
jj edit @-
```

一部のファイルだけを新しいコミットに：

```bash
jj restore --from @- frontend.js
jj commit -m "フロントエンドコードを追加"

# 残りのファイル
jj restore --from @- backend.py schema.sql
jj commit -m "バックエンドとデータベースを追加"
```

---

## 6. リベース（`jj rebase`）

コミットの親を変更します。

### シナリオ：ブランチの付け替え

```bash
jj new root()
echo "version 1" > base.txt
jj commit -m "バージョン1"
jj branch create v1

jj new root()
echo "version 2" > base.txt
jj commit -m "バージョン2"
jj branch create v2

jj new v1
echo "feature for v1" > feature.txt
jj commit -m "v1用の機能"
jj branch create feature
```

### 履歴を確認

```bash
jj log
```

`feature`ブランチは`v1`から派生しています。

### `feature`を`v2`ベースに変更

```bash
jj rebase -s feature -d v2
```

**引数の説明：**
- `-s feature`：source（移動元）
- `-d v2`：destination（移動先）

### 履歴を確認

```bash
jj log
```

`feature`ブランチが`v2`から派生するようになりました！

---

## 7. リベースの別パターン

### 範囲指定でリベース

複数のコミットをまとめて移動：

```bash
jj rebase -s "start_commit::end_commit" -d destination
```

### 現在のコミットをリベース

```bash
jj rebase -d new_parent
```

現在のコミット（`@`）が`new_parent`の子になります。

---

## 8. コミットの削除（`jj abandon`）

不要なコミットを削除します。

### コミットを削除

```bash
jj abandon REVISION
```

**重要：**
- コミット自体は削除されますが、変更内容は子コミットに引き継がれます
- 「コミットをスキップする」イメージ

### 例

```bash
jj new
echo "unwanted" > unwanted.txt
jj commit -m "不要なコミット"

jj new
echo "important" > important.txt
jj commit -m "重要なコミット"

# 不要なコミットを削除
jj abandon @--
```

「不要なコミット」は削除されますが、`unwanted.txt`の内容は「重要なコミット」に引き継がれます。

---

## 9. 複数コミットを同時に操作

### 範囲指定

```bash
# @からさかのぼって3つのコミット
jj log -r "@::@---"

# 特定のブランチまでのすべてのコミット
jj log -r "branch1::branch2"
```

### 複数のコミットを削除

```bash
jj abandon "start::end"
```

---

## 10. 操作の取り消し（`jj op undo`）

間違った操作をしてしまった場合、元に戻せます！

### 操作履歴を確認

```bash
jj op log
```

**期待される出力：**

```
@  12ab34cd taro@example.com 2025-10-23 11:30:45
│  abandon commit
│  args: jj abandon abc123
◉  56ef78gh taro@example.com 2025-10-23 11:29:22
│  commit f5e7d6c8abc123
│  args: jj commit -m "重要なコミット"
◉  90ij12kl taro@example.com 2025-10-23 11:28:10
│  commit 8e9f0a1bdef456
   args: jj commit -m "不要なコミット"
```

### 直前の操作を取り消し

```bash
jj op undo
```

最後の操作が取り消されます！

### 特定の操作まで戻る

```bash
jj op restore OPERATION_ID
```

---

## やってみよう！

以下の練習問題に挑戦してください：

### 練習1：コミットメッセージの修正

1. 3つのコミットを作成（意図的に誤字を入れる）
2. それぞれのメッセージを修正
3. `jj log`で確認

### 練習2：コミットの統合

1. 5つの小さなコミットを作成
2. 2番目と3番目のコミットを統合
3. さらに4番目と5番目を統合
4. 履歴が3つのコミットになっていることを確認

### 練習3：コミットの分割

1. 3つの異なるファイルを1つのコミットに含める
2. コミットを3つに分割
3. それぞれに適切なメッセージを付ける

### 練習4：リベース

1. `main`ブランチに2つのコミット
2. `feature`ブランチを1つ目のコミットから作成
3. `feature`を2つ目のコミットにリベース
4. 履歴を可視化して確認

### 練習5：操作の取り消し

1. いくつかの操作を実行
2. `jj op log`で履歴を確認
3. `jj op undo`で戻す
4. もう一度`jj op log`で確認

---

## トラブルシューティング

### change IDが見つからない

```bash
jj log --limit 20
```

で最近のコミットを表示

### リベース後にコンフリクト

```bash
jj status
```

でコンフリクトを確認し、前のセクションの方法で解決

### 操作を間違えた

```bash
jj op undo
```

で即座に元に戻す！

### 複雑な履歴で混乱

```bash
jj log --graph --color always | less -R
```

でグラフィカルな履歴を表示

---

## 重要な概念のまとめ

### jujutsuの履歴編集の利点

1. **安全**：すべての操作は`jj op log`に記録され、元に戻せる
2. **柔軟**：コミット後でも自由に編集可能
3. **強力**：複雑な履歴の整理が簡単

### 主要コマンド

| コマンド | 用途 |
|---------|------|
| `jj describe` | コミットメッセージを変更 |
| `jj edit` | 過去のコミットを編集 |
| `jj squash` | コミットを統合 |
| `jj split` | コミットを分割 |
| `jj rebase` | コミットの親を変更 |
| `jj abandon` | コミットを削除 |
| `jj op undo` | 操作を取り消し |
| `jj op restore` | 特定の状態に復元 |

### change IDの重要性

- 履歴を編集しても、change IDで同じコミットを追跡できる
- コミットハッシュは変わるが、change IDは変わらない
- コラボレーション時に非常に有用

---

## ベストプラクティス

### 履歴を整理するタイミング

1. **ローカルで作業中**：自由に編集
2. **プッシュ前**：履歴をクリーンアップ
3. **プッシュ後**：他の人に影響を与える可能性があるため慎重に

### クリーンな履歴のために

- コミットメッセージは明確に
- 1つのコミット = 1つの論理的な変更
- 関連する変更は`squash`で統合
- 無関係な変更は`split`で分割
- 意味のある順序にリベース

### 安全に実験する

```bash
jj op log
```

を実行して、いつでも戻れることを確認してから大胆に実験しましょう！

---

## チェックリスト

次のセクションに進む前に、以下を確認してください：

- [ ] `jj describe`でメッセージを変更できた
- [ ] `jj edit`で過去のコミットを編集できた
- [ ] `jj squash`でコミットを統合できた
- [ ] `jj split`でコミットを分割できた
- [ ] `jj rebase`でコミットを移動できた
- [ ] `jj abandon`でコミットを削除できた
- [ ] `jj op log`で操作履歴を確認できた
- [ ] `jj op undo`で操作を取り消せた

---

## 次のステップ

履歴編集の技術をマスターしました！次は [05-advanced.md](05-advanced.md) で、jujutsuの高度な機能を学びましょう。

- change IDの深い理解
- working copyの仕組み
- revset言語
- 操作履歴の活用

jujutsuの全機能を使いこなせるようになります！🚀

