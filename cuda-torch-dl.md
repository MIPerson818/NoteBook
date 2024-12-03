# linux

#### linux相关工具：

```
vim //编辑文档
fim //查看图片
grdit //图像化界面编辑文档
boot-repair //修复引导区
```

#### 动态库静态库：

```
linux下：
.a 静态库  "archive" 文件
.so 动态库 "shared object" 
win下：
.lib 静态库 “Library file”
.dll 动态库 “Dynamic Link Library”
```

### 环境变量

#### 1. 系统级环境变量配置文件（对所有用户生效）

- **`/etc/profile`**：
  这是系统范围内最重要的环境变量配置文件之一，在用户登录时会被执行（不管是通过命令行登录还是图形界面登录等方式），用于设置一些全局的系统环境变量以及执行一些需要在用户登录时就初始化的命令等。它会为所有用户配置基本的环境变量，比如设置 `PATH` 环境变量包含一些系统级的可执行文件目录路径（像 `/usr/bin` 、 `/usr/sbin` 等），定义一些和系统运行相关的通用变量等。
- **`/etc/bash.bashrc`（针对 bash 环境，不同系统可能文件名稍有差异，如 Ubuntu 中是这个名字，CentOS 中叫 `/etc/bashrc` ）**：
  主要用于为 bash shell 配置一些通用的、系统层面的环境变量和别名等内容，会被每个 bash shell 会话所读取（不管是交互式的 bash 终端还是通过脚本等方式启动的 bash 环境）。通常在这里设置一些和 bash 操作特性相关的变量，比如设置一些命令行提示符的格式等，并且可以补充一些额外的常用可执行文件目录到 `PATH` 环境变量中（在 `/etc/profile` 基础上进一步扩展）。

**优先级关系及特点**：
一般来说，`/etc/profile` 先于 `/etc/bash.bashrc` 执行（在 bash 环境下登录系统时）。`/etc/profile` 侧重于系统整体的、最基础的环境变量配置，`/etc/bash.bashrc` 则更多地是针对 bash shell 环境进行一些细化和补充配置。不过，它们都是对所有用户生效的，修改后通常需要重新登录或者通过 `source` 命令（如 `source /etc/profile` 、 `source /etc/bash.bashrc` ）来使新的配置在当前会话中生效（重新登录会让新配置自然生效，使用 `source` 命令则是针对当前已经打开的终端会话立即生效）。

#### 2. 用户级环境变量配置文件（仅对特定用户生效）

- **`~/.bash_profile`**：
  这是针对每个用户的个人环境变量配置文件，用于设置该用户登录时特定的环境变量等内容。当用户登录时，这个文件会被读取执行（如果存在的话），并且可以用来覆盖或者补充系统级环境变量配置文件中相关变量的设置，例如用户可以在这里将自己常用的一些开发工具目录添加到 `PATH` 环境变量中，使得自己登录后能方便地使用这些工具，而不影响其他用户的环境变量设置。
- **`~/.bashrc`**：
  类似于 `/etc/bash.bashrc` ，不过它是针对单个用户的 bash shell 环境的配置文件，用于配置用户个人在 bash 操作过程中常用的环境变量、别名等。每次打开一个新的 bash 终端时，这个文件都会被读取执行（除非通过特殊设置禁止了它的读取，不过这种情况很少见），所以它是用户自定义 bash 相关环境变量的常用位置，比如用户可以在这里添加自己编写的脚本所在目录到 `PATH` 环境变量中，方便直接在终端中执行脚本等操作。

**优先级关系及特点**：
当用户登录时，`~/.bash_profile` 会先被执行，如果该文件中没有特别设置，它通常会调用 `~/.bashrc` 文件（通过类似 `if [ -f ~/.bashrc ]; then. ~/.bashrc; fi` 这样的语句来实现自动调用），所以 `~/.bashrc` 可以看作是对 `~/.bash_profile` 在 bash 日常操作相关环境变量配置方面的补充。`~/.bash_profile` 侧重于登录时的初始环境变量配置，`~/.bashrc` 更关注在日常 bash 终端使用过程中的环境变量和别名等设置。用户对这两个文件进行修改后，一般可以通过 `source` 命令（如 `source ~/.bash_profile` 、 `source ~/.bashrc` ）来使新配置立即在当前终端生效，不需要重新登录系统。

#### 3. 会话级临时环境变量设置（仅在当前会话中生效）

可以直接在终端中使用 `export` 命令来设置环境变量，例如：

```bash
export MY_VARIABLE=value
```

这种方式设置的环境变量只在当前打开的终端会话中有效，关闭终端后就会消失。





```bash
/etc/profile：此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行，并从/etc/profile.d目录的配置文件中搜集shell的设置。
/etc/bashrc：为每一个运行bash shell的用户执行此文件，当bash shell被打开时，该文件被读取。
~/.bash_profile：每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次!默认情况下，他设置一些环境变量，执行用户的.bashrc文件。是交互式login 方式进入 bash 运行的。
~/.bashrc：该文件包含专用于你的bash shell的bash信息，当登录时以及每次打开新的shell时，该文件被读取。交互式 non-login 方式进入 bash 运行的。

https://www.cnblogs.com/renyz/p/11351934.html
（Linux进阶之环境变量文件/etc/profile、/etc/bashrc、/etc/environment）
```

