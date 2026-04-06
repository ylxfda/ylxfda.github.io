# 思想存档 | Thought Archive

一个暗色主题的个人博客，使用 GitHub Pages + Jekyll 搭建。每篇文章通过 Git 提交获得不可篡改的时间戳记录。

## 🚀 快速开始

### 1. Fork 或克隆本仓库

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_USERNAME.github.io.git
cd YOUR_USERNAME.github.io
```

### 2. 替换个人信息

打开 `_config.yml`，修改以下字段：
- `title`: 你的博客标题
- `description`: 博客描述
- `author`: 你的名字

全局搜索 `YOUR_USERNAME` 和 `your.email@example.com`，替换为你自己的信息。

### 3. 发布你的第一篇文章

在 `_posts/` 文件夹中创建文件，文件名格式：`YYYY-MM-DD-title.md`

```markdown
---
layout: post
title: "文章标题"
date: 2026-04-03 14:30:00 +0800
categories: ideas
tags: [tag1, tag2]
---

你的文章内容...
```

### 4. 推送到 GitHub

```bash
git add .
git commit -m "发布：文章标题"
git push origin main
```

几分钟后访问 `https://YOUR_USERNAME.github.io` 即可看到你的博客。

## 📁 目录结构

```
├── _config.yml          # 站点配置
├── _layouts/
│   ├── default.html     # 基础布局（含所有样式）
│   └── post.html        # 文章布局
├── _posts/              # 所有文章放这里
│   └── 2026-04-03-example.md
├── index.html           # 首页
├── categories.html      # 分类页
├── archive.html         # 归档页
├── about.md             # 关于页
├── images/              # 图片文件夹
└── Gemfile              # Ruby 依赖
```

## ✍️ 写作指南

### 文章分类

在 front matter 中设置 `categories`，预设了四种颜色主题：
- `ideas` — 蓝色
- `tech` — 绿色
- `research` — 紫色
- `opinion` — 橙色

### 插入图片

```markdown
![描述](images/photo.jpg)
```

### 嵌入 YouTube 视频

```html
<div class="video-container">
  <iframe src="https://www.youtube.com/embed/VIDEO_ID" 
    frameborder="0" allowfullscreen></iframe>
</div>
```

### 数学公式

行内公式用单个 `$`：`$E = mc^2$`

独立公式用双 `$$`：

```
$$\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}$$
```

### 代码块

用三个反引号加语言名：

````
```python
print("Hello World")
```
````

## 🔐 时间戳验证

每次 `git commit` 都会生成一条包含以下信息的记录：
- **SHA 哈希**: 基于内容计算的唯一标识符
- **时间戳**: 精确的提交时间
- **作者信息**: 你的 Git 用户名和邮箱

任何人都可以通过 `git log` 或访问 GitHub 的 commits 页面来验证。

## 📝 本地预览（可选）

如果想在发布前本地预览：

```bash
# 安装 Ruby（如果没有的话）
# macOS: brew install ruby
# Ubuntu: sudo apt install ruby ruby-dev

gem install bundler
bundle install
bundle exec jekyll serve
```

然后打开 http://localhost:4000 预览。

不想本地预览的话，直接 push 到 GitHub 也完全可以，几分钟后就能在线看到效果。
