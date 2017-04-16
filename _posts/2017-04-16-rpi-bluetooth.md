---
layout: post
title: RaspberryPiのBluetoothコントローラ
---

Raspberry Pi 3 Model B は, Wireless LANだけでなくBluetoothも搭載しています.

これまではUSBゲームパッドを接続して利用していたのですが,
折角Raspberry Pi本体が手のひらサイズで持ち運びしやすいのに,
優先で色々つなげてしまうとがちゃがちゃしてしまいます.

というわけで小型のBluetoothコントローラを買ってみました.

- [8Bitdo Zero Gamepad](https://www.amazon.co.jp/gp/product/B06XSL67CN/)

カラーがRaspberry Pi公式モデルケースに似ているので気に入りました.
8ボタン+方向キーのモデルです.

欲を言えば, このサイズだと方向キーや4ボタンは押しにくいので
スティック+2ボタン+電源ボタンのモデルが欲しいのですが,
なかなか求めているデザインは見つからないものです.

ペアリング後, GLFW3経由でジョイスティックとして値を取得出来たので
今後はこれをメインに操作を実装していきます.

