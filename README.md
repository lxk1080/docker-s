## docker-s

### 1、Linux 上安装 docker
> tips-1：在 Window 和 Mac 上直接安装 Desktop 版即可（注意在 Window 上需要做一些前置配置，进入 BIOS 将虚拟化技术 VT-x/AMD-V 设置为开启）
> <br/> tips-2：docker 是一个基于 Linux 的技术，所以它在 Window 和 Mac 上的 docker engine，实际上是通过创建一个 Linux 虚拟机的方式运行的

1. 下载安装脚本：`curl -fsSL get.docker.com -o get-docker.sh`
2. 执行安装：`sh get-docker.sh`
3. 启动 docker 服务：`sudo systemctl start docker`
4. 测试下：`docker version`

### 2、docker 常用命令

1. `docker version`：查看 docker 版本信息
2. `docker info`：查看docker 环境的一些基本信息
3. `docker image pull nginx`：拉取镜像（默认 registry 是 docker hub）
3. `docker image ls`：查看环境下所有的镜像
4. `docker container ls [-a]`：查看环境下所有的容器
    - `-a`：包括已经停止运行的
    - 也可以使用 `docker container ps`，或直接省略 container 关键字：`docker ps`
    - 对于 container 的大部分操作，都可以把 container 关键字省略掉
5. `docker container run nginx`：创建并启动容器（如果本地没有，则会尝试线上拉取）
6. `docker container stop <name or ID>`：停止容器运行
7. `docker container rm <name or ID> [-f]`：删除容器
    - `-f`：强制删除一个正在运行中的容器

总之，可以通过 `docker [xx] --help` 查看所有命令

其它技巧：

1. 停止所有容器：`docker container stop $(docker ps -aq)`
    - 其中，`docker ps -aq` 可以列出所有容器的 ID
    - 删除所有容器同理
2. `docker system prune -a -f` 可以快速对系统进行清理，删除停止的容器，不用的 image 等等

### 3、容器运行的模式

1. attach 模式 
    - 例如：`docker container run -p 80:80 nginx`
    - 透过这种方式创建容器的话，容器在前台执行
    - 容器的输入输出结果会反映到本地端，本地端的输入输出也会反映到容器，例如能在终端机看到网页浏览器的 log，ctrl + c 会让容器停止执行
    - 一般情况不推荐使用
2. detach 模式（推荐）
    - 例如：`docker container run -d -p 80:80 nginx`，加上 `--detach` 参数，缩写 `-d`
    - 容器会在后台执行
    - 此时可以使用 `docker attach <ID>` 转化成前台模式，转化后，如果使用 ctrl + c，也会结束掉 container 的运行，所以这种转化也不推荐使用
    - 我们可以通过 `docker container logs [-f] <ID>` 的方式查看输出的 log 日志
      - `-f`：对输出日志信息保持监听状态

### 4、连接容器的 shell

1. `docker container run -it <镜像> sh`：创建一个容器并进入交互式模式（shell）
    - 这种方式如果输入 `exit`，会退出 shell 模式，同时容器也会停止运行
2. `docker container exec -it <ID> sh`：在一个已经运行的容器里执行一个额外的 command（这个命令的使用后续将会非常频繁）
    - 例如：`docker container exec -it <nginxID> sh`，将会进入到容器内部的 shell 环境
    - 这种方式如果输入 `exit`，会退出 shell 模式，但容器不会停止运行，只是退出这个 shell 环境
    - `-it`：进入交互模式

### 5、docker 相关原理

1. 容器不是 Mini 虚拟机
    - 容器其实是进程
    - 容器中的进程被限制了对 CPU 内存等资源的访问
    - 当进程停止后，容器就退出了


2. `docker container run` 背后做了哪些事情？
    - 例如：`docker container run -d --publish 8080:80 --name webhost nginx`
    - 在本地查找是否有 nginx 这个 image 镜像，有就直接创建容器，如果没有发现
    - 去远程的 image registry 查找 nginx 镜像（默认的 registry 是 Docker Hub)
    - 下载最新版本的 nginx 镜像（默认 nginx:latest)
    - 基于 nginx 镜像来创建一个新的容器，并且准备运行
    - docker engine 分配给这个容器一个虚拟 IP 地址
    - 在宿主机上打开 8080 端口并把容器的 80 端口转发到宿主机上（可以通过 127.0.0.1:8080 访问）
    - 这里给容器指定了一个名字 `webhost`，如果不指定，则会自动生成一个随机的名字
    - 启动容器，运行指定的命令（这里默认会去执行一个 shell 脚本去启动 nginx，当然，你也可以自己指定一个命令去执行）
