## 思路
- 返回一个函数
- 可以传不定参

## 实现

### 返回一个函数
```js
Function.prototype.myBind = function (ctx) {
    var self = this;
    return function () {
        return self.apply(ctx);
    }

}
```

### 传参
- 在调用bind的时候可以传参
- 调用bind返回的函数时也可以传参
```js
Function.prototype.myBind = function (ctx) {
    var self = this;
    // 调用bind的时候的参数
    var bindArgs = Array.prototype.slice.call(arguments, 1);
    return function () {
        // 调用返回的函数时的参数
        var args = Array.prototype.slice.call(arguments);
        return self.apply(ctx, bindArgs.concat(args));
    }
}
```

### 返回的函数作为构造函数
**调用bind后返回的函数作为构造函数的时候，调用bind时指定的this值会失效。**
```js
Function.prototype.myBind = function (ctx) {
    var self = this;
    var bindArgs = Array.prototype.slice.call(arguments, 1);
    var boundFn = function () {
        var args = Array.prototype.slice.call(arguments);
        // 当boundFn当成构造函数使用时，this指向boundFn生成的实例，所以instanceof完为true，所以apply传入新生成的实例即可
        return self.apply(this instanceof boundFn ? this : ctx, bindArgs.concat(args));
    }
    // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
    boundFn.prototype = this.prototype;
    return boundFn;
}

var value = 'global'
var foo = {
    value: 'foo'
}
function bar(name, age) {
    console.log(name, age, this.value);
    return 'end!'
}
bar.prototype.func = function () {
    console.log('func');
}
var baz = bar.myBind(foo, 'abc')

console.log(baz(12))    // 当成普通函数调用，相当于 bar.apply(foo,args)

var nBaz = new baz(12) // 当成构造函数使用，相当于 new bar()

nBaz.func()

```

## References
- [JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)