# 你不知道的JavaScript（中）

## 类型

### 内置类型

1.空值 null

2.未定义 undefined

3.布尔值 boolean

4.数字 number

5.字符串 string

6.对象 object

7.符号 symbol

```javascript
typeof null === "object";  // true

// 修复
(!a && typeof a === "object"); // true
```

typeof运算符总是会返回一个字符串

typeof的安全防范机制检查undeclared变量

```javascript
if (typeof DEBUG !== 'undefined') {
  console.log("Debugging is starting");
}
```

## 值

使用delete运算符可以将单元从数组中删除，但是单元删除后，数组的length属性并不会发生变化。

可以通过indexOf(..)、concat(..)、forEach(..)、Array.form(..)等工具函数将类数组（一组通过数字索引的值）转换为真正的数组。

slice()方法返回一个数组副本

```javascript
0.1 + 0.2 === 0.3 // false

//polyfill 写法
if (!Number.EPSILON) {
  Number.EPSILON = Math.pow(2, -52)
}

function numbersCloseEnoughToEqual(n1, n2) {
  return Math.abs(n1 - n2) < Number.EPSILON
}

var a = 0.1 + 0.2
var b = 0.3

numbersCloseEnoughToEqual(a, b)  // true
numbersCloseEnoughToEqual(0.0000001, 0.0000002) // false
```

#### 整数的安全范围

Number.MAX_SAFE_INTEGER

Number.MIN_SAFE_INTEGER

#### 整数检测

Number.isInteger(..)

```javascript
if (!Number.isInteger) {
  Number.isInteger = function(num) {
    return typeof num == 'number' && num % 1 == 0
  }
}
```

#### 安全整数检测

Number.isSafeInteger(..)

```javascript
if (!Number.isSafeInteger) {
  Number.isSafeInteger = function(num) {
    return Number.isInteger(num) &&
      				Math.abs(num) <= Number.MAX_SAFE_INTEGER
  }
}
```

#### 32位有符号整数

a | 0  可以将变量a中的数值转换为32位有符号整数，因为数位运算符|只适用于32位整数（它只关心32位以内的值，其他的数位将被忽略）。因此与0进行操作即可截取a中的32位数位。

某些特殊的值并不是32位安全范围的，如NaN和Infinity，此时会对它们执行虚拟操作ToInt32，以便转换为符合数位运算符要求的+0值。

#### 特殊值

null指空值

undefined指没有值

或者

undefined指从未赋值

null指曾赋过值，但是目前没有值

void并不改变表达式的结果，只是让表达式不返回值。

#### 特殊的数字

不是数字的数字NaN，判断方法isNaN(..)

```javascript
// polyfill
if (!Number.isNaN) {
  Number.isNaN = function(n) {
    return (
      typeof n === "number" &&
      window.isNaN(n)
    )
  }
}

var a = 2 / "foo"
var b = "foo"
Number.isNaN(a); // true
Number.isNaN(b); // false----好！

if(!Number.isNaN) {
  Number.isNaN = function(n) {
    return n !== n
  }
}
```

#### 无穷数

```javascript
var a = 1 / 0 // Infinity
var b = -1 / 0 // -Infinity
```

#### 零值

```javascript
var a = 0 / -3;

// 至少在某些浏览器的控制台中显示是正确的
a; // -0

// 但是规范定义的返回结果是这样！
a.toString();  // "0"
a + ""; // "0"
String(a); // "0"

// JSON也如此，很奇怪
JSON.stringify(a) // "0"

-0 === 0 // true
0 > -0 // false

function isNegZero(n) {
  b = Number(n);
  return (n === 0) && (1 / n === -Infinity);
}

isNegZero(-0);  // true
isNegZero(0 / -3); // true
isNegZero(0); // false
```

#### 特殊等式

```javascript
var a = 2 / "foo"
var b = -3 * 0
Object.is(a, NaN)  // true
Object.is(b, -0)  //true
Object.is(b, 0) // false

if (!Object.is) {
  Object.is = function (v1, v2) {
    // 判断是否是-0
    if (v1 === 0 && v2 === 0) {
      return 1 / v1 === 1 / v2
    }
    // 判断是否是NaN
    if (v1 !== v1) {
      return v2 !== v2
    }
    // 其他情况
    return v1 === v2
  }
}
```

#### 值和引用

