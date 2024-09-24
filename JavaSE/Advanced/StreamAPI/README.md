 1Stream API的概述 

 1.1什么是StreamAPI呢？ 

从JDK1.8开始，Java语言引入了一个全新的流式Stream API，StreamAPI把真正的函数式编程风格运用到Java语言中，使用StreamAPI可以帮我们更方便地操作集合，允许开发人员在不改变原始数据源的情况下对集合进行操作，这使得代码更加简洁、易读和可维护。

使用Stream API对集合数据进行操作，就类似于使用SQL执行的数据库查询，也可以使用Stream API来并行执行的操作。简而言之，Stream API提供了一种高效且易于使用的处理数据的方式。

 1.2Stream和Collection的区别 

Collection：是静态的内存数据结构，强调的是数据。

Stream API：是跟集合相关的计算操作，强调的是计算。

总结：Collection面向的是内存，存储在内存中；StreamAPI面向的是CPU，通过CPU来计算。

 1.3Stream API的操作步骤 

1第一步：创建Stream

a通过数据源（如：集合、数组等）来获取一个Stream对象 。

2第二步：中间操作

a对数据源的数据进行处理，该操作会返回一个Stream对象，因此可以进行链式操作。

3第三步：终止操作

a执行终止操作时，则才会真正执行中间操作，并且并返回一个计算完毕后的结果。

 1.4Stream API的重要特点 

1Stream自己不会存储元素，只能对元素进行计算。

2Stream不会改变数据对象，反而可能会返回一个持有结果的新Stream。

3Stream上的操作属于延迟执行，只有等到用户真正需要结果的时候才会执行。

4Stream一旦执行了终止操作，则就不能再调用其它中间操作或终止操作了。







 2创建 Stream的方式 

 2.1通过Collection接口提供的方法 

通过Collection接口提供的stream()方法来创建Stream流。



 2.2通过Arrays类提供的方法 

通过Arrays类提供的stream()静态方法来创建Stream流。



注意：Stream、IntStream、LongStream和DoubleStream都继承于BaseStream接口。







 2.3使用Stream接口提供的方法 

通过Stream接口提供的of(T... values)静态方法来创建Stream流。



 2.4顺序流和并行流的理解 

在前面获得Stream对象的方式，我们都称之为“顺序流”，顺序流对Stream元素的处理是单线程的，即一个一个元素进行处理，处理数据的效率较低。

如果Stream流中的数据处理没有顺序要求，并且还希望可以并行处理Stream的元素，那么就可以使用“并行流”来实现，从而提高处理数据的效率。

一个普通Stream转换为可以并行处理的Stream非常简单，只需要用调用Stream提供的parallel()方法进行转换即可，这样就可以并行的处理Stream的元素。那么，我们不需要编写任何多线程代码就可以享受到并行处理带来的执行效率的提升。

【示例】把顺序流转化为并行流



在Collection接口中，还专门提供了一个parallelStream()方法，用于获得一个并行流。

【示例】使用parallelStream()方法获得一个并行流









 3Stream API的中间操作 

中间操作属于惰式执行，直到执行终止操作才会真正的进行数据的计算，此处调用中间操作只会返回一个标记了该操作的新Stream对象，因此可以进行链式操作。

在后续的操作中，我们调用StudentData类的getStudentList()静态方法，则就能获得一个存储Student对象的List集合，其代码实现如下：









 3.1筛选（filter） 

筛选（filter），按照一定的规则校验流中的元素，将符合条件的元素提取到新的流中的操作。该操作使用了Stream接口提供的“Stream<T> filter(Predicate<? super T> predicate);”方法来实现。

【示例】使用筛选的案例



 3.2映射（map） 

映射（map），将一个流的元素按照一定的映射规则映射到另一个流中。该操作使用了Stream接口提供的“<R> Stream<R> map(Function<? super T, ? extends R> mapper);”方法来实现。

【示例】使用映射的案例



在Stream接口中，可以实现“将多个集合中的元素映射到同一个流中”，该操作使用了Stream接口提供的“<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);”方法来实现。

【示例】将多个集合中的元素映射到同一个流中









 3.3除重（distinct） 

除重（distinct），也就是除去重复的元素，底层使用了hashCode()和equals(Object obj)方法来判断元素是否相等。该操作使用了Stream接口提供的“Stream<T> distinct();”方法来实现。

【示例】演示除重的操作



 3.4排序（sorted） 

排序（sorted），也就是对元素执行“升序”或“降序”的排列操作。在Stream接口中提供了“Stream<T> sorted();”方法，专门用于对元素执行“自然排序”，使用该方法则元素对应的类就必须实现Comparable接口。

【示例】使用自然排序的案例





