# gradle 入门基础

## 为什么要用gradle

gradle: spring,hibernate,okhttp
maven: netty,apache 自家项目

## gradle构建的基础元素

### SourceSet（输入）

### Dependency（外部依赖）

### Repository（外部依赖仓库，外部依赖来源）

### Artifact（产品,输出）

## gradle 构建文件组成

### setting.gradle/setting.gradle.kts（gradle构建的项目组成）

### build.gradle/build.gradle.kts（gradle 构建项目配置）

### 编写构建脚本的语言

- Groovy
  
- Kotlin

## gralde 执行的生命周期

### init

### configuration

### execution

## gradle 常用命令行

- gradle init
  
- gradle dependecies

## gradle 构建组建的基础核心概念

### Setting

### Gradle

### Project

### Task/Action

- task 中的Action执行顺序，跳过执行： StopActionException，StopExecutionException 分别终止 Action 的执行和 Task 的执行。

- task 的依赖关系，执行顺序： dependsOn,shouldRunAfter,mustRunAfter,FinalizedBy

- task 的增量构建，执行缓存。

### Configuration/ConfigurationContainer

### Convention/ExtensionContainer

COC (Convention Over Configuration),Project,Task 等继承自 ExtensionAware 都可以单独存储获取 Convention/ExtensionContainer

### Plugin

自定义的项目构建插件，可以打包生成 jar 提供给第三方使用。

## 可复用的构建配置

### buildSrc 项目

### Plugin 插件项目
