---
title: LearnGit  
date: 2021-04-08  
tags:
- Git
- Tools

categories:
- 版本管理
---
## 创建版本库

- git init 创建git仓库
- git add 添加多个文件 需要提交的文件修改通通放到暂存区
- git commit -m <message> 把文件提交到仓库（一次性提交暂存区的所有修改到分支上，默认分支为master）

##  时光机穿梭

- git status 查看仓库状态
- git diff  <filename> 查看文件改动

## 版本回退

- git log -- pretty=oneline 查看版本历史 按q退出
- HEAD 表示当前版本 HEAD^表示上一版本 HEAD^^表示上上版本...
- git reset --hard (HEAD^ 或者版本id) 回退到某一版本
- git reflog 查看所有操作历史（可以获取版本id）

## 管理修改

- git 面向的是修改而不是文件，要是文件的修改没有提交（add），那么这次修改不会被提交到分支
- git diff <版本id> -- 文件 可以查看工作区和版本库里面区别

## 撤销修改

- git checkout -- file 
  - 要是文件没有add，回到跟分支一样的状态
  - 要是文件已经add，回到缓存区的状态
- git reset HEAD <file> 可以把暂存区回退到跟分支一样（ustaged），然后通过checkout来回退工作区

## 删除文件

1. 要是文件没有add 直接删除
2. 要是文件已经add，git restore --staged <filename> 来取消暂存，然后删除即可
3. 要是文件已经add，然后又删除了，可以先 git restore <filename>来恢复文件，然后就回到了状态2
4. 要是文件已经add，并且commit了，这时候删除了文件
   1. git add/rm 文件 然后commit来提交修改
   2. git checkout -- filename 来恢复

## 远程仓库

- git remote add origin git路径 添加远程仓库，origin为远程仓库的默认名
- git push -u origin master 第一次推送本地master分支到origin/master下，并关联本地master与远程master
- 之后可以使用git push origin master来把本地修改推送到远程仓库
- git remote -v 查看远程仓库信息
- git remote rm 远程仓库名来移除远程仓库关联
- git clone git地址 克隆远程仓库到本地

## 创建与合并分支

- master指向最新的提交，Head指向master

- 创建新的分支的时候，新建一个指针dev，然后把head指向dev

- 以后的提交都是对dev的提交，要是合并的话，master直接指向dev就完成了合并

- ```
  git checkout -b dev 创建并切换到dev
  相当于git branch dev // 创建
  git checkout dev// 切换分支
  ```

- git branch 查看分支，带*的为当前分支

- git merge <分支名> 合并目标分支到当前分支

- git branch -d dev 删除dev分支

- git switch -c dev 创建并切换到 dev分支

- git switch master 切换到master分支

## 解决冲突

- 产生冲突后，在当前分支上修改冲突后提交即可

- ```
  git log --graph --pretty=oneline --abbrev-commit 可以查看合并路径图
  ```

## 分支管理策略

- ```
  git merge --no-ff -m "merge with no-ff" dev 禁用fast foward 模式，因为这种模式删除分支后会丢失分支信息
  禁用ff之后需要新建一个commit 需要一个commit message
  ```

## bug分支

- 场景：master分支出现bug，自己在dev分支上做开发，开发到一半，无法提交代码
  1. git stash 保存工作现场
  2. 切换到master分支，之后新建bug-fix分支 git switch -c bug-fix, 并commit，这里获取commit id
  3. 修改之后切换到master分支，合并bug-fix git merge --no-ff -m 'bugfix' bug-fix
  4. 删除bug-fix分支 git branch -d bug-fix
  5. 切换回dev分支，dev是从master 拉下来的 master的bug肯定在dev上也有，需要先git cherry-pick 加上之前的修改bug的commit id来修复dev上的bug
  6. 然后需要从stash list获取之前的工作现场
     1. git stash apply （stash{0}）恢复，但是stash list里面的内容不删除，需要git stash drop来删除
     2. git stash pop恢复，同时删除stash里面的内容
     3. 可以多次stash 只需要通过 git stash list来获取stash id即可

## feature分支

- 开发新的feature最好使用一个新的分支
- 新的分支开发完成后不想合并到dev分支的时候，而且想删除feature分支，但是此时feature还没有被合并 需要 git branch -D feature 这个命令来强制删除

## 多人协作

- git push origin branch-name 推送本地某个分支到origin

- git checkout -b branch-name origin/branch-name 从origin的分支检出到自己分支，名字最好一致

- git branch --set- upstream-to branch-name origin/branch-name 建立本地分支和远程分支的关联

- 场景是小伙伴在dev开发，并在origin/dev分支上做了提交，而自己也对相同的文件做了修改，并试图推送

  1. 直接推送会提示远端冲突，需要先git pull下来

  2. git pull的时候会失败，因为自己新建的dev分支没有和远端的dev关联

  3. ```
     git branch --set-upstream-to=origin/dev dev 这个命令来关联本地dev和origin/dev
     ```

  4. git pull 成功，但是和自己代码冲突，解决冲突后 add commit push 就能把自己的代码推送到origin

- 多人协作模式

  1. git push origin <branch-name> 推送自己的修改
  2. 推送失败，说明远程分支比自己新，需要git pull 并合并（如果git pull提示no tracking information 说明本地分支和远程分支没有建立关系，需要git branch --set- upstream-to 本地分支名 origin/远端分支名）
  3. 合并有冲突，解决冲突，add commit 
  4. 如果解决了冲突，则回到步骤一

# rebase

- git rebase可以把本地没有push的分叉历史整理成直线

## 标签

- git标签是版本库的快照，其实是指向某个commit的指针，跟分支指针的区别就是分支指针可以移动，但是标签不可以
- 默认标签是打在最新提交的commit上，找到历史commit id (git log)也能制定commit id 打标签
- git tag v1.0 / git tag v0.9 <commit id> （指定commit id）/ git tag -a <tagname> -m <message> （附加标签信息）
- git tag 查看所有的标签
- git show <tagname> 查看标签信息
- git tag -d <tagname> 删除标签（创建的标签默认存放在本地，不会自动推送到远程，可以放心删除）
- git push origin tagname 把标签推送到远程仓库
- git push origin --tags 一次性推送所有尚未推送的标签
- 删除远程仓库的标签需要两步
  1. git tag -d tagname
  2. git push origin :refs/tags/tagname

