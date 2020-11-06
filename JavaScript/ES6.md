# ES6
## let和const命令
### 变量提升
变量提升：变量可以在声明之前使用，值为undefined。理解为在程序编译的时候，会对于var声明的变量先进行定义。
```
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```
### TDZ暂时性死区
ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

```
var tmp = 123;
if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
### const 命令
const实际上保证的是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。
```
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```
如果真的想将对象冻结，应该使用Object.freeze方法。如果想将对象属性也冻结，可以参考以下函数。
```
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```
## 变量的解析构赋值
### 数组的解析构赋值
#### 基本用法
ES6允许按照一定模式，从数组和对象中提取，对变量进行赋值，这被称为解构（Destructuring）。
数据结构具有Iterator接口，都可以采用数组形式的结构赋值。这里需要参考Iterator部分。
#### 默认值
结构赋值允许指定默认值
默认值生效条件：***只有当一个数组成员严格等于 undefined，默认值才会生效。***
```
let [foo = true]=[]
foo//true
let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

上面代码中入如果一个数组成员是null，默认值就不会生效，因为null不严格等于undefined。
```
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```
如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。
```
function f() {
  console.log('aaa');
}

let [x = f()] = [1];
```
上面代码中，因为x能取到值，所以函数f根本不会执行。上面的代码其实等价于下面的代码。
```
let x;
if ([1][0] === undefined) {
  x = f();
} else {
  x = [1][0];
}
```
默认值可以引用解构赋值的其他变量，但该变量必须已经声明。
```
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
```
最后一个表达式之所以报错，是因为x用y做默认值时，y还没有声明。
### 对象的解构赋值
#### 简介
对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。对象的解构时，等号两边的同名属性次序可以不一致，对取值没有影响。解构失败，变量的值等于undefined。

对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。
```
// Math对象的对数、正弦、余弦三个方法，赋值到对应的变量上，使用起来就会方便很多
let { log, sin, cos } = Math;

// 将console.log赋值到log变量
const { log } = console;
log('hello') // hello
```

如果变量名与属性名不一致，必须写成下面这样。
```
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。
下面代码中，foo是匹配的模式，baz才是变量。真正被赋值的是变量baz，而不是模式foo。
```
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"
foo // error: foo is not defined
```
下面代码有三次解构赋值，分别是对loc、start、line三个属性的解构赋值。注意，最后一次对line属性的解构赋值之中，只有line是变量，loc和start都是模式，不是变量。
```
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
```
#### 默认值
对象的解构也可以指定默认值
```
var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```
### 字符串的解构赋值
字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。
```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```
类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值。
```
let {length : len} = 'hello';
len // 5
```
### 数值和布尔值的解构赋值 
解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

其实，数值和布尔值就是继承于对象的。
```
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```
数值和布尔值的包装对象都有toString属性，因此变量s都能取到值。

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。
### 函数参数的解构赋值

```
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

上面代码中，函数move的参数是一个对象，通过对这个对象进行解构，得到变量x和y的值。如果解构失败，x和y等于默认值。

注意，下面的写法会得到不一样的结果。


```
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

上面代码是为函数move的参数指定默认值，而不是为变量x和y指定默认值，所以会得到与前一种写法不同的结果。

undefined就会触发函数参数的默认值。


```
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```
### 圆括号的问题

### 解构的用途
#### （1）交换变量的值

```
let x = 1;
let y = 2;

[x, y] = [y, x];
// 交换数组的值值好像就不行???
```
上面代码交换变量x和y的值，这样的写法不仅简洁，而且易读，语义非常清晰。
#### （2）从函数返回多个值

函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。


```
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

#### （3）函数参数的定义

解构赋值可以方便地将一组参数与变量名对应起来。


```
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

#### （4）提取 JSON 数据

解构赋值对提取 JSON 对象中的数据，尤其有用。


```
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

上面代码可以**快速**提取 JSON 数据的值。

#### （5）函数参数的默认值


```
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写var foo = config.foo || 'default foo';这样的语句。

#### （6）遍历 Map 结构

任何部署了 Iterator 接口的对象，都可以用for...of循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。


```
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想获取键名，或者只想获取键值，可以写成下面这样。


