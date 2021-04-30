# 深入理解ES6

## 块级作用域绑定

### var声明及变量提升（Hoisting）机制

```javascript
function getValue(condition) {
  if (condition) {
    var value = "blue"
    return value
  } else {
    // 此处可以访问变量value，其值为undefined
    return null
  }
  // 此处可以访问变量value，其值为undefined
}

//上面代码等价于

function getValue(condition) {
  
  var value;
  
  if (condition) {
    value = "blue"
    
    // 其他代码
    
    return value
  } else {
    return null
  }
}
```

### 块级声明

块级声明用于声明在指定块的作用域之外无法访问的变量。块级作用域（亦被称为词法作用域）存在于：

- 函数内部
- 块中（字符{和}之间的区域）

### let

```javascript
function getValue(condition) {
  if (condition) {
    let value = "blue"
    // 	其他代码
    return value
  } else {
    //  变量value在此处不存在
    return null
  }
  // 变量value在此处不存在
}
```

### 禁止重声明

```javascript
var count = 30

// 抛出语法错误
let count = 40
```

### const声明

```javascript
// 有效的常量
const maxItems = 30;

// 语法错误：常量未初始化
const name;
```

const和let都不会被提升至作用域顶部。

在Javascript中的常量如果是对象，则对象中的值可以修改。

```javascript
const person = {
  name: "Nicholas"
}

// 可以修改对象属性的值
person.name = "Greg"


// 抛出语法错误
person = {
  name: "Greg"
}
```

### 临时死区

```javascript
if (condition) {
  console.log(typeof value); // 引用错误！
  let value = "blue";
}
```

JavaScript引擎在扫描代码发现变量声明时，要么将他们提升至作用域顶部（遇到var声明），要么将声明放到TDZ中（遇到let和const声明）。访问TDZ中的变量会触发运行时错误。只有执行过变量声明语句后，变量才会从TDZ中移出，然后方可正常访问。

```javascript
console.log(typeof value); // undefined

if (condition) {
  let value = "blue"
}
```

### 循环中的块级作用域绑定

```javascript
for (var i = 0; i < 10; i++) {
  process(items[i]);
}
// 在这里仍然可以访问变量i
console.log(i)

for (let i = 0; i < 10; i++) {
  process(items[i])
}

// i 在这里不可访问，抛出一个错误
console.log(i)
```

### 循环中的函数

```javascript
var funcs = []

for (var i = 0; i < 10; i++) {
  funcs.push(function() {
    console.log(i)
  });
}

funcs.forEach(function(func) {
  func(); // 输出10次数字10
})

// 使用立即调用函数表达式（IIFE）解决该问题
var funcs = []

for (var i = 0; i < 10; i++) {
  funcs.push((function(value){
    return function() {
      console.log(value)
    }
  }(i)));
}

funcs.forEach(function(func) {
  func(); // 输出0， 然后是1、2，直到9
})

// 使用let解决
var funcs = [];

for (let i = 0; i < 10; i++) {
  funcs.push(function() {
    console.log(i)
  })
}

funcs.forEach(function(func) {
  func(); // 输出0，然后是1、2，直到9
})
```

### 循环中的const声明

```javascript
var funcs = [];

// 完成一次迭代后抛出错误
for (const i = 0; i < 10; i++) {
  funcs.push(function() {
    console.log(i)
  })
}
```

在for-in或for-of循环中使用const时的行为与使用let一致。

```javascript
var funcs = [],
    object = {
      a: true,
      b: true,
      c: true,
    };

// 不会产生错误
for (const key in object) {
  funcs.push(function() {
    console.log(key)
  })
}

funcs.forEach(function(func) {
  func(); // 输出a、b和c
})
```

之所以可以用运用在for-in和for-of循环中，是因为每次迭代不会（像前面for循环的例子一样）修改已有的绑定，而是会创建一个新绑定。

### 全局块作用域绑定

当var被用于全局作用域时，它会创建一个新的全局变量作为全局对象（浏览器环境中的window对象）的属性。这意味着用var很可能会无意中覆盖一个已经存在的全局属性。

```javascript
// 在浏览器中
var RegExp = "Hello!"
console.log(window.RegExp); // "Hello!"
```

如果在全局作用域中使用let或const，会在全局作用域下创建一个新的绑定，但该绑定不会添加为全局对象的属性。

```javascript
// 在浏览器中
let RegExp = "Hello!"
console.log(RegExp);  // "Hello!"
console.log(window.RegExp === RegExp);  // false
```

当前使用块级绑定的最佳实践是：默认使用const，只在确实需要改变变量的值时使用let。

## 字符串和正则表达式

### codePointAt()方法

这个方法接受编码单元的位置而非字符位置作为参数，返回与字符串中给定位置对应的码位，即一个整数值。

### String.fromCodePoint()方法

可以使用codePointAt()方法在字符串中检索一个字符的码位，也可以使用String.fromCodePoint()方法根据指定的码位生成一个字符。

### normalize()方法

可以提供Unicode的标准化形式，能够使我们能够更准确地比较多个字符串。

Unicode标准化形式：

- 以标准等价方式分解，然后以标准等价方式重组（"NFC"），默认选项。
- 以标准等价方式分解（"NFD"）.
- 以兼容等价方式分解（“NFKC”）
- 以兼容等价方式分解，然后以标准等价方式重组（"NFKD"）

```javascript
values.sort(function(first, second) {
  let firstNormalized = first.normalize("NFD")
  let secondNormalized = second.normalize("NFD")
  
  if (firstNormalized < secondNormalized) {
    return -1
  } else if (firstNormalized === secondNormalized) {
    return 0
  } else {
    return 1
  }
})
```

### 正则表达式u修饰符

将编码单元操作模式切换位字符模式

#### 检测u修饰符支持

```javascript
function hasRegExpU() {
  try {
    var pattern = new RegExp(".", "u");
    return true;
  } catch (ex) {
    return false
  }
}
```

### 其他字符串变更

#### 字符串中的子串识别

ECMAScript6中，提供了以下3个类型的方法可以达到indexOf()的效果

- includes()方法，如果在字符串中检测到指定文本则返回true，否则返回false。
- startsWith()方法，如果在字符串的起始部分检测到指定文本则返回true，否则返回false。
- endsWith()方法，如果在字符串的结束部分检测到指定文本则返回true，否则返回false。

