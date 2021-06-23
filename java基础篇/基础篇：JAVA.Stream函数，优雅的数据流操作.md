# 前言
平时操作集合数据，我们一般都是for或者iterator去遍历，不是很好看。java提供了Stream的概念，它可以让我们把集合数据当做一个个元素在处理，并且提供多线程模式

-	流的创建
-	流的各种数据操作
-	流的终止操作
-	流的聚合处理
-	并发流和CompletableFuture的配合使用

**关注公众号，一起交流，微信搜一搜: 潜行前行**
---

# 1 stream的构造方式
## stream内置的构造方法
```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
public static<T> Builder<T> builder()
public static<T> Stream<T> of(T t)
public static<T> Stream<T> empty()
public static<T> Stream<T> generate(Supplier<T> s)
```
## Collection声明的stream函数
```java
default Stream<E> stream()
```
-	Collection声明了stream转化函数，也就是说，任意Collection子类都存在官方替我们实现的由Collection转为Stream的方法
-	示例，List转Stream
```java
public static void main(String[] args){
    List<String> demo =  Arrays.asList("a","b","c");
    long count = demo.stream().peek(System.out::println).count();
    System.out.println(count);
}
-------result--------
a
b
c
3
```

# 2 接口stream对元素的操作方法定义
## 过滤 filter
```java
Stream<T> filter(Predicate<? super T> predicate)
```
-	Predicate是函数式接口，可以直接用lambda代替；如果有复杂的过滤逻辑，则用or、and、negate方法组合
-	示例
```java
List<String> demo = Arrays.asList("a", "b", "c");
Predicate<String> f1 = item -> item.equals("a");
Predicate<String> f2 = item -> item.equals("b");
demo.stream().filter(f1.or(f2)).forEach(System.out::println);
-------result--------
a
b
```

