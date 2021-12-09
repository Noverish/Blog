---
layout: post
title: Apache Hadoop 설치하기 (Single Node) (+Docker)
date: 2021-12-09 19:45:00 +0900
cover: /covers/hadoop.png
disqusId: de8c4707b669b884b5bd1e5c771cec263fe235d7
toc: true
category: Hadoop
tags:
- hadoop
- hdfs
- docker
---

Single Node에서 Apache Hadoop을 설치하는 법을 알아보겠습니다.
그리고 글의 끝에 이 모든 내용을 한 번에 Docker로 올릴 수 있는 Dockerfile도 적어두었습니다.

<!-- more -->

<article class="message message-immersive is-warning">
  <div class="message-body">
    <i class="fas fa-exclamation-triangle mr-2"></i>
    Hadoop을 설치하기 위해서는 Java가 설치되어 있어야 합니다! Java를 설치하는 방법은 인터넷에 많으니 건너 뛰도록 하겠습니다.
  </div>
</article>

# 1. Hadoop 바이너리 다운로드

먼저 Hadoop 공식 홈페이지에서 바이너리를 다운로드 받습니다.
이 글을 작성하는 시점의 최신 버전은 `3.3.1` 이라서 이 버전으로 다운로드 받도록 하겠습니다.
현재 최신 버전을 알고 싶으면 [여기](https://hadoop.apache.org/releases.html)에서 확인해주세요.

아래 명령어를 실행해서 `Hadoop 3.3.1` 을 다운로드 받고 이를 원하는 폴더에 압축을 풀면 됩니다.
여기서는 `/usr/local` 폴더에 압축을 푸는 것으로 진행하겠습니다.

```shell shell
$ wget https://archive.apache.org/dist/hadoop/core/hadoop-3.3.1/hadoop-3.3.1.tar.gz -P ./
$ sudo tar zxf ./hadoop-3.3.1.tar.gz -C /usr/local
$ sudo mv /usr/local/hadoop-3.3.1 /usr/local/hadoop
```

# 2. SSH Key 생성

Hadoop의 각 노드들은 SSH를 이용하여 데이터를 주고 받습니다.
따라서 Hadoop을 설치 하기 전에 SSH를 설정해주어서 다른 노드에 접속할 때 비밀번호가 없이도 접속할 수 있게 해주어야 합니다.
여기서는 싱글노드라서 localhost에 접속할 것이고 localhost에 접속할 때도 ssh key는 필요합니다.

아래 명령어를 통해 SSH Key를 생성합니다.

```shell shell
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ''
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 400 ~/.ssh/id_rsa
```

그 다음 `ssh localhost` 명령어를 통해 localhost에 ssh 접속이 잘 되는지 확인합니다.

# 3. 환경 변수 설정

`~/.bashrc` 파일 맨 밑에 아래 내용을 입력하여 환경 변수를 설정해줍니다.
만약에 본인이 `zsh`를 쓰고 있으면 `~/.zshrc` 파일을 사용하면 됩니다.
만약 본인이 사용하고 있는 shell을 모른다면 `echo $0` 명령어를 사용하여 알 수 있습니다.

```shell .bashrc
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

위 내용을 입력한 후 `source ~/.bashrc` 명령어를 통해 환경 변수를 등록합니다.

그 다음에 `JAVA_HOME` 환경 변수를 아래 명령어를 통해 hadoop에 등록합니다.
아래 명령어를 쓰기 전에 `echo $JAVA_HOME` 명령어를 통해 제대로 `JAVA_HOME` 환경 변수가 등록 되어 있는지 확인하고 진행해주세요.

```shell shell
$ echo "export JAVA_HOME=$JAVA_HOME" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

# 4. 설정 파일 작성

Hadoop 설정 파일을 작성합니다. `/usr/local/hadoop/etc/hadoop/core-site.xml` 파일에 입력합니다.
아래처럼 설정을 함으로써 `9000` 포트에 hdfs 전용 포트가 열립니다.

```xml /usr/local/hadoop/etc/hadoop/core-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://0.0.0.0:9000</value>
    </property>
</configuration>
```

기본적으로 Hadoop의 모든 데이터들은 `/tmp/hadoop-${username}` 경로에 저장되도록 설정되어 있습니다.
이런 데이터 저장 경로를 변경하고 싶으면 아래의 설정을 추가하면 됩니다.

```xml /usr/local/hadoop/etc/hadoop/core-site.xml
<property>
    <name>hadoop.tmp.dir</name>
    <value>/path/to/save/hadoop/data</value>
</property>
```

<article class="message message-immersive is-primary">
  <div class="message-body">
    <i class="fas fa-info-circle mr-2"></i>
    <code>core-site.xml</code> 설정 파일에 대한 자세한 정보를 보고 싶으면
    <a href="https://hadoop.apache.org/docs/r3.3.1/hadoop-project-dist/hadoop-common/core-default.xml">여기</a>를 참고해주세요.
  </div>
</article>

# 5. 실행

```shell shell
# 먼저 Hadoop 파일 시스템(HDFS)을 포맷합니다.
$ hdfs namenode -format

# HDFS 인스턴스 구동. 이 파일은 $HADOOP_HOME/sbin 경로에 있습니다.
$ start-dfs.sh

# 아래 명령어를 통해 각 인스턴스가 제대로 떠 있는지 확인할 수 있습니다.
$ jps
4339 Jps
4230 SecondaryNameNode
4008 DataNode
3819 NameNode
```

# 6. 접속 확인

<http://localhost:9870>에 접속하면 Namenode의 정보를 확인할 수 있습니다.

![Hadoop Namenode Web UI](hadoop-namenode.png)

<http://localhost:9868>에 접속하면 Secondary Namenode의 정보를 확인할 수 있습니다.

![Hadoop Secondary Namenode Web UI](hadoop-secondary-namenode.png)

<http://localhost:9864>에 접속하면 Datanode의 정보를 확인할 수 있습니다.

![Hadoop Datanode Web UI](hadoop-datanode.png)

# 7. HDFS에 파일을 넣어보기

```shell shell
# 먼저 / 경로에 파일이 있는지 확인합니다. 당연히 아무 것도 없는 것이 정상입니다.
$ hdfs dfs -ls /

# hadoop 설치할 때 받아두었던 압축 파일을 HDFS에 넣어줍니다.
$ hdfs dfs -copyFromLocal hadoop-3.3.1.tar.gz /

# 다시 / 경로에 파일이 있는지 확인하면 아래와 같이 파일이 있는 것을 확인할 수 있습니다.
$ hdfs dfs -ls /
Found 1 items
-rw-r--r--   3 ubuntu supergroup  605187279 2021-12-09 14:40 /hadoop-3.3.1.tar.gz
```

hdfs 명령어에 대해 자세히 알고 싶다면 <https://blog.voidmainvoid.net/175>를 참고해주세요.

# 8. Docker로 한 번에 하기

지금까지 했던 것을 Docker로 한 번에 띄워버릴 수 있습니다. Dockerfile은 다음과 같습니다.

```Dockerfile Dockerfile
FROM ubuntu:20.04

RUN apt-get update

# Set TimeZone (이 작업이 없으면 openjdk 설치할 때 TimeZone 선택하라면서 막힘)
RUN ln -fs /usr/share/zoneinfo/Asia/Seoul /etc/localtime
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y tzdata

# 패키지 다운로드
RUN apt-get install -y wget vim curl net-tools dnsutils
RUN apt-get install -y openjdk-8-jdk openssh-server

# Hadoop 설치
RUN wget https://archive.apache.org/dist/hadoop/core/hadoop-3.3.1/hadoop-3.3.1.tar.gz -P ~/Downloads; \
    tar zxf ~/Downloads/hadoop-3.3.1.tar.gz -C /usr/local; \
    mv /usr/local/hadoop-3.3.1 /usr/local/hadoop; \
    rm ~/Downloads/hadoop-3.3.1.tar.gz

# ssh 설정
RUN ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ''; \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys; \
    chmod 400 ~/.ssh/id_rsa; \
    echo "Host *\n  StrictHostKeyChecking no" > ~/.ssh/config;

# Java 설정
ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre/ \
    PATH=$PATH:$JAVA_HOME/bin

# Hadoop 설정
ENV HADOOP_HOME=/usr/local/hadoop \
    HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
ENV PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
RUN echo "export JAVA_HOME=$JAVA_HOME" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh; \
    echo "export HDFS_NAMENODE_USER=root" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh; \
    echo "export HDFS_DATANODE_USER=root" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh; \
    echo "export HDFS_SECONDARYNAMENODE_USER=root" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh;

RUN echo '<?xml version="1.0" encoding="UTF-8"?>' > /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '<configuration>' >> /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '  <property>' >> /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '    <name>fs.defaultFS</name>' >> /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '    <value>hdfs://0.0.0.0:9000</value>' >> /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '  </property>' >> /usr/local/hadoop/etc/hadoop/core-site.xml; \
    echo '</configuration>' >> /usr/local/hadoop/etc/hadoop/core-site.xml;

ENTRYPOINT ["/bin/sh", "-c" , "service ssh start; hdfs namenode -format; start-dfs.sh; tail -f /dev/null"]
```

- 10번째 줄: wget, vim, curl, ifconfig, nslookup 등 유틸 설치
- 23번째 줄: 이 설정을 하지 않으면 hadoop에서 ssh를 사용할 때 정말로 접속하겠냐는 확인이 떠서 중간에 멈춥니다.
- 34~36번째 줄: hadoop을 root로 실행하기 위해서는 이런 설정이 필요합니다.

``` shell shell
# 도커 이미지 빌드
$ docker build -t hadoop:single-node .

# 도커 컨테이너 실행
$ docker run -d -p 9870:9870 -p 9868:9868 -p 9864:9864 hadoop:single-node
```
