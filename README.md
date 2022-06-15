# docker-hadoop-alluxio
# 1.Overview
First of all, it is recommended to know about [Alluxio](https://docs.alluxio.io/os/user/stable/en/Overview.html) and [Hadoop](https://hadoop.apache.org/).  
Through this project, you can use Docker to quickly deploy an Alluxio cluster with under storage is HDFS.In addition, all nodes in the cluster are placed in containers, so that we can deploy the cluster on one host, which is convenient for learning and experimenting with Alluxio and Hadoop.The project can only be used in the experimental environment, and temporarily does not support the production environment.Finally, an example is given using mapreduce based on hdfs and alluxio respectively.
# 2.Quick Start Guide
## 2.1 Deploy Alluxio container cluster
Download the whole project: 
```
git clone git@github.com:jasondrogba/docker-hadoop-alluxio.git
```
Move to move to directory:
```
cd docker-hadoop-alluxio
```
Start the alluxio cluster:
```
docker-compose up -d
```
If the following content is displayed on the command line, the startup is successful:
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
## 2.2 Start ssh
Enter the alluxio-master container and start ssh：
```
docker exec alluxio-master /etc/init.d/ssh start
```
Enter the alluxio-worker container and start ssh:
```
docker exec alluxio-worker /etc/init.d/ssh start
```

## 2.3 Configure basic information
Get the hostnames of alluxio-master, alluxio-worker and namenode:
```
docker exec <CONTAINER_NAME> hostname
```
Enter the alluxio-master container:
```
docker exec -it alluxio-master bash
```
Enter the /opt/alluxio directory in the container and modify the conf/alluxio-site-properties file:
```
cd /opt/alluxio
vim conf/alluxio-site-properties
```
Modify conf/alluxio-site-properties:
```
# Common properties
 alluxio.master.hostname=<ALLUXIO_MASTER_HOSTNAME>
 alluxio.master.mount.table.root.ufs=hdfs://<NAMENODE_HOSTNAME>:9000
 ```
 Add alluxio-master and alluxio-worker hostnames in conf/masters and conf/workers respectively.
 ```
 //Add aluxio-master hostname in conf/masters.
 <ALLUXIO_MASTER_HOSTNAME>
 ...
 //Add aluxio-worker hostname in conf/workers.
 <ALLUXIO_WORKER_HOSTNAME>
```
Distribute configuration information to all nodes in the cluster:
```
./bin/alluxio copyDir conf/
```
## 2.4 Launch Alluxio cluster
Alluxio needs to be formatted before starting the process. The following command formats the Alluxio journal and worker storage directories.Execute in the alluxio-master container：
```
./bin/alluxio format
```
In the master node, start the Alluxio cluster with the following command:
```
./bin/alluxio-start.sh all SudoMount
```
This will start Alluxio masters on all the nodes specified in conf/masters, and start the workers on all the nodes specified in conf/workers. Argument SudoMount indicates to mount the RamFS on each worker using sudo privilege, if it is not already mounted.  
This will start one Alluxio master and one Alluxio worker locally. You can see the master UI at http://localhost:19999.  
If the following content is displayed on the command line, the startup is successful:
```
-----------------------------------------
Starting to monitor all remote services.
-----------------------------------------
--- [ OK ] The master service @ <ALLUXIO_MASTER_HOSTNAME> is in a healthy state.
--- [ OK ] The job_master service @ <ALLUXIO_MASTER_HOSTNAME> is in a healthy state.
--- [ OK ] The worker service @ <ALLUXIO_WORKER_HOSTNAME> is in a healthy state.
--- [ OK ] The job_worker service @ <ALLUXIO_WORKER_HOSTNAME> is in a healthy state.
--- [ OK ] The proxy service @ <ALLUXIO_WORKER_HOSTNAME> is in a healthy state.
--- [ OK ] The proxy service @ <ALLUXIO_MASTER_HOSTNAME> is in a healthy state.
```
Alluxio comes with a simple program that writes and reads sample files in Alluxio. Run the sample program with:
```
./bin/alluxio runTests
```
## 2.5 Web UI
alluxio-master:<IP_address>:19999  
namenode:<IP_address>:9870  
datanode1:<IP_address>:9864  
datanode2:<IP_address>:9863  
resource manager:<IP_address>:8088  
history server:<IP_address>:8188  
# 3. Example: Mapreduce
First, you need to copy alluxio-2.8.0-client.jar to the directory of all hadoop containers. Alluxio-2.8.0-client.jar was originally in the alluxio file, but now it has been downloaded to the project file. Execute the mapreduce.sh file:
```
./mapreduce.sh
```
Enter the alluxio-master container, create the wordcount directory, and copy the LICENSE file into the file system:
```
//alluxio-master container
./bin/alluxio fs mkdir /wordcount
./bin/alluxio fs copyFromLocal LICENSE /wordcount/input.txt
```
This command will copy the LICENSE file into the Alluxio namespace with the path /wordcount/input.txt.  
Enter the namenode container, we can run a MapReduce job (using Hadoop 3.2.1 as example) for wordcount:
```
//namenode container
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount alluxio://<ALLUXIO_MASTER_HOSTNAME>:19998/wordcount/input.txt alluxio://<ALLUXIO_MASTER_HOSTNAME>:19998/wordcount/output
```
After this job completes, the result of the wordcount will be in the /wordcount/output directory in Alluxio. You can see the resulting files by running:
```
./bin/alluxio fs ls /wordcount/output
./bin/alluxio fs cat /wordcount/output/part-r-00000
```
On the resource manager:<IP_address>:8088 Web page, you can see the application information.
# 4. Conclusion
You have now successfully deployed an alluxio cluster with HDFS as the underlying storage and ran the wordcount example using mapreduce.





