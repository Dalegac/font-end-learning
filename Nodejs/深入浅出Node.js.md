# 第一章Node简介
## 1. Node的特点
### 异步IO
```
var fs = require('fs'); 
fs.readFile('/path', function (err, file) { 
 console.log('读取文件完成') 
}); 
console.log('发起读取文件')
```

![node异步调用](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_201025074259node%E5%BC%82%E6%AD%A5%E8%B0%83%E7%94%A8.png)

### 事件与回调函数
下面的例子展示的是Ajax异步提交的服务器端处理过程。Node创建一个Web服 务器，并侦听
8080端口。对于服务器，我们为其绑定了request事件，对于请求对象，我们为其绑定了data事件和end事件:
```
var http = require('http'); 
var querystring = require('querystring'); 
// 监听服务器的request事件
http.createServer(function (req, res) { 
 var postData = ''; 
 req.setEncoding('utf8'); 
 // 监听请求的data事件
 req.on('data', function (trunk) { 
 postData += trunk; 
 }); 
 // 监听请求的end事件
 req.on('end', function () { 
 res.end(postData); 
 }); 
}).listen(8080); 
console.log('服务器启动ྜ成');
```
相应地，我们在前端为Ajax请求绑定了success事件，在发出请求后，只需关心请求成功时
执行相应的业务逻辑即可，相关代码如下:
```
$.ajax({ 
 'url': '/url', 
 'method': 'POST', 
 'data': {}, 
 'success': function (data) { 
 // success事件
 } 
});
```
回调函数也是最好的接受异步调用返回数据的方式
### 单线程
Node保持了JavaScript在浏览器中单线程的特点。而且在Node中，JavaScript与其余线程是无法共享任何状态的。单线程的最大好处是不用像多线程编程那样处处在意状态的同步问题，这里没有死锁的存在，也没有线程上下文交换所带来的性能上的开销。
单线程的弱点：
- 无法利用多核CPU
- 错误会引起整个应用推出，应用的健壮性值得考验。
- 大量计算占用CPU导致无法继续调用异步I/O。

在Node中，长时间的CPU占用会导致后续的异步I/O发不出调用，已完成的异步I/O的回调函数也会得不到及时执行。

Node采用了与Web Workers相同的思路来解决单线程中大计算量的问题:child_process。

子进程的出现，意味着Node可以从容地应对单线程在健壮性和无法利用多核CPU方面的问题。通过将计算分发到各个子进程，可以将大量计算分解掉,然后再通过进程之间的事件消息来传递结果，这可以很好地保持应用模型的简单和低依赖。通过Master-Worker的管理方式，也可以很好地管理各个工作进程，以达到更高的健壮性。
### 跨平台
![node跨平台](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_201025093750Node%E8%B7%A8%E5%B9%B3%E5%8F%B0.png)

兼容Windows和*nix平台主要得益于Node在架构层面的改动,它在操作系统与Node上层模块系统之间构建了一层平台层架构,即libuv。目前, libuv已经成为许多系统实现跨平台的基础组件。

## 2.Node的应用场景
### I/O密集型
如果将所有的脚本语言拿到一处来评判，那么从单线程的角度来说，Node处理I/O的能力是值得竖起大拇指称赞的。通常，说N哦的擅长I/O密集型的应用场景基本上是没人反对的。Node面向网络且擅长并行I/O，能有效的组织起更多的硬件资源，从而提供更好的服务。

I/O密集的优势主要在于Node利用事件循环的处理能力，而不是启动每一个线程为每一个请求服务，资源占用极少。
### 是否不擅长CPUI密集型业务
- V8的执行效率是十分高的。
- 充分利用CPU的两种方式。
- - Node可以通过编写C/C++扩展的方式更高效地利用CPU，将一些V8不能做到性能极致的地方通过C/C++来实现。
- - 如果单线程的Node不能满足需求,甚至用了C/C++扩展后还觉得不够,那么通过子进程的方式，将一部分Node进程当做常驻服务进程用于计算，然后利用进程间的消息来传递结果，将计算与I/O分离，这样还能充分利用多CPU。


