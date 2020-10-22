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

依赖种类：

- implementation
  
  依赖不向外泄漏，优先考虑使用该种依赖方式。(*在 JavaLibraryPlugin 中使 implementation 依赖继承了 api 依赖*)

- api

  依赖直接向外暴露的依赖。(*在 JavaLibraryPlugin 中使 api 依赖继承了 compile 依赖*) api 的配置是在 JavaLibraryPlugin 中添加的。

- compile(Deprecated)

  老的不区分 api,implementation 的依赖方式。现在已经被废弃。

- compileOnly/provided(deprecated 等同于 compileOnly)
  
  只用于编译时的依赖满足和检查。不会出现在生成的zip,tar,distribution 等安装包目录中。程序需要正常运行通常需要用户额外提供与 compileOnly 的依赖具有相同的 ABI 的其他实现。

- runtimeOnly/runtime(deprecated 等同于 runtimeOnly)
  
  运行时的依赖，不参与编译的依赖检查。

- compileClassPath(内部聚合依赖)

  聚合了 compile compileOnly implementation api 三种依赖方式。代表编译时需要提供的 class 的路径。

- runtimeClassPath (内部聚合依赖)

  聚合了 compile implementation api runtime runtimeOnly。代表运行时所需要提供的类的路径。

- apiElements (内部定义的依赖)

  用于定义向依赖者暴露的当前项目内部的元素，用于编译时的依赖检查（源码的正常引用跳转)。继承自 api 依赖，同时也提供当前项目的源码依赖.

- runtimeElements（内部定义的依赖）

  用于定义向依赖者暴露的当前项目内部的元素，暴露的这些元素只用于项目的运行时提供。继承自 implementation,runtimeOnly,runtime.*default 默认配置继承自 runtimeElements,其他项目依赖 Project 默认依赖的是 default 的 Configuration.但是该处的 mplementation,runtimeOnly,runtime 配置的 Visible 属性均为 false 因此无法对外部的其他项目暴露其内部实现*

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

### sourceSets/SourceSet

默认的 SourceSet 为 main 和 test 分别放置项目的主代码与测试代码，用户可以通过 sourceSets 属性配置自己的 SourceSet。不同的 SourceSet 的依赖于编译任务是相互隔离。默认的 jar 任务只输出 main 下的 class 文件打包生成 jar文件。

SourceSet 向上承接了与 Configuration 的依赖，向下定义了不同 compile,jar,classes 任务与 SourceSet 的依赖。SourceSet 内部分外 java,allJava,resource,allResource 不同的 SourceDirectorySet 可以分别独立包含不同的目录进入相同 SourceSet。（该处需要注意 srcDir/setSrcDirs 的区别）

打包其他名称的 SourceSet:

```java
tasks.register<Jar>("jarJust"){
    this.from(sourceSets.getByName("just").output)
}
```

将其他名称的 SourceSet 添加进入默认的jar 任务：

```java
tasks.getByName("jar"){
    (this as Jar).from(sourceSets.getByName("just").output)
}
```

fatJar 将当前main的运行时依赖也打包进入同一个 jar:

```java
tasks.register<Jar>("fatJar") {

    manifest {
        attributes("Main-Class" to "com.github.hunter524.forlove.AppKt")
    }

    archiveClassifier.set("fat")

    from(sourceSets.main.get().output)
    dependsOn(configurations.runtimeClasspath)
    from({
        configurations.runtimeClasspath.get().filter { it.name.endsWith("jar") }.map { zipTree(it) }
    })
}
```

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

##### 基础配置选项和名称

- base
  
  由 BasePlugin 提供的，对外暴露的实现类为 BasePluginConvention，提供基础的 dists,libs,archivebaseName 的配置。

- java/sourcesets

  均由 JavaBasePlugin 提供。对外暴露的实现分别为 JavaPluginExtension,SourceSetContainer 前者用于配置 source/target 的 Compatibility.后者用于获得和创建 SourceSet,并且对获得的 SourceSet 进行配置。对于 SourceSet 的配置主要是获取 SourceSet 的 output 输出，对SourceSet#java(SourceDirectorySet) 添加，设置，过滤 java 源码目录用于编译。

  插件默认创建的 SourceSet 为 main 和 test.

