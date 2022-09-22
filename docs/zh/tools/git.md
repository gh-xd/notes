# Git



## 打印

简洁：`git log --pretty=oneline`

更加简洁：`git log --oneline`

打印HEAD指针信息（指针回到历史版本需要回退多少步）：`git reflog`



## 回退

`git reset --hard <索引UUID>`

`git reset --hard HEAD`

> --hard作用：本地库、暂存区、工作区三个状态，都跟着HEAD指针走（频繁使用）
>
> --soft作用：只有本地库状态变化，暂存区、工作区不动
>
> --mixed作用：移动本地库HEAD指针时，工作区状态不变（文件状态不回退）



## 比较

##### 比较暂存区和工作区的区别

单个文件的对比：`git diff <文件名>`

所有文件的对比：`git diff`

##### 比较暂存区和本地库的区别

与当前指针位置状态比较：`git diff HEAD`

与指定的状态进行比较：`git diff <HEAD指针UUID>`



## 分支

查看分支：`git branch -v`，有*的地方是当前分支

创建分支：`git branch <分支名>`，例如`git branch hot-fix`

切换分支：`git checkout <分支名>`， 例如`git checkout hot-fix`

##### 分支合并和冲突解决

1. 进入主分支
2. 将hot-fix分支内容与master分支合并`git merge hot-fix`：提示有冲突
3. 冲突需要自己解决，协商好以后手动修改
4. 在master中add+commit，解决合并冲突

 

## 远程仓库到本地

`git clone <远程库地址>`

## 远程仓库本地别名

查看远程库别名： `git remote -v`

给一个远程库增加别名：`git remote add <别名> <远程库地址>`，例如：

- `git remote add origin https://github.com/xxxxx.git`
- 经过`git remote -v`查看有两个
  - fetch
  - push



## 本地推送远程库

`git push <别名> <推送到哪个分支>`

例如：`git push origin master`

如果远程仓库没有`master`分支，则会创建`master`分支



## 克隆（外人开发）

1. 电脑中选择一个位置
2. `git clone <远程仓库的地址>`
3. 克隆操作完成三个事情
   1. 初始化本地库Git
   2. 远程库内容完整到本地
   3. 自动创建远程仓库的别名 `git remote -v`

外人推送到Git项目，失败

## 外人推送操作

命令也是 `git push origin master`，但是得先加入团队

在主账号邀请人员：settings > manage access

添加邮件，复制邀请链接，将邀请链接发送给外人

外人打开链接，接受邀请



## 将更新的远程库更新到本地

pull操作（pull = fetch + merge）

##### 操作fetch+merge

抓取（读取）：`git fetch <远程库别名> <远程库分支名>`

例子：`git fetch origin master` -- ==本地工作区里的内容不变！==

可以通过`git checkout origin/master`先查看远程库内容是否正确

然后进行合并：

1. 切换到本地的master`git checkout master`
2. 将远程库origin/master合并到本地的master：`git merge origin/master`

##### 操作pull

`git pull origin master`

等同于fetch + merge

##### 区别

- fetch+merge: 保险慎重
- pull: 不复杂的情况



## 多方开发远程库冲突

同时修改同样地方的话，push会失败

解决方法：先pull最新版本，人为修改/选择最佳代码，add/commit/push

结束



## 跨团队开发（FORK）

公司A创建的远程代码库

公司B使用fork，复制一份到公司B的远程仓库进行开发

公司B开发玩，发送pull request（Github上）

公司A进行审核

公司A进行merge