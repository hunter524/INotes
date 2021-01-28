# 使用备忘

## Url编码/转义规则

- auth 编码规则
- path 编码规则
- query 编码规则

## Retrofit 配置

- validateEagerly

false: 惰性加载,对应的 API 方法调用时才执行反射解析方法注解,生成 ServiceMethod 方法缓存

true: 即时加载,在 Rtrofit#create API 动态代理对象是,即解析 API 中的所有方法,生成 ServiceMethod 缓存

- callbackExecutor

配合 Platform#defaultCallAdapterFactory,当 API 方法要求返回 retrofit2.Call 使用 retrofit2.Call#enqueue 执行该请求时,retrofit2#Callback 回调执行的 Executor.

如果配置配置了 callbackExecutor 则使用 ExecutorCallAdapterFactory/ExecutorCallbackCall 封装 Call 的回调.

没有配置则使用 DefaultCallAdapterFactory 直接返回原始的 retrofit.Call

    - Android 平台:使用 Call 执行默认的 callbackExecutor 为 MainThreadExecutor Call 回调在主线程.返回的适配 Retrofit.Call 的 CallAdapter.Factory 为 ExecutorCallAdapterFactory.

    - Java 平台:回调默认在 OKhttp3 的请求线程池的线程执行.*retrofit 2.6.0 添加 SkipCallbackExecutor注解,用于跳过 callbackExecutor 回调的执行*

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
  
  head 请求响应不存在 ResponseBody 因此返回值类型需要标记为 Call< Void >,Call< Response< Void > >

- @DELETE
- @OPTIONS
  
上述定义的四种 Http 请求方法不能有 Body.

- @POST
- @PUT
- @PATCH

上述定义的三种 Http 请求方法可以正常有 Body

- @HTTP

用于自定义 Http 的请求方法,路径,是否有 Body 也需要自己定义.

## 注解分类

### 方法注解

- 请求方法注解
  
  DELETE
  GET
  HEAD
  PATCH
  POST
  PUT
  OPTIONS
  HTTP
  
  上述注解表示七种请求方法类型和一种自定义请求方法的方式.同时上面的 8 种请求方法也不能重复定义在一个方法上,因为一个 Http   请求只能有一个请求方法类型.
  
  上述请求方法的注解 value 值,表示路径不能携带 ?{query}=value 或者 ?key={value} 的动态查询参数,动态查询参数只能使用  @Query,@QueryMap,@QueryName 动态添加查询参数.
  
  但是 value 值可以使用 https://host/{path1}/{path2} 结合 @Path 对路径进行动态参数修改.

- Headers

方法注解标记请求头,value 值为一个 String[] , Http Header 格式如:Cache-Control: max-age=640000,Key 的值不同 value 的值也不同.

- Multipart

方法注解标记是 Multipart 模式的 Http 请求.参数部分需要添加 @Part 注解用于标记每一部分的参数.或者是通过 PartMap 标记一个 Key 为 String,值为 Part 的 Map.请求标记该注解时要求请求方法必须是可以包涵　Body 的请求如：Post,Put,Patch. 不能是　Get,Delete,Head,Options.如果是　HTTP 自定义的请求方法也需要包涵　Body 部分．

- FormUrlEncoded

方法注解标记请求参数 Body 部分需要执行 url encode 编码. 由于 multipart 请求与 from-url-encode 请求互斥因此不能同时标记在一个方法中.

该注解需要结合 @Field 用于发送 form-url 请求.

Multipart,FormUrlEncoded 标记的方法,请求方法必须是 @Post,@Patch,@Put 或者携带参数的 @Http 请求方法

- SkipCallbackExecutor
  
  retrofit 2.6.0 添加该注解,在请求方法 Api 要求的返回值是 Call< Bean > 时,用于标记当前 Call 的 Callback 跳过 Retrofit 配置的 CallBackExecutor 机制.(即该请求的 Callback 回调不切换执行线程,直接在 Ohttp3 的执行线程执行)

### 参数注解

同一个参数上,由于 Retrofit 的实现限制,无法在同一个参数上添加多个由Retrofit 定义的注解.

- Url
  
  参数的 @Url 注解不能和请求方法(@Post,@Put等)的 value 值共存.

  @Url 注解不能重复标记两次

  @Url　注解不能与　请求方法注解的　ｖalue 值共存．（即不能重复两次标识路径）

  @Path 注解不能和 @Url 注解同用 (你都完全自定义 @Url 了为啥还要 @Path???)

  @Query 注解要在 @Url 之后,同 @Path 查询参数的替换也依赖于 url 路径(包括　QueryName,QueryMap,)

  该注解标记的参数只能是　HttpUrl,String,URI,android.net.Uri 类型，不能是其他类型．

