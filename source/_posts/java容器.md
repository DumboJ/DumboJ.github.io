---
title: java容器
date: 2019-08-09 00:59:11
tags:
 -容器
 -list
 -set
 -map
categories:
 -list
 -set
 -map
cover: static/bgpic/javaCollection.jpg
---

### ArrayList

##### 1.概览：ArrayList基于数组实现，RandomAccess接口标识该类支持快速随机访问。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

数组默认初始大小为10

```java
private static final int DEFAULT_CAPACITY = 10;
```

##### 2.扩容：

添加元素时使用ensureCapacityInternal（）方法来保证容量足够，如果不够时，需要使用grow（）方法扩容，新容量的大小为旧容量的1.5倍。

扩容操作时需要调用Arrays.copyof()把原数组复制到新数组中，这个操作代价很高。因此最好在创建时ArrayList对象时就指定大概的容量大小，减少扩容操作的次数。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

##### 3.删除元素

删除元素时需要调用System.arraycopy()将index+1后面的元素都复制到index上，该操作的时间复杂度为O(N),可以看出ArrayList删除元素的代价也很高。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

##### 4.Fail-Fast

modeCount用来记录ArrayList结构发生变化的次数。结构发生变化是指ArrayList添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较前后的modeCount是否改变，改变了需要抛出ConcurrentModificationException

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

##### 5.序列化

ArrayList基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，没必要全部进行序列化。

保存元素的数组elementData使用transient修饰，该关键字声明数组默认不会被序列化。

```java
transient Object[] elementData; // non-private to simplify nested class access	
```

ArrayList实现了writteObject()和readObject来控制只序列化数组中有元素填充那部分内容。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

### 2.Vector

它的实现与ArrayList类似，但是它使用synchronized进行同步。

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

#### 与ArrayList比较

- Vector是同步的，因此开销比ArrayList更大，访问速度更慢。最好使用ArrayList而不是Vector，因为同步操作完全可以由程序员自己来控制。
- Vector每次扩容请求其大小的两倍（也可以通过构造函数来设置），ArrayList是1.5倍

##### 替代方案

1. ##### 使用Collections.synchroniedList();得到一个线程安全的ArrayList

   ```java
   List<String> list = new ArrayList<>();
   List<String> synList = Collections.synchronizedList(list);
   ```

2. ##### 并发包下的CopyOnWriteArrayList类

   ```java
   List<String> list = new CopyOnWriteArrayList<>();
   ```

   ###### 读写分离：

   写操作在复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

   写操作需要加锁，防止并发写入时导致写入数据丢失。

   写操作结束后，需要把原始数组指向新的复制数组。

   ```java
   public boolean add(E e) {
       final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           Object[] elements = getArray();
           int len = elements.length;
           Object[] newElements = Arrays.copyOf(elements, len + 1);
           newElements[len] = e;
           setArray(newElements);
           return true;
       } finally {
           lock.unlock();
       }
   }
   
   final void setArray(Object[] a) {
       array = a;
   }
   ```

   

   ```java
   @SuppressWarnings("unchecked")
   private E get(Object[] a, int index) {
       return (E) a[index];
   }
   ```

   

   ###### 适用场景：

   CopyOnWriteArrayList在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

   ###### 缺陷：

   - 内存占用：写操作时需要复制一个新的数组，使得内存占用为原来的两倍。
   - 数据不一致。读操作时不能读取实时性的数据，因为部分写操作还没同步到读数组中。

   所以CopyOnWriteArrayList不适合内存敏感以及实时性要求很高的场景。

### 3.LinkedList

##### 1.概览：基于双向链表实现，使用Node存储链表节点信息。

```
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

每个链表存储了first和last指针。

```
transient Node<E> first;
transient Node<E> last;
```

##### 2.与ArrayList比较

- ArrayList基于动态数组实现，LinkedList基于双向链表实现
- ArrayList支持随机访问，LinkedList不支持。
- LinkedList在任意位置添加删除元素更快。

### HashMap

##### 1.存储结构：内部包含一个Entry[]类型的数组table

java1.7中使用

```java
transient Entry[] table;
```

java1.8中使用

```
transient Node<K,V>[] table;
```

Entry和Node存储着键值对，从源码中next可以看出Entry或Node都是一个链表。数组中的每个位置都都被当成一个桶，一个桶存放一个链表。

HashMap使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的键值对对象（Entry或Node）

##### 2.拉链法的工作原理：

```
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```

- 新建一个HashMap，默认的大小为16

- 计算k1的hashCode为115，对16取模=3；k2,k3的hashCode为118，对16取模为6，

  链表的插入方式是以头插法方式进行的，对应<k3,v3>应在链表头部，即<k2，v2>前

###### 查找分为两步进行

1. 根据需要查找的key值确定桶所在位置
2. 在链表上顺序查找，时间幅度和链表长度成正比

##### 3.put操作

HashMap允许键为null的键值对。因无法确定null的哈希值，采用的是强制指定桶下标来存放。HashMap使用第0个桶存放键为null的键值对。

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    // 键为 null 单独处理
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    // 确定桶下标
    int i = indexFor(hash, table.length);
    // 先找出是否已经存在键为 key 的键值对，如果存在的话就更新这个键值对的值为 value
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    // 插入新键值对
    addEntry(hash, key, value, i);
    return null;
}
//键为null的键值对put（）
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
```

