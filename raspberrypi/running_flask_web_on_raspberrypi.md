# Running Flask web app with Nginx and Gunicorn on Raspberry Pi

A simple solution to host a small web app on Raspberry Pi using Nginx and Gunicorn.

## Required software and hardware

- uSD card with Raspbian 10
- Raspberry Pi Model B rev 2 (or newer)

## Overview

By using Nginx and Gunicorn one could easily deploy a Flask-based web application on Raspberry Pi.
Nginx is open-source HTTP server, reverse proxy server and IMAP/POP3 proxy server.
Gunicorn is a stand-alone WSGI server that provides an interface between any HTTP server and back-end Python web framework (like Flask).

This guide will present step-by-step instructions to set up a web server, which runs on Raspberry Pi and is accessable from Internet.

## Nginx installation

Install Nginx in Raspberry Pi

```
$ sudo apt-get update
$ sudo apt-get install nginx
```

Check if Nginx service runs

```
$ sudo service nginx status
```

Start Nginx manually, if it's not running

```
$ sudo service nginx start
```

Once it runs, browse to Raspberry Pi (either its hostname or IP address) from another host.
You will see following page. [picture]
By default, Nginx creates a root directory at **/var/www/html**.

## Flask installation

Install Flask in Raspberry Pi

```
$ sudo apt-get install python3-pip
$ sudo pip3 install flask
```

## Web application deployment

Deploy your web app in to the directory **/var/www/html**.

```
$ cd /var/www/html
$ git clone <url_of_your_app_repo>
```

Make the web app accessable from any host by adding **host=0.0.0.0** argument to your Flask app.

```
if __name__ == '__main__':
    app.run(host="0.0.0.0")
```

Now launch your Flask app (assume **myflask** is directory of your Flask repo and **app.py** is your Flask app)

```
$ cd /var/www/html/myflask
$ python3 app.py
```

Browse the web app from other host (use hostname/IP address and port 5000, e.g., **raspberrypi:5000**)
You should see the web page, if everything works properly.

### Nginx setup

This setup is only necessary if you want to optimize the web server performance.

The most part of settings relevant to the **event** and **http** directive blocks are in the **/etc/nginx/nginx.conf** file.

1) Set **multi_accept** directive to **on**. It informs each worker_process to accept all new connections at a time.

2) Change **keepalive_timeout** to **30**. This directive should be lowered so that idle connections can be closed earlier at 30 seconds.

3) Set **server_tokens** to **off**. This will disable emitting the Nginx version number in error messages and response headers.

4) Set **gzip_vary** to **on**. This tell proxies to cache both the gzipped and regular version of a resource.

5) Set **gzip_proxied** to **any**. It will ensure all proxied request responses are gzipped.

6) Set **gzip_comp_level** to **5**. It provides 75% reduction in ASCII files to achieve almost same result as level 9, but does not cause significant impact on CPU usage as level 9.

7) Uncomment **gzip_http_version 1.1;**. It enables compression both for HTTP/1.0 and HTTP/1.1.

8) Add **gzip_min_length 256;** before **gzip_types** directive. It ensures that the file smaller than 256 bytes would not be gzipped.

9) Replace **gzip_types** directive with the more comprehensive list of MIME types given below. This ensures that JavaScript, CSS and even SVG files are gzipped in addition to the HTML file type.

```
gzip_types
  application/atom+xml
  application/javascript
  application/json
  application/rss+xml
  application/vnd.ms-fontobject
  application/x-font-ttf
  application/x-web-app-manifest+json
  application/xhtml+xml
  application/xml
  font/opentype
  image/svg+xml
  image/x-icon
  text/css
  text/plain
  text/x-component
  text/javascript
  text/xml;
```

Other settings relevant to the **server** directive is usually stored ina separate configuration file and get loaded into the overall configuration structure through **include /etc/nginx/sites-enabled/*;** directive at the end of the **/etc/nginx/nginx.conf** file. Normally all available server configuration files are located in **/etc/nginx/sites-available** and are activated by a symbolic link in **/etc/nginx/sites-enabled** directory.

