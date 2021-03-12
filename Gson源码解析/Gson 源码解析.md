# Gson 源码解析

gson 项目 init 提交 2008,fastjson,jackson 项目init 均为 2011.

后面规划对比 [fastjson][https://github.com/alibaba/fastjson] [jackson][https://github.com/FasterXML/jackson] 的实现进行源码解析.

[jackson 源码][https://github.com/FasterXML/jackson-core]

哪儿有岁月静好?只是有人在为我们默默负重前行! What I cannot create,I do not understand --by Richard Feynaman

## 使用层面的 API

- GsonBuilder
  
  配置构建 Gson 对象时的选配属性(如:version,是否启用 Expose 注解,命名策略,是否解析非静态内部类,long 字段序列化策略(字符串形式解析 or 数字形式解析),toJson 时 Json String 是否美观,JsonSerializer/JsonDeserializer,TypeAdapter 配置自定义的类型解析器,InstanceCreator 指定对象的创建器)用于构建 Gson 对象.

- TypeToken

  通过实现该类的匿名内部类传递具化的泛型类型作为 TypeToken 的泛型参数从而获得具化的泛型类型的泛型的类型,封装泛型反射高级Api,Class#getGenericSuperclass 用于获得泛型类的泛型参数类型.提供便捷 Api 用于获得指定 component 类型的 TypeToke 携带的泛型信息为 GenericArrayType,获取指定泛型参数的 ParameterizedType 携带指定的泛型信息.
  
- TypeAdapter/JsonDeserializer/JsonSerializer
  
  实现这些接口,构造 Gson 时向 GsonBuilder 注册指定类型的序列化和反序列化操作.

- TypeAdapterFactory
  
  通过 GsonBuilder#registerTypeAdapterFactory 添加的 TypeAdapterFactory 后添加的优先级高于先添加,构建成 Gson 对象后会内置再添加 52 种内置的类型适配器(*内置的类型适配器位于 TypeAdapters*),用于 java 内置的类型进行序列化反序列化的转换.

  可以借助 Gson#getDelegateAdapter 跳过当前后配置的用于代理相同类型的 TypeToken 的 TypeAdapterFactory(*文档说无法代理基础类型long,int 等类型的转换器进行代理进行统计和增强功能!!!!实际测试可以统计基础类型的序列化/反序列化次数.参见 StaticIntAdapterMain!!!!*)

  TypeToken 与 TypeAdapter 的匹配策略: 初次匹配是通过遍历 Gson 中注册的 TypeAdapterFactory 并尝试通过这些 Factory 创建 TypeAdapter 如果创建的 TypeAdpater 非 null 则直接使用创建的 TypeAdapter 对该 TypeToken 进行转换,并建立 TypeToken 与 TypeAdapter 的缓存. 二次匹配则使用缓存进行匹配.

- InstanceCreator
  
  注册指定类型的对象在反序列化时的创建方式而不是使用,默认的 ConstructorConstructor 中定义的创建方式创建 ObjectConstructor 从而进一步创建 Object 对象.

- FieldNamingPolicy/FieldNamingStrategy
  
  FieldNamingPolicy 为枚举常量,继承自 FieldNamingStrategy ,其内置了6中默认的 Json 序列化/反序列化的实现.(*具体使用方法参见 Gson 使用备忘*).使用者也可以通过自定义 FieldNamingStrategy 实现自己的命名策略应用于序列化与反序列化中.通过 GsonBuilder#setFieldNamingStrategy 和 GsonBuilder#setFieldNamingPolicy 设置命名策略,重复设置遵循 last wins 规则.默认构建的 Gson使用的命名策略为 FieldNamingPolicy#IDENTITY 即名称不做任何转换和映射.
  
## 解析层面的源码

### TypeToken(java 泛型类型系统)

当被序列化反序列化的类型为已经参数化的类型，则需要通过实例化TypeToken子类传入已经参数化的泛型类型，间接通过Jdk Class#getGenericSuperclass 获得已经被参数化的泛型类型信息．如：

```java
 Type listType = new TypeToken<List<String>>() {}.getType(); // 该处即构建了TypeToken子类，为匿名类的写法，传递进入的泛型参数类型　List<String>　则可以通过上述　Api 获得　List 中的泛型类型．
 List<String> target = new LinkedList<String>();
 target.add("blah");

 Gson gson = new Gson();
 String json = gson.toJson(target, listType);
 List<String> target2 = gson.fromJson(json, listType);
```

TypeToken 存在get,getParameterized,getArray 等静态工厂方法，传入类型直接获得　TypeToken 从而获得序列化，反序列化时需要使用的 Type 类型．*int.class != Integer.class 基础类型的 Class 与其包装类型的 Class 并不相等*

### JsonElement(基础解析元素抽象)

Gson#toJsonTree 获得 JsonElement 解析树抽象,使使用者自行面向解析树进行解析.(*只能将 Bean Object 转换成为 JsonElement 无法转换 Json String 转换为 JsonElement*)

- JsonArray
- JsonNull
- JsonObject
- JsonPrimitive

### Excluder(解析过滤规则抽象)

- Version 过滤
  
  GsonBuilder#withVersion 设置当前的版本号,使用@Since/@Until 标记字段支持的起始版本号和结束版本号.

- Modifiers 过滤
  
  GsonBuilder#excludeFieldsWithModifiers 设置标记了哪些修饰符的字段被剔除(*默认配置为标记了 transient,static 的字段会被过滤掉,用户可以配置的访问修饰符合为 java.lang.reflect.Modifier 中的常量*)

- InnerClasses 过滤

  使用 GsonBuilder#disableInnerClassSerialization 标记某个字段为内部类时则忽略解析.
  *为什么要特意标记忽略对于内部类字段的解析?*

  因为内部类持有外部类的引用,当内部类存在某个字段持有外部类 OuterClass.this 的引用时会形成有环图,造成解析StackOverflowError 异常.

- Expose 过滤
  
  如:Gson 使用备忘.配合 GsonBuilder#excludeFieldsWithoutExposeAnnotation 构建配置标记某个字段是否需要被序列化和反序列化.(*如方法名称如果某个字段没有被标记注解则直接忽略*)

- ExclusionStrategy 策略过滤

参见 Gson 使用备忘的 自定义 ExclussionStrategy 解析策略.

### TypeAdapter/JsonReader/JsonWriter

通过对 ReflectiveTypeAdapterFactory#Adapter 的源码分析,Json 的序列化和反序列化均是类型主导的.(*该处 JsonReader 读取,JsonWriter 写入在底层API 的使用上均是存在状态的,如:通过 JsonDeserializer/JsonSerializer 实现自定义的序列化和反序列化解析数据*)

#### TypeAdapter

每一个类型的Json 序列化/反序列化的适配器.通过 TypeAdapter#write 读取指定类型 bean 的属性值向 JsonWriter 进行写入.通过 TypeAdapter#read 方法读取 JsonReader 构造指定类型的 Bean 进行返回.

#### JsonReader

Json String 反序列化的核心类,负责对Json String 进行解析返回 Bean 对象数据.

- JsonReader

- JsonTreeReader

#### JsonWriter

负责对 Bean 对象数据进行解析(序列化),将解析出来的值写入 JsonWriter 序列化成为　Json String.
*JsonWriter 的核心抽象方法为 beginObject,endObject,beginArray,endArray,name,value 6个三组方法分别标记对象的开始结束,数组的开始结束,字段基础值的名称/值的写入*
*JsonWriter 主导了整个 Bean 进行序列化时的状态控制,错误检测,同时因为 JsonWriter 没有做并发安全性的设计和实现,因此 Gson 官方文档称之为线程安全的类*

- JsonWriter

  基础的 Json 序列化解析 API 抽象,抽象成为更高级的 Json 序列化解析 API(如:beginArray,endArray,beginObject,endObject,name,value 等简便的 API),避免了对 { } [] , : " 等基础 Json 元素的操作.同时生成的字符串委托给了底层的 StringWriter,JsonWriter 借助 StringWriter 生成 Json String.

- JsonTreeWriter

  JsonWriter 的子类,覆写了 beginArray,endArray,name,value 等方法,用于生成 JsonElement 的解析树.

- JsonWriter

内部实现思路:

stack 字段用于存储当前写入值的状态,采用栈的模式进行压栈和出栈操作,stackSize 标记当前栈的深度

deferredName 为暂缓写入的基础字段,对象,数组的名称属性值,在写入字段值,开始写入对象内部元素,开始写入数组元素时会将 name 值写入最终的 Writer.

indent.separator: 缩进标识符,name,value 分割标识符.
美观模式下:Gson 创建 JsonWriter 时会设置 indent 为两个空格的缩进,separator为 : 后面紧跟一个空格.
普通(简洁模式下): indent 设置为 null,separator 设置为 : 没有前置后置空格.

状态机:

JsonWriter 分别通过 beforeName,beforeValue 在写入名称,写入值,开始写入对象,开始写入数组时执行状态机类型的转换.

*顶层写入的是值:*

EMPTY_DOCUMENT(6) --JsonWriter#value 写入值-> NONEMPTY_DOCUMENT(7) --> close 状态
*顶层可以写入单独的值,但是无法写入 "key":"value" 这样的简单键值对,这种键值对需要在 Object 作为顶层元素的内部,因为在 beforeName 执行状态机转换写入值时要求前一个状态机要么是 NONEMPTY_OBJECT(已经存在值的对象)或者是 EMPTY_OBJECT (不存在值的对象)否则抛出异常,无法写入值.写入完成 name 值后会将状态机顶部置为 DANGLING_NAME 状态等待值的写入(下一个写入的必须是值,数组,或者一个新的对象),值写入完成之后又会把状态值置为 NONEMPTY_OBJECT 等待下一个值的写入*
*上述条件要求名称的写入必须在对象内部,无法在数组或者顶层对象上*

宽容模式下能写入 n 个顶层值(如 {"top1":"value1"} {"top2":"value2"} 这一个 Json String 则有两个顶层值)

*顶层写入的是 Object:*

EMPTY_DOCUMENT(6) --JsonWriter#beginObject--> 先将栈定元素替换成为 NONEMPTY_DOCUMENT,然后压栈 EMPTY_OBJECT --JsonWriter#name--> 栈不变,只将名称暂存进入deferredName -- JsonWriter#value --> 检查 name 写入是否符合条件,先替换栈顶状态由 EMPTY_OBJECT 为 DANGLING_NAME 等

对应栈的状态变化:
[EMPTY_DOCUMENT]-beginObject->[NONEMPTY_DOCUMENT,EMPTY_OBJECT]-name->[NONEMPTY_DOCUMENT,EMPTY_OBJECT]只写入name 不改变栈 -

## ConstructorConstructor(对象构建)

在 Gson 库中主要负责两类对象的构建:

- JsonAdapter 注解标记的 TypeAdapter/TypeAdapterFactory/JsonSerializer/JsonDeserializer 或者其子类对象,用于构建注解标记的 TypeAdapter 对象的实例(*此时 ConstructorConstructor 会被传递进入 JsonAdapterAnnotationTypeAdapterFactory 辅助对 JsonAdapter注解的解析和 TypeAdapter 实例的构建*)

- Gson 反序列化时数据实体对象的构建.在 Gson 构建时将 ConstructorConstructor 分别传递给了 CollectionTypeAdapterFactory,MapTypeAdapterFactory,ReflectiveTypeAdapterFactory 分别用于集合对象,Map 对象,普通 JOPO 等对象的构建.

ConstructorConstructor 创建 ObjectConstructor 的策略:

- 首先通过带有泛型信息的 Type 类型查找 InstanceCreator 创建 Bean
  
- 上面的查找不到,则通过 rawType(Class) 不带有泛型信息的类型查找 InstanceCreator

- 上面查找不到,则查找不带有参数的默认构造方法,并尝试通过该默认构造方法构造该对象
  
- 如果传入的是 Collection(ArrayList),SortedSet(TreeSet),EnumSet(EnumSet),Set(LinkedHashSet),Queue(ArrayDeque),Map(LinkedTreeMap),ConcurrentNavigableMap(ConcurrentSkipListMap),ConcurrentMap(ConcurrentHashMap),SortedMap(TreeMap),的某个子接口类型(不存在构造方法).则尝试推断其实现类型(通常是 Set,Map,Collection 等集合的实现)

- 最后如果通过上述方法均无法查找到构建实例的方法,则通过终极大招,使用 Unsafe#allocateInstance 构造指定类的实例(构造的实例不会调用任何构造方法),如果还找不到,则尝试 dalvik jvm 虚拟机 2.3 之后的 ObjectStreamClass#newInstance(Class<?> clazz, long constructorId) 进行对象的构造.在 虚拟机 2.3 之前则通过 ObjectInputStream#newInstance 进行对象的构建.

- 如果还找不到,则放弃该类型的对象的构建,抛出异常.

### ObjectConstructor

只存在 ObjectConstructor#construct 调用该方法用于创建指定的对象,由 ConstructorConstructor#get 方法创建的匿名内部类,返回给调用者,用于被调用者用来构建指定对象.

## Gson 库的设计决策

[Gson 官方设计决策文档][https://github.com/google/gson/blob/master/GsonDesignDocument.md]


