---
title:  "Installing Docker on elementary OS 5.0 Juno"
description: " "
author: avojak
image: https://images.unsplash.com/photo-1504383633899-a17806f7e9ad
tags:
  - software
  - elementary-os
---

## Uninstall previous versions

From a clean install of elementary OS 5.0 Juno, there shouldn't be anything to remove.

```bash
$ sudo apt-get remove \
    docker \
    docker-engine \
    docker.io \
    containerd \
    runc
```

## Install Docker CE via the Docker repository

There are several ways to install Docker CE, however this is the approach recommended by Docker.

1. "Install packages to allow `apt` to use a repository over HTTPS:" [^1]

    ```bash
    $ sudo apt-get update
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg2 \
        software-properties-common
    ```

2. Add Docker's GPG key, and verify that you have the key with the fingerprint `9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88`:

    ```bash
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo apt-key fingerprint 0EBFCD88

    pub   rsa4096 2017-02-22 [SCEA]
        9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
    sub   rsa4096 2017-02-22 [S]
    ```

3. Add the **stable** Docker repository. The key here is to use `bionic`, because elementary OS 5.0 Juno is build on Ubuntu 18.04 LTS "Bionic Beaver".

    ```bash
    $ sudo apt-add-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
    ```
    
4. Install Docker CE

    ```bash
    $ sudo apt-get update
    $ sudo apt-get install docker-ce
    ```
    
5. Test it out!

    ```bash
    $ sudo docker run hello-world

    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    1b930d010525: Pull complete 
    Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
    3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

    For more examples and ideas, visit:
    https://docs.docker.com/get-started/
    ```
    
6. (Optional) Add your user to the `docker` group to avoid typing `sudo` for every `docker` command:

    ```bash
    $ sudo usermod -aG docker ${USER}
    $ su - ${USER}
    $ id -nG

    alice sudo docker
    ```

---

[^1]: [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/)