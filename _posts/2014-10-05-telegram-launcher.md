---
id: 33
title: Add Telegram messenger launcher to Gnome applications list
date: 2014-10-05T13:41:00+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=33
permalink: /blog/:year/:month/telegram-launcher/
categories:
  - GNU/Linux
---
[Telegram](https://telegram.org/) is a fast, open-source and secure messenger for mobile and desktop platforms. When you download telegram messenger for Linux, you just get two executables, *Telegram* and *Updater.* There is nothing to add these to gnome (or KDE or any other freedesktop.org-compliant desktops) applications list so you can not add this to your favorites or easily launch it with typing its name in gnome shell.

Here's where *desktop* files come in handy! Desktop files are standard way to *register* an application in desktop manager so they can be launched from the applications list, menus and whatever launcher the desktop uses. We build a desktop file for telegram and then put it in `/usr/share/applications` or `~/.local/applications`. If you put the file in first directory it will be accessible for all users (also you need root privileges to be able to add a file to that directory) but if you put the file in the second one, the file is accessible only for you.

Download [telegram logo](https://telegram.org/img/t_logo.png) to be used as an icon for launcher. Create a file named `telegtam.desktop` with following contents in `/usr/share/applications` (or `~/.local/applications` as mentioned above). You may change 'Exec' and 'Icon' lines based on your own configurations and directories.
```
[Desktop Entry]
Version=1.0
Name=Telegram
Exec=/path/to/telegram/executable/Telegram
Icon=/path/to/telegram/logo/t_logo.png
Terminal=false
Type=Application
Categories=Network;Internet
```
Learn more about *desktop entry* files [here](https://developer.gnome.org/integration-guide/stable/desktop-files.html.en) 
and start using Telegram [now](https://telegram.org/dl/webogram) if you haven't already!
