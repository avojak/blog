---
title:  "Raspberry Control Tower"
description: "Raspberry Pi + Apple AirPort Extreme + UniFi Controller = Raspberry Control Tower?"
author: avojak
image: https://images.unsplash.com/photo-1556388158-158ea5ccacbd
tags:
  - hardware
  - raspberry-pi
  - evergreen
---

About six months ago I started picking up some Ubiquiti network equipment for my home setup. Since then I had been running the management software (UniFi Controller) as-needed from my laptop. I'm a sucker for fancy charts and metrics, so I wanted a persistent controller that was always monitoring my home network. The expensive option is to use a CloudKey or the fancy Dream Machines that Ubiquiti sells. For cheapskates like myself, you can run the same software on a budget Raspberry Pi!

There are plenty of cool cases for Raspberry Pis on the market, but I'm all about the aesthetic (even if this will end up sitting on a shelf in the closet). Fortunately I came by an old, broken Apple AirPort Extreme. Naturally, putting one and two together I got three and here we are with a project to mount my UniFi Controller Raspberry Pi in an old Apple AirPort Extreme. I call it... the ***Raspberry Control Tower***!

## Gutting the AirPort Extreme

The first step was to gut the old AirPort Extreme. This was fairly easy, and in the end I was left with the white plastic top and the grey plastic base. All the metal parts and the logic board were removed.

<figure class="half" markdown="1">

![IMG_1818](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_1818.jpeg)
![IMG_1912](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_1912.jpeg)

</figure>

Originally I wanted to boot from an SSD instead of the SD card, but quickly realized there wasn't enough room.

## Power

Power-over-Ethernet (PoE) was an obvious choice here, because it meant fewer cables inside the case, and fewer plugs that I would have to mount. At first I tried to solder my own, bare, RJ45 jack to a board and wires due to size constraints in the case. However this failed miserably. The power was stable, but the network connection was very sporadic. In the end, I opted for a panel-mount cable and then cut off a bunch of excess plastic.

<figure class="half" markdown="1">

![IMG_2172](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2172.jpeg)
![IMG_2174](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2174.jpeg)

</figure>

I wanted to include a power button so that I could gracefully shutdown the Raspberry Pi without needing to SSH to it. I found some tiny momentary power buttons from Adafruit and would fit in the old power adapter opening in the case with some slight shaving. A bit of superglue keeps the button perfectly in place.

<figure class="half" markdown="1">

![IMG_2204-1](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2204-1.jpeg)
![IMG_2185](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2185.jpeg)

</figure>

## The Status LEDs

To emulate the behavior of the original AirPort Extreme, I used two LEDs - one yellow/orange/amber and one green. There's probably a simple way to do this with a single color-changing LED, but this was easiest for me.

Using the GPIO pins and some resistors, I hooked up the amber LED to light up when there is power to the Raspberry Pi. The green LED was connected to different GPIO pins to turn on when the operating system is booted. Because the Pi is powered via Power-over-Ethernet (PoE), it's useful to distinguish between power available vs. powered-on and running. Now, the amber LED doesn't turn off when the OS is booted, but the green light far outshines the amber so that you can't even tell it's still on.

![IMG_1956](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_1956.jpeg)

## Putting it All Together!

It's a bit of a mess on the inside, but it works! Plus the front looks perfectly clean, and that's what I'll be looking at.

<figure class="third" markdown="1">

![IMG_2218](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2218.jpeg)
![IMG_2220](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2220.jpeg)
![IMG_2222](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2222.jpeg)

</figure>

## Finished Product

<figure class="third" markdown="1">

![IMG_2221](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2221.jpeg)
![IMG_2224](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2224.jpeg)
![IMG_2226](https://s3.amazonaws.com/blog.avojak.ghost/2021/08/IMG_2226.jpeg)

</figure>