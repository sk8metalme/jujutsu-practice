# 05. 中級機能：jujutsuの真価を発揮する

このセクションでは、jujutsuの設計思想を理解し、高度な機能を使いこなす方法を学びます。

## 学習目標

- change IDの概念を深く理解する
- working copyの仕組みを学ぶ
- revset言語を使いこなす
- 操作履歴（operation log）を活用する
- Gitとの連携方法を学ぶ

---

## 1. change IDの深い理解

### change IDとは？

change IDは、コミットの**内容**を表す一意の識別子です。

### コミットハッシュとの違い

| 項目 | change ID | コミットハッシュ |
|------|-----------|----------------|
| 何を表す？ | 変更の内容 | コミット全体（親、作者、日時を含む） |
| 履歴編集時 | 変わらない | 変わる |
| 用途 | 内容の追跡 | コミットの一意識別 |

### 実験：change IDの不変性

```bash
cd ~/Desktop
mkdir jj-changeid-test
cd jj-changeid-test
jj git init
```

```bash
echo "original content" > file.txt
jj commit -m "最初のコミット"
```

change IDとハッシュを記録：

```bash
jj log --limit 1
```

**出力例：**

```
◉  qpvuntsm taro@example.com 2025-10-23 12:00:00 abc123de
│  最初のコミット
```

- change ID: `qpvuntsm`
- ハッシュ: `abc123de`

### コミットメッセージを変更

```bash
jj describe @- -m "最初のコミット（修正版）"
```

### 再度確認

```bash
jj log --limit 1
```

**出力例：**

```
◉  qpvuntsm taro@example.com 2025-10-23 12:01:00 xyz789fg
│  最初のコミット（修正版）
```

**注目！**
- change ID: `qpvuntsm` （変わっていない！）
- ハッシュ: `xyz789fg` （変わった！）

### 💡 change IDの利点

1. **履歴編集後も追跡可能**
2. **コラボレーションで同じ変更を識別**
3. **リベースやamend後も同じコミットと認識**

---

## 2. working copyの仕組み

### working copyとは？

jujutsuでは、作業ディレクトリの状態も「コミット」として扱われます。

### 実験：working copyを観察

```bash
jj new
echo "test" > test.txt
jj status
```

**出力：**

```
Working copy changes:
A test.txt
Working copy : rlvkpnrz 123456ab (no description set)
Parent commit: qpvuntsm xyz789fg 最初のコミット（修正版）
```

`rlvkpnrz`がworking copyのchange IDです。

### ファイルを編集

```bash
echo "more content" >> test.txt
jj status
```

working copy自体が変更されています。

### working copyの特徴

1. **常に存在する**：`@`で参照できる
2. **自動追跡**：変更は自動的に記録される
3. **コミット可能**：`jj commit`でスナップショットを確定

### working copyの切り替え

```bash
jj edit @-
```

working copyが過去のコミットに移動！

```bash
ls
```

ファイルの内容が過去の状態に戻っています。

```bash
jj edit @+
```

元のworking copyに戻ります。

---

## 3. revset言語：強力なコミット選択

revsetは、コミットを柔軟に指定するためのクエリ言語です。

### 基本的なrevset

#### 単一コミット

```bash
@           # 現在のworking copy
@-          # 親コミット
@--         # 祖父母コミット
root()      # ルートコミット
```

#### ブランチ

```bash
main        # mainブランチのコミット
feature/xyz # feature/xyzブランチのコミット
```

### 範囲指定

#### 先祖をすべて選択

```bash
::@         # @から始まるすべての先祖
::main      # mainブランチまでのすべての先祖
```

#### 子孫をすべて選択

```bash
@::         # @のすべての子孫
```

#### 範囲

```bash
main::@     # mainから@までのすべてのコミット
```

### フィルタリング

#### 作者でフィルタ

```bash
author("taro")
```

#### メッセージでフィルタ

```bash
description("bug fix")
```

#### 日付でフィルタ

```bash
committer_date(after:"2025-10-20")
```

### 論理演算

#### AND（かつ）

```bash
author("taro") & description("feature")
```

#### OR（または）

```bash
author("taro") | author("hanako")
```

