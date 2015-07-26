
## Key

* 值传递
* 不允许堆内存操作
* 函数 是 对象

## 函数作用域

通过声明函数方式定义函数，javascript引擎在预处理过程里就把函数定义和赋值操作都完成了

javascript里预处理的特性: 预处理是和执行环境相关。执行环境有两大类：全局执行环境和局部执行环境，执行环境是通过上下文变量体现的，其实这个过程都是在函数执行前完成，预处理就是构造执行环境的另一个说法，总而言之预处理和构造执行环境的主要目的就是明确变量定义，分清变量的边界。但是在全局作用域构造或者说全局变量预处理时候对于声明函数有些不同，声明函数会将变量定义和赋值操作同时完成。由于声明函数都会在全局作用域构造时候完成，因此声明函数都是window对象的属性，这就说明为什么我们不管在哪里声明函数，声明函数最终都是属于window对象。

在JavaScript中，使用function关键字声明的函数（不赋给变量）也会被提升。实际上，整个函数被提升，可供执行。

    myFunction(); //输出 "i exist"
     
    function myFunction() {
      console.log('i exist');
    }

当使用var形式来声明函数时，整个函数不会被提升：

    try {
      myFunction();
    } catch (e) {
      console.log(e); //抛出 "Uncaught TypeError: undefined is not a function"
    }
    var myFunction = function() {
      console.log('i exist');
    }
     
    myFunction();  //输出 "i exist"

Javascript还有一种方式可以改变this指针，这就是call方法和apply方法，call和apply方法的作用相同，就是参数不同，call和apply的第一个参数都是一样的，但是后面参数不同，apply第二个参数是个数组，call从第二个参数开始后面有许多参数。Call和apply的作用是什么，这个很重要，重点描述如下：

Call和apply是改变函数的作用域（有些书里叫做改变函数的上下文），Call和apply是将this指针指向方法的第一个参数。

迷惑：我们定义对象使用对象的字面表示法，字面表示法在简单的表示里我们很容易知道this指向对象本身，但是这个对象会有方法，方法的参数可能会是函数，而这个函数的定义里也可能会使用this指针，如果传入的函数没有被实例化过和被实例化过，this的指向是不同，有时我们还想在传入函数里通过this指向外部函数或者指向被定义对象本身，这些乱七八糟的情况使用交织在一起导致this变得很复杂，结果就变得糊里糊涂。

以定义对象里的方法里传入函数为例：

＊情形一：传入的参数是函数的别名，那么函数的this就是指向window；
＊情形二：传入的参数是被new过的构造函数，那么this就是指向实例化的对象本身；
＊情形三：如果我们想把被传入的函数对象里this的指针指向外部字面量定义的对象，那么我们就是用apply和call

    <script type="text/javascript">
    var name = "I am window";
    var obj = {
        name:"sharpxiajun",
        job:"Software",
        ftn01:function(obj){
            obj.show();
        },
        ftn02:function(ftn){
            ftn();
        },
        ftn03:function(ftn){
            ftn.call(this);
        }
    };
    function Person(name){
        this.name = name;
        this.show = function(){
            console.log("姓名:" + this.name);
            console.log(this);
        }
    }
    var p = new Person("Person");
    obj.ftn01(p);
    obj.ftn02(function(){
       console.log(this.name);
       console.log(this);
    });
    obj.ftn03(function(){
        console.log(this.name);
        console.log(this);
    });
    </script>
结果如下：

    姓名:Person
    Person {name: "Person"}name: "Person"show: ()__proto__: Person
    I am window
    Window {top: Window, location: Location, document: document, window: Window, external: Object…})W\
    sharpxiajun
    Object {name: "sharpxiajun", job: "Software"}

## 立即调用的函数表达式（IIFE)

Immediately-Invoked Function Expression - http://benalman.com/news/2010/11/immediately-invoked-function-expression/#iife)
简单的理解就是定义完成函数之后立即执行。
比较常见的三种写法：
```javascript
// Crockford's preference - parens on the inside
(function() {
  console.log('Welcome to the Internet. Please follow me.');
}());

(function() {
  console.log('Welcome to the Internet. Please follow me.');
})();

!function() {
  console.log('Welcome to the Internet. Please follow me.');
}();
``` 
如果对于其返回值没有特别的要求的话，我们还可以这样写：
```javascript
!function(){}();  // => true
~function(){}(); // => -1
+function(){}(); // => NaN
-function(){}();  // => NaN
```

