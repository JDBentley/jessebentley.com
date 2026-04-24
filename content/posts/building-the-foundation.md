+++
date = '2026-04-21T22:36:42-04:00'
draft = false
title = 'Building the Foundation'
description = 'How I set up a professional Hugo site with automated deployment from scratch and why I chose to do it right instead of doing it fast.'
author = 'Jesse Bentley'
tags = ['hugo', 'git', 'github-actions', 'bluehost', 'cloudflare']
categories = ['meta']
series = ['site-build']
showToc = true
tocopen = false
+++

## The Goal:

Get JesseBentley.com live with a proper automated deployment pipeline. Not the quick path but the right path. Every push to GitHub should automatically build and deploy the site without me touching anything manually.

---

## What I Expected

I expected this to take an hour. Install Hugo, pick a theme, point a domain at something, done. That's how most tutorials make it sound.

What I actually wanted was different. A CI/CD pipeline using GitHub Actions deploying via SSH to Bluehost. Cloudflare sitting in front for DNS, CDN, and security. PaperMod as the theme because it's fast, clean, and doesn't look like every other developer blog.

Doing it properly was always going to take longer than doing it fast. I chose properly.

---

## What I Ran Into

The first thing that slowed me down was understanding why each piece exists before implementing it. That's intentional as I don't want black boxes in my own infrastructure. If something breaks at 2am I need to know how it works well enough to fix it.

A few things worth noting for anyone following along:

The `code` command in terminal needs to be installed separately from VS Code itself. VS Code doesn't do this automatically. Command Palette → Shell Command: Install 'code' command in PATH. Easy fix, not obvious the first time.

Git multi-line commit messages through the terminal flag are messier than they look in tutorials. Setting VS Code as your default Git editor with `git config --global core.editor "code --wait"` and using `git commit` without the `-m` flag is cleaner and more professional.

Homebrew needs to be added to PATH after installation. The installer tells you the commands so run all three of them before testing `brew --version` or you'll get command not found and think something went wrong.

---

## What the Outcome Was

Hugo installed, PaperMod theme configured, About page written, blog post archetype built with a five section template that auto-generates on every new post. Everything committed to GitHub with conventional commit messages from day one.

The full automated deployment pipeline of GitHub Actions, SSH, Cloudflare is the next step. The foundation is solid. The automation gets built on top of it.

---

## GitHub

Everything built in this post lives in the site repository.
The Hugo configuration, PaperMod theme setup, and folder
structure are all visible there:

[jessebentley.com](https://github.com/JDBentley/jessebentley.com)

---

## What's Next

Setting up Cloudflare DNS, enabling SSH on Bluehost, and writing the GitHub Actions workflow that ties it all together. When that's done every push to GitHub deploys the site automatically. That post covers the full CI/CD pipeline from scratch.