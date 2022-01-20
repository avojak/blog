---
title:  "Iridium Turns 1"
description: "Iridium has reached the 1-year mark since its first release"
author: avojak
image: https://imgur.com/VUXSRYo
tags:
  - software
  - app
  - elementary-os
  - evergreen
---

January 21st marks the one-year anniversary of the first release of Iridium! I thought it would be fun to look back at all the changes that have been made in the past year. Time for
a trip down memory lane!

## History

- January 21, 2021 - 1.0.0 Release 🎉
- May 16, 2021 - 1.1.0 Release
    - 📚 Spanish translations by JeysonFlores
    - 📚 Dutch translations by Vistaus
    - ✨ Display available channel list
    - ✨ Allow suppressing join/part messages
    - ✨ Display date/time in chat views when message received after some time
    - ✨ Support opening irc:// links
    - ✨ Remember window size and position
    - ✨ Support /me actions
    - ✨ Add visual indicator for channel operator status
    - 🪲 Use Sqlite.Database.exec instead of Sqlite.Statement.step
    - 🪲 Fix in64_parse error on RPL_TOPICWHOTIME messages
    - 🔧 Force monospace font in chat views in preparation for elementary OS 6
- July 16, 2021 - 1.2.0 Release
    - 🔧 Support for elementary OS 6!
    - 🔧 Include Flatpak manifest
    - 📚 Updated Dutch translations by Vistaus
    - 🪲 Fixed 'servers != NULL' error on startup
- July 27, 2021 - 1.3.0 Release
    - ✨ Use libhandy for application window and headerbar
    - 🪲 Fix libgranite dependency version
    - 🪲 Fix QUIT messages not following join/part suppression preference
    - 🪲 Fix appdata to include Stripe public key
- August 2, 2021 - 1.3.1 Release
    - 🪲 Fix flatpak runtime version to be stable 6
- August 11, 2021 - 1.3.2 Release
    - 🪲 Fix Stripe public key
- November 16, 2021 - 1.4.0 Release
    - ✨ Respect the system light/dark style preference
    - ✨ Support SASL (plain and external) authentication
    - 🪲 Fix autoscrolling when receving QUIT messages
    - 🪲 Fix text not wrapping for errors on server connection
    - 🪲 Fix window size and position not saving
    - 🪲 Fix certificate warning dialog not appearing
    - 🪲 Fix secrets not saving when using the Flatpak installation
    - 🪲 Fix BrowseChannelsDialog not using Granite.Dialog
    - 🪲 Fix new auth tokens not saving when editing a connection
- November 18, 2021 - 1.5.0 Release
    - ✨ Add ability to browse a curated list of IRC servers
    - 🪲 Fix parsing of MODE messages which caused crashes when connecting to certain servers
- December 3, 2021 - 1.6.0 Release
    - ✨ Integrate with OS notifications to display a notification when mentioned in a channel or private message
    - ✨ Display the number of unread mentions as an application badge in the dock
    - ✨ Set initial default nickname and real name based on system user account
    - 🪲 Fix marker line in chat views showing incorrectly when font scaling other than 1x is used
    - 🪲 Fix incorrect handling of network availability state
    - 🪲 Fix the "restoring connections" overlay not appearing on startup
    - 🪲 Fix incorrect parsing of command-line arguments
    - 🪲 Revert the overly-restrictive 8-character nickname limit in the server connection dialog
- December 4, 2021 - 1.6.1 Release
    - 📚 Updated Dutch translations by Vistaus
- December 10, 2021 - 1.7.0 Release
    - ✨ Use non-symbolic icons in the headerbar
    - ✨ Re-open application to the last chat view
    - ✨ Use new icons for server items in the side panel
    - 🪲 Fix parsing of ACTION messages causing empty private messages to appear
    - 🪲 Fix inability to reconnect to a server if it dies without restarting the application
    - 🪲 Fix app icon badge not clearing with private message chat view regains focus
- December 30, 2021 - 1.8.0 Release
    - ✨ Redesign of the headerbar!
    - 🔧 Update elementary OS runtime to 6.1
    - 🪲 Remove unnecessary sandbox hole for accountsservice
    - 🪲 Fix slow switching of channel views when there is a large number of users
    - 🪲 Fix auto-scrolling to be more reliable and simpler

That's **27** bugs fixed, and **18** noteworthy features added just in the past year!

How about a side-by-side to compare the UI since the first release?

<figure class="half" markdown="1">

![iridium-screenshot-01-1.0.0](https://raw.githubusercontent.com/avojak/iridium/1.0.0/data/assets/screenshots/iridium-screenshot-01.png)
![iridium-screenshot-01-1.8.0](https://raw.githubusercontent.com/avojak/iridium/1.8.0/data/assets/screenshots/iridium-screenshot-01.png)


<figcaption><b>Left:</b> The 1.0.0 welcome view | <b>Right:</b> The 1.8.0 welcome view</figcaption>
</figure>

<figure class="half" markdown="1">

![iridium-screenshot-03-1.0.0](https://raw.githubusercontent.com/avojak/iridium/1.0.0/data/assets/screenshots/iridium-screenshot-03.png)
![iridium-screenshot-02-1.8.0](https://raw.githubusercontent.com/avojak/iridium/1.8.0/data/assets/screenshots/iridium-screenshot-02.png)

<figcaption><b>Left:</b> The 1.0.0 chat view | <b>Right:</b> The 1.8.0 chat view</figcaption>
</figure>

## Thank You!

A *massive* thank you to everyone who has supported the project in one way or another - I really appreciate it!