对于startsWith()、endsWith()及includes()这三个方法，如果你没有按照要求传入一个字符串，而是传入一个正则表达式，则会触发一个错误产生；而对于indexOf()和lastIndexOf()这两个方法，它们都会把传入的正则表达式转化为一个字符串并搜索它。

#### repeat()方法

接受一个number类型的参数，表示该字符串的重复次数，返回值时当前字符串重复一定次数后的新字符串。

### 其他正则表达式语法变更

#### 正则表达式y修饰符

它会影响正则表达式搜索过程中的sticky属性，当在字符串中开始字符匹配时，它会通知搜索从正则表达式的lastIndex属性开始进行，如果在指定位置没能成功匹配，则停止继续匹配。

可以通过sticky属性检测y修饰符是否存在。

```javascript
let pattern = /hello\d/y

console.log(pattern.sticky);  // true
```

也可以通过方法检测

```javascript
function hasRefExpY() {
  try {
    var pattern = new RegExp(":", "y");
    return true;
  } catch (ex) {
    return false
  }
}
```

#### 正则表达式的复制

```javascript
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

#### flags属性

获取修饰符

```javascript
// ES5中
function getFlags(re) {
  var text = re.toString();
  return text.substring(text.lastIndexOf("/") + 1, text.length)
} 

var re = /ab/g;

console.log(getFlags(re)); // "g"

// ES6中
let re = /ab/g

console.log(re.source); // "ab"
console.log(re.flags); // "g"
```

### 模板字面量

这个方案是扩展ECMAScript基础语法的语法糖，提供一套生成、查询并操作来自其他语言里面的内容的DSL，且可以免受注入攻击，例如，XSS、SQL注入，等等。

填补了ECMAScript5缺少的多行字符串、基本的字符串格式化、HTML转义的特性。

```javascript
let message = `\`Hello\` world!`;

console.log(message); // "`Hello` world!"
console.log(typeof message); // "string"
console.log(message.length); // 14
```

#### 多行字符串

```javascript
let message = `Multiline
string`;

console.log(message);  // "Multiline
											 //  string"
console.log(message.length); // 16
```

在反撇号中的所有空白符都属于字符串的一部分，所以千万要小心缩进。

解决方法：

```javascript
let html = `
<div>
	<h1>Title</h1>
</div>
`.trim();

// 亦可
let message = `Multiline\nstring`

console.log(message); // "Multiline
											//  string"
console.log(message.length) // 16
```

#### 字符串占位符

```javascript
let name = "Nicholas",
		message = `Hello, ${name}.`;

console.log(message); // "Hello,Nicholas."
```

#### 标签模板

```javascript
 // 标签为tag
let message = tag`Hello world`
```

##### 定义标签

标签可以是个函数，调用时传入加工过的模板字面量各部分数据，但必须结合每个部分来创建结果。

```javascript
function tag(literals, ...substitutions) {
  // 返回一个字符串
}

let count = 10,
		price = 0.25,
    message = passthru`${count} items cost $${(count*price).toFixed(2)}`;
```

如果有一个passthru()的函数，它会接受3个参数：首先是一个literals数组，包含以下元素：

- 第一个占位符前的空字符串（""）
- 第一、二个占位符之间的字符串（“ items cost $”）
- 第二个占位符后的字符串 （"."）

下一个参数是变量count的解释值，传参为10，它也称为了substitutions数组里的第一个元素：最后一个参数是(count * price).toFixed(2)的解释值，传参为“2.50”，他是substitutions数组里的第二个元素。

##### 在模板字面量中使用原始值

```javascript
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1); // "Multiline
											 //  string"
console.log(message2); //  "Multiline\\nstring"
```

##### 模拟String.raw()

```javascript
function raw(literals, ...substitutions) {
  let result = "";
  
  // 根据 substitution的数量来确定循环的执行次数
  for (let i = 0; i < substitutions.length; i++) {
    // 使用原生值
    result += literals.raw[i];
    result += substitutions[i];
  }
  
  // 合并最后一个literal
  result += literals.raw[literals.length - 1];
  return result;
}
```

## 函数

### 函数形参的默认值

```javascript
// ES5 
function makeRequest(url, timeout, callback) {
  timeout = (typeof timeout !== "undefined") ? timeout : 2000;
  callback = (typeof callback !== "undefined") ? callback : function() {}
  
  // 函数的其余部分
}

// ES6
function makeRequest(url, timeout = 2000, callback = function() {}) {
  // 函数的其余部分
}
```

对于默认参数值，null是一个合法值。

#### 默认参数值对arguments对象的影响

```javascript
function mixArgs(first, second) {
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
  first = "c";
  second = "d";
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
}

mixArgs("a", "b");
// true
// true
// true
// true

function mixArgs(first, second) {
  "use strict";
  
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
  first = "c";
  second = "d";
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
}

mixArgs("a", "b");
// true
// true
// false
// false

function mixArgs(first, second = "b") {
  console.log(arguments.length)
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
  first = "c";
  second = "d";
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
}

mixArgs("a");
// 1
// true
// false
// false
// false
```

#### 默认参数表达式

```javascript
function getValue() {
  return 5;
}

function add (first, second = getValue()) {
  return first + second
}

console.log(add(1, 1))
console.log(add(1))
```

初次解析函数声明时不会调用getValue()方法，只有当调用add()函数切不传入第二个参数时才会调用。

```javascript
let value = 5;

function getValue() {
  return value++;
}

function add(first, second = getValue()) {
  return first + second;
}

console.log(add(1, 1));  // 2
console.log(add(1));  // 6
console.log(add(1));  // 7
```

```javascript
function add(first = second, second) {
  return first + second;
}

