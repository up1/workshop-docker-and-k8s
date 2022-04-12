# Step to build and run

## 1. Build image for [Apache Hadoop](https://hadoop.apache.org/releases.html)
```
$docker image build -t somkiat/hadoop-base ./base
$docker image ls
```

## 2. Build image for Name node
```
$docker image build -t somkiat/hadoop-namenode ./namenode
$docker image ls
```

## 3. Build image for Data node
```
$docker image build -t somkiat/hadoop-datanode ./datanode
$docker image ls
```

Build image with Docker-compose
```
$docker-compose -f docker-compose-build.yml build base
$docker-compose -f docker-compose-build.yml build namenode
$docker-compose -f docker-compose-build.yml build datanode
```

## 4. Create cluster with Docker-compose
```
$docker-compose -f docker-compose-deploy.yml up -d

$docker-compose -f docker-compose-deploy.yml ps

  Name            Command              State                           Ports
-------------------------------------------------------------------------------------------------
datanode   /entrypoint.sh /run.sh   Up (healthy)   9864/tcp
namenode   /entrypoint.sh /run.sh   Up (healthy)   0.0.0.0:9000->9000/tcp, 0.0.0.0:9870->9870/tc
```

Checking URL
* Namenode: http://localhost:9870/dfshealth.html
* Datanode: http://localhost:9864

## 5. Run testing app :: [Example app Wordcount](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0)
* [Download JAR file](https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-examples/3.3.2/hadoop-mapreduce-examples-3.3.2.jar)

Copy jar file to Namenode to run test job
```
$docker container cp hadoop-mapreduce-examples-3.3.2.jar namenode:/tmp/
```

Create input file and copy to namenode
```
$vi data_test.txt

line1
line2
line3

$docker container cp data_test.txt namenode:/tmp/
```

Access to name node
```
$docker container exec -it namenode /bin/bash

/# hdfs dfs -mkdir -p /user/root/input
/# hdfs dfs -put /tmp/data_test.txt /user/root/input

Run
/# hadoop jar /tmp/hadoop-mapreduce-examples-3.3.2.jar wordcount input output

See success result
/# hdfs dfs -ls /user/root/output

Found 2 items
-rw-r--r--   3 root supergroup          0 2022-04-11 15:45 /user/root/output/_SUCCESS
-rw-r--r--   3 root supergroup         24 2022-04-11 15:45 /user/root/output/part-r-00000


/# hdfs dfs -cat /user/root/output/part-r-00000
line1	1
line2	1
line3	1
```

## 6. Delete all
```
$docker-compose -f docker-compose-deploy.yml down
```
