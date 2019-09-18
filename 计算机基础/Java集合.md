### 说说List、Set、Map的区别

- List是提供一个有序、允许重复的对象，你放进去什么，就按照什么顺序给你排列起来。
- Set是一个无序保证不重复的集合，你放进去什么。
- Map，是以健值对的形式存在的，Map会维护和Key有关联的值，Key不可以重复，但value可以重复。

### 集合框架的底层数据结构

### Collection

#### 1. List

- **Arraylist：** Object数组
- **Vector：** Object数组
- **LinkedList：** 双向链表(JDK1.6之前为循环链表，JDK1.7取消了循环)

#### 2. Set

- **HashSet（无序，唯一）:** 基于 HashMap 实现的，底层采用 HashMap 来保存元素
- **LinkedHashSet：** LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 HashMap 实现一样，不过还是有一点点区别的
- **TreeSet（有序，唯一）：** 红黑树(自平衡的排序二叉树)

### Map

- **HashMap：** JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间
- **LinkedHashMap：** LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931)
- **Hashtable：** 数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
- **TreeMap：** 红黑树（自平衡的排序二叉树）

### 如何选用集合?

主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用Map接口下的集合，需要排序时选择TreeMap,不需要排序时就选择HashMap,需要保证线程安全就选用ConcurrentHashMap.当我们只需要存放元素值时，就选择实现Collection接口的集合，需要保证元素唯一时选择实现Set接口的集合比如TreeSet或HashSet，不需要就选择实现List接口的比如ArrayList或LinkedList，然后再根据实现这些接口的集合的特点来选用。

### ArrayList和LinkedList区别？

从四个方面回答：

1.ArrayList内部是一个Object数组，LinkedList是一个双向链表的结构。

2.两个都不是线程安全的。

3.LinkedList不支持高效的随机元素访问，但ArrayList支持快速随机访问，但是ArrayList插入效率不如LinkedList高。

### ArrayList和Vector有什么区别呢？什么要用ArrayList取代Vector？

Vector类里面的是同步方法，所以效率比较低。建议在不要求线程安全的时候就不要使用Vector

### 说一说ArrayList的扩容机制吧

先不说扩容，先说其他的吧，以一个无参数方法创建ArrayList时，是创建了个空数组，真正对数据进行添加元素操作时，才真正分配容量，也就是向数组添加第一个元素的时候，数组容量变成10，下面就以无参数构造创建的ArrayList为例开始。

```java
    public boolean add(E e) {
      //1.添加元素之前，先调个ensureCapacityInternal
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

```java
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    //2.判断现在调用数组的这个elementData，是不是无参数构造出来那个，是的话就数组长度就返回10
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

```java
   private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        //3.判断是否需要扩容
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

好了，等到加到第11个元素的时候，要扩容了。

```java
  private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //1.新长度开成1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

  private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //对minCapacity和MAX_ARRAY_SIZE进行比较
        //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
        //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



### ensureCapacity()方法

```java
//没太明白想干什么，应该就是增大容量吧，这样add的时候少扩容几次，执行快一点。
public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```



### Arrays.copyOf（）和System.arraycopy()

刚才那个扩容，最后调用的就有这个方法，那么他们有什么区别？

看两者源代码可以发现 copyOf() 内部实际调用了 `System.arraycopy()` 方法，arraycopy()需要目标数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 `copyOf()` 是系统自动在内部新建一个数组，并返回该数组。

### HashMap和hashTable的区别

1.线程安全： HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过 synchronized 修饰。
2.效率： 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。
3.对Null key 和Null value的支持：hashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。但是在 HashTable不允许。
4.初始容量大小和每次扩充容量大小的不同 ：
 ①创建时如果不指定容量初始值，Hashtable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。

②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小。
5.底层数据结构： JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树。Hashtable 没有这样的机制。

### HashMap和HashSet的区别：

HashSet的功能基本都是调用HashMap的代码实现的。

Map存储键值对，Set存储对象，HashMap调用put()添加键值对，HashSet调用add()添加对象，HashMap使用Key计算hashCode，根据hashCode计算桶的位置。HashSet用对象计算HashCode，如果没有和他相等的HashCode会默认此时Set里面没有重复的元素，如果有相等的hashCode，再调用equals方法判断，如果此时也相等就不会添加进去，如果不相等那就会在同一个地方形成链表。所以在重写这两个方法的时候应该尽量让equals相等的时候，hashCode也相等。equals不想等的时候hashCode也不想等。

他们都不是线程安全的。

### HashSet如何去重

你把对象加入HashSe，HashSet会先计算对象的hashcode值来判断对象加入的位置，如果没有相等的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

**hashCode（）与equals（）的相关规定：**

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个equals方法返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。



### HashMap的底层实现

### JDK1.8之前

JDK1.8 之前 HashMap底层是 **数组和链表** 结合在一起使用也就是 链表散列。HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。**

### JDK1.8之后

相比于之前的版本， JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。

### HashMap 的长度为什么是2的幂次方

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648到2147483647，前后加起来大概40亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n代表数组长度）。这也就解释了 HashMap 的长度为什么是2的幂次方。

**这个算法应该如何设计呢？**

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。”** 并且 采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方。

### HashMap 多线程操作导致死循环问题

主要原因在于 并发下的Rehash 会造成元素之间会形成一个循环链表。不过，jdk 1.8 后解决了这个问题，但是还是不建议在多线程下使用 HashMap,因为多线程下使用 HashMap 还是会存在其他问题比如数据丢失。并发环境下推荐使用 ConcurrentHashMap 。

### ConcurrentHashMap和HashTable的实现

ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：** JDK1.7的 ConcurrentHashMap 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树，用CAS和Synchronized实现的。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
- **实现线程安全的方式（重要）：** ① **在JDK1.7的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② **Hashtable(同一把锁)** :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

### ConcurrentHashMap是如何保证线程安全的？

JDK7中使用的是分段锁，内部分成16个Segment即分段，每个分段可以看作是一个小型的HashMap，每次put只会锁定一个分段，降低了锁的粒度：

首先根据key计算出一个hash值，找到对应的Segment。
调用Segment的lock方法（Segment继承了重入锁），锁住该段内的数据，所以并没有锁住ConcurrentHashMap的全部数据。
根据key计算出hash值，找到Segment中数组中对应下标的链表，并将该数据放置到该链表中。
判断当前Segment包含元素的数量大于阈值，则Segment进行扩容。（Segment的个数是不能扩容的，但是单个Segment里面的数组是可以扩容的）
多线程put的时候，只要被加入的键值不属于同一个分段，就可以做到真正的并行put。对不同的Segment则无需考虑线程同步，对于同一个Segment的操作才需考虑。

JDK8中使用CAS+synchronized保证线程安全，也采取了数组+链表/红黑树的结构。put时使用synchronized锁住桶中链表的头结点。




https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98.md#%E5%89%96%E6%9E%90%E9%9D%A2%E8%AF%95%E6%9C%80%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E4%B9%8Bjava%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86