First, in the **/etc/nginx/sites-available/** directory a custom server configuration file **myflask** is created with following nginx directives:

```
$ sudo nano /etc/nginx/sites-available/myflask

server {
	listen 80 default_server;
	listen [::]:80;

	root /var/www/html/myflask;

	server_name songuuli2020.ddns.net;

	location /static {
		alias /var/www/html/myflask/static;
	}

	location / {
		try_files $uri @wsgi;
	}

	location @wsgi {
		proxy_pass http://unix:/tmp/gunicorn.sock;
		include proxy_params;
	}

	location ~* .(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|css|rss|atom|js|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
		access_log off;
		log_not_found off;
		expires max;
	}
}
```

With above configurations, Nginx listens on port 80 for HTTP requests directed at the domain **songuuli2020.ddns.net**. For each such request, Nginx references the directory **/var/www/html/myflask** as the document root.

:fire: __Please set up the port sharing in your router to allow HTTP access.__ Choose **port=80** and **protocol=TCP**.

The **location /static** block maps the HTTP requests to the static content (ie. sitemap.xml, robots.txt and icon) in the **/var/www/html/myflask/static** directory, which are not related to the Flask app.

The **location /** block maps the HTTP requests to a file inside the document root directory and return it as a HTTP response if there is one. This ensures the content such as robots.txt get served by the Nginx server. If the request is not a file that can be found at this root directory, then Nginx should issue an internal redirect to the upstream server defined in the **@wsgi** block.

The **location @wsgi** block proxies the HTTP requests to the **/tmp/gunicorn.sock** unix socket where Flask app will listen to, it also set up some common HTTP headers defined inside the **/etc/nginx/proxy_params** file.

The last **location ~** block tells the Nginx to turn off the access log and set the cache expiry date to maximum.

Now the default server configuration is removed and this custom server configuration is activated:

```
$ cd /etc/nginx/sites-enabled/
$ ll
total 0
lrwxrwxrwx 1 root root 34 Jun 17 13:46 default -> /etc/nginx/sites-available/default
$ sudo rm default
$ sudo ln -s /etc/nginx/sites-available/myflask .
```

Test the Nginx configuration and reload it if everything looks good:

```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo service nginx reload
```

## Gunicorn

Gunicorn is a stand-alone WSGI web application server that supports Flask framework.

### Basics (installation, setup)

```
$ sudo pip3 install gunicorn     # installation with pip
```

To run it manually:
```
$ gunicorn --bind=unix:/tmp/gunicorn.sock app:app
```

The **--bind=unix:/tmp/gunicorn.sock** parameter binds the unix socket to what is setup in Nginx **location @wsgi** configuration.
The **app:app** parameter has form of **module:calleable** and refers to a Flask object (app = Flask(__name__)) in a given Flask application (app.py).
There are other parameters that support better handling of the HTTP requests: **--workers** and **--threads**.

In order to check if both Nginx and Gunicorn have been properly setup, point to your local web server's IP address in your web browser. You should see your Flask web application! Stop it with __Control + C__ key combination.

### Configure as system service

**systemd** is used to automate the running of Gunicorn after system boot.
Below configuration should be given in **/etc/systemd/system/gunicorn.service** configuration file:

```
[Unit]
Description=gunicorn daemon for /var/www/html/myflask/app.py
After=network.target

[Service]
User=pi
Group=www-data
RuntimeDirectory=gunicorn
WorkingDirectory=/var/www/html/
ExecStart=/usr/local/bin/gunicorn --bind=unix:/tmp/gunicorn.sock --workers=2 app:app
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

Now start the Gunicorn service and enable it to start at boot:

```
$ sudo systemctl enable gunicorn  # enable startup at boot
$ sudo systemctl start gunicorn   # start now
$ sudo systemctl status gunicorn  # check if service process is running
```

Is any change is made to **/etc/systemd/system/gunicorn.service**, then reload the daemon service and restart the Gunicorn process:
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart gunicorn
```

### Gunicorn cannot start (unsufficient permission of an user file)

