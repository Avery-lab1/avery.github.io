# **docker**

### docker网址

官方镜像网址：hub.docker.com
镜像站：docker.fxxk.dedyn.io

### 安装wsl

```
wsl --set-default-version 2
wsl --update --web-download
```

### 安装docker

来到get.docker.com，只要第一步和第四步，如下

```
curl -fsSL https://get.docker.com -o install-docker.sh
sudo sh install-docker.sh
```

可选: 如果想自己指定安装目录，可以使用命令行的方式 参数 --installation-dir=D:\Docker可以指定安装位置

```
start /w "" "Docker Desktop Installer.exe" install --installation-dir=E:\program_file\docker
```



#### windows安装

搜索启动或关闭windows功能
开启适用于linux的windows子系统和虚拟机平台

### 启动docker：

```
sudo service docker start
```

### **docker pull**

**所有指令加上sudo**

```python
docker pull docker.io/library/nginx:latest
#docker.io是官方仓库的地址（registry:注册表），如果是要用官方仓库可以省略这个地址。

#library是命名空间：作者名，library是官方仓库的命名空间，可以省略不写

#最后的:后面是标签名（版本号），如果要用最新版可以不写

#https://hub.docker.com/_/ubuntu
#hub.docker.com是registry，而ubuntu是repository
```

pull后面可以加上选项

```python
docker pull --platform=xxxxxx nginx 
#表示拉去某某特殊cpu架构的镜像，一般会自动选合适主机架构，我们不用自己选
```



### 换源（配置镜像站）

https://github.com/tech-shrimp/docker_installer

sudo vim /etc/docker/daemon.json

```json
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1panel.live",
        "https://hub.rat.dev"
    ]
}
```

:wq!保存

重启docker
sudo service docker restart

windows上面：

设置->docker engine->配置文件打个逗号，把上面红字那坨赋值到下面->点击右下角apply and restart，然后在命令行可以使用了

### 常用指令：

```
sudo docker inspect
```

```python
sudo docker images
#列出镜像

sodu docker rmi  镜像名字或者id
#i代表image，这里是删除镜像
```

```python
#创建容器
docker create -p 80:80 镜像名
```

```python
docker logs 容器名/id #查看日志
docker logs 容器名/id -f或者--follow #滚动查看日志
```

#### docker exec

```python
#执行linux命令
docker exec 容器id linux命令
例如
sudo docker exec id ps -ef
#进入正在运行的docker容器内，获得交互式的命令行环境
#容器内容表现处理像一个独立的操作系统
docker exec -it id /bin/sh
```

#### **docker run**

使用镜像创建并运行容器

```python
sudo docker run 镜像名
sudo docker run (--name 自定义名字) 镜像名(或者id)
#自定义的名字在宿主机必须唯一
#这是使用镜像创建容器，如果没有镜像，会自动去拉拉取
sudo docker ps
#查看进程状况，这里是查看运行的容器
#第一列是容器的唯一id
#第二列是基于哪个镜像创建
#最后一个是容器名，不设置的话，系统会自动设置
sudo docker ps -a #查看所有容器，涵盖未启动的

sudo docker run -d
#我们一般使用，-d代表分离模式，容器去后台执行不会阻塞当前窗口，这样只会打印容器id

sudo docker run -p 80:80 镜像名
#-p是端口映射，容器和宿主机端口绑定，前面是主机端口
#容器运行在独立的虚拟环境里，容器网络和宿主机隔离，默认不能直接从宿主机访问docker内部网络
打开浏览器，localhost:80即可
```

#### docker run其他参数

