---
id: lab2
title: Lab 2 - Logically Addressing the Network, NAT, and FRR
sidebar_position: 2
description: Assigning IP addresses to all NICs, Setting up NAT on deb-router-1, and setting up routing with FRR OSPF
---

# Lab 2 - Logically Addressing the Network, NAT, and FRR

## Overview

This week's lab will cover the following:

- Logically addressing all the devices on our network
- Configuring NAT on our gateway to the Internet
- Manually routing so all devices can:
  - Ping their local subnet
  - Ping other subnets
  - Ping the Internet


## Lab 2 Notes

We will be using a basic IP subnetting scheme in this lab to keep things as simple as possible. However, it is still a good idea to keep a chart handy of all IP addresses you are using matched up with which NICs they are assigned to. This will make it easier for you to test your connections moving forward.

We are going to focus on a suite of tools called Free Range Routing (FRR) to set up the Open Shortest Path First (OSPF) routing on our network. This is not the only way to set up routing. In fact, there are many different methods that can be used to accomplish what we will do in this lab. We will be using FRR because it is clean and straightforward for the most part. You will also likely be using FRR next semester. But always remember that the "best" method to accomplishing something in Linux is often "in the eye of the eye admin".

Finally, you will likely notice that we mostly ignore hostnames/DNS in this lab and focus on IP addresses. This will change in lab 3 when we install and configure an internal DNS server for our network. Keep in mind that this means it is very important you stick to the IP address scheme we lay out in this lab as any deviations will likely cause problems in lab 3.

## Investigation 1: Logically Addressing the Network

Begin by turning on all your VMs. Run "ip address show" on deb-router-1, deb-router-2, and mint-client. Run "ipconfig" on win-client. Notice that right now, only deb-router-1 has a usable IP address.

We are now going to assign IP addresses manually to our VMs using the following addressing scheme: 

- The “backbone” network will use 192.168.100.0/24 
- The “cn1” network will use 192.168.125.0/24 
- The “cn2” network will use 192.168.150.0/24 
- We are going to leave the “default” network as it is because it is going to act as our path to the internet and it is already configured to act as such.  

It is very important that we address all the NICs correctly in this next section. Again, it is recommended that you make note of the IP addresses and the MAC addresses that they are attached to in case we need to refer to them later. 

We will begin by addressing the NICs on deb-router-1. 

Run “ip address show” in deb-router-1 and note the names of the NICs (ex enp7s0) and the MAC address associated with it. Use that MAC address to compare to the NIC info in the VM details screen to know which NICs are connected to which networks. The NIC that is already attached to the “default” NAT network will already have an IP address so we do not need to change it. It will likely be labelled “enp1s0” 

The next NIC (likely labelled enp7s0) should be the one that is connected to the “backbone” network. To address this interface, make the following entry in your /etc/network/interfaces file (note: do not remove any pre-existing configurations from this file and use the correct interface labels per your own system if they are diffferent from these instructions): 
```bash
# interface attached to “backbone” network 
auto enp7s0 
iface enp7s0 inet static 
	address 192.168.100.11/24 
```

While we are in the interfaces file we will also address the next NIC, the one that is attached to the “cn1” network: 
```bash
# interface attached to the “cn1” network 
auto enp8s0 
iface enp8s0 inet static 
  address 192.168.125.11/24 
```

Reboot your deb-router-1 VM. Log back into the VM and run the “ip address show” command. 

You should see 4 NICs in total:  

- the loopback with 127.0.0.1 
- enp1s0 has a 192.168.x.x address received by DHCP 
- enp7s0 has 192.168.100.11 
- enp8s0 has 192.168.125.11 

Next, we will address deb-router-2 

Use the instructions you were provided to configure deb-router-1's NICs to configure deb-router-2's NICs with the following addresses: 

- enp7s0 has 192.168.100.12/24 (attached to the “backbone” network) 
- enp8s0 has 192.168.150.11/24 (attached to the “cn2” network) 

When you have successfully made these configurations, try pinging between deb-router-1 and deb-router-2 using the 192.168.100.0/24 IP addresses we have assigned so far to ensure connectivity between the two routers. 

Try to ping 192.168.150.11 from deb-router-1. Try to ping 192.168.125.11 from deb-router-2. 
Why do you think these pings failed? 
Don’t worry, we will fix this shortly. 

Next we will address our mint-client VM. Log into your mint-client VM and make sure your deb-router-1 VM is also powered on. 

- Click on the Linux Mint logo in the very bottom left of the desktop 
- Type “advanced network configuration” and press “Enter” 
- Double-click on “Wired connection 1” 
- Click on “IPv4 settings" 
- Change “method “to “Manual” 
- Click “Add” and enter the following: 
  - Address: 192.168.125.12 
  - Netmask: 255.255.255.0 
  - Gateway: 192.168.125.11 
  - Leave the DNS servers blank for now 