console.log(add(1, 1))  // 2
console.log(undefined, 1)  // 因为second未定义，抛出错误
```

#### 默认参数的临时死区

在默认参数的临时死区中的参数不可访问。与let声明类似，定义参数时会为每个参数创建一个新的标识符绑定，该绑定在初始化之前不可被引用，如果试图访问会导致程序抛出错误。当调用函数时，会通过传入的值或参数的默认值初始化该参数。

**注意**：函数参数有自己的作用域和临时死区，其与函数体的作用域时各自独立的，就是说参数的默认值不可访问函数体内声明的变量。

### 处理无命名函数

#### 不定参数

```javascript
function pick(object, ...keys) {
  let result = Object.create(null);
  
  for (let i = 0, len = keys.length; i < len; i++) {
    result[keys[i]] = object[keys[i]];
  }
  
  return result;
}
//  length为1
```

注意：函数的length属性统计的是函数命名参数的数量，不定参数的加入不会影响length属性的值。

#### 不定参数的使用限制

1.每个函数最多只能声明一个不定参数。

2.一定要放在所有参数的末尾。

3.不能在setter中使用不定参数。

#### 不定参数对arguments对象的影响

不论是否使用不定参数，arguments对象总是包含所有传入函数的参数。

### 展开运算符

```javascript
// ES5 中的解决方案

let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values)); // 100

// ES6 中的解决方案

let values = [25, 50, 75, 100]

console.log(Math.max(...values)); // 100
```

### 明确函数的多重用途

```javascript
function Person(name) {
  this.name = name;
}

var person = new Person("Nicholas")
var notAPerson = Person("Nicholas")

console.log(person); // "[Object, object]"
console.log(notAPerson); // "undefined"
```

JavaScript函数有两个不同的内部方法：[[Call]] 和 [[Construct]]。当通过new关键词调用函数时，执行的是[[Construct]]函数，它负责创建一个通常被称作实例的新对象 ，然后再执行函数体，将this绑定到实例上；如果不通过new关键字调用函数，则执行[[Call]]函数，从而直接执行代码中的函数体。具有[[Construct]]方法的函数被统称为构造函数。

```javascript
// ES5中判断函数被调用的方法
function Person(name) {
  if (this instanceof Person) {
    this.name = name; // 如果通过new关键字调用
  } else {
    throw new Error("必须通过new关键字来调用Person")
  }
}

var Person = new Person("Nicholas");
var notAPerson = Person("Nicholas"); // 抛出错误

var notAPerson = Person.call(person, "Michael");  // 有效！
```

#### 元属性(Metaproperty) new.target

元属性是指非对象的属性，其可以提供非对象目标的补充信息。

当调用函数的[[Construct]]方法时，new.target被赋值为new操作符的目标，通常是新创建对象实例，也就是函数体内this的构造函数；如果调用[[Call]]方法，则new.target的值为undefined。

函数外使用new.target是一个语法错误。

### 块级函数

```javascript
"use strict"

if (true) {
  console.log(typeof doSomething); // "function"
  
  function doSomething() {
    // 空函数
  }
  
  doSomething();
}

console.log(typeof doSomething); // "undefined"

"use strict"

if (true) {
  console.log(typeof doSomething); // 抛出错误
  
  let doSomething = function () {
    // 空函数
  }
  
  doSomething();
}

console.log(typeof doSomething); // "undefined"
```

### 箭头函数

特点：

1.没有this、super、arguments和new.target绑定

2.不能通过new关键字调用

3.没有原型

4.不可以改变this的绑定(箭头函数中的this值取决于该函数外部非箭头函数的this值，且不能通过call()、apply()、或bind()方法来改变this的值。)

5.不支持arguments对象

6.不支持重复的命名参数

**注意：箭头函数同样也有一个name属性，这与其他函数的规则相同。**

#### 创建立即执行函数表达式

```javascript
let person = ((name) => {
  return {
    getName: function() {
      return name;
    }
  };
})("Nicholas");

console.log(person.getName()); // Nicholas
```

## 扩展对象的功能性

### 对象类别

普通对象：具有Javascript对象所有的默认内部行为。

特异对象；具有某些与默认行为不符的内部行为。

标准对象：ES6规范中定义的对象，例如，Array、Date等。标准对象既可以是普通对象，也可以是特异对象。

内建对象：脚本开始执行时存在于Javascript执行环境中的对象，所有标准对象都是内建对象。

### 对象字面量的扩充

#### 可计算属性名

```javascript
var suffix = " name"
var person = {
  ["first" + suffix]: "Nicholas",
  ["last" + suffix]: "Zakas"
};

console.log(person["first name"]) // "Nicholas"
console.log(person["last name"]) // "Zakas"
```

### 新增方法

#### Object.is()

为了应对+0与-0相等，NaN与NaN不相等的问题。

#### Object.assign()

该方法可以接受任意数量的源对象，并按制定的顺序将属性赋值到接收对象中。所以如果多个源对象具有同名属性，则排位靠后的源对象会覆盖排位靠前的。

注：Object.assign()方法不能将提供者的访问器属性复制到接受对象中。由于 Object.assign()方法执行了赋值操作，因此提供者的访问器属性最终会转变为接收对象中的一个数据属性。

```javascript
var receiver = {},
    supplier = {
      get name() {
        return "file.js"
      }
    };

Object.assign(receiver, supplier);

var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");

console.log(descriptor.value); // "file.js"
console.log(descriptor.get); // undefined
```

### 自有属性枚举顺序

自有属性枚举顺序的基本规则是：

1.所有数字键按升序排序。

2.所有字符串键按照它们被加入对象的顺序排序。

3.所有symbol键按照它们被加入对象的顺序排序。

```javascript
var obj = {
  a: 1,
  0: 1,
  c: 1,
  2: 1,
  b: 1,
  1: 1
};

obj.d = 1;

console.log(Object.getOwnPropertyNames(obj).join("")); // "012acbd"
```

### 增强对象原型

#### 改变对象的原型

Object.setPrototypeOf()

#### 简化原型访问的Super引用

super可以便捷的访问对象的原型。

```javascript
let friend = {
  getGreeting() {
    // Object.getPrototypeof(this).getGreeting.call(this)相同
    return super.getGreeting() + ", hi!";
  }
}
```

### 正式的方法定义

在ES6中正式将方法定义为一个函数，它会有一个内部的[[HomeObject]]属性来容纳这个方法从属的对象。

```javascript
let person = {
  
  // 是方法
  getGreeting() {
    return "Hello";
  }
};


// 不是方法
function shareGreeting() {
  return "Hi!";
}
```

Super的所有引用都通过[[HomeObject]]属性来确定后续的运行过程。第一步是在[[HomeObject]]属性上调用Object.getPrototypeof()方法来检索原型的引用；然后搜寻原型找到同名函数；最后，设置this绑定并且调用相应的方法。

## 解构：使数据访问更便捷

### 对象解构

```javascript
let node = {
  type: "Identifier",
  name: "foo"
}

