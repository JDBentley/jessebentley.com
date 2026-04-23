# JesseBentley.com

Personal portfolio and security blog documenting a transition from 
enterprise IT and military service into offensive security and 
penetration testing.

Built with Hugo and PaperMod. Deployed automatically via GitHub 
Actions over SSH to Bluehost, served through Cloudflare.

## About

20 years of enterprise IT experience spanning military service and 
Fortune 500 environments. Currently leading a team of 40 in hardware 
support and networking while actively building toward a dedicated 
penetration testing role.

This blog documents the journey in real time — the tools being built, 
the skills being developed, and the mistakes made along the way.

## Active Projects

**Flipper Zero Physical Security Suite**
A complete hardware security toolkit programmed from scratch in C, 
C++, and Python. WiFi device scanner, Sub-GHz analyzer, IR lens 
detector, NFC reader, GPS logging, and a CrowPanel LVGL touchscreen 
interface. Every line documented on the blog.

**GameMaster AI**
A reinforcement learning system that teaches AI agents to play Steam 
games from scratch. No Gymnasium, no Stable Baselines — everything 
built by hand in Python and PyTorch.

## Tech Stack # jessebentley.com
Hugo v0.160.1 extended    Static site generator
PaperMod                  Theme
GitHub Actions            CI/CD pipeline
Bluehost                  Hosting (SSH deployment)
Cloudflare                DNS, CDN, security

## Deployment

Every push to `main` triggers an automated build and deployment:
git push origin main
↓
GitHub Actions builds Hugo site
↓
Deploys via rsync over SSH to Bluehost
↓
Cloudflare serves it globally
↓
jessebentley.com updates in ~90 seconds

## Local Development

```bash
# Clone the repo
git clone https://github.com/JDBentley/jessebentley.com.git
cd jessebentley.com

# Install Hugo (Mac)
brew install hugo

# Start local server
hugo server -D

# View at http://localhost:1313
```

## Blog Schedule
Monday / Tuesday  → Flipper Zero build series
Friday            → GameMaster AI series
Bonus             → Conference recaps and community posts

## Connect

- [jessebentley.com](https://jessebentley.com)
- [LinkedIn](https://www.linkedin.com/in/jesse-bentley/)
- [GitHub](https://github.com/JDBentley)
- [Email](mailto:jesse@jessebentley.com)