或者也可以这样：
```javascript
~(function(){})();
void function(){}();
true && function(){ /* code */ }();
15.0, function(){ /* code */ }();
```
甚至，还可以这样写：
```javascript
new function(){ /* code */ }
31.new function(){ /* code */ }() //如果没有参数，最后的()就不需要了
```
看看 Ben Alman 引用的 Dmitry A. Soshnikov 的例子如下：
```javascript
console.log( typeof f )
function f(){ console.log('called') }(1)
```

函数 f 的定义被提升了，但并没有被立即调用——后面一行相当于在花括号闭合后，加入分号断行了。

语法错误的两种原因：
1) function (){ }()
期望是立即调用一个匿名函数表达式，结果是进行了函数声明，函数声明必须要有标识符做为函数名称。

2) function g(){ }()
期望是立即调用一个具名函数表达式，结果是声明了函数 g。末尾的括号作为分组运算符，必须要提供表达式做为参数。

所以那些匿名函数附近使用括号或一些一元运算符的惯用法，就是来引导解析器，指明运算符附近是一个表达式。
按照这个理解，可以举出五类，超过十几种的让匿名函数表达式立即调用的写法：

```javascript
( function() {}() );
( function() {} )();
[ function() {}() ];

~ function() {}();
! function() {}();
+ function() {}();
- function() {}();

delete function() {}();
typeof function() {}();
void function() {}();
new function() {}();
new function() {};

var f = function() {}();

1, function() {}();
1 ^ function() {}();
1 > function() {}();
// ...
```

另外值得再次注意的是，括号的含混使用——它可以用来执行一个函数，还可以做为分组运算符来对表达式求值。
比如使用圆括号或方括号的话，可以在行首加一个分号，避免被用做函数执行或下标运算：
```javascript
g()
// 可能放了几行注释——不知道是用自动化脚本合并的文件，还是哪里拷的函数。
;( function() {}() )
```


## Closure 闭包
__A closure is a function having access to the parent scope, even after the parent function has closed.__

Example
```javascript
var add = (function () {
    var counter = 0;
    return function () {return counter += 1;}
})();

add();
add();
add();
// the counter is now 3
```

#### Example Explained
The variable add is assigned the return value of a self invoking function.

The self-invoking function only runs once. It sets the counter to zero (0), and returns a function expression.

This way add becomes a function. The "wonderful" part is that it can access the counter in the parent scope.

This is called a JavaScript closure. It makes it possible for a function to have "private" variables.

The counter is protected by the scope of the anonymous function, and can only be changed using the add function.


## new

《javascript高级编程》里对new操作符的解释：
new操作符会让构造函数产生如下变化：

    1.  创建一个新对象；
    2.  将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
    3.  执行构造函数中的代码（为这个新对象添加属性）；
    4.  返回新对象
    
第二点：一个简单识别this方式就是看方法调用前的对象是哪个this就指向哪个，其实这个过程还可以这么理解，在全局执行环境里window就是上下文对象，那么在obj里局部作用域通过obj来代表了。

第四点：记住构造函数被new操作，要让new正常作用最好不能在构造函数里写return，没有return的构造函数都是按上面四点执行，有了return情况就复杂了。


## this
在JavaScript中，this 的概念比较复杂。除了在面向对象编程中，this 还是随处可用的。这篇文章介绍了this 的工作原理，它会造成什么样的问题以及this 的相关例子。 要根据this 所在的位置来理解它，情况大概可以分为3种：

在函数中：this 通常是一个隐含的参数。
在函数外（顶级作用域中）：在浏览器中this 指的是全局对象；在Node.js中指的是模块(module)的导出(exports)。
传递到eval()中的字符串：如果eval()是被直接调用的，this 指的是当前对象；如果eval()是被间接调用的，this 就是指全局对象。
对这几个分类，我们做了相应的测试：

### 1. 在函数中的this

函数基本可以代表JS中所有可被调用的结构，所以这是也最常见的使用this 的场景，而函数又能被子分为下列三种角色：
* 实函数
* 构造器
* 方法

#### 在实函数中的this

在实函数中，this 的值是取决于它所处的上下文的模式。

1. Sloppy模式：this 指的是全局对象（在浏览器中就是window）。

    function sloppyFunc() {
        console.log(this === window); // true
    }
    sloppyFunc();

2. Strict模式：this 的值是undefined。

    function strictFunc() {
        'use strict';
        console.log(this === undefined); // true
    }
    strictFunc();
