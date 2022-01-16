---
title:  "Review of the LinkedIn URL Detector"
description: " "
author: avojak
image: https://images.unsplash.com/photo-1550645612-83f5d594b671
tags:
  - software
---

Detecting URLs in a body of text is notoriously difficult[^1]. Does it start with `http`? `https`? Or no protocol? How about `www`? Maybe just the domain name? And did you know that parenthesis can be present in a URL[^2]? Me neither.

Recently I had the need to scrape a handful of webpages for text data, then find any URLs within that data and scrape those pages as well. I came across the Java [URL Detector](https://github.com/linkedin/URL-Detector) library developed by the LinkedIn Security Team and decided to see how it compares to another library ([autolink-java](https://github.com/robinst/autolink-java)), as well as good ol' fashioned regex.

Before getting too far into the details it's worth noting the motivations behind LinkedIn's URL Detector. From their GitHub page:

> Keep in mind that for security purposes, its better to overdetect urls and check more against blacklists than to not detect a url that was submitted. As such, some things that we detect might not be urls but somewhat look like urls. Also, instead of complying with [RFC 3986](http://www.ietf.org/rfc/rfc3986.txt), we try to detect based on browser behavior, optimizing detection for urls that are visitable through the address bar of Chrome, Firefox, Internet Explorer, and Safari.

We'll see later that this does indeed cause an inflated number of results (which may or may not be useful depending on your use-case).

## Using the URL Detector

It takes no more than a couple of lines to use the detector, with only the most basic configuration required:

```java
UrlDetector parser = new UrlDetector("hello this is a url Linkedin.com", UrlDetectorOptions.Default);
List<Url> found = parser.detect();

for(Url url : found) {
    System.out.println("Scheme: " + url.getScheme());
    System.out.println("Host: " + url.getHost());
    System.out.println("Path: " + url.getPath());
}
```

There are several detector options ([`UrlDetectorOptions`](https://github.com/linkedin/URL-Detector/blob/master/url-detector/src/main/java/com/linkedin/urls/detection/UrlDetectorOptions.java)) available: `Default`, `QUOTE_MATCH`, `SINGLE_QUOTE_MATCH`, `BRACKET_MATCH`, `JSON`, `JAVASCRIPT`, `XML`, `HTML`, and `ALLOW_SINGLE_LEVEL_DOMAIN`. For this review we will be using the `HTML` option, because we are interested in scraping URLs from raw HTML content.

## The Testing

The LinkedIn URL Detector will be compared against the [autolink-java](https://github.com/robinst/autolink-java) library, as well as a regular expression[^3]:

```
(?:^|[\\W])((ht|f)tp(s?):\\/\\/|www\\.)(([\\w\\-]+\\.){1,}?([\\w\\-.~]+\\/?)*[\\p{Alnum}.,%_=?&#\\-+()\\[\\]\\*$~@!:/{};']*)
```

Naturally different libraries return results in different formas, so in order to make a reasonable comparison between each tool, the timing metric will consider all steps required to detect URLs from a string, and return a list of string URLs. Loading the text data and initializing the detectors is not considered. 

Each detector is run against two datasets: a large HTML document (~30,000 lines) and a short HTML document (~5,000 lines). A total of 100 iterations will be run against each dataset to find an average runtime. We will also take a close look at the URLs returned by the detectors for each dataset.

All test code can be found in the [url-detector-eval](https://github.com/avojak/url-detection-eval) repository I've created on GitHub.

## The Results

```
**** Long Web Page ****
LinkedIn: 2398 URLs in an average of 60 ms
Autolink: 1169 URLs in an average of 8 ms
Regex:    1166 URLs in an average of 116 ms

**** Short Web Page ****
LinkedIn: 289 URLs in an average of 11 ms
Autolink: 162 URLs in an average of 1 ms
Regex:    155 URLs in an average of 29 ms
```

On the surface, the LinkedIn detector appears superior. While taking ~8-10x longer than the Autolink library, it appears to identify almost *twice* as many URLs. The regex approach seems to fall far behind, taking twice as long as the LinkedIn detector, yet identifying the fewest URLs. That said, let's take a look at the URLs themselves.

In [`long-out.txt`](https://github.com/avojak/url-detection-eval/blob/master/long-out.txt) and [`short-out.txt`](https://github.com/avojak/url-detection-eval/blob/master/short-out.txt) we can see a complete list of the URLs that were detected in the long and short datasets respectively, as well as which detector identified them.

Several differences jump out immediately when looking at the results:

1. The LinkedIn detector will identify URLs within `src` attributes in the HTML (e.g. `//emergency.webservices.illinois.edu/illinois.js` from `src="//emergency.webservices.illinois.edu/illinois.js"`)
2. The LinkedIn detector returns lots of false-positives, especially around resource files (e.g. `http://bootstrap.css/` from within a large JQuery statement)
3. The LinkedIn detector will append a `/` at the end of URLs even if it isnt present in the original content (e.g. `http://my.cs.illinois.edu` becomes `http://my.cs.illinois.edu/`)

### URLs Within `src` Attributes

```html
<script type="text/javascript" src="//emergency.webservices.illinois.edu/illinois.js"></script>
```

In my opinion this is one of the greatest advantages to the LinkedIn detector. These are clearly valid references to resource files - a clear miss on the part of the other methods.

### The False-positives

```javascript
document.querySelector('img.gsc-branding-img').alt = "Google";
```
```
http://img.gsc-branding-img/
```

On the flipside, we can see plenty of instances where the LinkedIn detector is tripped-up simply by the presence of a period. If your use-case needs to be precise, this is something to be aware of. To be fair, as I mentioned above the decision to over-detect is something that is clearly documented.

### URL Manipulation

```
http://my.cs.illinois.edu
http://my.cs.illinois.edu/
```

This final difference is barely worth mentioning, as it can easily be overcome with minimal effort. Once again, however, depending on your use-case you may need to identify *exact* matches.

## Summary

The LinkedIn URL Detector is powerful and much more thorough than other methods. If over-detection negatively affects your use-case, you will want to spend some time testing the various detection options to tune the detector.

---

[^1]: https://blog.codinghorror.com/the-problem-with-urls/
[^2]: https://www.ietf.org/rfc/rfc3986.txt
[^3]: https://stackoverflow.com/a/5713866/3300205