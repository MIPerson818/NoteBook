# Git

| 基础 1 | git clone  | git status        | git add         | git commit | git log    | git push  | git diff |
| ------ | ---------- | ----------------- | --------------- | ---------- | ---------- | --------- | -------- |
| 基础 2 | git branch | git checkout      | git pull        | git merge  | git rebase | git stash | git tag  |
| 进阶 1 | git fetch  | git filter-branch | git cherry-pick |            |            |           |          |

## Gitlab Repo 管理

- 开发人员提交代码到新的分支，验证功能无误后 rebase master 后提交 merge request
- 定期合并自己的 branch 到 master，暂定一个月一次
- 如何确定哪些 code 需要 merge：对别人有帮助的 code
- 每个 repo 的 maintainer 需要定期 release，写 changelog
- 每个 merge，需要制定组内的同学进行 peer review

- 系统层级: `/etc/gitconfig`，作用于系统中所有用户的 git 配置；
- 用户层级: `$HOME/.gitconfig`，作用于用户的 git 配置；
- 项目层级: `.git/config`，作用于项目中。

## 一、本地git

提交push：工作目录 ——> 暂存区 ——> 本地仓库 ——> 远程仓库：文件必须一步一步的提交
拉取pull：远程仓库 ——> 本地仓库 ——> 暂存区 ——> 工作目录：文件可以依次“检出”，也可以直接从远程仓库“检出”到工作目录

![image-20210821201040414](/home/nio/文档/NoteBook/typora-user-images/image-20210821201040414.png)

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
# -b 选项用于创建一个新的本地分支。
#--track 选项将新创建的本地分支设置为跟踪远程的 origin/feature/yu.fang/env_rendering 分支。执行此命令后，你将切换到新创建的本地分支 feature/yu.fang/env_rendering，并且该分支会自动与远程分支关联。
git checkout -b feature/yu.fang/vln_anno --track origin/feature/yu.fang/vln_anno
# -t 是 --track 的简写，此命令会自动创建并切换到本地分支，同时设置跟踪关系。
git checkout -t origin/feature/yu.fang/vln_anno
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



## 三、远程gitTODO







## 四、高级gitTODO

如何从detached HEAD状态切换回常规分支？：

**如果使用`git stash`暂存修改**

**如果使用`git reset --hard`放弃修改**

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
-  SSH and GPG keys 中，添加ssh key，把刚才复制的内容粘贴上去保存即可

##### 第五步：验证是否设置成功

```bash
ssh -T git@github.com
```

### 5. github

补知识点：【[Git高级操作：refs和reflog](https://zhuanlan.zhihu.com/p/521722781)】



