---
title:  "From Laptop to Rack Mount Server"
description: "I converted my old laptop into a rack-mountable server!"
author: avojak
image: https://images.unsplash.com/photo-1558494949-ef010cbdcc31
tags:
  - hardware
  - homelab
  - evergreen
---

Naturally, you're probably thinking "...why?"

![But why?](http://www.reactiongifs.com/r/but-why.gif)

Well, for starters, I don't like waste. I've had an old Lenovo W540 laying around for some time now, and it's a shame to let that processing power go unused. Other than that, I figured "why not?"

So let's get a couple of things out of the way right off the bat:

> Is it powerful?

Not really.

> Is this economical?

No.

> Why not just buy a cheap workstation?

Boring.

## The Guts

- Motherboard: Whatever is inside a Lenovo W540
- Processor: Quad-core Intel i7-4800MQ @ 2.7GHz
- RAM: 16GB
- Storage: 1x 500 GB HDD, 1x 250 GB SSD
- Operating System: VMware ESXi 6.7.0

## Dissection Time!

The first step was to take the laptop *completely* apart to make sure that it would boot without the screen, keyboard, trackpad, etc. attached.

![IMG_1202](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1202.jpeg)

I didn't have very high hopes, but sure enough, it booted right up and I was still able to reach the ESXi login page from my laptop!

Next, I continued my dissection and removed the motherboard completely. I left it attached to the metal base mount because I realized it would simplify things later on when it would be time to mount the motherboard to the server chassis. Rather than needing to measure the clearance from the bottom of the motherboard, buy new screws, nuts, spacers, etc., I could simply re-use the screws attaching the metal mount to the plastic laptop cause. I would eventually cut out the screw mounts from the laptop case and super glue them to the metal server chassis (described more later).

With the board completely out, I did one final boot test to be 100% sure that it would boot up without *any* display connected. 

<figure class="half" markdown="1">

![IMG_1205](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1205.jpeg)
![IMG_1206](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1206.jpeg)

</figure>

As before, it booted right up and the little green LED from the power button lit up.

## The Chassis
The next step was to measure the motherboard and find an empty server chassis. Fortunately I found one that fit just right: [1U 19" Rack Mount Enclosure - 300mm Deep Steel Chassis](https://www.circuitspecialists.com/rackmount-enclosure-37-1u.html). When it arrived, I was able to easily take it apart and (gently) drop the motherboard right in:

![IMG_1216](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1216.jpeg)

## Internal Layout
After staring at it for a long time, I decided to leave the board right where it was for two main reasons:

1. The fans on the motherboard vent towards the back and to the side, and I would rather cut fan holes in the rear of the chassis than the front. Also the top of the chassis has holes for airflow which lined up right above the fan intake.
2. The plug for power is already in the rear.

I did a bit of millimeter shuffling of the board to make sure that the hard drives could be easily removed, but where you see it in the picture above is pretty much where it stayed.

## The Power Button
The first major hurdle was the power button. I *really* wanted to re-use the power button. Partially because I knew it would work, but also to give the end product a similar feel to the original laptop. Unfortunately the ribbon cable for the power button was far too short to reach the front of the chassis. 

![IMG_1265](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1265.jpeg)

I did a bunch a searching for longer cables with the right spacing (pitch) and number of connectors, but to no avail. After watching a YouTube video, I got the crazy idea that I could make my own ribbon cable! The premise was simple: buy a cable with the same pitch (but more connectors), and then cut it down to size.

To give myself some room to work with, I used a knife to peel away one of the connectors. Then I cut it down to size and added some notches to roughly match the original.

<figure class="third" markdown="1">

![IMG_1266](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1266.jpeg)
![IMG_1267](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1267.jpeg)
![IMG_1269](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1269.jpeg)

</figure>

Unfortunately the connector to the motherboard is on the bottom of the board, so I had to feed it through a small opening in the metal mount. Once it was plugged in, I hit the button and... it didn't work...

Fortunately I anticipated this and had plenty of extra ribbon cable to try again. On the second attempt, I hit the power button, and voilà!

![IMG_1273](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1273.jpeg)

## Cable Management
The path of least resistance for cable management seemed to be "buy a bunch of panel mount cables and screw them to the chassis". So I did some rough measurements, figured out which cables needed right-angle plugs in order to fit, and went on a shopping spree.

The board has two video outputs: VGA and Mini DisplayPort. It was nearly impossible to find a small enough angled VGA plug to fit the space, so I ended up deciding that a DisplayPort plug would have to do.

After purchasing a couple of additional cables to meet the lengths, and some tiny fans to help with airflow, it was time to do a dry fit!

## Complete Internal Layout
All-in-all, in the rear there would be:

- 1x USB 3.0
- 1x DisplayPort
- 1x power connection (TBD)
- 2x USB-powered fans

And in the front:

- 1x RJ45 network port
- 1x power button

![62758359850__3386AD86-1DF9-45B1-B09E-1042AE2614C4](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/62758359850__3386AD86-1DF9-45B1-B09E-1042AE2614C4.jpeg)

It's a tight fit, and it looks messy, but it works!

![](https://i.pinimg.com/originals/0b/d7/63/0bd763368ca4302124f4604c047032f9.jpg)

## Measure Twice, Cut Once

It was finally time to start cutting into the laptop case and the server chassis. I picked up new drill bits meant to cut into steel. I wasn't sure how hard it would be, so I also bought some 3-in-1 oil just in case, but ended up not needing it.

The holes for the fans came first:

![IMG_1382](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1382.jpeg)

It was a bit rough at first, but that's why I started with the back panel! 

Finally I cut out the opening for the power button. Due to the thickness of the front panel it was a bit annoying trying to poke my finger into the hole for the power button, so I went back to the hardware store and picked up a countersink bit to smooth it out:

![IMG_1391](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1391.jpeg)

For the larger openings I made a series of small holes, followed up with some dremmel work to even things out. It didn't look the cleanest, but it worked.

To mount the motherboard, I cut out the plastic standoffs which the motherboard was screwed into through the metal base. I forgot to take pictures of this process, but the general idea was to re-use the screws and standoffs. This meant that I could level mount the motherboard without needing to put screws through the bottom of the chassis - that might scratch anything beneath it! I did the same process for mounting the small board for the power button as well.

![Screen-Shot-2020-11-24-at-5.02.18-PM](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/Screen-Shot-2020-11-24-at-5.02.18-PM.png)

## Assembly
With all the openings cut I gave the chassis a fresh coat of spray paint to cover up the rough cuts. Once the paint was dried I glued the power button in place and let it set while assembling the rest:

![IMG_1394](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1394.jpeg)

The fans were next. I used some plastic spacers between the rear panel and the fan case to help eliminate any rattle. I also added a bit of Loctite Blue to the screws.

<figure class="half" markdown="1">

![IMG_1393](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1393.jpeg)
![IMG_1400](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1400.jpeg)

</figure>

I took a break for the evening to spray paint all the screws. I didn't think it would bother me to have silver screws against the black chassis (especially on the rear), but it did...

<figure class="half" markdown="1">

![IMG_1401](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1401.jpeg)
![IMG_1402](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1402.jpeg)

</figure>

For the power cable I decided to use the original charger but make it permanently attached. This may have been a mistake, but I didn't feel like messing with different power adapters. I fed the cable through the larger opening that I cut and then used a pair of rubber grommets and some superglue to seal up the hole and keep the cable in place:

<figure class="half" markdown="1">

![IMG_1405](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1405.jpeg)
![IMG_1408](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1408.jpeg)

</figure>

## Mission Accomplished
And just like that, it was done! 

<figure class="third" markdown="1">

![IMG_1411](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1411.jpeg)
![IMG_1412](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1412.jpeg)
![IMG_1413](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1413.jpeg)

</figure>

I plugged it in, and just like before, it powered right up!

<figure class="half" markdown="1">

![IMG_1417](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1417.jpeg)
![IMG_1416](https://s3.amazonaws.com/blog.avojak.ghost/2020/11/IMG_1416.jpeg)

</figure>

This was a fun project to work on, although it certainly had its challenges. The smell of melting plastic while its being cut, metal shavings all over, cables that aren't quite long enough, figuring out the ribbon cable, etc. My only concern at this point is airflow and dust. I may need to apply some sort of screen to the holes on the top of the chassis to prevent dust buildup, and I'm a little worried that even with the little helper fans it won't cool off enough. Only time will tell.

I'm also not quite sure if I really succeeding in reducing waste - I not have a box of laptop parts that went unused in the process, including the screen, keyboard, etc. 

BUT it was fun, and now all that's left is to figure out what to run on it!