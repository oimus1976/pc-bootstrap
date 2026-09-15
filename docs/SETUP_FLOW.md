# pc-bootstrap 初期セットアップ手順（Windows / 寄贈ノートPC想定）

## 目的

新規 Windows PC（寄贈ノートPC含む）を、
**短時間で・安全に・再現可能な形**で
開発兼雑用PCとしてセットアップする。

本手順は以下を満たすことを目標とする。

- GitHub 未作成状態から開始できる
- 詰まりやすいポイント（Git / SSH / Keyboard）を明示する
- 2台目以降は「考えずに同じ手順を踏める」

---

## 前提条件

- OS：Windows 10 / 11
- 作業シェル：Git Bash / PowerShell
- ネットワーク：GitHub に HTTPS / SSH 接続可能
- Git は **winget で最小構成インストール**されている想定

---

## Step 0: キーボード配列の確認（最優先）

### 症状

- 日本語表示だが、キーボードが US（101/102）認識
- 記号位置が合わず作業に支障

### 対処（手動・即時）

1. 設定 → 時刻と言語 → 言語と地域
2. 関連設定 → キーボード
3. **ハードウェア キーボード レイアウト**
4. **日本語キーボード（106/109）** を選択
5. **サインアウト or 再起動**

> ※ 初回セットアップ中は最優先で対応する  
> ※ 自動化スクリプトは `keyboard/` 配下に配置予定

---

## Step 1: Git のインストール

```powershell
winget install --id Git.Git --source winget
```

確認：

```bash
git --version
```

---

## Step 2: Git identity の設定（必須）

GitHub が無くても **commit には必須**。

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

確認：

```bash
git config --global --list
```

### 注意

この設定が無いと、以下のエラーで止まる。

```text
Author identity unknown
fatal: unable to auto-detect email address
```

---

## Step 3: ローカルで pc-bootstrap を作成

### ディレクトリ作成

```bash
cd ~
mkdir pc-bootstrap
cd pc-bootstrap
```

### Git 初期化

```bash
git init
```

### フォルダ構成作成

```bash
mkdir docs bootstrap terminal git powershell keyboard install
```

---

## Step 4: tree コマンドの確認

Windows 10 / 11 には標準の `tree` コマンドがあるため、まず標準コマンドを使用する。
外部の `tree.exe` を `C:\Program Files\Git\cmd` などへ手動配置しない。

PowerShell / コマンドプロンプトで確認：

```powershell
tree /?
```

Git Bash から Windows 標準 `tree` を呼び出す場合：

```bash
/c/Windows/System32/tree.com /F
```

必要に応じて `.bashrc` に別名を定義する：

```bash
alias wintree='/c/Windows/System32/tree.com /F'
```

反映：

```bash
source ~/.bashrc
```

> 外部バイナリを追加する場合は、取得元・ハッシュ・ライセンス・更新方法を別途確認してから採用する。

---

## Step 5: README / .gitignore 作成

### README.md（最低限）

```markdown
# pc-bootstrap

Windows PC を再現可能にセットアップするための
bootstrap / dotfiles リポジトリ。
```

### .gitignore（最低限）

```gitignore
.env
.env.*
*.key
*.pem
*.pfx
*.p12
.ssh/
id_ed25519
id_ed25519.pub
id_rsa
id_rsa.pub
*.log
Thumbs.db
```

---

## Step 6: 最初の commit（GitHub 不要）

```bash
git add .
git commit -m "chore: initialize pc-bootstrap structure"
```

### 重要

- **GitHub リポジトリはまだ無くてよい**
- commit は完全にローカル操作

---

## Step 7: SSH 鍵の作成（このPC用）

### 鍵生成

```bash
ssh-keygen -t ed25519 -C "pc-bootstrap-dev"
```

鍵生成時は **推測されにくいパスフレーズを設定することを推奨**する。
入力したパスフレーズは表示されない。

頻繁に入力したくない場合は、秘密鍵をパスフレーズなしにするのではなく、`ssh-agent` などのエージェント利用を検討する。

確認：

```bash
ls ~/.ssh
```

期待される状態：

```text
id_ed25519
id_ed25519.pub
known_hosts
```

> `id_ed25519` は秘密鍵。内容を表示・共有・Git管理しない。  
> GitHub に登録するのは `id_ed25519.pub` の公開鍵のみ。

---

## Step 8: GitHub に SSH 公開鍵を登録

### 公開鍵表示

```bash
cat ~/.ssh/id_ed25519.pub
```

### GitHub 側操作

