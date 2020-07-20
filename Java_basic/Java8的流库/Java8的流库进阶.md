# Java8流库进阶

## 一、收集结果

- 当处理完流之后，通常想要查看其结果。此时可以调用`iterator`方法，它会产生用来访问元素的旧式风格的迭代器。

- 还可以调用`forEach`方法，将某个函数应用于每个元素：

```
stream.forEach(System.out::println);
```

> 在并行流上，`forEach`方法会以任意顺序遍历各个元素。

- 如果想要按照流中的顺序来处理它们，可以调用`forEachOrdered`方法。

> 这个方法会丧失并行处理的部分甚至全部优势。

- 如果想要将结果收集到数据结构中，可以调用`toArray`,获得由流的元素构成的数组。

> 因为无法在运行时创建泛型数组，所以表达式`stream.toArray()`会返回一个Object[ ]数组，如果想要让数组具有正确的类型，可以将其传递到数组构造器中：

```java
String[] result = stream.toArray(String[]::new);
	//stream.toArray() has type Object[]
```

- 针对将流中的元素收集到另一个目标中，有一个便捷方法`collect`可用，它会接受下一个`Collector`接口的实例。

> 收集器是一种收集众多元素并产生单一结果的对象，Collectors类型提供了大量用于生成常见收集器的工厂方法。

- 想要将流的元素收集到一个列表中，应该使用`Collectors.toList()`方法产生的收集器：

```java
List<String> result = stream.collect(Collectors.toList());
```

> 类似的，下面的代码展示了如何将流的元素收集到一个集中：

```java
Set<String> result = stream.collect(Collectors.toSet());
```

- 想要控制获得的集的种类，可以调用：

```java
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new));
```

- 想要通过连接操作来收集流中的所有字符串，可以调用：

```java
String result = stream.collect(Collectors.joining());
```

- 如果想要在元素之间增加分隔符，可以将分隔符传递给`joining`方法：

```java
String result = stream.collect(Collectors.joining(","));
```

- 如果流中包含除字符串以外的其他对象，那么需要先将其转换为字符串：

```java
String result = stream.map(Object::toString).collect(Colletors.joining(", "));
```

- 想要将流的结果约简为总和、数量、平均值、最大值、最小值，可以使用`summarizing(Int|Long|Double)`方法中的一个。这些方法接受一个将流对象映射为数值的函数，产生类型为`(Int|Long|Double)SummaryStatistics`的结果，同时计算总和、数量、平均值、最大值和最小值。

```java
IntSummaryStatistics summary = stream.collect(
	Collectors.summarizingInt(String::length));
double acerageWordLength = summary.getAverage();
double maxWordLength = summary.getMax();
```

## 二、收集到映射表中

- 假设有一个Stream<Person>, 想要将其元素收集到一个映射表中，这样后续就可以用过它们的ID来查找人员了。

- `Collectors.toMap`方法有两个函数引元，它们用来产生映射表的键和值。例如：

```java
Map<Integer, String> idToName = perple.collect(
	Collectors.toMap(Person::getId, Person::getName));
```

> 通常情况下，值应该是实际的元素，因此第二个函数可以使用`Function.identity()`;

```java
Map<Integer, Person> idToPerson = people.collect(
	Collectors.toMap(Person::getId, Function.identity()));
```

> 如果有多个元素具有相同的键，就会存在冲突，收集器将会抛出一个`IllegalStateException`异常。可以通过提供第3个函数引元来覆盖这种行为，该函数会针对给定的已有值和新值来解决冲突并确定键对应的值。这个函数应该返回已有值、新值或它们的组合。

- 假设想要了解给定国家的所有语言，就需要一个Map<String,Set<String>> 。为每种语言都存储一个单例集。找到了给定国家的新语言，就会对已有集和新集并行操作。

```java
Map<String, Set<String>> countryLanguageSets = locales.collect(
	Collectors.toMap(
		Locale::getDisplayCountry,
		l -> Collections.singleton(l.getDisplayLanguage)),
		(a, b) -> { //Union of a and b
			var union = new HashSet<String>(a);
			union.addAll(b);
			return union; }));
```

- 想要得到`TreeMap`,那么可以将构造器作为第4个引元来提供。必须提供一个合并函数。

```java
Map<Integer, Person> idToPerson = people.collect(
	Collectors.toMap(
		Person::getId,
		Function.identity(),
		(existingValue, newValue) -> { throw new IllegalStateException(); },
		TreeMap::new));
```

