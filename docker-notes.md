## 1. Docker介绍
- 先理解几个概念
（a）CGROUPS：https://zh.wikipedia.org/wiki/Cgroups <br/>
（b）LXC：https://zh.wikipedia.org/wiki/LXC <br/>
（c）AUFS：https://zh.wikipedia.org/wiki/Aufs <br/>
（d）Hypervisor: https://zh.wikipedia.org/wiki/Hypervisor <br/>

###（1）什么是Docker
创始人与由来： <br/>
dotCloud: Solomon Hykeys(在法国期间)的公司内部项目 <br/>
开始：Ubuntu 12.04上基于Go语言实现，RedHat：RHEL 6.5支持 <br/>
2013/3: Apache授权协议开源，GitHub上维护，加入了Linux基金会（开放容器联盟OCI） <br/>
2013年底：dotCloud改名docker <br/>

原理：
（1）go语言实现 <br/>
（2）基于Linux内核的cgroup, namespace, AUFS类的Union FS等主要技术对进程封装隔离 <br/>
（3）容器：属于操作系统层面的虚拟化技术，隔离的进程独立于宿主和其他的隔离进程。容器内可运行应用程序，不同的容器之间相互隔离，容器之间可相互通信 <br/>

cgroup技术： <br/>
名称源自控制组群（control groups）的简写，是Linux内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。在Linux内核中，提供了cgroups功能，来达成资源的区隔化。 <br/>

namespace名称空间技术： <br/>
名称空间技术使应用程序看到的操作系统环境被区隔成独立区间，包括进程树，网络，用户id，以及挂载的文件系统。 <br/>

AUFS类的UnionFS技术（分层存储技术）： <br/>
aufs（全称：advanced multi-layered unification filesystem，高级多层统一文件系统）用于为Linux文件系统实现联合挂载。由多层文件系统联合组成。 <br/>
该技术使得构建镜像时，可一层层构建，前一层是后一层的基础。当前层删除前一层的文件时，实际上不是真的删除，而是仅在当前层标记该文件已删除。 <br/>
容器运行时，虽然不会看不见此文件，但实际上该文件一直跟随镜像。主要用来实现容器的存储层。 <br/>


初始版本：实现基于LXC（Linux 软件容器，提供了cgroup与namespace等技术功能） <br/>
0.7版本：该版本后去除LXC，转而使用自行开发libcontainer <br/>
1.11版本：进一步演进为使用runc和containerd <br/>



### （2）Docker的优势
- 开发：交付和部署快，迁移和扩展方便
- 运维：资源利用率高，管理简单

为何要用Docker：
（a）充分利用资源： <br/>
——一个应用未必用得完整个机器的资源。 <br/>
——容器无需虚拟化硬件和运行整个操作系统等额外开销，与原生系统几乎一样 <br/>
（b）启动时间更快速： <br/>
——传统虚拟化技术启动需要数分钟，而Docker直接运行于宿主内核，无需启动完整的OS，运行秒级/甚至毫秒级 <br/>
（c）运行环境一致： <br/>
——开发/测试/生产:  Docker镜像可满足各个运行环境一致 <br/>
（d）持续交付和部署CI/CD： <br/>
——开发和运维DevOps只需一次创建/配置，可任意地方正常运行。 <br/>
——通过Dockerfile定制应用镜像来实现持续集成持续部署和交付CI/CD <br/>
（e）更轻松的迁移 <br/>
——运行环境的一致性使得迁移更加简单容易 <br/>
（f）维护和扩展更加轻松 <br/>
——Docker采用分层存储和镜像技术，使得应用重复部分的复用更加容易。 <br/>
——维护更加简单，基于基础镜像可延伸出扩展镜像。 <br/>
——Docker团队与项目团队维护了大量高质量的官方镜像，可直接使用或定制。 <br/>

### (3) 概念理解
#### 镜像（Image）
——操作系统分为内核和用户空间：对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 root 文件系统。
——Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。
——Docker镜像构建之后内容数据不会再改变。

#### 容器（Container）
——镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
——容器的实质是进程，与宿主执行的进程不同，容器进程运行于自己独立的命名空间。容器可拥有自己的root文件系统、网络配置、进程空间、用户ID等。
——容器进程运行在一个隔离的环境，独立于宿主的系统下。
——镜像存储层是以镜像构建时的存储层作为读写，容器消亡时，容器存储层也随之消亡。
——最佳实践：容器不应向其存储层内写入任何数据，容器存储层要保持无状态化。
——文件写入操作：应使用数据卷Volume或绑定宿主目录/网络存储（性能和稳定性更高）。
——数据卷的生存周期独立于容器：容器消亡，数据卷不会消亡，数据不会丢失。

#### 仓库（Repository）
（1）一个集中的存储和分发Docker镜像的服务，叫Docker Registry <br/>
（2）一个Docker Regitstry可包含多个仓库Repository <br/>
（3）每个仓库可包含多个标签，每个标签对应一个镜像。 <br/>
——通常“<仓库名>:<标签>”的格式指定镜像的标签。 <br/>
例如：<仓库名>一般为“用户名/软件名”，如：tommy/nginx-proxy
——如果不给出标签，默认标签是latest。 <br/>

Docker Registry 公开服务

