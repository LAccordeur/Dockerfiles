##################################
# version: rogerguo007/ssh:v0.0
# description: centos-7.3 with ssh
#################################

FROM centos:7.3.1611

# creator infomation
MAINTAINER rogerguo007 (610641902@qq.com)

# install ssh
RUN yum -y update && yum install -y openssh openssh-server openssh-clients && yum clean all && rm -rf /tmp/*

# generate ssh key and change password
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key ;\
    ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key ;\
    ssh-keygen -q -N "" -t ed25519 -f /etc/ssh/ssh_host_ED25519_key;\
    mkdir -p /root/.ssh && chown root.root /root && chmod 700 /root/.ssh ;\
    ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa && cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys \
    && chmod 600 /root/.ssh/authorized_keys \
    && echo -e "\tStrictHostKeyChecking no" >> /etc/ssh/ssh_config ;\
    echo 'root:root' | chpasswd 
    	
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]    
