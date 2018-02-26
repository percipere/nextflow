# nextflow

nextflowを使ってみる。

ワークフローの記述スクリプトとして [nextflow](https://www.nextflow.io)が良いという話を聞いたので、nextflowを利用してみる。

## 実行環境の構築

先ずは、実行環境を準備する。

この文書を作成しているシステムの環境は次の通り
```
$ uname -prsv
Darwin 17.4.0 Darwin Kernel Version 17.4.0: Sun Dec 17 09:19:54 PST 2017; root:xnu-4570.41.2~1/RELEASE_X86_64 i386
$ docker -v
Docker version 17.12.0-ce, build c97c6d6
```

* ホストOSは macOS
* nextflowが実行される環境は docker上に構築

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

## 動作を確認

Dockerfileを確認すると、`ENTRYPOINT ["nextflow"] `なので、実行環境を確認するために`info`コマンドを実行してみる。

```bash
$ docker run -it nextflow/nextflow info
  Version: 0.27.6 build 4775
  Modified: 19-02-2018 08:45 UTC (08:45 GMT)
  System: Linux 4.9.60-linuxkit-aufs
  Runtime: Groovy 2.4.13 on OpenJDK 64-Bit Server VM 1.8.0_151-b12
  Encoding: UTF-8 (UTF-8)
```

##サンプルの実行

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

##簡単なスクリプトの作成と実行

サンプルの実行が可能な事がわかったので、試しに `helloWorld.nf` を記述してみる。

```groovy:helloWorld.nf
#!/usr/bin/env nextflow

process helloWorld {

   output:
   result
   
   script:
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
* script: 以降の3個のダブルクォーテーションで囲まれた内容が処理の中核部分
 * ここでは単に"Hello World!"を標準出力 => この出力が resultに入る
 * script: の記述は無くても動作するようだ...
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


## 動作の様子を確認

`work`ディレクトリ下には、デフォルトでnextflowの実行時のファイルが全て保存されるので、コンテナの`/work`にマウントしているホストのディスクを調べれば動作の様子がわかる

```screen
$ ls -al
total 8
drwxr-xr-x   8 nexflw  staff   256 Feb 26 13:59 .
drwx------+ 41 nexflw  staff  1312 Feb 26 10:55 ..
...
drwxr-xr-x   3 nexflw  staff    96 Feb 26 13:52 a8
-rw-r--r--   1 nexflw  staff   154 Feb 26 13:59 helloWorld.nf
```

最新のディレクトリ(実行する度に別のフォルダ名が与えられるここでは`a8`)以下に次の様なファイルが残っている。

```screen
$ ls -a ./a8/e158e76efe62093dbc6d313ca5f915/
.		.command.begin	.command.log	.command.run	.exitcode
..		.command.err	.command.out	.command.sh
```

* `/a8/e158e76efe62093dbc6d313ca5f915/`はそれぞれ異なったものになる。

.command.shが実際に実行されたスクリプト。

```bash
$ cat ./a8/e158e76efe62093dbc6d313ca5f915/.command.sh
#!/bin/bash -ue
echo "Hello World!"
```

`.command.run`が上記のスクリプトを処理するためのラッパースクリプト。

```
$ ls -al
total 8
drwxr-xr-x   8 nexflw  staff   256 Feb 26 13:59 .
drwx------+ 41 nexflw  staff  1312 Feb 26 10:55 ..
drwxr-xr-x   3 nexflw  staff    96 Feb 26 13:34 15
drwxr-xr-x   3 nexflw  staff    96 Feb 26 13:49 3d
drwxr-xr-x   3 nexflw  staff    96 Feb 26 13:55 5b
drwxr-xr-x   3 nexflw  staff    96 Feb 26 13:33 9c
drwxr-xr-x   3 nexflw  staff    96 Feb 26 13:52 a8
-rw-r--r--   1 nexflw  staff   154 Feb 26 13:59 helloWorld.nf
```

## エラーが発生すると何が起きるのか?

`helloWorld.nf`の `echo "Hello World!"` の部分を `tell "Hello World!"`(少なくともここには無いコマンド)に書き換えてワークフローを実行してみる。

```screen
$ docker run -itv ${PWD}:/work nextflow/nextflow /work/helloWorld.nf
N E X T F L O W  ~  version 0.27.6
Launching `/work/helloWorld.nf` [ridiculous_payne] - revision: 1269cff8e1
[warm up] executor > local
[5e/e5b3a2] Submitted process > helloWorld
ERROR ~ Error executing process > 'helloWorld'

Caused by:
  Process `helloWorld` terminated with an error exit status (127)

Command executed:

  tell "Hello World!"

Command exit status:
  127

Command output:
  (empty)

Command error:
  .command.sh: line 2: tell: command not found

Work dir:
  /work/5e/e5b3a2741b7ddd5ec91c0ccb7ffeca

Tip: you can replicate the issue by changing to the process work dir and entering the command `bash .command.run`

 -- Check '.nextflow.log' file for details
```

上記のようなエラーが出力された。`cd /work/5e/e5b3a2741b7ddd5ec91c0ccb7ffeca/; bash .command.run`を実行すれば同じエラーが再現できるはずとのこと。

## エラーが起きたスクリプトを実際に実行してみる

dockerのエントリーポイントを書き換えて`/bin/bash`を起動し、`.command.run`をインタラクティブに実行してエラーを再現させてみる。

```screen
$ docker run -it --entrypoint="/bin/bash" -v ${PWD}:/temp nextflow/nextflow
bash-4.4# cd /work/5e/e5b3a2741b7ddd5ec91c0ccb7ffeca/; bash .command.run
/work/5e/e5b3a2741b7ddd5ec91c0ccb7ffeca/.command.sh: line 2: tell: command not found
```

dockerの内部でbashを起動して、インタラクティブに実行した結果確かに同じエラーが発生した事がわかる。
そこで、command.shの`tell`部分を`echo`に置き換えて再度実行してみる。

```screen
bash-4.4# cd /work/5e/e5b3a2741b7ddd5ec91c0ccb7ffeca/; bash .command.run
Hello World!
```

正常に動作することを確認した。

## Scriptブロックに色々書いてみる

script ブロックにはbashだけではなく、色々な言語が利用できる。

ここでは、python3のスクリプトを試してみるために、nextflowのコンテナにpython3の環境を追加したものを準備する。

```
$ docker run -it --entrypoint="/bin/bash" -v ${PWD}:/work nextflow/nextflow
bash-4.4# python
bash: python: command not found
bash-4.4# uname -a
Linux 6e5f62375a9a 4.9.60-linuxkit-aufs #1 SMP Mon Nov 6 16:00:12 UTC 2017 x86_64 GNU/Linux
bash-4.4# apk add python3
(1/4) Installing expat (2.2.5-r0)
(2/4) Installing gdbm (1.13-r1)
(3/4) Installing xz-libs (5.2.3-r1)
(4/4) Installing python3 (3.6.3-r9)
Executing busybox-1.27.2-r7.trigger
OK: 147 MiB in 66 packages
bash-4.4# python3 --version
Python 3.6.3
```

alpine linux のパッケージ管理ツール`apk`を利用してpython3をインストールする

`ctrl+p` `ctrl+q`でコンテナへの接続を一次的に解除して、ホスト上でpython3をインストールしたコンテナをコミットする。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6418c1e4274f        nextflow/nextflow   "/bin/bash"         57 seconds ago      Up 57 seconds                           keen_banach
$ docker commit --change='ENTRYPOINT ["nextflow"]' 6418c1e4274f nextflow_python
sha256:da018de73b15d346e85fbacb8999f30980a9a45d439aa9413af07a8c90e7754b
$ docker images nextflow_python
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nextflow_python     latest              ef3e88c7b6c5        13 seconds ago      171MB
$ docker attach 6418c1e4274f
bash-4.4# exit
```

* docker ps: 動作中のコンテナのCONTAINER IDを確認する
* docker commit: 動作中のコンテナを別名を付けてコミットする
* docker images: コミットしたコンテナのイメージを確認する
* docker attach: 先程接続を一次的に解除したコンテナに再接続する
* exitで再接続したコンテナから抜ける

python3のスクリプトをscriptブロックに記述したnextflowのファイルを作成

```groovy
#!/usr/bin/env nextflow

process testPython3 {

   output:
   stdout result

   script:
   """ 
   #!/usr/bin/env python3
   name = 'python3'
   print(f"I'm script of {name}.")
   """ 
}

result.subscribe { println "Output: " + it }
```
`testPython3.nf`として保存して、このスクリプトを実行してみる。

```
$ docker run -it -v ${PWD}:/work nextflow_python run /work/testPython3.nf 
N E X T F L O W  ~  version 0.27.6
Launching `/work/testPython3.nf` [shrivelled_wright] - revision: 7923251243
[warm up] executor > local
[e8/bb80fb] Submitted process > testPython3
Output: I'm script of python3.

```

python3で記述したファイルが実行できた。

## input ブロックを利用してみる

### スクリプト内部の変数を先に置き換えて実行の例

```#!/usr/bin/env nextflow

data_set = Channel.from( 'a', 'b', 'c', 'd' )

process testPython3 {

   input:
   val input_value from data_set

   output:
   stdout result

   script:
   """ 
   #!/usr/bin/env python3
   import sys
   name = 'python3'
   print(f"Item: ${input_value} => I'm script of {name}.")
   """ 
}

result.subscribe { print "Output: " + it }
```

* data_set: 文字から構成されるChannel
* input ブロックで、data_setから個別のデータを取得してinput_valueに格納するよう指示
* python3内のスクリプトをファイルに出力する前に、${input_value}の値を直接書き換える

`testPython3.nf`を上記の様に変更してこのスクリプトを実行してみる。

```
$ docker run -it -v ${PWD}:/work nextflow_python run /work/testPython3.nf 
N E X T F L O W  ~  version 0.27.6
Launching `/work/testPython3.nf` [exotic_volhard] - revision: d0ca217810
[warm up] executor > local
[86/93d554] Submitted process > testPython3 (4)
[7c/8402ef] Submitted process > testPython3 (1)
[91/304c8e] Submitted process > testPython3 (3)
[d3/3fc80e] Submitted process > testPython3 (2)
Output: Item: b => I'm script of python3.
Output: Item: a => I'm script of python3.
Output: Item: c => I'm script of python3.
Output: Item: d => I'm script of python3.

```

input_value毎に各処理が個別のディレクトリで実行される。結果は順不同。

`.command.sh`を確認してみる。

```
$ cat f6/fa346b55c85988b56dff51af1f3e81/.command.sh
#!/usr/bin/env python3
import sys
name = 'python3'
print(f"Item: a => I'm script of {name}.")
```

print出力部分`Item: ${input_value}`が`Item: a =>`に変更されている事が確認できる。

### スクリプトに標準入力(stdin)を利用してパラメーターを与える場合

```
#!/usr/bin/env nextflow

data_set = Channel.from( 'a', 'b', 'c', 'd')


process testPython3 {

   input:
   stdin data_set

   output:
   stdout result

   script:
   """ 
   #!/usr/bin/env python3
   import sys
   name = 'python3'
   print(f"Item: {sys.stdin.readline()} => I'm script of {name}.")
   """ 
}
```

data_setの内容がstdinを利用してscriptに渡している。
pythonスクリプトでは標準入力を`sys.stdin.readline()`を利用して受け取る。

```
$ docker run -it -v ${PWD}:/work nextflow_python run /work/testPython3.nf 
N E X T F L O W  ~  version 0.27.6
Launching `/work/testPython3.nf` [nasty_hawking] - revision: 37909ff192
[warm up] executor > local
[dd/f27427] Submitted process > testPython3 (4)
[f3/c145f8] Submitted process > testPython3 (3)
[71/a15943] Submitted process > testPython3 (1)
[77/52b140] Submitted process > testPython3 (2)
Output: Item: d => I'm script of python3.
Output: Item: b => I'm script of python3.
Output: Item: a => I'm script of python3.
Output: Item: c => I'm script of python3.
```

標準入力経由での実行を確認できた。


### 環境変数を利用してパラメーターを与える場合

```
#!/usr/bin/env nextflow
  
data_set = Channel.from( 'a', 'b', 'c', 'd')


process testPython3 {

   input:
   env PASS_PARAMETER from data_set

   output:
   stdout result

   script:
   """
   #!/usr/bin/env python3
   import os
   name = 'python3'
   print(f"Item: {os.environ['PASS_PARAMETER']} => I'm script of {name}.")
   """
}

result.subscribe { print "Output: " + it }
```

data_setの内容を環境変数PASS_PARAMETERを利用してscriptに渡している。
pythonスクリプトでは環境変数を`os.environ['PASS_PARAMETER']`を利用して受け取る。

```
$ docker run -it -v ${PWD}:/work nextflow_python run /work/testPython3.nf 
N E X T F L O W  ~  version 0.27.6
Launching `/work/testPython3.nf` [cranky_torricelli] - revision: 2e233b4d31
[warm up] executor > local
[2a/676a52] Submitted process > testPython3 (3)
[9e/170d93] Submitted process > testPython3 (2)
[22/d8b5bd] Submitted process > testPython3 (4)
[29/7168f9] Submitted process > testPython3 (1)
Output: Item: b => I'm script of python3.
Output: Item: c => I'm script of python3.
Output: Item: d => I'm script of python3.
Output: Item: a => I'm script of python3.
```

環境変数経由での実行を確認できた。