CPU密集型应用给Node带来的挑战主要是:由于JavaScript单线程的原因，如果有长时间运行的计算(比如大循环)，将会导致CPU时间片不能释放，使得后续I/O无法发起。但是适当调整和分解大型运算任务为多个小任务，使得运算能够适时释放，不阻塞I/O调用的发起，这样既可同时享受到并行异步I/O的好处，又能充分利用CPU。
关于CPU密集型应用,Node的异步I/O已经解决了在单线程上CPU与I/O之间阻塞无法重叠利用的问题，I/O阻塞造成的性能浪费远比CPU的影响小。对于长时间运行的计算，如果它的耗时超过普通阻塞I/O的耗时,那么应用场景就需要重新评估。

# 第二章模块机制

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS和AMD两种。前者用于服务器，后者用于浏览器。ES6在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS和AMD规范，成为浏览器和服务器通用的模块解决方案。
ES6和CommonJS的区别：
- CommonJS 模块输出的是一个值的拷贝，ES6 - 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
- CommonJS 模块的require()是同步加载模块，ES6 模块的import命令是异步加载，有一个独立的模块依赖的解析阶段。
### 1.CommonJSde 模块规范
#### 模块引用、模块定义、模块识别
- 使用require()方法接受模块标识，一次引入一个模块的API到当前上下文中。
- 上下文提供了exports对象用于导出当前模块的方法或者变量，并且他是惟一导出的出口。在模块中还存在一个module对象，他代表模块自身，而exports是module的属性。
- 模块标志其实就是传递给require()方法的参数。它必须是符合小驼峰命名的字符串，或者以`.`、`..`开头的相对路径，或者绝对路径。它可以没有文件后缀`.js`，Node会按.js、.json、.node的次序补足扩展名

## 1.Node的模块实现
在Node中引入模块，需要经历三个步骤：
- 路径分析
- 文件定位
- 编译执行

在Node中，模块分为两类：一类是Node提供的模块，称为核心模块；另一类是用户编写的模块，称为文件模块。
- 核心模块部分在Node源代码的编译过程中，编译进了二进制执行文件。在Node进程启动
时，部分核心模块就被直接加载进内存中，所以这部分核心模块引人时，文件定位和编
译执行这两个步骤可以省略掉，并且在路径分析中优先判断，所以它的加载速度是最快的。
- 文件模块则是在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程，速
度比核心模块慢。

### 优先从缓存加载
Node对引用过的模块都会进行缓存，减少二次引入时的开销。Node缓存的是编译和执行之后的对象。
### 路径分析和文件定位
- 模块标识符分析
    - 核心模块。优先级仅次于缓存加载，他在Node的源代码编译过程中，已经编译为二进制代码，其加载过程最快。
    - 路径形式的文件模块。reqire()方法将路径转为真实路径，以此作为索引，将编译执行后的结果存放到缓存中，以使得二次加载时更快。
    - 自定义模块。从当前目录开始，沿路径向上逐级递归，知道根目录下的node_modules.
- 文件定位
    - 文件扩展名分析。所以这里是一个会引起性能问题的地方。小诀窍是:如果是node和.json文件,在传递给require()的标识符中带上扩展名，会加快一点速度。另一个诀窍是:同步配合缓存，可以大幅度缓解Node单线程中阻塞式调用的缺陷。
    - 目录分析和包。未找到对应文件之后，首先，解析package.json,取出main属性指定的文件名进行定位（index是默认文件名）。

### 模块编译
- .js文件。通过fs模块同步读取文件后编译执行(头尾包装)。
- .node文件。这是用C/C++编写的扩展文件，通过dlopen()方法加载最后编译生成的文件。
- .json文件。通过fs模块同步读取文件后，用JSON.parse()解析返回结果。
- 其余扩展名文件。它们都被当做js文件载人。

## 2. 核心模块
### JavaScript核心模块的编译过程
### C/C++核心模块的编译过程
### 模块的引入流程
### 编写核心模块
## 3. C/C++扩展模块
![node中C++模块在不同平台下的编译](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010271339153.jpg)
### 前提条件
- 深厚的C/C++编程功底。
- GYP项目生成工具。
- V8引擎C++库。
- libuv库。
- Node内部库。
- 其他库。

