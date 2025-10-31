# Cursor User Rules 設定手順

このガイドでは、JujutsuワークフローをCursorのUser Rulesに設定する手順を説明します。

---

## User Rulesとは？

Cursor Settings > General > Rules for AI に設定するルールで、**すべてのプロジェクトに自動適用**されます。

### 特徴
- ✅ プロジェクト横断で使える
- ✅ プレーンテキスト形式
- ✅ 個人的なコーディング規約や応答スタイルに最適
- ❌ `.mdc`形式は不要（プレーンテキストのみ）
- ❌ `~/.cursor/rules`は機能しない

---

## 設定手順

### ステップ1: User Rules用テキストを開く

プロジェクト内の以下のファイルを開きます：

```
docs/cursor-user-rules-jujutsu.md
```

このファイルには、AIエージェントがJujutsuを使うための完全なルールが記述されています。

### ステップ2: Cursor設定を開く

**Mac:**
```
Cmd + ,
```

**Windows/Linux:**
```
Ctrl + ,
```

### ステップ3: Rules for AI セクションへ移動

1. 左サイドバーから **General** を選択
2. 下にスクロールして **Rules for AI** セクションを見つける

### ステップ4: ルールをコピー＆ペースト

1. `docs/cursor-user-rules-jujutsu.md` の内容をすべてコピー
2. **Rules for AI** のテキストボックスに貼り付け

**注意:**
- 既存のルールがある場合は、末尾に追加するか、適切にマージしてください
- ルールが長すぎる場合は、文字数制限に注意してください

### ステップ5: 設定を保存

Cursorは自動保存されるため、特別な保存操作は不要です。

---

## 設定後の動作確認

### 1. 新規プロジェクトで確認

```bash
# 1. 新規ディレクトリを作成
mkdir /tmp/jj-test
cd /tmp/jj-test

# 2. Jujutsuリポジトリを初期化
jj git init

# 3. Cursorで開く
cursor .
```

### 2. AIエージェントに依頼

Cursorで以下のように依頼してください：

```
新しい機能を開発したいです。
ユーザー登録フォームを作成してください。
```

### 3. 期待される動作

AIエージェントは以下のように動作するはずです：

1. **作業開始:**
   ```bash
   jj new main
   ```

2. **開発作業:**
   - ファイル作成・編集

3. **コミット:**
   ```bash
   jj commit -m "feat: ユーザー登録フォームを実装"
   ```

4. **ブックマーク作成（PR予定の場合）:**
   ```bash
   jj bookmark create feature/user-registration -r '@-'
   ```

5. **説明:**
   - Jujutsuの動作原理を説明
   - 次のステップを提案

---

## トラブルシューティング

### User Rulesが反映されない

**確認事項:**
1. Cursorを再起動してみる
2. Rules for AI のテキストボックスに正しく貼り付けられているか確認
3. 文字数制限を超えていないか確認

**対処法:**
- 必要に応じてルールを短縮
- 最も重要な部分だけを残す

### AIがJujutsuを使わない

**確認事項:**
1. User Rulesに「Jujutsu (jj) を優先使用」が記載されているか
2. プロジェクト内の`.git`ディレクトリが存在するか（jjはGitリポジトリと互換）

**対処法:**
- 明示的に「jjコマンドを使って」と指示
- `jj git init`でJujutsuリポジトリを初期化

### エイリアスが使われない

**確認事項:**
1. `jj config list`でエイリアスが設定されているか確認

**設定方法:**
```bash
# 主要なエイリアスを設定
jj config set --user aliases.bcp '["bookmark", "create", "-r", "@-"]'
jj config set --user aliases.nb '["new", "main"]'
jj config set --user aliases.rs '["resolve"]'
```

---

## ルールのカスタマイズ

### 個人の好みに合わせる

`docs/cursor-user-rules-jujutsu.md`を編集して、以下をカスタマイズできます：

- **ブランチ命名規則**: `feature/`, `fix/`, `docs/` など
- **コミットメッセージフォーマット**: Conventional Commits形式など
- **ワークフロー**: 個人開発重視 vs チーム開発重視

### プロジェクト固有のルールを追加

Cursorでは、User Rules（全プロジェクト）とProject Rules（特定プロジェクト）を組み合わせられます。

**Project Rules の場所:**
- `.cursor/rules/*.mdc`
- `AGENTS.md`
- `.cursorrules`

プロジェクト固有のJujutsuルールがある場合は、Project Rulesに記述してください。

---

## User Rules の管理

### バックアップ

User Rulesは以下の場所に保存されていますが、直接編集は推奨されません：

```
~/Library/Application Support/Cursor/User/settings.json (Mac)
```

**推奨バックアップ方法:**
1. `docs/cursor-user-rules-jujutsu.md`をGitで管理
2. 定期的にUser Rulesの内容をコピーして保存
3. dotfilesリポジトリに含める

### 更新

ルールを更新する場合：

1. `docs/cursor-user-rules-jujutsu.md`を編集
2. Cursor Settings > General > Rules for AI に再度コピー＆ペースト

---

## 参考資料

### このプロジェクトのドキュメント

- **ハンズオンガイド**: `docs/jujutsu-hands-on/`
- **Cursor実践ガイド**: `docs/jujutsu-hands-on/cursor-ai-workflow.md`
- **Jujutsuを選ぶ理由**: `docs/jujutsu-hands-on/why-jujutsu.md`

### 公式ドキュメント

- **Jujutsu公式**: https://martinvonz.github.io/jj/
- **Cursor公式**: https://docs.cursor.com/
- **Cursor Rules for AI**: https://docs.cursor.com/ja/context/rules

---

## よくある質問

### Q1: すべてのプロジェクトでJujutsuを使いたくない場合は？

**A:** Project Rulesで上書きできます。特定のプロジェクトで「Gitを使用してください」と明記すればOKです。

### Q2: チームメンバーとUser Rulesを共有できますか？

**A:** User Rulesは個人設定です。チーム全体で統一したい場合は、Project Rules（`.cursor/rules/*.mdc`）を使用してください。

### Q3: ルールが長すぎて貼り付けられません

**A:** 最も重要な部分（禁止事項、必須パターン）だけを残し、詳細はProject Rulesやドキュメントに記載してください。

---

## まとめ

✅ **設定完了チェックリスト:**

- [ ] `docs/cursor-user-rules-jujutsu.md`の内容を確認
- [ ] Cursor Settings > General > Rules for AI に貼り付け
- [ ] 新規プロジェクトで動作確認
- [ ] AIエージェントが`jj`コマンドを使うことを確認
- [ ] `jj config list`でエイリアスを確認

これで、どのプロジェクトでもAIエージェントがJujutsuを適切に使用できるようになります！🎉

