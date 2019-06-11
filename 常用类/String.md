## String

---

### 一、String 的定义

​	在 `Java` 中，其实字符串并不是内置的数据类型，而是通过一个类 `String` 来表示字符串。先来看 `String` 类的定义：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
```

从上面注意到，`String` 是被定义为 `final` 的，也就是不能被继承的。实现了几个常见的接口这里就不再细说了。定义完了接着就是里面的属性了：

```java
//哦豁，原来字符串就是一个字符数组哦
private final char value[];
//用于计算哈希值
private int hash;
```

完了接下来该构造器上场了，大概数了一下 `String` 提供了16个构造器，其中有两个已经废弃，也就是还有14个：

```java
//空参构造器，也就是说直接new String() 得到的是一个空串""
public String() {
    this.value = "".value;
}
//参数为一个字符串对象
public String(String original) {
    this.value = original.value;//将参数字符串对象维护的字符数组value赋值给这个新的字符串
    this.hash = original.hash;//将参数字符串对象的hash值赋值给这个新的字符串
}
//参数为一个字符数组
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
//参数为字符数组，偏移量，复制的数量
public String(char value[], int offset, int count) {
    if (offset < 0) {//偏移量小于0抛出异常
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {//复制数量小于0抛出异常
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset <= value.length) {
            this.value = "".value;
            return;
        }
    }
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
    //复制数组并赋值给value
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

哎~，这里就省略了剩下的构造器，原因就是很少直接去使用这些构造器来创建一个字符串，咱不是直接使用""写就好了么。

---

### 二、String 常用的方法

#### 1. 字符串比较

- `equals()` —— 判断内容是否相同。注意这里说的是内容！由于 `Java` 的字符串不是基本类型，而是一个对象，所以两个字符串之间的对比其实就是在拿两对象进行比较，那么问题来了，两个对象是如何比较的呢？可以通过 == 去比较吗？答案是，不太合适！！因为 == 比较的是两个对象的内存地址是否一致，这完全有可能会出现这样的结果：两个字符串的内容是一致的，但是通过运算符 == 比较的结果却是 `false` ，这往往不是我们想要看到的，咱们如果要拿两个对象来比较，那么肯定考虑的就是里面的内容是否一致，至于是否是同一个内存地址则没有要求。所以就有了 `equals()` 这个方法：

```java
public boolean equals(Object anObject) {
    //首先上来还是使用 == 来比较，因为如果两个对象的引用地址是一致的，那么这两个变量其实就是引用着方法区中的同一个区域，那么里面必然是一致的！！
    if (this == anObject) {
        return true;
    }
    //接着通过instanceof运算符检查这个参数是否为String的实例，如果不是，那么不用比较了，直接返回false
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;//字符串长度
        if (n == anotherString.value.length) {//如果长度不相等，则跳出if语句返回false
            char v1[] = value;//字符数组
            char v2[] = anotherString.value;//参数的字符数组
            //遍历，逐一字符比较，如果出现某一个位置上的字符是不相同的，则返回false，否则返回true
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

补充：一般来说通过 “” 方式来使用字符串，但是你可能会说 `String` 是一个类啊，不用`new`出一个实例来使用吗？其实，在 `Java` 虚拟机中存在着一个`字符串常量池`，使用一个字符串会先去常量池中检索是否存在，若存在则直接饮用，若不存在则先添加到常量池中，再返回饮用。但是通过 `new` 关键字创建出来的字符串都拥有独立的内存空间，它不会再去常量池中引用，例如：

```java
String str1 = new String("jonas");
String str2 = new String("jonas");
String str3 = "jonas";
String str4 = "jonas";
System.out.println(str1 == str2);//false  --> 证明str1与str2不是引用着同一块内存地址,尽管字符串内容是一致的，但是new出来两个对象了
System.out.println(str3 == str4);//true --> "" 使用的是常量池里面的引用，所以两者的引用是同一块内存地址
System.out.println(str1 == str3);//false  --> 常量池中与new出来的是独立的
```

- `compareTo()` —— 判断字符串的大小关系。

```java
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);//字符串长度的较短者
    char v1[] = value;
    char v2[] = anotherString.value;
	//从0开始到最短那个字符串的长度的位置，逐一比较对应位置上的字符是否一致，如果出现不一致，则返回他们字符编码的差值。
    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}
