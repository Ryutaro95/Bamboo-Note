---
title: '【Rails】ActiveRecordの`find_by`で大文字と小文字を区別しないで取得する方法'
date: 2020-07-21 18:17:32
category: 'Ruby'
draft: false
---


## はじめに

`find_by`メソッドで値を取得する際にハマったので解決方法を探してみました。  

```
Rails : 6.0
Ruby: 2.7
SQLite3
```

以上の環境で確認検証しました。  

<br>

## 検索対象と検索したい文字列を大文字に変換

保存されたタグを取得する際、大文字と小文字の違いで期待した値を取得することができない。  

```bash
$ rails console 
 # 大文字と小文字を混ぜた状態で保存します
 > Tag.create(tag_name: "RuBy")
 >
 # 小文字で先程作成したタグ名を検索してみます
 > Tag.find_by(tag_name: "ruby")
 #=> nil
```

<br>

Tags テーブルに保存された値は`"RuBy"`と保存しているため`find_by`メソッドで`"ruby"`と検索しても取得することができません。  

ユーザーがタグを検索したい場合、`"ruby"`や`"Ruby"`、`"RUBY"`など様々な入力で検索する可能性があるため、大文字と小文字で区別されてしまうのは不便です。  

<br>

大文字と小文字を区別せず`find_by`メソッドで取得するためには、SQL の`upper()`関数を使いカラムに含まれる文字列を大文字に変換して、検索したい文字列も Ruby の `upcase`メソッドで大文字に変換して取得します。  

```bash
$ rails console
# 大文字と小文字を混ぜた状態で保存します
 > Tag.create(tag_name: "RuBy")
 >
 # 検索対象のカラムと検索したい文字列をそれぞれ大文字に変換
 > Tag.find_by('UPPER(tag_name) = ?', "ruby".upcase)
 #=> tag_name: "RuBy"
```

これで Tags テーブルの値を大文字と小文字を区別することなく取得することができました。  

`'UPPER()'`と大文字にしているのは SQL 的な記述のためなので、特に意味はありません `upper()`としても動作します。  
