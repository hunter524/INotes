# Ant 与 Gradle

在 gradle 使用手册 P92 解释 Ant 与 Gradle 的关系时描述 ant 在 Gradle 中为一等公民.同时在 Project,Task 核心组件中添加了 Project#ant,Task#ant 用于获取 AntBuilder 组件,用于构建 Ant 任务.(但是 gradle 手册也提到使用 ant 构建的项目越来越少)

TODO:// 初步查看 ant 的构建模式 target 等概念和 gradle task 的概念很像.

## Ant 构建
