## 你 不知道的Javascript（上）

### 作用域是什么？

遍历嵌套作用域链的规则：引擎从当前的执行作用域开始查找变量，如果找不到，就向上一级继续查找。当抵达最外层的全局作用域时，无论找到还是没找到，查找过程都会停止。

```javascript
function foo(a) {
  console.log(a + b);
  b = a;
}
foo(2);
```

当引擎执行LHS查询时，如果在顶层（全局作用域） 中也无法找到目标变量，全局作用域中就会创建一个具有该名称的变量，并将其返还给引擎，前提是程序运行在非“严格模式”下。

在严格模式下禁止自动或隐式地创建全局变量。所以引擎会抛出如同RHS查询失败时类似的ReferenceError异常。

ReferenceError同作用域判别失败相关，而TypeError则代表作用域判别成功了，但是对结果的操作是非法或不合理的。

作用是一套规则，用于确定在何处以及如何查找变量（标志符）。如果查找的目的是对变量进行赋值，那么就会使用LHS查询，如果目的是获取变量的值，就会使用RHS查询。

赋值操作符会导致LHS查询。=操作符或调用函数时传入参数的操作都会导致关联作用域的赋值操作。

### 词法作用域

作用域查找会在找到第一个匹配的标志符时停止。

在多层的嵌套作用域中可以定义同名的标识符，这叫作“遮蔽效应”。（即，内部的标识符遮蔽了外部的标识符）。

作用域的查找始终从运行处所处的最内部作用域开始，逐级向外或者说向上进行，直到遇见第一个匹配的标识符为止。

全局变量会自动成为全局对象的属性（window），因此可以不直接通过全局对象的词法名称，而是间接地通过对全局对象属性的引用来对其进行访问。

无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。

词法作用域查找只会查找一级标识符，比如a、b和c。如果代码中引用了foo.bar.baz，词法作用域查找只会试图查找foo标识符，找到这个变量后，对象属性访问规则会分别接管对bar和baz属性的访问。

#### 欺骗词法

##### eval

在你写的代码中用程序生成代码并运行，就好像代码是写在那个位置的一样。

在执行eval之后的代码时，引擎并不“知道”或“在意”前面的代码是以动态形式插入进来，并对词法作用域的环境进行修改的。引擎只会如往常地进行词法作用域查找。

```javascript
function foo(obj) {
  with(obj) {
    a = 2;
  }
}

var o1 = {
  a: 3
}

var o2 = {
  b: 3
}

foo(o1);
console.log(o1.a); //2

foo(o2);
console.log(o2.a); //undefined
console.log(a); // 2 --- a被泄漏到全局作用域上了。
```

with可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对象的属性也会被处理为定义在这个作用域中的词法标识符。

尽管with块可以将一个对象处理为词法作用域，但是这个块内部正常的var声明并不会限制在这个块的作用域中，而是被添加到with所处的函数作用域中。

eval函数如果接受了含有一个或多个声明的代码，就会 修改其所处的词法作用域，而with声明实际上是根据你传递给它的对象凭空创建了 一个全新的词法作用域。

#### 性能

JavaScript引擎会在编译阶段进行数项的性能优化。其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。

但如果引擎在代码中发现eval或with，它只能简单地假设关于标识符位置的判断都是无效的，因为无法在词法分析阶段明确知道eval会接收到什么代码，这些代码会如何对作用域进行修改，也无法知道传递给with用来创建新词法作用域的对象的内容到底是什么。

最悲观的情况是如果出现了eval或with，所有的优化可能都是无意义的，因此最简单的做法就是完全不做任何优化。

如果代码中大量使用eval或with，那么运行起来一定会变得非常慢，无论引擎多聪明，试图将这些悲观的情况的副作用限制在最小范围内，也无法避免如果没有这些优化，代码会运行的更慢这个事实。

#### 小结

词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。编译的词法分析阶段基本能够知道全部标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。

JavaScript中有两个机制可以“欺骗”词法作用域：eval(..)和with。前者可以对一段包含一个或多个声明的“代码”字符串进行演算，并借此来修改已经存在的词法作用域(在运行时)。后者本质上是通过将一个对象的引用当作作用域来处理，将对象的属性当作作用域来处理，将对象的属性当作作用域中的标识符来处理，从而创建了一个新的词法作用域(同样是在运行时)。

