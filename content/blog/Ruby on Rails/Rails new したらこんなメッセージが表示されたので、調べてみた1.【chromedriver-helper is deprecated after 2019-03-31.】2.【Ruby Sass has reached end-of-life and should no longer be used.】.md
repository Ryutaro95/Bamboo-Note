---
title: 'Rails new したらこんなメッセージが表示されたので、調べてみた1.【chromedriver-helper is deprecated after 2019-03-31.】2.【Ruby Sass has reached end-of-life and should no longer be used.】'
date: 2020-06-11 00:06:93
category: 'Ruby on Rails'
draft: false
---





## 環境
```
ruby '2.6.5'  
gem 'rails', '~> 5.2.3'  
gem 'mysql2', '>= 0.4.4', '< 0.6.0'  
```


## メッセージ
```
 +--------------------------------------------------------------------+
  |                                                                    |
  |  NOTICE: chromedriver-helper is deprecated after 2019-03-31.       |
  |                                                                    |
  |  Please update to use the 'webdrivers' gem instead.                |
  |  See https://github.com/flavorjones/chromedriver-helper/issues/83  |
  |                                                                    |
  +--------------------------------------------------------------------+

Post-install message from sass:

Ruby Sass has reached end-of-life and should no longer be used.

* If you use Sass as a command-line tool, we recommend using Dart Sass, the new
  primary implementation: https://sass-lang.com/install

* If you use Sass as a plug-in for a Ruby web framework, we recommend using the
  sassc gem: https://github.com/sass/sassc-ruby#readme

* For more details, please refer to the Sass blog:
  https://sass-lang.com/blog/posts/7828841
```

こんなメッセージが表示されたので、とりあえずGoogle翻訳に頼ってみました

- 翻訳

```
 +--------------------------------------------------------------------+
  |                                                                    |
  |  注意：chromedriver-helperは2019-03-31以降廃止されます                   |
  |                                                                    |
  |  代わりに 'webdrivers' gemを使用するように更新してください。                   |
  |  https://github.com/flavorjones/chromedriver-helper/issues/83      |
  |  を御覧ください                                                        |
  +--------------------------------------------------------------------+

Post-install message from sass:

Ruby Sassはサポート終了になったため、使用しないでください。

* Sassをコマンドラインツールとして使用する場合は、新しいDart Sassを使用することをお勧めします
   主要な実装：https://sass-lang.com/install

* SassをRuby Webフレームワークのプラグインとして使用する場合は、
   sassc gem：https://github.com/sass/sassc-ruby#readme

*詳細については、Sassブログを参照してください。
   https://sass-lang.com/blog/posts/7828841
```


ここで表示されているメッセージで言おうとしていることは以下の2つみたいです

1.  **chromedriver-helper はサポート終わるから、代わりに `webdrivers` gem を使ってね**
2.  **Ruby Sass はサポート終了になるから対応策を載せておくよ。それぞれ対応してね**

ってことみたいです


## 1. chromedirver-helper から webdrivers へ移行する

Gemfileにデフォルトで` gem 'chromedriver-helper' `と記述されているので、` gem 'webdirvers' `へ書き換える


`Gemfile`
```ruby

group :test do
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '>= 2.15'
  gem 'selenium-webdriver'

  # Easy installation and use of chromedriver to run system tests with Chrome
  # gem 'chromedriver-helper' # この行を削除
  
  gem 'webdrivers' # この行を追加
end

```

` chromedriver-helper `について、` webdrivers `への詳しい移行手順を下の記事で載せてくれていますので、参考にさせてもらいました。

[サポートが終了したchromedriver-helperからwebdrivers gemに移行する手順 - Qiita](https://qiita.com/jnchito/items/f9c3be449fd164176efa)


## 2.sass-rails から sassc-rails へ書き換える

Gemfileの` gem 'sass-rails', '...'`の部分を `gem 'sassc-rails'`へ書き換える 

<br>

`Gemfile`
```ruby
# gem 'sass-rails', '~> 5.0' この行を削除
gem 'sassc-rails' # この行を追加

```

保存後` bundle install `を実行

これで先程のメッセージは表示されなくなりました。


[【rails5】Ruby Sass is deprecated and will be unmaintained as of 26 March 2019. というメッセージ - Qiita](https://qiita.com/mah666hhh/items/17cfbeb58efdd242253b)
上記の記事を参考にしました。こちらの記事では`Gemfile.lock`内の`sass`となっている箇所の削除となっていますが、自分は削除を行わず`bundle install `を実行してみました。

<br>

`Gemfile.lock`
```ruby

# 省略

sassc (2.2.1)
  ffi (~> 1.9)
sassc-rails (2.1.2)
  railties (>= 4.0.0)
  sassc (>= 2.0)
  sprockets (> 3.0)
  sprockets-rails
  tilt

# 省略

DEPENDENCIES

# 省略

sassc-rails
# 以下省略
```

`bundle install `後 Gemfile.lock ファイルを確認すると`sass-rails` から ` sassc-rails`へ書き換わっている

` rails server `を実行サーバーが起動したので、正常に動作してるっぽいです。
また何かあれば追記していこうと思います。



## 参考
- [サポートが終了したchromedriver-helperからwebdrivers gemに移行する手順 - Qiita](https://qiita.com/jnchito/items/f9c3be449fd164176efa)
- [【rails5】Ruby Sass is deprecated and will be unmaintained as of 26 March 2019. というメッセージ - Qiita](https://qiita.com/mah666hhh/items/17cfbeb58efdd242253b)
- [sassc-rails【github】](https://github.com/sass/sassc-ruby#readmef)
