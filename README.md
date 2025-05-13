# Project Networks attacks

## Introduction

The goal of this project is to implement a series of network attacks and defense mechanisms against them. These attacks are deployed against a network topology being emulated with mininet. Before lauching any attack, the topology is  made more secure by implementing basic network protection firewall with nftables.

Assuming the mininet-vm is launched, you should copy these files inside the VM.

To initialize the network topology, run the following command:

`sudo -E python3 ~/LINFO2347/topo.py`


## Attacks

### Network Scan

Network scanning is often the first step in the reconnaissance phase of an attack to learn about the host and services running inside a network. In our Mininet “enterprise” topology, an attacker sitting on the “internet” host will execute a scan against the DMZ (10.12.0.0/24) to discover live hosts and open ports. To launch this attack, run the following commands:

`internet python3 Network_Scan_attack.py`	


### SSH Brute Force attack 

The SSH brute-force attack is launched once hosts with an open SSH port have been identified, and consists of systematically attempting to connect by trying a list of commonly used username/passwords that can be found on the file passwords.txt.

internet python3 attack2.py [host_ip] [User] [Password_file]

Ex:

internet python3 attack2.py 10.12.0.40 ftp_user passwords.txt


### ARP cache poisoning
 
The goal of the attack is to send forged ARP packets to a targeted host pretending to be another device on the network, the gateway for example. This is done by sending a forged ARP reply to the targeted host, mapping the MAC addressof the attacker to IP address of the gateway. This will redirect all traffic send by the victim to the attcker's computer thus allowing for a Man-in-the-Middle (MITM) attack. To launch the arp cache poisoning attack from `ws3` to target `ws2`, follow these steps:

`ws3 python3 arp_cache_poisoning.py `

To see the traffic being redirected by `ws3`, enable ip forwarding on the workstation using the command: 

`echo 1 > /proc/sys/net/ipv4/ip_forward`



### TCP SYN flooding

This attack aims at overwhelming the targeted host with a large number of SYN packets, it is also a way to perform a DoS on a service that uses TCP. The victim will have to keep half-open connections in his SYN and listen backlog due to the unfinished 3-way handshake. Once the backlog is full, this will make the victim not be able to respond to legitimate requests or slow to respond. The IP address of the source host in randomized in the subnet range of 10.2.0.0/24 to simulate multiple different hosts from the internet.

To launch this attack, run the following command.

ws2 python3 SYN_flooding_attack.py 10.12.0.10


### DNS Reflected DOS

This attack take advantage of the fact that a DNS answer any request and that the response is usually larger than the request. The attacker (by default launched from the internet) send DNS requests to the DNS server spoofing the IP address of the victim host on our network. We use the scapy library to craft DNS packets with the specified target IP address as the source IP. The DNS server send the response back to the victim, which receives a large amount of traffic causing it to  slow down or to become unavailable. To run this attack run the following command: 

ws2 python3 DNS_Reflected_DOS.py`



## Defense mechanisms


### Network Scan



### SSH Brute force attack



### ARP Cache poisoning defense

The protection mechanism implemented against ARP cache poisoning relies on maintaining a static IP-to-MAC address mapping for all hosts in the network. When an ARP reply is received, the system checks whether the provided IP-to-MAC mapping matches the one stored in the cache. If the mapping differs, the ARP reply is discarded, and the cache remains unchanged. While this approach can prevent unauthorized changes to the ARP table, it is not easily scalable, as maintaining accurate static mappings is challenging in real-world network environments.

To launch the protection, run the following command:

r1 python3 arp_cache_defense.py

### TCP SYN flooding defense

In the TCP SYN attack implemented, the IP address of the source device is randomized as in a real-life environment, this make it difficult to block incoming request from a specific host. Blocking the entire subnet range of the internet will also block legitimate traffic. For the defense mechanism, we made use of SYN cookies. This works by responding to incoming SYN requests with SYN-ACK packets that include an encoded cookie. Only when the client replies with a valid ACK does the server complete the TCP handshake and allocate resources for the connection. If the handshake isn’t completed, the server avoids holding half-open connections, thus limiting the effectiveness of the attack. While this doesn't defend against the threat completely, minimizes its impact. To enable this protection, execute the following command on the HTTP server.

To launch the defense mechanism, run the following dommand:

http python SYN_flooding_defense.py

To test it: 

time curl http://10.12.0.10


### DNS Reflection DOS Defense

Completely blocking this type of attack reveal to be challenging, so our approach is to slow down the attacker by rate-limiting DNS requests. This script enforces a limit of 3 DNS requests per second. To activate this defense mechanism, run the following command on the DNS server:

dns python3 DNS_Reflect_DOS.py
