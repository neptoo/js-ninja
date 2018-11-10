# <center>JavaScript忍者秘籍</center>
##  简介
本文参考John Resig和Bear Bibeault的《JavaScript忍者秘籍》，是为方便未来学习和查阅JavaScript而创建的代码笔记。
## 目录
1. **[调试](#1-调试)**
2. **[函数](#2-函数)**
---
## 1. 调试
- 用于所有现代浏览器的日志记录
```javascript
function log(){
	try{
		console.log.apply(console,arguments);
		// 用console.log记录日志信息
	}
	catch(e){
		try{
			opera.postError.apply(opera,arguments);
			// 捕获失败，尝试过时版本的opera的专有方法记录
		}
		catch(e){
			alert(Arrary.prototype.join.call(arguments," "));
			// 如果都不行 使用alert()函数
		}
	}
}
```
- 用于测试jQuery的DOM测试用例
```html
<script src="dist/jquery.js"></script>
<script>
  $(document).ready(function(){
  	$("#test").append("test");
  });
</script>
<style>
  #test{ width:100px; height:100px; background:red; }
</style>
<div id="test"></div>
```
- 测试框架：QUnit | YUI Test |JSUnit
- Javascript assert()的简单实现
```html
<html>
<head>
	<title>Test Suite</title>
	<script>
		function assert(value,desc){
			var li = document.createElement("li");
			li.className = value ? "pass":"fail";
			li.appendChild(document.craeteTextNode(desc));
			document.getElementById("results").appendChild(li);
		}
		window.onload=function(){
			assert(true, "The test suite is running.");
			assert(false, "Fail!");
		};
	</script>
	<style>
		#result li.pass { color: green;}
		#result li.fail { color: red;}
	</style>
</head>
<body>
	<ul id="results"></ul>
</body>
</html>
```
- 测试分组
```html
<html>
<head>
	<title>Test Suite</title>
	<script>
		(function(){
			var results;
			this.assert=function assert(value,desc){
				var li = document.createElement("li");
				li.className = value ? "pass":"fail";
				li.appendChild(document.craeteTextNode(desc));
				document.getElementById("results").appendChild(li);
				if(!value){
					li.parentNode.parentNode.className="fail";
				}
				return li;
			};
			this.test=function test(name,fn){
				results=document.getElementById("results");
				results=assert(true,name).appendChild(document.createElement("ul"));
				fn();
			};
		})();
		
		window.onload=function(){
			test("A test.",function(){
				assert(true,"First assertion completed");
				assert(true,"Second assertion completed");
				assert(true,"Third assertion completed");
			});
			test("Another test.",function(){
				assert(true,"First assertion completed");
				assert(false,"Second test failed");
				assert(true,"Third assertion completed");
			});
			test("A third test.",function(){
				assert(null,"fail");
				assert(5,"pass");
			});
		};
	</script>

	<style>
		#result li.pass { color: green;}
		#result li.fail { color: red;}
	</style>
</head>
<body>
	<ul id="results"></ul>
</body>
</html>
```
**[返回目录](#目录)**

---
## 2. 函数

**[返回目录](#目录)**

---