this 是函数的隐含参数，所以它的值总是相同的。不过你是可以通过使用call()或者apply()的方法显示地定义好this的值的。

    function func(arg1, arg2) {
        console.log(this); // 1
        console.log(arg1); // 2
        console.log(arg2); // 3
    }
    func.call(1, 2, 3); // (this, arg1, arg2)
    func.apply(1, [2, 3]); // (this, arrayWithArgs)
    
####  构造器中的this

你可以通过new 将一个函数当做一个构造器来使用。new 操作创建了一个新的对象，并将这个对象通过this 传入构造器中。

    var savedThis;
    function Constr() {
        savedThis = this;
    }
    var inst = new Constr();
    console.log(savedThis === inst); // true

JS中new 操作的实现原理大概如下面的代码所示（更准确的实现请看这里，这个实现也比较复杂一些）：

    function newOperator(Constr, arrayWithArgs) {
        var thisValue = Object.create(Constr.prototype);
        Constr.apply(thisValue, arrayWithArgs);
        return thisValue;
    }

####  方法中的this

在方法中this 的用法更倾向于传统的面向对象语言：this 指向的接收方，也就是包含有这个方法的对象。

    var obj = {
        method: function () {
            console.log(this === obj); // true
        }
    }
    obj.method();
 
#### 作用域中的this
 
在浏览器中，作用域就是全局作用域，this 指的就是这个全局对象（就像window）：

    <script>
        console.log(this === window); // true
    </script>
    
在Node.js中，你通常都是在module中执行函数的。因此，顶级作用域是个很特别的模块作用域（module scope）：

    // `global` (not `window`) refers to global object:
    console.log(Math === global.Math); // true
     
    // `this` doesn’t refer to the global object:
    console.log(this !== global); // true
    // `this` refers to a module’s exports:
    console.log(this === module.exports); // true
     
#### eval()中的this
 
eval()可以被直接（通过调用这个函数名’eval’）或者间接（通过别的方式调用，比如call()）地调用。要了解更多细节，请看这里。

    // Real functions
    function sloppyFunc() {
        console.log(eval('this') === window); // true
    }
    sloppyFunc();
     
    function strictFunc() {
        'use strict';
        console.log(eval('this') === undefined); // true
    }
    strictFunc();
     
    // Constructors
    var savedThis;
    function Constr() {
        savedThis = eval('this');
    }
    var inst = new Constr();
    console.log(savedThis === inst); // true
     
    // Methods
    var obj = {
        method: function () {
            console.log(eval('this') === obj); // true
        }
    }
    obj.method();

### 与this有关的陷阱

你要小心下面将介绍的3个和this 有关的陷阱。要注意，在下面的例子中，使用Strict模式(strict mode)都能提高代码的安全性。由于在实函数中，this 的值是undefined，当出现问题的时候，你会得到警告。

1. 忘记使用new

    如果你不是使用new来调用构造器，那其实你就是在使用一个实函数。因此this就不会是你预期的值。在Sloppy模式中，this 指向的就是window 而你将会创建全局变量：
        
        function Point(x, y) {
            this.x = x;
            this.y = y;
        }
        var p = Point(7, 5); // we forgot new!
        console.log(p === undefined); // true
         
        // Global variables have been created:
        console.log(x); // 7
        console.log(y); // 5
        不过如果使用的是strict模式，那你还是会得到警告（this===undefined）：
        
        function Point(x, y) {
            'use strict';
            this.x = x;
            this.y = y;
        }
        var p = Point(7, 5);
        // TypeError: Cannot set property 'x' of undefined
    
2. 不恰当地使用方法

    如果你直接取得一个方法的值（不是调用它），你就是把这个方法当做函数在用。当你要将一个方法当做一个参数传入一个函数或者一个调用方法中，你很可能会这么做。setTimeout()和注册事件句柄（event handlers）就是这种情况。我将会使用callIt()方法来模拟这个场景：
    
        
        /** Similar to setTimeout() and setImmediate() */
        function callIt(func) {
            func();
        }
        
    如果你是在Sloppy模式下将一个方法当做函数来调用，*this*指向的就是全局对象，所以之后创建的都会是全局的变量。
    
        var counter = {
            count: 0,
            // Sloppy-mode method
            inc: function () {
                this.count++;
            }
        }
        callIt(counter.inc);
         
        // Didn’t work:
        console.log(counter.count); // 0
         
        // Instead, a global variable has been created
        // (NaN is result of applying ++ to undefined):
        console.log(count);  // NaN
        
    如果你是在Strict模式下这么做的话，this是undefined的，你还是得不到想要的结果，不过至少你会得到一句警告：
        
        var counter = {
            count: 0,
            // Strict-mode method
            inc: function () {
                'use strict';
                this.count++;
            }
        }
        callIt(counter.inc);
         
        // TypeError: Cannot read property 'count' of undefined
        console.log(counter.count);
        
    要想得到预期的结果，可以使用bind()：
    
        var counter = {
            count: 0,
            inc: function () {
                this.count++;
            }
        }
        callIt(counter.inc.bind(counter));
        // It worked!
        console.log(counter.count); // 1
        
    bind()又创建了一个总是能将this的值设置为counter 的函数。

