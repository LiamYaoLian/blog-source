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
* $("S1, S2, SN"): or
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
* form
  - $("selector:input")
  - $("selector:text")
  - $("selector:password")
  - $("selector:submit")
  - $("selector:reset")
* attr(name|properties|key): 获取或设置元素属性
* removeAttr(name)
```
<!DOCTYPE html >

<html>
<head>
<meta charset="UTF-8">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>操作元素属性</title>

<style>
.myclass {
	font-style: italic;
	color: darkblue;
}
/* 高亮css类 */
.highlight {
	color: red;
	font-size: 30px;
	background: lightblue;
}
</style>

</head>

<body>
	<div class="section">
		<h2>jQuery选择器实验室</h2>
		<input style="height: 24px" id="txtSelector" />
		<button id="btnSelect" style="height: 30px">选择</button>
		<hr />
		<div>
			<p id="welcome">欢迎来到选择器实验室</p>
			<ul>
				<li>搜索引擎：<a href="http://www.baidu.com">百度</a> <span> <a
						style="color: darkgreen" href="http://www.so.com">360</a>
				</span>
				</li>
				<li>电子邮箱：<a href="http://mail.163.com">网易邮箱</a> <span> <a
						style="color: darkgreen" href="http://mail.qq.com">QQ邮箱</a>
				</span>
				</li>
				<li>中国名校：<a href="http://www.tsinghua.edu.cn">清华大学</a> <span>
						<a style="color: darkgreen" href="https://www.pku.edu.cn/">北京大学</a>
				</span>
				</li>
			</ul>

			<span class="myclass ">我是拥有myclass类的span标签</span>

			<p class="myclass">我是拥有myclass的p标签</p>
			<form id="info" action="#" method="get">
				<div>
					用户名：<input type="text" name="uname" value="admin" /> 密码：<input
						type="password" name="upsd" value="123456" />
				</div>
				<div>
					婚姻状况： <select id="marital_status">
						<option value="1">未婚</option>
						<option value="2">已婚</option>
						<option value="3">离异</option>
						<option value="4">丧偶</option>
					</select>
				</div>
				<div class="left clear-left">
					<input type="submit" value="提交" /> <input type="reset" value="重置" />
				</div>
			</form>
		</div>
	</div>
	<script type="text/javascript" src="js/jquery-3.3.1.js"></script>
	<script type="text/javascript">
		var href_attr =  $("a[href*='163']").attr("href");
		alert(href_attr);
		$("a[href*='163']").attr("href" , "http://www.163.com"); // set href to "http://www.163.com". all items selected will be set
		var attr = $("a").attr("href"); // if multiple items are selected and use =, will assign the first
		alert(attr);
		$("a").removeAttr("href"); // if multiple items are selected, will remove all
	</script>
</body>
</html>

```
* css(): 获取或设置匹配元素的样式属性
* addClass()
* removeClass()
```
<script type="text/javascript">
  $("a").css("color" , "red");
  $("a").css({"color" : "red" , "font-weight" : "bold" , "font-style" : "italic"});

  $("li").addClass("highlight myclass");
  $("p").removeClass("myclass");
  var color = $("a").css("color");
  alert(color);
</script>
```
* val()
  - get: `$(input[name='username']).val()`
  - set: `$(input[name='username']).val("admin")`
* text(): get or set text of element
* html(): get or set html of element
```
<script>
  $("input[name='uname']").val("administrator");
  var v = $("input[name='uname']").val();
  alert(v);
  //text与html方法最大的区别在于对于文本中的html标签是否进行转义
  //$("span.myclass").text("<b>锄禾日当午，汗滴禾下土</b>");
  $("span.myclass").html("<b>锄禾日当午，汗滴禾下土</b>");

  var vspan = $("span.myclass").text();
  alert(vspan);
</script>
```
* on("click" , function) - 为选中的页面元素绑定单击事件
* click(function) - 是绑定事件的简写形式
```
<script type="text/javascript">
	//onload是指在页面所有资源加载完成后执行
	window.onload = function(){
		//alert(1);
	}

	//ready()则是在页面dom被浏览器解释完成后执行
	$(document).ready(function(){
		alert("页面准备就绪");
	})

	//简化形式
	$(function(){
		$("p.myclass").on("click" , function(){
			//this: the element on which the event is called
      //in $(this), we are just passing this as a parameter to jQuery.fn.init()
			$(this).css("background-color" , "yellow");
		});

		$("span.myclass").click(function(){
			$(this).css("background-color" , "lightgreen");
		})

		$("input[name='uname']").keypress(function(event){
			console.log(event);
			if(event.keyCode == 32){
				$(this).css("color" , "red");
			}
		})
	})
</script>
```

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
  // false means async
  xmlhttp.open("GET","http://localhost/test?name=admin", true);
  //发送到服务器
  xmlhttp.send();
  ```
* deals with response
  ```
  xmlhttp.onreadystatechange = function()
  {
    //响应已被接收且服务器处理成功时才执行
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200)
    {
    //获取响应体的文本
    var responseText = xmlhttp.responseText;
    //对服务器结果进行处理
    ...
    }
  }
  ```

### Asynchronous vs Synchronous
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript" src="js/jquery-3.3.1.js"></script>
<script type="text/javascript">
	//设置ajax的全局默认值
	$.ajaxSetup({
		"dataType" : "json",
		"error" : function(xmlhttp){
			if(xmlhttp.status == "405"){
				alert("无效的请求方式");
			}
		}
	});
	$(function(){
		/*
			1. url
			2. 要发送的参数，可以是字符串也可以是json
			3. 响应处理函数
			4. dataType(可选) 服务器返回的数据类型(text|json|xml|html|script)

			$.ajax()
		*/
		$.get("/ajax/news_list" , {"t" : "tiobe"} , function(json){
			for(var i = 0 ; i < json.length ; i++){
				$("#container").append("<h1>" + json[i].title + "</h1>")
			}
		})
	})
</script>
</head>
<body>
	<div id="container"></div>
</body>
</html>
```