Docker hub：https://hub.docker.com/

CoreOS： Quay.io

Google： https://cloud.google.com/container-registry/

Kubernetes：http://kubernetes.io/


阿里云镜像加速器：https://cr.console.aliyun.com/#/accelerator

DaoCloud加速器：https://www.daocloud.io/mirror#accelerator-doc

网易云镜像服务：https://c.163.com/hub#/m/library/

时速云镜像仓库：https://hub.tenxcloud.com/


Docker Registry私有服务

可在本地构建私有Docker Registry服务。或通过实现开源的Docker Registry API来方便自己。



## 2. 安装Docker
```
# 说明-y顺便安装docker依赖的软件包
yum install -y docker.x86_64

# 检查docker状态（验证docker）
systemctl status docker

# 激活和启动docker服务
systemctl start docker

# 再次检查docker状态
systemctl status docker

# 若已激活Active，则说明可正常使用docker

# 相关操作：
#检查运行的容器实例
docker ps
#列出镜像
docker images
#拉取指定名称的镜像
docker pull <名称>
```

## 3. docker镜像
### （1）拉取和删除镜像
docker pull <镜像名称>
例如：docker pull docker.io/redis

docker rmi <镜像ID>
例如：docker rmi -f 5004609dd30

### （2）查看镜像信息
下面命令是查看本地所有已下载的镜像列表：
docker images
仓库名/镜像标签信息/镜像ID/创建时间/镜像大小

查看某个镜像的详细信息：
docker inspect <镜像ID>

### （3）搜寻镜像(公有库)
docker search <镜像名>
docker search redis

### （4）创建镜像

1）基于已有的镜像容器创建
```
docker commit -a <作者> -m <提交信息> --pause=true
```

例如：
```
docker commit -a "test" -m "new images" c189b5f44060 testimage
```

test是作者
new images是提交信息
c189b5f44060是正在运行的容器id
testimage是镜像的名称


2）可基于本地模板导入进行构建镜像：
```
sudo cat <tar.gz> | docker import - <name>:<tag> https://openvz.org/Download/template/precreated

# 例如：
cat debian-7.0-x86_64.tar.gz | docker import - tommy:debianbase

debian-7.0-x86_64.tar.gz是下载的镜像模板
```

3）使用dockerfile配置


### （5）存出和载入镜像（可快速备份和恢复）
存出镜像：
```
docker save -o **.tar <name>:<tag>
```
(将现有的镜像打成tar包)
```
**.tar代表输出的镜像文件
```

将本地docker镜像文件保存成本地文件

载入镜像：
```
docker load --input / < **.tar
```
将本地的镜像文件载入到docker镜像

### （6）上传镜像(https://hub.docker.com)
```
docker images
docker login
docker tag <ID> <acountname>/<imageName>:<tag>
docker push <accountName>/<imageName>:<tag>
```

### （7）运行镜像
docker run -it <镜像id> /bin/bash

## 4. 容器和仓库
(1)创建容器/启动容器
```
#创建容器
docker create -it <name>:<tag>
#启动已经创建的容器
docker start <ID>
# 新建容器并启动容器
docker run -it <ID> /bin/bash
```
```
# 守护态运行
docker run -d <ID> /bin/sh -c 'while true;do echo hello world;sleep 1;done'
docker ps
# 查看前端打印的日志
docker logs <容器ID>
```

```
#查看已启动的容器实例
docker ps
```

(2)终止容器
```
docker stop <ID>
```

(3)进入容器
```
docker exec -it <ID> /bin/bash
```

(4)删除容器(-f强制删除的意思)
```
docker rm -f <ID>
```

(5)导入和载出容器
（作用：快速备份当前容器和恢复当前容器）
导入容器：
```
cat **.tar | docker import - <name>:<tag>
```
导出容器：
```
docker export <ID> > **.tar
```

(6)搭建私有仓库
```
# 拉取注册服务镜像
docker pull registry

# /opt/registry 本地文件挂载点
# /var/lib/registry 容器挂载点
# 5000:5000本地宿主端口与容器的端口映射
docker run -d -v /opt/registry:/var/lib/registry -p 5000:5000 --restart=always --name registry registry:latest

http://ip:5000/v2/_catalog
```

## 5. 数据卷
(1)数据卷
```
docker run -v <host DIR>:<container DIR> <name> <command>

docker run -d -P --name web -v /webapp:training/web -v /images:training/images python appy
```

(2)数据卷容器
借助数据卷容器来供其他容器共享。
```
docker run -it -v /dbdata --name dbdata ubuntu
```

(3)利用数据卷容器迁移数据
备份：将数据卷容器打包至宿主机
```
docker run --volume-from dbdata -v $(pwd):/backup --name worker tar cvf /backup/backup.tar /dbdata
```

恢复：将宿主机上的数据卷容器备份进行恢复
```
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash

docker run --volume-from dbdata2 -v2/_catalog $(pwd):/backup busybox tar xvf /backup/backup.tar
```

## 6. 网络配置
（1）访问容器
-P 随机端口
```
docker run -d -P training/webapp python app.py
```
-p 指定端口
```
docker run -d -p 5000:5000 --name web training/webapp python app.py
```

（2）映射端口配置查看
```
docker port
```
