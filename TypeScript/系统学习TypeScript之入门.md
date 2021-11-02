# 系统学习`TypeScript`
[《重学TS--阿宝哥》](http://book.bugstack.cn/#s/6TAYl8NQ)
## 类型
**注意：string和String,number和Number的区别。**
### `Any`类型和`Unknown`类型
- 两者都为TS的顶级类型
- `unknown`类型只能赋值给`any`类型和`unkonwn`类型本身
- `unkonwn`类型可以被赋任何值，但是不能做其他操作。
	```js
	let value: unknown;
	value.foo.bar; // Error
	value.trim(); // Error
	value(); // Error
	new value(); // Error
	value[0][1]; // Error
	// 若value为any类型，则以上操作是允许的
	```
### `Tuple`类型
元组类型。
```js
let tupleType: [string, boolean];
tupleType= ["semlinker", true];
```
### `Void`类型
声明一个`void`类型没有啥作用，在严格模式下，值只能为`undefined`。常见的场景是当函数没有返回值时，返回类型为`void`。
```js
let unusable: void = undefined;
```
### `Null`和`Undefined`类型
```js
// 拥有各自的类型
let u: undefined = undefined;
let n: null = null;
```
### `object`,`Object`和`{}`类型
#### `object`
`object`类型表示非原始类型
```js
// node_modules/typescript/lib/lib.es5.d.ts
interfaceObjectConstructor {
	create(o: object|null): any;	// 注意这里入参为object类型
	// ...
}
const proto= {};
Object.create(proto); // OK
Object.create(null); // OK
Object.create(undefined); // Error
Object.create(1337); // Error
Object.create(true); // Error
Object.create("oops"); // Error
```
#### `Object`
`Object`类型是所有`Object`类的*实例的类型*。由以下两个接口来定义。
##### `Object`接口
定义了[[Object.prototype]]原型对象上的属性。
```js
// node_modules/typescript/lib/lib.es5.d.ts
interface Object {
	constructor: Function;
	toString(): string;
	toLocaleString(): string;
	valueOf(): Object;
	hasOwnProperty(v: PropertyKey): boolean;
	isPrototypeOf(v: Object): boolean;
	propertyIsEnumerable(v: PropertyKey): boolean;
}
```
##### `ObjectConstructor`接口
定义了`Object`类的属性。
```js
// node_modules/typescript/lib/lib.es5.d.ts
interfaceObjectConstructor {
	/** Invocation via `new` */
	new(value?: any): Object;
	
	/** Invocation via function calls */
	(value?: any): any;
	
	readonly prototype: Object;
	getPrototypeOf(o: any): any;
	// ···
}
declare var Object: ObjectConstructor;
```
`Object`类的所有实例都继承`ObjectConstructor`接口中的所有熟悉。
#### `{}`类型
描述了一个没有成员的对象。只能访问原型链上的属性和方法。
```js
const obj = {};
obj.name = 'Elvis';	// Error: Property 'prop' does not exist on type '{}'.
obj.toString(); 	// [object object]
```
### `Never`类型
表示永不存在的值的类型。例如：
```js
// 返回never的函数必须存在无法达到的重点
function error(message: string): never {
	throw new Error(message);
}

function infiniteLoop(): never {
	while (true) {}
}
```
一个实现类型全面检查的场景：
```js
type Foo = string | number;
function controlFlowAnalysisWithNever(foo: Foo) {
	if (typeof foo === "string") {
		// 这⾥ foo 被收窄为 string 类型
	} else if (typeof foo === "number") {
		// 这⾥ foo 被收窄为 number 类型
	} else {
		// foo 在这⾥是 never
		const check: never = foo;
	}
}

// 如果改了Foo的类型，而没改controlFlowAnalysisWithNever，则else分支里foo类型会被收窄为boolean类型，则无法赋给check，报错。
type Foo = string | number | boolean;
```

## 断言
### 类型断言
类型断言有两种形式：
- 尖括号语法
	```js
	let someValue: any = "this is a string";
	let strLength: number = (<string>someValue).length;
	```
- as语法
	```js
	let someValue: any = "this is a string";
	let strLength: number = (someValue as string).length;
	```
### 非空断言
`!`可以断言为非`null`和非`undefined`类型。
忽略`undefined`和`null`类型，例：
```js
// 
function myFunc(maybeString: string | undefined | null) {
	// Type 'string | null | undefined' is not assignable to type 'string'.
	// 	Type 'undefined' is not assignable to type 'string'.
	const onlyString: string = maybeString; // Error
	const ignoreUndefinedAndNull: string = maybeString!; // Ok
}
```
调用函数忽略`undefined`类型，例：
```js
type NumGenerator = () => number;
function myFunc(numGenerator: NumGenerator | undefined) {
	// Object is possibly 'undefined'.(2532)
	// Cannot invoke an object which is possibly 'undefined'.(2722)
	const num1 = numGenerator(); // Error
	const num2 = numGenerator!(); //OK
}
```
使用时注意下面这种情况：
```js
const a: number | undefined = undefined;
const b: number = a!;
console.log(b);
// 以上代码可以通过ts的检查，但生成的js会这样：
"use strict";
const a = undefined;
const b = a;
console.log(b);	// undefined。虽然ts断言为number，但是编译为js后还是为undefined
```
### 确定赋值断言
告诉ts该属性会被明确地赋值。例：
```js
let x: number;
initialize();
// Variable 'x' is used before being assigned.(2454)
console.log(2 * x); // Error
function initialize() {
	x = 10;
}

// 可以通过确定赋值断言解决
let x!: number;
initialize();
console.log(2 * x); // Ok
function initialize() {
	x = 10;
}
```
## 类型守卫
类型守卫（类型保护）是指在运行时确保类型在一定范围内的表达式。其主要思想是尝试检测属性、⽅法或原型，以确定如何处理值。
### `in`关键字
例：
```js
interface Admin {
	name: string;
	privileges: string[];
}
interface Employee {
	name: string;
	startDate: Date; 
}
type UnknownEmployee = Employee | Admin;
function printEmployeeInformation(emp: UnknownEmployee) {
	console.log("Name: " + emp.name);
	if ("privileges" in emp) {
		console.log("Privileges: " + emp.privileges);
	}
	if ("startDate" in emp) {
		console.log("Start Date: " + emp.startDate);
	}
}
```
### `typeof`关键字
例：
```js
function padLeft(value: string, padding: string | number) {
	if (typeof padding === "number") {
		return Array(padding + 1).join(" ") + value;
	}
	if (typeof padding === "string") {
		return padding + value;
	}
	throw new Error(`Expected string or number, got '${padding}'.`);
}
```
要利用typeof关键字变成类型守卫，则必须是：
- `typeof v === [typename]`或
- `typeof v !== [typename]`
typename必须为`number`、`string`、`boolean`或`symbol`。
### `instanceof`关键字
例：
```js
interface Padder {
	getPaddingString(): string;
}
class SpaceRepeatingPadder implements Padder {
	constructor(private numSpaces: number) {}
	getPaddingString() {
		return Array(this.numSpaces + 1).join(" ");
	}
}
class StringPadder implements Padder {
	constructor(private value: string) {}
	getPaddingString() {
		return this.value;
	}
}
let padder: Padder = new SpaceRepeatingPadder(6);
if (padder instanceof SpaceRepeatingPadder) {
	// padder的类型收窄为 'SpaceRepeatingPadder'
}
```
### 自定义类型保护的类型谓词
*使用场景？*
例：
```js
function isNumber(x: any): x is number {
	return typeof x === "number";
}
function isString(x: any): x is string {
	return typeof x === "string"; 
}
```
## 联合类型和类型别名

### 联合类型
联合类型通常和`null`或`undefined`一起使用：
```js
const sayHello = (name: string | undefined) => {
	/* ... */
};

// 也可以用来约束取值。1、2或者'click'被称为字面量类型。
let num: 1 | 2 = 1;
type EventNames = 'click' | 'scroll' | 'mousemove';
```
### 可辨识联合
可辨识联合=可辨识+联合类型+类型守卫。本质是联合类型和字面量类型结合的类型保护方法。什么时候使用呢？**如果⼀个类型是多个类型的联合类型，且多个类型含有⼀个公共属性，那么就可以利⽤这个公共属性，来创建不同的类型保护区块。（结合例子仔细阅读！）**
- 可辨识
	```js
	// 可辨识要求联合类型中的每个元素都含有⼀个单例类型属性。例如vType，它作为一个可辨识的属性。
	enum CarTransmission {
		Automatic = 200,
		Manual = 300
	}
	interface Motorcycle {
		vType: "motorcycle"; // discriminant
		make: number; // year
	}
	interface Car {
		vType: "car"; // discriminant
		transmission: CarTransmission
	}
	interface Truck {
		vType: "truck"; // discriminant
		capacity: number; // in tons
	}
	```
- 联合类型
	```js
	// 基于上面的接口，创建一个联合类型
	type Vehicle = Motorcycle | Car | Truck;
	```
- 类型守卫
	```js
	const EVALUATION_FACTOR = Math.PI;
	// 根据车辆的类型、容量和评估因子计算价格
	function evaluatePrice(vehicle: Vehicle) {
		return vehicle.capacity * EVALUATION_FACTOR;
	}
	const myTruck: Truck = { vType: "truck", capacity: 9.5 };
	evaluatePrice(myTruck);
	
	// 以上代码ts会报错
	// Property 'capacity' does not exist on type 'Vehicle'.
	// Property 'capacity' does not exist on type 'Motorcycle'.
	// 所以需要类型守卫
	function evaluatePrice(vehicle: Vehicle) {
	switch(vehicle.vType) {
		case "car":
			return vehicle.transmission * EVALUATION_FACTOR;
		case "truck":
			return vehicle.capacity * EVALUATION_FACTOR;
		case "motorcycle":
			return vehicle.make * EVALUATION_FACTOR;
		}
	}
	```
**以上三种特效结合才是可辨识联合。**
### 类型别名
给类型起一个新名字。
```js
type Message = string | string[];
let greet = (message: Message) => {
	// ...
};
```
## 交叉类型
```js
type PartialPointX = { x: number; };
type Point = PartialPointX & { y: number; };
let point: Point = {
	x: 1,
	y: 1
}
```
### 同名基础类型属性的合并
合并多个类型时，如果出现成员相同但类型不同的情况，
例：
```js
interface X {
	c: string;
	d: string;
}
interface Y {
	c: number;
	e: string
}
type XY = X & Y;
type YX = Y & X;
let p: XY;
let q: YX;
```
结论：该成员会变成`never`类型。因为该例子中的c不存在同时满足既是`string`又是`number`的类型，则混入的类型为`never`。
### 同名非基础类型属性的合并
上面例子的同名成员c不一致的类型正好都为基本数据类型，如果是非基本数据类型呢？
```js
interface D { d: boolean; }
interface E { e: string; }
interface F { f: number; }
interface A { x: D; }
interface B { x: E; }
interface C { x: F; }
type ABC = A & B & C;
let abc: ABC = {
	x: {
		d: true,
		e: 'semlinker',
		f: 666
	}
};
// 以上代码ts可以编译通过，且abc内成员确实合并了。
```
## 函数
### 参数类型和返回类型
```js
function createUserId(name: string, id: number): string {
	return name + id;
}
```
### 函数类型
```js
let IdGenerator: (chars: string, nums: number) => string;	
// 变量接受一个函数，函数的入参类型和返回值类型已确定。

function createUserId(name: string, id: number): string {
	return name + id;
}
IdGenerator = createUserId;
```
### 可选参数和默认参数
```js
// 可选参数
function createUserId(name: string, id: number, age?: number): string {
	return name + id;
}
// 默认参数
function createUserId(
	name = "semlinker",
	id: number,
	age?: number
): string {
	return name + id;
}
```
**需要注意，可选参数必须放在普通参数后面，否则ts编译出错。**
### 函数重载
函数重载或⽅法重载是使⽤相同名称和不同参数数量或类型创建多个⽅法的⼀种能⼒。
例：
```js
type Combinable = string | number;

function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a: string, b: number): string;
function add(a: number, b: string): string;
function add(a: Combinable, b: Combinable) {
	if (typeof a === 'string' || typeof b === 'string') {
		return a.toString() + b.toString();
	}
	return a + b;
}
```
成员方法一致，例：
```js
type Combinable = string | number;

class Calculator {
	add(a: number, b: number): number;
	add(a: string, b: string): string;
	add(a: string, b: number): string;
	add(a: number, b: string): string;
	add(a: Combinable, b: Combinable) {
	if (typeof a === 'string' || typeof b === 'string') {
		return a.toString() + b.toString();
	}
		return a + b;
	}
}
const calculator = new Calculator();
const result = calculator.add('abc', ' efg');
```
**注意：TS处理函数重载时，会尝试使用第一个重载定义，所以要把最精确的定义放第一。**
## 数组
### 数组结构
```js
let x: number; let y: number; let z: number;
let five_array = [0,1,2,3,4];
[x,y,z] = five_array;
```
### 展开运算符
```js
let two_array = [0, 1];
let five_array = [...two_array, 2, 3, 4];
```
### 数组遍历
```js
let colors: string[] = ["red", "green", "blue"];
for (let i of colors) {
	console.log(i);
}
```
## 对象
### 对象结构
```js
let person = {
	name: "Semlinker",
	gender: "Male",
};
let { name, gender } = person;
```
### 展开运算符
```js
let person = {
	name: "Semlinker",
	gender: "Male",
	address: "Xiamen",
};
// 组装对象
let personWithAge = { ...person, age: 33 };
// 获取除了某些项外的其它项
let { name, ...rest } = person;
```
## TS接口
- 对对象的形状（Shape）进⾏描述
- 对类的⼀部分⾏为进⾏抽象
### 描述对象的形状（Shape）
```js
interface Person {
	name: string;
	num: number;
}
let semlinker: Person = {
	name: "abc",
	num: 123,
};
```
### 抽象类的⾏为
#### 类实现接口
```js
interface Alarm {
    alert(): void;
}

class Door {
}

class SecurityDoor extends Door implements Alarm {	// 注意这里的写法
    alert() {
        console.log('SecurityDoor alert');
    }
}

class Car implements Alarm {
    alert() {
        console.log('Car alert');
    }
}

// 实现多个接口
interface Alarm {
    alert(): void;
}

interface Light {
    lightOn(): void;
    lightOff(): void;
}

class Car implements Alarm, Light {
    alert() {
        console.log('Car alert');
    }
    lightOn() {
        console.log('Car light on');
    }
    lightOff() {
        console.log('Car light off');
    }
}
```
#### 接口继承接口
```js
interface Alarm {
    alert(): void;
}

interface LightableAlarm extends Alarm {
    lightOn(): void;
    lightOff(): void;
}
```
#### 接口继承类
常见的面向对象语言中，接口是不能继承类的，但TS可以。
```js
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```
为什么可以这样做呢？因为，当我们在声明 `class Point` 时，除了会创建一个名为 `Point` 的类之外，同时也创建了一个名为 `Point` 的类型（实例的类型）。所以，我们既可以把`Point`当成类使用，也可以把`Point`当成类型使用。
```js
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
	print() {
		console.log(this.x, this.y);
	}
}
const p = new Point(1, 2);

function printPoint(p: Point) {
    console.log(p.x, p.y);
}
printPoint(new Point(1, 2));

// 相当于TS自动声明了一个类型。并且成员不包括类的构造函数、静态属性或静态方法。因为这个接口其实是用来描述类的实例的。
interface PointInstanceType {
    x: number;
    y: number;
	print(): void;
}
```
### 可选|只读属性
```js
interface Person {
	readonly name: string;	// 只能在初始化的时候赋值
	age?: number;
}
```
- `ReadonlyArray<T>`类型
	```js
	// ReadonlyArray<T>类型与Array<T>类型相似，但它可以确保数组创建后再也不能被修改。
	let a: number[] = [1, 2, 3, 4];
	let ro: ReadonlyArray<number> = a;
	
	// error: Index signature in type 'readonly number[]' only permits reading.
	ro[0] = 12;
	
	// error: Property 'push' does not exist on type 'readonly number[]'.
	ro.push(5); 
	
	// error: Cannot assign to 'length' because it is a read-only property.
	ro.length = 100; 
	
	// error: The type 'readonly number[]' is 'readonly' and cannot be assigned to the mutable type 'number[]'.
	a = ro; 
	```
### 任意属性
```js
interface Person {
	name: string;
	age?: number;
	[propName: string]: any;	// 索引签名
}
const p = { name: "kakuqo", sex: 1 }
```
### 接口和类型别名的区别
#### Object/Function
接口和类型别名都可以描述对象形状和函数签名。
#### Other Types
#### Extend
#### Implements
#### Declaration Mergin


# 参考
[TypeScript入门教程](https://ts.xcatliu.com/advanced/class-and-interfaces.html)