# 你不知道的JavaScript

## 第一章 作用域是什么

### 编译原理

> 尽管将JavaScript 归类为“动态”或解释性的语言 但事实上它是一门编译语言
>
> 与传统编译语言不同的式，他不是提前编译的，编译结果也不能在分布式系统中进行移植
>
> javaScript 代码的编译过程不是发生在构建前的，而是发生在代码执行前的几微妙

**编译**

+ 分词/词法分析

  > 将字符组成的字符串分解成有意义的代码块，这些代码块被称为词法单元

+ 解析/语法分析

  > 这个过程式将于此单元流，转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树，这个树被称为抽象语法树

+ 代码生成

  > 将抽象语法数转换为可只想代码的过程被称为代码生成

比起编译过程只有三个步骤的语言的编译器，JavaScript 引擎要复杂得多，列如在语法分析和代码生成阶段由特定的步骤来对运行性能进行优化，包括对冗余元素进行优化

### 理解作用域

#### 演员表

> 列如程序 var a =2

+ 引擎

  > 从头到尾负责整个JavaScript 程序的编译及执行过程

+ 编译器

  > 引擎的好朋友之一，负责语法分析及代码生成

+ 作用域

  > 负责收集并维护所有声明的标识符，组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限

#### 对话

> 变量的赋值操作会执行两个动作
>
> 1. 编译器在当前作用域声明一个变量（如果之前没有被声明过）
> 2. 引擎运行时在作用域中查找该变量，如果找到就会对它赋值

对于`var a = 2`，编译器会进行如下处理

1. 遇到`var a`编译器会检查当前作用域下是否已经有一个该名称的变量存在，如果是，则忽略`var a`继续编译，否则它会要求`作用域`在当前作用域的集合中声明一个新的变量，并命名为a

2. 接着编译器会生成引擎运行时的代码，这些代码用来处理`a = 2`这个操作

   引擎运行时首先询问`作用域`在当前作用域集合中是否存在`变量a`如果村子啊，则使用此变量进行赋值，如果否，引擎会继续查找该变量，最终找到，进行赋值，找不到则抛出异常

#### 编译器有话说

> 引擎对变量a的查询，会影响最终的查找结果

对于`var a = 2`会进行LHS的查询，另一个查询为RHS

+ LHS查询

  > 列如 var a = 2

  + 当变量出现在赋值操作的左侧时进行LHS查询

  + LHS查询试图找到变量的容器本身，从而对其赋值

+ RHS查询

  > 列如 consolo.log(a)

  + 当变量出现在赋值操作的右侧时进行RHS查询

**既有LHS也有RHS的程序**

```javascript
function foo (a) {
    // 对a的RHS引用
	console.log(a)
}
// 对foo进行RHS引用，意味着“去找到foo的值”，带着()一位着foo要被执行
// 隐式的a = 2操作，进行了一次LHS查询
foo( 2 )
```

```javascript
function foo (a) {
    var b = a
    return a + b
}
var c = foo( 2 )

// 所有的LHS查询
c = ...
a = 2
b = ...
// 所有的RHS查询
foo()
 = a
a..
b..
```

### 作用域嵌套

当一个块或函数嵌套在另外一个块中，就发射管了作用域的嵌套

1. 首先， 在当前作用域中查找某个变量
2. 找到了则使用，无法找到则会在外层嵌套的作用域中继续查找，直到找到该变量，或抵达最外层的作用域

## 词法作用域

## 第三章 函数作用域和块作用域

## 第四章 提升

## 作用域闭包

> 当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包

```javascript
function foo () {
    var a = 2
    function bar () {
        console.log( a )
    }
    return bar
}

var baz = foo()
baz() // 2 这就是闭包
```

## 关于this

### 误解

#### 指向自身

> 人们很容易把this理解成指向函数自身

```javascript
function foo (num) {
	console.log("foo：" + num)
	this.count++
}
foo.count = 0

for (var i = 0; i < 10; i++) {
	if (i < 5) {
		foo( i )
	}
}
/*
	foo: 6
	foo: 7
	foo: 8
	foo: 9
*/
// foo 被调用了多少次
console.log(foo.count)  ==> 0
```

#### 指向它的作用域

> this在任何情况下都不指向函数的词法作用域（但可能指向函数的作用域）
>
> 作用域确实和对象类似，可见的标识符都是它的属性，但是作用域“对象”无法通过JavaScript代码访问，它存在于引擎内部

```javascript
function foo () {
    var a = 2
    this.bar()
}
function bar () {
    console.log(this.a)
}
foo() // ReferenceError: a is not defined
```



### this 到底是什么

+ this 实在运行时绑定的，并不是在编写时绑定的，它的上下文取决于函数调用时的各种条件，
+ this 的绑定和函数声明的位置没有任何关系，之取决于函数的调用方式

