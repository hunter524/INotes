# Gradle

## 主流三大构建工具

- Ant
- Maven
- Gradle

## Somethings

1. build.gradle 所处的环境是一个Project环境(可以调用Project接口中定义的方法),当前build.gradle 的脚本中如果没有特定方法和属性，则会委托调用到Project
   对象上的方法和属性。
   但是在其Closure中用this获取的实例为ProjectScript子类的实例，build.gradle其实是处于ProjectScript脚本中。本身groovy文件的执行便是处于一个脚本对象中进行执行的。

2. gradle安装目录存在一个init.d目录,内部可以放置InitScript（xxxx.gradle)文件，每次调用均会执行该处的代码。

3. init.d 目录下的xxx.gradle gradle初始化的时候进行执行，settings.gradle会在initialization阶段进行执行
build.gradle中的脚本会在configuration阶段进行执行，部分脚本在execution阶段进行执行。

4. gradle的执行过程分为三个阶段：initialization->configuration->execution
initialization：初始化阶段执行settings.gradle,加入需要构建的Project，为Project创建Project对象
configuration：配置阶段执行build.gradle文件，建立项目的依赖关系，以及Task的依赖关系。
execution：执行阶段按照顺序执行task任务完成项目的构建
三个阶段根据core部分的源码分包也可以看出:./gradle/subprojects/core/ 项目目录下即分为三个主要的包分别为initialization，configuration，和execution

5. Settings的相关命令：
include 'project1', 'project2:child', 'project3:child1' 根据DefaultSettings的源码，可以得知 project2和其下child两个Project均会被加入
当前项目的构建项目中。
includeFlat 则要求该目录是root的兄弟目录，不能是root的子目录，include则包含的目录是root的子目录。
project命令则是根据名称去寻找到Project，然后修改Project的dir和buildFile的文件名字。
默认的buildFile的名称是由 DefaultScriptFileResolver 类生成的为 build+(.)+支持的脚步文件的后缀名。支持的后缀名称配置在 ScriptingLanguages 文中(目前值 .gradle和.gradle.kts)

6. task相互之间的依赖关系可以使用dependsOn进行配置，且task的命令是属于Project中的命令。
参见同级目录中的build.gradle的定义，task中的{}其实是执行一个方法传入了一个Closure（闭包）。闭包方法会被立刻执行(也就说明了为什么闭包中的方法会在configuration时被调用)，doLast doFirst会在运行的时候执行。

7. gradle中的依赖管理，allprojects，repositories，dependencies在build.gradle 中均为project的方法，传入的参数均为Closure闭包({})。
repositories(Project):对应gradle的RepositoryHandler源码。
dependencies(Project)：对应gradle的DependencyHandler。
subprojects(Project 配置注入):在根项目为所有的子项目注入相应的配置

8.apply from ：可以从当前的gradle中加载另外一个文件的gradle,该新加载的gradle文件是用来配置同一个Project，即ObjectConfigurationAction#target。
  ObjectConfigurationAction的实现类为DefaultObjectConfigurationAction，其会编译提供的脚本路径生成多个Class文件，将其应用到Target上。
apply plugin:'com.android.application' 则为：调用apply 方法传入了一个map。用于应用于当前项目的插件。
  
9.Project#buildScript 通常在Project 根目录下用于配置 当前脚本的执行环境所依赖的jar和第三方库等。

## gradle 常用命令

- gradle
  
  不带有任何命令行参数，其会触发初始化，配置阶段，然后执行配置的 Project#defaultTasks 配置的默认task.
  
- gradle wrapper
  任务可以更新 gradle/wrapper 下面的wrapper至当前gradle版本。gradle-wrapper-x.x.x.jar 为存储在当前 GRADLE_HOME/lib 目录下面，
  gradle wrapper 任务只是将其复制进入 gradle/wrapper 目录下。
  
- gradle tasks
  
  展示当前项目配置的可以用于执行的任务。
  
- gradle dependencies
  
  用于展示当前项目的maven依赖结构，其是按照项目展示依赖的结构的，在根目录下执行则展示的是根目录的依赖结构，在app目录下执行即展示的是app目录的maven依赖。
  在Android 项目中其通常用于分析依赖关系和解决依赖冲突。
  
- gradle properties
  
  展示通过 gradle.properties 设置的所有属性，该属性是针对 build 脚本设置的，也可以使用 gradle 命令: gradle -Pkey=value 设置该脚本属性。与之相对应的
  则存在一个虚拟机属性，需要使用 gradle -Dkey=value
  
## build.gradle/setting.gradle 的构建流程

gradle 内置编译器，会将gradle dsl 的脚本按照一定的规则编译生成 xxx.Class 存储在 .gradle/cache/x.x.x/script 目录下，执行配置时再分别执行
对应的生成的 class 文件。  

- Android 中 assemblePreRelease

依赖于 TaskManager#createAssembleTask 依赖于 BasedVariantOutPutData 通过 buildType buildFlavor 去分别构建不同 Variant的 assembleXXXXXX
任务。

## GradleMain/GradleWrapperMain

- gradle-wrapper-x.x.x.jar
  
  gradlew 命令执行 gradlew同级目录下 ./gradle/wrapper/gradle-wrapper.jar 中的 GradleWrapperMain 类用于适配特定的 gradle 运行版本。

  - gradle.properties 中的参数配置
  
  distributionBase:
  distributionPath:
  下载的 gradle-x.x.x-all.zip,gradle-x.x.x-bin.zip 解压后的存储基目录与子目录。

  zipStoreBase:
  zipStorePath:
  下载的zip存储的基目录与子目录。
  其中基目录通常指向 GRADLE_USER_HOME ,默认的GRADLE_USER_HOME 为 /userdir/.gradle/, base 的可选配置参见 PathAssembler,有PROJECT，GRADLE_USER_HOME.默认的 GRADLE_USER_HOME 参见目录 BuildLayoutParameters#DEFAULT_GRADLE_USER_HOME.

  distributionUrl: 
  指定的wrapper 版本的 gradle-x.x.x-all.zip的下载路径
  
  distributionSha256Sum:
  下载的安装包指定的sha256校验和。

- gradle-launcher-x.x.x.jar

gradle 命令用于执行 gradle 安装包 lib 目录下的 gradle-launcher-x.x.x.jar 中的 GradleMain

## GradleMain的启动流程

### 模块管理与类加载

- 依赖包的加载与类的加载
  GradleInstallation
  DefaultModuleRegistry
  DefaultClassPathProvider
  DefaultClassPathRegitstry
  上述四个类在ProcessBootstrap 启动gradle时负责 gradle 安装目录下的 lib 目录下的 jar 模块类的加载，管理和索引。上述管理的模块即为jar包，一个模块可以由多个jar包构成。每个jar 包中通过 module_name-classpath.properties 文件指定当前 jar包模块依赖的 runtime，projects,optional 分别表示该模块运行时所依赖的class,运行时必须依赖的模块，运行时可选的模块。

- ClassLoaderFactory
  DefaultClassLoaderFactory
  负责构建 gradle 用于加载上述 模块jar包，构建的ClassLoader为 VisitableURLClassLoader

- org.gradle.luancher.Main
  gradle 命令启动了 JVM 虚拟机加载了 gradle-luancher-x.x.x.jar 但是加载真正运行的 org.gradle.luancher.Main#run 运行的并不是 gradle 命令启动的虚拟机的默认的 AppClassLoader.而是上述 VisitaleURLClassLoader ,其也符合 委托双亲的类加载规则，但是该 ClassLoader 的父类加载器为 AppClassLoader 的父类即为 ExtClassLoader 和 BootStrapClassLoader 则该处的VisitableClassLoader 避免了对 jdk 系统类的加载和委托。