3. 隐藏this

    当你在方法中使用函数的时候，常常会忽略了函数是有自己的this 的。这个this 又有别于方法，因此你不能把这两个this 混在一起使用。具体的请看下面这段代码：
        
        var obj = {
            name: 'Jane',
            friends: [ 'Tarzan', 'Cheeta' ],
            loop: function () {
                'use strict';
                this.friends.forEach(
                    function (friend) {
                        console.log(this.name+' knows '+friend);
                    }
                );
            }
        };
        obj.loop();
        // TypeError: Cannot read property 'name' of undefined
        
    上面的例子里函数中的this.name 不能使用，因为函数的this 的值是undefined，这和方法loop()中的this 不一样。下面提供了三种思路来解决这个问题：
    
    * that=this，[Never Do This!] 将this 赋值到一个变量上，这样就把this 显性地表现出来了（除了that，self 也是个很常见的用于存放this的变量名），之后就使用那个变量：
        
            loop: function () {
                'use strict';
                var that = this;
                this.friends.forEach(function (friend) {
                    console.log(that.name+' knows '+friend);
                });
            }
        
    * bind()。使用bind()来创建一个函数，这个函数的this 总是存有你想要传递的值（下面这个例子中，方法的this）：
        
            loop: function () {
                'use strict';
                this.friends.forEach(function (friend) {
                    console.log(this.name+' knows '+friend);
                }.bind(this));
            }
        
    * 用forEach的第二个参数。forEach的第二个参数会被传入回调函数中，作为回调函数的this 来使用。
        
            loop: function () {
                'use strict';
                this.friends.forEach(function (friend) {
                    console.log(this.name+' knows '+friend);
                }, this);
            }

### 最佳实践

理论上，我认为实函数并没有属于自己的this，而上述的解决方案也是按照这个思想的。ECMAScript 6是用箭头函数(arrow function)来实现这个效果的，箭头函数就是没有自己的this 的函数。在这样的函数中你可以随便使用this，也不用担心有没有隐式的存在。

    loop: function () {
        'use strict';
        // The parameter of forEach() is an arrow function
        this.friends.forEach(friend => {
            // `this` is loop’s `this`
            console.log(this.name+' knows '+friend);
        });
    }
    
我不喜欢有些API把this 当做实函数的一个附加参数：

    beforeEach(function () {  
        this.addMatchers({  
            toBeInRange: function (start, end) {  
                ...
            }  
        });  
    });
    
把一个隐性参数写成显性地样子传入，代码会显得更好理解，而且这样和箭头函数的要求也很一致：

    beforeEach(api => {
        api.addMatchers({
            toBeInRange(start, end) {
                ...
            }
        });
    });


### 函数绑定(Function binding) —— 解决如何在另一个函数中保持this上下文

Archibald 关于缓存 this 的微博(twitter)：

Jake Archibald: “我会为了作用域做任何事情，但是我不会使用 that = this”

我对这个问题更清晰的认识是在我看到Sindre Sorhus更清楚的描述之后：

Sindre Sorhus：“在jQuery中使用$this，但是对于纯JS我不会，我会使用.bind()”

## 浏览器支持

Browser	Version support
Chrome	7
Firefox (Gecko)	4.0 (2)
Internet Explorer	9
Opera	11.60
Safari	5.1.4
正如你看到的，很不幸，Function.prototype.bind 在IE8及以下的版本中不被支持，所以如果你没有一个备用方案的话，可能在运行时会出现问题。

