# Gradle Doc/Manual

## gradle 常用命令

- gradle
  
  不带有任何命令行参数，其会触发初始化，配置阶段，然后执行配置的 Project#defaultTasks 配置的默认task.
  
- gradle wrapper
  任务可以更新 gradle/wrapper 下面的wrapper至当前gradle版本。gradle-wrapper-x.x.x.jar 为存储在当前 GRADLE_HOME/lib 目录下面，
  gradle wrapper 任务只是将其复制进入 gradle/wrapper 目录下。

- gradle init

  初始化创建 gradle 项目。

- gradle help

   查看默认的 gradle 帮助选项。gradle help --task <task_name> 查看指定 task_name 的可选配置参数选项。
   如 gradle help --task init  则为查看 init 任务可选的配置参数。会列出 --dsl --package -- project-name 等参数用于配置项目的构建。

- gradle projects

   查看当前项目的目录结构。以及其包含的子项目的层级。
  
- gradle tasks
  
  展示当前项目配置的可以用于执行的任务。
  
- gradle dependencies
  
  用于展示当前项目的maven依赖结构，其是按照项目展示依赖的结构的，在根目录下执行则展示的是根目录的依赖结构，在app目录下执行即展示的是app目录的maven依赖。
  在Android 项目中其通常用于分析依赖关系和解决依赖冲突。

  gradle dependecies --configuration <configure_name> cfg_name 可以为 api,compileOnly,runtimeOnly 等等。

  gradle dependencyInsight:使用 --dependency , --configuration , --singlepath 可以过滤指定依赖，指定配置以及指定但行显示。主要用于追踪指定依赖是如何被依赖上来的。为 dependencies 的反向操作，dependencies 为正向操作查看当前依赖以及级联依赖。
  
- gradle properties
  
  展示通过 gradle.properties 设置的所有属性，该属性是针对 build 脚本设置的，也可以使用 gradle 命令: gradle -Pkey=value 设置该脚本属性。与之相对应的
  则存在一个虚拟机属性，需要使用 gradle -Dkey=value

- gradle init
  初始化 Gradle 项目。

- gradle -Dkey=value/gradle -Pkey=value
   D 设置的是系统配置参数，P 设置的是项目的配置参数。
   -D 对应 gradle.properties 文件的 systemProp 前缀。
   java -Dkey=value 也可以用于设置 JVM 系统属性。

  -Dorg.gradle.debug=true 该属性可以用于调试 gradle daemon 构建进程,DefaultDaemonStarter 类中有识别该参数。（该进程负责 gradle 项目构建任务的执行，并且可以在特定条件下复用，用于构建任务的执行，其入口函数为 GradleDaemon#main。但是无法调试 gradle 命令启动进程，如果需要调试 gradle 命令启动进程需要在 gradle ，gradlew 脚本执行 gradlexxx.jar 时携带 JVM 调试参数 : -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005 。

  属性配置的优先级：命令行->systemProp(gradle.properties,可以放置在 gradle 安装目录，项目根目录，gradle_user_home目录 目录优先级从低到高排序)->gradle prop (配置在 gradle.properties 中以 org.gradle.xxx.xxx=xxxx形式的属性)->env (全大写的属性名称，如 GRADLE_OPTS,GRADLE_USER_HOME,JAVA_HOME)

  systemProp,-D 属性会同时配置在 System#Properties 和 project.extensions.extraProperties.properties 上，前者上去除 前缀，后者携带前缀。
  
  Projects属性设置:
  配置服务端机器环境变量: ORG_GRADLE_PROJECT_<prop>=<somevalue> 
  命令行：添加 -Pkey=value
  gradle.properties添加属性：org.gradle.project.<key>=<value>

  属性配置对于JVM分类：gradle 的client端参数配置(实际上并没有必要配置该端参数，该端主要用于启动任务和输出日志),gradle 的 Daemon 端参数配置(主要的构建任务执行端)。
  配置的属性分类：
  系统的属性：gradle.properties

- gradle  <task_name> -m/gradle  <task_name> --dry-run

   不执行指定任务，只输出执行该 task 需要被执行的 task 任务列表

- gradle <task_name> --scan/-- profile
  
   用于分析构建任务的耗时，依赖 等。--scan 需要借助 gradle 的平台查看更加详细。-- profile 只能查看简单的各个流程和任务的耗时，无法提供更细致的内容。

- --build-cache
  
  与gradle clean 无关的属性,即使 项目执行了 gradle clean 项目可以被缓存的 task 缓存的内容依旧可以被使用.其被存储在 gradle_user_home/cache/目录下.

- gradle buildEnvironment
  
  自动内置在 gradle 构建项目内部的任务,用于查看gradle 的构建环境.主要用于查看 gradle 构建任务执行依赖的库,而并非构建的项目依赖的库.*即 buildscript{dependencies {classpath "gourp:artifact:version"}}依赖的库*

- gradle <task_execute> --exclude-task <task_exclude>
  
  执行某个 task 及其依赖的task 跳过某个特定的task不执行.

- gradle --continue <task_name> <task_name>
  
  当前面的 Task 任务运行失败之后后面的 task 任务继续运行.

## gradle 插件的编写/引用

- 在 build.gradle 中直接定义插件(脚本中直接定义一个类继承自Plugin类)
  
- 在项目根目录建立 buildSrc 目录，其中内置 gradle 插件项目。使一个类继承自另外一个类，buildSrc 同层的项目及其子项目即可以直接使用该类引用该插件。
  
- 引用第三方插件 jar 包，再引用插件。(第三方jar,开发时作为一个单独的 java 项目编译生成 jar 包)
  在 META-INF 目录下建立 <plugin_name>.properties 文件，文件内部放置 Implementation-class =<plugin_class_path> 如 Android 项目应用构建插件: plugin_name 为 :com.android.application,
  plugin_class_path 为 com.android.build.gradle.LibraryPlugin 指向了插件的全路径名称。

