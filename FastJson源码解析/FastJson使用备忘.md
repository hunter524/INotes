# FastJson 使用备忘

## 数据 Bean 分层封装

- VO（View Object）：视图对象，用于展示层，它的作用是把某个指定页面（或组件）的所有数据封装起来。

- DTO（Data Transfer Object）：数据传输对象，这个概念来源于J2EE的设计模式，原来的目的是为了EJB的分布式应用提供粗粒度的数据实体，以减少分布式调用的次数，从而提高分布式调用的性能和降低网络负载，但在这里，我泛指用于展示层与服务层之间的数据传输对象。

- DO（Domain Object）：领域对象，就是从现实世界中抽象出来的有形或无形的业务实体。

- PO（Persistent Object）：持久化对象，它跟持久层（通常是关系型数据库）的数据结构形成一一对应的映射关系，如果持久层是关系型数据库，那么，数据表中的每个字段（或若干个）就对应PO的一个（或若干个）属性。

由业务层(View) 触发的数据,由上向下进行到数据库持久层进行更新.

- VO(value object) 值对象,通常用于业务层之间的数据传递，和 PO 一样也是仅仅包含数据而已。但应是抽象出的业务对象 , 可以和表对应 , 也可以不 , 这根据业务的需要 。用 new 关键字创建，由 GC 回收的.

-TO(Transfer Object) ，数据传输对象,在应用程序不同 tie( 关系 ) 之间传输的对象.

- BO(business object) 业务对象,从业务模型的角度看 , 见 UML 元件领域模型中的领域对象。封装业务逻辑的 java 对象 , 通过调用 DAO 方法 , 结合 PO,VO 进行业务操作。 business object: 业务对象 主要作用是把业务逻辑封装为一个对象。这个对象可以包括一个或多个其它的对象。 比如一个简历，有教育经历、工作经历、社会关系等等。 我们可以把教育经历对应一个 PO ，工作经历对应一个 PO ，社会关系对应一个 PO 。 建立一个对应简历的 BO 对象处理简历，每个 BO 包含这些 PO 。 这样处理业务逻辑时，我们就可以针对 BO 去处理。
*BO 是一种聚合的概念*

- POJO(plain ordinary java object) 简单无规则 java 对象,纯的传统意义的 java 对象。就是说在一些 Object/Relation Mapping 工具中，能够做到维护数据库表记录的 persisent object 完全是一个符合 Java Bean 规范的纯 Java 对象，没有增加别的属性和方法。我的理解就是最基本的 Java Bean ，只有属性字段及 setter 和 getter 方法！。

- DAO(data access object) 数据访问对象,是一个 sun 的一个标准 j2ee 设计模式， 这个模式中有个接口就是 DAO ，它负持久层的操作。为业务层提供接口。此对象用于访问数据库。通常和 PO 结合使用， DAO 中包含了各种数据库的操作方法。通过它的方法 , 结合 PO 对数据库进行相关的操作。夹在业务逻辑与数据库资源中间。配合 VO, 提供数据库的 CRUD 操作.

## JsonPath

门面类,负责对于 JSONString,JSONObject 按照 JSONPath 的查询语法进行 JSONString/JSONObject 中指定字段的解析和抽提.

避免了当要处理多个未知对象时需要使用者手动遍历对象属性,提取值的操作.为遍历不同对象的指定的 key 的值提供了一种便捷的操作.

JSONPath 是一种匹配模式的抽象,一个 JSONPath 可以用于多个 JSONString/JSONObject 的抽象.*线程安全性,由JSONPath 使用一个线程安全的 MAP 存储 JSONPath 对象以供相同的 path 进行缓存使用,可以推测 JSONPath 为 线程安全*

JSONPath 的匹配查询结果的表示方式均为 List(集合方式),因此不能像 JSON#toJSONString 一样支持单个如 int,String 对象的选择匹配操作.JSONPath 支持 Bean Object,Map,Collection 的查询操作,不支持 int,String 等单数据对象的查询操作.

