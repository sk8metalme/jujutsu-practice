# Jujutsu Development Workflow for AI Agents

このドキュメントは、Cursor Settings > General > Rules for AI にコピペして使用します。

---

## バージョン管理システム: Jujutsu (jj)

- **優先使用**: Jujutsu (jj) コマンドを使用
- **Git経由**: Gitコマンドは使わず、jj経由でGit操作
- **基本哲学**: Change IDで管理、ブックマークは必要な時だけ

## Jujutsuの基本原則

### Change IDベース管理
- すべてのコミットはChange IDで一意に識別
- ブックマーク（ブランチ名）は「名札」程度の位置づけ
- Gitの「すべてのコミットがブランチに属する」という考えから脱却

### ブックマークが必要なタイミング
1. **リモートプッシュ時**（最重要）
2. **チーム共有時**
3. **長期参照ポイント**

### ブックマークが不要なタイミング
- 個人で作業中
- 実験的な変更
- 一時的な作業

## AIエージェントの必須動作ルール

### 新機能開発開始時

```bash
# 必ずmainから新しい作業を開始
jj new main

# または既存ブックマークから
jj new feature/existing
```

**ブランチ命名規則:**
- `feature/機能名` - 新機能
- `fix/バグ名` - バグ修正
- `refactor/対象` - リファクタリング
- `docs/対象` - ドキュメント更新

### コミット時の必須パターン

**チーム開発（PR作成予定）:**
```bash
# 開発作業
jj commit -m "機能説明"

# 必ずこのパターンでブックマーク作成
jj bookmark create feature-name -r '@-'
```

**個人開発（実験中）:**
```bash
jj commit -m "実験的変更"
# ブックマーク不要、Change IDで管理
```

### PR作成前のチェックリスト

1. ブックマークが存在するか確認
```bash
jj bookmark list
```

2. なければ作成
```bash
jj bookmark create feature-name -r '@-'
```

3. 履歴確認
```bash
jj log
```

4. プッシュとPR作成
```bash
jj git push --bookmark feature-name
gh pr create
```

### マージ（複数親コミット作成）

```bash
# 2つのブランチをマージ
jj new branch-a branch-b

# コンフリクトが発生したら
jj resolve  # 対話的解決

# または手動編集
vim <conflicted-file>
jj commit -m "merge: branch-aとbranch-bを統合"
jj bookmark create merged-feature -r '@-'
```

### リベース

```bash
# 特定のコミットを別の親にリベース
jj rebase -d target-branch

# コンフリクト発生時
jj resolve  # または手動編集
```

### コンフリクト解決

**対話的解決（推奨）:**
```bash
jj resolve
# インタラクティブなツールが起動
# 各コンフリクトを選択して解決
```

**手動解決:**
```bash
# コンフリクトファイルを編集
vim README.md

# 解決後は通常通りコミット
jj commit -m "conflict resolved"
```

## エイリアス活用

既存のjj設定エイリアスを優先使用:

| エイリアス | 実コマンド | 用途 |
|----------|-----------|------|
| `bcp` | `bookmark create -r @-` | **最重要**: コミット後にブックマーク作成 |
| `nb` | `new main` | mainから新規作業開始 |
| `rs` | `resolve` | コンフリクト解決 |
| `n` | `new` | 新規コミット作成 |
| `c` | `commit` | コミット |
| `l` | `log` | ログ表示 |
| `ll` | `log --limit 10` | 直近10件 |
| `st` | `status` | ステータス確認 |
| `co` | `edit` | 別コミットに移動 |

**例:**
```bash
jj nb              # mainから開始
# ... 開発 ...
jj c -m "実装完了"
jj bcp my-feature  # ブックマーク作成
```

## 禁止事項・エラー回避

### 絶対に使わないコマンド

❌ **`jj edit root()`**
- 理由: rootコミットはimmutable
- 代替: `jj new 'root()'`

❌ **`jj merge`**
- 理由: このコマンドは存在しない
- 代替: `jj new parent1 parent2`

❌ **`jj branch`**
- 理由: 古いコマンド
- 代替: `jj bookmark`

### 避けるべき操作

❌ **空コミットにブックマークを付ける**
```bash
# 悪い例
jj new main
jj bookmark create my-feature  # Warning: Target revision is empty.
```

