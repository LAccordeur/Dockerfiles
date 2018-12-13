#### 主要步骤及注意点
##### 安装SSH并设置免密登录

```
FROM centos:7.3.1611

# creator infomation
MAINTAINER rogerguo007 (610641902@qq.com)

# install ssh
RUN yum -y update && yum install -y openssh openssh-server openssh-clients net-tools && yum clean all && rm -rf /tmp/*

# generate ssh host key, configure ssh to make it communicate without and change password
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key ;\
    ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key ;\
    ssh-keygen -q -N "" -t ed25519 -f /etc/ssh/ssh_host_ED25519_key ;\
    mkdir -p /root/.ssh && chown root.root /root && chmod 700 /root/.ssh ;\
	ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa && cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys \
	&& chmod 600 /root/.ssh/authorized_keys \
	&& echo -e "\tStrictHostKeyChecking no" >> ssh_config ;\
    echo 'root:root' | chpasswd
	
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```
注：不同发行版的Linux免密登录可能略有不同，可将对应命令先本地测试是否可行再写入脚本

##### 安装Java、Hadoop、HBase并设置配置（文件）

```
FROM centos:rogerguo007/centos-sshd

# creator infomation
MAINTAINER rogerguo007 (610641902@qq.com)

# install and configure java environment by yum
RUN yum install -y java-1.6.0-openjdk java-1.6.0-openjdk-devel && yum clean all && rm -rf /tmp/*

# install and configure hadoop and hbase
COPY hadoop-0.20.2.tar.gz hbase-0.92.0.tar.gz /usr/local
RUN cd /usr/local && tar -zxvf hadoop-0.20.2.tar.gz && rm -f hadoop-0.20.2.tar.gz \
    && tar -zxvf hbase-0.92.0.tar.gz && rm -f hbase-0.92.0.tar.gz \
	&& cd /usr/local && ln -s ./hadoop-0.20.2 hadoop && ln -s ./hbase-0.92.0 hbase \
	&& rm -f /usr/local/hbase/lib/hadoop-core-1.0.0.jar \
	&& cp /usr/local/hadoop/hadoop-0.20.2-core.jar /usr/local/hbase/lib/

ENV HADOOP_HOME /usr/local/hadoop
ENV HBASE_HOME /usr/local/hbase
ENV PATH $PATH:${HADOOP_HOME}:${HBASE_HOME}

# add config file
COPY hadoop-conf/* $HADOOP_HOME/conf/
COPY hbase-conf/* $HBASE_HOME/conf/
```
注：  
1. 可根据需求，修改不同的Java、Hadoop、HBase版本以及对应的配置文件生成对应的镜像文件，具体各个版本的配置文件要求可参考官方文档或博客；
2. hadoop、hbase安装包以及它们对应的配置文件目录放置Dockerfile的所在目录中；
3. 可能需要将HBase中的hadoop.jar包替换为Hadoop中的那个版本的。


##### 启动生成镜像并设置Hostname等其他初始化设置
- 启动镜像时指定hostname、端口映射等信息

```
docker run --name=master -h master -idt -p 50001:50070 -p 50001:60010 <image-id>
```

- 进入容器

```
docker exec -it <container-id> /bin/bash
```

- 配置/etc/hosts文件

```
# 通过ifconfig获取ip，hosts文件示例如下
172.17.0.3      master
172.17.0.4      slave1
172.17.0.5      slave2


```

- 进入hadoop master容器执行 ./hadoop namenode -format然后执行./hdfs.sh，通过jps查看各个节点是否启动正常，至此hadoop集群启动过程完毕
- 进入hbase master容器执行./start-hbase.sh启动hbase

##### 其他可能需要关注的问题
- 单机上多个docker容器的互联（默认是可以互联的）
- 防火墙问题，centos7.3下默认设置不影响
- 该版本hbase启动时会出现File /hbase/hbase.version could only be replicated to 0 nodes, instead of 1的WARN，但启动完毕后初步验证来看不影响基本使用
- 启动HBase shell时会提示Error: Could not find or load main class org.apache.hadoop.util.PlatformName，在之前的伪分布模式测试下未出现该问题，这里出现的该问题初步验证后也不影响使用



参考：  
[1] [docker应用-3（搭建hadoop以及hbase集群）](https://www.jianshu.com/p/293370799c6f)  
[2] [Docker下HBase学习]( https://blog.csdn.net/boling_cavalry/article/details/78041811)  
[3] [Hadoop1.0.0,HBase0.92.0三节点安装](https://www.linuxidc.com/Linux/2012-07/65670.htm)  
[4] [Hadoop0.20.2 完全分布式安装和配置](https://blog.csdn.net/eudivkfdskf/article/details/79417786)  
[5] [Dockerfile:制作可ssh登录的镜像](http://blog.51cto.com/qicheng0211/1585398)


