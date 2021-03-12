---
title: 'LeetCodeでRubyとアルゴリズムを学びたい#1【67. Add binary】'
date: 2020-08-18 19:08:95
category: 'other'
draft: false
---

<br>

今まではpaizaのスキルチェックで試行錯誤してプログラミングの練習をしていました。ただpaizaのルール上ソースコードの公開ができず、他の人のコードを見ることが出来ません。

そこで見つけたのがLeetCodeなるGAFAなどで出題されているコーディングテストの問題が公開されているサービスを発見したので、挑戦してみました。

<br>



## 2つの2進数を足して合計を返したい

今回は挑戦した問題が「Add Binary」という問題でした。詳細は下記を参照してください。

[67. Add Binary | LeetCode](https://leetcode.com/problems/add-binary/)

2つの2進数を受け取り、合計を2進数で返すメソッドを作成すればいい問題です。


<br>

渡される値と返す値はこんな感じです。

**Exapmple 1:**
```
Input: a = "11", b = "1"
Output: "100"
```

**Example 2:**
```
Input: a = "1010", b = "1011"
Output: "10101"
```

<br>

こんなメソッドを作ればいいんでしょ？  

```ruby
def add_binary(a, b)
# ...
end

add_binary("1010", "1011")
#=> "10101"
```
「簡単そう...」と初めは思っていたんですが、気づいたら1時間以上もこの問題でハマってしまったので、復習を兼ねたアウトプットをしてきます。



<br>

## 実際に1時間以上も掛けたコード

```ruby
def add_binary(a, b)
  length = a.size > b.size ? a.size : b.size
  result, carry = [], 0


  1.upto(length) do |i|
    sum = a[i * -1].to_i + b[i * -1].to_i
    sum += carry

    result.unshift((sum % 2).to_s)
    carry = sum / 2
  end

  return result.unshift(carry.to_s).join unless carry.zero?
  result.join
end
```

<br>

2進数足し算の筆算をやっているイメージです。

```
  1 <- carry
 1010 <- a
+  10 <- b
------
 1100 <- result
```

<br>

処理を順を追って文章化すると以下になります。

1. a, bの桁数が多い方の数を`length`に代入します。
   これは後でループ処理するのに利用します

2. 結果を格納する配列（result）と繰り上がってしまった`1`を運ぶ（carry）を初期化しておきます。

3. `upto`メソッドで 1 ~ length回ループを回します。
  ```
  1.upto(5){ |i| print "#{i}, " }
  #=> 1, 2, 3, 4, 5
  ```
  こんな感じで`upto`は繰り上がってくれるので、その数字を`i * -1 => -i`で負の整数に変換します。
  `"1010"[i * -1] => "0"`
  `"10"[i * -1] => "0"`
  これで、末尾から1桁づつ取り出して計算した結果を`sum`に代入できるようになります。
  `carry`の値を`sum`に追加する。これは前回のループでの値を`carry`で持ってきてもらうためです。
  次に、`sum % 2`で余りを文字列に変換し`unshift`で先頭から`result`に追加していきます。
  `carry = sum / 2`で繰り上がった部分を次のループに持ち越します。

  ```
    1 <- ここをcarryで運ぶイメージ
   1010
  +  10
  ------
     1100
  ```

<br>

4. ループを抜け`carry`の値が`0`であれば、`join`した結果を返し。`carry`が`0`意外であれば、`unshift`で追加した結果を`join`して返します。

<br>

## to_i(2)を使うことでOnelineでコーディング出来たみたい

[Discuss]で色々コードを見てみようとした結果、ワンラインでのコーディングを見つけてしまいました。

```ruby
def add_binary(a, b)
    (a.to_i(2) + b.to_i(2)).to_s(2)
end
```

参考: [One line Ruby solution | LeetCode](https://leetcode.com/problems/add-binary/discuss/24650/One-line-Ruby-solution)


<br>

`to_i(2)`や`to_s(2)`で簡単に`10進数 <-> 2進数`が出来てしまうみたいです。Rubyすごい


<br>

## 時間はかかったけど、色々気づきがあった

Onelineコードを見たときは、ちょっとびっくりしましたが、これが他者の解答を見れるいいところですね。

ただ、時間を掛けて試行錯誤した結果、使ったことがなかった・知らなかった方法を理解することが出来たのでいい感じです。

習得した内容
```ruby
- upto / downto  # downtoとか繰り返し処理めっちゃ豊富
- "abcd"[2] => c # 単に文字列でも配列のように取り出しが出来る
- 0.zero?        # 0 == 0 と同じ
```
