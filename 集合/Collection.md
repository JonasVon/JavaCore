##  Collection 集合

### 一、Collection 接口

`Collection` 是站在集合“食物链”最顶端的接口，它是集合的最基本定义。实现了 `Collection` 接口的类都必须提供两套标准的构造函数，一个是无参的，用于创建一个空的 `Collection` ；另外一个是带有 `Collection` 参数的，用于创建一个新的 `Collection` 。

![](img/collection-2018-03-02.jpg)

从上图可见，咱们经常使用的 `List` 和 `Set` 是 `Collection` 的子接口，分别代表着两种不同的集合。

### 二、List 接口

`List` 接口是 `Collection` 接口的直接子接口。`List` 代表的是有序的，可插入重复元素的 `Collection` ，它用某种特定的插入顺序来维护元素顺序。咱们可以根据元素的整数索引访问元素，并搜索列表中的元素。其中，`List` 接口最主要的实现类有 `ArrayList` 、`LinkedList` 、 `Vector` 和 `Stack` 。

### 1. ArrayList

`ArrayList` 是一个动态数组，也是最常用的集合。它允许任何符合规则的元素插入（甚至包括`null`）。每个 `ArrayList` 都有一个初始容量 10，随着容器元素不断增加，容器的大小也会随着增加。在每次向容器中增加元素的同时都会进行容量检查，当快溢出时，就会进行扩容操作。所以，如果我们明确所插入的元素的数量，最好指定一个初始容量，以提高那么一点点的效率。

### 2. LinkedList

同样是实现 `List` 接口，但是 `LinkedList` 与 `ArrayList` 不同，`ArrayList` 是一个动态数组，而 `LinkedList` 是一个双向链表。所以它除了有 `ArrayList` 的基本操作方法外还额外提供了 `get,remove,insert` 方法。由于实现方式不同，`LinkedList` 不能随机访问，它所有的操作都是要按照双重链表的需要执行。在列表中索引的操作将从开始或结尾遍历列表（从靠近指定索引的一端），这样做的好处就是可以通过较低的代价在 `List` 中进行插入和删除操作。

### 3. Vector

与 `ArrayList` 相似，但是 `Vector` 是同步的。所以说它是线程安全的动态数组，操作与 `ArrayList` 几乎一样。

### 4. Stack

`Stack` 继承自 `Vector`，实现一个后进先出的堆栈。

### 三、Set 接口

`Set` 接口是一种不包括重复元素的 `Collection` ，它维护它自己的内部排序，所以随机访问没有任何意义。实现 `Set` 接口的类主要有 `EnumSet` 、`HashSet` 和 `TreeSet` 。

#### 1. EnumSet

所有元素都是枚举类型的 `Set`

#### 2. HashSet

`HashSet` 堪称查询速度最快的集合，因为它内部是以 `HashCode` 来实现的，它内部元素的顺序是由哈希码来决定的，所以它不保证`set` 的迭代顺序，特别是它不保证顺序恒久不变。

#### 3. TreeSet

基于 `TreeMap` ，生成一个总是处于排序状态的 `Set`，内部以 `TreeMap` 来实现。它是使用元素的自然顺序对元素进行排序，或者根据创建 `Set` 时提供的 `Comparator` 进行排序，具体取决于使用的构造方法。

### 四、Map 接口

`Map` 与 `List` 或 `Set` 都不同，它是由一系列的键值对组成的集合，提供了 `key` 到 `value` 的映射。就好像 `JavaScript` 中的对象或者 `Python` 中的字典。同时，它没有继承 `Collection` ，在 `Map` 中它保证了 `key` 与 `value` 之间的一一对应关系。实现类主要有 `HashMap` 、`TreeMap` 、`Hashtable` 。

#### 1. HashMap

以哈希表数据结构实现，查找对象时通过哈希函数计算其位置，它是为快速查询而设计的，其内部定义了一个 `hash` 表数组，元素会通过哈希转换函数将元素的哈希地址转换成数组中存放的索引，如果有冲突，则使用散列链表的形式将所有相同哈希地址的元素串起来。

#### 2. TreeMap

键以某种排序规则排序，内部以 `red-black` 红黑树数据结构实现，实现了 `SortedMap` 接口。

#### 3. Hashtable

以哈希表数据结构实现的，解决冲突时与 `HashMap` 也一样采用了散列链表的形式，不过性能比 `HashMap` 要低。

### 五、Queue

队列，主要分为两大类，一类是阻塞队列，队列满了以后再插入元素就会抛出异常，主要包括 `ArrayBlockQueue` 、`PriorityBlockingQueue` 和 `LinkedBlockingQueue` 。另一种队列则是双端队列，支持在头尾两端插入和移除元素，主要包括 `ArrayDeque` 、`LinkedBlockingDeque` 和 `LinkedList` 。

以上基本上就是集合的大家族，先在大体上对集合有点概念，接下来的文章再慢慢把它们吃透。

