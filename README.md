## Prerequisites

> - Docker Guides：<https://docs.docker.com/get-started/>
> - Docker CLI：<https://docs.docker.com/engine/reference/run/>
> - Dockerfile Reference： <https://docs.docker.com/engine/reference/builder/>
> - Docker Compose file reference：<https://docs.docker.com/compose/compose-file/>
> - 在CentOS上安装Docker CE：<https://docs.docker.com/install/linux/docker-ce/centos/>
> - Docker Tutorial Samples：<https://github.com/docker/labs/tree/master>
> - 十分钟学会用docker部署微服务：<https://my.oschina.net/u/3796575/blog/1838385?nocache=1530498237368>
> - Docker+Jenkins持续集成环境 1~5：<https://www.cnblogs.com/xiaoqi/p/docker-jenkins-cicd.html>
> - Docker监控工具：<http://dockone.io/article/397>
> - Docker容器监控的实现：<https://www.ipcpu.com/2016/04/docker-monitor/>
> - 搭建可自动化构建的微服务框架：<https://cloud.tencent.com/developer/article/1056972>



## 项目部署建议

### 一. 搭建Docker私有库

> - 快速代建私有镜像库：<https://blog.51cto.com/ganbing/2080140>
> - docker私有库搭建过程： <https://www.cnblogs.com/cloud-it/p/7070198.html>
> - 私有库改造：<https://blog.csdn.net/egworkspace/article/details/80518647>

访问Registry ：http://ip:port/v2/_catalog

```shell
## 示例
[root@develop ~]# curl http://127.0.0.1:5000/v2/_catalog
{"repositories":[]}
```



### 二. 使用Dockerfile构建项目镜像

> - 创建JDK镜像：<https://blog.csdn.net/qq_35981283/article/details/80738451>
> - 创建Java应用镜像：<https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch03-build-image.adoc>
> - 创建其他镜像：<https://docs.docker.com/samples/>

