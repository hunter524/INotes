# Guava(番石榴) 使用手册

## 基础工具

### Avoid Null

null 值是多义的（既可以表示成功，也可以表示失败，或者表示某一种特殊的状态），且容易导致崩溃（NPE)。在Map 中返回null值便是有歧义的，其既可以表示指定的 key 值为 null，或者该 key 不存在于 Map 中。如果要在 Map 中存储 null，推荐的做法是维护一个值为 null 的 key 的集合。而不是将其存储在Map 中。使用 Optional 或者 @Nullable，@Nonull 注解，则会强制使调用者进行思考该参数是否可以是null,或者返回值是否可以是 null.
对应的 com.google.common.base.Strings 也提供了对 Null 和 “” 相互转换的操作，当然也可以Optional 实现这个操作。null->"":Optional.of(str).or("")

#### guava 提供的类

com.google.common.base.Optional包下。jdk 1.8 开始引入了 java.util.Optional.
kotlin 中可以使用 any?:"default"，any?.method() 实现 null 安全避免 NPE.

- Optional\<T>
  
  - of:创建一个 Present,表示持有某值。如果传入的值为 Null 则直接抛出 NPE.采用的 Fail First 策略。

  - fromNullable:创建一个 Absent or Present。如果传入 Null 则创建 Absent，传入 nonull 则创建 Present

  - absent:直接创建一个 Absent,代表该值缺省。

  - 实例方法，get() 为 null 则抛出 IllegalStateException.or() 为空则返回用户传入的默认值，用户传入的默认值不能为空。isPresent() null return false,else return true. asSet() 返回一个 immutable Set.如果 Optional 为 null 则该 Set 是空的。

- Absent
  
  null 值的替代，表示缺省该值。

- Present

  表示 nonull ，持有 nonull 真实对象的引用。

### Precondition(前置条件)

  对应 com.google.common.base.Preconditions 方法，使用该类的方法是推荐的做法是静态导入该类的中的检查方法。避免频繁使用 Preconditions.checkXXX,造成编码时 Preconditons 的冗余。

  Tips:多个检查条件时，建议把不同的检查条件放置在不同的行，当抛出异常时则可以根据日志快速的定位出错的位置（条件)。

#### 提供的检查类型

下述对应的重载方法，均提供提供 message 的方法 和 通过 String.formate 进行格式化参数的方法。

- checkArgument
  
  false 则抛出 IllegalArgumentException.语义是用于检查参数是否合法。

- checkState

  false 则抛出 IllegalStateException.语义是用于检查状态是否合法。

- checkNotNull

  null 则抛出 NPE.语义是检查引用是否为空。对应的 JDK 1.7 中引入了

- checkElementIndex/checkPositionIndex/checkPositionIndexes(检查字符串的范围)

  检查 index 或者 position 范围是否越界，是否为负值，start 是否小于 end，size < 0 等。参数不合法则抛出 IllegalArgumentException,越界则抛出 IndexOutOfBoundsException。

### Objects 方法

  com.google.common.base.Objects 工具类。从 jdk 1.7 开始 jdk 官方添加了 java.util.Objects 对 Object 的方法进行了一些扩展。
  其中 hashCode,toString,compare/compareTo 相关方法，据guava的作者说是为了解决使用者写这些方法时的痛苦，但是目前借助 IDea 生成相关方法也很方便，并不痛苦。但是目前实现 Comparable 接口重写 compareTo 方法依旧比较痛苦。

  com.google.common.collect.ComparisonChain 目前 jdk 并没有提供相关替代方法。

- equals
  
  现有的判断两个对象是否相等，需要先判断某个对象是否为null,如果是null则需要判断另外一个对象是否为null，如果另外一个对象为null则相等，不为null则不相等。如果第一个对象非 null 则使用，第一个对象 equals 第二个对象进行判断。

  使用 Objects#equals 则避免了上述冗余复杂的逻辑。

- hashCode

  生成 hashCode 的方法可以在不同的地方随意进行定制（定制哪些地方使用哪些字段生成hashCode)，否则还是使用 Intellij IDea 的快捷生成hashCode 的方式在对象内部重写 hashCode 生成 hashCode 更加方便，重写 equals 方法判断是否相等。

  使用 Intellij IDea 时也可以指定使用 guava的 Objects 或者使用  jdk 1.7 的 Objects 方法进行生成 hashCode 和 equals 方法。

