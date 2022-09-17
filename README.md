# 标题项目部署步骤
## 标题部署环境为CentOS7.6
##MySQL下载
wget -i -c https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm


##Maven下载
wget -i -c https://archive.apache.org/dist/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz


##tomcat下载
wget -i -c http://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz


##elasticsearch下载
wget -i -c https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.3.tar.gz
wget -i -c https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.3/elasticsearch-analysis-ik-6.4.3.zip

##kafka下载
wget -i -c https://archive.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz



##安装jdk
yum list java*
yum install -y java-11-openjdk.i686


##解压缩maven到opt目录
tar -zvxf apache-maven-3.6.1-bin.tar.gz -C /opt

##配置环境变量
vim /etc/profile

##增加一行
export MAVEN_HOME=/opt/apache-maven-3.6.1
export PATH=$PATH:$MAVEN_HOME/bin
##让配置生效
source /etc/profile

##修改Maven的配置文件，修改为阿里源


##安装MySQL
yum install -y mysql80-community-release-el7-7.noarch.rpm
yum list mysql*
yum install -y mysql-community-server.x86_64

##启动MySQL
systemctl start mysqld

##查看MySQL的状态
systemctl status mysqld

##查看MySQL的初始密码
grep 'password' /var/log/mysqld.log

##修改数据库密码
alter user root@localhost identified by 'Cyh_1998'

##创建数据库
create database community
use community

##执行sql文件
source /root/init_data/init_schema.sql;
source /root/init_data/init_data.sql;
source /root/init_data/tables_mysql_innodb.sql;


##安装Redis
yum list redis*
yum install -y redis.x86_64

##启动Redis
systemctl start redis
systemctl status redis

##启动Redis客户端
redis-cli


##安装Kafka
##解压缩
tar -zvxf kafka_2.12-2.3.0.tgz -C /opt

##启动Kafka
##后台方式启动
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
nohup bin/kafka-server-start.sh config/server.properties 1>/dev/null 2>&1 & 
##查看Kafka所有的主题
bin/kafka-topics.sh --list --bootstrap-server localhost:9092


##安装elasticsearch
tar -zvxf elasticsearch-6.4.3.tar.gz -C /opt
unzip -d /opt/elasticsearch-6.4.3/plugins/ik elasticsearch-analysis-ik-6.4.3.zip
##修改配置文件(elasticsearch.yml  跟  jvm.options)
##elasticsearch.yml增加一行
xpack.ml.enabled: false

cd /opt/elasticsearch-6.4.3/config/
##启动elasticsearch(不允许用root用户启动)
groupadd nowcoder
useradd nowcoder1 -p 1998 -g nowcoder

cd /opt
chown -R nowcoder1:nowcoder *

cd /tmp
chown -R nowcoder1:nowcoder *

##启动elasticsearch
su nowcoder1
cd /opt/elasticsearch-6.4.3/
bin/elasticsearch -d

##查看elasticsearch是否成功启动
curl -X GET "localhost:9200/_cat/health?v"


##安装wkhtmltopdf
yum list wkhtmltopdf*
yum install -y wkhtmltopdf.x86_64

yum list *xvfb*
yum install -y xorg-x11-server-Xvfb.x86_64

##测试转换
xvfb-run --server-args="-screen 0, 1024x768x24" wkhtmltoimage https://www.baidu.com 1.png

##创建脚本wkhtmltoimage.sh
xvfb-run --server-args="-screen 0, 1024x768x24" wkhtmltoimage "$@"

##安装tomcat
tar -zvxf apache-tomcat-9.0.65.tar.gz -C /opt
##配置环境变量
export TOMCAT_HOME=/opt/apache-tomcat-9.0.65
export PATH=$PATH:$TOMCAT_HOME/bin
source /etc/profile

##安装nginx
yum list nginx*
yum install -y nginx.x86_64

##修改配置文件（/etc/nginx/ngin.conf）
upstream myserver {
        server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
}

server {
        listen 80;
        server_name 1.12.242.108;
        location / {
                proxy_pass http://myserver;
        }
}

##启动nginx
systemctl start nginx
systemctl status nginx