##### 4.扩容

设HashMap的长度为M，需要存储的键值对数量为N，如果哈希函数满足均匀性要求，则每条链表的长度为N/M。

因此，为了让查找的成本降低，应该让table也就是M尽可能大。HashMap采用动态扩容，根据N的数量来调整M的值，保证空间效率和时间效率。

扩容操作同样需要将oldTable的键值对重新插入newTable中，这一步很费时。

##### 5.重新计算桶下标

扩容时需要把键值对重新放在对应的桶上。HashMap使用了一个特殊的机制，降低重新计算桶下标的操作。

假设原数组长度为16，扩容后的new capacity为32；

```
capacity     : 00010000
new capacity : 00100000
```

对于一个key：

- 它的哈希值如果在第5位上为0，则取模和之前得到的哈希值一致
- 如果为1，则得到的结果为原来的结果+16

##### 6.链表转红黑树

JDK1.8后，当桶存储的链表的长度>=8时，会将链表转换为红黑树。

##### 7.HashMap和HashTable的比较

- HashTable使用synchronized进行同步，即HashTable是线程安全的
- HashMap允许键为null
- HashMap使用fail-fast迭代器
- HashMap不能保证随着时间的推移Map中的元素次序是不变的（扩容重新计算哈希值）

### 4.ConcurrentHashMap

`static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;
}`

##### ConcurrentHashMap和HashMap实现上类似。区别在于ConcurrentHashMap引入了分段锁（Segement）。

每个Segment维护着几个HashEntry，多个线程可以同时访问不同分段锁上的桶，并发度更高（并发量即Segment的个数）

##### Segment继承自ReentrantLock

`static final class Segment<K,V> extends ReentrantLock implements Serializable {`

`private static final long serialVersionUID = 2249069246763182397L;`

`static final int MAX_SCAN_RETRIES =`
    `Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;`

​	`transient volatile HashEntry<K,V>[] table;`

​	`transient int count;`

​	`transient int modCount;`

​	`transient int threshold;`

​	`final float loadFactor;`

`}`

##### Segment定义

`final Segment<K,V>[] segments;`

##### 默认Segment的并发级别为16

`static final int DEFAULT_CONCURRENCY_LEVEL = 16;`

##### JDK 1.8 的改动

JDK1.7使用Segment分段锁机制来实现并发更新操作，核心类即Segment,继承自重入锁ReentrantLock,并发度等于Segment个数。

JDK1.8中使用CAS操作来支持更高的并发度，在CAS操作失败时使用内置锁synchronized，同样在链表过长时会转换为红黑树。

### LinkedHashMap

##### 存储结构：

继承自HashMap,因此具有和HashMap一样的快速查找特性。

###### 内部维护一个双向链表，用来维护插入顺序或者LRU顺序。

```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
```

accessOrder决定了顺序，默认为false,此时维护的是插入顺序

```java
final boolean accessOrder;
```

LinkedHashMap最重要的是以下两个维护顺序的函数，会在put、get等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

##### afterNodeAccess()

当一个节点被访问时，如果accessOrder为true,则会把这个节点移到链表尾部。即指定了LRU顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，链表首部是最久没访问的节点。

##### afterNodeInsertion()

put操作等操作之后执行，当removeOldestEntry（）为true时，会移除链表首部节点first

removeOldestEntry默认为false,如果要让它为true,需继承自LinkedHashMap并覆盖这个方法的实现。这是实现LRU缓存的核心，移除最近最久未使用的节点，保证缓存空间足够，并且缓存的数据都是热点数据。

##### 实现LRU缓存的思路

a.继承LinkedHashMap

b.使用LinkedHashMap构造函数将accessOrder设置为true,开启LRU顺序。

c.覆盖removeOldestEntry（）实现，在节点多余MAX_ENTRY时，方法返回true，删除最近最久未使用的节点。

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final int MAX_ENTRIES = 3;

    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LRUCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }
}
```

```java
public static void main(String[] args) {
    LRUCache<Integer, String> cache = new LRUCache<>();
    cache.put(1, "a");
    cache.put(2, "b");
    cache.put(3, "c");
    cache.get(1);
    cache.put(4, "d");
    System.out.println(cache.keySet());
}
```

```java
[3,1,4]
```

### WeakHashMap

##### 存储结构

WeakHashMap的Entry继承自WeakReference,被weakReference关联的对象在下一次垃圾回收时会被回收。

WeakHashMap主要用来实现缓存，通过使用WeakHashMap来引用缓存对象，由JVM对这部分缓存进行回收。