```javascript
function foo(x) {
  x.push(4);
  x;
  
  x = [4,5,6];
  x.push(7);
  x;
}

var a = [1,2,3];

foo(a);

a; // [1,2,3,4]


// 另一种
function foo(x) {
  x.push(4);
  x; // [1,2,3,4]
  
  //然后
  x.length = 0; // 清空数组
  x.push(4,5,6,7);
  x; // [4,5,6,7]
}

var a = [1,2,3];

foo(a);

a; // 是[4,5,6,7],不是[1,2,3,4]

function foo(x) {
  x = x + 1
  x; // 3
}

var a = 2;
var b = new Number(a); // Object(a)也一样

foo(b)
console.log(b) // 是2，不是3
```

## 原生函数

常用的原生函数：

String()

Number()

Boolean()

Array()

Object()

Function()

RegExp()

Date()

Error()

Symbol()

#### 封装对象释疑

```javascript
var a = new Boolean(false);

if (!a) {
  console.log("Oops") //执行不到这里
}

var a = "abc"
var b = new String(a)
var c = Object(a)

typeof a; // string
typeof b; // object
typeof c; // object

b instanceof String // true
c instanceof String // true

Object.prototype.toString.call(b) // "[object String]"
Object.prototype.toString.call(c) // "[object String]"

var a = new String("abc")
var b = new Number(42)
var c = new Boolean(true)

a.valueOf() // "abc"
b.valueOf() // 42
c.valueOf() // true

var a = new String("abc")
var b = a + ""; // b的值为“abc”

typeof a; // "object"
typeof b; // "string"
```

#### 原生函数作为构造函数

```javascript
var a  = new Array( 1, 2, 3 );
a; // [1, 2, 3]

var b = [1, 2, 3]
b; // [1, 2, 3]
```

构造函数Array(..)不要求必须带new关键字。不带时，它会被自动补上。

Array构造函数只带一个数字参数的时候，该参数会被作为数组的预设长度，而非只充当数组中的一个元素。（只不过它的length被设置为指定值）

```javascript
var a = new Array(3)
var b = [undefined, undefined, undefined];
var c = []
c.length = 3

a // [undefined x 3]
b // [undefined, undefined, undefined]
c // [undefined x 3]

a.join("-") // "- -"
b.join("-") // "- -"

a.map(function(v, i) { return i; }) // [undefined x 3]
b.map(function(v, i) { return i; }) // [0, 1, 2]

function fakeJoin(arr, connector) {
  var str = ""
  for (var i = 0; i < arr.length; i++) {
    if (i > 0) {
      str += connector
    }
    if (arr[i] !== undefined) {
      str += arr[i]
    }
  }
  return str
}

var a = new Array(3)
fakeJoin(a, "-") // "- -"
```

Date(..)主要用来获得当前的Unix时间戳（从1970年1月1日开始计算，以秒为单位）。

```javascript
if (!Date.now) {
  Date.now = function() {
    return (new Date()).getTime()
  }
}
```

Symbol(..)

```javascript
var mysym = Symbol("my own symbol");
mysym; // Symbol(my own symbol)
mysym.toString(); // "Symbol(my own symbol)"
typeof mysym; // "symbol"

var a = { };
a[mysym] = "foobar"

Object.getOwnPropertySymbols( a ); // [ Symbol(my own symbol) ]
```

#### 原生原型

String#indexof(..)

在字符串中找到指定子字符串的位置

String#charAt(..)

获得字符串指定位置上的字符

String#substr(..)、String#substring(..)和String#slice(..)

获得字符串的指定部分

String#toUpperCase()和String#toLowerCase()

将字符串转换为大写或小写

String#trim()

去掉字符串前后的空格，返回新的字符串

以上方法并不改变原字符串的值，而是返回一个新字符串

## 强制类型转换

javascript中的强制类型转换总是返回标量基本类型值，如字符串，数字和布尔值，不会返回对象和函数。

类型转换发生在静态类型语言的编译阶段，而强制类型转换则发生在动态类型语言的运行时。

```javascript
var a = 42;

var b = a + "";  // 隐式强制类型转换

var c = String(a);  // 显式强制类型转换
```

对普通对象来说，除非自行定义，否则toString() （Object.prototype.toString()）返回内部属性[[Class]]的值，如"[object Object]"

不安全的JSON值，undefined、function、symbol和包含循环引用（对象之间相互引用，形成一个无限循环）的对象都不符合JSON结构标准。

JSON.stringify(..)在对象中遇到undefined、function和symbol时会自动将其忽略，在数组中则会返回null（以保证单元位置不变）。

