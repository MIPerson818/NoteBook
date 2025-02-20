# Docker

## docker的网络代理

https://blog.csdn.net/peng2hui1314/article/details/124267333

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

```bash
sudo touch /etc/systemd/system/docker.service.d/http-proxy.conf
```

```bash
vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```



【注】HTTP_PROXY 用于代理访问 http 请求，HTTPS_PROXY 用于代理访问 https 请求，如果想某个 IP或域名不走代理则配置到 NO_PROXY中。

添加完成后，保存即可。刷新更改并重新启动 Docker

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 1、基础用法

```bash
docker pull ubuntu:latest
docker ps
docker ps -a
docker images
docker stop
docker start
docker restart
docker rm <container_name>
docker rmi <image_name>
docker rmi -f <image_name> #强制删除

//改容器名称
docker rename <elder-contaner-name> <new-contaner-name>
// 改镜像名称
docker tag <elder-image-name> <new-image-name>
```

## 2、镜像 -> 容器

```bash
docker run -it \
	--name pytorch \
    --gpus all \
    --ipc=host  (或者--shm-size=2GB) \
    --network host \
    --device=/dev/video0 \
    --device=/dev/video1 \
    --device=/dev/video2 \
    --device=/dev/video3 \
    -e DISPLAY=$DISPLAY \
    -e XAUTHORITY=$XAUTHORITY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v $XAUTHORITY:$XAUTHORITY \
    -v /home/jack/workspace:/home/workspace \
    -v /home/jack/repository:/home/repository \     
     palm:1.4
     
  ( -v /home/user/conan2:/root/.conan2  //挂载conan到本地 )
```

--gpus all  //宿主机下载好NVIDIA Container Toolkit后，容器使用宿主机GPU
--ipc=host  (或者--shm-size=2GB) 运行容器的默认共享内存host是与宿主机共享，后者是指定大小
--device=/dev/video0  //挂载宿主机外设      --network host // 使用宿主机网络

`-e` 选项用于设置环境变量：

	-e DISPLAY=$DISPLAY \
	-e XAUTHORITY=$XAUTHORITY \
	-v /tmp/.X11-unix:/tmp/.X11-unix \  
	-v $XAUTHORITY:$XAUTHORITY \    //图像显示和摄像头显示端口映射到宿主机

以上四个参数的组合使得 Docker 容器可以访问主机的图形用户界面，容器中运行图形应用程序的窗口保证可以显示在主机的桌面上。

-v /home/jack/workspace:/home/workspace   //挂载宿主机文件夹



### 带有GPU的容器初始化

#### 1: 更新系统和安装依赖项

首先，打开终端，并更新你的系统。这将确保你获得最新的软件包和依赖项。

```
sudo apt updatesudo apt upgrade
```

####  2: 安装NVIDIA Container Toolkit

