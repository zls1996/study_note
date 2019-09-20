## JS高级技巧

​		函数是JS中最有趣的部分。她们本质上是非常简单和过程化的，但也可以是非常复杂和动态的。一些额外的功能可以通过使用闭包来实现，此外由于所有的函数都是对象，所以使用函数指针非常简单。

#### 函数绑定

​	函数绑定需要创建一个函数，可以**在特定的this环境中以指定参数调用另一个函数**。这个技巧常常和回调函数与事件处理程序一起使用，**以便在将函数作为变量传递的同时保留代码执行环境**。

```js
var handler = {
    message: "Event handled",
    handleClick : function(event) {
        console.log(this.message);
    }
};

var btn = document.querySelector("#my-btn");
EventUtil.addHandler(btn, "click", handler.handleClick);
```

​	在这个例子中，创建了一个叫handler的对象。handler.handleClick方法被分配为一个DOM按钮的事件处理程序，当按下改按钮时，就调用该函数，显示一个警告框。虽然貌似警告框应该显示Event Handled，然而实际显示的是undefined。这个问题在于没有保存handle.handleClick的环境，所以this对象最后指向的DOM按钮不是handler，而是window。可以使用闭包来解决这个问题。

​	

```js
var handler = {
    message: "Event Handled",
    handleClick : function(event) {
        console.log(this.message);
    }
}
var btn = document.getElementById('my-btn');
//使用闭包来调用函数
EventUtil.addHandler(btn, 'click', function(event) {
    handler.handler(event);
})

```

​	但是问题在于，**创建多个闭包会令代码变得难以理解和调试**。因此，很多JS库实现了一个可以将函数绑定到指定环境的函数。这个函数叫bind()。

​	一个简单的bind()函数**接收一个函数和一个环境**，并**返回一个在给定环境调用给定参数的函数**，并且将所有参数原封不动传递过去。语法如下：

```js
function bind(fn, context) {
    return function () {
        return fn.apply(context, arguments);
    }
}
```

​	该函数原理是：在bind中创建了一个闭包，闭包使用apply()调用传入的函数，并给apply()传递context对象和参数。在这里使用的arguments对象是内部函数的，而不是bind的。当调用返回的函数时，它会在给定环境中执行被传入的函数并给出所有参数。bind()函数按如下方式使用。

```js
var handler = {
    message: 'Event Handled',
    handleClick : function (event) {
        console.log(this.event);
    }
}

var btn  = document.getElementById('my-btn');
EventUtil.addHandler(btn , 'click', bind(handler.handleClick, handler));
```

​	只要是将某个函数指针以值的方式进行传递，同时该函数必须在特定函数中执行，被绑定函数的效果就凸显出来了。它们主要用于事件处理程序以及setTimeout()和setTimeInterval()。然而，**被绑定函数与普通函数相比有更多的开销，它们需要更多内存，同时也因为多重函数调用晒微慢一点，所以最好只在必要时使用**。



#### 函数柯里化

​	与函数绑定紧密相关的是函数柯里化(function currying),它用于创建已经设置好的一个或多个参数的函数。函数柯里化的基本方法与函数绑定是一样的:使用一个闭包返回一个函数。两者的最大区别在于，当函数被调用时，返回的函数还需要设置一些传入的参数。

```js
function add (num1, num2) {
    return num1 + num2;
}

function curriedAdd(num2) {
    return add(5, num2);
}

alert(add(2,3)); // 5
alert(curriedAdd(3)); //8

```

​	柯里化函数通常由以下步骤来动态创建：**调用另一个函数并为它传入要柯里化的函数和必要参数**。下面是创建柯里化函数的通用方式。

```js
function curry(func) {
    var args = Array.prototype.slice.call(arguments, 1);
    return function () {
        var innerArgs = Array.prototype.slice.call(arguments);
        var finalArgs = args.concat(innerArgs);
        return func.apply(null, finalArgs);
    }
}
```

​	curry函数的主要工作是将被返回函数的参数进行排序。curry的第一个参数是要将柯里化的函数，其它参数是要传入的值。为了获取第一个参数之后的所有参数，在arguments对象上调用了slice方法，并传入参数1表示被返回的数组包含第二个参数开始的所有参数。然后args数组包含了来自外部函数的参数。在内部函数中，创建了innerArgs数组用来存放所有传入的参数(又一次用到了slice)。有了存放来自外部函数和内部函数的参数数组后，就可以使用concat()方法将它们组合为finalArgs，然后使用apply()方法将结果传递给该函数。注意这个函数并没有考虑到执行环境，所以调用apply()时第一个参数是null。curry函数可以按照以下方式应用。

```js
function add (num1, num2) {
    return num1 + num2;
}

var curriedAdd = curry(add, 5);
alert(curriedAdd(3)); //8

```

​	在这个例子中，创建了第一个参数绑定为5的add()函数的柯里化版本。当调用curriedAdd()并传入3时，3会成为add()的第二个参数，同时第一个参数依然是5，最后结果便是8,。也可以按照下面例子来给出所有的参数参数。

```js
function add (num1, num2) {
	return num1 + num2;
}

var curriedAdd = curry(add, 5 ,12);
console.log(curriedAdd()); // 17
```

​		在这里，柯里化的add()函数两个参数都提供了，所以以后就无需再传递它们了。

​		函数柯里化还常常作为函数绑定的一部分包含在其中，构造出更加复杂的bind函数。例如：

```js
function bind(func , context) {
    var args = Array.prototype.slice.call(arguments);
    return function () {
        var innerArgs = Array.prototype.slice.call(arguments);
        var finalArgs = args.concat(innerArgs);
        return func.apply(context, finalArgs);
    }
}
```

​	JS中的柯里化函数和绑定函数提供了强大的动态函数创建功能。使用bind()还是curry()要根据是否需要object对象响应来决定，它们都能用于创建复杂的算法和功能，当然两者都不应该滥用，因为每个函数都会带来额外的开销。

