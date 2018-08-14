### ArrayList源码分析

##### 基本属性： 

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    //对象数组：ArrayList的底层数据结构,此是jdk1.8以前的定义的
    private transient Object[] elementData;
     //下面是jdk1.8后去掉了private，目的是简化嵌套访问
    transient Object[] elementData; // non-private to simplify nested class access
    //elementData中已存放的元素的个数，注意：不是elementData的容量
    private int size;
 .....
}
```

注意：

- `transient`关键字的作用：在采用**Java默认的**序列化机制的时候，被该关键字修饰的属性不会被序列化。
- `ArrayList`类实现了`java.io.Serializable`接口，即采用了Java默认的序列化机制
- 上面的`elementData`属性采用了`transient`来修饰，表明其不使用Java默认的序列化机制来实例化，但是该属性是`ArrayList`的底层数据结构，在网络传输中一定需要将其序列化，之后使用的时候还需要反序列化，那不采用Java默认的序列化机制，那采用什么呢？还有种方式就是我们自己去实现`writeObject`和`readObject` ，直接在类中搜索果真`ArrayList`自己实现了序列化和反序列化的方法

~~~java
/**
     * 保存Arraylist为stream流，为序列化
     */
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
    //可能存在并发问题，需要判断是否期望与实际的数量
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

/**
     * 根据ObjectInputStream 重新组装元素
     */
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
~~~

##### 构造器： 

~~~java
//给一个初始容量值
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
~~~

在实际使用中，如果所需的`ArrayList`的大小进行预判断，否则扩容会消耗性能。

#####  ArrayList中添加对象两个方法`add(E)`和`addAll(Collection<? extends E> c)`

`add(E)`方法分析

~~~java
public boolean add(E e) {
    //内部确保数字容量，modCount增加容量增加1
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //加入新元素e，size加1
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    //如果是同一个数组，获取最小容量
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

/**
*modCount在父类AbstractList中定义的。确定容量。
*modCount 检测是否发生了add、remove操作
*/
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    //确保数组的容量足够存放新加入的元素，若不够，要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
private void grow(int minCapacity) {
    //原有 元素数组的长度
    int oldCapacity = elementData.length;
    //新的容量 = oldCapacity + oldCapacity/2
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //如果新的容量都接近或者快超过Integer.MAX_VALUE - 8，
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
~~~

`addAll(Collection)`分析, 从上述代码可以看出，若加入的c是空集合，则返回false

~~~java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
~~~

`add(int index, E element)`分析

~~~java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
~~~

`set(int index, E element)`分析

~~~java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}

private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
~~~

##### `ArrayList`中的获取对象`get(int index)`

~~~java
public E get(int index) {
    rangeCheck(index);
	//直接从数组中获取
    return elementData(index);
}
~~~

##### `ArrayList`中的删除对象`remove(Object o)`和`remove(int index)`

通过查看源码可以发现remove(Object o)需要遍历数组，remove(int index)不需要，只需要判断索引符合范围即可，所以通常：后者效率更高。

~~~java
//删除第一个满足的条件的数据
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
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
//删除也将修改modCount
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

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
~~~

##### **对象是否存在于`ArrayList`中（`contains(E)`）** 

~~~java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
//indexOf(Object o)返回第一个出现的元素o的索引
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
~~~

##### **遍历`ArrayList`中的对象（`iterator()`）** 

~~~java
public Iterator<E> iterator() {
    return new Itr();//返回一个迭代器
}

private class Itr implements Iterator<E> {、
    //游标
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
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

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            //注意这里哦。将modcount赋值给expectedModCount所以在使用迭代器来遍历时可以删除元素，但是多线程会出现异常。单线程ok。
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
	//检查是否出现异常
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
~~~

还有几个迭代器`listIterator()`、`listIterator(int index)`这里就不做分析，可以自己尝试哦。

**`ArrayList`中的对象排序（`sort(Comparator)`）** 

通过源码可以看出也是调用了Arrays的排序算法，只不过比较容器是自己传入的，当日比较容器也可以为null，使用默认比较容器算法。排序后还要验证是否有`ConcurrentModificationException`异常。需要判断是否又添加了数据。

~~~java
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
~~~

`ArrayList`源码分析就到此为止了。总结下吧。

- `ArrayList`基于数组方式实现，无容量的限制（会扩容），如果超过`Integer.MAX_VALUE`会出现异常。
- 添加元素时可能要扩容（所以最好预判一下），删除元素时不会减少容量（若希望减少容量，`trimToSize()`），删除元素时，将删除掉的位置元素置为`null`，下次gc就会回收这些元素所占的内存空间。
- 线程不安全
- `add(int index, E element)`：添加元素到数组中指定位置的时候，需要将该位置及其后边所有的元素都整块向后复制一位
- `get(int index)`：获取指定位置上的元素时，可以通过索引直接获取时间复杂度（O(1)）
- `remove(Object o)`需要遍历数组
- `remove(int index)`不需要遍历数组，只需判断index是否符合条件即可，效率比remove(Object o)高
- `contains(E)`需要遍历数组
- 迭代器以及序列化的时候需要检验是否有`ConcurrentModificationException`异常

