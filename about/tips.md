# TIPS

MATH
---

使用`$$`实现整行公式，使用`$`实现行内公式
```
$$ x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$
```

$$ x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

Hint: 为了实现矩阵换行，需要四个`\\\\`，暂时不清楚问题出在那个地方

```
$$\begin{bmatrix}
1 & 0 & 0 \\\\
0 & 1 & 0 \\\\
0 & 0 & 1 \\\\
\end{bmatrix}$$
```

Hint:调整mathjax中符号文字的大小
Use \tiny{ }, \scriptsize{ }, \small{ }, \normal{ }, \large{ }, \Large{ }, \LARGE{ }, \huge{ }, \Huge{ }.

Alerts
----

Attention: This text is important.

Note! This is a note.

Hint: This is a hint.


index.html修改
---

#### 修改1

解决markdown渲染后下划线变为em标签

11911 行` g += "<em>" + this.output(f[2] || f[1]) + "</em>";` 中的`<em>`标签改为`_`

#### 修改2

为了实现md中的date和tag能够到h1标签的同一行，添加如下函数，同时在`function c`中调用

md中的date和tag：

```
date: 2018-07-03 15:41:10
tags: [机器学习,统计学]
```

添加的js：

```
function __tagDate() {
	var re = /(date.*)\<br\>(tags.*\[.*\])/;
	var tagDate = a('.md-text')[0];
	if(!tagDate){
	return false;
	}
	var tagDateText = tagDate.innerHTML;
	var _match = tagDateText.match(re);
	if (_match){
	var _date = _match[1],
			_tag = _match[2];
	var _html = `
	<p class="md-tag-date">
		<div class="md-text text-right" style="position: absolute; bottom: 10px; right: 0; font-size: 12px; font-weight: normal;color: #666;">
			<span class="tag">${_tag}</span>
			&nbsp;&nbsp;&nbsp;&nbsp;
			<span class="date">${_date}</span>
		</div>
	</p>`;
	var ele = a(_html);
	ele.appendTo("#md-title");
	tagDate.remove();
}
}
```