- Kotlin 中通过 PluginAwareExtensions.Kt 扩展了 PluginAware 使使用者可以更方便的添加插件，应用脚本等操作。
  但是也依旧可以使用 Project#apply 相关方法，或者通过 Project#apply(Closure/Action(ObjectConfigurationAction)) 通过 ObjectConfigurationAction 对项目进行配置。
  Gradle 内置插件在 BuiltinPluginIdExtensions.kt 通过短名称进行列出。

- plugins{} 语法为 ProjectScript 中的方法。该处获得的为 PluginDependenciesSpecScope 用于配置 plugin .*即脚本先是处于 xxxScript对象中，当脚本对象中有该方法时优先调用脚本中的该方法，当脚本中不存该方法时则委托调用到 Project,Setting,Gradle对象中进行* 其中 id version 必须要求为常量字符串，不可以动态的获取。

- Settings#pluginManagement 管理插件，对所有项目进行全局的插件管理。

- 遗留的插件引用方式，apply 直接引用 gradle core 中的插件。结合 buildscript 引用第三方库的插件。

### buildSrc

buildSrc目录置于根目录作为gradle 构建脚本,插件配置的默认目录.其中可以存放共用的gradle脚本,gradle插件项目(kotlin,java项目).为当前项目内部提供共享的脚本和插件.gradle 根项目及子项目会默认依赖和编译该项目.

## Build Cache

## Composite Build

并非是多项目结构构建(一个 Gradle Project 通过 setting 文件,包含多个sub_project),组合构建的项目实际上是并无直接关联的两个项目(独立的 gradle项目) 原本通过依赖 artifacts(项目的产出)更改为直接依赖该项目的模式.

组合构建比多项目结构构建相互之间的依赖更加独立.

## Settings

对应 settings.gradle/settings.gradle.kts 构建脚本最先被执行,用于识别gradle项目的目录结构.

### Settings API

- include
  
  包含当前根目录下的子目录作为子项目。如 include("path1:path2") 则会包含 path1 ,path2 两个 Project 分别为 Project 进入项目的构建。

- includeFlat
  
  包含 Root Project 的兄弟目录作为子项目构建。(只能包含兄弟目录，不能包含兄弟目录的子目录，因为无法传递： /\ 等目录分割符作为参数)

- includeBuild
  
  以 / 作为目录分割符，相对于当前根目录进行解析。与 include 不同的是其只包含指定的目录项目，不包含层级目录中的项目。
  includeBuild 包含的 Project无法被 Project#allProject 获取和配置。includeBuild 即为组合构建
  使用 includeBuild 则使用了 [Gradle CompositeBuild][https://docs.gradle.org/5.6.4/userguide/composite_builds.html#composite_build_intro] 特性.用于组合两个独立的 Gradle 项目参与构建。

- project

  settings 中获取的 Project 为 ProjectDescriptor Project 描述符 而不是 Project 对象。
  在 settings.gradle 文件中可以通过 ProjectDescriptor 修改 Project 的名称，项目目录，项目build.gradle 构建文件的目录。

## Project

### ProjectState

用于表示 Project 的配置状态。可以用于 Gradle#afterProject 中用于判断该 Project 是否已经成功的被配置了。

### ExtensionContainer （Project#getExtensions) 扩展

Project#extra,Task#extra 获取的 Extra 配置即为 Project#ExtensionContainer#ExtraPropertiesExtension 配置。
使用 Extension 可以很好的在 build.gradle/build.gradle.kts 中实现指定 extension 的 DSL 扩展模式修改属性值。

### Convention （Project#getConvention) 惯例

该处的 Convention 接口其实是实现了 ExtensionContainer。惯例通常是通过 Plugin 插件添加进入的一些默认的习惯性的参数配置。如:通过 BasePlugin 默认为 AbstractArchiveTask （其子类为 JAR,EAR,WAR,TAR)归档任务配置的归档文件的输出目录及归档文件名称。

Convention#getPlugins: 为Plugin 配置自己的 Convention 的地方。
*从长远来看 Convention 是需要被废弃，其是历史原因遗留产物，因此不推荐使用 Convention.从源码来看 在Projet#extensions 和 Project#convention 中获取的均为 Convention 实例，但是该 Convention 实例继承了 ExtensionContainer*

Convention 通过 DefaultConvention#plugins 自己持有/索引/添加 注册的 Convention 实例

### Configuration (Project#getConfigurations)

Project#getConfigurations实际上返回的为 ConfigurationContainer其内部持有 Configuration .
项目的 api,implementation,annotationProcessor,testApi 等等依赖则是做为项目的 Configuration 配置进入项目的。对于上述依赖配置 Configuration 通过 Configuration#getDependencies（DependencySet） 持有 Dependency.

## Task

创建 Task 时可以通过 Map 指定 Task 相关的属性.属性名称如下: name,description,group,type,dependsOn,overwrite,action,constructorArgs.DefaultTaskContainer#create(Map<String,?>)用于解析Map参数创建对应的Task.

- TaskState
  
  表示 Task 的执行状态的类。Skiped,Executed,Uptodate,NoSource,Fail(执行失败)，

### Copy（文件复制）/Sync(目录同步)

继承自 AbstractCopyTask.

- 实现文件从一个目录复制进入另外一个目录，并且添加过滤匹配规则
- 通过 from 结合 Project#zipTree Project#tarTree 实现zip文件的解压复制和归档文件的解归档和复制

Sync 任务继承自 Copy 任务其与 Copy 任务不同的是 Sync 会保持 destination 目录与 source 目录相同。如果 destination 目录中有多余的文件则会被删除。Copy 任务则是增量的向 destination 目录中复制 source目录中的内容，不会去删除 destination 目录多余的文件。

### 文件移动/文件重命名

文件移动的相关操作 gradle 没有内建提供相关支持，需要使用 Gradle 集成的 Project#ant 组件对文件提供相关支持。GRADLE_HOME/lib 目录下内置了 ant-1.9.14.jar 等 ant 构建相关的 jar 包以提供对 ant 构建任务和命令的支持。gradle 文件的移动则依赖于 ant 构建工具提供的任务。

