# JSONPath 解析规则树

## 0-9 和 a-z 和 A-Z

当且当 path 只有一个字符时,且为上述字符时才执行这些操作.

- 0-9: 表示对根元素假设为 List,Array,集合 按照下标进行取值,最多只能支持到 Map 下标当作 Key 进行取值.
- a-z 和 A-Z:对于单个字符当作属性名称进行取值.

## $

表示对于根元素的引用,在 path 表达式中可以显示的声明也可以省略.

### $?()

用于过滤根元素的匹配规则.匹配规则的写法和适配可以参见后面对于 JSONPathParser#parseArrayAccessFilter 的解析.

### $. 与 $/

#### $\.\.

递归操作遍历每一层的对象的属性

- $.. 与 $..[

 表示递归搜索,表示只取对象的值(不取内部的字段的值).使用 WildCardSegment.instance_deep_objectOnly

- $..\* 与 $..[\*]
  
  表示递归每一个对象层级获取元素的属性

- $..[\*].\*

 相对于 $..[\*] 的取值的结果再进行取对象的值的操作.

- $.\*
  
  使用 WildCardSegment.instance

#### $..name

该处则是相对于 $.. 的结果,再取其属性为 name 的值.*语义则为递归取 JSONObject 内部名称为 name 的属性值*

#### $.1

#### $.size() $.length() $.max() $.min() $.keySet() $.type() $.floor()

分别表示取根元素的值的个数,最大值,最小值,key 的集合,根元素的类型,以及将根元素进行向下取整的操作.

#### $.name

#### $.[1]

## . 和 /

省略 $ 对于根元素的引用,下面的表达式写法与 $. $/ 并无区别.

## ?()

为 $?() 省略 $ 表示根元素的写法.

## name

 取消 $ 表示根元素,使用 name 直接获取想对于根元素的指定的属性的名称

## [1]

 同上,只是另外一种冗余的写法而已.

## JSONPathParser#parseArrayAccessFilter

### [] 匹配

标注的 JSONPath 条件匹配表达式.

#### [?()]

- [?(@.name)]

- [?(name)]

- [?(name1) && (name2)]

  *TODO:// && 与 || 只能两个条件并列?*

#### [name]

name 可以是普通属性名称,也可以是特殊属性名称 last *只有当 last 是 JSONPath 路径的最后的一个值时才生效*.

#### [?(@.name op value)] 和 [?(name op value)] 和 [ name op value]

op 主要分为以下几种:

表达式语句:

- between/not between

  between 的两个值只能是 long 或者可以转换成为 long 的整数值(*如:byte,shor,int,long*).写法如:[?(@.name between 10 and 20)] 其他变种写法,not between 写法以此类推

- in /not in /nin
  
  in ,not in 要求匹配的值全部为同一种类型 long 或者 String 类型.

- like/not like
- rlike/not rlike

普通运算符:

- =,==
  
  EQ 相等条件匹配

- =~
  
  正则表达式条件匹配

- !=
  
  NE 不相等条件匹配

- < , <=  , > , >=

引用运算符:

- $
  
  判断当前路径的值和某个引用的值是否相等.

### ?() 匹配

该种表达式是 fastJson 对于 [?(@.name)] 的一种简化显示,在其他 JSONPath 解析库中不一定支持.