✅ **正しい方法**
```bash
jj new main
# ... 開発 ...
jj commit -m "実装"
jj bookmark create my-feature -r '@-'
```

❌ **リモートブックマークを直接編集**
```bash
# 悪い例
jj edit main  # Error: Commit XXX is immutable (main@origin が存在)
```

✅ **正しい方法**
```bash
jj new main  # mainから新しいコミットを作成
```

### エラー時の対処法

**`Error: Commit XXX is immutable`**
- 原因: リモートと同期されたコミットは変更不可
- 対処: `jj new <target>` で新規コミット作成

**`Warning: Target revision is empty`**
- 原因: 変更がないコミットにブックマークを作成しようとした
- 対処: `jj commit` してから `jj bookmark create -r '@-'`

**`Error: There are unresolved conflicts`**
- 対処: `jj resolve` または手動編集後 `jj commit`

**操作ミス全般**
```bash
# 直前の操作を取り消し
jj op undo

# 操作履歴を確認
jj op log

# 特定の操作時点に戻る
jj op restore <operation-id>
```

## 推奨ワークフロー例

### シナリオ1: 新機能開発（チーム開発）

```bash
# 1. mainから開始
jj new main

# 2. 開発作業
vim src/feature.rs
vim tests/test_feature.rs

# 3. コミット
jj commit -m "feat: ユーザー認証機能を実装"

# 4. ブックマーク作成（PR用）
jj bookmark create feature/user-auth -r '@-'

# 5. プッシュとPR作成
jj git push --bookmark feature/user-auth
gh pr create --title "ユーザー認証機能" --body "詳細説明"
```

### シナリオ2: バグ修正

```bash
jj new main
vim src/login.rs
jj commit -m "fix: ログイン時のnullポインタエラーを修正"
jj bookmark create fix/login-bug -r '@-'
jj git push --bookmark fix/login-bug
gh pr create
```

### シナリオ3: 個人実験（ブックマーク不要）

```bash
# 1. 実験開始
jj new main

# 2-4. 何度も試行錯誤
vim src/experimental.rs
jj commit -m "試行1"

vim src/experimental.rs
jj commit -m "試行2"

vim src/experimental.rs
jj commit -m "試行3 - うまくいった！"

# 5. 成功したらブックマーク付与
jj bookmark create experiment-success -r '@-'

# または失敗したら破棄
jj abandon @
```

### シナリオ4: 複数ブランチのマージ

```bash
# 1. 2つのブランチを確認
jj log

# 2. マージコミット作成
jj new feature-a feature-b

# 3. コンフリクト確認
jj status
# Warning: There are unresolved conflicts at these paths:
# README.md    2-sided conflict

# 4. 対話的解決
jj resolve

# 5. コミット
jj commit -m "merge: feature-aとfeature-bを統合"

# 6. ブックマーク作成
jj bookmark create merged-features -r '@-'
```

### シナリオ5: リベース

```bash
# 1. 現在の状態確認
jj log

# 2. feature-branchをmainの最新にリベース
jj rebase -d main -s feature-branch

# 3. コンフリクトがあれば解決
jj resolve

# 4. 結果確認
jj log
```

## 重要な心構え

1. **Change IDで追跡、名前は後から**
   - Jujutsuの核心的な考え方
   - 作業中は自由に、共有時にブックマーク

2. **失敗を恐れない**
   - `jj op undo` でいつでも戻れる
   - すべての操作が記録される

3. **ブックマークは「名札」**
   - コミットの本質ではない
   - 必要な時だけ付ける

4. **コンフリクトは自然なもの**
   - 並行開発の証
   - `jj resolve` で落ち着いて対処

## 開発フロー判断基準

**個人開発フロー（ブックマーク最小限）を使う場合:**
- ローカルでの実験
- プロトタイピング
- アイデアの試行錯誤
- 自分だけが見るコード

**チーム開発フロー（ブックマーク使用）を使う場合:**
- PR作成予定
- チームメンバーと共有
- レビューが必要
- 本番環境への反映予定

迷ったら、**まずブックマークなしで始めて、必要になったら付ける**のがJujutsu流です。

