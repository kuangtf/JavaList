# Java8的流库

## 一、从迭代到流的操作

- 处理集合时，我们通常会迭代遍历它的元素 ，并在每个元素上执行某项操作。

- 假设我们想要对本书中的所有长单词进行计数：

```java
var contents = new String(Files.readAllBytes(
	Paths.get("alice.txt")),StandardCharsets.UTF_8);  //read file into string             
List<String> words = List.of(contents.split("\\PL+"));
	//Split into words; nonletters are delimiters
```

迭代：

```java
int count = 0;
for(String w : words){
	if(w.length() > 12) count ++;
}
```

使用流：

```java
long count = words.stream()
	.filter(w -> w.length() > 12)
	.count();
```

> 仅将`stream`修改为`parallelStream`就可以让流库以并行方式来执行过滤和计数。

```java
long count = words.parallelStream()
	.filter(w -> w.length() > 12)
	.count();
```

> 流遵循了“做什么而非怎么做“的原则。

- 流表面上和集合很类似，都可以转换和获取数据，也存在显著的差异：

1. 流并不存储其他元素，这些元素可能存储在底层的集合中，或者是按需生成的。
2. 流的操作不会修改其他数据源。例如：filter方法不会从流中移除元素，而是会生成一个新流，其中不包含被过滤的元素。
3. 流的操作时尽可能惰性执行的。直至需要其结果时，操作才会执行。

- 包含3个阶段的操作管道：

1. 创建一个流。
2. 指定将初始流转换为其他流的中间操作，可能包含多个步骤。
3. 引用终止操作，从而产生结果。这个操作会强制执行之前的惰性操作，之后这个流就再也不能用了。



## 二、流的创建

- 可以用`Collection`接口的`stream`方法将任何集合转换为一个流。
- 如果你有一个数组，可以使用静态的`Stream.of`方法。

```java
Stream<String> words = Stream.of(contents.split("\\PL+"));
	//split returns s String[] array
```

> `of` 方法具有可变长参数，因此我们可以构建具有任意数量引元的流：

```java
Stream<String> song = Stream.of("gently","down","the","stream");
```

> 使用`Array.stream(array, from, to)` 可以用数组中的一部分元素来创建一个流。

- 为了创建不包含任何元素的流，可以使用静态的`Stream.empty`方法：

```java
Stream<String> silence = Stream.empty();
	//Generic type <String> is inferred; same as Stream.<String>empty()
```

- `Stream`接口有两个用于创建无限流的静态方法。
- `generate`方法会接受一个不包含任何引元的函数（从技术上讲，是一个`Supplier<T>` 接口的对象）。无论何时，只需要一个流类型的值，该函数就会产生一个这样的值。
- 可以像下面这样获得一个常量值的流：

```java
Stream<String> echos = Stream.generate(() -> "Echo");
```

- 像下面这样获取一个随机数的流：

```java
Stream<Double> randoms = Stream.generate(Math::random);
```

> 如果要生产像 0 1 2 3  ... 这样的序列，可以使用iterate方法。它会接受一个“种子”值，以及一个函数（从技术上讲，是一个`UnaryOperation<T>`),并且会反复的将该结果应用到之前的结果上，例如：

```java
Stream<BigInteger> integets
    = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
```

> 该序列中的第一个元素是种子`BigInteger.ZERO`, 第二个元素是`f(seed)`。

- `Stream.ofNullable`方法会用一个对象来创建一个非常短的流。如果该对象为`null` ,那么这个流的长度就为0；否则这个流的长度就为1,即只包含该对象。
- 如果我们持有的`Iterable`对象不是集合，那么可以通过下面的调用将其转换为一个流：

```java
StreamSupport.stream(iterable.spliterator(), false);
```

> 如果我们持有的是`Iterator`对象，并且希望得到一个由它的结果构成的流，那么可以使用下面的语句：

```java
StreamSupport.stream(Spliterators.spliteratorUnknowSize(
	iterator, Spliterator.ORDERED), false);
```

- 在执行流的操作时，并没有修改流背后的集合。流并没有收集其数据，数据一直存储在单独的集合中。如果修改了该集合，那么流操作的结果就会变成未定义的。`JDK`文档称这种要求为**不干涉性**。



## 三、filter、map 和 flatMap 方法

- 流的转换会产生一个新的流，它的元素派生自另一个流中的元素。
- `filter`转换会产生一个新流，它的元素与某种条件相匹配。将一个字符串流转换为只包含长单词的另一个流：

```java
List<String> words = ... ;
Stream<String> longWords = words.stream().filter(w -> w.length() > 12);
```

> `filter`的引元是`Predicate<T>` , 即从`T`到`boolean`的函数。

- 按照某种方式来转换流中的值，可以使用`map`方法并传递执行该转换的函数。可以像下面这样将所有单词转换为小写：

```java
Stream<String> lowercaseWords = words.stream().map(String :: toLowerCase);
```