```python
-e #往容器里面传递环境变量
#可以去docker镜像看自己的镜像下面的文档，里面有写可以传入的环境变量，开源项目可以去github
\  #然后换行输入其他参数

eg：
sudo docker run -d \ 
-p 80:80 \
-e MONGO_INITDB_ROOT_USERNAME=tech \
-e MONGO_INITDB_ROOT_PASSWORD=shrimp \
mongo
#这样就把账号名密码作为环境变量传入
在windows电脑启动时：mongosh "mongodb://tech:shrimp@111.231.100.92:80"

#其他参数
-it  #可以让控制台进入容器进行交互
--rm #容器停止时就删除容器
sudo docker run -it --rm alpine

--restart #用来配置容器停止时的重启策略
sudo docker run -d --restart unless-stoopped 镜像名#只有手动停止容器不会重启 
suco docker -d --restart always 镜像名 #容器停止了就立即重启，包含崩溃，断电
```

#### 容器内安装软件

```python
#先进入/bin/sh，再进行下列操作
cat /ext/os-release #查看linux是什么发行版
#ubuntu,kali,debian里面
apt update
apt install vim
```

### 调试容器

```python
docker start 容器id/名字
docker stop  容器id/名字
#端口映射，挂载卷，环境变量都不用重新设置
docker inspect 容器id 
#可以查看之前填写的参数，问ai就行
```

### Dockerfile

镜像是模具，那么Dockerfile是制作模具的图纸，里面列出来镜像是怎么制作的，D要大写

```c++
FROM python:3.13-silm

WORKDIR /app //类似cd，切换目录

COPY .  . //第一个是电脑的当前文件夹，第二个是镜像内的当前工作目录

RUN pip install -r requirments.txt

EXPOSE 8000 //镜像提供的服务端口。不是强制，只是声明，实际以-p参数为准

CMD ["python3","main.py"] //cmd是容器运行时的默认启动命令，一个Dockerfile只能有一个cmd，类似还有ENTRYPOINT，比CMD优先级高
```

有了dockerfile可以docker build -t 镜像名字 (:版本号) .  （.代表在当前文件夹创建）

### 推送镜像

```python
docker login
#给我验证码和网址，我去登陆一下
docker build -t 用户名/docker_test .
#推送
docker push 用户名/docker_test
```



### 挂载卷

#### 绑定挂载

数据的持久化保存
我们删除容器时，容器内所有数据同时会删除，如果挂载卷了，**共同的目录里的数据还会保存**

```python
docker run -v 宿主机目录:容器内目录
```

**绑定挂载时，宿主机目录会覆盖容器目录**

#### 命名卷挂载

docker自动创建存储空间，我们为存储空间起名，挂载时直接使用名字即可

```
docker run -v 卷的名字:容器内目录
```

```
sudo docker volume create 卷名
```

查看挂载卷在宿主机的真实目录

```python
sudo docker volume inspect 卷名
sudo -i #切换root，才能进入该目录
vim index.html #查看index文件
#里面有数据，因为命名卷第一次使用，docker会把容器文件夹同步到命名卷里面，进行初始化，绑定挂载没有这个功能
```

```python
docker volume list #可以列出创建过的卷
docker volume rm 卷名 #可以删除卷

docker volume prune -a #删除所有没有任何容器在使用的卷
```

### 删除镜像

```python
#列出安装过的镜像
sudo docker images
#删除镜像
sudo docker rmi id或名字
```

### 删除容器

```python
#查看id
sudo docker ps
#删除容器
sudo docker rm id
#删除正在运行的容器
sudo docker rm -f id
#注意，删除镜像
sudo docker rmi
```

### 退出虚拟机容器

```python
exit #不确定
```

### docker网络

#### 桥接模式/子网

默认是桥接模式，每个容器分配了一个ip地址，一般是172.17开头，容器可以通过内部ip互相访问，但和宿主机隔离

```python
#可以创建子网
docker network create network1
docker run --network1 #让容器加入子网
#同一子网的容器可以互相访问，而且可以用名字访问
docker run 其他容器的时候
-e ME_CONFIG_MONGODB_SERVER=my_mongodb#这里的mongodb是容器名，docker内的dns机制 会转为ip
```

#### Host模式

```python
#容器共享宿主机的ip，而且不用-p，容器内的服务直接运行于宿主机的端口
#通过宿主机的ip和端口可以直接访问容器
host启动
sudo docker run -d --network host 镜像名
```

#### None模式

不联网

#### 容器内查看ip

