---
title: "Spark on Kubernetes with Docker Desktop"
last_modified_at: 2021-05-02
tags:
  - spark
  - kubernetes
  - docker-desktop
---
# Intro
  Hello Apache Spark user. Spark on Kubernetes is finally [officially GA](https://issues.apache.org/jira/browse/SPARK-33005) 
and became a part of the [Spark 3.1.1 release](https://spark.apache.org/releases/spark-release-3-1-1.html).
  In this article I'll try to summarize the steps needed to run spark-pi example on kubernetes 
using docker-desktop installation. I believe it's the fastest way to get started with Kubernetes.
All commands are executed on MacOS and assume that [brew](https://brew.sh/) is already installed.
Commands may differ a bit from the one you'll need to run due to the new Spark releases.

# Commands

1. Install [docker-desktop](https://www.docker.com/products/docker-desktop) + 
[enable Kubernetes](https://docs.docker.com/desktop/kubernetes/#enable-kubernetes)

2. Install latest apache-spark locally
   ```
   brew install apache-spark
   ```
3. Get path to the current installation
   ```
   brew info apache-spark
   ```
4. Go to the path from the previous command
   ```
   cd /usr/local/Cellar/apache-spark/3.1.1/libexec
   ```
5. Build Spark docker image `spark:v3.1.1-hadoop3.2` locally
   ```
   ./bin/docker-image-tool.sh  -t v3.1.1-hadoop3.2 build
   ```
6. List Docker images to check it's build and ready
   docker images
   ```
   REPOSITORY   TAG                IMAGE ID       CREATED          SIZE
   spark        v3.1.1-hadoop3.2   c052996102ec   50 seconds ago   532MB
   ```

7. Submit spark-pi application to the local Kubernetes cluster.
   Note that jar file is already packed into docker image. 
   ```
   spark-submit \                                                               
   --master k8s://https://kubernetes.docker.internal:6443 \
   --deploy-mode cluster \
   --name spark-pi \
   --class org.apache.spark.examples.SparkPi \
   --conf spark.executor.instances=5 \
   --conf spark.kubernetes.container.image=spark:v3.1.1-hadoop3.2 \
   local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar
   ```
8. Once the job completes you can expect spark driver logs to find out computed PI value
   ```
   kubectl get pods                              
   NAME                                    READY   STATUS      RESTARTS   AGE
   spark-pi-951ce5792ed082ad-driver        0/1     Completed   0          2m23s
   
   kubectl logs  spark-pi-951ce5792ed082ad-driver
   ```
   Alter

You can find more details on running Spark on Kubernetes in [official documentation](https://spark.apache.org/docs/latest/running-on-kubernetes.html).
