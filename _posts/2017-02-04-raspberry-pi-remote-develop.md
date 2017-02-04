---
layout: post
title: Raspberry Piリモート開発の試行錯誤
---

普段はMacBookを常用しているため,
Raspberry Piにモニタとマウス/キーボードを接続して
使っていてもどうもしっくり来ませんでした.

折角LinuxでX11環境なので, リモートセッションで
作業すれば良いのではないかと思い試行錯誤を始めました.

まず下準備として, Raspberry PiのSSH接続を許可しておきます.
Raspbian Jessieをインストールした直後は, デフォルトではOFFになっているので,

    sudo raspi-config

をRaspberry Pi上で実行してSSHの設定を切り替えました.
8. Advanced Options -> A4 SSH -> Enable で切り替えることができます.

次に, MacOSはX11環境がデフォルトインストールではないので,
[XQuartz](https://www.xquartz.org/)を公式からインストールしておきます.

あとはMacBookからRaspberry PiにSSH(X11Fowarding付)で接続します:

    ssh -X pi@raspberrypi.local

接続後,

    startlxde-pi &

でRaspberryPiのデスクトップ環境を起動できます.

これでMacBook上からRaspberry Piのデスクトップ環境が使えます!

...が, 試してみたところOpenGLESの実行がすごぶる遅い状態でした.
(60fpsで動作するテスト描画コードが3fpsほどしか出ません...)

どうやらX11FowardingでのOpenGLESは,
ハードウェアアクセラレーションがうまく動作しないと思われます.(要出典)

そのため, この使い方で常用するのは厳しいと思い,
一旦リモートでの開発作業は保留することにしていました.

後日, 調べてみると, どうもRaspberryPiのユーザーは
一般的にはVNCを利用できるそうでした.

Raspbian JessieにはデフォルトでVNCサーバーがインストールされていて,
コンフィグからVNCを有効にすると, VNC Viewerでアクセスできるようになるようです.

VNC Viewerには手元にあったiPad miniを利用しました:

- [VNC Viewer](https://itunes.apple.com/jp/app/vnc-viewer/id352019548?mt=8)

VNCで接続して同様にOpenGLESのテストコードを実行すると...
こちらは8fpsほど確保しました.

若干描画のフレームレートは稼げるものの,
OpenGLESのアプリケーションを開発するには
十分なフレームレートを確保できないと判断しました.

最終的に, モニタはRaspberry PiにHDMIで接続して,
遠隔でGLESアプリケーションを起動する方式はどうだろうかと思い
試してみることにしました.

予めRaspberryPiをモニタに接続して起動させておきます.

次にホストのMacBookからsshで通常接続します.

    ssh pi@raspberrypi.local

SSH接続後, DISPLAYを `:0.0` に指定し直します.

    export DISPLAY=:0.0

あとはOpenGLESのテストコードを実行してみると...

これなら, fps60を確保しました!!

この方法であれば, RaspberryPi用のモニタは必要になるものの,
慣れたMacBook環境からssh経由で開発しつつ実機テストできそうです.

現在はデスクトップPC用の22インチモニタを利用していますが,
ゆくゆくはRaspberry Pi向けの小型液晶モニタに切り替えていきたいと思います.
(これならモニタの場所も取らなくなるので)

