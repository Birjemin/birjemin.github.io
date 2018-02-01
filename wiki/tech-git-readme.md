# Git的简易使用

## Git简介
![git简介](http://upload.ouliu.net/i/20180201210814tzrym.jpeg)

```
Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
``` 
git是一个免费且开源的分布式版本控制系统，专门处理高速发展的从小至大的项目。

## 概念
![git概念](http://upload.ouliu.net/i/201802012120532t2rl.jpeg)

工作区在计算机上的体现就是一个目录，与其它目录区别是其下有一个隐藏的 .git 目录，该目录就为版本库，是git的核心，不要随便手动修改此目录的内容。版本库中有三个最重要的文件，index，称为暂存区stage；分支文件，如master；HEAD文件，保存指向当前版本库中当前分支的指针。 
比如在某个目录`git init`的时候该目录就会变成工作区，修改里面的内容时，通过`git status`可以查看修改的情况，通过`git add fileName`将工作区的内容提交到暂存区，也可以通过`git status`查看当前情况，`git commit -m 'desc'`就会把暂存区的内容提交到本地版本库中，`git push`就会将本地的版本库推送到服务器。

暂存区：

## 普通操作
1. 安装git

2. 自报家门
```
git config --global user.name "birjemin"
git config --global user.email "birjemin@foxmail.com"
```

3. 设置默认分支（用于便捷的提交到服务器，比如直接git push dev,如果origin/dev存在则推送，反之服务器创建origin/dev）
```
git branch --set-upstream-to=origin/dev dev
```

4. 注册+提交代码
```
	git init
	vi readme.txt
	git add readme.txt
	git commit -m "This is readme.txt"
	git remote add origin https://github.com/Birjemin/demo1.git
	git push -u origin master
```

5. 拉取代码
```
git pull
```

6. 基本上的工作流程
```
git pull // 拉去服务器资源到本地
git checkout branch // 切换分支
git checkout -b branch // 切换到一个新分支（即创建分支并且切换到创建的分支上面）
git status // 查看状态
git add fileName // 提交file
git commit -m "desc" // 添加备注
git push // 推送分支到服务器
git diff fileName // 查看修改情况
```

## 骚操作
1. 储藏资源
情形：在branchA开发，突然线上出现bug，需要切换到master上面切分支修复bug,但是branchA上面的东西修改了一半，还没做完，你不想提交到版本库中（没开发完；提交到版本库中无法使用git diff fileName查看修改了什么），
```
git stash // 储藏这次修改
git checkout master // 切换到master
...
git checkout branchA // bug修复完，切换回branchA
git stash pop // 将储藏区取出修改
git status 
git merge --no-ff branchA // 合并分支到当前分支
```

2. 撤销
```
git checkout -- fileName // 撤销git add fileName
git reset HEAD filename // 撤销git commit 的文件
git log/git reset --hard 版本号 // 查看log，并且撤销到某个版本
```

3. 删除分支
```
git branch -d branchName // 删除本地版本库的分支branchName
git push origin :branchName // 将删除的分支推送到服务器（删除服务器对应的分支）
git push -f origin :branchName // 如果该分支没有合并到master,确定该分支不会再使用，可以通过此命令强删
```

4. 忽略文件
.gitignore文件添加相应的目录或者文件即可。

5. 解决冲突
```
git merge --no-ff branchA // 合并branchA分支到当前分支，冲突啦~~
git status // 查看哪里冲突了
... // 修改
git add .
git commit -m 'desc'
git push
```

6. 查看文件修改记录
```
git log -- fileName
git show commit-id fileName
```

## 综述
推荐软件:sourceTree
推荐ide都装上git插件，毕竟你看一个文件的历史修改记录总不会通过命令行来看吧。。。。
比如：
![phpstorm的git插件](http://upload.ouliu.net/i/20180201215612mlqly.jpeg)

## 参考
1. [https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
