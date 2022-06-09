---
title: "GitHub Cards"
description: "Introducing GitHub cards for my Jekyll blog"
author: avojak
image: https://images.unsplash.com/photo-1618401471353-b98afee0b2eb
tags:
  - software
  - evergreen
---

If you've been following my blog for a while, or saw my post from January, you'll know that I recently re-wrote my website and blog to use GitHub Pages and Jekyll.

<aside>
  <a class="featured with-image 1" href="https://avojak.com/blog/2022/01/19/new-website/">
    <div class="featured-image" alt="Featured image" style="background-image: url(https://images.unsplash.com/photo-1486312338219-ce68d2c6f44d);"></div>
    <header>
      <h2>New Website!</h2>
      <h3>Redesigned website and blog built around Jekyll and GitHub Pages</h3>
      <div class="byline">
        <div class="avatar">
          <img srcset="https://www.gravatar.com/avatar/4dfa9066e794647fdedf323ecab85333?s=96&amp;d=blank 2x" src="https://www.gravatar.com/avatar/4dfa9066e794647fdedf323ecab85333?s=48&amp;d=blank" alt="Avatar for Andrew Vojak">
        </div>
        <div class="author">
          <span class="name">Andrew Vojak</span>
        </div>
        <time class="post-date" datetime="2022-01-19">Wed, Jan 19, 2022</time>
        <span class="read-time" title="Estimated read time">4 min read</span>
      </div>
    </header>
  </a>
</aside>

The basis for the new design was the [elementary OS Blog](https://blog.elementary.io):

{% include github-card.html
  user="elementary"
  repository="blog-template"
%}

Within their template they provide beautifully-designed "cards" to format external content in a way that fits the style of the blog. There
are cards for YouTube and Twitter, but as you just saw above, I recently created my own to support GitHub projects.

The primary motivation was that I simply don't like the way GitHub's Open Graph images look within the context of my blog:

<figure class="constrained" markdown="1">
![Open Graph image for Warble](https://opengraph.githubassets.com/1/avojak/warble)
</figure>

They aren't *bad*, but they just don't really fit...

One of the original design tenants of the cards, especially the Twitter card, was to not rely on external sources for data. In other words, don't
link directly to Twitter to retrieve the content of the tweet. This is great, however for a GitHub card I really wanted to be able to display
an accurate number of open issues, stars, and forks, which isn't possible or feasible to do manually.

Instead, I opted to leverage the GitHub API to retrieve this information:

```javascript
fetch('https://api.github.com/repos/{{ fullname }}')
  .then(res => {
    if (!res.ok) {
      throw new Error("Bad network response");
    }
    return res.json();
  })
  .then((out) => {
    var stargazers = out['stargazers_count']
    var issues = out['open_issues_count']
    var forks = out['forks_count']
    
    // Populate the elements with the data from the response
    document.getElementById("{{ fullname }}-stargazers").innerHTML 
      += numeral(stargazers).format('0.[0]a') + " Star" + (stargazers != 1 ? "s" : "");
    document.getElementById("{{ fullname }}-issues").innerHTML
      += numeral(issues).format('0.[0]a') + " Issue" + (issues != 1 ? "s" : "");
    document.getElementById("{{ fullname }}-forks").innerHTML
      += numeral(forks).format('0.[0]a') + " Fork" + (forks != 1 ? "s" : "");
    document.getElementById("{{ fullname }}-description").innerHTML = out['description'];
}).catch((err) => {
  console.error(err);
  // Clear out the fields in case of error rather than showing 0 values
  document.getElementById("{{ fullname }}-stargazers").innerHTML = "";
  document.getElementById("{{ fullname }}-issues").innerHTML = "";
  document.getElementById("{{ fullname }}-forks").innerHTML = "";
});
```

The downside is that there is a rate limit enforced on the client-side (I think 60 requests per 15 minutes, or something like that), but it's
high enough that unless you're constantly refreshing my blog pages, you probably won't hit it.

If you do, then it will simply display basic information with a link to GitHub:

{% include github-card.html
  user="avojak"
  repository="super-cool-project"
%}

*The effect is the same when you link to a non-existent project.*

# What's Next?

The only other feature that would be nice to have is an indication of the primary programming language used. I believe this information is available
from the API, but I need to figure out how to display it cleanly, especially on a narrower display like a phone.

Anyway, it's a nice touch to have something a bit better than boring URLs listed in a blog post! Check out the full implementation over on
the GitHub repository for the blog:

{% include github-card.html
  user="avojak"
  repository="blog"
%}