这两个机制的副作用是引擎无法在编译时对作用域查找进行优化，因为引擎只能谨慎地认为这样的优化是无效的。使用这其中任何一个机制都将导致代码运行变慢，不要使用它们。 

### 函数作用域和块作用域

函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复用(事实上在嵌套的作用域中也可以使用)。

```javascript
function foo() {
  functioin bar(a) {
    i = 3; //修改for循环所属作用域中的i
    console.log(a + i)
  }
  
  for (var i = 0; i < 10; i++) {
    bar( i * 2); // 死循环了
  }
}
```

#### 函数作用域

区分函数声明和表达式最简单的方法是看function关键字出现在声明中的位置（不仅仅是一行代码，而是整个声明中的位置）。如果function是声明中的第一个词，那么就是一个函数声明，否则就是一个函数表达式。

函数声明和函数表达式之间最重要的区别是它们的名称标识符将会绑定在何处。

#### 匿名和具名

function()..没有名称标志符，因此叫作匿名函数。函数表达式可以是匿名的，而函数声明则不可以省略函数名。

##### 匿名函数的缺点

1.匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难。

2.如果没有函数名，当函数需要引用自身时只能使用已经过期的arguments.callee引用。

3.匿名函数省略了对于代码可读性/可理解性很重要的函数名。

#### 立即执行函数表达式

```javascript
var a = 2;
(function foo() {
  var a = 3;
  console.log(a); // 3
})();
console.log(a) // 2
```

由于函数被包含在一对()括号内部，因此成为了一个表达式，通过末尾加上另外一个()可以立即执行这个函数。

IIFE还有一种变化的用途是倒置代码的运行顺序，将需要运行的函数放在第二位，在IIFE执行之后当作参数传递进去。

```javascript
var a = 2;

(function IIFE(def){
  def(window)
})(function def(global) {
  var a = 3;
  console.log(a); //3
  console.log(global.a); // 2
})
```

#### 块作用域

```javascript
for (var i = 0; i < 10; i++) {
  console.log(i);
}

for (let i = 0; i < 10; i++) {
  console.log(i)
}
```

with 、 try/catch、都会创建自己的块作用域。

##### let

```javascript
var foo = true

if (foo) {
  let bar = foo * 2
  bar  = something(bar)
  console.log(bar)
}

console.log(bar) // ReferenceError


var foo = true

if (foo) {
  {
    // <-- 显示的块
    let bar = foo * 2;
    bar = something(bar);
    console.log(bar);
  }
}

console.log(bar) // ReferenceError
```

提升是指声明会被视为存在于其所出现的作用域的整个范围内。

但是使用let进行的声明不会在块作用域中进行提升。声明的代码被运行之前，声明并不存在。

优势

1.利于垃圾回收

2.let循环

#### const

```javascript
var foo = true

if (foo) {
  var a = 2;
  const b = 3; //包含在if中的块作用域常量
  
  a = 3; //正常
  b = 4; //错误
}

console.log(a); // 3
console.log(b); // ReferenceError!
```

#### 小结

块作用域指的是变量和函数不仅可以属于所处的作用域，也可以属于某个代码块（通常指 { ... }内部）

### 提升

```javascript
a = 2;
var a;
console.log(a);
```

包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。

只有声明本身会被提升，而赋值或其他运行逻辑会留在原地。

```javascript
function foo() {
  var a;
  console.log(a) // undefined
  a = 2;
}
foo()
```

```javascript
foo(); // 不是ReferenceError，而是TypeError

var foo = function bar() {
  // ...
}
```

函数声明会被提升，但是函数表达式却不会被提升。

```javascript
foo(); //TypeError
bar(); // ReferenceError

var foo = function bar() {
  //...
}
```

实际上是

```javascript
var foo

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
  var bar = ...self...
  // ...
}
```

#### 函数优先

函数声明和变量声明都会被提升。但是函数会首先被提升，然后才是变量。

```javascript
foo(); //1

var foo;

function foo() {
  console.log(1)
}

foo = function() {
  console.log(2)
}
```

引擎的理解

```javascript
function foo() {
  console.log(1);
}

foo(); // 1

foo = function() {
  console.log(2);
}
```

