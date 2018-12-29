#### rebase

###### 普通的rebase

会把当前分支的提交移动到目标分支上，并为每个提交都重新创建一个提交。相当于重写了历史

- 优点：
  - 使项目历史变得非常整理
- 缺点：
  - 无法追踪到整个项目的分支合并历史，因为rebase操作没有而外的commit

###### 交互式rebase

git rebase -i startpoint endpoint 包括endpoint ，不包括startpoint。所以一般使用startpoint^来指向前一个commit。endpoint可以省略，此时为当前的HEAD

- p(pick) 保留该commit 
- r(reword) 保留该commit，但我需要修改该commit的注释 
- e(edit) 保留该commit, 但我要停下来修改该提交(不仅仅修改注释) 
- s(squash) 将该commit和前一个commit合并 
- f(fixup) 将该commit和前一个commit合并，但我不要保留该提交的注释信息 
- x(exec) 执行shell命令 
- d(drop) 我要丢弃该commit 