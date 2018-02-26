# nextflow
nextflowを使ってみた

ワークフローの記述スクリプトとして [nextflow](https://www.nextflow.io)が良いという話を聞いたので、nextflowを利用してみようと思う。

先ずは、実行環境を準備してみる

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

先ずは、シンプルなワークフローを実行する

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


