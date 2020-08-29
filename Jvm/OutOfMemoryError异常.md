# OutOfMemoryError异常

- 除了程序计数器外，虚拟机内存的其他几个运行时区域都有发生`OutOfMemoryError`（下文称`OOM`）异常的可能。

## 一、Java堆溢出

- `Java`堆用于储存对象实例，只要不断地创建对象，并且保证`GC Roots`到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么随着对象数量的增加，总容量触及最大堆的容量限制后就会产生内存溢出异常。

- Java堆内存溢出异常测试:

```java
public class HeapOOM {
	static class OOMObject {
	}
    public static void main(String[] args) {
   		 List<OOMObject> list = new ArrayList<OOMObject>();
   		 while (true) {
   			 list.add(new OOMObject());
   		 }
    }	
}
```

> 运行结果：

```java
java.lang.OutOfMemoryError: Java heap space
```

- 如果不是内存泄漏，换句话说就是内存中的对象确实都是必须存活的，那就应当检查`Java`虚拟机的堆参数`（-Xmx与-Xms）`设置，与机器的内存对比，看看是否还有向上调整的空间。再从代码上检查是否存在某些对象生命周期过长、持有状态时间过长、存储结构设计不合理等情况，尽量减少程序运行期的内存消耗。



## 二、虚拟机栈和本地方法栈溢出

- 由于`HotSpot`虚拟机中并不区分虚拟机栈和本地方法栈，因此对于`HotSpot`来说，`-Xoss`参数（设置本地方法栈大小）虽然存在，但实际上是没有任何效果的，栈容量只能由`-Xss`参数来设定。

- 虚拟机栈和本地方法栈异常：
  1. 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出`StackOverflowError`异常。
  2. 如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出`OutOfMemoryError`异常。

- `Java`虚拟机实现自行选择是否支持栈的动态扩展，而`HotSpot`虚拟机的选择是不支持扩展，所以除非在创建线程申请内存时就因无法获得足够内存而出现`OutOfMemoryError`异常，否则在线程运行时是不会因为扩展而导致内存溢出的，只会因为栈容量无法容纳新的栈帧而导致`StackOverflowError`异常。

- 无论是由于栈帧太大还是虚拟机栈容量太小，当新的栈帧内存无法分配的时候，`HotSpot`虚拟机抛出的都是`StackOverflowError`异常。



## 三、方法区和运行时常量池溢出

- 由于运行时常量池是方法区的一部分，所以这两个区域的溢出测试可以放到一起进行。

- `HotSpot`从`JDK 7`开始逐步“去永久代”， 在`JDK 8`中完全使用元空间来代替永久代。

- `String::intern()`是一个本地方法，它的作用是如果字符串常量池中已经包含一个等于此`String`对象的字符串，则返回代表池中这个字符串的`String`对象的引用；否则，会将此`String`对象包含的字符串添加到常量池中，并且返回此`String`对象的引用。

- 运行时常量池导致的内存溢出异常：

```java
public class RuntimeConstantPoolOOM {
	public static void main(String[] args) {
		// 使用Set保持常量池引用，避免Full GC回收常量池行为
		Set<String> set = new HashSet<String>();
		// 在short范围内足以让6MB的PermSize产生OOM了
		short i = 0;
		while (true) {
			set.add(String.valueOf(i++).intern());
		}
	}
}
```

> 运行结果：

```java
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
```



## 四、本机直接内存溢出

- 直接内存（`Direct Memory`）的容量大小可通过`-XX`：`MaxDirectMemorySize`参数来指定，如果不去指定，则默认与Java堆最大值（由`-Xmx`指定）一致，下列代码越过了`DirectByteBuffer`类直接通过反射获取`Unsafe`实例进行内存分配（`Unsafe`类的`getUnsafe()`方法指定只有引导类加载器才会返回实例，体现了设计者希望只有虚拟机标准类库里面的类才能使用`Unsafe`的功能，在`JDK 10`时才将`Unsafe`的部分功能通过`VarHandle`开放给外部使用），因为虽然使用`DirectByteBuffer`分配内存也会抛出内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知内存无法分配就会在代码里手动抛出溢出异常，真正申请分配内存的方法是`Unsafe::allocateMemory()`。

```mysql
public class DirectMemoryOOM {
	private static final int _1MB = 1024 * 1024;
	public static void main (String[] args) throws Exception {
		Filed unsafeField = Unsafe.class.getDeclareFileds()[0];
		unsafeField.setAccessible(true);
		Unsafe unsafe = (Unsafe) unsafeField.get(null);
		while (true) {
			unsafe.allocateMemory(_1MB);
		}
	}
}
```

> 由直接内存导致的内存溢出，一个明显的特征是在`Heap Dump`文件中不会看见有什么明显的异常情况，如果读者发现内存溢出之后产生的`Dump`文件很小，而程序中又直接或间接使用了`DirectMemory`（典型的间接使用就是`NIO`）。

