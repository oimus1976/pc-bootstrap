\# SETUP\_FLOW.md

pc-bootstrap 初期セットアップ手順（Windows / 寄贈ノートPC想定）



\## 目的



新規 Windows PC（寄贈ノートPC含む）を、

\*\*短時間で・安全に・再現可能な形\*\*で

開発兼雑用PCとしてセットアップする。



本手順は以下を満たすことを目標とする。



\- GitHub 未作成状態から開始できる

\- 詰まりやすいポイント（Git / SSH / Keyboard）を明示する

\- 2台目以降は「考えずに同じ手順を踏める」



---



\## 前提条件



\- OS：Windows 10 / 11

\- 作業シェル：Git Bash / PowerShell

\- ネットワーク：GitHub に HTTPS / SSH 接続可能

\- Git は \*\*winget で最小構成インストール\*\*されている想定



---



\## Step 0: キーボード配列の確認（最優先）



\### 症状

\- 日本語表示だが、キーボードが US（101/102）認識

\- 記号位置が合わず作業に支障



\### 対処（手動・即時）

1\. 設定 → 時刻と言語 → 言語と地域

2\. 関連設定 → キーボード

3\. \*\*ハードウェア キーボード レイアウト\*\*

4\. \*\*日本語キーボード（106/109）\*\* を選択

5\. \*\*サインアウト or 再起動\*\*



> ※ 初回セットアップ中は最優先で対応する  

> ※ 自動化スクリプトは `keyboard/` 配下に配置予定



---



\## Step 1: Git のインストール



```powershell

winget install --id Git.Git --source winget

````



確認：



```bash

git --version

```



---



\## Step 2: Git identity の設定（必須）



GitHub が無くても \*\*commit には必須\*\*。



```bash

git config --global user.name "Your Name"

git config --global user.email "you@example.com"

```



確認：



```bash

git config --global --list

```



\### 注意



この設定が無いと、以下のエラーで止まる。



```

Author identity unknown

fatal: unable to auto-detect email address

```



---



\## Step 3: ローカルで pc-bootstrap を作成



\### ディレクトリ作成



```bash

cd ~

mkdir pc-bootstrap

cd pc-bootstrap

```



\### Git 初期化



```bash

git init

```



\### フォルダ構成作成



```bash

mkdir docs bootstrap terminal git powershell keyboard install

```



---



\## Step 4: tree コマンドの整備（Git Bash 用）



\### 問題



\* Git Bash には `tree` が無い

\* Windows 標準 `tree` は挙動が不安定

\* winget 版 Git は pacman 非搭載



\### 採用方針



\* 外部で安定した `tree.exe` を導入

\* Git Bash から直接利用可能にする



\### 実施内容



1\. 以下から `tree.exe` を取得

&nbsp;  \[https://gnuwin32.sourceforge.net/packages/tree.htm](https://gnuwin32.sourceforge.net/packages/tree.htm)

2\. 実行ファイルを配置：



```text

C:\\Program Files\\Git\\cmd\\tree.exe

```



\### 日本語文字化け対策



`.bashrc` に以下を追加：



```bash

alias tree='tree -N'

```



反映：



```bash

source ~/.bashrc

```



確認：



```bash

tree -a

```



---



\## Step 5: README / .gitignore 作成



\### README.md（最低限）



```markdown

\# pc-bootstrap



Windows PC を再現可能にセットアップするための

bootstrap / dotfiles リポジトリ。

```



\### .gitignore（最低限）



```gitignore

.env

\*.key

\*.log

Thumbs.db

```



---



\## Step 6: 最初の commit（GitHub 不要）



```bash

git add .

git commit -m "chore: initialize pc-bootstrap structure"

```



\### 重要



\* \*\*GitHub リポジトリはまだ無くてよい\*\*

\* commit は完全にローカル操作



---



\## Step 7: SSH 鍵の作成（このPC用）



\### 鍵生成



```bash

ssh-keygen -t ed25519 -C "pc-bootstrap-dev"

```



すべて Enter で進む（パスフレーズ空でも可）。



確認：



```bash

ls ~/.ssh

```



期待される状態：



```text

id\_ed25519

id\_ed25519.pub

known\_hosts

```



---



\## Step 8: GitHub に SSH 公開鍵を登録



\### 公開鍵表示



```bash

cat ~/.ssh/id\_ed25519.pub

```



\### GitHub 側操作



\* Settings → SSH and GPG keys → New SSH key

\* Title：`pc-bootstrap-dev`

\* Key type：Authentication Key

\* Key：表示された 1 行を貼り付け



---



\## Step 9: SSH 接続確認



```bash

ssh -T git@github.com

```



成功例：



```text

Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.

```



---



\## Step 10: GitHub リポジトリ作成 → push



\### GitHub Web



\* Repository name：`pc-bootstrap`

\* README / .gitignore は \*\*作らない\*\*



\### ローカル操作



```bash

git remote add origin git@github.com:<account>/pc-bootstrap.git

git branch -M main

git push -u origin main

```



---



\## トラブルシュート集



\### Permission denied (publickey)



\* SSH 鍵が GitHub に未登録

\* `~/.ssh/id\_ed25519.pub` を登録する



\### known\_hosts はあるが接続できない



\* 相手確認のみ完了

\* 自分の鍵が無い／未登録



\### tree が無い



\* Git Bash 前提では外部導入が必要



---



\## 到達状態（完了条件）



\* GitHub に `pc-bootstrap` が存在

\* `git push` が SSH で成功

\* フォルダ構成が確認できる

\* 次の PC でも同手順を再現可能



---



\## 次のステップ（未実施）



\* Terminal 設定の適用

\* キーボード自動切替スクリプト適用

\* PowerShell / Bash 設定の dotfiles 化

\* bootstrap.ps1 の作成



