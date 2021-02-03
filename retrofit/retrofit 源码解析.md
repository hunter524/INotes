# 源码解析

## Retrofit 流程

- 通过 Retrofit#Builder 构建 Retrofit:
  
  - 提供 okhttp3.Call.Factory
  
  - 提供 BaseUrl

  - 添加 CallAdapter#Factory/Converter#Factory

  - 添加 CallbackExecutor/配置构建 Api 接口对象是即时加载还是懒加载

- 通过 Retrofit 对象构建,Api 请求接口的动态代理对象

- 通过构建的动态代理对象调用 Api 接口,返回 Api 接口指定的返回值.如:Call< Bean >,Observable < Bean >,Call < Optional < Bean > > 等.该处负责解析 Api 接口指定的方法,生成 ServiceMethod 并且缓存 ServiceMethod,生成 ServiceMethod 的步骤如下:

  - 解析方法的 参数注解/方法注解 获取 RequestFactory/验证方法的返回参数泛型类型是否有未确定的类型,如:泛型变量,泛型通配符则为未确定类型.

  - 解析方法的返回值获取 CallAdapter,解析方法返回值的 Response 类型,获取 ResponseConverter,分别用于 retrofit2.OkHttpCall 向 Observable 等其他类型的转换,以及 ResponseBody 向 Response 的转换.

- 通过返回的 Call,Observable 对象执行真正的请求.无论通过 CallAdapter 怎么包装,最终执行请求的逻辑的均是 retrofit2.OkHttpCall.
  
  - retrofit2.OkHttpCall,也只是包转了一层 okhttp3,通过 retrofit2.ReequestFactory 构建 okhttp3.Request,通过先前配置的 Retrofit 的 okhttp3.Call.Factory 构建 okhttp3 的 Call,通过 OkHttp3 执行最终的 http 请求.

## 动态代理

## 反射类型系统

java 的反射类型系统,初步入门时通常只会提到 Class,如 String.class 可以用 Class表示,那么 List< E > 带有泛型的类如何表示?显然 Class 只能表示不带有泛型的类,带有泛型的类则只能表示出不带泛型的模式,无法表达出带有泛型的模式.

### 类型抽象

#### Class

不带有泛型系统的原始类型(retrofit 称之为 rawType),如:String.class 的表示则为 Class 类型,ArrayList < Bean > 通过实例对象获取到的类型也为Class 类型则为原始类型

#### GenericArrayType

方法声明的参数,返回值是泛型数组则返回该类型.Method#getGenericParameterTypes, Method#getGenericReturnType

#### ParameterizedType

泛型父类已经被参数化的类型,通常是一个子类实现了泛型父类,指定了父类泛型参数的类型.通过子类 Class#getGenericSuperclass 和 Class#getGenericInterfaces 可以获得该类的实例.

方法中声明的返回值,如果带有泛型类.则通过 Method#getGenericReturnType 也会获得该类型的一个实例.

方法中声明的参数,如果带有泛型类型.则通过 Method#getGenericParameterTypes 也可以该类型的一个实例.

#### TypeVariable< D >

ParameterizedType 中声明的泛型参数,(*ParameterizedType 虽然是已经参数化的类型,但是其传递的参数可能依旧是泛型类型,如 T,R 等没有被具化的类型*),也可以是方法,类中直接声明的泛型变量 T,R 等未被具化的泛型变量.

#### WildcardType

