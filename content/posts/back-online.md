---
title: Back Online
date: 2018-09-02
---

After a 4-year suspension, my personal website is back online again. To protect myself from being sentimental, I'll keep this post as a note on how to setup the new website.

To save budget, I bought a VPS instance from Vultr. And here is my referral code: https://www.vultr.com/?ref=7082522 
If you use the code to register, both you and me would get some money back.

I chose [Hugo](https://github.com/gohugoio/hugo) as the static site generator, and it's easy to install. On Ubuntu 18.04, it's simple as `apt-get install hugo`. Hugo has built a very successful ecosystem so tons of themes could be used for free, at [Hugo Themes](http://themes.gohugo.io/). After some comparison, I selected "Black & Light", which as you could see, stands its name.

To serve those generated files, I use nginx, which is kind of the default choice today. Another default choice is using [Let's Encrypt](https://letsencrypt.org/getting-started/) for https support. The not-so-default choice is to use [Dropbox Headless](https://www.dropbox.com/install-linux) to do backups.

By the way, "source code" of this site could be accessed [here](https://github.com/cannium/site2018). Feel free to peek into it.