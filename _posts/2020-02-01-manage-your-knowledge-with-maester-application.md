---
layout: post
title:  "Manage Your Knowledge with Maester Application"
author: devfans
categories: [ Knowledge, Mind, Bookmark ]
image: /images.pexels.com/photos/162622/facebook-login-office-laptop-business-162622.jpeg?cs=srgb&dl=communication-connection-data-162622.jpg&fm=jpg
image: /cdn.stocksnap.io/img-thumbs/960w/A4GPO5BBZD.jpg
tags: [sticky]
---

Sometime, you may find an interesting article or some awesome resource, but after a period, you are feeling hard to find where it is. Is it in your bookmark list? What's the name of it? After 10 minutes searching, you will feel depressed if you cannot really find it. Or one day you think out a brilliant idea, but after a month, you forget about it and you may forget it forever. Yes, you need a tool to help you to avoid this kind of annoying situations.

You may ever used some tree like structured knowledge chart to help you classify things and compose a clear map dumped from your mind and try to maintain it with adding more and more content. This is a common and popular way for human to organize things and manage tasks. I am sure you can easily find a lot of neat tools that could do this for you. 

However, sometime you may feel hard to compose a tree with the pieces of knowledge or content, and you even can not easily tell the dependency relationship of two items or which should be sub-content of another. This is true because it's quite common as you see some objects or things do not really have a tree structure like relationship, and one node could have relations with unlimited other nodes, you might feel like a mess. For this situation, we commonly use another way to manage them, a flat list with tags and categories. It's very common some of the objects share the same category or same tags. If you want to locate something, just search by tags directly. Yeah, the name Maester from GoT, is used to name the tool I will show in this post, and it's the simple tool to handle mess situations for your.

#### So what does Maester do:

- You can share the web page address to it and create a page in it with tags and a category.
- You can easily search with tags, categories or names or even just a keyword
- It provides cloud storage service and you can seamlessly synchronize things across platforms or devices.
- It's designed to support common content types, like a webpage address, a piece of text or other rich content.

Currently only MacOS versiona and IOS version are available.

<link rel="stylesheet" href="https://unpkg.com/flickity@2/dist/flickity.min.css">
<script src="https://unpkg.com/flickity@2/dist/flickity.pkgd.min.js"></script>
<!-- JavaScript -->

<style>
/* external css: flickity.css */

* { box-sizing: border-box; }

body { font-family: sans-serif; }
.main-carousel {
  width: 50%;
  margin: 0 auto;
}
.carousel {
}

.carousel-cell {
  width: 100%;
  /*height: 400px;*/
  margin-right: 10px;
  /*background: #333;*/
}

.carousel-cell-image {
  display: block;
  /*max-height: 100%;*/
  margin: 0 auto;
  width: 100%;
  opacity: 0;
  -webkit-transition: opacity 0.4s;
          transition: opacity 0.4s;
}

/* fade in lazy loaded image */
.carousel-cell-image.flickity-lazyloaded,
.carousel-cell-image.flickity-lazyerror {
  opacity: 1;
}
</style>
<p>
<div class="main-carousel" style="width: 100%" data-flickity='{ "cellAlign": "left", "contain": true, "lazyLoad": true }' js-flickity>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_mac_0.png" alt="Maester mac 0" />
  </div>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_mac_1.png" alt="Maester mac 1" />
  </div>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_mac_2.png" alt="Maester mac 2" />
  </div>
</div>
</p>


<p>
<div class="main-carousel" data-flickity='{ "cellAlign": "left", "contain": true, "lazyLoad": true }' js-flickity>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_ios_0.png" alt="Maester ios 0" />
  </div>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_ios_1.png" alt="Maester ios 1" />
  </div>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_ios_2.png" alt="Maester ios 2" />
  </div>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_ios_3.png" alt="Maester ios 3" />
  </div>
  <div class="carousel-cell">
    <img class="carousel-cell-image" data-flickity-lazyload="/assets/images/maester/maester_ios_4.png" alt="Maester ios 4" />
  </div>
</div>
</p>

