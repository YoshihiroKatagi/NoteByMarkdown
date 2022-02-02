# AWS Cloud9 環境構築

___

## AWSアカウント準備

___

## Cloud9のプロジェクト作成

___

## サーバー環境設定

___

## Laravel 環境構築

___
___

## データベース 環境構築


### MariaDBのインストール

```
# MariaDBのステータス確認
$ systemctl status mariadb
→「Unit mariadb.service could not be found.」と出力される。

# MariaDBのインストール実行
$ sudo yum install mariadb mariadb-server
→エラーなくインストールが完了する。(最後に「Complete!」と出力される。）

# MariaDBのステータス再確認
$ systemctl status mariadb
→インストールされたMariaDBの情報が出力される。(状態は「inactive」となっている。）

# MariaDBの起動
$ sudo systemctl start mariadb
$ systemctl status mariadb
→MariaDBの状態が「active」となっている。

# MariaDBの接続確認
$ mysql -u root
→正常に接続ができる。

# DB接続を終了する。
MariaDB [(none)]> exit;
→「Bye」の出力と共にLinuxのターミナルに戻る。
```

___

### MariaDBの初期設定

#### 文字コード設定

```
# 現在の文字コード設定を確認
$ mysql -u root -e "show variables like 'char%';"
→表示される結果に「utf8mb4」の設定が存在しない。

# DBのutf8mb4対応確認
$ mysql -u root -e "show charset like '%utf8mb4%';"
→「utf8mb4」の情報が一行出力されればOK。

# DBクライアント側の文字コード変更
$ sudo vim /etc/my.cnf.d/mysql-clients.cnf
→「[mysql]」ブロックに「default-character-set=utf8mb4」を追記する。

# DBクライアント側の文字コードの設定反映確認
$ mysql -u root -e "show variables like 'char%';"
→表示される結果の「character_set_client」「character_set_connection」
「character_set_database」「character_set_result」が「utf8mb4」になっていればよい。

# DBサーバー側の文字コード変更
$ sudo vim /etc/my.cnf.d/mariadb-server.cnf
→「[mysqld]」ブロックの最後に「character-set-server=utf8mb4」

# DBサーバーの再起動
$ sudo systemctl restart mariadb
$ systemctl status mariadb
→MariaDBの状態が「active」となっている。

# DBクライアント側の文字コードの設定反映確認
$ mysql -u root -e "show variables like 'char%';"
→表示される結果の「character_set_server」が「utf8mb4」となっていればよい。
```

#### 自動起動設定

```
# 起動時にMariaDBが自動起動しないことの確認
$ sudo reboot
→再起動が実行される。

$ systemctl status mariadb
→MariaDBの状態が「inactive」となっている。

# 自動起動設定
$ sudo systemctl enable mariadb
→「Created symlink from 〜」というメッセージが出力される。

# 起動時にMariaDBが自動起動することの確認
$ sudo reboot
→再起動が実行される。

$ systemctl status mariadb
→MariaDBの状態が「active」となっている。
```

___

### DBユーザーの初期設定

#### Rootユーザーのパスワード設定

```
# MariaDB接続&Database切り替え
$ mysql -u root
→mysqlにrootユーザでログイン

MariaDB [(none)]> use mysql;
→mysqlに切り替えが行われる

# userテーブルのrootユーザーに対するパスワード値を更新
MariaDB [mysql]> ALTER USER 'root'@'localhost' IDENTIFIED BY '********';

# 設定したパスワードの確認
MariaDB [mysql]> exit
$ mysql -u root -p
→passwordを聞かれるので設定したpasswordを入力。
```

#### 新規ユーザー作成

```
# MariaDB接続&Database切り替え
$ mysql -u root -p
→mysqlにrootユーザでログイン

MariaDB [(none)]> use mysql;
→mysqlに切り替えが行われる

# ユーザーの新規作成
# ユーザー名（今回はdbuserとした）とパスワードを設定
MariaDB [mysql]> create user 'dbuser'@'localhost' identified by '********';
→「Query OK, 0 rows affected (0.00 sec)」と出力されれば成功

# 作成ユーザーへのCreate権限付与
MariaDB [mysql]> grant create on *.* to 'dbuser'@'localhost';
→「Query OK, 0 rows affected (0.00 sec)」と出力されれば成功

# 作成ユーザーの確認
MariaDB [mysql]> exit
$ mysql -u dbuser -p
→passwordを聞かれるので設定したpasswordを入力。
```

___

### データベース作成

```
# MariaDB接続&Database切り替え
$ mysql -u root -p
→mysqlにrootユーザでログイン

MariaDB [(none)]> use mysql;
→mysqlに切り替えが行われる

# 新規データベース作成
MariaDB [mysql]> show databases;
→現時点でのデータベース一覧が表示される。

MariaDB [mysql]> create database TodoApp;
→「Query OK, 1 row affected (0.00 sec)」が出力されればOK

MariaDB [mysql]> show databases;
→作成したデータベースが一覧に表示される。

# ユーザーへの新規データベース権限付与
MariaDB [mysql]> GRANT ALL PRIVILEGES ON TodoApp.* to 'dbuser'
→「Query OK, 0 rows affected (0.00 sec)」と出力されれば成功

# 新規データベース接続確認
MariaDB [mysql]> exit
$ mysql -u dbuser -p TodoApp
→パスワード入力後、正常に接続され「MariaDB [TodoApp]>」と出力されれば成功
```

