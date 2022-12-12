---
title: Pwn The Network - Discover devices in the network using ARP scan
published: true
---

Once connected to a network, a hacker/penetration tester starts by discovering devices connected to the same network. This is the first part of the information gathering phase in every network pentesting assessment.<br>
There several ways to scan a network, for example, a technique like **ping sweep** is used to send ICMP echo request packets to every device and get responses from connected devices. Sometimes, this method is not reliable enough, as some devices running a firewall may not respond depending on the firewall rules. On the other hand, performing an **ARP scan** is more reliable as all devices must respond to ARP packets even the ones that are running firewalls, otherwise they cannot communicate with any other machine. In this article, we explain what is the ARP and how can we use it to discover devices in a network. As always, we will be implementing the whole thing in python.

### [](#header-3) What is ARP ?
ARP stands for address resolution protocol. This protocol is used to find the MAC address of the device corresponding to its IP address. This protocol aims to create communication between two devices on a local area network (Ethernet) by providing the other device's MAC address. To establish communication between two devices, the source device needs to generate the ARP request message. 

<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/arp-protocol.png" style="width:600px;"><br>
  <em>ARP protocol</em>
</p>
When a new computer joins a local area network (LAN), it will receive a unique IP address to use for identification and communication. <br>
Packets of data arrive at a gateway, destined for a particular host machine. The gateway, or the piece of hardware on a network that allows data to flow from one network to another, asks the ARP program to find a MAC address that matches the IP address. The ARP cache keeps a list of each IP address and its matching MAC address. The ARP cache is dynamic, but users on a network can also configure a static ARP table containing IP addresses and MAC addresses.<br>
In our case, we will use ARP to broadcast a request to the entire network. Devices that are connected to the same network must respond with ARP response.

### [](#header-3) What is a Scapy?
To perform an ARP scan, we need to forge our own packets from scratch, so we can send ARP request to the whole network. To do so, there is no better than the powerful scapy framework. Scapy is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, match requests and replies, and much more.
Scapy mainly does two things: sending packets and receiving answers. You define a set of packets, it sends them, receives answers, matches requests with answers and returns a list of packet couples (request, answer) and a list of unmatched packets.

### [](#header-3) Let’s code
The python script we are going to code is pretty simple to write yet it is a good introduction to scapy as we will use it often in the series **Pwn The Network** articles.<br> We use our script as follows: `python3 arp-scanner.py --network <net_addr>/<subnet_prefix>`

<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/arp-scan.PNG" style="width:600px;"><br>
  <em>ARP protocol</em>
</p>

We will only need the `scapy` and `argparse` libraries to parse the network address we want to scan.<br>
To install scapy, use `pip install scapy`
```python
import scapy.all as scapy
import argparse
```
The `scan` function is defined to take in the IP address of the network as an argument and return a list of answered ARP (Address Resolution Protocol) requests. An ARP request is a packet that is broadcasted to all devices on a network to find their MAC (Media Access Control) addresses. The function creates an ARP request packet and a broadcast packet, combines them, and sends them to the specified network using the `scapy.srp` function. The function then returns the list of answered requests.

```python
#define the function to scan the network
def scan(ip):
    arp_request = scapy.ARP(pdst=ip) #create an ARP request packet
    broadcast = scapy.Ether(dst='ff:ff:ff:ff:ff:ff') #create a broadcast packet
    arp_request_broadcast = broadcast/arp_request #combine the ARP request and broadcast packet
    answered_list= scapy.srp(arp_request_broadcast,timeout=1,verbose=False)[0] #send the packet and store the response in a list
    return answered_list #return the response
```
The `get_mac_vendor` function is defined to take in the MAC address of a device and return its vendor. The function first formats the MAC address by converting it to upper case and removing the colons. It then opens the `mac-vendor.txt` file and searches for the MAC address in each line of the file. If the MAC address is found, the function returns the vendor name from the line. If the MAC address is not found, the function returns 'Unknown'.

```python
#define the function to get the vendor of the given mac address
def get_mac_vendor(mac):
    mac = mac.upper().replace(':','')[0:6] #convert the mac address to upper case, remove the colons and get the first 6 characters
    with open("mac-vendor.txt","r") as f: #open the 'mac-vendor.txt' file in read mode
        for line in f : #iterate through the lines in the file
            if mac in line: #if the mac address is in the line
                return line[7:] #return the vendor name from the line
    return 'Unknown' #if the mac address is not found in the file, return 'Unknown'
```
The `main` function is defined as the entry point of the script. It creates a new `ArgumentParser` object and adds a required command line argument `network` of type string. The function then parses the command line arguments, scans the network using the `scan` function, and iterates through the list of hosts returned by the `scan` function. For each host, the function gets the vendor using the `get_mac_vendor` function and prints the IP address, MAC address and vendor of the host.

```python
#define the main function
def main():
    parser = argparse.ArgumentParser() #create a new ArgumentParser object
    parser.add_argument('-n','--network', type=str, required=True) #add a new argument 'network' which is required and is a string
    args = parser.parse_args() #parse the command line arguments
    hosts = scan(args.network) #scan the network and get the response
    for host in hosts: #iterate through the hosts in the response
        mac_vendor = get_mac_vendor(host[1].src).strip() #get the vendor of the host
        print(host[0].pdst + 2*'\t' + host[1].src + 2*'\t' + mac_vendor) #print the ip address, mac address and vendor of the host
```
### [](#header-3) Conclusion
A good understanding of ARP is necessary of every hacker/penetration tester interested in network hacking. It is used to conduct MITM attacks to intercept network traffic and perform even more sophisticated attacks.<br>
Full code is available on [github](https://github.com/Ahmed-Z/arp-scanner). <br>
HAPPY HACKING.
