# Java集合框架

## 一、集合接口与实现分离

- Java集合类库也将接口与实现分离，使用队列（queue）来解释：

> 队列接口指出可以在队列的尾部添加元素，在队列的头部删除元素并且可以查找队列中元素的个数；当需要收集对象时，并按照“先进先出”的方式检索对象，就应该使用队列。

图一：

- 队列的最简形式可能类似这样：

```
public interface Queue<E> {
	void add(E element);
	E remove();
	int size();
}
```

> 队列有两种实现方式：一种是使用循环数组，另一种是使用链表。

图二：

- 每个实现都可以用一个实现了Queue接口的类表示。

```java
public class CircularArrayQueue<E> implements Queue<E> {
	private int head;
	private int tail;
	CircularArrayQueue(int capacity) {...}
	public void add(E element){...}
	public E remove(){...}
	public int size(){...}
	private E[] elements;
}

public class LinkedListQueue<E> implements Queue<E> {
	private Link head;
	private Link tail;
	LinkedListQueue( ) {...}
	public void add(E element){...}
	public E remove(){...}
	public int size(){...}
}
```

> 循环数组要比链表更高效，但是循环数组是一个有界集合，容量有限，如果程序中收集的对象没有上限，最好使用链表来实现。

## 二、Collection 接口

- Java类库宏，集合类的基本接口是Collection接口，这个接口有两个基本方法：

```java
public interface Collection<E> {
	boolean add(E element);     //向集合中添加元素，成功返回true，失败返回false。
    Iterator<E> iterator(); //返回一个实现类Iterator接口的对象。可以使用该对象依次访问集合中的元素
}
```



## 三、迭代器

- Iterator接口包含4个方法：

```java
public interface Iterator<E> {
	E next();      //访问集合中的每个元素
	boolean hasNext();    //查看集合中是否还有元素，有则返回true, 没有返回false
	void remove();
	default void forEachRemaining(Consumer<? super E> action);
}
```

> 迭代器的使用：

```
Collection<String> c = ...;
Iterator<String> iter = c.iterator;
while(iter,hasNext()){
	String element = iter.next();
	do something with element  
}

//使用 “for-each”， 底层也是迭代器实现
for(String element : c){
	do something with element  
}
```

> "for-each"可以处理任何实现了Iterable接口的对象，这个接口只包含一个抽象方法：

```
public interface Iterable<E> {
	Iterator<E> iterator();
	...
}
```

> Collection接口扩展了Iterable接口，so对于标准类库中的任何集合都可以使用"for-each"循环。

- 可以调用forEachRemaining方法并提供一个lambda表达式（它会处理每一个元素），将对迭代器的每一个元素调用这个lambda表达式，直到没有元素为止。

```
iterator.forEachRemaining(element -> do something with element);
```

> 访问元素的顺序取决于元素类型：如果是ArrayList集合，从索引0开始，每次加1；如果是HashSet中的元素，会按照随机的方式获取元素。

- 可认为Java迭代器位于两个元素之间，迭代器就会越过下一个元素，并返回刚刚越过的那个元素的引用。

如图：

> Iterator接口的remove方法将会删除上次调用next方法时返回的元素，要删除指定位置上的元素，需要越过这个元素。调用remove方法之前没有调用next，将是不合法的，会抛出异常，必须要先调用next越过将要删除的元素 。



