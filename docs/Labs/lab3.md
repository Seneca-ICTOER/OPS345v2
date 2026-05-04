---
id: lab3
title: Lab 3 - DNS and Samba Server
sidebar_position: 3
description: Configure a DNS server on the local network that will serve queries to local and external destinations, set up Samba for Linux-Windows file sharing
---

# Lab 3 - Configuring DNS and Samba Server

## Overview

This week's lab will cover the following:

- Establishing a FQDN scheme for the local network
- Configuring deb-router-1 as a DNS server
- Enabling query resolving for both local and external destinations
- Installing and configuring Samba server for file transfers between mint-client and win-client

## Lab 3 Notes

The goal of this lab is to turn deb-router-1 into a DNS server that will provide answers for DNS queries regarding our local network as well as the Internet. Once it is properly configured we will configure all of our VMs to look to deb-router-1 for all DNS information. We will also be setting up Samba server at the end of this lab as this will provide a service that will be useful to us in Lab 4. 

As was mentioned at the end of Lab 3, we don’t necessarily need a local DNS server for Internet-based queries. We could use 8.8.8.8 or another publicly available DNS provided on the Internet for all of our VMs in our current network setup. But we are going to add that functionality so that you can learn how it is done because there are scenarios that require such a configuration and you won’t generally be relying on free resolvers.  

What we can’t get from the Internet is local FQDN to IP translations and that is what we will be primarily focusing on in this lab. When we are finished, all of our VMs will be reachable by both the IP addresses that we assigned in Lab 2 as well as a Fully Qualified Domain Name that we will be establishing in this lab. 

It is important to note that the FQDNs that we will be using will be established by our new DNS server - they will be created and mapped as our DNS server is configured. These names have nothing to do with the current hostnames of our VMs at this point. Even though we will be using the same hostnames in our FQDN scheme, that is a decision that we are making to keep things simple and prevent confusion. But from a technical perspective, we could use whatever names we wanted for our FQDNs, regardless of what the hostnames are. 

Finally, we are going to use a simple Domain naming scheme where our backbone will be referred to as “ops345.org” and our subnets will be referred to as subdomains “cnet1.ops345.org” and “cnet2.ops345.org”. Again, these distinctions are not predetermined by the structure of our network. We are simply choosing to use them as a way of organizing our devices/subnets. As such, our routers will have two FQDNs each (one for each interface) and both will work for any connecting/testing purposes when we are done this lab. This will be important to remember in Assignment 1... 

## Before You Get Started...

Before you begin this lab, double check that the configurations from Lab 2 are still working. You should be able to: 
- Ping 192.168.150.12 (win-client) from mint-client
- Ping 192.168.125.12 (mint-client) from win-client
- Ping 8.8.8.8 from all VMs
- Ping www.google.com from deb-router-1 and mint-client but NOT from deb-router-2 or win-client 

If all that is correct you can proceed with Lab 3. Otherwise, please double check with your professor. It is very important that your network is properly configured before moving forward so that you don’t mistake a network configuration problem for a DNS configuration problem later. 

## Investigation 1: Installing Bind and Configuring Basic Options

To get started, log into deb-router-1 and download and install “bind9” and utilities: 
```bash
apt install bind9 bind9-utils bind9-dnsutils 
```

Then, check the status of bind9. It should already be running. If it is not, enable it and reboot your system. 
```bash
sudo systemctl status bind9 
```

Navigate to the “/etc/bind/” directory. 

**Please note that you will be creating and modifying several configuration files moving forward.**
**Typos are the number 1 reason your configurations may not work the first time around.** 
**Double and triple and quadruple check your config files and make sure they are in the right place!**

Change the “/etc/bind/named.conf.options” file to contain the following: 
```bash
options { 
    directory "/var/cache/bind";

    // Listen on deb-router-1 only 

    listen-on { 
        127.0.0.1; 
        192.168.100.11; 
        192.168.125.11; 
    }; 

    listen-on-v6 { none; };

    // Internal networks allowed 

    allow-query { 
        127.0.0.1; 
        192.168.100.0/24; 
        192.168.125.0/24; 
        192.168.150.0/24; 
    }; 

    allow-recursion { 
        127.0.0.1; 
        192.168.100.0/24; 
        192.168.125.0/24; 
        192.168.150.0/24; 
    }; 

    recursion yes; 

    // Upstream resolvers 

    forwarders { 
        1.1.1.1; 
        8.8.8.8; 
    }; 

    forward only; 

    dnssec-validation auto; 

    minimal-responses yes; 

}; 
```

