---
title: Twitter “Embeds” While Respecting Privacy
description: A demo!
author: cassidyjames
---
Check it out:

```liquid
{% raw %}{% include twitter-card.html
  name="Cassidy James Blaede"
  account="CassidyJames"
  avatar="https://gravatar.com/avatar/41275ecc8271aca852ce2c0ff72d2610?s=128"
  id="1281816033343537152"
  timestamp="2020-07-10"
  contents="Got my PINEBOOK Pro! Super duper first impressions: damn, this
    hardware quality is nice for the price. I would love to see what they could
    do around the $500–750 price, honestly. Performance is great; I would not
    guess it was ARM just from using it. It’s fanless!"
%}{% endraw %}
```

Turns into:

{% include twitter-card.html
  name="Cassidy James Blaede"
  account="CassidyJames"
  avatar="https://gravatar.com/avatar/41275ecc8271aca852ce2c0ff72d2610?s=128"
  id="1281816033343537152"
  timestamp="2020-07-10"
  contents="Got my PINEBOOK Pro! Super duper first impressions: damn, this
    hardware quality is nice for the price. I would love to see what they could
    do around the $500–750 price, honestly. Performance is great; I would not
    guess it was ARM just from using it. It’s fanless!"
%}

It's even light/dark style aware!