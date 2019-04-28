##  HashMap

### 一、概述

`HashMap` 是基于 `Map` 接口的实现，它以 `key-value` 的形式存在，就好像 `JavaScript` 中的对象，`Python` 中的字典一样，都是通过 `key` 来访问对应的 `value` 。在翻阅它的源码之前，我对它的认知估计就是停留在这个层面上了吧，总想写点什么再开始阅读它的源码，结果我发现我真的不能说出别的了。那么就直接入正题吧。

### 二、源码分析

#### 1. 类的定义

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

从类的定义来看，`HashMap` 继承了抽象类 `AbstractMap`，提供了 `Map` 的基本实现，使得我们以后要实现一个 `Map` 不用从头开始，只需要继承 `AbstractMap` ，然后按需求实现或重写对应的方法即可。实现的三个接口都是再普通不过的接口了，这里就不再浪费口舌了。

#### 2. HashMap 的数据结构

`HashMap` 比其他的集合都要稍微复杂些，底层存储数据的结构是 **数组+散列链表** ，它并不像咱们之前分析的那些集合那么简单了。前面提到过的 `ArrayList` 其实就是在不断的改变一个数组；`LinkedList` 就是一个双向的链表，每个节点除了记录自身的元素以外，还记录着上一个和下一个节点的索引，翻阅源码也是很得劲的。但是 `HashMap` 却不一样了，我每次翻阅源码之前都有一个习惯，就是先把进度条拉到最后看看有多少行，然而至今为止 `HashMap` 是我见过源码最多的（当然除了框架以外哈），由于这样那样的原因，分析源码就不再像以前那样一点点的带着问题去介绍了，这里首先直接来说明底层的实现，再带着这些实现原理去翻阅源码就差不多了。

哈希表原理：

我们都知道数组是根据索引来查找元素的，一次定位就可以达到。哈希表也是利用了这个特性，哈希表的主干就是数组，链表则是主要为了解决哈希冲突而存在的，如果定位的数组不含链表（言外之外就是，这个位置上只有一个元素），那么对于访问这个元素的速度是要比其他的快的，因为只需计算一次这个地址即可；当然，如果某个位置上存在哈希冲突，那么这个位置就存在链表了，具体的复杂度在后面分析过程再来详细阐述。

看到这里或许有人会问，什么是哈希冲突？在解释这个玩意之前，先说一个哈希函数的东西。都知道 `HashMap` 是一个 `key-value` 形式的键值对，说白了就是一对对的映射关系，在访问具体的元素时，是通过 `key` 来访问对应的 `value` 的，再具体一点说就是，通过一个函数计算出一个值，然后就能得到对应的 `value` 了（理想状态下）。这里的函数就是哈希函数，一般情况下，不同的值通过哈希函数计算出来的结果是不同的（同样的值计算出来的结果一致），那么，两个`key` 的计算结果相同的时候怎么办呢？总不能使用数组的一个位置来存放两个`value` 吧，所以就有了链表这东西了，如果两个`key`通过哈希函数计算的结果一致，就会创建（已经有了就使用原来的）链表，将`value` 存放到链表上。上面这个过程也称为哈希冲突（也称为哈希碰撞）。

#### 3. 重要属性

```java
//桶（数组），初始化容量为16
transient Node<K,V>[] table;
//初始化数组的长度为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;//2^4
//最大数组的长度
static final int MAXIMUM_CAPACITY = 1 << 30;//2^30
//负载因子默认值，代表table的填充度
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//当添加元素到某个桶时，若链表长度达到8就会将链表转化为红黑树
static final int TREEIFY_THRESHOLD = 8;
//当某个桶的位置上的链表长度退减为6时，由红黑树转化为链表
static final int UNTREEIFY_THRESHOLD = 6;
//存储键值对的个数
transient int size;
//负载因子引用，一旦初始化就不能修改
final float loadFactor;
//数组的极限容量
int threshold;
```

其中，上面还提到一个 `Node` 类型，来看看源码到底是怎么回事：

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//哈希值
    //键值对 key-value
    final K key;
    V value;
    Node<K,V> next;//下一个节点的引用
	//构造方法，四个参数就是对应着上面的四个属性
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
	//键值对的get,set方法
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //toString方法
    public final String toString() { return key + "=" + value; }
	//hashCode方法
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
	//equals方法
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

从源码来看，这个类并不算复杂，只是定义了四个属性，一个带有所有参数的构造方法，还有访问和操作键值对的`get,set`方法，最后还重写了三个基本的方法。这下子有底气了吧，综合上面的源码来看，`HashMap` 其实也不是什么难啃的东西，在`HashMap`的内部其实就是维护着一个 `Node<K,V>[]`这样的数组。

#### 4. 构造方法

在 `HashMap` 中除了提供了 `Collection` 大家族中基本的两个构造方法以外，另外还提供了两个带参的构造方法，一共提供了四个，来看看：

