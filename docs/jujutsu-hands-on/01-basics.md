# 01. 基本操作：jujutsuの第一歩

このセクションでは、jujutsuの基本的な操作を実際に手を動かして学びます。

## 学習目標

- リポジトリを作成する
- ファイルを追加してコミットする
- コミット履歴を確認する
- 差分を確認する
- jujutsuの基本概念を理解する

---

## 準備：練習用ディレクトリの作成

まず、練習用のディレクトリを作成しましょう。

```bash
cd ~/Desktop
mkdir my-first-jj
cd my-first-jj
```

**現在のディレクトリを確認：**

```bash
pwd
```

`/Users/あなたの名前/Desktop/my-first-jj` のようなパスが表示されます。

---

## 1. リポジトリの初期化

jujutsuでバージョン管理を始めるには、リポジトリを初期化します。

```bash
jj git init
```

**期待される出力：**

```
Initialized repo in "."
```

### 💡 ポイント：`jj git init` コマンド

- `jj git init`で、Gitとの互換性を保ったリポジトリが作成されます
- 既存のGitツール（GitHub、GitLabなど）との連携が可能になります
- エイリアスを設定すれば`jj init`でも動作します（後述）

### 何が起きた？

```bash
ls -la
```

`.jj` ディレクトリと `.git` ディレクトリが作成されているはずです！

---

## 2. 最初のファイルを作成

簡単なファイルを作成しましょう。

```bash
echo "# My First Jujutsu Project" > README.md
```

ファイルが作成されたか確認：

```bash
cat README.md
```

**出力：**

```
# My First Jujutsu Project
```

---

## 3. 状態を確認する（重要！）

jujutsuでは、現在の状態を確認することが非常に重要です。

```bash
jj status
```

**期待される出力：**

```
Working copy changes:
A README.md
Working copy : qpvuntsm 614c49af (no description set)
Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```

### 📖 出力の読み方

- `A README.md`：README.mdが追加された（Added）
- `Working copy`：現在作業中のコミット
- `qpvuntsm`：コミットのchange ID（短縮版）
- `614c49af`：コミットハッシュ（短縮版）
- `(no description set)`：まだ説明が設定されていない

---

## 4. 変更を確認する

ファイルの内容を詳しく見てみましょう。

```bash
jj diff
```

**期待される出力：**

```diff
Added regular file README.md:
        1: # My First Jujutsu Project
```

これは「README.mdという新しいファイルが追加され、1行のテキストが含まれている」ことを示しています。

---

## 5. コミットを作成する

jujutsuでは、変更は自動的に「working copy」に記録されています。ここで明示的にコミットを確定します。

```bash
jj commit -m "最初のコミット：READMEを追加"
```

**期待される出力：**

```
Working copy now at: rlvkpnrz 20c2e23e (empty) (no description set)
Parent commit      : qpvuntsm f5e7d6c8 最初のコミット：READMEを追加
```

### 💡 重要：jujutsuのコミットの仕組み

- コミット後、**自動的に新しい空のworking copyが作成されます**
- これはjujutsuの大きな特徴です！
- 常にクリーンな環境で作業を続けられます

---

## 6. 履歴を確認する

コミット履歴を見てみましょう。

```bash
jj log
```

**期待される出力：**

```
@  rlvkpnrz taro@example.com 2025-10-23 10:23:45 20c2e23e
│  (empty) (no description set)
◉  qpvuntsm taro@example.com 2025-10-23 10:22:11 f5e7d6c8
│  最初のコミット：READMEを追加
◉  zzzzzzzz root() 00000000
   (empty) (no description set)
```

### 📖 ログの読み方

- `@`：現在の作業位置（working copy）
- `◉`：コミット
- `│`：親子関係を示す線
- 右側の英数字（`20c2e23e`など）：コミットハッシュ
- `root()`：リポジトリのルートコミット

---

## 7. もう一つファイルを追加してみよう

### ファイルを作成

```bash
echo "print('Hello, Jujutsu!')" > hello.py
```

### 状態を確認

```bash
jj status
```

**出力：**