### C/C++扩展模块的编写
普通的扩展模块与内建模块的区别在于无需将源代码编译进Node，二十通过dlopen()方法动态加载。编写普通的扩展模块是，无需将源码写进Node命名空间，也不需要提供头文件，将方法怪哉在target对象上，然后同各国NODE_MODULE声明即可。
```
#include <node. h>
#include <v8.h>
using namespace v8; 
// 实现预定义的方法
Handle<Value> SayHello(const Arguments& args) {
    HandleScope scope;
    return scope . Close( Str ing ::New("Hellow world!"));
}

//给传入的目标对象添加sayHello()方法
void Init. Hello(Handle<0bject> target) {
    target->Set(String::NewSymbol("sayHello"),FunctionTemplate::New(SayHello)->GetFunction());
}

//调用NODE_ MODULE()方法将注册方法定义到内存中
NODE_MODULE(hello, Init_Hello)
```
### C/C++扩展模块的编译
使用DYP工具，node-gyp约定.gyp文件为binding.gyp
```
{
    'target':[
    {
        'target_name':'hello',
        'sources':[
            'src/hello.cc'
        ],
        'conditions':[
        ['OS=="win"',
        {
            'libraries':['-lnode.lib']
        }
        ]
        ]
    }]
}
```
编译过程会工具平台不同，分别通过make或vcbuild进行编译。编译完成后，hello.node文件会生成在build/Release目录下。
### C/C++扩展模块的加载
![require()加载模块](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_201027142900require()%E5%8A%A0%E8%BD%BD%E6%A8%A1%E5%9D%97.jpg)
## 4.包与NPM
![包组织模块示意图](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_201027144400%E5%8C%85%E7%BB%84%E7%BB%87%E6%A8%A1%E5%9D%97%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)

### 包结构
- package.json: 包描述文件。
- bin: 用于存放可执行二进制文件的目录。
- lib: 用于存放Javascript代码的目录。
- doc: 用于存放文档的目录。
- test: 用于存放单元测试用例的代码。

### 包描述文件与NPM
CommonJS为package.json文件定义了如下一些必须的字段。
- name。包名。
- description。包简介。
- version。版本号。
- keywords。关键词数组，NPM中主要用来做分类搜索。
- maintainers。包维护者列表。权限认证。
- contributors。贡献者列表。
- bugs。一个可以反馈bug的网页地址或者邮件地址。
- licenses。当前包所使用的许可证列表，表示这个包可以在那些许可证下使用。
- repositories。托管源代码的位置列表，表明可以通过哪些方式和地址访问包的源代码。
- dependencies。使用当前包所需要依赖的包列表。
- homepage。当前包的网站地址。
- os。操作系统支持列表。
- cpu。CPU架构的支持列表，有效的架构名称有arm、mmmips、ppc、sparc、x86、x86_64。
- engine。支持的Javascript引擎列表，有效的引擎取值包括ejs、flusspferd、gpsee、jsc、spider monkey、narwhal、node和v8。
- builtin。标志当前包是否是内建在底层系统的标准组件。
- directories。包目录说明。
- implements。实现规范的列表。
- scripts。脚本说明对象。它主要被包管理器用来安装、编译、测试和卸载包。


包规范的区别在于多了author、bin、main和devDependencies这四个字段。
- author。包作者。
- bin。一些包作者希望包可以做为命令行工具使用。
- main。模块引入方法require()在引入包时，会优先检查这个字段，并将其作为包中其余模块的入口。
- devDependencies。一些模块只在开发时需要依赖。

### NPM常用功能
- 查看帮助
- 安装依赖包
- NPM钩子命令
- 发布包
- 分析包

