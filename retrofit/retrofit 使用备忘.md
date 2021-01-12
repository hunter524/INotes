# 使用备忘

## Retrofit 配置

- validateEagerly

false: 惰性加载,对应的 API 方法调用时才执行反射解析方法注解,生成 ServiceMethod 方法缓存

true: 即时加载,在 Rtrofit#create API 动态代理对象是,即解析 API 中的所有方法,生成 ServiceMethod 缓存

- callbackExecutor

配合 Platform#defaultCallAdapterFactory,当 API 方法要求返回 retrofit2.Call 使用 retrofit2.Call#enqueue 执行该请求时,retrofit2#Callback 回调执行的 Executor.

如果配置配置了 callbackExecutor 则使用 ExecutorCallAdapterFactory/ExecutorCallbackCall 封装 Call 的回调.

没有配置则使用 DefaultCallAdapterFactory 直接返回原始的 retrofit.Call

Android 平台下如果使用 Call 执行默认的 callbackExecutor 为 MainThreadExecutor Call 回调在主线程.返回的适配 Retrofit.Call 的 CallAdapter.Factory 为 ExecutorCallAdapterFactory.

## Retrofit#baseUrl 与 @Url 的配置

配置和解析方式参见okhttp.HttpUrl类对于 Url 的解析方式.下述的 EndPoint 为 @GET,@Post 等请求方法 或者 @Url 注解标记的 value 值.

```java

// 推荐这种方式 baseUrl 使用 / 结尾

Correct:
Base URL: http://example.com/api/
Endpoint: foo/bar/
Result: http://example.com/api/foo/bar/

// 不推荐 baseUrl 不使用 / 结尾 则相对路径忽略最后一个 path
Incorrect:
Base URL: http://example.com/api
Endpoint: foo/bar/
Result: http://example.com/foo/bar/
```

```java
//相对路径使用 / 开头则从host 根路径下添加该路径

Base URL: http://example.com/api/
Endpoint: /foo/bar/
Result: http://example.com/foo/bar/

Base URL: http://example.com/
Endpoint: /foo/bar/
Result: http://example.com/foo/bar/
```

```java
//相对路径使用 scheme host 结尾则会替换相对路径中对应的字段(scheme host) 生成新的请求 Url

Base URL: http://example.com/
Endpoint: https://github.com/square/retrofit/
Result: https://github.com/square/retrofit/

Base URL: http://example.com
Endpoint: //github.com/square/retrofit/
Result: http://github.com/square/retrofit/ (note the scheme stays 'http')
```

## Http 常用请求方法

- @GET
  
  注解 value 值可以传递绝对 url 与 相对 url.

- @HEAD
- @DELETE
- @OPTIONS
  
上述定义的四种 Http 请求方法不能有 Body.

- @POST
- @PUT
- @PATCH

上述定义的三种 Http 请求方法可以正常有 Body

- @HTTP

用于自定义 Http 的请求方法,路径,是否有 Body 也需要自己定义.

## @Path,@Query,@QueryName,@Field,@Url

- @Path

- @Field
- @Url

## GET 请求的配置

- @Query
- @QueryName
- @QueryMap

## @Multipart/@Part/@PartMap

## @FormUrlEncoded/@Multipart

- 该两个注解不能同时存在,即一个请求不能既是 form 请求又是 multipart 请求.

- 该两个请求同时需要请求方法可以放置 RequestBody

## 响应要求为 okhttp3.ResponseBody 与注解 @Streaming

响应要求为 okhttp3.ResponseBody 时,缓存与否通过 @Streaming 注解进行区分,源码的实现层面在 retrofit2.BuiltInConverters 中.分别通过 StreamingResponseBodyConverter 和 BufferingResponseBodyConverter 进行转换,ResponseBody 的 io 读取缓存操作则依赖于 retrofit2.Utils#buffer.

- 标记 @Streaming 注解返回的 ResponseBody 则是 okHttp 原始的 ResponseBody

- 未标记 @Streaming 返回的则是已经被读取过一次/缓存了读取数据的 ResponseBody 缓存版本

## 接口方法api 返回

- Call< Void >
Api 接口方法不能返回 void 但是可以使用 Call< Void > 等标记我不需要关注响应,对应的Retrofit 则会使用 VoidResponseBodyConverter 直接返回一个 null 给调用者.

- retrofit2.Response/okhttp3.Response

  retrofit2 在 ServiceMethod#build 时则检查了 Call< R > 或者 Observable < R > 其中 R 的类型不能是Response 的简单类型,可以是 Response< Bean > 需要带参数化类型
  *但是声明 Call< Bean > postBlog(@Body Bean bean) 方法时通过 Call#enqueue,Call#execute 返回的响应类型为 Response< Bean > 类型*  