```
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

#### （7）输入模块的指定方法

加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

```
const { SourceMapConsumer, SourceNode } = require("source-map");
```


## 字符串的扩展
### 字符的 Unicode 表示法
码元与码点：与计算机编程有关，早期的时候，存储空间比较宝贵，unicode存储文字，16位2进制叫做一个码元（code unit）计算机发展，对unicode文字进行了扩展，将某些文字扩展到了32位，占了两个码元，兼中国这些二进制的字符叫做码点（code point）
JavaScript 共有 6 种方法可以表示一个字符
```
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```
### 字符串的遍历器接口
ES6 为字符串添加了遍历器接口（详见《Iterator》一章），使得字符串可以被for...of循环遍历。

```
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
```

除了遍历字符串，这个遍历器最大的优点是可以识别大于0xFFFF的码点，传统的for循环无法识别这样的码点。


```
let text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "

for (let i of text) {
  console.log(i);
}
// "𠮷"
```

上面代码中，字符串text只有一个字符，但是for循环会认为它包含两个字符（都不可打印），而for...of循环会正确识别出这一个字符。
### 模板字符串
模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。所有模板字符串的空格和换行，都是被保留的。模板字符串中嵌入变量，需要将变量名写在${}之中。

```
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 传统写法为
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```
大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性。
```
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```
模板字符串之中还能调用函数。
```
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```
如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的toString方法。


如果需要引用模板字符串本身，在需要时执行，可以写成函数。


```
let func = (name) => `Hello ${name}!`;
func('Jack') // "Hello Jack!"
```

上面代码中，模板字符串写成了一个函数的返回值。执行这个函数，就相当于执行这个模板字符串了。
## 字符串的新增方法
JavaScript 内部，字符以 UTF-16 的格式储存，每个字符固定为2个字节。对于那些需要4个字节储存的字符（Unicode 码点大于0xFFFF的字符），JavaScript 会认为它们是两个字符。
### 静态方法
####  1. String.fromCodePoint()
ES5 提供String.fromCharCode()方法，用于从Unicode码点返回对应字符，但是这个方法不能识别码点大于0xFFFF的字符。码点大于0xFFFF，高位溢出。ES6提供了String.fromCodePoint()方法，可以识别大于0xFFFF的字符，弥补了String.fromCharCode()方法的不足。在作用上，正好与下面的codePointAt()方法相反。
#### 2. String.raw()
ES6 还为原生的 String 对象，提供了一个raw()方法。该方法返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。
### 实例方法
#### codePointAt()
4个字节的字符，JavaScript 不能正确处理，字符串长度会误判为2，而且charAt()方法无法读取整个字符，charCodeAt()方法只能分别返回前两个字节和后两个字节的值。

ES6 提供了codePointAt()方法，能够正确处理 4 个字节储存的字符，返回一个字符的码点。
codePointAt()方法返回的是码点的十进制值，如果想要十六进制的值，可以使用toString()方法转换一下。
```
let s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"

```
codePointAt()方法的参数，仍然是不正确的。比如，上面代码中，字符a在字符串s的正确位置序号应该是 1，但是必须向codePointAt()方法传入 2。解决这个问题的一个办法是使用for...of循环，因为它会正确识别 32 位的 UTF-16 字符。


```
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```

另一种方法也可以，使用扩展运算符（...）进行展开运算。


```
let arr = [...'𠮷a']; // arr.length === 2
arr.forEach(
  ch => console.log(ch.codePointAt(0).toString(16))
);
// 20bb7
// 61
```
#### 其他实例方法
- includes()：返回布尔值，表示是否找到了参数字符串。
- startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
- endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。
```
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```
- repeat()：返回一个新字符串，表示将原字符串重复n次。
参数是一个转化为整型之后非负的数。
```
'hello'.repeat(2) // "hellohello"
```
- padStart()，padEnd()

如果某个字符串不够指定长度，会在头部或尾部补全。padStart()用于头部补全，padEnd()用于尾部补全。常见用途：

(1)数值补全指定位数

```
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```
(2)提示字符串格式
```
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```
- trimStart()，trimEnd()

它们的行为与trim()一致，trimStart()消除字符串头部的空格，trimEnd()消除尾部的空格。它们返回的都是新字符串，不会修改原始字符串。除了空格键，这两个方法对字符串头部（或尾部）的 tab 键、换行符等不可见的空白符号也有效。trimLeft()是trimStart()的别名，trimRight()是trimEnd()的别名。

const s = '  abc  ';

s.trim() // "abc"
s.trimStart() // "abc  "
s.trimEnd() // "  abc"
## 正则的扩展
### 1.ES6的RegExp
第一种情况是，参数是字符串，这时第二个参数表示正则表达式的修饰符（flag）。

```
var regex = new RegExp('xyz', 'i');
// 等价于
var regex = /xyz/i;
```

第二种情况是,第一个参数是一个正则对象，第二个参数指定修饰符。而且，返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。
```
new RegExp(/abc/ig, 'i').flags
// "i"
```

上面代码中，原有正则对象的修饰符是ig，它会被第二个参数i覆盖。
### 2.字符串的正则方法
字符串对象共有 4 个方法，可以使用正则表达式：`match()`、`replace()`、`search()`和`split()`。

ES6 将这 4 个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全都定义在RegExp对象上。

`String.prototype.match` 调用 `RegExp.prototype[Symbol.match]`

`String.prototype.replace` 调用`RegExp.prototype[Symbol.replace]`

`String.prototype.search` 调用 `RegExp.prototype[Symbol.search]`

`String.prototype.split `调用` RegExp.prototype[Symbol.split]`
### 3. u修饰符
### 4. y修饰符
### 5. s修饰符：dotAll 模式
### 6. 后行断言 
### 7. Unicode 属性类
### 8. 具名组匹配
### 9. 正则匹配索引
### 10. String.prototype.matchAll() 
## 数值的扩展
## 函数的扩展
## Promise
### 模型
参考了github上找了一份[关于Promise的项目](https://github.com/barretlee/myPromise),看了挺久，感觉还不错。

```
new Promise(ready).then(getTpl).then(getData).then(makeHtml).resolve();
```

先将要事务按照执行顺序依次 push 到事务队列中，push 完了之后再通过 resolve 函数启动整个流程。

整个流程的操作模型如下：

```
promise(ok).then(ok_1).then(ok_2).then(ok_3).reslove(value)------+
         |         |          |          |                       |
         |         |          |          |        +=======+      |
         |         |          |          |        |       |      |
         |         |          |          |        |       |      |
         +---------|----------|----------|--------→  ok() ←------+
                   |          |          |        |   ↓   |
                   |          |          |        |   ↓   |
                   +----------|----------|--------→ ok_1()|
                              |          |        |   ↓   |
                              |          |        |   ↓   |
                              +----------|--------→ ok_2()|
                                         |        |   ↓   |
                                         |        |   ↓   |
                                         +--------→ ok_3()-----+
                                                  |       |    |       
                                                  |       |    ↓
