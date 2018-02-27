---
title: About
date: 2017-12-29 16:48:16
---
<img src="/images/cool.jpg" alt="aImg" id="coolImg">
<style type="text/css">
	#coolImg {
		position: absolute;
		top: : 20px;
		right: 170px;
	}
</style>

## 关于我
&ensp;&ensp;男，99年生人，现居湖南长沙！

## 联系我
* GitHub:	[zzhangffan](https://github.com/zzhangffan)
* Email:zzhangffan1@163.com

## 站点信息
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
		alert("移动端浏览");
	} else {
		alert("PC端浏览");
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