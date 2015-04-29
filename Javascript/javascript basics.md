
## Key
＊ 值传递
＊ 不允许堆内存操作
＊ 函数 是 对象

## 函数作用域

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
