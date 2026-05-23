# Hugo Friend Circle

> A backend-free friend-circle page for Hugo blogs, powered by RSS / Atom, HTML fallback, and GitHub Actions.
>
> 基于 RSS / Atom、HTML 兜底解析和 GitHub Actions 的 Hugo 无后端友链朋友圈。

[中文说明](#中文说明) | [English Guide](#english-guide)

## Example Site

Live example: <https://www.elainafan.one/friends/>

The example is integrated into a Hugo Stack theme friend-link page. Friend cards stay on the page, and the friend-circle stream is rendered below them.

示例站点：<https://www.elainafan.one/friends/>

这个示例接入在 Hugo Stack 主题的友链页面中：上方保留普通友链卡片，下方展示朋友们最近写了什么。

## 中文说明

### 这是什么

`hugo-friend-circle` 是一个给 Hugo 博客用的友链朋友圈组件。它不需要后端、不需要数据库，只靠脚本抓取朋友站点的 RSS / Atom 或归档页，再生成 Hugo 可读取的 `data/friend_posts.json`。

适合这些场景：

- 你的博客是 Hugo 静态站点。
- 你有一页友链，但想让它“活起来”。
- 你想展示朋友们最近更新了哪些文章。
- 你希望使用 GitHub Actions 自动更新，而不是自己维护后端。

### 功能

- 抓取 RSS / Atom。
- 如果站点没有暴露 RSS，会尝试从首页、`/archives/`、`/posts/`、`/post/` 解析带日期的文章链接。
- 按发布时间倒序展示。
- 卡片显示头像、文章标题、日期。
- 支持搜索、按朋友筛选、`Load More` 分批展开。
- 抓取失败只记录 warning，不会让 Hugo 构建失败。
- 支持配置总文章数、单个朋友文章数、起始日期、超时、重试和跳过站点。
- 可作为独立 Hugo 页面，也可以接入 Hugo Stack 主题的友链页。

### 数据说明

本仓库只保留示例数据，不放个人友链数据。

如果你把这个项目接入自己的博客，真正的 `data/friends.yaml` 和生成的 `data/friend_posts.json` 应该放在你的博客仓库里。不要把私人或不想公开的友链配置提交到模板仓库。

### 快速开始

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

### 如何保持最新

有两种推荐方式。

#### 方案 A：定时生成并提交数据

使用 `.github/workflows/update.yml`，让 GitHub Actions 定时运行脚本并提交 `data/friend_posts.json`。

在你自己的博客仓库里加上 schedule：

```yaml
on:
  schedule:
    - cron: "0 */3 * * *"
  workflow_dispatch:
```

优点是每次更新都能在 git 历史里看到。缺点是会产生很多自动提交。

#### 方案 B：构建前临时生成

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

## English Guide

### What Is This

`hugo-friend-circle` is a friend-circle component for Hugo blogs. It needs no backend and no database. A Python script fetches friends' RSS / Atom feeds or dated article links from HTML pages, writes `data/friend_posts.json`, and Hugo renders the page from that data.

It is useful if:

- your blog is a Hugo static site;
- you already have a friend-link page;
- you want to show what your friends have recently written;
- you prefer GitHub Actions over maintaining a backend service.

### Features

- Fetch RSS and Atom feeds.
- If RSS is missing, try dated links on the homepage, `/archives/`, `/posts/`, and `/post/`.
- Sort posts by publish time.
- Show avatar, article title, and date.
- Search, filter by friend, and expand with `Load More`.
- Keep failed feeds as warnings instead of breaking the build.
- Configure total post limit, per-friend limit, start date, timeout, retries, and skipped sites.
- Works as a standalone Hugo page or as a Stack-theme page.

### Data Policy

This repository intentionally contains only example data.

For a real blog, keep your own `data/friends.yaml` and generated `data/friend_posts.json` in your blog repository. Do not put private or personal friend data into this template repository unless you want it to be public.

### Quick Start

Copy these files into your Hugo site root:

- `scripts/fetch_feeds.py`
- `data/friends.yaml`
- `data/friend_posts.json`
- `layouts/page/friends-circle.html`
- `content/friends-circle/index.md`, or your own page path
- `.github/workflows/update.yml`, if you want committed generated data

Edit `data/friends.yaml`:

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
```

Generate data locally:

```bash
python scripts/fetch_feeds.py
```

Create a Hugo page:

```yaml
---
title: "Friend Circle"
description: "Recent posts from friends."
layout: "friends-circle"
slug: "friends-circle"
comments: false
---
```

### Keeping It Fresh

There are two common ways.

#### Option A: Commit Generated Data

Use `.github/workflows/update.yml`. It runs the fetch script and commits `data/friend_posts.json`.

Add a schedule in your own blog repository:

```yaml
on:
  schedule:
    - cron: "0 */3 * * *"
  workflow_dispatch:
```

This is easy to inspect because every update is committed, but it creates frequent generated-data commits.

#### Option B: Generate Before Building

Run the fetch script before `hugo` in your deploy workflow:

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.x"

- name: Fetch friend circle
  run: python scripts/fetch_feeds.py

- name: Build site
  run: hugo --minify
```

This keeps the repository cleaner. If you want automatic refreshes, schedule the deploy workflow itself.
