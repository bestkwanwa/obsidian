### 错误捕获
```js
Promise.resolve()
  .then(() => {
    return new Error('error!!!')
  })
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })
```
.then 或者 .catch 中 return 一个 error 对象并不会抛出错误，所以不会被后续的 .catch 捕获，需要改成其中一种：
```js
return Promise.reject(new Error('error!!!'))
```
```js
throw new Error('error!!!')
```

### 值穿透
```js
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)        // 结果为 1
```
.then 或者 .catch 的参数期望是函数，传入非函数则会发生值穿透。