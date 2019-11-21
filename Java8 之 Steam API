Java Stream API 是 Java 8 引入的函数式编程API


使用stream前：

```
List<String> myList = Lists.newArrayList(
        "bcd", "cde", "def", "abc");
List<String> result = Lists.newArrayListWithCapacity(myList.size());
for (String str : list) {
    if (str.length() >= 3) {
        char e = str.charAt(0);
        String tempStr = String.valueOf(e);
       result.add(tempStr);
    }
}
```

使用stream后：

```
List<String> myList = Lists.newArrayList(
        "bcd", "cde", "def", "abc");
List<String> result = myList.stream()
        .filter(e -> e.length() >= 3)
        .map(e -> e.charAt(0))
        .map(e -> String.valueOf(e))
        .collect(Collectors.toList());
```

Stream的优势
- 提升性能：stream会记录下过程操作、并对这些操作进行叠加，最后在一个迭代循环中执行所有叠加的操作，减少迭代次数
- 代码简洁：函数式编程写出的代码简洁且意图明确，使用stream接口让你从此告别for循环。
- 多核友好：只需要调用一下parallel()方法，即可实现并行程序，简化编码

流式操作中使用到了很多内置函数式接口，我们使用的时候只要使用Lambda表达式实现这些内置函数式接口就行

Java8 函数式编程 / Lambda表达式 之前已介绍过，常见的内置函数式接口如下：
(内置函数式接口)

## 创建Stream

Stream 可以从不同的数据类型创建，尤其是集合（Collection）；但不能从Map创建。

你可以直接通过调用 List 和 Set 的 stream() 方法和 parallelStream() 方法。其中 parallenStream() 会采用多线程执行。

从对象创建stream：

```
    // 从List创建
    Arrays.asList("a1", "a2", "a3")
            .stream()
            .forEach(s -> System.out.print(s + ","));
    // a1,a2,a3,

    // 直接从Stream创建
    Stream.of("a1", "a2", "a3")
        .forEach(s -> System.out.print(s + ","));
    // a1,a2,a3,
```

还可以使用 IntStream LongStream DoubleStream 从基本类型创建 Stream。基本类型stream支持一些特殊的“最终结果API”，比如 sum() average() max()；

```
    IntStream.range(1, 3)
            .max(); //3
```

普通对象 Stream 可以通过 mapToInt() mapToLong() mapToDouble() 转换成基本类型 Stream：

```
Stream.of("s1", "s2", "s3")
    .map(s -> s.substring(1))
    .mapToInt(Integer::parseInt)
    .max()
    .ifPresent(System.out::println); // 3
```

基本类型可以通过 mapToObject() 转换成普通对象 Stream：

```
IntStream.range(1, 4)
    .mapToObj(i -> "s" + i)
    .forEach(System.out::println);
// s1
// s2
// s3
```

## Stream操作

stream操作的特点：
- non-interfering：stream操作不会修改原始的数据。比如文章开始的例子，stream操作不会改变 myList，迭代结束之后，myList 还是保持着原来的样子。
- stateless：操作都是无状态的，不依赖外面的变量。因此在stream操作内引用外部非final变量会报异常。
- stream中会记录下过程操作、并对这些操作进行叠加，最后在一个迭代循环中执行所有叠加的操作

对stream的操作分为为两类：
- 中间操作：总是会惰式执行，调用中间操作只会生成一个标记了该操作的新stream，仅此而已。中间操作的结果扔是Stream，可以继续使用 Stream API 连续调用；
    - 无状态（Stateless）操作：元素的处理不受之前元素的影响
    - 有状态（Stateful）操作：该操作只有拿到所有元素之后才能继续下去
- 结束操作：会触发实际计算，计算发生时会把所有中间操作积攒的操作以pipeline的方式执行，这样可以减少迭代次数。结束操作的结果要么是 void ，要么是一个非 Stream 结果；计算完成之后stream就会失效。
    - 短路：遇到某些符合条件的元素就可以得到最终结果
    - 非短路操作：必须处理所有元素才能得到最终结果。

(图)

### 常用中间操作

#### ForEach


#### Filter

函数原型为

```
Stream<T> filter(Predicate<? super T> predicate)
```

作用是过滤出满足predicate条件的元素

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count(); // 2
```

#### Sorted

函数原型为

```
Stream<T> sorted(Comparator<? super T> comparator)
```

作用是对列表中的元素排序。

```
List<String> myList = Arrays.asList("1", "2", "3");
myList.stream()
        .sorted(Comparator.comparingInt(Integer::parseInt))
        .forEach(System.out::println);

