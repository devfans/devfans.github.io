---
layout: post
title:  "Setup a blog site on github with Jekyll"
author: devfans
categories: [ Jekyll, tutorial ]
image: https://static.livefeed.cn/static/blog/TZHYCANBO9.jpg
tags: [featured]
---

It's never too late to make yourself a personal blog site, as long as you would like to spend sometime on some writings for someone else to take a look when they are searching about the things you are sharing. I am saying this to myself when I decided to setup this site and write this blog as the first post on the site(https://blog.devfans.io).

#### Resources we would use

+ Github Pages: Github is an open platform for everyone whoever has the `open source/resource` idea in his mind. It provide static web site serving service for free which is named as github pages.
  (link: https://pages.github.com/)

+ Jekyll: Jekyll is the most popular tool to transform plain text into static websites and blogs.
  (link: https://jekyllrb.com/)
+ A domain name(optional): You may like to have your own domain name like `https://blog.devfans.io`, but it's optional because github provide you a free domain name as your-user-name.github.io

#### Steps to setup it up

- If you get yourself a domain name for your site, go to the provider's website and create a `CNAME` record for your domain name with value as `your-gihtub-username.github.io`.

- Create a repo after you have your own github account, name it as `your-gihtub-username.github.io` and enable the github pages, and configure the custom domain name if you have one. Check the `enforce https` option is recommended in addition.

- Clone a jekyll blog template you would like to choose (The one I am using is `https://github.com/wowthemesnet/mundana-theme-jekyll.git`).

- Config the site's name and author in `_config.yaml`, setup a comment tool (like `disqus`).

- Push the working copy to your github repo, and it's up.


#### Sum up

I was mainly talking about the ideas to setup a personal blog site on github pages with Jekyll and didnt cover every details during the process. If you really want the detailed procedures, leave a comment below to let me know.

Besides, I am not a native English speaker, just trying to use it and get used to it.


