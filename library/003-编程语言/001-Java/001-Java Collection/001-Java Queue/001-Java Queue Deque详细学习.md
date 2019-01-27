# Java Queue Deque详细解释

## 前言
之前一直没有全面、系统地学习Java里的各种知识点，因此许多细节我一直都不太清楚，一直以来提到Java自己就感到有点虚，所以趁这个机会，我来系统地学习一下Java的Queue，虽然还深入不到源码层次，但是我想尽可能全面地来学习每一个知识点。  
注意：我使用的Jdk版本为1.8，其它版本可能略有不同。

## Queue接口
### 寻找源码
首先，我在代码中按照常规使用Queue的写法写了一行代码：

```Java
Queue<String> q = new LinkedList<>();
```
然后使用自动导入的方法，引入了java.util.Queue，从这个可以确定Queue是在util包中的（这个其实是常识，可以直接打开util包）。  
接下来我们打开Java环境的util包，会发现实际上这个包位于：
**[JAVA_HOME]/jre/lib/rt.jar** 其中[JAVA_HOME]为你的Jdk安装目录。  
在Intellij IDEA中打开rt.jar  
![](assets/003/20190127-b8e816d1.png)  
![](assets/003/20190127-0ecbb83f.png)  
在rt.jar里面找到/java/util,然后在util上右键使用Intellij IDEA自带的图形化工具（目录右键选择Diagrams->Show Diagrams->Java Class Disgrams），可以直接看到类图。
![](assets/003/20190127-88c4467c.png)  
从这个图中可以看出，Queue继承了Collection接口。  
Collection相关的操作我之后会看，那边写好之后这边我会加一个链接跳过去，所以这里就不再细说。
### 源码分析
看Queue实现的源码
```Java
public interface Queue<E> extends Collection<E> {...}
```
因此我们可以确定：
1. Queue是一个接口。
2. Queue继承了Collection。

接下来我们看一看Queue接口中有哪些方法：  
![](assets/003/20190127-183b6284.png)  
也就是说所以实现或者继承Queue接口的类都需要实现这些方法，这里暂且不探究它们具体的使用方法，只是把源码注释稍微翻译一下（接口前面还有大量的注释描述Queue，这里不再翻译，有兴趣可以自行阅读）。
```Java
/*
队列一般是按照FIFO（即先进先出）的规则对元素进行排序，但是有例外：
1. 优先队列。优先队列是有限根据提供的比较器来对元素进行排序，其次才是元素的进入顺序；
2. LIFO队列。如堆栈等，它们按照LIFO（即后进先出）的规则对元素进行排序。
不管什么规则呢，head都是调用remove()或者poll()进行删除。在FIFO队列中所有元素插入队列tail，其它类型队列可能会采取不同的放置规则，而且每个Queue实现时都需要指定其排序属性。
*/
public interface Queue<E> extends Collection<E> {
    /*
    如果可以在不违反容量限制的情况下立即执行此操作，则将指定的元素插入此队列，成功时返回 true，如果当前没有空间，则抛出 IllegalStateException。
    */
    boolean add(E e);
    /*
    如果可以在不违反容量限制的情况下立即将指定的元素插入到此队列中。当使用容量受限的队列时，此方法通常比{@link #add}更可取，因为后者在容量受限时只能抛出异常，前者会返回false。
    （这个其实借口前面注释前面有说明，add在容量不足时会抛出未检查的异常，但是offer()却会返回false）
    */
    boolean offer(E e);
    /*
    检索并删除此队列的头部。此方法与{@link #poll}的不同之处在于，如果该队列为空，它将抛出异常。
    */
    E remove();
    /*
    检索并删除该队列的头部，如果该队列为空，则返回{@code null}。
    */
    E poll();
    /*
    检索但不删除此队列的头部。这个方法与{@link #peek peek}的不同之处在于，它只在队列为空时抛出异常。
    */
    E element();
    /*
    检索但不删除此队列的头部，如果该队列为空，则返回{@code null}。
    */
    E peek();
}
```

 其实总结一下，Queue接口其实就只是自带了三对接口：
 1. **add()/offer()** ：向队列中插入元素，不同之处在于前者在空间不足时会抛出异常，而后者会返回false
 2. **remove()/poll()**： 检索并且删除队首元素，不同之处在于队列为空时前者会抛出异常，后者会返回null
 3. **element()/peek()**： 检索但是不删除队首元素，不同之处在于队列为空时前者会抛出异常，后者会返回null

 其实这三对方法后者都是前者的不抛出异常的使用方法，我有点好奇的是后者不会抛出异常是否在性能上有所牺牲，等后续Queue的所有实现看完之后可以查看它们具体的实现并且做一下实验，这里暂时不再继续深入研究，目前来在容量无限时，用前三个方法比较合适。在容量有显示，推荐用后三个方法来代替前三个方法。
 ## Deque接口
从刚刚的图中我们可以继续向下寻找Queue的子类：
![](assets/003/20190127-03c3abc6.png)  
其中左边一个包展开之后是非常复杂的类，与线程相关，应该是线程安全的一些实现。这里暂且不深入研究，先看显示的一些继承和实现。  
Deque接口实现了Queue接口，不过拥有更多的方法，Deque的解释是双端队列，即在两端都可以进行读写的队列。  
这里面的方法也和Queue里面的类似，一般都有一对来实现同样的功能但是前者会抛出异常而后者会返回特定的值。
```Java
  /*
  如果可以在不违反容量限制的情况下立即在此deque前面插入指定元素，前者在当前没有可用空间的情况下抛出 IllegalStateException。在使用容量受限的deque时，通常最好使用方法 offerFirst，它在容量受限会返回false，插入成功时返回true
  */
  void addFirst(E e);
  boolean offerFirst(E e);

  /*
  如果可以在不违反容量限制的情况下立即在此deque后面插入指定元素，前者在当前没有可用空间的情况下抛出 IllegalStateException。在使用容量受限的deque时，通常最好使用方法 offerLast，它在容量受限会返回false，插入成功时返回true
  */
  void addLast(E e);
  boolean offerLast(E e);

  /*
  检索并删除该deque的第一个元素，如果该deque为空，前者抛出异常，后者返回 null。
  */
  E removeFirst();
  E pollFirst();

  /*
  检索并删除该deque的最后一个元素，如果该deque为空，前者抛出异常，后者返回 null。
  */
  E removeLast();
  E pollLast();

  /*
  检索但不删除此deque的第一个元素。队列为空时前者抛异常，后者返回null。
  */
  E getFirst();
  E peekFirst();

  /*
  检索但不删除此deque的最后一个元素。队列为空时前者抛异常，后者返回null。
  */
  E getLast();
  E peekLast();

  /*
  删除第一次出现的指定元素，如果有则返回true，没有则返回false
  */
  boolean removeFirstOccurrence(Object o);

  /*
  删除最后一次出现的指定元素，如果有则返回true，没有则返回false
  */
  boolean removeLastOccurrence(Object o);

  //其余为queue method和collection method
```

## 其它类实现
其它类的实现我会专门新建对应的文件，如果用到了专门去补充整理。
