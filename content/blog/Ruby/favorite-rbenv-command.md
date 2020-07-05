---
title: 'よく使うrbenvコマンド一覧'
date: 2020-07-05 15:12:32
category: 'Ruby'
draft: false
---


よく使うrbenvのコマンドの一覧を備忘録として残しておきます。

<br>

## 環境

```bash
MacOS Catalina 10.15.1
rbenv 1.1.2
```

<br>

## インストール済みのRubyバージョン一覧を確認
```bash
$ rbenv versions
```

<br>

## 現在稼働しているRubyバージョンを確認
```bash
$ rbenv version
2.6.5 (set by /Users/Username/.rbenv/version)

```

<br>

## インストール可能なRubyバージョン一覧
```bash
$ rbenv install -l

.
.
.
2.6.1
2.6.2
2.6.3
2.6.4
2.6.5
2.7.0-dev
2.7.0-preview1
2.7.0-preview2
2.7.0-preview3
2.7.0-rc1
2.7.0-rc2
2.7.0
2.8.0-dev
```


<br>

## 指定したバージョンのRubyをインストール
```bash
$ rbenv install 2.7.0
```

<br>

## ディレクトリごとのバージョンを指定する
```bash
$ rbenv local 2.7.0
```

<br>

## グローバルで使用するバージョンを変更
```bash
$ rbenv global 2.7.0
```

<br>

