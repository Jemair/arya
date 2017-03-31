title: CSS常用竖直居中方法总结
date: 2016-12-17 14:24:18
tags:
	- 学习笔记
---

> 竖直居中一直是前端页面中的一个大麻烦。今天把我所掌握的竖直居中方法整理一遍，一方面是做个梳理，另一方面也作为技能的巩固吧~

<!-- more -->

## 一. 单行文本的竖直居中
在容器上设置一个与容器等高的`line-height`值即可，
```html
<div class="middle">Lorem ipsum dolor sit amet, consectetur adipisicing elit.</div>
<style>
.middle{
    height: 40px;
    line-height: 40px;
    overflow: hidden;
}
</style>
```
优点：
1. 操作简便
2. 兼容性好
缺点：
1. 无法让图片居中
2. 无法让多行文本居中
3. 无法让整个容器居中
注意：**一定不要忘记给你的容器加上`overflow: hidden`属性，否则当文字过多超出一行时会非常难看**。
稍微复杂一点的写法可以为你的容器加上以下三句话：
* `overflow: hidden;`
* `white-space: no-wrap;`
* `text-overflow: ellipsis;`
这三句话可以用省略号替代掉超出容器边界的文字。

## 二、利用表格单元的特性将容器居中
这应该算是使用最多的垂直居中方式之一了，
原理是将需要居中的元素转化为表格组件，就可以使用表格的`vertical-align`属性居中了
```html
<div class="outter">
    <div class="inner">
         <img src="http://og08ttv6v.bkt.clouddn.com/user-icon.jpg?imageView2/3/w/200/q/90" />
    </div>
</div>
<style>
.outter{
    display: table;
    height: 500px;
}
.inner{
    display: table-cell;
    vertical-align: middle;
}
</style>
```
优点：
1. 效果出色，不论是多行文本还是图片都能很好的居中
2. 操作简便，没有用到特殊的语法
缺点：
1. 这全是CSS1属性！！IE8你凭什么不兼容！！！凭！！什！！么！！？？？
2. 外层容器高度需要固定

## 三、利用绝对定位和负margin
这也是常见的垂直居中方式之一，大多数情况下都可以满足需求
```html
<div class="outter">
     <div class="inner"></div>
 </div>
<style>
 .outter{
    position: relative;
    height: 500px;
 }
 .inner{
    position: absolute;
    top: 50%;
    width: 100%;
    height: 200px;
    margin-top: -100px;
 }
</style>
```
优点：
1. 效果拔群
2. 还算比较好写
3. 兼容性很好啊
缺点：
1. 需要固定内层容器高度，如果内层是经常需要替换的图片的话，扩展性就不是很好啦...
2. 用到了绝对定位，在页面中还是尽量少用绝对定位的好

## 四、绝对定位 + CSS3 2D平移
在不考虑兼容性的前提下，这应该算是常用的写法中最好用的写法了
```html
<div class="outter">
    <div class="inner"><div>
</div>
<style>
.outter{
    position: relative;
    width: 500px;
    height: 500px;
}
.inner{
    position: absolute;
    top: 50%;
    width: 100%;
    height: 200px;
    -webkit-transform: translateY(-50%);
    -ms-transform: translateY(-50%);
    -o-transform: translateY(-50%);
    transform: translateY(-50%);
}
</style>
```
优点：
1. 效果很好
2. 内外层容器均不用固定高度
3. 算是比较直观，没有难以理解的地方
缺点：
1. 很明显，不兼容IE8
2. 一大摞HACK的写法很复杂。不过好在用EMMET的写法可以直接写<kbd>trfty</kbd>，再按<kbd>tab</kbd>

## 五、纯绝对定位
让定位上下左右全为0
```html
<div class="outter">
   <div class="inner"></div>
</div>
<style>
.outter{
    position: relative;
    height: 500px;
}
.inner{
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
    height: 200px;
}
</style>
```
优点：
1. 兼容性好
2. 可以直接同时用于水平、垂直居中
缺点：
1. 用到了定位
2. 内部元素必须确定高度
注意：不要忘记了`margin: auto`

## 六、流式盒模型 -- 居中一切
属于未来的写法，非常简单的代码即可居中一切
```html
<div class="outter">
    <div class="inner"></div>
</div>
<style>
.outter{
    display: -webkit-flex;
    display: -moz-flex;
    display: -ms-flex;
    display: -o-flex;
    display: flex;
    -ms-align-items: center;
    align-items: center;
    width: 500px;
    height: 500px;
    background: #f00;
}
.inner{
    width: 500px;
    height: 200px;
    background: #ff0;
}
</style>
```
优点：
1. 方便、超级方便的
2. 再加一句`justify-content: center`即可实现水平居中
2. 流式盒模型属于未来
缺点：
1. HACK的写法并不是必须的，理论上来说在现代的浏览器中已经不用使用这种兼容性的写法了
2. 然而对于IE8来说必然是不能兼容的

## 七、利用列表标识符
算是一种比较奇怪的方法，不过还比较好写，可以了解一下
```html
<div class="outter">
    <ul class="inner">
        <li></li>
    </ul>
</div>
<style>
.outter{
    height: 500px;
}
.inner{
    line-height: 500px;
    margin-left: 50%;
}
.inner li{
    list-style-image: url(http://og08ttv6v.bkt.clouddn.com/user-icon.jpg?imageView2/3/w/200/q/90);
}
</style>
```
优点：
1. 写法简便
2. 兼容性好
缺点：
1. 只能用于图片居中
2. 语义化不强
3. 容易对其他元素产生影响

## 八、利用伪类
比较少见的方法，可是很好用
```html
<div class="outter">
    <div class="inner"></div>
</div>
<style>
.outter{
    height: 500px;
}
.outter::before{
    content: '';
    display: inline-block;
    height: 100%;
    vertical-align: middle;
}
.inner{
    display: inline-block;
    vertical-align: middle;
}
</style>
```
优点：
1. 内外层元素均不需要固定高度
2. 看起来比较高端（如果这个算的话23333）
缺点：
1. 写法复杂
2. IE8不能使用