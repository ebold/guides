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

Now launch your Flask app (assume **myflaskapp** is directory of your Flask repo and **app.py** is your Flask app)

```
$ cd /var/www/html/myflaskapp
$ python3 app.py
```

Browse the web app from other host (use hostname/IP address and port 5000, e.g., **raspberrypi:5000**)

### Set up Nginx

## Install, set up Gunicorn

## Enable public access from Internet

## Why don't you use heroku?

Heroku free hosting service could be used to deploy a web app developed with the Flask framework.
But because of "ephemeral" data storage used in heroku, the app can not store user data permanently.
It means user data is lost either app restart once every 24 hours or after 30 minutes of inactivity of the app.

## Links
1. How to properly host Flask application with Nginx and Gunicorn, [link](https://www.e-tinkers.com/2018/08/how-to-properly-host-flask-application-with-nginx-and-guincorn/)
2. Zugriff auf den eigenen Raspberry Pi aus dem Internet, [link](https://buyzero.de/blogs/news/zugriff-auf-den-eigenen-raspberry-pi-aus-dem-internet)
