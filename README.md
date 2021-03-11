# Epik knowledge graph binary log file encoder.

## How to submit domain-triple-files----> log files encoder job to spark on yarn.

Follow the instructions of [nebula-java](https://github.com/vesoft-inc/nebula-java) to build and install nebula-java client.

```bash
${SPARK_HOME}/bin/spark-submit --class com.epik.kbgateway.DomainKnowledge2LogFile --master yarn --deploy-mode cluster --driver-memory 256M --driver-java-options "-Dspark.testing.memory=536870912" --executor-memory 6g  --num-executors 4 --executor-cores 2 /root/epik-logfile-encoder-job.jar -f /cn_dbpedia_input/baike_triples.txt -d cn_dbpedia -t /epik_log_output
```
