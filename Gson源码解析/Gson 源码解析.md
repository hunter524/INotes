# Gson 源码解析

gson 项目 init 提交 2008,fastjson,jackson 项目init 均为 2011.

后面规划对比 [fastjson][https://github.com/alibaba/fastjson] [jackson][https://github.com/FasterXML/jackson] 的实现进行源码解析.

[jackson 源码][https://github.com/FasterXML/jackson-core]

哪儿有岁月静好?只是有人在为我们默默负重前行!

## 使用层面的 API

- GsonBuilder
- TypeToken
- TypeAdapter/JsonDeserializer/JsonSerializer
- TypeAdapterFactory
  
  通过 GsonBuilder#registerTypeAdapterFactory 添加的 TypeAdapterFactory 后添加的优先级高于先添加,构建成 Gson 对象后会内置再添加 52 种内置的类型适配器(*内置的类型适配器位于 TypeAdapters*),用于 java 内置的类型进行序列化反序列化的转换.

  可以见借助 Gson#getDelegateAdapter 跳过当前后配置的用于代理相同类型的 TypeToken 的 TypeAdapterFactory(*文档说无法代理基础类型long,int 等类型的转换器进行代理进行统计和增强功能!!!!实际测试可以统计基础类型的序列化/反序列化次数.参见 StaticIntAdapterMain!!!!*)

  TypeToken 与 TypeAdapter 的匹配策略: 初次匹配是通过遍历 Gson 中注册的 TypeAdapterFactory 并尝试通过这些 Factory 创建 TypeAdapter 如果创建的 TypeAdpater 非 null 则直接使用创建的 TypeAdapter 对该 TypeToken 进行转换,并建立 TypeToken 与 TypeAdapter 的缓存. 二次匹配则使用缓存进行匹配.

- InstanceCreator
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

- expose 过滤
  
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

负责对 Bean 对象数据进行解析(序列化),将解析出来的值写入 JsonWriter 序列化成为　JsonString

- JsonWriter

  基础的 Json 序列化解析 API 抽象,抽象成为更高级的 Json 序列化解析 API(如:beginArray,beginObject,name,value 等简便的 API),避免了对 { } [] , : " 等基础 Json 元素的操作.同时生成的字符串委托给了底层的 StringWriter,JsonWriter 借助 StringWriter 生成 Json String.

- JsonTreeWriter

  JsonWriter 的子类,覆写了 beginArray,endArray,name,value 等方法,用于生成 JsonElement 的解析树.

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