## 映射转化 map
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
IntStream mapToInt(ToIntFunction<? super T> mapper);
LongStream mapToLong(ToLongFunction<? super T> mapper);
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
```
-	示例
```java
static class User{
    public User(Integer id){this.id = id; }
    Integer id; public Integer getId() {  return id; }
}
public static void main(String[] args) {
    List<User> demo = Arrays.asList(new User(1), new User(2), new User(3));
    // User 转为 Integer(id)
    demo.stream().map(User::getId).forEach(System.out::println);
}
-------result--------
1
2
3
```
## 数据处理 peek
```java
Stream<T> peek(Consumer<? super T> action);
```
-	与map的区别是其无返回值
-	示例
```java
static class User{
    public User(Integer id){this.id = id; }
    Integer id;
    public Integer getId() {  return id; }
    public void setId(Integer id) {  this.id = id; }
}
public static void main(String[] args) {
    List<User> demo = Arrays.asList(new User(1), new User(2), new User(3));
    // id平方，User 转为 Integer(id)
    demo.stream().peek(user -> user.setId(user.id * user.id)).map(User::getId).forEach(System.out::println);
}
-------result--------
1
4
9
```

## 映射撵平 flatMap
```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);
LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);
DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);
```
-	flatMap：将元素为Stream\<T>类型的流撵平成一个元素类型为T的Stream流
-	示例
```java
public static void main(String[] args) {
    List<Stream<Integer>> demo = Arrays.asList(Stream.of(5), Stream.of(2), Stream.of(1));
    demo.stream().flatMap(Function.identity()).forEach(System.out::println);
}
-------result--------
5
2
1
```

## 去重 distinct
```java
Stream<T> distinct();
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 1, 2);
demo.stream().distinct().forEach(System.out::println);
-------result--------
1
2
```

## 排序 sorted
```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```
-	示例
```java
List<Integer> demo = Arrays.asList(5, 1, 2);
//默认升序
demo.stream().sorted().forEach(System.out::println);
//降序
Comparator<Integer> comparator = Comparator.<Integer, Integer>comparing(item -> item).reversed();
demo.stream().sorted(comparator).forEach(System.out::println);
-------默认升序 result--------
1
2
5
-------降序 result--------
5
2
1
```

## 个数限制limit和跳过skip
```java
//截取前maxSize个元素
Stream<T> limit(long maxSize);
//跳过前n个流
Stream<T> skip(long n);
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3, 4, 5, 6);
//跳过前两个，然后限制截取两个
demo.stream().skip(2).limit(2).forEach(System.out::println);
-------result--------
3
4
```

## JDK9提供的新操作
-	和filter的区别，takeWhile是取满足条件的元素，直到不满足为止；dropWhile是丢弃满足条件的元素，直到不满足为止
```java
default Stream<T> takeWhile(Predicate<? super T> predicate);
default Stream<T> dropWhile(Predicate<? super T> predicate);
```

# 3 stream的终止操作action
## 遍历消费
```java
//遍历消费
void forEach(Consumer<? super T> action);
//顺序遍历消费,和forEach的区别是forEachOrdered在多线程parallelStream执行，其顺序也不会乱
void forEachOrdered(Consumer<? super T> action);
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3);
demo.parallelStream().forEach(System.out::println);
demo.parallelStream().forEachOrdered(System.out::println);
-------forEach result--------
2
3
1
-------forEachOrdered result--------
1
2
3
```

## 获取数组结果
```java
//流转成Object数组
Object[] toArray();
//流转成Ａ[]数组，指定类型A
<A> A[] toArray(IntFunction<A[]> generator)
```
-	示例
```java
List<String> demo = Arrays.asList("1", "2", "3");
//<A> A[] toArray(IntFunction<A[]> generator)
String[] data = demo.stream().toArray(String[]::new);
```

## 最大最小值
```java
//获取最小值
Optional<T> min(Comparator<? super T> comparator)
//获取最大值
Optional<T> max(Comparator<? super T> comparator)
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3);
Optional<Integer> min = demo.stream().min(Comparator.comparing(item->item));
Optional<Integer> max = demo.stream().max(Comparator.comparing(item->item));
System.out.println(min.get()+"-"+max.get());
-------result--------
1-3
```

## 查找匹配
```java
//任意一个匹配
boolean anyMatch(Predicate<? super T> predicate)
//全部匹配
boolean allMatch(Predicate<? super T> predicate)
//不匹配 
boolean noneMatch(Predicate<? super T> predicate)
//查找第一个
Optional<T> findFirst();
//任意一个
Optional<T> findAny();
```

## 归约合并
```java
//两两合并
Optional<T> reduce(BinaryOperator<T> accumulator)
//两两合并，带初始值的
T reduce(T identity, BinaryOperator<T> accumulator)
//先转化元素类型再两两合并，带初始值的
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
//数字转化为字符串，然后使用“-”拼接起来
String data = demo.stream().reduce("0", (u, t) -> u + "-" + t, (s1, s2) -> s1 + "-" + s2);
System.out.println(data);
-------result--------
0-1-2-3-4-5-6-7-8
```

## 计算元素个数
```java
long count()
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3, 4, 5, 6);
System.out.println(demo.stream().count());
-------result--------
6
```

## 对流的聚合处理
```java
/**
 * supplier:返回结果类型的生产者
 * accumulator:元素消费者（处理并加入R）
 * combiner: 返回结果 R 怎么组合（多线程执行时，会产生多个返回值R，需要合并）
 */
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);
/**
 * collector一般是由 supplier、accumulator、combiner、finisher、characteristics组合成的聚合类
 * Collectors 可提供一些内置的聚合类或者方法
 */
<R, A> R collect(Collector<? super T, A, R> collector);
```
-	示例，看下面

# 4 Collector(聚合类)的工具类集Collectors
## 接口Collector和实现类CollectorImpl
```java
//返回值类型的生产者
Supplier<A> supplier();
//流元素消费者
BiConsumer<A, T> accumulator();
//返回值合并器（多个线程操作时，会产生多个返回值，需要合并）
BinaryOperator<A> combiner();
//返回值转化器（最后一步处理，实际返回结果，一般原样返回）
Function<A, R> finisher();
//流的特性
Set<Characteristics> characteristics();

