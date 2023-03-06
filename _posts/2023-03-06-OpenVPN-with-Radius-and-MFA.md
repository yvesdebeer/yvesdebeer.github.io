---
published: false
---
Setting up a VPN connection to access a remote site can be challenging task if you set this up for the first time. In this post I will guide you through the steps to setup an OpenVPN Server and to connect to it using a VPN Client.
Additionally I ill also show how to setup a Free Radius server and a plugin to implement multi-factor authentication for additional security.

1. Installation OpenVPN server on Linux (I will be using a fresh Centos 9 Linux)

```
# yum update
# curl -O https://raw.githubusercontent.com/angristan/openvpn-install/master/openvpn-install.sh
# chmod +x openvpn-install.sh
# ./openvpn-install.sh

```
Accept defaults for installation of OpenVPN and at the end provide a 'Client name' e.g. 'demouser'
I have chosen a passwordless client but if you want you can also add an additional password to protect your private key.
```
Client name: demouser

Do you want to protect the configuration file with a password?
(e.g. encrypt the private key with a password)
   1) Add a passwordless client
   2) Use a password for the client
Select an option [1-2]: 1
...
...
The configuration file has been written to /root/demouser.ovpn.
Download the .ovpn file and import it in your OpenVPN client.
```
Finally a client configuration file is ready to be imported into the VPN Client.

2. Installation 