> 当一个函数被调用时，会创建一个活动巨鹿（执行上下文），这个记录会包含函数在哪里被调用，函数的调用方法，传入的参数等信息，this就是记录的其中一个属性，会在函数执行的过程中用到

## this 全面解析

### 调用位置

> 寻找调用位置就是寻找函数被调用的位置

```javascript
function baz () {
    // 当前的调用栈时 baz
    // 因此，当前的调用位置时全局作用域
    console.log("baz")
    bar() // bar 的调用位置
}

function bar () {
    // 当前调用栈在 baz => bar
	// 因此，当前的调用位置在baz中
    console.log("bar")
    foo()
}

function foo () {
    // 当前的调用栈时baz => bar => foo
    // 因此，当前的调用位置在bar 中
    console.log("foo")
}

baz() // baz 的调用位置

```

### 绑定规则

> 必须先找到调用位置，然后判断需要应用下面四条规则的哪一条

#### 默认绑定

```javascript
function foo () {
console.log(this.a)
}
var a = 2
// foo  的调用位置是在全局调用的 其 this 是属于全局的this
// foo 其实就是全局对象(window)的一个属性
foo() // 2
```

> 但是在严格模式下 **全局对象将无法使用默认绑定，this会绑定到undefined**
>
> 严格模式的意思是这个函数体书不是严格模式，而不是说这个韩式调用的位置是不是严格模式
>
> ```javascript
> function foo() {
>  "use strict";
>  console.log( this.a );
> }
> var a = 2;
> foo(); // TypeError: this is undefined
> ```
>
> 

#### 隐式绑定

> 当函数引用有上个下文对象是，隐式绑定规则会把函数调用中的this绑定到这个上下文对象

```javascript
function foo () {
	console.log(this.a)
}
var obj = {
	a: 2,
    // 无论是直接在 obj 中定义还是先定义再添加为引用属性，这个函数严格来说都不属于
	// obj 对象。
    // foo 是属于全局的
	foo: foo
}
// foo函数被调用的时候，obj对象拥有或者包含它
obj.foo() //2
```

```javascript
function foo () {
	console.log(this.a)
}
var obj = {
	a: 42,
	foo: foo
}

var obj1 = {
    a: 2,
    obj2: obj2
}
obj1.obj2.foo() //42
```

##### 隐式丢失

> 隐式绑定的函数会i都市绑定对象，也就是说他会应用默认绑定，从而把this绑定到全局对象或者undefined 上

```javascript
function foo () {
	console.log(this.a)
}
var obj = {
	a: 2,
	foo: foo
}
// bar 虽然是obj.foo的一个引用，但是实际上引用的是foo函数本身 丢失了obj的上下文对象
var bar = obj.foo
var a = "oops, global"
// 在全局调用是 this 走的默认绑定方式
bar() //oop, global
```

**传入回调函数 this 丢失**

> 参数传递其实就是一种隐式赋值， 因此 我们传入函数时也会被隐式赋值

```javascript
function foo () {
    console.log(this.a)
}
function deFoo(fn) {
    fn();
}

var obj = {
    a: 2,
    foo:foo
}
var a = 'oops, globa'
// 其实传入的是全局的foo
deFoo(obj.fn) // oops, globa
```

#### 显示绑定

> 使用call() 和 apply () 方法进行显示的绑定
>
> 他们的第一个参数是对象，他们会把这个对象绑定到this，接着在调用函数时指定这个this
>
> 因为可以直接直嘀咕this的绑定对象，因为，我们称之为显示绑定

```javascript
function foo () {
console.log(this.a)
}

var obj = {
	a:2
}
// 强制把foo的this绑定到Obj上
foo.call(obj) // 2
```

##### 硬绑定解决隐式丢失的问题

```javascript
function foo () {
console.log(this.a)
}
var obj = {
	a:2
}
var bar = function () {
	foo.call(obj)
}
bar() // 2
setTimeout(bar, 1000) // 2
bar.call(window)  // 硬绑定的bar 不可能再修改它的this


```

##### 使用封装好的硬绑定的方法

```javascript
function foo (something) {
console.log(this.a, something)
}
var obj = {
	a:2
}
var bar = foo.bind(obj)
let b = bar(3) // 2
console.log(b)  // 3
```

#### new 绑定

**构造函数**

所有函数都可以用new来调用，这种函数调用被称为构造函数调用

实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”

> 构造函数只是一些使用new操作符时被调用的函数
>
> 他们并不会属于某个类也不会实列化一个类

**使用new会执行以下操作**

+ 创建一个全新的对象
+ 这个对象会被原型连接
+ 这个新对象会绑定到函数调用的this
+ 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

```
function foo (a) {
	this.a = a
}
var bar = new foo(2)
console.log(bar.a)  //2
```

> 在使用new 来调用foo()时，我们会构造一个新对象并把它绑定到foo调用中的this上

#### 判断this 的绑定对象