___
___

## Git 環境構築

### Gitパッケージのバージョン確認&最新化

```
# 現時点でインストールされているGitパッケージのバージョン確認
$ git --version

# Gitパッケージの更新実行
$ sudo yum update git

# Gitパッケージのバージョン再確認
$ git --version
```

___

### GitHub間の疎通設定

#### サーバー側のSSH公開鍵・秘密鍵作成

```
# 作成前のSSH関連ディレクトリ配下確認
$ ls -l ~/.ssh/

# 公開鍵・秘密鍵のキーペア作成
$ ssh-keygen -t rsa

# 作成後のSSH関連ディレクトリ配下確認
$ ls -l ~/.ssh/
→「id_rsa.pub」が公開鍵、「id_rsa」が秘密鍵のファイル
```

#### SSH公開鍵をGitHubに設定

GitHubにログイン（未登録の場合は新規アカウント作成）
↓
SSH and GPG keys
↓
New SSH key
↓
title: 何目的の公開鍵か識別できるようにタイトルを設定
Key: 「id_rsa.pub」ファイルの内容をコピーし貼り付け
↓
Add SSH key

#### サーバー側SSH設定&疎通確認

```
# SSH設定ファイルの作成 or 変更
$ vim ~/.ssh/config
```

以下の内容を追記
```
Host github
  HostName github.com
  IdentityFile ~/.ssh/id_rsa
  User 自分のGitHubユーザー名
```

保存してLinuxターミナルに戻る
```
:wq
```

疎通確認
```
# SSH設定ファイルの権限設定
$ chmod 600 ~/.ssh/config

# GitHub接続確認
$ ssh -T git@github.com
→「Hi 自分のGitHubユーザー名! You've successfully authenticated, but GitHub does not provide shell access.」
と表示されれば成功
```

___

### Gitコマンドの初期設定

```
$ git config --global user.name "GitHubのユーザ名"

$ git config --global user.email "GitHubの登録アドレス"

$ git config --global core.editor 'vim -c "set fenc=utf-8"'

$ git config --global color.diff auto

$ git config --global color.status auto

$ git config --global color.branch auto
```

___

### Gitリポジトリ初期設定

**GitHubリポジトリ作成**
Your repositories
↓
New
↓
Repository name: TodoApp(何のレポジトリ識別できる名前)
Description: 内容の説明（任意）
Public or Private: どちらでもよい
↓
Create repository

**サーバー側設定**

```
# カレントディレクトリに対するGit初期設定
$ git init

# バージョン管理状態を確認
# (※パスワードなどの不要ファイルが含まれていないか確認する）
$ git status

# ソース管理されていない全ファイルをステージングに追加
$ git add .

# バージョン管理状態を再確認
$ git status // もう一度確認

# コミットを実行
# (※コミットコメントはどのような変更かを理解できるように）
$ git commit -m "first commit"

# ブランチ名をmasterからmainに変更
$ git branch -M main

# リモートリポジトリの設定
$ git remote add origin git@github.com:ユーザ名/リポジトリ名.

# リモートリポジトリへの反映
$ git push origin main
```

___

### ※開発中のGitコマンド操作

#### コミット&プッシュの流れ

開発中には常にキリのいいところでコミット&プッシュすることを心掛ける。
細かくコミット&プッシュすることで、作業のやり直しなどがしやすい。
コミットだけでも問題ないが、同時にプッシュまでしておくのが無難。
```
$ git add .

$ git commit -m "ここには変更点などを書く"

$ git push origin main
```

#### ブランチを切って作業する

平行して複数の機能追加やバージョン管理を行う場合、それぞれの作業用ブランチを作成しそこで作業する。

##### ブランチの作成
```
$ git branch
→ローカルリポジトリのブランチ一覧が表示される

# 作業用ブランチ作成
$ git branch branch_for_function1
→mainリモートブランチからbranch_for_function1ローカルブランチを作成

$ git checkout branch_for_function1
→作業ブランチを「branch_for_function1」に切り替え

$ git branch
→ローカルブランチ一覧が表示され、作業ブランチが「branch_for_function1」になっている
```

##### 作業ブランチをプッシュ
```
$ git add .

$ git commit -m "コメント"

$ git push origin branch_for_function1
```

##### GitHub上でのブランチマージ

Compare & Request ボタンを押す
↓
Create pull request ボタンを押す
↓
Confirm merge ボタンを押す

##### リモートリポジトリの変更をローカルリポジトリのmainに反映

```
$ git pull origin main
```


___
___

## Laravel 初期設定

### 環境設定ファイル準備

```
# Laravelアプリケーションのルートディレクトリに移動
$ cd ~/environment/TodoApp

# 用意されている .env.exampleファイル(サンプルファイル)を .envファイルにコピー
$ cp .env.example .env

# .envファイルにアプリケーションキーを設定
$ php artisan key:generate

# .envファイルのDB接続関連情報を編集
$ vim .env
```
「DB_」で始まる値を適切な形に変更
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=TodoApp（作成したデータベース名）
DB_USERNAME=dbuser（DBで設定したユーザー名）
DB_PASSWORD=********（設定したパスワード）
```
保存してLinuxターミナルに戻る
```
:wq
```

### タイムゾーン設定

`config/app.php`内で定義されているtimezoneの値を変更する。
```
'timezone' => 'Asia/Tokyo',
```