# Set up NTP server in Raspberry Pi

If WRSs cannot synchronise system time with Raspberry Pi, then it might be that no NTP server is installed in Raspberry Pi.
In this case just install the NTP server in RPi.

```
$ sudo apt install ntp       # install NTP service
$ sudo systemctl status ntp  # verify if NTP Service is running
```

After that invoke **'ntpd -q -p 192.168.2.1'** from a WRS to syncronise with RPi.

Links:
- [Running NTP server in Raspberry Pi](https://forum-raspberrypi.de/forum/thread/46608-raspberry-pi-als-ntp-server-verwenden-mit-systemzeit-als-zeitquelle/)
- [Set up NTP server in Raspberry Pi](https://rishabhdevyadav.medium.com/how-to-install-ntp-server-and-client-s-on-ubuntu-18-04-lts-f0562e41d0e1)
