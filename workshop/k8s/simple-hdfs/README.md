## Step to deploy

## 1. Deploy ConfigMap
```
$kubectl apply -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/config.yml
$kubectl get configmap

NAME               DATA   AGE
hdfs-config        8      13s
kube-root-ca.crt   1      4h10m
```

## 2. Deploy NameNode

Create Statefulset and Pod
```
$kubectl apply -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/namenode-statefulset.yml
$kubectl get statefulset

NAME            READY   AGE
hdfs-namenode   1/1     50s

$kubectl get all

NAME                  READY   STATUS    RESTARTS   AGE
pod/hdfs-namenode-0   1/1     Running   0          45s
pod/nginx             1/1     Running   0          4h15m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4h23m

NAME                             READY   AGE
statefulset.apps/hdfs-namenode   1/1     45s

$kubectl logs pod/hdfs-namenode-0 --follow
```

Create service
```
$kubectl apply -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/namenode-service.yml

$kubectl get svc

NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
hdfs-namenode   ClusterIP   None         <none>        9870/TCP   9s
kubernetes      ClusterIP   10.96.0.1    <none>        443/TCP    4h24m
```

## 3. Deploy DataNode (3 nodes)

Create Statefulset and Pod
```
$kubectl apply -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/datanode-statefulset.yml
$kubectl get statefulset

NAME            READY   AGE
hdfs-datanode   3/3     15s
hdfs-namenode   1/1     113s

$kubectl get pods

NAME              READY   STATUS    RESTARTS   AGE
hdfs-datanode-0   1/1     Running   0          48s
hdfs-namenode-0   1/1     Running   0          2m26s

$kubectl logs pod/hdfs-datanode-0 --follow
```

Create service
```
$kubectl apply -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/datanode-service.yml

$kubectl get svc

NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
hdfs-datanode   ClusterIP   None         <none>        9874/TCP   21s
hdfs-namenode   ClusterIP   None         <none>        9870/TCP   2s
kubernetes      ClusterIP   10.96.0.1    <none>        443/TCP    4h43m
```

Check result 
```
$kubectl port-forward pod/hdfs-namenode-0   9870:9870 --address="0.0.0.0"
```
URL of NameNode = http://ip:9870/dfshealth.html


## 4. Run testing app :: [Example app Wordcount](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0)
* [Download JAR file](https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-examples/3.3.2/hadoop-mapreduce-examples-3.3.2.jar)

Copy jar file to Namenode to run test job
```
$kubectl cp hadoop-mapreduce-examples-3.3.2.jar default/hdfs-namenode-0:/tmp
```

Create input file and copy to namenode
```
$vi data_test.txt

line1
line2
line3

$kubectl cp data_test.txt default/hdfs-namenode-0:/tmp
```

Access to name node
```
$kubectl exec --stdin --tty hdfs-namenode-0 -- /bin/bash

/# hdfs dfs -mkdir -p /user/root/input
/# hdfs dfs -put /tmp/data_test.txt /user/root/input

Run
/# hadoop jar /tmp/hadoop-mapreduce-examples-3.3.2.jar wordcount input output

See success result
/# hdfs dfs -ls /user/root/output

Found 2 items
-rw-r--r--   3 root supergroup          0 2022-04-11 15:45 /user/root/output/_SUCCESS
-rw-r--r--   3 root supergroup         24 2022-04-11 15:45 /user/root/output/part-r-00000
```

/# hdfs dfs -cat /user/root/output/part-r-00000
line1	1
line2	1
line3	1

## 5. Delete all
```
$kubectl delete -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/namenode-statefulset.yml
$kubectl delete -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/namenode-service.yml

$kubectl delete -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/datanode-statefulset.yml
$kubectl delete -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/datanode-service.yml

$kubectl delete -f https://raw.githubusercontent.com/up1/workshop-docker-and-k8s/main/workshop/k8s/simple-hdfs/config.yml
```