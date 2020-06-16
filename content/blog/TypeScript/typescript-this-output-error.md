---
title: "メソッド内で使用しているthisをTypeScriptに認識させてエラーを出力してもらいたい"
date: 2020-06-17 06:06:73
category: "TypeScript"
draft: false
---


## 環境
VScode: 1.46.0  
node: 13.8.0  
TypeScript: 3.8.2


## this の仕様
TS/JS の`this`は定義時に参照するオブジェクトが決まるのではなく、*実行された場所*によって決まる。  
そしてTypeScriptはメソッドの中で`this`が使われていることを通常認識しないので思わぬ結果が表示されてしてしまう場面があります。

例としてPersonクラスを定義してその中で自己紹介をする`greeting`メソッドを定義したとします。

例)
```typescript
class Person {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  greeting() {
    console.log(`こんにちは、私の名前は${ this.name }です。`);
  }
}

const yamada = new Person('山田');
yamada.greeting();

```

<br>

- 実行


```bash
$ tsc index.ts
$ node index.js
こんにちは、私の名前は山田です。
```

期待通りの内容が出力されます。

<br>

ここで`Person`クラス内の`name`プロパティをコメントアウトすると、当たり前ですが`name`プロパティが無いぞ！とエラーが発生します。


```typescript
class Person {
  // コメントアウトする
  // name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  greeting() {
    // thisが参照しているPersonクラスにnameプロパティが無いためエラーが発生する
    console.log(`こんにちは、私の名前は${ this.name }です。`);
  }
}

const yamada = new Person('山田');
yamada.greeting();
```

<br>

- 実行


```bash
$ tsc index.ts
error TS2339: Property 'name' does not exist on type 'Person'.
```

ここでエラーに気づいて`name`プロパティを追加することができます。

<br>


## クラス外のオブジェクトでメソッドを呼び出す時もエラーを発生させたい

`this`は定義時ではなく実行時に参照するオブジェクトが変化するため別のオブジェクト内でも`name`プロパティがない場合、同じように`name`プロパティが無いよ！と怒って欲しいがエラーを出力してくれません。

確認のため新たに別のオブジェクトを作成してPersonクラス内の`greeting()`メソッドを呼び出してみます。

<br>

```typescript
class Person {
  
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  greeting() {
    console.log(`こんにちは、私の名前は${ this.name }です。`);
  }
}

const yamada = new Person('山田');

// 別のオブジェクトを作成する
const fullName = {
  fullNameGreeting: yamada.greeting
}
fullName.fullNameGreeting();

```

<br>

- 実行

```bash
$ tsc index.ts
$ node index.js
こんにちは、私の名前はundefinedです。
```

この場合、エラーが発生せず実行結果を見てみると`undefined`となっています。
なぜPerson内に`name`プロパティが無い時はエラーとなってくれたのに、作成した`fullName`内に`name`プロパティがなくてもエラーとならず`undefined`となってしまうのか。

TypeScriptはメソッドの中で`this`が使われていることを知らない（認識できていない）エラーとなってくれません  
そこで今回はTypeScriptに`greeting`メソッドの中で`this`が使われていることを明示的に認識させてあげます。

方法は次のとおりです。

<br>

`greeting`メソッドの第1引数に`this: { name: string }`を入れてコンパイルしてみます。
```typescript
class Person {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  // 第一引数に this を入れて型を指定します
  greeting(this: { name: string }) {
    console.log(`こんにちは、わたしの名前は${ this.name }です`);
  }
}

const yamada = new Person('山田');
yamada.greeting();

const fullName = {
  fullNameGreeting: yamada.greeting
}
fullName.fullNameGreeting();
```

<br>

- 実行

```bash
$ tsc index.ts
error TS2684: The 'this' context of type '{ fullNameGreeting: (this: { name: string; }) => void; }' is not assignable to method's 'this' of type '{ name: string; }'.
  Property 'name' is missing in type '{ fullNameGreeting: (this: { name: string; }) => void; }' but required in type '{ name: string; }'.
```

エラーとなってくれました！これで「あっ!nameプロパティ入れ忘れた」または「呼び出し側のnameプロパティを探しちゃってるじゃん」となれるわけです。  
確認のため`name`プロパティを追加して実行がしてみます

```typescript
class Person {
 .
 .
 .
  // 第一引数に this を入れて型を指定します
  greeting(this: { name: string }) {
    console.log(`こんにちは、わたしの名前は${ this.name }です`);
  }
}

const yamada = new Person('山田');

const fullName = {
  // エラーが出たのでnameプロパティを追加
  name: "山田 太郎",
  fullNameGreeting: yamada.greeting
}
fullName.fullNameGreeting();
```

<br>

- 実行

```bash
$ tsc index.ts
$ node index.js
こんにちは、わたしの名前は山田 太郎です
```

コンパイル時にエラーが表示されることで、未然に`undefined`を防ぐことができました。
