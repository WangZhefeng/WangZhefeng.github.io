---
title: 事件
author: 王哲峰
date: '2020-11-01'
slug: js-event
categories:
  - 前端
tags:
  - tool
---

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
h2 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}

details {
    border: 1px solid #aaa;
    border-radius: 4px;
    padding: .5em .5em 0;
}

summary {
    font-weight: bold;
    margin: -.5em -.5em 0;
    padding: .5em;
}

details[open] {
    padding: .5em;
}

details[open] summary {
    border-bottom: 1px solid #aaa;
    margin-bottom: .5em;
}
</style>

<details><summary>目录</summary><p>

- [事件](#事件)
	- [事件流](#事件流)
		- [事件冒泡](#事件冒泡)
		- [事件捕获](#事件捕获)
		- [DOM 事件流](#dom-事件流)
	- [事件处理程序](#事件处理程序)
		- [HTML 事件处理程序](#html-事件处理程序)
		- [DOM0 事件处理程序](#dom0-事件处理程序)
		- [DOM2 事件处理程序](#dom2-事件处理程序)
		- [IE 事件处理程序](#ie-事件处理程序)
		- [跨浏览器事件处理程序](#跨浏览器事件处理程序)
	- [事件对象](#事件对象)
	- [事件类型](#事件类型)
	- [内存与性能](#内存与性能)
	- [模拟事件](#模拟事件)
</p></details><p></p>

# 事件

- JavaScript 与 HTML 的交互式通过事件实现的，事件代表文档或浏览器窗口中某个有意义的时刻
- 可以使用仅在事件发生时执行的监听器(也叫处理程序)订阅事件
	- 在传统软件工程领域，这个模型叫做“观察者模式”，其能够做到页面行为 (在 JavaScript 中定义) 与页面展示 (在 HTML 和 CSS 中定义) 的分离

## 事件流

- 事件流描述了页面接收事件的顺序
	- IE 支持事件冒泡流
	- Netscape Communicator 支持事件捕获流

### 事件冒泡

- IE 事件流被称为事件冒泡，这是因为事件被定义为从最具体的元素（文档树中最深的节点）开始触发，然后向上传播至没有那么具体的元素（文档）

- 所有的现代浏览器都支持事件冒泡，只是在实现方式上会有一些变化

	- IE5.5 及早期版本会跳过 `<html>`元素（从 `<body>`直接到 document）。
	- 现代浏览器中的时间会一直冒泡到 window 对象

- 示例

	- HTML 页面

	```html
	<!DOCTYPE html>
	<html>
	    <head>
	        <title>Event Bubbling Example</title>
	    </head>
	    <body>
	        <div id="myDiv">
	            Click Me
	        </div>
	    </body>
	</html>
	```

	- 在点击页面中的 `<div>`元素后，click 事件会以如下顺序发生：
		- （1）`<div>`
		- （2）`<body>`
		- （3）`<html>`
		- （4）document
	- 过程图

	<img src="/Users/zfwang/Library/Application Support/typora-user-images/image-20210412172308547.png" alt="image-20210412172308547" style="zoom:50%;" />

### 事件捕获

- Netscape Communicator 团队提出了另一种名为事件捕获的事件流。事件捕获的意思是最不具体的节 点应该最先收到事件，而最具体的节点应该最后收到事件。事件捕获实际上是为了在事件到达最终目标前拦截事件
- 虽然这是 Netscape Communicator 唯一的事件流模型，但事件捕获得到了所有现代浏览器的支持
	- 所有浏览器都是从 window 对象开始捕获事件
	- DOM2 Events 规范规定的是从 document 开始
- 示例
	- 在点击页面中的 `<div>`元素后，click 事件会以如下顺序发生：
		- （1）document
		- （2）`<html>`
		- （3）`<body>`
		- （4）`<div>`
- 过程图

<img src="/Users/zfwang/Library/Application Support/typora-user-images/image-20210412172706526.png" alt="image-20210412172706526" style="zoom:50%;" />

### DOM 事件流

- DOM2 Events 规范规定事件流分为 3 个阶段：
	- 事件捕获：事件捕获最先发生，为提前拦截事件提供了可能
	- 到达目标：然后，实际的目标元素接收到事件后
	- 时间冒泡：最后一个阶段是冒泡，最迟要在这个阶段响应事件
- 所有现代浏览器都支持 DOM 事件流，只有 IE8 及更早版本不支持
- 大多数支持 DOM 事件流的浏览器实现了一个小小拓展。虽然 DOM2 Events 规范明确捕获阶段不命中事件目标，但现代浏览器都会在捕获阶段在事件目标上触发事件。最终的结果是在事件目标上有两个机会来处理事件
- 过程图

<img src="/Users/zfwang/Library/Application Support/typora-user-images/image-20210412173132838.png" alt="image-20210412173132838" style="zoom:50%;" />

## 事件处理程序

- 事件意味着用户或浏览器执行的某种动作，比如：
	- 单击（click）
	- 加载（load）
	- 鼠标悬停（mouseover）
- 为响应时间而调用的函数被称为**事件处理程序**（或**事件监听器**）
	- 事件处理程序的名字以 “on”开头，因此 
		- click 事件的处理程序叫作 onclick
		- load 事件的处理程序叫作 onload
- 有很多方式可以指定事件处理程序
	- HTML
	- DOM0
	- DOM2
	- IE

### HTML 事件处理程序

- 特定元素支持的每个事件都可以使用事件处理程序的名字以 HTML 属性的形式来指定，此时属性的值必须是能够执行的 JavaScript 代码

	- 作为事件处理程序执行的代码可以访问全局作用域中的一切
	- 在 HTML 中指定时间处理程序有一些问题：
		- （1）时机问题：有可能 HTML 元素已经显示在页面上，用户都与其交互了，而事件处理程序的代码还无法执行。为此，大多数 HTML 事件处理程序会封装在 try/catch 块中，以便在这种情况下静默失败
		- （2）对事件处理程序作用域链的扩展在不同浏览器中可能导致不同的结果。不同 JavaScript 引擎中标识符解析的规则存在差异，因此访问无限定的对象成员可能导致错误
		- （3）HTML 与 JavaScript 强耦合。如果需要修改事件处理程序，则必须在两个地方，即 HTML 和 JavaScript 中，修改代码。

- 示例

	- （1）要在按钮被点击时执行某些 JavaScript 代码，可以使用以下 HTML 属性，点击这个按钮后，控制台会输出一条消息，这种交互能力是通过为 onclick 属性指定 JavaScript 代码来实现的

		- 因为属性的值是 JavaScript 代码，所以不能在未经转义的情况下使用 HTML 语法字符，比如：&、"、<、>
		- 为了避免使用 HTML 实体，可以使用单引号代替双引号，如果确实需要使用双引号，将双引号用过 &quot 代替

		```html
		<input type="button" value="Click Me" onclick="console.log('Clicked')">
		<input type="button" value="Click Me" onclick="console.log(&quot;Clicked&quot;)"> // 
		```

	- （2）在 HTML 中定义的事件处理程序可以包含精确的动作指令，也可以调用在页面其他地方定义的脚本

		```html
		<script>
			function showMessage() {
		        console.log("Hello World!");
		    }
		</script>
		<input type="button" value="Click Me" onclick="showMessage()">
		```

	- （3）上面两种方式指定的事件处理程序，首先会创建一个函数来封装属性的值

		- 这个函数有一个特殊的局部变量 event，其中保存的就是 event 对象。有了这个对象，就不用另外定义其他变量，也不用从包装函数的参数列表中去取了

		```html
		<!-- 输出 "click" -->
		<input type="button" value="Click Me" onclick="console.log(event.type)">
		```

		- 在这个函数中，this 值相当于事件的目标元素

		```html
		<!-- 输出 "Click Me" -->
		<input type="button" value="Click Me" onclick="console.log(this.value)">
		```

		- 这个动态创建的包装函数其作用域链被扩展了。在这个函数中，document 和元素自身的成员都可以被当成局部变量来访问，这是通过使用 with 实现的

		```js
		// 1.document 和元素自身的成员都可以被当成局部变量来访问
		function () {
		    with(document) {
		        with(this) {
		            // 属性值
		        }
		    }
		}
		// 2.如果这个元素是一个表达输入框，则作用域中还会包含表单元素，事件处理程序对应的函数等价于
		function () {
		    with(document) {
		        with(this.form) {
		            with(this) {
		                // 属性值
		            }
		        }
		    }
		}
		```

		```html
		<!-- 1. 输出 "Click Me" -->
		<input type="button" value="Click Me" onclick="console.log(value)">
		
		<!-- 2. -->
		<form method="post">
		    <input type="text" name="username" value="">
		    <input type="button" value="Echo Username" onclick="console.log(username.value)">
		</form>
		```

	- （4）try/catch

		```html
		<input type="button" value="Click Me" onclick="try{showMessage();} catch(ex) {}">
		```

### DOM0 事件处理程序

- 在 JavaScript 中指定事件处理程序的传统方式是把一个函数赋值给（DOM元素的）一个事件处理程序属性
	- 这也是在第四代 Web 浏览器中开始支持的事件处理程序赋值方法，直到现在所有现代浏览器 仍然都支持此方法，主要原因是简单
	- 要使用 JavaScript 指定事件处理程序，必须先取得要操作对象的引用
- 每个元素（包括 window 和 document）都有通常小写的事件处理程序属性，比如：onclick，只要把这个属性赋值为一个函数即可。像这样使用 DOM0 方式为事件处理程序赋值时，所赋函数被视为元素的方法，因此，事件处理程序会在元素的作用域中运行，即 this 等于元素

```js
let btn = document.getElementById("myBtn");
btn.onclick = function () {
    console.log("Clicked");
}

let btn = document.getElementById("myBtn");
btn.onclick = function () {
    console.log(this.id); // "myBtn"
}
```

- 通过将事件处理程序属性的值设置为 null，可以移除通过 DOM0 方式添加的事件处理程序，把事件处理程序设置为 null，再点击按钮就不会执行任何操作了

```js
btn.onclick = null; // 移除事件处理程序
```

### DOM2 事件处理程序

### IE 事件处理程序

### 跨浏览器事件处理程序





## 事件对象

## 事件类型



## 内存与性能



## 模拟事件