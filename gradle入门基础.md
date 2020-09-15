# gradle 入门基础

## 为什么要用gradle

gradle: spring,hibernate,okhttp
maven: netty,apache 自家项目

## gradle构建的基础元素

### SourceSet（输入）

不同的 SourceSet 目录可以配置不同的 Source 依赖，该特性有利于在同一个项目中将不同的 SourceSet 依赖进行隔离(sourceSetImplementation,sourceSetApi)。

### Dependency（外部依赖）

依赖是通过 Project#dependencies 闭包，通过 DependencyHandler 配置进入 Configuration中的。不同类型的依赖（api,implementation 等)

其中 compileOnly,implementation 依赖会聚合成为 compileClasspath 配置，提供给 compileJava 任务使用。

不同的 SourceSet 可以配置不同的依赖（在创建新的 SourceSet 时，gradle 也会随之为其创建不同的 Configuration 用来为其进行不同的依赖配置)

implementation,runtimeOnly,runtime 会聚合成为 runtimeClasspath,gradle 的默认 jar 任务只会打包当前项目的代码进入jar,而不会将依赖的项目打包进入jar,但是如果使用 assmbleDist,installDist,distTar,distZip 任务，运行时的依赖jar会和项目生成的jar 共同加入生成的zip文件中,并且生成运行脚本，提供给用户直接通过脚本运行jar程序。如果需要将依赖的jar文件通过jar任务打包进入同一个jar文件包中则需要配置 fatJar 任务，即将 runtimeClasspath 配置中依赖的jar包解压，提供给 Jar 任务重新压缩进入新生成jar包中。

依赖种类
依赖解析策略：依赖替换，平台依赖，依赖版本，依赖限制
依赖缓存
依赖策略：编译时依赖（compileOnly)，运行时依赖(runtimeOnly),编译时和运行时共同依赖（implementation,api)

### Repository（外部依赖仓库，外部依赖来源）

仓库类型：maven,ivy,本地文件。

### Artifact（产品,输出）

### Publish(输出目的地和输出方式)

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
  
- task 延迟创建 create -> register,getByName -> named (gradle 4.9 以上才可以使用，提升 Configuration 时的性能)

### Configuration/ConfigurationContainer

### Convention/ExtensionContainer

COC (Convention Over Configuration),Project,Task 等继承自 ExtensionAware 都可以单独存储获取 Convention/ExtensionContainer

### Plugin

自定义的项目构建插件，可以打包生成 jar 提供给第三方使用。

#### Project 应用插件

- apply plugin:"plugin_name"（通过 Project#apply 方法添加插件），this.plugins.apply（通过 Project 引用 PluginContainer 再通过该 Container 添加插件）
  
- plugins{} (kotlin 构建脚本可以通过该方式添加插件，且添加的插件名称可以为BuiltinPluginIdExtensions 扩展的简写名称，按行添加即可。plugins 使用的是Kotlin 的 DSL 特性，该特性基于基础的扩展函数和plugins 方法传递函数进行，按行添加插件使用的是 扩展扩展属性 get 方法结合扩展方法)
  groovy 脚本也可以使用 plugins 闭包，但是需要在

#### Java项目中常用的插件

plugin的id名称和kotlin脚本中的简写名称参见 gradle_manual.md 的 *常见Plugin* 章节

##### 基础Java Plugin

- JavaPlugin

  [参考文档][https://docs.gradle.org/5.6.4/userguide/java_plugin.html]

  Jar 任务默认只打包 main SourceSet 生成的 class 文件，如果自定义了其他的 SourceSet 则需要自定义 jar 任务，定义该 jar 任务的输入为该 SourceSet 的输出（即为编译生成的class 文件和 resource 文件）。当然也可以找到现有的 Jar 任务，然后包含自定义的 SourceSet 的输出。

  在参考文档中的关于增量编译的描述，也提供了一些应该有的良好的代码设计和文档编写。

- JavaBasePlugin

  JavaPlugin 插件的生命周期任务则是通过该插件提供的。主要提供了以下Task:

  assemble:聚合任务，依赖于 jar 任务，打包所有 artifact 在 archive 的配置中。
  check:聚合任务，依赖于各种 test ,进行代码的单元测试和校验。
  build:聚合任务，依赖于 check 和 assemble 任务，进行项目的完整构建。
  buildNeeded:构建和测试当前项目以及所依赖的项目
  buildDependents:构建和测试当前项目以及依赖当前项目的项目
  buildConfigName:构建指定 configuration 中配置的 artifact 产品
  uploadConfigName:构建并上传指定 Configuration 中配置的 Artifact 产品。

  主要提供 compileJava（JavaCompile）,processResources（Copy),classes(聚合任务,依赖于前面连个任务),compileTestJav（JavaCompile）,processTestResources(Copy),testClasses,jar（Jar 任务依赖于 classes 任务主要用于输出 jar 文件）javadoc(JavaDoc,依赖于 classes 任务，生成javadoc 文档)，test(Test,依赖 testClasses 任务，通过Junit,TestNG 执行单测试)，uploadArchives（Upload，上传 archives 配置的 Artifact 进入指定的 Repository),clean(Delete 任务，删除build 目录下的文件)，*cleanTaskName(删除指定task名称的输出文件，如 cleanJar,则是删除 jar 任务的输出文件 jar包)

  对于一个Project 有不同的 SourceSet，可以分别使用 compileSourceSetJava，processSourceSetResources，SourceSetClasses 用于分别编译资源文件，java 文件或者一起编译资源文件java文件。(不同 SourceSet 的命名规则，除 main 之外，采用 动词:compile,process sourceSet 名称，任务处理文件类型的方式进行命名 )

  通过 java 扩展名称添加了 JavaPluginConvention ，通过 sourceSets 扩展名称添加了 SourceSetContainer 配置该插件可以配置的属性，如：添加SourceSet,修改 SourceCompatibility 和 TargetCompatibility 等。

