# Git 的使用

### Git 的工作区域和文件状态

#### 工作区域

- 工作区 Working Directory	

  实际操作的目录

- 暂存区 Staging Area/Index

  中间区域，用于临时存放即将提交的修改内容

- 本地仓库 Local Repository

  Git 存储代码和版本信息的主要位置

修改代码（工作区） —— git add —— 添加到暂存区 —— git commit —— 提交到本地仓库

#### 文件状态

- 未跟踪

  新建的文件，还没有添加到 git 管理起来

- 未修改

  被 git 管理中的文件，但是还没有修改

- 已修改

  已经修改过了的文件

- 已暂存

  修改过后已经 git add 到暂存区的文件

### Git 常用的命令

#### 基础命令

`git init` 	初始化 git 文件夹

`git statua` 	查看当前文件夹文件的状态

`git add`	将改动添加的暂存区

`git rm`	将添加到暂存区的文件移出暂存区

`git commit -m "message"`	将添加到暂存区的文件提交到本地仓库，并且添加提交信息 message

`git reset`	将提交到本地仓库的文件移出仓库

`git log`	查看提交记录（提交ID、提交作者邮箱、提交时间、提交信息 ）

#### git reset

是否保留			工作区	暂存区

`git reset --soft`	 √		 √

`git reset --hard`	 ×		 ×

`git reset --mixed`       √		 ×

先使用 `git log` 查看提交记录，再选择想要回退的版本号 ID，如：

```shell
b270efb	(HEAD -> main)	commit3
5af90b8	commit2
7100ee3	commit1
```

这个时候我们想要回退版本到 commit2 这次提交：

```bash
git reset --soft 5af90b8
```

再次使用 `git log` ：

```bash
5af90b8	(HEAD -> main)	commit2
7100ee3	commit1
```

使用 `ls` 查看工作目录，发现工作区、暂存区的内容都还在，因为使用的是 soft 参数。

我们也可以直接使用 `HEAD^` 来表示上一个版本：

```bash
git reset --hard HEAD^
```

使用 hard 参数回退，工作区和暂存区的内容，跟提交有关的，都会被清空。

也就是说，使用 `--soft` 方式回退，是回退到 `git commit` 之前的状态；

使用 `--mixed` 方式回退，是回退到 `git add` 之前的状态；

使用 `--hard` 方式回退，是回退到修改文件之前的状态。

谨慎使用 `--hard` 参数，因为他会删除暂存区和工作区中的文件改动内容！

如果不小心误删了改动，我们可以使用 `git reflog` 命令，来查看所有改动记录，包括 `reset` 的记录，然后找到 `reset` 之前的版本号，再次用 `git reset --hard [log_id]` 恢复到回退前。

#### git diff 