---
title: About
date: 2017-12-29 16:48:16
---
<img src="/images/cool.jpg" alt="a cool img" id="coolImg">
<style type="text/css">
	#coolImg {
		position: absolute;
		top: : 20px;
		right: 170px;
	}
</style>

## 关于我
&ensp;&ensp;男，99年生人，16年2月起培训，工作近年吧的Java从业者，这是一个非全技术的博客！

## 社交账号
* GitHub:	[zzhangffan](https://github.com/zzhangffan)
* QQ:	2300650579
* WeChat:	FANZ0503z
* Email:zzhangffan1@163.com | fan.zhang@powersi.com.cn(单位邮箱)
* 微博:	[用户张Z](https://weibo.com/6208723644)

## 关于站点
___[#RSS](/atom.xml)&ensp;&ensp;[#Sitemap](/sitemap.xml)___

<script type="text/javascript">
	var userAgent = navigator.userAgent;
	var img = document.getElementById("coolImg");
	if (/(?:Android|iPhone|SymbianOS|Windows Phone|iPad|iPod)/.test(userAgent)) {
		var Ele = document.createElement("img");
		Ele.src = "/images/cool.jpg";
		Ele.alt = "a cool img";
		var parentNode = img.parentNode;
		parentNode.appendChild(Ele);
		parentNode.removeChild(img);
		//alert("移动端浏览");
	} else {
		//alert("PC端浏览");
	}

	var docEle = document.documentElement;
	var clientHeight = docEle.clientHeight,
		clientWidth = docEle.clientWidth;

	const imgHeight = img.height;
	const imgWidth = img.width;
	var maxX = clientWidth - imgWidth,
		maxY = clientHeight - imgHeight;
	var isMouseDown = false, topDiff = 0, leftDiff = 0,
		intX = 0, intY = 0;
	function mousemove() {
		if (!isMouseDown)
			return;
		intX = getClientXOrY("X");
		intY = getClientXOrY("y");
		var finalX = intX - leftDiff;
		var finalY = intY - topDiff;
		if (finalX > maxX)
			finalX = maxX;
		if (finalY > maxY)
			finalY = maxY;
		img.style.left = finalX + "px";
		img.style.top = finalY + "px";
	}

	function mouseup() {
		isMouseDown = false;
	}

	function mousedown() {
		if (!isMouseDown)
			isMouseDown = true;
		intX = getClientXOrY("X");
		intY = getClientXOrY("y");
		leftDiff = intX - img.offsetLeft;
		topDiff = intY - img.offsetTop;
		return false;
	}

	img.onmousedown = mousedown;

	document.onmouseup = mouseup;
	document.onmousemove = mousemove;

	function getClientXOrY(xOrY) {
		var event = window.event;
		if ("X" == xOrY || "x" == xOrY)
			return event.clientX;
		else if ("Y" == xOrY || "y" == xOrY)
			return event.clientY;
	}

	window.onresize = function() {
		//重新获取窗口大小
		clientHeight = docEle.clientHeight,
		clientWidth = docEle.clientWidth;
		maxX = clientWidth - imgWidth;
		maxY = clientHeight - imgHeight;
	}
</script>