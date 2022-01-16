---
title:  "elementary OS in VMware Fusion"
description: " "
author: avojak
image: https://s3.amazonaws.com/blog.avojak.ghost/2019/12/Screen-Shot-2019-12-15-at-1.52.51-PM.png
tags:
  - elementary-os
---

I recently purchased a new 16" MacBook Pro to replace the clunky PC that I've been using for several years now. In order to continue developing for elementary OS, I decided to give VMware Fusion a try.

The initial installation is straightforward and very easy, however getting Linux to play nice with a Retina display is a bit complicated. 

## 1. Install VMware Tools

After completing the installation of elementary, install the VMware Tools on the guest OS (in this case, elementary). From the Virtual Machine menu in the VMware toolbar, select "Install VMware Tools". This will mount the VMware Tools disk image, which should now be visible in Files.

From Terminal, navigate to the mounted image:

```bash
$ cd /media/<username>/VMware\ Tools/
```

Copy the tools archive to your downloads folder:

```bash
$ cp VMwareTools-10.3.10-13959562.tar.gz ~/Downloads
```

Now extract the tools archive:

```bash
$ cd ~/Downloads
$ tar -xvzf VMwareTools-10.3.10-13959562.tar.gz
```

Finally, run the Perl install script:

```bash
$ cd vmware-tools-distrib/
$ sudo ./vmware-install.pl
```

During the installation I selected all of the defaults, but you can configure as desired.

When installation is complete, poweroff the VM.

## 2. Configure the VM Display Settings

From the VMware settings, choose the Display settings and check the box to "Use full resolution for Retina display".

![Display settings](https://s3.amazonaws.com/blog.avojak.ghost/2019/12/Screen-Shot-2019-12-15-at-1.44.22-PM.png)

Now start the VM and login.

## 3. Configure elementary

After logging back in to elementary, open the display settings. Click the gear icon and adjust the resolution to the full resolution of your MacBook Pro Retina display (in the case of the 16" model, this should be 3584x2240). Also set the Scaling factor to "Pixel Doubled".

![elementary Display Settings](https://s3.amazonaws.com/blog.avojak.ghost/2019/12/Screen-Shot-2019-12-15-at-1.48.10-PM.png)

Finally, open up the terminal and run the following two commands[^1]:

```bash
$ gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', <2>}]"
$ gsettings set org.gnome.desktop.interface scaling-factor 2
```

If all went well, you should now see a beautiful, crisp elementary OS desktop (if not, you may need to either logout or reboot the VM).

![elementary Desktop](https://s3.amazonaws.com/blog.avojak.ghost/2019/12/Screen-Shot-2019-12-15-at-1.52.51-PM.png)

## Additional Tweaks

### GTK File Chooser

I found that the Gtk file chooser dialog was not displaying the buttons[^2]. The solution is to create a gsettings entry: 

```bash
$ gsettings set org.gnome.settings-daemon.plugins.xsettings overrides \
  "{'Gtk/DialogsUseHeader': <0>, 'Gtk/ShellShowsAppMenu': <0>, 'Gtk/DecorationLayout': <'close:menu,maximize'>}"
```
---

## Updates

* **11/18/2020**: For quite some time I've been receiving a VMware notification on startup about VMware Tools not running. I think I found the solution: [Ubuntu 18.10 guest and Open VM Tools: screen resizing and Retina resolution don't work](https://communities.vmware.com/t5/VMware-Fusion-Discussions/Ubuntu-18-10-guest-and-Open-VM-Tools-screen-resizing-and-Retina/m-p/1369285)

> I delayed the service open-vm-tools to start after the display manager and it seems to work now.
> ~$ sudo vi /lib/systemd/system/open-vm-tools.service
> Add under [Unit] the following line:
> After=display-manager.service
> Save the file and reboot.

* **9/22/2021**: With the release of elementary OS 6, the only change that I apply to the display settings is setting the scaling factory to HiDPI (2x)

---

[^1]: [https://wiki.archlinux.org/index.php/HiDPI](https://wiki.archlinux.org/index.php/HiDPI)
[^2]: [https://elementaryos.stackexchange.com/q/1638/8280](https://elementaryos.stackexchange.com/q/1638/8280)