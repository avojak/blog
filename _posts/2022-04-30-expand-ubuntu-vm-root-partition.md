---
title: "Adding Disk Space to an elementary OS VM"
description: "Increase the size of your root partition on an elementary OS VM"
author: avojak
image: https://images.unsplash.com/photo-1597852074816-d933c7d2b988
tags:
  - elementary-os
  - evergreen
---

I do most of my elementary OS development work within a virtual machine. I'm also not very good at giving myself sufficient disk space, so I have often found myself needing to expand the root partition of my VM.

This post is mostly a reminder to my future self how to do this!

*Note: These same steps will work for Ubuntu or any other Ubuntu-based platform!*

1. Update the virtual disk

    This is the easy part - shut down the VM and expand the disk size in the virtual machine settings. Easy peasy.

2. Install GParted

    If not already installed, grab GParted.

        $ sudo apt install gparted

3. Expand the partition

    Open GParted and you should notice a block of free space at the end of the visualization. Right-click on the partition to expand (in my case `/dev/sda2`) and select `Resize/Move`. Next, drag the little slider all the way over to the right and click `Resize`. Finally hit the little green checkmark
    to apply the changes.

4. Expand the logical volume

    Pop open Terminal and double-check the name of the filesystem for the root (`/`) partition (In my case it's `/dev/mapper/data-root`).

        $ sudo df -k
        Filesystem            1K-blocks     Used Available Use% Mounted on
        udev                    3994004        0   3994004   0% /dev
        tmpfs                    810476     2124    808352   1% /run
        /dev/mapper/data-root  47133460 40273592   4435856  91% /
        ...

    Then we're going to use `lvextend` to expand the logical volume. Be sure to use the correct value for the amount of added space!

        $ sudo lvextend -L +25G /dev/mapper/data-root
          Size of logical volume data/root changed from 45.92 GiB (11756 extents) to 70.92 GiB (18156 extents).
          Logical volume data/root successfully resized.

5. Expand the filesystem

    With the newly added space in the logical volume, it's time to extend it with `resize2fs`:

        $ sudo resize2fs /dev/mapper/data-root
        resize2fs 1.45.5 (07-Jan-2020)
        Filesystem at /dev/mapper/data-root is mounted on /; on-line resizing required
        old_desc_blocks = 6, new_desc_blocks = 9
        The filesystem on /dev/mapper/data-root is now 18591744 (4k) blocks long.

    And verify the change:

        $ sudo df -k
        Filesystem            1K-blocks     Used Available Use% Mounted on
        udev                    3994004        0   3994004   0% /dev
        tmpfs                    810476     2124    808352   1% /run
        /dev/mapper/data-root  72936504 40273472  29193748  58% /
        ...

6. Fill up the disk and repeat!

---

Helpful resources:
- [https://brianchristner.io/how-to-resize-ubuntu-root-partition/](https://brianchristner.io/how-to-resize-ubuntu-root-partition/)