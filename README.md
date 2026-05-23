# Hugo Friend Circle

A backend-free friend-circle page for Hugo blogs, powered by RSS / Atom, HTML fallback, and GitHub Actions.

It does not need a server or a database. The fetch script reads `data/friends.yaml`, writes `data/friend_posts.json`, and Hugo renders the page from that data file.

## Features

- Fetch RSS and Atom feeds.
- If a site does not expose RSS, try dated links on the homepage, `/archives/`, `/posts/`, and `/post/`.
- Sort posts by publish time.
- Show avatar, article title, and date.
- Search, filter by friend, and expand with `Load More`.
- Keep failed feeds as warnings instead of breaking the build.
- Configure total post limit, per-friend limit, start date, timeout, retries, and skipped sites.
- Works as a standalone Hugo page or as a Stack-theme page.

## Data Policy

This repository intentionally contains only example data.

Do not put your private or personal friend list into this template repository unless you want it to be public. For a real blog, keep your own `data/friends.yaml` and generated `data/friend_posts.json` in your blog repository.

## Quick Start

Edit `data/friends.yaml` in your Hugo site:

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

`feed` is optional. If it is missing or unavailable, the script tries to parse dated article links from:

- the homepage
- `/archives/`
- `/posts/`
- `/post/`

For slow sites, override timeout and retries per friend:

```yaml
  - name: Slow Friend
    site: https://slow.example/
    feed: https://slow.example/rss.xml
    timeout: 25
    retries: 3
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

## Updating Automatically

There are two common ways to keep the page fresh.

### Option A: Commit Generated Data

Use `.github/workflows/update.yml`. It runs the fetch script and commits `data/friend_posts.json`.

Add a schedule in your own blog repository if you want automatic updates:

```yaml
on:
  schedule:
    - cron: "0 */3 * * *"
  workflow_dispatch:
```

This is easy to inspect because every update is committed, but it creates frequent generated-data commits.

### Option B: Generate Before Building

Run the fetch script in your deploy workflow before `hugo`:

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.x"

- name: Fetch friend circle
  run: python scripts/fetch_feeds.py

- name: Build site
  run: hugo --minify
```

This avoids committing generated data. If you want updates without manual commits, schedule the deploy workflow itself.

## Stack Theme

For a Stack-theme site, copy these files into the site root:

- `scripts/fetch_feeds.py`
- `data/friends.yaml`
- `data/friend_posts.json`
- `layouts/page/friends-circle.html`
- `content/friends-circle/index.md`, or your own page path
- `.github/workflows/update.yml`, if you want committed generated data

The template uses Stack-compatible CSS variables when available, and falls back to plain Hugo-friendly styling otherwise.