- application

  由 ApplicationPlugin 提供。对外暴露的实现为 JavaApplication。用于配置 run 任务需要的 mainClass,java 命令的指定目录,java 运行该 main Class 携带的参数（main 方法入参），该应用的名称 。

- manifest 配置

  上述 java 的 JavaPluginConvention 也提供了 manifest 方法，使用该方法是只会创建 manifest 对象供后续使用。

  如果需要将 manifest 写入 jar 包中则需要使用 Jar 任务对象中的 manifest 方法。*声明的manifest会存储在 jar 的 META-INF/MANIFEST.MF 文件中*

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

  主要提供 compileJava（JavaCompile）,processResources（Copy),classes(聚合任务,依赖于前面两个任务),compileTestJava（JavaCompile）,processTestResources(Copy),testClasses,jar（Jar 任务依赖于 classes 任务主要用于输出 jar 文件）javadoc(JavaDoc,依赖于 classes 任务，生成javadoc 文档)，test(Test,依赖 testClasses 任务，通过Junit,TestNG 执行单测试)，uploadArchives（Upload，上传 archives 配置的 Artifact 进入指定的 Repository),clean(Delete 任务，删除build 目录下的文件)，*cleanTaskName(删除指定task名称的输出文件，如 cleanJar,则是删除 jar 任务的输出文件 jar包)

  对于一个Project 有不同的 SourceSet，可以分别使用 compileSourceSetJava，processSourceSetResources，SourceSetClasses 用于分别编译资源文件，java 文件或者一起编译资源文件java文件。(不同 SourceSet 的命名规则，除 main 之外，采用 动词:compile,process sourceSet 名称，任务处理文件类型的方式进行命名 )

  通过 java 扩展名称添加了 JavaPluginConvention ，通过 sourceSets 扩展名称添加了 SourceSetContainer 配置该插件可以配置的属性，如：添加SourceSet,修改 SourceCompatibility 和 TargetCompatibility 等。

- ReportingBasePlugin

  为 Test,scan 等性能监控任务，测试任务提供基础的目录配置功能。该基础插件目前只向 Project 添加了名称为 reporting 类型为 ReportingExtension 的扩展。

- BasePlugin
  
  该插件是为 java,maven 仓库上传插件所提供的基础插件任务。提供一些上传maven仓库，构建特定依赖的组件，归档项目产出的任务。

  添加创建 buildTaskName 名称的 Task 规则（即向 Project 的 TaskContainer 添加 BuildConfigurationRule 规则,用于构建指定的 Configuration)。uploadxxx 对应 UploadRule 规则(用于使用 Upload 类型的 Task 上传指定的 Configuration)。

  提供了原始的名称为 uploadArchives 类型为 Upload 的上传任务。(*目前该Task 只提供上传 ivy 仓库的功能，上传 maven 仓库的功能由 MavenPlugin/MavenPublishPlugin 插件替代*)

  配置 AbstractArchiveTask 及其子 Task 主要是 Jar,Tar,War,Zip,Ear 类型任务的输出目录，Version,输出文件的 BaseName，Jar 任务的输出目录默认为 /build/libs 其他任务的输出目录默认为 /build/dist.

  该插件还会默认创建：archives，default 名称的 Configuration。以及名称为 defaultArtifacts 的DefaultArtifactPublicationSet 将其放置在 Project#extensions 中。（*该容器主要容纳Configuration(不包含名称为 archives 的配置) 中的 PublishArtifact,因为该 DefaultArtifactPublicationSet 是依赖 archives 名称的Configuration 中的 PublishArtifactSet 所创建的，因此本质上是将所有Configuration 中的 PublishArtifact 集成进入同一个 PublishArtifactSet 中*)。

  配置 LifecycleBasePlugin 创建的 assemble 任务，使其依赖于 名称为 archives 的 Configuration 的 PublishArtifactSet 依赖的 Task.

  通过 base 扩展名称添加了 BasePluginConvention。使用该插件可以配置如： archiveBaseName,dists 目录名称，libs 目录名称。

- LifecycleBasePlugin

  该插件并不是 java 项目所特有的功能插件，而是为所有项目共同提供的构建生命周期任务。如：clean(删除 Project 的 build 目录),cleanTaskName(删除指定task的output 输出),assemble（聚合其他任务，该任务不执行具体功能）,check（聚合任务，不执行具体功能）,build（也是聚合任务，但是聚合了 assemble 和 check 任务)。

