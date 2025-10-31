# Cursor + Jujutsu 実践ガイド

このガイドでは、Cursor AIエージェント（Claude Code）とJujutsuを組み合わせた実践的なペアプログラミング手法を解説します。

---

## 前提条件

### 必須設定

1. **Jujutsuのインストール**
   ```bash
   brew install jj  # macOS
   ```

2. **Cursor User Rules の設定**
   - `docs/setup-cursor-user-rules.md`を参照
   - `docs/cursor-user-rules-jujutsu.md`の内容をCursor Settingsに設定

3. **Jujutsuエイリアスの設定**
   ```bash
   jj config set --user aliases.bcp '["bookmark", "create", "-r", "@-"]'
   jj config set --user aliases.nb '["new", "main"]'
   jj config set --user aliases.rs '["resolve"]'
   ```

4. **GitHub CLI（オプション）**
   ```bash
   brew install gh
   gh auth login
   ```

---

## 基本的なワークフロー

### シナリオ1: 新機能開発（完全自動化）

#### ユーザーの指示

```
新しい機能を追加したいです。
ユーザープロフィール編集機能を実装してください。
```

#### AIエージェントの動作（期待）

1. **作業開始の宣言**
   ```
   Jujutsuで新しい機能を開発します。
   mainブランチから新しい作業を開始します。
   ```

2. **コマンド実行**
   ```bash
   jj new main
   ```

3. **実装作業**
   - ファイル作成・編集
   - コード実装
   - テスト追加

4. **コミット**
   ```bash
   jj commit -m "feat: ユーザープロフィール編集機能を実装"
   ```

5. **ブックマーク作成**
   ```bash
   jj bookmark create feature/user-profile-edit -r '@-'
   ```

6. **状況説明**
   ```
   実装が完了しました。
   
   作成されたファイル:
   - src/components/ProfileEdit.tsx
   - tests/profile-edit.test.ts
   
   次のステップ:
   1. テストを実行して確認
   2. PRを作成してレビュー依頼
   
   PR作成コマンド:
   jj git push --bookmark feature/user-profile-edit
   gh pr create
   ```

### シナリオ2: バグ修正（迅速対応）

#### ユーザーの指示

```
ログイン機能でエラーが出ています。
src/auth/login.ts の null チェックが不足しているようです。
```

#### AIエージェントの動作

1. **問題の確認**
   ```bash
   jj new main
   ```

2. **ファイルの確認と修正**
   ```typescript
   // 修正前
   const user = await getUser(email);
   return user.id;
   
   // 修正後
   const user = await getUser(email);
   if (!user) {
     throw new Error('User not found');
   }
   return user.id;
   ```

3. **コミットとブックマーク**
   ```bash
   jj commit -m "fix: ログイン時のnullチェックを追加"
   jj bookmark create fix/login-null-check -r '@-'
   ```

4. **テスト実行の提案**
   ```bash
   npm test src/auth/login.test.ts
   ```

### シナリオ3: 複数機能の並行開発

#### ユーザーの指示

```
以下の3つの機能を並行して開発したいです：
1. ダークモード対応
2. 通知機能
3. 検索機能の改善

それぞれ独立した作業として進めてください。
```

#### AIエージェントの動作

**機能1: ダークモード**
```bash
jj new main
# ... 実装 ...
jj commit -m "feat: ダークモードを実装"
jj bookmark create feature/dark-mode -r '@-'
```

**機能2: 通知機能**
```bash
jj new main
# ... 実装 ...
jj commit -m "feat: 通知機能を実装"
jj bookmark create feature/notifications -r '@-'
```

**機能3: 検索機能**
```bash
jj new main
# ... 実装 ...
jj commit -m "feat: 検索機能を改善"
jj bookmark create feature/search-improvements -r '@-'
```

**履歴確認**
```bash
jj log
```

出力イメージ：
```
@  feature/search-improvements
│  feat: 検索機能を改善
│
│ ◉ feature/notifications
├─╯ feat: 通知機能を実装
│
│ ◉ feature/dark-mode
├─╯ feat: ダークモードを実装
│
◉  main
│  (既存のコミット)
```

