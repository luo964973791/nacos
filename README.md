### 1、安装java mysql5.7，注意mysql8.0会有问题.

```javascript
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum repolist enabled | grep mysql
yum install java-1.8.0-openjdk* mysql-community-server -y
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

### 四、设置开机自启动

```javascript
[root@node1 nacos]# cat /etc/systemd/system/nacos.service 
[Unit]
Description=nacos
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/data/nacos/bin/startup.sh
ExecStop=/data/nacos/bin/shutdown.sh
ExecReload=/bin/kill -s HUP $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

### 五、启动nacos集群

```javascript
systemctl start nacos && systemctl enable nacos
```









### 六、k8s容器启动nacos集群

```javascript
git clone https://github.com/luo964973791/nacos.git

#创建数据表
mysql -uroot -p

CREATE DATABASE `nacos` CHARACTER SET utf8 COLLATE utf8_general_ci;

```

### 七、修改模板

```javascript
cd nacos && vi values.yaml

# 副本个数
replicaCount: 3


service:
  type: ClusterIP  # 一般不用修改
  ingressPort: 8848
  ports:
    http:
      port: 8848  # Service port number for client-a port.
      protocol: TCP  # Service port protocol for client-a port.


persistentVolume:   # 是否存储持久化
  enabled: true     # 启用持久化存储
  storageClass: "csi-rbd-sc"     # 设置 storageClass,使用kubectl get storageclass -A查看.
  accessMode: ReadWriteOnce
  annotations: {}
    # helm.sh/resource-policy: keep
  size: 1Gi  # 大小为 10G


ingress:  # 是否使用nginx暴露域名或端口
  enabled: false     # 启用 ingress
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  ingressClassName: ""
  path: /nacos
  pathType: ImplementationSpecific
  hosts:
    - nacos.evescn.com    # 设置域名
  tls: [] # 未启动 https认证


mysql:
  # false表示启用外部存储
  enabled: false
  # 可以自建 nacos 用户，这里此处直接使用了 root 账户进行 mysql 集群登陆
  # 需要去数据库新建 nacos 数据库
  external:
    mysqlMasterHost: "172.27.0.3"
    mysqlDatabase: "nacos"
    mysqlMasterPort: "3306"
    mysqlMasterUser: "root"
    mysqlMasterPassword: "root123"
    mysqlSlaveHost: "172.27.0.3"
    mysqlSlavePort: "3306"
```

### 八、启动.

```javascript
kubectl create ns nacos
helm install nacos -n nacos -f ./values.yaml .
```