##### 扩展Java Plugin

- JavaLibraryPlugin

  向下依赖与基础的 JavaPlugin. api 类型的依赖的 Configuration 是在该处进行添加的。
  使用该插件后，编译的依赖就会变成 build 目录下生成的 class 文件而不是 jar 文件，因此会增加编译时的内存消耗，尤其是在windows 平台上由于文件句柄的限制，尤其会降低编译时的性能，因此官方在介绍该插件时推荐在 windows 平台设置 org.gradle.java.compile-classpath-packaging 属性为 true 从而降低这种编译时的性能影响。
  
- JavaLibraryDistributionPlugin
  
- JavaPlatformPlugin
  
  gradle 5.2 添加的新特性，用于生成 Maven 的 BOM（Bill of Material) 文件，或者是 Gradle platforms 文件（Gradle Metadata）。多个不同的 java library 项目，可能会构成一个java 开发平台，在同一个java 开发平台中，这些项目之间的版本有着一系列的协调和约束，因此便通过该插件完成平台的声明和版本约束。

  Boms 项目：com.fasterxml.jackson:jackson-bom,org.springframework.boot:spring-boot-dependencies

  提供 api 和 runtime 两种依赖模式，通过 DependencyHandler#constraint 中的 DependencyConstraintHandler 声明依赖的约束模式。通常不可以在 java-platform 中产生和依赖二进制文件（jar,也不能同时依赖 java-platform 和 java 或者 java-library 插件。) 如果需要依赖二进制文件则需要通过 javaPlatform#allowDependencies 显示的声明可以依赖二进制文件。

- ApplicationPlugin

  提供了 JavaApplication ,给使用者配置 mainclass,应用名称，应用运行时的 jvm 参数。

- DistributionPlugin

  在 ApplicationPlugin 应用中会应用该插件，该插件提供 run（直接运行当前程序中设置的 Main Class),installDist(在 build/install 目录下 bin 中添加windowns/linux运行脚本，lib 中添加 jar,当前Project 文件生成的jar 以及依赖的 jar),distZip(将上述 installDist 生成的文件打包成zip,放置在 build/distributions 默认目录中),distTar(将上述 installDist 生成的文件打包成tar,放置在 build/distributions 默认目录中),assembleDist(聚合任务，同时执行 distTar,distZip 任务) 等任务。

  提供了 DistributionContainer 给使用者进行配置 Distribution。

- Kotlin 语言编译插件

