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

### 🚨 PR作成タスクの実行順序（厳守）

PR作成を指示された場合、**必ず以下の順序で実行**してください：

#### ✅ 必須実行: PR作成前のチェックリスト

**重要**: 以下のStepを**スキップせず、順番通りに**実行すること。

```bash
# Step 1: GitHub認証確認（最優先・スキップ禁止）
gh auth status

# ↓ エラーが出た場合は、以降の手順を中断し、ユーザーに認証対処を依頼

# Step 2: ブックマーク確認
jj bookmark list

# Step 3: リモート同期確認  
jj git fetch

# Step 4: プッシュコマンドを提示（実行しない）
# 以下のコマンドをユーザーに提示するのみ：
# jj git push --bookmark BOOKMARK_NAME --allow-new

# Step 5: PR作成コマンドを提示（実行しない）
# 以下のコマンドをユーザーに提示するのみ：
# gh pr create --head BOOKMARK_NAME --base main \
#   --title "タイトル" --body "説明"

# Step 6: CI監視コマンドを提示
# gh pr checks <pr-number>
```

**重要**: **Git pushとPR作成はAIエージェントが実行しません**。ユーザーが手動で実行してください。必要なコマンドを提示するのみです。

### ❌ 禁止事項

1. **Step 1（GitHub認証確認）をスキップすること**
   - 「後でエラーが出たら対処する」は禁止
   - 必ず最初に`gh auth status`を実行

2. **エラーを無視して次の手順に進むこと**
   - Step 1でエラーが出たら、即座に中断
   - ユーザーに認証対処を依頼

3. **チェックリストの順序を入れ替えること**
   - 「効率化」のための順序変更は禁止
   - 記載された順序には理由がある

4. **Git pushやPR作成を実行すること**
   - AIエージェントはpush/PR作成コマンドを実行しない
   - 必要なコマンドを提示するのみ

## ⚠️ 作業開始前の必須手順（全作業共通）

### 原則：作業前の状態同期

**重要**: どんな作業でも、開始前に必ずローカルとリモートの状態を同期・確認する

**理由:**
- ローカルとリモートの差異 = 認識のズレ
- 認識のズレ = 意図せぬ不具合の温床
- コンフリクト、重複作業、論理的矛盾を予防

### 必須チェック手順

```bash
# Step 1: リモートの最新状態を取得（必須）
jj git fetch

# Step 2: ローカルのmainに未プッシュのコミットがあるか確認（重要！）
jj log -r 'main & ~main@origin'

# 出力がある場合 = 未プッシュのコミットがある → Step 3-A へ
# 出力がない場合 = 安全 → Step 3-B へ
```

### Step 3-A: 未プッシュのコミットがある場合の対処

**⚠️ 警告**: `jj bookmark set main -r 'main@origin'` を実行すると、
未プッシュのコミットへの参照が失われます（孤立します）。

**対処法1: 一時ブックマークで保護してから更新（推奨）**
```bash
# 未プッシュのコミットを一時ブックマークで保護
jj bookmark create main-local-work -r 'main'

# その後、mainを最新に更新
jj bookmark set main -r 'main@origin'

# 後で必要に応じて統合
# jj rebase -d main -s 'main-local-work'
# jj bookmark set main -r 'main-local-work'
# jj bookmark delete main-local-work
```

**対処法2: プッシュしてから同期**
```bash
# 未プッシュのコミットをプッシュ
jj git push --bookmark main

# リモートを再取得
jj git fetch

# mainを更新（この時点で安全）
jj bookmark set main -r 'main@origin'
```

**対処法3: 不要なコミットの場合**
```bash
# 未プッシュのコミットが不要な場合のみ実行
jj bookmark set main -r 'main@origin'

# 孤立したコミットは後から `jj log -r 'all()'` で確認可能
```

### Step 3-B: 未プッシュのコミットがない場合（安全）

```bash
# リモートが進んでいる場合、mainを更新
jj bookmark set main -r 'main@origin'

# 確認
jj log -r 'main' -r 'main@origin'
```

