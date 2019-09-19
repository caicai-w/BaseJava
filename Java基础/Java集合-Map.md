# Map

## 1.Map类集合

Map是和Collection同级别的接口，但是Map也可以返回一些collection类型的，比如keySet()和value()返回所有key的视图和所有value的视图。

### 1.健值对是否允许空

HashTable                        |           key和value 都不允许为null                                                   
ConcurrentHashMap     |           key和value 都不允许为null       
TreeMap                          |           key不允许为null，value允许
HashMap                         |.          都允许
所以，在你用hashMap转换到conHashMap的时候要注意key有没有null。

### 2.TreeMap

TreeMap中的元素是有序且不重复的，但是，他的去重复和HashMap不同，TreeMap是通过实现comparator的compare方法或者compareable的compareTo()方法来实现的，如果比较后认为两个不相等，就可以存放。而HashMap的去重是通过重写equals和hashCode方法。

### 3.HashMap

HashMap和ConcurrentHashMap性能相差无几，但是hashMap有死链和扩容数据丢失问题。主要原因在于 并发下的Rehash 会造成元素之间会形成一个循环链表。HashMap每次扩容会扩成2倍。

hashMap的基本结构：
table：存放所有节点的数组；
solt：hash槽，也就是table[i]这个位置
bucket：哈希桶，table[i]上所有元素形成的树或者链表。

#### 3.1 以下基于jdk1.7

hashMap的坑：
1.put操作先把长度+1再做addEntry。
2.在createEntry方法中，当算出来要添加的位置之后，不管原本的数据对应下表元素是不是null，直接把新添加的元素放在表头，即便原来是链表，也挂在添加元素的后面，这是元素丢失的原因之一。
3.resize扩容操作，
4.transfer操作：把数据从旧表迁移到新的表，在迁移的过程中，旧表如果仍然可以进行增加操作，如果增加到的slot是新表已经遍历过的，那就白添加了。当resize完成，后续元素可以正常add了，迁移之后，resize线程会赋值给table线程的共享变量，从而覆盖其他线程。这样的话有的线程在add的时候就白add了。

##### 2.1 HashMap丢失对象场景：

 并发赋值时被覆盖；扩容时往已遍历的旧map写数据会丢失；并发扩容的时候有的线程创建的新表会被覆盖。

##### 2.2 死循环

两个线程执行transfer方法，虽然newTable是局部变量，但是原来的table中的Entry是共享的，产生问题的根源在于Entry的next会被并发的修改，可能导致：对象丢失、两个对象互链，对象自己互连。而且1.7这种从头节点就开始操作数据迁移的做法，会让链表数据反转。发生在扩容的时候，假设现在有HashMap，长度为4，有三个节点都挂在同一个table[i]里面，现在有个线程A、B同时触发了扩容操作，假设三个节点rehash之后还是在同一个table[i]，那么，就可能出现死循环。

#### 3.2 以下基于1.8

不想看了，如果我去写：

放入一个元素，判断长度是否足够，如果够的话之间放。

不够的话，先扩容，扩容要做的就是新建一个新的数组，把之前所有元素全都重新hash一遍，然后再把新的元素，加到它要在位置上去，因为1.8有红黑树，所以如果加上之后大于8个了，做个红黑树转换，如果还是链表，就放到链表最后。

#### 3.3  1.7和1.8的比较

1.7中采用数组+链表的组合，1.8中采用数组+链表/红黑树，当链表长度大于8就转成红黑树。
**put方法的不同**
1.8对putVal方法添加元素的分析如下：
①如果定位到的数组位置没有元素 就直接插入。
②如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

1.7对于put方法的分析如下：
①如果定位到的数组位置没有元素 就直接插入。
②如果定位到的数组位置有元素，遍历以这个元素为头结点的链表，依次和插入的key比较，如果key相同就直接覆盖，不同就采用头插法插入元素。

### 4.HashMap和hashTable的区别

1.线程安全： HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过 synchronized 修饰。
2.效率： 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。
3.对Null key 和Null value的支持：hashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。但是在 HashTable不允许。
4.初始容量大小和每次扩充容量大小的不同 ：
 ①创建时如果不指定容量初始值，Hashtable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。

②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小。
5.底层数据结构： JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树。Hashtable 没有这样的机制。

### 5.HashMap和HashSet的区别

HashSet的功能基本都是调用HashMap的代码实现的。

```java
public HashSet() {
        map = new HashMap<>();
    }
```

可以看到HashSet都是直接new了个HashMap出来，它的add操作也是调用的HashMap的put()。

Map存储键值对，Set存储对象，HashMap调用put()添加键值对，HashSet调用add()添加对象，HashMap使用Key计算hashCode，根据hashCode计算桶的位置。HashSet用对象计算HashCode，如果没有和他相等的HashCode会默认此时Set里面没有重复的元素，如果有相等的hashCode，再调用equals方法判断，如果此时也相等就不会添加进去，如果不相等那就会在同一个地方形成链表。所以在重写这两个方法的时候应该尽量让equals相等的时候，hashCode也相等。equals不想等的时候hashCode也不想等。

他们都不是线程安全的。

### 6.HashSet如何去重

你把对象加入HashSe，HashSet会先计算对象的hashcode值来判断对象加入的位置，如果没有相等的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

**hashCode（）与equals（）的相关规定：**

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个equals方法返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。



### 7.HashMap的底层实现

### JDK1.8之前

JDK1.8 之前 HashMap底层是 **数组和链表** 结合在一起使用也就是 链表散列。HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。**

### JDK1.8之后

相比于之前的版本， JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。

### 8.HashMap 的长度为什么是2的幂次方

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648到2147483647，前后加起来大概40亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n代表数组长度）。这也就解释了 HashMap 的长度为什么是2的幂次方。

**这个算法应该如何设计呢？**

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。”** 并且 采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方。

### 9.ConcurrentHashMap和HashTable的实现

ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：** JDK1.7的 ConcurrentHashMap 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树，用CAS和Synchronized实现的。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
- **实现线程安全的方式（重要）：** ① **在JDK1.7的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② **Hashtable(同一把锁)** :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

### 10.ConcurrentHashMap是如何保证线程安全的？

JDK7中使用的是分段锁，内部分成16个Segment即分段，每个分段可以看作是一个小型的HashMap，每次put只会锁定一个分段，降低了锁的粒度：

首先根据key计算出一个hash值，找到对应的Segment。
调用Segment的lock方法（Segment继承了重入锁），锁住该段内的数据，所以并没有锁住ConcurrentHashMap的全部数据。
根据key计算出hash值，找到Segment中数组中对应下标的链表，并将该数据放置到该链表中。
判断当前Segment包含元素的数量大于阈值，则Segment进行扩容。（Segment的个数是不能扩容的，但是单个Segment里面的数组是可以扩容的）
多线程put的时候，只要被加入的键值不属于同一个分段，就可以做到真正的并行put。对不同的Segment则无需考虑线程同步，对于同一个Segment的操作才需考虑。

JDK8中使用CAS+synchronized保证线程安全，也采取了数组+链表/红黑树的结构。put时使用synchronized锁住桶中链表的头结点。当某个链表长度大于8且table容量大于64转红黑，当某个solt内的元素个数小于6的时候，再从红黑树转成链表（折腾啊）。所以当table容量小于64的时候，只会扩容，不会把链表转成红黑树。再转化的过程中，用Syn锁住table[i],转换完成后用CAS替换原有的链表，

CAS有三个值：内存位置、预期值、新的值，只有内存位置的值和预期的一样，才会做修改。