JsonPath 是一种类似于 XPath 的 Json 查询语句表达式的规范.[其较早的java 实现库][https://github.com/json-path/JsonPath], 目前 FastJson 通过内置的 JSONPath 默认支持该特性.

其语法表达式[参见][https://github.com/json-path/JsonPath/blob/master/README.md]

### JSONPath Api

- new JsonPath
  
  无缓存的 JSONPath 构建.

- compile

  编译指定的 path 路径成为 JSONPath,并且提供缓存策略,避免对于相同的 path 重复构建 JSONPath.*缓存最大为 1024个 path大于该值则不再缓存 path 对应的 JSONPath*

- eval/read
  
  静态方法传入 path 和 Object/JSONString 直接提取指定的路径的值.即使传递 JSONString 底层的调用逻辑也会将 String 解析成为 JSONObject 进行提取操作.

  eval 存在对应的实例方法,避免了 path 的重复传入.

- extract

  静态方法传入 path 和 JSONString,直接在 JSONString 语法层面进行 path 指定路径的提取,因此更加高效.*但是并不是所有的匹配类型均支持 extract 进行匹配,如:有引用类型,type(),floor() 等类型的匹配 Segment 则不支持 extract 类型匹配,其会退化成为 eval 类型匹配,其他类型如:[1:10:2] range Step 类型的匹配则直接不支持 extract,退化并且抛出异常.*

  存在对应的实例方法,避免了 path 的重复传入

- size
  
  静态方法传入 path 和 Object,获得指定 path 匹配到的属性的个数.

  存在对应的实例方法,避免了 path 的重复传入

- keySet

  静态方法传入 path 和 Object,获得指定 path 对应的 key 的值.

  存在对应的实例方法,避免了 path 的重复传入

- paths

  传入 Object 用于获取该 Object 中每个属性对应的路径.属性路径采用 / 作为分割符号.

- contains/containsValue
  
  判断指定的路径是否存在值
  判断指定的路径的值是否与指定的值是否相等,通过 Object#equals 进行判断.

  存在对应的实例方法,避免了 path 的重复传入

---下面是涉及到 JSONPatch 相关的方法 ---

- arrayAdd/patchAdd
  
  array 向指定集合,数组添加元素,或者替换指定属性的值.

  patchAdd 是 arrayAdd 同类的实例方法.

- remove
  
  删除指定 path 对应的值,如果是 JOBO 则将对应的字段置为空.

  存在对应的实例方法,避免了 path 的重复传入

- set
  
  设置/修改指定的路径对应的值.
  
  存在对应的实例方法,避免了 path 的重复传入

### JSONPath 语法

fastJson 的 JSONPath 语法与通用的标准 [JSONPath][https://github.com/json-path/JsonPath] 略有不同.其对于标准的 JSONPath 做了扩展和优化.该处的 path 规则扩展是由 JSONPath 内部的各种功能 Segment 实现的.

如:

```json
  [
    {
      "type"  : "iPhone",
      "number": "0123-4567-8888",
      "count":1
    },
    {
      "type"  : "home",
      "number": "0123-4567-8910",
      "count":2
    }
  ]
```

在 fastJson 的匹配规则中,作为表示根元素的 $ 符合可以前置省略.

匹配第0个元素:

标准写法: $.[0] fastJson: [0]

范围匹配:

标准写法: $.[0:1] fastJson: [0:0] *标准 JSONPaht end 是 exclusive 的,fastJson end 则是 inclusive ,start 均是 inclusive*

条件过滤:

标准写法: $[?(@.count > 1)] fastJson: [count > 1] *简化了对于 @表示当前字符的表示,同时也省略了 $ 对于根元素的表示*

fastJson 简化了对于数组对象,条件匹配 @ 表示待处理对象的语法.(*但是 fastJson 支持 @表示的匹配语法*)

JSONPath 匹配的基础:

- 一些特殊的语法表示的含义
  
  $..[  与 $.[
  
  在 fastJson 中表示 object only ,递归提取当前元素以及其内部的子元素进入顶层,只要该元素是 Array/Object 而不是基础数据类型,从而进一步可以对提取到的元素进行匹配如: $.[.city

  $.[\*]
  $..\*
  $..[\*]
  
  递归提取当前对象中的元素,进入顶层.与 $.[ 的区别为 $.[ 为粒度只到对象层级,而 $.[\*] 力度到字段层级. *$..\* 与 $.[\*] 与 $..[*] 三者等价具有相同的语义*

  $.[\*] 一个 [] 等价于代替了一个 . 属性的引用.

  $[\*]

  与 $.[\*] 的区别为不递归提取子元素的属性到顶层,只提取当前 $ 所表示的元素进入顶层.

- 属性名称匹配(跨层级查找,逐层级查找)
  
  $.name
  $..name
  $['name']
  $['address']['province']
  $['id','name']
  用于匹配同一个对象的多个属性值,属性值之间使用 , 进行分割.
  name
  *匹配根元素的属性值可以省略 $ 作为根元素的引用,直接使用属性名称进行匹配*

  特殊属性名称的场景,参见 JSONPath_4 如: JSONString 为 {"key":"value","10.0.1.1":"haha"} 需要匹配 10.0.1.1 作为属性名称的值则规则的匹配模式会写成 $.10\.0\.1\.1 其中的 . 需要使用 \ 进行转义.

  如果 $ 表示的是 Bean 则直接获取 Bean 中的属性名称. *如果 $ 表示的是 List 则为获取 List 每个 Bean 属性为指定名称的值,如果 $ 表示 List ,且 List 中的某项依旧为 List 则递归向下查找,直到 List 为空或者 List 中的元素不再为 List,Array 的处理方式与 List 相同* *在标准的 JSON 中则不支持该语法,需要先对 $ 根元素进行范围匹配,才能对其中的元素进行属性匹配如: $[0:].firstName*

- 数组范围匹配
  
  [1] => index
  [-1] => length - index
  [1,2,3] => indexs
  [1:2] => start end (start inclusive end inclusive)
  [0:8:2] => start end step
  [:4] => 0 to end
  [1:] => 1 to length
  *对于 path 直接是 0,+1,-1 等数字表示则直接获取对象以该数字为key的元素*

- 属性值条件匹配
  
  JSONPath 的传统规范写法: $[?(@.key > 123)] *推荐使用*
  fastJson 可以去掉 中括号,?,@ 和 小括号 写成 : $[key > 123]
  也可以只去掉中括号 写为 : $?(@.key > 123) *推荐使用*

  ?,[key > 123] 等的条件匹配是先会抽象成为一个 FilterSegment 的 Segment 再内嵌各种 Filter 形成的一个匹配规则.

  *? 条件匹配符号后面只能跟着 () 并且必须跟着 ()且不能存在空格字符, 不能使用 []*

  上述三种写法均可以使用 && || 链接表示 and 和 or 操作,Filter 通过 FilterGroup 进行嵌套处理,采取的策略是类似 reduceLeft 的规则进行匹配处理.

  *推荐使用 [?(key>123) &&(id > 100)] 将多个匹配条件进行分组组合条件进行匹配,不推荐使用 [key > 123] && [id > 100] 的方式进行匹配,!!!!没*

  *特别需要注意:$.name.?(@.key > 123) 这种写法是否不支持的, . 标记后面不能跟着 ? 条件匹配.*

  上述三种写法 fastJson 都予以很好的支持.

  多层嵌套的属性值条件匹配: $.address.[?(@.city = 'Nara')]  <==> $.address[?(@.city = 'Nara')] <==> $.address?(@.city = 'Nara') *可以用 . 作为层级区分再嵌套 [] 作为筛选条件的表达式,也可以直接使用 [] 包裹条件表达式,但是该种表达式在 fastjson 中无法被识别,同样的 $.address?(@.city = 'Nara') 表达式在标准的 JSONPath 中无法被识别*

  *!!!!在 fastJson 中不支持 $.address.[?(@.city = 'Nara')] 这种冗余的写法,其会抛出异常,因为在 JOSNParser#readSegment 983行的解析要求读取到 . 后面需要跟一个 合法的name 而 [ 不是一个合法的name!!!,因此不能采用这种写法*

  *只有 $.address[?(@.city = 'Nara')] 这个表达式才可以既在 fastJson 中被识别,也可以被标准的 JSONPath 识别*

  *$.address?(@.city = 'Nara') 的 FastJson 简写的写法无法在 标准的JSONpath 中被识别*

  like,not like,rlike , not rlike ,in,notin,between,not between 的常见写法:

- like/not like

  只支持 '%' 通配符.其可以位于开始,中间,末尾等任何位置.其可以通配 0-n 个字符.即其可以匹配字符,也可以不匹配字符.

  *like 只能对字符串进行匹配,无法匹配其他数据类型*
  
  [key like 'aa%']
  [key rlike 'regexpr']

- rlike/not rlike
  
  使用 regex 的匹配模式和规则对字符串进行规则匹配.

  *rlike,like,not rlike,not like 由于是字符串匹配模式,因此需要使用 ' " 将匹配模式字符串进行包裹*

- in/notin
  
  $.departs[name in ('wenshao','Yako')]
  $.departs[id not in (101,102)]

  in ,not in 支持 int(可以转换成为 int 的类型:byte,short,long)类型 和 String 类型的规则匹配.

  *in ,not in 表示的集合必须是同一种类型,不可以是不同的类型*

- between/not between
  
  *between , not between 目前只支持 byte,short,int,long*

  $.departs[id between 101 and 201]
  $.departs[id not between 101 and 201]

- 引用匹配
  
  使用引用匹配可以匹配当前对象的某个指定 path 所获取的值.*但是目前只支持 int,long,short,byte 以及 BigDecimal 的数字类型,同时支持的操作符只支持 ==,!=,>=,<=,>,< 几种类型*

  匹配条件支持 >,<,!=,=,>=,<=,in,not in(nin),like,rlike,between, =~ 在多个条件需要匹配时支持如下 && || 写法用于匹配两个条件 and 和 or . 如:$.* ? (@.type() == "array" && @.size() > 1) 还可以使用 () 用于表示每一个匹配条件的层级如 :$.\*?( (@.type() == "array") && (@.size() > 1))

  $[?(@.c =~ /a+/)] 正则匹配也可以写成该种形式, ~ 表示正则规则,加空格再加 /a+/ 表示匹配规则.参见 JSONPath_15#test_3

- 扩展属性展示
  
  size,length: 表示获取到匹配的结果的长度
  max,min:表示获取匹配的值中的最大值与最小值*只支持数字类型的集合*
  keySet: 匹配的结果需要是 Map 获取其值的集合,或者是 POJO 用于获取其 Bean 的属性名称的集合.*不支持集合类型*
  type: 表示获取匹配到的值的类型,返回的类型为字符串,*支持 js 所表述的类型,如:number,boolean,string,object 额外扩展了对于 null,array 的区分,在 js 中该两种类型均会识别为 object*
  floor: 向 -Infinity 取最大的整数(不能大于当前数字)*支持数字类型,或者数字类型的集合*

- 数组值添加,属性值更改

JSONPath 使用的场景情况较为复杂,因此测试用例较多,在 test 的 com.alibaba.json.bvt.path 包下.

JSONPath 常用写法:

- $..[*].foo.bar

等同于 $..foo.bar 表示匹配对象任意曾的 foo.bar 的引用.

## JSON Facade 类

### JSON Facade 类常用方法

JSON 支持在 JSON String 中添加类型信息.如 TreeSetTest 中的序列化一个类可以添加类型信息,同样反序列化的字符串也可以支持带有类型信息.(*可以认为是 FastJson 对于 JSON 规范的一种自定义扩展,用于指示具体的泛型类型信息,通过 @type 类型标记的值可以指定类型*)

```java
Assert.assertEquals("{\"@type\":\"com.alibaba.json.bvt.serializer.TreeSetTest$VO\",\"value\":TreeSet[]}", JSON.toJSONString(vo, SerializerFeature.WriteClassName));
```

- JSON#parse
  
  JSON String => 内置的 JSONObject,JSONArray,Set

- JSON#parseObject
  
  转换为指定类型的 Bean(转换成普通 Bean 需要携带 Bean 的类型参数).也可以通过 JSON#parseObject(String text) 不携带类型参数,则直接转换成为 JSONObject.(*其底层是依赖于 DefaultJSONParser#parse,DefaultJSONParser#parseObject,DefaultJSONParser#parseArray 实现的*) 传递类型信息则返回指定的类型的实例,不传递类型则返回原始的基础类型,Set,TreeSet,JSONObject,JSONArray 等基础类型.

- JSON#parseArray

  解析成为 JSONArray,List <T:>

- JSON#toJSONString/JSON#toJSONBytes

  序列化成为 String 或者 byte[]

- JSON#writeJSONString
  
  将序列化的结果写入 Writer,OutputStream

-JSON#toJSON

  将指定的 Object 转换成为 JSONObject,JSONArray 甚至是 JSONString.

### JSON 及其子类

- JSONArray
  
- JSONObject

## 注解使用

### @JSONCreator

  当 Bean 缺乏默认构造函数时,使用该注解标记 构造方法, Bean的静态方法 为构造该类的方法.*不能标记在类的普通方法上*
  *TODO://被 JSONField 标记了参数的构造方法,等同于 JSONCreator 注解标记的构造方法.*
  *在JavaBeanInfo 中优先查找 Class 上 JSONType#builder 属性创建的Builder类 -> 默认构造方法 -> 被该注解标记的构造方法 -> 然后查找被该注解标记的静态工厂方法 -> 最后查找未被注解标记的构造方法,作为构造该对象的方式(查找策略在 JavaBeanInfo#build 方法中定义)*

### @JSONField

该注解是用于添加在 方法(setter,getter),字段,构造方法,静态工厂方法的参数上(结合 JSONCreator 使用). 通常用于标记单个字段在序列化/反序列化时的特性定制.与 JSONType 注解相互呼应，JSONType 注解用于添加在类上，用于标记该类的序列化／反序列化特性定义．

- ordinal(Since 1.1.42)
  
  用于标记对象内部字段序列化/反序列化时的顺序 -> JSONFieldTest_0 (*如果没有该字段标记,fastJson 默认使用字段名称的字母顺序进行排序*)

- name
  
  序列化/反序列化时指定的该字段的名称,不使用该字段的属性名称(通常用于混淆类) -> AnnotationTest3

- format
  
  序列化/反序列化时指定时间的格式 -> DateFieldTest4

- serialize
  
  是否序列化

- deserialize

  是否反序列化

- serialzeFeatures(SerializerFeature[])
  
  对特定字段指定序列化时的 SerialzeFeature.
  TODO://JSON#toJsonString 时指定的 Feature 与 对某个特定的字段指定的 Feature 谁的优先级更高?
  按照推测应该是注解的优先级更高

- parseFeatures (Feature[])
  
  对特定字段提供反序列化时的 Feature.

- label
  
  为 LabelFilter 提供过滤依据.

- jsonDirect(Since 1.2.12)
  
  该字段不做 Bean -> JSON String 的解析,值直接写入结果中. -> JSONDirectTest

- serializeUsing(Since 1.2.16)
  
  为指定字段指定 ObjectSerializer 执行序列化. 按照惯例该处配置的优先级 SerializeConfig#put 设置的 ObjectSerializer -> SerializeUsingTest.(优先级实现代码 FieldSerializer 274 Line)

  该注解不可以注解在 Class 上,因此对于最外成的 Bean 依旧是从 SerializeConfig 中获取 ObjectSerializer.

- deserializeUsing(Since 1.2.16)
  
  为指定字段指定 ObjectDeserializer 执行反序列化. 同上按照惯例优先级高于 ParseConfig#设置的 ObjectDeserializer -> SerializeUsingTest

- alternateNames(Since 1.2.21,String[])
  
  反序列化时指定的对应字段,在 JSONString 中的可选的名称.

- unwrapped(Since 1.2.31)
  
  缩减对象层级,将当前字段中的元素,摊平到外成对象中. -> JSONFieldTest_unwrapped_0-6

- defaultValue(Since 1.2.61)
  
  序列化时当某个字段为 null 时则采用该注解标记的默认值.(*只对对象类型有效,对于基础数据类型无效,因为基础数据类型即使不赋值,也存在默认值*)

### @JSONType

JSONType 注解用于添加在类上，用于标记该类的序列化／反序列化特性定义．

- asm
  
  用于标记当前类型是否使用　ASM 优化，使用　ASM 优化则使用　ASMSerializerFactory／ASMDeserializerFactory 工厂类基于动态生成字节码分别创建　JavaBeanSerializer／ObjectDeserializer　避免对于　JDK 反射　API 的使用带来的低效率．*ASM 优化是否能使用不仅仅在于该标记为　true,还取决于该类的内在特性，如：带有泛型参数的类则不使用　ASM 优化．也取决于　JVM 平台是否支持动态定义类，如：Android 平台则不支持使用　ASM*

- orders(String[])
  
  fastJson 在序列化时默认的字段排序顺序为字母表顺序，使用该字段可以定义自定义序列化完成的字段的顺序．*定义的名称的排序*

- includes (1.2.6,String[])
  
  生效规则:*includes 对　getter 方法(get,is 开头的方法)生效，但是对于字段标记无效*
  *具体策略的实现参见 TypeUtils#buildBeanInfo -> TypeUtils#computeGetters 先查找 getter 方法,再查找对应的字段*
  *表示当前 Bean 序列化时被包含的 getter 方法值*

  标记普通　JAVA Bean 时不影响序列化的结果．Bean 中的字段正常被序列化．使用　JSON＃addMixInAnnotations　的混入特性时原始　Bean 则只序列化混入类注解上标记的 includes 指定的字段．
  
- ignores

  生效规则:*includes 对　getter 方法(get,is 开头的方法)生效，但是对于字段标记无效*

  优先级为 includes > ignores(如果配置了 includes 则只会匹配 includes 规则,不在规则内的均会被忽略.未配置 includes 但是配置了 ignores 则对 ignores 规则进行匹配,在规则内的则均被忽略,否则则不会被忽略)

  TODO:// 默认 private 的属性不会被序列化

- serialzeFeatures
  
  为 Bean 特定的配置的序列化 SerializerFeature.

- parseFeatures

  为 Bean 配置的特定的反序列化的 Feature

- alphabetic
  
  功能同　orders　为指示序列化为　JSONString 时字段在其中的顺序．orders 为指定顺序，不指定则为默认字母表顺序，该字段　false 表示用于取消默认的字母顺序，采用默认顺序(*即通常是字段的定义顺序*)

- mappingTo

  标记当前类型字段在反序列化时,反序列化为某个指定类型*所有字段标记为该类型的均会执行 mappingTo 的映射规则*(*通常是当前类的子类,其实现策略是基于 mappingTo 的类型重新查找 ObjectDeserializer 进行序列化任务的标记*).使用参见 AbstractSerializeTest2.
  *为了确保类型安全通常是将指定父类的字段重新映射为其子类,即使该类型被 @type 字段标记了类型路径*

- builder

  指定对象反序列化时构建对象的 Builder 构造器.配合注解 JSONPOJOBuilder 使用,用于指定该对象的 Builder 构造模式.
  *builder 指定的 Builder 类,默认使用 withXXX 设置指定属性(通过查看 JavaBeanInfo#build 634 line 源码得知,Builder 也默认支持 setXXX 用于属性设置的方法,set,With 后面指定的属性的值要求大写),使用 build 创建该 Builder 创建的对象,如果 build 不存在则也默认支持 create 方法(通过查看源码获得 create 方法的优先级低于 build 方法,只有当 build 方法不存在时才启用 create 方法)*

  *Buidler 类中的方法可以使用 @JSONField 标记该构造方法,用于指定优先级和名称*

  *指定 builder 则 Builder 的 withXXX,setXXX 方法扩展了对象的属性列表 JavaBeanInfo#fields*

- typeName (1.2.11)
  
  typeName 配合 typeKey 使用,用于在序列化时标记 JSONString 对应的类型.如果不指定 typeKey 则默认的 typeKey 值为 @type 定义在 JSON#DEFAULT_TYPE_KEY 字段. *@type 因为不是一个合法的字段名称标识符,因此无法被反序列化,如果 typeKey 是一个 Bean 定义的合法的属性值则可以正常被反序列化*
  *该属性被启用需要配置 SerializerFeature.WriteClassName 序列化标记*

- typeKey (1.2.32)

  用于指定序列化类型时 对 @type 进行重新命名.目前存在 [Bug][https://github.com/alibaba/fastjson/issues/3479] 父类存在想对应的 typeKey 字段且 typeKey 字段的名称即为 typeKey 会导致反序列化失败.直接返回 null.

- seeAlso (1.2.11)
  
  反序列化时 JSON#parseObject 指定类型为父类时,父类通过 seeAlso 用于指定可能存在的子类型,需要配合 typeName 注解值和 JSONString 中的 @type 标记进行类型推测和匹配.
  *实现则是依赖于 typeName 进行对应的 JavaBeanDeserializer 的反序列化的类型的匹配*

- serializer
  
  用于对指定类型标记注解使其使用特定的 ObjectSerializer 进行序列化操作

- deserializer (1.2.14)
  
  对应于 serializer 用于指定反序列化的 ObjectDeserializer.

- serializeEnumAsJavaBean
  
  用于标记 enum 类是否当作普通 Bean 进行序列化.*因为枚举类实质为可数的静态常量对象,通常使用名称则可以获取实例,因此在 Gson fastJson 的默认实现中均为 false,通过 toString 序列化反序列化该类*

- naming
  
  用于选择 PropertyNamingStrategy 的枚举的命名策略.

- serialzeFilters (1.2.49)
  
  为该类型指定特殊的过滤器.

- autoTypeCheckHandler (1.2.71)

  基于 @type 的反序列化时的类型检查.用于限制未知类型的 @type 类型注解字符串导致的安全问题.
  *ParserConfig#addAutoTypeCheckHandler 如果不添加 ParserConfig#AutoTypeCheckHandler 则 @type注解标记的类型参数无法被识别和通过 [auto type 对应的配置参见][https://github.com/alibaba/fastjson/wiki/enable_autotype]*

### @JSONPOJOBuilder (1.2.8)

配合 JSONType#builder 使用,用于重新定义 Builder 构造方法的默认属性设置方法(with 前缀,set 前缀),Builder 模式的默认对象创建方法(build 方法,creat 方法).

## 基础配置

FastJson 的配置分为两部分:Feature 和 Config

FastJson 的配置策略是 JSON facade 类,主要功能类与配置进行抽离,ParserConfig 与 SerializeConfig 均提供 global 的全局配置,供默认使用.同时 JSON facade 类也提供默认的 DEFAULT_PARSER_FEATURE,DEFAULT_GENERATE_FEATURE 反序列化与序列化的特性配置.

### Config 配置

- ParserConfig
  
- SerializeConfig

### Feature 特性配置

#### SerializerFeature 配置对应语义

序列化配置.

#### Feature 配置对应语义

反序列化的配置. fastJson 通过 JSON#DEFAULT_PARSER_FEATURE 字段设置反序列化的默认配置为:AutoCloseSource,InternFieldNames,UseBigDecimal,AllowUnQuotedFieldNames,AllowSingleQuotes,AllowArbitraryCommas,SortFeidFastMatch,IgnoreNotMatch

- AllowArbitraryCommas
  
  允许在 JSONString 中有多个任意的 , 也认为是合法的 JSON.如:{,,,"value":null,,,,},{,,,"value":null,"id":123,,,,}

- AllowComment
  
  允许在 JSONString 中存在 /**/ // 等注解信息也认为是合法的 JSONString 可以正常进行反序列化

- AllowISO8601DateFormat
  
  在默认的集中时间格式对于时间字符串解析失败时,配置了该属性则尝试使用 ISO8601 规范对该时间字符串进行解析.*内置的时间反序列化解析逻辑在 AbstractDateDeserializer#deserialze 方法中*

  内置的时间解析类为 DateCodec,SqlDateDeserializer 用于将时间字符串分别解析为 java.util.Date 与 java.sql.Date.

- AllowSingleQuotes
  
  按照 JSON 的规范,name 值是需要被 "" 包裹的 如:{"a":3} 是一个合格的 JSONString. 但是 {'a':3} , {a:3} 并非是一个合格的 JSONString,配置该属性允许  {'a':3} 可以正常被反序列化.*如果不配置对应的特性则在反序列化时如果遇到上述 JSONString 则会报错 JSON Syntax Error*

- AllowUnQuotedFieldNames
  
  同 AllowSingleQuotes,只不过该标志位是允许 {a:3} 成为一个合格的 JSONString 可以被正常解析.

- AutoCloseSource

  不等同于 JSONReader/JSONWriter 的 Stream Api,用于处理 in/out 流的关闭操作.
  该标志位的设置主要是用于在 JSONString 解析结束时最后一个 token 是否为 JSONToken#EOF,如果不是且设置了该标志位则抛出 JSONException 异常.

- CustomMapDeserializer
  
  基于 [issue 1653][https://github.com/alibaba/fastjson/issues/1653] 添加的特性,主要用于全局使用解析为 JSONObject 时创建的 Map 为特定类型的 Map. 如:大小写不敏感的 Map(org.apache.commons.collections4.map.CaseInsensitiveMap)

- DisableASM

　用于标志禁止使用　ASM 进行优化．

- DisableCircularReferenceDetect
  
  禁止循环引用检测．*TODO://fastjson 是在什么地方执行循环引用检测的？*

- DisableFieldSmartMatch

  禁止在查找字段的 FieldDeserializer 时使用性能更高的基于字段名称的 fnv1a_64 hash 值的二分查找.

- DisableSpecialKeyDetect
  
  对于特殊的引用 key 如 $ref,@xxx,@type,.. 只是当普通键值进行处理.而不是递归查询引用,将引用的值放入构建的对象中.

- ErrorOnEnumNotMatch

  当反序列化的 Bean 中的字段是 Enum 类型时,基于 Enum 字符串匹配 Enum 常量未匹配到时抛出异常.(*基于 Enum 名称匹配 Enum 常量时是大小写不敏感的,具体参见 Issue 2249*)

- IgnoreAutoType
  
  用于配置该次反序列化忽略 @type,typeKey,typeName 指定的 type 类型.采用配置类型进行序列化与反序列化.

- IgnoreNotMatch
  
  如果 JSONString 存在多余的 key,在 Bean 中不存在对应的 Field 则配置了该特性才可以正常解析,否则会报出 JSONException setter not found.

- InitStringFieldAsEmpty

　配置则初始化　String 字段为　"" 而不是　null 引用．

- InternFieldNames
  
  用于调用 String#intern (将字符串放入常量池,用于节省相同字符串的内存空间占用).*目前较新版本的 JVM 默认字符串均会放入常量池,并且对相同的字符串引用相同的常量对象,因为 String 是不可变对象*
  *!!!所以目前并未发现 fastjson 使用该标志位做什么特殊操作,因此使用与不使用该标志位并没有什么差别!!!*

- NonStringKeyAsString
  
  用于在解析成为 JSONObject 对象时将非 String 类型(通常是数字类型,false,true)的 key 转换为 String 存储进入 JSONObject.
  *但是实际测试配置与不配置该属性对于 Issue1633 并无影响,当使用 JSONObject#get 取不到值时如果 key 类型为 Number,Character,Boolean,UUID 则转而会调用 toString 之后再次取值*

- OrderedField
  
  对反序列化得到的值进行排序．Bean 的反序列化该特性并没有什么用．主要用于对　JSONObject,Collection,Set,Map 反序列化时排序的意义较大．(*排序主要基于 LinkedHashMap,默认使用方式为使用插入的先后进行排序,而不是使用访问顺序进行排序*)

- SafeMode
  
  由于 @type ,自动类型在实践中出了很多的安全问题,因此添加了该属性用于在解析时禁止 @type, JSONType#typeName JSONType#typeKey.(从而避免因为动态类型反射导致的安全问题)

- SortFeidFastMatch
  
  反序列化时假设　JSONString 字段是按照字母顺序排列好的，因此在反序列化时利用该特性可以对性能进行提升．(＊后面源码里面并没有特殊处理这个标志位，!!!怀疑已经被废弃!!!＊)

- SupportArrayToBean
  
  将 ["aaa","vvvv"] 按照字段的默认顺序，指定顺序，字母顺序　对应解析到　Bean 的字段中．

- SupportAutoType

  用于配置该次序列化支持 @type autoType 类型注解.(*通常用于在系统配置为不支持 autoType 时,用于当次序列化特殊配置支持 autoType 类型*)

- SupportNonPublicField
  
  支持非 public 字段的序列化与反序列化.(*默认 fastjson 不支持非 public 字段,且没有 setter,getter 方法的Bean 的序列化与反序列化*)
  *如果字段是非 public,但是具有 public 的 setter/getter 也认为是 public 字段*
  *但是序列化时却不支持配置非 public 字段支持序列化,为什反序列化时支持非 public 字段支持反序列化?*

- TrimStringFieldValue
  
  为 [issues 3279][https://github.com/alibaba/fastjson/issues/3279] 添加的属性,用于将 String 类型的字段的值前后空格进行 trim 操作.

- UseBigDecimal
  
  NumberDeserializer 用于解析 byte,float,double 及其 boxed 类型.额外还提供对于 Number 类型的解析,*Number 作为抽象类并没有具体实现,因此在解析时遇到需要改类型的字段则需要 fastjson 进行决策提供实现,配置了该属性则统一使用 BigDecimal 的实现进行返回,如果未配置则提供 Double 类型的实现返回*

- UseObjectArray

  基于 [ISSUES 423][https://github.com/alibaba/fastjson/issues/423] 添加的对于该特性的支持.
  配置了该属性,使用 JSON#parse 进行的反序列化,内部的 JSONArray 表示均会转换成为 Object[] 的数组形式.

### PropertyNamingStrategy

## 自定义序列化/自定义反序列化/泛型支持/自定义序列化反序列化过滤器

### TypeReference

### ObjectSerializer/ObjectDeserializer

- JavaBeanSerializer/JavaBeanDeserializer
  
  默认的通用的 JAVA Bean 的序列化器与饭序列化器.该 Bean 如果在 PC JVM 平台如果支持 ASM 优化则通过 ASMSerializerFactory/ASMDeserializerFactory 创建继承自 JavaBeanSerializer/JavaBeanDeserializer 经过优化的 ASM 版本.

### 序列化过滤器(SerializeFilter)

下述 8 种 SerializeFilter 均由 SerializeFilterable 持有并且管理交互流程.版本 1.2.10 之后,fastjson 通过 SerializeConfig#addFilter 支持以类级别添加 SerializeFilter(为了提升性能).同时也支持通过 JSON#toJSONString 整体进行过滤器的添加.

- PropertyPreFilter
  
  *过滤字段是否需要序列化*
  根据PropertyName判断是否序列化(通常还可以结合 SerialContext 与 name 判断特定 JSON Path 的名称的字段是否需要进行序列化),*apply 参数的 Object 值并非 name 所对应的 value 值,而是 name 所属的 Object 的值*
  
- PropertyFilter
  
  *过滤字段是否需要序列化*
  根据PropertyName和PropertyValue来判断是否序列化,相比 PropertyPreFilter 多了一个对应 name 的值.*同样的 object 参数同样为 name 与 value 所属的对象的值*

  上述两个 Filter 均是用来过滤特定字段是否需要序列化.PropertyPreFilter 的实现为 2012/07 较 PropertyFilter 的实现 2011/07 晚.因此提供的功能更加强大(*可以根据 JSON Path 判断特定对象的 name 是否需要进行序列化*)

  上述两个 Filter 的应用规则也是先 PropertyPreFilter(261 行) 再 PropertyFilter(293 行).

- NameFilter

  *替换写入 JSON String 的 Name 值*
  修改Key，如果需要修改Key,process返回值即可,不修改则返回 key 的原值即可.其提供当前对象所在 Object,以及 name 和 Value 值.(应用在 304 行)

- ValueFilter
  
  *替换写入 JSON String 的 Value 值*
  修改Value,不修改则原值返回.(应用在 307 行)
  
- BeforeFilter
  
  *在序列化 JSON String 对象最前面添加 name value 值*
  序列化时在对象最前添加内容,在 JavaBeanSerializer 中被使用.(应用在 229 行)
  
- AfterFilter
  
  *在序列化 JSON String 对象最后面添加 name value 值*
  序列化时在对象最后添加内容,在 JavaBeanSerializer 中被使用 (应用在 502 行)

- ContextValueFilter

  *替换写入 JSON String 的 Value 值*
  带有字段上下文信息的,修改 Value 的过滤器.后面添加的 API 实现在 2016/04 ,前面的单纯的 ValueFilter 实现在 2011/07 (应用在 307 行,先 ContextValueFilter 再 ValueFilter)

- LabelFilter

  *过滤字段是否需要序列化*
  通常建议应用分组过滤时添加该标签
  基于字段注解 JSONField#label 的值进行匹配, excludes 规则高于 includes 规则. 同时未配置 label 的字段 label 则为空字符串,为空时该字段是否过滤依赖于 exclude/include 规则.空字符串不在 exclude 中则不过滤,空字符串在 include 中才输出.实现在 2015/08 .(应用在 262 行)

### 反序列化过滤器(ParseProcess)

通过 JSON#parseObject 可以应用 ParseProcess 的序列化处理规则

- ExtraProcessor
  
  用于处理多余的字段(多余字段定义: JSON String 中存在的字段,但是对应指定的 Bean 类型中并不存在的字段)
  处理不存在的 name 和 value 值.

- ExtraTypeProvider

  更改多余的字段的类型,然后再应用 ExtraProcessor.(多余的字段通常不是了行不同)

- FieldTypeResolver
  
  反序列化过程中用于推测内部字段的类型,避免了基于 reflect api 进行类型判断的低效.

  上述三个个接口可以同时实现,然后传递进入 JSON#parseObject 进行使用.

- PropertyProcessable

  Bean 实现该接口,用于处理 JSON String ,Bean 类型自己定义自己的反序列化过程.该类借助于 PropertyProcessableDeserializer 实现自己的反序列化过程.(2017/07 提供)

- ExtraProcessable

  Bean 类型实现该字段,用于处理反序列化时该 Bean 中不存在的字段.*实现 ExtraProcessable 的同时建议对称的实现 JSONSerializable 处理该 Bean 的序列化* (该 API 接口由 2016/04 提供)

## FastJson 中的扩展

- Json 的循环引用

[JSON 处理引用的语法][https://github.com/alibaba/fastjson/wiki/%E5%BE%AA%E7%8E%AF%E5%BC%95%E7%94%A8],Gson 对于循环引用则为配置不处理,或者直接引用循环引用解析导致 StackOverflow.

## 一些辅助功能

- JSONPObject

提供服务端便捷的返回 JSONP 格式的信息给 H5.(H5 通过 Script 标签请求数据,解决 ajax 请求无法跨域问题,被同源策略限制,但是存在 Cross-Site Request Forgery (CSRF) 攻击)

## 一些便捷之处(有趣的地方)

- retrofit 内置提供了对于 gson,guava,jackson,java8,jaxb,moshi,protobuf,scalars,simplexml,wire 等数据格式的支持的 Converter,但是没有提供对于 fastjson 支持的 Converter.因此 fastjson 在其官方库 support 包中提供了对于 retrofit 的支持.

- Android 端较多使用 Gson 而非 fastJson?
  
  因为 fastJson 的实现中嵌入了太多对于服务端框架的支持,如:serverlet,spring 等.
