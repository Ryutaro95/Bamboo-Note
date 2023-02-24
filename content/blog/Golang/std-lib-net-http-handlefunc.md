---
title: 'Goのnet/httpでHandlerとかHandleFuncとかHandleとか色々出てきたので整理してみる'
date: 2023-02-24 20:19:20
category: 'Go'
draft: false
---

net/httpに登場する。主要な型とインタフェースと関数/メソッド
* Handler     **インタフェース**
* HandlerFunc **型**
* ServeMux    **型**
* ServeHTTP   **メソッド**
* Handle      **関数**
* HandleFunc  **関数**

まず、登場人物の名前とそれが何であるかを一覧することで、どれも名前が似ているため混乱していましたが、少しだけ整理することができました。

では、一つずつ確認していきます。

## type Handler インタフェースとは

```go
type Handler interface {
    ServeHTTP(RespnoseWriter, *Request)
}
```
ServeHTTP メソッドを持つだけのインタフェースです。つまり、HTTPハンドラ(Handler)として振る舞うためは、ServeHTTPメソッドを実装する必要があることが分かります。

## func HandleFunc 関数とは
HandleFunc関数にパターンとHTTPハンドラを渡すことで、リクエストを受け取りレスポンスを返すことが出来るようになります。  
実装は以下のようになっています。
```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}
```

なるほど、では、 Hello World! を返す簡単なwebサーバを作成してみます。
```go
package main

import "net/http"

func Hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World!"))
}

func main() {
	http.HandleFunc("/hello", Hello)
	http.ListenAndServe(":8080", nil)
}
```

```bash
$ curl localhost:8080/hello
Hello World!
```

期待するレスポンスが返ってきたことが分かります。ですが、なぜ通常の関数である Hello はHTTPハンドラとして振る舞えているのか？  
HTTPハンドラとして振る舞うためには ServeHTTP メソッドを実装している必要があるのに

これを知るために、実装を順に追っていこうと思います。  
まずは、HandleFunc 関数です。
```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}
```

DefaultServeMux.HandleFuncへのラッパーのようなので、さらに確認していきます。
```go
// HandleFunc registers the handler function for the given pattern.
// HandleFunc は、与えられたパターンに対するハンドラ関数を登録する。
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}

	mux.Handle(pattern, HandlerFunc(handler))
}
```
DefaultServeMux は ServeMux 型のようです。  
そしてさらに、mux.Handle と HandlerFunc を見てみます。

まずは、 mux.Handle から
```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
    // パターンとハンドラをmapに格納している
	e := muxEntry{h: handler, pattern: pattern}
	mux.m[pattern] = e
	if pattern[len(pattern)-1] == '/' {
		mux.es = appendSorted(mux.es, e)
	}

	if pattern[0] != '/' {
		mux.hosts = true
	}
}
```

メソッド内の以下処理を見てみると、mapへハンドラとそれに対応するパターンが格納されるのが分かります。  
先程の mux.HandleFunc メソッドのコメントにあった 「HandleFunc は、与えられたパターンに対するハンドラ関数を登録する」の部分がこの処理で行われていそうです。
```go
e := muxEntry{h: handler, pattern: pattern}
mux.m[pattern] = e
if pattern[len(pattern)-1] == '/' {
    mux.es = appendSorted(mux.es, e)
}
```

では、次に HandlerFunc の確認です。  

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers. If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler that calls f.

// HandlerFunc 型は通常の関数を
// HTTP ハンドラとして使用できるようにするためのアダプタです
// fが適切なシグネチャを持つ関数である場合
// HandlerFunc(f)はfを呼び出すHandlerとなります。
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```
実装を見てみると、HandlerFunc型あり、ServeHTTPメソッドが実装されています。そして、コメントには適切なシグネチャを持つ関数をHTTPハンドラとして使用出来るようにするためのアダプタとあります。  
最初の疑問である、 Hello 関数はいつどこでHTTPハンドラとなったのか？という疑問はここで解決されました。 

つまり、Hello という通常の関数はここでHTTPハンドラとして振る舞えるようになったようです。


長くなったので一旦ここまで。