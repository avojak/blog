---
title:  "Installing ESXi with a USB NIC"
description: "Creating a custom ISO for ESXi 7 with a USB NIC"
author: avojak
image: https://images.unsplash.com/photo-1560732488-6b0df240254a
tags:
  - hardware
---

Several years ago I wrote about how to create a custom ISO for installing ESXi with a Realtek 8111 NIC. Unfortunately that process was only valid for ESXi versions up to 6.7, which has [reached its end of life](https://endoflife.date/esxi).

<aside>
  <a class="featured with-image 1" href="https://avojak.com/blog/2020/12/19/installing-esxi-for-realtek-8111-nic/">
    <div class="featured-image" alt="Featured image" style="background-image: url(https://images.unsplash.com/photo-1560732488-6b0df240254a);"></div>
    <header>
      <h2>Installing ESXi for Realtek 8111 NIC</h2>
      <h3>A classic tale of struggling for hours and hours before finally stumbling across the perfect blog post</h3>
      <div class="byline">
        <div class="avatar">
          <img srcset="https://www.gravatar.com/avatar/4dfa9066e794647fdedf323ecab85333?s=96&amp;d=blank 2x" src="https://www.gravatar.com/avatar/4dfa9066e794647fdedf323ecab85333?s=48&amp;d=blank" alt="Avatar for Andrew Vojak">
        </div>
        <div class="author">
          <span class="name">Andrew Vojak</span>
        </div>
        <time class="post-date" datetime="2020-12-19">Sat, Dec 19, 2020</time>
        <span class="read-time" title="Estimated read time">3 min read</span>
      </div>
    </header>
  </a>
</aside>

To upgrade the host that I previously wrote about, I looked into the [USB Network Native Driver for ESXi](https://communities.vmware.com/t5/Flings-Docs/USB-Network-Native-Driver-for-ESXi-Documentation/ta-p/2998944) Fling. This had listed support for a handful of inexpensive USB NICs, so I purchased a cheap RTL8153 USB NIC to try out.

Because of the incompatibility of the existing NIC with ESXi 7, I was previously unable to even complete the install on this hardware. This meant I would have to create a new installer ISO with the Fling bundled in it, rather than upgrade in place or upgrade to 7 and then install the Fling later.

The process was largely the same as before, but to make things easy to follow, these are the exact steps that I performed:

## 1. Retrieve the ESXi 7.0 Offline Bundle

To obtain the prerequisite tools, refer to the prior blog post. At the time of writing, the current latest release of ESXi 7 is 7.0.3, so I downloaded the offline bundle with the following command:

```
PS C:\Users\Andrew\Downloads> .\ESXi-Customizer-PS.ps1 -v70 -ozip
```

We now have an archive named `ESXi-7.0U3so-22348808-standard.zip` (or something similar if a newer version is released).

## 2. Download the USB NIC Fling

Download the latest version of the Fling for ESXi 7 from this page: [https://communities.vmware.com/t5/Flings/ct-p/77](https://communities.vmware.com/t5/Flings/ct-p/77). 

Currently the latest supported version is [1.10](https://download3.vmware.com/software/vmw-tools/USBNND/ESXi703-VMKUSB-NIC-FLING-55634242-component-19849370.zip).

## 3. Build the new ISO

With both of the offline bundles (ESXi and the Realtek driver) in the same directory, we can now run some more commands:

1. Create the software depot:

    ```
    PS C:\Users\Andrew\Downloads> Add-EsxSoftwareDepot ".\ESXi703-VMKUSB-NIC-FLING-55634242-component-19849370.zip", `
       "ESXi-7.0U3so-22348808-standard.zip" 
    ```

2. Create an image profile by cloning an existing one:

    ```
    PS C:\Users\Andrew\Downloads> Get-EsxImageProfile
    PS C:\Users\Andrew\Downloads> New-EsxImageProfile -CloneProfile ESXi-7.0U3so-22348808-standard `
       -name ESXi-7.0U3so-22348808-usb-nic -Vendor Razz
    PS C:\Users\Andrew\Downloads> Set-EsxImageProfile ESXi-7.0U3so-22348808-usb-nic `
       -AcceptanceLevel CommunitySupported
    ```

3. Add the driver to the new image profile:

    ```
    PS C:\Users\Andrew\Downloads> Get-EsxSoftwarePackage
    PS C:\Users\Andrew\Downloads> Add-EsxSoftwarePackage -ImageProfile ESXi-7.0U3so-22348808-usb-nic `
       -SoftwarePackage vmkusb-nic-fling
    ```

4. Create the ISO:

    ```
    PS C:\Users\Andrew\Downloads> Export-EsxImageProfile -ImageProfile ESXi-7.0U3so-22348808-usb-nic `
       -ExportToIso -filepath .\ESXi-7.0U3so-22348808-usb-nic.iso
    ```

## 4. Create a Bootable USB Installer

I used [Rufus](https://rufus.akeo.ie/) as well to create the bootable USB drive from the ISO, but you can use any tool that will get the job done.

## 5. Install and Expect a Failure!

Boot the installer and go through the prompts until you reach an error screen at 81% installed. Never fear! This is expected!

Thankfully I found this blog post ([Solution: ESXi Installation with USB NIC only fails at 81%](https://www.virten.net/2020/07/solution-esxi-installation-with-usb-nic-only-fails-at-81/)) that explained how technically ESXi was installed at this point, but not fully confiugred. There is a simple fix to force ESXi to recognize the USB NIC for the management interface.

In case that site goes down, I'll quote the steps below:

> 1. ... Create ISO ...
> 2. ... Install until 81% failure ...
> 3. At this point, ESXi is already **installed**, but not **configured**.
> 4. Remove the installation media and reboot the system
> 5. When ESXi is loaded, press **F2** and login as **"root" without password**. (The password entered during the installation has not been saved because the configuration failed)
> 6. You should notice that all Network Options are greyed out. Select **Network Restore Options**.
> 7. Select **Restore Network Settings**
> 8. Log out
> 9. Log back in
> 10. Network options are no longer greyed out and the vusb0 adapter has been detected

Once the steps are followed, you'll be up and running with ESXi 7.0 fully leveraging your new USB NIC! Even better, the default VM Network will be configured to use this NIC as well, so no extra steps are needed to hit the ground running with new VMs, or to update existing VMs.