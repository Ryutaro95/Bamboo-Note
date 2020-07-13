---
title: 'RubyプログラミングでRSpecを使う'
date: 2020-07-13 17:20:32
category: 'Ruby'
draft: false
---

# RubyプログラミングでRSpecを使う
Railsではなく純粋なRubyプログラムにRSpecを使ってみます。  

<br>

## 環境

```
ruby: 2.7.0
rspec: 3.9.0
```

## RSpecをインストール

RSpecをインストールする。このとき依存関係のあるGemも同時にインストールされます。  

```bash
$ gem install rspec
# ...省略
Done installing documentation for rspec-support, rspec-core, diff-lcs, rspec-expectations, rspec-mocks, rspec after 3 seconds
6 gems installed
```

<br>

インストールされたRSpec関連のGem  
計6つのGemがインストールされました。  

```bash
rspec-support
rspec-core
diff-lcs
rspec-expectations
rspec-mocks
rspec
```

## RSpec --init

インストール後に初期化して初期ファイルを作成する。  

```bash
$ rspec --init
  create   .rspec
  create   spec/spec_helper.rb
```

<br>


### .rspec

`rspec`コマンドのオプションを記述する設定ファイルです。  
`.rspec`に記述されているオプションをコマンドの実行時に自動付与されます。  

.rspec
```
--require spec_helper
--format documentation
```


<br>

`--format documentation`とすることでテストの実行結果がドキュメント形式で出力され見やすくなる。  

その他のオプションは次のコマンドで確認できます。  
```bash
$ rspec --help
```

<br>

## テスト実際に書いてみる
RSpecが動作するかをテスト実際に書いて確認してみます  


specファイルを作成して簡単なテストを書いて行きます。  
```bash
touch spec/sample_spec.rb
```

<br>

次の内容は後で作成する`add`メソッドが4を返すことを期待するテストです。

```rb
require './sample'

RSpec.describe do
  it '4であること' do
    expect(add).to eq 4
  end
end
```


<br>

実際に`add`メソッドを作成していきます。

```bash
$ touch sample.rb
```

<br>


まずは単純に4だけを返すメソッドを作成します。

```ruby
def add
  4
end
```


<br>

テストを実行してみます。  
```bash
$ rspec

4であること

Finished in 0.00272 seconds (files took 0.19695 seconds to load)
1 example, 0 failures
```

テストがパスしました。  
これで純粋なRubyプログラムでRSpecを使えるようになりました。