- toString

  com.google.common.base.MoreObjects 同上述方法类似。主要用于在 Bean 内部协助覆盖 toString 方法，生成当前 bean 的String 表示。

- compare/compareTo

  使用 ComparisonChain 工具类协助覆写 Comparable#compareTo ,则更加流畅，不容易犯错，避免冗余的比较。因此该处的 ComparisonChain 实现方式，也是比较经典的短路模式，值得借鉴。

  before:

    ```java
      class Person implements Comparable\<Person> {
      private String lastName;
      private String firstName;
      private int zipCode;

      public int compareTo(Person other) {
        int cmp = lastName.compareTo(other.lastName);
        if (cmp != 0) {
          return cmp;
        }
        cmp = firstName.compareTo(other.firstName);
        if (cmp != 0) {
          return cmp;
        }
        return Integer.compare(zipCode, other.zipCode);
      }
    }
    ```

  after:

    ```java
      public int compareTo(Foo that) {
       return ComparisonChain.start()
           .compare(this.aString, that.aString)
           .compare(this.anInt, that.anInt)
           .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
           .result();
     }
   ```

### Ordering

 com.google.common.collect.Ordering 将 java.util.Collections 库中的 sort,max,min,reverse 等方法进行了扩展，重写改变了需要传入 Iterator,Comparator 的调用模式（该模式需要容器内部的对象实现 Comparator 接口) 为：先使用 Ordering 构建排序规则，再额外扩展了 min,max,greatestOf(最大 n 项)，leastOf (最小 n 项)，reverse,compound(组合多个条件进行排序)，isOrder（判断特定集合是否已经排序，相等也认为是有序)，isStrictlyOrdered（有相等的元素则不认为是有序的）,sortedCopy(返回复制过的，排序列表)，binarySearch（二分查找已经排序的 List 并且返回该元素的下标）

 上面的 ComparisonChain 简化了继承自 Comparable 接口对 compareTo 的实现。Ordering 则是简化了 Comparator 比较器的写法。（Ordering 抽象类本身就是一个比较器）

- Ordering#natural

返回自然序列排序的排序规则 Ordering.

- Ordering#allEqual

返回所有元素顺序均相等的 Ordering.

- Ordering#explicit

根据指定元素的等级，对列表中的元素再进行排序。

- Ordering#arbitrary

对列表进行随机排序。

- Ordering#usingToString

使用对象的 toString 的值进行比较。

- nullsFirst/nullsLast

对已经定义 Ordering 规则中null位置进行定义。

### 异常处理工具

首先明确两个概念:

  第一：抛出异常，将方法栈进行出栈，将方法的执行逻辑/控制逻辑交回上层的调用方法并不会影响虚拟机的执行效率。与正常方法调用的返回执行逻辑并没有太大的差别。

  第二：抛出异常耗时主要是因为构建异常时使用了 Throwable#fillInStackTrace。填充和构建当前方法的调用栈便于问题的排查。jdk 也提供了一种类型为 java.util.EmptyStackException 不填充调用栈的 RuntimeException。

目前注意到的使用异常进行跨多个方法调用层进行业务处理的实例：

Netty:AbstractNioChannel#doFinishConnect 在建立链接时，通过抛出不同的异常，标记是因为什么原因导致无法链接的。
ByteToMessageDecoder#channelRead 抛出 DecoderException 标记某个失败的原因是解码器解码失败导致的。

Gradle: 执行 Task 中的 Action 时分别抛出 StopActionException 用于标记终止当前 Action 的执行，执行该 Task 的其他 Action. StopExecutionException 用于标记终止Task 的执行，即使该 Task 中还有很多其他 Action 没有执行。

因此使用抛出异常进行业务逻辑判断处理其实是为了使跨方法调用栈的状态传递和表达更为简单，但是java的异常机制也是java被BB不好的最大的一个特性。（其实应该是开发者不够合格，滥用了java的异常，导致满屏 try catch 代码可读性变差，其实好的语言不应该是即使开发者不够合格，但是也能写出还行的代码吗？因此java的异常机制就一直被BB了)

com.google.common.base.Throwables

解决的核心问题是声明捕获异常时通常是为了捕获受查异常，而不是 Error,RuntimeException,但是一旦声明的 catch 为 Throwable 则 Error,RuntimeException 也会被捕获，这个时候 Throwables 的出现则使这种场景的处理更加简单了。

