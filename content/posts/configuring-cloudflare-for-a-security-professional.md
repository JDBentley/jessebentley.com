+++
date = '2026-05-04T00:00:00-04:00'
draft = false
title = 'Configuring Cloudflare for a Security Professional'
description = 'Why I put Cloudflare in front of my personal site, what the free tier actually gives you, and why it matters more when you work in security.'
author = 'Jesse Bentley'
tags = ['cloudflare', 'dns', 'security', 'email', 'cdn']
categories = ['meta']
series = ['site-build']
showToc = true
tocopen = false
+++

## The Goal

Get Cloudflare sitting in front of JesseBentley.com handling DNS,
CDN, and security. The goal wasn't just faster load times and it was
making sure the infrastructure behind a security portfolio reflects
the same thinking I want to bring to the field professionally.

---

## What I Expected

I knew Cloudflare existed and I knew it was used to prevent DDoS
attacks. That was basically it. I assumed it was a commercial product
for enterprise customers with budgets and contracts. I didn't think
a free tier would be worth much.

I was wrong about that.

---

## What I Ran Into

The first thing that hit me opening the Cloudflare dashboard was
how wide the free tier actually is. DDoS mitigation, global CDN,
SSL certificates, DNS management, analytics, bot protection, and
email routing and all of it free, all active the moment you point your
nameservers at Cloudflare.

I'd assumed most of it would be locked behind a paid plan. It isn't.

**DNS setup was straightforward.** Cloudflare imported all my
existing Bluehost DNS records automatically when I added the domain.
I just had to clean up the records I didn't need, the old mail server
entries that were no longer relevant, and point the nameservers.
The hardest part was waiting for propagation, which felt slow but
was actually under an hour.

**The mail records needed attention.** Cloudflare can't proxy mail
traffic the same way it proxies web traffic. Mail records need to
be DNS only and no orange cloud. Getting that wrong would have broken
email delivery silently which is exactly the kind of subtle mistake
that's annoying to debug later.

**The email routing was the unexpected one.** I wasn't planning to
set up email. I knew services like this existed but didn't know
Cloudflare was one of them. For free Cloudflare will receive email
sent to any address at your domain and forward it wherever you want.

So jesse@jessebentley.com now forwards to my ProtonMail account.
No mail server to maintain. No deliverability issues from shared
hosting. No ongoing cost. One rule in the Cloudflare dashboard
and it just works.

**Why ProtonMail specifically:**
End to end encrypted email from
a provider that takes privacy
seriously is the right call for
someone moving into security.
Using a custom domain address
for professional contact while
keeping ProtonMail's security
for actual message handling
is the best of both options.

**The security angle that most tutorials skip:**

Most Cloudflare tutorials are written for people who want a faster
website. That's valid but it's not the whole picture for someone
working in security.

Security professionals deal with sensitive data, client
infrastructure details, vulnerability reports, engagement findings.
If your personal site is sitting naked on shared hosting with no
protection in front of it you're creating an unnecessary attack
surface. A compromised personal site can be used as a pivot point.
It reflects poorly on your judgment professionally. And if you're
handling any client adjacent communication through that domain
the risk compounds.

Cloudflare in front of everything means:

- Traffic goes through Cloudflare before it ever touches Bluehost
- DDoS attacks get absorbed before reaching the origin server
- The WAF blocks common web attack patterns automatically
- Your real server IP stays hidden behind Cloudflare's network
- SSL is handled and enforced at the edge
- Bot traffic gets filtered before it hits your hosting

None of that costs anything on the free tier. There's no reason
not to have it and especially if you're actively building a public
profile in the security community.

---

## What the Outcome Was

JesseBentley.com now sits behind Cloudflare. DNS resolves through
their global network. SSL is enforced. Email routes from
jesse@jessebentley.com to ProtonMail automatically. The real
Bluehost server IP is hidden. All of it runs on the free tier.

The dashboard has more features than I've explored yet. That's
fine, the foundation is right and I understand what's in place
and why. That's what matters.

---

## GitHub

The site repository shows the full Hugo configuration that
Cloudflare serves:

[jessebentley.com](https://github.com/JDBentley/jessebentley.com)

Cloudflare configuration is managed through the Cloudflare
dashboard directly — there are no config files to commit.
The DNS records, email routing rules, and security settings
all live in the Cloudflare UI.

---

## What's Next

The site infrastructure series is nearly wrapped up. Next post
covers the editorial structure, Hugo archetypes, the five section
blog format, and how the Monday, Tuesday, Friday posting schedule
keeps two active project blogs running without either one falling
behind.