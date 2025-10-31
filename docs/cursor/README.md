# Cursor設定ガイド

CursorとJujutsuを組み合わせた開発環境の設定と使い方に関するドキュメントです。

## 📚 ドキュメント構成

### はじめての方
1. **[セットアップガイド](setup.md)** - User Rulesの設定方法
2. **[実践ワークフロー](workflow.md)** - 日常的な使い方

### 設定用テキスト
- **[User Rulesテキスト](user-rules.md)** - Cursor Settingsにコピー＆ペースト

## 🚀 クイックスタート

1. **User Rulesを設定**
   ```bash
   # docs/cursor/user-rules.md の内容をコピー
   # Cursor Settings > General > Rules for AI に貼り付け
   ```

2. **動作確認**
   ```bash
   jj new main
   # AIエージェントがjjコマンドを使うことを確認
   ```

## 📖 関連ドキュメント

- **Jujutsuハンズオン**: `../jujutsu-hands-on/`
- **プロジェクトルール**: `../../AGENTS.md`
- **GitHub認証のトラブルシューティング**: `../retrospective-gh-auth-failure.md`

## ⚠️ 重要な注意事項

### Cursor環境でのGitHub認証

**AIエージェントは認証コマンドを実行しません。**認証が必要な場合は、ユーザーがCursor内のターミナルまたは通常のターミナルアプリで手動で実行してください。

**注意**: Cursor内のターミナルで`gh auth login`が失敗する場合は、通常のターミナルアプリ（Terminal.app、iTerm2等）で実行してください。

詳細は [User Rules](user-rules.md#cursor環境での特別な対処法) を参照してください。

### Git Pushについて

AIエージェントは**pushコマンドを実行しません**。必要なコマンドを提示するのみです。
ユーザーが手動で実行してください。

## 🎯 よくある質問

**Q: User RulesとProject Rulesの違いは？**

A: User Rulesは全プロジェクトに適用、Project Rulesは特定プロジェクトのみ。詳細は[setup.md](setup.md)を参照。

**Q: 認証エラーが発生したら？**

A: [setup.md](setup.md#トラブルシューティング) のトラブルシューティングセクションを確認してください。

