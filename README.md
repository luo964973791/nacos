### 1、安装java mysql5.7，注意mysql8.0会有问题.

```javascript
git clone https://github.com/luo964973791/nacos.git
cd nacos
yum install ./*.rpm -y
systemctl start mysqld && systemctl enable mysqld
mysql -uroot -p

CREATE DATABASE `nacos` CHARACTER SET utf8 COLLATE utf8_general_ci;
```

### 二、导入数据表

```javascript
cd  /data && https://github.com/alibaba/nacos/releases/download/2.0.3/nacos-server-2.0.3.tar.gz
tar zxvf nacos-server-2.0.3.tar.gz
mysql -uroot -p nacos < /data/nacos/conf/nacos-mysql.sql
```

### 三、修改配置后启动集群版本nacos

```javascript
[root@node1 conf]# cat /data/nacos/conf/cluster.conf 
#
# Copyright 1999-2018 Alibaba Group Holding Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

#it is ip
#example
172.27.0.3:8848
172.27.0.4:8848
172.27.0.5:8848



[root@node1 conf]# grep -v ^# /data/nacos/conf/application.properties
server.servlet.contextPath=/nacos
server.port=8848
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://172.27.0.3:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=CST
db.user.0=root
db.password.0='Test!@123'
```