```javascript
JSON.stringify(undefined);  // undefined
JSON.stringify( function(){}); // undefined

JSON.stringify(
  [1, undefined, function(){}, 4]
); // "[1, null, null, 4]"

JSON.stringify(
	{ a: 2, b: function(){}}
); // "{"a": 2}"

var a =  {
  val: [1,2,3],
  toJSON: function() {
    return this.val.slice(1);
  }
}

var b = {
  val: [1,2,3],
  // 在字符串化前调用
  toJSON: function() {
    return "["+
      this.val.slice(1).join()+
      "]"
  }
}
// 这里第二个函数是对toJSON返回的字符串做字符串化，而非数组本身
JSON.stringify(a); // "[2, 3]"
JSON.stringify(b); // ""[2, 3]""
```

JSON.stringify(..)可选参数replacer

如果为数组，那么必须是一个字符串数组，其中包含序列化要处理的对象的属性名称，除此之外其他的属性则被忽略。

如果replacer是一个函数，它会对对象本身调用一次，然后对对象中的每个属性各调用一次，每次传递两个参数，键和值。如果要忽略某个键就返回undefined，否则返回指定的值。

```javascript
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3]
}

JSON.stringify(a, ["b", "c"]); // "{"b": 42, "c": 42}"

JSON.stringify(a, function(k, v) {
  if (k !== "c") return v;
});
//"{"b": 42, "d": [1, 2, 3]}"
```

JSON.stringify(..)可选参数space

用以指定输出的缩进格式。space为正整数时是指定每一级缩进的字符数。

```javascript
Number(true)  // 1
Number(false) // 0
Number(undefined) // NaN
Number(null) // 0
```

对象（包含数组）会首先被转换为相应的基本类型值，如果返回的是非数字的基本类型值，那么再强制转换为数字。

为了将值转换为相应的基本类型值，抽象操作ToPrimitive会首先检查该值是否有valueOf()方法。如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就使用toString的返回值来进行强制类型转换。

从ES5开始，使用Object.create(null)创建的对象[[Prototype]]属性为null，并且没有valueOf()和toString()方法，因此无法进行强制类型转换。

```javascript
var a = {
  valueOf: function() {
    return "42"
  }
}

var b = {
  toString: function() {
    return "42"
  }
}

var c = [4, 2]
c.toString() = function() {
  return this.join( "" ); // "42"
}

Number(a)  // 42
Number(b)  // 42
Number(c)  // 42
Number( "" ) // 0
Number( [] ) // 0
Number( ["abc"] )  // NaN
```

```javascript
// 假值
undefined
null
false
+0、-0和NaN
""
```

### 显式强制类型转换

日期显式转换为数字

```javascript
var d = new Date("Mon, 18 Aug 2014 08:53:06 CDT")

+d; // 1408369986000
```

获得时间戳的方法

```javascript
var timestamp = +new Date();

// 亦可 因为构造函数没有参数时可以不用带()
var timestamp = +new Date

// 不使用强制类型转化
var timestamp = new Date().getTime()
// var timestamp = (new Date()).getTime()
// var timestamp = (new Date).getTime()

// ES5中的新方法
var timestamp = Date.now()
// ES5中的polyfill
if (!Date.now) {
  Date.now = function() {
    return +new Date();
  }
}
```

~操作符，它首先将值强制类型转换为32位数字，然后执行字位操作“非”（对每一个字位进行反转）。

!操作符，将值强制类型转换位布尔值，还对其做字位反转。

```javascript
var a = "Hello World";

~a.indexOf("lo"); // -4 <--真值

if (~a.indexOf("lo")) { // true
  // 找到匹配！
}

~a.indexOf("ol") // 0 <--假值
!~a.indexOf("ol") // true

if (!~a.indexOf("ol")) { //true
  // 没有找到匹配！
}
```

～～只适用于32位数字，可以将值截除为一个32位整数，x | 0 也可以。

```javascript
Math.floor(-49.6) // -50
~~49.6 // -49
```

显式解析数字字符串

```javascript
var a = "42"
var b = "42px"

Number(a) // 42
parseInt(a) // 42

Number(b) // NaN
parseInt(b) // 42
```

parseInt处理日期的会出现问题，如果第一个字符是x或者X时，会按照十六进制进行处理，而第一个字符是0的时候会按照八进制进行处理。

因此需要第二个参数辅助进行处理，设置进制。

