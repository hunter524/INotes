# FastJson 源码分析

## 序列化流程

### SerializeWriter

持有 Writer 负责序列化过程中　JSON String 的构造（根据　SerializerFeature　对特殊情况进行一些字符串的处理逻辑)

### JSONSerializer

序列化流程控制，解析待序列化的数据 Bean.持有　SerializeWriter，SerializeFilter．

### SerialContext

用于标记当前序列化流程中正在序列化的对象的 JSON Path 表示.

### SerializeFilter/SerializeFilterable

SerializeFilterable 持有 SerializeFilter 负责管理和应用添加进入的 SerializeFilter.

SerializeFilterable 的实现类有:

- JSONSerializer
  
  JSON#toJSONString 相关方法,将 Json Object 序列化成为 Json String 时使用的为该类.

- JavaBeanSerializer
- MapSerializer

## 反序列化流程

### DefaultJSONParser

持有 JSONLexer,ParserConfig 通过 JSONLexer 主导解析流程,协调 ParserConfig 进行解析数据的配置状态控制.

### JSONLexer/JSONLexerBase/JSONScanner/JSONReaderScanner

不同类型的 Json String 的语法解析器

### ObjectDeserializer

持有 DefaultJSONParser 负责不同类型的对象类型进行反序列化.

- 未注册序类型序列化

ParserConfig#createJavaBeanDeserializer,如果配置 java 平台支持 ASM 则使用 ASM 进行性能优化.(通过 ASMDeserializerFactory#createJavaBeanDeserializer)如果平台不支持 ASM 或者 配置不使用 ASM,则直接构建 JavaBeanDeserializer(同 Gson 使用反射 API 进行解析反序列化对象)

*使用 ASM 和 不使用 ASM 相比该处的性能优化在何处?*

性能优化在于对于被构建对象的赋值操作,不使用 ASM 则需要通过 Field 提供的反射 API 进行字段赋值,使用 ASM 则直接可以使用生成字节码,直接访问被构建的对象字段和进行赋值.生成赋值的字节码相对使用使用反射 API 则性能更优.*然而对于 Android 平台这点优化并不能被使用*

### ParseContext

## OQL(JSONPath)

### JSONPath

JSON OQL 匹配路径的表示形式

### JSONPathParser

JSONPath 的内部类负责对于 JSONPath 的 OQL(Object Query Language) 语言的解析.将 path 查询匹配语句解析成为一组不同功能的 Segment 表述用于查找对应的字段属性.

JSONPath#readSegement 是整个 JSONPath 对象的核心内容,其负责解析 JSONPath 的 OQL 语法形成 Segment 链用于对 JSONObject/JSONString 进行 eval 估值操作.

### Segment

通过对匹配 Path 的解析,将匹配过程抽象成为不同阶段的不同类型的 Segment 实现.Segment 通过 eval,extract 抽象对于不同匹配规则的匹配过程.*前一个 Segment 的匹配结果交由后一个 Segment 进行匹配,直到最后一个 Segment 获得结果*

```json
{
  "firstName": "John",
  "lastName" : "doe",
  "age"      : 26,
  "address"  : {
    "streetAddress": "naist street",
    "city"         : "Nara",
    "postalCode"   : "630-0192"
  },
  "phoneNumbers": [
    {
      "type"  : "iPhone",
      "number": "0123-4567-8888"
    },
    {
      "type"  : "home",
      "number": "0123-4567-8910"
    }
  ]
}
```

- ArrayAccessSegment
- FilterSegment
- FloorSegment
  
  用于支持 $.c.floor() 语法.小数向 -Infinity 舍入取整. 支持 Float,Double,BigDecimal.同时也支持 Byte,Short,Interger,Long,BigInteger 则是直接取整数返回.

- KeySetSegment
- MaxSegment
- MinSegment
  
  用于支持如 $..c.min() 的 path 语法.获取到匹配的结果,然后获取其中最小的值.*匹配到的值的类型是需要是 Number 的子类,或者继承了 Comparable 接口,且匹配到的结果必须是 Collection 集合类型*

- MultiIndexSegment
  
  用于支持 $[1,2,3] 的写法去匹配数组中指定多个下标的元素.同时也支持 $[last] 的表示 等同于 $[-1] 用于表示匹配数组的最后一个元素.*但是不支持 first .*

- MultiPropertySegment
- PropertySegment
- RangeSegment
- SizeSegment
  
  获取匹配到的结果的大小.如: $.size()/$.length() => 根元素如果是 Collection,array,Map 则直接返回其大小,如果是 POJO 对象则通过 JavaBeanSerializer#getSize 获得其可以被序列化的元素的个数.更多的使用参见 JSONPath_16 的测试用列.*测试用列中显示 size 匹配可以多个组合进行使用,然后进行条件匹配,如匹配规则: $.* ? (@.type() == "array" && @.size() > 1)*
  *$.size()/$.length() <==> $.size/$.length*

- TypeSegment
  
  获取匹配到的元素的类型.
  常见的使用方法如 : $.type() => 获取根对象的类型, $.id.type() => 获取根对象名称为 id 的值的类型, $[0].xx.type() => 获取数组对象的第 0 个元素的 xxx 属性的 类型.

  type() 语法主要能区分识别的类型有: null,array,number,boolean,string(包含 String,UUID,Enum) 其他类型统一识别为 object.

  更多的使用参见 JSONPath_16 的测试用列.

- WildCardSegment

  TODO://objectOnly 只在 extract 时才有效?eval 时并没有判断该状态
  
  通用的泛匹配模式,主要分为三种状态:

  deep:false objectOnly: false -> 匹配path: * , $.\* ,$.phoneNumbers.\*

  获取当前匹配对象的所有的值的集合,并且不向下进行对象的递归操作

  匹配上述JSON的匹配结果分别为:

  ```json
  [
    "John",
    "doe",
    26,
    {
      "streetAddress": "naist street",
      "city": "Nara",
      "postalCode": "630-0192"
    },
    [
      {
        "type": "iPhone",
        "number": "0123-4567-8888"
      },
      {
        "type": "home",
        "number": "0123-4567-8910"
      }
    ]
  ]
  ```

  ```json
  [
  {
    "type": "iPhone",
    "number": "0123-4567-8888"
  },
  {
    "type": "home",
    "number": "0123-4567-8910"
  }
  ]
  ```

  deep:true objectOnly: false -> 匹配path: $..*

  递归匹配当前匹配对象的儿子,孙子对象的属性,并且将儿子,孙子对象的属性值摊平输出到外层 List 结果中.

  deep:true objectOnly: true -> 匹配path: $..*

### 基于 Filter 的 Segment

主要是实现了 Filter 接口,用于按照指定条件过滤匹配到的值

- DoubleOpSegement
- FilterGroup
- IntBetweenSegement
- IntInSegement
- IntObjInSegement
- IntOpSegement
- MatchSegement
- NotNullSegement
- NullSegement
- PropertyFilter
- RefOpSegement
- RegMatchSegement
- RlikeSegement
- StringInSegement
- StringOpSegement
- ValueSegment

## JSONPatch(补丁更新 JSON)

为 JSON 字符串提供了 patch 补丁更新的操作.提供了基于 path 路径的 add,remove,replace,copy,move test操作.

### JSONPatch#Operation

为 JSON 补丁操作的数据模型,其可以序列化成为 JSONString 也可以直接提供给 JSONPatch 进行补丁更新操作.

## 泛型支持(TypeReference)

等同于 Gson 的 TypeToken 通过 JDK 内置的 Class#getGenericSuperclass 获得已经被具化的泛型类的TypeReference 的泛型类型.