#### NOT（否定）

```bash
~empty()    # 空でないコミット
```

### 便利なrevset

#### 空のコミットを除外

```bash
jj log -r "~empty()"
```

#### 特定のファイルを変更したコミット

```bash
jj log -r "file(README.md)"
```

#### ブランチがないコミット

```bash
jj log -r "~branches()"
```

---

## 4. revsetの実践例

### 例1：最近の10件のコミット

```bash
jj log -r "present((@----)..@)" --limit 10
```

### 例2：特定の期間のコミット

```bash
jj log -r 'committer_date(after:"2025-10-01") & committer_date(before:"2025-10-31")'
```

### 例3：複数のブランチのコミット

```bash
jj log -r "main | feature/a | feature/b"
```

### 例4：未完了の作業を探す

```bash
jj log -r 'description("TODO") | description("FIXME")'
```

### 例5：マージコミットを表示

```bash
jj log -r "merges()"
```

---

## 5. 操作履歴の活用

### 操作履歴とは？

すべてのjujutsuコマンドは操作として記録されます。

```bash
jj op log
```

**出力例：**

```
@  ab12cd34 taro@example.com 2025-10-23 12:15:30
│  commit
│  args: jj commit -m "test"
◉  ef56gh78 taro@example.com 2025-10-23 12:14:15
│  new
│  args: jj new
◉  ij90kl12 taro@example.com 2025-10-23 12:13:00
│  describe
   args: jj describe -m "最初のコミット（修正版）"
```

### 操作を取り消す

#### 直前の操作

```bash
jj op undo
```

#### 複数回取り消し

```bash
jj op undo
jj op undo
```

### 特定の操作に戻る

```bash
jj op restore ab12cd34
```

### 操作のやり直し（redo）

```bash
jj op undo      # 取り消し
jj op restore @ # やり直し
```

### 操作履歴のバックアップとして活用

操作履歴により、jujutsuは事実上「タイムマシン」になります：

```bash
# 危険な操作を実行する前
jj op log --limit 1  # 現在のIDを記録

# 危険な操作
jj rebase ...
jj abandon ...

# 問題があれば
jj op restore SAVED_ID  # 元に戻す
```

---

## 6. Gitとの連携

jujutsuは裏でGitを使っています。相互運用が可能です。

### Gitリポジトリから始める

既存のGitリポジトリをjujutsuで使う：

```bash
cd ~/existing-git-repo
jj git init --git-repo .
```

### Gitコマンドとの併用

jujutsuとGitコマンドを同じリポジトリで使えます：

```bash
jj commit -m "jujutsuでコミット"
git log          # Gitでも見える
git commit       # Gitでもコミットできる
jj log           # jujutsuでも見える
```

### リモートとの同期

#### Gitリモートの設定

```bash
git remote add origin https://github.com/user/repo.git
```

#### フェッチ

```bash
jj git fetch
```

#### プッシュ

```bash
jj git push
```

### ブランチの同期

jujutsuのブランチはGitのブランチとして扱われます：

```bash
jj branch create feature/new-feature
jj git push --branch feature/new-feature
```

GitHubやGitLabで通常通りプルリクエストを作成できます！

---

## 7. 設定のカスタマイズ

### 設定ファイル

jujutsuの設定は`~/.jjconfig.toml`に保存されます。

### エイリアスの設定

よく使うコマンドに短縮名を付ける：

```bash
jj config set --user aliases.l 'log --limit 20'
jj config set --user aliases.s 'status'
jj config set --user aliases.d 'diff'
```

使用例：

```bash
jj l   # jj log --limit 20 と同じ
jj s   # jj status と同じ
jj d   # jj diff と同じ
```

### ログの表示形式をカスタマイズ

```bash
jj config set --user ui.log-template 'builtin_log_compact'
```

または独自のテンプレート：

```bash
jj config set --user ui.log-template '
  separate(" ",
    change_id.short(),
    author.name(),
    description.first_line()
  )'
```

### デフォルトのリベース動作

```bash
jj config set --user ui.default-revset '@ | trunk()'
```

---

## 8. 高度なワークフロー

### 複数の作業を同時進行

