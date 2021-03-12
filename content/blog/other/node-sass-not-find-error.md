---
title: 'nodeを再インストールしたらnode-Sassが見つからないと怒られた、ERROR #98123 WEBPACK ... Node Sass could not find a binding for your current environment'
date: 2020-07-08 23:10:31
category: 'other'
draft: false
---


## 環境
```
$ node -v
v14.5.0

macOS Catalina: 10.15.1
```


nodeを再インストールした結果、タイトルのエラーが発生した可能性があります。  

## 原因

次のコマンドを実行した結果Webpackでバンドルできないという内容のエラーが発生しました。エラーの原因は`node sass`が次のパスで見つからないことが原因のようです。  

`/Users/username/bamboo-note/node_modules/node-sass/vendor/darwin-x64-83/binding.node`


<br>


```bash
$ gatsby develop

 ERROR #98123  WEBPACK

Generating development JavaScript bundle failed

Missing binding /Users/username/bamboo-note/node_modules/node-sass/vendor/darwin-x64-83/binding.node
Node Sass could not find a binding for your current environment: OS X 64-bit with Node.js 14.x

Found bindings for the following environments:
  - OS X 64-bit with Node.js 13.x

This usually happens because your environment has changed since running `npm install`.
Run `npm rebuild node-sass` to download the binding for your current environment.

File: src/styles/code.scss
```


<br>

## 解決方法

`node sass`が見つからないようなので、エラーメッセージで指示されている次のコマンドを実行  

```bash
$ npm rebuild node-sass
```


<br>

これで`gatsby develop`が正常に起動しました。