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

### 反序列化流程

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
