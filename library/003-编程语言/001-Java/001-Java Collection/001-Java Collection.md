# Java Collection
Java Collection是Java非常重要的一个接口，Queue，List和Set都是Collection的实现。这里我专门来看一下Collection有哪些接口，分别能够实现什么功能。  
如接口上面的注释所描述：Collection表示了一组对象。具体实现可能是多种多样的，之后我们挑选出其中一些比较常用的。
## 接口列表
![](assets/003/20190128-ddc9ca2c.png)  
下面对这些常用的方法做一些介绍：
```java
  /*
  返回集合中元素的数量。 注意，如果集合中的元素的数量大于了Integer.MAX_VALUE，那么返回Integer.MAX_VALUE
  */
  int size();

  /*
  如果集合中有元素那么返回true，否则返回false
  */
  boolean isEmpty();

  /*
  如果集合中含有指定元素那么返回true，否则返回false
  */
  boolean contains(Object o);

  /*
  返回集合中元素的迭代器，无法保证元素的返回顺序，（除非集合的实现规定了元素的顺序）
  */
  Iterator<E> iterator();

  /*
  返回包含此集合中所有元素的Array。顺序同迭代器。
  */
  Object[] toArray();

  /*
  使用泛型的上面的方法，类型动态指定。
  */
  <T> T[] toArray(T[] a);

  /*
  向集合中添加元素，但是有可能具体实现会对此做出限制。
  */
  boolean add(E e);

  /*
  同上，在集合中删除元素，同样可能会对此做出限制。
  */
  boolean remove(Object o);

  /*
  如果集合中包含集合c中的所有元素，那么返回true，否则返回false
  */
  boolean containsAll(Collection<?> c);

  /*
  添加集合中的所有元素，可能会抛出各种异常。
  */
  boolean addAll(Collection<? extends E> c);

  /*
  移除集合中c集合中的元素
  */
  boolean removeAll(Collection<?> c);

  /*
  在集合中移除满足指定条件的元素，其中Predicate之后会专门研究
  */
  default boolean removeIf(Predicate<? super E> filter)

  /*
  从此集合中移除未包含在指定集合c中的元素
  */
  boolean retainAll(Collection<?> c);

  /*
  移除集合中的所有元素
  */
  void clear();

  /*判断是否相等*/
  boolean equals(Object o);

  /*
  返回集合的hash code
  */
  int hashCode();

  default Spliterator<E> spliterator()

  default Stream<E> stream()

  default Stream<E> parallelStream()
```
## AbstractCollection

除此之外Abstract是实现了Collection接口的一个抽象类，它实现了Collection接口的部分方法，其实后面可以看到有不少实现类都继承了这个抽象类。  
从这里可以总结出一种比较优秀的设计思想：当一个接口有很多派生子类时，可以实现一个抽象类，子类可以选择实现接口的同时继承这个抽象类，那么可以减少重复性的开发。