**失去条件控制的情况**

```javascript
foo(); // "b"

var a = true;
if (a) {
  function foo() {console.log("a");}
} else {
  function foo() {console.log("b");}
}
```

### 作用域闭包

闭包---当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

```javascript
function foo() {
  var a = 2;
  
  function bar() {
    console.log(a);
  }
  
  return bar;
}

var baz = foo();
baz(); // 2. -----闭包
```

在定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包。

```javascript
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i *1000)
}
//输出5次6

for (var i = 1; i<=5; i++) {
  (function() {
    setTimeout(function timer() {
      console.log(i)
    }, i*1000)
  })();
}
// 1 2 3 4 5

//优化
for( var i=1; i <=5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j)
    }, j*1000)
  })(i)
}
```

**模块模式需要具备两个必要条件**

1.必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。

2.封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

**模块单例**

```javascript
var foo = (function CoolModule() {
  var something = "cool"
  var another = [1,2,3]
  
  function doSomething() {
    console.log(something)
  }
  
  function doAnother() {
    console.log(another.join("!"));
  }
  
  return {
    doSomething: doSomething,
    doAnother: doAnother
  }
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3 ! 
```

通过在模块实例的内部保留对公共API对象的内部引用，可以从内部对模块实例进行修改，包括添加或删除方法和属性，以及修改他们的值。

```javascript
var foo = (function CoolModule(id) {
  function change() {
    // 修改公共API
    publicAPI.identify = identify2;
  }
  
  function identify1() {
    console.log(id);
  }
  
  function identify2() {
    console.log(id.toUpperCase());
  }
  
  var publicAPI - {
    change: change,
    identify: identify1
  };
  
  return publicAPI
})("foo module")

foo.identify(); // foo module
foo.change();
foo.identify(); // foo MODULE
```

基于函数的模块并不是一个能被稳定识别的模式（编译器无法识别），它们的API语义只有在运行时才会被考虑进来。因此可以在运行时修改一个模块的API。但是ES6模块API更加稳定（API不会在运行时改变）

ES6的模块没有“行内”格式，必须被定义在独立的文件中（一个文件一个模块）。浏览器或引擎有一个默认的“模块加载器”（可以被重载）可以在导入模块时异步地加载模块文件。

import可以将一个模块中的一个或多个API导入到当前作用域中，并分别绑定在一个变量上。export会将当前模块的一个标识符（变量、函数）导出为公共API。这些操作可以在模块定义中根据需要使用任意多次。

#### 小结

模块有两个主要特征：

（1）为创建内部作用域而调用了一个包装函数

（2）包装函数的返回值必须至少包括一个对内部函数的引用，这样就会创建涵盖整个包装函数内部作用域的闭包。

箭头函数是用当前的词法作用域覆盖了this本来的值。

### this和对象原型

### 关于this

#### 为什么要使用this ？

随着使用模式越来越复杂，显式传递上下文对象会让代码变得越来越混乱，使用this则不会这样。

```javascript
//经典错误
function foo() {
    var a = 2;
    this.bar();
}

function bar() {
    console.log( this.a );
}

foo()
```

this是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this就是记录的其中一个属性，会在函数执行的过程中用到。

### this全面解析

在理解this的绑定过程之前，首先要理解调用位置。

调用位置就是函数在代码中被调用的位置(而不是声明的位置)。

#### 绑定规则

##### 1.默认绑定

this会指向全局对象

在严格模式下，全局对象将无法使用默认绑定，因此this会绑定到undefined

##### 2.隐式绑定

```javascript
function foo() {
  console.log(this.a)
}

var obj2 = {
  a: 42,
  foo: foo
};

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo(); //42
```

###### 隐式丢失

一个最常见的this绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把this绑定到全局对象或undefined上，取决于是否是严格模式。

```javascript
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

var bar = obj.foo //函数别名！

var a = "oops, global" // a是全局对象的属性

bar(); // "oops, global"
```

```javascript
function foo() {
  console.log(this.a)
}

function doFoo(fn) {
  fn()
}

var obj = {
  a: 2,
  foo: foo
}

var a = "oops, global"

doFoo(obj.foo) // "oops, global"
```

参数传递其实就是一种隐式赋值。

##### 3.显示绑定

通过call，apply，bind

