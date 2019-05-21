##  ArrayList

---

### 一、概述

`ArrayList` 是实现 `List` 接口的有序的，元素可重复的动态数组。每个`ArrayList`实例都有一个容量，该容量是指用来存储列表元素的数组的大小。默认初始容量为10。随着ArrayList中元素的增加，它的容量也会不断的自动增长。在每次添加新的元素时，`ArrayList`都会检查是否需要进行扩容操作，扩容操作带来数据向新数组的重新拷贝，所以如果我们知道具体业务数据量，在构造`ArrayList`时可以给ArrayList指定一个初始容量，这样就会减少扩容时数据的拷贝问题。当然在添加大量元素前，应用程序也可以使用`ensureCapacity`操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。

注意，`ArrayList` 实现不是同步的。如果多个线程同时访问一个 `ArrayList` 实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。所以为了保证同步，最好的办法是在创建时完成，以防止以外对列表进行不同步的访问：

```java
List list = Collections.synchronizedList(new ArrayList(...));
```

---

### 二、ArrayList 源码分析

#### 1. ArrayList 的继承与实现关系

先上代码：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

- 继承了 `AbstractList`，实现了`List` ，提供了相关的增删改查等功能。
- 实现 `RandomAccess` 接口，提供了随机访问功能，在 `ArrayList` 中，我们可以通过元素的序号快速获取元素，这就是快速随机访问。
- 实现 `Cloneable` 接口，就是覆盖了 `clone()` 方法，能被克隆。
- 实现 `java.io.Serializable` 接口，支持序列化，能通过序列化进行传输。

#### 2. ArrayList 主要的属性

```java
//定义了初始化数组的容量
private static final int DEFAULT_CAPACITY = 10;
//这个就是存储数据的动态数组
transient Object[] elementData;
//记录数组的长度
private int size;
//一个空数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//空数组
private static final Object[] EMPTY_ELEMENTDATA = {};
```

在上面出现了一个比较陌生的关键字 `transient` ，在日常开发过程中确实很少使用，在第一次看到这个关键字的时候也蒙圈了。其实，`transient` 不是特别复杂的东西，就一句话来解析：一个类的某些属性需要序列化，而其他属性不需要被序列化（比如一些用户的敏感信息包括密码银行卡号等等），为了安全起见，不希望在网络操作中被传输，所以就加上这个关键字。换句话说，如果一个属性被 `transient`修饰，那么这个属性的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

#### 3. 构造方法

`ArrayList` 提供了三个构造方法：

```java
//空参构造器，初始化容器设置为空数组
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
//参数为初始化容量
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {//如果初始化容量合法（正数），那么就使用这个指定的容量初始化一个Object[]
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {//如果为0，则使用一个空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {//如果为负数，则抛异常
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
//参数为一个Collection
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();//转为数组，并赋值给动态数组的引用变量
    if ((size = elementData.length) != 0) {
        // 如果类型不是Object[]，则转换为Object[]
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 如果传递进来的数组是空的，那么直接使用咱们在之前定义的一个空数组
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

#### 4. 常用方法

##### i. 添加方法

在 `ArrayList` 中，提供了5个增加元素的方法：

1. `boolean add(E e)`
2. `void add(int index, E element)`
3. `boolean addAll(Collection<? extends E> c)`
4. `boolean addAll(int index, Collection<? extends E> c)`
5. `E set(int index, E element)`

先来看第一个：

```java
public boolean add(E e) {
    //增加之前先检查容量是否满足
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;//表示插入成功
}
```

作用：将元素 e 添加到容器的末尾。`ensureCapacityInternal`方法用于检查容器的容量，上源码分析：

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);//使用初始化容量10和参数minCapacity的较大值作为容量
    }
	//实际检查容量的方法
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    //记录 ArrayList 结构发生变化的次数，这个属性定义在父类中。因为每次进行add()或addAll()时都需要调用ensureExplicitCapacity()，因此直接在ensureExplicitCapacity()中对modCount进行修改
    modCount++;

    if (minCapacity - elementData.length > 0)
        //扩大容器容量的方法
        grow(minCapacity);
}

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    //// 新容量是旧容量的1.5倍,即扩容 50%
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

到这里终于完成了扩增容量，接下来的增加方法都是大同小异了：

```java
public void add(int index, E element) {
    //检查传递过来的index是否合法
    rangeCheckForAdd(index);
	//跑的也是上面提到那套逻辑，判断容量，扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //将目标元素插入到index位置后，之后的元素都会被整体后移一个单位长度，会涉及到整体复制，性能消耗大，ArrayList尽量不要使用该操作，如果需要，则可以考虑使用 LinkedList
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

作用：将指定的元素插入此列表中的 `index` 位置。

```java
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)//如果大于数组的长度或小于0，则抛出异常
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

接下来第三个：

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();//转数组
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // 扩容
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;//表示插入成功
}
```

作用：按照指定的 `Collection` 的迭代器所返回的元素顺序，将该集合的所有元素添加到此列表的末尾。

同样的，`addAll()`也可以在指定位置插入，性能的问题还是存在的，所以这里就不再阐述该方法了。

最后一个：

```java
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

作用：用指定的元素 `element` 替代列表中 `index` 位置上的元素，返回值是被替代的值！！

##### ii. 删除方法

在 `ArrayList` 中提供了 `remove(int index)、remove(Object o)、removeRange(int fromIndex, int toIndex)、clear()` 四个方法用于删除元素。

老规矩，一个个来，第一个：

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

**作用：删除指定位置 `index` 的元素，并且将这个元素返回。**



```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);//移除元素的位置
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

**作用：删除指定的元素（如果有重复的，则删除首次出现的）**



