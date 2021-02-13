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

First, in the **/etc/nginx/sites-available/** directory a custom server configuration file **myflaskapp** is created with following nginx directives:

```
$ sudo nano /etc/nginx/sites-available/myflaskapp

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
$ sudo ln -s /etc/nginx/sites-available/myflaskapp .
```

Test the Nginx configuration and reload it if everything looks good:

```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo service nginx reload
```

## Install, set up Gunicorn

## Enable public access from Internet

## Why don't you use heroku?

Heroku free hosting service could be used to deploy a web app developed with the Flask framework.
But because of "ephemeral" data storage used in heroku, the app can not store user data permanently.
It means user data is lost either app restart once every 24 hours or after 30 minutes of inactivity of the app.

## Links
1. How to properly host Flask application with Nginx and Gunicorn, [link](https://www.e-tinkers.com/2018/08/how-to-properly-host-flask-application-with-nginx-and-guincorn/)
2. Zugriff auf den eigenen Raspberry Pi aus dem Internet, [link](https://buyzero.de/blogs/news/zugriff-auf-den-eigenen-raspberry-pi-aus-dem-internet)
