---
title:  "New Website!"
description: "Redesigned website and blog built around Jekyll and GitHub Pages"
author: avojak
image: https://images.unsplash.com/photo-1486312338219-ce68d2c6f44d
tags:
  - software
---

I just completed a redesign of my website based around Jekyll and GitHub Pages! The driving factor here
was cost. Until now I had been paying $14/month for hosting on Heroku ($7 each for my homepage and my blog).
In addition to the cost, making changes was somewhat challenging. I rolled my own template for project
pages, and making changes was a bit tedious. 

In comes the elementary OS blog template: [elementary/blog-template](https://github.com/elementary/blog-template). Big thanks to [Cassidy](https://cassidyjames.com) from the elementary OS team for pointing
me to their GitHub repo a couple months ago!

## The Blog

I really enjoy the clean look of the elementary OS blog, as well as the
model for creating new posts: simply create a new markdown file and push
it to GitHub! Doesn't get much easier than that.

I made a few changes to removing any elementary OS branding, tweak the items in the navbar, change the footer, fix the positioning of an arrow
symbol at the bottom of the page, etc. but the vast majority stayed the same.

Importing the old content was as easy as copy-paste. I had been using
Ghost as a blogging platform, and it too used markdown files.

I created an account with [Cusdis](https://cusdis.com) to handle comments.
This is a change from Disqus which I was using before. I really like
the clean look of Cusdis, and the fact that it doesn't require users
to login to a 3rd party service to leave comments. Bonus: it supports
GitHub markdown! There's currently a bug which prevented automatic
importing of the Disqus comments, but doing this manually took just
a few minutes - there weren't many!

## The Homepage

To create the homepage, about page, and project pages, I copied my blog repository and made minor alterations to display project pages instead of
blog post pages. Under the hood, the mechanism is identical. I also made
some minor changes to use "Categories & Technologies" labels instead of
"tags", along with a change in color.

I also created a new "cover" page layout that would allow the page content to expand to fill available space. This was particularly useful
for the root page which simply displays my name and photo.

As with blog posts, create a new project page is as simple as drafting
a new markdown file.

## GitHub Pages

Setting up GitHub pages was very easy until I ran into a few hiccups near the end.

I started out by moving my homepage code into a new `avojak.github.io` repository as is expected for GitHub pages to serve content for an apex
domain (in this case `avojak.com`). I then made the required changes
to my Cloudflare DNS configuration. Within moments, it was live!

Setting up the blog was where I started to run into issues. I wanted
to keep the blog and homepage repositories separate to simplify the
way Jekyll handled project and blog content. Trying to do pagination
and whatnot for both within the same project seemed too complicated.

Originally I wanted to continue serving my blog from `blog.avojak.com`,
however because I was already using GitHub Pages for `avojak.com`, this
was not possible. Instead, GitHub Pages would allow me to create
`avojak.com/blog`. Unfortunately this means that any links to my old
blog are now dead, but I don't think anyone linked to it anyway,
so it's a small price to pay.

This did complicate the Jekyll side a bit, and I had to make some minor
tweaks to the blog configuration to fix pagination. In some cases it
was attempting to direct users to `avojak.com/archive/2` instead of
`avojak.com/blog/archive/2`, etc. In other places I ended up with
links to `avojak.com/blog/blog/archive/2`, etc. In the end I managed
to hack together something that works!

## Drawbacks

There are only two drawbacks that I've encounted during this entire process:

1. Duplication of CSS

    At the end of the day, this is hardly a drawback because from here on out it *should* be pretty much constant.

2. Weirdness with GitHub Pages

    Now that it's working, I just won't touch it!

## Bottom-line

This has been a fantastic transition, and I'm very happy with how easy it was to setup! Plus, the low low price of $0 - that's tough to beat!