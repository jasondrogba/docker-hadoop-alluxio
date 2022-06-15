# docker-hadoop-alluxio
# 1.Overview
First of all, it is recommended to know about [Alluxio](https://docs.alluxio.io/os/user/stable/en/Overview.html) and [Hadoop](https://hadoop.apache.org/).  
Through this project, you can use Docker to quickly deploy an Alluxio cluster with under storage is HDFS.In addition, all nodes in the cluster are placed in containers, so that we can deploy the cluster on one host, which is convenient for learning and experimenting with Alluxio and Hadoop.The project can only be used in the experimental environment, and temporarily does not support the production environment.
# 2.Quick Start Guide
Download the whole project.  
```
git clone git@github.com:jasondrogba/docker-hadoop-alluxio.git
```
Move to move to directory.
```
cd docker-hadoop-alluxio
```
Start the alluxio cluster.
```
docker-compose up -d
```
If the following content is displayed on the command line, the startup is successful.
```
[+] Running 8/8
 ⠿ Container alluxio-worker   Started                                            1.2s
 ⠿ Container resourcemanager  Started                                            1.4s
 ⠿ Container alluxio-master   Started                                            1.4s
 ⠿ Container datanode1        Started                                            1.8s
 ⠿ Container namenode         Started                                            1.8s
 ⠿ Container datanode2        Started                                            1.5s
 ⠿ Container nodemanager      Started                                            1.8s
 ⠿ Container historyserver    Started                                            1.2s
 ```
 



