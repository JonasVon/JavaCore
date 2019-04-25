##  LinkedList

### 一、概述

`LinkedList` 与 `ArrayList` 一样实现了 `List` 接口，不同的是， `ArrayList` 底层储存数据的是大小“可变”的数组（具体的分析请看 `ArrayList` 那篇文章）；而 `LinkedList` 是对链表的实现。而且是一个双向的链表：

![](C:\Users\jonas\Desktop\JavaCore\集合\img\linkedlist-2018-03-03.png)

### 二、源码分析

#### 1. 类的定义

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

`LinkedList` 继承了 `AbstractSequentialList`，实现了 `List,Deque,Cloneable,java.io.Serializable` 。其中，`AbstractSequentialList` 提供了 `List` 接口的骨干实现，从而最大限度减少了实现 “连续访问” 数据存储支持的此接口所需的工作，从而减少实现 `List` 接口的复杂度。

#### 2. 属性

```java
//代表LinkedList的长度
transient int size = 0;
//第一个元素
transient Node<E> first;
//最后一个元素
transient Node<E> last;
```

再啰嗦一句，`transient` 定义的属性就是不参与序列化的。除此以外你肯定还会对 `Node` 这个家伙感兴趣吧，它是 `LinkedList` 的一个内部类：

```java
private static class Node<E> {
    E item;//当前的这个元素
    Node<E> next;//下一个元素
    Node<E> prev;//上一个元素

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### 3. 构造方法

构造方法只有两个，也是 `Collection` 大家族最基本的那两个，一个是空参构造器，另外一个就是带有 `Collection` 类型的参数，用于将这个参数作为初始化容器的数据。空参的就不贴代码了，来看看带参数的吧：

```java
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

代码也是很简单的，就是将参数 `Collection` 的元素作为容器初始化的元素。

#### 4. 常用方法

##### i. 增加方法

在 `LinkedList` 中提供了四个用于增加元素的方法，分别是：`add()、addAll()、addFirst()、addLast()`，那么就从最简单的开始吧：

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
```

哟，直接看这个名字好像是往链表的末尾添加元素喔。但是，上面不是有个 `addLast()` 方法吗？又是干嘛的啊？来看看：

```java
public void addLast(E e) {
    linkLast(e);
}
```

哎哟，又是调用的同一个方法哦，这两方法实现的功能都是一致的，都是往链表的末尾添加元素，区别就在于返回值，前者如果操作成功就返回 `true` ，后者没有返回值。再刨根问底，看看这个操作是怎么样的：

```java
void linkLast(E e) {
    final Node<E> l = last;//获取最后一个元素（这是没有添加前的哈）
    final Node<E> newNode = new Node<>(l, e, null);//构造一个Node，把刚才获取的最后一个元素作为上一个元素，e就是传递过来的元素作为最后一个元素，下元素必然为null
    last = newNode;//last是类中的一个属性，维护的就是最后一个元素，所以此处得修改
    if (l == null)//如果这个链表是空的，那就将这个新的Node赋给first这个属性，表示这是第一个元素
        first = newNode;
    else//否则，那就得去修改原来的最后一个元素，因为我们的最后一个元素变了，原来的最后一个元素的下一个元素在原来是null的，现在得改为新的这个元素了
        l.next = newNode;
    size++;//长度+1
    modCount++;//在ArrayList的源码里面已经提到这个东西了，就是记录器。
}
```

`add()` 方法还有一个重载的方法，它的参数是一个`int` 和一个元素，再来看看：

```java
public void add(int index, E element) {
    checkPositionIndex(index);//检查索引是否合法

    if (index == size)//如果你要在链表的最后插入该元素，则调用上面提到过的linkLast()
        linkLast(element);
    else//否则，在指定的位置插入该元素
        linkBefore(element, node(index));
}
```

这里出现了一个 `linkBefore` ，来看看：

```java
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;//长度+1
    modCount++;//更新记录器
}
```

这个方法跟上面分析的 `linkLast` 大致是一样的，不同的只是插入元素的位置不同罢了，同样都是新造一个 `Node` ，然后修改上一个元素的`next`指向该元素（前提还得判断该元素是否是第一个元素）。

由于 `addFirst()` 跟 `addLast()` 实现的思路也是大致一样的，这里就不过多的废话了。最后来看看 `addAll()` 方法吧，它也有重载方法：

```java
//默认在链表的默认添加
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
//在指定的index处添加元素
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);//检查index是否合法

    Object[] a = c.toArray();//将collection转为Object数组
    int numNew = a.length;//获取数组的长度
    if (numNew == 0)//如果是一个空数组，直接返回false，表示添加失败（你逗我玩呢）
        return false;

    Node<E> pred, succ;
    if (index == size) {//如果插入的位置是链表的末尾，下一个元素就是null，上一个元素就是链表原来的最后一个元素
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }
	//遍历Object数组
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);//构造Node
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

