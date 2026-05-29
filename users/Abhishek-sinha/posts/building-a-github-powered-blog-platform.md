---
title: Building a GitHub-Powered Blog Platform
date: 2026-05-30
author: Abhishek-sinha
authorName: Abhishek Sinha
authorRole: Full Stack And Mobile Developer
authorAvatar: https://avatars.githubusercontent.com/abhisheksinha20p
authorBio: Full Stack And Mobile Developer
authorGithub: https://github.com/abhisheksinha20p
authorLinkedin: https://www.linkedin.com/in/abhishek-sinha-0897aa23b/
tags: [web-development, github, react, python, blog]
category: Web-Development
excerpt: >-
  I built a blog platform where I own every byte of content — no database, no CMS, and no platform lock-in.. Just three GitHub repos and some clean code..
readTime: 2 min read
coverImage: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/building-a-github-powered-blog-platform/images/cover.png
---

I built a blog platform where I own every byte of content — no database, no CMS, and no platform lock-in. Just three GitHub repos and some clean code.

## The Idea

Most blogging platforms hold your content hostage. You write great articles, but moving them elsewhere is a nightmare. I wanted something different — a system where my content is just markdown files in a GitHub repo, fully portable and version-controlled from day one.

## How It Works

The platform is split across three repos that work together:

- **website** — A React frontend that reads articles from a public GitHub repo. No authentication, no database, just fetch and render.
- **mcp** — A Python MCP server that handles content creation and publishing. It generates proper YAML front-matter, cover images, and updates the article registry.
- **blog-store** — A public GitHub repo that stores all articles as plain markdown files with a registry.json index.

## The Article Pipeline

Every article goes through the same pipeline:

1. **Write** — Markdown content with YAML front-matter (title, date, author, tags, excerpt, read time)
2. **Cover** — A cover image is generated from the title and first paragraph
3. **Upload** — The markdown file is pushed to the blog-store repo via GitHub API
4. **Register** — The article metadata is added to registry.json
5. **Display** — The website fetches registry.json and renders the article

## Why This Approach Works

- **Zero hosting costs** — GitHub serves everything
- **Full ownership** — Your content is plain markdown, portable to any platform
- **Version control** — Every edit is tracked in git history
- **Multi-author ready** — Each author gets their own directory in the repo
- **No lock-in** — Migrate to another platform anytime by exporting your markdown files

## The Tech Stack

- **Frontend**: React + Vite + TypeScript, deployed as static files
- **Backend**: Python MCP server using FastMCP
- **Storage**: GitHub REST API for file uploads
- **Content**: Markdown with YAML front-matter
- **Cover Images**: HTML template rendered by Chrome headless, uploaded as PNG

## What's Next

I will keep writing about web development, mobile development, and the tools I build along the way. This platform will grow with me — every article I publish is another step in my coding journey.

---

Built with React, Python, and GitHub. No databases were hurt in the making of this blog.