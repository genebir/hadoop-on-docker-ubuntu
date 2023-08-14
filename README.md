<h1>Install Hadoop On Ubuntu</h1>
<h3>Use Docker</h3>
<br/><br/>
<h6>1. Pull Ubuntu image</h6>

```bash
docker pull ubuntu:22.04
```
<br/>
<h6>2. Create Network</h6>

```bash
docker swarm init
docker network create --driver overlay --subnet 33.33.33.0/24 --attachable lc-hadoop
```
<br/>
<h6>3. Create Container</h6>

```bash
docker run -itd --network lc-hadoop --name hadoop-master --ip 33.33.33.3 -p 29870:9870 -p 28088:8088 -p 29888:19888 ubuntu:22.04 /bin/bash
docker run -itd --network lc-hadoop --name hadoop-slave1 --ip 33.33.33.4 ubuntu:22.04 /bin/bash
docker run -itd --network lc-hadoop --name hadoop-slave2 --ip 33.33.33.5 ubuntu:22.04 /bin/bash
```
<br>
<h6>4. Install Package</h6>
<p>Every Node</p>

```bash
apt-get update
apt-get upgrade -y
apt-get install -y curl openssh-server rsync wget vim iputils-ping htop openjdk-11-jdk
```
<br/>
<h6>5. Set Hosts</h6>
<p>Every Node</p>
    
```bash
vim /etc/hosts
```
```
33.33.33.3 master
33.33.33.4 worker1
33.33.33.5 worker2
```  

<br/>
<h6>6. Set SSH</h6>
<p>Every Node</p>

```bash
ssh-keygen -t rsa
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
cat ~/.ssh/authorized_keys
```
- 모든 노드에서 실행하고 각 authorized_keys의 내용을 복사하여 모든 노드의 authorized_keys에 붙여넣기
- Copy all authorized_keys and paste it to all nodes' authorized_keys

```bash
service ssh start
```

<br/>
<h6>7. Set Hadoop</h6>
<p>Every Node</p>

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
tar -xvf hadoop-3.3.1.tar.gz
rm hadoop-3.3.1.tar.gz
mv hadoop-3.3.1 /usr/local/hadoop
```
<p>Master Node</p>

```bash
cd /usr/local/hadoop
mkdir -p hadoop_tmp/hdfs/namenode
mkdir -p hadoop_tmp/hdfs/datanode
chmod 777 hadoop_tmp
```

<p>Workers Node</p>

```bash
cd /usr/local/hadoop
mkdir -p hadoop_tmp/hdfs/datanode
chmod 777 hadoop_tmp
```

<p>Every Node</p>

```bash
vi ~/.bashrc

export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
```

```bash
source ~/.bashrc
```
<p>Master Node</p>

```bash
vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh

export JAVA_HOME='/usr/lib/jvm/java-11-openjdk-amd64'

export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_NODEMANAGER_USER=root
export YARN_RESOURCEMANAGER_USER=root
```
<p>Workers Node</p>

```bash
vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh

export JAVA_HOME='/usr/lib/jvm/java-11-openjdk-amd64'
```

<p>Every Node</p>

```bash
vi $HADOOP_HOME/etc/hadoop/core-site.xml

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
</configuration>
```

<p>Master Node</p>

```bash
vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/usr/local/hadoop_tmp/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop_tmp/hdfs/datanode</value>
    </property>
</configuration>
```

<p>Workers Node</p>

```bash
vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop_tmp/hdfs/datanode</value>
    </property>
</configuration>
```

<p>Every Node</p>

```bash
vi $HADOOP_HOME/etc/hadoop/yarn-site.xml

<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
</configuration>
---
vi $HADOOP_HOME/etc/hadoop/mapred-site.xml

<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
</configuration>
---
vi $HADOOP_HOME/etc/hadoop/masters

master
---
vi $HADOOP_HOME/etc/hadoop/workers

master
worker1
worker2
```

<p>Master Node</p>

```bash
hdfs namenode -format
start-all.sh
```