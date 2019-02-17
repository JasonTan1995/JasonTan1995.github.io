---
title: LinkedList源码分析
date: 2019-02-17 21:52:36
tags: Java
---

#### LinkedList 源码分析

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g08ed58cu2j20rs0etq6z.jpg)

##### 1. 概述

------

- LinkedList的底层是由一个双向链表实现的。
- 与ArrayList不同的是，LinkedList没有扩容机制；而是通过节点之间的前驱与后继的引用来存储元素。
- 在链表的头部与尾部插入与删除元素的效率极高，但在指定位置插入或删除效率就一般了。原因是需要定位到某个节点，时间复杂度为`o(N)`。
- 是一个非线程安全的集合类。

<!-- more -->

##### 2. 继承关系

------

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g055q19ipxj218g0q8tn4.jpg)

- LinkedList继承自AbstractSequentialList,而通过源码分析就可以知道AbstractSequentialList大部分方法都是由ListIterator实现的。
- AbstractSequentialList提供的一系列接口，使得继承它的类可以基于顺序访问（例如链表）。
- 但LinkedList比较特别的是，它并没有直接使用父类的方法，而是自己实现了一套方法。

```java
public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
}

public void add(int index, E element) {
        try {
            listIterator(index).add(element);
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
}

public E remove(int index) {
        try {
            ListIterator<E> e = listIterator(index);
            E outCast = e.next();
            e.remove();
            return outCast;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
}

// 留给子类实现
public abstract ListIterator<E> listIterator(int index);
```

除此之外，LinkedList还实现了Deque接口，而Deque又继承自Queque，这样LinkedList就具备了队列的功能了。也就可以使用LinkedList实现Stack（栈）了。

##### 3. 源码分析

------

###### 3.1 结构组成

- LinkedList是靠多个节点组成的。

```java
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

- 用图来表示就是这样：

  ![](https://ws1.sinaimg.cn/large/6b297ce5gy1g057hxmgswj20b30580sq.jpg)

- 通过对前驱以及后继的引用来连成一条双向链表。

- 通过节点之间两两互相引用形成一条链表。就例如：（一条链表的样子）

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g057v27e4pj20nb08udi5.jpg)

###### 3.2 查找

LinkedList中有关查找的方法：

```java
//用来记录头节点
transient Node<E> first;
//用来记录尾节点
transient Node<E> last;

// 获取第一个节点
public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
}

// 获取最后一个节点
public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
}

