---
title: jQuery API的一些细节(一)--JQ核心、属性和CSS
date: 2016-05-01 23:11:37
tags: 
    - js
---

> 最近在学jQ的时候发现好多细节没有注意到。比如`each()`方法、`index()`方法。所以决定干脆仔仔细细把手册读一遍。本文的所有内容都是基于jQuery 1.11版本，如果以后学力有余再回来把兼容性的问题再补上。

<!-- more -->
## 一、核心

### jQuery核心函数
* `jQuery()` 即 `$()`
    * 传入CSS选择器时，选中匹配的元素
    * 传入第二个参数`[context]`时，代表在`context`中查找元素
    
        *在一个由AJAX返回的XML文档中，查找所有的div元素*
        ```javascript
        $("div", xml.responseXML);
        ```
    * 传入一个jQuery对象时，克隆传入的这个对象
    
        *克隆一个自定义对象*
        ```javascript
		function F (name) {
			this.name = name;
		}
		var f = new F('f');
		var a = $(f);
		a.name = 'x';
		console.info(f);    //F {name: "f"}
		console.info(a);    //[F, name: "x"]
        ```
    * 不传入参数时，返回一个空的jQuery对象
    * 传入HTML对象时，创建一个DOM元素的HTML字符串，作用类似`document.createElement()`
    
        *动态创建一个div元素（以及其中的内容），并将它追加到body元素中*
        ```javascript
        $("<div><p>Hello</p></div>").appendTo("body");
        ```
    * 传入HTML对象时，可以再传入一个json对象，表示要为这个对象添加上的属性、事件或方法（在实际测试中发现传入的这个HTML对象不能含有内容，包括HTML标签和文本。）
        
        *创建一个 `<input>` 元素，同时设定 type 属性、属性值，以及一些事件*
        ```javascript
        $("<input>", {
          type: "text",
          val: "Test",
          focusin: function() {
            $(this).addClass("active");
          },
          focusout: function() {
            $(this).removeClass("active");
          }
        }).appendTo("form");
        ```
* `$(function(){})` 即 `$(document).ready()`的简写
    * 和`window.onload` 的区别：
        * 一个页面中只能有一个`window.onload`，如果存在多个，程序只会执行最后一个
        * 一个页面中可以有多个`$(document).ready()`，程序会顺序执行所有的

### jQuery事件绑定
* `each()`
    * `each()`方法的作用是**以每一个匹配的元素作为上下文来执行一个函数**。这意味着可以使用`this`关键字来指代循环中的每一个元素(此处指代的是普通的JS对象而非jQuery对象，需要使用$(this))。
    * 可以使用`return`语句来提前跳出循环。
        * `return false`将会停止循环，作用类似于普通的循环中使用`break`
        * `return true`将会跳入下一循环，作用类似于普通循环中的`continue`
    * `each()`中使用`index()`方法有要特别注意的地方
* `index()`
    * `index()`方法可以搜索匹配的元素，并返回相应元素的索引值，从0开始计数
        
        *查找元素的索引值*      
        *html代码*
        ```html
        <ul>
            <li id="foo">foo</li>
            <li id="bar">bar</li>
            <li id="baz">baz</li>
        </ul>
        <ul>
            <li id="car">car</li>
        </ul>
        ```
        *jQuery代码*
        ```javascript
        $('#car').index()       //0；不传递参数，返回这个元素在同级HTML元素中的索引位置
        $('#car').index('li')   //3；传递一个选择器，返回这个元素在参数选中元素集合中的索引位置
        $('li').index(document.getElementById('car'))
                                //3；传递一个DOM对象，返回参数对象在这个集合中的位置，这个DEMO中返回#car在li中的位置
        $('li').index($('#bar')); //1
        $('li').index($('li:gt(0)')); //1；传递一组元素，返回这组元素中第一个元素在原先集合中的索引位置
        ```

### 数据缓存
* `data()`和`removeData()`
    * 用于在DOM元素上添加和移除名值对数据
    * 名值对在HTML上以自定义属性存储，这个自定义属性以`data-`开头

----------------

