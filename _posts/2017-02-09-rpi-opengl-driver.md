---
layout: post
title: Raspberry PiのOpenGL Driverを有効化する
---

前々回, X11ForwardingだとOpenGLの
ハードウェアアクセラレーションが有効になっていない
と思われると書いたのですが,
そもそもRaspbianのOpenGL DriverがデフォルトOFFのため
GPUを利用していなかったようです.

GL Driverを有効化するためには
`sudo raspi-config` からAdvanced Settingsを選択して
GL DriverをEnableに切り替える必要がありました.

しかし手元の環境で試したところ,
再起動してGL Driverを有効化した後,
デスクトップがBlack Screenになってしまいました...

色々調べていると, Raspbianのシステムを最新にしておくと良いという話を見つけ,
次に, 各種パッケージを最新にアップデートしてみることにしました.

    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-update
    sudo apt-get autoremove
    sudo reboot

再起動:

    sudo rpi-update
    sudo reboot

アップデート後に再び `GL Driver` を有効化すると,
今度はデスクトップ環境が表示できる状態になりました!

また, OpenGLESのテストコードが
安定して60FPS動作するようになったため
GL Driverの有効化に成功したようです.

ついでにGL Driver有効後,
改めてX11ForwardingでOpenGLESテストコードを実行してみましたが,
やはり1-2FPSしか出ない状態は変わらずでした.

RaspbianのGL Driver有効後でも
X11ForwardingでのGL Driver使用は厳しいようです.

