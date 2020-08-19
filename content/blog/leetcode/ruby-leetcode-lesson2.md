---
title: 'LeetCodeでRubyとアルゴリズムを学びたい#2【66. Plus One】'
date: 2020-08-19 18:05:34
category: 'leetcode'
draft: false
---


今回の問題が**「Plus One」**という問題に挑戦しました。

次のような整数の配列が渡されるので、`+ 1`して返す関数を作るというのが問題です。

**Example 1:**
```
Input: [1, 2, 3]
Output: [1, 2, 4]
```
<br>

**Example 2:**
```
Input: [4, 3, 2, 1]
Output: [4, 3, 2, 2]
```

渡された配列を整数に変換して`1`を足して、返すだけなので簡単に実装できそうです。


[66. Plus One | LeetCode](https://leetcode.com/problems/plus-one/)


<br>

こんな感じの関数を作ります。
```ruby
def plus_one(digits)
# ...
end

plus_one([1, 2, 3]) #=> [1, 2, 4]
```

<br>



## 型を変換して`+ 1`して型を変換して返す


処理の手順をステップ毎に分割すると次のようなイメージです。  

```
Input
[1, 2, 3]
↓
"123"
↓
123
↓
124 = 123 + 1
↓
"124"
↓
["1", "2", "4"]
↓
Output
[1, 2, 4]
```

<br>

## 実際のコード

結合、型変換、分割だらけの実装になってしまいました。

```ruby
def plus_one(digits)
  (digits.join.to_i + 1).to_s.chars.map(&:to_i)
end
```


<br>

渡された配列を結合して整数に変換して`+ 1`する
```ruby
# [1, 2, 3] -> "123" -> 123 + 1 -> 124
(digits.join.to_i + 1) #=> 124
```

<br>

あとは`+ 1`された整数を配列に戻して返すだけです。
```ruby
# 124 -> "124" -> ["1", "2", "3"] -> [1, 2, 4]
124.to_s.chars.map(&:to_i) #=> [1, 2, 4]
```