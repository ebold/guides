# Running freeRADIUS on Raspberry Pi

A short guide to set up a RADIUS server on Raspberry Pi with Raspbian (v4.1.7-v7).

## Installation of freeRADIUS

```
sudo apt-get install freeradius -y  # install
/etc/init.d/freeradius status       # check status
```

## Configuration of freeRADIUS

At least two components must be configured:

- users that requires authentication (supplicant)
- physical devices that decides access to network (authenticator)

Let's assume that timing receivers must be authenticated by this RADIUS server.
They are connected to the WR switches that are capable to do RADIUS authentication with this server and 802.1x with timing receivers.

In this case timing receivers are given in **/etc/freeradius/users** as users and WR switches are specified in **/etc/freeradius/clients.conf** as clients.


### Specify authentication security and configuration information for user in **/etc/freeradius/clients.conf**

Add following entry in the given file:

```
00267b0004da	Cleartext-Password := "00267b0004da"
		Auth-Type := Accept,
		Reply-Message := "Hello %{User-Name}"
```

Here '00267b0004da' is an username of a timing receiver with an MAC address of '00:26:7b:00:04:da'.
The user has clear text password same as its username.

### Specify secret for a network of WR switches in **/etc/freeradius/clients.conf**

An IP address range from 192.168.2.0-255 is assigned to WR switches.
They have to provide a secret specified here.

Add following entry in the given file:

```
cleint 192.168.2.0/24 {
	secret    = mysecret
	shortname = wrs-ho-network
}
```

## Restart the RADIUS server

Once the RADIUS server is configured, it requires to be re-started.

```
sudo /etc/init.d/freeradius restart   # restart the freeradius daemon
```

## Usefull commands

Debug access-requests from any client (on Raspberry Pi)

```
sudo freeradius -xX | tee -a /tmp/freeradius.log         # launch freeRADIUS in debug mode
```

Test with a RADIUS server (on WR switches)

```
radtest 00267b0004da 00267b0004da 192.168.2.1 10 mysecret # send access request to the RADIUS server running on Raspberry Pi with the IP address of 192.168.2.1
```
Depending on configuration one should see either 'Access' or 'Reject' packet.

## Links
[ubuntuusers](https://wiki.ubuntuusers.de/FreeRADIUS/)
