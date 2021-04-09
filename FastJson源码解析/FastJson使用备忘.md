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

## JSON Facade 常用方法

- JSON#parse
  
  JSON String => 内置的 JSONObject,JSONArray,Set

- JSON#parseObject
  
  转换为指定类型的 Bean

- JSON#parseArray

  解析成为 JSONArray,List <T:>

- JSON#toJSONString/JSON#toJSONBytes

  序列化成为 String 或者 byte[]

- JSON#writeJSONString
  
  将序列化的结果写入 Writer,OutputStream

-JSON#toJSON

  将指定的 Object 转换成为 JSONObject,JSONArray 甚至是 JSONString.

## 注解使用

### @JSONCreator

### @JSONField

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

下述 8 种 SerializeFilter 均由 SerializeFilterable 持有并且管理交互流程.

- PropertyPreFilter
  
  根据PropertyName判断是否序列化(通常还可以结合 SerialContext 与 name 判断特定 JSON Path 的名称的字段是否需要进行序列化)
  
- PropertyFilter
  
  根据PropertyName和PropertyValue来判断是否序列化,相比 PropertyPreFilter 多了一个对应 key 的值.

- NameFilter

  修改Key，如果需要修改Key,process返回值即可.其提供当前对象所在 Object,以及 Key 和 Value 值.
  
- ValueFilter
  
  修改Value
  
- BeforeFilter
  
  序列化时在最前添加内容,在 JavaBeanSerializer 中被使用
  
- AfterFilter
  
  序列化时在最后添加内容,在 JavaBeanSerializer 中被使用

- ContextValueFilter

  带有字段上下文信息的,修改 Value 的过滤器.

- LabelFilter

  基于字段注解 JSONField#label 的值进行匹配, excludes 规则高于 includes 规则.

### 反序列化过滤器(ParseProcess)

- ExtraProcessor

- ExtraTypeProvider

- PropertyProcessable

- FieldTypeResolver

## 一些便捷之处(有趣的地方)

- retrofit 内置提供了对于 gson,guava,jackson,java8,jaxb,moshi,protobuf,scalars,simplexml,wire 等数据格式的支持的 Converter,但是没有提供对于 fastjson 支持的 Converter.因此 fastjson 在其官方库 support 包中提供了对于 retrofit 的支持.
