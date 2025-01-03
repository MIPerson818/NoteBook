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
    --name conda-torch \
    --gpus all \
    --ipc=host \
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
	05d1b981bb5b
     



# Git

- 系统层级: `/etc/gitconfig`，作用于系统中所有用户的 git 配置；
- 用户层级: `$HOME/.gitconfig`，作用于用户的 git 配置；
- 项目层级: `.git/config`，作用于项目中。

## 一、本地git

提交push：工作目录 ——> 暂存区 ——> 本地仓库 ——> 远程仓库：文件必须一步一步的提交
拉取pull：远程仓库 ——> 本地仓库 ——> 暂存区 ——> 工作目录：文件可以依次“检出”，也可以直接从远程仓库“检出”到工作目录

![image-20210821201040414](./typora-user-images/image-20210821201040414.png)

### 1.Git 使用前的配置命令

在使用前告诉 git 你是谁：

1. 第一次使用 git，配置用户信息

   1. 配置用户名：`git config --global user.name "your name"`;
   2. 配置用户邮箱：`git config --global user.email "youremail@github.com"`;

2. > 查询配置信息

   1. 列出当前配置：`git config --list`;
   2. 列出 repository 配置：`git config --local --list`;
   3. 列出全局配置：`git config --global --list`;
   4. 列出系统配置：`git config --system --list`;

3. > 其他配置

   1. 配置解决冲突时使用哪种差异分析工具，比如要使用 vimdiff：`git config --global merge.tool vimdiff`;
   2. 配置 git 命令输出为彩色的：`git config --global color.ui auto`;
   3. 配置 git 使用的文本编辑器：`git config --global core.editor vi`;

4. > 注：

   1. 更改-->重复上述命令
   2. 也可直接修改 `C:\Users\用户\.gitconfig`

### 2. 初始化和克隆仓库

```bash
git init  # 创建新的 Git 仓库
git clone <repository-url>  # 克隆远程仓库
```

### 3. 查看状态和历史

```bash
git status  	   # 查看当前状态
git diff		   # 查看当前工作目录和暂存区之间的差异
git diff -cached   # 查看暂存区和最近一次提交之间的差异
git log			   # 查看提交历史，将显示每个提交的哈希值、作者、日期和提交信息
git log --oneline  # 查看简洁历史（单行显示）
git log --graph -- pretty=oneline # 查看带有冲突解决的日志
```

### 4. Git三板斧——添加、提交、拉取和推送 

```bash
git add <file-name>  # 添加文件到暂存区
git add .    		 # 添加所有更改
```

```bash
git commit -m "日志内容"     # 提交更改
git commit -a -m "日志内容"  # 将所有已经使用 git 管理过的文件暂存后一并提交
git commit --amend 	        # 修改最近一次的提交，可以将新的更改和以前的提交合并为一个新的提交
```

```bash
git pull  # 从远程仓库拉取最新更改，相当于  git fetch origin + git merge origin/next
git push origin <branch-name>  # 将本地更改推送到远程仓库
```

### 5.版本管理 

```bash
git checkout <commit-hash>  # 检出特定的提交
git checkout -- <file-path> # 恢复文件到某个提交的状态
git checkout <commit-hash> -- <file-path>  # 将文件恢复到工作目录的最新状态
```



## 二、分支

### 1.分支细分

1. 主分支（master）：第一次向 git 仓库提交更新记录时自动产生的一个分支。
2. 开发分支（develop）：作为开发的分支，基于 master 分支创建。
3. 功能分支（feature）：作为开发具体功能的分支基于开发分支创建。

### 2.分支操作

```bash
git branch  ( -r 远程 -a所有 本地+远程)     # 查看分支
git branch <branch-name>        # 创建分支
git branch -m <new-branch-name> # 重命名分支
git breach -d <branch-name>     # 删除分支（分支合并后才允许被删除）（-D 大写强制删除）

git checkout <branch-name>    # 切换分支
git checkout -b <branch-name> # 创建+切换分支
git checkout <commit-hash>  # 检出特定的提交
git checkout -- <file-path> # 恢复文件到某个提交的状态
git checkout <commit-hash> -- <file-path>  # 将文件恢复到工作目录的最新状态
```

### 3. 合并和变基

