データの編集を行うプログラムを書いていた際に,  
データにアクセスするプログラムが,  
お互いにデータ編集があったことを気にしながら処理できるように,  
リスト構造で管理できないかと思いついたネタです.

```
(use srfi-1)

(define (hlist obj)
  (list obj))

(define hlist-current last)
(define hlist-current-pair last-pair)

(define (hlist-update! proc lst)
  (let* ((prev (hlist-current-pair lst))
         (next (hlist (proc (car prev)))))
    (set-cdr! prev next)
    next))
    
(define (hlist-updated? lst)
  (not (null? (cdr lst))))
```

プログラムの途中で変更を受けることがあるデータを表すために,  
`hlist` (名前は適当) という構造で包みます.

このリスト上では, `hlist-current` にあるものが最新の値で,  
それ以外は計算途中を格納したアドレスになります.

`hlist-update!`による更新はリストを辿って,  
常に最新の値に対して適用されます.

```
(define (append-1! lst)
  (append! lst (list 1)))

; hlistオブジェクト
(define obj (hlist '()))

; 副作用のあるコード
(let loop ((data obj) (repeat 5))
  (when (< repeat 0)
    (loop
      (hlist-update! append-1! data)
      (- repeat 1))))

; hlistから情報を回収
(hlist-updated? obj) ; ==> #t
(hlist-current obj) ; ==> (1 1 1 1 1)
```

`hlist-update!`に線形更新な手続きを渡せるように,  
便宜上, `last`でないコンスセルの`car`は,  
有効な値ではないということにしておきます.

(または,`proc`や継続のような付加情報で埋めておくことで,  
 履歴を検索する属性にするのもいいかもしれないです)

