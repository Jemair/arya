---
title: jQuery API的一些细节(二)--选择器、文档处理和筛选
date: 2016-05-12 10:36:12
tags: 
    - js
---

> 最近学习感觉自己都没什么动力，每天无所事事，忙着一些莫名其妙的事情。不要在最能吃苦的时候选择安逸。

<!-- more -->
## 四、选择器
### 基本选择器
* `#id` `.class` `,` `(HTML元素)`
* 可以将多个选择器用`, `分隔，一起返回。
### 层级选择器
* `(空格)` `>` `+` `~`
### 基本伪类选择器
* `:not()`
* `:eq()` `:gt()` `:lt()` 从0开始计数
* `:header` 匹配h1,h2,h3....
* `:animate` 匹配**正在**执行动画效果的元素
### 内容伪类选择器
* `:contains(text)` 查找含有某字符串的元素 **想要用HTML标签筛选需要用`:has()`**
* `:has(selector)` 查找含有匹配的HTML元素的HTML元素
* `:empty` 查找不包含子元素或文本的空元素
* `:parent` 查找含有子元素或文本的元素（**而不是查找父元素！！！**）
```html
    <div>123
        <li>123</li>            //只有这一行li被高亮
        <li></li>
    </div>
    <script>
        $('li:parent').css('background', '#f00');
    </script>
```
### 可见性伪类选择器
* `:hidden` `:visible`
### 属性选择器(正则的写法)
* `[attr]`
* `[att=value]`
* `[attr!=value]`   !   不含有指定属性
* `[attr^=value]`   ^   开头
* `[attr$=value]`   $   结尾
* `[attr*=value]`   *   包含
* `[selector1][selector2][selector3]`   复合属性选择器。选择符合所有条件的元素
### 子元素伪类选择器
* 索引值从0开始 first指的第0个
* `:only-child` `only-of-type`  唯一子元素。如果父级的子元素不唯一，则没有元素会被选中
### 表单
* 有了这个就不用写`[type='text']`这种坑爹写法了。
### 表单对象属性
* `:enabled` `:disabled` `:checked` `:selected`

----------------------------

## 五、文档处理
### 插入
* `append()` `prepend()` `after()` `before()`  被插对象在前，插入对象在后，我没有在说什么XE的东西
* `appendTo()` `perpendTo()` `insertAfter()` `insertBefore()`   插入对象在前，被插对象在后

### 包裹
* `wrap()` `wrapAll()` `wrapInner()`
```html
<div class="container">
    <p></p>
    <p></p>
</div>
<script>
    $('.container').wrap('<div class="wrap"></div>');
/*<div class="wrap">
    <div class="container">
        <p></p>
        <p></p>
    </div>
</div>*/
    $('.container').wrapAll('<div class="wrap"></div>');
/*<div class="container">
    <div class="wrap"><p></p></div>
    <div class="wrap"><p></p></div>
</div>*/
    $('.container').wrapInner('<div class="wrap"></div>');
/*<div class="container">
    <div class="wrap">
        <p></p>
        <p></p>
    </div>
</div>*/
</script>
```

### 替换
* `replaceAll()` `replaceWith()`
    * 这两个方法都是对**文档节点**的替换
    * 前者是用调用者替换参数，后者是用参数替换调用者
### 克隆
* `clone()`
    * 这个方法可以接受两个参数，两个参数都是Boolean。第一个参数控制是否复制元素上的事件处理函数，第二个参数控制是否深复制（是否克隆元素的子元素）
    * 两个参数的默认值都是`false`，实际测试中没发现第二个参数有什么卵用
## 六、筛选
> It takes a bit more code, but it's faster
> 在官方发布的教程中推荐使用这种方式选取元素
### 过滤
* `eq()`
    * `eq()`方法的参数可以为负数，当为负数时表示从最后一个元素开始倒数。（1算起）
* `filter()`
    * `filter`方法用于缩小匹配的范围。多个表达式用`,`分隔
* `map()`
    * 将一组元素转换为数组**有用！**
    *把`form`中每一个input元素的值建立一个列表*
```html
<p><b>Values: </b></p>
<form>
  <input type="text" name="name" value="John"/>
  <input type="text" name="password" value="password"/>
  <input type="text" name="url" value="http://ejohn.org/"/>
</form>
<script>
$("p").append( $("input").map(function(){
  return $(this).val();
}).get().join(", ") );  //这里的get()方法是必须的，它可以让你对一个DOM对象进行操作，而不是jQ对象
//result:   <p>John, password, http://ejohn.org/</p>

</script>
```
* `has()`
    * 保留包含特定后代的元素，去掉那些不含有指定后代的元素。
* `not()`
    * 去除与指定表达式匹配的元素
* `slice()`
    * 是的,jQ里可以直接对一个选择器使用`slice()`方法，具体使用方法和js无异。但是看起来就很厉害呀嘤嘤

### 查找
* `children()`  `find()` `content()`
    * `children()`只考虑子元素，而`find()`找到所有的后代元素
    * `content()`会找到匹配元素的所有子节点
    *查找所有文本节点，并将其加粗*

```html
<p>Hello <a href="http://ejohn.org/">John</a>, how are you doing?</p>
<script>
    $("p").contents().not("[nodeType=1]").wrap("<b/>");
<script>
```

* `closest()` `parent()` `parents()` `parentsUntil()`
    * 这三个方法都是在DOM树上上溯，找到集合中所有元素的父级节点
    * `closest()`是从DOM树上一直上溯，直到找到符合要求的元素
    * `parent()`只查找集合中元素的上一级节点，并返回符合要求的元素
    * `parents()`则查找集合中所有元素的所有父节点，并返回符合要求的元素
    * `parentsUntil()`会查找所有元素的所有祖先元素，直到找到和参数匹配的元素才会停止，这个jQ对象里包含所有找到的父元素，但**不包含那个选择器匹配到的元素**
* `next()` `nextAll()` `nextUntil()`/ `prev()` `prevAll()` `prevUntil()` / `siblings()`
    * 这几个方法都是在集合元素的同级查找元素
    * **值得特别注意的是所有的`Until`都不会包含传入参数所匹配的那个元素！**
    * 另外`Until`在不传入参数的情况下作用等同于`All`

### 串联
* `add()` 将于表达式匹配的元素添加的jQuery对象中，这个函数可以用于连接分别于两个表达式匹配的结果集
* `addBack()` 加回去。简单理解会将遍历匹配到的元素集合与之前一次遍历匹配到的元素集合拼接起来，并按文档中的顺序排序，例如
```javascript
$('ul').find('li').addBack()    //这个语句会同时选中'ul'和所有的'li'
```
* `end()`
    * 看起来很厉害的一个方法，作用是回到jQ元素的上一个“破坏性”的操作之前。
    * 所谓“破坏性”的操作，是指任何改变所匹配的jQuery元素的操作。具体包括 jQ遍历中的``add()``, `andSelf()`, `children()`, `filter()`, `find()`, `map()`, `next()`, `nextAll()`, `not()`, `parent()`, `parents()`, `prev()`, `prevAll()`, `siblings()` 和 `slice()`，还有文档处理中的`clone()`。
    * 例如

```javascript
$('p').find('span').end()   //这个语句最后会返回'p'标签
```
    