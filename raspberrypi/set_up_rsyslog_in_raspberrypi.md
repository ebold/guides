# Set up rsyslog in Raspberry Pi

Install **rsyslog**:

```
$ sudo apt install rsyslog
```

Edit syslog config to enable receive messages via **UDP/TCP port 514**:
```
$ sudo nano /etc/rsyslog.conf    # locate below lines and uncomment them:

#module(load="imudp")
#input(type="imudp" port="514")
#module(load="imtcp")
#input(type="imtcp" port="514")
```

Add new configuration that routes received messages from a network **192.168.2.0** to a log file **/var/log/remote_wrs.log**:

```
$ sudo nano /etc/rsyslog.d/remote_wrs.conf  # add below content to this file

if $fromhost-ip startswith "192.168.2" then /var/log/remote_wrs.log
& stop
```

Restart rsyslog to take changes in effect:

```
$ sudo systemctl restart rsyslog
```

To test if rsyslog receives syslog event messages from remote hosts you can create a log message with the **logger** command on a remote host:

```
$ logger -d -t mylog -n <server> This is test message # -d: use datagrams (UDP), -t tag: specify a tag, -n server: write to remote syslog server
```

If everything works as expected, then you will see log messages in **/var/log/remote_wrs.log**.

```
$ less /var/log/remote_wrs.log
```

And finally specify log rotation in a **/etc/logrotate.d/remote_wrs** configuration file:

```
$ sudo nano /etc/logrotate.d/remote_wrs   # add below content to this file

/var/log/remote_wrs.log {
	daily
	rotate 30
	compress
	delaycompress
	missingok
	notifempty
	create 644 root root
}
```

Finally restart logrotate service:

```
$ sudo systemctl restart logrotate.service
```
