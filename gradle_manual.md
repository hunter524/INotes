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
  
  自动内置在 gradle 构建项目内部的任务,用于查看gradle 的构建环境.主要用于查看 gradle 构建任务执行依赖的库,而并非构建的项目依赖的库

- gradle <task_execute> --exclude-task <task_exclude>
  
  执行某个 task 及其依赖的task 跳过某个特定的task不执行.

- gradle --continue <task_name> <task_name>
  
  当前面的 Task 任务运行失败之后后面的 task 任务继续运行.

## gradle 插件的编写/引用

- 在 build.gradle 中直接定义插件
- 在项目根目录建立 buildSrc 目录，其中内置 gradle 插件项目
- 引用第三方插件 jar 包，再引用插件

### buildSrc

buildSrc目录置于根目录作为gradle 构建脚本,插件配置的默认目录.其中可以存放共用的gradle脚本,gradle插件项目(kotlin,java项目).为当前项目内部提供共享的脚本和插件.gradle 根项目及子项目会默认依赖和编译该项目.

## Build Cache

## Composite Build

并非是多项目结构构建(一个 Gradle Project 通过 setting 文件,包行多个sub_project),组合构建的项目实际上是并无直接关联的两个项目(独立的 gradle项目) 原本通过依赖 artifacts(项目的产出)更改为直接依赖该项目的模式.

组合构建比多项目结构构建相互之间的依赖更加独立.(该层之间的独立是由)

## Setting

对应 setting.gradle/setting.gradle.kts 构建脚本最先被执行,用于识别gradle项目的目录结构.

## Project

### ExtensionContainer （Project#getExtensions) 扩展

### Convention （Project#getConvention) 惯例

该处的 Convention 接口其实是实现了 ExtensionContainer。

## Task

创建 Task 时可以通过 Map 指定 Task 相关的属性.属性名称如下: name,description,group,type,dependsOn,overwrite,action,constructorArgs.DefaultTaskContainer#create(Map<String,?>)用于解析Map参数创建对应的Task.

### Copy（文件复制）

继承自 AbstractCopyTask.

- 实现文件从一个目录复制进入另外一个目录，并且添加过滤匹配规则
- 通过 from 结合 Project#zipTree Project#tarTree 实现zip文件的解压复制和归档文件的复

### 文件移动/文件重命名

文件移动的相关操作 gradle 没有内建提供相关支持，需要使用 Gradle 集成的 Project#ant 组件对文件提供相关支持。GRADLE_HOME/lib 目录下内置了 ant-1.9.14.jar 等 ant 构建相关的 jar 包以提供对 ant 构建任务和命令的支持。gradle 文件的移动则依赖于 ant 构建工具提供的任务。

文件的重命名任务则只需要依赖于 CopySpec#rename 配合 Copy 任务即可完成。

### 文件内容的过滤/替换操作

使用 CopySpec#expand,CopySpec#filter(Token),CopySpec#filter(Closure) 方式通过模板方式替换制定 token,通过 Closure 闭包返回修改后的字符串方式。模板方式分别支持 Ant 样式模板，GString 样式模板，Groovy 的 SimpleTemplateEngine 模板引擎。

### Zip/Tar//Jar/War/Ear （文件压缩）

上述 Task 均继承自 AbstractArchiveTask ，同时 AbstractArchiveTask 又继承自 AbstractCopyTask

### Delete (文件删除)

可以使用 Delete 任务和 Project#delete 指令实现文件的删除操作。但是匹配待删除文件时无法使用像 CopySpec 这样的 include,exclude 指令进行文件的过滤和包含。需要使用 FileTree 和 FileCollection 相关的指令进行文件的过滤和筛选。

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
