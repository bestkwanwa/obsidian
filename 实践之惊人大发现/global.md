# global
在代码中看到`global.$$`，这样会声明一个全局的`$$`变量，但JS里的全局对象在浏览器环境为`window`，那这个`global`又是怎样变成在浏览器环境里可以使用的全局变量呢？
首先，我在webpack打包后生成的dist目录里搜索`global.$$`,找到该js文件后，在js里搜索`global`，然后发现只要是使用到了`global`这个变量的，其实都是在一个在一个函数中，而这个函数的第二个入参，正是`global`。
```js
// 简化一下代码
(function(module,global){
	...
}.call(this,__webpack_require__('3UD+')(module),__webpack_require__('yLpj')))
```
我们顺藤摸瓜，看看到底传入了啥（前面的注释很关键）。
![global](../assets/global.png)
原来这里导入了一个模块！而这个模块判断环境并导出了全局对象。
```js
// ./../../node_modules/webpack/buildin/global.js
var g;

// This works in non-strict mode
g = (function() {
	return this;
})();

try {
	// This works if eval is allowed (see CSP)
	g = g || new Function("return this")();
} catch (e) {
	// This works if the window reference is available
	if (typeof window === "object") g = window;
}

// g can still be undefined, but nothing to do about it...
// We return undefined, instead of nothing here, so it's
// easier to handle this case. if(!global) { ...}

module.exports = g;
```
## 全局对象
- 浏览器中为`window`
- `Worker`为`WorkerGlobalScope`
- `Node`环境中为`global`