如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作this的绑定对象，这个原始值会被转换成它的对象形式（也就是new String(...)、 new Boolean(...) 或者 new Number(...)）。这通常被称为“装箱”。

###### (1) 硬绑定

```javascript
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2
}

var bar = function() {
  foo.call(obj)
}

bar(); // 2
setTimeout(bar, 100) // 2

// 硬绑定的bar不可能再修改它的this
bar.call(window) // 2
```

###### (2) API调用的“上下文”

```javascript
function foo(el) {
  console.log(el, this.id);
}

var obj = {
  id: "awesome"
}

// 调用foo(...)时把this绑定到obj
[1, 2, 3].forEach( foo, obj )
// 1 awesome 2 awesome 3 awesome
```

这些函数实际上就是通过call(...)或者apply(...)实现了显示绑定。

#### new绑定

在JavaScript中，构造函数只是一些使用 new操作符时被调用的函数。它们并不会属于某个类，也不会实例化某个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被new操作符调用的普通函数而已。

使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1.创建（或者说构造)一个全新的对象。

2.这个新对象会被执行[[原型]]连接。

3.这个新对象会绑定到函数调用的this。

4.如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

#### 优先级

 默认绑定的优先级是四条规则中最低的。

```javascript
// 测试隐式绑定和显式绑定哪个优先级更高
function foo() {
  console.log(this.a)
}

var obj1 = {
  a: 2,
  foo: foo
}

var obj2 = {
  a: 3,
  foo: foo
}

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call(obj2) // 3
obj2.foo.call(obj1) // 2
```

显示绑定优先级更高，也就是说在判断时应当考虑是否可以应用显示绑定。

```javascript
// new绑定和隐式绑定的优先级哪个更高

function foo(something) {
  this.a = something;
}

var obj1 = {
  foo: foo
}

var obj2 = {}

obj1.foo(2);
console.log(obj1.a); // 2

obj1.foo.call(obj2, 3); // 3

var bar = new obj1.foo(4);
console.log(obj1.a) // 2
console.log(bar.a) // 4
```

new绑定比隐式绑定优先级更高。

new和call/apply 无法一起使用，因此无法通过new foo.call(obj1)来直接进行测试。但是我们可以使用硬绑定来测试它俩的优先级。

```javascript
function foo(something) {
  this.a = something;
}

var obj1 = {}

var bar = foo.bind(obj1)
bar(2)
console.log(obj1.a)  // 2

var baz = new bar(3)
console.log(obj1.a) // 2
console.log(baz.a)  // 3
```

bar被硬绑定到obj1上，但是new bar(3)并没有像我们预计的那样把obj1.a修改为3。相反，new修改了硬绑定（到obj1的）调用bar(..)中的this。因为使用了new绑定，我们得到了一个名字为baz的新对象，并且baz.a的值是3。

#### 手写bind

```javascript
function bind(fn, obj) {
  return function() {
    fn.apply(obj, arguments);
  }
}

if(!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== "function") {
      // 与 ECMAScript 5 最接近的
      // 内部 IsCallable 函数
      throw new TypeError(
        "Function.prototype.bind - what is trying" +
        "to be bound is not callable"
      );
    }
    
    var aArgs = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP = function(){},
        fBound = function() {
          return fToBind.apply(
            (
              this instanceof fNOP &&
              oThis ? this : oThis
            ),
            aArgs.concat(
              Array.prototype.slice.call(arguments)
            );
          )
        };
    fNOP.prototype = this.prototype
    fBound.prototype = new fNOP()
    
    return fBound
  }
}
```

之所以要在new中使用硬绑定函数，主要目的是预先设置函数的一些参数，这样在使用new进行初始化时就可以只传入其余参数。bind的功能之一就是可以把除了第一个参数之外的其他参数都传给下层的函数（柯里化的一种）。

```javascript
function foo(p1,p2) {
	this.val = p1 + p2
}

// 之所以使用null是因为在本例中我们并不关心硬绑定的this是什么
// 反正使用new时this会被修改

var bar = foo.bind(null, "p1");

var baz = new bar("p2");

baz.val; // p1p2
```

#### 判断this

1.函数是否在new中调用（new绑定）？如果是的话this绑定的是新创建的对象。

```javascript
var bar = new foo()
```

