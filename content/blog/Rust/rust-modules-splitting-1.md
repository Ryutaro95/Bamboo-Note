---
title: 'Rustにおけるモジュール分割を理解する'
date: 2022-02-21 23:27:21
category: 'Rust'
draft: false
---

## Rustのファイルによるモジュール分割には２通りある


- src/my_module/mod.rs
- src/my_module.rs と src/my_module/sub_module.rs

上記のどちらかのディレクトリとファイルを用意する方法。
今回は後者の方を試してみる。

まずは、以下の様にmain.rsに複数のモジュールが詰め込まれた状態から、徐々に分割して行く。

```rust
// src/main.rs

mod animals {
    pub mod cat {
        pub fn explanation() -> String {
            String::from("ネコ科の動物で鳴き声はにゃー")
        }
    }
    pub mod dog {
        pub fn explanation() -> String {
            String::from("イヌ科の動物で鳴き声はワン")
        }
    }
    pub mod bird {
        pub fn explanation() -> String {
            String::from("鳥科の動物で鳴き声は様々")
        }
        pub fn name() -> String {
            String::from("鷹")
        }
    }
}

mod zoo {
    pub mod breeder {
        pub fn greet() {
            let bird_name = take_charge();
            println!("私は鳥科の{}を担当しています。", bird_name);
        }
        fn take_charge() -> String {
            use crate::animals::bird;
            bird::name()
        }
    }
}

fn main() {
    use animals::bird;
    use animals::cat;
    use animals::dog;

    println!("{}", cat::explanation());
    println!("{}", dog::explanation());
    println!("{}", bird::explanation());

    use zoo::breeder;
    breeder::greet();
}

// 鳥科の動物で鳴き声は様々
// イヌ科の動物で鳴き声はワン
// ネコ科の動物で鳴き声はにゃー
// 私は鳥科の鷹を担当しています。
```

animalsモジュールを分割するため、 `src/animals` ディレクトリを作成する。

```bash
mkdir src/animals
```

作成したディレクトリ配下に以下の３ファイルを作成。

```bash
touch src/animals/cat.rs src/animals/bird.rs src/animals/dog.rs
```

現在のディレクトリ構造

```bash
src
├── animals
│  ├── bird.rs
│  ├── cat.rs
│  └── dog.rs
└── main.rs
```

main.rs に記載されていた、以下モジュールをディレクトリとファイルへ分割しました。

Rustでは、ディレクトリ名とファイル名がそのままモジュール名として認識されるため、それぞれの関数を、作成した３ファイルに追記していきます。

```rust
// main.rs
mod animals {
    pub mod cat {
        pub fn explanation() -> String {
            String::from("ネコ科の動物で鳴き声はにゃー")
        }
    }
    pub mod dog {
        pub fn explanation() -> String {
            String::from("イヌ科の動物で鳴き声はワン")
        }
    }
    pub mod bird {
        pub fn explanation() -> String {
            String::from("鳥科の動物で鳴き声は様々")
        }
        pub fn name() -> String {
            String::from("鷹")
        }
    }
}
// ...
```

```rust
// src/animals/cat.rs
pub fn explanation() -> String {
    String::from("ネコ科の動物で鳴き声はにゃー")
}
```

```rust
// src/animals/dog.rs
pub fn explanation() -> String {
    String::from("イヌ科の動物で鳴き声はワン")
}
```

```rust
// src/animals/bird.rs
pub fn explanation() -> String {
    String::from("鳥科の動物で鳴き声は様々")
}
pub fn name() -> String {
    String::from("鷹")
}
```

これでモジュールをディレクトリ・ファイル毎に分割することが出来ました。

次は、main.rsでこのモジュールを呼び出せるように、main.rsと同じ階層に以下ファイルを作成します。

これは、ディレクトリ名と同等のモジュール名のファイルを作成する必要がある。

```bash
touch src/animals.rs
```

```bash
src
├── animals
│  ├── bird.rs
│  ├── cat.rs
│  └── dog.rs
├── animals.rs // module名と同じのファイル名を用意
└── main.rs
```

作成した、animals.rs にサブモジュールを呼び出します。

```rust
// src/animals.rs
pub mod cat;
pub mod dog;
pub mod bird;
```

次に、animals.rs を main.rs を呼び出してみます。

```rust
// src/main.rs
mod animals;

mod zoo {
    pub mod breeder {
        pub fn greet() {
            let bird_name = take_charge();
            println!("私は鳥科の{}を担当しています。", bird_name);
        }
        fn take_charge() -> String {
            use crate::animals::bird;
            bird::name()
        }
    }
}

fn main() {
    println!("{}", animals::bird::explanation());
    println!("{}", animals::dog::explanation());
    println!("{}", animals::cat::explanation());

    use zoo::breeder;
    breeder::greet();
}

// 鳥科の動物で鳴き声は様々
// イヌ科の動物で鳴き声はワン
// ネコ科の動物で鳴き声はにゃー
// 私は鳥科の鷹を担当しています。
```

最初と同じ出力を確認出来たので、これでモジュール分割が出来ました。

次は zoo/breeder モジュールを同様の方法で分割していきます。

```bash
mkdir srczoo/
touch src/zoo/breeder.rs
touch src/zoo.rs
```

```bash
src
├── animals
│  ├── bird.rs
│  ├── cat.rs
│  └── dog.rs
├── animals.rs
├── main.rs
├── zoo // モジュール名でディレクトリを作成
│  └── breeder.rs // サブモジュール名でファイル作成
└── zoo.rs // モジュール名ディレクトリと同じファイルを作成
```

```rust
// src/zoo/breeder.rs
pub fn greet() {
    let bird_name = take_charge();
    println!("私は鳥科の{}を担当しています。", bird_name);
}

fn take_charge() -> String {
    use crate::animals::bird;
    bird::name()
}
```

```rust
// src/zoo.rs
pub mod breeder;
```

分割出来たので、zooモジュールを呼び出します。

```rust
// src/main.rs
mod animals;
mod zoo;

fn main() {
    println!("{}", animals::bird::explanation());
    println!("{}", animals::dog::explanation());
    println!("{}", animals::cat::explanation());

    use zoo::breeder;
    breeder::greet();
}

// 鳥科の動物で鳴き声は様々
// イヌ科の動物で鳴き声はワン
// ネコ科の動物で鳴き声はにゃー
// 私は鳥科の鷹を担当しています。
```
