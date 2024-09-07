## 理论部分
1. **Docker 的基本概念**
- 什么是 Docker？它解决了哪些问题？
docker是一种容器技术，实现服务最小化部署；解决了服务的一键部署，能够极大的保障安全问题。

2. **Docker 架构**
- 描述 Docker 的基本架构，包括 Docker 客户端、Docker 守护进程和 Docker 镜像。
不清楚

3. **镜像与容器**
- 解释 Docker 镜像和容器的区别。
docker镜像和容器相当于linux发行版镜像和安装完linux后的操作系统一样

4. **Docker Registry**
- 什么是 Docker Registry？Docker Hub 是什么？
docker hub是docker公共的镜像仓库，用户可以将自己创建的镜像放在docker hub上。

5. **Docker 存储**
- Docker 中的卷（Volumes）和绑定挂载（Bind Mounts）有什么区别？
volumes是docker自有的存储空间，数据会存放在指定的目录中；bind mounts能够实现和宿主机数据同步

6. **Docker 网络**
- Docker 中的桥接网络（Bridge Network）和主机网络（Host Network）有什么区别？
bridge network能够实现

7. **Dockerfile**
- Dockerfile 中的 FROM、RUN 和 CMD 指令分别有什么作用？
FROM为指定使用的docker images。RUN为在构建镜像的时候需要执行的命令。CMD为最终生成执行的命令，以确保镜像有前台运行的进程

8. **层（Layers）**
- 解释 Docker 镜像中的层（Layers）是如何工作的。
最上层为可写层，只有改层能够修改数据。往下都是只读层。当需要删除只读层的数据的时候，只是屏蔽用户访问，并非真正删除。多层之间利用联合文件系统

9. **Docker Compose**
- Docker Compose 的作用是什么？它的基本结构是怎样的？
不了解

10. **端口映射**
- Docker 如何实现端口映射？举例说明。
在运行docker时候使用-p参数指定端口映射，或者使用-P随机分配宿主机未使用端口来映射容器端口。-p 8080:80

11. **Docker 安全性**
- 提出几个提高 Docker 安全性的最佳实践。
最小化权限分配

12. **容器编排**
- 什么是容器编排？举例说明一种常用的编排工具。
不了解

13. **数据持久化**
- 如何在 Docker 容器中实现数据持久化？
使用docker的卷存储方式。

14. **Docker 网络模式**
- 简述 Docker 的不同网络模式及其应用场景。
birdge网络，给容器分配一个独立的网段，利用nat转发到宿主机实现外网访问；host主机模式通过直接利用宿主机网络进行外网访问，效率高但是不安全

15. **容器生命周期**
- 解释 Docker 容器的生命周期管理。


## 实践部分

1. **安装 Docker**
- 描述如何在你的操作系统上安装 Docker。
配置dockeryum源，通过yum镜像进行安装

1. **创建容器**
- 使用 Docker 命令创建一个运行 Nginx 的容器，并使其在浏览器中可访问。
```perl
docker pull nginx
docker run -d -p 80:80 nginx
```

1. **构建镜像**
- 编写一个简单的 Dockerfile，并使用它构建一个自定义镜像。
```dockerfile
FROM centos:latest
RUN cat /etc/system-relase
CMD ["/bin/bash","-D","FOREGROUND"]
```

1. **管理容器**
- 使用 Docker 命令启动、停止、重启和删除一个容器。
```bash
docker start [id]
docker stop [id]
docker restart [id]
docker rm [id]
```

1. **数据卷**
- 创建一个数据卷，并将其挂载到一个容器中。
```bash
docker volume create --driver local data_test
docker run -d -p 80:80 -v data_test:/data nginx
```

21. **端口映射**
- 创建一个容器，并将其内部的端口 80 映射到主机的端口 8080。
```bash
docker run -d -p 8080:80 nginx
```

22. **Docker Compose 文件**
- 编写一个 Docker Compose 文件，定义一个包含多个服务的应用程序。
```yaml

```

23. **容器日志**
- 查看并分析一个正在运行的容器的日志。


24. **网络管理**
- 创建一个自定义的 Docker 网络，并将多个容器连接到该网络。
```bash
docker network create --dribge bridge network_test
# 在创建容器的时候添加参数：--network network_test
```

25. **镜像管理**
- 拉取一个镜像、列出本地镜像、删除不需要的镜像。
```bash
docker pull [images name]
docker images
docker rmi [images id]
```

26. **容器资源限制**
- 创建一个容器，并限制其使用的 CPU 和内存资源。
```perl
# 限制docker cpu资源

# 限制docker memory资源


```

27. **容器互联**
- 启动两个容器，并让它们通过自定义网络进行通信。
```bash
docker network create --dribge bridge network_test
docker run -d -p 8080:80 --name nginx_docker nginx
docker run -it centos bash 
ping nginx_docker
```

28. **健康检查**
- 在 Dockerfile 中添加健康检查指令，并验证其效果。
```perl
docker health check
```

29. **备份与恢复**
- 演示如何备份和恢复 Docker 容器中的数据。
```perl
docker export [id] .tar
docker import [.tar] .
```

30. **集成 CI/CD**
- 在一个 CI/CD 工具（如 Jenkins、GitLab CI）中集成 Docker，以实现自动化构建和部署。