@ Created By Barret Lee                           +=======+   exit
```

在 resolve 之前，promise 的每一个 then 都会将回调函数压入队列，resolve 后，将 resolve 的值送给队列的第一个函数，第一个函数执行完毕后，将执行结果再送入下一个函数，依次执行完队列。
### 基本用法
#### 状态及特点
状态类型： pending（进行中）、fufilled(已成功)和rejected（已失败）
1. 对象的状态不受外界影响。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。（pending->resolved）
#### 创建实例
```
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```
Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。

参数 | 描述
---|---
resolve | 将Promise对象的状态从pending变为fulfilled，在异步操作成功时调用，并将异步操作得到的结果，作为参数传递出去
reject | 将Promise对象的状态从pending变为rejected，在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去



立即 resolved 的 Promise 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。
### 方法
#### Promise.prototype.then()
Promise.prototype.then()的作用是为 Promise 实例添加状态改变时的回调函数。then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。

```
Promise.prototype.then(callback1,callback2)
```

参数 | 描述
---|---
callback1 | fulfilled状态的回调函数
callback2 | rejected状态的回调函数

这里将Promise理解为一个异步的事务管理器。
```
// 初始化事务管理器
var promise = new Promise(function(data){
    console.log(data);
    return 1;
});
// 添加事务
promise.then(function(data){
    console.log(data);
    return 2;
}).then(function(data){
    console.log(data);
    return 3;
}).then(function(data){
    console.log(data);
    console.log("end");
});
// 启动事务
promise.resolve("start");
```

#### Promise.prototype.catch()
Promise.prototype.catch()用于指定发生错误时的回调函数。
1. 在Promise对象状态变为rejected，就会调用catch()方法指定的回调函数。
2. then()方法指定的回调函数，运行中抛出的错误throw new Error('test')，也会被catch()方法捕获。

```
Promise.prototype.catch(callback)
```

参数 | 描述
---|---
callback | catch()方法捕获到错误信息之后的回调函数
```
Promise.prototype.catch()
// 等价于
Promise.prototype.then(null, rejection)
// 等价于
Promise.prototype.then(undefined, rejection)
```
注意：
1. 如果 Promise 状态已经变成resolved，再抛出错误是无效的。（状态无法改变）
2. Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。

#### Promise.prototype.finally()
 Promise.prototype.finally()方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。**finally方法总是会返回原来的值**。
 
```
Promise.prototype.finally(callback)
```
参数 | 描述
---|---
callback | 指定的一定执行的回调函数
```
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```

#### Promise.all() 

Promise.all()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。

```
Promise.all(arr)

