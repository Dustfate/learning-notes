Docker

### 一、Docker介绍





### 二、Docker的基本操作

#### 2.1安装Docker



#### 2.2Docker的中央仓库

需要修改配置文件 /etc/docker/daemon.json （改文件需要新创建）添加内容如下：

```json
{ 
	"registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"] 
}
```

```sh
# 重启docker
systemctl restart docker
```



#### 2.3镜像的操作

```sh
#1.拉取镜像到本地
docker pull 镜像名称[:tag]
# 例如
docker pull daocloud.io/library/tomcat:8.5.15-jre8
```

----

```sh
#2.查看全部本地镜像
docker images
```

---

```sh
#3.删除本地镜像
docker rmi 镜像标识
```

----

```sh
#4.镜像的导入导出(不规范)
# 将本地的镜像导出
docker save -o 导出的路径/文件名 镜像id
# 加载本地的镜像文件
docker load -i 镜像文件
# 修改镜像名称
docker tag 镜像id 新镜像名称:版本
```

#### 2.4容器的操作

```sh
#1.运行容器
# 简单操作
docker run 镜像标识|镜像名称:[tag]
# 常用的参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像标识|镜像名称[:tag]
#-d：代表后台运行容器
#-p 宿主机端口：容器端口:为了映射当前Linux的端口和容器的端口
#--name 容器名称：指定容器的名称
# docker run -d -p 8080:8080 --name tomcat b8
```

------

```sh
#2.查看正在运行的容器
docker ps [-qa]
#-a：查看全部的容器，包括没有运行的
#-q：只查看容器的标识
```

------

```sh
#3.查看容器的日志
docker logs -f 容器id
```

-------------------

```sh
#4.进入到容器内部
docker exec -it 容器id bash
# 退出
exit
```

------

```sh
#5.删除容器（删除容器前需要先停止容器）
# 停止指定容器
docker stop 容器id
# 停止全部容器
docker stop $(docker ps -qa)
# 删除指定容器
docker rm 容器id
# 删除全部容器
docker rm $(docker ps -qa)
```

---

```sh
#6.启动容器
docker start 容器id
```

### 三、Docker应用

#### 3.1准备项目

#### 3.2准备MySQL

```sh
# 运行MySQL容器
docker run -d -p 3306:3306 -- name mysql -e MYSQL_ROOT_PASSWORD=root daocloud.io/library/mysql:5.7.4
```

#### 3.3准备Tomcat

```sh
# 运行Tomcat容器，前面已经搞定，只需要将项目的war包部署到Tomcat容器内部即可
# 可以通过命令将宿主机的内容复制到容器内部
docker cp 文件名 容器id:容器内部路径
# 例如
docker cp xxx.war 8c8bf6842d77:/usr/local/tomcat/webapps/
```

#### 3.4数据卷

为了部署xxx.war工程，需要使用cp的命令将宿主机内的xxx.war文件复制到容器内部。

数据卷：将宿主机的一个目录映射到容器的一个目录中。

可以在宿主机中操作目录中的内容，那么容器内的映射文件也会跟着一起改变。

```sh
#1.创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认会存放在一个目录下/var/lib/docker/volumes/数据卷名称/_data
```

---

```sh
#2.查看数据卷的详细信息
docker volume inspect 数据卷名称
```

---

```sh
#3.查看全部数据卷信息
docker volume ls
```

---

```sh
#4.删除数据卷
docker volume rm 数据卷名称
```

---

```sh
#5.应用数据卷
# 当需要映射数据卷时，如果数据卷不存在，Docker会自动创建，会将容器内部自带的文件存储在默认的存放路径中
docker run -v 数据卷名称:容器内部路径 镜像id
# 直接指定一个路径作为数据卷的存放位置，这个路径下是空的，需要手动添加
docker run -v 路径:容器内部零件 镜像id
# 例如
# 方式1
docker volume create volume_tomcat
docker run -d -p 8080:8080 --name tomcat -v volume_tomcat:/usr/local/tomcat/webapps 8c8bf6842d77

# 方式2（推荐）
docker run -d -p 8080:8080 --name tomcat -v /opt/volume_tomcat:/usr/local/tomcat/webapps 8c8bf6842d77
```

### 四、自定义镜像

中央仓库上的镜像也是Docker的用户自己上传上去的。

```sh
#1.创建一个Dockerfile文件，并且指定自定义镜像信息。
#Dockerfile文件中常用的内容
  from：指定当前自定义镜像依赖的环境
  copy：将相对路径下的内容复制到自定义镜像中
  workdir：声明镜像的默认工作目录
  cmd：需要执行的命令（在workdir下执行的cmd可以写多个，但只以最后一个为准）
# 例如，自定义一个Tomcat镜像，并且将xxx.war部署到Tomcat中
from daocloud.io/library/tomcat:8.5.15-jre8
copy xxx.war /usr/local/tomcat/webapps
```

----

```sh
#2.将准备好的Dockerfile和相应的文件复制到Linux系统中，通过Docker的命令制作镜像
docker build -t 镜像名称:[tag]
```

### 五、Dokcer-Compose

```
之前运行一个镜像需要添加大量的参数。
可以通过Docker-Compose编写这些参数。
Docker-Compose可以帮助我们批量管理容器。
只需要通过一个docker-compose.yml文件维护即可
```

#### 5.1下载Docker-Compose

