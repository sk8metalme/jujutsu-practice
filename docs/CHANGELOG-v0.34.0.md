# Jujutsu v0.34.0 対応変更点

このドキュメントは、本ハンズオンガイドがv0.21.0からv0.34.0に対応する際に確認した変更点をまとめたものです。

## 📅 更新日

**2025年10月31日** - v0.21.0 → v0.34.0 対応完了

---

## ✨ v0.34.0で追加された新機能

### 1. `jj absorb` - 変更の自動吸収

作業コピーの変更を、最後にその行を変更したコミットに自動的に移動します。

**メリット:**
- 複数ファイルの修正を一括処理
- `jj squash`の手動実行が不要
- Git の `git absorb` に相当

**追加箇所:** `docs/jujutsu-hands-on/05-advanced.md` セクション9.1

### 2. `jj parallelize` - 履歴の並列化

直列な履歴を並列（兄弟）関係に変換します。

**メリット:**
- 独立した変更を後から並列化
- 複数PRの並行レビューが可能
- ブランチ依存関係の整理

**追加箇所:** `docs/jujutsu-hands-on/05-advanced.md` セクション9.2

### 3. `jj evolog` - 変更の進化履歴

同じchange IDを持つコミットの履歴を表示します。

**メリット:**
- コミットの進化を可視化
- レビュー対応履歴の追跡
- change ID概念の体感

**追加箇所:** `docs/jujutsu-hands-on/05-advanced.md` セクション9.3

### 4. `jj fix` - 自動フォーマット適用

コード整形ツールを複数のコミットに自動適用します。

**メリット:**
- 複数コミットへの一括適用
- 子孫コミットの自動更新
- CI/CDエラーの事前防止

**追加箇所:** `docs/jujutsu-hands-on/05-advanced.md` セクション9.4

### 5. その他の新機能

- `jj next` / `jj prev` - 履歴の前後移動
- `jj simplify-parents` - 親関係の最適化
- `jj sign` / `jj unsign` - GPG署名の追加/削除
- `jj sparse edit` - スパースチェックアウトの編集

**追加箇所:** `docs/jujutsu-hands-on/05-advanced.md` セクション9.5

---

## ⚠️ 破壊的変更（Breaking Changes）

### 削除されたコマンド

以下のコマンドは **v0.34.0で完全に削除されました**：

❌ **`jj move`** - 代替: `jj squash` または `jj rebase` を使用  
❌ **`jj checkout`** - 代替: `jj edit` を使用  
❌ **`jj merge`** - 代替: `jj new parent1 parent2` を使用

### 非推奨となったコマンド

⚠️ **`jj branch`** → 代替: **`jj bookmark`**

本ガイドは元々 `jj bookmark` を使用していたため、影響なし。

---

## ✅ 互換性確認

### 本ハンズオンガイドの状態

✅ **すべてのドキュメントが v0.34.0 に対応済み**

| ファイル | 状態 | 確認内容 |
|---------|------|---------|
| `README.md` | ✅ 対応済み | バージョン番号更新、新機能追記 |
| `docs/jujutsu-hands-on/00-setup.md` | ✅ 対応済み | 期待される出力例を v0.34.0 に更新 |
| `docs/jujutsu-hands-on/01-basics.md` | ✅ 問題なし | 削除コマンド未使用 |
| `docs/jujutsu-hands-on/02-branches.md` | ✅ 問題なし | `jj bookmark` を使用済み |
| `docs/jujutsu-hands-on/03-merging.md` | ✅ 問題なし | `jj merge` 不存在を明記済み |
| `docs/jujutsu-hands-on/04-history-editing.md` | ✅ 問題なし | 削除コマンド未使用 |
| `docs/jujutsu-hands-on/05-advanced.md` | ✅ 更新済み | v0.34.0新機能セクション追加 |
| `docs/jujutsu-hands-on/06-workflows.md` | ✅ 問題なし | 削除コマンド未使用 |
| `docs/jujutsu-hands-on/why-jujutsu.md` | ✅ 対応済み | バージョン番号更新 |

### 削除コマンドの確認結果

```bash
grep -r "jj move\|jj checkout\|jj merge" docs/jujutsu-hands-on/
```

**結果:** `jj merge` について「存在しない」と正しく明記されている箇所のみ。実際の使用例はゼロ。

```bash
grep -r "jj branch" docs/jujutsu-hands-on/
```

**結果:** ゼロ件。すべて `jj bookmark` を使用。

---

## 📚 学習コンテンツへの影響

### 新規追加項目

1. **05-advanced.md セクション9** - v0.34.0の新機能（全177行追加）
   - `jj absorb` の使い方と実例
   - `jj parallelize` のビジュアル説明
   - `jj evolog` の出力例
   - `jj fix` の設定例とワークフロー
   - その他の改善点（`next`, `prev`, `sign`, `unsign`）

2. **README.md** - 新機能の告知
   - 05-advanced.mdの学習内容に新機能を追記
   - 🆕 絵文字で視認性向上

3. **チェックリスト更新**
   - 05-advanced.mdの到達目標に新機能の習得を追加

### 学習時間の変更

**変更なし** - セクション5は元々1時間30分を想定しており、新機能は応用編として位置づけ。

---

## 🔍 テスト実施内容

### バージョン確認

```bash
jj --version
# jj 0.34.0 ✅
```

### 基本コマンドの動作確認

```bash
jj git init      # ✅ 動作確認済み
jj new main      # ✅ 動作確認済み
jj commit -m     # ✅ 動作確認済み
jj bookmark create  # ✅ 動作確認済み
jj log           # ✅ 動作確認済み
jj status        # ✅ 動作確認済み
```

### 新機能コマンドのヘルプ確認

```bash
jj help absorb      # ✅ 詳細確認済み
jj help parallelize # ✅ 詳細確認済み
jj help evolog      # ✅ 詳細確認済み
jj help fix         # ✅ 詳細確認済み
```

---

## 📝 今後の推奨アクション

### ユーザーへの推奨

1. **環境確認**
   ```bash
   jj --version
   ```
   v0.34.0以上であることを確認

2. **新機能を試す**
   - 05-advanced.md のセクション9を実践
   - 特に `jj absorb` と `jj evolog` は日常的に便利

3. **古い情報に注意**
   - ウェブ上のJujutsu記事がv0.21.0以前の場合、コマンドが異なる可能性
   - 本ガイドは常に最新版に対応

### 保守担当者への注意

- 今後の破壊的変更に注意
- Jujutsuは活発に開発中（年4-6回のリリース）
- 主要リリース時には本ドキュメントを更新

---

## 🔗 参考リソース

- [Jujutsu 公式リポジトリ](https://github.com/martinvonz/jj)
- [Jujutsu リリースノート](https://github.com/martinvonz/jj/releases)
- [本ガイドのREADME](../README.md)

---

**最終更新:** 2025年10月31日  
**対応者:** Cursor AI (Codex)  
**対応バージョン:** v0.21.0 → v0.34.0