### Step 4: 作業開始

```bash
# 最新のmainから新しい作業を開始
jj new main
```

### ベストプラクティス

**原則:**
- ✅ mainは常にリモートの追跡用（ローカルでmainに直接コミットしない）
- ✅ ローカルの作業は別のブックマークで行う
- ✅ 未プッシュのコミットがある場合は必ず保護してから更新
- ✅ 作業開始前に必ず `jj git fetch` を実行
- ✅ ユーザーから「マージ済み」「プッシュ済み」の情報を受け取ったら必ず同期確認

**悪い例:**
```bash
# ❌ Bad: mainで直接作業
jj edit main
# コミット...
```

**良い例:**
```bash
# ✅ Good: mainから派生した別のブックマークで作業
jj new main
# 作業...
jj commit -m "実装完了"
jj bookmark create feature/my-work -r '@-'
```

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

**必須: 以下のチェックを順番に実行**

#### Step 1: GitHub認証確認

```bash
# 認証状態を確認
gh auth status

# 認証が失敗している場合はログイン
# ⚠️ 注意: AIエージェントは実行しません。ユーザーが手動で実行してください。
```

**重要**: `gh auth login` だけでは不十分です。必ず `gh auth setup-git` を実行してください。
これによりGitのHTTPS認証がGitHub CLIの認証情報を使用するようになります。

#### Step 2: ブックマーク確認

```bash
# 現在のブックマーク一覧を確認
jj bookmark list

# ブックマークがなければ作成（必ず -r '@-' を使用）
jj bookmark create feature-name -r '@-'
```

#### Step 3: リモート同期確認

```bash
# リモートの最新状態を取得
jj git fetch

# 履歴を確認
jj log
```

#### Step 4: プッシュ（新規ブックマークの場合）

**AIエージェントは実行しません。以下のコマンドをユーザーに提示するのみ：**

```bash
# 新規ブックマークは --allow-new が必須
jj git push --bookmark feature-name --allow-new
```

#### Step 5: PR作成（Jujutsu特有の注意）

**AIエージェントは実行しません。以下のコマンドをユーザーに提示するのみ：**

```bash
# --head と --base を明示的に指定（必須）
gh pr create --head feature-name --base main \
  --title "タイトル" \
  --body "説明"
```

**Jujutsu特有の問題**: `gh pr create` をオプションなしで実行すると
`could not determine the current branch` エラーが発生します。
Jujutsuは `.git/HEAD` を更新しないため、必ず `--head --base` を指定してください。

#### Step 6: CI/CD監視

**AIエージェントは実行しません。以下のコマンドをユーザーに提示するのみ：**

```bash
# PR番号を確認
gh pr view <pr-number>

# CI状態を監視
gh pr checks <pr-number>

# CIが完了するまで定期的にチェック
```

## GitHub統合とトラブルシューティング

このセクションでは、JujutsuとGitHub/GitHub CLIを統合する際によく発生する問題と解決策を説明します。

### GitHub認証問題

#### 問題1: `could not read Username for 'https://github.com'`

**エラー全文:**
```
Error: Git process failed: External git program failed:
fatal: could not read Username for 'https://github.com': Device not configured
```

**原因:**
- HTTPS経由でリポジトリが設定されている
- `gh auth login` を実行しただけでは、GitのHTTPS認証には反映されない
- Git credential helperが正しく設定されていない

**解決方法:**
```bash
# 1. GitHub CLIで認証
gh auth login

# 2. Git credential helperを設定（必須）
gh auth setup-git

# 3. 認証状態を確認
gh auth status
# 出力に "✓ Logged in to github.com" が表示されることを確認

# 4. 再度プッシュを試行
jj git push --bookmark feature-name --allow-new
```

**重要**: `gh auth setup-git` は、`~/.gitconfig` に以下の設定を追加します：
```ini
[credential "https://github.com"]
    helper = 
    helper = !/path/to/gh auth git-credential
```

これにより、GitのHTTPS認証がGitHub CLIの認証トークンを使用するようになります。

