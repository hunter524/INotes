# 文件集合/文件操作/目录解析

## FileLookup

## FileResolver

## FileCollection/FileTree

### DefaultConfigurableFileCollection

### CompositeFileCollection

### DefaultCompositeFileTree/CompositeFileTree

## Project#files

## Project#fileTree/Project#tarTree/Project#zipTree

## AbstractCopyTask

- Copy
  
  配置 Copy Task 等同于配置 CopySpec.如:setDestinationDir 则等同于调用 into.

  - ProcessResources
  
- Sync

### AbstractCopyTask#copy

通过 CopyActionExecuter 与 CopyAction 配合解析当前 Copy Task 配置的 CopySpec 进行复制操作的执行.

### AbstractArchiveTask

- Tar
  
- Zip
  
  - Jar
  
  - War
  
  - EAR

### CopyAction

- DuplicateHandlingCopyActionDecorator
  
  包装下面的 NormalizingCopyActionDecorator 用于对 Copy 任务重复的文件按照策略(报错,告警,覆盖 等)进行处理.

- NormalizingCopyActionDecorator
  
  包装下面的 FileCopyAction,SyncCopyActionDecorator,TarCopyAction,ZipCopyAction 用于执行最终的不同类型的 Copy 子任务.
  
- FileCopyAction
  
  Copy Task 创建的执行 copy 操作的认为　Action.

- SyncCopyActionDecorator
  
  继承　FileCopyAction．为　Sync 任务创建的　Action

- TarCopyAction

　继承　FileCopyAction．为　Tar 任务创建的　Action

- ZipCopyAction

  继承　FileCopyAction．为　Zip 任务创建的　Action

### CopyActionProcessingStream

CopyActionProcessingStream＃process　需要传递进入　CopyActionProcessingStreamAction

CopyActionProcessingStream　通过上述的　CopyAction 层层包装实现不同的功能　CopyActionProcessingStream 包装最外层提供的进行实际任务执行的．

CopyActionProcessingStream　从外向内包装．调用　CopyActionProcessingStream＠process 则是从　Stream 内部向外部进行调用．

lambda 表达式中调用的　action 为内层的　action,调用的　stream 为外层的　stream

- CopySpecBackedCopyActionProcessingStream

该　Stream 则为整个　CopyActionExecuter　的最外层的　Stream 用于遍历　CopySpec 中的文件．遍历文件时再由外向内层级调用　Action 直到调用到　FileCopyAction#FileCopyDetailsInternalAction 进行最终的文件　copy 操作．

### CopyActionProcessingStreamAction

CopyActionProcessingStreamAction＃processFile　需要传递进入　FileCopyDetailsInternal

CopyActionProcessingStreamAction　从内向外包装．
  
## CopySpec

定义 Copy 任务类型的操作规范(定义复制操作的源文件和目标文件,并且可以过滤需要被copy的文件,替换被 copy 文件中的内容，通常是变量占位符)

Copy Task 虽然实现了　CopySpec　但是　from,into,include,exclude... 等等操作都是委托给该任务内部的名称为　mainSpec 的　DefaultSpec 进行操作．

CopySpec 可以进行多层嵌套的定义(如:目录文件夹一个结构)
  
- from
  
  不具有嵌套定义的语义,其均是相对于当前Project 的根目录进行定义的.
  
- into
  
  操作内层是基于外层 CopySpec 定义的into的子目录.
  
- include/exclude:
  
  只定义了include,只copy该目录下include指定的文件.
  只定义了exclude,只copy该目录下除exclude之外的所有文件.
  同时定义了 include 与 exclude 则 copy include 文件中除去 exclude 文件的文件.

- rename
  
  按照定义的替换规则,重命名被 copy 的文件.

- filter

  应用各种 FilterReader 替换器替换待 copy 文件中的变量占位符.如:org.apache.tools.ant.filters.ReplaceTokens::class, "tokens" to mapOf("year" to "2009","month" to "06","day" to "day")

  同时 filter 也支持Tranformer/Closure 的整行模式.

- expand

  使用 Groovy SimpleTemplateEngine 替换 GString 占位符号,GString 表达式中的占位符.

- with
  
  复制/重用 既有的 CopySpec 进入当前 CopySpec

- eachFile

  遍历每个from文件,设置exclude,copy 操作的目标目录

### DelegatingCopySpecInternal

继承自　CopySpecInternal　实现包装委托功能．

### DestinationRootCopySpec

继承自　DelegatingCopySpecInternal　添加　destinationDir 属性，其他方法调用，全部委托给被包装的DefaultCopySpec/SingleParentCopySpec .

### DefaultCopySpec

### SingleParentCopySpec

继承自　DefaultCopySpec．表示只有一个父目录的　CopySpec 用于在　DefaultCopySpec ／ SingleParentCopySpec　添加子　CopySpec 时使用．*Todo:// 目前并没有发现该处的 DefaultCopySpec与SingleParentCopySpec定义的不同之处*

## CopySpecResolver

用以耦合(解析) 对应的 CopySpec　形成　FileTree 提供给　CopyAction 进行文件的　copy,zip,tar...等等操作．该处的耦合是通过　CopySpecBackedCopyActionProcessingStream＃process　和　CopySpecInternal＃walk　进行解析和操作的．
*在解析　CopySpec 成为　FileTree 的过程中也进行了对应的　PatternSet 匹配规则的应用，用于过滤和包含相应的文件*

## PatternFilterable/PatternSet

模式匹配接口,用于定义 include,exclude 规则.*TOOD:// 查找在 FileTree/FileCollection 中结合该模式匹配所进行的文件规则过滤操作*

## FileCopyDetails/DefaultFileCopyDetails

执行最终的文件复制任务以及文件复制过程中的文件内容过滤/替换操作等.同时在执行 CopyFileVisitorImpl#processFile 时会遍历 CopySpecResolver 即 CopySpec 中的 CopyActions 从而对 FileCopyDetails 进行配置操作(如:rename 操作即在该处进行,替换 FileCopyDetails 对象中的目标文件的名称)