2.函数是否通过call、apply（显式绑定）或者硬绑定调用？如果是的话，this绑定的是指定的对象。

```javascript
var bar = foo.call(obj2)
```

3.函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。

```javascript
var bar = obj1.foo()
```

4.如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。

```javascript
var bar = foo()
```

#### 被忽略的this

如果你把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。

```javascript
function foo() {
  console.log(this.a)
}

var a = 2;

foo.call(null) // 2
```

#### 更安全的this

创建一个空的非委托的对象。DMZ

Object.create(null)和{}很像，但是并不会创建Object.prototype这个委托，所以它比{}"更空"。

```javascript
function foo(a, b) {
  console.log("a:" + a + "", ",b:" + b);
}

// 我们的DMZ空对象
var kong = Object.create(null);

// 把数组展开成参数
foo.apply(kong, [2, 3]);

// 使用bind(...)进行柯里化
var bar = foo.bind(kong, 2);
bar(3); // a: 2, b: 3
```

#### 间接引用

```javascript
function foo() {
  console.log(this.a)
}

var a = 2
var o = { a: 3, foo: foo }
var p = { a: 4 }

o.foo() // 3
(p.foo = o.foo)() // 2
```

注意：对于默认绑定来说，决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this会被绑定到undefined，否则this会被绑定到全局对象。

#### 软绑定

```javascript
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this;
    // 捕获所有curried参数
    var curried = [].slice.call(arguments, 1);
    var bound = function() {
      return fn.apply(
        (!this || this === (window || global)) ? 
        	obj : this,
        curried.concat.apply(curried, arguments)
      )
    }
    bound.prototype = Object.create(fn.prototype)
    return bound
  }
}
```

#### this词法

##### 箭头函数

```javascript
function foo() {
  return (a) => {
    console.log(this.a)
  }
}

var obj1 = {
  a: 2
}

var obj2 = {
  a: 3
}

var bar = foo.call(obj1)
bar.call(obj2) // 2
```

foo()内部创建的箭头函数会捕获调用时foo()的this绑定到obj1，bar的this也会绑定到obj1，箭头函数的绑定无法被修改。(new也不行！)

#### 小结

判断this绑定的对象的四条规则

1.由new调用？绑定到新创建的对象

2.由call或者apply（或者bind）调用？绑定到指定的对象

3.由上下文对象调用？绑定到那个上下文对象。

4.默认：在严格模式下绑定到undefined，否则绑定到全局对象。

### 对象

在对象属性名中数字会被转换成字符串。

##### 深拷贝的解决方案

1.在JSON安全的情况下

```javascript
var newObj = JSON.parse(JSON.stringify(someObj))
```

##### 浅拷贝的解决方案

```javascript
var newObj = Object.assign({}, myObject);

newObj.a // 2
newObj.b === anotherObject // true
newObj.c === anotherArray // true
newObj.d === anotherFunction // true
```

由于Object.assign(..)就是使用=操作符来赋值，所以源对象属性的一些特性(比如writable)不会被复制到目标对象。

##### 属性描述符

getOwnPropertyDescriptor()

defineProperty()

###### 1.Writable

决定是否可以修改属性的值。

###### 2.Configurable

只要属性是可配置的，就可以使用defineProperty(..)方法来修改属性描述符。

注意：即便属性是configurable:false，我们还是可以把writable的状态由true改为false，但是无法由false改为true。除了无法修改，configurable:false还会禁止删除这个属性。

###### 3.Enumerable

如果你不希望某些特殊属性出现在枚举中，那就把它设置成enumerable:false

#### 不变性 

1.对象常量

writable:false 和 configurable: false 可以创建一个真正的常量属性。

2.禁止扩展

Object.prevent.Extensions(..)

3.密封

Object.seal(..)

这个方法实际上会在一个现有对象上调用Object.preventExtensions(..)并把所有现有属性标记为configuable:false

所以密封后不仅不能添加新属性，也不能重新配置或者删除任何现有属性（虽然可以修改属性的值）

4.冻结

Object.freeze(..)会创建一个冻结对象，这个方法实际上会在一个现有对象上调用Object.seal(..)并把所有“数据访问”属性标记为writable: false，这样就无法修改它们的值。

#### GET

#### PUT