// 1
// 2
// 3
```

#### Map

函数原型为

```
Stream<R> map(Function<? super T,? extends R> mapper)
```

作用是对容器中的每个元素按照mapper操作进行转换，转换前后Stream中元素的个数不会改变，但元素的类型取决于转换之后的类型。

```
List<String> myList = Arrays.asList("a1", "a2", "a3");
myList.stream()
    .map(s -> Integer.parseInt(s.subString(1)))
    .forEach(System.out::println);
// 1
// 2
// 3
```

#### flatMap
map 方法只能把一个对象转换成另一个对象；如果需要将一个对象转换成多个，则需要用 flatMap。

flatMap 函数原型为 

```
Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)
```

作用是对每个元素执行mapper指定的转换操作，转换前后元素的个数和类型都可能会改变。

```
Stream<List<Integer>> integerListStream = Stream.of(
  Arrays.asList(1, 2), 
  Arrays.asList(3, 4), 
  Arrays.asList(5)
);
Stream<Integer> integerStream = integerListStream .flatMap(list -> list.stream());
// 或：Stream<Integer> integerStream = integerListStream .flatMap(Collection::stream);
integerStream.forEach(System.out::println);

// 1
// 2
// 3
// 4
// 5
```

### 常用结束操作

#### ForEach

函数原型为 

```
void forEach(Consumer<? super T> action)
```

作用是对每一个元素的执行action操作

```
List<String> myList = Arrays.asList("1", "2", "3");
myList.stream()
    .forEach(System.out::println);
// 1
// 2
// 3
```

#### Collector

collect 方法接收一个 Collector，Collector 可以将 Stream 转换成集合结果，比如 List Set Map。

JDK内置了常用的 Collector，所以大多数情况下我们不需要自己实现：


```
List<String> myList = Arrays.asList("a1", "b2", "c3");
List<String> filtered = myList.stream()
                                .filter(s -> !s.startsWith("a"))
                                .collect(Collectors.toList());
System.out.println(filtered); // [b2, c3]

List<String> myList = Arrays.asList("a1", "b2", "c3");
String merged = myList.stream()
                            .filter(s -> !s.startsWith("a"))
                            .collect(Collectors.joining(","));
System.out.println(merged); // b2,c3
```

将 Stream 元素转换成 map 的时候，需要特别注意：key 必须是唯一的，否则会抛出 IllegalStateException 。但是我们可以传入一个 merge function，来指定重复的元素映射的方式：

```
List<String> myList = Arrays.asList("a1", "b2", "c3", "c4");
Map<String, String> map =
        myList.stream()
                .collect(Collectors.toMap(
                        s -> s.substring(0,1), // keyMapper
                        s -> s.substring(1,2), // valueMapper
                        (val1, val2) -> val1 + "," + val2) // mergeFunction：合并key重复元素
                );

System.out.println(map); // {a=1, b=2, c=3,4}

```

自定义 Collector，需要提供4个入参：

```
supplier：Supplier<R> // 创建新的结果容器
accumulator：BiConsumer<R, T> // 将元素添加到结果容器
combiner：BinaryOperator<R> // 将两个并行执行的结果容器合并为一个结果容器。并行执行(parallelStream)时才会用到，非并行执行(stream)不会被调用到
finisher：Function<A, R> // 对结果容器变换为整个集合操作的最终结果
```

使用

```
List<String> myList = Arrays.asList("1", "2", "3");
String joined = myList.stream()
        .sorted(Comparator.comparingInt(Integer::parseInt))
        // .collect(Collectors.joining("|"));
        .collect(Collector.of
                (() -> new StringJoiner("|"), // supplier：创建一个空的累加器实例
                        ((joiner, s) -> joiner.add(s)), // accumulator：第一个参数是累计值，第二参数是第n个元素。累加值与元素n如何做运算就是accumulator做的事情
                        (joiner1, joiner2) -> joiner1.add(joiner2.toString()),  // combiner：并行执行后才会调用，在combiner中对并行流的各个子部分的累加值结果合并为一个累加值结果
                        joiner -> joiner.toString() // finisher：参数是累加值，返回的结果就是最终的结果。finisher就是将累加器对象转换为整个集合操作的最终结果
                ));
                
