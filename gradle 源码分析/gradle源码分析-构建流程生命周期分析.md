# 构建流程生命周期分析

## gradle 构建的生命周期

Gradle 的生命周期其实是 DefaultGradleLauncher 中定义的 Gradle 执行流程的生命周期.

LoadSettings->Configure->TaskGraph->RunTasks->Finished

### Init

### Configuration

### Execution

## Gradle#addListener机制

### BuildListener

- buildStarted

开始构建,处于 init 阶段.

- beforeSettings

即将加载 Setting 文件,处于 init 阶段.

- settingsEvaluated

setting.gradle 文件解析完毕,主要解析当前项目包含哪些 Project. 处于  Configuration 阶段.

- projectsLoaded

settting.gradle 文件解析完毕,创建对应的 Project 完毕.处于  Configuration 阶段.

- projectsEvaluated

对应的 Project 解析 Project 目录下的 build.gradle 文件解析配置完毕.可以开始确定 Task 执行的 DAG 图.

- buildFinished

gradle 构建完成,也就是当前需要执行的所有 Task 任务执行完毕.

### TaskExecutionGraphListener

Task 执行 DAG 图确定完毕之后的监听.

### ProjectEvaluationListener

## TaskExecutionListener

### TaskActionListener

### DependencyResolutionListener
