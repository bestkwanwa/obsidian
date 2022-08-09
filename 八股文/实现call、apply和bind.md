## 思路
```js
var foo = {
    value: 1,
};
function bar(){
    console.log(this.value);
};
bar.call(foo);  // 1
```

### 效果
```js
var foo = {
    value: 1,
    bar(){
        console.log(this.value);
    },
};
foo.bar();  // 1
```
最终实现的效果如上，所以需要：
- 将函数设置为对象的方法
- 执行该方法
- 删除该方法

## 实现

### 改变this指向
```js
Function.prototype.myCall = function (ctx) {
    ctx.fn = this
    ctx.fn()    // 这里改变了this指向
    delete ctx.fn
}

var foo = {
    value: 'foo'
}

function bar() {
    console.log(this.value);
}

bar.myCall(foo)
```

### 传不定参
```js
Function.prototype.myCall = function (ctx) {
    args = []
    for (var i = 1; i < arguments.length; i++) {
        args.push(`arguments[${i}]`)
    }
    ctx.fn = this
    eval(`ctx.fn(${args})`) // 这里改变了this指向，并传入不定参。这里数组隐式转换成了字符串
    delete ctx.fn
}

var foo = {
    value: 'foo'
}

function bar(name, age) {
    console.log(name, age, this.value);
}

bar.myCall(foo, 'abc', 12) 
```

### 传入null与返回值
```js
Function.prototype.myCall = function (ctx) {
    args = []
    for (var i = 1; i < arguments.length; i++) {
        args.push(`arguments[${i}]`)
    }
    ctx = ctx || window
    ctx.fn = this
    var result = eval(`ctx.fn(${args})`) // 这里改变了this指向，并传入不定参。这里数组隐式转换成了字符串
    delete ctx.fn
    return result
}

var value = 'global'

var foo = {
    value: 'foo'
}

function bar(name, age) {
    console.log(name, age, this.value);
    return 'end!'
}

console.log(bar.myCall(null, 'abc', 12))
```

## References
- [JavaScript深入之call和apply的模拟实现
](https://github.com/mqyqingfeng/Blog/issues/11)