```

参数 | 描述
---|---
arr | 一个Promise实例的数组


```
// 例如
const p = Promise.all([p1,p2,p3])
```

Promise.all()状态由p1、p2、p3决定，分成两种情况。

（1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

**简而言之：**只有在实例们的状态都为fulfilled或者其中一个实例rejectd的时候，才会调用Promise.all方法后面的回调函数。

#### Promise.race()
Promise.race()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。
```
Promise.race(arr)

```

参数 | 描述
---|---
arr | 一个Promise实例的数组


```
// 例如
const p = Promise.race([p1,p2,p3])
```
Promise.race()状态由p1、p2、p3决定:
只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。


```
// 如果 5 秒之内fetch方法无法返回结果，变量p的状态就会变为rejected，从而触发catch方法指定的回调函数。
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);

p
.then(console.log)
.catch(console.error);
```
#### Promise.allSettled() 
Promise.allSettled() 方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。
 
 
```
Promise.allSettled(arr) 
```
参数 | 描述
---|---
arr | 一个Promise实例的数组

只有等到所有这些参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。

```
const resolved = Promise.resolve(42);
const rejected = Promise.reject(-1);

const allSettledPromise = Promise.allSettled([resolved, rejected]);

allSettledPromise.then(function (results) {
  console.log(results);
});
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```
相较于Promise.all()方法:
1. Promise.all()只有当所有实例状态转为fulfilled,返回fulfilled,这时候才确保所有实例操作都结束。
2. Promise.allSettled()只有等到所有实例都返回结果，才返回状态且只可能变成fulfilled。

```
const urls = [ /* ... */ ];
const requests = urls.map(x => fetch(x));

try {
  await Promise.all(requests);
  console.log('所有请求都成功。');
} catch {
  console.log('至少一个请求失败，其他请求可能还没结束。');
}
```

#### Promise.any() 
Promise.any()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。

Promise.any()抛出的错误，不是一个一般的错误，而是一个 AggregateError 实例。它相当于一个数组，每个成员对应一个被rejected的操作所抛出的错误。

```
Promise.any(arr) 
```
参数 | 描述
---|---
arr | 一个Promise实例的数组


```
var resolved = Promise.resolve(42);
var rejected = Promise.reject(-1);
var alsoRejected = Promise.reject(Infinity);

Promise.any([resolved, rejected, alsoRejected]).then(function (result) {
  console.log(result); // 42
});

Promise.any([rejected, alsoRejected]).catch(function (results) {
  console.log(results); // [-1, Infinity]
});
```
#### Promise.resolve()

Promise.resolve()将现有对象转为 Promise 对象，Promise.resolve()方法就起到这个作用。

Promise.resolve()方法的参数分成四种情况：

（1）参数是一个 Promise 实例

如果参数是 Promise 实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。

（2）参数是一个thenable对象

thenable对象指的是具有then方法的对象，比如下面这个对象。Promise.resolve()方法会将这个对象转为 Promise 对象，然后就立即执行thenable对象的then()方法。

```
/* thenable对象的then()方法执行后，对象p1的状态就变为resolved，
*从而立即执行最后那个then()方法指定的回调函数，输出42。
*/
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function (value) {
  console.log(value);  // 42
});
```

（3）参数不是具有then()方法的对象，或根本就不是对象

如果参数是一个原始值，或者是一个不具有then()方法的对象，则Promise.resolve()方法返回一个新的 Promise 对象，状态为resolved。


```
const p = Promise.resolve('Hello');

p.then(function (s) {
  console.log(s)
});
// Hello
```

上面代码生成一个新的 Promise 对象的实例p。由于字符串Hello不属于异步操作（判断方法是字符串对象不具有 then 方法），返回 Promise 实例的状态从一生成就是resolved，所以回调函数会立即执行。Promise.resolve()方法的参数，会同时传给回调函数。

（4）不带有任何参数

Promise.resolve()方法允许调用时不带参数，直接返回一个resolved状态的 Promise 对象。

注意：立即resolve()的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。

```
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
```
上面代码中，setTimeout(fn, 0)在下一轮“事件循环”开始时执行，Promise.resolve()在本轮“事件循环”结束时执行，console.log('one')则是立即执行，因此最先输出。

#### Promise.reject(reason)

Promise.reject()方法也会返回一个新的 Promise 实例，该实例的状态为rejected。
```
Promise.reject(reason)
```
参数 | 描述
---|---
reason | reject的理由

Promise.reject()方法的参数，会原封不动地作为reject的理由，变成后续方法的参数。


```
Promise.reject('出错了')
.catch(e => {
  console.log(e === '出错了')
})
// true
```
