# Running Flask web app with Nginx and Gunicorn on Raspberry Pi

A simple solution to host a small web app is deploy it on Raspberry Pi using Nginx and Gunicorn.

## Required items

- uSD card with Raspbian 10
- Raspberry Pi Model B rev 2 (minimum)

## Overview

Following instructions in below links one could relatively easily deploy a Flask app on Raspberry Pi using Nginx and Gunicorn.
Nginx is open-source HTTP server, reverse proxy server and IMAP/POP3 proxy server.
Gunicorn is a stand-alone WSGI server that provides an interface between any HTTP server and back-end Python web framework (like Flask).

This guide will present step-by-step instructions to set up a web server, which runs on Raspberry Pi and accessable from Internet.

## Install Nginx

Install Nginx

```
sudo apt-get update
sudo apt-get install nginx
```

Check if Nginx service runs
```
sudo service nginx status
```

Start Nginx manually, if it's not running
```
sudo service nginx start
```

Once it runs, browse to Raspberry Pi (either its hostname or IP address) from any other host.
You will see following page. [picture]
By default, Nginx creates a root directory at **/var/www/html**.

## Install Flask, deploy your web app

Install Flask
```
sudo apt-get install python3-pip
sudo pip3 install flask
```

Deploy your web app in to the directory **/var/www/html**.
```
cd /var/www/html
git clone <url_of_your_app_repo>
```

Allow your web app accessable from any host by adding **host=0.0.0.0** argument to your Flask app.
```
if __name__ == '__main__':
    app.run(host="0.0.0.0")
```

Now launch your Flask app (assume **myflaskapp** is directory of your Flask repo and **app.py** is your Flask app)
```
cd /var/www/html/myflaskapp
python3 app.py
```
Visit your web app from other host by specifing either hostname or IP address of Raspberry Pi and port 5000
```
raspberrypi:5000
```

### Set up Nginx

## Install, set up Gunicorn

## Enable public access from Internet

## Why don't you use heroku?

One could use heroku free hosting service to deploy a web app developed with the Flask framework.
But because of "ephemeral" data storage used in heroku, the app can not store user data permanently.
It means user data is lost either app restart once every 24 hours or after 30 minutes of inactivity of the app.

## Links
- How to properly host Flask application with Nginx and Gunicorn https://www.e-tinkers.com/2018/08/how-to-properly-host-flask-application-with-nginx-and-guincorn/
- Zugriff auf den eigenen Raspberry Pi aus dem Internet https://buyzero.de/blogs/news/zugriff-auf-den-eigenen-raspberry-pi-aus-dem-internet
