# 笔记

## 内存管理

### **为什么要管理内存:**

- 减少浏览器的负担:内存过大会导致浏览器压力过大,导致浏览器卡顿
- node端:内存如果不够,服务就会终端,而nodejs开启的服务,如果不管理内存,就会中断

**堆空间:  **队列优先,先进先出   数据结构(二叉树)  一般存储key-value形式的对象

**栈空间: ** 先进后出	数据结构(先进后出的数据结构,如杯子中放东西,先放的后拿,后方的先拿)   一般存储基本数据类型和堆中对象的引用

![](.\neicun.png)

![](.\neicun2xin.png)

![](.\neicun2lao.png)

### **新生代和老生代如何转化:**

- 新生代发现本次复制后,会占用超过百分之二十五的to空间
- 这个对象已经经历过一次回收

### **监测内存:**

- process.memoryUsage()  ===>Node端
- window.performance.memory  ===>浏览器端

### **Node特殊点:**

1. Node可以手动触发垃圾回收-   global.gc
2. Node端可以设置内存-  node --max-old-space-size=1700 test.js和node --max-new-space-size=1024 test.js

## 函数式编程后执行的一些问题

**值的传递写起来不方便**

**连续调用写起来很麻烦**

```javascript
/**
* Compose函数
* Compose函数可以理解为为了方便我们连续执行方法,把自己调用传值的过程封装了起来,
* 我们只需要给compose函数我们要执行哪些方法,他会自动执行(compose从右向左执行)
*/
function compose() {
	var arr = [].slice.apply(arguments)
	return function(num) {
		var _result = num
		/**方式一*/
		// for(var i = arr.length-1;i>=0;i--){
		// 	_result = arr[i](_result)
		// }
		// return _result
		/**方式二(reduceRight(当前参数,当前需要执行方法,下标,目标对象)) 从最后一个开始*/
		return arr.reduceRight(
			(res, cb, index, obj) {
				console.log(res,cb,index,obj)
				return cb(res)
			}, num)
	}
}
//调用 fun3>fun2>fun1 倒序执行
compose(fun1,fun2,fun3...)(num)
/**
* pipe函数
* pipe函数功能和compose功能一样只是执行顺序相反
*/
function pipe() {
	var arr = [].slice.apply(arguments)
	return num => {
		var _result = num
		/**方式一*/
		// for(var i=0;i<=arr.length-1;i++){
		// 	_result = arr[i](_result)
		// }
		// return _result
		/**
		* 方式二
		* reduce(当前参数,当前需要执行方法,下标,目标对象) 从第一个开始
		* */
		return arr.reduce((res, cb, index, obj) => {
			console.log(res, cb, index, obj)
			return cb(res)
		}, num)
	}
}
//调用 fun1>fun2>fun3 顺序执行
pipe(fun1,fun2,fun3...)(num)

//链式调用
Promise.resolve(num).then(fun1).then(fun2).then(fun3)...
```



## 高阶函数

### **apply:**  

