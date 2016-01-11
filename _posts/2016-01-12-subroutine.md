---
layout: post
post: 返り値を使わない手続きの呼び出し
---

新年1発目. ブログを移転してからあまり投稿できていないので, 今年はエントリを増やしていきたいところです.

手続き的なプログラムや副作用を前提とした処理を書いていて,
setter手続きやdisplay手続きのように, 返り値は意味を持たない手続きを呼び出したいが, その引数は後の計算で使いたい時があります.

例えば:

```scheme
;;; レコードやベクタなどのデータ構造
;;; (ここでは例として2次元上の点データ)
(define (make-position) (cons 0 0))
(define position        car+cdr)
(define set-position-x! set-cdr!)
(define set-position-y! set-cdr!)

;;; ユーザによる初期化コード
(define my-position
  (let ((obj (make-position))
    (set-position-x! obj 10)
    (set-position-y! obj 20)
    obj))
    
(position my-position) ; ==> [10, 20]
```

のようなものです. 

このようにいくつかの返り値を考慮しない手続きを呼び出ししつつ, 引数をそのまま継続に受け渡すパターンはよくありそうです. 返り値を考慮しない手続きを, ここではfortranに倣って`subroutine`と呼ぶことにします.

Common Lispでは利便性重視のためと思われますが`write`は引数を返し, `setf`は新しい値を返すようにしてあるようです. Schemeは機能の直交性を重視した言語仕様とされており, 変数の再代入や出力手続きは返り値を`unspecified-value`と規定しています. (機能の直交性に関する解釈はまとまりがついたらテキストに書き起こしてみたいネタです)

そうは言ってもコードを書く際に頻出しているパターンを放置したくないものなので, ここはひとつシンプルな糊を定義してみます.

名前は良いものが思いつかなかったため, 説明的なものにしています.

```scheme
(define (call-with-subroutine data subroutine)
  (subroutine data)
  data)
```

先ほどのコードを書き換えてみます:

```scheme
(call-with-subroutine (make-position)
  (lambda (obj)
    (set-position-x! obj 10)
    (set-position-y! obj 20))
```

ついでにもう一つ, あるオブジェクトに対していくつかの手続きを実行するlambda式に対しても名前付けしてみます.

```scheme
(define-syntax compose-subroutine
  (syntax-rules ()
    ((_ (subroutine . arguments) ...)
      (lambda (obj) (subroutine obj . arguments) ...))))
```

```scheme
(call-with-subroutine (make-position)
  (compose-subroutine
    (set-position-x! 10)
    (set-position-y! 20)))
```

この二つの糊を利用することで, 一時的な変数名を減らしつつサブルーチン的な呼び出しをコード上に組み入れることが見やすくならないかと思います.