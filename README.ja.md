## 概要
Ruby on Rails5 の公式サイトを参考に作成したdocker-composeのファイルとセットアップ手順のまとめです。

## ゴール
Linux（Ubuntu20.04 LTS）上に、Rails5 と PostgreSQLのコンテナを連携した開発環境を構築します。

## 前提条件
ドメインを取得済み

Letsencrypt DNS-01 challengeでワイルドカード証明書を取得済み

jwilder/nginx-proxyコンテナでnginx-proxyが稼働済み

.envファイルを編集します


## .envファイルの編集方法
.env ファイルの変数を編集して使います

- USERNAME: ユーザー名を指定する
- UID: user idを指定する
- RUBY_TAG: rubyのバージョンを指定する
- POSTGRE_TAG:  PostgreSQLのバージョンを指定する


## ディレクトリ構造
```
/cert (/home/$USERNAME/certs)
+-- /letsencrypt
|       accounts/
|       archive/
|       accounts/
|       :
|       ...
|       
/docker (/home/$USERNAME/docker)
+-- /example.com
|   +-- /rails5 <= subdomain: rails5.example.com
|       +-- .env
|       +-- docker-compose.yml
|       +-- Dockerfile
|       +-- entrypoint.sh
|       +-- Gemfile
|       +-- Gemfile.lock(empty)
|
+-- /proxy  
|   +-- /log
|       +-- /nginx
|           +-- access.log
|           +-- error.log
|   +-- docker-compose.yml
```

## 使い方・セットアップ手順

1. git clone します
2. .envファイルを編集します
3. rails コマンドで新しいプロジェクトを作成する
```
docker-compose run --no-deps web rails new . --force --database=postgresql
```

4. chownコマンドでパーミッションを変更する
```
sudo chown -R $USER:$USER .
```

5. Gemfileが変更されているためイメージを再構築する
```
docker-compose build
```

6. database.ymlを編集する
```
vi /home/$USERNAME/docker/example.com/raisl6/config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development


test:
  <<: *default
  database: myapp_test
```

7. コンテナを起動してみる
```
docker-compose up
```
dbコンテナは起動しているが、webコンテナからdbコンテナには接続出来ない。
理由：railsのDB設定が完了していないため

8. 一度、起動中のコンテナを停止する
```
Ctrl + C
```

9. webコンテナでデータベースを作成する
```
docker-compose run web rake db:create
```

10. コンテナをバックグラウンドで起動する
```
docker-compose up -d
```

11. ブラウザから https://rails5.example.com　に接続する

## 参考資料
https://docs.docker.com/compose/rails/

https://blog.codecamp.jp/docker-ruby-on-rails-mac

