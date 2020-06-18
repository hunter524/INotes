# 常用java库汇总

## Android 平台

## JAVA 平台

- guava

  google 的 java 工具库

- Gson/fastjson/jackson

  Json 序列化反序列化

- slf4j

  日志框架门面，只提供定义不提供实现。提供了日志抽象层的协议[slf4j][http://www.slf4j.org/] 为Android 平台也适配了一版。[slf4j-adnroid][http://www.slf4j.org/android/]。
  常用日志实现库：
  java.logging:   java.util.loggin 包下的相关类，为java 日志官方库。库名简称 JUL(java.util.logging)
  com.orhanobut.logger:  android 第三方提供的日志记录库。该库提供 Json,xml 格式化打印，方法调用层级打印。但是与 slf4j 定义的规范不兼容，且需要每次手动传入 Tag 。
  android.util.Log: 该库实现日志单行打印没有提供更加详细的可选参数和打印内容格式化。
  log4j: apache 的对于日志框架的实现
  logback:log4j 的升级版本，也提供了 [Android 实现][http://logback.qos.ch/android/]
  

- dagger2/dagger hilt

  APT 依赖注入框架。为了解决 dagger2 在 Android 平台的不适配与过于复杂，因此 android 平台开发了 [dagger hilt 框架][https://dagger.dev/]。

- javax.mail:javax.mail-api:1.6.0/com.sun.mail:javax.mail:1.6.0
  
  java 平台用于发送邮件

- FreeMarker,Velocity 模板引擎
  
  用于按照模板生成代码.
  