深度学习部署相关：

```bash
# https://zhuanlan.zhihu.com/p/91334380 讲解
nvcc --version  # cuda版本
nvidia-smi  # 显示显卡信息
watch -n 1 nvidia-smi  # 定时刷新这个信息
```

```
YoLo的Darknet框架：
.cfg 和 .weights 文件，一个是描述模型结构的配置文件（通常是 .cfg 文件），另一个是包含权重的二进制文件（通常是 .weights 文件）。

PyTorch 框架：
.pt 文件：PyTorch 模型文件，包含了 PyTorch 框架中的模型结构和权重。

TensorFlow 框架：
.pb 文件：TensorFlow 的 Protocol Buffer 文件，包含了 TensorFlow 框架中的模型结构和权重。

MXNet 模型文件结构：
.json 文件：存储模型的结构，包括网络的定义。
.params 文件：存储模型的权重和参数。

Caffe 模型文件结构：
.prototxt 文件：存储模型的结构，包括网络的定义。
.caffemodel 文件：存储模型的权重和参数。

跨框架的模型交换格式，支持多个深度学习框架，如 PyTorch、TensorFlow、Caffe2 等。
可以通过相应的导出函数进行模型转换：

.onnx 文件：Open Neural Network Exchange 格式，用于跨框架的模型交换，可以从 PyTorch 或 TensorFlow 导出，然后通过 TensorRT 进行优化。

TensorRT ：
.plan 文件：通常由 ONNX 模型转换而来，通过 TensorRT 的 API 进行优化和生成。
```

# 

# CUDA

### 切换cuda版本

#### **1. 注册 `nvcc` 的替代项**

为不同版本的 `nvcc` 注册替代项。运行以下命令，越大优先级越高：

```
sudo update-alternatives --install /usr/bin/nvcc nvcc /usr/local/cuda-11.1/bin/nvcc 111
sudo update-alternatives --install /usr/bin/nvcc nvcc /usr/local/cuda-12.4/bin/nvcc 124
```

#### **2. 注册 CUDA 的符号链接**

如果想通过 `/usr/local/cuda` 来切换 CUDA 版本，还需要为 CUDA 的路径注册替代项：

```
sudo update-alternatives --install /usr/local/cuda cuda /usr/local/cuda-11.1 111
sudo update-alternatives --install /usr/local/cuda cuda /usr/local/cuda-12.4 124
```

#### **3. 选择 `nvcc` 和 `cuda` 的版本**

配置替代项，让系统选择默认的 `nvcc` 和 `cuda`：

```
sudo update-alternatives --config nvcc
sudo update-alternatives --config cuda
```

| **命令**        | **影响范围**              | **目标文件/路径** |
| --------------- | ------------------------- | ----------------- |
| `--config nvcc` | 仅影响 CUDA 编译器 `nvcc` | `/usr/bin/nvcc`   |
| `--config cuda` | 影响整个 CUDA 工具链环境  | `/usr/local/cuda` |

你会看到一个列表，类似于：

```
There are 2 choices for the alternative nvcc (providing /usr/bin/nvcc).

  Selection    Path                       Priority   Status
------------------------------------------------------------
* 1            /usr/local/cuda-11.1/bin/nvcc   111       auto mode
  2            /usr/local/cuda-12.4/bin/nvcc   124       manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```

输入对应的编号选择版本。

类似地，配置 `/usr/local/cuda` 的版本。

#### **4. 验证配置**

确保 `nvcc` 和 `/usr/local/cuda` 都正确指向预期的版本：

```
nvcc --version
ls -l /usr/local/cuda
```

------

#### **5. 注意事项**

1. **环境变量的更新** 编辑 `~/.bashrc` 或 `~/.zshrc`，确保 `PATH` 和 `LD_LIBRARY_PATH` 设置为使用 `/usr/local/cuda` 符号链接：

   ```
   export PATH=/usr/local/cuda/bin:$PATH
   export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
   ```

   然后运行：

   ```
   source ~/.bashrc
   ```

2. **权限问题** 确保 `/usr/local/cuda-*` 路径和 `/etc/alternatives` 目录的权限正确，普通用户可以访问。

3. **动态链接库问题** 如果 CUDA 相关库无法加载，检查 `/etc/ld.so.conf.d/` 中是否有指向 CUDA 的配置文件（如 `/etc/ld.so.conf.d/cuda.conf`），如果没有，可以添加一行：

   ```
   /usr/local/cuda/lib64
   ```

   然后运行：

   ```
   sudo ldconfig
   ```

4.    **update-alternatives同样可以用于切换 gcc、g++等工具版本**











# 报错历史

#### 报错34：

```
   11 | #include <Python.h>
      |          ^~~~~~~~~~
compilation terminated.

# 解决：
sudo apt-get install python3.8-dev
```



