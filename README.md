<style>
.step {
    color: skyblue;
    padding-top: 20pt;
}
</style>
<h1>Install Hadoop On Ubuntu</h1>

<h3>Use Docker</h3>

<p class="step" style="padding-top: 40pt">1. Pull Ubuntu image</p>

```bash
docker pull ubuntu:22.04
```

<p class="step">2. Create Network</p>

```bash
docker swarm init
docker network create --driver overlay --subnet 33.33.33.0/24 --attachable lc-hadoop
```

<p class="step">3. Create Container</p>

```bash
docker run -itd --network lc-hadoop --name hadoop-master --ip 33.33.33.3 -p 29870:9870 -p 28088:8088 -p 29888:19888 ubuntu:22.04 /bin/bash
docker run -itd --network lc-hadoop --name hadoop-slave1 --ip 33.33.33.4 ubuntu:22.04 /bin/bash
docker run -itd --network lc-hadoop --name hadoop-slave2 --ip 33.33.33.5 ubuntu:22.04 /bin/bash
```

<p class="step">4. Install Package</p>
<p>Every Node</p>

```bash
apt-get update
apt-get upgrade -y
apt-get install -y curl openssh-server rsync wget vim iputils-ping htop openjdk-11-jdk
```
