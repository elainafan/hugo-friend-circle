# Hugo Friend Circle

一个基于 RSS / Atom、HTML fallback 和 GitHub Actions 的 Hugo 友链朋友圈。它不需要后端、不需要数据库，只会定时抓取 `data/friends.yaml` 中的友链动态，并生成 Hugo 可直接读取的 `data/friend_posts.json`。

## Features

- RSS / Atom 抓取
- 未暴露 RSS 时，自动尝试解析首页 / 归档页中的日期文章链接
- GitHub Actions 定时更新
- 按发布时间倒序展示
- 支持朋友头像、文章标题、发布时间
- 支持搜索、按朋友筛选、Load More 分批展开
- feed 抓取失败时只记录 warning，不阻塞构建
- 支持最大文章数、单站点文章数、起始日期、超时和重试配置
- 页面样式适配 Hugo Stack，也可以作为普通 Hugo 页面运行

## Quick Start

1. 编辑 `data/friends.yaml`：

```yaml
settings:
  max_posts: 240
  max_posts_per_friend: 24
  max_days: 180
  since: 2024-01-01
  timeout: 8
  retries: 2

friends:
  - name: Example Friend
    site: https://example.com/
    feed: https://example.com/index.xml
    avatar: https://example.com/avatar.png
    description: A friend with RSS.

  - name: HTML Only Friend
    site: https://example.org/
    avatar: https://example.org/avatar.png
    description: No RSS exposed; fallback will try dated links on homepage/archives.

  - name: Offline Friend
    site: https://old-domain.example/
    avatar: https://old-domain.example/avatar.png
    description: Keep the link card, but skip friend-circle crawling.
    circle: false
    circle_reason: domain unavailable
```

`feed` 是可选项。未填写或抓取失败时，脚本会尝试解析这些页面中的日期文章链接：

- 站点首页
- `/archives/`
- `/posts/`
- `/post/`

如果某个站点很慢，可以给单个朋友覆盖超时：

```yaml
  - name: Slow Friend
    site: https://slow.example/
    feed: https://slow.example/rss.xml
    timeout: 25
    retries: 3
```

2. 本地生成数据：

```bash
python scripts/fetch_feeds.py
```

3. 在 Hugo 中创建页面：

```yaml
---
title: "友链朋友圈"
layout: "friends-circle"
slug: "friends-circle"
comments: false
---
```

4. 启用 `.github/workflows/update.yml` 后，GitHub Actions 会每 3 小时更新一次 `data/friend_posts.json`。

## Stack Theme

如果你使用 Hugo Stack，把这些文件复制到站点根目录即可：

- `scripts/fetch_feeds.py`
- `data/friends.yaml`
- `data/friend_posts.json`
- `layouts/page/friends-circle.html`
- `content/friends-circle/index.md` 或你自己的页面路径
- `.github/workflows/update.yml`

页面会使用 Stack 的 `baseof.html`、暗色模式变量和卡片变量。
