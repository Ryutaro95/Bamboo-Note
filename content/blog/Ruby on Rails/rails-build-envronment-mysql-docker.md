---
title: 'すぐにRails6 x MySQL x docker-compose環境構築'
date: 2020-11-08 2:33:32
category: 'Ruby on Rails'
draft: false
---

よく動作確認や学習用でRailsプロジェクトを作成したいときがあるので、備忘録として残しておきます。

<br>

ファイルの作成が面倒なときは Githubに用意してあるので、クローン後READMEに従って環境構築を行ってください。
[quick build rails6 mysql environment | github](https://github.com/Ryutaro95/quick_build_rails6_mysql_envronment/tree/master)


<br>

## ファイルの用意


以下のファイルを任意のディレクトリに用意

- Gemfile
- Gemfile.lcok
- Dockerfile
- docker-compose.yml

```bash
# vscode
$ code Gemfile Gemfile.lock Dockerfile docker-compose.yml

# or

$ touch Gemfile Gemfile.lock Dockerfile docker-compose.yml
```

Dockerfile

```docker
FROM ruby:2.6.5 # 好きなRubyバージョンを指定

ENV LANG C.UTF-8
ENV APP_ROOT /api

RUN apt-get update -qq && apt-get install -y  build-essential \
                                              default-mysql-client \
                                              nodejs

RUN mkdir ${APP_ROOT}
WORKDIR ${APP_ROOT}

COPY Gemfile ${APP_ROOT}/Gemfile
COPY Gemfile.lock ${APP_ROOT}/Gemfile.lock

RUN bundle install

COPY . ${APP_ROOT}
```

docker-compose.yml

```yaml
version: '3'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    volumes:
      - .:/api
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    tty: true
    stdin_open: true
  db:
    image: mysql:5.7
    volumes:
      - mysql_data:/var/lib/mysql/
    environment:
      MYSQL_ROOT_PASSWORD: password
    ports:
      - "3306:3306"
volumes:
  mysql_data:
```

Gemfile

```ruby
source 'https://rubygems.org'

gem 'rails', '~> 6'
```

Gemfile.lock

```ruby
# 空のまま
```

## rails new


```bash
$ docker-compose run api rails new . --force --database=mysql --skip-bundle --api
```

`new .` : カントディレクトリにRailsプロジェクトを作成する

`--force` : ファイルが存在するときに上書きする

`--database=mysql` : mysqlを利用する

`--skip-bundle` : bundle をスキップする

`--api` : Rails6だと、webpackerとか面倒なのでAPIモードで起動

## イメージ作成



```bash
$ docker-compose build
```

## イメージからコンテナを立ち上げる


```bash
$ docker-compose up # -d
```

バックグラウンドで立ち上げたいときは `-d` オプションをつける

起動できれば成功です。

MySQLのパスワードなどを隠したい場合は `.env` を作成して別途指定できます。