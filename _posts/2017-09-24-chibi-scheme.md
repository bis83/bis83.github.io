---
layout: post
title: chibi-scheme見習い
---

普段はScheme処理系はChicken Schemeを使っており,
自分の利用目的と大体マッチしていることもあり,
他の処理系を使うことがあまりなかったです.

知見を広げるため, 他の処理系を使い始めてみようと思い,
今回はchibi-schemeを使ってみることにしました.

- [chibi-scheme](http://synthcode.com/wiki/chibi-scheme)

公式によるとchibi-schemeは主にCプログラムの拡張や
スクリプト言語として利用するためのライブラリとして
開発されている処理系のようです.

気に入ったポイントは:

- C組み込み用途でミニマルな実装らしい
- R7RS-small
- 複素数がきちんとサポートされている
- 公式でEmscriptenサポート, `make js` で `chibi.js` を生成できる

### ビルド

```
git clone https://github.com/ashinn/chibi-scheme.git
cd chibi-scheme/
make
make install
```

REPLの起動は `chibi-scheme` です:

```
chibi-scheme
> (+ 1 2)
3
> (exit)
```

### 最初の作業

まずはこれから利用していくにあたって,
普段使いしている `glfw3` をライブラリとして
利用出来るかテストしてみます.

glfw3はC向けのライブラリなので,
chibi-schemeのFFI機能を利用してバインディングしていくことになります.

FFI機能の使い方はchibi-schemeのドキュメントが参考になりますが,
[公式のライブラリ](https://github.com/ashinn/chibi-scheme/tree/master/lib/chibi)
でも多用されているため, こちらがサンプルコードとして活用出来ました.

### stubファイルの作成

まず `glfw.stub` というファイルを作成します.

```scheme
(c-system-include "GLFW/glfw3.h")
(define-c void (init "glfwInit") ())
(define-c void (poll-events "glfwPollEvents") ())
(define-c (pointer void) (create-window "glfwCreateWindow")
  (int int string (maybe-null pointer void) (maybe-null pointer void)))
```

このstubファイルはFFIバインディング用の
動的ライブラリを生成するためのソースコードです.

今回はとりあえず動作テストのため,
ウィンドウの出すまでの最小限の関数のみバインディングします.

バインディングのためC関数の戻り値と引数の型を指定する必要があります.
`maybe-null` を指定しておくと, Scheme側から `#f` を渡すと
NULLポインタとして処理してくれます.

### ビルドスクリプト

先程のstubファイルから動的ライブラリを作成します.

ビルドは二工程になり, まず `chibi-ffi` でstubファイルをcファイルに変換して,
cファイルを環境の `cc` で動的ライブラリに変換します.

`cc` の使い方と必要なオプションはプラットフォーム毎に異なります.

今回テストしたローカルのMacOS上では,
下記のビルドスクリプトで行いました.

```scheme
(import (scheme base)
        (chibi process))

(system "chibi-ffi" "glfw.stub")
(system "cc" "-fPIC" "-dynamiclib"
        "glfw.c"
        "-o" "glfw.dylib"
        "-lchibi-scheme"
        "-lglfw3"
        "-framework" "Cocoa"
        "-framework" "IOKit"
        "-framework" "CoreVideo")
```

ビルドスクリプトはMakefileでもシェルでも良いですが,
折角なのでchibi-schemeのprocessライブラリを利用しました.

作成した動的ライブラリファイルは,
chibi-schemeの `load` から直接読み込んで使用することが出来ます.

```scheme
> (load "glfw.dylib")
> init
#<opcode "init">
```

このままでも使用できますが,
動的ライブラリの拡張子はプラットフォーム毎に異なりますし,
グローバルな名前空間を汚染してしまうので, ライブラリ化します.

### sldファイルの作成

chibi-schemeはR7RSサポートなので, `define-library` が使えます.

ライブラリ定義用のファイルは `glfw.sld` と命名します.
この `.sld` 拡張子にすることで, chibi-schemeの処理系が `import` 時に
自動的に環境からライブラリを検索してきて読み込んでくれるようになります.

```scheme
(define-library (glfw)
  (export init
          poll-events
          create-window)
  (include-shared "glfw"))
```

`include-shared` 構文がchibi-schemeの拡張で,
先程stubから作成した動的ライブラリを読み込んでくれます.

### テスト

`glfw-test.scm` ファイルを作成して,
作成したライブラリをテストで使ってみます.

```scheme
(import (scheme base)
        (prefix (glfw) glfw:))

(glfw:init)
(define w (glfw:create-window 320 240 "test" #f #f))
(let loop ()
  (glfw:poll-events)
  (loop))
```

これを `chibi-scheme glfw-test.scm` で実行してテスト出来ます.

### 次に

これでよく利用していたCライブラリを
chibi-schemeから呼び出すことが出来たので,
今後もchibi-scheme上で開発を続けることが出来そうです.

触った感想としては, 組み込みライブラリ用途として開発されているだけあって,
ビルド周りは非常に簡素な印象を感じました.

今後chibi-schemeで開発していくにあたって,
Cプログラムを生成する手順やCコンパイラを使ったビルド手順を
どうやって実装するかに悩みそうです.

