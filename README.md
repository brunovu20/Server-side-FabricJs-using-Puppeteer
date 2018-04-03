# Server side FabricJs using Puppeteer

Creating a PNG from fabricjs canvas server side is not reliable, as demonstrated in this repo:

https://github.com/radiolondra/Fabric-and-or-Node-Canvas-Issues

and discussed in this post:
https://github.com/kangax/fabric.js/issues/4812

Node-Canvas, used by Node FabricJS, is not able to correctly work with objects' scaling/positioning, specially for Text and patterns.

Imagine you created a video editor where the user can create and overlay some fabricjs objects with animations. Each object starts at specific frame of the video and ends at another specific frame. To create the overlays you need to scale all the objects to the real video width and height, create one PNG for each frame and then use ffmpeg to put all together and create the final (overlayed) video.

It's unthinkable to create the PNGs client side and send them to the server for ffmpeg processing, because it needs a lot of time and resources. So we have to create the PNG files server side.

Anyway, whatever the job you want to do, if it needs to export the fabricjs objects in some image files and perform some time-consuming processing on the image, and you want to do it server side, the usage of Nodejs versions of FabricJs and Node-Canvas is not the right way.

What can we do?

The only reliable and fast way is to use some headless browser server side and "simulate" exactly what you'd have done client side.

In this repo I used Puppeteer ( https://github.com/GoogleChrome/puppeteer ) to create a simple NodeJs app able to create a PNG file starting from a Json encoded FabricJs canvas remotely sent by the client browser. The Fabric version used is the same version used client side, so no Node-Canvas, nothing.

#### Note: the http server host is a Nginx server. All the http server settings reported here are for Nginx only.

### Used Behaviour:

Ubuntu 16.04.3 as VMWare virtual machine (remember to install VMWare Tools too)
Nginx 1.10.3
NodeJs 8.10.0

### INSTALL NGINX

$ sudo apt-get update

$ sudo apt-get install nginx

### INSTALL CURL

$ sudo apt-get install curl

### INSTALL NODEJS LTS (includes NPM)

$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

$ sudo apt-get install nodejs

### INSTALL PM2 MANAGER GLOBALLY (useful to launch/control NodeJs apps)

$ sudo npm install pm2 -g

Now copy the repo folder (testpuppetgithub) into your linux user folder and :

$ cd testpuppetgithub

### INSTALL THE NEEDED NODE MODULES

$ npm install

This will use the package.json file to install all the needed modules. Puppeteer is installed too together with the Chromium lib.


Now it's time to configure your Nginx server to answer the app requests.

To configure it, you need to know the IP address of your linux box. To do this you can use ipconfig. Write down the IP address.
Now lets configure Nginx.

$ cd /etc/nginx/sites-available

This is not the right way but to make thing simpler and faster, make a copy of the "default" file:
$ sudo cp default default-ORIGINAL

Now edit the default file, delete all rows and put the following:

```
upstream http_backend {
  server 127.0.0.1:44533;
}

server {
	listen 80;
	server_name 192.168.248.132; # PUT HERE THE IP ADDRESS OF YOUR LINUX BOX (ifconfig)
	root /var/www/html;
	index index.html;

	location / {
		proxy_pass http://http_backend;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}
}
```

All done. Now it's time to run our app:

$ cd

$ cd testpuppetgithub

$ pm2 start ftestpuppet.js

To inspect the app logs while running:

$ pm2 logs ftestpuppet --lines 10000

Now the app is running, waiting for connections.

In your Windows box, open your browser and type the address of the linux box (in my case http://192.168.248.132)

You will see a page with an input text box, one button and 2 canvas: the canvas at top is the original canvas, the canvas at bottom is the final (resized) canvas. The input text box contains the fabric canvas objects Json encoded. Clicking the button the Json data will be sent to the server application which will create the final PNG file in the testpuppetgithub/pngs folder.

Esplore the code to understand what happened and how.

To exit the logs type CTRL-C.

To stop all the running apps:

$ pm2 stop all

To clean the logs:

$ pm2 flush

