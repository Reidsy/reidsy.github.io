---
layout:	post
title:	"Installing io.js"
date:	2015-02-17
categories: sysadmin
---
1. Download a copy of io.js

		wget https://iojs.org/dist/v1.2.0/iojs-v1.2.0-linux-x64.tar.xz

2. Extract

		tar xf iojs-v1.2.0-linux-x64.tar.xz

3. Copy to `/opt`

		cd iojs-v1.2.0-linux-x64
        cp -R * /opt
        
4. Add `/opt/bin` to your path by editing your `~/.profile` and adding the line

		PATH="$PATH:/opt/bin"
