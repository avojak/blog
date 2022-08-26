---
title: "Adding Disk Space to an CentOS VM"
description: "Increase the size of your root partition on an CentOS VM"
author: avojak
image: https://images.unsplash.com/photo-1597852074816-d933c7d2b988
tags:
  - evergreen
  - centos
  - virtual-machine
---

This post is mostly a reminder to my future self how to do this!

1. Update the virtual disk

    This is the easy part - shut down the VM and expand the disk size in the virtual machine settings. Easy peasy.

2. Install growpart

    If not already installed, grab growpart.

        $ sudo yum install cloud-utils-growpart

3. Expand the partition

    I found the name of my device using the Disks appliction. In my case the root partition was on `/dev/nvme0n1p3`, so the device is `/dev/nvme0n1` and the partition is `3`.

    Using growpart (first with `-N` to do a dry-run):

        $ sudo growpart /dev/nvme0n1 3 -N

    If no errors, run again without the `-N` option.

4. Expand the physical volume

        $ sudo pvresize /dev/nvme0n1p3
        Physical volume "/dev/nvme0n1p3" changed
        1 physical volume(s) resized or updated / 0 physical volume(s) not resized
        $ sudo pvs
        PV             VG Fmt  Attr PSize  PFree 
        /dev/nvme0n1p3 cs lvm2 a--  48.41g 30.00g
        $ sudo vgs
        VG #PV #LV #SN Attr   VSize  VFree 
        cs   1   2   0 wz--n- 48.41g 30.00g

5. Expand the logical volume

    Using the output from `vgs` above (you can also find the name from the Disks application), we can now expand `/root`:

        $ sudo lvextend -r -l +100%FREE /dev/cs/root
          Size of logical volume cs/root changed from 16.41 GiB (4201 extents) to 46.41 GiB (11881 extents).
          Logical volume cs/root successfully resized.
          meta-data=/dev/mapper/cs-root    isize=512    agcount=4, agsize=1075456 blks
                   =                       sectsz=512   attr=2, projid32bit=1
                   =                       crc=1        finobt=1, sparse=1, rmapbt=0
                   =                       reflink=1    bigtime=0 inobtcount=0
          data     =                       bsize=4096   blocks=4301824, imaxpct=25
                   =                       sunit=0      swidth=0 blks
          naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
          log      =internal log           bsize=4096   blocks=2560, version=2
                   =                       sectsz=512   sunit=0 blks, lazy-count=1
          realtime =none                   extsz=4096   blocks=0, rtextents=0
          data blocks changed from 4301824 to 12166144

    Verify:

        $ df -h
          Filesystem           Size  Used Avail Use% Mounted on
          devtmpfs             3.8G     0  3.8G   0% /dev
          tmpfs                3.8G     0  3.8G   0% /dev/shm
          tmpfs                3.8G  9.9M  3.8G   1% /run
          tmpfs                3.8G     0  3.8G   0% /sys/fs/cgroup
          /dev/mapper/cs-root   47G   13G   34G  28% /
          /dev/nvme0n1p2      1014M  251M  764M  25% /boot
          /dev/nvme0n1p1       599M  7.3M  592M   2% /boot/efi
          tmpfs                774M     0  774M   0% /run/user/976
          tmpfs                774M   28K  774M   1% /run/user/1000

6. Fill up the disk and repeat!

---

Helpful resources:
- [https://www.unixarena.com/2022/01/how-to-extend-the-root-filesystem-in-rhel-8-centos-8.html/](https://www.unixarena.com/2022/01/how-to-extend-the-root-filesystem-in-rhel-8-centos-8.html/)