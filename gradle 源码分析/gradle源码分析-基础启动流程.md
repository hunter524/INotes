# Gradle 源码分析-基础启动流程

## 启动脚本

### gradlew/gradlew.bat

### GradleHome(gradle/gradle.bat)

## 启动入口

### gradlew 启动

启动当前gradle project 目录下的 gradle/wrapper/gradle-wrapper.jar 的 org.gradle.wrapper.GradleWrapperMain 识别 gradle/wrapper/gradle-wrapper.properties 配置的 wrapper 指定的 gradle 版本.下载指定的 gradle 版本执行 gradle 任务.

### gradle 启动

启动 GradleHome 目录下 的lib/gradle-launcher-5.6.4.jar 的 org.gradle.launcher.GradleMain 类.

## 启动类分析

### org.gradle.wrapper.GradleWrapperMain

### org.gradle.launcher.GradleMain

### org.gradle.launcher.Main

## org.gradle.launcher.Main 启动分析

### ParseAndBuildAction

### CommandLineAction

- BuiltInActions

gralde 内建任务创造器.负责创建 gradle -h/gradle --help 和 gradle -v/gradle --version 内置的任务.

- BuildActionsFactory

根据 gradle 命令创建不同的真实任务执行器.如:gradle --daemon,gradle --no-daemon 分别以 daemon 和 非 daemon 执行 gradle 构建任务.

### BuildActionExecuter

### GradleBuildController

控制 gralde 的执行流程.
TODO:// gradle 的三大流程: init configuration execution 的代码分别在什么地方?

### DefaultGradleLauncher

### DefaultSettingsPreparer
