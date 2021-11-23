# Modify default route on raspberrypi

```
$ ip route
default via 192.168.1.1 dev eth0 onlink                # wrong/bad device 'eth0', must be 'wlan0'
$ sudo ip route delete default
$ sudo ip route add default via 192.168.1.1 dev wlan0
```