NVIDIA Container Toolkit是一个用于在Docker容器中运行NVIDIA GPU加速应用程序的工具包。首先，你需要添加NVIDIA Container Toolkit的[存储](https://cloud.baidu.com/product/bos.html)库到你的系统中。

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
```

然后，更新你的软件包列表并安装NVIDIA Container Toolkit。

```
sudo apt updatesudo apt install -y nvidia-container-toolkit
```

####  3: 配置Docker以使用NVIDIA GPU

安装完成后，你需要配置Docker以使用NVIDIA GPU。首先，重启Docker服务。

```
sudo systemctl restart docker
```

然后，验证Docker是否配置正确以使用NVIDIA GPU。运行以下命令：

```
docker run --gpus all nvidia/cuda:11.0-base nvidia-smi
```

如果一切顺利，你应该能看到NVIDIA GPU的详细信息。



## 3、容器、镜像的运行与保存

###        容器的启动与运行

```bash
//在一个运行的容器中打开一个新的终端会话
docker run -it --name <container_name> <image_name>  //先生成容器
docker exec -it <container_name> /bin/bash    // 终端打开运行的容器（若并未运行则先start运行）

//启动一个 Docker 容器，在退出容器后自动删除它
docker run -it --rm <镜像名> /bin/bash
```

###          保存和导入容器/镜像

```bash
//将一个容器的文件系统导出为一个 tar 文件
docker export -o my_container.tar my_container

//从一个 tar 文件中导入容器文件系统
docker import my_container.tar

// 将一个镜像保存为一个 tar 文件
docker save -o my_image.tar ubuntu:latest

// 从一个 tar 文件中加载镜像
docker load -i my_image.tar

// 将容器保存为镜像
docker commit <要commit的容器名> <要命名的镜像名:版本号>
```

## 4、Docker登录

https://hub.docker.com/

### 1. 登录 / 退出 Docker Hub

```shell
docker login  # 登录 Docker Hub
docker logout  # 退出 docker hub 
```

### 2. 正确命名镜像（包括用户名）

当推送镜像到 Docker Hub 时，镜像的名称必须包含你的 Docker Hub 用户名。例如，如果你的用户名是 `myusername`，并且你要推送的镜像叫做 `myapp`，你需要重新标记镜像，使其名称包含用户名：

```bash
docker tag app:v1 myusername/app:v1
```

### 3. 推送 / 拉取镜像

```bash
docker push myusername/myapp:v1   # 确保镜像名称和版本正确后，执行推送命令
docker search ubuntu
docker pull ubuntu  # 拉取镜像,将官方 ubuntu 镜像下载到本地
```



## 5、Dockerfile

**Dockerfile 举例：**

```dockerfile
# 使用 Ubuntu 2004作为基础镜像
FROM ubuntu:20.04

# 设置非交互模式
ENV DEBIAN_FRONTEND=noninteractive

# 设置工作目录
WORKDIR /home/workspace

COPY ./sources.list /etc/apt/sources.list

# 更新包列表并安装基础工具
RUN set -eux \
    && apt-get update \
    && apt-get -yq upgrade \
    && apt-get -yq install \
    build-essential \
    wget curl \
    git \
    vim fim\
    sudo \
    lsb-release \
    openssh-server \
    software-properties-common \
    gnupg2 \
    python3-pip \
    libssl-dev \
    ca-certificates \
    ninja-build \
    openssh-server \
    sudo \
    pkg-config \
    valgrind \
    tini \

    clang \
    clangd \
    lldb \
    cmake \
    && rm -rf /var/lib/apt/lists/*

# 安装 Conan 2.x 包管理器
RUN pip3 install conan==2.0.5

# 设置 C++ 编译环境，使用 clang 作为默认编译器
RUN update-alternatives --install /usr/bin/cc cc /usr/bin/clang 100 \
    && update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang++ 100

# 清理临时文件
RUN apt-get clean && rm -rf /var/lib/apt/lists/*


# 配置环境变量
ENV CC=clang
ENV CXX=clang++

# 将容器内的 clangd 与 lldb 设置为 VS Code Remote C++ 插件的调试工具
RUN ln -s /usr/bin/clangd /usr/local/bin/clangd \
    && ln -s /usr/bin/lldb /usr/local/bin/lldb

# 安装完成后展示版本信息
RUN clang --version && lldb --version && cmake --version && conan --version

# 设置 Conan 默认配置
RUN conan profile detect --force

# 清理临时文件
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# 设置 Tini 作为 init 进程
ENTRYPOINT ["/usr/bin/tini", "--"]

# 设置默认命令
CMD ["/bin/bash", "-l"]

# 指定镜像名称
LABEL version="4.9" description="C++ clangd lldb Cmake Conan2 + OpenCV development environment"

```

**source.list 举例：**

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

**基本构建命令：**

```bash
docker build -t <镜像名称>:<标签> <Dockerfile 所在的目录>
EG:
docker build -t my-image:latest .
docker build --network host -t my-image:latest .  #使用宿主进的网络，否则很可能下载不通
```

在构建时，可以使用 `--build-arg` 来传递参数：

```bash
docker build --build-arg APP_ENV=development -t my-dev-image .
```

这样构建出的镜像中会使用 `APP_ENV=development`。

## 6VScode的docker插件报错

在配置完docker之后，尝试使用[vscode](https://zhida.zhihu.com/search?content_id=239404960&content_type=Article&match_order=1&q=vscode&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3MzI0MTc1MDgsInEiOiJ2c2NvZGUiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyMzk0MDQ5NjAsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.LIfhVRt1bVPCrSTMkoXtD3a4XPASqWE41JvdlYHjHJ0&zhida_source=entity)的docker插件连接进容器时，docker插件无法正常显示容器信息，报错Error: permission denied while trying to connect to the Docker daemon socket at unix。这是因为vscode插件权限不足，可以为docker插件设置权限解决。

```bash
sudo chmod 666 /var/run/docker.sock
```



## 7、创建容器修改地

docker run -it \
    --name avp_mapping \
    --gpus all \
    --ipc=host \
    -e DISPLAY=$DISPLAY \
    -e XAUTHORITY=$XAUTHORITY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v $XAUTHORITY:$XAUTHORITY \
    -v /home/nio/Workspace:/home/workspace \
    -v /home/nio/repository:/home/repository \
	937116efad1d