在Stream接口中还提供了“Stream<T> sorted(Comparator<? super T> comparator);”方法，专门用于对元素执行“指定排序”，这样就能对某一个类设置多种排序规则。

【示例】使用指定排序的案例









 3.5合并（concat） 

合并（concat），也就是将两个Stream合并为一个Stream，此处使用Stream接口提供的“public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)”静态方法来实现。

【示例】将两个Stream合并为一个Stream。



 3.6截断和跳过 

跳过（skip），指的就是跳过n个元素开始操作，此处使用Stream接口提供的“Stream<T> skip(long n);”方法来实现。

截断（limit），指的是截取n个元素的操作，此处使用Stream接口提供的“Stream<T> limit(long maxSize);”方法来实现。

【示例】从指定位置开始截取n个元素









 4Stream API的终止操作 

触发终止操作时才会真正执行中间操作，终止操作执行完毕会返回计算的结果，并且终止操作执行完毕那么操作的Stream就失效，也就是不能再执行中间操作或终止操作啦。

 4.1遍历（forEach） 

遍历（forEach），使用Stream接口提供的“void forEach(Consumer<? super T> action);”方法来遍历计算的结果。

【示例】遍历操作的案例



 4.2匹配（match） 

匹配（match），就是判断Stream中是否存在某些元素，Stream接口提供的匹配方法如下：

1boolean allMatch(Predicate<? super T> predicate);  检查是否匹配所有的元素

2boolean anyMatch(Predicate<? super T> predicate);  检查是否至少匹配一个元素

3boolean noneMatch(Predicate<? super T> predicate); 检查是否一个元素都不匹配

4Optional<T> findFirst(); 获得第一个元素

注意：此处的Optional是一个值的容器，可以通过get()方法获得容器的值。

【示例】匹配操作的案例









 4.3归约（reduce） 

归约（reduce），可以将Stream中元素反复结合起来，从而得到一个值。在Stream接口中，常用的归约方法如下：

1Optional<T> reduce(BinaryOperator<T> accumulator);

2T reduce(T identity, BinaryOperator<T> accumulator);

【示例】归约操作的案例



reduce操作可以实现从一组元素中生成一个值，而max()、min()、count()等方法都属于reduce操作，将它们单独设为方法只是因为常用，在Stream接口中这些方法如下：

long count(); 获得元素的个数

Optional<T> max(Comparator<? super T> comparator); 获得最大的元素

Optional<T> min(Comparator<? super T> comparator); 获得最小的元素

【示例】获得最大、最小和元素的个数









 4.4收集（collect） 

收集（collect），可以说是内容最繁多、功能最丰富的部分了。从字面上去理解，就是把一个流收集起来，最终可以是收集成一个值也可以收集成一个新的集合。

调用Stream接口提供的“<R, A> R collect(Collector<? super T, A, R> collector);”方法来实现收集操作，并且参数中的Collector对象大都是直接通过Collectors工具类获得，实际上传入的Collector决定了collect()的行为。

 4.4.1归集（toList/toSet/toMap） 

因为Stream流不存储数据，那么在Stream流中的数据完成处理后，如果需要把Stream流的数据存入到集合中，那么就需要使用归集的操作。在Collectors提供的toList、toSet和toMap比较常用，另外还有Collectors提供的toCollection等比较复杂一些的用法。

【示例】演示toList、toSet和toMap的实现



在以上的代码中，我们将Stream流中计算的数据转化为List和Set集合时，此时并没有明确存储数据对应集合的具体类型，想要明确存储数据对应集合的具体类型，则就需要使用toCollection来实现。

【示例】演示toCollection的实现



【示例】获得年龄大于20岁的女同学，然后返回按照年龄进行升序排序后的List集合





在归集的知识点中，我们实现了将Stream中计算的数据转化为集合或Map，那么能否将Stream中计算的数据转化为数组呢？答案是可以的，我们可以使用Stream提供的toArray静态方法来实现。

【示例】将Stream中计算的数据转化为数组









 4.4.2统计（count/averaging） 

Collectors提供了一系列用于数据统计的静态方法：

1计数：counting

2平均值：averagingInt、averagingLong、averagingDouble

3最值：maxBy、minBy

4求和：summingInt、summingLong、summingDouble

5统计以上所有：summarizingInt、summarizingLong、summarizingDouble

【示例】对学生的年龄进行统计









 4.4.3分组（groupingBy） 

分组（groupingBy），将Stream按条件分为两个Map，比如按照学生年龄分为两个Map集合。

【示例】按照学生年龄分为两个Map集合



 4.4.4接合（joining） 

接合（joining），把Stream计算的数据按照一定的规则进行拼接。

【示例】获得所有学生的名字拼接成一个字符