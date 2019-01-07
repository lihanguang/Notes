#### 分支

* 创建分支和切换分支

  ```
  git branch branch-name // 创建一个分支
  git checkout -b branch-name // 创建一个分支并切换到该分支
  git checkout branch-name // 切换到该分支
  ```

* 删除分支

  ```
  git branch -d // 删除一个分支，如果该分支没有被合并过则会发生失败
  git branch -D // 强制删除一个分支，无论是否合并过
  ```

* 查看分支

  ```
  git branch // 查看本地分支列表
  git branch -v // 查看本地分支列表，并显示该分支的HEAD位置
  git branch --merged // 查看本地已经合并过的分支，如果一个分支创建出来后没有任何提交也会被当做已经合并过的
  git branch --no-merged // 查看本地没有合并过的分支
  git branch -a // 查看所有分支（包括远程分支）
  git branch --all // 同-a
  git branch --remote // 查看远程分支列表
  ```

* 关联远程分支

  ```
  git branch --set-upstream-to=origin/branch-name // 这里的origin是远程仓库的名称
  git branch --set-upstream-to=origin/branch-name branch-name1 // 这里的branch-name1不是当前分支的分支名
  
  // 下面的命令要Git 1.8.0才支持
  git branch -u origin/branch-name // 这里的origin是远程仓库的名称
  git branch -u origin/branch-name branch-name1 // 这里的branch-name1不是当前分支的分支名
  ```

* 拉取远程分支

  ```
  git pull // 拉取远程分支到本地并合并
  git pull <远程主机名> <远程分支名>:<本地分支名> // 拉取远程分支并merge到某个本地分支
  git pull <远程主机名> <远程分支名> // 拉取远程分支并merge到本地当前的分支
  git pull --rebase // 拉取远程分支并rebase到本地分支
  
  git fetch // 拉取远程分支到本地但不合并
  ```

* 推送本地分支到远程

  ```
  git push // 推送当前分支到远程的分支
  git push <远程主机名> <远程分支名>:<本地分支名> // 推送当前分支到远程的分支
  git push --force // 强制覆盖远程分支
  git push -f // 同--force
  ```

  