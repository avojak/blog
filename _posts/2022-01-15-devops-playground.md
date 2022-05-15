---
title:  "DevOps Playground"
description: "A small DevOps playground in my homelab"
author: avojak
image: https://images.unsplash.com/photo-1604005366359-2f8f2a044336
hidden: true
tags:
  - software
  - homelab
---

This will be the first in an series of posts about my homelab setup. But first, what exactly *is* a homelab?

> A homelab, in the simplest terms is a sandbox that you can learn and play with new or unfamiliar technologies. They can be as simple as a set of VM's on an old PC or laptop... [^1]

For me, it's currently comprised of an old laptop that I converted into a server (For more detail on that project, check out my post: [From Laptop to Rack Mount Server](https://avojak.com/blog/2020/11/25/diy-rack-mount-server/)). I'm still working on installing the software, but the current focus is on home automation software.

The complete source mentioned in this post can be found over in the GitHub repository:

{% include github-card.html
  user="avojak"
  repository="homelab"
%}

# Overview

My setup is currently very simple, as I only have one piece of hardware to work with. The server will be running VMware ESXi as the hypervisor. In a nutshell, a hypervisor is what is installed on the bare metal machine (host) to run and manage virtual machines (guests). The hypervisor provides the guests with isolated, [virtual hardware](https://en.wikipedia.org/wiki/Hardware_virtualization) on which to run.

![Screen-Shot-2020-12-04-at-8.05.43-PM](https://s3.amazonaws.com/blog.avojak.ghost/2020/12/Screen-Shot-2020-12-04-at-8.05.43-PM.png)

For now, I will be working with a single VM running in ESXi. It's good to start out small!

# Automation and Configuration-as-Code

Using virtual machines is not a necessity, however there are many benefits. At scale, virtual machines can be spun up or down as demand ebbs and flows. In a small homelab setting, using virtual machines allows me to easily move between hardware hosts, create backups, and quickly test and rollback changes.

I set out with the goal of learning some industry standard tools along the way. Sure, it might be overkill, but part of the fun of a homelab is having an environment to learn in!

The three main pieces of the puzzle that I'm going to tackle in this Part 1 post are:
1. Creating the guest OS image using [Packer](https://www.packer.io)
2. Deploying the guest VM using [Terraform](https://www.terraform.io)
3. Installing software VM using [Ansible](https://www.ansible.com)

There isn't a particular reason I chose each of these tools over others that are available. I already knew *about* them, so I decided to give it a go!

## Packer - Creating the OS Image

### What is Packer?

> Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration.
> ...
> A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc. [^2]

### Why use Packer?

There are already images available for server operating systems, so why create your own?

Pre-built images a great, but think about what happens when you install it for the first time: you click through the installer, pick some date/time/keyboard/location options, create a user, etc. etc... Definitely not something I want to repeat every time I deploy a new VM!

With Packer, I took the Ubuntu 18.04 LTS Server ISO and created an OVA image file to deploy to ESXi. The OVA has Ubuntu Server already pre-installed, pre-configured, and an SSH key is setup for use later on with Ansible!

<small>*Side note: I'm aware that Ubuntu 20.04 LTS Server is available, however I was running into a lot of trouble getting through the installer without it hanging and waiting for input*</small>

### How?

Packer uses JSON template files to define how to build and provision images. My template file was largely influenced by this post: [How to use Packer to Build an Ubuntu 18.04 Template for VMware vSphere](https://gmusumeci.medium.com/how-to-use-packer-to-build-an-ubuntu-18-04-template-for-vmware-vsphere-da5bbf1be92f). Even though I'm ultimatel not using vSphere, the process is nearly identical.

The `builders` block defines basic information about the ISO to use, the commands to run on boot to automate the installation, and where to place the output image upon completion. Note that some details, such as the `memory`, can be modified once the image is actual used to provision a VM in ESXi.

The `preseed_server.cfg` file provides configuration values for the language, keyboard configuration, user creation, etc.

```json
{
    ...
    "builders": [
        {
            "type": "vmware-iso",
            "guest_os_type": "ubuntu-64",
            "memory": 1024,
            "name": "ubuntu-18.04",
            "floppy_files": ["./preseed_server.cfg"],
            "iso_urls": [
                "http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.5-server-amd64.iso"
            ],
            "iso_checksum": "sha256:8c5fc24894394035402f66f3824beb7234b757dd2b5531379cb310cedfdf0996",
            "http_directory": "http",
            "boot_wait": "10s",
            "boot_command": [
                "<enter><wait><f6><wait><esc><wait>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
                "<bs><bs><bs>",
                "/install/vmlinuz",
                " initrd=/install/initrd.gz",
                " priority=critical",
                " locale=en_US",
                " file=/media/preseed_server.cfg",
                "<enter>"
            ],
            "shutdown_command": "echo 'packer' | sudo -S shutdown -P now",
            "ssh_username": "packer",
            "ssh_password": "packer",
            "ssh_pty": true,
            "ssh_timeout": "1800s",
            "ssh_handshake_attempts": "20",
            "output_directory": "output/ubuntu-18.04-server"
        }
    ]
    ...
}
```

Within the `provisioners` block, we can define commands to run against the installed operating system. This allows you to install software and make other changes before the template is finalized.

There are other types of provisioners, but in my case I only wanted to accmoplish two tasks:
1. Install software updates
2. Install an SSH key

{% raw %}
```json
{
    ...
    "provisioners": [
        {
            "type": "shell",
            "inline": [
                "sudo apt-get update",
                "sudo apt-get upgrade -y",
                "sudo apt-get install -y curl cloud-init python3-distutils",
                "curl -sSL https://raw.githubusercontent.com/vmware/cloud-init-vmware-guestinfo/master/install.sh | sudo sh -"
            ]
        },
        {
            "type": "shell",
            "environment_vars": [
                "SSH_KEY={{user `ssh_key`}}"
            ],
            "inline": [
                "sudo mkdir /root/.ssh/",
                "sudo chmod -R 700 /root/.ssh",
                "echo $SSH_KEY | sudo tee /root/.ssh/authorized_keys",
                "sudo chmod 600 /root/.ssh/authorized_keys"
            ]
        },
        {
            "type": "shell",
            "inline": [
                "echo 'Packer Template Build -- Complete'"
            ]    
        } 
    ]
    ...
}
```
{% endraw %}

The SSH key is provided at build time using a variable:

```json
{
    "variables": {
        "ssh_key": ""
    }
    ...
}
```

```bash
$ packer build -var 'ssh_key=...' ubuntu-18.04-server-template.json
```

The final piece of the template file is a post-processor to create the OVA file:

```json
{
    ...
    "post-processors": [
        {
            "type": "shell-local",
            "inline": [ "ovftool output/ubuntu-18.04-server/packer-ubuntu-18.04.vmx output/ubuntu-18.04-server/packer-ubuntu-18.04.ova" ]
        }
    ]
    ...
}
```

With the `vmware-iso` type builder we get VMDK/VMX files for free, but I wanted to deploy using an OVA image (I had better luck deploying OVAs with Terraform). To do this I simply leveraged the `ovftool` from VMware [^3].

## Terraform - Deploying the VMs

## Ansible - Installing Software

[^1]: https://www.epmmarshall.com/homelab-intro/
[^2]: https://www.packer.io/intro
[^3]: https://code.vmware.com/web/tool/4.4.0/ovf