---

## 高度なワークフロー

### マージとコンフリクト解決

#### シナリオ: 2つの機能をマージ

**状況:**
- `feature/ui-update` - UIコンポーネント更新
- `feature/api-integration` - API統合
- 両方とも同じファイル（`src/App.tsx`）を変更

#### ユーザーの指示

```
feature/ui-update と feature/api-integration をマージしてください。
コンフリクトがあれば解決してください。
```

#### AIエージェントの動作

1. **マージ実行**
   ```bash
   jj new feature/ui-update feature/api-integration
   ```

2. **コンフリクト検出**
   ```
   Warning: There are unresolved conflicts at these paths:
   src/App.tsx    2-sided conflict
   ```

3. **コンフリクト解決**
   
   **対話的解決:**
   ```bash
   jj resolve
   ```
   
   AIが選択：
   - Option 1: feature/ui-updateの変更を採用
   - Option 2: feature/api-integrationの変更を採用
   - Option 3: 手動マージが必要
   
   **手動マージが必要な場合:**
   ```
   src/App.tsx にコンフリクトがあります。
   以下の変更を統合する必要があります：
   
   feature/ui-update:
   - UIコンポーネントの刷新
   - 新しいスタイリング
   
   feature/api-integration:
   - API呼び出しロジック
   - エラーハンドリング
   
   両方の変更を保持する形でマージします。
   ```

4. **マージコミット作成**
   ```bash
   jj commit -m "merge: UIアップデートとAPI統合を統合"
   jj bookmark create feature/integrated -r '@-'
   ```

### リベース

#### シナリオ: feature branchをmainの最新に同期

**状況:**
- `feature/long-running` - 数日かけて開発中
- `main` - 他のメンバーが更新している

#### ユーザーの指示

```
feature/long-running を main の最新にリベースしてください。
```

#### AIエージェントの動作

1. **現在の状態確認**
   ```bash
   jj log -r 'feature/long-running::main'
   ```

2. **リベース実行**
   ```bash
   jj rebase -d main -s feature/long-running
   ```

3. **コンフリクト処理**
   ```bash
   # コンフリクトがあれば
   jj resolve
   ```

4. **結果確認**
   ```bash
   jj log
   ```

---

## ベストプラクティス

### 1. 作業開始時の確認

AIエージェントに以下を確認させる：

```
現在の状態を確認してください：
1. 現在のブランチ
2. 未コミットの変更
3. 最新のmainからの差分
```

**期待される動作:**
```bash
jj status
jj log -r @
jj log -r 'main::@'
```

### 2. コミット前のレビュー

```
変更内容を確認してからコミットしてください。
```

**期待される動作:**
```bash
jj diff
jj status
# レビュー後
jj commit -m "..."
```

### 3. PR作成前のチェック

```
PR作成の準備をしてください。
```

**期待される動作:**
```bash
# 1. ブックマーク確認
jj bookmark list

# 2. ブックマーク作成（なければ）
jj bookmark create feature/xxx -r '@-'

# 3. 履歴確認
jj log

# 4. 変更内容確認
jj diff -r 'main::@'

# 5. PR作成
jj git push --bookmark feature/xxx
gh pr create --title "..." --body "..."
```

---

## トラブルシューティング

### AIがGitコマンドを使ってしまう

**原因:**
- User Rulesが正しく設定されていない
- 明示的な指示が不足

**対処法:**
```
jjコマンドを使用してください。
Gitコマンドは使わないでください。
```

### 空コミットにブックマークが作成される

**症状:**
```bash
Warning: Target revision is empty.
```

**原因:**
- `jj commit` する前に `jj bookmark create`

**対処法（AIに指示）:**
```
ブックマークは必ず -r '@-' オプションを使って、
直前のコミットに作成してください。
```

### immutableエラー

**症状:**
```bash
Error: Commit XXX is immutable
```