public static<T, A, R> Collector<T, A, R> of(Supplier<A> supplier,
	BiConsumer<A, T> accumulator, BinaryOperator<A> combiner,
	Function<A, R> finisher, Characteristics... characteristics)
```

## 流聚合转换成List, Set
```java
//流转化成List
public static <T> Collector<T, ?, List<T>> toList()
//流转化成Set
public static <T> Collector<T, ?, Set<T>> toSet()
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3);
List<Integer> col = demo.stream().collect(Collectors.toList());
Set<Integer> set = demo.stream().collect(Collectors.toSet());
```

## 流聚合转化成Map
```java
//流转化成Map
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(
	Function<? super T, ? extends K> keyMapper,
    Function<? super T, ? extends U> valueMapper)
/**
 * mergeFunction:相同的key,值怎么合并
 */
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(
	Function<? super T, ? extends K> keyMapper,
	Function<? super T, ? extends U> valueMapper,
    BinaryOperator<U> mergeFunction)
/**
 * mergeFunction:相同的key,值怎么合并
 * mapSupplier：返回值Map的生产者
 */
public static <T, K, U, M extends Map<K, U>> Collector<T, ?, M> toMap(
	Function<? super T, ? extends K> keyMapper,
	Function<? super T, ? extends U> valueMapper,
	BinaryOperator<U> mergeFunction,
    Supplier<M> mapSupplier)
```
-	如果存在相同key的元素，会报错;或者使用groupBy
-	示例
```java
List<User> demo = Arrays.asList(new User(1), new User(2), new User(3));
Map<Integer,User> map = demo.stream().collect(Collectors.toMap(User::getId,item->item));
System.out.println(map);
-------result-------
{1=TestS$User@7b23ec81, 2=TestS$User@6acbcfc0, 3=TestS$User@5f184fc6}
```

## 字符串流聚合拼接
```java
//多个字符串拼接成一个字符串
public static Collector<CharSequence, ?, String> joining();
//多个字符串拼接成一个字符串（指定分隔符）
public static Collector<CharSequence, ?, String> joining(CharSequence delimiter)
```
-	示例
```java
List<String> demo = Arrays.asList("c", "s", "c","w","潜行前行");
String name = demo.stream().collect(Collectors.joining("-"));
System.out.println(name);
-------result-------
c-s-c-w-潜行前行
```


## 映射处理再聚合流
-	相当于先map再collect
```java
/**
 * mapper:映射处理器
 * downstream:映射处理后需要再次聚合处理
 */
public static <T, U, A, R> Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper, 
		Collector<? super U, A, R> downstream);
```
-	示例
```java
List<String> demo = Arrays.asList("1", "2", "3");
List<Integer> data = demo.stream().collect(Collectors.mapping(Integer::valueOf, Collectors.toList()));
System.out.println(data);
-------result-------
[1, 2, 3]
```


## 聚合后再转换结果
```java
/**
 * downstream:聚合处理
 * finisher:结果转换处理
 */
public static<T,A,R,RR> Collector<T,A,RR> collectingAndThen(Collector<T,A,R> downstream,
		Function<R, RR> finisher); 
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 3, 4, 5, 6);
//聚合成List,最后提取数组的size作为返回值
Integer size = demo.stream().collect(Collectors.collectingAndThen(Collectors.toList(), List::size));
System.out.println(size);
---------result----------
6
```

## 流分组（Map是HashMap）
```java
/**
 * classifier 指定T类型某一属性作为Key值分组
 * 分组后，使用List作为每个流的容器
 */
public static <T, K> Collector<T, ?, Map<K, List<T>>> groupingBy(
		Function<? super T, ? extends K> classifier);           
/**
 * classifier: 流分组器
 * downstream: 每组流的聚合处理器
 */
public static <T, K, A, D> Collector<T, ?, Map<K, D>> groupingBy(
		Function<? super T, ? extends K> classifier， 
		Collector<? super T, A, D> downstream)
