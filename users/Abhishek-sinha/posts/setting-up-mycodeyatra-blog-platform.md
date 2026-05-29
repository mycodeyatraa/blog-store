---
title: Setting Up MyCodeYatra: A GitHub-Powered Blog Platform
date: 2026-05-30
author: Abhishek-sinha
authorName: Abhishek Sinha
authorRole: Full Stack And Mobile Developer
authorAvatar: https://avatars.githubusercontent.com/abhisheksinha20p
authorBio: Full Stack And Mobile Developer
authorGithub: https://github.com/abhisheksinha20p
authorLinkedin: https://www.linkedin.com/in/abhishek-sinha-0897aa23b/
tags: [web-development, github, blog, mcp, react]
category: Web Development
excerpt: >-
  How I built a multi-author blog platform with a React frontend, an MCP server for content management, and a public GitHub repo as the data store — all without a traditional database.
readTime: 3 min read
coverImage: 
---

# Setting Up MyCodeYatra: A GitHub-Powered Blog Platform

I wanted a blog where I own my content completely — no platform lock-in, no database to manage, just markdown files in a GitHub repo.

Here's how I built it.

## The Architecture

The platform uses three colocated repos:

- **website** (React + Vite) — reads articles from a public GitHub repo
- **mcp** (Python MCP server) — creates and publishes articles via GitHub API
- **blog-store** (public repo) — stores markdown articles and a egistry.json index

No database, no CMS, no SaaS. Just GitHub and simple code.

## How It Works

### The Blog-Store Repo

This is the heart of the system. It's a public GitHub repo containing:

`
registry.json
users/
  Abhishek-sinha/
    posts/
      my-article.md
`

The egistry.json file acts as an index, listing all articles with their metadata. Each article is a markdown file with YAML front-matter containing title, date, author details, tags, and excerpt.

### The Website

The frontend fetches egistry.json from aw.githubusercontent.com, displays a grid of recent posts, and loads individual articles on demand. No authentication needed — everything is public.

### The MCP Server

The MCP (Model Context Protocol) server handles content creation. It generates articles with proper front-matter, uploads them to the blog-store, and updates the registry — all through the GitHub API.

## Why This Approach

- **Zero hosting costs** — GitHub serves the files
- **Full content ownership** — articles are plain markdown, portable anywhere
- **Version control** — every change is tracked in git
- **Multi-author ready** — each author gets their own directory
- **No lock-in** — migrate away anytime

## What's Next

I'll be writing more articles about web development, mobile development, and the tools I build. Stay tuned!

---

*Built with React, Vite, Python, and GitHub.*