System.out.println(joined); // 1|2|3
```

#### Reduce

reduce操作可以实现从一组元素中生成一个值，sum()、max()、min()、count()等都是reduce操作，将他们单独设为函数只是因为常用。

Reduce 的函数原型为 Optional<T> reduce(BinaryOperator<T> accumulator), 作用是可以将所有的元素变成一个结果，该操作结果会放在一个Optional变量里返回。

reduce()的方法定义有三种重载形式：主要的参数 accumulator 用于把多个元素合并成一个；其他多的参数只是为了指明初始值（参数identity），或者是指定并行执行时多个部分结果的合并方式（参数combiner）。

```
Optional<T> reduce(BinaryOperator<T> accumulator)
reduce(T identity, BinaryOperator<T> accumulator)
U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)
```


第一种可以将 Stream 中的元素聚合成一个：

```
List<String> myList = Arrays.asList("a1", "b2", "c3");
Optional<String> reduced =
    myList.stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);
reduced.ifPresent(System.out::println);

// a1#b2#c3
```

第二种可以多接收一个初始对象。通常可以用于聚合操作（比如累加）。

```
int sum = Stream.of(1, 2, 3, 4)
                .reduce(
                    0,  // 初始值 identity
                    (acc, element) -> acc + element // 合并元素操作 accumulator
                );

System.out.println(sum); // 10
```

第三种可以多接收一个初始对象和一个并行操作时的combiner函数。

```
// 求单词长度之和
Stream<String> stream = Stream.of("a", "bb", "ccc", "dddd");
int lengthSum = stream.reduce(0, // 初始值　// (1)
    (sum, str) -> sum+str.length(), // 累加器 // (2)
    (a, b) -> a+b); // 部分结果拼接器，并行执行(parallelStream)时才会用到，非并行执行(stream)不会被调用到 // (3)
// 等同于 int lengthSum = stream.mapToInt(String::length).sum();

System.out.println(lengthSum); // 10
```

#### Match

用来判断某一种规则是否与流对象匹配。所有的匹配操作都是终结操作，只返回一个boolean类型的结果。

```
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));
System.out.println(anyStartsWithA); // true

boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));
System.out.println(allStartsWithA); // false

boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));
System.out.println(noneStartsWithZ); // true
```

## Parallel Streams 

流操作可以是顺序的，也可以是并行的。顺序操作通过单线程执行，而并行操作则通过多线程执行。

Parallel Stream 底层使用公共的 ForkJoinPool 来并行计算，底层的真正的线程数据取决于 CPU 的核数，默认是3。

Collections 可以通过 parallelStream() 来创建一个并行执行的 Stream；也可以在普通的 Stream 上执行 parallel() 来转换成并行执行的 Stream。

下面这个例子，将并行执行的每一步的线程执行者打印出来：

```
Arrays.asList("a1", "a2", "b1", "c2", "c1")
    .parallelStream()
    .filter(s -> {
        System.out.format("filter: %s [%s]\n",
            s, Thread.currentThread().getName());
        return true;
    })
    .map(s -> {
        System.out.format("map: %s [%s]\n",
            s, Thread.currentThread().getName());
        return s.toUpperCase();
    })
    .forEach(s -> System.out.format("forEach: %s [%s]\n",
        s, Thread.currentThread().getName()));
```

输出如下，展示了每一步都是由哪一个线程来执行的：

```
filter: b1 [main]
filter: c2 [ForkJoinPool.commonPool-worker-4]
map: c2 [ForkJoinPool.commonPool-worker-4]
filter: c1 [ForkJoinPool.commonPool-worker-2]
filter: a1 [ForkJoinPool.commonPool-worker-3]
map: a1 [ForkJoinPool.commonPool-worker-3]
filter: a2 [ForkJoinPool.commonPool-worker-1]
map: a2 [ForkJoinPool.commonPool-worker-1]
forEach: A1 [ForkJoinPool.commonPool-worker-3]
map: c1 [ForkJoinPool.commonPool-worker-2]
forEach: C2 [ForkJoinPool.commonPool-worker-4]
map: b1 [main]
forEach: C1 [ForkJoinPool.commonPool-worker-2]
forEach: A2 [ForkJoinPool.commonPool-worker-1]
forEach: B1 [main]
```

由于Parallel Stream 底层使用的通用的 ForkJoinPool，所以需要注意不要在并行的 Stream 中出现很慢或阻塞的操作，这样会影响其他并行任务。

## 参考

- https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html
- https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html
- https://www.kawabangga.com/posts/3698

（文末）


