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

Alerts
----

Attention: This text is important.

Note! This is a note.

Hint: This is a hint.


index.html修改
---

解决markdown渲染后下划线变为em标签

11911 行` g += "<em>" + this.output(f[2] || f[1]) + "</em>";` 中的`<em>`标签改为`_`


