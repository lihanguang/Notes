### 集合框架
#### Collection
* Java集合框架的基类，继承于Iterable，可以使用foreach迭代方式。
* size()：返回容器的大小
* isEmpty()：返回容器是否为空
* contains(Object o)：容器是否包含该对象
* iterator()：返回迭代器
* toArray()：将容器转换为对象数组
* toArray(T[] a)：将容器转换为指定对象数组
* add(E e)：添加一个元素进入容器
* remove(Object o)：删除一个一个对象
* containsAll(Collection<?> c)：是否包含c中所有元素
* addAll(Collection<? extends E> c)：添加c中所有元素进入容器
* removeAll(Collection<?> c)：删除容器中包含c中的元素的元素
* retainAll(Collection<?> c)：删除容器所有元素，除了c中的元素
* clear()：删除容器中的所有元素

#### Iterator
* 迭代器接口
* hasNext()：有下一个元素
* next()：下一个元素
* remove()：删除，默认抛出异常

#### ListIterator
* 可以向后或者向前移动的迭代器，并且还可以替换当前的位置和插入元素到当前位置，即插入到当前位置的前面

#### List
* List是有序的Collection
* indexOf(Object o)：返回o在数组中的索引
* lastIndexOf(Object o) ：逆序返回o在数组中的索引
* listIterator()：返回ListIterator迭代器
* subList(int fromIndex, int toIndex)：返回子列表
* 常用的List容器：
    * ArrayList：使用一个对象数组来保存元素，没有进行同步。查找快，插入删除慢。迭代的时候能删除但是不能插入，否则会抛出异常
    * LinkedList：使用一个双向循环链表来保存元素，迭代器实现了ListIterator
    * Vetor：和ArrayList一样使用一个对象数组来保存元素，实现代码大部分一样的，对大部分public的方法都使用了 synchronized进行同步
    * CopyOnWriteArrayList：在修改（写）的时候先把原数组复制到一个临时数组然后在对临时数组进行操作，最后把临时数组赋值给原数组（通过volatile来保证每一个获取数组时都是最新的值），实现读写分离。只有在修改的时候使用了同步机制，读取的时候没有使用同步，这在高并发中效率比Vetor要高，但是需要更多的内存

#### Arrays
* 封装了许多静态方法用来处理数组，包括复制、排序、查找等。
* binarySearch(Object[] a, Object key)：二分查找，返回索引
* List<T> asList(T... a)：根据变长参数生成一个列表，生成的是ArrayList
* <T> T[] copyOf(T[] original, int newLength)：复制数组
* copyOfRange(double[] original, int from, int to)：根据范围复制
* deepEquals(Object[] a1, Object[] a2) ：深度比较，如果数组的每个项也是数组则递归进行比较
* sort(int[] a) ：按升序进行排序

#### Collections
* 封装了许多静态方法来操作列表，包括复制、查找、排序、生成线程安全的列表
* binarySearch(List<? extends Comparable<? super T>> list, T key) 二分查找
* checkedList(List<E> list, Class<E> type) 返回一个线程安全的List，相同的还有Set、Map等。
* sort(List<T> list) 排序

#### Set
* Set是一个不包含重复元素的Collection，实现基本上都是基于Map
* 常用的Set：
    * HashSet：使用了HashMap来实现
    * TreeSet:使用了TreeMap来实现
    * LinkedHashSet：使用了LinkedHashMap来实现

#### Map
* Map不是Collection的子类而是一个独立的接口
* size()：Map的大小
* isEmpty()：Map是否为空
* containsKey(Object key)：是否包含key
* containsValue(Object value)是否包含value
* get(Object key)：根据key获取value
* put(K key, V value)：添加键值对
* remove(Object key)：删除key
* putAll(Map<? extends K, ? extends V> m)：添加全部
* clear()：清除
* keySet()：获取key的集合
* values()：获取value的集合
* entrySet()：获取实体的集合
* Entry<K,V>：Map中的键值对定义
    * getKey()：获得key
    * getValue()：获得value
    * setValue(V value)：设置value
* 常用的实现类：
    * HashMap：采用一个散列（链表的数组）来进行存储。存取的速度都为O(1)，遍历时，key是有序的，根据key的hash值来排
    * LinkedHashMap：继承于HashMap，存取速度和HashMap相同，但是覆盖了HashMap的Entry，添加了两个变量before和after用来指向前驱和后继（通过重写hashMap的监听放来和newNode方法来实现），迭代过程使用after而不是next作为下一个元素，这样就可以实现最先获取的都是最先加入map中的值。
    * TreeMap：使用了红黑树来实现的一个排序map。查找是二叉查找树的查找，速度为O(logn)，插入和删除也是O(logn)。传入的值必须是可排序的值（构造函数中传入Comparator对象或value实现了Comparable接口，否则会报错）。
    * HashTable：与HashMap一样采用一个散列（链表的数组）来进行存储，，与HashMap的父类不同，对公共方法进行加锁（包括get方法），不允许key和value为空
    * ConcurrenntHashMap:与HashMap一样采用链表数组来进行存储，和HashTable不同处在于不对方法进行加锁，而是在修改的时候对链表进行加锁，高并发优于HashTable

#### Stack
* 后进先出（LIFO）
* push(E item)：压入一个元素
* E pop()：弹出一个元素
* E peek()：获取栈顶元素
* Java中使用了Vetor来实现的栈，所以可以使用Vetor的所有功能并且是线程同步的

#### Queue
* 先进先出（FIFO）
* add(E e)：插入一个元素，如果队列已满则抛出异常
* offer(E e)：插入一个元素，如果队列已满则返回false
* remove()：删除一个元素，如果队列为空则抛出异常
* poll()：删除一个元素，如果队列为空则返回null
* element()：返回队首元素，如果队列为空则抛出异常
* peek()返回队首元素，如果队列为空则返回null
* Queue的常用实现：
    * LinkedList：上面已写
    * PriorityQueue：优先队列，使用小根堆的方式实现，传入的值必须是可排序的值（构造函数中传入Comparator对象或value实现了Comparable接口，否则会报错），最先出队的总是最合适的数
    * ArrayBlockingQueue：阻塞队列，采用循环数组的方式实现
    * LinkedBlockingQueue：采用链表的方式实现，如不传如参数，则取Integer.MAX_VALUE为满队列的元素个数
    * PriorityBlockingQueue：采用二叉堆的方式实现

#### BlockingQueue
* 阻塞队列，在插入或删除时如队列已满或为空则等待一定时间
* add(E e)：插入元素，不阻塞，已满时抛出异常
* offer(E e)：插入元素，不阻塞，已满则返回false
* put(E e)：插入元素，阻塞已满时等待，会抛出中断异常
* take()：删除并返回一个元素，阻塞为空时等待，会抛出中断异常

#### Deque
* 一个可以在两端插入和删除的队列，相当于有栈和队列的功能
* 继承于Queue
* 常用的实现：
    * ArrayDeque：数组的方式
    * LinkeList：链表的方式