### 局域NPM
### NPM存在的问题
## 5. 前后端共用模块
### AMD规范
模块定义：define(id?, dependencies?, factory);
它的模块id和依赖是可选的。
```
define(function() { 
 var exports = {}; 
 exports.sayHello = function() { 
 alert('Hello from module: ' + module.id); 
 }; 
 return exports; 
});
```
AMD模块需要用define来明确定义一个模块，而在Node实现中是隐士包装的，他们的目的是进行作用域隔离，仅在需要的时候被引入，避免掉过去那种通过全局变量或者全局命名空间的方式，以免变量污染和不小心被修改。另一个区别则是内容需要通过返回的方式实现导出。
# 第三章异步I/O
[线程池概念](https://www.jianshu.com/p/7726c70cdc40)

PHP语言从头到尾都是以同步阻塞的发士来执行的。与Noode的事件驱动、异步I/O设计理念比较相近的一个知名产品为Nginx。
## 1. 为什么要异步I/O
### 用户体验
《高性能Javascript》一书中总结过，如果脚本的执行时间超过100ms，用户就会感到页面卡顿，以为网页停止响应。采用异步请求，在下载资源期间，JavaScript和UI的执行都不会处于等待状态，可以继续响应用户的交互行为，给用户一个鲜活的页面。同步的情景耗时：M+N+···,异步的情景耗时：max(M,N,···)。随着网站或者应用不断膨胀，数据会分布到堕胎服务器上，分布式将会是常态。
### 资源分配
多线程的代价在于创建线程和执行期线程上下文切换的开销较大。在复杂的业务中多线程编程经常面临锁、状态同步等问题，这是多线程被诟病的主要原因。Node方案：利用单线程，远离多线程死锁、状态同步等问题；利用异步I/O，让单线程原理阻塞，以更好地使用CPU，使用子进程高效的利用CPU和I/O。
## 2. 异步I/O实现现状
### 异步I/O与非阻塞I/O

- 阻塞I/O一个特点是调用之后一定要等到系统内核层面完成所有操作后，调用才结束。
- 非阻塞I/O调用之后会立即返回。需要轮询去确认是否完全完成数据获取。

### 理想中的异步I/O
![理想中的异步I/O](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010291506571603983969(1).jpg)
### 现实的异步I/O
通过让部分线程进行阻塞I/O或者非阻塞I/O加轮询技术来完成数据获取，让一个线程进行计算处理，通过线程之间的通信将I/O得到的数据进行传递。
![异步I/O](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010291513211603984366(1).jpg)
![libuv的架构示意图](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010291515381603984492(1).jpg)

## 3. Node的异步I/O
### 事件循环
首先,我们着重强调一下Node自 身的执行模型一事件 循环，正是它使得回调函数十分普遍。

在进程启动时，Node便会创建一个类似于while(true)的循环，每执行一次循环体的过程我们称为Tick。每个Tick的过程就是查看是否有事件待处理，如果有，就取出事件及其相关的回调函数。如果存在关联的回调函数,就执行它们。然后进入下个循环，如果不再有事件处理，就退出进程。流程图如图3-11所示。


![Tick流程图](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_201029152106WPIYAG(Z)T361RUQY8LX%7B0I.png)

### 观察者
每个事件循环中有一个或者多个观察者，而判断视频有事件要处理的过程就是向这些观察者询问是否有要处理的事件。

事件循环是一个典型的生产者/消费者模型。异步I/O
、网络请求等则是事件的生产者，这些事件传递到对应的观察者中，事件循环则从观察者那里取出事件并处理。

在Windows下，这个循环基于IOCP创建，而在*nix下则基于多线程创建。
### 请求对象
从JavaScript发起电泳到内核执行完I/O操作的过渡过程中，存在一种中间产物，他叫做请求对象。请求对象保存了所有的状态，包括送入线程池等待执行一级I/O操作完毕后的回调处理。
### 执行回调
![整个异步I/O的流程](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010291531291603985468(1).jpg)

事件循环、观察者、请求对象、I/O线程池这四者共同构成了Node异步I/O模型的基本要素。
## 4. 非I/O的异步API
Node中存在一些I/O无关的异步API，分别是setTimeout()、setInterval()、steImmediate()和proccess.nextTick()。
### 定时器
它们的实现原理与异步I/O比较类似，只是不需要I/O线程池的参与。调用setTimeout()或者setInterval()创建的定时器会被插入到定时器观察者内部的一个红黑树中。 每次Tick执行时，会从该红黑树中迭代取出定时器对象，检查是否超过定时时间，如果超过,就形成一一个事件，它的回调函数将立即执行。定时器需要在时间片，运行之后执行，它并非精确的（在容忍范围内）。
![setTimeout()的行为](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010291548321603986490(1).jpg)
### process.nextTick()
具体代码

```
process. nextTick = function(callback) {
// on the way out, don't bother .
// it won't get fired anyway
if (process. exiting) retumn;
if (tickDepth >= process .maxTickDepth)
maxTickWarn();
var tock = { callback: callback };
if (process .domain) tock .domain = process . domain;
nextTickQueue . push(tock);
if (nextTickQueue.length) {
process. _needTickCallback();
}
};
```
每次调用proccess.nextTick()方法，只会讲回调函数放入队列中，在下一轮Tick时去除执行。定时器采用红黑树的操作时间复杂度为O(lg(n)),nextTick()的时间复杂为O(1)。
### setImmediate()
setImmediate)()和process.nextTick()都是将回调函数延迟执行。但是时间循环对观察者的检查是有先后顺序的，processnextTick()属于idle观察者，setImmediate()属于check观察者。在每一轮循环检查中，idle观察者优先于I/O观察者，I/O观察者先于check观察者。
```
process.nextTick(function(){
    console.log('nextTick延迟执行');
})
setImmediate(function(){
    console.log('setImmediate延迟执行');
})
console.log('正常执行');

// 执行结果
正常执行
nexttTick延迟执行
setImmediate延迟执行
```
在具体的实现上，process.nextTick()的回调函数，保存在一个数组，setImmediate()的结果则是保存在链表中。行为上，process.nextTick()在每轮循环中会将数组中的回调函数全部执行完，而setImmediate()在每轮循环中执行链表中的一个回调函数。
```
//加入两个nextTick()的回调函数
process.nextTick(function () {
console.log('nextTick延迟执行1');
});
process.nextTick(function () {
console.log('nextTick延迟执行2');
});

//加入两个setImmediate()的回调函数
set Immediate(function () {
console.log('setImmediate延迟执行1' );

//进入下次循环
process.nextTick(function () {
console.1og('强势插入');
});
});
setImmediate(function () {
console.log(' setImmediate延迟执行2');
});
console.1og('正常执行' );

其执行结果如下:
正常执行
nextTick延迟执行1
nextTick延迟执行2
set Immediate延迟执行1
强势插入
set Immediate延迟执行2
```
从执行结果上可以看出，当第一个setImmediate()的回调函数执行后，并没有立即执行第二个，而是进入了下一轮循环，再次按process.nextTick()优先、steImmediate()次后的顺序执行。之所以这样设计，是为了保证每轮循环能够较快地执行结束，复制CPU占用过多而阻塞后续I/O调用的情况。
## 5. 事件驱动与高性能服务器
事件驱动的实质：通过主循环加事件触发的方式来运行程序。
![利用Node构建Web服务器的流程图](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2010300545071604036606(1).jpg)

