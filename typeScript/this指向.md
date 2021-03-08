this的指向问题

在ES5中，this的指向是永远指向最后一次调用他的那个对象。

想要改变this的指向，有如下解决方案：

1、在函数内部使用_this=this

​		编码过程中，将函数内部的this赋值给_this,使得函数使用的一直是自身的this，这是最简单的使用方法。

2、使用apply 、call、bind

使用他们的主要作用就是给变this的指向问题。

call和apply的共同点是：能够改变函数执行的上下文，将一个对象的方法交给另一个函数执行，并且是立即执行的。

调用call和apply的对象，必须是一个函数Function。

call和apply的区别是：主要体现在参数的写法不同

如：

call的写法：Function.call(obj,[param1]);

注意：调用call的对象，必须是个函数function

​			call的第一个参数，是一个对象，function的调用者，如果不传，则默认是全局对象window。

​			第二个参数开始，可以接收人一个参数，每个参数都会映射到相应位置的function的参数上。如果将所有的参数作为数组传入，他们会作为一个整体映射到function对应的第一个参数上，之后的参数为空。

```
function func (a,b,c) {}

func.call(obj, 1,2,3)
// func 接收到的参数实际上是 1,2,3

func.call(obj, [1,2,3])
// func 接收到的参数实际上是 [1,2,3],undefined,undefined
```

apply的写法：Function.apply(obj,[argArray])

注意：apply的调用者必须是函数function，并且只接收两个参数，第一个参数规则与call一致。

​			第二个参数，必须是数组或者类数组。他们会被转换成类数组，传入function中，并且会被映射到function对应的参数上。

​			第二个参数与call之间的一个很大的不同。

```
func.apply(obj, [1,2,3])
// func 接收到的参数实际上是 1,2,3

func.apply(obj, {
    0: 1,
    1: 2,
    2: 3,
    length: 3
})
// func 接收到的参数实际上是 1,2,3
```

bind的写法：function.bind(thisArg[, arg1[, arg2[, ...]]])

```
thisArg
```

调用绑定函数时作为 `this` 参数传递给目标函数的值。 如果使用[`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)运算符构造绑定函数，则忽略该值。当使用 `bind` 在 `setTimeout` 中创建一个函数（作为回调提供）时，作为 `thisArg` 传递的任何原始值都将转换为 `object`。如果 `bind` 函数的参数列表为空，或者`thisArg`是`null`或`undefined`，执行作用域的 `this` 将被视为新函数的 `thisArg`。

```
arg1, arg2, ...
```

当目标函数被调用时，被预置入绑定函数的参数列表中的参数。

bind返回一个函数function，不会立即执行，调用后执行。

3、new实例化一个对象

4、使用ES的箭头函数

箭头函数里面根本没有自己的`this`，而是引用外层的，所以剪头函数的this默认是指向定义函数的对象。

5：使用es6的::指向操作符





