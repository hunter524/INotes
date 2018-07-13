#Gradle

##Somethings
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
    默认的buildFile的名称是由DefaultScriptFileResolver类生成的为 build+(.)+支持的脚步文件的后缀名。支持的后缀名称配置在ScriptingLanguages文件中(目前值 .gradle和.gradle.kts)
    
6. task相互之间的依赖关系可以使用dependsOn进行配置，且task的命令是属于Project中的命令。
参见同级目录中的build.gradle的定义，task中的{}其实是执行一个方法传入了一个Closure（闭包）。闭包方法会被立刻执行(也就说明了为什么闭包中的方法会在configuration时被调用)，doLast doFirst会在运行的时候执行。

7. gradle中的依赖管理，allprojects，repositories，dependencies在build.gradle 中均为project的方法，传入的参数均为Closure闭包({})。
repositories:对应gradle的RepositoryHandler源码。
dependencies：对应gradle的DependencyHandler。
subprojects(配置注入):在根项目为所有的子项目注入相应的配置

8.apply from ：可以从当前的gradle中加载另外一个文件的gradle
  在Gradle Project Task SourceSet等增强对象上使用ext一次扩张添加多个属性
  apply plugin:'com.android.application' 则为：调用apply 方法传入了一个map。
9.gradle properties可以列出项目的所有属性。

##gradlew 与 gradlew.bat 执行流程
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
   
##Gradle执行流程
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

####Settings文件的加载
1. 由CompositeBuildSettingsLoader组合DefaultSettingsLoader进行setting.gradle文件的查找和加载.
找settings.gradle文件后,使用BuildOperationSettingsProcessor委托给RootBuildCacheControllerSettingsProcessor对settings文件进行处理.
委托流程BuildOperationSettingsProcessor -> RootBuildCacheControllerSettingsProcessor -> SettingsEvaluatedCallbackFiringSettingsProcessor -> PropertiesLoadingSettingsProcessor
 -> ScriptEvaluatingSettingsProcessor
 然后使用AsmBackedClassGenerator 生成DefaultSettings_Decorated对象->ScriptEvaluatingSettingsProcessor#applySettingsScript(负责使用setting.gradle生成groovy类,并生成的DefaultSettings_Decorated对象进行配置)->ScriptPluginImpl#apply方法对生成Setting类执行相关配置方法.
settings.gradle 文件的处理流程:

5.DaemonService#createDaemonCommandActions 方法提供了DaemonCommandExecuter#executeCommand式所需要的Actions.
然后DaemonCommandExecution会被循环调用,从而不停地去执行actions.DaemonCommandExecution#proceed 调用DaemonCommandAction#execute 方法,再调用DaemonCommandExecution#proceed从而实现actions的遍历移除被处理.
实现从Actions的第0项元素向最大项元素进行移除操作.
然后通过GradleBuildController调用进入GradleLauncher即DefaultGradleLauncher#executeTask等方法.
           
           
   
2. EntryPoint:有两个子类分别是Main(为gradle 命令启动的java进程) 与DaemonMain(Main进程启动后 启动的编译进程)
3. DefaultScriptCompilationHandler为将Setting.gradle build.gradle 文件编译成为普通java的class文件的类. DefaultScriptCompilationHandler#compileScript为真正执行编译的地方.
## Plugin的实现
1. 插件均实现了gradle的Plugin接口.如Java中的JavaPlugin,Android中的AppPlugin,gradle框架层初始化时会调用一次Plugin#apply方法传入一个Project对象,便于插件添加自己的Task任务并建立依赖关系.

## OtherTips
1.UserHomeInitScriptFinder 查找gradle安装目录下 init.d 目录中的初始化gradle(即该目录下以 .gradle 与 gradle.kts 结尾的文件).

## Gradle中的架构接口整理
1.BuildAction(I)
2.BuildActionExecutor(I)<----BuildExecutor<BuildActionParameters>(I)
BuildActionExecutor 负责最后Build任务的执行以及责任链的向下传递.
3.BuildActionRunner(I)
4.BuildState(I) 
5.Plugin(I) :各种语言的构建插件,java 语言的构建apply plugin:'java'. (即为JavaPlugin),Groovy语言的构建插件为(GroovyPlugin)
6. Gradle(I) :一个项目,就只有一个gradle对象,并且只有一个rootProject.
7. Project(I) :一个build.gradle 通常即为一个Project,一个Gradle项目可以由多个Project组成
8. Task(I) :一个Project有多个Task,实际上的每一个Project即由这些Task组成的.


一个gradle项目 只有一个gradle
一个gradle可以由多个Project组成
一个Project可以由多个task组成

Project中调用的方法,不一定是来自Project,可以来自Convention

## Gradle SubProjects子项目整理
1.cli (Common Language Infrastructure) 通用语言基础框架
2.core 核心项目初始化构建层
3.core-api 对外通用提供的构建接口层

##Groovy Tips 
1. 同Kotlin一样定义plus的对象则可以使用+算术运算附，对两个对象进行运算。  

##Gradle Tips
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





