---
title: 'Rubyで配列の配列を合計したいときはinjectが便利'
date: 2020-06-15 06:06:37
category: 'Ruby'
draft: false
---


例えば次のような配列の配列があり、数字の部分だけを合計して出力したかった時、injectメソッドが便利だったので残しておきます。

```
cards = [["スペード", 5], ["スペード", 9], ["ハート", 7]]
```

<br>

```ruby
puts cards.inject(0) { |sum, card| sum += card[1] }
#=> 21
```
見事に文字列の部分を省いて、数値の合計だけを出力してくれました。

<br>


いままではこんな感じで、数値の部分を配列に入れ直したり
```ruby
arr = []
cards.each do |card|
  arr << card[1]
end
puts arr.sum
#=> 21
```
<br>

こんな感じだったり
```ruby
# 上のinjectとやっていることは同じ
sum = 0
cards.each do |card|
  sum += card[1]
end
pust sum
#=> 21
```


みたいな方法で合計を取得していました。