## bind
```js
function fn(a,b,c){
	console.log(a+b+c)
}
a={}
b=fn.bind(a,1,2,3)
b(4,5,6)	// 6
b=fn.bind(a)
b(4,5,6)	// 15
```