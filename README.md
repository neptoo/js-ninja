# <h1 align="center">JavaScript忍者秘籍</h1>
##  简介
本文参考John Resig和Bear Bibeault的《JavaScript忍者秘籍》，是为方便未来学习和查阅JavaScript而创建的代码笔记。
## 目录
1. **[调试](#1-调试)**
2. **[函数](#2-函数)**
3. **[函数续](#3-函数续)**
4. **[闭包](#4-闭包)**
5. **[原型与面向对象](#5-原型与面向对象)**
---
## 1. 调试
- 用于所有现代浏览器的日志记录
```javascript
function log() {
    try {
        console.log.apply(console, arguments);
        // 用console.log记录日志信息
    } catch (e) {
        try {
            opera.postError.apply(opera, arguments);
            // 捕获失败，尝试过时版本的opera的专有方法记录
        } catch (e) {
            alert(Arrary.prototype.join.call(arguments, " "));
            // 如果都不行 使用alert()函数
        }
    }
}
```
- 用于测试jQuery的DOM测试用例
```html
<script src="dist/jquery.js"></script>
<script>
$(document).ready(function() {
    $("#test").append("test");
});
</script>
<style>
#test {
    width: 100px;
    height: 100px;
    background: red;
}
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
    function assert(value, desc) {
        var li = document.createElement("li");
        li.className = value ? "pass" : "fail";
        li.appendChild(document.craeteTextNode(desc));
        document.getElementById("results").appendChild(li);
    }
    window.onload = function() {
        assert(true, "The test suite is running.");
        assert(false, "Fail!");
    };
    </script>
    <style>
    #result li.pass {
        color: green;
    }
    #result li.fail {
        color: red;
    }
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
    (function() {
        var results;
        this.assert = function assert(value, desc) {
            var li = document.createElement("li");
            li.className = value ? "pass" : "fail";
            li.appendChild(document.craeteTextNode(desc));
            document.getElementById("results").appendChild(li);
            if (!value) {
                li.parentNode.parentNode.className = "fail";
            }
            return li;
        };
        this.test = function test(name, fn) {
            results = document.getElementById("results");
            results = assert(true, name).appendChild(document.createElement("ul"));
            fn();
        };
    })();

    window.onload = function() {
        test("A test.", function() {
            assert(true, "First assertion completed");
            assert(true, "Second assertion completed");
            assert(true, "Third assertion completed");
        });
        test("Another test.", function() {
            assert(true, "First assertion completed");
            assert(false, "Second test failed");
            assert(true, "Third assertion completed");
        });
        test("A third test.", function() {
            assert(null, "fail");
            assert(5, "pass");
        });
    };
    </script>
    <style>
    #result li.pass {
        color: green;
    }

    #result li.fail {
        color: red;
    }
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
    (function() {
        var quenue = [],
            paused = false,
            results;
        this.test = function(name, fn) {
            quenue.push(function() {
                results = document.getElementById("results");
                results = assert(true, name).appendChild(document.createElement("ul"));
                fn();
            });
            runTest();
        };
        this.pause = function() {
            paused = true;
        };
        this.resume = function() {
            paused = false;
            setTimeOut(runTest, 1);
        };

        function runTest() {
            if (!paused && quenue.length) {
                quenue.shift()();
                if (!paused) {
                    resume();
                }
            }
        }

        this.assert = function assert(value, desc) {
            var li = document.createElement("li");
            li.className = value ? "pass" : "fail";
            li.appendChild(document.craeteTextNode(desc));
            results.appendChild(li);
            if (!value) {
                li.parentNode.parentNode.className = "fail";
            }
            return li;
        };
    })();
    window.onload = function() {
        test("Async Test #1", function() {
            pause();
            setTimeOut(function() {
                assert(true, "First test completed");
                resume();
            }, 1000);
        });
        test("Async Test #2", function() {
            pause();
            setTimeOut(function() {
                assert(true, "Second test completed");
                resume();
            }, 1000);
        });
    };
    </script>
    <style>
    #result li.pass {
        color: green;
    }

    #result li.fail {
        color: red;
    }
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
<script type = "text/javascript" >
// 声明一个命名函数
function isNimble() { return true; }
assert(typeof window.isNimble === "function", "isNimble() defined");
assert(typeof isNimble.name === "isNimble", "isNimble() has a name");

// 创建匿名函数，并赋值给变量
var canFly = function() { return true; };
assert(typeof window.canFly === "function", "canFly() defined");
assert(canFly.name = "", "canFly() has no name");

// 创建匿名函数，并引用到window的一个属性上
window.isDeadly = function() { return true; };
assert(typeof window.isDeadly === "function", "isDeadly() defined");

function outer() {
    assert(typeof inner === "function", "inner() in scope before declaration");

    function inner() {}
    assert(typeof inner === "function", "inner() in scope after declaration");
    assert(window.inner === undefined, "inner() not in global scope");
}
// outer()可以在全局作用域访问到，inner()不行
outer();
assert(window.inner === undefined, "inner() still not in global scope");

// 真正起控制作用的是该函数的真正的字面量名称
window.wieldSword = function swingSword() { return true; };
assert(window.wieldSword.name === 'swingSword', "wieldSword's real name is swingSword"); 
</script>
```
- 作为函数调用与作为方法调用
```html
<script type="text/javascript">
function creep() { return this; }
// 作为函数进行调用并验证该函数上下文是全局作用域
assert(creep() === window, "creeping in window");
var sneak = creep;
// 使用sneak变量调用函数
assert(sneak() === window, "sneaking in window");
var ninja = {
skulk: creep
};
// 通过skulk属性调用，creep()作为ninja的一个方法进行调用
assert(ninja.skulk() === ninja, "The 1st ninja is skulking");
</script>
```
- 使用构造器进行调用
创建一个名为Ninja()的函数，该函数将设置ninja的skulk技能，用于构建ninjas。
```html
<script type="text/javascript">
funtion Ninja() {
    this.skulk = function() { return this; };
}
var ninja1 = new Ninja();
var ninja2 = new Ninja();
assert(ninja1.skulk() === ninja1, "The 1st ninja is skulking");
assert(ninja2.skulk() === ninja2, "The 2nd ninja is skulking");
</script>
```
**作为方法进行调用，该上下文是方法的拥有者；作为全局函数进行调用，上下文永远是window，作为构造器进行调用，其上下文是新创建的对象实例。在函数调用时，JavaScript提供了apply()和call()方法，可以显示指定任何一个对象为其函数的上下文。**
- 使用apply()和call()方法指定函数上下文
```html
<script type="text/javascript">
  function juggle() {
      var result = 0;
      for (var n = 0; n < arguments.length; n++) {
          result += arguments[n];
      }
      this.result = result;
  }
  var ninja1 = {};
  var ninja2 = {};
  juggle.apply(ninja1, [1, 2, 3, 4]);
  juggle.call(ninja2, 5, 6, 7, 8);
  assert(ninja1.result === 10, "juggled via apply");
  assert(ninja2.result === 26, "juggled via call");
</script>
```
call()和apply()功能基本相同。如果在变量里有很多无关的值或者是指定为字面量，使用call()方法可以直接将其作为参数列表传进去。但是如果这些参数，已经在一个数组里，或者容易收集到数组里，apply()是更好的选择。

**[返回目录](#目录)**

---
## 3. 函数续
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
- 检测并遍历可变长度的参数列表
```html
<script type="text/javascript">
    function merge(root) {
        for (var i = 1; i < arguments.length; i++) {
            for (var key in arguments[i]) {
                root[key] = arguments[i][key];
            }
        }
        return root;
    }
    // 调用merge()函数
    var merged = merge({ name: "Batou" }, { city: "Nihama" });
    assert(merged.name == "Batou", "The original name is intact.");
    assert(merged.city == "Nihama", "And the city has been copied over.");
    </script>
```
- 对arguments列表进行切片
```html
<script type="text/javascript">
    function multiMax(multi) {
        return multi * Math.max.apply(Math, Array.prototype.slice.call(arguments, 1));
        /* 如果是multi*Math.max.apply(Math,arguments.slice(1));会报错
        因为arguments参数引用的不是真正的数组；Array.prototype.slice()这一原生JS数组方法，
        通常是通过其函数上下文操作数组的，这里通过call()方法将我们的对象强制作为slice()方法的上下文*/
        assert(multiMax(3, 1, 2, 3) == 9, "3*3=9 (第一个参数,剩余最大参数.)");
    }
    </script>
```
- 重载函数的方法及测试
```html
<script type="text/javascript">
    function addMethod(object, name, fn) {
        // 保存原有函数，因为调用的时候可能不匹配传入的参数个数
        var old = object[name];
        // 创建一个新匿名函数作为新方法
        object[name] = function() {
            // 若匿名函数的形参个数和实参个数匹配，调用该函数
            if (fn.length == arguments.length)
                return fn.apply(this, arguments)
            // 若传入参数不匹配，调用原来的参数
            else if (typeof old == 'function')
                return old.apply(this, arguments);
        };
    }
    var ninjas = {
        values: ["Dave Edwards", "Sam Stephen", "Alex Russell"]
    };
    // 在基础对象上绑定一个无参数方法
    addMethod(ninjas, "find", function() {
        return this.values;
    });
    // 在基础对象上绑定一个单参数的方法
    addMethod(ninjas, "find", function(name) {
        var ret = [];
        for (var i = 0; i < this.values.length; i++)
            if (this.values[i].indexOf(name) == 0)
                ret.push(this.values[i]);
        return ret;
    });
    // 在基础对象上绑定两个参数的方法
    addMethod(ninjas, "find", function(first, last) {
        var ret = [];
        for (var i = 0; i < this.values.length; i++)
            if (this.values[i] == (first + " " + last))
                ret.push(this.values[i]);
        return ret;
    });
    assert(ninjas.find().length == 3, "Found all ninjas");
    assert(ninjas.find("Sam").length == 1, "Found ninja by first name");
    assert(ninjas.find("Dave", "Edwards").length == 1, "Found ninja by first and last name");
    assert(ninjas.find("Alex", "Russell", "Jr") == null, "Found nothing");
    </script>
```
- 函数判断
如何判断一个给定对象是一个函数的实例，并且是可调用的。通常typeof语句就可以满足要求。但也有跨浏览器的问题：Firefox--在html的object元素上使用typeof，会返回function而不是object；IE--IE会将DOM元素的方法报告成object,如typeof domNode.getAttribute=="object"等；Safari--Safari认为DOM的NodeList是一个function。所以typeof childNodes=="function"。

**[返回目录](#目录)**

---
## 4. 闭包
- 不那么简单的闭包

```html
<script type="text/javascript">
var outValue = 'ninja';
var later;
function outerFunction() {
    var innerValue = 'samurai';
    function innerFunction(paramValue) {
        assert(outerValue, "Inner can see the ninja.");
        assert(innerValue, "Inner can see the samurai.");
        assert(paramValue, "Inner can see the wakiz.");
        assert(tooLate, "Inner can see the ronin.");
    }
    later = innerFunction;
}
assert(!tooLate, "Outer can't see the ronin.");
var tooLate = 'ronin';
outerFunction();
later('wakiz');
</script>
```
内部闭包可以访问到tooLate，而外部闭包不能，因为：作用域之外的所有变量，即便是函数声明之后的那些声明，也都包含在闭包中；相同的作用域内，尚未声明的变量不能进行提前引用。
- 使用闭包封装一些信息作为"私有变量"
```html
<script type="text/javascript">
function Ninja() {
    // 在函数(构造器)内部声明一个私有变量
    var feints = 0;
    // 创建一个访问feints计数的方法，该变量在构造器内部无法被访问，只能读取
    this.getFeints = function() {
        return feints;
    };
    this.feint = function() {
        feints++;
    };
}
var ninja = new Ninja();
ninja.feint();
// 验证我们不能直接获取该变量值
assert(ninja.getFeints() == 1, "We're able to access the internal feint count.");
// 我们可以操作feints的值，因为即便构造器执行完且没有作用域了，feints变量还是会绑定在feint()方法声明创建的闭包上，并且可以在feint()方法内使用
assert(ninja.feints === undefined, "And the private data is inaccessible to us.");
</script>
```
- 在ajax请求的callback里使用闭包
```html
<div id="testSubject"></div>
<button type="button" id="testButton">Go!</button>
<script type="text/javascript">
  jQuery('#testButton').click(function(){
    var elem$ = jQuery("#testSubject");
    elem$.html("Loading...");
    jQuery.ajax({
      url("test.html"),
      success: function(html){
        // 定义一个匿名函数作为响应回调，在回调中通过闭包引用了elem$变量，使用该变量将响应文本填充到div元素中
        assert(elem$,"We can see elem$, via the closure for this callback.");
        elem$.html(html);
      }
    });
  });
</script>
```
- 计时器中使用闭包
```html
<div id="box">Some text</div>
<script type="text/javascript">
  function animateIt(elementId){
    var elem = document.getElementById(elementId);
    var tick = 0;
    var timer = setInterval(function(){
      if(tick < 100) {
        elem.style.left = elem.style.top = tick + "px";
        tick++;
      }
      else {
        clearInterval(timer);
        assert(tick == 100, "Tick accessed via a closure.");
        assert(elem, "Elment also accessed via a closure.");
        assert(timer, "Timer reference also obtained via a closure.");
      }
    },10);
  }
  animateIt('box');
</script>
```
- 给事件处理程序绑定特定的上下文
```html
<button id="test">Click it!</button>
<script type="text/javascript">
  // bind()方法创建并返回一个匿名函数，可以强制将上下文设置为想要的任何对象
  function bind(context,name){
    return function(){
        return context[name].apply(context,arguments);
    };
  }
  var button = {
    clicked: false;
    click: function(){
      this.clicked = true;
      assert(button.clicked,"The button has been clicked.");
      concole.log(this);
    }
  };
  var elem = document.getElementById("test");
  elem.addEventListener("click",bind(button,"click"),false);
</script>
```
- 使用闭包实现缓存记忆
```html
<script type="text/javascript">
Function.prototype.memorized = function(key) {
    this._values = this._values || {};
    return this._values[key] !== undefined ? this._values[key] : this._values[key] = this.apply(this, arguments);
};
Function.prototype.memorize = function() {
    var fn = this;
    // 通过变量赋值将上下文带到闭包中
    return function() {
        // 在缓存记忆函数中封装原始函数
        return fn.memorized.apply(fn, arguments);
    };
};
var isPrime = (function(num) {
    var prime = num != 1;
    for (var i = 2; i < num; i++) {
        if (num % i == 0) {
            prime = false;
            break;
        }
    }
    return prime;
}).memorize();
assert(isPrime(17), "17 is prime");
</script>
```
- 利用即时函数处理迭代问题
```html
<div>DIV 0</div>
<div>DIV 1</div>
<script type="text/javascript">
var div = document.getElementsByTagName("div");
// 通过for循环内加入即时函数，将正确的值传递给即时函数，进而让处理程序得到正确的值
for (var i = 0; i < div.length; i++)(function(n) {
    div[n].addEventListener("click", function() {
        assert("div #" + n + "was clicked.");
    }, false);
}
})(i);
</script>
```

**[返回目录](#目录)**

---
## 5. 原型与面向对象
- 使用原型方法创建一个新实例
```html
<script type="text/javascript">
function Ninja() {}
Ninja.prototype.swingSword = function() {
    return true;
};
// 将函数作为构造器进行调用，不仅新对象实例被创建，函数原型上的方法也可以调用了
var ninja = new Ninja();
// 判断一个实例的类型以及其构造器
assert(typeof ninja == "object", "The type of instance is object.");
assert(ninja instanceof Ninja, "instanceof identifies the constructor.");
assert(ninja.constructor == Ninja, "The ninja object was created by the Ninja function.");
</script>
```
- 使用constructor实例化一个新对象
```html
<script type="text/javascript">
  function Ninja(){}
  var ninja = new Ninja();
  var ninja2 = new ninja.constructor();
  assert(ninja2 instanceof Ninja,"It's a ninja");
  assert(ninja != ninja2,"But not the same Ninja");
</script>
```
- 使用原型实现继承
```html
<script type="text/javascript">
  function Person(){}
  Person.prototype.dance = function(){};
  function Ninja(){}
  Ninja.prototype = new Person();
  var ninja = new Ninja();
  assert(ninja instanceof Ninja, "ninja receives functionality from Ninja prototype.");
  assert(ninja instanceof Person, "...and Person prototype.");
  assert(ninja instanceof Object, "...and Object prototype.");
  assert(typeof ninja.dance == "function","...and can dance!");
</script>
```
- forEach()兼容旧版本浏览器
```html
<script type="text/javascript">
if (!Array.prototype.forEach) {
    Array.prototype.forEach = function(callback, context) {
        for (var i = 0; i < this.length; i++) {
            // 在每个数组条目上都调用callback方法 context||null表达式可防止将undefined传递给call()
            callback.call(context || null, this[i], i, this);
        }
    };
}
["a", "b", "c"].forEach(function(value, index, array) {
    assert(value, "Is in position" + index + " out of " + (array.length - 1));
});
</script>
```
- 通过HTMLElement的原型给所有html元素添加方法
```html
<div id="parent">
    <div id="a">To be removed</div>
    <div id="b">Me too!</div>
</div>
<script type="text/javascript">
HTMLElement.prototype.remove = function() {
    if (this.parentNode) {
        this.parentNode.removeChild(this);
    }
};
// 用原生方法删除元素a
var a = document.getElementById("a");
a.parentNode.removeChild(a);
// 用新方法删除元素b
document.getElementById("b").remove();
assert(!document.getElementById("a"), "a is gone");
assert(!document.getElementById("b"), "b is gone too");
</script>
```
- 使用hasOwnProperty()方法解决原型对象拓展问题
```html
<script type="text/javascript">
Object.prototype.keys = function() {
    var keys = [];
    for (var i in ths)
        // 忽略掉非实例对象的属性
        if (this.hasOwnProperty(i)) keys.push(i);
    return keys;
};
var obj = { a: 1, b: 2, c: 3 };
assert(obj.keys().length == 3, "There are three properties in this object.");
</script>
```
- 模拟Array，而不是扩展成子类,可兼容所有浏览器
```html
<script type="text/javascript">
// 创建一个含有原型属性length的新类
function MyArray() {}
MyArray.prototype.length = 0;
// 使用即时函数，用apply()将Array中选中方法复制到新类上
(function() {
    var methods = ['push', 'pop', 'shift', 'unshift', 'slice', 'splice', 'join'];
    for (var i = 0; i < methods.lenth; i++)(function name) {
        MyArray.prototype[name] = function() {
            return Array.prototype[name].apply(this, arguments);
        };
    }(methods[i]);
})();
var mine = new MyArray();
mine.push(1, 2, 3);
assert(mine.length == 3,"All the items are on our sub-classed array.");
assert(!(mine instanceof Array),"We aren't subclassing Array,though.");
</script>
```
- 经典继承语法示例
```html
<script type="text/javascript">
  var Person = Object.subClass() {
      init: function() { this.dancing = isDancing; },
      dance: function() { return this.dancing; }
  };
  var Ninja = Person.subClass() {
      init: function() { this._super(false); },
      dance: function() { return this._super(); },
      swingSword: function() { return true; }
  };
var person = new Person(true);
assert(person.dance(),"The person is dancing");
var ninja = new Ninja();
assert(ninja.swingSword(),"The sword is swinging");
assert(!ninja.dance(),"The ninja is not dancing");
assert(person instanceof Person,"person is a Person");
assert(ninja instanceof Ninja && ninja instanceof Person,"ninja is a Ninja and a Person");
</script>
```

**[返回目录](#目录)**

---