```java
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // clear to let GC do its work
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}
```

**作用：删除一个区间的元素。**



```java
public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

**作用：删除所有元素。**



##### iii. 查询方法

常用的就一个 `get`

```java
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
```

根据索引查询一个元素。

---

### 三、ArrayList 高级特性

#### 1. 线程不安全问题

`ArrayList` 是线程不安全的，比如以添加方法为例：

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

`ArrayList` 在添加一个元素的时候，它可能会有两步来完成：

1. 在 `elementDate[size]` 的位置上存放此元素
2. 增大 `size` 的值

其本质上来讲是 `size++` 不是一个原子操作导致的线程安全问题，在单线程环境下，如果 `size=0` ，添加一个元素后，此元素在位置0，而且 `size=1` ；但是在多线程环境下，比如两个线程，A线程先将元素存放在位置0，但是此时 `CPU` 调度线程A暂停，线程B得到运行的机会，线程B也向此列表添加元素，因为此时 `size` 依然为0，所以线程 B也将元素存放到位置0，最后，两个线程都继续运行，都增加了`size`的值。

解决方案：

- 使用同步代码块 `synchronized` 
- 使用 `Collections.synchronizedList(new ArrayList())`
- 使用 `JUC` 中的 `CopyOnWriteArrayList`



#### 2. fail-fast 问题

`fail-fast` 机制是 `java` 集合中的一种错误机制，当多个线程对同一个集合的内容进行操作时，就可能会产生 `fail-fast` 事件。例如：当线程A通过`iterator`去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出`ConcurrentModificationException` ，产生 `fail-fast` 事件。先看示例：

```java
public static void main(String[] args) {
    ArrayList<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(3);
    list.add(4);
    for (Integer integer : list) {
        if (integer == 2) {
            list.remove(integer);
        }
        System.out.println(integer);
    }
}
//运行结果
Exception in thread "main" java.util.ConcurrentModificationException
```

奇怪了，为什么在遍历`list`的时候删除元素会报异常呢？

首先得明确，`foreach` 本质上就是迭代器遍历集合元素，因此必须从迭代器的角度去寻找问题。先说明这个问题前，请回忆一下在前面分析 `ArrayList` 源码的时候是不是在类中维护着一个属性 `modCount`，而且在增加或删除的时候都会修改这个值，它是记录改变的次数的。

再来看看 `ArrayList` 中的 `iterator()` 源码：

```java
//返回List对应迭代器，实际上是返回一个Itr对象。Itr是ArrayList的一个内部类
public Iterator<E> iterator() {
    return new Itr();
}
//迭代器的实现类
private class Itr implements Iterator<E> {
    int cursor;       // 代表下一个要访问的元素下标
    int lastRet = -1; // 上一个要访问的元素下标
    //记录修改次数，每次创建Itr对象时，都会保存新建该对象时对应的Count，以后每次遍历List中的元素时，都会比较expectedModCount和modCount是否相等，如果不相等，则抛出ConcurrentModificationException异常，产生fail-fast
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    //迭代器的next()方法获取下一个元素
    @SuppressWarnings("unchecked")
    public E next() {
        //获取下一个元素之前，先判断新建的Itr对象时保存的modCount和当前的modCount是否相等，不相等则抛异常
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }
	//迭代器的删除
    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }
	//检查modCount和expectedModCount的值是否相同，如果不同，则直接抛出异常
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

从上面可以发现，迭代器的`next()` 和 `remove()`都会执行 `checkForComodification()`，如果 `modCount` 与 `expectedModCount` 不相等，则抛出 `ConcurrentModificationException` 异常。

总结：在使用迭代器遍历元素集合的时候，每次调用 `next()` 方法都会检查 `modCount` 和 `expectedModCount` 的值是否相同，如果在遍历的同时使用 `list` 的 `remove()`方法去删除元素，则会修改`modCount`的值，此时两者就不同，迭代器遍历一下个元素时就会察觉到这两个值不同了，从而抛出异常。

#### 3. fail-fast 原理

其实说白了就是一句话：`modCount` 和 `exceptedModCount` 在同一时刻出现了不同的情况，从而产生的异常。

在 `ArrayList` 中，无论是 `add()、remove()、clear()` ，只要涉及到修改集合中的元素个数时，都会改变 `modCount` 的值。下面再来梳理 `fail-fast` 是如何产生的：

1. 创建一个 `ArrayList`
2. 向集合中添加元素
3. 通过 `Iterator` 读取集合的值
4. 删除集合中的某个节点

当多个线程对同一个集合进行操作的时候，某线程访问集合的过程中，该集合的内容被其他线程所改变(即其它线程通过`add、remove、clear`等方法，改变了`modCount`的值)；这时，就会抛出`ConcurrentModificationException`异常，产生`fail-fast`事件。

#### 4. fail-fast 的解决办法

方案一：直接使用同步的集合 `Collections.synchronizedList` ，但是同步锁有可能会阻塞遍历操作，不推荐。

方案二：使用 `CopyOnWriteArrayList` 。推荐使用

`CopyOnWriteArrayList`  是一个线程安全的 `ArrayList`，其中所有的可变操作（`add、set`等）都是通过对底层数组进行一次新的复制来实现的。该类产生的开销其实是比较大的，但是在两种情况下，它适合使用。

1. 在不能或不想同步遍历，但又需要从并发线程中排除冲突时。
2. 当遍历操作数量大大超过可变操作的数量时

`CopyOnWriteArrayList` 的核心概念是：任何对 `array` 在结构上有所改变的操作(`add、remove、clear`等)，都会 `copy` 现有的数据，在 `copy` 的数据上修改。 