​			**`apply()`** 方法调用一个具有给定`this`值的函数，以及以一个数组（或[类数组对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Indexed_collections#working_with_array-like_objects)）的形式提供的参数 

 			**备注：**call()方法的作用和 apply() 方法类似，区别就是`call()`方法接受的是**参数列表**，而`apply()`方法接受的是**一个参数数组**。 

​			**语法:*func.apply(thisArg,[argsArray])***

​			**thisArg:** 必选的。在 *`func`* 函数运行时使用的 `this` 值。请注意，`this`可能不是该方法看到的实际值：如果这个函数处于[非严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)下，则指定为 `null` 或 `undefined` 时会自动替换为指向全局对象，原始值会被包装。

​			 **argsArray:** 可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 `func` 函数。如果该参数的值为 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null) 或 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象。[浏览器兼容性](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply#浏览器兼容性) 请参阅本文底部内容。 

​			**返回值:**  调用有指定`**this**`值和参数的函数的结果。 

​			**示例:将一个数组合并到一个数组中**

```javascript

//手写apply
Function.prototype.myApply = function(thisArg){
	var _thisArg = thisArg || window
	var _Args = arguments[1]
	_thisArg.fn = this
	var result = _thisArg.fn(..._Args)
	delete _thisArg.fn
	return result
}
var array = [1,2,3]
var ags = [4,5,6]
array.push.myApply(array,ags)
console.log(array) //[1,2,3,4,5,6]
```



### **call:** 

​		     **`call()`**  方法使用一个指定的 值和单独给出的一个或多个参数来调用一个函数。`this` 

 			**备注：**该方法的语法和作用与 [`apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法类似，只有一个区别，就是**call()**方法接受的是**一个参数列表**，而 **apply()**方法接受的是**一个包含多个参数的数组**。`call() apply()`  

​			**语法:*func.call(thisArg,arg1,arg2,...)***

​			**thisArg:**  可选的。在 *`function`* 函数运行时使用的 值。请注意，可能不是该方法看到的实际值：如果这个函数处于[非严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)下，则指定为 或 时会自动替换为指向全局对象，原始值会被包装。`this this null undefined` 

​			 **arg1,arg2,...:**  指定的参数列表。 

​			**返回值:**   使用调用者提供的 值和参数调用该函数的返回值。若该方法没有返回值，则返回 。`this undefined`  

​			**示例:将一个数组合并到一个数组中**

```javascript

//手写call
Function.prototype.myCall = function(thisArg){
	var _thisArg = thisArg || window
	var _Args = [],i=1;
    for(i;i<=arguments.length-1;i++){
        _Args.push(arguments[i])
    }
	_thisArg.fn = this
	var result = _thisArg.fn(..._Args)
	delete _thisArg.fn
	return result
}
var array = [1,2,3]
var ags = [4,5,6]
array.push.myCall(array,...ags)
console.log(array) //[1,2,3,4,5,6]
```



### **bind:** 

​			 **`bind()`**   方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。  

​			**备注:** 由于返回的是一个函数,所以不会立即执行,而`apply()`和`call()`都可以立即执行

​			**语法:*function.bind(thisArg[, arg1[, arg2[, ...]]])***

​			**thisArg:**   调用绑定函数时作为 `this` 参数传递给目标函数的值。 如果使用[`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)运算符构造绑定函数，则忽略该值。当使用 `bind` 在 `setTimeout` 中创建一个函数（作为回调提供）时，作为 `thisArg` 传递的任何原始值都将转换为 `object`。如果 `bind` 函数的参数列表为空，或者`thisArg`是`null`或`undefined`，执行作用域的 `this` 将被视为新函数的 `thisArg`。 

​			 **arg1,arg2,...:**   当目标函数被调用时，被预置入绑定函数的参数列表中的参数。  

​			**返回值:**    返回一个原函数的拷贝，并拥有指定的 **`this`** 值和初始参数。 

​			**示例:将一个数组合并到一个数组中**

```javascript
//手写bind
Function.prototype.myBind = function(thisArg){
	var _thisArg = thisArg || window
	var _Args = []
	var i=1
	for(i;i<=arguments.length-1;i++){
		_Args.push(arguments[i])
	}
	var _self = this;
	return function(){
		return _self.apply(thisArg,_Args)
	}
}
var array = [1,2,3]
var a = array.push.myBind(array,4,5,6)
console.log(a(),array) //6,[1,2,3,4,5,6]
```



## 防抖节流

**相同点:**防抖和节流都是为了阻止操作高频触发,从而浪费性能

**不同点:**防抖是多次触发,只生效最后一次.适用于我们只需要一次触发生效的场景  节流是让操作每隔一段才能触发一次,适用于我们多次触发要多次生效的场景

例:

```javascript
//防抖
function AntiShake (fn,time){
	let timid = null
	return function(){
		clearTimeout(timid)
		timid = setTimeout(()=>{
			return fn.apply(this,arguments)
		},time)
	}
}
//使用
AntiShake((e)=>{
    /**代码块*/
},1000)
//节流
function throttle(fn,time){
	let timid = null
	return function(){
		if(timid)return
		else {
			timid = setTimeout(()=>{
				fn.apply(this,arguments)
				timid=null
			},time)
		}
	}
}
throttle((e)=>{
    /**代码块*/
},1000)
```



## 考核测试

### 函数

#### 以下哪些方法在javaScript中可以获取数据类型?

- [x] Typeof
- [x] instanceOf
- [x] toString
- [ ] undefined

解析:获取数据类型有五种方法

1. typeof

   *typeof返回的是表示数据类型的字符串,可以判断基本数据类型,对引用类型的判断都是object*

   ```javascript
   typeof 123 //number
   typeof [1,2,3] //object
   typeof null //object
   ```

2. instanceof

    *instanceof用来判断A是否为B的实例，不能判断A具体为什么数据类型，B一定要是对象类型，区分大小写*

   ```javascript
   [] instanceof Array //true
   {} instanceof Object //true
   ```

   注: *判断一个变量为数组还是对象，可以通过instanceof反向推出。[] instanceof Object为true，但是{} instanceof Array为false。* 

3. constructor

   *通过constructor来判断数据的类型，但是**除了null、undefined**，因为他们不是由对象构建。数字、布尔值、字符串是包装类对象，所以有constructor*

   ```javascript
   ''.constructor == String //true
   var a = 2;a.constructor == Number //true
   [].constructor == Array //true
   ```

   

4. Object.prototype.toString.call()

    *Object的toString方法可以精准的判断对象的数据类型* 

   ```javascript
   Object.prototype.toString.call('') ;  // [object String]
   Object.prototype.toString.call(1) ;   // [object Number]
   Object.prototype.toString.call(true) ;// [object Boolean]
   Object.prototype.toString.call(Symbol());//[object Symbol]
   Object.prototype.toString.call(undefined) ;// [object Undefined]
   Object.prototype.toString.call(null) ;// [object Null]
   Object.prototype.toString.call(new Function()) ;// [object Function]
   Object.prototype.toString.call(new Date()) ;// [object Date]
   Object.prototype.toString.call([]) ;// [object Array]
   Object.prototype.toString.call(new RegExp()) ;// [object RegExp]
   Object.prototype.toString.call(new Error()) ;// [object Error]
   Object.prototype.toString.call(document) ;// [object HTMLDocument]
   Object.prototype.toString.call(window) ;//[object global] window 是全局对象 global 的引用
   
   ```

   

5. Array.isArray()

    *ECMAScript5将Array.isArray()正式引入JavaScript，目的就是准确地检测一个值是否为数组。* 

```javascript
Array.isArray([1, 2, 3]);  // true
Array.isArray({foo: 123}); // false
Array.isArray("foobar");   // false
Array.isArray(undefined);  // false
```

#### 以下关于作用域的说法中正确的是?

- [x] Var 声明时全局作用域或函数作用域,而let和const是块作用域
- [ ] let的声明会被提升到全局中
- [x] var的声明会被提升到全局中
- [x] const不可重复声明





#### 以下关于普通函数和箭头函数的说法中正确的有哪些?

- [x] 普通函数可通过bind/call/apply改变this指向

- [x] 普通函数可使用new

- [x] 箭头函数本身没有this指向

- [ ] 箭头函数可以通过bind/call/apply改变this指向

  箭头函数:

  1. 本身没有this指向
  2. 他的this在定义的时候继承自外层第一个普通函数的this
  3. 被继承的普通函数的this指向改变,箭头函数的this指向也会跟着改变
  4. 箭头函数外层如果没有普通函数,this指向window
  5. 不能通过bind/call/apply改变this指向
  6. 使用new调用箭头函数,因为箭头函数没有Constructor

  普通函数:

  - 可以通过**bind**/**call**/**apply**改变this指向

  - 可以使用new关键字

    



#### 下列关于闭包中说法正确的有哪些?

- [ ] 闭包使已经运行结束的函数上下文中的变量对象在内存中清除
- [x] 创建闭包的最常见的方式就是在一个函数内创建另一个函数
- [x] 闭包使我们在函数外部能够访问到函数内部的变量
- [x] 闭包的本质就是作用域链的一个特殊应用

**什么是闭包:**

​		闭包就是能够读取其他函数内部变量的函数,闭包使指有权访问另一个函数作用域中变量的函数,创建闭包的最常见发方式就是在一个函数中创建另一个函数,通过另一个函数访问这个函数的局部变量,利用闭包可以突破作用域链

**变量的作用域:**

​        根据作用域的不同，JavaScript 中的变量可分为两种：**全局变量**和**局部变量**。 

​        **函数内部**可以直接读取**全局变量**; 但是，**函数外部**不能读取函数内的**局部变量**

**闭包的作用:**

1. 在函数外部访问到函数内部的变量
2. 使变量留存在内存中
3. 模拟私有方法

**闭包的副作用:**

- **常驻内存**

  *闭包会使得函数中的变量都被保存在**内存**中，内存消耗很大，不能滥用闭包，否则会造成严重的性能问题，在**IE浏览器**中可能由于**循环引用**而导致**内存泄露**。*

  **解决方法**：在退出函数之前，将不使用的**局部变量**全部清除。

- **误改函数内部的值**

   *不要随便改变父函数**内部变量**的值。* 

  

#### 以下哪些方法可以解决跨域问题?

- [x] 通过jsonp跨域

- [x] Document.domain+iframe 跨域

- [x] Window.name + iframe 跨域

- [x] PostMessage 跨域

  **解决跨域的方式**

  1. document.domain + iframe的设置
  2. 动态创建script
  3. 利用iframe + location.hash
  4. window.name 实现的跨域数据传输
  5. 使用HTML5 postMessage
  6. 利用flash
  7. 通过jsonp跨域
  8. 跨域资源共享(CORS)
  9. nodeJavaScript 中间件代理跨域
  10. WebSocket协议跨域
  11. nginx代理跨域

####  以下哪些方法可以解决浏览器缓存问题？ 

- [x] 在Ajax发送请求前加上anyAjaxObj.setRequestHeader("If-Modified-Since",'0')

- [x]  在 Ajax 发送请求前加上 anyAjaxObj.setRequestHeader(“Cache-Control”,”no-cache”) 

- [x]  在 URL 后面加上一个随机数：”fresh=” + Math.random(); 

- [ ]  在 URL 后面加上时间戳：”nowtime=” + new Date().getTime(); 

  **解决浏览器缓存问题的几种方式:**

  1. meta方式

     ```javascript
     <META HTTP-EQUIV="pragma" CONTENT="no-cache"> 
     <META HTTP-EQUIV="Cache-Control" CONTENT="no-cache, must-revalidate"> 
     <META HTTP-EQUIV="expires" CONTENT="0">
     ```

  2. 清理form表单的临时缓存:

     -  用ajax请求服务器最新文件，并加上请求头If-Modified-Since和Cache-Control 

     -  直接用cache:false 

     -  用随机数，随机数也是避免缓存的一种很不错的方法！ 

     -  用随机时间，和随机数一样。 

     -  window.location.replace("WebForm1.aspx");

       

       ```javascript
       //方式一
       $.ajax({
            url:'www.haorooms.com',
            dataType:'json',
            data:{},
            beforeSend :function(xmlHttp){ 
               xmlHttp.setRequestHeader("If-Modified-Since","0"); 
               xmlHttp.setRequestHeader("Cache-Control","no-cache");
            },
            success:function(response){
                //操作
            }
            async:false
         });
       //方式二
        $.ajax({
            url:'www.haorooms.com',
            dataType:'json',
            data:{},
            cache:false, 
            ifModified :true ,
       
            success:function(response){
                //操作
            }
            async:false
         });
       //方式三
       URL 参数后加上 "?ran=" + Math.random(); //当然这里参数 ran可以任意取了
       eg:
       <script> 
       document.write("<s"+"cript type='text/javascript' src='/js/test.js?"+Math.random()+"'></scr"+"ipt>"); 
       </script>
       
       其他的类似，只需在地址后加上+Math.random() 
       注意：因为Math.random() 只能在Javascript 下起作用，故只能通过Javascript的调用才可以 
       
       //方式四
       在 URL 参数后加上 "?timestamp=" + new Date().getTime(); 
       在服务端加 header("Cache-Control: no-cache, must-revalidate");等等(如php中)
       
       //方式五
       window.location.replace("WebForm1.aspx");   
       参数就是你要覆盖的页面，replace的原理就是用当前页面替换掉replace参数指定的页面。   
       这样可以防止用户点击back键
       ```

       

####  以下关于微任务和宏任务说法正确的有哪些？ 

- [ ]  JavaScript是单线程的，不分同步异步 

- [x] 微任务和宏任务皆为异步任务，它们都属于一个队列 

- [x] 宏任务一般是：script，setTimeout，setInterval、setImmediate 

- [ ]  遇到宏任务，先执行宏任务，执行完后如果没有宏任务，就执行下一个微任务，如果有宏任务，就按顺序一个一个执行宏任务 

  **宏任务与微任务的区分:**

  1. JavaScript是单线程的,但是分同步异步
2. 微任务与宏任务都是异步任务,他们都属于一个队列
  3. 宏任务一般是:script  setTimeout  setInterval  setImmediate
4. 微任务:原生Promise
  5. 遇到微任务,先执行微任务,执行完后如果没有微任务,就执行下一个宏任务,如果有微任务,就按顺序一个一个执行微任务

  

  **宏任务和微任务的执行顺序:**

  *每一个宏任务执行完之后,都会检查是否存在待执行的微任务,如果有,则执行完所有微任务后,再继续执行下一个宏任务*

  **什么是宏任务和微任务**

  ​	js把异步任务做了划分,分为两类:

  1. 宏任务
  
     异步ajax请求 / setTimeout / setInterval / 文件操作(IO) / MessageChannel / setImmediate(node环境) / script
  
  2. 微任务
  
     promise.then / .catch / .finally / process.nextTick /  MutationObserver / queueMicrotask
  
  
  

####  解决JavaScript的加载、解析和执行会阻塞页面渲染过程问题的方法有哪些？ 

- [x]  我们一般采用的是将 JavaScript 脚本放在文档的底部，来使 JavaScript 脚本尽可能的在最后来加载执行 

- [x]  给 JavaScript 脚本添加 defer 属性，这个属性会让脚本的加载与文档的解析同步解析，然后在文档解析完成后再执行这个脚本文件 

- [x]  给 JavaScript 脚本添加 Async 属性，这个属性会使脚本异步加载，不会阻塞页面的解析过程 

- [x]  动态创建 DOM 标签的方式，对文档的加载事件进行监听，当文档加载完成后再动态的创建 script 标签来引入 JavaScript 脚本 

  阻塞:

  1. CSS不会阻塞DOM解析，但是会阻塞DOM渲染，严谨一点则是CSS会阻塞render tree的生成，进而会阻塞DOM的渲染
  2. JS会阻塞DOM解析
  3. CSS会阻塞JS的执行
  4. 浏览器遇到<script>标签且没有defer或async属性时会触发页面渲染
  5. Body内部的外链CSS较为特殊，请慎用

  解决方案:

  1.  将 JavaScript 脚本放在文档的底部，来使 JavaScript 脚本尽可能的在最后来加载执行 
  2.  给 JavaScript 脚本添加 defer 属性，这个属性会让脚本的加载与文档的解析同步解析，然后在文档解析完成后再执行这个脚本文件，这样的话就能使页面的渲染不被阻塞。多个设置了 defer 属性的脚本按规范来说最后是顺序执行的，但是在一些浏览器中可能不是这样 
  3.  给 JavaScript 脚本添加 Async 属性，这个属性会使脚本异步加载，不会阻塞页面的解析过程，但是当脚本加载完成后立即执行 JavaScript 脚本，这个时候如果文档没有解析完成的话同样会阻塞。多个 Async 属性的脚本的执行顺序是不可预测的，一般不会按照代码的顺序依次执行 
  4.  动态创建 DOM 标签的方式，我们可以对文档的加载事件进行监听，当文档加载完成后再动态的创建 script 标签来引入 JavaScript 脚本。 