**原因:**
- リモートと同期されたコミットを編集しようとした

**対処法（AIに指示）:**
```
リモートブックマークは編集できません。
jj new main で新しいコミットを作成してください。
```

---

## 実践例：完全なフロー

### プロジェクト: TODOアプリの機能追加

#### 1. プロジェクト開始

**ユーザー:**
```
TODOアプリに優先度機能を追加したいです。
以下の要件で実装してください：

1. タスクに優先度（高・中・低）を設定可能
2. 優先度でフィルタリング
3. 優先度順にソート
```

**AI:**
```
TODOアプリに優先度機能を実装します。
Jujutsuで新しいブランチを作成して開発を開始します。
```

```bash
jj new main
```

#### 2. 実装

**AI が実行:**
1. データモデル更新
2. UIコンポーネント追加
3. フィルタリングロジック実装
4. ソート機能実装
5. テスト追加

#### 3. コミットとブックマーク

```bash
jj commit -m "feat: タスクに優先度機能を追加

- 優先度（高・中・低）の設定
- 優先度フィルタリング
- 優先度ソート
- テスト追加"

jj bookmark create feature/task-priority -r '@-'
```

#### 4. レビュー依頼

**ユーザー:**
```
実装内容を確認して、PR作成してください。
```

**AI:**
```bash
# 変更内容確認
jj diff -r 'main::@'

# PR作成
jj git push --bookmark feature/task-priority
gh pr create \
  --title "feat: タスクに優先度機能を追加" \
  --body "## 概要
TODOアプリに優先度機能を実装しました。

## 変更内容
- タスクに優先度（高・中・低）を設定可能
- 優先度でフィルタリング
- 優先度順にソート
- 関連するテストを追加

## テスト
\`npm test\` でテストが通ることを確認済み"
```

#### 5. レビューフィードバック対応

**ユーザー:**
```
レビューで「デフォルト優先度は中にすべき」とコメントがありました。
修正してください。
```

**AI:**
```bash
# 同じブランチで作業継続
jj new feature/task-priority

# 修正実装
# ...

jj commit -m "fix: デフォルト優先度を中に変更"

# ブックマーク更新
jj bookmark set feature/task-priority -r '@-'

# プッシュ
jj git push --bookmark feature/task-priority
```

#### 6. マージ後のクリーンアップ

**ユーザー:**
```
PRがマージされました。
ブランチをクリーンアップしてください。
```

**AI:**
```bash
# mainを最新化
jj git fetch

# ブックマーク削除
jj bookmark delete feature/task-priority

# 履歴確認
jj log
```

---

## まとめ

### Cursor + Jujutsu の利点

1. **柔軟な開発**
   - ブックマークなしで自由に実験
   - 必要になったら名前を付ける

2. **並行開発が簡単**
   - 複数の機能を同時進行
   - `jj new main` で簡単に切り替え

3. **安全な操作**
   - `jj op undo` でいつでも戻れる
   - すべての操作が記録される

4. **AIとの相性**
   - 明確なコマンド体系
   - エラーメッセージが分かりやすい
   - User Rulesで一貫した動作

### 次のステップ

1. **User Rulesを設定**
   - `docs/setup-cursor-user-rules.md`を参照

2. **実際に試す**
   - 小さな機能追加から始める
   - AIに指示を出してみる

3. **ワークフローをカスタマイズ**
   - 自分のスタイルに合わせて調整
   - User Rulesを更新

4. **チームに共有**
   - このドキュメントを共有
   - Project Rulesでチーム標準を設定

---

## 参考資料

- **ハンズオンガイド**: `docs/jujutsu-hands-on/`
- **User Rules設定**: `docs/setup-cursor-user-rules.md`
- **User Rulesテキスト**: `docs/cursor-user-rules-jujutsu.md`
- **Jujutsuを選ぶ理由**: `docs/jujutsu-hands-on/why-jujutsu.md`

Happy Coding with Cursor + Jujutsu! 🚀