> 对于每一个`toMap`方法，都有一个等价的可以产生并发映射表的`toConcurrentMap`方法，单个并发映射表可以用于并行集合处理。当使用并行流时，共享的映射表比合并映射表更高效。注意，元素不再是按照流中的顺序收集的，但是通常这不会有什么问题。



## 三、群组和分区

- 将具有相同特性的值群聚成组是非常常见的，并且`groupingBy`方法直接就支持它。

- 看看通过国家聚成组Locale的问题。构建该映射表：

```java
Map<String, List<Locale>> countryTocales = locales.collect(
	Collectors.groupingBy(Locale::getCountry));
```

> 函数`Locale::getCountry`是群组的分类函数，可以查找给定国家代码对应的所有地点了，例如：

```java
List<Locale> swissLocales  = countryToLocales.get("CH");
 	// Yields locales de_CH, fr_CH, it_CH and maybe more
```

> 当分类函数式断言函数（即返回boolean值的函数）时，流的元素可以分为两个列表：该函数返回true的元素和其他的元素。这种情况下，使用`partitioningBy`比使用`groupingBy`更高效。例如，下面的代码中，将所有locale分成了使用英语和使用所有其他语言的两类:

```
Map<Boolean,List<Locale>> enlishAndOtherLocales = locales.collect(
	Collectors.partitioningBy(l -> l.getLanguage().equals("en")));
List<Locale> englishLocales = englishAndOtherLocales.get(true);
```

> 如果调用`groupingByConcurrent`方法，就会在使用并行流时获得一个被并行组装的并行映射表。这与`toConcurrentMap`方法完全类似。



## 四、下游收集器

- `groupingBy`方法会产生一个映射表，它的每个值都是一个列表。如果想要以某种方式来处理这些列表，就需要提供一个“下游处理器”。
- 如果想要获得集而不是列表，那么可以使用`Collectors.toSet`收集器：

```java
Map<String, Set<Locale>> countryToLocaleSet = locales.collect(
	groupingBy(Locale::getCountry, toSet()));
```

- [ ] Java提供了多种可以将收集到的元素约简为数字的收集器：

- counting 会产生收集到的元素的个数。

```java
Map<String, Long> countryToLocaleCounts = locales.collect(
	groupingBy(Locale::getCountry, counting()));
```

> 可以对每个国家有多少个`locale`进行计数。

- summing(Int|Long|Double) 会接受一个函数作为引元，将该函数应用到下游元素中，并产生它们的和。

```java
Map<String, Integer> stateToCityPopulation = cities.collect(
	groupingBy(City::getState, sumingInt(City::getPopulation)));
```

> 可以计算城市流中每个州的人口总和。

- `maxBy` 和 `minBy` 会接受一个比较器，并分别产生下游元素的最大值和最小值。

```java
Map<String, Optional<City>> stateToLargestCity = cities.collect(
	groupingBy(City::getState,
		maxBy(Comparator.comparing(City::getPopulation))));
```

> 可以产生每个州中最大的城市。

- `collectingAndThen`收集器在收集器后面添加了一个最终处理步骤。例如：我们想要知道有多少不同的结果，可以将它们收集到一个集中，计算尺寸：

```java
Map<Character, Integer> stringCountsByStartingLetter = strings.collect(
	groupingBy(s -> s.charAt(0),
		collectingAndThen(toSet(), Set::size)));
```

- mapping收集器的做法相反，它会将一个函数应用于收集到的每个元素，并将结果传递给下游收集器。

```java
Map<Character, Set<Integer> stringCountsByStartingLetter = strings.collect(
	groupingBy(s -> s.charAt(0),
    	mapping(String::length, toSet())));
```

> 这里按照首字符对字符串进行了分组，在每个组内部，计算字符串的长度，然后将这些长度收集到一个集中。

- 将收集器组合起来是一个很强大的方式，但是它也可能会产生非常复杂的表达式。最佳用法食欲`groupingBy`和`partitioningBy`一起处理“下游的”映射表中的值。否则应该直接在流上诸如 `map`、`reduce`、`count`、`max`或 `min`这样的方法。



## 五、约简操作

- `reduce`方法是一种用于从流中计算某个值的通用机制，其最简单的形式将接受一个二元函数，并从前两个元素开始持续应用它。

- 如果该函数是求和函数，那么就容易解释这种机制：

```java 
List<Integer> values = ... ;
Optional<Integer> sum = values.stream().reduce((x, y) -> x + y);
```

> 如果流为空，那么该方法会返回一个Optional, 因为没有任何有效的结果。上面的情况中，`reduce(Integer::sum)`而不是`reduce( ( x,y) -> x + y)`;