/**
 * classifier: 流分组器
 * mapFactory: 返回值map的工厂（Map的子类）
 * downstream: 每组流的聚合处理器
 */
public static <T, K, D, A, M extends Map<K, D>> Collector<T, ?, M> groupingBy(
		Function<? super T, ? extends K> classifier,
		Supplier<M> mapFactory,
		Collector<? super T, A, D> downstream)
```
-	示例
```java
public static void main(String[] args) throws Exception {
    List<Integer> demo = Stream.iterate(0, item -> item + 1)
            .limit(15)
            .collect(Collectors.toList());
    // 分成三组，并且每组元素转化为String类型        
    Map<Integer, List<String>> map = demo.stream()
            .collect(Collectors.groupingBy(item -> item % 3,
                    HashMap::new,
                    Collectors.mapping(String::valueOf, Collectors.toList())));
    System.out.println(map);
}
---------result----------    
{0=[0, 3, 6, 9, 12], 1=[1, 4, 7, 10, 13], 2=[2, 5, 8, 11, 14]}    
```


## 流分组(分组使用的Map是ConcurrentHashMap)
```java
/**
 * classifier: 分组器 ； 分组后，使用List作为每个流的容器
 */
public static <T, K> Collector<T, ?, ConcurrentMap<K, List<T>>> groupingByConcurrent(
		Function<? super T, ? extends K> classifier);
/**
 * classifier: 分组器
 * downstream: 流的聚合处理器
 */
public static <T, K, A, D> Collector<T, ?, ConcurrentMap<K, D>> groupingByConcurrent(
		Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream)
/**
 * classifier: 分组器
 * mapFactory: 返回值类型map的生产工厂（ConcurrentMap的子类）
 * downstream: 流的聚合处理器
 */
public static <T, K, A, D, M extends ConcurrentMap<K, D>> Collector<T, ?, M> groupingByConcurrent(
		Function<? super T, ? extends K> classifier, 
		Supplier<M> mapFactory,
		Collector<? super T, A, D> downstream);
```
-	用法和groupingBy一样

## 拆分流，一变二（相当于特殊的groupingBy）
```java
public static <T> Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(
		Predicate<? super T> predicate)
/**
 * predicate: 二分器
 * downstream: 流的聚合处理器
 */
public static <T, D, A> Collector<T, ?, Map<Boolean, D>> partitioningBy(
		Predicate<? super T> predicate, Collector<? super T, A, D> downstream)
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2,3,4, 5,6);
// 奇数偶数分组
Map<Boolean, List<Integer>> map = demo.stream()
	.collect(Collectors.partitioningBy(item -> item % 2 == 0));
System.out.println(map);
---------result----------
{false=[1, 3, 5], true=[2, 4, 6]}
```

## 聚合求平均值
```java
// 返回Double类型
public static <T> Collector<T, ?, Double> averagingDouble(ToDoubleFunction<? super T> mapper)
// 返回Long 类型
public static <T> Collector<T, ?, Double> averagingLong(ToLongFunction<? super T> mapper)
//返回Int 类型
public static <T> Collector<T, ?, Double> averagingInt(ToIntFunction<? super T> mapper)
```
- 示例
```java
List<Integer> demo = Arrays.asList(1, 2, 5);
Double data = demo.stream().collect(Collectors.averagingInt(Integer::intValue));
System.out.println(data);
---------result----------
2.6666666666666665
```

## 流聚合查找最大最小值
```java
//最小值
public static <T> Collector<T, ?, Optional<T>> minBy(Comparator<? super T> comparator) 
//最大值
public static <T> Collector<T, ?, Optional<T>> maxBy(Comparator<? super T> comparator)    
```
-	示例
```java
List<Integer> demo = Arrays.asList(1, 2, 5);
Optional<Integer> min = demo.stream().collect(Collectors.minBy(Comparator.comparing(item -> item)));
Optional<Integer> max = demo.stream().collect(Collectors.maxBy(Comparator.comparing(item -> item)));
System.out.println(min.get()+"-"+max.get());
---------result----------
1-5
```

## 聚合计算统计结果
-	可以获得元素总个数，元素累计总和，最小值，最大值，平均值
```java
//返回Int 类型
public static <T> Collector<T, ?, IntSummaryStatistics> summarizingInt(
		ToIntFunction<? super T> mapper)