同时如果 catch 住的异常类型为 Throwable ，再次 throw 抛出时，会要求方法声明抛出受查异常。

其中 Throwables#propagate 方法存在争议，已经被声明废弃。（其会将受查异常包装成非受查异常，如果包装也应该包装成更具体的异常类型如:IllegalStateException,IllegalArgumentException 等 而不是统一包装成为：RuntimeException）

- Throwables#throwIfUnchecked

  非受查异常，直接抛出。

- Throwables#throwIfInstanceOf

  指定类型的异常直接抛出。

- Throwables#propagateIfPossible
  
  抛出非受查异常和指定类型异常。

- Throwables#getCauseAs（22.0 加入）

  将该异常原因转换为指定类型的异常，转换失败则抛出 ClassCastException。cause 的使用其实是表达的语义是处理异常时又出现了异常，需要将内部的异常原因包含进入其中。

- Throwables#getRootCause/getCausalChain/getStackTraceAsString

  获取最初的异常/获取抛出异常的异常链/将异常堆栈转换成为 String 字符串。

- Throwables#lazyStackTrace/lazyStackTraceIsLazy

  支持lazy 的平台，每次获取一个层级的 StackTraceElement 调用一次 Throwable#getStackTraceElement。不支持 lazy 的平台则一次性使用 Throwable#getStackTrace 获得该异常的所有方法调用栈。

## 集合

位于 com.google.common.collect 包下。

### 不可变集合

不可变集合/不可变对象的提供其实一种防御性编程[defensive programming]的方式，尤其是在面向对象的编程语言中，这种防御性编程的模式可以缩小出现异常时的代码排查范围。

使用不可变集合要求方法返回参数需要显示的声明为 ImmutableXXX 明确告诉调用者返回的集合是不可以改变的。入参则需要使用 List/Iterator 等比较宽泛的集合接口以使调用者减少无聊代码的编写。在方法内部可以将入参再包装成为不可变集合再在内部使用。

同时要求放入不可变集合中的元素需要是不可变的，如果是可变的影响到了 Object#equals 结果，则会触发意想不到的bug.(如:Set 中添加时不存在两个相同的元素，当修改元素属性，导致了存在两个相等的元素)该问题等同于深拷贝和浅拷贝的问题。当使用不可变集合时则要求对象需要是深不可变的。

kotlin 的语法中使用了 data class 为数据类提供了 copy 语法，实现了数据的保护性拷贝和防御性编程。

guava 只提供了不可变集合的操作。需要使用不可变对象则需要使用 [immutables 基于 APT 生成不可变对象 ][https://github.com/immutables/immutables] 或者 cglib 的 net.sf.cglib.beans.ImmutableBean 基于 asm 的字节码生成技术动态生成不可变对象的类的字节码，并加载进入虚拟机。

为什么要用不可变集合？

- 当对象被不可信的库调用时，不可变形式是安全的

- 不可变对象被多个线程调用时，不存在竞态条件问题

- 不可变集合不需要考虑变化，因此可以节省时间和空间。所有不可变的集合都比它们的可变形式有更好的内存利用率（分析和测试细节）
- 不可变对象因为有固定不变，可以作为常量来安全使用。

为什么要用 guava 的不可变集合？而不是用 java.util.Collections#unmodifiableXXX 提供的不可变集合？

- 笨重而且累赘：不能舒适地用在所有想做防御性拷贝的场景（缺少 of,Builder 等各种快速通过单个元素构建不可变集合的方法，构建不可变集合需要先构建可变集合)

- 不安全：要保证没人通过原集合的引用进行修改，返回的集合才是事实上不可变的；(guava 不依赖底层集合，真正的实现了不可变)

- 低效：包装过的集合（其实该处只是使用代理模式，进行了一层访问控制）仍然保有可变集合的开销，比如并发修改的检查、散列表的额外空间，等等。（真正的不可变集合实现的功能更少，考虑的使用场景更少，因此需要更加的高效和低内存开销:如 Map 中扩容多余的没有元素的空间，被包装集合的并发修改检查等)
*guava 的不可变集合不支持 null 值,为了更方便的进行 Set List 进行可变不可变，线程安全集合的切换，同时也是符合面向接口编程良好编程实践，所以方法的入参返回使用接口定义类型，而不是使用实现类型*

JDK 中提供的集合和 Guava 中提供的集合对应 guava 提供的其 Immutable 的集合的类型。

