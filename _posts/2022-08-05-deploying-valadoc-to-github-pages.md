---
title: "Deploying Valadoc to GitHub Pages"
description: "How I automatically deploy Valadoc documentation to GitHub Pages"
author: avojak
image: https://imgur.com/7UvOE2L.png
tags:
  - evergreen
  - software
---

I recently wrote a few libraries in Vala and was struggling to make the [Valadoc](https://wiki.gnome.org/Projects/Valadoc)
easily accessible to users. I found several examples of developers generating the docs locally and simply checking them
into version control in a `docs/` directory. This doesn't appeal to me, as documentation like this - in my opinion - is a 
product of a successful build and should be hosted separately.

After some digging I came across a GitHub Action for deploying to GitHub Pages. This seemed like a great fit because it will
easily integrate with my existing CI process.

## Preparation

### Generate Valadoc

The first thing to do is ensure that your library will generate Valadoc documentation. I'm not going to go into the details
of that here, but at the bottom of the post you will find a link to an example repository of mine that you can use for
reference.

In most cases you will end up with a pair of commands that look something like this:

```bash
meson build -Ddocumentation=true
ninja -C build
```

Et voilà! There's documentation in the `build/doc/` directory!

### Enable GitHub Pages

The next step is to create a separate *empty* branch on your repository named `gh-pages`. Simply create a new
branch, delete all the files, and push.

Now you'll want to enable GitHub Pages for your repository. For this, you can refer to GitHub's documentation: [Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

## Setup the Action

Now it's time to create the GitHub action! If you don't already, create a `.github/workflows/` directory in the root of
your repository. Then, create a YAML file (e.g. `ci.yml`, or `valadoc.yml`).

I already have a `ci.yml` file here with jobs to build and execute unit tests, so I'm omitting the irrelevant jobs below. I wanted
to only publish the documentation if the build and tests were successful, so I put everything in a single file. You don't need
to do this if you don't want to.

My `ci.yml` file looks like this:

{% raw %}
```yaml
name: CI
on: [pull_request, push]
permissions:
  contents: write
jobs:
  # ... other jobs omitted...
  Valadoc:
    name: Deploy Valadoc
    runs-on: ubuntu-latest
    needs: Test
    container: 
      image: elementary/docker:unstable
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Install dependencies
        run: apt-get update && apt-get -y install libvala-dev valac meson git
      - name: Build
        run: meson build -Ddocumentation=true && ninja -C build
      - name: Deploy
        if: ${{ github.ref == 'refs/heads/master' }}
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build/doc/catalyst-1-vala
```
{% endraw %}

The first three steps simply checkout the source code, install some dependencies, and build the documentation using the
same command described previously.

The fourth step is the real magic. I have an `if` check to only publish documentation from the `master` branch. You can
remove this or change the branch name to another branch if you'd like. Then we include the `peaceiris/actions-gh-pages@v3`
action. We don't need to re-invent the wheel!

We provide two variables - `github_token` and `publish_dir`. The token will allow the build to have write access to publish
to the `gh-pages` branch. The `publish_dir` points to the specific directory that will be published. In my case I have a
specific subdirectory within `build/doc/` that I want to publish.

And that's it!

You can find complete documentation for the `peaceiris/actions-gh-pages` action on GitHub: [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages).

## End Result

When it's all said and done you'll have publically-accessible documentation hosted for free via GitHub Pages. I have
a custom domain setup for my GitHub pages (the same one you're looking at right now!) so you can view an example over at:
[https://avojak.com/libcatalyst/catalyst-1/](https://avojak.com/libcatalyst/catalyst-1/).

And no project like this would be completely without a shiny new badge for your README:

```markdown
[![Documentation](https://img.shields.io/badge/documentation-valadoc-a56de2)](https://avojak.com/libcatalyst/catalyst-1/)
```

[![Documentation](https://img.shields.io/badge/documentation-valadoc-a56de2)](https://avojak.com/libcatalyst/catalyst-1/)

You can see a full example project using this exact process over on my GitHub:

{% include github-card.html
  user="avojak"
  repository="libcatalyst"
%}