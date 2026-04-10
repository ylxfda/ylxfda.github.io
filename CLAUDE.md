# CLAUDE.md — Front Row Future

## Git Identity

**Always verify before committing.** This repo must not leak personal info.

The local repo git config is set to:
- `user.name = ylxfda`
- `user.email = ylxfda@users.noreply.github.com`

Before any `git commit` or `git push`, confirm with:
```
git config user.name
git config user.email
```
If either returns a real name, hostname, or personal email, stop and fix with:
```
git config user.name "ylxfda"
git config user.email "ylxfda@users.noreply.github.com"
```

## Adding a New Post

### File naming
```
_posts/YYYY-MM-DD-slug-with-hyphens.md
```

### Frontmatter (required)
```yaml
---
layout: post
title: "Post Title Here"
date: YYYY-MM-DD
categories: [one category]
---
```

### Categories — pick exactly one
| Category | Use for |
|----------|---------|
| `research` | Paper analysis, technical deep-dives |
| `opinion` | Personal views, commentary, predictions |
| `ideas` | Original concepts, thought experiments |
| `tech` | Tools, engineering, how-things-work |

### Style guidelines
- Write in English (Chinese content in the body is fine)
- No frontmatter `tags` required, but can be added
- Keep the voice personal and direct — first person, honest about uncertainty
- Prefer concrete examples over abstract claims
- No need to add a summary or conclusion section if the ending lands naturally

### Linking papers or references
Use inline markdown links directly in the text:
```markdown
[*Paper Title*](https://arxiv.org/abs/XXXXXXX)
```
