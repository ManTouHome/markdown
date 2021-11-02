# Git学习笔记:

**常用linux命令:**

```快捷键
crtl + a 光标最前
crtl + e光标最后
pwd 当前目录结构
cat 查看文件内容
echo "world" >  hello.txt     //往hello.txt中写
```

**vi命令**

```git
:set number #显示行号
:1,5d   #删除1到5行
```

## 1.设置邮箱和用户名

### 1.给整个计算机一次性设置（基本不用）

```git
git config --global user.name '用户名'
git config --global user.email '邮箱'
```

### 2.给当前用户一次性设置

```git
git config --system user.name '用户名'
git config --system user.email '邮箱'
```

### 3.给当前项目一次性设置

```git
git config --local user.name '用户名'
git config --local user.email '邮箱'
```

### 4.重置用户名和邮箱：

```git
git config --system --unset user.name '用户名'
git config --system --unset user.email '邮箱'
```

## 2.git 初始化

```git
git init
```

## 3.查看提交状态

```git
git status
```

## 4.暂存区操作:

### 1.工作区--->暂存区

```git
git add 文件名
```

### 2.暂存区--->工作区

​	第一种:

```git
git restore --staged 文件名
```

​	第二种:

```git
git rm --cache 文件名
```

​	第三种:

```git
git reset head 文件名
```

## 5.对象区操作

### 1.暂存区--->对象区

```git
git commit -m '提交说明'
```

### 2.对象区--->工作区  （直接vi修改）

### 3.工作区--->对象区（即撤销修改）

​	第一种:

```git
git restore 文件名
```

​	第二种:

```git
git checkout -- 文件名
```

## 6.查看提交日志

### 1.查看所有日志

```git
git log
```

### 2.查看指定条数的日志

```git
git log -n
```

### 3.日志在一行显示

```git
git log --pretty=oneline
```

### 4.自定义日志格式:

```git
git log --pretty=format:"%h - %an , %ar : %s"
```

### 5.图形化显示日志

```git
git log --graph
```

### 6.简洁显示日志

```git
git log --graph --pretty=oneline --abbrev-commit
```

## 7.删除操作

### 1.删除文件:

```git
git rm 文件名
git commit -m '彻底删除'
```

### 2.撤销工作区的删除命令

​	第一种方式:

```git
git restore 文件名
```

​	第二种方式:

```git
git checkout -- 文件名
```

### 3.撤销暂存区的删除命令

​	第一种方式:

```git
git restore --staged 文件名
```

​	第二种方式:

```git
git reset head 文件名
```

### 4.撤销已经提交的删除命令

```git
git reset --hard <commit_id>  # 回到其中你想要的某个版
git reset --hard HEAD^  # 回到最新的一次提交
git reset HEAD^  # 此时代码保留，回到 git add 之前
```

## 8.重写提交说明

1.重写最近一次提交说明

```git
 git commit --amend -m '新的提交说明'
```

## 9.忽略文件ignore

1.新建.ignore文件

```linux
touch .ignore
```

2.编辑.ignore文件

```linux
vi .ignore  #编辑.ignore文件
dir/ #忽略dir目录下的所有文件
dir/*.txt #忽略dir目录下的所有txt文件
!c.properties #排除c.properties文件
```

## 10.分支操作

**快照**:每次提交后的所有文件。

**分支**:一个commit链，一条工作记录线。

**分支名**:指向当前提交commit

**HEAD**:指向当前分支(HEAD--->分支名)

### 1.查看分支

```git
git branch
```

### 2.创建分支

```git
git branch 分支名称
git checkout -b 分支名称    #创建并切换分支
```

### 3.切换分支

```git
git checkout 分支名称
```

### 4.删除分支

```git
git branch -d 分支名称   #不能删除当前分支和未合并的分支
```

### 5.合并分支

```git
git merge 分支名称  #默认fast forward
git merge --no-ff 分支名称   #禁止fast forward
```

fast foward:分支指针的移动

​	1.两个合并的分支归于一点

​	2.丢失分支信息

--no-ff:

​	1.不归于一点，主动合并的分支会前进一步。

​	2.不丢失分支信息

注意:不同分支之间未提交的操作是可以看到的，例如在master分支新建a.txt在dev分支是可以看到的，如果master新建并提交了，切换到dev分支是看不到的。

### 6.查看分支最近一次提交的sha1值

```git
git branch -v
```

### 7.解决冲突 

master修改a.txt并提交

dev分支修改a.txt并提交

master试图合并dev,产生冲突

```git
git merge dev
```

![](..\img\冲突.png)

1.解决冲突:

