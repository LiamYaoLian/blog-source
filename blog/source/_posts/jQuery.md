---
title: jQuery
date: 2022-04-19 16:46:32
tags:
---

## Syntax
* jQuery(选择器表达式)
* $(选择器表达式)

* "*": all

* $("#id")
* $("标签")
* $(".class")
* $("S1, S2, SN"): 组合
* $("ancestor descendant"): 后代
* $("ancestor>descendant"): 子
* $("prev~siblings"): 兄弟
* $("selector[attribute=value]")
* $("selector[attribute^=value]"): 以某值开头
* $("selector[attribute$=value]"): 以某值结尾
* $("selector[attribute*=value]"): 包含某值
* $("selector:first")
* $("selector:last")
* $("selector:even")
* $("selector:odd")
* $("selector:eq(n)")
* $("selector:input")
* $("selector:text")
* $("selector:password")
* $("selector:submit")
* $("selector:reset")
* attr(name|properties|key): 获取或设置元素属性
* removeAttr(name)
* css(): 获取或设置匹配元素的样式属性
* addClass()
* removeClass()
* val() - 获取或设置输入项的值
* text() - 获取或设置元素的纯文本
* html() - 获取或设置元素内部的HTML
* on("click" , function) - 为选中的页面元素绑定单击事件
* click(function) - 是绑定事件的简写形式

## Event
### Mouse
* click   
* dblclick   
* mouseenter   
* mouseleave  
* mouseover
### Keyboard
* keypress
* keydown
* keyup
### Form
* submit
* change
* focus
### Document / Window
* load
* resize
* scroll
* unload

## Ready
* $(document).ready(function)
* $(function)

## Ajax
* Asynchronous JavaScript And XML
* allow to update some without refreshing
### Step
* create XmlHttpReqeust object
  ```
  var xmlhttp;
  if (window.XMLHttpRequest)
  {
  // IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
  xmlhttp=new XMLHttpRequest();
  }
  else
  {
  // IE6, IE5 浏览器执行代码
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
  ```
* send Ajax request
  ```
  //创建请求
  xmlhttp.open("GET","http://localhost/test?name=admin",true);
  //发送到服务器
  xmlhttp.send();
  ```
* deals with response
  ```
  xmlhttp.onreadystatechange=function()
  {
  //响应已被接收且服务器处理成功时才执行
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
  {
  //获取响应体的文本
  var responseText = xmlhttp.responseText;
  //对服务器结果进行处理
  ...
  }
  }
  ```


## jQuery supports Ajax
url 发送请求地址
type 请求类型
get | post
data 向服务器传递的参数
dataType 服务器响应的数据类型
text | json | xml | html | jsonp| script
success 接收响应时的处理函数
error 请求失败时的处理函数

## 从ajax方法衍生的简化方法
$.get() 发送Get方式Ajax请求
$.post() 发送Post方式Ajax请求
$.ajaxSetup() 设置Ajax全局默认值
