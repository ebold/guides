# Run a Flask app in a Debian based web server

This is a simple solution to run a Flask app in a Debian based web server, which is deployed on BeagleBone Black

## Overview

Flask web apps can be easily deployed by using **'nginx'** and **'gunicorn'**:
- nginx: open-source HTTP server, reverse proxy server and IMAP/POP3 proxy server
- gunicorn: stand-alone WSGI server that provides an interface between any HTTP server and back-end Python web framework (like Flask).

This guide will present step-by-step instructions to set up a web server, which runs on Beaglebone Black and is accessable from Internet.

## Platform

The desired web server consists of:

- Beaglebone Black (BBB)
- Debian Bookworm 12.2 for IoT (deployed in eMMC of BBB)

## Disable unnecessary services

The default Debian image comes with certain services (ssh, dnsmasq, nginx, cloud9 etc).
Disable any unnecessary services with following command (ie., cloud9):

```
$ sudo systemctl status cloud9  # check activity of the cloud9 service
$ sudo systemctl stop cloud9    # stop the cloud9 service
$ sudo systemctl disable cloud9 # and disable it permanently
```

## Check the 'nginx' installation

Check if 'nginx' is running (it should be installed by default):

```
$ sudo systemctl status nginx
$ nginx -v                     # check the actual version
nginx version: nginx/1.22.1
```

If the nginx service is not found, then install it:

```
$ sudo apt-get update
$ sudo apt-get install nginx
```

Start 'nginx' manually, if it's not running and enable it at system startup.

```
$ sudo systemctl enable nginx
$ sudo systemctl start nginx
```

Once it runs, browse to Beaglebone Black (either its hostname or IP address) from another host.
You will see similar content that is shown in this [screenshot](screenshot_nginx_bbb_debian_12.2.png).
By default, 'nginx' creates a root directory at **/var/www/html**.

```
$ ls /var/www/html/
BeaglePlay-io.html  Node-RED.html       Node-RED-ui.html    VSCode.html
```

## Check the Python installation

By default, Python 3.11 version is installed:

```
$ python --version
Python 3.11.2
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

Other settings relevant to the **server** directive is usually stored in a separate configuration file and get loaded into the overall configuration structure through **include /etc/nginx/sites-enabled/*;** directive at the end of the **/etc/nginx/nginx.conf** file. Normally all available server configuration files are located in **/etc/nginx/sites-available** and are activated by a symbolic link in **/etc/nginx/sites-enabled** directory.

First, in the **/etc/nginx/sites-available/** directory a custom server configuration file **myflask** is created with following nginx directives:

```
$ sudo vi /etc/nginx/sites-available/myflask

