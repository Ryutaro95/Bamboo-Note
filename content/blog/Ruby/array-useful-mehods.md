---
title: '【Ruby】配列を操作するときに便利そうなメソッド達「compact, flatten, flat_map, index, min/max...など」'
date: 2020-07-13 17:35:43
category: 'Ruby'
draft: false
---

## compact/compact! | nilを取り除く

```ruby
a = [1, nil, 'abc', false]
b = a.compact
b #=> [1, 'abc', false]
```

<br>

## empty? | 配列内の要素が空かを調べる

配列の要素数が0 の場合に **true**

```ruby
[1, 2, 3].empty?  #=> false
[].empty?         #=> true
```

<br>

## flatten/flatten! | 入れ子になった配列を平坦にする

```ruby
a = [[1, 2], [3, 4], [5, 6]]
a.flatten #=> [1, 2, 3, 4, 5, 6]
```

<br>

## flat_map | mapとflattenを組み合わせたようなメソッド

mapを使うと配列の配列（ネスト）した状態で返ってくる

```ruby
sentences = ['Ruby is fan', 'I love Ruby']
sentences.map { |s| s.split(' ') }
#=> [["Ruby", "is", "fan"], ["I", "love", "Ruby"]]
```

<br>

ネストされていないフラットな配列が返ってくる

```ruby
sentences = ['Ruby is fan', 'I love Ruby']
sentences.flat_map { |s| s.split(' ') } #=> ["Ruby", "is", "fan", "I", "love", "Ruby"]
```

<br>

## index / find_index | index番号を返す

引数と一致する要素が最初に見つかったときに添え字を返す

```ruby
a = [10, 20, 30, 10, 20, 30]
a.index(30) #=> 2
```


ブロックを使うと戻り地が最初に真になった要素の添字が返ってくる

```ruby
a = [10, 20, 30, 10, 20, 30]
a.index { |n| n >= 20 } #=> 1
```

<br>

## include? | 特定の要素が含まれているかを調べる

引数と等しい要素が配列に含まれる場合に `true`

```ruby
a = [10, 20, 30]
a.include?(20) #=> true
a.include?(50) #=> false
```

<br>

## join | 配列内の要素を連結する

配列の各要素を１つに連結した文字列を返す

```ruby
words = ['apple', 'melon', 'orange']
words.join #=> "applemelonorange"
```

<br>

引数に文字列を渡すとその文字列が挟み込まれる

```ruby
words = ['apple', 'melon', 'orange']
words.join(', ') #=> "apple, melon, orange"
```

<br>

## min / max / minmax  | 最大・最小を取得

配列内の`min`は最小の`max`は最大の要素を取得

```ruby
a = [3, 6, 2, 1, 5, 4]
a.min #=> 1
a.max #=> 6
```

<br>

`minmax`は最小の要素と最大の要素を配列に入れて取得

```ruby
a = [3, 6, 2, 1, 5, 4]
a.minmax #=> [1, 6]
```

<br>

## reverse / reverse! / reverse_each

`reverse`メソッドは配列を逆順に並び替えす

```ruby
a = [1, 2, 3, 4]
a.reverse #=> [4, 3, 2, 1]
```

<br>

 `reverse_each`メソッドは逆順に繰り返し処理を行うメソッド

```ruby
a = [1, 2, 3, 4]
a.reverse_each.map { |n| n * 10 } #=> [40, 30, 20, 10]
```

<br>

## shuffle / shuffle! / sample | 要素をシャッフル / 要素を取り出す

 `shuffle`メソッドは配列の要素をランダムに並び替えるメソッド 

```ruby
a = [1, 2, 3, 4, 5 ,6]
a.shuffle #=> [2, 4, 5, 6, 3, 1]
```

<br>

 `sample`メソッドは配列からランダムに1つの要素を取り出します。引数を渡すと、その個数だけランダムに取り出した要素（ただし重複したインデックスは選択しない）を返します。 

```ruby
a = [1, 2, 3, 4, 5 ,6]
a.sample #=> 2
a.sample(3) #=> [3, 5, 4]
```

<br>

## sort / sort! | 要素を並び替える

 `sort`メソッドは配列の要素を順番に並び替えた新しい配列を返します。 

```ruby
a = [5, 2, 6, 3, 1, 4]
a.sort #=> [1, 2, 3, 4, 5, 6]
```

<br>

## sum | 配列内の合計

 配列の中身の合計値を求める

```ruby
[1, 2, 3, 4].sum
#=> 10
```

<br>

 デフォルトの初期値は0ですが、引数で変更できます。 

```ruby
[].sum
#=> 0

[1, 2, 3, 4].sum(5)
#=> 15
```

<br>

 文字列の配列を連結する場合は初期値に何らかの文字列を渡します。 

```ruby
['foo', 'bar'].sum('')
#=> 'foobar'

['foo', 'bar'].sum('>>')
#=> '>>foobar'
```

<br>

配列の配列を連結することも可能



```ruby
[[1, 2], [3, 1, 5]].sum([])
#=> [1, 2, 3, 1, 5]
```

<br>

## uniq / uniq! | 重複を取り除いた配列を返す

```ruby
a = [1, 1, 2, 2, 3, 4]
a.uniq #=> [1, 2, 3, 4]
```

<br>

## transpose | 配列の配列の行と列を入れ替える

配列の配列を行列として見立て、行と列を入れ替えた配列を返す

```ruby
a = [[10, 20], [30, 40], [50, 60]]
a.transpose #=> [[10, 30, 50], [20, 40, 60]]
```

<br>

## zip | 配列と配列を同じインデックス同士で組み合わせる





```ruby
# テストに回答した生徒の名前
names = ['Alice', 'Bob', 'Carol']
# 各生徒の点数
points = [80, 90, 60]
# 名前と点数を組み合わせる
names.zip(points) #=> [["Alice", 80], ["Bob", 90], ["Carol", 60]]
```

<br>

ここからさらに`to_h`メソッドを使うとハッシュに変換でき、生徒の名前で点数を取得することができます 

```ruby
point_table = names.zip(points).to_h #=> {"Alice"=>80, "Bob"=>90, "Carol"=>60}
point_table['Alice'] #=> 80
point_table['Carol'] #=> 60
```

<br>

`zip`メソッドの引数は可変長引数になっているので、2つ以上の配列を渡すこともできます。 

```ruby
lengths = [1, 2, 3]
widths = [10, 20, 30]
heights = [100, 200, 300]
lengths.zip(widths, heights) #=> [[1, 10, 100], [2, 20, 200], [3, 30, 300]]
```


<br>