```git
vi a.txt   #打开冲突的文件，并进行修改
```

![](..\img\冲突文件.png)

修改后:

![](..\img\文件修改.png)

告知git冲突解决并再次提交:

```git
git add a.txt
git commit -m '冲突解决'
```

在master分支上会产生两次提交:

![](..\img\master冲突提交.png)

注意:只有在同一时间轴上的两个分支操作才会产生冲突，两个分支指向的commit不一致时可以直接合并。

### 8.分支重命名

```git
git  branch -m master master2
```

## 11.版本穿梭

### 1.合并add和commit

```git
git commit -am '提交说明'   #不能用于第一次commit
```

###  2.回退到某一次commit

```git
git reset --hard HEAD^^  #有几个^符号就说明回退多少次

git reset --hard HEAD~n  #回退到前n次

git reset --hard sha1值   #跳转到任意一次commit，需要结合git reflog

git reflog  #查看记录，记录所有操作，实现后悔操作，但是要有良好的注释习惯。
```

## 12.stash保存现场

### 1.建议与规定

1.建议(规范): 在功能未开发完毕前，不要commit。

2.规定(必须)：在commit之前，不能chekout切换分支（两个分支不在同一个commit阶段）。

还有将一个功能开发完毕就要切换分支，先保存现场再切换。

### 2.保存现场

```git
git stash
```

### 3.还原现场

```git
git stash pop #默认还原最后一次，将原来保存的内容删除,用于还原内容
```

```git
git stash apply #还原内容，不删除原来保存的内容。
```

```git
git stash apply stash@{1} #指定还原某一次现场。
```

### 4.删除现场

```git
git stash drop stash@{0}
```

### 5.查看现场

```git
git stash list
```

## 13.Tag标签

Tag标签适用于整个项目，和具体分支每关系

### 1.设置标签

```git
git tag 标签名 
git tag -a 标签名 -m "标签说明"   #更加具体
```

### 2.查看标签

```git
git tag
```

### 3.删除标签

```git
git tag -d 标签名
```

## 14.blame责任

```git
git blame a.txt  #查看a.txt的所有提交sha1值，以及每一行的作者。
```

## 15.差异性diff

### 1.linux系统的diff

```linux
diff a.txt b.txt  #直接比较两个文件
```

### 2.Git中的diff

1.暂存区和工作区比较

```git
git diff
```

2.对象区和工作区比较

```git
git diff sha1值
git diff HEAD  #最新对象区和工作区的差异
```

3.对象区和暂存区比较

```git
git diff --cached sha1值
git diff --cached HEAD  #最新对象区和暂存区的差异
```

## 16.GitHub

连接远程仓库要先关联仓库，再关联分支。

### 1.生成公钥/私钥

```git
ssh-keygen -t rsa -C  邮箱

ssh -T git@github.com   #检测连通性

git remote add origin ssh值  #对项目进行关联
```

注意:公钥可以放到github的项目setting中，也可以放在账户的setting中。

### 2.提交到远程

```git
git push -u origin master   #第一次提交需要写全

git push   #以后可以不用带参数
```

### 3.查看远程仓库

```git
git remote show #查看有哪些远程仓库

git remote show origin  #查看origin远程仓库的信息
```

### 4.感知远程仓库

通过origin/master分支进行感知

```git
git branch -a #显示所有分支

git branch -av #显示所有分支以及指向的Commit
```

### 5.拉取远程项目

第一次拉取：

```git
git clone ssh值  #不用初始化，直接克隆即可
```

之后每次拉取:

```git
git pull       #pull = fetch + merge
```

### 6.与远程仓库冲突

进行push操作的时候显示冲突

1.先进行pull操作

```git
git pull
```

2.再对冲突的文件进行编辑

```linux
vi 冲突的文件名
```

3.修改完成后通知git冲突解决

```git
git add 修改后的文件名
```

4.提交到本地版本库

```git
git commit -m '修改后的说明'
```

5.再把本地版本库提交到远程仓库

```git
git push
```

### 7.查看远程仓库日志

```git
git log refs/remotes/origin/master
```

### 8.本地与远程分支关联操作

1.本地到远程

```git
#方法一
git push -u origin 分支名
#方法二
git push --set-upstream origin 分支名
#方法三
git push origin 本地分支名:远程分支名
```

2.远程到本地

```git
git pull  #将远程分支拉取到追踪分支
#第一种方式
git checkout -b 分支名称 origin/分支名称  #将基于追踪分支新建本地分支
#第二种方式
git checkout --track origin/分支名
```

一步完成

