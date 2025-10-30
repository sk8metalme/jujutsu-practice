# 補足：jj resolveの使い方完全ガイド

`jj resolve`は、コンフリクト（競合）を対話的に解決するための強力なツールです。このガイドでは、実践的な使い方を学びます。

## 目次

1. [コンフリクト解決の基本](#コンフリクト解決の基本)
2. [実践：コンフリクトの作成](#実践コンフリクトの作成)
3. [jj resolveの使い方](#jj-resolveの使い方)
4. [画面の見方と操作](#画面の見方と操作)
5. [実践的な解決手順](#実践的な解決手順)
6. [手動編集との比較](#手動編集との比較)
7. [よくあるシナリオ](#よくあるシナリオ)

---

## コンフリクト解決の基本

### コンフリクトが発生する条件

- 同じファイルの同じ箇所を異なる方法で変更
- 2つ以上のブランチをマージする際に発生

### 解決方法は2つ

1. **`jj resolve`コマンドを使う**（推奨）：対話的な3ペインエディタ
2. **手動でファイルを編集**：シンプルだが、複雑なコンフリクトには不向き

---

## 実践：コンフリクトの作成

まず、練習用のコンフリクトを作成します。

### ステップ1：準備

```bash
# 新しいディレクトリで練習
cd ~/Desktop
mkdir jj-resolve-practice
cd jj-resolve-practice
jj git init
```

### ステップ2：ベースファイルを作成

```bash
echo "タイトル：未定
内容：これから書きます
作者：山田太郎
日付：2025年1月" > document.txt

jj commit -m "初期ドキュメント"
jj bookmark create base -r '@-'
```

**💡 ポイント：** コミット後に`-r @-`でブックマークを作成しています。

### ステップ3：ブランチ1を作成

```bash
jj new base

echo "タイトル：技術ドキュメント
内容：これから書きます
作者：山田太郎
日付：2025年1月" > document.txt

jj commit -m "タイトルを技術ドキュメントに変更"
jj bookmark create branch-a -r '@-'
```

**💡 ポイント：** コミット後に`-r @-`でブックマークを作成しています。

### ステップ4：ブランチ2を作成

```bash
jj new base

echo "タイトル：製品仕様書
内容：これから書きます
作者：山田太郎
日付：2025年1月" > document.txt

jj commit -m "タイトルを製品仕様書に変更"
jj bookmark create branch-b -r '@-'
```

### ステップ5：マージしてコンフリクトを発生

```bash
jj new branch-a branch-b -m "ブランチをマージ"
```

**期待される出力：**

```
Working copy now at: pqrstuv 9h8g7f6e (conflict) ブランチをマージ
Parent commit      : abc123 branch-a
Parent commit      : def456 branch-b
New conflicts appeared in these commits:
  pqrstuv 9h8g7f6e (conflict) ブランチをマージ
To resolve the conflicts, start by updating to it:
  jj new pqrstuv
Then use `jj resolve` to resolve the conflicts.
```

### ステップ6：コンフリクトを確認

```bash
jj status
```

**出力：**

```
Working copy changes:
C document.txt
Working copy : pqrstuv 9h8g7f6e (conflict) ブランチをマージ
Parent commit: abc123 branch-a
Parent commit: def456 branch-b
```

`C document.txt`：Conflict（コンフリクト）が発生しています！

---

## jj resolveの使い方

### 基本コマンド

```bash
jj resolve
```

これで対話的なコンフリクト解決が始まります。

### 起動時の画面

```
Conflict in document.txt:
1: [l] diff base left
2: [r] diff base right  
3: [o] diff left right
4: [e] edit sides
5: [s] snapshot merge

Select an option (or q to quit):
```

---

## 画面の見方と操作

### オプション一覧

| 番号 | キー | コマンド | 説明 |
|------|------|----------|------|
| 1 | `l` | diff base left | ベースとleft（最初のブランチ）の差分を表示 |
| 2 | `r` | diff base right | ベースとright（2番目のブランチ）の差分を表示 |
| 3 | `o` | diff left right | leftとrightの差分を表示 |
| 4 | **`e`** | **edit sides** | **3ペインエディタで編集（最もよく使う）** |
| 5 | `s` | snapshot merge | 現在のマージ状態を保存 |
| - | `q` | quit | 終了（変更を保存せずに） |

### 用語の説明

- **Base**（ベース）：共通の祖先（両方のブランチが分岐する前のコミット）
- **Left**：最初の親コミット（branch-a）
- **Right**：2番目の親コミット（branch-b）
- **Result**（結果）：最終的なマージ結果（あなたが編集する部分）

---

## 実践的な解決手順

### ステップ1：差分を確認（オプション）

まず、どこが違うのか確認します：

```
Select an option: l
```

**Base vs Left の差分が表示されます：**

```diff
- タイトル：未定
+ タイトル：技術ドキュメント
```

次にRightも確認：

```
Select an option: r
```

**Base vs Right の差分が表示されます：**

```diff
- タイトル：未定
+ タイトル：製品仕様書
```

### ステップ2：エディタで編集

```
Select an option: e
```

**3ペインエディタが開きます！**

```
┌─────────────────────┬─────────────────────┬─────────────────────┐
│      Base           │       Left          │       Right         │
│   （共通の祖先）      │   （branch-a）       │   （branch-b）       │
│                     │                     │                     │
│ タイトル：未定        │ タイトル：技術ドキュ  │ タイトル：製品仕様書  │
│ 内容：これから書き... │ 内容：これから書き... │ 内容：これから書き... │
│ 作者：山田太郎        │ 作者：山田太郎        │ 作者：山田太郎        │
│ 日付：2025年1月      │ 日付：2025年1月      │ 日付：2025年1月      │
├─────────────────────┴─────────────────────┴─────────────────────┤
│                        Result（結果）                            │
│                     ↓ここを編集する↓                              │
│                                                                  │
│ <<<<<<<                                                          │
│ %%%%%%%                                                          │
│ タイトル：未定                                                     │
│ +++++++                                                          │
│ タイトル：技術ドキュメント                                          │
│ >>>>>>>                                                          │
│ <<<<<<<                                                          │
│ %%%%%%%                                                          │
│ タイトル：未定                                                     │
│ +++++++                                                          │
│ タイトル：製品仕様書                                               │
│ >>>>>>>                                                          │
│ 内容：これから書きます                                             │
│ 作者：山田太郎                                                     │
│ 日付：2025年1月                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### ステップ3：結果を編集

下部の**Result**ペインで、最終的な内容を決定します。

#### パターン1：両方を統合

```
タイトル：技術ドキュメント（製品仕様書）
内容：これから書きます
作者：山田太郎
日付：2025年1月
```

#### パターン2：Leftを採用

```
タイトル：技術ドキュメント
内容：これから書きます
作者：山田太郎
日付：2025年1月
```

#### パターン3：Rightを採用

```
タイトル：製品仕様書
内容：これから書きます
作者：山田太郎
日付：2025年1月
```

#### パターン4：新しい内容

```
タイトル：統合ドキュメント
内容：これから書きます
作者：山田太郎
日付：2025年1月
```

### ステップ4：保存して終了

エディタによって操作が異なります：

- **nano**: `Ctrl+O`（保存）→ `Ctrl+X`（終了）
- **vim**: `:wq`
- **VS Code**: 保存してウィンドウを閉じる

### ステップ5：確認

```bash
jj status
```

**出力：**

```
Working copy : pqrstuv 9h8g7f6e ブランチをマージ
Parent commit: abc123 branch-a
Parent commit: def456 branch-b
```

`C document.txt`が消えていれば成功！✅

```bash
# 結果を確認
cat document.txt
```

---

## 手動編集との比較

### 方法1：jj resolve（推奨）

**メリット：**
- ✅ Base、Left、Rightを並べて表示
- ✅ どこが違うのか一目瞭然
- ✅ 複雑なコンフリクトも対応しやすい
- ✅ 差分を確認してから編集できる

**デメリット：**
- ❌ エディタの設定が必要
- ❌ 初めは操作に慣れが必要

### 方法2：手動編集

```bash
# ファイルを直接開く
nano document.txt
```

**コンフリクトマーカー：**

```
<<<<<<<
%%%%%%%
タイトル：未定
+++++++
タイトル：技術ドキュメント
>>>>>>>
<<<<<<<
%%%%%%%
タイトル：未定
+++++++
タイトル：製品仕様書
>>>>>>>
内容：これから書きます
作者：山田太郎
日付：2025年1月
```

マーカーを削除して編集：

```
タイトル：統合ドキュメント
内容：これから書きます
作者：山田太郎
日付：2025年1月
```

**メリット：**
- ✅ シンプル
- ✅ すぐに編集できる

**デメリット：**
- ❌ マーカーの意味を理解する必要
- ❌ 複雑なコンフリクトは見にくい
- ❌ BaseとLeftとRightの区別が分かりにくい

---

## よくあるシナリオ

### シナリオ1：単純な選択

**状況：** どちらか一方を採用したい

**操作：**
1. `jj resolve` → `e`
2. 上部のLeftまたはRightペインの内容をコピー
3. 下部のResultペインに貼り付け
4. 保存して終了

### シナリオ2：部分的に統合

**状況：** 両方の変更を取り入れたい

**操作：**
1. `jj resolve` → `e`
2. LeftとRightの内容を確認
3. Resultペインで両方を組み合わせた内容を記述
4. 保存して終了

### シナリオ3：複数ファイルのコンフリクト

```bash
# 1つ目のファイルを解決
jj resolve
# e を選択して編集

# 2つ目のファイルを解決
jj resolve
# 再度 e を選択して編集

# すべて解決されたか確認
jj status
```

### シナリオ4：コンフリクト解決を中断

```bash
# 解決を中止して元に戻す
jj abandon '@'
```

### シナリオ5：すべてのコンフリクトを一度に確認

```bash
jj resolve --all
```

---

## エディタの設定

### VS Codeを使う（推奨）

```bash
jj config set --user merge-tools.code.program "code"
jj config set --user merge-tools.code.merge-args '["--wait", "--merge", "$left", "$right", "$base", "$output"]'
jj config set --user ui.merge-editor "code"
```

### vimを使う

```bash
jj config set --user ui.merge-editor "vimdiff"
```

### カスタムツールを使う

```bash
# meld
jj config set --user ui.merge-editor "meld"

# kdiff3
jj config set --user ui.merge-editor "kdiff3"
```

---

## トラブルシューティング

### エラー：「No conflicts to resolve」

**原因：** コンフリクトがない

**解決：**
```bash
jj status
# コンフリクトがあるか確認
```

### エディタが開かない

**原因：** エディタが設定されていない

**解決：**
```bash
jj config set --user ui.editor "nano"
```

### 間違えて保存してしまった

**解決：**
```bash
# 操作を取り消し
jj op undo
```

---

## コマンドリファレンス

```bash
# 基本的な使い方
jj resolve                    # コンフリクトを対話的に解決

# すべてのコンフリクトを処理
jj resolve --all

# 特定のファイルだけ解決
jj resolve path/to/file.txt

# コンフリクトの状態を確認
jj status

# コンフリクトの内容を確認
jj diff

# マージツールの設定を確認
jj config list merge-tools
```

---

## ベストプラクティス

### 1. まず差分を確認

```bash
jj resolve
# l でLeft差分を確認
# r でRight差分を確認
# その後 e で編集
```

### 2. 小さな変更から解決

複数のコンフリクトがある場合、簡単なものから解決していく

### 3. テストを実行

コンフリクト解決後、必ずテストを実行して動作確認

### 4. コミットメッセージを更新

```bash
jj describe -m "マージ完了：コンフリクトを解決"
```

### 5. 不安なら実験用ブランチで練習

```bash
# 本番前に練習
jj new test-merge
# コンフリクト解決を練習
# 問題なければ abandon して本番へ
```

---

## まとめ

### jj resolveのワークフロー

```
1. マージでコンフリクト発生
   ↓
2. jj status で確認
   ↓
3. jj resolve 実行
   ↓
4. "e" でエディタ起動
   ↓
5. Base/Left/Right を見ながら Result を編集
   ↓
6. 保存して終了
   ↓
7. jj status で解決を確認
   ↓
8. 完了！
```

### いつ使うべきか

| コンフリクトの種類 | 推奨方法 |
|------------------|---------|
| 単純（1〜2箇所） | 手動編集でもOK |
| 複雑（3箇所以上） | **jj resolve推奨** |
| 複数ファイル | **jj resolve推奨** |
| 大きなファイル | **jj resolve推奨** |

---

## 練習問題

### 練習1：基本的なコンフリクト

1. 2つのブランチで同じファイルを編集
2. マージしてコンフリクトを発生させる
3. `jj resolve`で解決

### 練習2：複雑なコンフリクト

1. 3つのファイルで異なる部分を編集
2. マージして複数のコンフリクトを発生
3. 1つずつ`jj resolve`で解決

### 練習3：手動編集との比較

1. 同じコンフリクトを作成
2. 片方を`jj resolve`で解決
3. もう片方を手動編集で解決
4. どちらが使いやすいか体感

---

お疲れ様でした！🎉

`jj resolve`をマスターすれば、コンフリクト解決が怖くなくなります。実際のプロジェクトでも自信を持って使えるようになりましょう！

**関連ドキュメント：**
- [03-merging.md](03-merging.md) - マージの基本
- [04-history-editing.md](04-history-editing.md) - 履歴編集

**前のセクション：** [03-merging.md](03-merging.md)  
**次のセクション：** [04-history-editing.md](04-history-editing.md)

