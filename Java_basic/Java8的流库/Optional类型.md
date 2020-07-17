# Optional类型

## 一、获取Optional值

- `Optional<T>` 对象是一种包装器对象，要么包装了类型T的对象，要么没有包装任何对象。

- 有效使用`Optional`的关键是要使用这样的方法：它的值不存在的情况下会产生一个可替代物，而只有在值存在的情况下才会使用这个值。
- 在没有任何匹配时，我们会希望使用某种默认值，可能是空字符串：

```java
String result = optionalString.orElse("");
	//The wrapped string, or "" if none
```

还可以调用代码来计算默认值：

```java
String result = optionalString.orElseGet(() -> System.getProperty("myapp.default"));
	//The function is only called when needed
```

可以在没有任何值时抛出异常

```
String result = optionalString.orElseThrow(IllegalStateException::new);
	// Supply a method that yields an exception object
```



## 二、消费Optional值

- `ifPresent`方法接受一个函数。如果可选值存在，它会被传递给该函数，否则，不会发生任何事情。

```
optionalValue.ifPresent(v -> Process v);
```

例如：如果在该值存在的情况下，想要将其添加到某个集中，可以调用：

```
optionValue.ifPresent(v -> results.add(v));
```

或者直接调用：

```java
optionalValue.ifPresent(result::add(v));
```

- 如果在可选值存在时执行一种动作，在可选值不存在时执行另一种动作，可以使用`ifPresentOrElse`:

```java
optionalValue.ifPresentOrElse(
	v -> System.out.println("Found" + v),
	() -> logger.warning("No match"));
```



## 三、管道化Optional值

- 保持Optional完整，使用map方法来转换内部值的：

```java
Optional<String> transformed = optionalString.map(String::toUpperCase);
```

>  如果`optionalString`为空，那么`transformed`也为空

- 将一个结果添加到列表中，如果它存在的话：

```
optionalValue.map(result::add);
```

> 如果`optionalValue`为空，则什么也不会发生。

- 可以使用`filter`方法来处理那些转换它之前或之后满足某种特定属性的`Optional`值，如果不满足该属性，那么管道会产生空的结果：

```java
Optional<String> transformed = optionalString
	.filter(s -> s.length() >= 8)
	.map(String::toUpperCase);
```

也可以用`or`方法将空`Optional`替换为一个可替代的`Optional`，这个可替代值将以惰性方式计算。

```java
Optional<String> result = optionalString.or(() ->   // Supply an Optional
	alternatives.stream().findFirst());
```

> 如果`OptionalString`的值存在，那么`result`为`optionalString`。如果值不存在，那么就会计算`lambda`表达式，并使用计算出来的结果。



## 四、不适合使用Optional值的方式

- `get`方法会在`Optional`值存在的情况下获得其中包装的元素，或者在不存在的情况下抛出一个`NoSuchElementException`异常，因此：

```java 
Option<T> optionalValue = ... ;
optionalValue.get().someMethod()
```

并不比下面的方式更安全：

```
T value = ... ;
value.someMethod();
```

`isPresent`方法会报告某个`Optional<T>` 对象是否有值。但是：

```java
if(optionalValue.isPresent()) optionalValue.get().someMethod();
```

并不比下面的方式更容易处理：

```
if(value != null) value.someMethod();
```

- [ ] 下面是一些有关`Optional`类型正确用法的提示：

- `Optional`类型的变量永远都不应该为`null`。

- 不要使用`Optional`类型的域，因为其代价是额外多出来一个对象。在类的内部，使用`null`表示缺失的域更易于操作。

- 不要在集合中放置`Optional`对象，并且不要将它们用作`map`，应该直接收集其中的值。



## 五、创建Optional值

- 要编写方法来创建`Optional`对象，可以用`Optional.of(result)` 和 `Optional.empty()`。

例如：

```java
public static Optional<Double> inverse(Double x){
	return x == 0 ? Optional.empty() : Optional.of(1 / x);
}
```

> `ofNullable` 方法被用来作为可能出现的null值和可选值之间的桥梁。`Optional.ofNullable(obj)`会在`obj`不为`null`的情况下返回 `Optional.of(obj).of(obj)`, 否则返回`Optional.empty()`。



## 六、用flat构建Optional值的函数

- 假设有一个可以产生`Option<T>` 对象的方法f,并且目标类型T具有一个可以产生`Optional<U>` 对象的方法g。 

> 如果它们都是普通方法，那么你可以用过调用`s.f().g()`来将它们组合起来，但是这种组合无法工作，因为`s.f()`的类型为`Option<T>`,  而不是`T`。因此需要调用：

```
Optional<U> result = s.f().flatMap(T::g);
```

> 如果 `s.f()` 的值存在，那么`g`就可以引用到它上面。否则就会返回一个空的`Optional<U>`。很明显，如果有更多可以产生`Optional`值的方法或`lambda`表达式，那么就可以重复此过程。你可以直接对`flatMap`的调用链接起来，从而构建由这些步骤构成的管道，只有所有步骤都成功，该管道才会成功。

- 例如：考虑前一节中安全的`inverse`方法。假设我们还有一个安全的平方根：

```java
public static Optional<Double> squareRoot(Double x){
	return x < 0 ? Optional.empty() : Option.of(Math.sqrt(x));
}
```

那么你可以像下面这样计算倒数的平方根：

```java
Optional<DOuble> result = inverse(x).flatMap(MyMath::squareRoot);
```

或者你可以选择下面的方式：

```java
Optional<Double> result 
	= Optional.of(-4.0).flatMap(Demo::inverse).flatMap(Demo::squareRoot);	
```

> 无论是`inverse`方法还是`squareRoot`方法返回`Optional.empty()`, 整个结果都会为空。



## 七、将Optional转换为流。

- stream方法会将一个Optional<T> 对象转换为一个具有0个或1个元素的Stream<T> 对象。

> 这会使返回Optional结果的方法变得很有用。假设我们有一个用户ID流和下面的方法：

```
Optional<User> lookup(String id)
```

> 怎样才能在获取用户流时，过滤调用无效ID。可以过滤掉无效ID，然后将get方法应用于剩余的ID：

```
Stream<String> ids = ... ;
Stream<User> users = ids.map(Users::lookup)
	.filter(Optional::isPresent)
	.map(Optional::get);
```

> 但是这样就需要使用我们之前要慎用的`isPresent`和`get`方法，下面的调用更优雅：

```
Stream<User> users = ids.map(Users::lookup)
	.flatMap(Optional::stream);
```

> 每一个对`stream`的调用都会返回一个具有0或1个元素的流。`flagMap`方法将这些方法组合在一起，这意味着不存在的用户会直接被丢弃。