# DartOverview

从不一样的角度看 Dart

## 变量如何声明

- java
  
  静态类型,强类型.新版本支持 var 类型推导

- kotlin
  
  静态类型,强类型.区分 var val.依赖于类型推导,可以不声明变量类型

- dart
  
  var,dynamic,类型注解声明变量

- js
  
  var,let 声明变量,变量提升

## 方法如何被重载(名称相同的方法如何被识别)

- Java(参数的类型和个数,????? 好像记得不区分返回值类型???)

- Kotlin 基本同 java 相同,但是添加了命名参数,默认参数,避免了 java 多个参数重载的复杂度

- dart,添加了可选参数与命名参数特性,但是无法实现参数个数,类型不同的方法重载模式

- js 则可以随便传递多少个方法,既可以同 java 一样实现重载方法(??? 不确定是否真的可以,回头试一下???),也可以通过 arguments 参数获取未声明形式参数,但是实际传递了的参数.

## 程序如何被组合

### JAVA

- 包名
- JAR
- ClassLoader 隔离

### Kotlin

基本同 java.

### Dart

library,part of(part),export show(hide)

### JS

- 不同文件
- 全局变量污染(导入变量不再放在全局)
- node(export,import)

## chain call(链式调用如何实现)

- java
  
  类似于 Builder 模式,方法返回 this 自己.

- kotlin
  
  let,run,apply 扩展方法封装

- dart
  
  内置 ..(双点) 级联调用操作符号

- js

  同 java.依赖于方法返回自己