- Click “Save” and reboot your mint-client 

When mint-client reboots, open a terminal and run “ip address show” to confirm that your single interface is addressed with 192.168.125.12. Then, run “ping 192.168.125.11" to ensure your mint-client can connect to your deb-router 1. 

Try pinging 192.168.100.11 from your mint-client. Why did this succeed? 

Try pinging 192.168.100.12 from your mint-client. Why did this fail? 

Next we will address our win-client VM. Log into your win-client VM and make sure your deb-router-2 VM is also powered on. 

- In Windows, click on the Windows icon at the bottom of the screen, type “network”, and click on “View network connections” 
- Right-click on the “Ethernet” device and click on “Properties” 
- Double-click on “Internet Protocol Version 4 (TCP/IPv4)” 
- Click “Use the following IP address and put in the following configuration: 
  - IP address: 192.168.150.12 
  - Subnet Mask: 255.255.255.0 
  - Default Gateway: 192.168.150.11 
  - Leave the DNS fields empty for now 

- Click “OK” 

Open the command line and run “ipconfig” to confirm your NIC is addressed correctly. Then ping 192.168.150.11 to ensure your win-client can connect with your deb-router-2. 

Try pinging 192.168.100.12. Why was it successful? 

Try pinging 192.168.100.11. Why did it fail? 

We have now successfully addressed all of the NICs on our network. At this point we have a basic level of connectivity between our VMs in that we have the following capability: 

- deb-router-1 and deb-router-2 can connect to each other 
- mint-client and deb-router 1 can connect to each other 
- win-client and deb-router-2 can connect to each other 

However, mint-client and win-client cannot connect to each other and no VMs other than deb-router-1 can connect to the internet. Next, we will rectify these problems to allow full connectivity on our network. 

## Investigation 2: Enabling Internet Connectivity for mint-client

Our mint-client VM is actually already pset up to access the Internet via deb-router-1 because it is already directly connected to it. It doesn't need any additional information to know that data that is dentined for the Internet must be sent to deb-router-1 because it already sends **ALL** of its data to deb-router-1. However, deb-router-1 is not yet configured to allow data from our network to travel through it and out onto the Internet. To get this working we have to make two very important configurations.

First, we are going to turn on IP forwarding which is pretty much what it sounds like. It allows an OS (like Debian) to accept incoming network packets on an interface, determine their destination, and pass them on to another interface just like a router does. 

**NOTE – IT IS VERY IMPORTANT THAT THIS STEP IS NOT MISSED. IT MUST BE DONE BEFORE FRR IS INSTALLED LATER IN THIS LAB. FAILURE TO DO SO WILL RESULT IN YOU HAVING TO REINSTALL YOUR DEB-ROUTER-1 VM. YOU HAVE BEEN WARNED!** 

To begin, make sure that deb-router-1 and mint-client are both turned on and can ping one another using the 192.168.125.0/24 network. Also, confirm that deb-router-1 has internet connectivity by pinging google.com. Once those have been done successfully, we will enable IP forwarding on deb-router-1. 

Log into deb-router-1 and run:
```bash
sudo sysctl net.ipv4.ip_forward
```

The resulting output should show that the parameter you input is set to 0, meaning IP forwarding is currently turned off. We are going to make a change to the system to enable this. 

Create/open the file “/etc/sysctl.d/ipforward.conf” and enter the following inside: 
```bash
# allow IP forwarding 
net.ipv4.ip_forward=1 
```

Reboot your deb-router-1 VM. Log back in and run the sysctl command again:
```bash
sudo sysctl net.ipv4.ip_forward
```
 The output should now show the parameter is set to 1, meaning IP forwarding has now been turned on.   

**NOTE – AGAIN, IT IS VERY IMPORTANT THAT THIS STEP IS NOT MISSED. IT MUST BE COMPLETED SUCCESSFULLY BEFORE FRR IS INSTALLED LATER IN THIS LAB. IF YOU ARE NOT GETTING A "1" HERE, CHECK WITH YOUR TEACHER BEFORE MOVING FORWARD.** 

Next, we are going to allow deb-router-1 to act as a NAT device between our internal network and the outside world (although keep in mind that because this is a virtual network, we still technically have another layer of NAT already running in the form of our Host Ubuntu system). 

First, install iptables onto deb-router-1:
```bash
sudo apt install iptables” 
```

Next, add the following special iptables rule: 
```bash
sudo iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE 
```

Then, make your iptables persistent just like we did in OPS245: 
```bash
sudo iptables-save -f /etc/iptables.rules
```

Create a script file at “/etc/network/if-pre-up.d/iptables”.