- Path

  Query 注解要在 Path 注解 之后

  Path 注解不能和 @Url 同用．（即　＠Path 注解替换的　Path 路径值，只能为请求方法标记中的的ｖalue 值）

  使用 Path 注解是一定需要有一个相对路径 Url.(先有路径再有相对的 Url)

  Path 注解标记需要提供　ｖalue 值用于替换　path 路径中　{name} 标记的路径的占位符号．

- Query
  
  Query 注解的参数可以是 Iterable,Array.如果是 Iterable,Array 则值会以相同的 key 加入 Request 中.(如:文档形成 /friends?group=coworker&group=bowling 的查询参数)

  Query 只是在　Url 后面添加查询参数，Query 既可以添加在　Get请求之上，也可以添加在　Post 请求上，还可以添加在其他任何　Http 请求之上．（QueryMap,QueryName 注解与该注解基本相同)

- QueryName

  向 url 后面添加查询参数,但是只添加 key 值不再添加 value 值.使用 @Query 传递空字符串会形成.<https://www.wanandroid.com/user/register?username=&password=&repassword=>,使用 QueryName 则不会添加多余的 = 号.*= 号的添加规则是在 HttpUrl 的库中解析的*

- QueryMap
  
  注解的参数类型 rawType 必须是 Map 及其子类.Map 的参数化类型的第一个类型必须是 String 类型.第二个类型可以是任意类型.*其javadoc 说是使用 String.valueof 但是其实使用的是 BuiltInConverters#ToStringConverter 对 value 值进行转换,本质上使用的是Object#toString 进行到字符串的转换,在基础数据和基础数据的数组类型的转换上,这两种转换成为字符串的模式可能存在差别*

- Header
  
  Header 注解为参数注解,标记参数的 value 值为 Header 的 Key 值,参数值为 key 对应的Header 的 value 值.
  对应一个 Headers 注解,是批量添加 Header 参数的,该 Headers 注解是添加在方法上的.

- HeaderMap

  与 QueryMap 对应,该处是为了可以动态添加为 Map 的 Header 值.注解标记的参数值必须是Map,且　Map 的 Key　类型需要为　String,Map 的　ｖalue 类型可以为任意值，其会被通过　StringConverter 转换成为字符串．

- Field

　参数添加了该注解则方法必须添加　@FormUrlEncoded 标记是　form 请求．
  
  为 form 请求需要添加的参数的 key 和 参数作为 value 值.

- FieldMap
  
  同 QueryMap 功能对应,该处是为了可以动态添加 Map 作为 form 值.同上参数必须是　Map 类型，Map 的　key 值必须是　String 类型，ｖalue 值可以为任意类型，通过　StringConverter 抓换成为String 类型．

- Part
  
  与方法的 MultiPart 注解需要同时使用,MultiPart 标记在请求方法上,标记当前方法的Http 请求方法为 MultiPart.

  *Part注解如果 value 是空则被注解的参数需要是MultipartBody#Part 类型,或者组件类型为 MultipartBody#Part 的　数组　或者　Iterable*

  *Part注解如果 value 不为空,Part 注解的参数则会通过 RequestBodyConverter 对 Part 注解的参数进行进行转换，同时结合　ｖalue 值构建　multipar 请求的每一个　part 部分*

  *Part注解如果 value 不为空,但是注解的参数类型为 MultipartBody#Part 则会抛出错误(此时　Part 中的　Header 会与　ｖalue 中的　Header 相互冲突*

  *如上 Part 注解无论 value 是否为空参数均可以为 Iterable,List 类型,如果 value 不为空则会以相同的注解配置,添加多个Part部分*

  Part 注解的　value 为空则注解标记的参数类型必须为　Part 或则其数组列表，Part 注解的　ｖalue 不为空则　ｖalue 至必须不能为　Part,但是可以为其他任何类型，通过　RequestBodyConverter 转换成为　RequestBody 结合　Headers 组合成为　Part.

- PartMap

  同 QueryMap,key 值的类型必须为 String,用于作为 MultiPart 请求部分每个 Part 部分的名称,名称主要构建在每个　Part　部分的　Header中,value 值可以为任意类型,会被通过 RequestBodyConverter 转换成为 MultiPart 部分的每个 Part 中的 RequestBody 部分.*value 可以为任意类型,但是 MultipartBody.Part 类型除外(因为一个 Part 部分既包括了请求的 Header部分,Header 存在一个该部分的 Key值,也包含了 Body 部分),如果需要是 Part类型,则可以使用注解 @Part List< Part > 类型进行处理*

