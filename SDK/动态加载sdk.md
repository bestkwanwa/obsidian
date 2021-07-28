# 新sdk
## Application类
```js
let Application=realibox.studio
let app=new Application(canvas,options)
```
- canvas 画布
- options
	- dracoDecoderConfig draco解码器配置
	- mode （不知道有什么mode）
	- inputSettings
		- useMouse
		- useTcouch
		- useGamepads
	- assetPrefix 资源路径前缀（应该是加载数据为local时才需要设置）
	- preferWebGl2 是否优先使用webgl2
	- showGrid 是否显示网格（不知道干嘛）
### 实例方法
- loadData(data, options)
	- data 场景数据（info）
	- options
		- id 当前数据的唯一ID，重复ID的数据将不会进行初始化
		- postRendering 后期渲染，将在数据加载完成后渲染在场景中（可以不阻塞loading页渲染）
		- mountNodeID 挂载节点ID，挂载参数允许将数据挂载在其他场景的子节点上，在被挂载节点发生运动时跟随运动
		- mountNodeEntityID 挂载节点的渲染对象ID，具体的渲染对象节点
- removeData(id)
	- id为loadData设置的id
- resize() 重置应用画布大小（监听屏幕尺寸变化）
- destroy()
	```js
	var app = new realibox.studio(canvas, options);
	app.destroy();
	app = null;
	```
```js

// 可以针对某个场景进行操作
var app = new realibox.studio(canvas, options);
app.loadData(data, {
     id: 'realibox',
     postRendering: true
});
app.events.on('edk:scene:resource:preload:end', (id) => {
     if( id === 'realibox' ){
         // 在前一个数据加载完成后，加载第二个数据
         app.loadData(data, {
             id: 'realibox2',
             postRendering: true,
             mountNodeID: 'realibox',
             mountNodeEntityID: 'test' // 此处的渲染对象ID从编辑器中获取
         });
     }
});
```
```js
// 移除场景数据
var app = new realibox.studio(canvas, options);
app.loadData(data, {
     id: 'realibox',
     postRendering: true
});
app.events.on('edk:scene:resource:preload:end', (id) => {
     if( id === 'realibox' ){
         // 在主数据加载完成后，加载第二个数据
         app.loadData(data, {
             id: 'realibox2',
             postRendering: true,
             mountNodeID: 'realibox',
             mountNodeEntityID: 'test' // 此处的渲染对象ID从编辑器中获取
         });
     } else if( id === 'realibox2' ) {
         // 在第二个数据加载完成后两秒，移除数据
         setTimeout(() => {
             app.app.removeData('realibox2')
         }, 2000)
     }
});
```
### 实例事件
- edk:app 获取应用实例
	```js
	var app = new realibox.studio(canvas, options);
	var app2 = app.events.call('edk:app');
	console.log(app === app2); // true
	```
- edk:app:resize 重置应用画布大小
	```js
	var app = new realibox.studio(canvas, options);
	app.events.call('edk:app:resize');
	```
### SDK事件系统
- on 给事件绑定事件处理函数
- off 解绑事件处理函数
- fire 广播一个事件，让所有事件监听器接收传参
- once 事件处理函数触发一次后删除
- hasEvent 检测是否绑定了事件处理函数
- call 执行一个事件，接收fire传过来的附加参数

## SceneResource类
场景资源管理类，用于场景中资源管理的实现
- constructor()
	系统内部类，外部仅能通过Event Method调用

### 事件
- edk:scene:resource:preload:start
	广播场景资源开始加载事件，通过events.on监听广播事件
	```js
	var app = new realibox.studio(canvas, options);
	// sceneID 开始加载资源的场景id
	app.events.on('edk:scene:resource:preload:start', (sceneID) => {
   		console.log('场景ID：' + sceneID + '开始加载')
	});
	```
- edk:scene:resource:preload:end
	广播场景资源加载结束事件，通过events.on监听广播事件
	```js
	var app = new realibox.studio(canvas, options);
	app.events.on('edk:scene:resource:preload:end', (sceneID) => {
     console.log('场景ID：' + sceneID + '加载结束')
	});
	```
- edk:scene:resource:preload:progress
	广播场景资源开始正在加载中的事件，通过events.on监听广播事件
	- sceneID 正在加载资源的场景ID
	- progress
		- length 当前加载中的资源长度
		- count 当前已加载完成的数量，无论成功失败
		- successful 当前已加载成功的数量
		- aborted 当前已加载失败的数量
		- progress 当前加载的进度，通过 count / length 计算得出
	```js
	var app = new realibox.studio(canvas, options);
	app.events.on('edk:scene:resource:preload:progress', (sceneID, progress) => {
     console.log('场景ID：' + sceneID + '已加载' + progress.progress)
	});
	```
	
## SceneSystem类
场景系统，用于管理多场景中的API和数据
- constructor
	系统内部类，外部仅能通过Event Method调用

### 事件
- edk:scene:resource:get:default 
	获取指定场景的默认资源 （不懂）
	```js
	var app = new realibox.studio(canvas, options);
	var materialData = app.events.call('edk:scene:resource:get:default', 'material');
	```
- edk:scene:resource:get
	获取指定场景的资源
	- sceneID 场景ID，要运行哪个场景的时间线，可在加载数据时设定（loadData的id？）
	- resourceID 资源ID，要获取指定场景中的资源
	```js
	var app = new realibox.studio(canvas, options);
	var resource = app.events.call('edk:scene:resource:get', 'sceneID', 'resourceID');
	```
- edk:scene:resource:create
	创建一个资源（不知道有啥用）
- edk:scene:interaction:run
	运行交互事件时间线
	- sceneID 场景ID，要运行哪个场景的时间线，可在加载数据时设定
	- timelineID 时间线ID，在编辑器时间线面板处获取
	```js
	var app = new realibox.studio(canvas, options);
	app.events.call('edk:scene:interaction:run', 'sceneID', 'timelineID');
	```
- edk:scene:interaction:stop
	停止运行交互事件时间线
	```js
	// 参数与运行事件线一致
	var app = new realibox.studio(canvas, options);
	app.events.call('edk:scene:interaction:stop', 'sceneID', 'timelineID');
	```
- edk:scene:interaction:pause
	暂停运行交互事件时间线（参数与上一个事件一致）
- edk:scene:interaction:resume
	恢复被暂停运行交互事件时间线（参数与上一个事件一致）
- edk:scene:interaction:clear
	清除允许中断的交互事件时间线（参数与上一个事件一致）（不知道又啥用）
	
## 实例事件总览
- edk:app 
- edk:app:resize
- edk:scene:resource:preload:start
- edk:scene:resource:preload:end
- edk:scene:resource:preload:progress
- edk:scene:resource:get:default
- edk:scene:resource:get
- edk:scene:resource:create
- edk:scene:interaction:run
- edk:scene:interaction:stop
- edk:scene:interaction:pause
- edk:scene:interaction:resume
- edk:scene:interaction:clear
	
	