使用 \* 通配符号定义泛型类型时才会该类型的实例.只有使用 \* 时才可以使用 super 关键字.(*java.reflect.WildcardType抽象中的方法定义可以看出,只有其具有 WildcardType#getLowerBounds 用于对应 super关键字*)

使用super关键字指定通配符类型时:

- lowerBounds 为 super 指定的类型
- upperBounds 为 Object 类型
- 如果使用 extends 指定通配符类型则只存在,upperBounds 不存在 lowerBounds.
- 如果只用 ? 声明泛型通配符,则 UpperBounds 默认为 Object 不存在 lowerBounds.

### 其他辅助类型抽象

#### Member

- AccessibleObject
  
- AnnotatedElement

#### Parameter

#### Field

#### Executable

- Constructor

- Method

#### GenericDeclaration

可以声明泛型参数的元素继承自该接口,如 Class,Method,Constructor.

### 泛型声明的限制

只有实现了 java.lang.reflect.GenericDeclaration 接口的子类才可以声明泛型,目前实现该接口的类只有 Class,Method,Constructor.即只有类上,方法上,构造方法上才可以声明泛型.

T,R 泛型可以声明在 Method,Class 上,反射时得到的泛型类型是 TypeVariable,调用方法时可以指定泛型类型(方法返回值的强制向下转型的过程).构建指定类型的子类时指定具化的泛型类型(实际上同方法调用并无本质区别).*具体的转型过程可以查看泛型类及其子类生成的字节码(子类指定泛型参数类型,覆写对应父类方法时会生成桥接代码(字节码方法)用于进行类型的转换)*

- 类声明的泛型参数,如果子类覆盖了父类的泛型方法,则会生成对应的桥接方法.如果没有则调用父类的泛型方法时则只在调用处执行类型转换.
  
- 方法声明泛型参数,则在调用处插入 checkcast 字节码,用于将方法返回值进行类型转换,转换成为声明的待赋值的变量的类型.(checkCast 则检查类型是否可以转换,如不可以转换则抛出 ClassCastException 异常,该处是虚拟机执行的操作)

声明在 Method,Class 上的泛型变量可以使用 T,R 等具名的泛型名称,只能使用 extends 限定上界,具有多个上界时使用 < T extends Runnable & Callable > 的写法.

? 只可以使用在具体化泛型类,类型时,? 声明的泛型反射得到的类型是 WildcardType,只有该类型的泛型声明才可以使用 extends super 指定类型的上下界.(*因为? 是匿名的泛型参数声明,因此无法*)

## Platform

## Retrofit/Retrofit#Builder

- Retrofit#Builder#build/Retrofit#newBuilder

  实现 Retrofit 和 Retrofit#Builder 之间的相互转换

## Converter.Factory/CallAdapter.Factory/Converter/CallAdapter

### Converter.Factory

- Factory#requestBodyConverter
  
  - 可以获取到请求 API 方法 @Body 注解标记的参数类型 Type,所有参数标记的注解*但是无法获取其他具体的参数*,方法上面的所有注解,以及 Retrofit 实例.*理论上可以通过方法注解返回不同的 Converter 进行请求参数向 RequestBody 的转换*

- Factory#responseBodyConverter

- 只能获取到方法的注解*无法获取方法参数的注解*,还可以获取需要返回的参数的类型,以及 Retrofit.*响应转换器也可以通过相应注解返回不同的转换器,用于对不同的请求进行响应解码*

### CallAdapter.Factory

Factory 在解析 Method#getGenericReturnType 已经将外成的 ParameterizedType 进行了一次解析,如果 Call< Optional < Bean > >,在 Factory 中剥离了 Call 类型,用于匹配如何对Call 进行转换,如:转换成为 Observable,CompleteableFuture 等.

Factory 根据不同的 Call 或者 Observable 类型用于匹配不同的 CallAdapter,对 Call 进行转换.同时将内层的泛型参数作为 ResponseType,提供给 Converter.Factory 用于匹配对应的 ResponseConverter 用于对 ResponseBody 进行转换.如该处的 Optional < Bean > 类型则需要通过 OptionalConverterFactory 进行两次 Response 转换,第一次转换 ResponseBody -> Bean ,第二次将 Bean 包转成为 Optional < Bean >

### 内置 Converter/BuiltInConverters

- VoidResponseBodyConverter

  方法返回值为 void 则直接返回 null,并且对 ResponseBody 执行 close 操作,避免 in,out 流的内存泄漏.

- StreamingResponseBodyConverter

  返回参数是 ResponseBody 且标记 @Streaming 则返回原始的 ResponseBody 供使用者直接读取

- BufferingResponseBodyConverter
  
  返回参数是 ResponseBody 且未标记 @Streaming 注解时对原始请求的 ResponseBody 进行 buffer 之后返回

- ToStringConverter

  将请求 Body 直接使用 toString 方法转换成为字符串.
  
- RequestBodyConverter
  
  如果请求@Body 参数是 okhttp3.RequestBody 则直接不转换

### 内置 CallAdapter

- ExecutorCallAdapterFactory

- DefaultCallAdapterFactory

## ParameterHandler

## retrofit2.Call/okhttp3.Call

### retrofit2.Call

在 Retrofit 中的实现为 OkHttpCall 和 ExecutorCallbackCall.

- OkHttpCall

  retrofit2 中的核心实现类,用于代理 okhttp3.Call 的请求执行和回调执行.

  OkHttpCall 在要被获取 Request,enqueue 和 execute 需要执行时,通过 OkHttpCall#createRawCall(ServiceMethod#toRequest 和 ServiceMethod#CallFactory ) 分别创建 Request 和 okhttp3.Call 进入 retrofit.OkHttpCall. okhttp3.Call被 retrofit.OkHttpCall 代理.

- OkHttpCall#parseResponse

  retrofit 解析 okhttp3.Response 的行为在该处,并且在该处通过 ServiceMethod#toResponse 与 Response 的 Converter 相耦合解析 ResponseBody 为用户定义的 Bean 类型.

- ExecutorCallbackCall
  
  如果设置了 Retrofit#callbackExecutor 再次代理 OkHttpCall 的回调执行.*该实现类的被代理类为 OkHttpCall 目标是代理 OkHttpCall 的回调进行切换线程执行*

- retrofit2.Call< T >/retrofit.Callback< Response < T > >

  Api 接口如果声明的返回值类型为 Call< Bean > 则其通过 execute/enqueue 执行得到的结果是 Response< Bean >.Todo:// 该处的泛型是如何传递的.

### okhttp3.Call

该接口的实现类在 okhttp3 中只有 okhttp3.RealCall ,其中 AsyncCall 并非真实的实现该接口的 Call.AsyncCall 只是作为 RealCall 的一个内部类包装了 RealCall 的异步执行逻辑,本质上 AsyncCall 是一个 Runnable 用于提交到 okhttp3 内部的线程池中执行.

## retrofit.Callback/okhttp3.Callback

如同上面的 retrofit.Call 与 okhttp3.Call 的代理与被代理的关系,retrofit 使用自己的 Callback 提供给调用者使用,retrofit 真正执行时则依旧依赖于 okhttp3.Call 与 okhttp3.Callback 通过代理模式分别暴露了 retrofit.Call 和 retrofit.Callback 给使用者使用.

- Call< T > , Callback< T >,Response< T >,

 在 java 的泛型世界,泛型参数在泛型类中并不存在具体类型,如 ArrayList 的泛型类在存储元素时是均是存储为 Object 类型(将存储对象的类型向上转型),在获取数据时再根据泛型类型向下转型.

 Call < T > 的泛型类型由用户定义的请求 Api 的方法定义,所以实际上只是在调用方法处执行了一次类型转换.(*retrofit 内部其实并不关注 Api 接口方法中的声明的泛型类型,因为其实现类 CallAdapted 在构建时并没有指定泛型的具体类型*)

 因此 retrofit2.Call 的实现类 retrofit.OkHttpCall 声明泛型参数只是为了从源码层面制定实现类的泛型规则.

## ServiceMethod/ServiceMethod#Builder

- ServiceMethod#Builder#build

  解析方法的注解,参数的注解,判断是否符合 HTTP 请求的规则,同时通过 Retrofit 和 注解,方法返回值类型(Call?Observable?Single?guava ListenableFuture? java CompletableFuture? 等等,retrofit 均提供了相应的 CallAdapter 的实现),@Body 注解的参数类型,方法返回值的参数化类型(gson,protobuf 等数据类型) 分别获取合适的 CallAdapter,RequestConverter,ResponseConverter,

  *CallAdapter/ResponseConverter 都只能获取到方法的注解,RequestBody 则可以额外获取到@Body 标记的单个参数字段所标记的所有注解(无法获取到其他参数上标记的注解)*

  依次获取 CallAdapter

  获取 ResponseConverter

  解析方法注解

  判断请求方法与 MutliPart/FormUrlEncode 是否匹配

  解析参数注解(因为 Body 也在参数中,因此 Body 的转换操作也在该处进行),构建与参数注解想对应的 ParameterHandler 用于在 OkhttpCall 调用 ServiceMethod#toRequest 构建 okhttp3.Request

  build 结束,检查所有注解参数是否符合 Http 请求规范.
  *请求方法注解 value 标记的 relativeUrl 和 Url注解标记的参数必须要有一个*
  *FormUrlEncoded 注解标记的方法请求,必要要有 Field 注解标记的参数*
  *MultiPart 注解标记的方法,必须要有 Part 注解标记的参数*

- ServiceMethod#parseMethodAnnotation

### ServiceMethod/HttpServiceMethod

retrofit 2.5.0 版本之后 ServiceMethod 被抽象成为基类,其实现类只有 HttpServiceMethod ,通过 ServiceMethod#invoke 这一层抽象层,实现对后期除 OkHttpCall 之外的其他请求的调用.(从 2018/06/16 抽象出该类,但是至今没有除 HttpServiceMethod 的其他实现)

### HttpServiceMethod#adapt

HttpServiceMethod#adapt 用于在执行 ServiceMethod#invoke 时对于 OkHttpCall 的 Call 的转换,主要是为了支持 kotlin 的携程,挂起方法的请求模式.主要实现类有如下三种:

- CallAdapted
  
  非 suspend 携程版本,应用老版本定义的 callAdapter 对 OkhttpCall 进行转换,返回 rxjava1,rxjava2,guava ListenableFuture 等返回对象.

- SuspendForResponse
  
  返回的为 Response 版本.(*该处的 Body 也通过 ResponseBodyConverter 对原始的Body 进行了转换,但是会将这个 Response 通过 Continuation 回调到 suspend fun 继续执行,suspend fun 在 通过 kotlinc 生成字节码时会在方法参数最后添加一个参数类型为 Continuation 的参数, retrofit 则通过该参数进行 Continuation方法的回调*)

- SuspendForBody

  返回的是 Response 的 Body部分.(返回到该处的 Response 的 Body 部分其实是已经)

### RequestFactory

retrofit 2.5.0 的发布日期是 2018/11/19

retrofit 2.5.0 版本之后,将 ServiceMethod 中对于 Method 方法注解,参数注解的解析构建 okhttp3.Request 的功能抽象进入了该类进行,减轻了 HttpServiceMethod 职责范围.

retrofit 2.5.0 同时也在 RequestFactory 添加了对 kotlin croutine 支持*本质上是对 Api 中声明的接口方法,添加了 suspend fun ,kotlinc 编译器会对　suspend fun 方法添加一个额外的参数　Continuation 用以实现携程的编程模式*.

- RequestFactory#create

  被　OkHttpCall 用以创建　okhttp2.Request 对象．

## RequestBuilder

被 ServiceMethod 和 ParameterHandler 用以解析注解参数构建 okhttp3.Request 对象.

在　2.6.2　之后的版本中则是被　RequestFactory　进行配置，用于构建　okhttp3.Request.
retrofit 中的　RequestBuilder　只是一个桥接类，用于桥接　RequestFactory　与　okhttp3.Request.Builder 最终　okhttp3.Request 的构建还是交由　okhttp3.Request.Builder　进行构建．

## 一些辅助类

### ExceptionCatchingRequestBody

    只是为了包装真实的 ResponseBody#read 操作,捕获 read 操作中潜在的可能抛出的异常,便于将这个异常在 OkHttpCall#parseResponse 读取时继续向外进行抛出.

### retrofit.Invocation

为 retrofit 2018/09/21 添加的新特性,通过 Invocation 类向 okhttp3.Request 类添加 tag,用于 Okhttp3 在拦截器中获得当前通过 retrofit 封装的请求的接口的方法的抽象和调用参数.*可以在拦截器中统计每个 API 接口,每个方法的调用次数,请求响应时间,mock 特定接口方法的返回值等切面功能*

### adapter-rxjava2

- BodyObservable

  取出了回调过程中的 Response< Bean > 中的 body 即 Bean 向下调用传递给其他使用者.也可以获取意义不大的 Response< Bean > 则 Api 需要声明为 Observable< Response< Bean > > postBlog(@Body Bean bean)

## 一些问题

### java 已经提供了 URL 和 URI 为什么 Okhttp 要自己创造一个 HttpUrl ?

Answer: [https://square.github.io/okhttp/3.x/okhttp/] 参见 HttpUrl doc.

### Retrofit 对于定义在 Object 中的方法采用 Method#invoke 进行调用,但是对于 Default 方法采用的是 Lookup 类和 MethodHandler 的方式进行调用,为什对于不同的方法要区别对待?

### retrofit 2.6.1 之前的版本不支持定义 Api 的接口继承其他接口,2.6.1 及以后支持继承其他接口,但是接口不能定义泛型参数为什么?



