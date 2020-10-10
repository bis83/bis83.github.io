---
layout: post
title: 最近使っているDockerイメージ
---

開発環境構築は面倒なものです.

ちょっとした興味で新しい環境を試してみたいだけで,
普段使用しているローカルPCに色々とインストールしてみると
あとで整理整頓が大変になりますね.

しかも, そういったケースでは大抵慣れていない作業になるため
中途半端だったり余計なインストールをしてしまいがちです.

そこで, ものぐさな私は開発環境をDockernizeして,
再構築やCI環境を楽にしようと思うわけです.

さらに考えを進めましょう.

手を抜いて自分でDockerfileを書かずに,
DockerHubに上がっているイメージを使わせてもらうのです.
先人が環境構築に成功したイメージがそこにあるわけです.

フリーライダーの精神ですね.

[Docker Hub](https://hub.docker.com/)

最近よく使っているものを書き残しておきます:

## [jekyll/jekyll](https://hub.docker.com/r/jekyll/jekyll/)

GitHub Pagesで使われているjekyllですが,
Ruby製なのでローカルで動かそうとするとRubyの環境を入れないといけないです.

Rubyは普段使いしておらず環境のバージョンを維持することは面倒なので
ここはjekyll公式のイメージに身を委ねます.

    docker run --rm -it -v `pwd`:/srv/jekyll -p 127.0.0.1:4000:4000 jekyll/jekyll:pages jekyll serve

>2020-10-11 Updated: 最新のコンテナは実行コマンド jekyll serve を明示的に渡す必要がある

## [apiaryio/emcc](https://hub.docker.com/r/apiaryio/emcc/)

Emscriptenの公式ページを見るとSDKという身構えたくなるような単語と
インストーラが目につきますが, 読むとJava? Python? Node? 色々なものに依存してますね.

これまた環境のバージョンを維持するのは面倒なので,
有志の方のイメージを有難く使わせて頂きます.

    docker run --rm -v `pwd`:/src -t apiaryio/emcc emcc test.c -o test.html 

## (2017-05-23追記) [auchida/markdown-pdf](https://hub.docker.com/r/auchida/markdown-pdf/)

Markdownをpdf化するmarkdown-pdfのdocker-image.

    docker run -v `pwd`:/opt/docs auchida/markdown-pdf markdown-pdf test.md

----

公式でDockerイメージを公開してもらえるとGetting startが楽になって嬉しいですね.
Dockerがより普及すれば, `README.md` のGetting startにまずDockerの項目から
スタートするのが一般的になる時代も近いでしょうか.
(すでにそうなりつつありますが)

