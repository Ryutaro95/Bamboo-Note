---
title: 'Rustでcliを作る - コマンドライン引数を受け取りたい'
date: 2021-09-12 10:10:21
category: 'Rust'
draft: false
---

Rustでcliツールを作る際にコマンドライン引数を標準ライブラリとサードパーティライブラリのclapを使ってみる。

## 標準ライブラリで引数を受け取る


まずは標準ライブラリで引数を受け取る。  
std::env::args()を使うことでコマンドライン引数をイテレータで取得できる。

collect()でイテレータをコレクションに変換できるが、collect()は型注釈が必要みたい。

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    println!("{:?}", args);
}

$ cargo run -- ひきすう
...
["target/debug/argscli", "ひきすう"]
```

渡した引数は2番目以降に含まれるみたい。1番目は `cargo run` でビルドされたパスみたいなので以下でも実行可能

```rust
$ ./target/debug/argscli ひきすう
["./target/debug/argscli", "ひきすう"]
```

collect()で型注釈以外の方法があるか確認したところ、turbofishを使うと注釈は不要みたいだけどこれなら注釈したほうが良さそう。

[https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)

```rust
let args = env::args().collect::<Vec<String>>();
```

## clapを使ってみる


[https://github.com/clap-rs/clap](https://github.com/clap-rs/clap)

clapを使うとオプション引数とかも簡単にできそうなので、標準ライブラリよりこっちを使えば良さそう。

```rust
$ cargo add clap@3.0.0-bata.4
```

clapにはderiveを使った方法とBuilderを使った方法の2スタイルがあるみたい。

今回はBuilderの方を使って、コマンドライン引数にファイル名を指定して、ファイル内のテキストを取得してみる。

App::new("...")に続くversion(), authour()関数などを使ってこのcliの情報を設定する。

arg()でファイル名を受け取って index(1) とするとでコマンドライン引数の１つ目にファイルパスを受け取ることを指定している。

get_matches()で指定されたコマンドライン引数と照合して、その結果を if let のところで取り出している

```rust
use clap::{App, Arg};
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() {
    let matches = App::new("MyApp")
        .version("1.0")
        .author("Ryutaro <ryutaro@gmail.com>")
        .about("Does awesome things")
        .arg(
            Arg::new("output")
                .about("Sets an optional output file")
                .index(1),
        )
        .get_matches();
    
    if let Some(o) = matches.value_of("output") {
        println!("Value for output: {}", o);
    }

    if let Some(path) = matches.value_of("output") {
        let file = File::open(path).unwrap();
        let reader = BufReader::new(file);

        for line in reader.lines() {
            let line = line.unwrap();
            println!("{}", line);
        } 
    } else {
        // 引数が指定されていない場合
        println!("No file is specified");
    }
}

```

プロジェクトのルートに出力用のtxtファイルを用意する

```rust
$ touch output.txt
// output.txt
Hello world!!
```

作成したファイル名を指定すると、ファイル内の文字列を出力できた！

```rust
$ cargo run ouput.txt
Value for output: output.txt
Hello world!!
```

ファイル指定せずに実行してみるとメッセージが表示される

```rust
$ cargo run
No file is specified
```