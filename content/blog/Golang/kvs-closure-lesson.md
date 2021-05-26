---
title: 'Golangのクロージャで何が便利なのか理解するためにRedisのようなKVS機能を作る'
date: 2021-05-27 06:57:13
category: 'Go'
draft: false
---

Goの学習をはじめてクロージャという概念の理解に苦しんでました。「クロージャってどんな場面で使うんだろう？何が嬉しくてクロージャを使っているんだろう？」

何に使うか理解出来なものを、理解できないので調べていたらクロージャを使うことでRedisのようなKVSをGoのクロージャで作ることができるようなので、クロージャの理解と合わせてRedisのようなKVSを作ってみます

## Redis

---

Redisとは、key - valueをペアとした値をメモリ上で永続化することができるソフトウェアです。

任意のkeyとvalueをセットすることで、あるvalueが必要なときにセットしたkeyを指定することで呼び出すことが出来ます。

```bash
# key と valueをセット
> set mykey "my value"
OK

# keyに紐づくvalueを呼び出す
> get mykey
"my value"
```

この様な仕組みをGoのクロージャを用いることで実装することが出来ます。

## KVSをクロージャで実装する

---

まずは、任意のkeyとvalueを登録する機能をクロージャで実装してみます。

期待する動作を第1引数で渡し、第2引数と第3引数には key, valueを渡す実装を考えてみます

以下のように実行すれば、対応するkey, valueが保存される想定です。

```go
kvs := closureKvs()

// "命令", "キー", "値"
kvs("ADD", "mykey", "my value")
```

`closureKvs()` という関数は、戻り値として関数を返します。この関数は、命令・キー・値を引数として受け取り、どんな型でも返せるように `interface{}` 型を戻り値として返す関数です。

```go
func closureKvs() func(cmd string, key string, val interface{}) interface{} {

		store := make(map[string]interface{})

		return func(cmd string, key string, val interface{}) interface{} {
		// ...
		}
}
```

第３引数の `val` では、string型でもint型でも受け取れるように `interface{}` としています。

そして、 `closureKvs()` 内に変数storeを作成します。変数storeに指定したキーと値を格納できるように、キーをstring型、値にinterface型を指定しています。

一見、変数storeはローカル変数に見えますが、内部的にはクロージャに属する変数として機能しています。クロージャに属する変数として機能するとは、変数storeが何かしらの形で参照され続ける限り、格納されている値が破棄されることがありません。

通常、関数内のローカル変数は、関数の実行が完了した時点で破棄されるためクロージャによって捕捉された場合の変数と挙動が異なります。

この挙動を理解できずに、クロージャの理解を苦しめていました。では、どの様な変数がクロージャに属する変数としてみなされるのか？という疑問が残りますが、後ほど説明したいと思います。

### キーと値を格納する処理を実装する

---

以下の処理を実行することで、キーと値を保持し続ける `closureKvs()` を実装していきます。

`OK` と返ってこれば、保存は成功しているとみなします。

```go
func main() {
		kvs := closureKvs()

		// "命令", "キー", "値"
		result := kvs("ADD", "mykey", "my value")
		fmt.Println(result) // => OK
}
```

switch文を使って、命令として受け取った文字列を判定して処理を分けていきます。

今回はキーと値を格納する処理なので `"ADD"` を受け取った場合は、 クロージャに属する変数storeに `store[key] = val` として値を代入しています。そして成功したことを知らせる文字列 `"OK"` を返しています。

```go
func closureKvs() func(cmd string, key string, val interface{}) interface{} {
	store := make(map[string]interface{})

	return func(cmd string, key string, val interface{}) interface{} {
		switch cmd {
		case "ADD":
			store[key] = val
			return "OK"
		}

		return nil
	}
}
```

以下で処理確認することができるので任意のキーと値を指定して実行してみてください。

OKと標準出力されれば成功です。

[https://play.golang.org/p/g0fzf1gagBZ](https://play.golang.org/p/g0fzf1gagBZ)

### 格納した値を取得する

---

キーと値を保持する機能が作れたので、次はキーに紐づく値を取得できるようにしていきます。

これで一気にRedisらしくなっていきます。

値を取得したいので次は `"GET"` と命令して、取得したい キーを指定できるように実装していきます。

呼び出し側の処理はこんな感じです

```go
func main() {
		kvs := closureKvs()
		
		// キーと値を格納する
		result := kvs("ADD", "mykey", "my value")
		fmt.Println(result) // => OK

		// 取得したいキーを取得してGETと命令する
		value := kvs("GET", "mykey", "")
		fmt.Println(value) // => my value
}
```

では、GETを渡したときの処理を追加していきます。

`"GET"` という命令を受け取ったとき、変数storeからキーを取得してその値を返す処理を追加することで、対応するキーに対応する値を取得できるようになります。

```go
func closureKvs() func(cmd string, key string, val interface{}) interface{} {
	store := make(map[string]interface{})

	return func(cmd string, key string, val interface{}) interface{} {
		switch cmd {
		case "ADD":
			store[key] = val
			return "OK"
		// ここから
		case "GET":
			getVal := store[key]
			return getVal
		// ここまでを追加
		}

		return nil
	}
}
```

ここで処理を確認できます。

[https://play.golang.org/p/ks4pIVqHPr8](https://play.golang.org/p/ks4pIVqHPr8)

### その他の処理  ...Coming Soon

---

Redisでは、格納されているキーを全件取得する処理や削除だったりがあるので、switch文の条件に追加していくことで実現ができます。

その他処理は時間があるときに、更新して行きたいと思います。

## どの様な変数がクロージャに属するのか？

---

先程、変数storeはクロージャに属する変数として捕捉され、通常のローカル関数とは挙動が異なると説明しましたが、ではどの様な状況で変数はクロージャに属するのかを見ていきます。

```go
func closureKvs() func(cmd string, key string, val interface{}) interface{} {

		// クロージャに属する変数
		store := make(map[string]interface{})

		return func(cmd string, key string, val interface{}) interface{} {
		// ...
		}
}
```

複数の変数を関数内に定義して、どの変数がクロージャに属し、どの変数が通常のローカル変数としてみなされるのか見ていきます。

```go
func checkClosureVariable() func() int {
	a := 1
	b := 5
	c := 10

	return func() int {
		c *= 10
		return c
	}
}
```

上記のクロージャを定義した場合、クロージャに属される変数と補足されるのは変数cのみです。

その他の変数a, cは `checkClosureVariable()` の処理が完了した時点でメモリ上から値が破棄されるが、クロージャに属される変数cは、何かしらの形で参照され続ける限り値を保持し続けます。

## まとめ

---

「Go クロージャ」と調べると、クロージャの説明はたくさんありましたが、実際にどの様な使い方ができるのかのイメージが出来ずに理解に苦しでいました。

ただ、クロージャを利用することでRedisのようなKVSを実装できることを知り、楽しみながらクロージャを理解できた気がします。

参考

書籍: スターティングGO言語

[https://www.okb-shelf.work/entry/2020/04/10/235523](https://www.okb-shelf.work/entry/2020/04/10/235523)