+ 函数是否再new中调用

  > 如果是的话，this绑定的时新创建的对象
  >
  > ```javascript
  > let bar = new foo()
  > ```

+ 函数是否通过call、apply或者硬绑定调用

  > 如果是的话，那么this绑定的就是此指定的对象
  >
  > ```javascript
  > let bar = foo.call(obj2)
  > ```

+ 函数是否再某个上下文对象中调用

  > 如果是，this绑定的时那个上下文对象
  >
  > ```javascript
  > var bar = obj1.foo()
  > ```

+ 如果都不是的话， 那么再严格模式下，就绑定到undefined，否则绑定到全局对象

  > ```javascript
  > let bar = foo()
  > ```


##### 更安全的this

> 我们想忽略一个this，再这个this绑定到一个空对象上，这样，此this的使用都会被现在在这个空对象中，不会对全局对象产生任何影响

```javascript
function foo (a, b) {
	console.log("a:" + a, "b:" + b)
}
// 创建一个空对象
let $_emptyObj = Object.create(null)

// 把数组展开成参数
foo.apply($_emptyObj, [2, 3])
// 使用bind(...) 进行柯里化
let bar = foo.bind($_emptyObj, 2)
bar( 3 )
```

## 对象

+ 在对象中，属性名永远都是字符串，如果你使用string之外的其他值作为属性名，拿他首先会被转换为一个字符串，即使是数字也不列外

  ```javascript
  var myObject = { };
  myObject[true] = "foo";
  myObject[3] = "bar";
  myObject[myObject] = "baz";
  myObject["true"]; // "foo"
  myObject["3"]; // "bar"
  myObject["[object Object]"]; // "baz"
  ```

### 存在性

我们可以在不访问属性值的情况下判断对象中是否存在这个属性

```javascript
var myObject = {
 a:2
};
("a" in myObject); // true
("b" in myObject); // false
myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```

> in操作符会检查属性是否在对象及其[[Prototype]]原型链中
>
> hasOwnProperty(...)指挥检查属性是否在myObject对象中，不会检查原型链

> 4 in [2, 4, 6] 不会为true
>
> 因为[2, 4, 6] 的属性名是0、1、2 没有4

#### 枚举

> 可枚举就i相当于可以出现在对象属性的遍历中

```javascript
var myObject = { };
Object.defineProperty(
 myObject,
 "a",
 // 让 a 像普通属性一样可以枚举
 { enumerable: true, value: 2 }
);
Object.defineProperty(
 myObject,
 "b",
 // 让 b 不可枚举
 { enumerable: false, value: 3 }
);
// 设置为不可枚举也是可以赋值的
myObject.b; // 3
// 设置为不可枚举也还是可以被in操作符检测到的
("b" in myObject); // true
// 设置为不可枚举还是可以通过方法检测到
myObject.hasOwnProperty( "b" ); // true
// ..
for (var k in myObject) {
 console.log( k, myObject[k] );
}
// "a" 2
```

判断对象属性是否可枚举

```javascript
var myObject = { };
Object.defineProperty(
 myObject,
 "a",
 // 让 a 像普通属性一样可以枚举
 { enumerable: true, value: 2 }
);
Object.defineProperty(
 myObject,
 "b",
 // 让 b 不可枚举
 { enumerable: false, value: 3 }
);
// 判断此对象的属性是否可枚举 会检查给定的属性名是否直接存在于对象中（而不是在原型链
// 上）
myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false
// 通过keys获取对象属性列表 不会出现不可枚举的属性 会返回一个数组，包含所有可枚举属性
Object.keys( myObject ); // ["a"]
// 获取对象属性列表 会出现不可枚举的属性 会返回一个数组，包含所有属性，无论它们是否可枚举
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
// Object.keys(..) 和 Object.getOwnPropertyNames(..) 都只会查找对象直接包含的属性。
```

## 混合对象“类”

### 类理论

#### 构造函数

> 函数不是构造函数，当时当且仅当使用new时，函数调用会变成“构造函数调用”



```javascript
function Foo () {
    //...
}
var a = new Foo()
```

事实上,FOO和其他函数没有任何区别。函数本身并不是构造函数，然而， 当你在普通的函数调用前面加上New关键字之后，就会把这个函数调用变成一个“构造函数调用”

**实际上，new会劫持所有普通函数，并用构造对象的形式来调用它**

> 类的实例是由一个特殊的类方法构造的，这个方法名通常和类名相同，被称为构造函数，这个方法的任务就是初始化实列所需要的信息

类构造函数属于类，而且通常和类同名，此外，构造函数大多需要用new来调，这样语言引擎才知道你想要构造一个新的类实例

### 类的继承

## 原型

继承意味着复制操作，JavaScript并不会复制对象属性，JavaScript会在两个对象之间创建一个关联，这样一个对象就可以通过委托访问另一个对象的属性和函数

**委托这个术语，可以更加准确的描述JavaScript中对象的关联机制**



