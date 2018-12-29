#### 存储

* 保存当前未提交的修改

  ```
  git stash
  ```

* 应用上次的存储

  ```
  git stash pop
  ```

* 查看所有存储

  ```
  git stash list
  ```

* 保存当前未提交的修改并添加注释

  ```
  git stash save "msg"
  ```

* 应用某个存储

  ```
  git stash apply [stash@{?}]
  ```

* 删除某个存储

  ```
  git stash drop [stash@{?}]
  ```

* 删除全部存储

  ```
  git stash clear
  ```

* 查看某个存储和当前目录的差异

  ```
  git stash show [stash@{?}]
  ```

* 最最新的存储创建分支

  ```
  git stash branch [branch-name]
  ```

  