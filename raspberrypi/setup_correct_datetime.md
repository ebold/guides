# Set up correct datetime on Raspberry Pi

```
$ timedatectl                       # get current time info
$ sudo timedatectl set-timezone Europe/Berlin   # choose a correct time zone
$ cat /etc/fake-hwclock.data        # verify the fake HW clock
$ sudo fake-hwclock save            # update the fake HW clock
```

Links:
- [Set up correct datetime on Raspberry Pi using ntp](https://blog.doenselmann.com/richtige-uhrzeit-raspberry-pi-mit-ntp/)
