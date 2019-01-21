### 关于iframe
其实在最开始学习html时就接触过iframe，那时候觉得这是一个low到爆的东西，用iframe远远不如使用div和css自定义布局来得优雅和自由。现在看来那时候的想法还是太天真了，iframe还是有不少地方有特殊的用途的。具体哪些地方我也不想研究，这里就主要说一下用到的一个地方：使用iframe全屏加载页面以使后续页面能够自动播放音乐。

### iframe的使用
W3c iframe：http://www.w3school.com.cn/tags/tag_iframe.asp

```html
<iframe class="full" src="./html/login.html" id="iframePage" onload="changeFrameHeight()" frameborder="0" scrolling="no" marginwidth="0" marginheight="0"></iframe>
```

其中，比较重要的是full，这个其实就是设置iframe的宽度，我定义的是100%，src是指显示的需要显示的页面，具体的一些属性可以参考src。

onload则是重新定义iframe的宽度，这样可以让iframe能够随着页面的高度的变化而发生变化。

```javascript
function changeFrameHeight(){
	var ifm = document.getElementById("iframePage");
	ifm.height = document.documentElement.clientHeight;
}
```
用起来就这么简单，顺便把实现全屏的js也放一下

```javascript
function requestFullScreen(element) {
	var element=document.documentElement;
	var requestMethod = element.requestFullScreen || //W3C
	element.webkitRequestFullScreen || //Chrome等
	element.mozRequestFullScreen || //FireFox
	element.msRequestFullScreen; //IE11
	if (requestMethod) {
		requestMethod.call(element);
	}
	else if (typeof window.ActiveXObject !== "undefined") {//for Internet Explorer
		var wscript = new ActiveXObject("WScript.Shell");
		if (wscript !== null) {
		 wscript.SendKeys("{F11}");
		}
	}
}
```
