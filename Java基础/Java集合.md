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

- **LinkedHashSet：** LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。

  ```java
      public LinkedHashSet() {
          super(16, .75f, true);
      }
      //进入super
      HashSet(int initialCapacity, float loadFactor, boolean dummy) {
          map = new LinkedHashMap<>(initialCapacity, loadFactor);
      }
  
  ```

  这个就很奇怪，LinkedHashSet继承HashSet，然后HashSet又是用LinkedHashMap实现的。

- **TreeSet（有序，唯一）：** 它是用TreeMap来实现的，底层红黑树(自平衡的排序二叉树)，虽然TreeSet也能去重，但和HashSet的去重方式还是不一样的，HashSet是通过计算hashCode和equals来比较，但是TreeSet是通过compareTo().

### Queue

队列是一种先进先出的数据结构，队列是一种特殊的线性表，自从BlockingQueue诞生以来，在各种高并发场景中，由于其本身FIFO的特性和阻塞操作的特点，经常被用作Buffer。

### Map

- **HashMap：** JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间
- **LinkedHashMap：** LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931)
- **Hashtable：** 数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
- **TreeMap：** 底层红黑树（自平衡的排序二叉树）

### 如何选用集合?

主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用Map接口下的集合，需要排序时选择TreeMap,不需要排序时就选择HashMap,需要保证线程安全就选用ConcurrentHashMap.当我们只需要存放元素值时，就选择实现Collection接口的集合，需要保证元素唯一时选择实现Set接口的集合比如TreeSet或HashSet，不需要就选择实现List接口的比如ArrayList或LinkedList，然后再根据实现这些接口的集合的特点来选用。

### 集合初始化



### ArrayList和LinkedList区别？

从4个方面回答：

1.ArrayList内部是一个Object数组，LinkedList是一个双向链表的结构,它可以向队头或者队尾插入元素。

2.两个都不是线程安全的。

3.LinkedList不支持高效的随机元素访问，但ArrayList支持快速随机访问，但是ArrayList插入效率不如LinkedList高。

4.LinkedList因为继承了Deque接口，还具有栈和队列的性质。

### ArrayList和Vector有什么区别呢？什么要用ArrayList取代Vector？

Vector类里面的是同步方法，所以效率比较低。建议在不要求线程安全的时候就不要使用Vector

### 说一说ArrayList的扩容机制吧

先不说扩容，先说其他的吧，以一个无参数方法创建ArrayList时，是指向一个空数组，真正对数据进行添加元素操作时，才真正分配容量，也就是向数组添加第一个元素的时候，就直接扩容把数组容量变成10，下面就以无参数构造创建的ArrayList为例开始。

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
        //3.判断是否需要扩容，第一次添加元素就直接扩容了。
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

好了，等到加到第11个元素的时候，又要扩容了。

```java
  private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //1.新长度开成1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       //2.如果新长度比数组定义的最大长度还大，就用当前长度和MAX_ARRAY_SIZE之间做个比较
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



### HashMap的扩容和初始化

如果一个hashMap你要放入1000的元素，但是没有设置初始容量大小，随着元素的加入，需要被动扩容7次才可以完成存储，扩容每次重建hash表，非常影响性能。HashMap中比较影响性能的参数：Capacity，和Load factor，Capacity是决定了存储容量，默认16，基于这两个数字的乘积，HashMap内部使用threshold变量表示hashmap中能放入的元素个数。同样，hashMap也不会在new的时候分配内存，而是在第一次put的时候分配。

如果在初始化的时候制定了初始化大小，那么会先计算出比initialCapacity大的2的幂存入，此后每次扩容都是乘2倍。

### Arrays.copyOf（）和System.arraycopy()

刚才那个扩容，最后调用的就有这个方法，那么他们有什么区别？

看两者源代码可以发现 copyOf() 内部实际调用了 `System.arraycopy()` 方法，arraycopy()需要目标数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 `copyOf()` 是系统自动在内部新建一个数组，并返回该数组。

### 数组转集合 集合转数组

在数组转集合的过程中，要注意是否使用了视图方式直接返回数组中的元素。

```java
public class ArrayTest {
    public static void main(String[] args) {
        String[] str = new String[3];
        str[0]="aaa";
        str[1]="bbb";
        str[2]="ccc";
        //数组转List
        List<String> stringList = Arrays.asList(str);
        stringList.set(0,"newAAA");
        System.out.println(stringList.get(0));//newAAA
        System.out.println(str[0]);           //newAAA
        /**
         * 从上面的转换可以发现，你用Arrays.asList(str)返回一个数组之后，是允许修改其中的值的，并且原数组的值也会跟着改
         * 但是！
         * 下面这三句虽然可以编译通过，但都抛出UnsupportedOperationException。
         * Arrays.asList(str)体现的是适配器模式，后台的数据仍然是数组，set()方法可以间接的对数组进行值的修改。
         * asList()返回的对象是一个Arrays的内部类，它没有实现集合个数的相关修改方法。
         */
        stringList.add("ddd");
        stringList.remove(2);
        //remove所有元素，执行完之后就是空List了
        stringList.clear();
    }
}
```

但是很奇怪啊，那不就是ArrayList对象么，为什么不能改呢？

因为这个ArrayList是Arrays自己定义的一个内部类，根本不是你想的那个！下面就是它的实现，你看看和真的没法比。

```java
   private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        private static final long serialVersionUID = -2764017481108945198L;
        //final修饰引用不能改，你传进来什么就一直指向这个
        private final E[] a;

        ArrayList(E[] array) {
            a = Objects.requireNonNull(array);
        }
       // 省略，实现了set修改某个位置数组的元素
```

就是因为这个ArrayList没有实现，修改元素个数的相关方法，所以会报错，虽然它要是覆盖AbstractList就不会出错，但它就是不覆盖。那么，想要数组转集合，正确的方式应该是这样的：

```java
List<String> objectList  = new ArrayList<>(Arrays.asList(str));
```

那么，集合转数组会稍微容易点，毕竟是从个自由的转到一个苛刻的，一般什么情况下要去转成数组呢？适配别人的接口，或者进行局部方法计算。

```java
 public static void main(String[] args) {
        List<String> list = new ArrayList<>(3);
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");
        //范型丢失，String[]是不能接收toArray的返回结果的，所以一般不要用toArray().
        Object[] array1 = list.toArray();
        System.out.println(array1[0]);  //aaa

        /**
         * 这样是可以成功转的
         * 但是如果你数组给开2，也就是数组空间不够，他会给你重新建一个返回给你，但要注意用的不是你传入的那个数组了。
         */
        String[] str = new String[3];
        list.toArray(str);
    }
```

### 