文件的重命名任务则只需要依赖于 CopySpec#rename 配合 Copy 任务即可完成。

### 文件内容的过滤/替换操作

使用 CopySpec#expand,CopySpec#filter(Token),CopySpec#filter(Closure) 方式通过模板方式替换制定 token,通过 Closure 闭包返回修改后的字符串方式。模板方式分别支持 Ant 样式模板，GString 样式模板，Groovy 的 SimpleTemplateEngine 模板引擎。

### Zip/Tar//Jar/War/Ear （文件压缩）

上述 Task 均继承自 AbstractArchiveTask ，同时 AbstractArchiveTask 又继承自 AbstractCopyTask

### Delete (文件删除)

可以使用 Delete 任务和 Project#delete 指令实现文件的删除操作。但是匹配待删除文件时无法使用像 CopySpec 这样的 include,exclude 指令进行文件的过滤和包含。需要使用 FileTree 和 FileCollection 相关的指令进行文件的过滤和筛选。

### GradleBuild

在当前项目中构建其他目录下的 Gradle 项目。*gradle 文档强烈不建议使用该 Task,因为其在某些场景下会导致意外的构建状况发生，无法保证构建的正确性* Gradle 建议采用多项目构建或者组合构建的方式，完成 Gradle Build 完成的任务。

### Task依赖管理TIPS

- dependsOn 建立 task 之间的依赖关系,并不建立task之间的执行顺序.
- shouldRunAfter,mustRunAfter,shouldRunAfter 只建立Task之间的执行顺序,但是不建立 task 之间的依赖关系.(A.shouldRunAfter(B) 执行 A ,B不会被执行.A.shouldRunAfter(B)&& A.dependsOn(B),执行A时B才会被执行)
- finalizedBy ,当前任务执行结束之后才会执行其他任务.
- 建立 Task 之间的依赖关系时指定的 Task 未必要已经被创建.(即可以先指定依赖,稍后再创建被依赖的Task,文档称其为惰性依赖)
- defaultTasks 用于配置 gradle 命令不携带任何参数时需要执行的 Task 任务
  
## 文件操作

FileTree 与 FileCollection 是 gradle 文件操作 API 的核心。java 编译中的 SouceSet 数据抽象则持有待编译文件，资源文件，编译完成的文件的 FileCollection.

FileTree 继承自 FileCollection 因此能使用 FileCollection 的地方均可以使用 FileTree 代替。

### FileTree

层级的文件集合表示，使用Project#filetree,Project#zipTree,Project#tarTree 均可以获得 FileTree 的文件描述，其中zipTree 表示 zip 压缩文件的描述，tarTree 表示归档文件的描述。fileTree 则表示普通文件目录的描述。

上述的 Project#zipTree,Project#tarTree 其实是 Gradle 帮助我们完成了 zip,tar 文件的读取和解压操作。将其转换成为了 Gradle 使用者所关心的 FileTree 文件格式。

### FileCollection

flat 的文件集合表示。使用 Project#file,Project#files 可以获得，其只表示当前引入的目录或者文件，不包含目录下的子文件，而 FileTree 包含一个目录时也会其目录下的子文件。

### CopySpec

CopySpec 可以通过 Project#copySepc 进行创建，并且独立于 Task,因此 CopySpec 可以被独立的进行共享。同时 CopySpec 也是具有层级关系，可以被层级嵌套。

CopySpec#with(CopySpec) 可以将一个Spec 叠加到另外一个 Spec 上面。

CopySpec#from,CopySpec#into 携带 Closure，Action 的均为创建子 CopySpec 其与主 CopySpec 是可以相互独立配置 include,exclude,filter,rename.但是其与主 CopySpec 又是相互协作的如 into.主 into 指定 CopySpec 的根目录，子CopySpec 指定的 into 则是相对主根目录指定的into.
子 CopySpec 同时也会继承 附 CopySpec 的 into,include,exclude,rename,filter 等配置。

使用 Project#copy API 则无需创建 Copy Task 即可使用 CopySpec 配置 Copy 操作，同时执行 Copy 操作。

## 依赖配置

依赖配置分类为 implementation,api,runtimeOnly,compileOnly,annotationProcessor，分别表示不同的依赖方式，不同的依赖均会配置进入 Configuration 中。依赖配置通常通过 Project#dependencies 提供的 DependencyHandler 进行，不建议使用者直接操纵 Configuration 进行依赖配置。配置某个依赖时可以传递进入某个闭包，即通过 ExternalModuleDependency 配置该依赖的依赖约束（如：是否进行级联依赖？exclude 剔除指定级联依赖的 group 或者 module)

### 依赖类型
  
  使用 Project#dependencies 进行依赖配置。

- moudle
  
  通过 group,name,version 依赖第三方 maven,ivy 仓库中的组件。通过 DependencyStringNotationConverter，ModuleIdentifierNotationConverter，DependencyMapNotationConverter 三个解析器解析该类型依赖的不同表述形式。

  gradle 内部使用 Dependency 的子类 ExternalModuleDependency 描述该依赖。

- file

  依赖 project/libs 或者其他目录下的 jar,aar 等模块文件。同时可以通过 Project#files,Project#file 建立被依赖的文件，以及产生该文件的task的关联。即满足该依赖必须先执行该task.通过 DependencyFilesNotationConverter 解析文件依赖描述，该依赖的描述通常为 FileCollection 及其子类如：ConfigurableFileCollection，ConfigurableFileTree 不能直接为 File 即该处可以使用 Project#files,Project#fileTree 描述该文件依赖，不能使用 Project#file 描述该依赖。

  gradle 内部使用 Dependency 的子类 FileCollectionDependency 描述该依赖。

- project

  gradle 多项目构建中描述 Project 之间的依赖。

  gradle 内部使用 Dependency 的子类 ProjectDependency 描述该依赖。

- Gradle 内置的特殊依赖

  定义在 DependencyHandler 内部，如：gradleApi 定义依赖当前的 gradle api jar 通常用于开发Plugin 和自定义任务,gradleTestKit 通常用于进行单元测试，集成测试的开发，localGroovy 依赖 gradle 内置的 Groovy 通常用于使用 groovy 语言开发 plugin ,task.

