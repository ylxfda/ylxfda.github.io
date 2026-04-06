---
layout: post
title: "功能演示：图片、视频、公式与代码"
date: 2026-04-03 10:00:00 +0800
categories: ideas
tags: [demo, tutorial]
---

这篇文章演示了博客支持的各种内容格式。

## 插入图片

使用标准 Markdown 语法：

```markdown
![图片描述](图片路径或URL)
```

你可以把图片放在项目的 `images/` 文件夹中，也可以使用外部链接。

## 嵌入 YouTube 视频

使用 HTML 嵌入：

<div class="video-container">
  <iframe src="https://www.youtube.com/embed/dQw4w9WgXcQ" frameborder="0" allowfullscreen></iframe>
</div>

代码如下：
```html
<div class="video-container">
  <iframe src="https://www.youtube.com/embed/VIDEO_ID" 
    frameborder="0" allowfullscreen></iframe>
</div>
```

## 数学公式

行内公式：质能方程 $E = mc^2$ 是物理学中最著名的公式之一。

独立公式（使用 LaTeX 语法）：

$$\nabla \times \mathbf{E} = -\frac{\partial \mathbf{B}}{\partial t}$$

更复杂的公式：

$$\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}$$

薛定谔方程：

$$i\hbar\frac{\partial}{\partial t}\Psi(\mathbf{r},t) = \hat{H}\Psi(\mathbf{r},t)$$

## 代码高亮

Python 示例：

```python
import hashlib
import datetime

def create_thought_hash(content: str) -> str:
    """为你的想法创建一个不可变的哈希"""
    timestamp = datetime.datetime.now().isoformat()
    data = f"{timestamp}:{content}"
    return hashlib.sha256(data.encode()).hexdigest()

idea = "量子计算将彻底改变密码学"
print(f"Hash: {create_thought_hash(idea)}")
```

## 引用块

> 真正的问题不是计算机能否思考，
> 而是人类是否在思考。
> —— B.F. Skinner

## 表格

| 特性 | 支持状态 | 说明 |
|------|---------|------|
| Markdown | ✅ | 基础写作格式 |
| 数学公式 | ✅ | MathJax / LaTeX |
| 代码高亮 | ✅ | 多语言支持 |
| YouTube | ✅ | iframe 嵌入 |
| 图片 | ✅ | 本地或外部链接 |
| GIF 动画 | ✅ | 与图片相同语法 |

## 链接

你可以随时插入 [外部链接](https://github.com)，Markdown 会自动渲染。