```sh
#1.去github官网搜索docker-compose，下载1.24.1版本的Docker-Compose
下载路径：https://github.com/docker/compose/releases/download/1.24.1/docker-compose-Linux-x86_64
#2.将下载好的文件复制到Linux系统中
#3.需要将DockerCompose文件的名称修改一下，给予DockerCompose文件一个可执行的权限
mv docker-compose-Linux-x86_64 docker-compose
chmod 777 docker-compose
#4.方便后期操作，配置一个环境变量
# 将docker-compose文件移动到了/usr/local/bin，修改了/etc/profile文件，给/usr/local/bin配置到了PATH中
mv docker-compose /usr/local/bin
vi /etc/profile
# 添加内容：export PATH=$JAVA_HOME:/usr/local/bin:$PATH
source /etc/profile
#5.测试
# 在任意目录下输入docker-compose
```

#### 5.2Docker-Compose管理MySQL和Tomcat容器

编写docker-compose.yml文件

```yml
version: '3.1'                      # 版本号
services:                           # 代表可以管理多个容器
  mysql:                            # 服务的名称
    restart: always                 # 代表只要docker启动，那么这个容器就跟着一起启动
    image: daocloud.io/library/mysql:5.7.4  # 指定镜像路径
    container_name: mysql                   # 指定容器名称,等同--name
    ports:
      - 3306:3306                           # 指定端口号的映射
    environment:
      MYSQL_ROOT_PASSWORD: root             # 指定MySQL的ROOT用户登录密码
      TZ: Asia/Shanghai                     # 指定时区
    volumes:
      - /opt/docker_mysql_tomcat/mysql_data:/var/lib/mysql   # 映射数据卷
  tomcat:
    restart: always
    image: daocloud.io/library/tomcat:8.5.15-jre8
    container_name: tomcat
    ports:
      - 8080:8080
    environment:
      TZ: Asia/Shanghai
    volumes:
      - /opt/docker_mysql_tomcat/tomcat_webapps:/usr/local/tomcat/webapps
      - /opt/docker_mysql_tomcat/tomcat_logs:/usr/local/tomcat/logs
```

#### 5.3.使用docker-compose命令管理容器

在使用docker-compose的命令时，默认会在当前目录下找docker-compose.yml文件

```sh
#1.基于docker-compose.yml启动管理容器
docker-compose up -d
#2.关闭并删除容器
docker-compose down
#3.开启或关闭已经存在的由docker-compose维护的容器
docker-compose start|stop|restart
#4.查看有docker-compose管理的容器
docker-compose ps
#5.查看日志
docker-compose logs -f
```



#### 5.4.docker-compose配合Dockerfile使用

使用docker-compose.yml文件以及Dockerfile文件在生成自定义镜像的同时启动当前镜像，并且由docker-compose去管理容器

[docker-compose.yml]()文件

```yml
# yml文件
version: '3.1'
services:
  ssm:					       # 服务的名称
    restart: always
    build:                     # 构建自定义镜像
      context: ../             # 指定dockerfile文件的所在路径
      dockerfile: Dockerfile   # 指定Dockerfile文件名称
    image: ssm:1.0.1
    container_name: ssm
    ports:
      - 8081:8080
    environment:
      TZ: Asia/Shanghai
```

[Dockerfile]()文件

```sh
from daocloud.io/library/tomcat:8.5.15-jre8
copy ssm.war /usr/local/tomcat/webapps
```

```sh
#可以直接基于docker-compose.yml以及Dockerfile文件构建的自定义镜像
docker-compose up -d
#如果自定义镜像不存在，会帮助我们构建出自定义镜像，如果自定义镜像已经存在，会直接运行这个自定义镜像
#重新构建自定义镜像
docker-compose build
#运行当前内容，并重新构建
docker-compose up -d --build
```

### 六、Docker CI、CD

#### 6.1引言

```
项目部署
  1.将项目通过maven进行编译打包
  2.将文件上传到指定的服务器中
  3.将war包放到tomcat的目录中
  4.通过Dockerfile将Tomcat和war包转成一个镜像，由DockerCompose去运行容器
项目更新后，需要将上述流程再次的从头到尾的执行一次，如果每次更新一次都执行一次上述操作，很费时，费力。我们就可以通过CI、CD帮助我们实现持续集成，持续交付和部署
```

#### 6.2CI介绍

```
CI（continuous intergration）持续集成
持续集成：编写代码时，完成了一个功能后，立即提交代码到Git仓库中，将项目重新的构建并且测试。
  1.快速发现错误。
  2.防止代码偏离主分支。
```

#### 6.3搭建Gitlab服务器

```sh
实现CI，需要使用到Gitlab远程仓库，先通过Docker搭建Gitlab
1、创建一个全新的虚拟机，并且至少指定4G的运行内存。
2、安装Docker以及Docker-Compose
3、通过docker-compose.yml文件安装gitlab
修改ssh的22端口
#将ssh的默认22端口，修改为60022端口，因为Gitlab需要占用22端口
vi /etc/ssh/sshd_config
# PORT 22 -> 60022
```

```yml
version: '3.1'
services:
 gitlab:
  image: 'twang2218/gitlab-ce-zh:11.1.4'
  container_name: "gitlab"
  restart: always
  privileged: true
  hostname: 'gitlab'
  environment:
   TZ: 'Asia/Shanghai'
   GITLAB_OMNIBUS_CONFIG: |
    external_url 'http://192.168.199.110'
    gitlab_rails['time_zone'] = 'Asia/Shanghai'
    gitlab_rails['smtp_enable'] = true
    gitlab_rails['gitlab_shell_ssh_port'] = 22
  ports:
   - '80:80'
   - '443:443'
   - '22:22'
  volumes:
   - /opt/docker_gitlab/config:/etc/gitlab
   - /opt/docker_gitlab/data:/var/opt/gitlab
   - /opt/docker_gitlab/logs:/var/log/gitlab
```

#### 6.4搭建GitlabRunner

https://www.lixian.fun/3812.html#title-8