### 依赖仓库类型

- flatDir
  
  本地非 maven 仓库格式的布局文件夹中的文件依赖。类似于 libs 文件及其下面文件的直接依赖。

- mavenCentenral/jcenter/google
  
  [maven 中央仓][https://repo.maven.apache.org/maven2/]
  [jcenter 中央仓][https://jcenter.bintray.com/]
  [google 中央仓][https://maven.google.com/]
  [google maven 基地址][https://dl.google.com/dl/android/maven2/]

- mavenLocal

  用户本地的 maven 仓库。用户需要安装 maven 工具。本地仓的存储规范需要按照中央仓的目录存储规范进行文件夹布局。通常用于用户将 Project 发布到本地，再通过本地进行依赖的操作。本地仓的默认目录 USER_HOME/.m2/repository 也可以通过 settings.xml 配置文件进行本地仓库的配置。配置优先级为 USER_HOME/.m2 > M2_HOME/conf

- maven(自定义maven 仓库地址)

  惯例使用 maven 添加私有仓库，配置仓库地址。实际上也可以使用 mevenCentral,jcenter,google,ivy 设置私有仓库地址，配置私有仓库属性

- ivy

  apache ant 的子项目，主要负责 依赖管理

- 仓库依赖的配置模式

  不同的仓库（ivy,maven)，不同类型的依赖(java,js) 对于依赖的命名方式对应在仓库中的位置是不同的。
  其中maven 类型的仓库有着对应的文件在仓库中的布局规范，ivy 仓库可以定义仓库中文件的位置规范。(通过IvyArtifactRepository#patternLayout 对仓库的依赖布局进行配置,声明依赖时依旧采用orgnization:group:version:classifier@ext 的模式进行声明，对应到仓库的位置再做相应的转换,其中 classifier 表示分级，如：java 中的 jar,doc,src 表示 class 文件，doc 文件，源码文件，js 中 min 表示压缩混淆，不加表示未压缩混淆)

- MavenArtiffactRepository#mavenContent/ArtifactRepository#content

 gradle 5.1 版本新添加的特性，用于指示指定的 dependecies 只在指定的仓库可以查找到，或者用于指定指定的 dependencies 在指定的仓库无法查到。据官方文档说这样有助于提升性能，可靠性 和 安全性。

### 依赖配置特性

每一个依赖均属于一种依赖类型。
依赖类型：
before 3.0 : compile,provided,apk,
after 3.0 : compile->(implementation,api) provided->(compileOnly) apk->(runtimeOnly)

- version 限制语义

  用于限制和指定依赖版本，通过 VersionConstraint 的 strictly,require,prefer,rejects 进行依赖版本的限制。

- 依赖约束

  通过 api/implementation "group:artifact:version"{
    配置下述依赖特性
  }
  ModuleDependency#isTransitive/Configuration#isTransitive: 是否对该依赖的级联依赖进行导入,可以在配置级别进行控制，也可以在依赖级别进行控制。
  ExternalDependency#isForce：是否对该类型的依赖强制使用某个版本，而不进行自动选择
  Configuration#resolutionStrategy: 对 Configuration 中的某个依赖使用特定某个版本，该处只能控制依赖的版本而不能控制是否使用该依赖

- DependencyHandler#platform/DependencyHandler#enforcedPlatform

  使用 maven 的 bom 类型的 pom 文件对依赖版本进行限制。platform 的限制较弱，只有在依赖未声明版本号时使用 bom 中的版本号。enforcedPlatform 则限制较强，即使依赖声明了版本号也会使用 bom 中的版本号。

- DependencyHandler#components

  通过 ComponentMetadataHandler/ComponentMetadataDetails 对依赖进行配置，使用 bom 约束 或者替换依赖，约束版本号限制。

- Component Capabilities (gradle 4.7 添加该功能)
  
  避免具有相同功能的不同的依赖库被重复引用。可以通过 DependencyHandler#getComponents 获得 ComponentMetadataHandler，再通过 ComponentMetadataHandler#all 获得处理每一个依赖的 Component 的机会即 ComponentMetadataDetails 再通过 ComponentMetadataDetails#allVariants 获得 VariantMetadata 进而可以对 Compoent 中的没一个 Variant 变种获得处理机会。再通过 VariantMetadata#withCapabilities 告知 gradle 该依赖所能完成的功能。当 Gradle 感知到有两个相同的依赖完成同一个功能时即进行报错提示。

  当 Capabilities 相关的依赖出现冲突时可以使用 使用上面提到的 Configuration#ResolutionStrategy#capabilitiesResolution 去解决功能依赖上的冲突

- Project#dependencyLocking/ResolutionStrategy#activateDependencyLocking （gradle 4.8 添加的特性）

  对依赖进行锁住，避免依赖动态版本导致的依赖的更新。
  通过 gradle <task_name> --write-locks 生成lock 文件，lock 文件位于 gradle/dependency-locks/目录下。
  生成 lock 文件之后再更改依赖版本编译是无法编译通过的，因为其不符合 lock 文件的依赖约束。

  gradle <task_name> --update-locks group:artifact:version,group*:artifact*:version* 只更新制定的依赖的lock文件，可以使用 * 通配符号匹配指定的依赖。

  停用 lock 的两种方式：删除 xxxx.lockfile 文件。不调用 Project#dependencyLocking/ResolutionStrategy#activateDependcyLocking 方法，这种情况即使 xxx.lockfile 文件存在也不会启用锁。

- resolutionStrategy.cacheDynamicVersionsFor/resolutionStrategy.cacheChangingModulesFor

  分别用于配置动态版本依赖的本地版本缓存有效期和 SNAPSHOT 依赖的依赖文件的缓存有效期。 执行任务时也可以通过命令行参数控制缓存使用策略。 --offline 强制使用本地缓存，--refresh-dependencies 强制本地version缓存和snapshot 缓存无效，从服务端重新刷新缓存。

- ResolutionStragegy 的使用
  - ResolutionStragegy#eachDependency（resolve 阶段)

    build.gradle 直接声明的依赖，和依赖的级联依赖均可以被获取和重新指定解析。

    - 统一不同module 依赖的不同版本的第三方库，如 liba 依赖 gson 2.0 ,libb 依赖 gson 2.8.6,app 同时依赖  liba与libb 这时 gson 库的依赖则存在版本冲突。gradle 默认选用高版本的库。此时也可以使用  ResolutionStrategy#eachDependency 过滤匹配对 gson 的依赖，指定使用特定的版本。

    - 使用 default 标记库的依赖版本，提供统一Plugin 解析 group:name 对应的 defaul 具体版本号。主要用于通  过 Plugin 统一组织内部使用的第三方库的版本。存在的问题是版本依赖不再清晰的在 build.gradle 构建脚本中清晰  可见，需要 gradle dependencies 任务查看。或者使用 IDE 查看。
  
    - 对特定 group:name 的 特定版本的 artifact 实现黑名单机制。可能该版本的库存在不稳定性，开源协议，特殊  bug等等原因,替换该 group:name 的特定版本为其他稳定版本。主要用于公司内部的后置检查替换。*该处的  dependency 替换与 setForce 属性不同，该处的版本替换不影响其他module 依赖更高版本导致的 gralde 默认使用  更高版本的依赖选择*
  
    - 兼容性的版本库的替换。如:使用 log4j-over-slf4j 替换 log4j,使用 groovy 替换 groovy-all.这种替换的  目地通常是 jar 库依赖的精简，实现的统一，有功能实现替换成空功能实现。

  - ResolutionStrategy#dependencySubstitution

    解析完成之后用来替换特定的依赖，通常也可以完成 resolve 阶段可以完成的绝大部分的替换任务。
  
    - DependencySubstitutions#all

      遍历所有的 Dependency 执行替换规则。

    - DependencySubstitutions#module,DependencySubstitutions#project/DependencySubstitutions#substitute

      先使用 module/project 选取指定的依赖，再通过 substitute 执行替换规则。使用方法为：substitute (module) with (module)

  - ResolutionStrategy#componentSelection

    selection 只能根据 group,name,version 拒绝指定版本的使用。
    当拒绝 gradle 选择的特定版本时，gralde 会自动向下查找版本提供用户再次进行选择。如果不拒绝该版本则使用该版本作为依赖。如：build.gradle 配置依赖 com.google.code.gson:gson:2+ gradle 会自动查找到最新版本为 2.8.6 提供给用户进行 Selection 当用户拒绝该版本时，则会使用 2.8.5 版本提供给用户进行选择，用户不拒绝则使用该版本作为依赖。

  - DependencyHandler#modules/ComponentModuleMetadataHandler#module/ComponentModuleMetadataDetails#replacedBy

    两种依赖均存在时使用 replacedBy 替换 module 依赖。但是这种替换只有 module 依赖和 replacedBy 依赖均存在时才执行这种替换。因此并不能替换 resolve 和 substitution 的相关功能。且该 module replacedBy 替换执行在 resolve 阶段之前。即在 Configure 阶段即执行 module 的替换操作。

  - Configuration#withDependencies/Configuration#defaultDependencies

    在 build.gradle 脚本 configure 阶段执行，在 resolve 阶段之前执行。用于遍历 Configuration 中的所有依赖和向 Configuration 中添加默认依赖。

- 依赖的解析流程

replacedBy,withDependencies,defaultDependencies阶段,处于解析build.gradle 脚本的阶段 -> Configuration#ResolutionStategy#each(resolve 阶段）->Configuration#ResolutionStategy# dependencySubstitution (substitution 阶段)->DependencyHandler#getComponents(解析直接依赖的 ComponnetMetaData 数据) -> Configuration#ResolutionStategy#componentSelection(component selection 阶段）Configuration#ResolutionStategy#each(resolve 阶段）->Configuration#ResolutionStategy# dependencySubstitution (substitution 阶段)

resolve/substitution 阶段会被执行两次，一次是在解析 Componnet 依赖的 metaData 数据之前，一次是在解析完成之后。

- ComponentMetadataDetails
  
  TODO:// 该 Details 有什么用？

### Cofiguration/Dependency/PublishArtifact

### Configuration#getIncoming

- ResolvableDependencies
  
  通过 Configuration#getIncoming 获得该对象实例。表示可以被解析的依赖。该依赖被获取之后即被解析，被解析完成之后可以使用 ResolvableDependencies#getFiles 只获得解析的依赖文件。ResolvableDependencies#getResolutionResult 获得依赖之间相互的依赖关系图。ResolvableDependencies#getArtifacts 获得依赖的一些元信息。

- ResolutionResult

  通过 ResolvableDependencies#getResolutionResult 该 ResolutionResult 携带了依赖层级和级联依赖关系的视图。表示某个依赖是如何被引入的。通过该依赖的级联图，可以追溯某个依赖是被如何引入的。

- DependencyResult

  该处只提供依赖的解析信息（如：被谁依赖，依赖的约束条件，2+ 等），但是不提供解析完成的依赖的模块的信息，该信息由下面的 ComponentResult 提供解析完成的依赖信息。
  分为两大子类：ResolvedDependencyResult， UnresolvedDependencyResult  分别表示已经被成功解析的依赖和无法被成功解析的依赖。

  通过 ResolutionResult#getAllDependencies 可获得该类的Set集合对象。用于表示当前该依赖。

- ComponentResult

  通过 DependencyResult#getFrom,ResolvedDependencyResult#getSelected 均可以获得对象。
  用于描述依赖的组建的信息（如：group,name,version,variant 等信息），还通过 ResolvedComponentResult#getDependencies 与 DependencyResult 建立树形结构，用于表示依赖的级联关系。

  组件查询结果分为以下三种类型：
    ComponentArtifactsResult
    ResolvedComponentResult
    UnresolvedComponentResult

### Configuration#getOutgoing

### DependencyHandler#createArtifactResolutionQuery

用于查询当前项目依赖的 module 的一些元数据信息。

- ArtifactResolutionQuery

  forComponents，forModule 指定要查询的组件的坐标，GAV 模式或者 ComponentIdentifier 描述符。

  withArtifacts 指定要获取的 Component 和 Artifact 类型。

- ArtifactResolutionResult

  通过 ArtifactResolutionQuery#execute 可以获得该对象。该对象用于表示要查询的依赖的模块解析结果。通过ArtifactResolutionResult#getComponents 可以获得前面描述的 ComponentResult 进而获得该依赖的详细信息，通常是级联依赖的相关信息。通过 ArtifactResolutionResult#getResolvedComponents 获得 ComponentArtifactsResult 表示 Component 的 Artifact 的查询结果。

- ArtifactResult

  通过 ComponentArtifactsResult#getArtifacts 获得 ArtifactResult 的Set 用于表示查询获得的 Artifact 信息。
  通过 ResolvedArtifactResult#getFile 即可获得查询到的 Artifact 的文件信息。如果是 pom 等 xml 文件即可通过 Xml 相关的解析工具用于获得 xml 文件中的各个节点的信息。如：可以通过 groovy.utils.XmlSlurper 对 xml 文件进行解析操作。

  ArtifactResult 分为以下两种类型:
  ResolvedArtifactResult
  UnresolvedArtifactResult
  
- Component
  
  Library
  下述四种类型通常为 java 项目依赖的组件类型。
  Application
  IvyModule
  JvmLibrary
  MavenModule
  下述四种组件类型通常为 c,c++ 依赖的组件类型。
  DefaultPrebuiltLibrary
  NativeExecutable
  NativeLibrary
  PrebuiltLibrary
  
- Artifact
  
  下述四种 Artifact 为组件提供的一些元信息。

  IvyDescriptorArtifact
  JavadocArtifact
  MavenPomArtifact
  SourcesArtifact

### Configuration#attributes

该方法返回的为 AttributeContainer 为 Configruation 添加额外的属性。通常是为 Consumer 和 Produce 做额外的变种匹配工作。

- Attribute
  
  指定 Attribute 属性的key值，并且指定 Atrribute 属性对应的值的类型。Attribute 为 AttributeContainer 中的 key.

- AttributeContainer

  Attribute 作为 key.Attribute 指定的值类型的值作为值，存储在该容器中。

- 常用的 Attribute 属性 key 值

  - artifactType

    可选类型为 jar,java-classes-directory 等。对应 ArtifactTypeDefinition 中定义的变量。

  - org.gradle.libraryelements

    可选类型为 jar,classes，resources 等。对应 LibraryElements 中定义的变量。

  - org.gradle.usage

    可选类型为 java-api,java-runtime，java-api-classes,java-api-jar.对应 Usage 接口中定义的类型。

#### AttributesSchema

通过 DependencyHandler#getAttributesSchema 可以获得该类型。用于设置 Attribute 匹配时的兼容类型。如：Consumer 依赖 api 但是 Producer 没有 api 属性。此时可以配置兼容策略，Producer 的 api 兼容于 runtime,因此可以用 runtime 代替 api 提供给 Consumer 使用。

- AttributeMatchingStrategy
  
  属性（变种）的匹配策略，内部持有 CompatibilityRuleChain 和 DisambiguationRuleChain 用于变种的兼容和消歧规则的匹配。

- CompatibilityRuleChain

  用于容纳下述的 AttributeCompatibilityRule 兼容规则。实现类同时也继承了 CompatibilityRule 用于对兼容属性的选择。一旦选择到合适的兼容变种，则不再进行后续的选择。

- DisambiguationRuleChain
  
  用于容纳下述的 DisambiguationRuleChain 消歧规则。实现类同时也继承了 DisambiguationRule 用于对消歧规则的选择。

- AttributeCompatibilityRule

  兼容规则 Action 匹配动作的描述。

- AttributeDisambiguationRule

  消歧规则 Action 匹配动作的描述。

- CompatibilityCheckDetails

  传递给兼容规则 Action 进行匹配动作。

- MultipleCandidatesDetails

  传递给消歧规则 Action 进行匹配动作。

- CompatibilityRule
  
  AttributesSchemaInternal#compatibilityRules 内部使用的属性。

- DisambiguationRule
  
  AttributesSchemaInternal#disambiguationRules

### Configuration 分类

JAVA 项目中的 Configuration 分类。

- default
- api/implementation/runtimeOnly/compileOnly
  
  gradle 3.4 后新提供的四种依赖分类。

- compile/provided/runtime

  gradle 3.4 之前提供的依赖，更加粗犷，不能对依赖进行细分

- apiElements/runtimeElements
  
- runtimeClasspath/compileClasspath

- annotationProcessor

- testCompile/testImplementation/testCompileOnly/testRuntime/testRuntimeOnly/testCompileClasspath/testAnnotationProcessor/testRuntimeClasspath

### DependencyHandler#registerTransform

  该处的 Transformer 与 Android 中使用的 ASM 等字节码插桩的 Trasformer 不同。Android 中使用的为 com.android.build.api.transform.Transform.但是目前 android 的 gradle 插件推荐使用 gradle 自带的 TransformAction 机制。但是 gradle 的该机制是从 gradle 5.3 才添加进入 gradle 的。

- TransformAction
- TransformParameters
  
  - @Input
  - @InputFiles
     上述用于标记输入的文件，参考 gradle  task 的输入标记。
  
- TransformOutputs

  输出目录在  /GRADLE_USER_HOME/caches/transforms-2/files-2.1/<hash_code>/
  java 项目中输入通常是原始的 jar 文件。

- @InputArtifact
  
  只注入当前的直接依赖。

- @InputArtifactDependencies

  注入当前依赖的关联依赖。

- @CacheableTransform

  实现可以缓存的 TransformAction ，实现 TransformAction 的缓存机制。

- InputChanges

  实现增量的 TransformAction 依赖转换

- @CompileClasspath

## Publishing 配置

publish 为所有 publis<publicatin_name>PublicationTo<Repo——Name>Repository 任务的聚合任务。通过 Project#Artifacts 生成的 Artifact 可以配置进入 Configuration 被其他 Project 引用和消费。

### Project#Artifacts

  用于生成 Artifacts 向 Publication 中添加。同时会将该 Artifacts 保存进入 Configuration,便于其他项目引用和消费。

### 主要插件

- MavenPublishPlugin

  maven 仓库 publish 组件。简称 maven-publish，声明插件的资源文件: org.gradle.maven-publish

- IvyPublishPlugin

  ivy 仓库 publish 组件。简称 ivy-publish , 声明插件的资源文件: org.gradle.ivy-publish

- PublishingPlugin
  
  publish 共用的组件

### PublishingExtension  配置

使用 publishing 配置该 Extension

- PublishingExtension#RepositoryHandler
  
  配置 publish 发布任务的仓库。

- PublishingExtension#PublicationContainer

  配置 publish 需要发布的构建

- Publication
  
  待发布的构建，具体为 IvyPublication 和 MavenPublication 。
  
- IvyPublication
- MavenPublication

### Publish 任务

 publis<publicatin_name>PublicationTo<Repo——Name>Repository 的任务是在 Project Evaluated 之后才创建的（或者说是执行前创建的，因此需要使用 TaskCollection#withType#configureEach 进行任务的配置操作。

- PublishToMavenRepository/PublishToMavenLocal

  具体执行 publish 到远程仓库和本地仓库的任务。可以通过创建 task 并且使用 Task#dependsOn 和  TaskCollection.withType 传递上述类型，进行 Task 过滤，实现publish 指定类型的聚合任务，如：只执行 publish 到 external 仓库的所有任务，从而实现 publish 任务的聚合。

- PublishToIvyRepository

  具体执行 publish 到远程仓库的任务。

## 软件组建

- Moudule

  单个软件组件 如 java 中的 guava,gson 等第三方依赖jar

- Platform

  一系列软件组件组成的开发平台包。其内在包含版本的一致性与协调性。

- SoftwareComponent

  用于定义指定项目生成的便于发布的组件。如：JavaPlugin#registerSoftwareComponents 注册的 AdhocComponentWithVariants 组件。

- Artifact (org.gradle.api.artifacts.PublishArtifact)

  表示一系列的软件产出产品，可以放置 Configuration 中表示该软件的产出(由 ArtifactHandler 执行添加任务)，交由其他 Project 进行消费。也可以放置于 Publication 中交由 publish 进行发布。

- Publication

  由一系列的 Artifact 组成，用于发布的对象。Publication 有 maven,ivy 仓库不同导致产出的不同。

## Gradle 执行阶段

### Init

### Configuration

配置阶段依赖于*配置注入*,而非继承的方式进行共用配置的提取。配置注入 则是将配置与待配置对象进行隔离，通过类似依赖注入的inject 机制进行配置的配置操作。继承共用配置则是将配置提取到共有的父类进行配置操作。

### Execution

## Gradle 项目目录结构

### 项目目录下的重要文件

- init.gradle/init.gradle.kts
  
  初始化脚本，在 settings.gradle 执行之前执行，该处可以获得 Gradle 对象，无法直接获得 Project 对象，但是可以指定稍后怎么配置 Project.
  如果项目中存在 buildSrc 项目，则 init 脚本会被执行两次，buildSrc 项目被构建视为单独的项目。

  init脚本可以放置在 GRADLE_HOME/init.d 与 GRADLE_USER_HOME/init.d 用于所有 GRADLE 项目的初始化操作。也可以使用 -I --init-script 指定单个项目的 init 初始化脚本

- settings.gradle/settings.gradle (根目录下)
  
  根目录下指示项目目录布局，包含哪些子项目。

- build.gradle/build.gradle.kts (根目录与每个子项目目录)
  
  根项目以及每个子项目的构建配置文件

- gradle.properties (根目录下即 setting 文件的同级目录 or GRADLE_USER_HOME or GRADLE_HOME 目录下)

  根目录下用于配置单个项目所独有的构建属性，该属性会影响 gradle 的构建行为。GRADLE_USER_HOME/GRADLE_HOME 目录下则影响当前用户的所有的 gradle 项目的构建行为。该文件属性的加载由 DefaultGradlePropertiesLoader 负责进行加载。
  
## JAVA 内置 Plugin 及常见 task

### 常见Plugin

- JavaGradlePluginPlugin(org.gradle.java-gradle-plugin.properties,kotlin 简短名称:java-gradle-plugin)
  
  gradle 插件项目依赖的 Plugin.依赖 JavaPlugin。自动导入 gradle api 库的依赖，提供了一些便捷的仓库发布配置.如：便捷的向 gradle plugin 中央仓库发布 plugin,生成 xxx.properties 插件索引属性。
  引入了 GradlePluginDevelopmentExtension 用于给用户配置 Plugin,Test 源码文件的目录。通过 gradlePlugin{ plugins {barPlugin {id = "barid",implementationClass = "package.Implementation"}}}

- JavaPlatformPlugin（org.gradle.java-platform.properties,kotlin 简短名称：java-platform

  用于 构建 maven 的 bom 文件，用于协调相同平台下不同依赖的版本一致性。不同库的版本对齐。

- JavaLibraryDistributionPlugin(org.gradle.java-library-distribution.properties ,kotlin 简短名称:java-library-distribution )
  
- ApplicationPlugin(org.gradle.application.properties,kotlin 简短名称:application)
  
  引入了 JavaPlugin，DistributionPlugin 为其提供基础支持。
  java 应用项目的plugin，提供 java 文件编译，依赖的第三方的 lib jar 打包，zip,tar 并且生成java 应用的启动脚本。依赖 JavaPlugin，DistributionPlugin。提供了一些脚本，依赖jar包的包含，方便使用者可以通过脚本等启动该程序。
  通过 application 名称引入了 ApplicationPluginConvention 惯例配置。该惯例主要服务于 该插件提供的Task。如：run 需要执行的 mainClass,执行需要传递给 jvm 的参数等。

- JavaLibraryPlugin(org.gradle.java-library.properties,kotlin 简短名称:java-library)

  java-library 项目的 Plugin.依赖 JavaPlugin 基本没有添加额外的插件行为。没有添加 Convention

- JavaPlugin (org.gradle.java.properties,kotlin 简短名称: java )

  [java Plugin Manual][https://docs.gradle.org/5.6.4/userguide/java_plugin.html#java_plugin]
  java plugin 则使用了 JavaBasePlugin 的 java Convention 进行配置。
  
- JavaBasePlugin (org.gradle.java-base.properties,,kotlin 简短名称: java-base )

  通过 java 名称引入 JavaPluginConvention 。用于配置 java 的 SourceSet,SourceCompatibility,TargetCompatibility,DocDir,TestDir 等目录属性。
  
- BasePlugin (org.gradle.base.properties,kotlin 简短名称: base )

  通过 base 名称引入了 BasePluginConvention 配置。用于配置 打包完成的相对于 build 目录的 libs,dist 目录的名称，以及生成的 lib jar 的 ArchiveBaseName.

- LifecycleBasePlugin （无 xxx.propperties,kotlin 简短名称：无)
  
  提供 clean,assemble,check,build 等项目基础任务。

- GroovyPlugin(org.gradle.groovy.properties,kotlin 简短名称: groovy )
  
- GroovyBasePlugin(org.gradle.groovy-base.properties,kotlin 简短名称: groovy-base )

### 聚合 Task

- clean
- assemble
  assembleDebug,assemblePreDebug
- check
  test(运行测试用例)
- build
  jar
- publish
  publis<Publication_Name>PublicationTo<Repo_Name>Repository

### 常见 task

- build
  
  只构建和测试当前项目。根目录运行则构建测试当前根项目和所有子项目

- buildNeeded

  构建和测试当前项目以及所依赖的项目

- buildDependents

  构建和测试当前项目以及依赖当前项目的项目
- javaDoc

  根据java代码中的注释生成 html 格式的java 文档。如果要根据 kotlin 代码中的注释生成文档则需要使用 dokka 工具才可以进行。
  
  生成javaDoc 文档根据注释

### java 项目编译过程中的 task

- JavaCompile
  
  在 JavaBasePlugin#configureSourceSetDefaults 方法中创建。

- ProcessResources

- Javadoc
- Classes

  名称为 classes 其实实际类型为 DefaultTask,用于聚合上述的 JavaCompile 和 ProcessResources 任务

- Jar/War/Ear

  Jar 在 JavaPlugin#configureArchivesAndComponent 中创建，依赖 SourceSet#output，用于压缩上述文件进入jar包。Jar 任务只执行压缩成jar包的操作，因此其既可以压缩 JavaCompile 任务生成的 class 文件，也可以压缩 Source 文件生成源码包。
  War 则由 WarPlugin 进行创建。
  Ear 则由 EarPlugin 进行创建。
  War Ear 任务均继承自 Jar 任务。

  jar 中可以设置 jar 包中的 MANIFEST.MF 属性值，通过 JavaPluginConvention 设置的属性无法直接应用于 jar 任务，仍然需要通过 jar 任务设置该属性，并且 Jar#setManifest(Manifest) 类提供属性合并的操作。(*Android 中的Manifest.xml的合并也基本同 jar 中的Manifest的合并操作*)

  使用 application (JavaApplication) 设置的 mainClassName 对生成 jar 包中提供的 MANIFEST.MF 文件并不是有效。该处的 mainClassName 只对 run 任务起效果，run 时会把该 mainClassName 当作 class 进行运行。

- Test

  执行 gradle test 命令时执行该任务。用于执行 junit,TestNG 的测试任务。

- JavaExec

  执行 gradle run 命令时执行该任务。用于运行生成的 java 程序。

- WriteProperties
  
  解决了 java.util.Properties 每次使用 Properties#store 均会生成时间戳更新文件的时间，导致增量构建无法生效的问题。WriteProperties主要在以下几个地方进行了生成文件的优化:

  - comments 不添加时间戳
  - 换行符号默认使用 \n (与系统无关)
  - 属性值按照字母表顺序排序(避免了属性相同只是位置不同导致生成的文件不同的问题)

### Java/JVM 项目中的惯例配置

- JavaPluginConvention

## 自己项目中的 TODO 内容

- 抽离 AARC 项目的共用配置,并且通过 Project#extra 配置 第三方插件的 lib 和 App 项目。
- 尝试将 AARC 子项目的依赖关系配置抽取提出到一个共用的地方。
- 第三方组件项目当不是一个很大的单独项目时，不要使用目前的单独项目模式，可以使用单独的源码目录模式进行管理。
- 引入 google error Prone,PMD,CodeCheck 等代码检查工具，用于检查不规范的写法。
- 寻找 kotlin 的代码检测工具

## References

- [Gradle Src][https://github.com/gradle/gradle]

- [Gradle GetStart Guide][https://gradle.org/guides/]

  通过指导快速的配置 java,objectC,js,android，gradle plugin 等项目的gradle 构建配置。
  [guides 示例项目地址][https://github.com/gradle/guides]

- [Gradle Doc][https://docs.gradle.org/5.6.4/userguide/]
  
  guide 中的用例处于 gradle 项目目录的 /subprojects/docs/src/samples/userguide 子目录下。
  展示 gradle 的基础构Api 使用，以及 gradle 构建框架的基础组件概念，gradle 构建框架的执行生命周期概念。
  
- [Gradle Plugin][https://plugins.gradle.org/]

  gradle 官方开发的插件发布和搜索平台。
  
- [Gradle Api Doc][https://docs.gradle.org/5.6.4/javadoc/]

  gradle public api 文档。
  
- [Gradle Dsl][https://docs.gradle.org/5.6.4/dsl/]
  
  Gradle 核心插件 DSL 配置索引
  
## gradle 设计准则

- COC(Convention Over Configuration)