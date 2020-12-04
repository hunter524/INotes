# 函数一等公民

## 函数式编程

函数式编程（functional programming）或称函数程序设计、泛函编程，是一种编程范式，它将计算机运算视为函数运算，并且避免使用程序状态以及易变对象。其中，λ演算（lambda calculus）为该语言最重要的基础。而且，λ演算的函数可以接受函数当作输入（引数）和输出（传出值）。

### 函数式编程的本质

函数式编程中的函数这个术语不是指计算机中的函数（实际上是Subroutine），而是指数学中的函数，即自变量的映射。也就是说一个函数的值仅决定于函数参数的值，不依赖其他状态。比如sqrt(x)函数计算x的平方根，只要x不变，不论什么时候调用，调用几次，值都是不变的。

在函数式语言中，函数作为一等公民，可以在任何地方定义，在函数内或函数外，可以作为函数的参数和返回值，可以对函数进行组合。

纯函数式编程语言中的变量也不是命令式编程语言中的变量，即存储状态的单元，而是代数中的变量，即一个值的名称。变量的值是不可变的（immutable），也就是说不允许像命令式编程语言中那样多次给一个变量赋值。比如说在命令式编程语言我们写“x = x + 1”，这依赖可变状态的事实，拿给程序员看说是对的，但拿给数学家看，却被认为这个等式为假。

函数式语言的如条件语句，循环语句也不是命令式编程语言中的控制语句，而是函数的语法糖，比如在Scala,Kotlin语言中，if else不是语句而是三元运算符，是有返回值的。

严格意义上的函数式编程意味着不使用可变的变量，赋值，循环和其他命令式控制结构进行编程。

从理论上说，函数式语言也不是通过冯诺伊曼体系结构的机器上运行的，而是通过λ演算来运行的，就是通过变量替换的方式进行，变量替换为其值或表达式，函数也替换为其表达式，并根据运算符进行计算。λ演算是图灵完全（Turing completeness）的，但是大多数情况，函数式程序还是被编译成（冯诺依曼机的）机器语言的指令执行的.

### 函数式编程的特性

只用"表达式"，不用"语句"：通过表达式（expression）计算过程得到一个返回值，而不是通过一个语句（statement）修改某一个状态。

- 无副作用：不污染变量，同一个输入永远得到同一个数据。
  
  也可以理解为 stateless ,函数不维护任何状态。函数式编程的核心精神是 stateless，简而言之就是它不能存在状态，也不能改变和依赖外部的状态.因此称为无状态,正是因为无状态才可以保证同一个输入得到同一个输出.

- 不可变性：输入数据是不能动的，动了输入数据就有危险，所以要返回新的数据集,同时新的数据集也需要保证不可变性。(同时不可变性也是对无副作用的一种保证,不动输入参数则保证了不改变外部状态,返回不可变的数据则可以保证函数式编程的这两个特性的传递)
  
由于变量值是不可变的，对于值的操作并不是修改原来的值，而是修改新产生的值，原来的值保持不便。 通常来说，算法都有递推（iterative）和递归（recursive）两种定义。

由于变量不可变，纯函数编程语言无法实现循环，这是因为For循环使用可变的状态作为计数器，而While循环或Do While循环需要可变的状态作为跳出循环的条件。因此在函数式语言里就只能使用递归来解决迭代问题，这使得函数式编程严重依赖递归.*在实际的实现过程中并不可能完全做到无副作用,通常的做法是将副作用集中在某个点,而不是到处都会产生副作用,同时更做不到不依赖语句,除了像 Haskell 这种专门为函数式编程而诞生的语言*

### 函数式编程的其他特性

- 高阶函数（Higher-order function）：就是参数为函数或返回值为函数的函数。有了高阶函数，就可以将复用的粒度降低到函数级别，相对于面向对象语言，复用的粒度更低。
  
- 偏应用函数（Partially Applied Functions）：一个函数接收一个有多个参数的函数，返回一个需要较少参数的函数。偏函数将一到多个参数在内部固定，然后返回新函数，返回的函数接收剩余的参数完成函数的应用。
  
- 柯里化（Currying）：输入一个有多个参数的函数， 返回一个只接收单个参数的函数。(将一个函数变成了一个偏应用函数)