let {type, name} = node;
```

解构赋值表达式（也就是=右侧的表达式）如果为null或undefined会导致程序抛出错误。也就是说，任何尝试读取null或undefined的属性的行为都会触发运行时错误。

#### 默认值

使用解构赋值表达式时，如果指定的局部变量名称在对象中不存在，那么这个局部变量会被赋值为undefined。

### 数组解构

```javascript
let colors = ["red", "green", "blue"]
let [ firstColor, secondColor ] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

#### 默认值

```javascript
let colors = ["red"];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);
console.log(secondColor);
```

#### 数组克隆

```javascript
let colors = ["red", "green", "blue"];
let [...clonedColors] = colors;

console.log(cloneedColors); // "[red, green, blue]"
```

## Symbol和Symbol属性

### 创建Symbol

```javascript
let firstName = Symbol();
let person = {};

person[firstName] = "Nicholas"
```

### Symbol共享体系

```javascript
let uid = Symbol.for("uid");
let object = {
  [uid]: "12345"
}；

console.log(object[uid]); // "12345"
console.log(uid); // "Symbol(uid)"

let uid2 = Symbol.for("uid");

console.log(uid === uid2); // true
console.log(object[uid2]); // "12345"
console.log(uid2); // "Symbol(uid)"
```

在这个示例中，uid和uid2包含相同的Symbol并且可以互换调用。第一次调用Symbol.for()方法创建这个Symbol，第二次调用可以直接从Symbol的全局注册表中检索到这个Symbol。

可以使用Symbol.keyFor()方法在Symbol全局注册表中检索与Symbol有关的键。

```javascript
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));  // "uid"

let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));  // "uid"

let uid3 = Symbol("uid")
console.log(Symbol.keyFor(uid3)); // undefined
```

### Symbol属性检索

Object.keys()方法和Object.getOwnPropertyNames()方法可以检索对象中所有的属性名：前一个方法返回所有可枚举的属性名；后一个方法不考虑属性的可枚举性一律返回。但是，这两个方法都不支持Symbol属性。

Object.getOwnPropertySymbols()方法来检索对象中的Symbol属性。

#### Symbol.hasInstance方法

每个函数中都有一个Symbol.hasInstance方法，用于确定对象是否为函数的实例。

与instanceOf相关

#### Symbol.isConcatSpreadable属性

与concat()函数相关

Symbol.isConcatSpreadable属性是一个布尔值，如果该属性值为true，则表示对象有length属性和数字键，故它的数值型属性值应该被独立添加到concat()调用的结果中。

#### Symbol.match、Symbol.replace、Symbol.search 和 Symbol.split属性

类似于对应的函数。

#### Symbol.toPrimitive方法

每当执行原始值转换时，总会调用Symbol.toPrimitive方法并传入一个值作为参数，这个值在规范中被称作类型提示（hint）。类型提示参数的值只有三种选择："number"、"string"、"deafult"，传递这些参数时，Symbol.toPrimitive返回的分别是：数字、字符串或无类型便好的值。

数字模式下的优先级：

1.调用valueOf()方法，如果结果为原始值，则返回。

2.否则，调用toString()方法，如果结果为原始值，则返回。

3.如果再无可选值，则抛出错误。

字符串模式的优先级：

1.调用toString()方法，如果结果为原始值，则返回。

2.否则，调用valueOf()方法，如果结果为原始值，则返回。

3.如果再无可选值，则抛出错误。

在大多数情况下，标准对象会将默认模式按照数字模式处理（除了Date对象，在这种情况下，会将默认模式按照字符串模式处理）。如果自定义Symbol.toPrimitive方法，则可以覆盖这些默认的强制转换特性。

注：默认模式只用于==运算、+运算及给Date构造函数传递一个参数时。在大多数的操作中，使用的都是字符串模式或数字模式。

```javascript
function Temperature(degrees) {
  this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {
  switch(hint) {
    case "string":
      return this.degrees + "\u00b0"; // degrees symbol
    case "number":
      return this.degrees;
    case "default":
      return this.degrees + " degrees";
  }
};

var freezing = new Temperature(32);

console.log(freezing + "!");  // "32 degrees!"
console.log(freezing / 2);  // 16
console.log(String(freezing)) // "32o"
```

#### Symbol.toStringTag属性

```javascript
Array.prototype[Symbol.toStringTag] = "Magic";

var values = [];

console.log(Object.prototype.toString.call(values)); // "[object Magic]"
```

不建议

#### Symbol.unscopables属性

针对with，不需要了解。

## Set集合与Map集合

### Set集合

#### 创建Set集合并添加元素

```javascript
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size); // 2

let set = new Set(),
		key1 = {},
    key2 = {}

set.add(key1);
set.add(key2);

console.log(set.size); // 2
```

Set构造函数可以接受所有可迭代对象作为参数，数组、Set集合、Map集合都是可迭代的。构造函数通过迭代器从参数中提取值。

has()方法可以检测Set集合中是否存在某个值。

#### 移除元素

调用clear()方法会移除集合中的所有元素

#### Set集合的forEach()方法

```javascript
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
  console.log(key + " " + value);
  console.log(ownerSet === set);
})

// 1 1
// true
// 2 2
// true

let set = new Set([1, 2]);

let processor = {
  output(value) {
    console.log(value);
  },
  process(dataSet) {
    dataSet.forEach(function(value) {
      this.output(value)
    }, this);
  }
};

processor.process(set);
```

#### 将Set集合转换为数组

```javascript
let set = new Set([1,2,3,3,3,4,5]),
    array = [...set];

console.log(array); // [1,2,3,4,5]
```

#### Weak Set集合

将对象存储在Set的实例与存储在变量中完全一样，只要Set实例中的引用存在，垃圾回收机制就不能释放对象的内存空间，于是之前提到的Set类型可以被看作是一个强引用的Set集合。

```javascript
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);  // 1

// 移除原始引用
key = null;

console.log(set.size); // 1

// 重新取回原始引用
key = [...set][0];
// 内存泄漏
```

