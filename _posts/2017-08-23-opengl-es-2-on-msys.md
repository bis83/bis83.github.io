---
layout: post
title: MSYS2環境下でOpenGLES2のアプリケーションをビルドするまで
---

Raspberry Pi上で動作するゲームを作ろうと思い
GLFW+OpenGLES2を使ってちまちま作業していました.

今後, 動作するようになったゲームがRaspberryPi上でのみ動作だと
遊べる環境が少なくなってしまうので, Windows環境下でビルド出来るように
ビルド環境の構築を進めていました.

マルチプラットフォームをサポートするにあたって,
一つの方法はアプリケーション側で互換レイヤーを作成して
複数のシステム下で動作するようにする手がありますが,
今回はそこまでコストをかけてコーディングしたくなかったので,
先人の武器を使用することにします.

### Raspberry Pi上の環境

Raspberry PiではRaspbian上でGCCを使用した環境で,
Chicken Schemeをビルドして処理系として利用していました.

Windows環境でビルドするためにWindows上でGCCが使える環境が必要になりますが
前からMSYSを利用して開発いていたので, 今回MSYS2を使うことにします.

またアプリケーションはGLFW3, OpenGL ES 2.0, OpenALを利用していました.

このうちGLFW3, OpenALはWindows上でも互換動作します.

OpenGL ES 2.0はWindowsに入っているOpenGL32.dll経由で動作させることは
あまり期待できないため, 何らかのエミュレータか互換レイヤーが必要になります.

今回は[ANGLE](https://github.com/google/angle)を使っていきます.

### MSYS2のインストール

[MSYS2](http://www.msys2.org/) からインストーラを入手してインストールします.

起動後, MinGW ToolChainとよく使うコマンドをインストールしておきます.

```
pacman -Syu
pacman -Su
pacman -S base-devel mingw-w64-i686-toolchain mingw-w64-x86_64-toolchain
pacman -S vim tar
```

### Chicken Schemeのインストール

[CHICKEN source code](http://code.call-cc.org/) からソースコードを取得して,
MSYS2のホームディレクトリに移動します.

MSYS2環境下なら楽にビルドできるので, 後は解凍してビルドするだけです.

```
tar zxvf chicken-4.12.0.tar.gz
cd chicken-4.12.0/
make PLATFORM=mingw-msys
make install
```

### 使用ライブラリ(OpenAL, GLFW3, ANGLE)のインストール

MSYS2のpacman経由でOpenAL, GLFW3, ANGLEともにパッケージを取得可能なので,
今回はpacmanでインストールします.

```
pacman -S mingw-w64-i686-openal mingw-w64-x86_64-openal
pacman -S mingw-w64-i686-glfw mingw-w64-x86_64-glfw
pacman -S mingw-w64-i686-angleproject-git mingw-w64-x86_64-angleproject-git
```

### ビルド

以下, ビルドテスト時のサンプルコード.
(解放処理とエラー処理は省略)

```c
#define GLFW_INCLUDE_ES2
#include <GLFW/glfw3.h>
#include <AL/al.h>
#include <AL/alc.h>

int main() {
  ALCdevice* ad = alcOpenDevice(NULL);
  ALCcontext* ac = alcCreateContext(ad, NULL);
  alcMakeContextCurrent(ac);

  glfwInit();
  glfwWindowHint(GLFW_CONTEXT_CREATION_API, GLFW_EGL_CONTEXT_API);
  glfwWindowHint(GLFW_CLIENT_API, GLFW_OPENGL_ES_API);
  glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 2);
  glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);

  GLFWwindow* wp = glfwCreateWindow(320, 240, "test", NULL, NULL);
  glfwMakeContextCurrent(wp);

  while(!glfwWindowShouldClose(wp)) {
    glfwPollEvents();
    glClearColor(1,0,0,1);
    glClear(GL_COLOR_BUFFER_BIT);
    glfwSwapBuffers(wp);
  }
  return 0;
}
```

```
gcc test.c -lopenal -lEGL -lGLESv2 -lglfw3
```

ANGLEのおかげで, GLESv2とEGLが使えるようになるため,
インクルードやリンクするライブラリもRPI環境と同一名に揃えることが出来ます.

`glfwWindowHint` もRPI環境と同一内容で初期化出来るようになったため,
プラットフォーム依存のコード分岐がほぼなくなりました.

### からの思わぬ落とし穴

と, ビルド環境の導入からANGLEによるGLES2アプリケーションの実行までうまくいったのですが,
元々RPI上で実行していた描画コードを移植するとアサーションで停止することが発覚しました.

原因調査を進めていたところ, どうやら何らかのシェーダープログラムを`glUseProgram`で設定し,
かつ `GL_ELEMENT_ARRAY_BUFFER` をバインドした状態で
`glDrawElements` を呼び出す... つまりインデックスバッファありの描画をすると
アサーションにひかかってしまう模様です.

- [検証コードのGist](https://gist.github.com/bis83/0e30b6ef15092909ae890055ad57b6f1)

発生するアサーションは下記:

```
File: ../src/libANGLE/ContextState.h, Line 153

Expression: mSavedArgsType->hasDynamicType(T::TypeInfo)
```

うぬぬ...

ANGLE内部のようですが, pacmanで取得しただけでANGLEの内部実装に詳しくないので,
原因が特定できず, 現在はANGLEのマニュアルビルド環境を整えつつ原因調査中です.

原因が見つかったら追記します.

