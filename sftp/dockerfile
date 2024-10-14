# 基于Ubuntu镜像
FROM ubuntu:latest

# 更新软件包列表并安装OpenSSH服务器
RUN apt-get update && \
    apt-get install -y openssh-server && \
    apt-get install -y vim

# 创建运行SFTP服务所需的目录
RUN mkdir /var/run/sshd

# 添加用户并设置密码（这里使用sftpuser作为示例）
RUN useradd -m testuser && \
    echo "testuser:testpasswd" | chpasswd

# 为SFTP配置SSH，使用内置的SFTP子系统（sshd_config文件中）
# RUN echo 'ForceCommand internal-sftp' >> /etc/ssh/sshd_config 
# RUN echo 'AllowTCPForwarding yes' >> /etc/ssh/sshd_config 
# RUN echo 'X11Forwarding no' >> /etc/ssh/sshd_config
# RUN echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config
RUN echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config
RUN echo 'RSAAuthentication yes' >> /etc/ssh/sshd_config
RUN echo 'PubkeyAcceptedAlgorithms +ssh-rsa' >> /etc/ssh/sshd_config
RUN echo 'AuthorizedKeysFile .ssh/authorized_keys' >> /etc/ssh/sshd_config
RUN echo 'KexAlgorithms +diffie-hellman-group1-sha1' >> /etc/ssh/sshd_config

USER testuser
RUN mkdir -p /home/testuser/.ssh 
# 这里可以配置私钥信息

USER root

# 容器需要开放22端口
EXPOSE 22

# 启动SSH服务
CMD ["/usr/sbin/sshd", "-D"]