1. 构建JDK镜像并上传至私有库中（同理可以搞定REDIS、MYSQL、ZOOKEEPER等各类需要的镜像）

   ```shell
   # 一. 拉取centos7镜像
   [root@develop ~]# docker pull centos:7
   7: Pulling from library/centos
   8ba884070f61: Pull complete 
   Digest: sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861
   Status: Downloaded newer image for centos:7
   # 二. 去官网下载需要的jdk版本，并将jdk上传至服务器上，用于构建镜像
   [root@develop jdk]# pwd
   /opt/jdk
   [root@develop jdk]# ls
   jdk-8u211-linux-x64.tar.gz
   # 三. 创建Dockerfile
   [root@develop jdk]# vi Dockerfile
   [root@develop jdk]# ls
   Dockerfile  jdk-8u211-linux-x64.tar.gz
   [root@develop jdk]# cat Dockerfile 
   FROM centos:7
   LABEL maintainer="xxx@xxx.com"
   ADD jdk-8u211-linux-x64.tar.gz /usr/local/
   ENV JAVA_HOME /usr/local/jdk1.8.0_211
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ENV PATH $PATH:$JAVA_HOME/bin
   # 四. 构建镜像
   [root@develop jdk]# docker build -t jdk-8u211-linux-x64:20190508 .
   Sending build context to Docker daemon    195MB
   Step 1/6 : FROM centos:7
    ---> 9f38484d220f
   Step 2/6 : LABEL maintainer="xxx@xxx.com"
    ---> Running in eeb9ab978bd0
   Removing intermediate container eeb9ab978bd0
    ---> 205d7066523e
   Step 3/6 : ADD jdk-8u211-linux-x64.tar.gz /usr/local/
    ---> d0fcd6f2e8b3
   Step 4/6 : ENV JAVA_HOME /usr/local/jdk1.8.0_211
    ---> Running in 2386945f45ef
   Removing intermediate container 2386945f45ef
    ---> 3bf34cd7b483
   Step 5/6 : ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ---> Running in 03bd8af13bab
   Removing intermediate container 03bd8af13bab
    ---> 5dc73e611a3a
   Step 6/6 : ENV PATH $PATH:$JAVA_HOME/bin
    ---> Running in f18d324b3b84
   Removing intermediate container f18d324b3b84
    ---> 84aed422bc6d
   Successfully built 84aed422bc6d
   Successfully tagged jdk-8u211-linux-x64:20190508
   [root@develop jdk]# docker images
   REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
   jdk-8u211-linux-x64   20190508            84aed422bc6d        46 seconds ago      608MB
   centos                7                   9f38484d220f        7 weeks ago         202MB
   registry              latest              f32a97de94e1        2 months ago        25.8MB
   # 五. 运行并验证镜像是否有效
   [root@develop jdk]# docker run -d -it jdk-8u211-linux-x64:20190508 /bin/bash
   ad357a506b57a88d59fd6cc8a4644702643e12ce600e162aacc63eb950236021
   [root@develop jdk]# docker ps
   CONTAINER ID  IMAGE    COMMAND    CREATED       STATUS         PORTS               NAMES
   ad357a506b57  jdk-8u211-xxx  "/bin/bash" 21 seconds ago Up 20 seconds        dreamy_spence
   24aed5ebe853  registry  "/entry…"  3h ago Up 3 h  0.0.0.0:5000->5000/tcp     suspicious_clarke
   [root@develop jdk]# docker exec -it ad357 /bin/bash
   [root@ad357a506b57 /]# java -version
   java version "1.8.0_211"
   Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
   Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
   # 六. 将JDK镜像上传至Registry中，需指定ip和port
   [root@develop jdk]# docker tag jdk-8u211-linux-x64:20190508 127.0.0.1:5000/jdk-8u211-linux-x64:20190508
   [root@develop jdk]# docker images
   REPOSITORY                           TAG          IMAGE ID         CREATED           SIZE
   127.0.0.1:5000/jdk-8u211-linux-x64   20190508    84aed422bc6d    14 minutes ago      608MB
   jdk-8u211-linux-x64                  20190508    84aed422bc6d    14 minutes ago      608MB
   centos                               7           9f38484d220f    7 weeks ago         202MB
   registry                             latest      f32a97de94e1    2 months ago        25.8MB
   [root@develop jdk]# docker push 127.0.0.1:5000/jdk-8u211-linux-x64:20190508
   The push refers to repository [127.0.0.1:5000/jdk-8u211-linux-x64]
   9ed6d60f0bd4: Pushed 
   d69483a6face: Pushed 
   20190508: digest: sha256:afdfd9625abe7df6de6f377fdce51a9bf17251bb1822ecbceab8ae9 size: 742
   [root@develop jdk]# curl http://127.0.0.1:5000/v2/_catalog
   {"repositories":["jdk-8u211-linux-x64"]}
   ```