- 如果要用并行流来约简，那么这项约简操作必须是可结合的，即组合元素时使用的顺序不会产生任何影响。
- 通常会有一个幺元值，例如: `0`是加法的幺元值，可以使用第二种形式的`reduce`:

```java
List<Integer> values = ... ;
Integer sum = values.stream().reduce(0, (x, y) -> x + y);
	// Computes 0 + v + ...
```

> 如果流为空，则会返回幺元值，不需要处理`Optional`类了。



## 六、基本类型流

- 现在都是将整数收集到`Stream<Integer>` 中，每个整数都包装到包装器对象中却是很低效的。对其他基本类型来说，情况也一样，这些基本类型是：double、float、long、short、char、byte 和 boolean ;
- 流库中具有专门的类型 `IntStream`、`LongStream` 和 `DoubleStream`,可以直接存储基本类型值。而无需使用包装器。
- 如果想要存储`short`、`char`、`byte` 和 `boolean`， 可以使用 `IntStream`；而对于`float`,可以使用`DoubleStream`。

- 为了创建`IntStream`,需要调用`IntStream.of`和`Arrays`.stream方法：

```java
IntStream stream = IntStream.of(1,1,2,3,5);
stream = Arrays.stream(values, from, to);  //values is an int[] array
```

- 与对象流一样，可以使用静态的`generate` 和 `iterate`方法。`IntStream` 和 `LongStream` 有静态方法 `range` 和 `rangeClosed`， 可以生成步长为1的整数范围：

```java
IntStream zeroToNinetyNine = IntStream.range(0,100;) //Upper bound is excluded
IntStream zeroToHundred = IntStream.range(0,100;) //Upper bound is excluded
```

> `CharSequence` 接口拥有 `codePoints` 和 `chars` 方法，可以生成由字符的 `Unicode`码 或 `UTF-16`编码机制的码元构成的`IntStream`。

- 当你有一个对象流时，可以用`mapToInt`、`mapToLong` 或  `mapToDouble` 将其转换为基本类型流。

- [ ] 通常基本类型上的方法与对象流上的方法类似，有以下差异：

- `toArray`方法会返回基本类型数组。
- 产生可选结果的方法会返回一个`OptionalInt`、`OptionalLong` 或 `OptionalDouble`。这些类与`Optional`类类似，但是具有`getAsInt`、`getAsLong` 和 `getAsDouble`方法，而不是`get`方法。
- 具有分别返回总和、平均值、最大值和最小值的`sum`、`average`、`max` 和 `min` 方法。对象流没有定义这些方法。
- `summaryStatistics`方法会产生一个类型为`IntSummaryStatistics`、`LongSummaryStatistics`或`DoubleSummaryStatistaics`对象，它们可以同时报告流的总和，平均值、最大值和最小值。

> `Random`类具有`ints`、`longs`和`doubles`方法，它们会返回由随机数构成的基本类型流。如果需要的是并行流中的随机数，那么需要使用`SplittableRandom`类。



## 七、并行流

- 流使并行处理块操作变得很容易。这个过程几乎是自动的，但是需要遵守一些规则。

> 首先必须要有一个并行流。可以用`Collection.parallelStream()`方法从任何集合中获取一个并行流：

```java
Stream<String> paralleWords = words.parallelStream();
```

> 而且,`parallel`方法可以将任意的顺序流转换为并行流。

```java
Stream<String> parallelWords = Stream.of(wordArray).parallel();
```

> 只要在终结方法执行时流处于并行模式，所有的中间流操作就都将被并行化。

- 当流操作并行运行时，其目标是让其返回结果与顺序执行时返回的结果相同，重要的是，这些操作是无状态的，并且可以以任意顺序执行。
- 确保传递给并行流操作的任何函数都可以安全地并行执行，达到这个目的的最佳方式是远离易变状态。

> 默认情况下，从有序集合（数组和列表）、范围、生成器和迭代器产生的流，或者通过调用`Stream.sorted`产生的流，都是有序的。

- 排序并不排斥高效的并行处理。

> 例如：当计算`stream.map(fun)`时，流可以被划分为n部分，它们会被并行的处理。然后，结果将会按照顺序重新组装起来。

- 当放弃排序需求时，有些操作可以被更有效地并行化。

> 通过在流上调用`Stream.unordered`方法，可以明确表示我们对排序不感兴趣。

- [ ] 不要指望通过将所有流都转换为并行流就能够加速操作，记住下面几条：

- 并行化会导致大量的开销，只有面对非常大的数据才划算。
- 只有在底层的数据源可以被有效地分割为多个部分时，进流并行化才有意义。
- 并行流使用的线程池可能会因为诸如文件I/O或网络访问这样的操作被阻塞而饿死。