Node通过事件驱动的方式处理请求，无需为每个请求创建额外的对应线程。可以省掉创建线程和销毁线程的开销。同时操作系统在调度任务时因为线程较少，上下文切换的代价很低。

# 异步编程
[JavaScript中的apply、call、bind](https://blog.csdn.net/u010176097/article/details/80348447?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight)
## 1. 函数式编程
### 高阶函数
高阶函数是可以把函数作为参数，或者将函数作为返回值的函数。
```
function foo (x){
    return function(){
        return x;
    };
}
```
除了通常意义的函数调用返回外，还形成了一种后续传递风格（Continuation Passing Style）的结果接收方式，而非单一的返回值形式。后续传递风格的程序编写将函数的业务重点从返回值转移到了回调函数中：
```
// 对于相同的foo()函数，传入的bar参数不同，则可以得到不同的结果。
function foo(x,bar){
    return bar(x);
}

var points = [40,100,1,5,25,10];
points.sort(function(a,b){
    return a-b;
});
// [1,5,10,25,40,100]
```
一些数组的方法：forEach()、map()、reduce()、reduceRight()、filter()、every()、some()都是高阶函数的应用。由于高阶函数，事件可以十分方便地进行业务逻辑的解耦。

### 偏函数用法
偏函数的用法是指创建一个调用另外一个部分————参数或者变量已经预置的函数————的函数的用法。
```
var isType = function (type){
    return function(obj){
        return toString.call(obj)=='[object' +type+']';
    };
};
var isString= isType('String');
var isFunction = isType('Function');
```
引入isType()函数后，创建isString()、isFunction()函数就变得简单多了。这种通过指定部分参数来产生一个新的定制函数的形式就是偏函数。

偏函数应用在异步编程中也十分常见，著名类库Underscore提供的after()方法就是偏函数应用：
```
_.after = function(times,func){
    if(times <= 0) return func();
    return function(){
        if(--time < 1){ 
        return func.apply(this,arguments);}
    };
};
```
这个函数可以根据传入的times参数和具体方法，生成一个需要调用多次才真正执行实际函数的函数。

## 2. 异步编程的优势与难点
### 优势
Node最大的特性：基于事件驱动的非阻塞I/O模型。
![异步I/O调用示意图](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2011040603481604469750.jpg)

![同步I/O调用示意图](https://images.cnblogs.com/cnblogs_com/dalegac/1825105/o_2011040610171604470168.jpg)
非阻塞I/O可以使CPU与I/O并不相互依赖等待让资源得到更好的利用。在网络应用、分布式和云中，并行使得各个单点之间能够更有效地组织起来。

### 难点
#### 异常处理
#### 函数嵌套过深
#### 阻塞代码
#### 多线程编程
#### 异步转同步

## 3. 异步编程解决方案
### 事件发布/订阅模式
[作者自己写的EventProxy](https://github.com/JacksonTian/eventproxy)

事件监听器模式是一种广泛用于异步编程的模式，是回调函数的事件化，又称发布/订阅模式。
```
// 订阅
emitter.on('event1',function(message){
    console.log(message);
});
//发布
emitter.emit('event1','I am message!');
```
订阅事件就是一个高阶函数。事件发布/订阅模式可以实现一个事件与多个回调函数的关联，这些回调函数又称为事件侦听器。

事件侦听器模式也是一种钩子（hook）机制，利用钩子导出内部数据或状态给外部的调用者。

Node对事件发布/订阅的机制做了一些额外的处理，着大多是基于健壮性而考虑的。

- 如果对一个事件添加了超过10个侦听器，将会得到一条警告这一处设计与Node自身单线程运行有关，设计者认为侦听器太多可能导致内存泄漏，所以存在这样一条警告。调用emitter.setMaxListeners(0);可以将这个限制去掉。另-方面，由于事件发布会引起一系列侦听器执行，如果事件相关的侦听器过多，可能存在过多占用CPU的情景。
- 为了处理异常，EventEmitter对象对error事件进行了特殊对待。如果运行期间的错误触发了error事件，EventEmitter会检查是否有对error事件添加过侦听器。如果添加了,这个错误将会交由该监听器处理，否则这个错误将会作为异常抛出。如果外部没有捕获这个异常，将会引起线程退出。一个健壮的EventEmitter实例应该对error事件做处理。

#### 继承events模块
#### 利用事件队列解决雪崩问题
#### 多异步之间的协作方案
#### EventProxy的原理
#### EventProxy的异常处理
### Promise/Defferred模式
### 流程控制库
## 4. 异步并发控制
### bagpiper的解决方案
### async的解决方案
# 内存控制
## 1. V8的垃圾回收机制与内存限制
- 在Node中通过JavaScript使用内存时就会发现智能使用部分内存（64位系统下约为1.4GB，32位系统下约为0.7G）
- 在V8中，所有的JavaScript对象都是通过堆进行分配的。值类型的通过栈进行分配。Node提供了V8中内存使用量的查看方式：
 ```
 $node
 >process.memoryUsage();
 {  rss: 14958592,
    heapTotal: 7195904,
    heapUsed: 2821496 }
 ```
 heapTotal是已申请到的堆内存，heapUsed是当前使用的量。rss是
 已申请的堆空闲内存不够分配给新的对象，将继续申请堆内存，直到堆的大小超过V8的限制为止。
 
 # 理解Buffer
 ps：接下来的章节内容和后端关系较大，粗略地翻阅了一下。就不整理笔记了，有的也需要实践coding，这本书的内容就先暂缓。node的学习整理了这几个方向：webpack react angular tarojs。感觉需要从实践开始。