### GRADLE 启动执行流程

- org.gradle.launcher.GradleMain
  
  gradle 命令启动的伪启动类，该类只负责启动模块加载机制，初始化模块加载机制的需要使用的模块化的 VisitableClassLoader ，并且通过该 ClassLoader 启动Gradle 程序的真实入口。

- EntryPoint
  - org.gradle.launcher.Main
  - org.gradle.launcher.daemon.bootstrap.DaemonMain

- org.gradle.launcher.Main
  
  - org.gradle.launcher.cli.CommandLineActionFactory

## Gradle 中的服务注册/发现机制(ServiceRegistry)

- DefaultServiceRegistry
  
  构建 DefaultServiceRegistry 时会调用 DefaultServiceRegistry#findProviderMethods 方法，查找当前 DefaultServiceRegistry的实现类，提供的 createXXX,configureXXX,decorateXXX 用于在 ServiceRegistry 获取相关服务时对相关服务进行构建。

- ServiceMethodFactory

  将上述查找的用于create,decorate,configure 的工厂方法包装成为 ServiceMethod 。便于后续 ServiceRegistry 对于相关构造工厂方法的调用。
  该处的方法调用存在两种方式：一种是反射方式调用，另外一种使用 jdk MethodHandles#Lookup
  *此处需要理解反射方法调用与MethodHandles 进行方法调用的区别，以及 MethodHandles ，MethodHandles#Lookup,LambdaMetaFactory 与 lambda 和 invokedynamic 指令的关联*

  - DefaultServiceMethodFactory
  该类不创建 ServiceMethod 只是委托给 其内部找到的 Delegate ServiceMethodFactory 进行 ServiceMethod 的创建。

  - MethodHandleBasedServiceMethodFactory
  
  基于 MethodHandles 封装的 MethodHandlesBasedServiceMethod 方法调用，可能更加底层更加高效，可以使用JIT 虚拟机的方法字节码优化？

  - ReflectionBasedServiceMethodFactory
  
  基于反射 API 的方法调用，只能进行字节码调用而无法使用虚拟机层面的优化操作
  
- LoggingServiceRegistry
  
  gradle 的日志服务管理系统。

- WithLogging

- CommandLineConverter/BuildOption