Weak Set集合只存储对象的弱引用，并且不可以存储原始值；集合中的弱引用如果是对象唯一的引用，则会被回收并释放相应内存。

```javascript
let set = new WeakSet(),
    key = {};

// 向集合set中添加对象
set.add(key);

console.log(set.has(key)); // true

set.delete(key);

console.log(set.has(key)); // false
```

#### WeakSet和Set的区别

- 在WeakSet的实例中，如果向add()方法传入非对象参数会导致程序报错，而向has()和delete()方法传入非对象参数则会返回false。
- Weak Set集合不可迭代，所以不能被用于for-of循环。
- Weak Set集合不暴露任何迭代器（例如keys()和values()方法），所以无法通过程序本身来检测器中的内容
- Weak Set集合不支持forEach()方法
- Weak Set集合不支持size属性

### Map集合

```javascript
let map = new Map();
map.set("title", "Understanding ECMAScript 6")
map.set("year", 2016)

console.log(map.get("title")); // "Understanding ECMAScript 6"
console.log(map.get("year")); // 2016

let map = new Map(),
    key1 = {},
    key2 = {};

map.set(key1, 5)
map.set(key2, 42)

console.log(map.get(key1));  // 5
console.log(map.get(key2));  // 42
```

#### Map集合支持的方法

- has(key) 检测指定的键名在Map集合中是否已经存在。
- delete(key) 从Map集合中移除指定键名及其对应的值。
- clear() 移除Map集合中的所有键值对。

#### Map集合的初始化方法

```javascript
let map = new Map([["name", "Nicholas"], ["age", 25]])

console.log(map.has("name"));  // true
console.log(map.get("name")); // "Nicholas"
console.log(map.has("age")); // true
console.log(map.get("age")); // 25
console.log(map.size); // 2
```

#### Map集合的forEach()方法

和set类似。

#### Weak Map集合

Weak Map集合只支持两个可以操作键值对的方法：has()方法可以检测给定的键在集合中是否存在；delete()方法可以移除指定的键值对。Weak Map集合与 Weak Set集合一样，二者都不支持键名枚举，从而也不支持clear()方法。

### 小结

Set集合是一种包含多个非重复值的有序列表，值与值之间的等价性是通过Object.is()方法来判断的，如果相同，则会自动过滤重复的值，所以可以用来去重。Weak Set值支持存放对象的弱引用，当该对象的其他强引用都被清除时，集合中的弱引用也会自动被垃圾回收。

Map集合是多个键值对 组成的有序集合，键名支持任意数据结构，Map集合也是通过Object.is()方法来过滤重复的值。Weak Map和Weak Set类似。

## 迭代器和生成器

```javascript
// ES5 版本
function createIterator(items) {
  var i = 0;
  
  return {
    next: function() {
      
      var done = (i >= items.length);
      var value = !done ? items[i++] : undefined;
      
      return {
        done: done,
        value: value
      };
    }
  };
}

var iterator = createIterator([1,2,3]);

console.log(iterator.next()) // "{ value: 1, done: false}"
console.log(iterator.next()) // "{ value: 2, done: false}"
console.log(iterator.next()) // "{ value: 3, done: false}"
console.log(iterator.next()) // "{ value: undefined, done: true}"

// 之后所有的调用都会返回相同内容
console.log(iterator.next()) // "{ value: undefined, done: true}"
```

### 生成器

```javascript
function *createIterator() {
  yield 1;
  yield 2;
  yield 3;
}

// 生成器的调用方式和普通函数相同，只不过返回的是一个迭代器
let iterator = createIterator();

console.log(iterator.next().value); // 1
console.log(iterator.next().value); // 2
console.log(iterator.next().value); // 3
```

### 可迭代对象和for-of循环

可迭代对象具有Symbol.iterator属性，Symbol.iterator通过指定过的函数可以返回一个作用于附属对象的迭代器。在ES6中，所有的集合对象（数组、Set集合及Map集合）和字符串都是可迭代对象，这些对象中都有默认的迭代器。

由于生成器默认会为Symbol.iterator属性赋值，因此所有通过生成器创建的迭代器都是可迭代对象。

注意：如果将for-of语句用于不可迭代对象、null或undefined将会导致程序抛出错误。

#### 访问默认迭代器

```javascript
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();

console.log(iterator.next()); // "{ value: 1, done: false}"
console.log(iterator.next()); // "{ value: 2, done: false}"
console.log(iterator.next()); // "{ value: 3, done: false}"
console.log(iterator.next()); // "{ value: undefined, done: true}"
```

检测对象是否为可迭代对象：

```javascript
function isIterable(object) {
  return typeof object[Symbol.iterator] === "function";
}
```

#### 创建可迭代对象

```javascript
let collection = {
  items: [],
  *[Symbol.iterator]() {
    for (let item of this.items) {
      yield item;
    }
  }
};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for(let x of collection) {
  console.log(x)
}
// 1
// 2
// 3 
```

### 内建迭代器

#### 集合对象迭代器

- entries() 返回一个迭代器，其值为多个键值对。
- values() 返回一个迭代器，其值为集合的值。 
- keys() 返回一个迭代器，其值为集合中的所有键名。

对于数组对象来说，无论是否为数组添加命名属性，for-of打印出来的都是数字类型的索引；而for-in循环迭代的是数组属性而不是数字类型的索引。

数组和Set集合的默认迭代器是values()方法，Map集合的默认迭代器是entries()方法。

#### 解构与for-of循环

```javascript
let data = new Map()

data.set("title", "Understanding ECMAScript 6")
data.set("format", "ebook")

// 与使用 data.entries()方法相同
for (let [key, value] of data) {
  console.log(key + "=" + value )
}
```

### 高级迭代器功能

#### 给迭代器传递参数

```javascript
function *createIterator() {
  let first = yield 1;
  let second = yield first + 2; // 4 + 2
  yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next()) // "{ value: 1, done: false }"
console.log(iterator.next(4)) // "{ value: 6, done: false }"
console.log(iterator.next(5)) // "{ value: 8, done: false }"
console.log(iterator.next()) // "{ value: undefined, done: true }"
```

#### 在迭代器中抛出错误

