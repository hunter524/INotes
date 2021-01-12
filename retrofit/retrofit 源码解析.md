# 源码解析

## 动态代理

## 反射类型系统

## Platform

## Retrofit/Retrofit#Builder

## Converter.Factory/CallAdapter.Factory/Converter/CallAdapter

## Converter.Factory

- Factory#requestBodyConverter
  
  - 可以获取到请求 API 方法 @Body 注解标记的参数类型 Type,所有参数标记的注解*但是无法获取其他具体的参数*,方法上面的所有注解,以及 Retrofit 实例.*理论上可以通过方法注解返回不同的 Converter 进行请求参数向 RequestBody 的转换*

- Factory#responseBodyConverter

- 只能获取到方法的注解*无法获取方法参数的注解*,还可以获取需要返回的参数的类型,以及 Retrofit.*响应转换器也可以通过相应注解返回不同的转换器,用于对不同的请求进行响应解码*

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

## retrofit.Callback/okhttp3.Callback

如同上面的 retrofit.Call 与 okhttp3.Call 的代理与被代理的关系,retrofit 使用自己的 Callback 提供给调用者使用,retrofit 真正执行时则依旧依赖于 okhttp3.Call 与 okhttp3.Callback 通过代理模式分别暴露了 retrofit.Call 和 retrofit.Callback 给使用者使用.

- Call< T > , Callback< T >,Response< T >,
  
  TODO://源码层面的泛型(类型规则)传递

## ServiceMethod

- ServiceMethod#build

  依次获取 CallAdapter

  获取 ResponseConverter

  解析方法注解

  判断请求方法与 MutliPart/FormUrlEncode 是否匹配

  解析参数注解(因为 Body 也在参数中,因此 Body 的转换操作也在该处进行)

- ServiceMethod#parseMethodAnnotation

## 一些辅助类

### ExceptionCatchingRequestBody

    只是为了包装真实的 ResponseBody#read 操作,捕获 read 操作中潜在的可能抛出的异常,便于将这个异常在 OkHttpCall#parseResponse 读取时继续向外进行抛出.

### adapter-rxjava2

- BodyObservable

  取出了回调过程中的 Response< Bean > 中的 body 即 Bean 向下调用传递给其他使用者.也可以获取意义不大的 Response< Bean > 则 Api 需要声明为 Observable< Response< Bean > > postBlog(@Body Bean bean)

## 一些问题

### java 已经提供了 URL 和 URI 为什么 Okhttp 要自己创造一个 HttpUrl ?

Answer: [https://square.github.io/okhttp/3.x/okhttp/] 参见 HttpUrl doc.
