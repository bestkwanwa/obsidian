# API路由
`/pages/api`映射为`/api/*`，会被当做服务端的接口。例：
```js
export default function handler(req, res) {
  res.status(200).json({ name: 'John Doe' })
}
```
处理不同method的请求：
```js
export default function handler(req, res) {
  if (req.method === 'POST') {
    // Process a POST request
  } else {
    // Handle any other HTTP method
  }
}
```

## 注意事项
- API路由不指定`CORS`头，默认为`same-origin only`，可以通过中间件改变这个默认行为。
- 不能和`next export`一起使用

## 动态API路由
例：
```js
// pages/api/post/[pid].js
export default function handler(req, res) {
  const { pid } = req.query
  res.end(`Post: ${pid}`)
}
// 当请求 /api/post/abc 时，服务端响应文本 Post: abc
```

### 索引路由和动态API路由
常见[[RESTful API | RESTFul]]模式的接口会这样设计：
- `GET api/posts` - 获取帖子列表，可能是被分页了
- `GET api/posts/12345` - 获取指定文章
那在`API路由`里可以这样设计：
- `/api/posts.js`
- `/api/posts/[postId].js`
或者：
- `/api/posts/index.js`
- `/api/posts/[postId].js`

### `Catch all API routers`
可以通过在方括号内添加三个点(...)来扩展 `API Routes` 以捕获所有路径。例如：
- `pages/api/post/[...slug].js`匹配`/api/post/a`、`/api/post/a/b`, `/api/post/a/b/c`等等。
- `slug`可以是其他名称。
- 匹配上的参数会存入一个数组，如：
	```js
	// pages/api/post/[...slug].js
	export default function handler(req, res) {
	  const { slug } = req.query
	  res.end(`Post: ${slug.join(', ')}`)
	}
	```
	访问`/api/post/a/b/c`，则`slug`为`{ "slug": ["a", "b", "c"] }`。

### `Optional catch all API routes`
例：
`pages/api/post/[[...slug]].js`将匹配`/api/post`, `/api/post/a`, `/api/post/a/b`。比`Catch all`要多匹配一个没有参数的情况。
```js
{ } // GET `/api/post` (empty object)
{ "slug": ["a"] } // `GET /api/post/a` (single-element array)
{ "slug": ["a", "b"] } // `GET /api/post/a/b` (multi-element array)
```

## 优先级
高到低：
- `pages/api/post/create.js`
- `pages/api/post/[pid].js`
- `pages/api/post/[...slug].js`
请求`/api/post/create`，由`pages/api/post/create.js`响应。
请求`/api/post/abc`，由`pages/api/post/[pid].js`响应。
请求`/api/post/1/2`，由`pages/api/post/[...slug].js`响应。

## 中间件
examples:
- examples/api-routes-with-middleware
- examples/api-routes-cors

### 自定义配置
每个`API路由`都可以导出一个`config`对象，去改变这个`API路由`的默认行为，例：
```js
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '1mb',
    },
  },
}
// 其他配置
```

### 中间件
见examples/api-routes-cors

### 使用`TypeScript`扩展`req`/`res`对象
不建议直接在`req`/`res`对象上加属性。如果一定要加，使用`TS`进行类型声明。详见[官网文档](https://nextjs.org/docs/api-routes/api-middlewares#extending-the-reqres-objects-with-typescript)。

## 响应
- `res.status(code)` - 设置状态码。
- `res.json(body)` - 返回`JSON`格式的响应（序列化）。
- `res.send(body)` - 返回`HTTP`响应，响应体可以是`string`、`object`或者`Buffer`。
- `res.redirect([status,] path)` - 重定向。不指定状态码则默认为`307`。
具体用法详见[官网文档](https://nextjs.org/docs/api-routes/response-helpers)。