BuildOption 配合 CommandLineConverter 划分配置层次，CommandLineConverter  负责设置 CommandLineParser 支持哪些 Option 参数的解析(CommandLineConverter#configure，如 -h,--help,-v,--verion,-DPro=Value,-i,--info 等 gradle 命令行参数。配置完 CommandLineParser 后通过 CommandLineParser 将命令行传入的 args 解析为 ParsedCommandLine ,再通过 CommandLineConverter#covert 传递 ParsedCommandLine和 Configuration 如 LoggingConfiguration 用于解析命令行参数配置 Configuration。

- CommandLineConverters
  
  - LoggingCommandLineConverter
  
   以下Option gradle 参数用于配置 日志的显示级别，异常日志堆栈的显示深度，控制台的显示样式。
   LogLevelOption:-q,--quit,-w,--warn,-i,--info,-d,--debug 
   属性控制: org.gradle.logging.level
   StackTraceOption: -s,--stacktrace,-S,--full-stacktrace 
   属性控制: 无
   ConsoleOption: --console = <Plain,Auto,Rich,Verbose> 
   属性控制: org.gradle.console

  - LayoutCommandLineConverter
  
  用于配置 gradle 的用户目录，该目录用于缓存 gradle 的生成的各种临时文件，如maven 依赖的jar,编译 setting.gradle,build.gradle 等待生成的字节码文件，gradle wrapper 下载的特定版本的 gradle-x.x.x.zip 以及其解压之后的文件的放置目录。默认的 gradle_user_home 目录为计算机用户目录下的 .gradle 隐藏目录。
  
   GradleUserHomeOption: -g,--gradle-user-home=/path/path
   属性控制: gradle.user.home
   ProjectDirOption: -p,--project-dir
   属性控制: 无
   NoSearchUpwardsOption: -u,--no-search-upward
   属性控制: 无

  - ParallelismConfigurationCommandLineConverter
  - SystemPropertiesCommandLineConverter

- CommandLineParser/ParsedCommandLine
  
## gradlew 与 gradlew.bat 执行流程

1. gradlew 与 gradlew.bat即是为了：

   - 方便本身没有安装gradle的用户进行构建项目

   - 统一gradle的构建版本

   实际执行的是/wrapper/gradle-wrapper.jar 中的GradleWrapperMain类的main方法,（tips：其中的main方法通过GradleWrapperMain.class.getProtectionDomain().getCodeSource().getLocation().toURI()方法获取到
   当前/wrapper/gradle-wrapper.jar的路径从而读取到相应的Properties。

   *平时我们定义在gradle-wrapper.properties中的属性的key值，其实是在gradle/subprojects/wrapper/项目中的WrapperExecutor类中的一些静态常量*
   如果在gradle-wrapper.properties遗漏相关属性，则其默认会去WrapperConfiguration中去获取，如果WrapperConfiguration中也没有定义相应属性，则根据是否是必须属性，从而决定是否抛出异常。
   *wrapper中的Install 获取的wrapperVersion是什么？为什么要获取gradle-wrapper.jar中的build-receipt.properties中的versionNumber属性？*

   GradleWrapperMain:其中的main函数是gradlew命令启动的java程序负责指定wrapper的gradle版本的下载与启动.

   Install:负责查看本地是否已经缓存了gradle，没有则去下载解压gradle/wrapper/指定的gradle版本。(并且将下载解压之后的gradle版本,传递给WrapperExecutor)
   （tips: .gradle/wrapper/dists/gradle-xxx-xxx/目录下的文件夹的名字 为distribution url MD5之后的比特数组以36为基数生成的字符串(字符串范围在0-9a-z中)
   WrapperExecutor:负责先启动Install的下载任务,检查指定版本的gradle是否已经在指定的缓存目录下(不存在则Install去下载解压),如果存在则返回已经存在的指定的版本的gradle的home的目录,
   交由BootStrapMainStarter去启动执行执行真正的任务.

   BootStrapMainStarter:根据gradle home目录,进入lib目录下面查找gradle-launcher-*.jar文件,从查找到的jar问价加载class,查找org.gradle.launcher.GradleMain类,并且执行该类的main方法.
   进入正题(Gradle的执行流程)  

## Gradle执行流程

1. GradleMain:调用*ProcessBootStrap*加载org.gradle.launcher.main类,执行Main类的run方法,Main类也是一个EntryPoint(即执行EntryPoint的run方法)
   run方法执行到Main自己的Main#doAction方法,doAction构造一个自己的CommandLineActionFactory去转换参数,最终execute的为CommandLineActionFactory.WithLogging#execute方法.

   ->构建ExceptionReportingAction的实例执行其的ExceptionReportingAction#execute方法.(传入了ParseAndBuildAction和BuildExceptionReporter的实例)

   ->执行传入的ParseAndBuildAction#execute try catch包装执行传入的ParseAndBuildAction,执行结束后LoggingOutPutInternal输出日志,同时ExceptionReporter输出错误日志记录.

   ->ParseAndBuildAction#execute时:先加入的Action为BuiltInActions负责处理 -help -version 的命令参数. BuiltInActions和BuildActionsFactory均继承自CommandLineAction

   ->根据命令行输入的args选择需要执行的Action,(tips:命令行是-h -v则通过BuiltInActions返回需要执行的Action,否则通过BuildActionsFactory返回需要执行的Action)

   ->执行BuildActionsFactory#createAction方法,通过参数选择需要执行的任务(决定守护进程的使用策略,新建,复用,直接在当前进程执行),最后均调用BuildActionsFactory#runBuildAndCloseServices
   执行该方法根据参数构建一个RunBuildAction(实现了Runnable接口的一个对象)返回给CommandLineActionFactory#createAction方法,然后通过Actions#toAction将Runnable包装成一个RunnableActionAdapter(适配器模式),
   外层调用Action#execute方法传入的ExecutionListener其实对于RunBuildAction对象是无法获取到的.

   ->创建RunBuildAction对象时传递的ServiceRegistry参数,通常情况下是DefaultServiceRegistry.
   ServiceRegistryBuilder#build->构建DefaultServiceRegistry->调用DefaultServiceRegistry#addProvider->调用DefaultServiceRegistry#findProviderMethods->调用RelevantMethods查找addProvider的以configure,create,decorator开头的方法

           ->Service服务被分为三类 create(创建服务,服务当中以create开头的方法),configure(配置服务,服务当中以configure开头的方法),decorator(装饰服务,服务当中以create,或者decorate 开头的方法,且方法参数类型 和 返回类型 相同的方法)
           
           ->DefaultServiceRegistry#findProviderMethods查找完成所有Provider Object(服务的提供方)的方法之后将 Method 与 Object对象封装成 ServiceMethod实例,DefaultServiceRegistry再将
           ServiceMethod 封装一次形成FactoryMethodService将其加入,DefaultServiceRegistry#ownServices中进行缓存.然后将所有需要configure的方法运行一次,configure方法需要的参数在其他Service的提供方进行查找

   ->适配包装一层RunBuildAction之后,CommandLineActionFactory执行的Action#execute方法实际执行的是RunBuildAction的run方法.

           ->在BuildActionFactory#runBuildInProcess方法,构建RunBuildAction时第三个参数传递进入的BuildExecutor是通过DefaultRegistry去获取的,实际调用的是
           ToolingGlobalScopeServices#createBuildExecuter 去构造的Executor.ToolingGlobalScopeServices是在GlobalScopeServices#configure的时候被添加进入的.
           (*实际的操作是读取gradle-launcher-4.1.jar 中/META-INF/services/目录下的配置文件完成反射获取的*参见DefaultServiceLocator#findServiceImplementations)
           
           ->LauncherServices中的ToolingGlobalScopeService#createBuildExecutor层层包装,返回给RunBuildAction的BuildExecutor为SetupLoggingActionExecuter,
           SetupLoggingActionExecuter内部层层包装各种BuildExecutor,
           
           ->中间的Executor做的工作为添加日志,捕捉执行异常打印,校验执行路径是否正确 等工作.InProcessBuildActionExecutor的工作重点为执行GradleLauncher.
           先通过上下文的ServiceRegistry获取到BuildStateRegistry
           然后通过BuildStateRegistry获取到RootBuildState(通常是DefaultRootBuildState)
           DefaultRootBuildState通过GradleLauncherFactory获取到GradleLauncher(通常是DefaultGradleLauncher)
           RootBuildState会将GradleLauncher包装一层形成形成GradleBuildController
           
           ->BuildController向下传递进入SubscribableBuildActionRunner最终进入ChainingBuildActionRunner,交由LauncherService创建时提供的BuildRunnerActions依次执行传递给其buildController.
           
           ->随后进入DefaultGradleLauncher的执行流程 调用DefaultGradleLauncher#executeTasks 进入Build构建流程

2. DefaultGradleLaunch的执行流程(Gradle主要的构建与执行流程)

    通常将Gradle的执行流程划分为 Initialization,Configuration,Execution三个阶段.(初始化,配置,执行)

    DefaultGradleLaunch将执行流程分为三个阶段:

    Load(主要是执行init.d目录下的初始化脚本,加载settings.gradle文件,构建DefaultSetting对象,建立Project Tree,构建DefaultGradle对象)

    Configure(根据DefaultSetting对象,构建根Project以及子Project对象,并且编译对应的build.gradle文件对对应的Project对象执行配置操作)

    TaskGraph(根据命令行参数,筛选需要执行的Task任务,并且构建需要执行的任务的依赖图(有向无环图),DefaultTaskExecutionGraph(任务的执行图是属于Gradle对象的))
    (任务的循环依赖 图的圈是由 CachingDirectedGraphWalker 该类进行遍历查找)

    Build(根据构建好的Task执行依赖图,执行相关的任务)

### GradleMain 与 GradleDaemon 以及相关调用流程分析

1.GradleMain 是gradle 脚本启动的进程,随后使用ProcessBootStrap启动的Main类
2.GradleDamon是由 GradleMain启动的进程,随后使用ProcessBootStrap启动的DaemonMain类
3.GradleMain通过启动后持有的Process 的InputStream将要处理的命令和任务参数传递给 DaemonMain进程,由DaemonMain进程负责处理和启动任务.
4.ForwardStdinStreamHandler 对于当前进程是向子进程输入,对于子进程则是通过Process#getOutPutStream 获得的流,对于当前进程是output向子进程输入, 对应于子进程则是input流.
Main进程启动 Daemon进程首先通过Process#getInputStream 和 Process#getOutPutStream 获得和Daemon进程的连接通信,然后Daemon进程启动ServerSocket进行通信前的准备,并将启动的SeverSocket的ip port信息通过Process#getOutputStream
发送给Main进程告诉链接的相关参数.
DaemonMain#daemonStarted 方法会在Daemon进程被启动之后,且DaemonMain的Server端的Socket并非是ServerSocket而是SocketChannel(java nio 中的通信方式)

DaemonOutputConsumer即为Client段接收原始Server段通过Process#getInputStream发送过来的信息的工具.
ExecHandle#start 方法也会将 DaemonOutputConsumer#start被调用.

### GradleDaemon 接收到 Build Command之后的流程

1.gradle 命令启动的Client进程,准备参数,将参数封装成为一个Build对象,然后将Build对象序列化之后通过SocketChannel将信息传递给GradleDaemon,GradleDaemon接收到消息后,做编译环境准备,
将参数反序列化之后,最终启动GradleLauncher(DefaultGradleLauncher)的执行流程.
2. DefaultGradleLauncher的大致流程分为:加载Init脚本,Setting脚本(Load阶段),(Config阶段),建立Task任务之间的依赖并且执行Task按照依赖关系(TaskGraph任务依赖关系建立)

#### Settings文件的加载

1. 由CompositeBuildSettingsLoader组合DefaultSettingsLoader进行setting.gradle文件的查找和加载.
找settings.gradle文件后,使用BuildOperationSettingsProcessor委托给RootBuildCacheControllerSettingsProcessor对settings文件进行处理.
委托流程BuildOperationSettingsProcessor -> RootBuildCacheControllerSettingsProcessor -> SettingsEvaluatedCallbackFiringSettingsProcessor -> PropertiesLoadingSettingsProcessor
 -> ScriptEvaluatingSettingsProcessor
 然后使用AsmBackedClassGenerator 生成DefaultSettings_Decorated对象->ScriptEvaluatingSettingsProcessor#applySettingsScript(负责使用setting.gradle生成groovy类,并生成的DefaultSettings_Decorated对象进行配置)->ScriptPluginImpl#apply方法对生成Setting类执行相关配置方法.

 -> setting.gradle(build.gradle)文件其实视为groovy脚本,会被编译生成class(如:/home/hunter/.gradle/caches/4.8/scripts-remapped/build_389d1eleyd18i3jdncorbr7m6/4o32rmyd074n1dom6idjz2idm/proj099cf95f1e5312fd31ac5a8c95b57f40/classes)
 下面的文件即为build.gradle生成的class文件(通过DefaultScriptCompilationHandler#compileScript编译build.gradle文件)

 ->进入DefaultScriptPluginFactory对文件进行编译(两次编译脚本)

 ->BuildScriptTransformer为将文件build.grade文件转换为Class文件(*override GroovyClassLoader#createCompilationUnit,向CompilationUnit中添加ParseOperation完成对xxx.gradle 即Groovy脚本的解析*)
   生成的class文件并不会按照GroovyClassLoader生成的类命名方式进行命名,而是通过RemappingScriptSource进行一次重新命名
   (*build.gradle 会被GroovyClassLoader编译两次,一次是对buildScript块的代码进行编译生成class存放在 cp_proj目录下的class文件中,一次是对其他代码块进行编译(如task等配置任务),存放在proj目录下*)
   使用的是Groovy的AST机制对相关的代码xxx.gradle脚本进行编译.(TaskDefinitionScriptTransformer)
   同时会被重新映射到scripts-remapped目录下面

 ->编译生成_BuildScript_.class会被ScriptRunnerImpl包装一层

1. settings.gradle 文件的处理流程:
通常脚本文件的处理分为 xxx.gradle(build.gradle,setting.gradle) xxx.gradle.kts(kotlin编写的脚本文件),分别使用不同的工厂模式生成处理对象.
kotlin使用:KotlinScriptPluginFactory生成KotlinScriptPlugin用于配置DefaultSetting_Decorated
groovy使用:DefaultScriptPluginFactory生成ScriptPluginImpl用于配置DefaultSetting_Decorated.
Gradle对象是在进入DefaultGradleLauncher之前就已经创建好了的.
加载init.gradle脚本时(init文件中只能获取到gradle对象)
加载Setting.gradle脚本时其只能获取到Setting对象(ScriptEvaluatingSettingsProcessor#applySettingsScript)
加载build.gradle脚本是能获取到Project对象(BuildScriptProcessor#execute方法,即DefaultGradleLauncher configure阶段执行Project#evaluate时执行相关的脚本引擎)

2. DaemonService#createDaemonCommandActions 方法提供了DaemonCommandExecuter#executeCommand式所需要的Actions.
然后DaemonCommandExecution会被循环调用,从而不停地去执行actions.DaemonCommandExecution#proceed 调用DaemonCommandAction#execute 方法,再调用DaemonCommandExecution#proceed从而实现actions的遍历移除被处理.
实现从Actions的第0项元素向最大项元素进行移除操作.
然后通过GradleBuildController调用进入GradleLauncher即DefaultGradleLauncher#executeTask等方法.

3. EntryPoint:有两个子类分别是Main(为gradle 命令启动的java进程) 与DaemonMain(Main进程启动后 启动的编译进程)

4. DefaultScriptCompilationHandler为将Setting.gradle build.gradle 文件编译成为普通java的class文件的类. DefaultScriptCompilationHandler#compileScript为真正执行编译的地方.
5. 脚本文件的处理流程,

## Plugin的实现

1. 插件均实现了gradle的Plugin接口.如Java中的JavaPlugin,Android中的AppPlugin,gradle框架层初始化时会调用一次Plugin#apply方法传入一个Project对象,便于插件添加自己的Task任务并建立依赖关系.

## OtherTips

1.UserHomeInitScriptFinder 查找gradle安装目录下 init.d 目录中的初始化gradle(即该目录下以 .gradle 与 gradle.kts 结尾的文件).

## Gradle中的接口,类,架构整理汇总

### 接口及功能实现

1. BuildAction(I)

2. BuildActionExecutor(I)<----BuildExecutor\<BuildActionParameters\>(I)
BuildActionExecutor 负责最后Build任务的执行以及责任链的向下传递.

3. BuildActionRunner(I)

4. BuildState(I) 

5. Plugin(I) :各种语言的构建插件,java 语言的构建apply plugin:'java'. (即为JavaPlugin),Groovy语言的构建插件为(GroovyPlugin)

6. Gradle(I) :一个项目,就只有一个gradle对象,并且只有一个rootProject.

7. Project(I) :一个build.gradle 通常即为一个Project,一个Gradle项目可以由多个Project组成

8. Task(I) :一个Project有多个Task,实际上的每一个Project即由这些Task组成的.

9. TaskContainer(I) :组合进入DefaultProject对象用户管理,创建相关的Task任务.

10. DomainObjectCollection(I):以及继承他们的子接口,实现了Configuration,Task,Artifact等元素的集合与管理

11. DynamicObject(I) :注册接口实现了Convention插件,DefaultScript等接口和方法的动态寻找和调用,避免了调用者直接使用反射操作对添加的插件和方法进行反射调用.

12. TaskActionListener(I) TaskExecutionListener(I) :两个接口均是监听task任务执行的监听器.TaskExecutionListener start 先于 TaskActionListener start被调用.

13. RepositoryHandler(I) build.gradle 使用 repositories {} 在闭包内部调用的方法即是RepositoryHandler接口的方法,
   在闭包内部调用一次Jcenter则向DefaultArtifactRepositoryContainer中添加一个库.通常一个Project只有一个RepositoryHandler

14. ScriptHandler(I) build.gradle 中使用buildscript{}传入可以调用的实例在该闭包中可以调用ScriptHandler中的方法,用于配置编译需要使用的
类和Plugins插件的路径.

BuildExecutionAction :Build阶段执行Task任务的动作接口
BuildConfigurationAction :Configuration阶段需要执行的任务动作的接口,如根据命令行参数 获取需要执行的Task

### 类的职责

1. HelpTasksPlugin (gradle properties dependencies dependencyInsight)等相关Task任务插件在此处添加

一个gradle项目 只有一个gradle
一个gradle可以由多个Project组成
一个Project可以由多个task组成

Project中调用的方法,不一定是来自Project,可以来自Convention

## Gradle SubProjects子项目整理

1.cli (Common Language Infrastructure) 通用语言基础框架
2.core 核心项目初始化构建层
3.core-api 对外通用提供的构建接口层

## Groovy Tips 

1. 同Kotlin一样定义plus的对象则可以使用+算术运算附，对两个对象进行运算。  

## Gradle Tips

1. 在Project的:
    dependencies{compile 'commons-lang:commons-lang:2.6'} 
   等同于:
   dependencies{ def Dependency = add ('compile','commons-lang:commons-lang:2.6')} 
   实际调用的为DependencyHandler中的add方法,即DefaultDependencyHandler中的add方法.

2. apply plugin: 'com.android.application' 是怎么找到插件的?
   首先会在buildscript 中添加dependencies 添加classpath 'com.android.tools.build:gradle:xxxx'的引用,即引用一个jar,该jar中会有META-INF的资源文件,
   下面会放置com.android.application.properties文件,内部会使用Implementation-class = com.android.build.gradle.LibraryPlugin 指明该id所对应的类.

   apply plugin 'java' java插件则全称为 org.gradle.java.位于gradle-plugins-[version].jar包下.其中的META-INF目录下存在与一个org.gradle.java.properties的
   文件指明了该插件的具体实现类.

3. 在gradle.properties 中配置系统属性(即通过 System.getProperty()可以获取到的属性),如配置gradle.user.home 在gradle.properties中需要配置为:
systemProp.gradle.user.home = /home/GradleHome/

4. buildscript 中的dependencies与 Project中的dependencies的区别:
buildscript中配置的dependencies是在DefaultScriptHandler内部ConfigurationContainer中的一个 classpath的Configuration.
Project中配置的 api 等依赖,是放置在Project的ConfigurationContainer中的,以api为名称的Configuration中的Dependencies中的.

5. Project中的ext Project#ext 属性其实是 Project#extensions#extraProperties 属性
 在应用 com.android.application,kotlin-apt 等插件之后会在Project#extensions#extensionsSchema 中添加以:
 android : com.android.build.gradle.AppExtensions
 kapt : org.jetbrains.kotlin.gradle.plugin.KaptExtension 
 等相关属性.
 即在build.gradle 中使用 kapt group:name:version 和 android{}闭包均来自于 Project#extensions

  拓展:

    - build.gradle 中使用的android kapt apply等方法的提供方和查找优先级:

  属性优先级:
   Project对象自身的方法 -> project.ext.xxx(即Project#extensions#extraProperties中添加的xxx)
   -> 添加到Project#extensions中的Extensions -> 向Project#extensions添加的Convention -> Project中添加的task属性
   -> extra 和 Conventions属性是可以继承的,然后一直向上层Project寻找直到RootProject

   ext.xxx属性(即extensions中的extraProperty)可以定义的Project,Task,Sub-Project上.因为以上每一层级的对象均有extensions属性,
   内部均可以放置extraProperty属性.

   获取相应对象内部定义的ext.xxx属性,只需要在相应的对象内部使用ext.xxx即可获取对应的属性,使用ext.properties 可以获取在Task,Project,
   SubProject内部定义的所有其他属性.

  方法优先级:(Dynamic Methods)
  即build.gradle 可以调用的方法的查找策略:
  
  Project对象自己->build.gradle 文件中声明的方法->DefaultConvention#extensions->DefaultConvention#plugins(即Conventions)->进入Task方法的调用(通常只需要传入一个参数)
  用于配置给定的task的任务->向上递归查找父Project中相关的方法,直到RootProject为止
  
  tips:
    - 如果Property中每个属性的值是一个Closure则该Closure也会被当做方法被调用.
  
    - build.gradle 中的属性调用 与 方法调用的优先级 (以及ext Convention extensions中声明的属性以及方法) 是通过BasicScript 及其子类 DefaultScript,
  ProjectScript,InitScript,SettingScript与build.gradle 通过ScriptDynamicObject耦合在一起的,实现了方法以及属性的优先级策略.
  
    - extensions中添加的是以名字或者类型(class)作为标识符的类型实例,通过名字获取的是整个添加的extension实例
    Convention添加的plugin是单个对象,plugin对象中的方法和属性均可以被单独的调用和获取.

1. Project#absolutePath返回的路径并非正常认为的 /home/hunter/xxx/xxx/ 路径 而是 :app:sub路径的表示

2. Task任务的执行同其他大部分任务一样,也是嵌套执行的 外部每一层添加自己的任务的执行属性,最终委托给内层的Executor对Task进行真正的执行操作(ExecuteActionsTaskExecuter)

3. Gradle Setting Project Task 等对象的构造顺序:

   - 在DefaultGradleLauncherFactory中先构造Gradle 对象,然后使用gradle对象构造DefaultGradleLauncher对象.

   - 在DefaultGradleLauncher的LoadSetting阶段 查找settings.gradle 文件所在的位置.然后创建Settings对象,并使用settings.gradle配置该对象
     *此时的include等添加Project的操作并没有真正的创建Project,只是创建了ProjectDescriptor对象,用于标记subProject的build.gradle,目录地址等属性*
     *此时通常可以修改ProjectDescriptor一些可读 可写属性 比如name属性, gradle :app:get 任务 修改name属性为rename_app 之后则需要执行 gradle :rename_app:get才可以执行该任务*
     *Settings#includeFlat是去包含与当前RootProject同级的项目目录*

   - 在DefaultGradleLauncher的configureBuild阶段 才真正的去根据之前Settings中的ProjectDescriptor去创建RootProject SubProject等Project对象并且配置该对象

   - 在Project的Configure阶段会去创建当前Project中的Task对象.

4. dependencies 闭包中 api Implementation debugApi 等均为configuration的名称,其中包含有DependencySet,其中可以可以包含多个Dependency.在SourceSet被配置的阶段会将
Configuration 加入到对应的SourceSet中.AbstractCompile的Task在配置阶段会去向SourceSet索引 Source,Resource,OutPut等的位置.

5. main test 为JavaPlugin 添加进入JavaPluginConvention的sourceSets(SourceSetContainer)中的一个SourceSet.一个SourceSet中包含多个SourceDirectorySet(e.g java allJava resource allResources)

在SourceSet#output 中classesDir中的分配是 build/classes/$SourceDirectorySet.name/$SourceSet.name/ (i.e 在class编译时 class先按照 SourceDirectorySet的名称进行分类 然后再是SourceSet的名称 
参见 DefaultGroovySourceSet 该方案会按照语言对编译生成的class文件进行分类)

一个SourceSet(i.e DefaultSourceSet)中只有一个outPut (即SourceSetOutput i.e DefaultSourceSetOutput).SourceSetOutput 即为配置classes 和 resources 编译过程中文件的输出目录

11. artifacts的使用和流程 (以及新的PublishPlugin 插件 PublishingExtension 的配置流程)
``
artifiacts{
    archives someFile
}
``
其实是在使用 ArtifactHandler 向名称为 archives的Configuration的artifacts中添加PublishArtifact.
名称为archives的Configuration实际上是在BasePlugin插件被应用时所添加的,BasePlugin还添加了名称为default的Configuration
ArchivePublishArtifact同时还支持使用 AbstractArchiveTask依赖进行构造.

BasePlugin#configureUploadRules 给Task添加配置规则以upload为起始名称的Task会被匹配添加Task,当Task的名称与Project中configurations的名称匹配时则将该Configuration添加到Upload的Task中
*UploadTask 最终执行会进入DefaultArtifactPublisher#publish 方法中执行上传?*

Project之间的依赖与Artifact也与Configuration有关
参见:https://github.com/hunter524/AArcDemo/blob/master/mobile/build.gradle 文件中下述两行:
``
  debugCompile project(path: ':commRouter', configuration: 'debug')
  releaseCompile project(path: ':commRouter', configuration: 'release')
``
依赖指定Project的指定Configuration,具体在源码中是如何实现的?
如果不指定Project依赖则之间依赖default Configuration中的Artifact(Android中通常是 一个生成aar 的 ArchivePublishArtifact)
publishNonDefault true 在android {}中不配置该属性则默认使用default 的Configuration进行Artifact的输出.

如果上传Maven仓库需要依赖maven的插件才可以执行上传任务
mavenDeployer是在MavenPlugin 添加的DefaultMavenRepositoryHandlerConvention中添加的方法.

Project#artifacts是使用DefaultArtifactHandler向指定的Configuration的artifacts中添加 ConfigurablePublishArtifact

-----------------------PublishPlugin 与 PublishExtension -------------------------------------
*通过 PublishingExtension 取代在Configurations 中添加 Configuration 再向Configuration#artifacts(PublishArtifactSet) 中添加
PublishArtifact 的操作 然后依赖于Upload 这个Task去上传指定的 Configuration 中的Artifact*

PublishingExtension 是被添加在Project#extensions 具体参见 PublishPlugin中创建PublishingExtension和设置PublishExtension的方法.

PublishingExtension 只是添加了基础 Publication(发布的产品) repositories(待发布到的仓库的配置功能) 以及添加了 publish 这个名称的task任务

具体publish名称的这个Task任务实际上并没有做什么实质性的任务 而是 MavenPublishPlugin 具体负责向maven仓库的发布任务,同时生成具体发布操作的Task
然后 publish 这个Task再去依赖(dependsOn这些Task)
用户只需要输入 gradle publish 即可以完成所有的任务的发布工作

IvyPublishPlugin MavenPublishPlugin 均依赖于PublishPlugin这个基础插件(i.e publish这个Task 和 publishing 这个Extension 用于配置需要发布的任务和执行发布任务的汇总)

PublishingExtension#publication 是用来添加需要发布的Publication(i.e IvyPublication 与 MavenPublication )

*Configuration中存储的Artifact 与 Publication 中存储的Artifact的比较*
Publication 中存储的Artifact为 PublicationArtifact 
artifact 其 Notation 的解析方式分为 IvyArtifactNotationParserFactory 与 MavenArtifactNotationParserFactory 两个工厂方法


Configuration 中存储的 Artifact 为 PublishArtifact

*IvyPublishPlugin 与 MavenPublishPlugin 是如何执行发布操作的?*
- apply plugin:"ivy-publish"
  通过插件的查找机制 查找到 IvyPublishPlugin 并apply(Project project) 对其进行应用
  ->其再应用基础插件 PublishingPlugin (主要是添加名为publishing 的 PublishingExtension 以及名为 publish 的Task任务)
  ->IvyPublishPlugin 在apply时向Project中类型为PublishingExtension 的Extension中添加 Configure动作,同时为该类型的Extension中的publication添加创建IvyPublication的工厂
  ->为该类型的Extension的每一个Publication中的每一个需要上传的Repositories创建一个类型为 PublishToIvyRepository 名称为"publish" + capitalize(publicationName) + "PublicationTo" + capitalize(repositoryName) + "Repository"的Task
  同时使publish 这个总管Task依赖于该Task(i.e 执行gradle publish 这一个Task便可以完成所有 配置的发布的Task的任务的执行)
  ->PublishToIvyRepository (这个Task的创建依赖于 AnnotationProcessingTaskFactory 解析注解注入需要执行的Action, TaskFactory 反射构造函数构建指定类型的Task)
  ->依赖于标有注解 @TaskAction的方法去执行publish上传至相关ivy与maven库的具体操作(i.e MavenPublishPlugin 与 IvyPublishPlugin 均依赖于 PublishOperation#publish 方法去执行相关操作)
  ->publish发布即是上传 xxx.jar(发布包 可能为xxx.aar) xxx.pom(当前包的依赖配置等文件) xxx-sources.jar(源码包) *一个Artifact 即为一个需要发布的文件,此处的Artifact为 PublicationArtifact 而非Configuration中的PublishArtifact*
  ->最终由ExternalResourceResolver 及其子类 IvyResolver 与 MavenResolver 最终执行相关发布操作
  ->然后交由 ExternalResourceRepository 执行上传操作,是上传本地文件还是上传远程仓库.
  
  https://docs.gradle.org/current/dsl/org.gradle.api.publish.ivy.IvyPublication.html
  中的publish 示例代码 from components.java 取出的Project#Components 中的名称为java的SoftComponent,实际是在JavaPlugin中添加进入的
  JavaLibrary 类型
  artifact source: 'my-docs-file.htm', classifier: 'docs', extension: 'html', builtBy: genDocs // Publish a file generated by the 'genDocs' task
  传递IvyPublication#artifact的参数为Map类型,第一个source参数为创建artifact时使用的参数,后面的参数为配置创建的artifact需要的参数,该处为配置创建好的IvyArtifact的参数
  
  PublicationArtifact -> Buildable#getBuildDependencies 为获取构建该输出(artifact)的Task任务
  
  一个Artifact只可以产出一个文件(配置一个文件发布至maven仓库),但是一个Publication可以有多个Artifact,从而可以实现一次发布既可以发布库,源码,文档三个组成部分.
  
  以配置IvyModuleDescriptor 为例子,IvyPublication 中已经新建好了 IvyModuleDescriptorSpec 的实例,每次执行配置操作时接收到的是Spec实例便是真正存储
  该配置规格的实例.Spec内部对其Spec再进行进一步的划分(e.g IvyModuleDescriptorSpec 内部存在着可以配置的 IvyModuleDescriptorAuthor 的实例)
  在脚本当中闭包内层可以调用外层的方式,是通过闭包的委托实现的.
  
  PublicationArtifact (适用于Publication 中的Artifact输出) 可以由 PublishArtifact (适用于Configuration 中的Artifact输出 ) 解析转换而来. xxx(Ivy,Maven)ArtifactNotationParserFactory
  *PublishArtifact 向 PublicationArtifact 的转换也只是使用 适配器模式 将 PublishArtifact 包装了一层
  AbstractArchiveTask -> PublicationArtifact 的转换也只是将Task进行了一层包装*
  
  使用 MavenPublication 或者 IvyPublication 的artifact方法时,传递的Object对象如果无法被xxx(Ivy,Maven)ArtifactNotationParserFactory
  所解析的话则会无法创建 MavenArtifact 或者 IvyArtifact 则会抛出异常.
  
  在build.gradle 添加Publication( MavenPublication IvyPublication ) 向其中添加 PublicationArtifact ( IvyArtifact,MavenArtifact)
  
  在 MavenPublishPlugin IvyPublishPlugin 中添加了策略(监控Publication的变化) 一旦新添加了Publication 则遍历 PublishingExtension中的repos
  根据当前Publication 和 Repositories 生成新的发布任务,同时使 publish (PublishingPlugin 添加的基础任务)任务 依赖该 Task.
  
  MavenPublication 与 IvyPublication 中from (SoftWareComponent component) 的用法:
  以MavenPublication为例: 
  publish 任务的执行依赖于 PublishToMavenRepository 这个Task,该Task中执行发布时先调用对应publication#asNormalisedPublication
  (先获取到 UsageContext 由对应的 SoftComponent 提供,然后获取到 相应的PublishArtifact 调用 Publication#artifact 包装一下形成 PublicationArtifact 从而添加到 Publication中)
  
  ->通过AbstractMavenPublisher 执行publish操作
  
  ->再通过 AbstractMavenPublishAction #publish执行具体的发布Artifact的操作
  
  
12. Configuration中既可以添加Artifact也可以添加Dependency,artifact和Dependency均区分为两类:一是当前Configuration自己的Artifact和
Dependency.二是当前Configuration继承的Configuration中的所有的allArtifacts和allDependencies.

Configuration可以有层级关系,设置当前的Configuration继承自其他的Configuration
*Configuration中通常只配置dependency,artifacts,其中的一种.*(i.e api的Configuration中只配置了Dependency,default的Configuration中只配置了Artifact)

*api 为配置依赖的Configuration,default为当前Project 做为library被其他项目引用的artifact*
如果是default则可以使用Configuration#getResolvedConfiguration 直接进行Resolve操作,如果是api则不可以使用该方法直接进行Resolve操作
(i.e DefaultConfiguration中的canBeResolved为false断言没有通过)

Configuration同时也分为三个State阶段(RESOLVED,UNRESOLVED,RESOLVED_WITH_FAILURES)已经成功解析,未解析,解析失败

*依赖配置中常用的exclude操作*
Configuration中的exclude 可以在单个Configuration中exclude(i.e 从而影响一种类型的configuration的依赖解析操作,如 api名称的configuration)
exclude也可以在单个ModuleDependency中进行配置,从而影响某种特定的库的依赖搜索(i.e 级联依赖的剔除操作)

从ConfigurationInternal#InternalState的枚举变量值也可以看出Configuration中通常也只配置Artifact或者Dependency中的任意一种,但是也有一些特殊的Configuration中会同时出现配置
Configuration中的artifacts 与 Dependencies的场景(e.g Android app 项目上 devDebugApiElements即同时出现了 artifacts 与 Configuration)
InternalState 分为三种状态 UNRESOLVED(未被解析) GRAPH_RESOLVED(依赖已经被解析) ARTIFACTS_RESOLVED(输出产品已经被解析)

*Configuration#resolveToStateOrLater的解析策略 在解析ARTIFACTS之前需要先解析Dependency*

DefaultConfiguration#defaultDependencies使用的是ImmutableActionSet.empty(),第一次add会变成一个单列Set,第二次add会变成一个Composite Set.
*因此一个Configuration的defaultDependencies 可以有多个 Action 用以配置DependencySet向其中添加Dependency,但是只要一个Action向其中添加成功之后DependencySet不为空时其他Action便不会再被执行*
*该处的DependencySet只与当前的Configuration有关,不与它继承的Configuration有关*



Configuration#resolutionStrategy 属性配置了当Configuration中的Dependency依赖出现冲突时的解决策略
(i.e 通常是依赖的依赖 与 当前 ProjectModule项目的依赖出现版本号等不一致的场景时筛选目前需要的依赖的策略)
ResolutionStrategy#dependencySubstitution中的DependencySubstitutions 定义了build.gradle中配置的依赖的替换策略.
(e.g :
```
project.configurations.all{
    resolutionStrategy.dependencySubstitution{
        substitute module ("com.google.code.gson:gson:2.3.1") with module ("com.google.code.gson:gson:3.0.0")
    }
}
```
使用gson 2.3.1的库替换 3.0.0的依赖库
)

DefaultResolutionStrategy#conflictResolution 当依赖发生冲突的时的的解决策略 目前分为三种 strict(冲突即失败的策略) latest(使用最新的版本库) preferProjectModules(当前Project配置的依赖优先,依赖的依赖次之)


DefaultResolutionStrategy#componentSelection 可以向DefaultComponentSelectionRules 添加ComponentSelection规则(也就是传入一个闭包,闭包接收的参数为ComponentSelection 从而去选择Module的依赖)

DefaultComponentSelectionRules 中可以通过添加被@Mutate注解的方法的对象添加ComponentSelection规则,被@Mutate注解的方法必须第一个参数接收ComponentSelection返回 void


*!!!Configuration中配置好Dependency 同时配置好resolutionStrategy之后是何时去解析获得最终的依赖的结果的?!!!*
*可能与DefaultConfiguration 中的 ConfigurationResolver有关,使用的是DefaultConfigurationResolver
同时Configuration中的Dependency的依赖在被解析时也会被识别成一种图的解构,其内部使用DependencyGraphNode数据解构表示,inComing outComing被识别成为一种图的节点的出度 与 入度
分别代表当前module被哪些module依赖 以及当前module依赖哪些module*


13. gradle中Notation解析机制

此处的Notation 指Dependencies中依赖的表示方式(e.g notation :"io.reactivex:rxjava:1.12.0" 需要被解析为 Dependency),Configuration中exclude(e.g notation:标记map 需要被解析 为ExcludeRule)
Project#file 可以传入多种类型的Object(i.e 传入的Object即为Notation 需要 被解析成为File对象)
Project#task 创建task任务其实也可以使用Notation进行解析的方式进行,然而其并没有这么做.

实际上是通过NotationParserBuilder 和 NotationConverter 和 NotationParser 三个接口实现了NotationParser 和NotationConverter的组合模式
NotationConverterToNotationParserAdapter(类型适配器模式) 将NotationConverter转换成为NotationParser
CompositeNotationConverter 组合多种类型的Converter,形成类似于责任链的解析模式.一旦解析到结果便直接返回结果,而不进行接下来的解析操作.

Dependency的Notation解析依赖于:DependencyNotationParser
Project#file Notation的解析依赖于:FileOrUriNotationConverter#parser
Configuration#exclude的Notation的解析依赖于:ExcludeRuleNotationConverter#parser

14. ArtifactResolutionQuery 
通过ArtifactResolutionQueryFactory进行构造,ArtifactResolutionQueryFactory聚合在DependencyHandler内部,通过Service Locate Pattern创建该Factory实例满足
ArtifactResolutionQueryFactory的依赖
*!!!其中 ComponentIdentifier Component Artifact的相关的定义,分别代表的语义是什么?!!!*
目前查看ComponentIdentifier  ModuleComponentIdentifier 代表的是group:module:version 组件
目前可以理解为是maven library project依赖的一种元数据的依赖和代表.是依赖表示的一种metadata(元数据:描述数据的数据)

Component 与 Artifact 是一种相互关联的关系,向 DefaultComponentTypeRegistry 以Component作为Key注册Artifact.
(e.g 以JvmLibrary这个Component作为Key 注册了 JavaDocArtifact 和 SourcesArtifact)

*!!!该处的Artifact与 Configuration中artifacts的区别 即与 PublishArtifact的区别!!!*

15. build.gradle 中 plugins{ id 'xxx.xxx.xxx' } 与 apply plugin:'xxxx.xxx.xxx'
    plugins 语句块必须放置于 build.gradle 文件的最开始的位置 只有buildscript 和 plugins语句块自己能在 plugins语句块之前.
    
    该处也进一步验证了plugins{} 与 buildscript{} 闭包一样在编译的第一阶段即进行了脚本的解释与运行操作
    其是通过InitialPassStatementTransformer->PluginUseScriptBlockMetadataExtractor->PluginRequestCollector#createSpec创建PluginDependenciesSpec的实例传递调用id方法->
    id方法调用后将DependencySpecImpl放置进入PluginRequestCollector#specs中
    
    tips:gradle 内置的plugin 可以使用 id "java" (非全限定的表示方式) 也可以使用 id "org.gradle.java" (全限定的表示方式)
    DefaultPluginRegistry#lookup 查找时会拼上 DefaultPluginManager.CORE_PLUGIN_NAMESPACE(org.gradle) 插件名称的限定前缀
    
    tips:根据 DefaultPluginResolutionStrategy 一旦Project Loaded则Plugin就会被锁死无法再被进行调用

## JavaPlugin机制

apply plugin:"java" 则回去加载plugins项目的org.gradle.java.properties 中配置的JavaPlugin.class.
->JavaPlugin
添加JavaBasePlugin
添加 main 和 test 的SourceSet 同时配置了相关目录

->JavaBasePlugin 
添加了JavaPluginConvention

配置与编译java,生成classes,resources文件相关的Task.以及test,生成java doc相关的任务 

以下操作的触发时机是当 SourceSet被添加到Project时才会被触发执行,main 和 test 的SourceSet是在JavaPlugin中才会被添加的
JavaBasePlugin#defineConfigurationsForSourceSet方法,根据SourceSet#getImplementationConfigurationName(Implementation) 等方法获取的名称 生成对应的Configuration添加到Project的Configurations中,便于后续
编译Task从Project的Configurations中获取编译的任务.
tips:api 名称的Configuration 是在JavaLibraryPlugin中被创建添加到Project的Configurations中的.
    *一般的Java项目如果不是Library则 api 与 Implementation 没有区别 (library依赖关系:A->B->C B依赖于C如果是api则A可以使用C中的类和方法 B依赖于C如果是Implementation则A不可以使用C中的类和方法)*


->BasePlugin 与 ReportingBasePlugin


  -> BasePlugin
  添加LifeCycleBasePlugin 
  BasePluginConvention (用于配置 distributions libs目录和名称)
  配置以build名称开头的Task:获取以build为名称的Configuration,设置该类型Task的Dependency
  配置以upload名称开头的Task://TODO:具体行为待应用到了再探究
  配置assemble名称的Task:
  
  ->ReportingBasePlugin
  添加了ReportingExtension 
  
->LifeCycleBasePlugin
添加 clean名称的Task 实际为Delete的Task,删除当前Project的build目录
给当前Project的task添加一个规则 删除所有以clean 开头命名的Task的OutputPut目录
同时添加assemble check build 三个Task

#TaskContainer 的作用
TaskContainer被Project持有用于管理Project内部的Task.
Project通过TaskContainer间接的持有Task,创建Task,添加TaskProvider(便于在需要时才去真正的构建Task),并且通过DefaultDomainObjectCollection可以实现对指定类型的Task集合执行配置任务.
TaskContainer内部的很多创建Task的方法并不完全的对build.gradle 脚本暴露.只有Project#task 的几个重载参数的方法可以在build.gradle 中使用.
TaskContainer中的其他方法可以在Plugin中使用,直接使用Project#getTasks 去使用其中的其他的(高级)创建策略





## Gradle Debug调试分析源码流程
1. 修改 GRADLE_HOME/bin/gradle 文件,添加运行时参数:
 -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005 
 即
 eval set -- $DEFAULT_JVM_OPTS $JAVA_OPTS $GRADLE_OPTS "\"-Dorg.gradle.appname=$APP_BASE_NAME\"" -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005 -classpath "\"$CLASSPATH\"" org.gradle.launcher.GradleMain "$APP_ARGS"
 这样才可以调试GradleMain函数.
 
 如果只是在运行gradle命令时添加 -Dorg.gradle.debug=true 即 gradle clean -Dorg.gradle.debug=true  --no-daemon 只可以调试DaemonMain函数
 
 原因是gradle命令运行时添加的参数只对gradle运行之后启动的进程有效,但是GradleMain是命令行运行时启动的参数


##Problem 
1. (resolved) build.gradle (Project) 和 Task 对象中调用ext是什么语法?
   在build.gradle 中 使用 ext{} ext.key="value" 等语法,为获取Project的 ExtensionsContainer,然后传递闭包调用,或者直接向自己的ExtensionsContainer添加键值对的值
   在task中使用ext 基本同在build.gradle使用ext关键字.
   通常在Task或者Project中获取到的ExtensionsContainer为 org.gradle.api.internal.plugins.DefaultConvention的实例.
   使用获取到的ExtensionsContainer获取的ext 通常为 DefaultExtraPropertiesExtension
   ext 代表 ExtensionContainer中的key为ext value为DefaultExtraPropertiesExtension的实例.参见ExtraPropertiesExtension#EXTENSION_NAME 变量
   大多数情况下我们可以自己向ExtensionContainer中添加自己的key 与 value值.
   
2. (resolved) build.gradle中获取到的Gradle为DefaultGradle_Decorated,Project为DefaultProject_Decorated,Task为DefaultTask_Decorated何时产生的?
   是AsmBackedClassGenerator创建了AsmBackedClassGenerator.ClassBuilderImpl,内部持有了一个AsmClassGenerator,并且定义了使用Asm生成的类的名称后缀为_Decorated.
   
3. DefaultProject DefaultGradle 中部分方法未实现,如:getConfigurationActions方法,直接调用会抛出unSupportOperateException,其方法调用会被正常生成的类代理,何处代理了该方法的调用?

4. 如Gradle Tips 第一条所述,dependencies 闭包中的 compile "group:artifact:version" 和 compile project (':subproject')是如何被识别的?
   Project#dependencies传入的闭包,可以调用DependencyHandler,但是生成的BuildScript脚本,初步查看只传入了Project对象并没有传入DependencyHandler对象
   
5. (resolved)dependencies操作是如何实现的?
dependencies传入的闭包引用的是DependencyHandler,DependencyHandler持有Project的ConfigurationContainer,使用 api "xxxx.xxx.xxx:xxx:version"{} 会在ConfigurationContainer中创建一个以api为key的Configuration,然后根据"group:artifact:version"生成Dependency对象加入Configuration中

6. configurations 获取到ConfigurationContainer是什么时候使用,配置Configuration有什么目的?

7. Project中方法优先级中均提及了Extension 和 Convention ,同时Extension的优先级通常高于Convention.
   Extension通常是存储在ExtensionsStorage中,而Convention是存储在什么地方的?
   
   目前查看相关插件的源码发现Convention是添加在DefaultConvention中的plugins中,然后在存储进入ExtensionsDynamicObject中
   方法的调用是通过BaseScript获取到DynamicObject然后通过DynamicObject对相应的方法进行调用.
   DefaultConvention#ExtensionsDynamicObject查找属性和方法的策略则是优先在extensionsStorage中进行查找,当查找不到时才会进入plugins中进行查找,即Convention中进行查找.
   
   *Extension 与 Convention 在Project中使用的区别: 
   Extension只以自己整体添加时候的Name做为Project的一个属性,Project不能引用到Extension内部的属性和方法
   Convention的添加则对于Project来说Convention对象内部的属性和方法均为Project对象的内部的属性和方法
   (e.g 可以参见Project中对属性和方法的获取策略) *
   
   
   
   
8. Project#artifacts 与 ArtifactHandler 是什么关系,如何使用的?
目前已经发现在Android项目中,如果某个项目为Library Project,其会生成一个 名为default的configuration,其中会存储一个ArchivePublishArtifact_Decorated 实例:
ArchivePublishArtifact_Decorated projectname:aar:aar 为其他项目提供依赖?

主项目也会生成一个名为default的Configuration,但是其中没有Artifact

子项目定义构建?主项目使用构建? 主项目可以控制与依赖不同的子项目的构建

9. BeanDynamicObject:职责为封装Convention的属性和方法的调用,当插件调用Project#getConvention#getPlugins#put 添加一个Convention时,Convention的添加是为了
方便Project,Task直接使用Convention中的property和Method.BeanDynamicObject相当于充当方法调用与属性调用的反射工具的功能.

10. 项目最外层的build.gradle(rootProject) buildscript{}语句块中引入的 classpath依赖中的Plugin为什么可以直接在 subProject中直接使用 apply plugin:'xxx.xxx.xxx'进行引用.
但是在apply from 'sendemail.gradle'的脚本中却无法直接使用(需要在当前脚本中重新定义buildscript{}语句块引入相关联的依赖库)

11. Gradle#useLogger是什么用途? 

12. Settings中的includeBuild有何用途?

13. Android Library Project的依赖是何种机制实现的?是否可以通过 改变其 artifact的依赖机制,使Library Project 只编译一次生成一个aar包,然后当前项目均可以依赖该aar包,Library Project
项目不需要重新再进行编译是否会加快编译速度?



  
## 内置的部分Plugin功能间接
- DistributionPlugin:
  
  添加了distributions 方法,用于将指定的源码等目录打包生成 zip 文件.生成的zip或者tar包的名称默认同当前project的名字.
  可以通过baseName属性配置生成zip or tar包的名称.生成的压缩包位于当前Project下 目录为:<project>/build/distributions(暂且未找到配置该目录路径的方法)
  配置详情见Distribution接口的定义.

- EAR JAR WAR
  三种class压缩包打包插件



