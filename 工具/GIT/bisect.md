#### Bisect

bisect是Git中用来查找某个引起错误的commit，他的原理是使用二分法来查找。

* 使用方法

  ```
  //1.使用下面的命令，表示从startpoint到endpoint中查找
  //此时git会自动将工作区回滚到两点的中间节点
  git bisect start startpoint endpoint
  
  //2.标记错误或者正常
  //运行当前节点的代码，如果可以成功运行则使用good，否则ba
  //使用该命令后Git会根据二分法的原则来移动到下一个节点
  git bisect good
  git bisect bad
  
  //3.如果Git可以判断出是那个commit引入错误之后会显示
  XXX is the first bad commit
  
  //4.退出bisect，此时会回到最近一次的commit
  git bisect reset
  ```

  