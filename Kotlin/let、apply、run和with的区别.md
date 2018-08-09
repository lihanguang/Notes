#### let
* 官方源码
```
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)
```
* 例子
```
fun main(args: Array<String>) {
    val list: MutableList<String> = mutableListOf("A","B","C")
    val change: Any
    change = list.let {
        it.add("D")
        it.add("E")
        it.size
    }
    println("list = $list")
    println("change = $change")
}
```
* 结论：在let中，用it表示引用对象，并可调用其方法，it不可省略。返回值是语句块的最后一行，若最后一行语句无返回值，则整个let语句块也无返回值

#### apply
* 官方源码
```
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```
* 例子
```
fun main(args: Array<String>) {
  val list: MutableList<String> = mutableListOf("A", "B", "C")
  val change: Any
  change = list.apply {
      add("D")
      add("E")
      this.add("F")
      size
  }
  println("list = $list")
  println("change = $change")
}
```
* 结论：在apply中，用this代表当前引用对象，并且调用其方法时，this可省略。
apply必有返回值，且返回值是当前引用对象

#### run
* 官方源码
```
public inline fun <T, R> T.run(block: T.() -> R): R = block()
```
* 例子
```
fun main(args: Array<String>) {
  val list: MutableList<String> = mutableListOf("A", "B", "C")
  val change: Any
  change = list.run {
      add("D")
      add("E")
      this.add("F")
      size
  }
  println("list = $list")
  println("change = $change")
}
```
* 结论：在run中，用this代表当前引用对象，并且调用其方法时，this可省略。返回值是语句块的最后一行，若最后一行语句无返回值，则整个run语句块也无返回值

#### with
* 官方源码
```
public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```
* 例子
```

fun main(args: Array<String>) {
  val list: MutableList<String> = mutableListOf("A", "B", "C")
  val change: Any
  change = with(list) {
      add("D")
      add("E")
      this.add("F")
      size
  }
  println("list = $list")
  println("change = $change")
}
```
* 结论：with函数是一个单独的函数，并不是Kotlin中的extension，所以调用方式有点不一样，返回是最后一行，然后可以直接调用对象的方法，感觉像是let和apply的结合。

#### let和apply的一些区别
* let中的this织带外部类的this，it指代自己，而apply中的this指代自己