2. 在JDK镜像的基础上，构建项目镜像，并上传至私有库

   ```shell
   # 一. 要使用自己的镜像，先修改docker镜像源
   [root@develop ~]# vim /etc/sysconfig/docker 
   [root@develop ~]# cat /etc/sysconfig/docker 
   OPTIONS='--selinux-enabled --insecure-registry=127.0.0.1:5000'
   [root@develop ~]# systemctl restart docker
   # 二. 在项目根目录下创建Dockerfile,以dubbo的demo-provider,demo-consumer为例
           # demo-provider-dockerfile
           FROM jdk-8u211-linux-x64:20190508
           LABEL maintainer="xxx@xxx.com"
           COPY target/provider-1.0.0-SNAPSHOT.jar /usr/demo/provider.jar
           WORKDIR /usr/demo
           EXPOSE 8080
           CMD ["java", "-jar", "provider.jar"]
           # demo-consumer-dockerfile
           FROM jdk-8u211-linux-x64:20190508
           LABEL maintainer="xxx@xxx.com"
           COPY target/consumer-1.0.0-SNAPSHOT.jar /usr/demo/consumer.jar
           WORKDIR /usr/demo
           EXPOSE 8081
           CMD ["java", "-jar", "consumer.jar"]
   # 三. 将项目target包上传至服务器，使用Dockerfile构建镜像
   [root@develop provider]# pwd
   /opt/demo/provider
   [root@develop provider]# ls
   Dockerfile  target
   [root@develop provider]# docker build -t demo-provider:20190508 .
   Sending build context to Docker daemon  17.84MB
   Step 1/6 : FROM jdk-8u211-linux-x64:20190508
    ---> 84aed422bc6d
   Step 2/6 : LABEL maintainer="xxx@xxx.com"
    ---> Running in 6c58d957d5b0
   Removing intermediate container 6c58d957d5b0
    ---> 7432a28272bb
   Step 3/6 : COPY target/provider-1.0.0-SNAPSHOT.jar /usr/demo/provider.jar
    ---> eef32e4c0063
   Step 4/6 : WORKDIR /usr/demo
    ---> Running in 0d342561a568
   Removing intermediate container 0d342561a568
    ---> 6d8f13b08090
   Step 5/6 : EXPOSE 8080
    ---> Running in 0b89fbc41e94
   Removing intermediate container 0b89fbc41e94
    ---> 0273bb6f2b56
   Step 6/6 : CMD ["java", "-jar", "provider.jar"]
    ---> Running in 25c5a8ebf260
   Removing intermediate container 25c5a8ebf260
    ---> 17dd186ae884
   Successfully built 17dd186ae884
   Successfully tagged demo-provider:20190508
   [root@develop consumer]# pwd
   /opt/demo/consumer
   [root@develop consumer]# ls
   Dockerfile  target
   [root@develop consumer]# docker build -t demo-consumer:20190508 .
   Sending build context to Docker daemon   27.1MB
   Step 1/6 : FROM jdk-8u211-linux-x64:20190508
    ---> 84aed422bc6d
   Step 2/6 : LABEL maintainer="xxx@xxx.com"
    ---> Using cache
    ---> 7432a28272bb
   Step 3/6 : COPY target/consumer-1.0.0-SNAPSHOT.jar /usr/demo/consumer.jar
    ---> 081903838672
   Step 4/6 : WORKDIR /usr/demo
    ---> Running in abdbb88117b7
   Removing intermediate container abdbb88117b7
    ---> 336e80766671
   Step 5/6 : EXPOSE 8081
    ---> Running in f8caf5a72bc6
   Removing intermediate container f8caf5a72bc6
    ---> 56683bb7d660
   Step 6/6 : CMD ["java", "-jar", "consumer.jar"]
    ---> Running in 84d39e22dd30
   Removing intermediate container 84d39e22dd30
    ---> dcc87e7b87c9
   Successfully built dcc87e7b87c9
   Successfully tagged demo-consumer:20190508
   [root@develop consumer]# docker images
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   demo-consumer        20190508            dcc87e7b87c9        38 seconds ago      635MB
   demo-provider        20190508            17dd186ae884        3 minutes ago       626MB
   127.0.0.1:5000/jdk-8u211-linux-x64  20190508  84aed422bc6d   20 hours ago        608MB
   jdk-8u211-linux-x64        20190508            84aed422bc6d   20 hours ago      608MB
   centos              7                   9f38484d220f        7 weeks ago         202MB
   registry           latest              f32a97de94e1        2 months ago        25.8MB
   # 四. 将镜像上传至Registry中
   [root@develop provider]# docker tag demo-provider:20190508 127.0.0.1:5000/demo-provider:v1.0.0
   [root@develop provider]# docker push 127.0.0.1:5000/demo-provider:v1.0.0
   The push refers to repository [127.0.0.1:5000/demo-provider]
   84a6e22f5958: Pushed 
   9ed6d60f0bd4: Mounted from jdk-8u211-linux-x64 
   d69483a6face: Mounted from jdk-8u211-linux-x64 
   v1.0.0: digest: sha256:ef0f57c37075203897196e18800dfd7d70f11aff41e12fba0ba76 size: 954
   [root@develop consumer]# docker tag demo-consumer:20190508 127.0.0.1:5000/demo-consumer:v1.0.0
   [root@develop consumer]# docker push 127.0.0.1:5000/demo-consumer:v1.0.0
   The push refers to repository [127.0.0.1:5000/demo-consumer]
   ad1afd3719a6: Pushed 
   9ed6d60f0bd4: Mounted from demo-provider 
   d69483a6face: Mounted from demo-provider 
   v1.0.0: digest: sha256:dd0eac2d6a703b0dc23b832d12afbbe97a7fe89768ba419a size: 954
   [root@develop consumer]# curl http://127.0.0.1:5000/v2/_catalog
   {"repositories":["demo-consumer","demo-provider","jdk-8u211-linux-x64"]}
   # 五. 示例项目需要zookeeper，所以讲zookeeper也上传至Registry中
   [root@develop ~]# docker pull zookeeper
   [root@develop ~]# docker tag zookeeper 127.0.0.1:5000/zookeeper:20190508
   [root@develop ~]# docker push 127.0.0.1:5000/zookeeper:20190508
   The push refers to repository [127.0.0.1:5000/zookeeper]
   90f4e39e7f3f: Pushed 
   53daba0f3e03: Pushed 
   772332ba3d00: Pushed 
   752857429115: Pushed 
   b280365ecb1e: Pushed 
   dee6aef5c2b6: Pushed 
   a464c54f93a9: Pushed 
   20190508: digest: sha256:e81a9dedbe3abe05a7d4ef4c3bb44baf65f2ceea8473479c40170b size: 1785
   ```