Interface|JDK or Guava|Immutable Version
:-------:|:------------:|:----------------:
Collection|JDK|ImmutableCollection
List|JDK|ImmutableList
Set|JDK|ImmutableSet
SortedSet/NavigableSet|JDK|ImmutableSortedSet
Map|JDK|ImmutableMap
SortedMap|JDK|ImmutableSortedMap
Multiset|Guava|ImmutableMultiset
SortedMultiset|Guava|ImmutableSortedMultiset
Multimap|Guava|ImmutableMultimap
ListMultimap|Guava|ImmutableListMultimap
SetMultimap|Guava|ImmutableSetMultimap
BiMap|Guava|ImmutableBiMap
ClassToInstanceMap|Guava|ImmutableClassToInstanceMap
Table|Guava|ImmutableTable

构建上述不可变集合的方法：

- ImmutableXXXX#of
  
  传入可变参数直接构建不可变集合。(对于不同数量的元素构建的集合类型进行了优化，如：empty 只需要统一返回同一个空指示对象，而不是每次构建一个空集合都需要新构建一个空集合对象，对于只有一个集合的元素，则可以通过很简单的方式实现该类型的集合。对于元素大于2 的集合则可以通过正常的方式进行构建)

- ImmutableXXXX#copyOf

  从集合，迭代器拷贝对象进入不可变集合。

- ImmutableXXXX#builder()/ImmutableSet#Builder

  通过直接创建构造器，通过方法调用创建构造器然后构造指定的不可变集合。

- ImmutableXXXX#orderBy/ImmutableXXXX#reverseOrder/ImmutableXXXX#naturalOrder

  构建可排序类的元素。

- 不可变集合在实现过程中对于性能和内存所做的优化

ImmutableList:

主要通过创建试图和进行实例类型检测避免不必要的数组容器对象的创建和重复的无意义的不可变对象的创建。

List 的实现对该处进行了性能优化，对同一个 ImmutableList 多次调用 copyOf 则返回的始终是最早的 ImmutableList 而不是每调用一次创建一次对象。（从而避免了创建ImmutableList内存消耗)。同时 ImmutableList#subList 为了避免创建过度的数组容纳子List,subList 方法只是创建了一个视图引用了原始的 ImmutableList 局部元素。 ImmutableList#reverse 也是同等的创建了试图，而不是将元素逆序之后重新创建了一个容器数组。

ImmutableSet:

Builder 构建该Set时先通过 RegularSetBuilderImpl 进行构建，当检测到 Hash洪泛（hash 冲突使 Hash 表退化成链表，增加查找时间)且超过指定阀值时为了避免更快速的退化成为链表，此时切换实现为 JdkBackedSetBuilderImpl 使用 jdk 提供的 HashMap 解决hash冲突问题（HashMap 中 hash 冲突超过指定阀值时会退化成为红黑树，在红黑树内部再通过添加的对象内存地址，对象实例类等信息进行排序构建红黑树）

TODO:// 集合中的 Spliterators，Collector，Supplier 的使用。

### guava 扩展的集合

对于JDK 中提供的集合从并发安全上主要分为线程安全的集合和线程不安全的集合，通常线程安全的集合为了减少线程阻塞，通常会设计一些巧妙但是十分复杂的算法避免线程不安全（COW,局部锁，CAS 清理算法)

从功能上分为：

List:

表示一个链表存储元素的集合（可以有重复元素）

  LinkedList:
  
  随机访问效率低（通过下标索引），但是空间利用率高。迭代访问效率等同于 ArrayList.在数据删除插入上各具优势。双向链表实  现。
  
  ArrayList:
  
  随机访问效率高，但是内存占用较大（移除元素时只删除元素，紧缩数组，并不会缩减数组大小)。*jdk 中继承自 RandomAccess 接口的通常随机访问效率较高*

Set:

同 List 基本相同不能有重复元素。

  SortedSet:

  排序的Set

Map:

  SortedMap/TreeMap:

  以key排序元素的Map

  IdentityHashMap:

  使用对象的内存地址做 hashCode,而不是使用对象覆写的 hashCode 方法作为 hashCode 插入。（System#identityHashCode，Object#hashCode )

  LinkedHashMap:

  在 HashMap 的基础上添加了链表实现。

Queue:

[不同并发特性的集合 JCTools][https://github.com/JCTools/JCTools]
[apache 实现的集合类][https://commons.apache.org/proper/commons-collections/]

#### MultiSet
