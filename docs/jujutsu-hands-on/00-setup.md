# 00. セットアップ：jujutsuの環境構築

このセクションでは、jujutsuをインストールして基本的な設定を行います。

## 学習目標

- jujutsuをインストールする
- 初期設定を行う
- 動作確認をする

---

## jujutsuとは？

**jujutsu（コマンド名：`jj`）** は、Googleが開発した次世代のバージョン管理システムです。

### 主な特徴

- **簡単な履歴編集**：コミットの修正や並び替えが簡単
- **working copyの概念**：常にクリーンな作業環境
- **自動的なコンフリクト追跡**：コンフリクトを状態として保持
- **Gitとの互換性**：Gitリポジトリと相互運用可能

---

## 1. インストール

### macOS（Homebrew）

```bash
brew install jj
```

### Linux（Cargo）

```bash
cargo install --git https://github.com/martinvonz/jj.git --locked --bin jj jj-cli
```

### その他のインストール方法

公式ドキュメント参照：https://github.com/martinvonz/jj

---

## 2. インストール確認

以下のコマンドを実行してください：

```bash
jj --version
```

**期待される出力例：**

```
jj 0.21.0
```

バージョン番号が表示されれば、インストール成功です！

---

## 3. 初期設定

jujutsuを使う前に、ユーザー情報を設定します。

### ユーザー名とメールアドレスの設定

```bash
jj config set --user user.name "あなたの名前"
jj config set --user user.email "your.email@example.com"
```

**実際の例：**

```bash
jj config set --user user.name "Taro Yamada"
jj config set --user user.email "taro@example.com"
```

### 設定の確認

```bash
jj config list user
```

**期待される出力例：**

```
user.name = "Taro Yamada"
user.email = "taro@example.com"
```

---

## 4. エディタの設定（オプション）

コミットメッセージを書くためのエディタを設定できます。

### Visual Studio Code を使う場合

```bash
jj config set --user ui.editor "code --wait"
```

### Vim を使う場合

```bash
jj config set --user ui.editor "vim"
```

### nano を使う場合（初心者におすすめ）

```bash
jj config set --user ui.editor "nano"
```

---

## 5. 便利なエイリアスの設定（オプション）

jujutsuの新しいバージョンでは、リポジトリ初期化コマンドが`jj git init`に変更されました。
より簡単に`jj init`で使えるようにエイリアスを設定できます。

### `jj init`でGitバックエンドを使う

```bash
jj config set --user aliases.init '["git", "init"]'
```

これで`jj init`が`jj git init`と同じ動作をします！

### その他の便利なエイリアス

```bash
# jj l で最近の20件のログを表示
jj config set --user aliases.l 'log --limit 20'

# jj s で status
jj config set --user aliases.s 'status'

# jj d で diff
jj config set --user aliases.d 'diff'
```

---

## 6. 動作確認

簡単なテストを実行して、すべてが正しく動作するか確認しましょう。

### ヘルプコマンドを実行

```bash
jj help
```

主要なコマンドの一覧が表示されます。

### よく使うコマンドを確認

```bash
jj help init
```

`jj init`コマンドの詳細なヘルプが表示されます。

---

## 7. 便利なツール：lazyjj（オプション）

**lazyjj** は、jujutsuのためのTUI（Terminal User Interface）ツールです。ログやコミット履歴を視覚的に見ることができ、非常に便利です。

### lazyjjとは？

- **視覚的なログ表示**：コミット履歴をグラフィカルに表示
- **対話的な操作**：キーボードで直感的に操作可能
- **コミット詳細の確認**：差分やメタデータを簡単に確認
- **ブランチ管理**：ブックマークの一覧と操作

### インストール方法

#### macOS（Homebrew）

```bash
brew install lazyjj
```

#### Cargo（全プラットフォーム対応）

```bash
cargo install lazyjj
```

### 使い方

リポジトリ内で以下のコマンドを実行：

```bash
lazyjj
```

または、エイリアスを設定して素早く起動：

```bash
# zshの場合（~/.zshrcに追加）
alias ljj='lazyjj'

# 設定を反映
source ~/.zshrc
```

これで `ljj` と入力するだけで起動できます！

### 基本操作

起動後、以下のキーで操作できます：

| キー | 動作 |
|------|------|
| `j` / `k` または `↑` / `↓` | コミット間を移動 |
| `Enter` | コミット詳細を表示 |
| `d` | 差分を表示 |
| `b` | ブックマーク一覧 |
| `?` | ヘルプを表示 |
| `q` | 終了 |

### 💡 なぜlazyjjが便利？

1. **ログが見やすい**：`jj log` の出力をグラフィカルに表示
2. **差分確認が簡単**：コミットを選択して即座に差分表示
3. **ブランチ管理**：ブックマークの状態を一目で確認
4. **学習に最適**：jujutsuの状態を視覚的に理解できる

### インストール確認

```bash
lazyjj --version
```

バージョンが表示されれば成功です！

---

## やってみよう！

以下のコマンドを実際に実行してみましょう：

1. バージョンを確認する
2. あなたの名前とメールアドレスを設定する
3. 設定が正しく保存されたか確認する
4. お好みのエディタを設定する
5. エイリアスを設定する（`jj init`で使えるように）
6. `jj help log` を実行して、logコマンドの使い方を確認する

---

## トラブルシューティング

### コマンドが見つからない

**エラー：** `command not found: jj`

**解決方法：**
- インストールが正しく完了しているか確認
- ターミナルを再起動
- PATHが正しく設定されているか確認

### 設定が保存されない

**解決方法：**
- `--user` オプションを付けているか確認
- 設定ファイルの権限を確認：`~/.jjconfig.toml`

---

## チェックリスト

次のセクションに進む前に、以下を確認してください：

- [ ] jujutsuがインストールされている（`jj --version`が動作する）
- [ ] ユーザー名とメールアドレスが設定されている
- [ ] `jj config list user` で設定が表示される
- [ ] エディタの設定が完了している（オプション）

---

## 次のステップ

セットアップが完了しました！次は [01-basics.md](01-basics.md) で、jujutsuの基本操作を学びましょう。

- リポジトリの作成
- ファイルの追加とコミット
- 履歴の確認

準備ができたら、次のセクションに進んでください！