```git
git pull origin 远程分支名:本地分支名    #相当于 git pull + git checkout --track origin/分支名
```

通过底层配置文件完成拉取分支:

```git
git fetch origin master:refs/remotes/origin/分支名称
```

3.删除远端分支

```git
#第一种方式
git push origin  :远程分支名称
#第二种方式
git push origin --delete 远端分支名称
```

4.清理无效的追踪分支

```git
#检测无效的追踪分支
git remote prune origin --dry-run
#清理无效的追踪分支
git remote prune origin
```

### 9.给命令起别名

```git
#给checkout起别名为ch
git config --global alias.ch checkout
```

### 10.本地标签和远程标签

1.查看标签

```git
#查看有哪些标签
git tag
#查看某个标签的详细信息
git show 标签名
```

2.创建标签

```git
#第一种方式（简单标签），只存储当前commit的sha1值
git tag 标签名
#第二种方式（常规标签），生成sha1值，其中包含了当前commit的sha1值
git tag -a 标签名 -m '标签说明'
```

3.推送标签

```git
#第一种方式，写明推送哪个标签
git push origin 标签名1 标签名2 ....
#第二中方式，推送所有标签
git push origin --tags
```

完整写法:

```git
git push origin refs/tags/本地标签名:远程标签名
```

4.获取远程标签

```git
#git pull可以获取一切，但是只可以获取到新增的标签，删除的标签不能感知
git pull
#只拉取标签
git fetch origin tag 标签名
```

5.删除远程标签

```git
#就是把远程标签替换为空
git push origin  :远程标签名
```

注意:如果将远程标签删除，其他用户无法直接感知，还需要通过pull感知

## 17.Git GC

```git
#进入.git目录，执行git gc进行压缩
git gc
```

objects、refs中记录了很多commit的sha1值，如果执行gc则会将这么多的sha1值存放到一个压缩文件packed-refs中

refs:标签、head、remote

objects:对象（即git每一次version的全量内容）

## 18.GIt裸库

没有工作区的工作仓库称之为裸库，裸库一般存在于服务端。

### 1.创建裸库

```git
git init --bare
```

### 2.推送到远程仓库

```git
git push
```

## 19.Submodule子模块

应用场景:在一个仓库中引用另一个仓库的代码。

 远程两个仓库A和B，本地两个仓库A和B，要在A中引入B。

### 1.引入子模块

```git
git submodule add B的ssh
```

### 2.更新子模块

在B仓库更新后，A仓库中的B的子模块时无法直接感知的。

1.在A仓库更新每个子模块

```git
git submodule foreach git pull
```

2.更新远程A仓库

```git
git add .
git commit -m '更新说明'、
git push
```

### 3.克隆子模块

克隆的项目包含submodule

```git
#克隆A仓库及其子模块
git clone A的ssh --recursive
```

### 4.删除submodule

1.删除本地暂存区的子模块

```git
git rm -cached 子模块名称
```

2.删除本地工作区子模块

```linux
rm -rf 子模块名称
```

3.把删除操作提交到远程仓库

```git
git add .
git commit -m '删除子模块说明'
git push
```

建议:submodule单向操作，如果双向操作使用subtree。

## 20.Subtree

## 21.Cherry-pick

如果写了一半（已经提交），发现写错分支了，需要将已提交的commit转移到别的分支

每次只能转移(复制)一个commit，只是复制内容但是sha1会变。

思路: cherry-pick 复制到应该编写的分支上，把写错的分支删除，再新建删除的分支

注意:cherry-pick不要跨节点复制。

```git
#在正确的分支上操作
git cherry-pick 要复制的commit的sha1值
#删除错误的分支  -D是强行删除
git branch -D 分支名   
#新建并切换分支
git checkout -b 刚刚删除的分支名
```

注意:cherry-pick是在对的分支上操作，也就是要将B转移到A需要在A中操作。

## 22.Rebase变基操作

rebase是在错误的分支上操作，也就是要将B转移到A需要在B中操作。

rebase会改变提交历史。

建议:

​	1.rebase分支只在本机操作，不要推送带github

​	2.不要在master上直接rebase操作

1.变基操作

```git
#把dev转移到master,在dev分支操作
git rebase master
```

2.冲突

```git
#忽略冲突，会把自身的修改放弃，用正确的分支修改替换，例如dev中的操作会被放弃，会用master分支的修改替换
git rebase --skip

#也可以解决冲突,每个冲突的节点都需要解决
vi 冲突文件名
git add .
git rebase --continue
```

3.还原成rebase之前的场景

```git
git rebase --abort
```

## 23.搭建GitLab