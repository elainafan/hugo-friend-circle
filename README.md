# Hugo Friend Circle

一个基于 RSS / Atom 和 GitHub Actions 的 Hugo 友链朋友圈。它不需要后端、不需要数据库，只会定时抓取 `data/friends.yaml` 中的友链 feed，并生成 Hugo 可直接读取的 `data/friend_posts.json`。

## Features

- RSS / Atom 抓取
- GitHub Actions 定时更新
- 按发布时间倒序展示
- 支持朋友头像、站点名、文章标题、发布时间、摘要
- feed 抓取失败时只记录 warning，不阻塞构建
- 支持最大文章数、单站点文章数、保留天数配置
- 页面样式适配 Hugo Stack，也可以作为普通 Hugo 页面运行

## Quick Start

1. 编辑 `data/friends.yaml`：

```yaml
settings:
  max_posts: 80
  max_posts_per_friend: 6
  max_days: 180
  timeout: 15

friends:
  - name: Elainafan
    site: https://www.elainafan.one/
    feed: https://www.elainafan.one/index.xml
    avatar: https://www.elainafan.one/img/elainafan.jpg
    description: 痛饮所有蹇蹇之后？
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