// 根据索引获取元素
public E get(int index) {
       //检查索引是否合法
        checkElementIndex(index);
        return node(index).item;
}
```

- getFirst()与getLast()方法都是获取已记录链表中的头节点以及尾节点。至于头节点与尾节点是怎么形成的，等下在接下来的添加元素会说明。

- 至于get(int index)根据索引获取元素这里作了一个优化，如果不作这个优化的话，在链表中根据索引查找是一件很恐怖的事情。要从头节点一直遍厉，直到该索引的位置，来看看是怎样优化：

  ```java
  Node<E> node(int index) {
          // assert isElementIndex(index);
          // 这里是把链表的长度的一半跟所需查询的索引比较了
          if (index < (size >> 1)) {
              //如果小于一半就从链表头部开始遍历，否则就从链表尾部开始
              Node<E> x = first;
              for (int i = 0; i < index; i++)
                  x = x.next;
              return x;
          } else {
              Node<E> x = last;
              for (int i = size - 1; i > index; i--)
                  x = x.prev;
              return x;
          }
      }
  ```

  无论怎么优化，在链表中根据索引查找元素都是一件非常耗时的工作。`时间复杂度为o(N)`.并没有数组那样随机访问的能力。

  ![](https://cdn-images-1.medium.com/max/1600/1*G43FVT5xJ1n1QDKVNZUxXQ.jpeg)

###### 3.3 插入（添加元素）

- 关于添加的几个方法：

  ```java
  //直接将要插入的元素放在链表头
  public void addFirst(E e) {
          linkFirst(e);
  }
  //将要插入的元素放在链表的尾端
  public void addLast(E e) {
          linkLast(e);
  }
  
  //这个添加元素方法是默认将元素追加到链表的尾端
  public boolean add(E e) {
          linkLast(e);
          return true;
  }
  //将元素添加到指定的位置
  public void add(int index, E element) {
      //检查索引是否合法
          checkPositionIndex(index);
          if (index == size)
              //看要插入的位置是否是最后一个，是的话就直接追加到链表的尾端
              linkLast(element);
          else
              linkBefore(element, node(index));
  }
  ```

  可以看出来这几个添加方法其实都被包装了一层，接下来就揭开神秘的面纱来分析这些被包装的方法：

  ```java
  /**
   * Links e as first element.
   * 该方法是将包含该元素的节点直接置为链条头部的节点
   */
  private void linkFirst(E e) {
      //获取链表头
      final Node<E> f = first;
      // 创建一个新的节点，建立后继节点引用
      final Node<E> newNode = new Node<>(null, e, f);
      // 将新的节点变成连链表的第一个节点
      first = newNode;
      //如果原来的链表头是空的话，证明链表中还没有任何元素，那么头跟尾都是现在的这个元素
      if (f == null)
          last = newNode;
      else
          //如果不是空的话，就将原来头节点的前驱引用为新的节点
          f.prev = newNode;
      size++;
      modCount++;
  }
  
  /**
   * Links e as last element.
   * 将包含该元素的节点置为链表尾端的节点
   */
  void linkLast(E e) {
      //获取链表尾端的节点
      final Node<E> l = last;
      //因为要将新的节点作为链表的尾节点。所以创建该新元素要跟原来的尾节点建立前继关系
      final Node<E> newNode = new Node<>(l, e, null);
      // 将新节点置为链表的尾节点
      last = newNode;
      // 判断原来的尾节点是否为空，如果为空就证明原来的链表中没有元素
      if (l == null)
          //那么证明插入的新元素是第一个元素，它是头节点同时也是尾节点
          first = newNode;
      else
          //不为空的话，原来的尾节点的后继引用就是新插入的节点
          l.next = newNode;
      size++;
      modCount++;
  }
  ```

  链表头部插入和链表尾部插入都是大同小异的。而接下来我把指定位置插入的方法抽出来分析一下：

  ```java
  /**
   * Inserts element e before non-null Node succ.
   */
  void linkBefore(E e, Node<E> succ) {
      // assert succ != null;
      final Node<E> pred = succ.prev;
      final Node<E> newNode = new Node<>(pred, e, succ);
      succ.prev = newNode;
      if (pred == null)
          first = newNode;
      else
          pred.next = newNode;
      size++;
      modCount++;
  }
  ```

  看图会比较容易理解一点：

  ![](https://ws1.sinaimg.cn/large/6b297ce5gy1g087c6a7t3j218e0p0h05.jpg)

  linkBefore的主要步骤为：

  1.创建新节点，并为新节点建立前驱与后继关系

  2.将succ前驱指向新节点

  3.如果succ的前驱不为空，则将succ前驱的后继引用指向新节点

以上就是LinkedList关于插入添加的方法解析。总结就是插入新元素还是比较方便的。只需要理清前驱以及后继引用的关系就好了。

###### 3.4 删除

上面的插入理解了的话，删除也易如反掌了。删除是通过解除前驱与后继的关系，并将该节点置空来完成。头跟尾的删除就不仔细讲了，要看懂也不难。

```java
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
           //如果是空的话，会把所有空的元素都删除了。
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            //查找需要删除的元素
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

public E remove(int index) {
    //检查索引是否合法
    checkElementIndex(index);
    return unlink(node(index));
}


/**
 * Unlinks non-null node x.
 */
E unlink(Node<E> x) {
    // assert x != null;
    //获取元素
    final E element = x.item;
    //后继引用的节点
    final Node<E> next = x.next;
    //前驱引用的节点
    final Node<E> prev = x.prev;
    
    //如果为空的话，证明是头节点
    if (prev == null) {
        first = next;
    } else {
        //如果不是的话，将x的前驱的后继指向x的后继
        prev.next = next;
        //将x的前驱置空，断开引用关系
        x.prev = null;
    }

    // 如果为空，next就是尾节点
    if (next == null) {
        last = prev;
    } else {
        //将x的后继的前驱指向x的前驱
        next.prev = prev;
        //同理断开引用关系
        x.next = null;
    }
    
    //将元素置空，方便GC回收
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

还是看图会比较容易理解：

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g08awquba5j218e0n8wtb.jpg)

假设删除的节点在不是头也不是尾，步骤：

1.将前驱的节点的后继引用指向待删除节点的后继节点，断开待删除节点的前驱关系。

2.将后继的节点的前驱引用指向待删除节点的前驱节点，同理的断开待删除节点的后继关系。

3.将待删除节点的item置空。

###### 3.5 坑

LinkedList的随机访问是一件很恐怖的事情，因此在遍历的时候应该尽量避免随机位置的访问。

```java
List<Integet> list = new LinkedList<>();
list.add(1)
list.add(2)
......
for (int i = 0; i < list.size(); i++) {
    Integet item = list.get(i);
    // do something
}
```

当链表中有大量元素时，对于效率来说简直是灾难级。因为每次获取元素的时候，都需要从链表头或者链表尾一个一个的寻找遍历，因此平常开发中应尽量避免。

------

##### 参考资料

http://www.tianxiaobo.com/2018/01/31/LinkedList-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-JDK-1-8/