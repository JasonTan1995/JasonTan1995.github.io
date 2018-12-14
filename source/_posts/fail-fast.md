---
title: fail-fast
date: 2018-08-14 21:25:58
tags: Java
categories: Java
---

  平常在使用集合的时候肯定会遇到过遍历删除集合中的某些元素，哪么在进行操作的时候大家有
没有遇到过什么问题呢？最近在使用集合进行遍历删除元素的时候，我就碰到了这个异常：java.util.ConcurrentModificationException。该异常是一个并发修改异常，于是引起我了的思考。

平常会使用遍历删除的几种方式：

1. **通过增强for删除符合要求的元素(一个或者多个)**

- 会出现异常java.util.ConcurrentModificationException

- 但假如加上break的话异常就不会出现，因为不会影响到后续的元素。

  <!--more-->
```
  public void listRemove(){
      List<Integer> list = new ArrayList<>();
      for (Integer i:list){
          list.remove();
      }
  }
************************************************  
  public void listRemove(){
      List<Integer> list = new ArrayList<>();
      for (Integer i:list){
          list.remove();
          break;
      }
  }
```
2. **通过普通for删除符合要求元素**
- 这里虽然不会出现异常，但是你会发现会有些元素没有被删除掉的。原因是：比如我添加10个元素，然后把第三个删除掉了。那么原本下标为4的元素就会变成下标为3的元素了。如此类推就会出现数据不正确的情况。
```
  public void listRemove(){
      List<Student> students = new ArrayList<>();
      student.add........ // 添加元素就略了
      for (int i = 0; i<list.size();i++){
           Student s  = list.get(0);
           list.remove(s);
      }
  }
```
3. **通过Iteator删除符合要求的元素**
- 假如这里删除用的是List的remove方法还是会照样抛出java.util.ConcurrentModificationException
- 如果不想抛出异常又能正确删除的话，建议使用Itreator的remove方法。
```
   public void listRemove(){
      List<Student> students = new ArrayList<>();
      student.add........ // 添加元素就略了
      Itreator<Student> it = students.Itreator();
      while(it.hasNext()){
          Student student = it.next();
          if (student.getAge() == 10){
              it.remove();
          }
      }
  }
```


**那么什么会出现以上的情况呢？这就跟Fail-Fast机制有关了。**

> Fail-Fast机制是Java集合中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就会发生Fail-Fast事件。

> 例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

> ConcurrentModificationException异常:当方法检测到对象的并发修改，但不允许这种修改时就会抛出这个异常。需要注意的是：如果单线程违反了规则，同样也会抛出这个异常

> 迭代器的快速失败行为无法得到保证，它不能保证一定会出现该错误，但是快速失败操作会尽最大努力抛出ConcurrentModificationException异常，所以因此，为提高此类操作的正确性而编写一个依赖于此异常的程序是错误的做法

结论：ConcurrentModificationException异常应该仅用于检测BUG。

**那么为什么Itreator修改就不会抛出ConcurrentModificationException异常呢？**

Itreator是工作在一个独立的线程中，并有一个mutex锁(互斥锁)。

- synchronized修饰的同步代码块就是一个互斥锁。

> 线程在进入同步代码块之前会自动获取锁，并且在退出同步代码块时会自动释放锁，当某个线程请求一个由其他线程持有的锁时，发出请求的线程就会阻塞。互斥锁其实提供了一种原子操作，让所有线程以串行的方式执行同步代码块。

* Iterator被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，

* 所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照 fail-fast 原则 Iterator会马上抛出java.util.ConcurrentModificationException 异常。

* 所以 Iterator 在工作的时候是不允许被迭代的对象被改变的。但你可以使用 Iterator 本身的方法 remove() 来删除对象，Iterator.remove()方法会在删除当前迭代对象的同时维护索引的一致性。

#### 但如果你的Collection/Map对象实际只有一个或两个元素的时候，ConcurrentModificationException异常并不会抛出。

***

本文内容参考自：http://www.hollischuang.com/archives/33