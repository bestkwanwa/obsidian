## 执行上下文
- 变量环境
- 词法环境
- outer
- this

## this
- 当函数作为对象的方法调用时，函数中的this就是该对象。
- 函数被正常调用时，严格模式下，this指向undefined，非严格模式下，this指向window。
- 嵌套函数中的this不会继承外部函数的this。
- 箭头函数不会创建完整的执行上下文
- `setTimeout`的回调函数的this会指向全局环境/undefined，

## 更为原理的解释
[从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7)

### ECMAScript类型
- 语言类型：可操作的，即基本类型和复杂类型。
- 规范类型：不可操作的，只在规范中存在的抽象类型，用来描述语言底层行为逻辑。包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。

#### Reference
- base value: 属性所在的对象或者就是 EnvironmentRecord，它的值只可能是 undefined, an Object, a Boolean, a String, a Number, or an environment record 其中的一种。
- referenced name
- strict reference

##### 方法
- GetBase: 返回 base value。
- IsPropertyReference: 如果 base value 是一个对象，就返回true。
- GetValue: 从 Reference 类型获取对应值。返回的将是具体的值，而不再是一个 Reference。

### 如何确定this值
函数调用时，如何确定this值：
- 计算 MemberExpression 的结果赋值给 ref
- 判断 ref 是不是一个 Reference 类型
    - 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)
    - 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)
    - 如果 ref 不是 Reference，那么 this 的值为 undefined

#### MemberExpression
- PrimaryExpression // 原始表达式
- FunctionExpression // 函数定义表达式
- MemberExpression [ Expression ] // 属性访问表达式
- MemberExpression . IdentifierName // 属性访问表达式
- new MemberExpression Arguments // 对象创建表达式

#### 例子
```js
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

function baz(){
    console.log(this)
}

//示例1
console.log(foo.bar());
//示例2
console.log((foo.bar)());
//示例3
console.log((foo.bar = foo.bar)());
//示例4
console.log((false || foo.bar)());
//示例5
console.log((foo.bar, foo.bar)());
//示例6
baz()
```

- foo.bar(): 
    - MemberExpression 为 foo.bar
    - Reference 为 `base: foo, name: 'bar', strict: false`
    - IsPropertyReference 为 foo，为 true
    - 则 this = GetBase(ref) ，即为 foo。

- (foo.bar)():
    - foo.bar虽然被`()`包着，但`()`并没有对 MemberExpression 进行计算。
    - 所以(foo.bar)()===foo.bar()
    
- (foo.bar = foo.bar)()
    - 赋值操作符，使用了GetValue，则返回的不是Reference类型。
    - 则 this = undefined ，非严格模式下undefined 隐式转换为全局对象

- (false || foo.bar)()
    - 逻辑与算法，使用了GetValue，则返回的不是Reference类型。
    - this = undefined

- (foo.bar, foo.bar)()
    - 逗号操作符，使用了GetValue，则返回的不是Reference类型。

- baz()
    - Reference 为 `base: EnvironmentRecord, name: 'baz', strict: false`
    - IsPropertyReference 为 false
    - this = ImplicitThisValue(ref)，ImplicitThisValue函数始终返回 undefined。
    - this = undefined




