### 语法

#### Undefined类型

一个已经被定义但是未被赋值的变量，js会为其默认设置为undefined值。

```js
// 在给已定义变量message和未定义变量info显示类型时，都会显示undefined类型。
var message;
// var info;   //info未定义
alert(typeof message); //undefined
alert(type of info); //undefined
```

#### Null类型

Null类型是第二个只有一个值的数据类型，这个特殊的值是null。null值表示一个空对象指针，因此，null的数据类型是Object。实际上，undefined是衍生于null的，ECMA规定这两个值的相等值测试要为真，因此null值与undefined相等， ==返回true；但是===是比较数值和类型的，undefined和null类型不一致，所以===返回false

```js
var obj = null;
console.log(typeof obj); // Object
console.log(null == undefined); // true
console.log(null === undefined); //false

typeof undefined; // "undefined"
typeof null; // "Object"
```

#### Number

Number包括整数和浮点数，具体还表现为八进制、十六进制、十进制。

NaN,表示非数值，它是一个特殊的数值，这个数值用于表示一个本来要返回数值的操作数未返回数值的情况，这样就不会抛出错误了。

NaN本身有两个非同寻常的特点。首先，**任何涉及NaN的操作都会返回NaN，这个特点会在多步计算中可能会导致问题。其次，NaN与任何值都不相等，包括NaN本身**。

```js
var binary = 0110 // 八进制，对应十进制数值为64+8=72
var hex = 0x11; //十六进制，对应十进制数值为16+1=17

typeof NaN ; // "Number"
console.log(NaN == NaN); //false

console.log(isNaN(NaN)); //true
console.log(isNaN(10)); //false,10可以被转换成数字
console.log(isNaN("10")); //false,"10"可以被转换成数字
console.log(isNaN("hi")); //true，"hi"不能被转换成数字
console.log(isNaN(true)); //false, true可以被转换成数字1
console.log(NaN == NaN); //false
```



#### 垃圾回收

​	JS具有在自动垃圾回收机制，也就是说，执行环境会负责管理代码执行过程中使用的内存。

##### 垃圾回收机制

- 标记清除  ： 垃圾收集器会在运行时刻给存储在内存中的所有变量都添加上标记。然后会去掉环境中的变量以及被环境中的变量引用的变量的标记。
- 引用计数：跟踪每个值被引用的次数。当某个值的引用次数变为0时，说明不能再访问这个值，烟于是可以将其占用的内存回收。这样，当垃圾收集器下次再运行时，它就会释放那些引用次数为0的值所占用的内存。



##### 垃圾回收性能

垃圾收集器是周期性运行的，而且如果为变量分配的内存数量比较大，那么回收工作量也是相当大的。



##### 管理内存

​	由于分配给浏览器的可用内存数量通常比分配个桌面应用程序少，因此必须确保占用最少的内存来让页面获得更好的性能。优化内存占用得最佳方式，就是为执行中的代码只保存必要的数据。**一旦数据不再使用，最好通过将其值设置为null来释放引用**。这种方法叫做解除引用，这一做法适用于大多数全局变量和全局变量的属性。**局部变量会在它们离开执行环境时自动解除引用**。

​	不过，解除一个值的引用并不意味着自动回收该值所占用的内存，解除引用的真正作用是让值脱离执行环境，以便垃圾回收器下次运行时自动将其回收。

### 引用类型

#### 	Function类型

​	**函数就是一个对象**。每个函数都是一个Function类型的实例，而且**与其他引用类型一样具有属性和方法**。由于函数是对象，因此**函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定**。函数通常是使用函数声明语法一致。

```js
// 使用函数声明式语法定义函数
function sum (s1, s2) {
    return s1 + s2;
}
// 使用函数表达式定义函数
var sum = function(s1, s2) {
    return s1 + s2;
}

// 上述两种定义函数的方式效果是一致的。
```

需要注意的是：**ECMA中是没有函数重载的**。

```js
// 创建一个reload函数
function reload(num) {
    return num + 1;
}
//重新定义reload函数，会覆盖原有reload函数
function reload(num) {
    return num + 2;
}

var result = reload(3); // 5
```

ECMA中的函数名本身就是变量，所以**函数也是可以作为值来使用的，也可以作为参数来传值**。

```js
// 定义函数，第一个参数是函数，第二个参数值
function invoke(func, params) {
	return func(params);
}
function multiply(params) {
    return params + 1;
}
//调用invoke函数，返回结果为multiply函数执行的结果
var result = invoke(multiply, 1); // 2
```

#### 函数属性和方法

ECMA中的函数式对象，因此函数也有属性和方法。每个函数都包含两个属性：length和prototype。其中，length属性表示函数希望接受的命名参数的个数。

```js
function say(name) {
    console.log(name);
}

alert(say.length) // 1, 接收命名参数个数为1
```

​	对于ECMA中的引用类型而言，prototype是保存它们所有实例方法的真正所在。比如，toString()和valueOf()等方法实际上保存在prototype下，只不过是通过各自对象的实例访问而已。创建自定义引用雷星以及实现继承时，prototype属性是极为重要的。

​	每个函数都包含两个非继承而来的方法，apply()和call。这两个方法的用途都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值。apply接收两个参数:一个是在运行函数的作用域，另一个是参数数组。

```js
function sum(n1, n2) {
    return n1 + n2;
}

//applySum在执行sum函数时传入了this作为this值和argument对象。这里的argument就是参数形成的数组
function applySum(n1, n2) {
    return sum.apply(this,argument);
}

console.log(applySum(1, 2)); // 结果为3

// 使用call
function sum1(n1, n2) {
    return n1 +n2;
}

function callSum(n1, n2) {
    return sum1.call(this, n1, n2);
}

console.log(callSum(1, 2)); // 结果为3
```

​	在使用call方法的情况下，callSum必须明确传入每一个参数。结果与apply没有什么不同。

实际上，传递参数并非**apply和call**的真正用武之地；它们真正强大地方在于**能够扩充函数赖以运行的作用域**，如下段代码所示。

```js
window.color = "red";
var o = {color : "black"};

function printColor() {
    console.log(this.color);
}

printColor(); // red
printColor.call(this); //red
printColor.call(window); // red
printColor.call(o); // black
```

​	使用call和apply来扩充作用域的好处就是，对象不需要与方法有任何耦合关系。

​	ECMA还定义了一个方法：bind。 **bind主要用于绑定一个函数的作用域,并返回针对指定作用域的函数。**

```js
window.color = "blue";
var o = {color : "red"};

function printColor() {
    console.log(this.color);
}

var printO = printColor.bind(o);
printO(); // red；

var printThis = printColor.bind(this);
printThis(); // blue;
```

