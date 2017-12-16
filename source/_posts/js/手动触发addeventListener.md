---
title: js手动触发addEventListener
tags: [javascript]
---

# js 中 document.createEvent的用法-转载

js 中 document.createEvent的用法

<a class="comment-mod" onclick="alert('ss')" href="#">评论</a>

如果用户直接查看文章列表，那么所有的评论以及评论框都是不显示的，但是如果用户通过别的页面比如首页的个人动态直接定位到这篇日志，那么评论就应该全部显示。而列表页和查看单个条目的页面是同一个页面，这就要求我判断一下用户是否定位到该篇日志，如果是，就通过JS来触发 A 标签的点击事件。

一开始我尝试了一些方法，想当然地以为 A 标签和按钮一样是有 onclick() 事件的，结果发现没有，后来从网上搜了一些资料之后，成功解决了这个问题^_^ 。解决办法是针对 IE 和 FF编写不同的逻辑，部分代码如下：
<script>
 var comment = document.getElementsByTagName('a')[0];
 
if (document.all) {
 // For IE

 comment.click();

} else if (document.createEvent) {
   //FOR DOM2
var ev = document.createEvent('HTMLEvents');

 ev.initEvent('click', false, true);
 comment.dispatchEvent(ev);
} 
</script>

   

语法：

createEvent(eventType)

参数

描述

eventType

想获取的 Event 对象的事件模块名。

关于有效的事件类型列表，请参阅"说明"部分。

返回值

返回新创建的 Event 对象，具有指定的类型。

抛出

如果实现支持需要的事件类型，该方法将抛出代码为 NOT_SUPPORTED_ERR 的 DOMException 异常。

说明

该方法将创建一种新的事件类型，该类型由参数 eventType 指定。注意，该参数的值不是要创建的事件接口的名称，而是定义那个接口的 DOM 模块的名称。

下表列出了 eventType 的合法值和每个值创建的事件接口：

参数

事件接口

初始化方法

HTMLEvents

HTMLEvent

iniEvent()

MouseEvents

MouseEvent

iniMouseEvent()

UIEvents

UIEvent

iniUIEvent()

用该方法创建了 Event 对象以后，必须用上表中所示的初始化方法初始化对象。关于初始化方法的详细信息，请参阅 Event 对象参考。

该方法实际上不是由 Document 接口定义的，而是由 DocumentEvent 接口定义的。如果一个实现支持 Event 模块，那么 Document 对象就会实现 DocumentEvent 接口并支持该方法。