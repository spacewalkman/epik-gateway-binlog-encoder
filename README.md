# Epik knowledge graph binary log file encoder.
Spark job that encoding triplets to Epik knowledge graph binary-log files.

## Prerequisites
1. JDK1.8
Download Oracle JDK1.8, set JAVA_HOME environment variable. and add $JAVA_HOME/bin to your PATH.

2. Maven3.6+
Install [apache Maven](http://maven.apache.org/install.html), make M2_HOME environment variable point to it, and add $M2_HOME/bin to your PATH.

3. [Hadoop 3.2.2](https://archive.apache.org/dist/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz)
Setup a [pseudo-distributed](http://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation) hadoop cluster. Or [full-distributed](http://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/SingleCluster.html#Fully-Distributed_Operation) if you have spare resources.
Make HADOOP_HOME environment variable point to where you extracted tar.gz files, and add $HADOOP_HOME/bin and $HADOOP_HOME/sbin to your PATH.
Start hdfs and yarn services.

4. Spark 3.0.1
Since we use spark-on-yarn as our distributed computing engine, and use a custom version of Hadoop 3.2.2, we need to build spark from source.
```bash
curl https://codeload.github.com/apache/spark/tar.gz/refs/tags/v3.0.1 -o spark-3.0.1.tar.gz
tar zxvf spark-3.0.1.tar.gz
cd spark-3.0.1
./dev/make-distribution.sh --name epik --tgz -Phive -Phadoop-3.2 -Dhadoop.version=3.2.2 -Pscala-2.12 -Phive-thriftserver -Pyarn -Pkubernetes -DskipTests -X
```
When done, extract `spark-3.0.1-bin-epik-spark-3.0.1.tgz`, make environment variable SPARK_HOME point to it, and add $SPARK_HOME/bin to your PATH.

5. sbt
Install scala build tool [sbt](https://www.scala-sbt.org/), need it when building epik-gateway-binlog-encoder spark job.
```bash
curl https://github.com/sbt/sbt/releases/download/v1.4.8/sbt-1.4.8.tgz 
tar zxvf sbt-1.4.8.tgz  -C /opt/
ln -s /opt/sbt-1.4.8 /opt/sbt
export SBT_HOME=/opt/sbt
export PATH=$SBT_HOME/bin:$PATH
```

## Build java [nebula-client](https://github.com/vesoft-inc/nebula-java)
```bash
git clone https://github.com/vesoft-inc/nebula-java.git
cd nebula-java
mvn clean install -Dcheckstyle.skip=true -DskipTests -Dgpg.skip -X
```
Current version is 2.0.0-SNAPSHOT(note we disable the annoying checks.)

## Build epik-gateway-binlog-encoder spark job
```bash
git clone https://github.com/EpiK-Protocol/epik-gateway-binlog-encoder.git
cd epik-gateway-binlog-encoder
sbt assembly
```

binlog-encoder spark job jar file should be generated in:

```bash
epik-gateway-binlog-encoder/target/scala-2.12/epik-logfile-encoder-job.jar
```

## Prepare the input cn_pedia input files
Download [cn_dbpedia archive file](http://openkg1.oss-cn-beijing.aliyuncs.com/35f5fa1d-57a8-49b4-81ac-eace85f7a578/baiketriples.zip), which is in triplet format.
Unzip it, got `baike_cnpedia.txt`, make a dir in hdfs and put it over there.
```bash
hdfs dfs -mkdir /cn_dbpedia_input
hdfs dfs -put baike_cnpedia.txt /cn_dbpedia_input/
```

## Submit epik-gateway-binlog-encoder spark job to spark-on-yarn cluster.
```bash
${SPARK_HOME}/bin/spark-submit --class com.epik.kbgateway.DomainKnowledge2LogFile --master yarn --deploy-mode cluster --driver-memory 256M --driver-java-options "-Dspark.testing.memory=536870912" --executor-memory 6g  --num-executors 4 --executor-cores 2 /root/epik-logfile-encoder-job.jar -f /cn_dbpedia_input/baike_triples.txt -d cn_dbpedia -t /epik_log_output
```
The main class name is `com.epik.kbgateway.DomainKnowledge2LogFile`. Note We put epik-logfile-encoder-job.jar under /root dir, point to where you put it if it is not the case.

Application options:

option name| example |note
:---:|:---|---
-f | /cn_dbpedia_input/baike_triples.txt | hdfs input file, location of baike_cnpedia.txt
-d | cn_dbpedia | nebula database name. 
-t | /epik_log_output | hdfs output files, where the binlog locates.