```javascript
function *createIterator() {
  let first = yield 1;
  let second = yield first + 2; // yield 4 + 2, 然后抛出错误
  yield second + 3; // 永远不会被执行
}

let iterator = createIterator();

console.log(iterator.next()); // "{ value: 1, done: false}"
console.log(iterator.next(4)); // "{ value: 6, done: false}"
console.log(iterator.throw(new Error("Boom"))); // 从生成器中抛出的错误
```

解决方案：

```javascript
function *createIterator() {
  let first = yield 1;
  let second;
  
  try {
    second = yield first + 2; // yield 4 + 2，然后抛出错误
  } catch(ex) {
    second = 6;
  }
  yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next()); // "{ value: 1, done: false}"
console.log(iterator.next(4)); // "{ value: 6, done: false}"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false}"
console.log(iterator.next()); // "{ value: undefined, done: true}"
```

#### 生成器返回语句

```javascript
function *createIterator() {
  yield 1;
  return;
  yield 2; // 不会再执行
  yield 3; // 不会再执行
}

let iterator = createIterator();

console.log(iterator.next()); // "{ value: 1, done: false}"
console.log(iterator.next()); // "{ value: undefined, done: false}"
```

```javascript
function *createIterator() {
  yield 1;
  return 42;
}

let iterator = createIterator();

console.log(iterator.next()); // "{value: 1, done: false}"
console.log(iterator.next()); // "{value: 42, done: true}"
console.log(iterator.next()); // "{value: undefined, done: true}"
```

注意：展开运算符与for-of循环语句会直接忽略通过return语句指定的任何返回值，只要done一变为true就立即停止读取其他的值。

#### 异步任务执行

```javascript
function run(taskDef) {
  // 创建一个无使用限制的迭代器
  let task = taskDef();
  
  // 开始执行任务
  let result = task.next();
  
  // 循环调用next()函数
  function step() {
    // 如果任务未完成，则继续执行
    if (!result.done) {
      if (typeof result.value === "function") {
        result.value(function(err,data) {
          if (err) {
            result = task.throw(err);
            return;
          }
          result = task.next(data);
          step();
        })
      } else {
        result = task.next(result.value);
        step();
      }
    }
  }
  // 开始迭代执行
  step();
}
```

## JavaScript中的类

### 类的声明

```javascript
class PersonClass {
  // 等价于PersonType构造函数
  constructor(name) {
    this.name = name;
  }
  
  // 等价于 PersonType.prototype.sayName
  sayName() {
    console.log(this.name)
  }
}

let person = new PersonClass("Nicholas");
person.sayName(); // outputs "Nicholas"

console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true

console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass.prototype.sayName); // "function"
```

#### 为何使用类语法

- 函数声明可以被提升，而类声明与let声明类似，不能被提升；真正执行声明语句之前，它们会一直存在于临时死区中。
- 类声明中的所有代码将自动在严格模式下，而且无法强行让代码脱离严格模式执行。
- 在自定义类型中，需要通过Object.defineProperty()方法手工指定某个方法为不可枚举的；而在类中，所有方法都是不可枚举的。 
- 每个类都有一个名为[[Construct]]的内部方法，通过关键字new调用那些不含[[Construct]]的方法会导致程序抛出错误。
- 使用除关键字new以外的方式调用类的构造函数会导致程序抛出错误。
- 在类中修改类名会导致程序报错。

### 类表达式

#### 基本的类表达式语法

```javascript
let PersonClass = class {
  // 等价于PersonType构造函数
  constructor(name) {
    this.name = name;
  }
  
  // 等价于 PersonType.prototype.sayName
  sayName() {
    console.log(this.name)
  }
}

let person = new PersonClass("Nicholas");
person.sayName(); // outputs "Nicholas"

console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true

console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass.prototype.sayName); // "function"
```

#### 可计算成员名称

```javascript
let methodName = "sayName";

class PersonClass {
  constructor(name) {
    this.name = name
  }
  [methodName]() {
    console.log(this.name)
  }
}

let me = new PersonClass("Nicholas");
me.sayName();

let propertyName = "html"

class CustomHTMLElement {
  
  constructor(element) {
    this.element = element;
  }
  
  get [propertyName]() {
    return this.element.innerHTML
  }
  
  set [propertyName]() {
    this.element.innerHTML = value;
  }
}
```

### 继承与派生类

```javascript
class Square extends Rectangle {
  // 没有构造函数
}

// 等价于
class Square extends Rectangle {
  constructor(...args) {
    super(...args)
  }
}
```

#### 使用super()的小贴士

- 只可在派生类的构造函数中使用super()。
- 在构造函数中访问this之前一定要调用super()，它负责初始化this，如果在调用super()之前尝试访问this会导致程序出错。
- 如果不想调用super()，则唯一的方法是让类的构造函数返回一个对象。

#### mixin

```javascript
let SerializableMixin = {
  serialize() {
    return JSON.stringify(this)
  }
}

let AreaMixin = {
  getArea() {
    return this.length * this.width
  }
}

function mixin(...mixins) {
  var base = function() {};
  Object.assign(base.prototype, ...mixins)
  return base
}

class Square extends mixin(AreaMixin, SerializableMixin) {
  constructor(length) {
    super();
    this.length = length;
    this.width = length;
  }
}

var x = new Square(3)
console.log(x.getArea());  // 9
console.log(x.serialize()); // "{"length":3, "width":3}"
```

可以通过Symbol.species重写返回类型。

## 改进的数组功能

```javascript
let items = new Array(2);
console.log(items.length);  // 2
console.log(items[0]); // undefined
console.log(items[1]); // undefined

items = new Array("2");
console.log(items.length); // 1
console.log(items[0]); // "2"

items = new Array(1,2);
console.log(items.length);  // 2
console.log(items[0]); // 1
console.log(items[1]); // 2

items = new Array(3, "2");
console.log(items.length);  // 2
console.log(items[0]); // 3
console.log(items[1]); // "2"
```

### Array.of()

```javascript
let items = Array.of(1,2)
console.log(items.length); // 2
console.log(items[0]); // 1
console.log(items[1]); // 2

items = Array.of(2);
console.log(items.length); // 1
console.log(items[0]); // 2
```

### Array.from()

```javascript
// ES5
function makeArray(arrayLike) {
  var result = [];
  for (var i = 0, len = arraryLike.length; i < len; i++) {
    result.push(arrayLike[i]);
  }
  return result;
} 

// 也可以
function makeArray(arrayLike) {
  return Array.prototype.slice.call(arrayLike);
}
```