- 闭包（Closure）：闭包就是有权访问另一个函数作用域中变量的函数.闭包的三个特性：1.闭包是定义在函数中的函数 。2.闭包能访问包含函数的变量。3.即使包含函数执行完了, 被闭包引用的变量也得不到释放。具体参看 [闲话闭包](https://www.zhoulujun.cn/html/webfront/ECMAScript/js6/2015_0814_240.html)

### 函数式编程的好处

由于命令式编程语言也可以通过类似函数指针的方式来实现高阶函数，函数式的最主要的好处主要是不可变性带来的。没有可变的状态，函数就是引用透明（Referential transparency）的和没有副作用（No Side Effect）。

函数既不依赖外部的状态也不修改外部的状态，函数调用的结果不依赖调用的时间和位置，这样写的代码容易进行推理，不容易出错。这使得单元测试和调试都更容易。

由于（多个线程之间）不共享状态，不会造成资源争用(Race condition)，也就不需要用锁来保护可变状态，也就不会出现死锁，这样可以更好地并发起来，尤其是在对称多处理器（SMP）架构下能够更好地利用多个处理器（核）提供的并行处理能力。

我觉得函数编程的好处就不用管js里面该死的this指向.

函数式编程语言还提供惰性求值-Lazy evaluation，也称作call-by-need，是在将表达式赋值给变量（或称作绑定）时并不计算表达式的值，而在变量第一次被使用时才进行计算。这样就可以通过避免不必要的求值提升性能。(在面向对象时某些值传递给某个对象的方法时需要提前计算得出,这个计算或许会很耗资源,但是使用者可能压根不使用这个值,面向函数时则可以传递求该值的方法进入即可.例如:警察需要调查某个人,要获得该人从出生到当前的所有经历,这个经历只有被调查人自己知道,那么面向对象就是该人事先准备好经历文案给警察调查,面向函数则是该人告诉警察你需要数据的时候打我电话,我来告诉你.可能警察压根不关心你自己提供的经历)

函数式编程天生亲和单元测试(特别是黑盒测试)，因为FP关注就是输入与输出。反观Java或者C++，仅仅检查函数的返回值是不够的：代码可能修改外部状态值，因此我们还需要验证这些外部的状态值的正确性。在FP语言中呢，就完全不需要。

调试查错方面，因为FP程序中的错误不依赖于之前运行过的不相关的代码。而在一个指令式程序中，一个bug可能有时能重现而有些时候又不能。因为这些函数的运行依赖于某些外部状态， 而这些外部状态又需要由某些与这个bug完全不相关的代码通过某个特别的执行流程才能修改。在FP中这种情况完全不存在：如果一个函数的返回值出错了，它一直都会出错，无论你之前运行了什么代码。而整个程序就是函数接龙。

## 一个简单的例子

我们从一个愚蠢的例子开始。下面是一个海鸥程序，鸟群合并则变成了一个更大的鸟群，繁殖则增加了鸟群的数量，增加的数量就是它们繁殖出来的海鸥的数量。注意这个程序并不是面向对象的良好实践，它只是强调当前这种变量赋值方式的一些弊端。

```js
var Flock = function(n) {
  this.seagulls = n;
};

Flock.prototype.conjoin = function(other) {
  this.seagulls += other.seagulls;
  return this;
};

Flock.prototype.breed = function(other) {
  this.seagulls = this.seagulls * other.seagulls;
  return this;
};

var flock_a = new Flock(4);
var flock_b = new Flock(2);
var flock_c = new Flock(0);

var result = flock_a.conjoin(flock_c).breed(flock_b).conjoin(flock_a.breed(flock_b)).seagulls;
//=> 32
```

我相信没人会写这样糟糕透顶的程序。代码的内部可变状态非常难以追踪，而且，最终的答案还是错的！正确答案是 16，但是因为 flock_a 在运算过程中永久地改变了，所以得出了错误的结果。这是 IT 部门混乱的表现，非常粗暴的计算方式。
如果你看不懂这个程序，没关系，我也看不懂。重点是状态和可变值非常难以追踪，即便是在这么小的一个程序中也不例外。

我们试试另一种更函数式的写法：

```js
var conjoin = function(flock_x, flock_y) { return flock_x + flock_y };
var breed = function(flock_x, flock_y) { return flock_x * flock_y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = conjoin(breed(flock_b, conjoin(flock_a, flock_c)), breed(flock_a, flock_b));
//=>16
```

很好，这次我们得到了正确的答案，而且少写了很多代码。不过函数嵌套有点让人费解...（我们会在后面的组合(compose)）。这种写法也更优雅，不过代码肯定是越直白越好，所以如果我们再深入挖掘，看看这段代码究竟做了什么事，我们会发现，它不过是在进行简单的加（conjoin） 和乘（breed）运算而已。

代码中的两个函数除了函数名有些特殊，其他没有任何难以理解的地方。我们把它们重命名一下，看看它们的真面目。

```js
var add = function(x, y) { return x + y };
var multiply = function(x, y) { return x * y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));
//=>16
```

这么一来，你会发现我们不过是在运用古人早已获得的知识：

```js
// 结合律（assosiative）
add(add(x, y), z) == add(x, add(y, z));

// 交换律（commutative）
add(x, y) == add(y, x);

// 同一律（identity）
add(x, 0) == x;

// 分配律（distributive）
multiply(x, add(y,z)) == add(multiply(x, y), multiply(x, z));
```

是的，这些经典的数学定律迟早会派上用场。不过如果你一时想不起来也没关系，多数人已经很久没复习过这些数学知识了。我们来看看能否运用这些定律简化这个海鸥小程序。

```js
// 原有代码
add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));

// 应用同一律，去掉多余的加法操作（add(flock_a, flock_c) == flock_a）
add(multiply(flock_b, flock_a), multiply(flock_a, flock_b));

// 再应用分配律
multiply(flock_b, add(flock_a, flock_a));
```

漂亮！除了调用的函数，一点多余的代码都不需要写。当然这里我们定义 add 和 multiply 是为了代码完整性，实际上并不必要——在调用之前它们肯定已经在某个类库里定义好了。

你可能在想“你也太偷换概念了吧，居然举一个这么数学的例子”，或者“真实世界的应用程序比这复杂太多，不能这么简单地推理”。我之所以选择这样一个例子，是因为大多数人都知道加法和乘法，所以很容易就能理解数学可以如何为我们所用。

## 一等公民的函数

当我们说函数是“一等公民”的时候，我们实际上说的是它们和其他对象都一样...所以就是普通公民（坐经济舱的人？）。函数真没什么特殊的，你可以像对待任何其他数据类型一样对待它们——把它们存在数组里，当作参数传递，赋值给变量...等等。

函数是 JavaScript 语言的基础概念，不过还是值得提一提的，因为在 Github 上随便一搜就能看到对这个概念的集体无视，或者也可能是无知。

像使用其他所有变量一样使用函数!!!

```js
// 太傻了
const getServerStuff = callback => ajaxCall(json => callback(json));

// 这才像样
const getServerStuff = ajaxCall;
```

世界上到处都充斥着这样的垃圾 ajax 代码。以下是上述两种写法等价的原因：

```js
// 这行
ajaxCall(json => callback(json));

// 等价于这行
ajaxCall(callback);

// 那么，重构下 getServerStuff
const getServerStuff = callback => ajaxCall(callback);

// ...就等于
const getServerStuff = ajaxCall // <-- 看，没有括号哦
```

```js
const BlogController = {
  index(posts) { return Views.index(posts); },
  show(post) { return Views.show(post); },
  create(attrs) { return Db.create(attrs); },
  update(post, attrs) { return Db.update(post, attrs); },
  destroy(post) { return Db.destroy(post); },
};
```

```js
const BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};
```

现在我们来看看钟爱一等公民的原因是什么。前面 getServerStuff 和 BlogController 两个例子你也都看到了，虽说添加一些没有实际用处的间接层实现起来很容易，但这样做除了徒增代码量，提高维护和检索代码的成本外，没有任何用处。

另外，如果一个函数被不必要地包裹起来了，而且发生了改动，那么包裹它的那个函数也要做相应的变更。

```js
httpGet('/post/2', json => renderPost(json));
```

如果 httpGet 要改成可以抛出一个可能出现的 err 异常，那我们还要回过头去把“胶水”函数也改了。

```js
// 把整个应用里的所有 httpGet 调用都改成这样，可以传递 err 参数。
httpGet('/post/2', (json, err) => renderPost(json, err));
```

写成一等公民函数的形式，要做的改动将会少得多：

```js
httpGet('/post/2', renderPost);  // renderPost 将会在 httpGet 中调用，想要多少参数都行
```

除了删除不必要的函数，正确地为参数命名也必不可少。当然命名不是什么大问题，但还是有可能存在一些不当的命名，尤其随着代码量的增长以及需求的变更，这种可能性也会增加。

项目中常见的一种造成混淆的原因是，针对同一个概念使用不同的命名。还有通用代码的问题。比如，下面这两个函数做的事情一模一样，但后一个就显得更加通用，可重用性也更高：

```js
// 只针对当前的博客
const validArticles = articles =>
  articles.filter(article => article !== null && article !== undefined),

// 对未来的项目更友好
const compact = xs => xs.filter(x => x !== null && x !== undefined);
```

在命名的时候，我们特别容易把自己限定在特定的数据上（本例中是 articles）。这种现象很常见，也是重复造轮子的一大原因。

有一点我必须得指出，你一定要非常小心 this 值，别让它反咬你一口，这一点与面向对象代码类似。如果一个底层函数使用了 this，而且是以一等公民的方式被调用的，那你就等着 JS 这个蹩脚的抽象概念发怒吧。

```js
var fs = require('fs');

// 太可怕了
fs.readFile('freaky_friday.txt', Db.save);

// 好一点点
fs.readFile('freaky_friday.txt', Db.save.bind(Db));
```

把 Db 绑定（bind）到它自己身上以后，你就可以随心所欲地调用它的原型链式垃圾代码了。this 就像一块脏尿布，我尽可能地避免使用它，因为在函数式编程中根本用不到它。然而，在使用其他的类库时，你却不得不向这个疯狂的世界低头。

也有人反驳说 this 能提高执行速度。那就看你怎么取舍了.

## References

- [闲话闭包](https://www.zhoulujun.cn/html/webfront/ECMAScript/js6/2015_0814_240.html)
  
- [函数式编程漫谈](https://cloud.tencent.com/developer/article/1190773)

- [函数式编程指北]()

- [HasKell](https://www.haskell.org/)