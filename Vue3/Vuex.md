# Vuex
- store里的状态是响应式的
- 不能直接改store中的状态，而是通过commit mutation

## Api
- commit 触发状态变更
- dispatch 触发action处理函数

## 函数参数
- action函数参数
- getter函数参数
- mutation处理函数参数

## 获取
- 最简单的读取状态方法是在*计算属性*中返回
	```js
	computed:{
		count(){
			return store.state.count
		}
	}
	```
- 注入
	组件内通过`this.$store`访问store实例
- 多个状态
	```js
	import {mapState} from 'vues'
	export default{
		computed:mapState({
			count:state=>state.count,
			countAlias:'count',
			countPlusLocalState(stete){
				return state.count+this.localCount
			}
		})
	}
	
	// 当同名时
	computed:mapState([
		'count'
	])
	
	// 可以与组件内计算属性合并
	computer:{
		localComputed(){/*...*/},
		...mapState({
			// ...
		})
	}
	```
	
## 派生状态
Vuex允许我们在store中定义”getter“（可以认为是store的计算属性！）
```js
const store=createStore({
	state:{
		todos:[
			{id:1,txt:'...',done:true},
			{id:2,txt:'...',done:false},
		]
	},
	getter:{
		doneTodos(stete)=>{
			// 筛选done为true的
			return state.todos.filter(todo=>todo.done)
		}
	}
})

// 通过属性访问
store.getters.doneTodos

// 接收第二个参数
getters:{
	doneTodosCount(state,getters){
		return getters.doneTodos.length
	}
}

// 通过方法访问
getters:{
	getTodoById:(state)=>(id)=>{
		return state.todos.find(todo=>todo.id===id)
	}
}
// ...
// 可以传参
store.getters.getTodoById(2)
````
在组件中使用
```js
computer:{
	doneTodosCount(){
		return this.$store.getters.doneTodosCount
	}
}
```
mapGetters辅助函数
```js
computed:{
	// 合并计算属性
	...mapGetters([
		'doneTodosCount',
		'anotherGetter',
	])
}
// 别名
...mapGetters({
	// this.doneCount映射为this.$store.getters.doneTodosCount
	doneCount:'doneTodosCount'
})
```

## 提交Mutation
每个mutation都有一个字符串的事件类型（type）和一个回调函数（handler）。**Mutation必须是同步函数。**
```js
const store=createStore({
	state:{
		count:1
	},
	mutations:{
		// 接收state作为第一个参数
		increment(state){
			state.count++
		}
	}
})
// 不能直接调用mutation处理函数，而是调用commit
store.commit('increment')
```
Payload（提交载荷）
```js
mutations:{
	increment(state,n){
		state.count+=n
	}
}
// ...
store.commit('increment',10)

// 载荷为对象
mutations:{
	increment(state,payload){
		state.count+=payload.amount
	}
}
// ...
store.commit('increment',{
	amount:10
})
```
提交对象
```js
store.commit({
	type:'increment',
	amount:10
})
// 提交的整个对象会作为payload
```
在组件中使用：`this.$store.commit('xxx')`或者`mapMutations`
```js
// mapMutations将组件中methods映射为store.commit调用
methods:{
	...mapMutations([
		// 将 this.increment() 映射为 this.$store.commit('increment') 
		'increment',
		// 支持payload。将 this.incrementBy() 映射为 this.$store.commit('incrementBy',amount)
		'incrementBy'
	]),
	...mpaMutations({
		// 别名
		add:'increment'
	})
}
```
## Action
Action提交mutation，不直接变更状态，允许异步操作。
```js
const store=createStore({
	state:{
		count:0
	},
	mutations:{
		increment(state){
			state.count++
		}
	},
	actions:{
		// 接收一个与store实例具有相同方法和属性的context对象（注意：context对象不是store实例本身）
		increment(context){
			context.commit('increment')
		}
	}
})

// 解构context
actions:{
	increment({commit}){
		commit('increment')
	}
}
```
通过`store.dispatch`触发
```js
store.dispatch('increment')
```
异步操作
```js
actions:{
	incrementAsync({commit}){
		setTimeout(()=>{
			commit('increment')
		},100)
	}
}
// 支持payload
store.dispatch('incrementAsync',{amoount:10})
store.dispatch({
	type:'incrementAsync',
	amount:10
})
```
在组件中使用
```js
// 1.
this.$store.dispatch('xxx')

// 2.使用mapActions将组件内methods映射为store.dispatch调用
// 与mapMutations一致
methods:{
	...mapActions([
		// 将 this.increment() 映射为 this.$store.dispatch('increment') 
		'increment',
		// 支持payload。将 this.incrementBy() 映射为 this.$store.dispatch('incrementBy',amount)
		'incrementBy'
	]),
	...mapActions({
		// 别名
		add:'increment'
	})
}
```
组合action
```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }，
  // 再另一个action也可以
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
// ...
store.dispatch('actionA').then(()=>{
	// ...
})
// dispatch可以处理被触发action处理函数返回的Promise,并且dispatch仍然返回Promise
```
使用async/await
```js
// 假设 getData() 和 getOtherData() 返回的是 Promise
actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```
## Module
将store分割成多个module，每个module有自己的state、mutation、action、getter，甚至有自己的子模块。
```js
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```
模块内mutation和getter第一个接收的参数是**局部的状态对象state**

todo:模块没读完

## 项目结构
- 应用层级的状态应该集中到单个 store 对象中。即集中管理。
- 提交 **mutation** 是更改状态的唯一方法，并且这个过程是同步的。
- 异步逻辑都应该封装到 **action** 里面。
```
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

## Composition Api
访问State和Getter
```js
import { computed } from 'vue'
import { useStore } from 'vuex'

// 在setup中使用useStore
export default{
	setup(){
		const store=useStore()
		// 为了访问state和getter，需要创建computed引用以保留响应式
		return {
			// 访问state
			count:computed(()=>store.state.count),
			// 访问getter
			double:computer(()=>store.getters.double)
		}
	}
}
```
访问Mutation和Action
```js
// 在setup中调用commit和dispatch函数
export default {
  setup () {
    const store = useStore()

    return {
      // 使用 mutation
      increment: () => store.commit('increment'),

      // 使用 action
      asyncIncrement: () => store.dispatch('asyncIncrement')
    }
  }
}
```

## 插件
不懂

## 表单处理
- 受控
	```html
	<input :value="message" @input="updateMessage">
	```
	```js
	computed:{
		...mapState({
			message:stage=>state.obj.message
		})
	},
	methods:{
		updateMessage(e){
			this.$store.commit('updaetMessage',e.target.value)
		}
	}
	// ...
	// mutation函数
	mutations:{
		updateMessage(state,message){
			state.obj.message=message
		}
	}
	```
- 双绑
	```html
	input v-model="message">
	```
	```js
	computed:{
		message:{
			get(){
				return this.$store.state.obj.message
			},
			set(value){
				this.$store.commit('updateMessage',value)
			}
		}
	},