---
title:  "My elementary OS Desktop Build"
description: "First Linux workstation build"
author: avojak
image: https://s3.amazonaws.com/blog.avojak.ghost/2020/09/395198E4-4DAE-4F88-9378-70806C1615EF_1_105_c.jpeg
tags:
  - hardware
  - elementary-os
  - evergreen
---

I just built my first PC! It's something I've wanted to do for a long time, but never had a good reason until now. My primary device is a MacBook Pro, however I've been doing a lot of development in my free time on an app[^1] for [elementary OS](https://elementary.io) and decided it would be beneficial to develop on a dedicated machine. As great as VMware Fusion is, there will always be some amount of lag when using a virtual machine.

## The Parts

When deciding on which parts to buy, my three main priorities were:

1. Good-enough performance
2. Flexibility
3. Aesthetic 😎

My plan for the short-term was a decent PC for development. I won't be editing video, rendering, etc., so I don't need top-of-the-line performance. Naturally my plans may change, so I also wanted some amount of flexibility. Not cutting too many corners was important in case I wanted to turn the box into a home server. Finally, aesthetic - this thing will be sitting on the top of my desk, so it has to look good!

- Motherboard: [ASUS Prime H310T R2.0/CSM](https://www.newegg.com/p/N82E16813119185?item=9SIAN0MBAR1408)
- Processor: [Intel i5-9400 (6 Cores up to 4.1 GHz)](https://www.amazon.com/gp/product/B07MGZ9FJZ)
- Memory: 1x [Crucial 16GB 260-Pin DDR4 2666](https://www.newegg.com/crucial-16gb-260-pin-ddr4-so-dimm/p/1X5-001S-002N6?Item=9SIAKRHB566338)
- Storage: 500GB Samsung EVO 850 (from my desk drawer)
- Case: [LUNA Design DNK-H (Cool Gray)](https://luna-design.org/en_dnk-h)

## The Case

Honestly, the case was the driving part for everything else. I stumbled across it while researching Mini-ITX builds but it was very difficult to find anyone who had actually used it in a build. The one video I found on YouTube focused on an older 2015 model from LUNA Design.

I wasn't quite sure what to expect when ordering something from overseas (LUNA Design is based in Russia), but I had a fantastic experience. My order was processed immediately during their first business day after ordering, and they were quick to respond to my email correcting the shipping address. Their website warns about potentially slow shipping due to COVID-19, but I received my order about two weeks after ordering, which was faster than their estimated duration.

<figure class="half" markdown="1">
![46D62397-C416-485C-8A9C-DF59E0DE37F8_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/46D62397-C416-485C-8A9C-DF59E0DE37F8_1_105_c.jpeg)
![BC169947-45C1-4D64-A79E-408639456DB9_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/BC169947-45C1-4D64-A79E-408639456DB9_1_105_c.jpeg)
</figure>

<figure class="half" markdown="1">
![CA2ACB24-67B4-4EB4-BACA-4C0600113E01_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/CA2ACB24-67B4-4EB4-BACA-4C0600113E01_1_105_c.jpeg)
![3C6F502D-D7C8-4D07-ABB6-FAA36F2C9313_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/3C6F502D-D7C8-4D07-ABB6-FAA36F2C9313_1_105_c.jpeg)
</figure>

The case arrived with zero damage - everything was packaged very securely with lots of padding. All required cables were provided, as well as an I/O shield to match the design of the rest of the case.

One of LUNA Designs big selling points for the DNK-H is the ability to build a Hackintosh resembling a Mac Mini. They even provide some instructions to do so using the ASUS Prime H310T that I bought. All in all, the case was worth every dollar.

## The Motherboard

The tested compatibility and fit with the case drove me towards the ASUS Prime H310T motherboard. Combined with the low price, already having a nice I/O shield, good processor and memory compatibility, choosing the motherboard was a no-brainer. 

<figure class="constrained" markdown="1">
![5EDF74B3-ACB6-4DCC-9654-54A027A69A35_1_201_a](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/5EDF74B3-ACB6-4DCC-9654-54A027A69A35_1_201_a.jpeg)
</figure>

It has two RAM slots, so I opted to go with a single 16GB stick to start with leaving plenty of room for a future upgrade.

## The Processor

My initial thought was to get an i3 because it would be "good enough" for the level of work I would be doing. However, priority #2 above was flexibility, and so I went up to the 6-core i5.

<figure class="constrained" markdown="1">
![30863042-ED57-49C1-B29E-E429DF96BBCC_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/30863042-ED57-49C1-B29E-E429DF96BBCC_1_105_c.jpeg)
</figure>

## The Build

Not much to say here - everything went smoothly!

<figure class="constrained" markdown="1">
![EA2C48AB-729A-4E3C-A827-9C6B570E3D18_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/EA2C48AB-729A-4E3C-A827-9C6B570E3D18_1_105_c.jpeg)
</figure>

The only challenge was figuring out where to plug in some of the cables on the case to the motherboard. The instructions from LUNA Design on YouTube were outdated for the latest model of the case, so I had to read the manual for the motherboard, and do some educated guessing. Turns out I'm a good guesser, and the labels on some of the wires helped too! There are both green and white LEDs on the front of the case, and I wired the white LED for the power indicator, and the green LED for the hard drive indicator.

<figure class="third" markdown="1">
![AC1B4A52-643C-4DC9-90D4-50E4565BDC1F_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/AC1B4A52-643C-4DC9-90D4-50E4565BDC1F_1_105_c.jpeg)
![69FB9D35-A6F5-48C1-B88B-804B9C01A2AD_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/69FB9D35-A6F5-48C1-B88B-804B9C01A2AD_1_105_c.jpeg)
![E18A58B2-00B7-4036-976B-4BD2AEBBF1A1_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/E18A58B2-00B7-4036-976B-4BD2AEBBF1A1_1_105_c.jpeg)
</figure>

After it was all put together, I fired it up and it worked!

<figure class="half" markdown="1">
![CE050B60-7209-4BFC-8944-B8531C783B0E_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/CE050B60-7209-4BFC-8944-B8531C783B0E_1_105_c.jpeg)
![5AFDFFF8-9DDD-4081-B5D8-CEDFF6D767F7_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/5AFDFFF8-9DDD-4081-B5D8-CEDFF6D767F7_1_105_c.jpeg)
</figure>

The case came with some recommended BIOS changes, presumably to optimize for the built-in fan. Some of the changes seemed to cause issues booting Linux, so I ended up resetting the BIOS. I will most likely re-apply some of the fan changes though, because the fan has since been spinning up and down a little unevenly. I should note that the fan is pretty quiet, and I didn't even notice it before I reset the recommended settings.

## The Operating System

Finally time to see this thing boot up elementary OS!

<figure class="constrained" markdown="1">
![Screen-Shot-2019-12-15-at-1.52.51-PM](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/Screen-Shot-2019-12-15-at-1.52.51-PM.png)
</figure>

Perfection! 🔥

## Additional Photos

After posting my photos over on the Small Form-factor PC Subreddit[^2] (/r/sffpc), one user asked for some additional photos, so I figured I'd add them here as well:

<figure class="third" markdown="1">
![A889A3AC-3A3F-44AA-A289-C61CF25BF0E0_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/A889A3AC-3A3F-44AA-A289-C61CF25BF0E0_1_105_c.jpeg)
![BC4E47BE-C39F-4553-AA65-D0AF1A0AF7AA_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/BC4E47BE-C39F-4553-AA65-D0AF1A0AF7AA_1_105_c.jpeg)
![E50CFF3F-8E51-4E69-B0C9-1E0800F25022_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/E50CFF3F-8E51-4E69-B0C9-1E0800F25022_1_105_c.jpeg)
</figure>
<figure class="third" markdown="1">
![4017599B-AD3A-433C-9EB0-6C06F8046CEF_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/4017599B-AD3A-433C-9EB0-6C06F8046CEF_1_105_c.jpeg)
![D0679130-4BA0-4010-91B0-B7E716397108_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/D0679130-4BA0-4010-91B0-B7E716397108_1_105_c.jpeg)
![F8AC5D9E-455B-45F4-8839-8F74AB4CF2AE_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/F8AC5D9E-455B-45F4-8839-8F74AB4CF2AE_1_105_c.jpeg)
</figure>
<figure class="third" markdown="1">
![E941CF76-2B23-4DC6-BC0E-F39053F339C7_1_105_c](https://s3.amazonaws.com/blog.avojak.ghost/2020/09/E941CF76-2B23-4DC6-BC0E-F39053F339C7_1_105_c.jpeg)
</figure>

---

## Updates

* **11/18/2020**: One downside that I recently discovered is that the motherboard is not capable of outputting two displays at 4K@60Hz. For most people this may not be a problem, but it was a bit of a disappointment when I picked up my second display the other day.

[^1]: Iridium: [https://github.com/avojak/iridium](https://github.com/avojak/iridium)
[^2]: [https://www.reddit.com/r/sffpc/comments/iqrkf9/luna_design_dnkh_miniitx_build](https://www.reddit.com/r/sffpc/comments/iqrkf9/luna_design_dnkh_miniitx_build)