# Gson 使用备忘

Gson 是线程安全的,多个线程可以共享同一个 Gson 对象执行序列化反序列化操作.(*即Gson 对象是 Stateless 的,其内部不维护任何状态*)

实现策略为由类型主导序列化/反序列化流程,而不是由Json String 主导该流程.

## Gson 序列化/反序列化(Object)

- 推荐使用 private 字段.(配合 setter,getter 使用)

- 需要 序列化/反序列化 的字段不需要注解,默认会包涵当前类及父类中所有的字段.

- transient 字段在 序列化/反序列化 时会默认被忽略(*为了配合 java 语言自带的 序列化/反序列化规范*)

- null 字段的处理:
  
  - 序列化时 null 字段会被葫芦

  - 反序列化时 Json String 中不存在的字段,Object 中存在的字段会被设置为默认值.(object -> null, number type -> 0,boolean -> false)

- 序列化/反序列化 合成字段会被忽略

- 序列化/反序列化 内部类,匿名类,本地类指向外部类的字段会被忽略.

## Gson 序列化/反序列化(嵌套类/内部类)

### 静态内部类/嵌套类

可以被正常 序列化/反序列化

### 非静态内部类/嵌套类

非静态内部类因为其持有外部类的引用,因此其如果需要在不改变为静态类的情况下可以被序列化则需要使用 InstanceCreator 机制,由用户通过外部类的实例创建内部类和嵌套类.(*但是 Gson 不推荐这么做*)

```java
public class A { 
  public String a; 

  class B { 

    public String b; 

    public B() {
      // No args constructor for B
    }
  } 
}
```

```java
public class InstanceCreatorForB implements InstanceCreator<A.B> {
  private final A a;
  public InstanceCreatorForB(A a)  {
    this.a = a;
  }
  public A.B createInstance(Type type) {
    return a.new B();
  }
}
```

## Gson 序列化/反序列化(数组/集合)

反序列化 对于集合需要使用 TypeToken,因为集合存在泛型参数,java 中对于泛型的实现为类型擦除.

### 数组/集合的类型不统一

如:\['hello',5,{name:'GREETINGS',source:'guest'}\]