##### ii. 移除方法

在 `LinkedList` 中，提供了5个用于移除元素的方法：`clear()、remove()、removeFirst()、removeLast()、removeLastOccurrence()`。先来看个最简单的：

```java
public void clear() {
    //这是一个简单又粗暴的方法，直接将所有元素以及所有元素的上一个元素、下一个元素的引用全部置为null，最后让GC处理
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

再来看看 `remove()` ，其实这也是重载方法，一共有三个 `remove()`方法，先看一个空参的：

```java
public E remove() {
    //哦豁，从方法的名字上看应该是默认移除链表的第一个元素耶，继续走下去看看猜得对不
    return removeFirst();
}

public E removeFirst() {
    final Node<E> f = first;
    if (f == null)//如果第一个元素为空，则抛出异常，否则调用unlinkFirst(f)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

private E unlinkFirst(Node<E> f) {//注意，这里的参数f是第一个元素的Node哦
    final E element = f.item;//获取元素
    final Node<E> next = f.next;//获取下一个元素的引用
    f.item = null;//将第一个元素和下一个元素的引用置为空，让GC处理
    f.next = null; // help GC
    first = next;//将原来的第二个元素放在首位
    if (next == null)//如果原来的第二位置为空，那么证明这是一个空链表了
        last = null;
    else//否则，将原来第二位置的元素的上一个元素的引用置为null，因为第一个给移除了，现在它就是第一个
        next.prev = null;
    size--;
    modCount++;
    return element;//返回被移除的元素
}
```

因为`remove()`默认删除的就是第一个元素，那么不用猜 `removeFirst()` 所走的逻辑估计也是上面分析的这一套了，所以 `removeFirst()`就不分析了，来看看 `removeLast()` 吧：

```java
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
```

看到这里你估计也会非常熟悉了，跟删除第一个元素的套路是一样的，就连方法名也是类似的，这里也不分析了。来看看另外一个`remove()`吧：

```java
//删除指定的元素o，这里比较有意思的是，会删除链表中所有的o，一看下面的两个循环，迭代条件一直取下一个节点，所以操作的自然就是整个链表的o
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
//上面真正执行删除的方法就是这玩意
E unlink(Node<E> x) {
    //获取这个节点的元素，上一个元素和下一个元素
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {//如果上一个节点为空，那么证明x.item就是第一个元素，那么它的下一个节点就应该赋给first
        first = next;
    } else {//让上一个节点的下一个节点的引用指向下一个节点，并且将这个节点的上一个节点的引用置为null
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {//判断是否为最后一个节点，套路与上面的相似
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
	//把当前节点的元素置为null
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

还有两个移除元素的方法 `removeFirstOccurrence(Object o)` 和 `removeLastOccurrence(Object o)`，分别表示移除第一次出现的元素和最后一次出现的元素o

##### iii. 查找方法

在 `LinkedList` 中提供了5个查找元素的方法：

- `get(int index)` ，返回链表中指定位置的元素
- `indexOf(Object o)`，返回链表中首次出现指定元素的索引，如果没有，则返回-1
- `lastIndexOf(Object o)`，返回链表中最后出现指定元素的索引，如果没有，则返回-1

查找方法就不做源码分析，都是简单的逻辑，有兴趣的自己去刨吧。