Array.from()方法可以接受可迭代对象或类数组对象作为第一个参数，最终返回一个数组。

Array.from()方法调用会基于arguments对象中的元素创建一个新数组，args是Array的一个实例，包含arguments对象中同位置的相同值。

Array.from()方法也是通过this来确定返回数组的类型的。

##### 映射转化

```javascript
function translate() {
  return Array.from(arguments, (value) => value + 1);
}

let numbers = translate(1,2,3)
console.log(numbers); // 2, 3, 4

// 第三个参数
let helper = {
  diff: 1,
  
  add(value) {
    return value + this.diff;
  }
};

function translate() {
  return Array.from(arguments, helper.add, helper);
}

let numbers = translate(1, 2, 3);
console.log(numbers); // 2, 3, 4
```

##### 用Array.from()转换可迭代对象

```javascript
let numbers = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  }
};

let numbers2 = Array.from(numbers, (value) => value + 1);
console.log(numbers2); // 2, 3, 4
```

如果一个对象既是类数组又是可迭代的，那么Array.from()方法会根据迭代器来决定转换哪个值。

### 为所有数组添加的新方法

#### find()方法和findIndex()方法

```javascript
let numbers = [25, 30, 35, 40, 45];

console.log(numbers.find(n => n > 33)); // 35
console.log(numbers.findIndex(n => n > 33)) // 2
```

#### fill()方法

```javascript
let numbers = [1, 2, 3, 4];

numbers.fill(1);

console.log(numbers.toString()); // 1, 1, 1, 1

let numbers = [1, 2, 3, 4];

numbers.fill(1, 2);

console.log(numbers.toString()); // 1, 2, 1, 1

numbers.fill(0, 1, 3);

console.log(numbers.toString()); // 1, 0, 0, 1
```

#### copyWithin()方法

```javascript
let numbers = [1, 2, 3, 4];

// 从数组的索引2开始粘贴值
// 从数组的索引0开始复制值
numbers.copyWithin(2, 0);

console.log(numbers.toString()); // 1, 2, 1, 2

let numbers = [1, 2, 3, 4]

// 从数组的索引2开始粘贴值
// 从数组的索引0开始复制值
// 当位于索引1时停止复制值
numbers.copyWithin(2, 0, 1);

console.log(numbers.toString()); // 1, 2, 1, 4
```

正如fill()方法一样，copyWithin()方法的所有参数都接受负数值，并且会自动与数组长度相加来作为最终使用的索引。

#### 数组缓冲区

```javascript
let buffer = new ArrayBuffer(10) // 分配10字节
let buffer2 = buffer.slice(4, 6)
console.log(buffer2.byteLength) // 2
```

注：数组缓冲区包含的实际字节数量在创建时就已确定，可以修改缓冲区内的数据，但是不能改变缓冲区的尺寸大小。

#### 通过视图操作数组缓冲区

数组缓冲区是内存中的一段地址，视图是用来操作内存的接口。视图可以操作数组缓冲区或缓冲区字节的子集，并按照其中一种数值型数据类型来读取和写入数据。DataView类型是一种通用的数组缓冲区视图，其支持所有8种数值型数据类型。

```javascript
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer, 5, 2);  // 包含位于索引5和6的字节
```

##### 获取视图信息

-  buffer 视图绑定的数组缓冲区。
-  byteOffset DataView 构造函数的第二个参数，默认是0，只有传入参数时才有值。
-  byteLength DataView 构造函数的第三个参数，默认时缓冲区的长度byteLength

```javascript
let buffer = new ArrayBuffer(10),
    view1 = new DataView(buffer),  // 覆盖所有字节
    view2 = new DataView(buffer, 5, 2); // 覆盖位于索引5和索引6的字节

console.log(view1.buffer === buffer);  // true
console.log(view2.buffer === buffer);  // true
console.log(view1.byteOffset);  // 0
console.log(view2.byteOffset);  // 5
console.log(view1.byteLength); // 10
console.log(view2.byteLength); // 2
```

##### 读取和写入数据

- getInt8(byteOffset, littleEndian) 读取位于byteOffset后的 int8类型数据。
- setInt8(byteOffset, value, littleEndian) 在byteOffset处写入int8类型数据。
- getUint8(byteOffset, littleEndian) 读取位于byteOffset后的 uint8类型数据。
- setUint8(byteOffset, value, littleEndian) 在byteOffset处写入uint8类型数据。

- getFloat32(byteOffset, littleEndian) 读取位于byteOffset后的 float32类型数据。
- setFloat32(byteOffset, value, littleEndian) 在byteOffset处写入float32类型数据。
- getFloat64(byteOffset, littleEndian) 读取位于byteOffset后的 float64类型数据。
- setFloat64(byteOffset, value, littleEndian) 在byteOffset处写入float64类型数据。

定型数组是视图

注意：调用定型数组的构造函数时如果不传参数，会按照传入0来处理，这样由于缓冲区没有分配到任何比特，因而创建的定型数组不能用来保存数据。

普通数组中可以修改length属性来改变普通数组的大小，而定型数组的length属性时一个不可写属性，所以不能修改定型数组的大小，如果尝试修改这个值，在非严格模式下会直接忽略该操作，在严格模式下会抛出错误。

```javascript
let ints = new Int16Array([25, 50]);

console.log(ints instanceof Array);  // false
console.log(Array.isArray(ints));  // false
```

##### 定型数组缺失的方法

concat()、shift()、pop()、splice()、push()、unshift()、

##### 附加方法

set()方法将其他数组复制到已有的定型数组。

subarray()提取已有定型数组的一部分作为新的定型数组。

## Promise与异步编程

Javascript引擎同一时刻只能执行一个代码块，所以需要跟踪即将运行的代码，那些代码被放在一个任务队列中，每当一段代码准备执行时，都会被添加到任务队列。每当Javascript引擎中的一段代码结束执行，事件循环会执行队列中的下一个任务，它是Javascriptpt引擎中的一段程序，负责监控代码执行并管理任务队列。请记住，队列中的任务会从第一个执行到最后一个。

### Promise的基础知识

#### Promise的声明周期