如果已存在这个属性，PUT算法大致会检查下面的内容

1.属性是否是访问描述符？如果是并且存在setter就调用setter

2.属性的数据描述符中writable是否是false？如果是，在非严格模式下静默失败，在严格模式下抛出TypeError异常

3.如果都不是，将该值设置为属性的值

如果不存在这个属性，PUT操作更复杂

#### Getter和Setter函数

set和get函数

#### 存在性

in操作符会检查属性是否在对象及其[[Prototype]]原型链中。

hasOwnProperty()只会检查属性是否存在myObject对象中，不会检查原型链。

注意：在数组上应用for...in循环有时会产生出人意料的结果，因为这种枚举不仅 会包含所有数值索引，还会包含所有可枚举属性。最好只在对象上应用for..in循环，如果要遍历数组就使用传统的for循环来遍历数值索引。

propertyIsEnumerable()会检查给定的属性名是否直接存在于对象中（而不是在原型链上）并且满足enumerable:true。

Object.keys(..)会返回一个数组，包含所有可枚举属性，Object.getOwnPropertyNames(..)会返回一个数组，包含所有属性，无论它们是否可枚举。

in 和 hasOwnProperty(..)的区别在于是否查找prototype链，然而，Object.keys(..)和Object.getOwnPropertyNames(..)都只会查找对象直接包含的属性。

#### 遍历

遍历数组下标时采用的是数字顺序(for 循环或者其他迭代器)，但是遍历对象属性时的顺序是不确定的，在不同的JavaScript引擎中可能不一样。

for...of循环首先会向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的next()方法来遍历所有返回值。

为任意对象定义@@iterator

```javascript
var myObject = {
  a: 2,
  v: 3
}

Object.defineProperty(myObject, Symbol.iterator, {
  enumerable: false,
  writable: false,
  configurable: true,
  value: function() {
    var o = this
    var idx = 0
    var ks = Object.keys(0)
    return {
      next: function() {
        return {
          value: o[ks[idx++]],
          done: (idx > ks.length)
        };
      }
    };
  }
});

// 手动遍历myObject
var it = myObjectp[Symbol.iterator]()
it.next(); // {value: 2, done: false}
it.next(); // {value: 3, done: false}
it.next(); // {value: undefined, done: true}

// 用for..of遍历myObject
for(var v of myObject) {
  console.log(v);
}

// 2
// 3
```

### 混合对象”类“

##### 显示混入

```javascript
function mixin(sourceObj, targetObj) {
  for(var key in sourceObj) {
    // 只会在不存在的情况下复制
    if (!(key in targetObj)) {
      targetObj[key] = sourceObj[key]
    }
  }
  return targetObj;
}

var Vehicle = {
  engines: 1,
  
  ignition: function() {
    console.log("Turning on my engine.")
  },
  
  drive: function() {
    this.ignition()
    console.log("Steering and moving forward")
  }  
}

var Car = mixin(Vehicle, {
  wheels: 4,
  
  drive: function() {
    Vehicle.drive.call(this)
    console.log(
    	"Rolling on all " + this.wheels + " wheels!"
    )
  }
})
```

##### 混合复制

```javascript
function mixin(sourceObj, targetObj) {
  
  for (var key in sourceObj) {
    targetObj[key] = sourceObj[key]
  }
  
  return targetObj
}

var Vehicle = {
   // ...
};


// 首先创建一个空对象并把Vehicle的内容复制进去
var Car = mixin(Vehicle, {});

// 然后把新内容复制到Car中
mixin({
  wheels: 4,
  
  drive: function() {
     // ...
  }
}, Car)
```

JavaScript中的函数无法（用标准、可靠的方法）真正的复制，所以你只能复制对共享函数对象的引用。

##### 寄生继承

```javascript
// 传统的JavaScript类 Vehicle
function Vehicle() {
  this.engines = 1;
}

Vehicle.prototype.ignition = function () {
  console.log("Turning on myengine.");
}

Vehicle.prototype.drive = function() {
  this.ignition();
  console.log("steering and moving forward!");
}

// "寄生类" Car
function Car() {
  // 首先, car 是一个Vehicle
  var car = new Vehicle();
  
  // 接着我们对car进行定制
  car.wheels = 4;
  
  // 保存到Vehicle::drive()的特殊引用
  var vehDrive = car.drive;
  
  // 重写Vehicle::drive()
  car.drive = function() {
    vehDrive.call(this)
    console.log(
    	"Rolling on all" + this.wheels + " wheels"
    )
  }
  return car;
}
```

