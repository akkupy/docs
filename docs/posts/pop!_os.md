---
template: post.html
title: "How I Customized My Pop!_OS"
date: 2023-08-30
authors:
  - akkupy
tags: pop!_os linux os customization linuxdistro opensource
image:
  src: /assets/pop_home.png
  add_to_post: yes
  class: crop-excerpt
---


## Why i choose Pop!_os ?

Pop!_OS is a Linux distribution on top of Ubuntu. So, you get all the benefits of using Ubuntu and it`s well maintained.

No Snap garbage (.deb or flatpak only).

Built-in tileing window manager features.

Better implementation of Gnome 3.

Provides an AMD iso and a separate nVidia ISO with no video driver setup needed.

More hardware support, especially newer devices as System76 sells hardware with Pop installed.

Probably more.... so there is no reason to use Ubuntu over Pop!_OS

After i installed Pop!_OS I spend some time to customize it , so let me show you what I was able to do.


## Install Gnome Tweaks Tool

Gnome tweaks tool gives you additional configuration, and first of all, add minimize and maximize buttons.

```
sudo apt install gnome-tweak-tool
```

then open tweak tool and make the changes.

![Tweak Tool Screenshot](../assets/tweak_tool.png)

## Install GNOME Shell integration extension for google chrome or Mozilla fire fox