由 Kotlin 语言开发者 JetBrains 自己编写的 [gradle 插件][https://plugins.gradle.org/plugin/org.jetbrains.kotlin.jvm] 提供kotlin 语言的支持。但是 gradle 自己编写了 scala,groovy 语言的编译插件。

#### 组件发布插件

- PublishingPlugin
  
- MavenPublishPlugin

- IvyPublishPlugin

#### 插件项目插件

－ JavaGradlePluginPlugin

 依赖于　JavaPlugin 通常在　buildSrc 项目或者插件项目中应用．其默认引入　gradleApi 引用（便于插件的编写）．并且提供了名称为　gradlePlugin　类型为　GradlePluginDevelopmentExtension　的配置文件．该配置便于了生成xxxx.properties 并且内置　implementation-class 为其实现类.(如当前插件声明文件：org.gradle.java-gradle-plugin.properties，内容为：implementation-class=org.gradle.plugin.devel.plugins.JavaGradlePluginPlugin　指明当前插件ｉd和实现类路径，放置于　resoureces/META-INF/gradle-plugins/　目录下)

## 从 maven 向 gralde 转换

[官方指导文档][https://docs.gradle.org/current/userguide/migrating_from_maven.html]

## 可复用的构建配置

### buildSrc 项目

### Plugin 插件项目

## 构建的核心

### Configuration

表示一组产品和产品的依赖。api,implementation,runtimeOnly 配置的依赖则是添加到与之对应名称的Configuration#getDependencies 中。其中也通过 Configuration#getAllArtifacts 配置一组当前 Configuration 的产出产品。

在 BasePlugin 中除了上述依赖类型的 Configuration 还会默认创建 archives,default 这两个 Configuration。

### Upload(Task)

只负责上传任务的配置。（配置上传到哪几个 maven,哪几个 ivy 仓库。需要上传哪个 configuration 中的内容）。真实的上传任务交由 ArtifactPublisher 去进行。其再通过识别不同的仓库再交由不同的仓库类型实现的 ModuleVersionPublisher 进行最终的上传任务。

通过 RepositoryHandler 创建的每一个 maven,ivy 仓库均具有上传组件的功能。该处配置 repo 和 Project#repositories 是存在区别的.
Project#repositories 只用于下载操作,该处配置的 repositories 则只用于上传操作.

默认的 archives Configuration 聚合了其他的 Configuration 中的 Artifact 组件功能.

### JavaCompile/CompileOption/JavaToolChain/Compiler

对 javac 命令的抽象，用于执行 javac 任务，并且配置执行 javac 任务时需要携带的选项参数。javac 命令的执行模式又分为:直接 fork 又称为 Daemo 模式，java home 模式，当前进程直接启动进程执行。JDK 模式,在当前执行进程调用 tools.jar 中的JavacTool 代码执行编译过程.该种模式是 gradle 的默认编译模式,因为该种模式下不需要新建进程,性能损耗最小(*由于fork是计算机基础的程序复制工具,复制出来的程序与原程序配置相同,因此fork java 程序时会执行严格的参数检查,同时符合要求才进行fork操作,如果不符合要求则新建程序(新建 Daemo 进程)*)

JavaToolChain 则是对于当前构建编译工具的行为的抽象。

JvmVersionDetector：通过当前提供的 java 命令 或者 javahome目录,javadoc,java 等二进制执行，tools.jar 目录（JavaInfo）去探测当前的java版本，或者指定的 java 版本。

Compiler:为对应执行编译功能的抽象,在 JavaCompile 的编译执行过程中主要涉及 AnnotationProcessorDiscoveringCompiler->NormalizingJavaCompiler->(CommandLineJavaCompiler,DaemonJavaCompiler,JdkJavaCompiler) 逐层嵌套,分别执行 APT class 文件发现,规范化数据打印,最终交由底层jdk 编译工具进行编译操作.

默认情况下执行的是 JdkJavaCompiler 使用默认的jdk提供的编译工具进行编译,在当前进程直接执行编译过程. (小于 java 9 com.sun.tools.javac.api.JavacTool,大于等于 java9 使用 javax.tools.ToolProvider#getSystemJavaCompiler 作为编译工具*这些工具是 sun 公司编写的代码位于 jdk 目录的 tools.jar 包中,javac命令(二进制文件)只是封装了调用该类执行编译的过程,同时也观察到 kotlinc 命令(其实只是一个 shell 命令调用 java 编写的编译程序执行编译)*).

当 CompileOptions#fork 为 ture 时如果配置了 ForkOptions#getExecutable 或者 ForkOptions#getJavaHome 时则使用 CommandLineJavaCompiler(通过指定的 javac 路径在当前进程直接执行 javac 命令) 作为编译器,如果没有配置则使用 DaemonJavaCompiler(在 daemon 进程执行javac编译命令) 作为编译器.
TIPS:配置 fork为true同时配置javaHome在 gradle 执行 java 编译过程时可以通过 jps -l 命令查看当前的java进程,可以观察到: 21749 com.sun.tools.javac.Main 进程,因此可以确定 javac 的编译代码其实是 java 程序.因此分析 com.sun.tools.javac.Main类的源码更有利于加深对 APT 编译过程的理解.该编译器的代码处于 tools.jar 安装包中.

外层再通过 IncrementalResultStoringCompiler -> CleaningJavaCompiler 包装上述创建的基础 Compiler 分别实现增量编译,编译文件清理等工作和功能.

CommandLineJavaCompiler:
DaemonJavaCompiler:
JdkJavaCompiler:

### ExecHandleFactory/ExecHandleBuilder

构建过程中需要执行大量的命令行.如:javac,java,gcc 等命令.因此该处对 java 原始的命令行执行机制进行了封装(ProcessBuilder/Process).*java -version 命令输出的版本号信息是输出 在 err 信息中,不像 git --version 输出的信息是在 out 中*