## jQuery supports Ajax
$().ajax(options)

* url 发送请求地址
* type: get | post
* data 向服务器传递的参数
* dataType: text | json | xml | html | jsonp| script
* success 接收响应时的处理函数
* error 请求失败时的处理函数
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript" src="js/jquery-3.3.1.js"></script>
<script type="text/javascript">
	//设置ajax的全局默认值
	$.ajaxSetup({
		"dataType" : "json",
		"error" : function(xmlhttp){
			if(xmlhttp.status == "405"){
				alert("无效的请求方式");
			}
		}
	});
	$(function(){
		/*
			1. url
			2. 要发送的参数，可以是字符串也可以是json
			3. 响应处理函数
			4. dataType(可选) 服务器返回的数据类型(text|json|xml|html|script)

			$.ajax()
		*/
		$.get("/ajax/news_list" , {"t" : "tiobe"} , function(json){
			for(var i = 0 ; i < json.length ; i++){
				$("#container").append("<h1>" + json[i].title + "</h1>")
			}
		})
	})
</script>
</head>
<body>
	<div id="container"></div>
</body>
</html>
```

```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript" src="js/jquery-3.3.1.js"></script>
<script type="text/javascript">
	$(function(){
		$.ajax({
			"url" : "/ajax/news_list",
			"type" : "get" ,
			"data" : {"t":"pypl" , "abc":"123" , "uu":"777"},
			//"data" : "t=pypl&abc=123&uu=777" ,
			"dataType" : "json" ,
			"success" : function(json){
				console.log(json);
				for(var i = 0 ; i < json.length ; i++){
					$("#container").append("<h1>" + json[i].title + "</h1>");
				}
			},
			"error" : function(xmlhttp , errorText){
				console.log(xmlhttp);
				console.log(errorText);
				if (xmlhttp.status == "405") {
					alert("无效的请求方式");
				} else if(xmlhttp.status == "404"){
					alert("未找到URL资源");
				} else if(xmlhttp.status == "500"){
					alert("服务器内部错误，请联系管理员");
				} else{
					alert("产生异常，请联系管理员");
				}
			}

		})
	})
</script>
</head>
<body>
	<div id="container"></div>
</body>
</html>

```
## Example 实现二级联动菜单
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript" src="js/jquery-3.3.1.js"></script>
<script type="text/javascript">
	$(function(){
		$.ajax({
			"url" : "/ajax/channel",
			"data" : {"level" : "1"},
			"type" : "get" ,
			"dataType" : "json" ,
			"success" : function(json){
				console.log(json);
				for(var i = 0 ; i < json.length ; i++){
					var ch = json[i];
					$("#lv1").append("<option value='" + ch.code + "'>" + ch.name + "</option>")
				}
			}
		})
	})

	$(function(){
		$("#lv1").on("change" , function(){
			var parent = $(this).val();
			console.log(parent);
			$.ajax({
				"url" : "/ajax/channel" ,
				"data" : {"level" : "2" , "parent" : parent},
				"dataType" : "json" ,
				"type" : "get" ,
				"success" : function(json){
					console.log(json);
					//移除所有lv2下的原始option选项
					$("#lv2>option").remove();
					for(var i = 0 ; i < json.length ; i++){
						var ch = json[i];
						$("#lv2").append("<option value='" + ch.code +"'>" + ch.name + "</option>")
					}
				}
			})
		})
	})

</script>
</head>
<body>
<select id="lv1" style="width:200px;height:30px">
	<option selected="selected">请选择</option>
</select>
<select id="lv2" style="width:200px;height:30px"></select>
</body>
</html>
```

## 从ajax方法衍生的简化方法
$.get() 发送Get方式Ajax请求
$.post() 发送Post方式Ajax请求
$.ajaxSetup() 设置Ajax全局默认值