内部属性[[PromiseState]]被用来表示Promise的3种状态: "pending"、"fulfilled"、"rejected"

注意：如果一个对象实现了上述的then()方法，那这个对象我们称之为thenable对象。所有的Promise都是thenable对象，但并非所有thenable对象都是Promise。

注意：每次调用then()方法或catch()方法都会创建一个新任务，当Promise被解决（resolved）时执行。这些任务最终会被加入到一个为Promise量身定制的独立队列中。

### 全局的Promise拒绝处理

如果在没有拒绝处理程序的情况下拒绝一个Promise，那么不会提示失败信息。

#### Node.js 环境的拒绝处理

在Node.js中，处理Promise拒绝时会触发process对象上的两个事件：

- unhandledRejection 在一个事件循环中，当Promise被拒绝，并且没有提供处理程序时，触发该事件。
- rejectionHandled 在一个事件循环后，当Promise被拒绝时，若拒绝处理程序被调用，触发该事件。

#### 浏览器环境的拒绝处理

类似node.js环境的拒绝处理（事件名）

区别：

在Node.js实现中，事件处理程序接收多个独立参数；而在浏览器中，事件处理程序接受一个有以下属性的事件对象作为参数；

- type 事件名称
- promise 被拒绝的Promise对象
- reason 来自Promise的拒绝值

### 串联Promise

务必在Promise链的末尾留有一个拒绝处理程序以确保能够正确处理所有可能发生的错误。

### 响应多个Promise

#### Promise.all()方法

该方法只接受一个参数并返回一个Promise，该参数时一个含有多个受监视Promise的可迭代对象，只有当可迭代对象中所有Promise都被解决后返回的Promise才会被解决，只有当可迭代对象中所有Promise都被完成后返回的Promise才会被完成。

#### Promise.race()方法

它也接受含多个受监视Promise的可迭代对象作为唯一参数并返回一个Promise，但只要有一个Promise被解决返回的Promise就被解决，无须等到所有Promise都被完成。

### 自Promise继承

```javascript
class MyPromise extends Promise {
  // 使用默认的构造函数
  
  success(resolve, reject) {
    return this.then(resolve, reject);
  }
  
  failure(reject) {
    return this.catch(reject)
  }
}

let promise = new MyPromise(function(resolve, reject) {
  resolve(42);
})

promise.success(function(value) {
  console.log(value);   // 42
}).failure(function(vlaue) {
  console.log(value);
})
```

## 代理（Proxy）和反射（Reflection）API

### 创建一个简单的代理

```javascript
let target = {};
let proxy = new Proxy(target, {});

proxy.name = "proxy";
console.log(proxy.name); // "proxy"
console.log(target.name); // "proxy"

target.name = "target"
console.log(proxy.name);  // "target"
console.log(target.name); // "target"
```

### 使用set陷阱验证属性

set陷阱接受4个参数

- trapTarget 用于接受属性（代理的目标）的对象
- key 要写入的属性键（字符串或Symbol类型）
- value 被写入属性的值
- receiver 操作发生的对象

```javascript
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
    set(trapTarget, key, value, receiver) {
        // 忽略不希望受到影响的己有属性
        if(!trapTarget.hasOwnProperty(key)) {
            if (isNaN(value)) {
                throw new TypeError("属性必须是数字")
            }
        }
        
        // 添加属性
        return Reflect.set(trapTarget, key, value, receiver);
    }
})
```

### 用get陷阱验证对象结构

get陷阱接受3个参数

- trapTarget 被读取属性的源对象
- key 要读取的属性键
- receiver 操作发生的对象

```javascript
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
    get(trapTarget, key, receiver) {
            if (!(key in receiver)) {
                throw new TypeError("属性" + key + "不存在")
            }
        // 添加属性
        return Reflect.Get(trapTarget, key, receiver);
    }
})
```

### 使用has陷阱隐藏已有属性

has陷阱接受2个参数

- trapTarget 读取属性的对象（代理的目标）
- key 要检查的属性键（字符串或Symbol）

```javascript
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
    	has(trapTarget, key) {
            if (key === "value") {
                return false
            } else {
                return Reflect.has(trapTarget, key)
            }
        }
 
})
```

### 用deleteProperty陷阱防止删除属性

它接受两个参数：

- trapTarget 要删除属性的对象（代理的目标）
- key 要删除的属性键（字符串或Symbol）

```javascript
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
    	deleteProperty(trapTarget, key) {
            if (key === "value") {
                return false
            } else {
                return Reflect.deleteProperty(trapTarget, key)
            }
        }
 
})
```

### 原型代理陷阱

```javascript
let target = {}
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return Reflect.getPrototypeOf(trapTarget);
    },
    setPrototypeOf(trapTarget, proto) {
        return Reflect.setPrototypeOf(trapTarget, proto);
    }
})
```

### 对象可扩展性陷阱

Reflect.isExtensible

Reflect.preventExtensions

### 属性描述符陷阱

Reflect.defineProperty

### 描述符对象限制

Reflect.getOwnPropertyDescriptor

### ownKeys陷阱

Reflect.ownKeys()

### 函数代理中的apply和construct陷阱

Reflect.apply

### 可撤销代理

可以使用Proxy.revocable()方法创建可撤销的代理，该方法采用与Proxy构造函数相同的参数：目标对象和代理处理程序。

- proxy 可被撤销的代理对象
- revoke 撤销代理要调用的函数

```javascript
let target = {
    name: "target"
}

let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name); "target"

revoke();

//抛出错误
console.log(proxy.name)
```

## 用模块封装代码

export和import的一个重要的限制是：他们必须在其他语句和函数之外使用。

#### 导入绑定的一个微妙怪异之处

```javascript
export var name = "Nicholas";
export function setName(newName) {
    name = newName
}

import { name, setName } from './example.js'

console.log(name); // "Nicholas"
setName("Greg");
console.log(name); // "Greg"

name = "Nicholas" // 抛出错误
```

#### Web浏览器中的模块加载顺序

用<script type="module">显示引入和用import隐式导入的所有模块都是按需加载并执行的。

## ES7

### 指数运算符

``` javascript
let result = 5**2

console.log(result);  // 25
console.log(result === Math.pow(5, 2)) // true
```

求幂运算符在JavaScript所有二进制运算符中具有最高的优先级。