```java
//空参构造方法，所有的属性都使用默认值
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; 
}
//设置数组的初始化容量，在这个构造器中调用的另外一个构造器，请看下面
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);//负载因子系数为默认的0.75
}
//HashMap中比较重要的一个构造方法（上面的也是调用了该方法）
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)//如果传递过来的容量不合法（小于0），则抛出异常（你逗我玩呢传我一个负数）
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)//如果传递过来的初始化容量大于最大值2^30，就使用最大值作为初始容量
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))//如果负载因子的系数小于或等于0，则抛出异常（逗我呢，小于等于0就不就是意味着这个容器装不了东西了吗，所以小于或等于0是没有意义的。
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;//初始化负载因子系数
    this.threshold = tableSizeFor(initialCapacity);//初始化数组容量
}
```

也许从上面一直读下来都会很顺利，但是到了最后出现了一个方法 `tableSizeFor`() ，你肯定会对此非常感兴趣吧，到底是如何初始化数组的容量的呢，来看看：

```java
static final int tableSizeFor(int cap) {//以参数10为例解析这段代码
    int n = cap - 1;//此时n为9
    n |= n >>> 1;// 9 无符号右移1位： 1001 >>> 1 结果为 0100 (十进制为4)，最后，9 |= 4 等价于 1001 | 100 ，结果为 1101 十进制数为13
    n |= n >>> 2;// 根据上面的结果n为13，1101 >>> 2 结果为 0011（十进制为3），最后 1101 | 0011 ，结果为 1111
    n |= n >>> 4;// 根据上面的结果n为15，1111 >>> 4 结果为 0000（十进制为0），最后 1111 | 0000，结果为 1111
    n |= n >>> 8;// 根据上面的结果n为15，1111 >>> 8 结果为 00000000（十进制为0），最后 1111 | 0000，结果为1111
    n |= n >>> 16;// 根据上面的结果n为15，1111 >>> 8 结果为 00000000（十进制为0），最后 1111 | 0000，结果为1111
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;//最后返回的就是 15+1 =16，这样就找到了大于输入数且最接近2的整数次幂的数
}
```

这是在干嘛？直接给答案吧，这个方法的作用就是找到大于参数`cap`且最接近2的整数次幂的数。比如，参数为10，则返回的结果就是16。

**这也就是说，这个数组的容量不是你能随便指定的，它会根据你的输入找到一个2的整数次幂的数，并将这个数设置为数组的容量。**

最后的一个构造方法就是`Collection` 家族都有的，参数为 `Collection` 集合：

```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;//负载因子为默认值0.75
    putMapEntries(m, false);//不看源码估计也能猜出这个方法的作用就是将集合的所有k-v加入到咱们的数组容器中作为初始化值。
}

final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();//获取参数集合的长度，长度大于0才进行下面的操作（小于0没有意义嘛）
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {//循环添加每对键值对
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);//这里提到的putVal方法在后面再详细分析
        }
    }
}
```

#### 5. 重要方法

##### i. 获取哈希值：

```java
static final int hash(Object key) {//将key映射为哈希值，如果key为null，则直接返回0，否则通过(h = key.hashCode()) ^ (h >>> 16) 计算得出，hashCode()是Object中定义的。
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

##### ii. 增加方法

在 `HashMap` 中提供了5个增加的方法，分别是 `put(),putAll(),putIfAbsent(),putMapEntries(),putVal()`：

先从 `put` 开始看：

```java
public V put(K key, V value) {//向数组容器中添加一对键值对key-value
    return putVal(hash(key), key, value, false, true);//真正执行添加逻辑的方法
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

是否已经懵了？这里就不再每行每句的分析了，直接上最终的结果：

- 首先判断 `table` 是否为空或者 `null` ，如果是，则初始化数组 `table`。
- 根据键值`key`计算`hash`值，并得到桶索引位置，如果`table[i]=null`，直接新建节点添加，判断实际存在的键值对数量`size`是否超过了最大容量，如果超过则进行扩容；如果不为空，则执行下面的步骤。
- 判断`table`的首个元素是否和`key`相同，如果相同的话直接覆盖`value`，否则执行下一步。（注：这里的相同指的是`hashCode`和`equals`）
- 判断`table[i]`是否为 `treeNode`（是否为红黑树），如果是红黑树，则直接在树中插入键值对，否则执行下一步。
- 遍历`table[i]`，判断链表长度是否大于8，大于8则将链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作。
- 插入成功，判断容量，超过(阈值)则进行扩容。

##### iii. 获取方法

先来看看最常用的通过`key` 获取 `value` 的方法：

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;//如果获取的某个节点为空，则返回空，否则返回对应的value
}
//真正执行获取节点逻辑的方法（先放下暂不分析）
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