```bash
git merge <branch-name>  // 合并分支
git merge branch1 branch2    # 合并多个分支
git rebase <branch-name> // 变基分支
```

```bash
git add <file-with-conflict>   # 解决冲突：在合并或变基时，可能会出现冲突。Git 会提示你去解决这些冲突，编辑冲突的文件，解决后，使用此命令标记为已解决
git commit  # 完成合并
```

### 4. 查看和管理远程仓库

```bash
git remote -v  # 查看远程仓库信息
git remote add <name> <repository-url>  # 添加远程仓库
git remote remove <name>  # 删除远程仓库
```

### 5. 标签管理

```bash
git tag             # 查看标签
git tag <tag-name>  # 创建标签
git tag -a <tag-name> -m "Tag message"  # 附注标签： 包含标签的元数据（如作者、日期和标签说明）

git push origin <tag-name>  # 推送标签到远程
git push origin --tags      # 要推送所有标签
```

### 6. 其他常用命令

```bash
git reset <file-name>   # 撤销暂存区文件
git rm <name> # 删除仓库中的文件
git .git rm   # 删除本地仓库 
git help <command>  # 查看帮助
```

## 三、远程github

### 1、fetch与push

​	**fetch**： `fetch` 命令用于从远程仓库下载最新的提交和数据到本地，但并不会自动合并这些更改。执行 `git fetch` 后，你可以查看和审查远程仓库的更新，而不会影响你当前的工作版本。这是一个安全的操作，可以有效地让开发者保持与远程仓库的同步，在深入了解远程更新后决定如何处理这些更改，比如是否合并到当前工作分支。

​	**push**： `push` 命令则用于将本地的提交推送到远程仓库。这意味着你可以将自己完成的工作共享到远程 repository，以便其它团队成员可以访问和使用这些更改。当你执行 `git push` 时，本地分支的提交会被上传到远程仓库中相应的分支，这样可以将你的工作整合到团队的协作中。

**基本用法**

1. **获取远程仓库的更新**： 要从名为 `origin` 的远程仓库获取更新，这将下载 `origin` 的所有分支和标签的最新提交，但你的本地分支不会受到影响。

   ```bash
   git fetch origin
   ```

2. **查看所有分支的更新，或使用 `git log` 查看特定远程分支的提交历史**： 

   ```bash
   git branch -r
   git log origin/master --oneline --graph
   ```

3. **合并或变基更新**：

   ```bash
   git merge origin/master   # 获取的更新合并到你的当前分支
   git rebase origin/master  # 变基，改变放到这个版本后面，线性
   ```

### 2、远端与本地有冲突的默认选择模式：

```bash
git config pull.rebase false #合并: Git会通过创建一个新的合并提交来将远程的更改合并到你的本地分支。
git config pull.rebase true  #变基： Git会将你的本地提交“移动”到远程提交之上，这样可以保持更线性的提交历史。
git config pull.ff only      #仅快进：仅在可以快进合并时进行合并，如果不可以快进，会拒绝操作。这通常用于保持提交历史的简洁。
```

### 3、refs是什么

1. **`refs/heads/`**：指向本地分支。例如，`refs/heads/master` 代表名为 `master` 的本地分支。
2. **`refs/remotes/`**：指向远程分支。例如，`refs/remotes/origin/master` 代表 `origin` 远程仓库中的 `master` 分支。
3. **`refs/tags/`**：指向标签，例如，`refs/tags/v1.0` 代表一个标签。

```bash
origin/master` 是指名为 `origin` 的远程仓库中的 `master` 分支。Git 默认将你克隆的远程仓库命名为 `origin
```

### 4、SSH连接

##### 第一步：检查本地主机是否已经存在ssh key， 如果存在，直接跳到第三步

```bash
cd ~/.ssh
ls     #看是否存在 id_rsa(密钥)和 id_rsa.pub(公钥)文件，如果存在，说明已经有SSH Key
```

##### 第二步：生成ssh key

```bash
ssh-keygen -t rsa -C "xxx@xxx.com"    #执行后一直回车即可
```

##### 第三步：获取ssh key公钥内容（id_rsa.pub）

```bash
cd ~/.ssh
vim id_rsa.pub # 复制公钥
```

##### 第四步：Github账号上添加公钥

