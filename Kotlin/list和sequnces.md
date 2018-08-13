#### List的用法

```
val list = listOf(1, 2, 3, 4, 5, 6)
val result = list
        .map{ println("In Map"); it * 2 }
        .filter { println("In Filter");it % 3  == 0 }
println("Before Average")
println(result.average())
```

结果如下：

```
In Map
In Map
In Map
In Map
In Map
In Map
In Filter
In Filter
In Filter
In Filter
In Filter
In Filter
Before Average
9.0
```

### Sequence的用法

```
val list = listOf(1, 2, 3, 4, 5, 6)
val result = list.asSequence()
        .map{ println("In Map"); it * 2 }
        .filter { println("In Filter");it % 3  == 0 }
println("Before Average")
println(result.average())
```

结果如下：

```
Before Average
In Map
In Filter
In Map
In Filter
In Map
In Filter
In Map
In Filter
In Map
In Filter
In Map
In Filter
9.0
```

#### 两者对比

* 当不需要中间操作时，使用List 
* 当仅仅只有map操作时，使用sequence 
* 当仅仅只有filter操作时，使用List 
* 如果末端操作是first时，使用sequence 