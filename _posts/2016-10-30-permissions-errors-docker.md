---
layout: single
title:  "Permissions Error with Docker on Windows"
date:   2016-10-30 14:25:39 +0000
categories: jekyll docker permissions
author_profile: true
---

The other day I came across a Jekyll project I wanted to check out on my Windows 10
machine. I added a little Dockerfile and tried to build a container, and then I got
this error when running it:

![Permissions error](/assets/media/docker/permissions_error.png)

At first I thought it was a jekyll problem, but then I realised it is actually
a docker issue. Recently I had to re-install the OS my machine and had forgot to share
the folder with docker. If you go into Docker Settings and "Shared Drives":

![Permissions error](/assets/media/docker/docker_shared_folder.png)

Now when your docker container tries to access your C Drive (or whichever drive you
shared) it should have permission now.
