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

## 集合