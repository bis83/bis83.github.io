---
layout: post
title: Raspberry Pi事始め in 2017
---

すっかり放置気味になっていた Raspberry Pi を掘り起こしてきました.

冬の寒さに堪え忍んでいる間に, ふとRaspberry Pi上で動作するゲームを開発したいと思い立ち,
折角なので, 環境構築を始めることにしました.

まずは, 公式Raspbianのデスクトップ環境をインストールするべく,
公式サイトからディスクイメージを取得してブートデバイスを作成しました.

    diskutil list
    sudo diskutil unmount /dev/disk2
    sudo diskutil unmountDisk /dev/disk2
    sudo dd 1m if=Downloads/2016-11-25-raspbian-jessie.img of=/dev/disk2

イメージのダウンロードと書き込みには時間がかかるため,
暖かい緑茶とみかんを食べながら待っていましょう.

ディスクへの書き込みが完了したら, Raspberry PiにSDカードを差し込み電源を接続しました.
HDMIで液晶モニタに接続しておくと, そのままデスクトップ環境が立ち上がるので,
特に初期設定など意識せずに使い始めることができます.

Raspberry Pi が立ち上がったら,
まずはWi-Fi設定を行ってネットワークに接続しましょう.
ネットワーク設定はGUIでちょいちょいと行えるので, 特に困ることはありませんでした.

特に開発では困らないのですが,
ついでにロケール設定を日本に切り替えておきました.

さて, Raspberry Piが一通り使える状態になりましたが,
一番はじめにやることは Vim のインストールでした.

    sudo apt-get update
    sudo apt-get install vim
    curl https://gist.githubusercontent.com/bis83/a8faaef6cce201baeb73/raw/de4930cad2333435441e2adec60044baee0f4dc5/.vimrc > ~/.vimrc

あらかじめGistによく使う.vimrcをアップロードしておいたので,
それをダウンロードしてきて利用することにしました.

次に, 最も頻繁に利用するパッケージ(Chicken SchemeとGLFW3)を,
Raspbian Jessie stableでは取得できるバージョンが古いため,
手動でインストールすることにしました.

どちらのパッケージもビルドにそれほど時間がかからないため,
そんなに手間にはなりませんでした.

まずは公式の tar/zip を取得しました.

    cd Downloads/
    curl -O -L https://github.com/glfw/glfw/releases/download/3.2.1/glfw-3.2.1.zip
    curl -O -L https://code.call-cc.org/releases/4.11.0/chicken-4.11.0.tar.gz
    tar zxvf chicken-4.11.0.tar.gz
    unzip glfw-3.2.1.zip

Chicken Schemeのビルドとインストールは下記のコマンドで行いました:

    cd chicken-4.11.0/
    make PLATFORM=linux
    sudo make PLATFORM=linux install

次に GLFW のビルドとインストールですが,
こちらは事前にいくつかのツールと関連パッケージをインストールしておく必要がありました.
(ついでにOpenALもインストールしておきました)

    cd ../
    sudo apt-get install cmake
    sudo apt-get install libgl1-mesa-dev libgles2-mesa-dev
    sudo apt-get install xorg-dev
    sudo apt-get install libopenal-dev
    cd glfw-3.2.1/
    cmake .
    make
    sudo make install

GLFWをChicken上でバインディングして使用する前に,
まずはGCC上でビルドテストを行いました.

```c
#include <AL/al.h>
#include <AL/alc.h>

#define GLFW_INCLUDE_GLES2
#include <GLFW/glfw3.h>

ALCdevice* alc_device = 0;
ALCcontext* alc_context = 0;
GLFWwindow* glfw_window = 0;

int main() {
  alc_device = alcOpenDevice(0);
  alc_context = alcCreateContext(alc_device, 0);

  glfwInit();
  glfwWindowHint(GLFW_CONTEXT_CREATION_API, GLFW_EGL_CONTEXT_API);
  glfwWindowHint(GLFW_CLIENT_API, GLFW_OPENGL_ES_API);
  glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 2);
  glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);
  glfw_window = glfwCreateWindow(320, 240, "", 0, 0);
  glfwMakeContextCurrent(glfw_window);

  while(!glfwWindowShouldClose(glfw_window)) {
    glfwPollEvents();
    glfwSwapBuffers(glfw_window);
  }
  return 0;
}
```

ビルドは下記コマンドでテストしました:

    gcc test.c `pkg-config --static --libs glfw3 openal` -lGLESv2
    ./a.out

Raspberry PiではOpenGLES2を利用することになります.

これでプログラミングする上で十分なツールが揃いましたので,
あとはコーディングしていくだけですね.

