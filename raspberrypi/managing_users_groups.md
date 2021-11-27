# Create a new user account

```
root@raspbx:~# useradd -md /home/boled -s /bin/bash boled   # create an account 'boled' with home directory, [1] p.365

root@raspbx:~# grep ^boled /etc/passwd      # view account's information
boled:x:1001:1002::/home/boled:/bin/bash

root@raspbx:~# grep ^boled /etc/shadow      # view accounts' password information
boled:!:18958:0:99999:7:::

root@raspbx:~# groups boled                 # view account's group, [1] p.372
boled : boled
```

# Set password for a new account

```
root@raspbx:~# passwd boled                 # set account password, [1] p.366
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

Now check a new user account by switching to it

```
root@raspbx:~# su - boled                   # switch to 'boled'
boled@raspbx:~$ pwd                         # logged in
/home/boled

boled@raspbx:~$ logout                      # log out
root@raspbx:~#
```

# Add user to the sudo group

```
root@raspbx:~# grep -v ^$ /etc/sudoers | grep -v ^#   # view sudoers configuration, [1] p.545
Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
root	ALL=(ALL:ALL) ALL
%admin ALL=(ALL) ALL
%sudo	ALL=(ALL:ALL) ALL                  # 'sudo' group
%admin  ALL=(ALL) ALL

root@raspbx:~# usermod -aG sudo boled      # add 'boled' as a member of the 'sudo' group, [1] p.374
root@raspbx:~# groups boled                # view groups to which 'boled' belongs to
boled : boled sudo
```

# Disable root login [2]

```
root@raspbx:~# grep nologin /etc/passwd            # find 'nologin' shell directive
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...

root@raspbx:~# usermod -s /usr/sbin/nologin root   # simple method to disable root login

root@raspbx:~# grep root /etc/passwd               # check shell for the root account
root:x:0:0:root:/root:/usr/sbin/nologin            # root login is disabled

```

Now exit or logout and try to login as root:

```
$ ssh root@raspbx
root@raspbx's password:                            # prompt root password, but login is disallowed
...
This account is currently not available.
Connection to raspbx closed.
```

Source:
- [1] LPIC-1 Study Guide
- [2] [4 Ways to Disable Root Account in Linux](https://www.tecmint.com/disable-root-login-in-linux/)