-  进入Settings设置
- SSH and GPG keys 中，添加ssh key，把刚才复制的内容粘贴上去保存即可

##### 第五步：验证是否设置成功

```bash
ssh -T git@github.com
```

# SSH

https://blog.csdn.net/GitHub_miao/article/details/135050696

### 1. 安装 OpenSSH Server

如果你在目标机器上没有安装 SSH 服务，可以通过以下命令安装 OpenSSH Server：

```
sudo apt update
sudo apt install openssh-server
```

### 2. 启动 SSH 服务

安装完成后，确保 SSH 服务正在运行：

```
sudo systemctl start ssh
sudo systemctl enable ssh  # 使其在启动时自动运行

# 重启SSH服务使配置生效
sudo systemctl restart ssh

# 检查 SSH 服务状态
sudo systemctl status ssh
```

你应该看到服务的状态是 "active (running)"。

### 3. 防火墙设置

确保防火墙允许 SSH 连接。你可以使用以下命令来检查和修改防火墙设置：

```bash
sudo ufw allow ssh  # 允许SSH连接
sudo ufw enable     # 启用防火墙（如果尚未启用）disable（这里工位电脑开启的话下使用conan下载不了github文件，所以就关闭了）
```

### 4. 确认 SSH 端口和配置

SSH 配置文件通常位于 `/etc/ssh/sshd_config`。你可以检查这个文件，确保没有配置错误。

```bash
# 示例代码：编辑sshd_config文件
sudo nano /etc/ssh/sshd_config
```

在配置文件中，可以设置SSH服务监听的端口、允许的用户、禁止root登录等。

```bash
# 示例代码：更改SSH服务端口
Port 2222

# 示例代码：禁止root用户直接登录
PermitRootLogin no
```

默认情况下，SSH 使用端口 22。你可以确认端口配置，确保没有其他服务占用这个端口。

如果你对 SSH 配置文件做了更改，记得重启 SSH 服务使其生效：

```bash
sudo systemctl restart ssh
```



### 5.配置SSH密钥对，登录服务端+配置客户端

```bash
# 示例代码：生成SSH密钥对
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_key
```

#### 5.1将公钥复制到目标服务器：

```bash
# 示例代码：复制公钥到目标服务器
ssh-copy-id user@remote_server
```

手动复制：

```bash
# 示例代码：手动复制公钥到目标服务器
cat ~/.ssh/my_key.pub | ssh user@remote_server 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

#### 5.2 配置SSH客户端

```bash
# 示例代码：编辑SSH客户端配置文件
nano ~/.ssh/config
```

```
# 示例代码：为远程主机配置别名
Host my_server
    HostName remote_server
    User user
    Port 2222
    IdentityFile ~/.ssh/my_key
```

#### 5.3使用SSH传输文件

```bash
# 示例代码：使用SCP传输文件到远程主机
scp /path/to/local/file user@remote_server:/path/to/remote/directory
```

### 6.使用SSH隧道

```bash
# 示例代码：建立SSH隧道
ssh -L 8080:localhost:80 user@remote_server
```

#### 6.1使用ProxyJump配置跳板主机

```bash
# 示例代码：配置ProxyJump跳板主机
Host final_server
    HostName final_server
    User user
    Port 2222
    IdentityFile ~/.ssh/my_key

Host jump_server
    HostName jump_server
    User user
    Port 2222
    IdentityFile ~/.ssh/my_key

Host remote_server
    HostName remote_server
    User user
    Port 2222
    IdentityFile ~/.ssh/my_key
    ProxyJump jump_server
```

#### 6.2使用SSH配置文件提高安全性

```bash
# 示例代码：设置SSH配置文件权限
chmod 600 ~/.ssh/config
```

#### 6.3 SSH的相关配置

/etc/ssh/sshd_config中：

```bash
# 在配置文件中添加Banner，以显示登录前的提示信息
Banner /etc/ssh/banner_message

# 确保密码登录被禁用，只允许公钥认证
PasswordAuthentication no
```

#### 6.4限制SSH登录时间和IP范围

```bash
# 示例代码：限制SSH登录时间
sudo nano /etc/security/time.conf

# 添加：
sshd;*;user;Al0800-1700
```

```bash
# 示例代码：限制SSH登录IP范围
sudo nano /etc/hosts.allow

