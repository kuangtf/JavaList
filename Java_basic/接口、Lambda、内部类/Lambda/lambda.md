
#  lambda
## 一、为什么使用lambda表达式
**1.定义：**  lambda表达式是一个可传递的代码块，可以在以后执行一次或多次。
> 在java中传递一个代码块并不容易，不能直接传递代码块。必须构造一个对象，这个对象的类需要有一个方法包含所需的代码。使用lambda表达式，可以解决冗余的代码。

**2.lambda表达式的语法**：参数，箭头（->）, 一个表达式。
```
(参数类型 参数名称) ‐> { 代码语句 }
```
> 注意：即使lambda表达式没有参数，仍然要提供空括号，就像无参方法一样：
```
( ) ‐> { 代码语句 }
```
> 注意：
1.如果编译器可以推导出参数的类型，则参数的类型可以省略；如果方法只有一个参数，且类型可以推导出，那么可以省略小括号和参数类型。
2.无需指定lambda表达式的返回类型。
3.如果一个lambda表达式只在某些分支返回值，而另一些分支不返回值，这是不合法的。

```
（int x）-> {
         if(x>=0) {
              return 1;          //这是不合法的
          }
   }
```

## 二、函数式接口
**1.定义：**  对于只有一个抽象方法的接口，成为函数式接口。
> 注意：接口中可以含有其他方法。需要这种接口的对象时，就可以提供一个lambda表达式。最好把lambda表达式看成是一个函数。

**2.使用lambda表达式的前提：**  
- 1. 使用Lambda必须具有接口，且要求接口中有且仅有一个抽象方法。
无论是JDK内置的  ```Runnable 、Comparator```  接口还是自定义的接口，只有当接口中的抽象方法存在且唯一
时，才可以使用Lambda。
- 2. 使用Lambda必须具有上下文推断。
也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为该接口的实例。

## 三、方法引用
**例子：**  假设你希望只要出现一个定时器就打印这个事件对象。
- 1.使用lambda表达式：```var timer = new Timer(1000, event -> System.out.println(event));```
- 2.使用方法引用：  ```var timer = new Timer(1000, System.out:: println);```
* 表达式 System.out **::** println  就是一个方法引用，它只是编译器生成一个函数式接口的实例，覆盖这个接口的抽象方法来调用给定的方法。
> 注意：
- 1.类似于lambda表达式，方法引用也不是一个对象。不过，为一个类型为函数式接口的变量赋值时会生成一个对象。
- 2.只有当lambda表达式的体只调用一个方法而不做其他操作时，才能把lambda表达式重写为方法引用。
- 3.例如:```s -> s.length==0```   , 这里有一个方法调用和比较，所以不能使用方法引用。
- 4.如果有多个重名的重载方法，编译器就会尝试从上下文中找出你所指定的是哪一个方法。
- 5.包含对象的方法引用于等价的lambda表达式还有一个细微的差别。考虑一个方法引用，如 ```separator::equals``` 。如果 ```separator``` 为null，构造 ```separator::equals``` 就会立即抛出一个 ```NullPointExceptoin``` 异常。而lambda表达式只有在调用是才会抛出  ```NullPointExceptoin``` 异常。

## 四、构造器引用
- 1.构造器引用与方法引用很相似，只不过方法名为new 。
> 例如：Person :: new 是Person构造器的一个引用。

- 2.可以用数组建立构造器引用。
> 例如：int [ ] :: new  是一个构造器引用，它有一个参数：即数组的长度，等价于lambda表达式  x -> new int[x]。

**注意：**  无法构造泛型类型T的数组， 表达式 new T [ n ] 会产生错误， 这回改变为 new Object [n] 。如果想要得到一个Person类型的数组。可以这样做：
~~~
Person [ ]   person = stream.toArray ( Person [ ] :: new )
~~~

## 五、变量作用域
- 1. 考虑小面这个例子：
~~~
public static void repeatMessage(String text, int delay)
{
       ActionListener listener = event -> 
             {
                 System.out.println(text);
                  Toolkit.getDefaultToolkit( ).beep( );
              };
         new Timer(delay, listener).start( );
~~~

> lambda 表达式中的text 变量并不是在 lambda 表达式中定义的，而是 repeatMessage 方法的一个参数变量。可能 lambda表达式在 repeatMessage 调用很久以后才执行， 此时那个text 变量已经不存在了 ， 那么怎么保留 text 变量呢？ 其实 text 被 lambda表达式捕获了。
注释：关于代码块以及自由变量值有一个术语： 闭包 。   在  java 中， lambda表达式就是闭包 。

- 由上面的例子可以看到 ，lambda表达式可以 外围作用域中变量的值 ，但是只能引用值不会改变的变量 ，例如，下面的代码是不合法的：
~~~
public static void countDown(int start, int delay)
{
        ActionListener  listener = event -> 
             {
                  start -- ;   //error           在lambda 表达式内部改变 
                   System.out.println(start);
             };
        new Timer(delay  ,  listener ).start( );
}


public static void repeat (String text , int count)
{
         for ( int i = 1 ; i <= count ;  i++)            //  在lambda表达式的外部发生改变 。  
          {
                    ActionListener  listener = event -> 
                       {
                                System.out.println( i + "  :  "  + text );
             };
        new Timer(   1000,  listener ).start( );
}
~~~

>  这里有一条规则 ： lambda表达式捕获的变量 必须是 **事实最终变量**   。  **事实最终变量**   是指这个变量初始化之后就不能为 这个变量 赋新值 。

- 2. lambda表达式的体 与嵌套块有相同的作用域 。

  lambda表达式中声明一个与局部变量同名的 参数 或局部变量是不和法的 。 lambda表达式中不允许有 同名的局部变量 。在lambda表达式中使用this关键字 时， 是指创建这个 lambda表达式的方法的 this 参数 。
~~~
Path first = Path.of( " /usr/bin" ) ;
Comparator<String> comp =  ( first , second )  -> first.length( ) - second.length( ) ;
~~~

##  六、处理lambda表达式 
- 1. lambda表达式的重点是延迟执行 。
**例如：**  
~~~
           1. 在一个单独的线程中运行代码。
           2.多次运行代码。
           3.在算法的适当位置运行代码 。 （例如排序中的比较操作）
           4.发生某种情况时执行代码 。 （例如 点击了一个按钮 ）
           5.只在必要是才运行代码 。
~~~
这时，lambda表达式是一个很好的选择。
>  注释：如果自己设计一个函数式接口，最好使用 @FunctionalInterface 来标记这个接口，如果 无意中增加了另一个 抽象方法，编译器 将会报错。
