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

```java
transient Entry[] table;
```

java1.8中使用

```
transient Node<K,V>[] table;
```