```javascript
parseInt(0.000008);  // 0 (来自于0.000008)
parseInt(0.0000008); // 8 （来自于“8e-7”）
parseInt(false, 16); // 250 (“fa”来自于false)
parseInt(parseInt, 16); // 15 ("f"来自于"function..")
parseInt("0x10"); // 16
parseInt("103", 2); // 2
```

布尔值的强制类型转换：Boolean(..) 和 !!

### 隐式强制类型转换

隐式的强制类型转换的作用是减少冗余，让代码更简洁。

```javascript
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42

var a = [1, 2];
var b = [3, 4];

a + b; // "1,23,4"
```

如果+的其中一个操作数是字符串，则执行字符串拼接，否则执行数字加法。

```javascript
var a = {
  valueOf: function() {return 42;},
  toString: function() {return 4;}
}

// ToPrimitive抽象操作规则，会对a调用valueOf()方法，然后通过ToString抽象操作将返回值转换为字符串。
a + "";  // "42"

// 直接调用ToString()
String( a ); // "4"
```

使用操作符- 、 * 、 /  可以将字符串强制转换为数字

```javascript
function onlyOne() {
  var sum = 0
  for (var i = 0; i < arguments.length; i++) {
    if (arguments[i]) {
      sum += arguments[i];
    }
  }
  return sum == 1
}

var a = true;
var b = false;

onlyOne(b, a) // true
onlyOne(b, a, b, b, b, b) // true

// 显式做法
function onlyOne() {
  var sum = 0;
  for (var i = 0; i < arguments.length; i++) {
    sum += Number(!!arguments[i])
  }
  return sum === 1
}
```

####  || 与 &&

```javascript
var a = 42
var b = "abc"
var c = null

a || b  // 42
a && b  // "abc"

c || b  // "abc"
c && b  // null
```

||和&&首先会对第一个操作数执行判断条件，如果其不是布尔值就先进行ToBoolean强制类型转换，然后再执行条件判断。

对于 || 来说，如果条件判断结果为true就返回第一个操作数的值，如果为false则返回第二个操作数的值。

而&&则相反，如果条件判断结果为true则返回第二个操作数的值，如果为false则返回第一个操作数的值。

```javascript
a || b
// 大致相当于
a ? a : b

a && b
// 大致相当于
a ? b : a
```

但是|| 与  && 操作数只执行一次，然而三目则会执行2次

```javascript
function foo(a, b) {
  //空值合并运算符
  a = a || "hello"
  b = b || "world"
  
  console.log(a + " " + b);  
}

foo();  // "hello world"
foo("yeah", "yeah!"); // "yeah yeah!"
// 注意Bug
foo("That's it!", ""); // "That's it! world"
```

```javascript
function foo() {
  console.log(a);
}

var a = 42;
// 守护运算符 短路机制
a && foo(); //42
```

ES6允许从符号到字符串的显式强制类型转换，然而隐式强制类型转换会发生错误。

```javascript
var s1 = Symbol("cool");
String(s1); //  "Symbol(cool)"

var s2 = Symbol("not cool");
s2 + "" // TypeError
```

符号不能强制转换成数字，但是可以强制被转换为布尔值，不管显式还是隐式都是true。

#### 宽松相等和严格相等

==允许在相等比较中进行强制类型转换，而===不允许。

==和===都会检查操作数的类型，区别在于操作数类型不同时它们的处理方式不同。

在比较两个对象的时候，==和===的工作原理是一样的。

##### ==的规范

（1）如果Type(x)是数字，Type(y)是字符串，则返回 x == ToNumber(y)的结果。

（2）如果Type(x)是字符串，Type(y)是数字，则返回 ToNumber(x) == y的结果。

```javascript
var a = "42"
var b = true

a == b; // false
```

（1）如果Type(x)是布尔类型，则返回ToNumber(x) == y的结果。

（2）如果Type(x)是布尔类型，则返回  x == ToNumber(y)的结果。

```javascript
var a = null;
var b;

a == b;  // true
a == null; // true
b == null; // true

a == false;  // false
b == false;  // false
a == ""  // false
b == ""  // false
a == 0   // false
b == 0   // false
```

```javascript
var a = 42;
var b = [42];

a == b; // true
```

（1）如果Type(x)是字符串或数字，Type(y)是对象，则返回 x == ToPrimitive(y)的结果

（2）如果Type(x)是对象，Type(y)是字符串或数字，则返回ToPrimitive(x) == y 的结果

布尔值会先被强制类型转换为数字

```javascript
var a = "abc"
var b = Object(a) // 和new String(a)一样

a === b // false
a == b // true

var a = null
var b = Object(a)
a == b; // false

var c = undefined
var d = Object(c)
c == d // false

var e = NaN
var f = Object(e)
e == f // false
```