Enter the following into the “iptables” script file:
```bash
#!/bin/bash 
 
/sbin/iptables-restore /etc/iptables.rules 
```

Save the file and run:
```bash
sudo chmod u+x /etc/network/if-pre-up.d/iptables
```

This will ensure the system can run the script at startup.

Reboot your system and run:
```bash
sudo iptables –L POSTROUTING –t nat
```

If you saved your iptables correctly you should see the MASQUERADE rule that allows from anywhere to anywhere. 

What we have just done is turned deb-router-1 into a NAT device. Other devices that know to look to deb-router-1 to exit the internal virtual network will now be able to do so. Let’s test this out. Turn on mint-client if it is not already.  

On mint-client, open a terminal and run the following commands: 
```bash
ping google.com 
ping 8.8.8.8 
```

Why did the first ping fail? Why did the second succeed? 

Recall that when we configured mint-client, we left the DNS server field empty. The mint-client does not know where to look to get answers to DNS queries. It can contact the Internet through deb-router-1 but it can only do so with the use of IP addresses. In Lab 3 we will be setting up a permanent solution to this but for now we will use a temporary one. 

Go back to the Wired connection 1 configuration window in mint-client 

In the “DNS servers” field enter “8.8.8.8”. 8.8.8.8 is  the IP address of one of Google’s free DNS resolvers. 

Reboot your mint-client 

Open a terminal and try pinging google.com again. The ping should now succeed because mint-client has access to the Internet through deb-router-1 and it knows to send its DNS queries to 8.8.8.8 (google). 

## Investigation 3: Enabling Internal and External Connectivy for the Entire Network using FRR

Power on your deb-router-2 and win-client VMs and try to ping 8.8.8.8 from them. What happened? 

Right now, neither of those VMs knows how to send data out of the network because they have no way of knowing that deb-router-1 is the way out. The mint-client VM knows because it is directly connected to deb-router-1 and we set its default gateway to deb-router-1's IP address. When it tries to send data out of the network, it knows to send it to deb-router-1 (because deb-router-1 is literally the only thing it **can** send data to). The next steps will be to set up deb-router-1 and deb-router-2 with a service called FRR/OSPF. When properly configured, this will allow all devices on the network to access the Internet via deb-router-1 and it will also allow all devices on our network to connect to one another as well. 

Start by confirming that IP forwarding is still enabled on deb-router-1:
```bash
sudo sysctl net.ipv4.ip_forward 
```

The result should show “net.ipv4.ip_forward=1”. **FINAL WARNING - If it shows 0, make sure you go back and rectify this before continuing past this point.** 

Next we will install FRR (Free Range Routing) and its OSPF (Open Shortest Path First) service on deb-router-1. 

On deb-router-1: 

- Update your system: 
```bash
sudo apt update && sudo apt upgrade 
```

- Install FRR:
```bash
sudo apt install frr frr-pythontools 
```

Once installed, we will enable the necessary daemon: 

- Open the “/etc/frr/daemons” file and set “ospfd=yes”. Save your changes.  
- Restart the frr service:  
```bash
sudo systemctl restart frr  
```

- Reboot deb-router-1  
- Log back in and check to make sure that IP forwarding is still turned on and that the frr service is running:  
```bash
sudo systctl net.ipv4.ip_forward
```
The output should still be “net.ipv4.ip_forward=1”.

```bash
sudo systemctl status frr 
```

The output should show enabled and active (you can ignore any errors below the initial output for now).  

We also need to install FRR/OPSF on deb-router-2 to properly route our whole network but we have a problem. We used the “apt install” command on deb-router-1 to install FRR and its dependencies. But this won’t work on deb-router-2 because right now it has no way to exit the local network and access the Internet. So how can we solve this problem? 

Not to worry, we have several options available to us. We are going to use a combination of tools that we have learned about in OPS145 and OPS245. 

First, power on and log into deb-router-2 and make sure that deb-router-1 and deb-router-2 can ping one another. Then set up IP forwarding on deb-router-2 just like we did with deb-router-1. 

On deb-router-2, enable IP forwarding by creating/opening the file “/etc/sysctl.d/ipforward.conf” and enter the following inside: 
```bash
#allow IP forwarding 
net.ipv4.ip_forward=1 
```

Reboot your deb-router-2 VM. Log back in and run “sudo sysctl net.ipv4.ip_forward". The output should show the parameter is set to 1, meaning IP forwarding has now been turned on. **NOTE – LIKE DEB-ROUTER-1, THIS MUST BE DONE BEFORE FRR IS INSTALLED** 

Next we will download the necessary packages and store them as local files on deb-router-1 so we can manually send them over to deb-router-2. 

