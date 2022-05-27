# git

[TOC]

## 一 初步

### 1. 配置

git 自带 `git config` 来控制git的外观和行为配置，分为三个级别，system, global, local。

1. `/etc/gitconfig` 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果在执行 `git config` 时带上 `--system` 选项，那么它就会读写该文件中的配置变量。 （由于它是系统配置文件，因此你需要管理员或超级用户权限来修改它。）
2. `~/.gitconfig` 或 `~/.config/git/config` 文件：只针对当前用户。 你可以传递 `--global` 选项让 Git 读写此文件，这会对你系统上 **所有** 的仓库生效。
3. 当前使用仓库的 Git 目录中的 `config` 文件（即 `.git/config`）：针对该仓库。 你可以传递 `--local` 选项让 Git 强制读写此文件，虽然默认情况下用的就是它。。 （当然，你需要进入某个 Git 仓库中才能让该选项生效。）

查看三个级别的配置信息：

```sh
git config --list			# 查看所有级别配置信息
git config --list --system
git config --list --global	# 查看当前用户的配置信息
git config --list --local	# 查看当前仓库的配置信息
git config user.name		# 查看特定一条信息
```

第一次使用git需要配置用户信息：

```sh
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

对于中国用户，需要拉取github代码，因此需要设置代理：

```sh
# 设置 socks5 代理
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
# 删除 socks5 代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 2. 基础使用

从 Github 上拉取代码：

```sh
git clone https://github.com/golang/go.git
git clone https://github.com/golang/go.git mygo # 为拉取的仓库设置别名
```

为对应的项目添加 git 版本控制：

```sh
cd project
git init	# 初始化仓库
git add -A	# 将所有文件添加到仓库
git commit -m "init"	# 提交这次修改，提交信息为 init
```

Git 文件状态：

![image-20220523123842910](git_notes.assets/image-20220523123842910.png)

未添加的文件为 Untracked，添加文件 add 后文件状态为 Staged。使用 add 命令可以将 untracked 和 modified 状态的文件转为 Staged 状态。

使用以下命令查看仓库中文件状态：

```sh
git status
git status -s 	# 简略的方式查看状态
git diff	# 查看文件 modified 状态修改的部分
git add -A 	# 添加所有文件
```

如果后悔添加一个文件怎么办？

```sh
git rm --cache 1.txt	# 从 Staged 到 untracked
git reset HEAD .\1.txt	# 从 Staged 到 Modified
git restore --staged .\1.txt	# 从 Staged 到 Modified
```

如果后悔修改一个文件怎么办？(从 modified 到 unmodified)

```sh
git restore 1.txt	# 将文件内容还原到最近一次提交的状态（推荐）
git checkout -- 1.txt	# 将文件内容还原到最近一次提交的状态
```

提交修改：

```sh
git commit -m "msg"
git commit -a -m "mag"	# 用于无 untracked 的情况
git commit --amend		# 覆盖上一次提交(谨慎使用，最好上一次是本地commit时使用)
```

忽略文件 gitignore：

```sh
*.a		# 忽略所有 .a 文件
!lib.a	# 虽然忽略所有 .a 文件，但是 lib.a 除外
/TODO	# 忽略当前目录下 TODO 文件
build/	# 忽略任何目录下的build文件夹
data/*	# 忽略当前目录下data目录下所有文件夹
*/result/*	# 忽略任何目录下 result 目录下所有的文件
doc/*.txt	# 忽略 doc/note.txt 但不忽略 doc/a/note.txt
doc/**/*.pdf	# 忽略所有doc目录下所有 pdf 文件
```

github提供了所有语言忽略文件模板：https://github.com/github/gitignore

移除和移动文件：

```sh
git rm a.txt	# 不仅从暂存区删除，也删除文件
git rm --cache a.txt	# 仅从暂存区删除
git mv 1.txt 2.txt	# 改名
git mv 1.txt a/	# 移动
```

查看仓库修改日志：

```sh
git log
git log --oneline
git log -p -2 # 查看最近两次提交并且查看差异
git log --oneline --graph --all	# 查看合并和分支
```

格式化日志输出：

```sh
$ git log --pretty=format:"%h - %an(%ae), %ar : %s"
1028bda - 2Mw(aaaa@gmail.com), 41 minutes ago : Update 1.txt
09f4627 - 2Mw(aaaa@gmail.com), 46 minutes ago : init
```

### 3. 远程仓库

查看远程仓库：

```sh
git remote	# 查看所有远程仓库名称
git remote -v 	# 仓库远程仓库信息
git remote show <remote>	# 查看具体某个远程仓库具体信息
```

添加和移除远程仓库：

> 如果使用`git clone` 的话，git 会默认添加创建一个 origin 的远程仓库

```sh
git remote add <name> <url>
git remote remove <name>
git remote set-url --add --push origin <url>	# 为某个origin设置不同的fetch和push的url
```

修改远程仓库名称：

```sh
git remote rename <old> <new>
```

拉取与推送仓库：

```sh
git fetch origin	# 只从新推送的所有代码，但是不会自动合并和修改commit(不推荐)
git pull origin				# 拉取代码并且会自动合并分支(推荐)
git push <name> <branch>	# 将对应分支的代码推送到远程仓库中 
```

### 4. 添加标签

查看标签：

```sh
git tag
git tag -l `v1.*`	# 查看以 v1. 开头的版本号
git show v1.0
```

添加和删除标签：

```sh
git tag -a v1.0 -m "First version"
git tag -a v1.2 9fceb02 -m "aaa"	# 为过去某个版本添加标签
git tag -d v1.0						# 删除某个标签
```

共享标签：

git push 命令不会讲标签上传，必须显示推送标签到服务器

```sh
git push origin v1.0 	# 只将 1.0 tag推送
git push origin --tags	# 推送所有的标签
git push origin --delete <tagname>	# 删除远程仓库上的标签
```

## 二. Git 分支

### 1. 分支基础

```sh
git branch test		# 创建名为test的分支
git checkout test
git checkout -b test	# 创建名为test的分支并且切换
git branch -d <name>	# 删除指定name 的分支
```

查看分支：

```sh
git branch				# 查看所有的分支
git branch -v			# 查看各个分支情况
git log --oneline --graph --all # 查看分支状况
```



合并分支：

```sh
git merge <name>	# 将对应的分支合并到当前分支
```

合并分支分为两种方式：Fast-Forward 和 recursive 。前者两个分支是互为父子关系，后者为兄弟关系。兄弟关系的合并可能需要解决分歧，如果遇到分歧需要手动进行解决：

```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> test:index.html
```

`=======` 上半部分是当前分支的，下半部分是另一个分支的，将`<<<<<<<` 和 `>>>>>>>` 之间内容手动解决后，将对应的标志删除后进行 add 和 commit 后就合并成功了。

### 2. 变基 rebase

## 三. Git 高阶

### 1. Git 仓库清理

`git fsck` 命令用来检查内部数据库的问题或者不一致性。

`git gc` 命令在你的仓库中执行 “garbage collection”，删除数据库中不需要的文件和将其他文件打包成一种更有效的格式。

清理仓库的方法：

```sh
git reflog expire --expire=now --all
git gc --prune=now
```

### 2. 设置别名

对于输入长度较多命令，git支持使用别名来进行快速开发：

```sh
git config --global alias.graph 'log --oneline --graph --all'
```

相当于：

```sh
git log --oneline --graph --all
```

