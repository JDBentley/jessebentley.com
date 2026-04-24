+++
date = '2026-04-27T08:00:00-04:00'
draft = false
title = 'Deploying Hugo with GitHub Actions and Bluehost'
description = 'How I built a proper automated deployment pipeline instead of taking the easy Netlify path — and what I learned along the way.'
author = 'Jesse Bentley'
tags = ['hugo', 'github-actions', 'bluehost', 'cloudflare', 'ssh', 'ci-cd']
categories = ['meta']
series = ['site-build']
showToc = true
tocopen = false
+++

## The Goal

Get JesseBentley.com deploying automatically every time I push to GitHub. Not the quick path but the right one. I wanted a proper CI/CD pipeline using GitHub Actions, SSH deployment to Bluehost, and Cloudflare handling DNS and security in front of everything.

---

## What I Expected

Honestly I didn't know what to expect. I'd never set up a deployment pipeline before with Github Actions. GitHub Actions was completely new territory for me. I knew Netlify was the easier option but I was already paying for Bluehost hosting and the domain, so using what I had made more sense than adding another service to the stack.

There were other reasons too. A static site has essentially zero attack surface compared to WordPress and contains no database to inject, no PHP to exploit, no plugins with unpatched CVEs. For someone moving into security that felt like the right call. Running your own pipeline also means you understand exactly how the deployment works. No black boxes.

---

## What I Ran Into

The waiting. DNS propagation sounds simple until you're sitting there watching a progress bar that moves on its own schedule. Moving nameservers from Bluehost to Cloudflare felt like it took forever even though it was actually pretty fast. That's just how DNS works and there's nothing you can do but wait.

Outside of that everything went slower than I anticipated but that was intentional. I took my time with every step, made sure I understood what each command was doing before running it, and it paid off. The GitHub Actions workflow file, the SSH key generation, wiring the secrets into the repo and none of it was as complicated as it looked at first glance. It just required patience and reading carefully.

A few small things tripped me up:

The `code` command in terminal needs to be installed separately even if VS Code is already installed. Command Palette → Shell Command: Install 'code' command in PATH. Easy fix, not obvious the first time.

The Personal Access Token needs the `workflow` scope specifically to push files inside `.github/workflows/`. GitHub blocks that push without it and the error message is clear once you know what you're looking at.

SSH key generation needs the `.ssh` folder to exist first. If it doesn't, create it with `mkdir -p ~/.ssh` and set permissions with `chmod 700 ~/.ssh` before generating keys.

---

## What the Outcome Was

Watching the GitHub Actions pipeline run for the first time and seeing the site update live was genuinely satisfying. Push to GitHub, watch the workflow tab, site updates in under 90 seconds. Every future blog post works the same way you write it, commit it, push it, done.

More than the automation itself, it feels like a real foundation. Cloudflare is sitting in front of everything handling CDN, DDoS protection, and SSL. Email routing through Cloudflare forwards jesse@jessebentley.com to ProtonMail without running a mail server. The whole stack is clean, documented, and I understand every piece of it.

That last part matters more than anything else. When something breaks at 2am I'll know where to look.

---

## What's Next

The site is live and deploying automatically but the pipeline itself deserves a deeper look. Next post breaks down the GitHub Actions workflow file line by line — what each step does, how the cron schedule works, and why CI/CD matters even for a personal blog.