### 三. 使用Docker Compose

> - 使用Docker Compose编排容器：<https://docs.docker.com/compose/install/>

1. ```shell
   # 虽然可以通过docker run命令运行容器，但是如果项目包含多个服务，每个服务又要部署多个实例，通过手动运行
   # 效率太低，这就可以用Docker Compose来管理
   
   # 一. 安装Docker Compose和命令补全工具
   [root@develop ~]# sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   [root@develop ~]# sudo chmod +x /usr/local/bin/docker-compose
   [root@develop ~]# docker-compose --version
   docker-compose version 1.24.0, build 0aa59064
   [root@develop ~]# sudo curl -L https://raw.githubusercontent.com/docker/compose/1.24.0/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
   # 二. 编写docker-compose.yml
   [root@develop docker-compose]# pwd
   /opt/demo/docker-compose
   [root@develop docker-compose]# vim docker-compose.yml
   [root@develop docker-compose]# cat docker-compose.yml 
   version: "3"
   services:
     demo-provider:
       image: "127.0.0.1:5000/demo-provider:${TAG}"
       restart: always
       ports:
         - "8080:8080"
       network:
   
     demo-consumer:
       image: ""
   [root@develop docker-compose]# cat docker-compose.yml 
   version: "3"
   services:
     demo-provider:
       image: "127.0.0.1:5000/demo-provider:${TAG}"
       restart: on-failure
       #container_name: demo-provider
       #deploy:
         #replicas: 2
         #restart_policy:
           #condition: on-failure
         #resources:
           #limits:
             #cpus: "0.1"
             #memory: 50M
       #ports:
         #- "8080:8080"
       network_mode: "host"
   
     demo-consumer:
       image: "127.0.0.1:5000/demo-consumer:${TAG}"
       restart: on-failure
       ports:
         - "8081:8081"
       network_mode: "host"
     
     zookeeper:
       image: "127.0.0.1:5000/zookeeper:20190508"
       restart: on-failure
       hostname: zoo1
       ports:
         - 2181:2181
       environment:
         ZOO_MY_ID: 1
         ZOO_SERVERS: server.1=0.0.0.0:2888:3888
   [root@develop docker-compose]# vim .env
   [root@develop docker-compose]# cat .env
   TAG=v1.0.0
   # 三. 启动容器，访问服务
   [root@develop docker-compose]# docker-compose up -d
   Creating network "docker-compose_default" with the default driver
   Creating docker-compose_demo-consumer_1 ... done
   Creating docker-compose_demo-provider_1 ... done
   Creating docker-compose_zookeeper_1     ... done
   [root@develop docker-compose]# docker-compose ps
     Name                           Command                  State                     Ports   
   docker-compose_demo-consumer_1   java -jar consumer.jar     Up                          
   docker-compose_demo-provider_1   java -jar provider.jar     Up                       
   docker-compose_zookeeper_1  /docker-en...    Up    0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
   [root@develop docker-compose]# curl 127.0.0.1:8081
   Hello, World !
   ```
