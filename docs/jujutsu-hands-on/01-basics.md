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

## 5.5. `jj commit`の内部動作を理解する

### `jj commit`は実はショートカット！

`jj commit`は、内部的に2つのコマンドを実行するショートカットです：

```bash
# この2つのコマンドは...
jj describe -m "機能を実装"
jj new

# ↓ これと同じ！
jj commit -m "機能を実装"
```

### 各コマンドの役割

| コマンド | 役割 | 効果 |
|---------|------|------|
| `jj describe` | コミットメッセージを設定 | 現在の変更に説明を付ける |
| `jj new` | 新しいworking copyを作成 | 次の作業を開始する準備 |
| `jj commit` | 上記2つを実行 | 変更を確定して次へ進む |

### 使い分けのポイント

#### ✅ `jj commit`を使うべき時

**「この変更を確定して、次の作業に移りたい」時**

```bash
# 機能Aを実装した
vim feature_a.py

# 確定して次へ
jj commit -m "機能A完了"

# ← 自動的に新しい空のworking copyが作成される
# 次の作業を開始できる
vim feature_b.py
```

このワークフローはGitに最も近い使い方で、Git経験者には馴染みやすいです。

#### ❌ `jj commit`を使わない時

**1. 変更を続けて編集したい時**

```bash
# ❌ 間違い：commitすると新しい変更が作られる
jj commit -m "WIP"
# ← 新しい変更が作られてしまう

# ✅ 正しい：describeだけで説明を更新
jj describe -m "実装中"
# そのまま編集を続けられる
```

**2. スカッシュワークフロー（後述）を使っている時**

```bash
# スカッシュワークフローでは commit は使わない
jj describe -m "機能X"
jj new
# ... 作業 ...
jj squash  # 親にスカッシュ（詳細は04-history-editing.mdへ）
```

### 💡 初心者へのアドバイス

- **最初は`jj commit`を使う**：Gitと同じ感覚で使えます
- **慣れてきたら**：`describe`と`new`を使い分けることで、より柔軟な作業が可能に
- **どちらも正解**：プロジェクトやチームの方針に合わせて選択できます

---

## 5.6. `jj new -m` vs `jj commit -m`：最も混乱しやすいポイント

### 決定的な違い

```bash
# jj new -m："これから作る変更"の入れ物
jj new -m "ログイン機能"
# ↓ 空の変更が作られる
# ここからコードを書く

# jj commit -m："今ある変更"を確定
# ... コードを書く ...
jj commit -m "ログイン機能"
# ↓ 今の変更が確定 + 新しい空の変更が作られる
```

### 考え方の違い

#### `jj new -m`：**先に宣言**

```bash
jj new -m "機能A"        # 「これからAを作ります」と宣言
# ... 実装 ...           # 自動的に記録される

jj new -m "機能B"        # 「次はBを作ります」と宣言  
# ... 実装 ...
```

**イメージ：** ノートに見出しを先に書いてから内容を書く

#### `jj commit -m`：**後でまとめる**

```bash
# ... 実装 ...           # まず作業
jj commit -m "機能A"     # 「今のはAでした」と確定

# ... 実装 ...
jj commit -m "機能B"     # 「今のはBでした」と確定
```

**イメージ：** 作業してから「これは○○だった」とラベルを付ける（Git風）

---

### 実践的な使い分け基準

#### ✅ `jj new -m`を使う時

**「これから何を作るか明確」な時**

```bash
# タスクが明確
jj new -m "ユーザー認証API実装"
jj new -m "バグ #123 を修正"
jj new -m "テストカバレッジ向上"

# → 目的駆動で開発できる
```

**メリット：**
- 目的が明確になる
- タスク管理がしやすい
- 履歴を見たときに何をしようとしていたかわかりやすい
- Issue番号などを先に書ける

#### ✅ `jj commit -m`を使う時

**「探索的に作業した後」**

```bash
# いろいろ試した
# ... あれこれコード修正 ...

# 結果を見てから名前を付ける
jj commit -m "パフォーマンス改善"

# 次も探索
# ... さらに実験 ...
jj commit -m "キャッシュ戦略の変更"
```

**メリット：**
- Git経験者に馴染みやすい
- 柔軟に試行錯誤できる
- 作業内容が明確になってから命名できる
- 予期しない展開にも対応しやすい

---

### 実際のワークフロー例

#### シナリオ1：計画的な開発

```bash
# Issue見て作業内容が明確
jj new -m "Issue #456: ログイン画面にパスワード表示切替"
# ... 実装（15分）...

jj new -m "Issue #457: エラーメッセージの多言語対応"
# ... 実装（30分）...

# ✅ 各変更が明確で履歴がきれい
```

**履歴：**
```
@  (empty)
◉  Issue #457: エラーメッセージの多言語対応
◉  Issue #456: ログイン画面にパスワード表示切替
◉  main
```

#### シナリオ2：試行錯誤しながら開発

```bash
# とりあえず作業開始
jj new
# ... いろいろ試す（1時間）...

# 落ち着いたところで確定
jj commit -m "リファクタリング: 認証ロジックを分離"

# また試す
# ... さらに試行錯誤 ...
jj commit -m "バリデーション処理を改善"

# ✅ 柔軟に作業できる
```

**履歴：**
```
@  (empty)
◉  バリデーション処理を改善
◉  リファクタリング: 認証ロジックを分離
◉  main
```

#### シナリオ3：混合スタイル（おすすめ）

```bash
# 大まかに宣言
jj new -m "決済機能のリファクタリング"

# 細かく分割していく
# ... 一部実装 ...
jj split  # 一部を分割
jj describe -m "データベース層の分離"

# 残りも分割
jj split
jj describe -m "API層の更新"

# ✅ 柔軟かつ整理された履歴
```

---

### 🤔 迷った時の判断フロー

```
作業を始める前に目的が明確？
├─ YES → jj new -m "目的"
└─ NO  → jj new (説明なし)
           ↓
        作業後に jj describe -m "内容"
        または jj commit -m "内容"
```

**簡単な質問：**
- 「今から何をするか説明できる？」→ YES なら `jj new -m`
- 「とりあえず始めてみる？」→ YES なら `jj new` → 後で `jj describe/commit`

---

### 🎯 チーム導入時の推奨

#### 初心者向け：`jj commit -m`（Git風）

```bash
# Gitからの移行者に優しい
# ... 作業 ...
jj commit -m "変更内容"

# 慣れたやり方で始められる
```

**理由：** Git経験者にとって最も自然で、学習コストが低い

#### 中級者向け：`jj new -m`（Jujutsu流）

```bash
# 目的を明確にしてから実装
jj new -m "目的"
# ... 作業 ...

# より計画的な開発
```

**理由：** jujutsuの特徴を活かした、より計画的な開発が可能

#### 上級者向け：混合スタイル

```bash
# 状況に応じて使い分け
jj new -m "明確なタスク"    # 計画的な部分
# ...

jj new                      # 探索的な部分
# ...
jj commit -m "試行錯誤の結果"
```

**理由：** 柔軟性と明確さのバランスが取れる

---

### 💡 個人的な推奨（チーム導入時）

1. **最初は`jj commit`で教える**：Git経験者に優しい
2. **慣れたら`jj new -m`を紹介**：より計画的に
3. **最終的には混合**：状況に応じて使い分け

### 重要なポイント

実は、**どちらでも問題ありません！** 重要なのは：

- **`jj new -m` = 先に宣言**（目的駆動）
- **`jj commit -m` = 後でまとめる**（結果駆動）

この違いを理解していれば、自然に使い分けられるようになります。

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