> 这里使用的是带有方法引用的`map`，但是我们可以使用`lambda`表达式来代替：

```java
Stream<String> firstLetters = words.stream().map(s -> s.sunstring(0,1));
```

> 上面的语句所产生的流中包含所有单词的首字母。

- 在使用map时，会有一个函数应用到每个元素上，并且其结果是包含了应用该函数后所产生的所有结果的流。

- 假设有一个函数，返回的不是一个值，而是一个包含众多值的流。下面的实例展示的方法会将字符串转换为字符串流，即一个个的编码点：

```java
public static Stream<String> codePoints(String s){
	var result = new ArrayList<String>();
	int i = 0;
	while(i < s.length){
		int j = s.offsetByCodePoints(i,1);
		result.add(s.substring(i,j));
		i = j;
	}
	return result.stream();
}
```

> 这个方法可以正确的处理需要两个`char`值来表示`Unicode`字符。

- 假设将`codePoints`方法映射到一个字符串流上：

```java
Stream<Stream<String>> result = words.stream().map(w -> codePoints(w));
```

> 就会得到一个包含流的流，就像 `[...["y","o","u","r"],["b","o","a","t"], ...]`, 将其摊平为单个流：`[..."y","o","u","r","b","o","a","t", ...]` ，可以使用`flat`方法而不是`map`方法：

```
Stream<String> flatResult = words.stream().flatMap(w -> codePoints(w));
	//Calls codePoints on each word and flattens the results
```



## 四、抽取子流和组合流

- 调用`stream.limit(n)`会返回一个新的流，它在n个元素之后结束（如果原来的流比 n 短，那么就会在该流结束时结束）。

> 这个方法对于裁剪无限流的尺寸特别有用：

```java
Stream<Double> randoms = Stream.generate(Math::random).limit(100);
```

> 会产生一个包含100个随机数的流。

- 调用`stream.skip(n)` 正好相反：它会丢弃前n个元素。

> 例如，跳过第一个元素：

```java
Stream<String> words = Stream.of(contents.split("\\PL+")).skip(1);
```

- `stream.takeWhile(predicate)` 调用会在谓词为真时获取流中的所有元素，然后停止。

> 假设我们使用上一节的`codePoints`方法将字符串分割为字符，然后收集所有的数字元素。

```java
Stream<String> initialDigits = codePoint(str).takeWhile(
	s -> "0123456789".contains(s));
```

- `dropWhile`方法的做法正好相反，会在条件为真时丢弃元素，并产生一个由第一个使该条件为假的字符开头的所有元素构成的流：

```java
Stream<String> withoutInitialWhiteSpace = codePoints(str).dropWhile(
	s -> s.trim().length() == 0);
```

- 可以使用Stream类的静态`concat`方法将两个流连接起来：

```
Stream<String> sombined = Stream.concat(
	codePoints("Hello"), codePoints("World"));
	//Yields the stream ["H", "e", "l", "l", "0", "w", "0", "r", "l", "d"]
```

> 当然第一个流不应该是无限的，否则第二个流就永远都不会有机会处理。



## 五、其他的流转换

- `distinct`方法会返回一个流，它的元素是从原有流中产生的，即原来的元素按照同样的顺序剔除重复元素后产生的。这些重复元素并不一定是相邻的。

```java
Stream<String> uniqueWords
 	= Stream.of("merrily", "merruly","merrily","gently").distinct();
 	//Only one "merrily" is retained
```

> 对于流的排序，有多种sorted方法的变体可用。其中一种用于操作`Comparable`元素的流，而另一种可以接受一个`Comparator`。

- 对字符串排序，使得最长的字符串排在最前面;

```java
Stream<String> longestFirst
 	= words.stream().sorted(Comparator.comparing(String::length).reversed());
```

> 与所有的流一样，sorted方法会产生一个新的流，它的元素是原有流中按照顺序排列的元素。

- 我们在对集合排序时可以不使用流，但是，当排序处理是流管道的一部分时，sorted方法就会显得很有用。

- pack方法会产生另一个流，它的元素与原来流中的元素相同，但是在每一次获取一个元素时，都会调用一个函数。

```
Object[] powers = Stream.iterate(1.0, p -> p * 2)
	.peek(e -> System.out.println("Ferching" + e)
	.limit(20).toArray();
```



## 六、简单约简

- 约简是一种**终结操作**，它们会将流约简为可以在程序中使用的非流值。

> 一种简单的约简：`count`方法会返回流中元素的数量。

> 这些方法返回的是一个类型Optional<T> 的值，它要么在其中包装了答案，要么表示没有任何值（因为流碰巧为空）。

- 获得流中的最大值：

```java
Option<String> largest = words.max(String::compareToIgnoreCase);
System.out.println("largest:" +largest.orElse(""));
```

> `findFirst`返回的是非空集合中的第一个值。