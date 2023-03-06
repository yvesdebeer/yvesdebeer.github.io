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

2. Installation OpenVPN client for Windows

Download the OpenVPN Client software from http://openvpn.net/vpn-client

Install the OpenVPN Client:

![]({{site.baseurl}}/https://github.com/yvesdebeer/yvesdebeer.github.io/blob/master/images/OpenVPN-Client-Install.png)

Once the installation is finished we can import the configuration file 'demouser.ovpn' which was generated on the OpenVPN server but before importing we need to modify the IP-address of our OpenVPN server in this file:

```
client
proto udp
explicit-exit-notify
remote **192.168.0.150** 1194
dev tun
resolv-retry infinite
nobind
persist-key
persist-tun
...
```

Normally the remote IP by default will be the address of your public IP which is normal if you have your VPN server on your local network and need remote access from outside this network.
You can leave the public IP address in the config but then you will have to open up the correct port and set the routing on your internet accces point.

![]({{site.baseurl}}/https://github.com/yvesdebeer/yvesdebeer.github.io/blob/master/images/OVPN-Import-Config.png)

Finally we can test the VPN connection. The first time the connection will probably fail as the firewall on the OpenVPN Linux server is blocking the access.
To quickly test this we can just disable the firewall using the command:

```
# systemctl stop firewalld
```

Now the connection should work:

![]({{site.baseurl}}/https://github.com/yvesdebeer/yvesdebeer.github.io/blob/master/images/OpenVPN-Client-Connect.png)

3. In the next step I will describe ho to use Radius with OpenVPN.

First we will install the IBM Security Verify Gateway for RADIUS on a Windows machine.
Documentation is available at: https://www.ibm.com/docs/en/security-verify?topic=integrations-security-verify-gateway-radius

This package can be downloaded from the IBM Security AppExchange here: 
https://exchange.xforce.ibmcloud.com/hub/IdentityandAccess 
(you will need to use your IBMid to login) 

![]({{site.baseurl}}/https://github.com/yvesdebeer/yvesdebeer.github.io/blob/master/images/IBM-Gateway-For-Radius.png)

Extract and run the installation 'setup_radius'.

Edit the Radius configuration file 'c:\Program Files\IBM\IbmRadius\IbmRadiusConfig.json':

```
{
    "address":"::",
    "port":1812,
/*
    "trace-file":"c:/tmp/ibm-auth-api.log",
    "trace-rollover":12697600,
*/
    "ibm-auth-api":{
        "client-id":"???????",
        "obf-client-secret":"???????", /* See IbmRadius -obf "the-secret" */
        "protocol":"https",
        "host":"???????.verify.ibm.com",
        "port":443,
        "max-handles":16
    },
    "clients":[
      {
        "name": "OpenVPN",
        "address": "192.168.0.0",
        "mask": "255.255.0.0",
        "secret": "Passw0rd",
        "auth-method": "password-and-device",
        "use-external-ldap": false,
        "reject-on-missing-auth-method": true,
        "device-prompt": "A push notification has been sent to your device:[%D].",
        "poll-device": true,
        "poll-timeout": 60
      }
    ] 
}
````

Complete the fields 'client-id', 'obf-client-secret' and 'host' with the correct information to point to your IBM Verify Saas API.
Before we can do this we will need to setup API access in IBM Verify Saas.
Login to your environment or go for a trial account if you don't have one : https://www.ibm.com/products/verify-identity

Select Security > API Access > Add API client
Create a new API Client :
- Specify the entitlements by selecting the check bow from the list:
	- Authenticate any user
    - Read authenticator registrations for all users
    - Read users and groups
    - Read second-factor authentication enrollment for all users
- Click next on the following screens and finally give the API client a name : e.g. 'MFA-Client'

A Client-ID and Secret will automatically be created for you. Use this information to complete the readius config. Use the c:\Program Files\IBM\IbmRadius\IbmRadius.exe -obf <client-secret> command to generate the obfuscated secret value.

Finally configure the IBM Radius service to startup automatically and start the service:
  
![]({{site.baseurl}}/https://github.com/yvesdebeer/yvesdebeer.github.io/blob/master/images/IBM-Radius-Service.png)
  
Test Radius Authentication using the Radius tool : NTRadPing (https://ntradping.apponic.com)
You should get a push notification on the IBM Verify app on the iPhone.
  
![]({{site.baseurl}}/https://github.com/yvesdebeer/yvesdebeer.github.io/blob/master/images/NTRadPing.png)
  




