---
title: 'Rubyで配列の配列を合計したいときはinjectが便利'
date: 2020-06-15 06:06:37
category: 'Ruby'
draft: false
---


例えば次のような配列の配列があったとして、数字の部分だけを合計して出力したい場合はinjectメソッドが便利でした。

```
cards = [["スペード", 5], ["スペード", 9], ["ハート", 7]]
```

```ruby
puts cards.inject(0) { |sum, card| sum += card[1] }
#=> 21
```
見事に文字列の部分を省いて、数値の合計だけを出力してくれました。

<br>


いままでは次のように、数値の部分を配列に入れ直したり
```ruby
arr = []
cards.each do |card|
  arr << card[1]
end
puts arr.sum
#=> 21
```
<br>

だったり
```ruby
sum = 0
cards.each do |card|
  sum += card[1]
end
pust sum
#=> 21
```

のような方法で合計を取得していました。