- Body
  
  不能与 FormUrlEncoded　或　MultiPart 注解同时使用,也不能一个请求方法中标记了两个 @Body 注解.FormUrlEncoded 有自己的格式的Body不需要用户提供body,MultiPart 会由多个 Part 构建形成真实的 Body.

  同时对于不需要 Body 的请求方法(如:Delete,Get,Head,Option 均不需要Body,Patch,Post,Put 则需要 Body),也不能添加 @Body 注解标记的参数.(*ServiceMethod#Builder#build 时会检查相关请求配置的注解和参数是否符合 Http 请求的要求,2.6.2 以后的版本会在　RequestFactory 中添加　Http 请求的合法性检查*)

- Tag

  retrofit 2.6.0 添加的特性,用于在用于注解参数,以参数类型作为 Key,实例值作为 value 添加到 okhttp3.Request 对象上.对于 List< Bean > 类型的 tag 则使用的是原始类型 List 作为 key 添加到 okhttp3.Request 的对象上（即对于　Tag 的　key 类型取原始类型，而不是泛型类型）.一个请求方法可以标记有多个 Tag 但是key值不能相同,即 Class 不能相同.

  搭配　retrofit2.Invocation Class 可以在　OkHttp 中追踪到这个特定的请求是否是由　Retrofit 发送出去的，以及放松该请求的方法，以及传递进入的参数．

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
*响应标记 Call< Bean > 时实际得到的 CallBack 中的类型为 Response< Bean >*

- java.util.Optional 支持

  retrofit 2.5.0 之后添加内置的 OptionalConverterFactory 用于对于返回值要求是 Call< Optional < Bean > > 格式的返回值的支持

- retrofit2.Response/okhttp3.Response
  
  retrofit2 在 ServiceMethod#build 时则检查了 Call< R > 或者 Observable < R > 其中 R 的类型不能是Response 的简单类型(泛型的 rawType),可以是 Response< Bean > 需要带参数化类型
  *但是声明 Call< Bean > postBlog(@Body Bean bean) 方法时通过 Call#enqueue,Call#execute 返回的响应类型为 Response< Bean > 类型*  

  - 响应可以标记为 retrofit,Response< T > 类型,但是不能标记为 okhttp3.Response 类型,因为在 retrofit.OkHttpCall 的源码实现层面已经不提供 okhttp3.Response,只提供了 ResponseBody 给 Converter 进行类型转换.

  - 返回值要求是 retrofit2.Response 则必须具化泛型参数.无法要求返回值是 okhtt3.Response,因为 retrofit 不提供.(retrofit2 的木的便是屏蔽 okhttp3 的实现细节)

## Converters

负责用于将任意数据转换为 okhttp3.RequestBody, 同时将 okhttp3.ResponseBody 转换成为用户想要的数据.

在 retrofit2 的子项目 retrofit-converters 中内置了 gson,guava,jackson,moshi,jaxb,protobuf,simlexml,wire 等默认的数据请求与响应转换器.

存在的问题与解决方案:

- 如 jackson,gson 转换器并没有识别方法的注解,直接对 ResponseBody 读取执行 Json 转换.(会导致在同一个 retrofit 添加的多个转换器如果 gson,jackson 的转换器添加在前面会导致后面的转换器失效)

- 因此转换器的添加需要注意优先级(*添加顺序*),如需要支持相应数据支持 Optional 同时支持 json 则需要先添加 guava 转换器,再添加 jackson,gson 转换器.

- 使用者也可以添加自己的 Converter 转换器,用于通过如识别不同自定义方法注解返回不同的转换器的功能.

## CallAdapters

用于将 retrofit2.Call (retrofit2.OkhttpCall) 单回调模式包装成其他模式.如 rxjava 的响应模式,guava ListenableFuture 多回调模式,且每个回调可以在用户指定的携程进行执行.

在 retrofit 的子项目 retrofit-adapters 中内置了 guava(ListenableFuture) java8 (CompletableFuture *目前retrofit 已经内置支持该类型,该子项目基本废弃*) rxjava(Observable,Single,Completable) rxjava2(Observable,Flowable,Single,Maybe,Completable) rxjava3(与 rxjava2基本相同) scala(scala 自己的 Future).额外的目前 retrofit2 目前内置了对 kotlin 携程的支持(支持使用 kotlin 写 suspend fun 方法),同时也通过内置的 DefaultCallAdapterFactory 添加了对回调切换线程操作的支持.
*对比Converter的实现则该处的实现更加优雅,其根据方法不同的返回类型判断 当前 CallAdapter.Factory 判断是否要返回 Converter*
