---
title: JS中this的快速总结
date: 2016-11-16 23:23:16
tags:
	- js
---

> JS的坑真的是数不胜数，我一直想写一篇笔记来好好记录一下`this`这一巨坑的使用方法，正好在[掘金](http://gold.xitu.io/)翻到了这篇文章[JavaScript中的this陷阱的最全收集--没有之一](https://segmentfault.com/a/1190000002640298#articleHeader3)。自己跟着代码敲了一遍之后结合自己平时的实践写下了这篇心得，收获颇深。

<!-- more -->
在JS中使用`this`，大致分为以下几种情况：

## 一、在全局范围下使用`this`
在全局范围下，`this`永远指向的是JS代码中的`global`对象，即在浏览器中，`this`指向的就是`window`对象

## 二、在调用一个方法时使用`this`
在一个方法（函数）中，`this`指代的是包含`this`的这个方法所属的对象。换句话说，包含`this`的这个函数是谁的方法，谁就是`this`所指代的对象。
比如说在这个例子中
```javascript
var obj.fn = function(){console.info(this)};
obj.fn();
```
这里的`this`指代的就是`obj`这个对象。
值得注意的是，在浏览器下，`window`对象在调用函数时是可以被省略的，也就是说，在**任何情况下**（无论当前函数的作用域是否包含在其他函数中），只要使用的函数未声明调用者，`this`都指向`window`对象，来看下面这个例子：
```javascript
var timer = setInterval(function (){
    console.info(this)              //window
}, 30)
```
BTW: 在严格模式下，对于没有指定调用者的函数中的`this`会显示为`undefined`
BTW2: 如果在函数内部想要改变`this`的指向，可以有两种方法：一种是将`this`作为参数传递给函数。事实上`window`对象的`alert()`方法、`console`对象的`log()`方法都是将`this`作为参数传递到方法中，所以在这些方法中`this`的指向也就不存在问题了；第二种方式是在`this`指向正常的时候使用一个变量将`this`保存起来，这种方法适用于函数多重嵌套的情形，比如以下的例子：
```javascript
var oDiv1 = document.getElementById('div1');    //这里使用DOM元素只是因为最为常见，在其他对象中情况也是相似的
var oDiv2 = document.getElementById('div2');
oDiv1.onmouseenter = function (){
    console.info(this);         //oDiv  此处非重点，详情见DOM EVENT中的this一节
    var that = this;
    oDiv2.onclick = function (){
        console.info(this)      //oDiv2 此处this的指向改变，使用this无法访问到oDiv1了
        console.info(that)      //声明that变量将上一个this的值保存起来后，就可以通过that访问到oDiv1了
    }
}
```
当然，还有更特殊的情况，是在方法无法接受参数的情况下，如果要使用调用父级作用域中`this`所指向的值的话就更麻烦了，需要使用到闭包。最常见的情况就是在定时器中调用这个定时器所处的作用域，再举个栗子：
```javascript
var oDiv1 = document.getElementById('div1');
oDiv1.onclick = function (){
    var that = this;        //此处将this作为值保存在了that中
//setInterval()方法只能接受一个不带参数的函数作为参数。具体原因此处不表，可以搜索JS调用函数时加括号和不加括号的区别
    var timer = setInterval(function (that){        
    return function (){ //此处将这个不带参数的匿名函数作为返回值返回，这是一个最简单的闭包实现
       console.info(that)   //oDiv1 
    }
}, 30)
}
```

## 三、当用`new`操作符调用函数时使用`this`
所谓用`new`操作符调用函数，讲人话，就是`new`一个对象时，对象的构造函数里的this。毫无疑问，这时的`this`指向了这个新创建出来的对象。
```javascript
function Foo (name){
    this.name = name;
    console.info(this);     //Foo {name: "xiaoming"}
}
var obj = new Foo('xiaoming');
```
同样的，如果是在一个构造函数中嵌套了一个子作用域，如果要找到父作用域中的`this`，也可以用一个变量将`this`保存起来
```javascript
function Foo (name, obj){
    this.name = name;
    this.obj = obj;
    var _that = this;        //记得这里一定要使用var操作符声明变量，否则你将生成一个全局变量
    //注意这里不能将this作为这个对象的属性来保存(this.that = this;)，这样会使你在下面的代码中陷入
    //我要找对象 -> 我要找到对象的this属性 -> 我要找对象  的死循环....
    this.obj.onclick = function (){
        console.info(this)      //oDiv
        console.info(that)      //Foo {name: "xiaoming", obj: div#div1}
    }
}
var oDiv = document.getElementById('div1')
var obj = new Foo('xiaoming', oDiv);
```

## 四、在原型中使用`this`
在原型中我们所做的事情无非两类。一是定义原型的方法，而是定义原型共有的属性。
1. 在定义方法时，`this`的指向和构造函数中一样，都是指向了新创建的这个实例。
2. 而在定义属性时，`this`则指向了`window`对象（和前面一样，在严格模式下这里会返回`undefined`）。那么在需要在原型中定义属性时使用`this`的时候，需要先在构造函数里明确返回新实例，然后再在原型里通过这个方法获取到属性（感觉有这种奇葩需求的地方少之又少，干脆权当奇技淫巧记住好了）
```javascript
function Thing(name) {
    this.name = name;
    return this;
}
Thing.prototype.sayThat = function () {
	console.info(this);     //在原型定义的函数中，this如我们所愿的指向了新创建的这个对象
}
Thing.prototype.name2 = Thing().name;
var thing = new Thing('xiaoming');
thing.sayThat();
console.info(thing.name2);      //这里各种hack写法...已然无语
```

## 五、对象字面量中使用`this`
在一个对象字面量里，你可以直接通过`this`来引用这个对象的其他属性。这和用`new`来新建一个实例是不一样的
```javascript
var obj = {
    foo: 'bar',
    logFoo: function (){
        console.info(this.foo)      //'bar'
    }
}
```
当你需要越过当前对象去访问父级对象上的属性的时候，直接使用父级对象的名字就好了
```javascript
var obj = {
    foo: 'bar',
    deeper: {
        logFoo: function (){
            console.info(obj.foo);  //'bar'
        }
    }
}
```

## 六、DOM Event中使用`this`
* 我们已经知道，在一个HTML DOM事件处理程序里面，this始终指向这个处理程序被所绑定到的HTML DOM节点
```javascript
var oDiv1 = document.getElementById('div1');
oDiv1.onclick = function (){
    console.info(this);         //oDiv1
}
```
* 除非你自己通过`bind()`切换了上下文
```javascript
var oDiv1 = document.getElementById('div1');
var oDiv2 = document.getElementById('div2');
function handleClick (){
    console.info(this);
}
oDiv1.onclick = handleClick.bind(oDiv2);    //oDiv2
```

## 七、其他的一些关于`this`的小知识
1. 你不能重写`this`，因为它是系统的保留字
2. 在jQuery中`this`能发挥更大的威力，因为jQ中的数组遍历方法改变了`this`的指向，使它指向了当前处于循环中的元素
```javascript
$('li').forEach{
    function (){
        console.info($('li').index(this));      //这里会顺序返回每一个'li'元素的索引
    }
}
```

写完了。以上。