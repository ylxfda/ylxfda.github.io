---
layout: post
title: "用 Python 自动为文章生成内容哈希"
date: 2026-04-01 09:00:00 +0800
categories: tech
tags: [python, hashing, automation]
---

虽然 Git 提交已经为我们提供了时间戳证明，但如果你想要额外的一层验证，可以在每篇文章中嵌入内容哈希。

## 基本思路

对文章内容生成 SHA-256 哈希，并将其写入文章的元数据中。这样即使文章被转载到其他平台，读者也能通过哈希验证内容未被篡改。

```python
import hashlib
from pathlib import Path

def hash_post(filepath: str) -> str:
    content = Path(filepath).read_text(encoding='utf-8')
    # 只对正文部分做哈希（跳过 front matter）
    parts = content.split('---', 2)
    if len(parts) >= 3:
        body = parts[2].strip()
    else:
        body = content
    return hashlib.sha256(body.encode()).hexdigest()

# 使用示例
h = hash_post('_posts/2026-04-01-content-hashing.md')
print(f"Content SHA-256: {h[:16]}...")
```

## 数学验证

哈希碰撞的概率极低。对于 SHA-256：

$$P(\text{collision}) \approx \frac{n^2}{2^{257}}$$

当 $n = 10^{18}$（远超人类历史上所有文章总和）时，碰撞概率仍然可以忽略不计。
