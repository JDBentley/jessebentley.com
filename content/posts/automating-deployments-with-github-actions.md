+++
date = '2026-04-28T08:00:00-04:00'
draft = false
title = 'Automating Deployments with GitHub Actions'
description = 'A line by line breakdown of the GitHub Actions workflow that automatically builds and deploys this site every time I push to GitHub — and how the cron schedule handles timed publishing.'
author = 'Jesse Bentley'
tags = ['github-actions', 'ci-cd', 'automation', 'hugo', 'bluehost', 'ssh']
categories = ['meta']
series = ['site-build']
showToc = true
tocopen = false
+++

## The Goal

Understand and document the GitHub Actions workflow powering this site. Not just get it working but actually understand what every line does and why it's there. The deployment pipeline is the backbone of everything published here and I wanted to know it well enough to fix it at 2am if something breaks.

---

## What I Expected

I expected GitHub Actions to be complicated. Another tool with its own syntax, its own quirks, its own documentation rabbit hole. I figured it would take a while to wrap my head around.

It wasn't complicated. That surprised me more than anything else.

---

## What I Ran Into

The concept that took the most time to internalize was the difference between the workflow trigger and the workflow steps. They look similar on the page but they do completely different things.

The trigger is the condition and what has to happen before the workflow runs at all. The steps are what the workflow actually does when it runs. Once that distinction clicked everything else made sense.

The other thing that surprised me was finding out GitHub Actions supports cron jobs natively. I knew cron from Linux and it's how you schedule recurring tasks on a server. Finding it built directly into a GitHub repo felt like discovering a feature that had been sitting there the whole time without anyone mentioning it.

Here's the full workflow file with every section explained:

```yaml
name: Deploy to Bluehost
```

This is just a label. It shows up in the GitHub Actions tab so you know what you're looking at when multiple workflows exist.

```yaml
on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 12 * * 1,5'
```

This is the trigger section. Two things cause this workflow to run:

The first is a push to the main branch. Every time new code or content gets committed and pushed GitHub fires this workflow automatically.

The second is a cron schedule. The string `0 12 * * 1,5` breaks down like this — 0 minutes past the hour, at hour 12 (noon UTC which is 8am Eastern), any day of the month, any month, on days 1 and 5 (Monday and Friday). So the site rebuilds automatically every Monday and Friday at 8am whether I push anything or not. Any post with a future date that has arrived gets published. I write a post, set the date, push it as a draft, and it goes live on its own schedule without me touching anything.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```

GitHub spins up a fresh Ubuntu server every time this runs. Completely clean, nothing carried over from previous runs. It builds the site, deploys it, then the server is destroyed. Free for public repositories.

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4
    with:
      submodules: true
      fetch-depth: 0
```

Downloads the repository code onto the Ubuntu server. The `submodules: true` flag is critical and without it the PaperMod theme doesn't download and Hugo can't build. The `fetch-depth: 0` pulls the full Git history which Hugo uses for accurate post dates.

```yaml
  - name: Install Hugo
    run: |
      wget -O hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.160.1/hugo_extended_0.160.1_linux-amd64.deb
      sudo dpkg -i hugo.deb
```

Downloads and installs the exact same Hugo version running locally. Version pinning matters here because if Hugo auto updated and introduced a breaking change the site would stop deploying until I noticed and fixed it. Pinning to v0.160.1 means the build is predictable every time.

```yaml
  - name: Build site
    run: hugo --minify
```

Runs Hugo and builds the site into the `public/` folder. The `--minify` flag compresses HTML, CSS, and JavaScript automatically. Smaller files load faster.

```yaml
  - name: Deploy to Bluehost
    env:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      SSH_HOST: ${{ secrets.SSH_HOST }}
      SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
      SSH_PORT: ${{ secrets.SSH_PORT }}
    run: |
      mkdir -p ~/.ssh
      echo "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
      chmod 600 ~/.ssh/deploy_key
      ssh-keyscan -p $SSH_PORT $SSH_HOST >> ~/.ssh/known_hosts
      rsync -avz --delete -e "ssh -i ~/.ssh/deploy_key -p $SSH_PORT -o StrictHostKeyChecking=no" public/ $SSH_USERNAME@$SSH_HOST:public_html/
```

This is the deployment step. Breaking it down:

The `env` block pulls the four GitHub secrets specifically our SSH private key, host IP, username, and port. These values are never visible in the code or logs. They only exist inside GitHub's encrypted secret storage and get injected here at runtime.

`mkdir -p ~/.ssh` creates the SSH directory on the Ubuntu runner.

`echo "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key` writes the private key to a file so SSH can use it.

`chmod 600` sets the correct permissions on the key file. SSH refuses to use a key file with too-open permissions and it's a security requirement not a suggestion.

`ssh-keyscan` adds Bluehost to the known hosts file. Without this step SSH would ask "are you sure you want to connect?" interactively and the automated pipeline would hang waiting for an answer that never comes.

`rsync` does the actual file transfer. The `--delete` flag removes any files on Bluehost that no longer exist locally, keeping the server clean. Only changed files get transferred so deployments are fast even as the site grows.

---

## What the Outcome Was

Every push to GitHub rebuilds and redeploys the site in under 90 seconds. Posts scheduled for Monday or Friday publish automatically without me doing anything. The workflow file is 40 lines and I understand every single one of them.

GitHub has a lot going on under the hood that you'd never discover just using it casually. Actions is one of those features. It's been sitting there the whole time.

---

## What's Next

Next post covers Cloudflare and why a security professional should care about what sits in front of their personal site, how DNS routing works, and what the free tier actually gives you beyond just faster load times.