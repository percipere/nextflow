# nextflow
nextflowを使ってみる。

ワークフローの記述スクリプトとして [nextflow](https://www.nextflow.io)が良いという話を聞いたので、nextflowを利用してみる。

先ずは、実行環境を準備する。

docker hubにPaolo Di Tommaso氏による[イメージ](https://hub.docker.com/r/nextflow/nextflow/)が存在するので、このイメージを先ずはそのまま利用してみる。

```bash
$ docker pull nextflow/nextflow
Using default tag: latest
latest: Pulling from nextflow/nextflow
ff3a5c916c92: Pull complete 
5de5f69f42d7: Pull complete 
fa7536dd895a: Pull complete 
e49da66123f1: Pull complete 
4441b18a4ca0: Pull complete 
Digest: sha256:3b0e64c667a0d1256844d6150ef1dadbbd11119ad6c88aa5b7ef7515d7968ba1
Status: Downloaded newer image for nextflow/nextflow:latest
```

Dockerfileを確認すると、`ENTRYPOINT ["nextflow"] `なので、実行環境を確認するために`info`コマンドを実行してみる。

```bash
$ docker run -it nextflow/nextflow info
  Version: 0.27.6 build 4775
  Modified: 19-02-2018 08:45 UTC (08:45 GMT)
  System: Linux 4.9.60-linuxkit-aufs
  Runtime: Groovy 2.4.13 on OpenJDK 64-Bit Server VM 1.8.0_151-b12
  Encoding: UTF-8 (UTF-8)
```

先ずは、nextflowの"Getting Started"の通りお試ししてみる。
`hello.nf`がサンプルとして含まれているのかな？

```
$ docker run -it nextflow/nextflow run hello
N E X T F L O W  ~  version 0.27.6
Pulling nextflow-io/hello ...
 downloaded from https://github.com/nextflow-io/hello.git
Launching `nextflow-io/hello` [pedantic_lavoisier] - revision: d4c9ea84de [master]
[warm up] executor > local
[46/eb7d72] Submitted process > sayHello (4)
[36/787d99] Submitted process > sayHello (3)
[e4/c572c5] Submitted process > sayHello (1)
[6f/ea7a64] Submitted process > sayHello (2)
Hola world!
Hello world!
Bonjour world!
Ciao world!
```

github から`nextflow-io/hello`がダウンロードされて実行された。

どういうことなのだろう。

```bash
$ docker run -it nextflow/nextflow run -h
Execute a pipeline project
Usage: run [options] Project name or repository url
```

run の引数として`repository url` と記載されている。つまり該当のワークフローがローカルに存在しなければ、githubを探索してワークフローを探して実行するというシステムとなっている。
githubにワークフローを登録しておくことで、ワークフローをお手軽に利用できるという生態系を作ろうとしているのだろう。

サンプルの実行が可能な事がわかったので、試しに `helloWorld.nf` を記述してみる。

```groovy:helloWorld.nf
#!/usr/bin/env nextflow

process helloWorld {

   output:
   result

   """
   echo "Hello World!"
   """
}

result.subscribe { println "Output: " + it }
```

`helloWorld.nf`　をカレントディレクトリに作成

簡単な説明

* 1行目 shbang 今回はあまり意味は無い
* process で helloWorldを定義する
* output: で resultというチャンネルに出力する事を定義する
* 3個のダブルクォーテーションで囲まれた内容が処理の中核部分
 * ここでは単に"Hello World!"を標準出力 => この出力が resultに入る
* resultに データが入力されたら、 println の部分を実行するように subscribe で定義する
 * 表示の先頭に"Output: " を出力することで、echo で出力されたデータでは無く、変更している事がわかるようにした

これを実行してみる。

```screen
$ docker run -itv ${PWD}:/work nextflow/nextflow /work/helloWorld.nf 
N E X T F L O W  ~  version 0.27.6
Launching `/work/helloWorld.nf` [golden_caravaggio] - revision: 1752cdf669
[warm up] executor > local
[5b/1095f1] Submitted process > helloWorld
Output: Hello World!
```

ここでは、

* dockerに対して`-v`コマンドでホストの現在のディレクトリ`${PWD}`をコンテナの`/work`にマウントするように指定
  * ホストのカレントディレクトリに作成したファイルは、コンテナの中から`/work/helloWorld.nf`で参照できる

正常に動作している。




