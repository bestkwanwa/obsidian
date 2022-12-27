```js
const NOOP=()=>{}

function fn(cb=NOOP){
    // 不需要 cb&&cb()
    cb()
}
```
- 可读性强，知道该变量是函数
- 不需要判断是否undefined就可以执行
- 不需要每次创建一个空函数，都指向一个noop引用即可