# docker-word2vec

word2vecを簡単に使うためのDockerイメージを作成するDockerfileです。

* word2vec

https://code.google.com/p/word2vec/ 

* "word2vec"のCOPYRIGHT  

Copyright 2013 Google Inc. All Rights Reserved.

## イメージ構築

```bash
% git clone https://github.com/naoa/docker-word2vec.git
% cd docker-word2vec
% mkdir /var/lib/word2vec
% docker build -t naoa/word2vec .
```

## 使い方
* コンテナにターミナル接続する場合  
```bash
% docker run -v /var/lib/word2vec:/var/lib/word2vec -i -t naoa/word2vec /bin/bash
bash-4.2# word2vec
```

* (必要な場合)コンテナ上でカスタマイズしたMeCab辞書を利用するために共有フォルダにMeCab辞書をコピー。もしくは、Dockerコンテナ上に直接送ってコミットしてもよいです。

```bash
% cp -rf /usr/lib64/mecab/dic/naist-jdic/ /var/lib/word2vec/naist-jdic
```

* プレーンテキストを正規表現フィルタ、NFKC正規化、分かち書き(string-splitter)
```bash
% cat example/open_source.txt | docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec \
  string-splitter --mecab_dic /var/lib/word2vec/naist-jdic > /var/lib/word2vec/wakati.txt
```

<code>string-splitter</code>はプレーンテキストを正規表現フィルタ、NFKC正規化、分かち書きをしてくれる自作C++プログラムです。 https://github.com/naoa/string-splitter

ここではカスタマイズしたMeCab辞書を使用するために、<code>--mecab_dic</code>オプションを使用しています。

* 分かち書き済みテキストをトレーニング(word2vec)
```bash
% docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec word2vec \
  -train /var/lib/word2vec/wakati.txt -output /var/lib/word2vec/learn.bin \
  -size 200 -window 5 -negative 0 -hs 1 -sample 1e-3 -threads 12 -binary 1
```

* 学習モデルでベクトル演算(word2vec-calc)
```bash
% docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec word2vec-calc
```

<code>word2vec-calc</code>は、<code>distance</code>や<code>analogy</code>だけでなく、
自由に足し引き演算(+-)をできるようにし、いくつか出力結果を調整できるようにした
自作C++プログラムです。 https://github.com/naoa/word2vec-calc

* オリジナルのword2vecに同梱されている<code>word-analogy</code>や<code>distance</code>等も利用可能です。
```bash
% docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec distance
```

* クラスタリング
```bash
% docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec word2vec \
  -train /var/lib/word2vec/wakati.txt -output /var/lib/word2vec/classes.txt \
  -cbow 0 -size 200 -window 5 -negative 0 -hs 1 -sample 1e-3 -threads 12 -classes 500
% sort /var/lib/word2vec/classes.txt -k 2 -n > /var/lib/word2vec/classes.sorted.txt
```

* コマンドがめんどくさいが、エイリアスをはれば短縮できます。常用的に使う場合は、~/.bashrcなどの起動スクリプトに書きます。

```bash
% alias word2vec="docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec word2vec"
% alias word2vec-calc="docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec word2vec-calc"
% alias string-splitter="docker run -v /var/lib/word2vec:/var/lib/word2vec \
  -a stdin -a stdout -a stderr -i naoa/word2vec string-splitter"
```

* 入出力形式  
UTF8の文字コードのテキストのみ対応しています。

各コマンドのリファレンスを参照してください。

* word2vec
https://code.google.com/p/word2vec/
* word2vec-calc
https://github.com/naoa/word2vec-calc
* string-splitter
https://github.com/naoa/string-splitter

## 環境

このDockerfileでは、以下の環境のコンテナが構築されます。

| 項目        | バージョン | 備考 |
|:-----------|:------------|:------------|
| CentOS     | 7 | ja_JP.UTF-8|
| MeCab     | 0.996 | --enable-utf8-only|
| MeCab IPAdic | 2.7.0-20070801 |--with-charset=utf8|
| GCC | 4.8.2-16 ||
| word2vec | https://github.com/svn2github/word2vec.git | |
| ICU | 50.1.2-11 ||
| RE2 | 20130115-2 ||
| WordNet | 3.0-21 ||
| glib2 | 2.36.3-5 ||
| gflags | 1.3-7 ||
| string-splitter |https://github.com/naoa/string-splitter|プレーンテキストを正規表現フィルタ、NFKC正規化、分かち書きをしてくれる自作C++プログラム|
| word2vec-calc |https://github.com/naoa/word2vec-calc|word2vecで学習したモデルを使ってベクトルの足し引き演算ができる自作C++プログラム|

## Author

Naoya Murakami naoya@createfield.com

## License

Public domain. You can copy and modify this project freely.