```python
#装工具
apt update
apt install iproute2
#操作
ip addr show
#inet后面是内网ip，也可以在云服务器控制面板看
```

```python
docker network list #展示所有docker网络
```

删除子网

```
docker network rm 刚刚list的id
```

### docker compose

一个轻量级容器编排技术
大规模的用kubernetes

**执行compose**：

```python
docker compose up -d 
#执行当前目录下面严格叫做docker-compose.yaml的文件
docker compose -f 非标准的yaml文件名 up -d 
#执行非标准的yaml文件
docker compose stop #只停止容器
docker compose start #启动刚刚停止的容器
docker compose down #停止并删除容器
```

使用yml  /  yaml文件管理多个容器，里面列出了docker是如何创建，以及如何协同工作的
可以理解为一个或多个docker run命令，按特定格式例如yml文件，以下是对应关系

**直接把docker命令喂给ai，他写compose文件**

```python
#docker会为每一个compose文件创建一个子网，同一个compose文件里面的容器都会加入子网
docker compose还可以自定义容器启动顺序
加上：
depends_on:
    - my_mongodb
```

```yml
# --------------------------------------------
# 🐳 Docker CLI 命令方式
# --------------------------------------------

# 创建网络（Compose 会自动创建，不需要这一步）
docker network create network1

# 启动 MongoDB 容器
docker run -d \
  --name my_mongodb \                         # 对应 Compose 的服务名 my_mongodb
  -e MONGO_INITDB_ROOT_USERNAME=name \        # 对应 environment 中的变量
  -e MONGO_INITDB_ROOT_PASSWORD=pass \        # 同上
  -v /my/datadir:/data/db \                   # 对应 Compose 的 volumes
  --network network1 \                         # Compose 默认创建并使用网络
  mongo                                       # 对应 Compose 的 image: mongo

# 启动 Mongo Express 容器
docker run -d \
  --name my_mongodb_express \                 # 对应 Compose 的服务名 my_mongodb_express
  -p 8081:8081 \                              # 对应 Compose 的 ports
  -e ME_CONFIG_MONGODB_SERVER=my_mongodb \    # 使用 Mongo 服务容器名连接
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=name \   # 对应 environment 中的变量
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=pass \   # 同上
  --network network1 \                        # 与 MongoDB 在同一个网络中
  mongo-express                               # 对应 Compose 的 image: mongo-express


# --------------------------------------------
# 📄 docker-compose.yaml 等效配置
# --------------------------------------------

version: '3'
services:
  my_mongodb:                                 # 对应 CLI 中的 --name my_mongodb
    image: mongo                              # 对应 mongo 镜像
    environment:
      MONGO_INITDB_ROOT_USERNAME: name        # 对应 -e 传参
      MONGO_INITDB_ROOT_PASSWORD: pass
    volumes:
      - /my/datadir:/data/db                  # 对应 -v 参数
    # 网络配置省略，Compose 会自动创建网络，并服务间可 DNS 访问

  my_mongodb_express:                         # 对应 CLI 中的 --name my_mongodb_express
    image: mongo-express
    ports:
      - "8081:8081"                           # 对应 -p 参数
    environment:
      ME_CONFIG_MONGODB_SERVER: my_mongodb    # MongoDB 服务名可直接作为主机名
      ME_CONFIG_MONGODB_ADMINUSERNAME: name
      ME_CONFIG_MONGODB_ADMINPASSWORD: pass
    depends_on:
      - my_mongodb                            # 确保 Mongo 启动后再启动 Express

```

```python
一个services对应一个容器，--name对应service名，镜像名对应image后面的名字，-e对应environment，-v对应volumes也就是挂载卷，-p对应ports
```







### docker原理

每个容器相当于一个独立的linux系统

```
利用了linux两大核心
用Cgroups来限制和隔离进程的资源使用
用Namespace隔离进程的资源视窗，只能看到容器内的
```

docker里是及其简单的操作系统，很多工具例如vim是缺少的，自己安装

自己dockerhub的账号名就是命名空间