On deb-router-1:  
- Download and store the necessary FRR packages and all of its dependencies.  
```bash
sudo apt-get download frr frr-pythontools libcares2 liblua5.3-0 libunwind8 libyang3 
```

- List all the files in your current directory. You should see 6 .deb files 
- Now use scp to copy these files over to deb-router-2 (change “hheim” to your user name):
```bash
scp *.deb hheim@192.168.100.12: 
```

On deb-router-2: 
- List the files in your current directory to make sure the files were copied correctly 
- If they are all there, install them:
```bash
sudo dpkg –i *.deb 
```

If any errors occur, go back and double check that all files were downloaded and transferred over to deb-router-2 correctly.  

Next we can configure FRR/OSPF on deb-router-2: 
- Open the “/etc/frr/daemons” file and set “ospfd=yes”. Save your changes.  
- Restart the frr service:  
```bash
sudo systemctl restart frr  
```

- Reboot deb-router-2  
- Log back in and check to make sure that IP forwarding is still turned on and that the FRR service is running:  
```bash
sudo systctl net.ipv4.ip_forward 
```
Output should be "net.ipv4.ip_forward=1".

```bash
sudo systemctl status frr 
```

Output should show enabled and active (you can ignore any errors below the initial output for now).

Now we are going to configure deb-router-1 and deb-router-2 with OSPF routing entries so that they both know about all subnets connected in the local network as well as how to get to the internet through deb-router-1. 

On deb-router-1 run the following commands: 
```bash
sudo vtysh 
configure terminal 
router ospf 
network 192.168.100.0/24 area 0 
network 192.168.125.0/24 area 0 
default-information originate always 
exit 
exit 
write memory 
```

On deb-router-2 run the following commands: 
```bash
sudo vtysh 
configure terminal 
router ospf 
network 192.168.100.0/24 area 0 
network 192.168.150.0/24 area 0 
exit 
exit 
write memory 
```

What we have just done is populated our routers with entries for the networks that they are directly attached to and which they will advertise to neighboring routers. Run the following on both routers to check that they can see each other as routers and that they are sharing their local networks with one another (note, these commands are run after running the "sudo vtysh" command, not on the VM's main terminal): 
```bash
show ip ospf neighbor 
show ip route ospf 
```

The results of these commands provide some important information. The first command shows us the other OSPF router(s) that are connected to this system. Right now there is only one entry for deb-router-1 and deb-router-2 because they are each only connected to one another. The second command shows us which networks are directly connected to the router and which networks are connected to other routers.  

Of particular note is the “0.0.0.0” entry you should see on deb-router-2. The “default-information originate always” command we entered into deb-router-1's OSPF routing entries specified that any traffic with a destination that is not inside our local network is to be sent to deb-router-1. Combined with the NAT setup we added to deb-router-1 earlier, our internal devices can now find the Internet whether they are directly connected to deb-router-1 or not.  

Furthermore (and just as importantly) they can now all find each other. Let’s test this out.  

Power on mint-client and win-client if they aren’t already. 

On win-client, ping mint-client:
```bash
ping 192.168.125.12 
```

This should succeed. 

On mint-client, ping win-client: 
```bash
ping 192.168.150.12 
```

This will still fail, but not because of our network setup. 

- Go into win-client
- click the windows icon
- type “firewall” and click on “Windows Defender Firewall”
- click on “Turn Windows Defender Firewall on or off” on the left hand side
- click both “Turn off Windows Defender Firewall” buttons here and then click “OK”  

Now, go back to mint-client and try the ping one more time. 
```bash
ping 192.168.150.12 
```

This time, the ping should succeed. 

At this point, all the devices in our network can ping one another by their IP address. Our deb-router-1 and mint-client VMs can ping sites on the Internet by IP address and by FQDNs. And while our deb-router-2 and win-client VMs can ping sites on the Internet by IP address, they cannot ping them by FQDN.  

This problem could be easily fixed by adding the same DNS server to their network configurations that we added to mint-client (8.8.8.8). We are not going to do this, however. Instead, in the next lab, we are going to set up deb-router-1 as a local DNS server that will be able to handle queries regarding the local network as well as queries to external endpoints (the Internet). 

## Lab 2 Sign-Off

Take screenshots showing the following: 

- The output of the "ip address show" command on:
  - deb-router-1
  - deb-router-2
  - mint-client
- The output of the "ipconfig" command on win-client
- The output of the "show ip ospf neighbor" command in the FRR terminal in deb-router-1 and deb-router-2
- The output of the "show ip route ospf" command in the FRR terminal in deb-router-1 and deb-router-2
- A successful ping from mint-client to win-client

The following Exploration Questions are for furthering your knowledge only, and may appear on quizzes or tests at any time later in this course.

## Exploration Questions

1. 
