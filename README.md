# Distributed Framework First Try (分布式框架初体验)

Project of **CS236 Cloud Computing, Shanghai Jiao Tong University**, which will give you an excellent experience of **Hadoop & Spark & GraphX**.

Just enjoy distributed computing!

## Introduction

This project uses the Hadoop and Spark distributed framework to build clusters on three hosts, and uses the GraphX API to implement [ConnectedComponents](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/graphx/ConnectedComponentsExample.scala) and PageRank algorithm.

[**ConnectedComponents**](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/graphx/ConnectedComponentsExample.scala) is the example program of Spark and the [raw data](https://github.com/apache/spark/blob/master/data/graphx/) is also provided by Spark. **I just add the function of saving outputs to HDFS**.

**PageRank** runs on the [Wiki vote dataset](http://snap.stanford.edu/data/wiki-Vote.html), **print the list of the top 20 candidates with the highest reputation on screen, and save the sorted output to HDFS.**

本项目使用Hadoop和Spark分布式框架在三台主机上搭建集群，使用GraphX API实现[ConnectedComponents](https://github.com/apache/spark/blob/master/examples/src/main/scala)和PageRank算法。

[**ConnectedComponents**](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/graphx/ConnectedComponentsExample.scala) 是示例程序 Spark 和 [原始数据](https://github.com/apache/spark/blob/master/data/graphx/) 也由 Spark 提供。 **我只是添加了将输出保存到 HDFS 的功能**。

**PageRank** 在 [Wiki vote dataset](http://snap.stanford.edu/data/wiki-Vote.html) 上运行，**在屏幕上打印声誉最高的前 20 名候选人名单， 并将排序后的输出保存到 HDFS。**

## Installation of Hadoop&Spark&SBT

**If you already have Hadoop, Spark, and Simple Build Tool installed, just skip to Usage section.**

This part is not a detailed installation tutorial, but only links to tutorial and fixes the parts of the tutorial that do not apply to the new versions.

All the tutorials are in Chinese. If you don't understand Chinese, please refer to the official instructions.

### Hadoop

Hadoop Version 3.3.1

The Hadoop installation process refers to this [tutorial](http://dblab.xmu.edu.cn/blog/1177-2/). The Hadoop version used in the tutorial is 2.7, while I met two differences in the actual installation process from the tutorial:

1.  `JAVA_HOME` needs to be added to `HADOOP_HOME/etc/hadoop/hadoop-env.sh`
2. `dfs.name.dir` need to be specified in `HADOOP_HOME/etc/hadoop/hdfs-site.xml` like this

```xml
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.name.dir</name>
                <value>/usr/local/hadoop/hdfs/name</value>
        </property>
</configuration>

```

### Spark

Spark Version 3.2.0

The installation of Spark is totally according to this [tutorial](http://dblab.xmu.edu.cn/blog/1714-2/).

### Simple Build Tool

SBT Version 1.3.8

The installation of SBT is totally according to this [tutorial](http://dblab.xmu.edu.cn/blog/1307-2/).

## Usage

Step 1 and Step 2 are to package the scala source code into a `.jar` file. I have packaged them for scala 1.2 and Spark 3.2.0, which are in `Jar Package` folder.

### Step 1: Configure SBT

After installing SBT, use vim to create a file under the SBT installation path(mine is `/usr/local/sbt`)，**this file is used for build scala code**.

```shell
vim /usr/local/sbt/sbt
```

and add the contents below to the file

```bash
#!/bin/bash
SBT_OPTS="-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M"
java $SBT_OPTS -jar `dirname $0`/sbt-launch.jar "$@"
```

then add executable permissions to the shell script file, just enter the following commands into shell:

```shell
chmod u+x /usr/local/sbt/sbt
```

then, you can use the following command to view the SBT version information:

```shell
cd /usr/local/sbt # /usr/local/sbt is my SBT installation path
./sbt sbtVersion
```

### Step2: Build Scala Code

**Make sure you have configuring SBT done before building.**

If you want to build `PageRank`, just:

```shell
cd PageRank
<SBT path>/sbt package
```

The `.jar`package is in `PageRank/target/scala-x.x/page-rank_x.x-1.0.jar`, and x.x is your scala version.

If you want to build `ConnectedComponents`, just:

```shell
cd ConnectedComponents
<SBT path>/sbt package
```

The `.jar`package is in `ConnectedComponents/target/scala-x.x/connected-components_x.x-1.0.jar`, and x.x is your scala version.

#### Some explanations of the process of building

If you are not interested in the details of building, just skip this section.

The scala source code must be located in a directory with a specific structure：

```
PageRank
├── PR.sbt
└── src
    └── main
        └── scala
            └── PageRankExample.scala

```

Above is the folder structure of PageRank. 

`PageRankExample.scala` is the scala source code. 

`PR.sbt` is used for building, which is shown below

```
name := "Page Rank"
version := "1.0"
scalaVersion := "2.12.15"
libraryDependencies += "org.apache.spark" %% "spark-core" % "3.2.0"
libraryDependencies += "org.apache.spark" %% "spark-graphx" %"3.2.0"
libraryDependencies += "org.apache.spark" %% "spark-sql" %"3.2.0"
```

The versions of SBT and Spark should be specified in the `.sbt` file. 

* `scalaVersion` is the version of scala. 

* Each dependent library in the source code needs to be specified in the sbt file, starting with `libraryDependencies`

* `3.2.0` is the version of Spark. 

### Run on Spark

To run the `.jar` file, you must open Spark first.

Then use the following command to run the `.jar` file in Spark:

```shell
<Spark path>/bin/spark-submit --class <class name> <jar package>
```

#### PageRank

The `PageRankExample.scala` reads `Wiki-vote.txt` from the `/PageRankInput` folder on HDFS(Hadoop distributed file system), so you must make the directory on HDFS and put the `Wiki-vote.txt` in `/PageRankInput` by the commands below:

````shell
hadoop fs -mkdir /PageRankInput
hadoop fs -put Wiki-vote.txt /PageRankInput
````

Then use the command below to run it:

```shell
<Spark path>/bin/spark-submit --class "org.apache.spark.examples.graphx.PageRankExample" <jar package>
```

The output file is in directory `/PageRankOutput` on HDFS.

#### Connected Components

The `ConnectedComponentsExample.scala` reads `followers.txt` and `users.txt ` from the `/ConnectedComponentsInput` folder on HDFS, so you must make the directory on HDFS and put the two files in `/ConnectedComponentsInput` by the commands below:

```shell
hadoop fs -mkdir /ConnectedComponentsInput
hadoop fs -put followers.txt /ConnectedComponentsInput
hadoop fs -put users.txt /ConnectedComponentsInput
```

Then use the command below to run it:

```shell
<Spark path>/bin/spark-submit --class "org.apache.spark.examples.graphx.ConnectedComponentsExample" <jar package>
```

The output file is in directory `/PageRankOutput` on HDFS.

## Contact Me

If you have any problems of this project, feel free to send an e-mail to me. My e-mail address is guanrenyang@qq.com