server {
	listen 80 default_server;
	listen [::]:80;

	root /var/www/html/myflask;

	server_name sudalgaa.ddns.net www.sudalgaa.ddns.net;

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

With above configurations, Nginx listens on port 80 for HTTP requests directed at the domain **sudalgaa.ddns.net**. For each such request, Nginx references the directory **/var/www/html/myflask** as the document root.

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

If you get an error given below, then it might caused by a long domain name:
```
nginx[2312]: nginx: [emerg] could not build server_names_hash, you should increase server_names_hash_bucket_size: 32
nginx[2312]: nginx: configuration file /etc/nginx/nginx.conf test failed
```

Add/modify entry **server_names_hash_bucket_size** (in _http_ block) in **/etc/nginx/nginx.conf** to fix this error:
```
$ sudo vi /etc/nginx/nginx.conf

http {
        # ... other entries
        server_names_hash_bucket_size  64;
}
```

## Gunicorn

Gunicorn is a stand-alone WSGI web application server that supports Flask framework.

### Basics (installation, setup) [3]

Where possible, a Python library module should be installed into a virtual environment.
On Debian/Ubuntu systems, you need to install the 'python3-full' package to get the 'python3.11-venv' tool:

```
$ sudo apt install python3-full
```

Create the desired virtual environment for Flask app:

```
$ mkdir -p ~/python_venvs
$ python3 -m venv ~/python_venvs/flask_app
$ source ~/python_venvs/flask_app/bin/activate
$ pip install gunicorn                          # installs gunicorn-21.2.0
$ pip install Flask                             # installs Flask-3.0.0 click-8.1.7 itsdangerous-2.1.2 Jinaj2-3.1.3 MarkupSafe-2.1.3 Werkzeug-3.0.1 blinker-1.7.0
```

The general syntax to run 'gunicorn' is **'gunicorn [options] module:callable'**, where 'module' is the dotted import name of application module and 'callable' is the variable with the application.

Launch 'gunicorn' to load the desired Flask application:

```
$ cd /var/www/html/myflask
$ gunicorn --bind=unix:/tmp/gunicorn.sock app:app
[2024-01-15 20:30:08 +0000] [2012] [INFO] Starting gunicorn 21.2.0
[2024-01-15 20:30:08 +0000] [2012] [INFO] Listening at: unix:/tmp/gunicorn.sock (2012)
[2024-01-15 20:30:08 +0000] [2012] [INFO] Using worker: sync
[2024-01-15 20:30:08 +0000] [2013] [INFO] Booting worker with pid: 2013

# press CRTL+C to stop the application
[2024-01-15 20:31:31 +0000] [2012] [INFO] Handling signal: int
[2024-01-15 20:31:31 +0000] [2013] [INFO] Worker exiting (pid: 2013)
[2024-01-15 20:31:32 +0000] [2012] [INFO] Shutting down: Master
```

The **--bind=unix:/tmp/gunicorn.sock** parameter binds the unix socket to what is setup in Nginx **location @wsgi** configuration.
The **app:app** parameter has the form of **module:calleable** as mentioned and refers to a Flask object (app = Flask(__name__)) in a given the Flask application (app.py).
There are other parameters that support better handling of the HTTP requests: **--workers** and **--threads**.

In order to check if both 'nginx' and 'gunicorn' have been properly setup, point to your local web server's IP address in your web browser. You should see your Flask web application! Stop it with __Control + C__ key combination.

### Set up 'gunicorn' as system service [4]

**systemd** is used to automate the running of 'gunicorn' after system boot.
Below configuration should be given in **/etc/systemd/system/gunicorn.service** configuration file:

```
[Unit]
Description=gunicorn daemon for myflask
After=network.target

[Service]
Group=www-data
RuntimeDirectory=gunicorn
WorkingDirectory=/var/www/html/myflask
ExecStart=/path/to/venv/bin/gunicorn --bind=unix:/tmp/gunicorn.sock --workers=2 app:app
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

Now start the Gunicorn service and enable it to start at boot:

```
$ sudo systemctl enable gunicorn  # enable startup at boot
Created symlink /etc/systemd/system/multi-user.target.wants/gunicorn.service → /etc/systemd/system/gunicorn.service.
$ sudo systemctl start gunicorn   # start now
$ sudo systemctl status gunicorn  # check if service process is enabled and running

# optional, check if the gunicorn service is enabled
$ sudo systemctl list-unit-files --state=enabled
```

Is any change is made to **/etc/systemd/system/gunicorn.service**, then reload the daemon service and restart the Gunicorn process:
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart gunicorn
```

### gunicorn cannot start (unsufficient permission of an user file)

In this example 'gunicorn' cannot start because of missing permission for an user file.
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

### No-IP dynamic update client (DUC)

[No-IP](https://www.noip.com) is one of such freely available services. Its free-of-charge option only requires that DNS name must be called at least every 30 days.

You should sign up in No-IP to get a persistent domain name. Create a **hostname** (example: yourname.ddns.net).
This will be the URL you will use to connect to your device from anywhere. You need to know your IP address assigned by your ISP.

### Installation, configuration of No-IP DUC

Once a hostname is created, install and configure the No-IP dynamic update client (DUC).

Download a tarball and decompress:
```
$ $ mkdir -p ~/noip && cd ~/noip
$ wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz
$ tar vzxf noip-duc-linux.tar.gz
$ cd noip-2.1.9-1/
```

To install the update client, invoke:
```
$ make
$ sudo make install         # your No-IP credential is asked here!

# leave the default update interval
# nothing to run at successful update
# noip2 configuration file is created in /usr/local/etc/no-ip2.conf
```

After installation the update client can be started by invoking:
```
$ sudo /usr/local/bin/noip2
$ sudo noip2 -S             # check if the service runs successfully
```

By default, the service does not start at boot. In order to enable automatic startup at boot, create a configuration in **/etc/systemd/system/noip2.service**:

```
$ sudo vi /etc/systemd/system/noip2.service

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
Created symlink /etc/systemd/system/noip.service → /etc/systemd/system/noip2.service.
Created symlink /etc/systemd/system/multi-user.target.wants/noip2.service → /etc/systemd/system/noip2.service.
$ sudo systemctl start noip2.service
$ sudo systemctl status noip2.service
```

If the service was failed to start, alter the default start limit in **/etc/systemd/system.conf** and reload the configuration, restart the service:
```
$ sudo vi /etc/systemd/system.conf     # DefaultStartLimitIntervalSec=10s, DefaultStartLimitBurst=5
$ sudo systemctl daemon-reload
$ sudo systemctl restart noip2.service
```

If you are behind a router (ie., FritzBox) or firewall, you will need to open and forward the TCP ports (80 and 443) for the web services (HTTP/HTTPS).

**FritzBox**: Assume that your 'BeagleBone' device is in a network behind 'FritzBox'. Port forwarding is set up in 'FritzBox' devices under port sharing: **'Internet -> Permit Access -> Port Sharing'**. Select the TCP ports 80 and 443 (HTTP-Server, HTTPS-Server) and assign them to your 'BeagleBone' device.

## Secure your 'nginx' web server with a TLS/SSL certificate [5]

A secure communication between your **web server** and **user's web browser** is guaranteed with a SSL/TLS (Secure Sockets Layer, Transport Layer Security) certificate. One of the CAs (Certificate Authority) that issue such certificates is [Let’s Encrypt](https://letsencrypt.org). It is a free, automated, and open source CA, which implement the ACME (Automatic Certificate Management Environment) protocol. It run for the public and provides SSL/TLS certificates freely to enable HTTPS (HTTP-Secure) on websites.

Certificates issued by Let's Encrypt are **domain-validated**: they have to check that the certificate request comes from a person who actually controls the domain. They do this by sending the client a unique token, and then making a web or DNS request to retrieve a key derived from that token.

### Prerequisites

Ensure that you have met the following prerequisites before you proceed:
- domain name pointing to your public IP address
- 'nginx' web server
- firewall configured to accept connections on ports 80 and 443

### Install 'certbot' [6]

Certificates issued by Let's Encrypt are trusted by all popular browsers and valid for 90 days from the issue date. Certbot is a popular open-source client for automating the process of obtaining and renewing SSL/TLS certificates and configuring web servers to use the certificates.

The recommended way to install 'certbot' is the installation through 'snap':
```
$ sudo apt update
$ sudo apt install snapd                        # install 'snap'
$ sudo snap install --classic certbot           # install 'certbot'
2024-01-17T15:29:47Z INFO Waiting for automatic snapd restart...
certbot 2.8.0 from Certbot Project (certbot-eff✓) installed
```

With the following command prepare the 'certbot' command to be run:

```
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Confirm the nginx configuration  [7]

Verify the **'server_name'** in the nginx configuration in '/etc/nginx/sites-available/myflask':

```
$ grep server_name /etc/nginx/sites-available/myflask
```

It should return 'server_name sudalgaa.ddns.net www.sudalgaa.ddns.net;', otherwise fix the domain name.

### Obtain and install an SSL/TSL certificate

Certbot can obtain an SSL/TSL certificate in a variety of ways through plugins. In our case, the nginx plugin will take care of reconfiguring nginx and reloading the config whenever necessary.

Invoke a following command with the 'nginx' plugin and the 'sudalgaa.ddns.net' domain name ('-d www.sudalgaa.ddns.net' is intentionally not specified, because it is not assigned in 'noip' hostname entry):

```
$ sudo certbot --nginx -d sudalgaa.ddns.net

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for sudalgaa.ddns.net
```

It will prompt to enter an email address (for urgent renewal and security notices), to agree the terms of service and to accept to share your email with other partners (can be disagreed).

When the requested certificate is issued successfully, 'certbot' will return following confirmation:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/sudalgaa.ddns.net/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/sudalgaa.ddns.net/privkey.pem
This certificate expires on 2024-04-16.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for sudalgaa.ddns.net to /etc/nginx/sites-enabled/myflask
Congratulations! You have successfully enabled HTTPS on https://sudalgaa.ddns.net

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Verify the secure web access just reloading your website using 'https://' and look at the security indicator. Moreover, test the web server using the [SSL Labs Server Test](https://www.ssllabs.com/ssltest): it will get an **A** grade.

### Verify auto-renewal by certbot

As mentioned before, the certificates issued by Let's Encrypt are valid for 90 days.
'certbot' can handle automatic certificate renewal of the installed certificate (if it's within 30 days of expiration). By default, it is added to into systemd timers and will run twice a day.

Verify all systemd timers:

```
$ sudo systemctl list-timers  # list all systemd timers


NEXT                        LEFT        LAST                        PASSED       UNIT                   >
Thu 2024-01-18 02:52:00 UTC 10h left    -                           -            snap.certbot.renew.time>
```

To test the renewal task, run 'certbot' renewal with the 'dry-run' option:

```
$ sudo certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/sudalgaa.ddns.net.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Account registered.
Simulating renewal of an existing certificate for sudalgaa.ddns.net

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/sudalgaa.ddns.net/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

## Troubleshootings

### Renew expired certificate (issued by Let's Encrypt)

Assume that a client tool, Certbot, is installed. Invoke following commands to renew the certificate:

```
$ certbot --version                         # optional, check certbot installation
$ sudo certbot certificates                 # optional, check the certificate expiration date
$ sudo certbot --nginx -d sudalgaa.ddns.net # renew the certificate
$ sudo systemctl list-timers                # check the renewal period
$ sudo certbot certificates                 # optional
```

### Update 'gunicorn'

New updates for installed software packeges are necessary regarding security fixes or some other reasons.
'gunicorn' is a Python module and installed via 'pip'.
Assume that Python (eg, v3.11.2) is installed under virtual environment.

Hence, updating 'gunicorn' requires at least 2 steps:
- activate the target virtual environment
- update the 'gunicorn' using 'pip'

```
me@BeagleBone:~$ source ~/python_venvs/flask_app/bin/activate

(flask_app) me@BeagleBone:~$ pip install --upgrade gunicorn
Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Requirement already satisfied: gunicorn in ./python_venvs/flask_app/lib/python3.11/site-packages (21.2.0)
Collecting gunicorn
  Downloading https://www.piwheels.org/simple/gunicorn/gunicorn-22.0.0-py3-none-any.whl (84 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 84.4/84.4 kB 717.5 kB/s eta 0:00:00
Requirement already satisfied: packaging in ./python_venvs/flask_app/lib/python3.11/site-packages (from gunicorn) (23.2)
Installing collected packages: gunicorn
  Attempting uninstall: gunicorn
    Found existing installation: gunicorn 21.2.0
    Uninstalling gunicorn-21.2.0:
      Successfully uninstalled gunicorn-21.2.0
Successfully installed gunicorn-22.0.0
```

### 'git pull' not permitted

Web site is normally often updated. But 'git pull' is failed with unsufficient permission:

```
$ git pull
error: cannot open '.git/FETCH_HEAD': Permission denied
```

__Solution__: invoke the command with 'sudo':

```
$ sudo git pull
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (2/2), done.
remote: Total 4 (delta 1), reused 1 (delta 1), pack-reused 2
Unpacking objects: 100% (4/4), 2.09 KiB | 51.00 KiB/s, done.
From https://github.com/ebold/myflask
   c95981f..b76978a  master     -> origin/master
Updating c95981f..b76978a
Fast-forward
 requirements.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

## Links
1. How to properly host Flask application with Nginx and Gunicorn, [link](https://www.e-tinkers.com/2018/08/how-to-properly-host-flask-application-with-nginx-and-guincorn/)
2. Zugriff auf den eigenen Raspberry Pi aus dem Internet, [link](https://buyzero.de/blogs/news/zugriff-auf-den-eigenen-raspberry-pi-aus-dem-internet)
3. Using Virtualenv to install gunicorn, [link](https://docs.gunicorn.org/en/stable/deploy.html#using-virtualenv)
4. Systemd configuration for gunicorn, [link](https://docs.gunicorn.org/en/stable/deploy.html#systemd)
5. Getting Started with Let's Encrypt, [link](https://letsencrypt.org/getting-started/)
6. Certbot ACME client, [link](https://certbot.eff.org)
7. How to Secure Nginx with Let's Encrypt on Debian 11, [link](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-debian-11)
