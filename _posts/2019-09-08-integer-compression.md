---
title:  "Integer Compression"
description: "It never occurred to me that there would be a way to compress an integer..."
author: avojak
image: https://images.unsplash.com/photo-1531171378784-437df0400ec1
tags:
  - algorithm
  - evergreen
---

I recently learned about three methods of encoding integer values:
1. [Unary](https://en.wikipedia.org/wiki/Unary_coding)
2. [&gamma; (Elias gamma coding)](https://en.wikipedia.org/wiki/Elias_gamma_coding)
3. [&delta; (Elias delta coding)](https://en.wikipedia.org/wiki/Elias_delta_coding)

This sort of blew my mind because I never thought about trying to take a basic, primitive type (such as an integer) and store it using _less_ disk space. 

In a simple example, let's assume a basic signed integer in C which takes 16 bits. If we were to save the number 3, we would be storing the following bit sequence:

`00000000 00000011`

BUT, we can apply one of the methods above and actually use less than 16 bits. Let's take a look at each method.

*(It's important to note that in all scenarios in this post we will be using log base 2)*

## Unary encoding

Unary encoding is by far the simplest. For an integer $$x$$ where $$x \geq 1$$, the unary encoding is $$x-1$$ 1s, followed by a 0.

For example, the number 3 will be two 1s, followed by a 0:

`110`

And the number 9 will be eight 1s, followed by a 0:

`111111110`

For small numbers this is great - we've saved 13 bits and 7 bits respectively in these examples. However, you'll likely notice that as the number grows larger, we end up taking _more_ space than we would have by just storing the integer itself.

## &gamma;-encoding

For &gamma;-encoding we need to start doing some math.

We start by finding the unary code for $$1+\lfloor log x \rfloor$$. So, for the number 3 we end up with:

$$1+\lfloor log 3 \rfloor = 2$$, which in unary code becomes:

`10`

Next, we find the uniform (binary) code for $$x-2^{\lfloor log x \rfloor}$$, but represented in $$\lfloor log x \rfloor$$ bits:

$$3-2^{\lfloor log 3 \rfloor} = 3-2^1 = 1$$, which when represented in $$\lfloor log 3 \rfloor = 1$$ bit is:

`1`

Piecing the two parts together (unary part first, then the uniform), we get:

`101`

Definitely a savings compared to storing the raw value of 3 in 16 bits, but there is no gain over the unary encoding. What about the number 9?

$$1+\lfloor log 9 \rfloor = 4$$, which in unary code becomes:

`1110`

$$9-2^{\lfloor log 9 \rfloor} = 9-2^3 = 1$$, represented in $$\lfloor log 9 \rfloor = 3$$ bits is:

`001`

Piecing the two parts together (unary part first, then the uniform), we get:

`1110001`

A 2-bit savings over the unary encoding! As the numbers grow, so do the savings.

For example, 15 in unary-encoding:

`111111111111110`

...and in &gamma;-encoding:

`1110111`

## &delta;-encoding

The &delta; encoding is more complicated. It is nearly identical to &gamma;-encoding, however instead of starting with the unary encoding of $$1+\lfloor log x \rfloor$$, we start with the _&gamma;-encoding_ of $$1+\lfloor log x \rfloor$$. This can get tricky to keep track of.

Let's use the number 3 again for our first example. We start with the &gamma;-encoding of $$1+\lfloor log x \rfloor$$:

1+&LeftFloor;log 3&RightFloor; = 1+1 = 2, so we find the &gamma;-encoding of the number 2. Following the steps in the section above, you will see that the &gamma;-encoding is:

`100`

The rest is identical to the &gamma;-encoding. We find the uniform (binary) code for $$x-2^{\lfloor log x \rfloor}$$, but represented in $$\lfloor log x \rfloor$$ bits:

$$3-2^{\lfloor log 3 \rfloor} = 1$$, represented in $$\lfloor log 3\rfloor = 1$$ bit is simply:

`1`

Piecing the two parts together (&gamma;-encoding first, then the uniform):

`1001`

Yikes! This took 4 bits instead of the 3 bits used by the &gamma;-encoding! What about a larger number like 125?

We start with the &gamma;-encoding of $$1 + \lfloor log x \rfloor$$:

$$1 + \lfloor log 125 \rfloor = 7$$, represented in &gamma;-encoding is:

`11011`

Next we find the uniform (binary) code for $$x - 2^{\lfloor log x \rfloor}$$, but represented in $$\lfloor log x \rfloor$$ bits:

$$125-2^{\lfloor log 125 \rfloor} = 125 - 2^6 = 61$$, represented in binary in $$\lfloor log 125 \rfloor = 6$$ bits is:

`111101`

Piecing the two parts together (&gamma;-encoding first, then the uniform):

`11011111101`

On the other hand, the &gamma;-encoding for 125 is:

`1111110111101`

A 2-bit savings!

## Summary

Long story short, I though these were some pretty cool methods for encoding integers. I won't write about it here, but you can just as simply decode these bit strings. Actually, I won't lie, it's not quite as simple.

Here is a link to a GitHub repo where I've put together a couple Python scripts to perform both the compression and decompression techniques:

{% include github-card.html
  user="avojak"
  repository="integer-compression"
%}