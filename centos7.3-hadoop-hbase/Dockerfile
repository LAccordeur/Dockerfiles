##################################
# version: rogerguo007/centos7-hadoop-hbase:base
# description: centos-7.3 for hbase (ssh, java, hadoop, hbase, zookeeper(which is built-in hbase))
# notes: the required hadoop.tar.gz and hbase.tar.gz need to be put in the context directory
#################################

FROM rogerguo007/centos-sshd:7.3

# creator infomation
MAINTAINER rogerguo007 (610641902@qq.com)

# install and configure java environment by yum
RUN yum install -y java-1.6.0-openjdk java-1.6.0-openjdk-devel && yum clean all && rm -rf /tmp/*

# install and configure hadoop and hbase
COPY hadoop-0.20.2.tar.gz hbase-0.92.0.tar.gz /usr/local/
RUN cd /usr/local && tar -zxvf hadoop-0.20.2.tar.gz && rm -f hadoop-0.20.2.tar.gz \
    && tar -zxvf hbase-0.92.0.tar.gz && rm -f hbase-0.92.0.tar.gz \
    && cd /usr/local && ln -s ./hadoop-0.20.2 hadoop && ln -s ./hbase-0.92.0 hbase

ENV HADOOP_HOME /usr/local/hadoop
ENV HBASE_HOME /usr/local/hbase
ENV PATH $PATH:${HADOOP_HOME}:${HBASE_HOME}	