```

- `equalsIgnoreCase()` —— 与 `equals()` 基本一致，但是忽略大小写。

- `compareToIgnoreCase()` —— 与 `compareTo()`基本一致，但是忽略大小写。

#### 2. 字符串查找

- `charAt(index)` —— 返回指定索引位置的字符

```java
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {//参数小于0或大于字符串长度则抛出异常
        throw new StringIndexOutOfBoundsException(index);
    }
    //返回字符数组value中的index位置的值
    return value[index];
}
```

- `indexOf(String str)` —— 从索引为0开始检索，返回第一次出现参数str的位置的索引。

```java
public int indexOf(String str) {
    return indexOf(str, 0);//调用了indexOf(String str, int fromIndex)方法，而且开始索引默认为0(也就是从0位置开始查找)
}
public int indexOf(String str, int fromIndex) {
    return indexOf(value, 0, value.length,
                   str.value, 0, str.value.length, fromIndex);//又调其他方法了~~
}
//终于找到你了吧，原来几乎所有的indexOf都调用了该方法，所以就来分析这个东西就完事了。
static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
           	int fromIndex) {
    if (fromIndex >= sourceCount) {
        return (targetCount == 0 ? sourceCount : -1);
    }
    if (fromIndex < 0) {
        fromIndex = 0;
    }
    if (targetCount == 0) {
        return fromIndex;
    }

    char first = target[targetOffset];
    int max = sourceOffset + (sourceCount - targetCount);

    for (int i = sourceOffset + fromIndex; i <= max; i++) {
        /* Look for first character. */
        if (source[i] != first) {
            while (++i <= max && source[i] != first);
        }

        /* Found first character, now look at the rest of v2 */
        if (i <= max) {
            int j = i + 1;
            int end = j + targetCount - 1;
            for (int k = targetOffset + 1; j < end && source[j]
                 == target[k]; j++, k++);

            if (j == end) {
                /* Found whole string. */
                return i - sourceOffset;
            }
        }
    }
    return -1;
}
```

- `starsWith(String str)` —— 是否以 str 开头，返回布尔值
- `endsWith(String str)` —— 是否以 str 结尾，返回布尔值

#### 3. 字符串截取

- `subString()` —— 截取字符串，有两个重载方法，一个参数的时候表示开始索引开始截取到最后；两个参数时则表示截取这个区间，返回值是一个新的字符串，而不是在原来的字符串上修改，字符串具有不可变性！！就拿其中的一个为例：

```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);//最后此处是直接new了一个String，所以这是新的对象！
}
```

#### 4. 字符串替换

- `replace(char oldChar, char newChar)` —— 替换字符，结果还是会new一个新的String
- `replaceAll(String regex,String replacement)` —— 第一个参数是正则表达式。

---

### 三、StringBuffer

​	在上面就提到过 `String` 具有不可变性，意思就是你不能在原来的字符串上面进行修改操作，上面操作字符串的方法都会新建一个实例。然而，`StringBuffer` 却相反了，若进行修改操作，它并不会产生新的字符串，而是在原字符串上进行修改。在 `StringBuffer`中维护着一个字符数组：

```java
private transient char[] toStringCache;//默认初识容量为16
```

如果需要使用 `StringBuffer` 就不能简单的“”来表示了，必须手动new实例了。里面的方法大致分为三类（因为很多重载方法，就不详细说了）：

- `append()` —— 追加内容
- `insert()` —— 插入内容
- `delete()` —— 移除内容

---

### 四、StringBuilder

`StringBuilder` 也是一个可变的字符串，他与 `StringBuffer` 不同之处就在于它是线程不安全的，基于这点，它的速度一般都会比 `StringBuffer` 快。