Double check your work and save your changes.

The file you just modified establishes several important configurations for our DNS server. First, it is dictating which interfaces to listen for DNS queries on (the interfaces attached to deb-router-1). It is also specifying which networks it will allow DNS queries from (our 3 subnets; important because you don't want any unknowns making query requests). It is also specifying which networks are allowed to ask the server about queries not local to our network. And then we have two IP addresses that will act as DNS servers that our DNS server will query for FQDNs it does not know (1.1.1.1 and 8.8.8.8 ). This last part will allow us to point all of our local devices to deb-router-1 for DNS queries for destinations external to our network (the Internet).

## Investigation 2: Creating and Configuring Lookup Zones

Next we will establish our forward and reverse lookup zones and map them to their own specific configuration files which we will configure in a moment. This is done via a configuration file that points to other configuration files for detailed information on that particular zone.

Change the “/etc/bind/named.conf.local” file to contain: 
```bash
//forward lookup zones
zone "ops345.org" { 
    type master; 
    file "/etc/bind/db.ops345.org"; 
}; 

zone "cnet1.ops345.org" { 
    type master; 
    file "/etc/bind/db.cnet1.ops345.org"; 
}; 

zone "cnet2.ops345.ca" { 
    type master; 
    file "/etc/bind/db.cnet2.ops345.org"; 
}; 

//reverse lookup zones 
zone "100.168.192.in-addr.arpa" { 
    type master; 
    file "/etc/bind/db.192.168.100"; 
}; 

zone "125.168.192.in-addr.arpa" { 
    type master; 
    file "/etc/bind/db.192.168.125"; 
}; 

zone "150.168.192.in-addr.arpa" { 
    type master; 
    file "/etc/bind/db.192.168.150"; 
}; 
```

Save your changes.

Next we will create our parent zone configuration files that we mapped to in the previous step. 

Create the “/etc/bind/db.ops345.org” forward lookup zone configuration file and enter the following inside (be very careful with spacing): 
```bash
$TTL 3600 
@ IN SOA deb-router-1.ops345.org. admin.ops345.org. ( 
    2026031202
    3600 
    900 
    1209600 
    300 
) 

  IN NS deb-router-1.ops345.org. 

deb-router-1 IN A 192.168.100.11 
deb-router-2 IN A 192.168.100.12

; Delegate subdomains 
cnet1 IN NS deb-router-1.ops345.org. 
cnet2 IN NS deb-router-1.ops345.org. 
```

Save your changes.

Before moving on to the next configuration files, let's just briefly break this file down so we understand what is happening at each line. Keep in mind that no one else will be accessing your DNS server other than you so some of these parameters don't really matter and the timer values are somewhat arbitrary in our set up but are very important in real set ups.

- The "$TTL" line specifies the default time in seconds that records should be cached by resolvers
- The SOA line defines who the authority for DNS records are locally along with the following:
  - a serial number that is updated every time the zone file changes
  - The Refresh timer: the time in seconds a secondary server waits before asking the primary server for the SOA record to check for changes
  - The Retry timer: The time in seconds a secondary server waits before retrying a failed refresh request
  - The Expire timer: The time in seconds a secondary server will continue to hold the zone data as valid if it cannot reach the primary server
  - The Minimum timer: the time (in seconds) a resolver should cache negative responses
- The next lines designate the A records mapping deb-router-1 and deb-router-2 to their IP addresses as noted by the Name Server on deb-router-1.
- And finally the subdomains (cnet1 and cnet2) which our client VMs are located on are delegated as a part of the main ops345.org domain

Moving on, create the “/etc/bind/db.cnet1.ops345.org” forward lookup zone configuration file and enter the following inside: 
```bash
$TTL 3600 
@ IN SOA deb-router-1.ops345.org. admin.ops345.org. ( 
    2026031201 
    3600 
    900 
    1209600 
    300 
) 

  IN NS deb-router-1.ops345.org. 

mint-client IN A 192.168.125.12 
deb-router-1 IN A 192.168.125.11 
```

Save your changes.

Create the “/etc/bind/db.cnet2.ops345.org” forward lookup zone configuration file and enter the following inside: 
```bash
$TTL 3600 
@ IN SOA deb-router-1.ops345.org. admin.ops345.org. ( 
    2026031201 
    3600 
    900 
    1209600 
    300 
) 

  IN NS deb-router-1.ops345.org. 

win-client IN A 192.168.150.12 
deb-router-2 IN A 192.168.150.11 
```

Save your changes.

Create the “/etc/bind/db.192.168.100” reverse lookup zone config file and enter the following inside: 
```bash
$TTL 3600 
@ IN SOA deb-router-1.ops345.org. admin.ops345.org. ( 
    2026031201 
    3600 
    900 
    1209600 
    300  
) 

  IN NS deb-router-1.ops345.org. 

11 IN PTR deb-router-1.ops345.org. 
12 IN PTR deb-router-2.ops345.org. 
```

Save your changes.

Note that the reverse lookup zone config files are reverse maps of what we provided in the forward lookup zone config files.

Create the “/etc/bind/db.192.168.125” reverse lookup zone config file and enter the following inside: 
```bash
$TTL 3600 
@ IN SOA deb-router-1.ops345.org. admin.ops345.org. ( 
    2026031201 
    3600 
    900 
    1209600 
    300
) 

  IN NS deb-router-1.ops345.org. 

11 IN PTR deb-router-1.cnet1.ops345.org. 
12 IN PTR mint-client.cnet1.ops345.org. 
```

Save your changes.

Create the “/etc/bind/db.192.168.150” reverse lookup zone config file and enter the following inside: 
```bash
$TTL 3600 
@ IN SOA deb-router-1.ops345.org. admin.ops345.org. ( 
    2026031201 
    3600 
    900 
    1209600 
    300
) 

  IN NS deb-router-1.ops345.org. 

11 IN PTR deb-router-2.cnet2.ops345.org. 
12 IN PTR win-client.cnet2.ops345.org. 
```

Save your changes.



## Investigation 3: Checking Configuration File Syntax and Testing the DNS Server

When you have completed filling out your files, run the commands below. These commands will test your DNS configuration files and let you know if there are any typos. Keep in mind that these tests are NOT perfect. You may still need to double check your configuration files if your DNS server fails to start or answer queries properly. 
```bash
sudo named-checkconf 
sudo named-checkzone ops345.org /etc/bind/db.ops345.org 
sudo named-checkzone cnet1.ops345.org /etc/bind/db.cnet1.ops345.org 
sudo named-checkzone cnet2.ops345.org /etc/bind/db.cnet2.ops345.org 
sudo named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100 
sudo named-checkzone 125.168.192.in-addr.arpa /etc/bind/db.192.168.125 
sudo named-checkzone 150.168.192.in-addr.arpa /etc/bind/db.192.168.150 
```
 
The last thing we need to do on deb-router-1 is to change its default nameserver to itself. Open the “/etc/resolv.conf” file and add the following: 
```bash
nameserver 192.168.100.11 
```

Save your changes.

Normally, this would be enough to ensure that deb-router-1 will use itself for DNS queries. However, due to the virtual nature of our network, our setup is layered in such a way that we have deb-router-1 acting as a NAT device underneath our host machine. Our host machine is also providing NAT and giving deb-router-1 an IP address it can use to get to the internet via a form of DHCP. Because of this, DHCP will override the “/etc/resolv.conf” file on startup so we are going to use a special command to lock that file so that other services cannot write to it. 

Run the following command: 
```bash
chattr +i /etc/resolv.conf 
```

This will render the “/etc/resolv.conf” file immutable so that no one on the system (not even root) can change it. 

Once that is done, restart deb-router-1 and run: 
```bash
nslookup 
server 
```
The result should be deb-router-1's IP address (192.168.100.11) 

Now run:  
```bash
dig google.com 
```
Again, you should see that deb-router-1's IP address is showing up as the server providing answers to the query. 

Now all that’s left to do is make sure that all VMs are using deb-router-1 (192.168.100.11) as their primary DNS server 

On deb-router-2, change the /etc/resolv.conf file to:
```bash
nameserver 192.168.100.11
```

On mint-client: 
- Go to "Advanced Network Configuration"
- Click on "Wired Connection 1"
- Click on "IPv4 Settings"
- Change “DNS Servers” to 192.168.100.11 
- Save your changes

On win-client: 
- Go to "View Network Connections"
- Click on "Ethernet" 
- Click on "Properties"
- Click on "IPv4"  
- Add 192.168.100.11 to “Preferred DNS server” 
- Save your changes

Restart all of your VMs. 

When they all come back on, test that your DNS server is working for everything with the following commands: 

On mint-client: 
```bash
ping google.com 
ping win-client.cnet2.ops345.org 
ping deb-router-2.ops345.org 
```
On win-client: 
```bash
ping google.com 
ping mint-client.cnet1.ops345.org 
ping deb-router-1.ops345.org 
```

If everything works, then CONGRATULATIONS! Your network is now fully supported by DNS both internally and externally.  

This would be a good time to make sure all your VMs are updated. 

## Investigation 4: Configuring and Testing Samba Server

Samba Server is a tool that allows for the trasnferring of files easily between Linux and Windows operating systems. While it is not the only method to accomplish this, it is one that is very easy to use once set up correctly. We will be using Samba Server in Lab 4 when we look at containers. 

Before we begin, make sure that your mint-client has internet access and is updated. 

Then install Samba onto mint-client: 
```bash
sudo apt install samba 
```

Check to make sure the samba services are running: 
```bash
sudo systemctl status smbd 
sudo systemctl status nmbd 
```

If either service is disabled, enable it and reboot mint-client. 

Next, add your user to the "sambashare" group and set a samba password for your user (replace “hheim” with your username and use the same password you have been using for everything else). 
```bash
sudo usermod –aG sambashare hheim 
sudo smbpasswd –a hheim 
```

Now you will create a directory in your home directory that will be our special Samba server directory: 
```bash
cd ~ 
sudo mkdir sambadir  
```

Change the ownership of this directory to the Samba service and change its permissions for access: 
```bash
sudo chgrp sambashare sambadir 
sudo chmod 2770 sambadir 
```

The 2770 permission is special in that the 2 allows new files/subdirectories created in the directory to inherit the directory's group ownership rather than the creator's primary group. This is necessary since we will be accessing this directory from another system (our win-client). The 770 is just normal permissions as usual. 

Next, we will configure Samba Server. First, backup the current Samba configuration file in case we need to revert to it later (you should do this with all configuration files). 
```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bkp 
```

Now go into the “/etc/samba/smb.conf” file and add the following to the end of the file (don’t forget to replace hheim with your username): 
```bash
[Share]  
path = /home/hheim/sambadir 
browseable = yes 
writable = yes 
valid users = hheim 
create mask = 0660 
directory mask = 2770 
```

Save your changes and reboot your mint-client. When it comes back up, double check that your Samba server is running: 
```bash
sudo systemctl status smbd 
sudo systemctl status nmbd 
```

Next, in your win-client: 

- Navigate to the file browser. On the left hand side you should see “Network”.
- Right-click on “Network” and click on “Map network drive...”
- Select “Drive H:” and enter the following for “Folder:”
  - "\\mint-client.cnet1.ops345.org\Share"
- Click “Finish”. 
- Enter the credentials you set up for Samba Server (your username and password) and click “OK”.

You should now be in the sambadir directory on your win-client. Keep your win-client vm on your screen. 

In your mint-client: 
- navigate into the sambadir directory
- create an empty file called “test.txt”.

You should see that file appear in the mapped network drive to sambadir in your win-client. In your win-client:
- Double click the newly created file. This should open the file in Notepad.
- Type in “Hello World!” and hit “Enter”.
- Save the file and then close it.  

Back in your mint-client, run: 
```bash
cat test.txt 
```

You should see the changes you made to the file in win-client. 

Close the file browser in your win-client and then re-open it. Where did your Samba server mapped network drive go? 

Click on “This PC”. You should see your “Share” listed there.  Double click on it to access it again. Now close it and restart win-client. Notice that when you go back into the file browser, your share will still be there. 

Power off your win-client and your mint-client.  

Power your win-client back on and try to go to the share. There should be a red “X” on the share. 

Power on your mint-client. After logging in, go back to your win-client and try to open the share again. It should work but be advised that moving forward, when you are moving files between your mint-client and win-client always be sure to turn on mint-client first. 

You now have an easy way to share files between mint-client and win-client. 

## Lab 3 Sign-Off

Take screenshots showing the following:

- A successful ping from mint-client to win-client using win-client's FQDN
- A successful ping from win-client to mint-client using mint-client's FQDN
- A successful ping from deb-router-1 to deb-router-2 using deb-router-2's FQDN
- A successful ping from deb-router-2 to deb-router-1 using deb-router-1's FQDN
- The test.txt file in the "\\mint-client.cnet1.ops345.org\Share" on win-client

## Exploration Questions

1. 