#### 🔧 Cursor環境での特別な対処法

**問題**: AIエージェントがCursor内で`gh auth login`を自動実行しようとすると、環境によってはTLS証明書エラーが発生する場合があります。

**エラー例**:
```
failed to authenticate via web browser: 
Post "https://github.com/login/device/code": 
tls: failed to verify certificate: x509: OSStatus -26276
```

**原因:**
- Cursorのサンドボックス環境での証明書検証の制約
- macOSのキーチェーンへのアクセス制限
- ブラウザベースの認証フローが動作しない場合がある

**重要**: AIエージェントは認証コマンドを実行しません。認証が必要な場合は、**ユーザーがCursor内のターミナルまたは通常のターミナルアプリで手動で実行**してください。Cursor内のターミナルでも、ユーザーが手動で実行する場合は問題なく動作する場合がほとんどです。

**解決方法:**

**方法1**: Cursor内のターミナルまたは通常のターミナルアプリで実行

```bash
# Cursor内のターミナルまたはTerminal.app、iTerm2等で実行
gh auth login
gh auth setup-git

# 完了後、認証状態を確認
gh auth status
```

**注意**: Cursor内のターミナルで`gh auth login`が失敗する場合は、通常のターミナルアプリ（Terminal.app、iTerm2等）で実行してください。

**方法2**: Personal Access Tokenを使用
```bash
# 1. GitHubでトークンを生成
#    https://github.com/settings/tokens
#    スコープ: repo, workflow, read:org

# 2. 通常のターミナルでトークンで認証
echo "YOUR_TOKEN_HERE" | gh auth login --with-token
gh auth setup-git
```

**方法3**: SSHに切り替え
```bash
# リモートURLをSSHに変更（SSH鍵設定済みの場合）
git remote set-url origin git@github.com:username/repo.git

# プッシュ時にパスワード不要
jj git push --bookmark feature-name --allow-new
```

#### HTTPS vs SSH

**HTTPS（デフォルト）:**
- `gh auth setup-git` が必要
- トークンベースの認証
- ファイアウォール越しでも動作
- **Cursor環境（サンドボックス）では認証に工夫が必要な場合がある**

**SSH（代替案）:**
- SSH鍵の設定が必要
- パスワード不要
- **Cursor環境でも問題なく動作**

### GitHub CLI と Jujutsu の統合問題

#### 問題2: `could not determine the current branch`

**エラー全文:**
```
could not determine the current branch: could not determine current branch: 
failed to run git: not on any branch
```

**原因:**
- GitHub CLI (`gh pr create`) は `.git/HEAD` を読んで現在のブランチを判断
- Jujutsuは Change IDベースで管理しており、`.git/HEAD` を特定のブランチに向けない
- Jujutsuの working copy (`@`) は常に "detached HEAD" 状態に相当

**解決方法:**
```bash
# ❌ これはエラーになる
gh pr create

# ✅ --head と --base を明示的に指定
gh pr create --head docs/feature-name --base main \
  --title "タイトル" \
  --body "説明"
```

**背景知識:**
- **Git**: ブランチベース管理 → すべてのコミットはブランチに属する
- **Jujutsu**: Change IDベース管理 → ブックマークは「名札」程度

この思想の違いにより、GitHub CLIのような「Gitを前提とした」ツールとの統合時には注意が必要です。

### プッシュ時のよくあるエラー

#### 問題3: `Refusing to create new remote bookmark`

**エラー全文:**
```
Error: Refusing to create new remote bookmark docs/feature-name@origin
Hint: Use --allow-new to push new bookmark. 
Use --remote to specify the remote to push to.
```

**原因:**
- 新しいブックマークをリモートにプッシュする際の安全機構
- 誤って新しいブランチを作成するのを防ぐ

**解決方法:**
```bash
# 新規ブックマークには --allow-new を付ける
jj git push --bookmark feature-name --allow-new
```

#### 問題4: 既存ブックマークの更新

```bash
# 既存のブックマークを更新する場合は --allow-new 不要
jj git push --bookmark feature-name
```