幸运的是，Mozilla Developer Network（很棒的资源库），为没有自身实现 .bind() 方法的浏览器提供了一个绝对可靠的替代方案：

    if (!Function.prototype.bind) {
      Function.prototype.bind = function (oThis) {
        if (typeof this !== "function") {
          // closest thing possible to the ECMAScript 5 internal IsCallable function
          throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
        }
 
        var aArgs = Array.prototype.slice.call(arguments, 1), 
            fToBind = this, 
            fNOP = function () {},
            fBound = function () {
              return fToBind.apply(this instanceof fNOP &amp;&amp; oThis
                                     ? this
                                     : oThis,
                                   aArgs.concat(Array.prototype.slice.call(arguments)));
            };
 
        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
 
        return fBound;
      };
    }
 
## 适用的模式

在学习技术点的时候，我发现有用的不仅仅在于彻底学习和理解概念，更在于看看在手头的工作中有没有适用它的地方，或者比较接近它的的东西。我希望，下面的某些例子能够适用于你的代码或者解决你正在面对的问题。

### CLICK HANDLERS（点击处理函数）
一个用途是记录点击事件（或者在点击之后执行一个操作），这可能需要我们在一个对象中存入一些信息，比如：

var logger = {
    x: 0,       
    updateCount: function(){
        this.x++;
        console.log(this.x);
    }
}
我们可能会以下面的方式来指定点击处理函数，随后调用 logger 对象中的 updateCount() 方法。


```javascript 
document.querySelector('button').addEventListener('click', function(){
    logger.updateCount();
});
```
但是我们必须要创建一个多余的匿名函数，来确保 updateCount()函数中的 this 关键字有正确的值。

我们可以使用如下更干净的方式：

document.querySelector('button').addEventListener('click', logger.updateCount.bind(logger));
我们巧妙地使用了方便的 .bind() 函数来创建一个新的函数，而将它的作用域绑定为 logger 对象。

SETTIMEOUT
如果你使用过模板引擎（比如Handlebars）或者尤其使用过某些MV*框架（从我的经验我只能谈论Backbone.js），那么你也许知道下面讨论的关于在渲染模板之后立即访问新的DOM节点时会遇到的问题。

假设我们想要实例化一个jQuery插件：

var myView = {
 
    template: '/* 一个包含 <select /> 的模板字符串*/',
 
    $el: $('#content'),
 
    afterRender: function () {
        this.$el.find('select').myPlugin();
    },
 
    render: function () {
        this.$el.html(this.template());
        this.afterRender();
    }
}
 
myView.render();
你或许发现它能正常工作——但并不是每次都行，因为里面存在着问题。这是一个竞争的问题：只有先到达的才能获胜。有时候是渲染先到，而有时候是插件的实例化先到。【译者注：如果渲染过程还没有完成（DOM Node还没有被添加到DOM树上），那么find(‘select’)将无法找到相应的节点来执行实例化。】

现在，或许并不被很多人知晓，我们可以使用基于 setTimeout() 的 slight hack来解决问题。

我们稍微改写一下我们的代码，就在DOM节点加载后再安全的实例化我们的jQuery插件：

    afterRender: function () {
        this.$el.find('select').myPlugin();
    },
 
    render: function () {
        this.$el.html(this.template());
        setTimeout(this.afterRender, 0);        
    }
 
然而，我们获得的是 函数 .afterRender() 不能找到 的错误信息。

我们接下来要做的，就是将.bind()使用到我们的代码中：

    afterRender: function () {
        this.$el.find('select').myPlugin();
    },
 
    render: function () {
        this.$el.html(this.template());
        setTimeout(this.afterRender.bind(this), 0);        
    }
 
现在，我们的 afterRender() 函数就能够在正确的上下文环境中执行了。

### 梳理基于 QUERYSELECTORALL的事件绑定
如今的DOM API引入了很多非常有用的方法，比如 querySelector, querySelectorAll 和 classList接口，这些方法给DOM API带来了非常显著的进步。

然而，迄今为止并没有一个真正的原生的为 NodeList 添加事件的方法。于是我们最终从 Array.prototype中剽窃了 forEach 方法来完成遍历，例如：

```javascript
Array.prototype.forEach.call(document.querySelectorAll('.klasses'), function(el){
    el.addEventListener('click', someFunction);
});
```

仍然，我们可以做的更好，通过使用我们的好朋友 .bind()。

```javascript
var unboundForEach = Array.prototype.forEach,
    forEach = Function.prototype.call.bind(unboundForEach);
 
forEach(document.querySelectorAll('.klasses'), function (el) {
    el.addEventListener('click', someFunction);
});
```

现在，我们拥有了一个简洁的遍历DOM节点的函数。

