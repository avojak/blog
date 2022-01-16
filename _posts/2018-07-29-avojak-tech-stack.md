---
title: "The avojak.com Tech Stack"
description: "There are two main parts to avojak.com - the home page, and the blog, which is where you are right now!"
author: avojak
image: https://images.unsplash.com/photo-1472494731104-3ba69e52845b
tags:
  - software
---

There are two main parts to avojak.com - the "home page" ([avojak.com](https://avojak.com)), and the blog, which is where you are right now!

## Home

The avojak.com website is a single [Express](https://expressjs.com/) + [EJS](http://ejs.co/) application. I started out with the [sticky-footer-navbar](https://getbootstrap.com/docs/4.0/examples/sticky-footer-navbar/) Bootstrap template, but after making lots of changes it's not really recognizable as that anymore. I'm certainly not a UI/UX expert by any means, but my goal was to create a simple yet aesthetically pleasing interface.

![avojak.com](https://s3.amazonaws.com/blog.avojak.ghost/2018/07/avojak.com.png)

After a lot of trial and error - which could be an entire blog post - on a couple of other hosting platforms ([Firebase](https://firebase.google.com/), [AWS](https://aws.amazon.com/)), I ultimately chose [Heroku](https://www.heroku.com/). Deployment is simple with the pipelines feature: 

1. Push code to GitHub and your application is automatically built into a staging app.
2. When you're satisfied, promote to master.

![avojak.com Heroku Pipeline](https://s3.amazonaws.com/blog.avojak.ghost/2018/05/Capture.PNG)

GitHub: [avojak](https://github.com/avojak/avojak)

## The Blog

I chose [Ghost](https://ghost.org/) as my blogging platform because it's free, it has tons of easy-to-use features right out of the box, it's easily customizable, and there are plenty of hosting options. I started out by self-hosting on a Raspberry Pi, however I came across a handy [GitHub project](https://github.com/cobyism/ghost-on-heroku) which deploys Ghost to Heroku with the click of a button. Since Heroku has a free tier [^1], it was a very easy decision to switch.

![blog.avojak.com](https://s3.amazonaws.com/blog.avojak.ghost/2018/07/blog.avojak.com.png)

Even though Heroku is the primary hosting platform, Ghost still leverages [AWS S3](https://aws.amazon.com/s3/) under the hood to store data, such as the screenshots in this post.

For the UI theme, I started with the default Casper theme, and then forked it to make a few minor customizations.

GitHub: [ghost-on-heroku](https://github.com/avojak/ghost-on-heroku) (A fork), [Casper](https://github.com/avojak/Casper) (Also a fork)

---

[^1]: Even the ["Hobby" tier](https://www.heroku.com/pricing) is very reasonably priced at only $7/month - which I sprung for eventually.