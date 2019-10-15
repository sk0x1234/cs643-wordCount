# NJIT CS 643 WOrd count  by Srinath Koilakonda

Please note that Project code is derived from the MapReduce WordCount example found here: https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html 

And note that the cluster it built loosely based on the steps listed here: https://www.tutorialspoint.com/hadoop/hadoop_multi_node_cluster.htm 

## AMI Build Steps

check the AMI.txt to base setup of the linux machine. 
## My AMI
My AMI:

AMI ID: ami-0597b5282880d2404

AMI Name: Cs-643-Cloud-Srinath-koilakonda

Owner: 184366793262

Source: 184366793262/Cs-643-Cloud-Srinath-koilakonda

## Configure Cluster

Using the above custom Hadoop AMI, perform following steps:

1. Pick a master IP and slave IPs. Add following lines to /etc/hosts on each node (subsitute IP addresses with public IPs of your cluster)
```
172.31.18.34 nnode
172.31.28.215 dnode1
172.31.31.203 dnode2
172.31.18.119 dnode3

```
2. Configure passwordless login on each node, using the .pem file provided by AWS. This is assuming each node uses the same key configured by AWS. 
*  ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
*   cat id_rsa.pub >> authorized_keys   export the key to the all the datanodes

3. configure the hadoop environments 

* etc/hadoop/core-site.xml
```xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://nnode:9000</value>
	</property>
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
</configuration>
```
* etc/hadoop/yarn-site.xml
```xml
<configuration>

<!-- Site specific YARN configuration properties -->
   <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop-master</value>
  </property>
</configuration>
```
* etc/hadoop/hdfs-site.xml
```xml
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
</configuration>
```
* etc/hadoop/mapred-site.xml
```xml
<configuration>
	<property>
		<name>mapreduce.jobtracker.address</name>
		<value>nnode:54311</value>
	</property>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```
* etc/hadoop/masters
```
nnode
```
* etc/hadoop/slaves
```
dnode1
dnode2
dnode3
```
* Distribute xml files to other nodes
```bash
scp etc/hadoop/*.xml dnode1:~/hadoop/etc/hadoop/
scp etc/hadoop/*.xml dnode2-slave2:~/hadoop/etc/hadoop/
scp etc/hadoop/*.xml dnode3-slave3:~/hadoop/etc/hadoop/
```
4. Format namenode on master
```bash
bin/hadoop namenode -format
```
5. Start Hadoop Services
```bash
sbin/start-dfs.sh
sbin/start-yarn.sh
```

Here is some sample output of start-dfs.sh for my cluster, to prove I got it working in my AWS EC2 environment:
```bash
[ubuntu@ip-172-31-18-34 hadoop]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost6 localhost6.localdomain6

172.31.18.34 nnode
172.31.28.215 dnode1
172.31.31.203 dnode2
172.31.18.119 dnode3


[ubuntu@ip-172-31-18-34 hadoop]$ sbin/start-dfs.sh
Starting namenodes on [hadoop-master]
nnode: starting namenode, logging to /home/ubuntu/hadoop/logs/hadoop-ec2-user-namenode-ip-172-31-18-34.out
dnode1: starting datanode, logging to /home/ubuntu/hadoop/logs/hadoop-ec2-user-datanode-ip-172-31-29-215.out
dnode2: starting datanode, logging to /home/ubuntu/hadoop/logs/hadoop-ec2-user-datanode-ip-172-31-31-203.out
dnode3: starting datanode, logging to /home/ubuntu/hadoop/logs/hadoop-ec2-user-datanode-ip-172-31-19-119.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /home/ubuntu/hadoop/logs/hadoop-ec2-user-secondarynamenode-ip-172-31-18-34.out
```

## Run My MapReduce Project

1. Pull down my code
```bash
cd ~
git clone https://github.com/sk0x1234/cs643-wordCount
```
```
2. Make input/output directories in hdfs and move states to it
```bash
cd hadoop
bin/hdfs dfs -mkdir /input/
bin/hdfs dfs -mkdir /input/states
bin/hdfs dfs -mkdir /output/
bin/hdfs dfs -put ~/states/* /input/states/
```
4. Run code
```bash
bin/hadoop jar ~/wordCount/sw.jar StateWords /input/states /output/
bin/hdfs dfs -cat /output/job1/*
bin/hdfs dfs -cat /output/job2/*
bin/hdfs dfs -cat /output/job3/*
bin/hdfs dfs -cat /output/job4/*
```

the output of job2 and 3 cover problem 2a and the output of job4 covers problem 2b