# 仅允许特定IP范围访问SSH
# 添加：
sshd : 192.168.1.0/24
```

等等。。。太多了，链接中的大部分还没用过。

### 7. 再次尝试连接

完成上述步骤后，再次尝试从你的计算机使用 SSH 连接到目标机器：

```
ssh user@XJTU_MIP  # 使用目标主机的用户名
ssh -i ~/.ssh/id_rsa jack.leey@172.23.53.162  # -i后是密匙
```



#### **3. 使用 SSH 密钥登录（推荐）**

如果目标服务器已配置公钥认证，可以用私钥连接：

#### **步骤 1：生成密钥对（如果没有）**

在本地生成 SSH 密钥对：

```
ssh-keygen -t rsa -b 4096
```

- 默认会在 `~/.ssh/id_rsa` 生成私钥，`~/.ssh/id_rsa.pub` 生成公钥。

#### **步骤 2：将公钥复制到目标服务器**

用以下命令将公钥上传到服务器：

```
ssh-copy-id <username>@172.23.53.162
```

- 如果没有 `ssh-copy-id` 工具，可以手动将公钥内容追加到远程服务器的 `~/.ssh/authorized_keys` 文件中。

#### **步骤 3：使用私钥登录**

```
ssh -i ~/.ssh/id_rsa <username>@172.23.53.162
ssh -i ~/.ssh/id_rsa -X jack.leey@172.23.53.162
（-X 使用 X11 转发，用于显示图像）
```

------

### 使用 SCP 命令（安全复制）

1. **从本地复制文件到远程服务器**，本地输入：

   ```
   scp /home/user/local.txt jack.leey@172.22.205.164:~/Repository
   ```

2. **从远程服务器复制文件到本地**, 本地输入：

   ```
   scp test@192.168.1.100:/home/test/remote.txt /home/user/
   ```



### 8. 如果连接失败，可能的原因和解决方法

#### **1. 服务器未运行 SSH 服务**

解决方法：

- 确保目标服务器的 sshd 服务已启动：

  ```
  sudo systemctl start sshd
  ```

#### **2. 服务器防火墙拦截**

解决方法：

- 检查服务器的防火墙规则，确保开放了 22端口：

  ```
  sudo ufw allow 22
  ```

#### **3. 网络不通**

解决方法：

- 检查本地和服务器的网络连通性：

  ```
  ping 172.23.53.162
  ```

#### **4. 用户或密钥配置问题**

解决方法：

- 检查用户名是否正确。
- 检查 `~/.ssh/authorized_keys` 文件是否包含正确的公钥。





### 9. VSCode上SSH连接

1、下载拓展Remote - SSH

2、点击远程资源管理器选项卡，选择远程（隧道/SSH）类别。

![image-20241008121802387](./typora-user-images/image-20241008121802387.png)

3、点击SSH右边的齿轮案件，在中间上部分弹出的配置文件中点击第一个....config。

![image-20241008121917066](./typora-user-images/image-20241008121917066.png)

4、在点进的config文件中输入以下内容。

![image-20241008122030708](./typora-user-images/image-20241008122030708.png)

- Host是服务器主机的用户名


- hostname是服务器的ip地址；（ifcofig）


- port端口号有就写上，没有的话可以不写；


- user是服务器上用户的用户名


（例如：Linux中 “用户名”+@+“服务器ip地址” 就是访问服务器上用户的服务器用户访问地址。）

  点击保存后点击刷新按钮，这时候就可以看到刚刚创建的配置了。

5、Ctrl + Shift + P，打开命令窗口，输入ssh connect to host，选择第一个，选择刚刚创建好的那个配置。或者直接左侧栏点击配置好的jack，点击新窗口打开。询问是否保存known_hosts，选择Continue，输入服务器上用户的密码。

6、该用户第一次访问该服务器可以看到该提示信息，耐心等待，这时是插件在服务器上面安装需要的依赖，大约会占用服务器150mb左右的空间。如果长时间都一直是该情况，可以使用Ctrl + Shift + P，打开命令窗口，输入reload window来重新加载窗口（会要求你重新手动输入密码）。

### 总结

如果在安装或启动过程中遇到其他错误，请查看系统日志，可能会提供更具体的信息：

```
journalctl -xe  # 查看系统日志以获取详细信息
```
