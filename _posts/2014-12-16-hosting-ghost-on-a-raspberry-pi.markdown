---
layout:	post
title:	Hosting Ghost on a Raspberry Pi
date:	2014-12-16
categories:	sysadmin
---
Ghost is pretty awesome blogging software but unfortunately it fails to install out of the box on arch linux and after finally installing it, the browser will time out when trying to login. After many hours of google searching and trying many different approaches to get ghost working, I finally found a solution.

I will also explain how to configure nginx to act as a reverse proxy and get ghost to start on system startup.

This guide assumes that you have a new installation of arch on a raspberry pi.

# Install Ghost
I will be installing ghost to `/var/www/ghost`.

1. Ensure arch is up to date and install nodejs and build tools. Build tools are required because Ghost uses sqlite3 which must be compiled for the raspberry pi.

		$ pacman -Syu;
    	$ pacman -S nodejs base-devel python2;

	Pacman will ask you what software you want to install from the base-devel package. Press 'Enter' to select all of the packages and then 'y' and 'Enter' to confirm.
    
2. Download and extract ghost

		$ wget http://ghost.org/zip/ghost-0.4.2.zip
        $ unzip ghost-0.4.2.zip -d /var/www/ghost
        
3. Get the dependencies for ghost

		$ cd /var/www/ghost
        $ npm install --production
        
4. Fix Ghosts performance issues

		$ npm install bcrypt
        $ nano ./core/server/models/user.js
   
    Find the line that looks like
   
   		bcrypt = require('bcryptjs')
        
	and replace it with
    
        bcrypt = require('bcrypt')

# Setup start script
This script will start ghost on system startup, it will also run ghost as the user `http` and group `http` (which is a little safer than running ghost as root).

	$ nano /etc/systemd/system/ghost.service
    
And put the following inside the file

    [Unit]
    Description=Ghost blogging software
    After=syslog.target
    After=network.target
    
    [Service]
    User=http
    Group=http
    Environment="NODE_ENV=production"
    ExecStart=/usr/bin/npm start /var/www/ghost
    
    [Install]
    WantedBy=multi-user.target

Save the file.

To test if everything is working correctly ghost needs to be configured to listen on all interfaces instead of just localhost (so you can access it remotely). You will need to change this back after testing. To do this:

	$ cd /var/www/ghost
    $ cp config.example.js config.js
    $ nano config.js
    
Find the production section of the file and change the line

	host: '127.0.0.1',

To

	host: '0.0.0.0',
    
Start ghost by running the following

    $ systemctl start ghost
    
Wait about 30 seconds while ghost starts and then go to your browser and enter the ip address of the raspberry pi and port 2368. For example my raspberry pi is at `192.168.1.5` so I would go to the address `http://192.168.1.5:2368/`.

After you know ghost is working, reconfigure ghost to only listen on localhost. After making these changes you will not be able to access ghost in your web browser, this is intended.

	$ systemctl stop ghost
    $ nano config.js

Find the production section of the file and change the line

	host: '0.0.0.0',

to

	host: '127.0.0.1',

Also in the production section change `url` to the url of your blog.

Start ghost and enable ghost to run at startup

	$ systemctl start ghost
    $ systemctl enable ghost
    
#Configure Nginx

This will allow you to run ghost on port 80 without giving ghost root privileges.

	$ nano /etc/nginx/sites-available/ghost

And put the following into the file

	upstream ghost {
    	server 127.0.0.1:2368;
    }
    
    server {
    	listen 80;
        server_name blog.reidsy.com;
        location / {
        	proxy_pass http://ghost/;
        }
    }

Replace `blog.reidsy.com` with the url to your blog.

Save and close the file.

Enable the site and restart nginx

	$ ln -s /etc/nginx/sites-available/ghost /etc/nginx/sites-enabled/ghost
    $ systemctl restart nginx

Go to the url of your blog in your browser, for me this would be `http://blog.reidsy.com` and you should see your new blog.