- Settings → SSH and GPG keys → New SSH key
- Title：`pc-bootstrap-dev`
- Key type：Authentication Key
- Key：表示された **公開鍵** 1 行を貼り付け

---

## Step 9: SSH 接続確認

```bash
ssh -T git@github.com
```

成功例：

```text
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## Step 10: GitHub リポジトリ作成 → push

### GitHub Web

- Repository name：`pc-bootstrap`
- README / .gitignore は **作らない**

### ローカル操作

```bash
git remote add origin git@github.com:<account>/pc-bootstrap.git
git branch -M main
git push -u origin main
```

---

## トラブルシュート集

### Permission denied (publickey)

- SSH 鍵が GitHub に未登録
- `~/.ssh/id_ed25519.pub` を登録する

### known_hosts はあるが接続できない

- 相手確認のみ完了
- 自分の鍵が無い／未登録

### tree が無い／Git Bashから使いにくい

- Windows 標準 `tree.com` を `/c/Windows/System32/tree.com` として呼び出す
- 外部バイナリの手動配置を前提にしない

---

## 到達状態（完了条件）

- GitHub に `pc-bootstrap` が存在
- `git push` が SSH で成功
- フォルダ構成が確認できる
- 次の PC でも同手順を再現可能

---

## Step X: VS Code のインストール

```powershell
winget install --id Microsoft.VisualStudioCode
```

- 初回は Settings Sync を有効化しない
- 最小拡張のみ手動導入

---

## ① まず動作確認（30秒）

```bash
code --version
```

バージョンが出れば OK。
出ない場合は **再ログイン or 再起動**で PATH が反映されます。

---

## ② VS Code 初期起動でやること（手動・最小）

### 1️⃣ VS Code を起動

```bash
code .
```

### 2️⃣ Settings Sync は「まだONにしない」

理由：

- 今は **dotfiles 化の設計中**
- Sync を先に入れると「どこから来た設定か分からない状態」になる

**今日は OFF のまま**で正解。

---

## ③ 最低限の拡張だけ入れる（dev-lite）

いま入れていいのは **これだけ**。

```bash
code --install-extension ms-python.python
code --install-extension yzhang.markdown-all-in-one
```

（Git は内蔵で十分）

> Copilot / Playwright / Docker 拡張は **まだ入れない**  
> 後で「役割別」に切る。

---

## ④ settings.json を「素材として取り出す」

次の dotfiles 化に使うため、**いまの素の状態を保存**します。

### 設定ファイルの場所

```text
%APPDATA%\Code\User\settings.json
```

PowerShell なら：

```powershell
copy `
  "$env:APPDATA\Code\User\settings.json" `
  "$HOME\pc-bootstrap\vscode.settings.base.json"
```

（まだ編集しない。**素材保管**）

> `settings.json` には拡張機能や利用環境によってトークン、内部URL、ユーザー固有パスなどが含まれる可能性がある。  
> 内容を確認せず、そのままGit管理対象に追加しない。

---

## ⑤ pc-bootstrap 側に VS Code 用フォルダを用意

```bash
cd ~/pc-bootstrap
mkdir vscode
```

最終的にはこうなる予定：

```text
pc-bootstrap/
└─ vscode/
   ├─ settings.base.json     # 素の状態
   ├─ settings.dev-lite.json # 開発用差分
   ├─ extensions.dev-lite.txt
   └─ apply-vscode.ps1
```

※ 今回は **まだ作らない**。構成を固めただけ。

---

## ⑥ ここまでで「完成」とみなす理由

この時点でできること：

- Python / Markdown 編集
- GitHub リポジトリ編集
- docs/SETUP_FLOW.md 更新
- VS Code を **汚さず**使える

**「足りないから足す」より「必要になるまで足さない」** を基本とする。

---

### 注意: VS Code の Clone Repository について

既存の Git リポジトリ配下で Clone を実行すると、
フォルダが入れ子になり、意図しないリポジトリを参照する。
`pc-bootstrap` では clone / init は必ず CLI で行う。

---

## 次のステップ（未実施）

### A️⃣ VS Code の dotfiles 設計に入る

settings / extensions / apply スクリプト。

### B️⃣ Python 環境（venv 前提）を固める

VS Code と噛み合う構成にする。

### C️⃣ 残り2台の寄贈PCで再現テスト

`SETUP_FLOW.md` をなぞって確認する。

### その他

- Terminal 設定の適用
- キーボード自動切替スクリプト適用
- PowerShell / Bash 設定の dotfiles 化
- bootstrap.ps1 の作成
