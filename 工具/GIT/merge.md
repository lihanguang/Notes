#### merge

- --abort 合并分支有冲突的时候用来中断merge
- --commit 合并后产生一个合并结果的Commit节点
- --no-commit 合并后不会产生一个Commt节点，需要手动进行commit
- --edit 合并成功后提交前调用编辑器来编辑commit msg
- --no-edit 与edit相反
- --ff 使用fast-forward模式来进行合并，意思就是不会生成一个merge commit
- --no--ff 与--ff相反
- --ff-only 除非当前HEAD节点已经up-to-date（更新指向到最新节点）或者能够使用fast-forward模式进行合并，否则的话将拒绝合并，并返回一个失败状态。 
- --log [=<n] 将在合并提交时，除了含有分支名以外，还将含有最多n个被合并commit节点的日志信息。 
- --no-log 不会列出该信息
- -q --quiet 静默操作，不显示合并进度信息
- -v --verbose 显示详细的合并结果信息