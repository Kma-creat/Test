# AWSフルコース第3回 作成日:2022.10.20

### Ruby on Rails概要
Railsとは、プログラミング言語「Ruby」で書かれたWebアプリケーションフレームワークです。Railsは、あらゆる開発者がWebアプリケーション開発で必要となる作業やリソースを事前に想定することで、Webアプリケーションをより手軽に開発できるよう設計されています。アプリケーションを開発する際のコード量がより少なくて済むにもかかわらず、より多くの機能を実現できます。

---
### 1. データの反映
・git cloneでアプリ格納場所のリポジトリをローカルに落とす

```
$ git clone <HTTPSのURL>
```

### 2. Rubyのバージョン確認
・Railsのバージョン確認
```
$ ruby -v
```

### 3. MySQLのインストール

```
sudo yum -y update
sudo yum remove -y mysql-server
sudo yum remove -y mariadb*
MYSQL_PACKAGE_URL="https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm"
sudo yum localinstall -y $MYSQL_PACKAGE_URL
sudo yum install -y mysql-community-devel
sudo yum install -y mysql-community-server
```

・ MySQLサーバーの起動＆確認
```
sudo service mysqld start && sudo service mysqld status
```

・ 初期パスワードの確認
```
sudo cat /var/log/mysqld.log | grep "temporary password" | awk '{print $13}'
# MySQLにログイン（パスワードが表示されるので、必ずコピーしておく）
mysql -u root -p
```

### 4.Mysqlのパスワードの修正

・パスワードの修正は下記の記事を参考

https://qiita.com/ttyoku/items/f7b5c763cbe5ff80eed4


### 5. database.ymlのファイルの修正（usernameやパスワードの追加）

**開発環境のコードを修正する必要があるので、修正
→これをやらないと、エラー沼にハマるので注意**

```
# database.ymlのファイル内を修正
adapter: mysql2
encoding: utf8
database: ~~~~~~（デフォルト名）
pool: 5
username: root（今回は開発環境なのでroot）
password: MySQLインストール後に変更したパスワード（これないとデプロイできない）
host: localhost
```

上書き保存をする

**注意**：Railsには、「開発」「テスト」「本番」の3つの環境があり、今回のデプロイは「開発環境」でのデプロイを想定


### 6. データベース（DB）の作成

MySQLのインストールと、database.ymlの編集が終わったらいよいよデータベースの作成に入る


下記コマンドでconfig/database.ymlの設定内容にをもとにデータベースが作成される

```
$ rails db:create
```

余談だが、本番環境のデータベースを作成したい場合は
```
$ rake db:create RAILS_ENV=production
```


migrationファイルの内容をDBに反映する
→メリットは、SQL構文を書かずともDBにアクセスしてデータをいじれることらしい

→まだよくわかってないが、下記記事を参照

https://qiita.com/ichihara-development/items/9e9a8454178947eb519d

```
$ rails db:migrate
```


### 7. Rails アプリを起動

アプリをデプロイします。
```
・rails s
```

プレビューでアプリを見てましょう！

・preview→preview Running Applicatonで起動


・起動が遅い、できない場合は下記のコマンドで不要なコンテナを削除
```
$ docker system prune -a
```

・無事デプロイ成功！！

## 遭遇したエラー
1. 「rails s」で起動時「Can't connect to local MySQL server through socket '/tmp/mysql.sock' (13)」が発生

→「database.yml」のコードをいじる必要ある

○Before
```
# database.ymlのファイル内を修正
adapter: mysql2
database: ~~~~~~（デフォルト名）
```

○After
```
# database.ymlのファイル内を修正
adapter: mysql2
encoding: utf8
database: ~~~~~~（デフォルト名）
pool: 5
username: root（今回は開発環境なのでroot）
password: MySQLインストール後に変更したパスワード（これないとデプロイできない）
host: localhost
```

2. 容量不足でアプリの起動が遅い
→`$ docker system prune -a`で解決。できれば最初にやっておいた方がいいみたい


## 今回の課題で使用したコマンドたち
```
git clone
cd raisetech-live8-sample-app/
sudo yum -y update
sudo yum remove -y mysql-server
sudo yum remove -y mariadb*
MYSQL_PACKAGE_URL="https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm"
sudo yum localinstall -y $MYSQL_PACKAGE_URL
sudo yum install -y mysql-community-devel
sudo yum install -y mysql-community-server
sudo service mysqld start && sudo service mysqld status
bundle install
ruby -v
mysql -u root -p
rails db:create
rails db:migrate
rails s
docker system prune -a
```




## 課題
- APサーバ、DBサーバ、構成管理ツールの名前とバージョン

| 種類 | 項目名 | バージョン |
| --- | --- | --- |
| APサーバ | Puma | 8.0.30 |
| DBサーバ | MySQL | 8.0.30 |
| 構成管理ツール | Bundler | 2.3.22 |

- APサーバを終了させる -> URLにアクセスできない
```
Ctrl + Cを入力
```

- APサーバを再開させる -> URLにアクセスできる。
```
bundle exec rails s
```

- DBサーバを終了させる -> DBにアクセスできない
```
sudo service mysqld stop
```

- DBサーバを再開させる -> DBにアクセスできる。
```
sudo service mysqld start
```

- DBサーバ状態確認
```
sudo service mysqld status
```

---

## Rails環境構築
1. サンプルコードのクローン

```
git clone https://github.com/yuta-ushijima/raisetech-live8-sample-app.git
```

2. MySQLのインストール
  - CLoud9のEC2インスタンスの容量追加
    - https://blog.proglus.jp/4574/
  - yumが使えればLinux、aptが使えればUbuntu
  - GPGキーの有効期限切れ対策
    - https://blog.katsubemakito.net/mysql/mysql-update-error-gpg

3. database.ymlを環境に合わせて書き換える
```
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: 'root'
  password: 'パスワード' # ここを書き換える
development:
  <<: *default
  database: raisetech_live8_sample_app_development
  socket: /var/lib/mysql/mysql.sock  # ここを書き換える
  #socket: /tmp/mysql.sock
```

4. 構成管理ツールbundlerのインストール
```
gem install bundler
```

5. 必要なGemのインストール
```
bundle install
```

6. DBを作成、マイグレーションを実行して起動する
```
bundle exec rails db:create
bundle exec rails db:migrate
bundle exec rails s
```

7. Blocked Hostのエラーが出るのでdevelopment.rbに下記を追記。追記するホスト名はエラー画面に表示されているものをコピーする。

config/environment/development.rb
```
config.hosts << "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.vfs.cloud9.ap-northeast-1.amazonaws.com"
```

8. webpackのエラーが出たら下記コマンドを実施。yarnが入って無ければyarnもインストール
    - https://qiita.com/NaokiIshimura/items/8203f74f8dfd5f6b87a0

```
npm install -g yarn
bundle exec rails webpacker:install 
```

9. もう一度APサーバを起動する
```
bundle exec rails s