In this example Gunicorn cannot start because of missing permission for an user file.
Setting file permission to writeable for all solves the problem.
```
$ grep gunicorn /var/log/syslog
...
Jun 17 16:12:51 ghost gunicorn[369]:   File "/var/www/html/myflask/app.py", line 118, in poll
Jun 17 16:12:51 ghost gunicorn[369]:     with open(result_filename, 'w') as f:
Jun 17 16:12:51 ghost gunicorn[369]: PermissionError: [Errno 13] Permission denied: 'result.json'

$ cd /var/www/html/myflask
$ ll
total 272
...
-rw-r--r-- 1 root root   6175 Jun 17 14:10 result.json
$ sudo chmod 666 result.json
```

## Enable public access from Internet

Our web server is ready and accessable from local network.
But if this server should be accessed from Internet and your Internet service provider (ISP) assigns dynamic IP address to your router, then dynamic domain name system (DynDNS) needs to be installed. One concept is to update DNS records by using an update client, which provides a persistent addressing method for devices that change their IP addresses frequently.

### No-IP dynamic update client

[No-IP](https://www.noip.com) is one of such freely available services. Its free-of-charge option only requires that DNS name must be called at least every 30 days.

You should sign up in No-IP to get a persistent domain name. Notice your No-IP credential to configure its dynamic update client (DUC). Once the domain name registration is completed, the update client is installed and configured.

Download a tarball and decompress it as follows:
```
$ cd && mkdir noip
$ cd noip/
$ wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz
$ tar vzxf noip-duc-linux.tar.gz
$ cd noip-2.1.9-1/
```

### Installation, configuration

To install the update client, invoke:
```
$ sudo make
$ sudo make install         # your No-IP credential is asked here!
```

After installation the update client can be started by invoking:
```
$ sudo /usr/local/bin/noip2
$ sudo noip2 -S             # check if the service runs successfully
```

By default, the service does not start at boot. In order to enable automatic startup at boot, alter the configuration in **/etc/systemd/system/noip2.service**:

```
$ sudo nano /etc/systemd/system/noip2.service

[Unit]
Description=No-ip.com dynamic IP address updater
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target
Alias=noip.service

[Service]
ExecStart=/usr/local/bin/noip2
Restart=always
Type=forking
```

Now start the update client service and enable it to start at boot:
```
$ sudo systemctl enable noip2.service
$ sudo systemctl start noip2.service
$ sudo systemctl status noip2.service
```

If the service was failed to start, alter the default start limit in **/etc/systemd/system.conf** and reload the configuration, restart the service:
```
$ sudo nano /etc/systemd/system.conf     # DefaultStartLimitIntervalSec=10s, DefaultStartLimitBurst=5
$ sudo systemctl daemon-reload
$ sudo systemctl restart noip2.service
```

## (optional) Set the static IP address

Make sure that Raspberry Pi has a static IP address. Otherwise, set the static IP address.

```
$ ip addr show | grep global   # 'dynamic' flag indicates a dynamic IP address assignment
    inet 192.168.1.197/24 brd 192.168.1.255 scope global dynamic noprefixroute eth0

$ ip route | grep default      # show the default route
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.197 metric 202

$ cat /etc/resolv.conf
# Generated by resolvconf
domain fritz.box
nameserver 192.168.1.1

$ sudo nano /etc/dhcpcd.conf   # edit the configuration file
$ sudo reboot                  # finally reboot your system
```

An example of the static IP configuration in **/etc/dhcpd.conf** looks like below:
```
interface eth0
static ip_address=192.168.1.197/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```

## Why don't you use heroku?

Heroku free hosting service could be used to deploy a web app developed with the Flask framework.
But because of "ephemeral" data storage used in heroku, the app can not store user data permanently.
It means user data is lost either app restart once every 24 hours or after 30 minutes of inactivity of the app.

## Links
1. How to properly host Flask application with Nginx and Gunicorn, [link](https://www.e-tinkers.com/2018/08/how-to-properly-host-flask-application-with-nginx-and-guincorn/)
2. Zugriff auf den eigenen Raspberry Pi aus dem Internet, [link](https://buyzero.de/blogs/news/zugriff-auf-den-eigenen-raspberry-pi-aus-dem-internet)