```bash
# 機能Aの作業
jj new main
jj branch create feature/a
echo "feature A" > a.txt
jj commit -m "機能A"

# 機能Bの作業（機能Aとは独立）
jj new main
jj branch create feature/b
echo "feature B" > b.txt
jj commit -m "機能B"

# 機能Aに戻る
jj edit feature/a

# 機能Bに戻る
jj edit feature/b
```

### スタックワークフロー

複数の関連するコミットをスタックとして管理：

```bash
jj new main
echo "base" > base.txt
jj commit -m "ベース機能"

jj new
echo "extension 1" > ext1.txt
jj commit -m "拡張1"

jj new
echo "extension 2" > ext2.txt
jj commit -m "拡張2"

jj log
```

**出力：**

```
@  xyz ... (empty)
◉  abc ... 拡張2
◉  def ... 拡張1
◉  ghi ... ベース機能
◉  main
```

それぞれを個別のプルリクエストとして送信可能！

---

## 9. トラブルシューティングとデバッグ

### デバッグモード

```bash
jj --debug log
```

詳細な情報が表示されます。

### 設定の確認

```bash
jj config list
```

すべての設定を表示。

### リポジトリの整合性チェック

```bash
jj debug rebuild
```

jujutsuのデータベースを再構築。

### Gitとの同期問題

```bash
jj git import
```

Gitの変更をjujutsuに取り込む。

---

## やってみよう！

### 練習1：change IDの追跡

1. コミットを作成してchange IDを記録
2. メッセージ、内容を変更
3. change IDが変わらないことを確認

### 練習2：revsetマスター

1. 10個のコミットを作成
2. 3つ目から7つ目のコミットを選択
3. 特定の単語を含むコミットを検索
4. 空でないコミットだけを表示

### 練習3：操作履歴の活用

1. いくつかのコミットを作成
2. `jj op log`で履歴を確認
3. いくつかの操作を実行
4. `jj op undo`で元に戻す
5. 特定の操作IDに復元

### 練習4：Gitとの連携

1. GitHubで新しいリポジトリを作成
2. jujutsuで初期化
3. コミットを作成
4. リモートにプッシュ
5. GitHubで確認

---

## チェックリスト

このセクションを完了したら、以下を確認してください：

- [ ] change IDとコミットハッシュの違いを理解した
- [ ] working copyの仕組みを理解した
- [ ] 基本的なrevsetを使える
- [ ] `jj op log`で操作履歴を確認できる
- [ ] `jj op undo`と`jj op restore`を使える
- [ ] Gitリポジトリとの連携方法を理解した
- [ ] 設定をカスタマイズできる
- [ ] 高度なワークフローを理解した

---

## 次のステップ

おめでとうございます！🎉

jujutsuの基本から中級レベルまでをマスターしました。

### さらに学ぶには

1. **公式ドキュメント**：https://github.com/martinvonz/jj
2. **公式チュートリアル**：`jj tutorial`コマンド
3. **コミュニティ**：GitHubのDiscussionsやIssues
4. **実践**：実際のプロジェクトでjujutsuを使ってみる

### 推奨される次のステップ

- 実際のプロジェクトをjujutsuで管理
- チームでの使用を検討
- 高度なrevsetパターンを探求
- カスタムエイリアスとワークフローを確立

---

## まとめ

### jujutsuの哲学

1. **変更は失われない**：すべてが記録される
2. **履歴は編集可能**：後から整理できる
3. **柔軟なワークフロー**：自分に合った方法で使える
4. **Gitとの互換性**：既存のツールと共存

### jujutsuの利点

- ✅ ステージングエリアが不要
- ✅ 強力な履歴編集機能
- ✅ working copyの概念
- ✅ 安全な操作（いつでも元に戻せる）
- ✅ 直感的なコマンド
- ✅ Gitとの完全な互換性

### 活用シーン

- 個人開発：自由に履歴を整理
- チーム開発：クリーンな履歴でPR
- 実験的な開発：いつでも元に戻せる安心感
- 複雑なマージ：コンフリクトを状態として保持

---

お疲れ様でした！jujutsuでの開発を楽しんでください！🚀

