---
template: post.html
title: "FileBrowser On Docker Container"
date: 2022-11-16
authors:
  - akkupy
tags: filebrowser raspberrypi docker homelab rpi4 fileexplorer
image:
  src: /assets/filebrowser.gif
  add_to_post: yes
  class: crop-excerpt
---

# Filebrowser


## Introduction

[Filebrowser](https://filebrowser.org/) provides a file managing interface within a specified directory and it can be used to upload, delete, preview, rename and edit your files. It allows the creation of multiple users and each user can have its own directory. It can be used as a standalone app.

## Installation

1. **Run the following script**

```
wget -qO- https://raw.githubusercontent.com/akkupy/Homelab/main/scripts/install_filebrowser.sh | bash
```

This will preserve any persistent data under /filebrowser of User Directory, you can adapt the path to whatever suits you.

## Post Installation

Access the web UI through http://raspberry_ip:8123 or using reverse proxy.

(Optional): Configure Reverse Proxy using the documentation [here](https://github.com/akkupy/Homelab/blob/main/docs/nginx_proxy_manager.md#first-proxy-host-setup)

**NOTE**: Default username: admin and password: admin