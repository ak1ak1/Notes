# 一、Git工作流程

![image-20230531184616314](G:\笔记\git\assets\image-20230531184616314.png)

命令：

- clone（克隆）：从远程仓库中克隆代码到本地仓库
- checkout（检出）：从本地仓库中检出一个仓库分支然后进行修订
- add（添加）：在提交前先将代码提交到暂存区
- commit（提交）：提交到本地仓库。本地仓库中保存修改的各个历史版本
- fetch（抓取）：从远程库，抓取到本地仓库，不进行任何的合并操作，一般操作较少
- pull（拉取）：从远程库拉到本地仓库，自动进行合并（merge），然后放到工作区，相当于fetch+merge
- push（推送）：修改完成后，需要和团队成员共享代码时，将代码推送到远程仓库



# 二、Git安装与常用命令

## 1.下载与安装

安装流程：

- 安装地址：http://git-scm.com/downloads

- 下载完后安装，一路next
- 安装完后，在桌面右键会有两个新图标
  - Git GUI：GIt提供的图形界面工具
  - Git Bash：Git提供的命令行工具
- 安装Git后首先要做的事情就是设置用户名称和email地址。这是非常重要的，因为每次Git提交都会使用该用户信息

## 2.基本配置

- 打开Git Bash

- 设置用户信息

  - git config --global user.name "ak1"
  - git config --global user.email "gwq1540478105@foxmail.com"

- 查看用户信息

  - git config --global user.name
  - git config --global user.email

- 为常用指令设置别名（可选,有些常用指令参数非常多，每次都要输入好多参数，可以使用别名）

  - 打开用户目录，创建.bashrc文件（打开gitBash，执行touch ~/.bashrc）

  - 在.bashrc文件中输入以下内容：

    - ```shell
      # 用于输出git提交日志
      alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
      # 用于输出当前目录所有文件以及基本信息
      alias ll='ls -al'
      ```

  - 打开GitBash，执行``` source ~/.bashrc```

  - 解决GitBash乱码问题

    - 1.打开GitBash执行 ``` git config --global core.quotepath false```

    - 2.${GIT_HOME}/etc/bash.bashrc 文件最后加入下面两行

    - ```
      export LANG="zh_CN.UTF-8"
      export LC_ALL="zh_CN.UTF-8"
      ```



## 3.获取本地仓库

要使用Git对我们的代码进行版本控制，首先要获得本地仓库

- 创建空目录，当作仓库
- 进入空目录，打开GitBash窗口
- 执行命令 ```git init```
- 创建成功后可以看到.git目录。
- 

## 4.基础命令

Git工作目录下对于文件的修改（增加、删除、更新）会存在几个状态，这些修改的状态会随着我们执行Git的命令而发生变化

![image-20230531204618364](G:\笔记\git\assets\image-20230531204618364.png)

本章节主要讲解如何使用命令来控制这些状态之间的转换：

- git add（工作区-->暂存区）
- git commit（暂存区-->本地仓库）

### 4.1查看修改的状态（status）

- 作用：查看修改的状态（暂存区、工作区）
- 命令形式：```git status```

### 4.2添加工作区到暂存区（add）

- 作用：添加工作区一个或多个文件的修改到暂存区
- 命令形式：```git add 单个文件名|通配符```
  - 将所有修改加入暂存区：```git add .```

### 4.3提交暂存区到本地仓库（commit）

- 作用：提交暂存区内容到本地仓库的当前分支
- 命令形式：```git commit -m "注释内容"```
- 

### 4.4查看提交日志（log）

- 作用：查看提交记录
- 命令形式：git log [option]
  - options:
    - --all：显示所有分支
    - --pretty=oneline：将提交信息显示为一行
    - --abbrev-commit：使得输出的commitId更简短
    - --graph：以图的形式显示

### 4.5版本回退

- 作用：版本切换
- 命令形式：git reset --hard commitID
  - commitID可以使用```git-log```或者```git log```指令查看
- 如何查看已经删除的记录：
  - ```git reflog```

### 4.6添加文件至忽略列表

​	一般我们总会有些文件无需纳入Git的管理，也不希望他们总出现在未跟踪文件列表。通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。在这种情况下，我们可以在工作目录中创建一个名为.gitignore的文件（文件名称是固定的），列出需要忽略的文件模式。

实例：

```
# no .a files
*.a
# but do track lib.a, even though you're ignoring .a files above
!lib.a
# only ignore thr TODO file in the current directory, not subdir /TODO
/TODO
# ignore all files in the build/ directory
build/
# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt
# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```



## 5.分支

​		几乎所有的版本控制系统都以某种形式支持分支。使用分支意味着你可以把你的工作从开发主线上分离来进行重要的bug修改、开发新功能，以免影响开发主线。

### 5.1查看本地分支

- 命令：```git branch```

### 5.2创建本地分支

- 命令：```git branch 分支名```

### 5.3切换分支（checkout）

- 命令：```git checkout 分支名```

切换到一个不存在的分支（创建并切换）：

- 命令：```git checkout -b 分支名```

### 5.4合并分支（merge）

一个分支上的提交可以合并到另一个分支

- 命令：```git merge 分支名称```

