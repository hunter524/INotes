# 属性配置

## gradle -D 属性

## gradle -P 属性

## 环境变量/系统属性/Project#ext/Task#ext

### Project#ext属性

### Task#ext属性

## ExtensionAware

继承自 ExtensionAware 接口的实现都具有 ext,extra 属性.ext/extra 实质为同一个配置容器的不同名称均为 ExtensionContainer 中的 ExtraPropertiesExtension.

使用 Task#extra 无法读取到 Project#ext 的属性.反之也无法读取. Task 与 Project 分别有各自的 ext 配置属性.

## ExtensionContainer/ExtraPropertiesExtension(ext)/DefaultExtraPropertiesExtension

## ExtensibleDynamicObject

TODO://可继承动态属性分析 分析 DefaultProject 创建时是如何创建和构造该对象的.

## Project#ext/Taks#ext

## ExtensionContainer/Convention

### Convention#getPlugins

添加用户自定的惯例,便于配置默认的构建配置.通常由 Plugin 提供.

### ExtensionContainer/Project#getExtensions

通常添加用户自定义的数据模型扩展. 如 JavaBasePlugin 中的 sourceSets,java 扩展.扩展可以持有 Convention 用于对 Extention 进行默认的配置.