因为没有对应的封装对象，所以null和undefined不能够被封装，Object(null)和Object()均返回一个常规对象。

NaN能够被封装为数字封装对象，但是拆封后NaN == NaN返回false。

##### 安全运用隐式强制类型转换

如果两边的值中有true或者false，千万不要使用==。

如果两边的值中有[]、“”或者0，尽量不要用==。

#### 抽象关系的比较

如果比较双方都是字符串，则按字母顺序来进行比较

```javascript
var a = { b: 42 }
var b = { b: 43 }

a < b // false
a == b // false
a > b // false

a <= b // true
a >= b // true
```

## 语法

```javascript
var a = 42
var b = a++

a; // 43
b; // 42

var a = 42, b;
b = (a++, a);

a; // 43
b; // 43
```

```javascript
{
  foo: for(var i = 0; i < 4; i++) {
    for(var j = 0; j < 4; j++) {
      // 如果j和i相等，继续外层循环
      if (j == i) {
        // 跳转到foo的下一个循环
        continue foo;
      }
      
      // 跳过奇数结果
      if ((j*i) % 2 == 1) {
        // 继续内层循环（没有标签的）
        continue;
      }
      
      console.log(i, j);
    }
  }
}
// 1 0
// 2 0
// 2 1
// 3 0
// 3 2

{
  foo: for(var i = 0; i < 4; i++) {
    for (var j = 0; j < 4; j++) {
      if ((i * j) >= 3) {
        console.log("stopping!", i, j);
        break foo;
      }
      console.log(i, j)
    }
  }
}
// 0 0
// 0 1
// 0 2
// 0 3
// 1 0
// 1 1
// 1 2
// 停止! 1 3

[] + {}; // "[object Object]"
{} + []; // 0
```

## 事件循环

JavaScript程序总是至少分为两个块：第一块现在运行，下一块将来运行，以响应某个事件。尽管程序是一块一块执行的，但是所有这些块共享对程序作用域和状态的访问，所以对状态的修改都是在之前积累的修改上进行的。

一旦有事件需要运行，事件循环就会运行，直到队列清空。事件循环的每一轮称为一个tick。用户交互、IO和定时器会向事件队列中加入事件。任意时刻，一次只能从队列中处理一个事件。执行事件的时候，可能直接或间接地引发一个或多个后续事件。

并发是指两个或多个事件链随时间发展交替执行，以至于从更高的层次来看，就像是同时在运行。

## 回调

回调的问题

1.大脑对于事情的计划方式是线性的、阻塞的、单线程的语义，但是回调表达异步流程的方式是非线性的、非顺序的。

2.回调会受到控制反转的影响，因为回调暗中把控制权交给第三方来调用你代码中的continuation。

## Promise

从外部看，由于Promise封装了依赖于时间的状态——等待底层值的完成或拒绝，所以Promise本身是与时间无关的。因此，Promise可以按照可预测的方式组成（组合），而不用关系时序或底层的结果。

另外，一旦Promise决议，它就永远保持在这个状态。此时它就成为了不变值，可以根据需求多次查看。

Promise决议后就是外部不可变的值，我们可以安全地把这个值传递给第三方，并确信它不会被有意无意的修改。特别是对于多方查看同一个Promise决议的情况，尤其如此。一方不可能影响另一方对Promise决议的结果。

#### 完成事件

可以从另一个角度看待Promise的决议：一种在异步任务中作为两个或更多步骤的流程控制机制，时序上的this-then-that。

```javascript
function timeoutPromise(delay) {
  return new Promise(function(resolve, reject) {
    setTimeout(function () {
      reject("Timeout!");
    }, delay)
  })
}

// 设置foo()超时
Promise.race([
  foo(), // 试着开始foo()
  timeoutPromise(3000)  // 给它三秒钟
])
.then(
  function() {
    // foo(..)及时完成！
  },
  function(err) {
    // 或者foo()被拒绝，或者只是没能按时完成
    // 查看err来了解是哪种情况
  }
)
```
如果向Promise.resolve(..)传递一个非Promise、非thenable的立即值，就会得到一个用这个值填充的promise。

```javascript
var p1 = new Promise(function(resolve, reject) {
  resolve(42);
})

var p2 = Promise.resolve(42)

var p1 = Promise.resolve(42)

var p2 = Promise.resolve(p1)

p1 === p2  // true
```


