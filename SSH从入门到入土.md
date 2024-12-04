

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

![image-20241008121802387](/home/nio/文档/NoteBook/typora-user-images/image-20241008121802387.png)

3、点击SSH右边的齿轮案件，在中间上部分弹出的配置文件中点击第一个....config。

![image-20241008121917066](/home/nio/文档/NoteBook/typora-user-images/image-20241008121917066.png)

4、在点进的config文件中输入以下内容。

![image-20241008122030708](/home/nio/文档/NoteBook/typora-user-images/image-20241008122030708.png)

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