```
Working copy changes:
A hello.py
Working copy : rlvkpnrz 20c2e23e (no description set)
Parent commit: qpvuntsm f5e7d6c8 最初のコミット：READMEを追加
```

### 差分を確認

```bash
jj diff
```

### コミット

```bash
jj commit -m "Pythonスクリプトを追加"
```

### 履歴を確認

```bash
jj log
```

**期待される出力：**

```
@  sqvxwpyz taro@example.com 2025-10-23 10:25:33 3a2b1c0d
│  (empty) (no description set)
◉  rlvkpnrz taro@example.com 2025-10-23 10:24:50 8e9f0a1b
│  Pythonスクリプトを追加
◉  qpvuntsm taro@example.com 2025-10-23 10:22:11 f5e7d6c8
│  最初のコミット：READMEを追加
◉  zzzzzzzz root() 00000000
   (empty) (no description set)
```

綺麗な直線の履歴が作成されました！

---

## 8. Gitとjujutsuの違い

### Gitの場合

```
作業 → git add → git commit
```

- `git add`でステージングエリアに追加
- `git commit`でコミット作成

### jujutsuの場合

```
作業 → jj commit
```

- **ステージングエリアがない！**
- すべての変更は自動的に追跡される
- コミット後、自動的に新しいworking copyが作成される

### working copyとは？

- jujutsuでは、現在作業中の状態も「コミット」として扱われます
- これを**working copy**と呼びます
- `@`マークで表示されます
- この仕組みにより、いつでも変更を元に戻せます

---

## やってみよう！

以下の練習問題に挑戦してください：

### 練習1：新しいファイルを追加

1. `notes.txt`というファイルを作成し、好きな内容を書く
2. `jj status`で状態を確認
3. `jj diff`で変更内容を確認
4. `jj commit -m "メモファイルを追加"`でコミット
5. `jj log`で履歴を確認

### 練習2：複数のファイルを同時にコミット

1. `file1.txt`と`file2.txt`を作成
2. 両方のファイルに内容を書く
3. `jj status`で両方が追跡されていることを確認
4. 一度のコミットで両方をコミット
5. `jj log`で履歴を確認

### 練習3：既存ファイルの編集

1. `README.md`を開いて、内容を追加
2. `jj diff`で変更を確認（`M README.md`と表示される）
3. コミットして履歴を確認

---

## トラブルシューティング

### エラー：「No repo found」

```
Error: There is no jj repo in "."
```

**原因：** jujutsuリポジトリではないディレクトリで実行している

**解決方法：** `jj git init`を実行してリポジトリを初期化

### working copyが表示されない

**解決方法：** `jj log`の代わりに`jj log -r 'all()'`を試す

### コミットメッセージを間違えた

心配無用です！次のセクションで修正方法を学びます。

---

## 重要な概念のまとめ

### 1. change ID

- 各コミットには一意のchange IDが付けられます
- 例：`qpvuntsm`
- コミットの内容が同じなら、change IDも同じ
- 履歴を編集してもchange IDで追跡可能

### 2. working copy

- 現在作業中の状態
- `@`マークで表示
- それ自体もコミットとして扱われる
- コミット後、自動的に新しいworking copyが作成される

### 3. ステージングレス

- Gitの`git add`に相当する操作が不要
- すべての変更は自動的に追跡される
- よりシンプルなワークフロー

---

## チェックリスト

次のセクションに進む前に、以下を確認してください：

- [ ] `jj git init`でリポジトリを作成できた
- [ ] `jj status`で現在の状態を確認できた
- [ ] ファイルを作成してコミットできた
- [ ] `jj log`でコミット履歴を確認できた
- [ ] `jj diff`で変更内容を確認できた
- [ ] working copyの概念を理解した
- [ ] 3つ以上のコミットを作成した

---

## 次のステップ

基本操作をマスターしました！次は [02-branches.md](02-branches.md) で、ブランチの操作方法を学びましょう。

- ブランチの作成
- ブランチの切り替え
- 複数のブランチでの作業

お疲れ様でした！🎉

