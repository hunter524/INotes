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

- POJO(plain ordinary java object) 简单无规则 java 对象,纯的传统意义的 java 对象。就是说在一些 Object/Relation Mapping 工具中，能够做到维护数据库表记录的 persisent object 完全是一个符合 Java Bean 规范的纯 Java 对象，没有增加别的属性和方法。我的理解就是最基本的 Java Bean ，只有属性字段及 setter 和 getter 方法！。

- DAO(data access object) 数据访问对象,是一个 sun 的一个标准 j2ee 设计模式， 这个模式中有个接口就是 DAO ，它负持久层的操作。为业务层提供接口。此对象用于访问数据库。通常和 PO 结合使用， DAO 中包含了各种数据库的操作方法。通过它的方法 , 结合 PO 对数据库进行相关的操作。夹在业务逻辑与数据库资源中间。配合 VO, 提供数据库的 CRUD 操作.

## JsonPath

JsonPath 是一种类似于 XPath 的 Json 查询语句表达式的规范.[其较早的java 实现库][https://github.com/json-path/JsonPath], 目前 FastJson 通过内置的 JSONPath 默认支持该特性.

其语法表达式[参见][https://github.com/json-path/JsonPath/blob/master/README.md]

## JSON Facade 类

### JSON Facade 类常用方法

JSON 支持在 JSON String 中添加类型信息.如 TreeSetTest 中的序列化一个类可以添加类型信息,同样反序列化的字符串也可以支持带有类型信息.(*可以认为是 FastJson 对于 JSON 规范的一种自定义扩展,用语指示具体的泛型类型信息*)

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

### @JSONField

- ordinal(Since 1.1.42)
  
  用于标记对象内部字段序列化/反序列化时的顺序 -> JSONFieldTest_0

- name
  
  序列化/反序列化时指定的该字段的名称,不使用该字段的属性名称(通常用于混淆类) -> AnnotationTest3

- format
  
  序列化/反序列化时指定时间的格式 -> DateFieldTest4

- serialize
  
  是否序列化

- deserialize

  是否反序列化

- serialzeFeatures
  
  对特定字段指定序列化时的 SerialzeFeature.
  TODO://JSON#toJsonString 时指定的 Feature 与 对某个特定的字段指定的 Feature 谁的优先级更高?
  按照推测应该是注解的优先级更高

- parseFeatures
  
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

- alternateNames(Since 1.2.21)
  
  反序列化时指定的对应字段,在 JSONString 中的可选的名称.

- unwrapped(Since 1.2.31)
  
  缩减对象层级,将当前字段中的元素,摊平到外成对象中. -> JSONFieldTest_unwrapped_0-6

- defaultValue(Since 1.2.61)
  
  序列化时当某个字段为 null 时则采用该注解标记的默认值.(*只对对象类型有效,对于基础数据类型无效,因为基础数据类型即使不赋值,也存在默认值*)

### @JSONPOJOBuilder

### @JSONType

## 基础配置

FastJson 的配置分为两部分:Feature 和 Config

FastJson 的配置策略是 JSON facade 类,主要功能类与配置进行抽离,ParserConfig 与 SerializeConfig 均提供 global 的全局配置,供默认使用.同时 JSON facade 类也提供默认的 DEFAULT_PARSER_FEATURE,DEFAULT_GENERATE_FEATURE 反序列化与序列化的特性配置.

### Config 配置

- ParserConfig
  
- SerializeConfig

### Feature 特性配置

- SerializerFeature

序列化配置.

- Feature

与序列化对应的反序列化的配置.

### PropertyNamingStrategy

## 自定义序列化/自定义反序列化/泛型支持/自定义序列化反序列化过滤器

### TypeReference

### ObjectSerializer/ObjectDeserializer

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

## 一些辅助功能

- JSONPObject

提供服务端便捷的返回 JSONP 格式的信息给 H5.(H5 通过 Script 标签请求数据,解决 ajax 请求无法跨域问题,被同源策略限制,但是存在 Cross-Site Request Forgery (CSRF) 攻击)

## 一些便捷之处(有趣的地方)

- retrofit 内置提供了对于 gson,guava,jackson,java8,jaxb,moshi,protobuf,scalars,simplexml,wire 等数据格式的支持的 Converter,但是没有提供对于 fastjson 支持的 Converter.因此 fastjson 在其官方库 support 包中提供了对于 retrofit 的支持.
