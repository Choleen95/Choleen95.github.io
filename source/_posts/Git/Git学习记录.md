---
title: Git学习记录v1.0
categories: Git
tags: Git
abbrlink: 4680ac6f
date: 2024-06-16 00:00:00
---
## 1、常用操作

- git clone 
- git config
- git branch
- gitt checkout
- git status
- git add
- git commit
- git push
- git pull
- git log
- git tag
<!-- more -->
### 1.1 git clone

从git服务器拉取代码

```bash
git clone https://gitee.com/xxx/studyJava.git
```

### 1.2 git config

配置开发者用户名和邮箱

```bash
git config user.name xxx
git config user.email xxx@qq.com
```

每次代码提交的时候都会生成一条记录，其中就会包含自己配置的用户名和邮箱

若想查看配置的用户名和邮箱

```bash
git config user.name
git configt user.email
```

### 1.3 git branch

创建、重名名、查看、删除分支

- 新增

```bash
git branch feature-dev
```

- 查看

```bash
git branch
```

- 删除

```bash
git branch -d feature-dev
```

### 1.4 git checkout

切换分支

```bash
git checkout feature-dev
```

### 1.5 git status

查看文件变动状态，有哪些需要add，哪些学院commit

On branch feature-dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/testGit/Demo.java

no changes added to commit (use "git add" and/or "git commit -a")

### 1.6 git add

天骄文件变动到暂存区

```bash
 git add src/testGit/Demo.java
```

添加所有文件

```bash
git add.
```

![git-add](D:\BlogFile\2024\0616\git-add.png)



### 1.7 git commit

提交文件变动到版本库

```bash
git commit -m '提交文件到远程版本库'
```

*-m* 参数可直接在命令行里输入提交描述文本

### 1.8 git push

将本地的代码改动推送到服务器

```bash
git pull origin feature-dev
```



![git-push](D:\BlogFile\2024\0616\git-push.png)

origin 当前的git服务器地址

### 1.9 git pull

将服务器上的最新代码拉取到本地

```bash
git pull origin feature-dev
```

项目成员对项目做了改动并推送到服务器，我们需要将最新的改动更新到本地，这里我们来模拟一下这种情况。

到 gitee上把文件改动一下

![git-pull](D:\BlogFile\2024\0616\git-pull.png)

### 1.10 git log

查看版本提交记录

```bash
git log
```

查看整个项目的版本提交记录，大多数情况下，看的都是自己的记录

按 **J** 键往下翻，按 **K** 键往上翻，按 **Q**键退出查看



#### 1.10.1 git commit -m提交后如何回退

最近提交代码，和任务单号挂钩，有时需要回退。这个需要学习一下。

##### 1.10.1.1 使用 soft

```bash
git reset --soft HEAD~1
```

这会撤销上一次的提交，但保留所有更改在你的工作区。意味着自己的更改仍然被 Git 跟踪，可以再次提交它们，或者修改之后提交。

##### 1.10.1.2 使用 hard

```bash
git reset --hard HEAD~1
```

这会撤销上次的提交，并且丢且所有更改。这意味着你的工作区将会回到上一次提交的状态。

##### 1.10.1.3 使用 具体哈希值

用 git log 命令 找到想回退的指定提交海西值

```ba
git reset --hard ae1057b65dffc3e6586ce3c9ee308f102c0c79ac^
```

**注意** 这里的 **^** 表示前一个提交

使用 **--hard** 会丢失你自上一次提交以来的所有未提交的更改。

#### 1.10.2 覆盖提交信息

当然我们只想覆盖信息，代码不想回退到工作区间。

```bash
git commit --amend -m 'add bbb'
```

这将打开你的默认文本编辑器(或者你使用了*-m*,则直接创建新的提交记录并覆盖)，修改并保存。

不管加不加 -m 提交记录的哈希值都会创建新的。