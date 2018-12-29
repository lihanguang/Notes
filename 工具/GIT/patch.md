##### 生成patch

* 某次提交（含）之前的几次提交：

  ```
  git format-patch 【commit sha1 id】-n
  ```

* 某次提交的patch：

  ```
  git format-patch 【commit sha1 id】 -1 // -1是必须的
  ```

* 某两次提交之间的所有commit：

  ```
  git format-patch 【commit sha1 id】..【commit sha1 id】 //前开后闭
  ```

* 某次提交到HEAD之间的所有commit：

  ```
  git format-patch 【commit sha1 id】 //不包括当前的Commit
  ```

##### 应用patch

* 检查patch

  ```
  git apply --check 【path/to/xxx.patch】
  ```

* 打入patch

  ```
  git apply 【path/to/xxx.patch】
  git  am 【path/to/xxx.patch】
  ```

* 解决冲突

  ```
  git  apply --reject  xxxx.patch
  git  am --reject  xxxx.patch
  
  // 如果有冲突则会生成一个.rej文件，根据rej文件的冲突来修改当前的文件
  // 修改完成后执行git add 添加到暂存区
  // 之后执行 git am --resolved 或 git am --continue
  
  ```

* 退出patch

  ```
  git am --skip 跳过此次冲突
  git am --abort 取消patch
  ```