- ReportingBasePlugin

  为 Test,scan 等性能监控任务，测试任务提供基础的目录配置功能。该基础插件目前只向 Project 添加了名称为 reporting 类型为 ReportingExtension 的扩展。

- BasePlugin
  
  该插件是为 java,maven 仓库上传插件所提供的基础插件任务。提供一些上传maven仓库，构建特定依赖的组件，归档项目产出的任务。

  添加创建 buildTaskName 名称的 Task 规则（即向 Project 的 TaskContainer 添加 BuildConfigurationRule 规则,用于构建指定的 Configuration)。uploadxxx 对应 UploadRule 规则(用于使用 Upload 类型的 Task 上传指定的 Configuration)。

  提供了原始的名称为 uploadArchives 类型为 Upload 的上传任务。(*目前该Task 只提供上传 ivy 仓库的功能，上传 maven 仓库的功能由 MavenPlugin 插件替代*)

  配置 AbstractArchiveTask 及其子 Task 主要是 Jar,Tar,War,Zip,Ear 类型任务的输出目录，Version,输出文件的 BaseName，Jar 任务的输出目录默认为 /build/libs 其他任务的输出目录默认为 /build/dist.

  该插件还会默认创建：archives，default 名称的默认的 Configuration，同时也会创建名称为 defaultArtifacts 的 DefaultArtifactPublicationSet（*该容器主要容纳Configuration 中的 PublishArtifact*)。TODO:// 分别在项目中承担什么职责。

  配置 LifecycleBasePlugin 创建的 assemble 任务，使其依赖于 名称为 archives 的 Configuration 的 PublishArtifactSet 依赖的 Task.

  通过 base 扩展名称添加了 BasePluginConvention。使用该插件可以配置如： archiveBaseName,dists 目录名称，libs 目录名称。

- LifecycleBasePlugin

  该插件并不是 java 项目所特有的功能插件，而是为所有项目共同提供的构建生命周期任务。如：clean(删除 Project 的 build 目录),cleanTaskName(删除指定task的output 输出),assemble（聚合其他任务，该任务不执行具体功能）,check（聚合任务，不执行具体功能）,build（也是聚合任务，但是聚合了 assemble 和 check 任务)。

##### 扩展Java Plugin

- JavaLibraryPlugin

  向下依赖与基础的 JavaPlugin. api 类型的依赖的 Configuration 是在该处进行添加的。
  
- JavaLibraryDistributionPlugin
  
- JavaPlatformPlugin
  
  gradle 5.2 添加的新特性，用于生成 Maven 的 BOM（Bill of Material) 文件。

- ApplicationPlugin

  提供了 JavaApplication ,给使用者配置 mainclass,应用名称，应用运行时的 jvm 参数。

- DistributionPlugin

  在 ApplicationPlugin 应用中会应用该插件，该插件提供 run（直接运行当前程序中设置的 Main Class),installDist(在 build/install 目录下 bin 中添加windowns/linux运行脚本，lib 中添加 jar,当前Project 文件生成的jar 以及依赖的 jar),distZip(将上述 installDist 生成的文件打包成zip,放置在 build/distributions 默认目录中),distTar(将上述 installDist 生成的文件打包成tar,放置在 build/distributions 默认目录中),assembleDist(聚合任务，同时执行 distTar,distZip 任务) 等任务。

  提供了 DistributionContainer 给使用者进行配置 Distribution。

#### 组件发布插件

- PublishingPlugin
  
- MavenPublishPlugin

- IvyPublishPlugin

## 从 maven 向 gralde 转换

[官方指导文档][https://docs.gradle.org/current/userguide/migrating_from_maven.html]

## 可复用的构建配置

### buildSrc 项目

### Plugin 插件项目
