[toc]





### 1、docker安装

```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
yum install -y yum-utils


# 阿里云yum源
yum-config-manager \
    --add-repo \
  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  
# 将软件包信息提前在本地缓存一份，用来提高搜索安装软件的速度
yum makecache fast
yum install docker-ce docker-ce-cli containerd.io
systemctl start docker

docker -v
docker images
# 开机启动
systemctl enable docker

#修改docker源为阿里云
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://aeb49y3h.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker

```







### 2、mysql

拉取镜像

```powershell
docker pull mysql:5.7
```

运行容器并挂在相关目录

```shell
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7
```

配置config文件

```shell
vim /mydata/mysql/conf/my.conf

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collection-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

进入容器并连接mysql

```shell
docker exec -it mysql /bin/bash
mysql -u root -p
```

开机自启动

```shell
docker update mysql --restart=always
```



### 3、redis

拉去镜像

```shell
docker pull redis
```

运行容器并挂在相关目录

```shell
mkdir -p /mydata/redis/conf
touch /mydata/redis/conf/redis.conf

docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```

设置持久化

```shell
vim /mydata/redis/conf/redis.conf

# 配置持久化
appendonly yes
```

开机自启动

```shell
docker update mysql --restart=always
```

进入容器并连接redis客户端

```shelll
docker exec -it redis redis-cli
```



### 4、tomcat

拉取镜像

```powershell
docker pull tomcat
```

运行容器并挂在目录

```shell
docker run -p 8080:8080 --name tomcat --privileged=true -v /etc/localtime:/etc/localtime:ro -v /mydata/tomcat/webapps:/usr/local/tomcat/webapps -v /mydata/tomcat/conf:/usr/local/tomcat/conf -v /mydata/tomcat/logs:/usr/local/tomcat/logs -d tomcat 
```

此时启动容器会失败，因为进行挂在conf目录的时候不会生成相关的文件？？

解决方法：

1、先创建一个基础容器

```shell
docker run -p 8080:8080 --name tomcat1 -d tomcat 
```

2、把基础容器的相关文件复制到目标容器的挂在目录中

```shell\
docker cp tomcat1:/usr/local/tomcat/conf /mydata/tomcat/
docker cp tomcat1:/usr/local/tomcat/webapps /mydata/tomcat/
```

3、基础容器删除(删不删都无所谓)

```
docker rm 容器id
```

4、重启目标容器

```shell
docker restart tomcat
```



