# Git 基本语法

## git bash命令行的快捷操作

1. `上、下箭头  （查找最近几次的输入命令`
2. `history 查看命令历史）`
    1. `ctrl + r    （进入搜索命令状态）`
    2. `ctrl + p, ctrl + n  （翻看历史记录）`
    3. `ctrl + shift + c    （复制）`
    4. `ctrl + shift + v    （粘贴）`
3. `左、右箭头  （修改命令里错误的内容，例如拼写错误的单词）`
    1. `alt + b, alt + f    （逐单词的左移，右移）`
    2. `ctrl + a, ctrl + e  （移动到整条命令的起始位置、终止位置）`
    3. `ctrl + d    （删除，相当于Delete）`
    4. `ctrl + h    （删除，相当于Backspace）`
    5. `ctrl + u    （删除光标左侧的所有内容）`
    6. `ctrl + k    （删除光标右侧的所有内容）`
    7. `ctrl + w    （删除光标左侧的单词）`
    8. `alt + d （删除光标右侧的单词）`

## 将代码上传到master分支（github中使用master作为主分支）

1. `git init    //工作空间创建.git文件夹（默认隐藏了该文件夹）`
2. `git add .   //添加到暂存区`
3. `git commit -m "你的提交注释注释"`
4. `git remote add origin http://xxxxxxxxx.git  //本地仓库和远程github关联`
5. `git pull --rebase origin master //远程有readme.md，拉一下`
6. `git push -u origin master   //代码合并`

## 将代码上传到main分支（github中使用main作为主分支）

1. `git init    (工作空间创建.git文件夹，默认隐藏了该文件夹）`
2. `git add .   （添加到暂存区）`
3. `git commit -m "你的提交注释注释"`
4. `git remote add origin http://xxxxxxxxx.git  （本地仓库和远程github关联）`
5. `git pull --rebase origin main   （远程有readme.md，拉一下）`
6. `git push -u origin main （代码合并）`
7. `git branch -m master main   （将本地分支改名字）`

## 工程多人开发

1. `git remote add origin git@github.com:Jia-Baos/Yolov1.git`
2. `git pull origin master（对于自己的库）`
3. `git clone git@github.com:Jia-Baos/Yolov1.git（对于别人的库）`
4. `git remote remove origin（删除之前添加的远程仓库）`
5. `git remote -v（查看远程版本库信息）`

## 本地文件操作

1. `git ls-files    （查看已存在仓库的文件）`
2. `git status  （查看还没有添加的文件）`

## log模式的退出

1. `git log （查看最近日志），退出：q`
2. `git branch （查看本地所有分支）`
3. `git branch -r （查看远程所有分支）`
4. `git branch -d branch_name （删除本地分支）`
5. `git push origin --delete branch_name （删除远程分支）`

## 多人协作加分支

1. `git checkout master （切换到基础分支）`
2. `git checkout -b newBranch   （创建并切换到新分支）`
3. `更新分支代码并提交`
    1. `git add.`
    2. `git commit -m "init new Branch"`
    3. `git push origin newBranch`
4. `git branch -a   （查看所有分支）`
5. `git brach   （查看当前使用分支，结果列表前面有*号，代表当前使用的分支）`
6. `git checkout <newbrach_name>     （切换分支）`
7. `git merge branch_name （将分支合并到主分支，本地文件）`

## Git撤销、回退

1. `git reset --soft HEAD~1 （撤销最近一次的commit(撤销commit，不撤销git add)）`

2. `git reset --mixed HEAD~1    （撤销最近一次的commit(撤销commit，撤销git add)）`

3. `git reset --hard HEAD~1 （撤销最近一次的commit(撤销commit，撤销git add，工作区的代码改动将丢失。操作完成后回到上一次commit状态)）`
