# 02. ブランチ操作：並行作業を始めよう

このセクションでは、ブランチを使って複数の作業を並行して進める方法を学びます。

## 学習目標

- ブランチとは何かを理解する
- 新しいコミットを作成する方法を学ぶ
- ブランチを切り替える
- 過去のコミットに戻る
- 複数のブランチで作業する

---

## 準備：前のセクションの続きから

前のセクションで作成した`my-first-jj`ディレクトリを引き続き使います。

```bash
cd ~/Desktop/my-first-jj
```

現在の状態を確認：

```bash
jj log
```

---

## 1. ブランチとは？

### バージョン管理におけるブランチ

- **ブランチ**：開発の「枝分かれ」
- 複数の機能を並行して開発できる
- メインの作業に影響を与えずに実験できる

### jujutsuのブランチの特徴

- **軽量**：ブランチの作成・切り替えが高速
- **柔軟**：後からいくらでも変更・整理可能
- **Gitと互換**：Gitのブランチとしてpush可能

---

## 2. 新しいコミットを作成する（`jj new`）

jujutsuでは、`jj new`コマンドで新しいコミット（working copy）を作成します。

### 基本的な使い方

```bash
jj new
```

**期待される出力：**

```
Working copy now at: pqzwxryv 7d8e9f0a (empty) (no description set)
Parent commit      : sqvxwpyz 3a2b1c0d (empty) (no description set)
```

新しい空のコミットが現在のコミットの上に作成されました！

### 履歴を確認

```bash
jj log
```

**期待される出力：**

```
@  pqzwxryv taro@example.com 2025-10-23 10:30:15 7d8e9f0a
│  (empty) (no description set)
◉  sqvxwpyz taro@example.com 2025-10-23 10:25:33 3a2b1c0d
│  (empty) (no description set)
◉  rlvkpnrz taro@example.com 2025-10-23 10:24:50 8e9f0a1b
│  Pythonスクリプトを追加
◉  qpvuntsm taro@example.com 2025-10-23 10:22:11 f5e7d6c8
│  最初のコミット：READMEを追加
◉  zzzzzzzz root() 00000000
   (empty) (no description set)
```

---

## 3. 新機能の開発を開始する

新しい機能を開発するシナリオで練習しましょう。

### シナリオ：数学関数を追加する

```bash
echo "def add(a, b):
    return a + b

def subtract(a, b):
    return a - b" > math_utils.py
```

### 状態を確認

```bash
jj status
```

### コミット

```bash
jj commit -m "数学ユーティリティ関数を追加"
```

---

## 4. ブランチに名前を付ける

現在のコミットにブランチ名を付けましょう。

```bash
jj branch create feature/math-utils
```

**期待される出力：**

```
Created branch feature/math-utils pointing to pqzwxryv
```

### ブランチの一覧を確認

```bash
jj branch list
```

**期待される出力：**

```
feature/math-utils: pqzwxryv 2f3g4h5i 数学ユーティリティ関数を追加
```

---

## 5. 過去のコミットに戻る（`jj edit`）

過去のコミットに戻って、そこから別の作業を始めることができます。

### 特定のコミットを編集

まず、現在の履歴を確認：

```bash
jj log
```

README.mdを追加したコミット（`qpvuntsm`）に戻りましょう：

```bash
jj edit qpvuntsm
```

**💡 Tip:** change IDは最初の数文字だけでOKです。例：`jj edit qpv`

**期待される出力：**

```
Working copy now at: qpvuntsm f5e7d6c8 最初のコミット：READMEを追加
Parent commit      : zzzzzzzz 00000000 (empty) (no description set)
Added 2 files, modified 0 files, removed 0 files
```

### 現在のファイルを確認

```bash
ls
```

**出力：**

```
README.md
```

`hello.py`や`math_utils.py`が消えました！これは正常です。過去の時点に戻ったためです。

---

## 6. 新しいブランチを作成

この時点から別の機能を開発します。

### 新しいコミットを作成

```bash
jj new
```

### ドキュメントを追加

```bash
echo "# プロジェクトの使い方

このプロジェクトは練習用です。

## インストール方法

特に必要ありません。

## 使い方

ファイルを編集して学びましょう！" > USAGE.md
```

### コミット

```bash
jj commit -m "使い方のドキュメントを追加"
```

### ブランチ名を付ける

```bash
jj branch create feature/docs
```

---

## 7. 履歴を確認する

ここで履歴を見てみましょう：

```bash
jj log
```

**期待される出力：**

```
@  xyzabc12 taro@example.com 2025-10-23 10:35:22 9i8h7g6f
│  (empty) (no description set)
│ ◉  pqzwxryv taro@example.com 2025-10-23 10:30:55 2f3g4h5i feature/math-utils
│ │  数学ユーティリティ関数を追加
│ ◉  sqvxwpyz taro@example.com 2025-10-23 10:25:33 3a2b1c0d
│ │  (empty) (no description set)
│ ◉  rlvkpnrz taro@example.com 2025-10-23 10:24:50 8e9f0a1b
│ │  Pythonスクリプトを追加
├─╯
◉  qpvuntsm taro@example.com 2025-10-23 10:22:11 f5e7d6c8 feature/docs
│  最初のコミット：READMEを追加  
◉  zzzzzzzz root() 00000000
   (empty) (no description set)
```