## 二、属性
* `attr()` 和 `removeAttr()`， `prop()` 和 `removeProp()`
    * 这两组方法的作用都是添加（获取）和删除被选元素的属性值
    * 值得特别注意的是，不同于其他大多数方法对集合的操作会对集合中每一个元素生效，用这种方式来获取属性值时，**只能获取到集合中第一个元素的属性值**（设置和删除时依旧是对集合操作）
    * `attr()`和`prop()`的区别
        * attr 和 prop都可以被翻译为属性，但为了以示区别，通常把attr翻译为属性，prop翻译为特性
        * 当用这两个方法获取元素的src属性时，HTML中是怎么写的`attr()`就会返回什么值，而`prop()`则会返回地址的经过计算后的完整路径，但是在设置src时这两个方法没有区别
        ```html
    	<img src="../images/15.jpg" alt="1" />
    	<img src="../images/16.jpg" alt="1" />
    	<img src="../images/17.jpg" alt="1" />
    	<img src="../images/18.jpg" alt="1" />
    	<img src="../images/19.jpg" alt="1" />
    	<script src="https://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>
    	<script>
    	console.info('prop:' + $('img').prop('src'));       //prop:file:///C:/Users/1/images/15.jpg
    	console.info('attr:' + $('img').attr('src'));       //attr:../images/15.jpg
    	</script>
        ```
        * 对于属性值为Boolean型的属性（比如input的checked属性）来说，attr只表示在HTML页面诞生时这个属性的状态，所以不能用`attr()`方法来动态的修改这些属性的值。而prop是随着checked属性变化而变化的，也就是说**在修改属性值为Boolean的属性时，一定要使用`prop()`方法**

### CSS类操作
* `addClass()`、`removeClass()`和`toggleClass()`
    * 这三个方法都用来操作DOM元素的类名
    * 可以稍微注意一下的是他们的参数都可以是一个函数，这个函数都会接受两个参数。第一个参数为对象在这个集合中的索引值(index),第二个参数时这个对象原先的class属性值，返回值都是一个字符串，其中保存了要添加或删除时的类名，以空格分隔
        *为每个li添加不一样的类名*
        ```javascript
        $('li').addClass(function(i){
            return 'list' + i;
        })
        ```
        *从最后一个元素上删除与前面元素重复的类名*
        ```javascript
    	$('li:last').removeClass(function() {
    	    var str = "";
    	    $('li').not(this).each(function () {
    	    	str += $(this).attr('class')
    	    	str += " "
    	    })
    	    return str;
    	});
        ```
        *根据情况决定要切换的类名（而不是在两个类名中切换）*
        ```javascript
        $('div.foo').toggleClass(function() {
            if ($(this).parent().is('.bar') {
                return 'happy';
            } else {
                return 'sad';
            }
        });
        ```

### HTML代码、文本、值
* `html()`、`text()`
    * 作用类似于`innerHTML`和`innerText`。在text()方法中传入的以html标签形式书写的字符串(如：`<div></div>`)会被原样打印
    * **不同于innerText，text()方法传入的内容不会添加到DOM元素的文本节点上，而是直接覆盖DOM元素的所有内容**

---------------------

## 三、CSS
### CSS
* `css()`
    * 在用JQ写动画的时候css()方法有时比animate()方法管用
    ```javascript
    setInterval(function(){
        $('div').css({
            width: oDiv.width() + 5
        })
    }, 30)
    ```

### 位置
**此条目下所有方法都只能获得第一个匹配元素的对应属性**
* `offset()`
    * 不要忘了这里的`offset()`是一个方法了，也就是说要获取left值需要写`offset().left`，也可以直接用这个表达式为元素赋位置值
* `position()`
    * 获取元素相对于定位父级元素的偏移
    * 仅对可见元素有效
* `height()` `width()`
    * 除了可以获取也可以用来设置，不需要用CSS()方法
* `innerHeight()`
    * 获取元素内部的区域高度（包括padding，不包括border）
* `outerHeight`
    * 包括border
    * 默认传入参数为false，不包括margin
    * 传入参数true时也包括margin