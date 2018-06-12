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
   
#Gradle执行流程
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
   
   
2. EntryPoint:有两个子类分别是Main 与DaemonMain


##Groovy Tips 
1. 同Kotlin一样定义plus的对象则可以使用+算术运算附，对两个对象进行运算。  

