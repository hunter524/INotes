# Regex

## 正则表达式常用语法备忘

### 匹配模式和正则表达式

在表达式内部指定匹配模式 （?s)-.+ 表示指定 -.+ 正则表达式的匹配模式为SingleLine.

常用的正则表达式匹配模式：

TODO://下面的匹配模式暂且备忘，等后续整理正则

这个是正则表达式的模式修饰符。
　　(?i)即匹配时不区分大小写。表示匹配时不区分大小写。

　　(?s)即Singleline(单行模式)。表示更改.的含义，使它与每一个字符匹配（包括换行 符\n）。

　　(?m)即Multiline(多行模式) 。 表示更改^和$的 含义，使它们分别在任意一行的行首和行尾匹配，而不仅仅在整个字符串的开头和结尾匹配。(在此模式下,$的 精确含意是:匹配\n之前的位置以及字符串结束前的位置.) 
　　(?x)：表示如果加上该修饰符，表达式中的空白字符将会被忽略，除非它已经被转义。 
　　(?e)：表示本修饰符仅仅对于replacement有用，代表在replacement中作为PHP代码。 
　　(?A)：表示如果使用这个修饰符，那么表达式必须是匹配的字符串中的开头部分。比如说"/a/A"匹配"abcd"。 
　　(?E)：与"m"相反，表示如果使用这个修饰符，那么"$"将匹配绝对字符串的结尾，而不是换行符前面，默认就打开了这个模式。 
　　(?U)：表示和问号的作用差不多，用于设置"贪婪模式"。



?:  （？）单个问号是不捕捉模式 

写法如：（?:）

  如何关闭圆括号的捕获能力？
      而只是用它来做分组，方法是在左括号的后边加上:?，
这里第一个圆括弧只是用来分组，而不会占用捕获变量，*/ 

    "(?:\\w+\\s(\\w+))"