//返回Double 类型
public static <T> Collector<T, ?, DoubleSummaryStatistics> summarizingDouble(
		ToDoubleFunction<? super T> mapper)
//返回Long 类型
public static <T> Collector<T, ?, LongSummaryStatistics> summarizingLong(
		ToLongFunction<? super T> mapper)        
```
- 示例
```java
List<Integer> demo = Arrays.asList(1, 2, 5);
IntSummaryStatistics data = demo.stream().collect(Collectors.summarizingInt(Integer::intValue));
System.out.println(data);
---------result----------
IntSummaryStatistics{count=3, sum=8, min=1, average=2.666667, max=5}
```

## JDK12提供的新聚合方法
```java
//流分别经过downstream1、downstream2聚合处理，再合并两聚合结果
public static <T, R1, R2, R> Collector<T, ?, R> teeing(
		Collector<? super T, ?, R1> downstream1,
		Collector<? super T, ?, R2> downstream2,
		BiFunction<? super R1, ? super R2, R> merger) 
```

# 5 并发paralleStream的使用
- 配合CompletableFuture和线程池的使用
- 示例
```java
public static void main(String[] args)  throws Exception{
    List<Integer> demo = Stream.iterate(0, item -> item + 1)
            .limit(5)
            .collect(Collectors.toList());
    //示例1
    Stopwatch stopwatch = Stopwatch.createStarted(Ticker.systemTicker());
    demo.stream().forEach(item -> {
        try {
            Thread.sleep(500);
            System.out.println("示例1-"+Thread.currentThread().getName());
        } catch (Exception e) { }
    });
    System.out.println("示例1-"+stopwatch.stop().elapsed(TimeUnit.MILLISECONDS));

    //示例2, 注意需要ForkJoinPool，parallelStream才会使用executor指定的线程，否则还是用默认的 ForkJoinPool.commonPool()
    ExecutorService executor = new ForkJoinPool(10);
    stopwatch.reset(); stopwatch.start();
    CompletableFuture.runAsync(() -> demo.parallelStream().forEach(item -> {
        try {
            Thread.sleep(1000);
            System.out.println("示例2-" + Thread.currentThread().getName());
        } catch (Exception e) { }
    }), executor).join();
    System.out.println("示例2-"+stopwatch.stop().elapsed(TimeUnit.MILLISECONDS));
    //示例３
    stopwatch.reset(); stopwatch.start();
    demo.parallelStream().forEach(item -> {
        try {
            Thread.sleep(1000);
            System.out.println("示例3-"+Thread.currentThread().getName());
        } catch (Exception e) { }
    });
    System.out.println("示例3-"+stopwatch.stop().elapsed(TimeUnit.MILLISECONDS));
    executor.shutdown();

}
```
-	-------------------result--------------------------
```java
示例1-main
示例1-main
示例1-main
示例1-main
示例1-main
示例1-2501
示例2-ForkJoinPool-1-worker-19
示例2-ForkJoinPool-1-worker-9
示例2-ForkJoinPool-1-worker-5
示例2-ForkJoinPool-1-worker-27
示例2-ForkJoinPool-1-worker-23
示例2-1004
示例3-main
示例3-ForkJoinPool.commonPool-worker-5
示例3-ForkJoinPool.commonPool-worker-7
示例3-ForkJoinPool.commonPool-worker-9
示例3-ForkJoinPool.commonPool-worker-3
示例3-1001
```
-	parallelStream的方法确实会使用多线程去运行，并且可以指定线程池，不过自定义线程必须是ForkJoinPool类型，否则会默认使ForkJoinPool.commonPool()的线程