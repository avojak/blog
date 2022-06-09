---
title: "Libsemver"
description: "A short exercise in library development and unit testing in Vala"
author: avojak
image: https://images.unsplash.com/photo-1556917452-890eed890648
hidden: true
tags:
  - software
  - evergreen
---

As I've been developing apps in Vala and GTK for a few years now, there have been two questions nagging at me the entire time:

1. How libraries are written in Vala?
2. How do you properly unit test code written in Vala?

Frankly the second question is one that I *should* have investigated from the start, but better late than never, no time like the present to learn, and other adages.

To learn by doing, I settled on writing a library to handle Semantic Versions that I aptly named `libsemver`:

{% include github-card.html
  user="avojak"
  repository="libsemver"
%}

## How Are Libraries Written in Vala?

## How Do I Properly Write Unit Tests in Vala?

Writing unit tests with GLib Test feels a bit cludgy.

1. Poor feedback from failed assertions (needs better messaging)
2. Poor visibility as to which test case failed (output is only on a class level, so if a class has multiple assertions they all get globbed together)

## What's Next?

### Better Unit Testing

As a next step, I would really like to spend some time digging into a project that I recently discovered called Valadate, which is a testing framework built on top of GLib Test:

{% include github-card.html
  user="chebizarro"
  repository="valadate"
%}

Although it is no longer maintained, I've forked the project and hope that it can still be of use.

### Deployment of Documentation

Although the library is currently configured to generate a documentation site using Valadoc, it's not deployed anywhere. I would like to explore some options, including leveraging GitHub Pages for static hosting. One challenge here will be managing multiple versions of the documentation (e.g. `1.0.0` vs. `latest`).