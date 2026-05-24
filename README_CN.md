# Hugo Friend Circle 中文说明

> 基于 RSS / Atom、HTML 兜底解析和 GitHub Actions 的 Hugo 无后端友链朋友圈。

[English README](./README.md)

## 示例站点

示例站点：<https://www.elainafan.one/friends/>

这个示例接入在 Hugo Stack 主题的友链页面中：上方保留普通友链卡片，下方展示朋友们最近写了什么。

搭建记录：<https://www.elainafan.one/p/在hugo博客中优雅地添加轻量朋友圈/>

## 这是什么

`hugo-friend-circle` 是一个给 Hugo 博客用的友链朋友圈组件。它不需要后端、不需要数据库，只靠脚本抓取朋友站点的 RSS / Atom 或归档页，再生成 Hugo 可读取的 `data/friend_posts.json`。

适合这些场景：

- 你的博客是 Hugo 静态站点。
- 你有一页友链，但想让它“活起来”。
- 你想展示朋友们最近更新了哪些文章。
- 你希望使用 GitHub Actions 自动更新，而不是自己维护后端。

## 功能

- 抓取 RSS / Atom。
- 如果站点没有暴露 RSS，会尝试从首页、`/archives/`、`/posts/`、`/post/` 解析带日期的文章链接。
- 按发布时间倒序展示。
- 卡片显示头像、文章标题、日期。
- 支持搜索、按朋友筛选、`Load More` 分批展开。
- 抓取失败只记录 warning，不会让 Hugo 构建失败。
- 支持配置总文章数、单个朋友文章数、起始日期、超时、重试和跳过站点。
- 可作为独立 Hugo 页面，也可以接入 Hugo Stack 主题的友链页。

## 数据说明

本仓库只保留示例数据，不放个人友链数据。

如果你把这个项目接入自己的博客，真正的 `data/friends.yaml` 和生成的 `data/friend_posts.json` 应该放在你的博客仓库里。不要把私人或不想公开的友链配置提交到模板仓库。

## 快速开始

把这些文件复制到你的 Hugo 站点根目录：

- `scripts/fetch_feeds.py`
- `data/friends.yaml`
- `data/friend_posts.json`
- `layouts/page/friends-circle.html`
- `content/friends-circle/index.md`，或你自己的页面路径
- `.github/workflows/update.yml`，如果你想把生成数据提交进仓库

编辑 `data/friends.yaml`：

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
    description: No RSS exposed; fallback will try dated links on homepage and archives.

  - name: Offline Friend
    site: https://old-domain.example/
    avatar: https://old-domain.example/avatar.png
    description: Keep the link card, but skip friend-circle crawling.
    circle: false
    circle_reason: domain unavailable
```

`feed` 是可选项。没有填写或抓取失败时，脚本会尝试解析：

- 站点首页
- `/archives/`
- `/posts/`
- `/post/`

如果某个朋友的站点比较慢，可以单独覆盖超时和重试：

```yaml
  - name: Slow Friend
    site: https://slow.example/
    feed: https://slow.example/rss.xml
    timeout: 25
    retries: 3
```

本地生成数据：

```bash
python scripts/fetch_feeds.py
```

创建 Hugo 页面：

```yaml
---
title: "Friend Circle"
description: "Recent posts from friends."
layout: "friends-circle"
slug: "friends-circle"
comments: false
---
```

## 如何保持最新

有两种推荐方式。

### 方案 A：定时生成并提交数据

使用 `.github/workflows/update.yml`，让 GitHub Actions 定时运行脚本并提交 `data/friend_posts.json`。

在你自己的博客仓库里加上 schedule：

```yaml
on:
  schedule:
    - cron: "0 */3 * * *"
  workflow_dispatch:
```

优点是每次更新都能在 git 历史里看到。缺点是会产生很多自动提交。

### 方案 B：构建前临时生成

在部署 workflow 中先抓取，再运行 Hugo：

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.x"

- name: Fetch friend circle
  run: python scripts/fetch_feeds.py

- name: Build site
  run: hugo --minify
```

优点是仓库更干净，不需要提交生成数据。缺点是如果想自动刷新页面，需要让部署 workflow 本身定时运行。
