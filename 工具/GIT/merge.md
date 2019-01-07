#### 合并（merge）

* 取消合并，有冲突的时候可以用该参数来中断merge

  ```
  git merge --abort
  ```

* 合并后是否直接产生一个Commit

  ```
  git merge --commit // 合并后直接产生一个commit
  git merge --no-commit // 合并后不直接产生一个commit，而需要手动执行git commit
  ```

* 合并提交前是否调用编辑器来编辑commit msg

  ```
  git merge --edit // 合并提交前调用编辑器来编辑msg
  git merge --no-edit // 合并提交前不会调用编辑器来编辑msg
  ```

* 是否使用fast-forward模式来合并，意思就是会不会生成一个merge commit

  ```
  git merge --ff // 使用
  git merge --no-ff // 不使用
  git merge --ff-noly // 除非当前HEAD节点已经up-to-date（更新指向到最新节点）或者能够使用fast-forward模式进行合并，否则的话将拒绝合并，并返回一个失败状态。 
  ```

* 显示合并的日志

  ```
  git merge --log [=<n] // 将在合并提交时，除了含有分支名以外，还将含有最多n个被合并commit节点的日志信息。 
  git merge --no-log // 不会列出该信息
  git merge -v // 显示合并的详细信息
  git merge --verbose // 同-v
  ```

* 静默合并

  ```
  git merge -q
  git merge --quit
  ```

* 是否把需要合并的内容合并成一个commit

  ```
  git merge --squash // 合并成一个commit
  git merge --no-squash // 不会
  ```