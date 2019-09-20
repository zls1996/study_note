## ECMAScirpt Harmony（ECMAScript 5）

#### 常量

​			const使用方式与var类似，但**const声明的变量在初始化赋值后，就不能再重新赋值了**。

```js
const MAX_SIZE = 25;

const FLAG = true;
var FLAG = false;//错误

const FLAG = true;
FLAG = false; //错误
alert(FLAG); //true
```

#### 块级作用域和其它作用域

​	JS没有块级作用域。在语句块中定义的变量与包含在函数中定义的变量共享相同的作用域。Harmony新增了定义块级作用域的语法：使用let关键字。

​	与const和var类似，可以使用let在任何地方定义变量并为变量赋值。区别在于，使用let定义的变量在定义它的代码块之外没有定义。

```js
//使用var定义变量
for(var i = 0 ; i < 10; i++) {
    doSomething();
}
alert(i) ; //10

//使用let定义变量
for(let i = 0 ; i < 10; i++) {
    doSomething();
}

alert(i) ; //undefined
```

​	还有一种使用let的方式，即创建let语句，在其中定义只能在后续代码块中使用的变量，像这样的例子：

```js
var num = 5;

//在let语句块中，num乘以multiplier。出了let语句块之后，num的变量的值依然是5
//这是因为let语句创建了自己的作用域，这个作用域里面的变量与外面的变量无关。
let(num = 10, multiplier = 2) {
    alert(num * multiplier); //20
}

alert(num); //5

//同理
var result = let(num = 10, multiplier = 2) num * multiplier;
alert(result) ; // 20
```

#### 函数

​	**Harmony中不再有arguments对象**，因此也就无法通过它来读取到未声明的参数。不过，使用**剩余参数(rest arguments)**语法，也能表示期待给函数传入可变数量的参数。剩余参数的语法形式是三个点后跟一个标识符。**使用这种语法可以定义可能会传进来的更多参数，然后把它们收集到一个数组中**。

```js
function sum(num1, num2, ... nums) {
    let result = num1 + num2;
    for(let i in nums) {
        result += nums[i];
    }
    return result;
}

let result = sum(1, 2, 3, 4, 5, 6);
```

​	与剩余参数紧密相关的另一种参数语法是**分布参数(spread arguments**)。通过分布参数,可以向函数中传入一个数组，然后数组中的元素会映射到函数的每个参数上。分布参数的语法形式与剩余参数的语法相同，就是在值的前面加上三个点。**唯一的区别是分布参数在调用函数的时候调用**。

```js
const result = sum(...[1, 2, 3, 4, 5, 6]);
```

​	也可以用apply来表示：

```js
const result = sum.apply(this, [1, 2, 3, 4, 5, 6]);
```

#### 默认参数值

​	ES中所有参数都是可选的，因为实现不会检查传入的参数数量。不过，**除了手工检查传入了哪些参数以外，还可以为参数指定默认值**。

```js
function sum(num1, num2 = 5) {
    return num1 + num2;
}

const result1 = sum(3); //8
const result2 = sum(2, 3); //5
```

#### 生成器

​	所谓**生成器，其实就是一个对象，它每次能生成一系列值中的一个**。对于Harmony而言，要创建生成器，可以让函数通过**yield**操作符返回某个特殊的值。对于使用yield操作符返回的函数，调用它时就会创建并返回一个新的Generator实例。然后，在这个实例上调用**next()**方法就能取得生成器的第一个值。此时，执行的是原来的函数，但执行流到yield语句就会停止，只返回特定的值。从这个角度看，yield和return很相似。如果再次调用next()方法，原来函数中位于yield语句后的代码会继续执行，直到再次遇见yield时停止执行，此时再返回一个新值。

```js
function getNumbers() {
    for(let i = 0 ; i < 100; i++) {
        yield i * 2;
    }
}

let generator = getNumbers();

try {
    while(true) {
        console.log(generator.next()); // 0, 2 ,4 , 6 ..... 200
    }
}catch(e) {
    console.log("执行生成器过程失败", e);
}finally {
    generator.close();
}

```

#### 数组及其他

##### 	迭代器

​	迭代器也是一个对象，它能迭代一系列值并返回其中一个值。

```js
const person = {
    name : 'Leeson',
    age: 23,
}

const iterator = new Iterator(person);

try {
    while(true) {
        let value = iterator.next();
        console.log(value.join(':') + '<br/>')
    }
} catch(e) {
    console.error('迭代出现错误:', e);
}
```

##### 	数组领悟

​	数组领悟(Array comprehension), 指的是用一组符合某个条件的值来初始化数组。Harmony定义的这项功能借鉴了python的一个语言结构。其数组领悟的基本形式如下：

```js
array = [value for each (variable in variables) consition]

const numbers = [0,1,2,3,4,5,6,7,8,9];

//复制numbers数组
const duplicate = [i for each (i in numbers)];

//提取偶数
const evens = [i for each (i in numbers) if (i % 2 == 0) ];

//数据翻倍
const doubled = [ i * 2 for each (i in numbers)]

//每个奇数乘以三
const tripleOdds = [i*3 for each (i in numbers) if (i % 2 ) !== 0] 
```

##### 	解构

​	从一组值中挑出一个或多个值，然后把它们分别赋给独立的变量，这也是很常见的需求。

```js
const person = {
	name : 'Leeson',
    age : 23,
}
const { name, age } = person;
console.log('name: ' + name + ', age : ' + age);//name: Leeson, age : 23
```

#### 映射与集合

​	map类型，也称为简单映射，只有一个目的：保存一组键值对。

```js
const map = new Map();
map.set('name', "Leeson");
map.set('age', 23)
const name = map.get('name')
map.delete('name')
```

