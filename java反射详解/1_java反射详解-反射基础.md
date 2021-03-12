# java反射基础

## 什么是反射?

Java反射机制是指在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

用一句话总结就是反射可以实现在运行时可以知道任意一个类的属性和方法。*Jvm 是如何做到这一点的?*

![OpenJdk 吉祥物 Reflect](imgs/reflect.jpeg)

## 我们能利用反射做什么?

Gson,FastJson,Jackson 等等序列化组件能离开反射吗?

普通平台与 Android 平台的判断 ?

jvm 运行环境的判断?当然可以通过环境变量/运行时属性进行判断,但是这些真的靠谱吗? 还是通过反射查找指定类是否存在更加靠谱?(如:查找 java.util.Optional 如果存在则可以判断当前 jvm 至少是 1.8)

我真的很像调用 private 方法怎么办?

Java SPI 机制的实现?(java.util.ServiceLoader)

## 反射的优缺点

- 优点
  
  可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中它的灵活性就表现的十分明显。比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能的更新，而采用反射机制的话，它就可以不用卸载，只需要在运行时才动态的创建和编译，就可以实现该功能。

- 缺点
  
  慢!另外，反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。*sun 的实现对于反射方法调用分为两种策略直接 native 方法调用 和(当反射调用方法次数 > 15) 生成字节码注入 Jvm 进行方法调用*

## java.lang.reflect 包下有些什么?

### 反射基础类

- java.lang.Package
  
  并不在 java.lang.reflect 包下,但是为了 java 反射语言的完整性该数据对象必不可少.

- java.lang.Class

  - forName
  - getMethods/getDeclaredMethods
     Field,
  - getEnclosingClass/getDeclaringClass
  - getInterfaces/getGenericInterfaces

- java.lang.reflect.Constructor

- java.lang.reflect.Method

- java.lang.reflect.Parameter(1.8 添加)

  用于表示方法中的参数.

- java.lang.reflect.Field

### 反射高阶类(泛型的反射表示)

- java.lang.reflect.GenericArrayType

方法声明的参数,返回值是泛型数组则返回该类型.Method#getGenericParameterTypes, Method#getGenericReturnType

- java.lang.reflect.ParameterizedType

泛型父类已经被参数化的类型,通常是一个子类实现了泛型父类,指定了父类泛型参数的类型.通过子类 Class#getGenericSuperclass 和 Class#getGenericInterfaces 可以获得该类的实例.

方法中声明的返回值,如果带有泛型类.则通过 Method#getGenericReturnType 也会获得该类型的一个实例.

方法中声明的参数,如果带有泛型类型.则通过 Method#getGenericParameterTypes 也可以该类型的一个实例.

- java.lang.reflect.TypeVariable< D >

ParameterizedType 中声明的泛型参数,(*ParameterizedType 虽然是已经参数化的类型,但是其传递的参数可能依旧是泛型类型,如 T,R 等没有被具化的类型*),也可以是方法,类中直接声明的泛型变量 T,R 等未被具化的泛型变量.

- java.lang.reflect.WildcardType

使用 \* 通配符号定义泛型类型时才会该类型的实例.只有使用 \* 时才可以使用 super 关键字.(*java.reflect.WildcardType抽象中的方法定义可以看出,只有其具有 WildcardType#getLowerBounds 用于对应 super关键字*)

使用super关键字指定通配符类型时:

- lowerBounds 为 super 指定的类型
- upperBounds 为 Object 类型
- 如果使用 extends 指定通配符类型则只存在,upperBounds 不存在 lowerBounds.
- 如果只用 ? 声明泛型通配符,则 UpperBounds 默认为 Object 不存在 lowerBounds.

### 抽象接口定义

- Type

  表示 Java 类型系统有哪些类型.主要类型定义接口和实现类为:Class,GenericArrayType,ParameterizedType,TypeVariable,WildcardType

- Member
  
  该接口描述的是 Class 类可以有哪些成员.实现类为:Field,Method,Constructor

- Executable
  
  该接口描述哪些元素是可以被执行.可以被执行的元素只有:Method,Constructor

- AnnotatedElement

  继承自该接口的元素可以被添加注解,实现类或接口为:Package,Class,Constructor,Field,Method,Parameter,TypeVariable

- GenericDeclaration

  实现该接口的元素可以在其上声明泛型变量.

### 辅助类

- Array

  辅助操作数组的类(通过 Class 为 Component 类型构建数组,获取数组指定位置的元素)

- Modifier

  访问修饰符号的定义

- Proxy
  
  我们平常经常所说的 java 动态代理.*动态代理类是怎么生成的?为什动态代理类只能代理接口?源码层面看一下?*

## 从不一样的角度看一下反射(字节码层面)

## Next 下一篇泛型与反射泛型

Java 中的泛型是什么?类型擦出了就真的获取不到类型了吗?什么是泛型 PECS(Produce Extends Consumer Super) 原则?PECS 怎么使用?