- [使用 JsonParser 低级 API,自己解析数组中的每个元素][https://github.com/google/gson/blob/master/extras/src/main/java/com/google/gson/extras/examples/rawcollections/RawCollectionsExample.java]

- 自定义 TypeAdapter 用于解析Collection.class(*可能会错误的解析其他集合*)

- 自定义 TypeAdapter 用于解析 Collection< MyCollectionMemberType >

## Gson 序列化/反序列化(泛型类)

由于 java 类型擦除,同集合反序列化需要使用 TypeToken.

```java
class Foo<T> {
  T value;
}
Gson gson = new Gson();
Foo<Bar> foo = new Foo<Bar>();
gson.toJson(foo); // May not serialize foo.value correctly

gson.fromJson(json, foo.getClass()); // Fails to deserialize foo.value as Bar
```

```java
Type fooType = new TypeToken<Foo<Bar>>() {}.getType();//匿名本地内部类
gson.toJson(foo, fooType);

gson.fromJson(json, fooType);
```

## Gson 自定义序列化/反序列化

- JsonDeserializer/JsonSerializer

序列化时返回 JsonElement,返回序列化时直接解析 JsonElement 生成指定的对象类型.

  支持以下具体的不同类型数据的序列化,反序列化方式:

- 不带泛型参数的类

- 带有泛型参数的类,每一种泛型单独指定序列化反序列化方法

- 带有泛型参数的类,但是不需要对每一个单独泛型的序列化,反序列化方法

- TypeAdapter/TypeAdapterFactory

  - TypeAdapter

    反序列化是 TypeAdapter#read 从 JsonReader 直接转换成为 Bean.
    序列化是 TypeAdapter#write 从 Bean 写入 JsonWriter .

  - TypeAdapterFactory

- InstanceCreator

  反序列化时创建实例的方法,通常是不需要特殊定义 Bean 的创建方式,因为 Gson 对于没有定义 InstanceCreator 的类型默认采用默认的无参构造方法进行对象的创建.(*但是对于没有无参构造方法的类,内部类则需要通过定义 InstanceCreator 进行创建*)

## Compat/Pretty 输出 Json String

配置 prettyPrinting ,控制 Bean 转换成为 Json String 时是压缩输出还是带有空格的美观输出.美观输出每一个层级缩进为两个空格字符.

## null 支持

- 默认不输出 null,可以通过配置 serializeNulls 用于序列化时输出 null

- 数组/集合 中的 null 会被输出(*如果不输出,则序列 index 对应的值的语义就被改变了*)

## 序列化版本支持(@Since/@Until)

- @Since/@Until
  
  从某个版本开始,到某个版本结束.要配合设置 Gson 对象的版本号使用.

  使用 Since/Until 注解时没有添加注解的字段始终会被序列化/反序列化.注解目前可以添加在 Field,Type(Class) 字段上用语标记 Field,Class 的版本号.
  
  添加 Since Version < CurrentVersion 有效
  添加 Until Version > CurrentVersion 有效

## 序列化/反序列化 Exclude 支持

- 通过 Modifier 修饰符
  
  默认不被序列化/反序列化 的访问修饰符为 Static,Transient,可以通过 GsonBuilder#excludeFieldsWithModifiers 指定exclude 的修饰符号类型.

  可选的修饰符号有:PUBLIC,PRIVATE,PROTECTED,STATIC,FINAL,SYNCHRONIZED,VOLATILE,TRANSIENT,NATIVE,INTERFACE,ABSTRACT,STRICT 等.具体参见 java.lang.reflect.Modifier.

- @Expose 注解
  
通过该注解配合 GsonBuilder#excludeFieldsWithoutExposeAnnotation 可以实现指定字段剔除,不进行序列化和反序列化操作.

通过 new Gson 直接创建的 Gson 对象不支持 @Expose 注解.(无论怎么标记均按照 修饰符规则进行序列化与反序列化)

- 自定义 ExclussionStrategy

该接口包含两个方法 shouldSkipClass,shouldSkipField.同时使用该接口配置 Gson 时又分为 GsonBuilder#addDeserializationExclusionStrategy 和 GsonBuilder#addSerializationExclusionStrategy 两种配置方式.前者用于配置反序列化的 exclude 策略后者用于配置序列化的 exclude 策略.Gson 内部的判断策略:
  
- 判断待序列化/反序列化的类是否整个都不需要解析

- 判断该类的内部字段类型,如果该类型不需要序列化/反序列化则直接跳过该字段.如果该类型需要进行解析,则判断该字段的具体属性(如:名称,注解等)是否要跳过.

## 序列化/反序列化名称转换策略

### 内置策略(FieldNamingPolicy)

由于在Java 中 Bean 属性的命名规范为 lowerCamelCase,因此该处的命名策略只提供由lowerCamelCase 向其他命名方式的转换.

- FieldNamingPolicy#IDENTITY
  
  名称不转换

- FieldNamingPolicy#UPPER_CAMEL_CASE
  
   someFieldName ---> SomeFieldName

- FieldNamingPolicy#UPPER_CAMEL_CASE_WITH_SPACES
  
  someFieldName ---> Some Field Name

- FieldNamingPolicy#LOWER_CASE_WITH_UNDERSCORES
  
  someFieldName ---> some_field_name
  aStringField ---> a_string_field
  aURL ---> a_u_r_l

- FieldNamingPolicy#LOWER_CASE_WITH_DASHES
  
  someFieldName ---> some-field-name
  aStringField ---> a-string-field
  aURL ---> a-u-r-l

- FieldNamingPolicy#LOWER_CASE_WITH_DOTS
  
  someFieldName ---> some.field.name
  aStringField ---> a.string.field
  aURL ---> a.u.r.l

### 基于 FieldNamingStrategy 自定义的命名策略

自定义名称转换策略,上面的 FieldNamingPolicy 也是基于该接口实现.(*GsonBuilder#setFieldNamingPolicy与GsonBuilder#setFieldNamingStrategy两种策略会相互替换,他们是互斥的关系*)

### 基于注解的策略(@SerializedName)

直接在字段上添加该注解,用于标记该字段在序列化/反序列化中的名称.*标记注解的字段名称优先级高于上面两种命名策略的优先级*

## 注解总结

- @Expose

配合 GsonBuilder#excludeFieldsWithoutExposeAnnotation 构建配置标记某个字段是否需要被序列化和反序列化.(*如方法名称如果某个字段没有被标记注解则直接忽略*)

- @JsonAdapter

用于标记指定的类/字段使用指定的类型适配器.
该注解的值可以是:TypeAdapter/TypeAdapterFactory/JsonSerializer/JsonDeserializer 或者其子类.(*不推荐一个类同时实现上述多个接口,如果实现上述多个接口优先级从高到低,该特性由源码分析得到*)

- @SerializedName

用于标记指定字段序列化和反序列化时对应的名称.

- @Since/@Until

用于标记指定字段的序列化和反序列化时支持的版本号.(*注解的值是 Double 类型,不符合语义化版本号标准*)
[语义化版本][https://semver.org/lang/zh-CN/]

## 高级使用技巧

### Gson#getDelegateAdapter

主要代理用于已有的 TypeAdapterFactory生成 TypeAdapter,对其添加诸如统计,增强,审计相关的附加功能.具体使用方法参见 Gson 该方法注解.
*与 Gson#getAdapter存在明显区别,Gson#getAdapter 获得的 TypeAdapter 是被包装过的 TypeAdapter 且无法跳过指定的 TypeAdapter*
