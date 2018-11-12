# <h1 align="center">JavaScript忍者秘籍</h1>
##  简介
本文参考John Resig和Bear Bibeault的《JavaScript忍者秘籍》，是为方便未来学习和查阅JavaScript而创建的代码笔记。
## 目录
1. **[调试](#1-调试)**
2. **[函数](#2-函数)**
3. **[函数(二)](#3-函数(二))**
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
- 测试框架：QUnit | YUI Test | JSUnit
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
- 异步测试
```html
<html>
<head>
	<title>Test Suite</title>
	<script>
		(function(){
			var quenue=[],paused=false, results;
			this.test=function(name, fn){
				quenue.push(function(){
					results=document.getElementById("results");
					results=assert(true,name).appendChild(document.createElement("ul"));
					fn();
				});
				runTest();
			};
			this.pause=function(){
				paused=true;
			};
			this.resume=function(){
				paused=false;
				setTimeOut(runTest,1);
			};
			function runTest(){
				if(!paused && quenue.length){
					quenue.shift()();
					if(!paused){
						resume();
					}
				}
			}

			this.assert=function assert(value,desc){
				var li=document.createElement("li");
				li.className=value ? "pass":"fail";
				li.appendChild(document.craeteTextNode(desc));
				results.appendChild(li);
				if(!value){
					li.parentNode.parentNode.className="fail";
				}
				return li;
			};
		})();
		window.onload=function(){
			test("Async Test #1",function(){
				pause();
				setTimeOut(function(){
					assert(true,"First test completed");
					resume();
				},1000);
			});
			test("Async Test #2",function(){
				pause();
				setTimeOut(function(){
					assert(true,"Second test completed");
					resume();
				},1000);
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
- 使用断言测试函数声明
```html
<script type="text/javascript">
	// 声明一个命名函数
	function isNimble(){ return true; }
	assert(typeof window.isNimble==="function","isNimble() defined");
	assert(typeof isNimble.name==="isNimble","isNimble() has a name");

	// 创建匿名函数，并赋值给变量
	var canFly=function(){ return true; };
	assert(typeof window.canFly==="function","canFly() defined");
	assert(canFly.name="","canFly() has no name");

	// 创建匿名函数，并引用到window的一个属性上
	window.isDeadly=function(){ return true; };
	assert(typeof window.isDeadly==="function","isDeadly() defined");

	function outer(){
		assert(typeof inner==="function","inner() in scope before declaration");
		function inner(){}
		assert(typeof inner==="function","inner() in scope after declaration");
		assert(window.inner===undefined,"inner() not in global scope");
	}
	// outer()可以在全局作用域访问到，inner()不行
	outer();
	assert(window.inner===undefined,"inner() still not in global scope");

	// 真正起控制作用的是该函数的真正的字面量名称
	window.wieldSword=function swingSword(){return true;};
	assert(window.wieldSword.name==='swingSword',"wieldSword's real name is swingSword");
</script>
```
- 作为函数调用与作为方法调用
```html
<script type="text/javascript">
	function creep(){ return this; }
	// 作为函数进行调用并验证该函数上下文是全局作用域
	assert(creep()===window,"creeping in window");

	var sneak=creep;
	// 使用sneak变量调用函数
	assert(sneak()===window,"sneaking in window");

	var ninja={
		skulk:creep
	};
	// 通过skulk属性调用，creep()作为ninja的一个方法进行调用
	assert(ninja.skulk()===ninja,"The 1st ninja is skulking");
</script>
```
- 使用构造器进行调用
创建一个名为Ninja()的函数，该函数将设置ninja的skulk技能，用于构建ninjas。
```html
<script type="text/javascript">
	funtion Ninja(){
		this.skulk=function(){ return this; };
	}
	var ninja1=new Ninja();
	var ninja2=new Ninja();

	assert(ninja1.skulk()===ninja1,"The 1st ninja is skulking");
	assert(ninja2.skulk()===ninja2,"The 2nd ninja is skulking");
</script>
```
**作为方法进行调用，该上下文是方法的拥有者；作为全局函数进行调用，上下文永远是window，作为构造器进行调用，其上下文是新创建的对象实例。在函数调用时，JavaScript提供了apply()和call()方法，可以显示指定任何一个对象为其函数的上下文。**
- 使用apply()和call()方法指定函数上下文
```html
<script type="text/javascript">
	function juggle(){
		var result=0;
		for(var n = 0; n <arguments.length; n++){
			result += arguments[n];
		}
		this.result = result;
	}
	var ninja1={};
	var ninja2={};
	juggle.apply(ninja1,[1,2,3,4]);
	juggle.call(ninja2,5,6,7,8);
	assert(ninja1.result===10,"juggled via apply");
	assert(ninja2.result===26,"juggled via call");
</script>
```
call()和apply()功能基本相同。如果在变量里有很多无关的值或者是指定为字面量，使用call()方法可以直接将其作为参数列表传进去。但是如果这些参数，已经在一个数组里，或者容易收集到数组里，apply()是更好的选择。

**[返回目录](#目录)**

---
## 3. 函数(二)
- 使用匿名函数的示例
```html
<script type="text/javascript">
  // 为load事件创建一个匿名函数作为事件处理程序
  window.onload = function() {
    assert(true, 'power!');
  };
  // 将其作为ninja的一个方法，使用shout属性调用
  var ninja = {
    shout: function() {
      assert(true, "Ninja");
    }
  };
  ninja.shout();
  // 作为参数传递给window对象的setTimeOut()函数
  setTimeOut(function() {
    assert(true, 'Forever!');
  }, 500);
</script>
```
- 使用内联函数进行递归
```html
<script type="text/javascript">
  var ninja = {
    // 定义内联函数signal 在函数体内使用名称进行递归调用
    chrip: function signal(n) {
      return n > 1 ? signal(n - 1) + "-chrip" : "chrip";
    }
  };
  // signal作为ninja对象的方法调用正常
  assert(ninja.chrip(3) == "chrip-chrip-chrip", "Works as we would expect!");
  // 将函数的引用复制给samurai
  var samurai = {
    chrip: ninja.chrip
  };
  // 清空ninja对象
  ninja = {};
  // 清除ninja的chrip属性 不影响内联函数用名字进行递归调用
  assert(samurai.chrip(3) == "chrip-chrip-chrip", "The method correctly calls itself.");
</script>
```
- 缓存记忆计算过的结果
```html
<script type="text/javascript">
  function isPrime(value) {
  	// 创建缓存
    if (!isPrime.answers) isPrime.answers = {};
    if (isPrime.answers[value] != null) {
      return isPrime.answers[value];
    }
    var prime = value != 1;
    for (var i = 2; i < value; i++) {
      if (value % i == 0) {
        prime = false;
        break;
      }
    }
    // 没有缓存，判断该值是否为素数，将结果缓存
    return isPrime.answers[value] = prime;
  }
  assert(isPrime(5), "5 is prime!");
  assert(isPrime.answers[5], "The answer was cached!");
</script>
```
- 缓存DOM元素
```javascript
function getElements(name) { 
	if(!getElements.cache) getElements.cache={}; 
	return getElements.cache[name]=getElements.cache[name] || document.getElementsByTagName(name);
}
```
**[返回目录](#目录)**

---