### CI/CD監視のベストプラクティス

#### PR作成後の必須確認

```bash
# 1. PR詳細を確認
gh pr view <pr-number>

# 2. CI/CDの状態を確認
gh pr checks <pr-number>
# 出力例:
# CodeRabbit    pending    0    Review in progress
# Tests         success    ✓    All tests passed
```

#### CI完了まで監視する理由

- CI失敗時は即座に修正が必要
- レビュワーはCI成功を待ってからレビュー開始
- 自動マージ設定がある場合、CI成功が必須

#### 定期的な確認コマンド

```bash
# 10秒おきにCI状態を確認（シェルループ）
while true; do 
  gh pr checks <pr-number>
  sleep 10
done
```

### エラーパターン早見表

| エラーメッセージ | 原因 | 解決方法 |
|----------------|------|---------|
| `could not read Username for 'https://github.com'` | Git credential helper未設定 | `gh auth setup-git` 実行 |
| `could not determine the current branch` | JujutsuはHEADを更新しない | `gh pr create --head X --base Y` |
| `Refusing to create new remote bookmark` | 新規ブックマークの安全機構 | `--allow-new` フラグ追加 |
| `Commit XXX is immutable` | リモートコミットの変更試行 | `jj new <target>` で新規作成 |
| `Warning: Target revision is empty` | 空コミットにブックマーク作成 | `jj commit` 後に `jj bookmark create -r '@-'` |
| `There are unresolved conflicts` | コンフリクト未解決 | `jj resolve` または手動編集 |

### トラブルシューティングフローチャート

```
PR作成時にエラー発生
    ↓
認証エラー？
    YES → gh auth setup-git → 再試行
    NO → ↓
    ↓
ブランチ判定エラー？
    YES → --head --base を追加 → 再試行
    NO → ↓
    ↓
プッシュエラー？
    YES → --allow-new を追加 → 再試行
    NO → ↓
    ↓
その他のエラー → jj op log を確認 → jj op undo で戻る
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

#### Jujutsu基本エラー

**`Error: Commit XXX is immutable`**
- 原因: リモートと同期されたコミットは変更不可
- 対処: `jj new <target>` で新規コミット作成

**`Warning: Target revision is empty`**
- 原因: 変更がないコミットにブックマークを作成しようとした
- 対処: `jj commit` してから `jj bookmark create -r '@-'`

**`Error: There are unresolved conflicts`**
- 対処: `jj resolve` または手動編集後 `jj commit`

#### GitHub統合エラー

**`Error: could not read Username for 'https://github.com'`**
- 原因: Git credential helperが未設定
- 対処: 
  ```bash
  # ⚠️ AIエージェントは実行しません。ユーザーが手動で実行してください。
  # Cursor内のターミナルまたは通常のターミナルアプリで実行
  gh auth login
  gh auth setup-git  # 必須
  ```

**`Error: could not determine the current branch`**
- 原因: JujutsuはGitの `.git/HEAD` を更新しない
- 対処: `gh pr create` に `--head` と `--base` を明示的に指定
  ```bash
  gh pr create --head feature-name --base main
  ```

**`Error: Refusing to create new remote bookmark`**
- 原因: 新規ブックマークの安全機構
- 対処: `--allow-new` フラグを追加
  ```bash
  jj git push --bookmark feature-name --allow-new
  ```

#### 操作ミス全般

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

# 5. プッシュとPR作成（ユーザーが実行）
# jj git push --bookmark feature/user-auth --allow-new
# gh pr create --head feature/user-auth --base main \
#   --title "ユーザー認証機能" --body "詳細説明"
```

### シナリオ2: バグ修正

```bash
jj new main
vim src/login.rs
jj commit -m "fix: ログイン時のnullポインタエラーを修正"
jj bookmark create fix/login-bug -r '@-'
# ユーザーが実行: jj git push --bookmark fix/login-bug --allow-new
# ユーザーが実行: gh pr create --head fix/login-bug --base main
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

