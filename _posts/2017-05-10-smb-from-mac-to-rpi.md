---
layout: post
title: MacBookからRaspberryPiにフォルダ共有
---

こでまで, MacBookからsshでRaspberry Piにログインして,
Raspberry Piでvimを立ち上げてファイル編集をしていましたが,
ssh経由のせいかRaspberry Piのスペックのせいか動作が重たくなることがありました.

Raspberry Pi上ではcsi (Chicken Scheme Interpreter) を主に実行しているので,
テキストエディタ等になるべくリソースを割かないようにしたいところです.

また, テキストエディタは一番頻繁に利用するツールなので,
そちらの動作が軽快なほうが良いだろうと思い付き,
vimはMacBook側で起動する構成を取ることにしました.

流れとしては,

1. MacBookのディスク上にGitリポジトリをクローン
2. Gitリポジトリがあるフォルダをファイル共有(SMB)で公開
3. RaspberryPi上で作業用フォルダとしてマウント
4. 編集やコミットはMacBook上で行うようにして, 実行はssh経由でRaspberryPi上で行う

といった感じです.

まずはMacBookのファイル共有ですが,
システム環境設定 > 共有 > ファイル共有 からすぐに設定できます.

今回はSMBを利用したいため, SMB共有をONにしておきます.

忘れずにWindowsファイル共有項目のアカウント名の左にあるチェックボックスを有効にしておかないと,
Raspberry Piからマウントする際にパーミッションでこけます.

後はRaspberry Piから共有フォルダをマウントして,
作業用フォルダに設定します.

    sudo mkdir /mnt/macbook
    sudo mount.cifs //macbook.local/Public /mnt/macbook/ -o user=hoge,nounix,sec=ntlmssp,file_mode=0777,dir_mode=0777
    cd /mnt/macbook

`macbook.local` はファイル共有設定をしたMacのIPアドレス,
`hoge` は共有フォルダにアクセスするアカウント名のつもりです.

`nounix,sec=ntlmssp` がMacの共有フォルダをLinuxからマウントする際に重要です.

参考: [https://www.raspberrypi.org/forums/viewtopic.php?f=5&t=4978](https://www.raspberrypi.org/forums/viewtopic.php?f=5&t=4978)

`file_mode=0777,dir_mode=0777` はマウント時のパーミッション設定です.
作業用フォルダ代わりなので, とりあえず読み書き実行全部許可してます.

参考: [https://askubuntu.com/questions/245787/how-to-mount-writable-samba-shares](https://askubuntu.com/questions/245787/how-to-mount-writable-samba-shares)

本当は編集作業はMacBookで行う想定なので読み取り専用で十分なつもりだったのですが,
プロジェクトのビルド時に生成する一時ファイル(`.h` `.c` `.o`等)をカレントディレクトリから切り分けていなかったので,
書き込み許可がないと, 直接共有フォルダ上でビルドが実行できなかったためです.

書き込み許可を出すのはあまり良い対策ではないので,
将来的にはビルド時の一時ファイル置き場を切り分けられるようにするか,
共有フォルダからRaspberry Piのローカルに一度ファイルをコピーしてビルドするようなフローにするか
検討したいと思っています.

作業完了したら `umount` でアンマウントします.

    sudo umount /mnt/macbook
    or
    sudo umount -f /mnt/macbook

MacBook側に共有フォルダを作ったので,
当然Raspberry Piでリソースロード時間が長くなりました.

とりあえず, 現状はリソースロードよりもテキスト編集している時間のほうが長そうなので,
今の構成で作業をしてみることにします.

本当はMacBookとRaspberryPiで作業用フォルダをミラーリングして,
編集分を差分転送するようにすれば, アクセス速度も維持できると思いますが,
必要になるまではSMBでも十分そうです.