### 5.5删除分支

不能删除当前分支，只能删除其他分支

- 删除时，需要先做检查：```git branch -d b1```
- 强制删除：```git branch -D b1```

### 5.6解决冲突

当两个分支上对文件的修改可能会存在冲突，例如同时修改了同一个文件的同一行，这时就需要手动解决冲突，解决冲突步骤：

- 处理文件中冲突的地方
- 将解决完冲突的文件加入暂存区（add）
- 提交到仓库（commit）

### 5.7开发中分支使用原则与流程

​		几乎所有的版本控制系统都以某种形式支持分支。使用分支意味着你可以把你的工作从开发主线上分离开来进行重大的bug修改、开发新功能，以免影响开发主线。

在开发中，一般有如下分支使用原则与流程：

- master（生产）分支：
  - 线上分支，主分支，中小规模项目作为线上运行的应用对应的分支；
- develop（开发）分支：
  - 是从master创建的分支，一般作为开发部门的主要开发分支，如果没有其他并行开发不同期上线要求，都可以在此版本进行开发，阶段开发完成后，需要合并到master分支，准备上线。
- feature/xxx分支：
  - 从develop创建的分支，一般是同期并行开发，但不同期上线时创建的分支，分支上的研发任务完成后会合并到develop分支。
- hotfix/xxx分支：
  - 从master派生的分支，一般作为线上bug修复使用，修复完成后需要合并到master、test、develop分支。
- 还有 test分支（代码测试）、pre分支（预上线分支）等等。

![image-20230601114453859](G:\笔记\git\assets\image-20230601114453859.png)

# 三、Git远程仓库

## 1.常用的托管服务（远程仓库）

```
	Git中存在两种类型的仓库，即本地仓库和远程仓库。常见代码托管服务：GitHub、码云、GitLab等
	GitHub（地址：https://github.com/）是一个面向开源及私有软件项目的托管平台,只支持Git作为唯一的版本库格式进行托管。
	码云（地址：https://gitee.com/）是国内的一个代码托管平台，由于服务器在国内，相比GitHub，码云速度快
	GitLab（地址：https://about.gitlab.com/）是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的web服务，一般用于在企业、学校等内部网络搭建git私服。
```

## 2.码云

### 2.1注册码云

略

### 2.2创建远程仓库

略

### 2.3配置SSH公钥

- 生成SSH公钥
  - ssh-keygen -t rsa
  - 不断回车
  - 如果公钥存在，则自动覆盖
- Gitee设置账户公钥
  - 获取公钥
    - cat ~/.ssh/id_ras.pub
  - Gitee设置公钥
    - 路径：头像->设置->安全设置->SSH公钥 
  - 验证是否配置成功
    - ssh -T git@gitee.com

### 2.4操作远程仓库

- 添加远程仓库
  - 此操作是先初始化本地库，然后与已创建的远程库进行对接
  - **命令**：```git remote add <远端名称> <仓库路径>```
  - 远端名称：默认是origin，取决于远端服务器配置
  - 仓库路径：从远端服务器获取此URL
  - eg：```git remote add origin git@gitee.com:ak1ak1/gitee_test.git```
- 查看远程仓库
  - **命令**：```git remote```
- 推送到远程仓库
  - **命令**：```git push [-f] [--set-upstream] [远端名称 [本地分支名][:远端分支名]]```
  - 如果远端分支名和本地分支名相同，则可以只写本地分支名
    - ``` git push origin master```
  - --set-upstream：推送到远端的同时并建立起和远端分支的关联关系
    - ```git push --set-upstream origin master```
  - 如果当前分支已经和远端分支关联，则可以省略分支名和远端名。
    - git push 将master分支推送到已关联的远端分支
  - git branch -vv ：查看本地分支和远端分支的关系
  - -f ：有冲突时强制覆盖（建议少用）。
- 从远程仓库克隆
  - 如果有一个远端仓库，可以直接clone到本地
  - 命令：git clone <仓库路径> [本地目录]
  - 本地目录可以省略，会自动生成目录
- 从远程仓库中抓取和拉取
  - 远程的分支和本地的分支一样，我们可以进行merge操作，只是需要先把远端仓库里的更新都下载到本地，在进行操作。
  - 抓取 **命令**：```git fetch [remote name] [branch name]```
    -  抓取指令就是将仓库里的更新都抓取到本地，不会进行合并
    - 如果不指定远端名称，则抓取所有分支

  - 拉取 **命令**：```git pull [remote name] [branch name]```
    - 拉取指令就是将远端仓库的修改拉到本地并自动进行合并，等同于fetch+merge
    - 如果不指定远端名称和分支名，则抓取所有并更新当前分支

- 解决合并冲突
  - 在一段时间，A和B用户修改了同一个文件，且修改了同一行位置的代码，此时会发生合并冲突。
  - A用户在本地修改代码后优先推送到远程仓库，此时B用户在本地修改代码，提交到本地仓库后，也需要推送到远程仓库，此时B用户晚于A用户，故需要先拉取远端仓库的提交，经过合并后才能推送到远端分支。


# 四、在Idea中使用Git

## 1.在Idea中配置Git