##### 隐式混入

```javascript
var Something = {
  cool: function () {
    this.greeting = "Hello World";
    this.count = this.count ? this.count + 1 : 1;
  }
}

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
  cool: function () {
    // 隐式把Something混入Another
    Something.cool.call(this)
  }
}

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1(非共享状态)
```

### 原型

##### Prototype 原型链

##### Object.prototype 原型链的尽头

##### 属性设置和屏蔽

如果属性名foo既出现在myObject中也出现在myObject的prototype链上层，那么就会发生屏蔽。myObject中包含的foo属性会屏蔽原型链上层的所有foo属性，因为myObject.foo总是会选择原型链中最底层的foo属性。

如果foo不直接存在于myObject中而是存在于原型链上层时 myObject.foo = "bar"会出现三种情况。

1.如果在prototype链上层存在名为foo的普通数据访问属性并且没有被标记为只读，那就会直接在myObject中添加一个名为foo的新属性，它是屏蔽属性。

2.如果在prototype链上层存在foo，但是它被标记为只读，那么无法修改已有属性或者myObject上创建屏蔽属性。如果在严格模式下，代码会抛出一个错误。否则 ，这条赋值语句会被忽略。总之，不会发生屏蔽。

3.如果在prototype链上层存在foo并且它是一个setter，那就一定会调用这个setter。foo不会被添加到myObject，也不会重新定义foo这个setter。

```javascript
var anotherObject = {
  a: 2
}

var myObject = Object.create(anotherObject)

anotherObject.a  // 2
myObject.a // 2

anotherObject.hasOwnProperty("a") // true
myObject.hasOwnProperty("a") // false

myObject.a++ //隐式屏蔽

anotherObject.a // 2
myObject.a // 3

myObject.hasOwnProperty("a") //true
```

#### "类"函数

所有的函数默认都会拥有一个名为prototype的公有并且不可枚举的属性，它会指向另一个对象。

```javascript
function NothingSpecial() {
  console.log("Don't mind me!")
}

var a = new NothingSpecail();
// "Don't mind me!"

a // {}
```

Foo.prototype 的.constructor 属性只是Foo函数在声明时的默认属性。如果你创建了一个新对象并替换了函数默认的.prototype对象引用，那么新对象并不会自动获得.constructor属性。

```javascript
function Foo() {/* .. */}

Foo.prototype = {/* .. */} // 创建一个新原型对象

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

Object.create(null)会创建一个拥有空（或者说null）[[Prototype]]链接的对象，这个对象无法进行委托。由于这个对象没有原型链，所以instanceof操作符无法进行判断，因此总是会返回false。

```javascript
// Object.create()的polyfill代码
if (!Object.create) {
  Object.create = function(o) {
    function F(){}
    F.prototype = o
    return new F()
  }
}
```

### 行为委托

```javascript
Task = {
  setID: function(ID) { this.id = ID; },
  outputID: function() { console.log(this.id) }
}

// 让XYZ委托Task
XYZ = Object.create(Task)
XYZ.prepareTask = function(ID, Label) {
  this.setID(ID);
  this.label = Label
}

XYZ.outputTaskDetails = function() {
  this.outputID();
  console.log(this.label)
}
```

### Class的陷阱

class基本上是现有prototype委托机制的一种语法糖。

class并不会像传统面向类的语言一样在声明时静态复制所有行为。如果你修改或者替换了父类中的一个方法，那子类和所有实例都会受到影响，因为他们在定义时并没有进行复制 ，只是使用基于prototype的实时委托。

class语法无法定义类成员属性（只能定义方法），如果为了跟踪实例之间共享状态必须要这么做，只能使用.prototype语法。

```javascript 
// 意外屏蔽
class C {
  constructor(id) {
    // id属性屏蔽了id()方法
    this.id = id
  }
  id() {
    console.log("Id: " + id)
  }
}

var c1 = new C("c1")
c1.id(); // TypeError -- c1.id现在是字符串"c1"
```

