### 📖 履歴の読み方

履歴が**枝分かれ**しています！

- 左側：`feature/math-utils`ブランチ（数学関数）
- 右側：`feature/docs`ブランチ（ドキュメント）
- 両方とも「最初のコミット：READMEを追加」から分岐

---

## 8. ブランチ間を移動する

### 別のブランチに切り替え

`feature/math-utils`ブランチに切り替えましょう：

```bash
jj edit feature/math-utils
```

### ファイルを確認

```bash
ls
```

**出力：**

```
README.md  hello.py  math_utils.py
```

`USAGE.md`はありません。これは`feature/docs`ブランチにしか存在しないからです。

### もう一度切り替え

```bash
jj edit feature/docs
ls
```

**出力：**

```
README.md  USAGE.md
```

今度は`hello.py`や`math_utils.py`がありません！

---

## 9. 新しいコミットを特定の親から作成

特定のコミットを親として新しいコミットを作成できます。

### 数学ブランチから新しいコミット

```bash
jj new feature/math-utils
```

**期待される出力：**

```
Working copy now at: mnopqrst 5j6k7l8m (empty) (no description set)
Parent commit      : pqzwxryv 2f3g4h5i feature/math-utils 数学ユーティリティ関数を追加
```

### テストファイルを追加

```bash
echo "from math_utils import add, subtract

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0

def test_subtract():
    assert subtract(5, 3) == 2
    assert subtract(0, 0) == 0

print('All tests passed!')" > test_math.py
```

### コミット

```bash
jj commit -m "数学関数のテストを追加"
```

---

## 10. 現在地を確認する重要なコマンド

### `jj log`のオプション

より見やすい表示：

```bash
jj log -r 'all()' --limit 10
```

### 現在のworking copyだけを確認

```bash
jj log -r @
```

### 特定のブランチだけを確認

```bash
jj log -r feature/math-utils
```

---

## やってみよう！

以下の練習問題に挑戦してください：

### 練習1：新しいブランチを作成

1. ルートコミットに戻る：`jj edit root()`
2. 新しいコミットを作成：`jj new`
3. `.gitignore`ファイルを作成
4. コミットして`feature/gitignore`ブランチを作成

### 練習2：ブランチを行き来する

1. `jj branch list`で全ブランチを確認
2. それぞれのブランチに`jj edit`で移動
3. 各ブランチで`ls`を実行してファイルを確認
4. どのブランチにどのファイルがあるか理解する

### 練習3：履歴を可視化

1. `jj log`で全体の履歴を確認
2. 枝分かれの構造を理解する
3. 各ブランチがどこから分岐しているか確認

---

## トラブルシューティング

### change IDが分からない

```bash
jj log
```

で履歴を確認し、左側の英数字（例：`qpvuntsm`）を使用

### ブランチが見つからない

```bash
jj branch list
```

で全ブランチを確認

### 変更が消えた？

心配しないでください！jujutsuでは変更は失われません。

```bash
jj op log
```

で操作履歴を確認し、必要なら元に戻せます（次のセクションで学習）。

---

## 重要な概念のまとめ

### `jj new` vs `jj edit`

| コマンド | 動作 |
|---------|------|
| `jj new` | 新しいコミットを作成して移動 |
| `jj new REVISION` | 指定したコミットの子として新しいコミットを作成 |
| `jj edit REVISION` | 既存のコミットに移動して編集 |

### change IDの指定方法

- 完全なchange ID：`qpvuntsm`
- 短縮版：`qpv`（一意に識別できる最小の文字数）
- ブランチ名：`feature/math-utils`
- 特殊な記号：
  - `@`：現在のworking copy
  - `@-`：親コミット
  - `@+`：子コミット

### ブランチの役割

- コミットに人間が読みやすい名前を付ける
- 複数の作業を整理する
- Gitリポジトリとの連携に使う

---

## チェックリスト

次のセクションに進む前に、以下を確認してください：

- [ ] `jj new`で新しいコミットを作成できた
- [ ] `jj edit`で過去のコミットに移動できた
- [ ] `jj branch create`でブランチを作成できた
- [ ] `jj branch list`でブランチ一覧を確認できた
- [ ] 複数のブランチを行き来できた
- [ ] `jj log`で枝分かれした履歴を確認できた
- [ ] change IDの指定方法を理解した

---

## 次のステップ

ブランチ操作をマスターしました！次は [03-merging.md](03-merging.md) で、ブランチをマージする方法を学びましょう。

- 変更をマージする
- コンフリクトを解決する
- 複雑な履歴を扱う

準備ができたら次に進みましょう！🚀

