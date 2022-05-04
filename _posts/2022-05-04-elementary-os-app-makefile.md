---
title: "Simple Makefile for elementary OS Apps"
description: "Can't remember all the commands for initializing, building, translating and linting elementary OS Apps? Me neither!"
author: avojak
image: https://images.unsplash.com/photo-1586864387789-628af9feed72
tags:
  - elementary-os
  - software
  - evergreen
---

I have a terrible time trying to remember all the various commands for setting up a Flatpak application for development on a clean system,
building a Flatpak application, generating updated translations files, AND running the Vala linter. I'd much prefer to remember one basic
command. Thank goodness for makefiles!

Before pointing out some of the little features, here's the Makefile that I use in most of my projects - only needing to change the `APP_ID` at the top:

```makefile
SHELL := /bin/bash

APP_ID := com.github.avojak.warble

FLATPAK_REMOTE_URL  := https://flatpak.elementary.io/repo.flatpakrepo
FLATPAK_REMOTE_NAME := appcenter
PLATFORM_VERSION    := 7

BUILD_DIR        := build
NINJA_BUILD_FILE := $(BUILD_DIR)/build.ninja

FLATPAK_BUILDER_FLAGS := --user --install --force-clean
ifdef OFFLINE_BUILD
FLATPAK_BUILDER_FLAGS += --disable-download
endif

# Check for executables which are assumed to already be present on the system
EXECUTABLES = flatpak flatpak-builder
K := $(foreach exec,$(EXECUTABLES),\
        $(if $(shell which $(exec)),some string,$(error "No $(exec) in PATH")))

.DEFAULT_GOAL := flatpak

.PHONY: all
all: translations flatpak

.PHONY: flatpak-init
flatpak-init:
	flatpak remote-add --if-not-exists --system $(FLATPAK_REMOTE_NAME) $(FLATPAK_REMOTE_URL)
	flatpak install -y --user $(FLATPAK_REMOTE_NAME) io.elementary.Platform//$(PLATFORM_VERSION) 
	flatpak install -y --user $(FLATPAK_REMOTE_NAME) io.elementary.Sdk//$(PLATFORM_VERSION)

.PHONY: init
init: flatpak-init

.PHONY: flatpak
flatpak:
	flatpak-builder build $(APP_ID).yml $(FLATPAK_BUILDER_FLAGS)

.PHONY: lint
lint:
	io.elementary.vala-lint ./src

$(NINJA_BUILD_FILE):
	meson build --prefix=/user

.PHONY: translations
translations: $(NINJA_BUILD_FILE)
	ninja -C build $(APP_ID)-pot
	ninja -C build $(APP_ID)-update-po

.PHONY: clean
clean:
	rm -rf ./.flatpak-builder/
	rm -rf ./build/
	rm -rf ./builddir/
```

With that in place, building an app is as simple as:

```bash
$ make init # Only once!
$ make
```

## Features

The most significant feature here for me was adding the `OFFLINE_BUILD` option. Periodically I've run into scenarios where pulling something from `gitlab.gnome.org` will fail for 15+ minutes at a time. That's no good, so being able to easily build entirely from the local cache is a big plus!

Including the linter is a nice touch as well. Often I would defer linter checks until I pushed code and waited for a GitHub action to send me an angry email. No more!

## Drawbacks

I wouldn't say that there are many drawbacks to this approach, however some people may say that this isn't really doing things the `make` way, since almost every rule is `PHONY`. And to that I'd say I don't really care - it makes my life easier!

Due to the way that Flatpak creates the build output I haven't found a better way to optimize this Makefile. Plus, the Flatpak build itself handles a lot of caching to prevent repeat builds of dependencies.

# Bonus: "Run" Script!

As an added bonus, here's a little script that most of my projects use as well to simplify running apps in various modes:

```bash
#!/bin/bash

EXTRA_ARGS=
if [[ "$1" == "--debug" ]]; then
    EXTRA_ARGS="--env=GTK_DEBUG=interactive"
fi
if [[ "$1" == "--inspect" ]]; then
    EXTRA_ARGS="--command=sh --devel"
fi
flatpak run --env=G_MESSAGES_DEBUG=all $EXTRA_ARGS com.github.avojak.warble ${@:2}
```

After using a simple `make` to build the app, I can then just call: `./run`
- 99% of the time I want to see all the debug messages anyway, so I no longer need to remember to type all that out!
- Want to debug some GTK stuff? Use: `./run --debug`
- Some files not quite right in the sandbox? Check them out with: `./run --inspect`

Other arguments/options can be tacked on as well thanks to the bit of BASH magic at the end: `${@:2}`