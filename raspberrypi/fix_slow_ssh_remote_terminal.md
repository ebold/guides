# Fix slow SSH remote terminal with your Raspberry Pi

## First action: DNS, [1]

Some users refer about delays that may be introduced by SSH reverse DNS queries.

SSH daemon controls that client it is communicating with is the same during the entire connection by checking periodically the match with IP address and related hostname. It can add some reverse DNS resolution load to your device, so resulting in slower connection in some configurations (expecially when SSH connection is made from internet. For local network connection it shouldn’t give great impacts).

Try disable DNS check by uncommenting **'#UseDNS no'** line from **'/etc/ssh/sshd_config'** configuration file.

## Second action: public-key authentication, [1]

Instead of password-based authentication, try to use a public-key authentication.

Copy your public key to your Raspberry Pi:

```
$ ssh-copy-id -i <path/to/public/key> <user>@<rpi>

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/<user>/.ssh/id_ecdsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
<user>@<rpi>'s password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '<user>@<rpi>'"
and check to make sure that only the key(s) you wanted were added.
```

Source:
- [1] [Fixing slow SSH remote terminal with your Raspberry Pi](https://peppe8o.com/fix-slow-ssh-remote-terminal-issue-in-raspberry-pi-os/)