## 结论
正如你所看到的，.bind() 函数可以巧妙地运用于很多不同的用途，同时可以精简现有的代码。但愿这篇概述的内容，能够在你想在代码中使用.bind()（如果需要的话）时派上用场，并且帮助你更好地驾驭改变this值所带来的好处。

## Dom Selectors
getElementById,
getElementsByName,
getElementsByTagName
getElementsByClassName
querySelector
querySelectorAll


```javascript
document.querySelector("div.test>p:first-child");
document.querySelectorAll("div.test>p:first-child")[0];
```
querySelector 和 querySelectorAll 的区别在于 querySelector 用来获取一个元素，而querySelectorAll 可以获取多个元素。querySelector 将返回匹配到的第一个元素，如果没有匹配的元素则返回 Null。querySelectorAll 返回一个包含匹配到的元素的数组，如果没有匹配的元素则返回的数组为空。
```javascript
var emphasisText = document.querySelectorAll(".emphasis");
for( var i = 0 , j = emphasisText.length ; i < j ; i++ ){
    emphasisText[i].style.fontWeight = "bold";
}
```

这是原生方法，比起jquery速度快，缺点是IE6、7不支持。

W3C的规范与库中的实现

querySelector：return the first matching Element node within the node’s subtrees. If there is no such node, the method must return null .（返回指定元素节点的子树中匹配selector的集合中的第一个，如果没有匹配，返回null）

querySelectorAll：return a NodeList containing all of the matching Element nodes within the node’s subtrees, in document order. If there are no such nodes, the method must return an empty NodeList. （返回指定元素节点的子树中匹配selector的节点集合，采用的是深度优先预查找；如果没有匹配的，这个方法返回空集合）

这在BaseElement 为document的时候，没有什么问题，各浏览器的实现基本一致；但是，当BaseElement 为一个普通的dom Node的时候（支持这两个方法的dom Node ），浏览器的实现就有点奇怪了，举个例子：

```javascript
<div class= "test"   id= "testId" >
     <p><span>Test</span></p>
</div>
<script type= "text/javascript" >    
     var   testElement= document.getElementById( 'testId' ); 
     var   element = testElement.querySelector( '.test span' ); 
     var   elementList = document.querySelectorAll( '.test span' ); 
     console.log(element);  // <span>Test</span>
     console.log(elementList);  // 1   
</script>
```
按照W3C的来理解，这个例子应该返回：element：null；elementList：[];因为作为baseElement的 testElement里面根本没有符合selectors的匹配子节点；但浏览器却好像无视了baseElement，只在乎selectors，也就是说此时baseElement近乎document；这和我们的预期结果不合，也许随着浏览器的不断升级，这个问题会得到统一口径！

人的智慧总是无穷的，Andrew Dupont发明了一种方法暂时修正了这个怪问题，就是在selectors前面指定baseElement的id，限制匹配的范围；这个方法被广泛的应用在各大流行框架中；

Jquery的实现：

```javascript
var   oldContext = context,
old = context.getAttribute(  "id"   ),
nid = old || id,
try   {
    if   ( !relativeHierarchySelector || hasParent ) {
        return   makeArray( context.querySelectorAll(  "[id='"   + nid +  "'] "   + query ), extra );
    }   
} 
catch (pseudoError) {}
finally {
    if   ( !old ) {
        oldContext.removeAttribute(  "id"   );
    }
}
```
先不看这点代码中其他的地方，只看他如何实现这个方法的；这点代码是JQuery1.6的片段；当baseElement没有ID的时候，给他设置一个id = "__sizzle__”，然后再使用的时候加在selectors的前面，做到范围限制；context.querySelectorAll( "[id='" + nid + "'] " + query ；最后，因为这个ID本身不是baseElement应该有的，所以，还需要移除：oldContext.removeAttribute( "id" ); ，Mootools的实现：

```javascript
var   currentId = _context.getAttribute( 'id' ), slickid =  'slickid__' ;
_context.setAttribute( 'id' , slickid);
_expression =  '#'   + slickid +  ' '   + _expression;
context = _context.parentNode;
```
Mootools和Jquery类似：只不过slickid = 'slickid__'；其实意义是一样的；方法兼容性：FF3.5+/IE8+/Chrome 1+/opera 10+/Safari 3.2+;IE 8 ：不支持baseElement为object；

## jQuery Selector
jQuery内部使用Sizzle引擎，处理各种选择器。Sizzle引擎的选择顺序是从右到左.
