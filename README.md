# Project Networks attacks

## Attacks

### ARP cache poisoning
 
The goal of the attack is to send forged ARP packets to a targeted host pretending to be another device on the network, the gateway for example. This is done by sending a forged ARP reply to the targeted host, mapping the MAC addressof the attacker to IP address of the gateway. This will redirect all traffic send by the victim to the attcker's computer thus allowing for a Man-in-the-Middle (MITM) attack. To launch the arp cache poisoning attack from `ws3` to target `ws2`, follow these steps:

1. Open `ws3` terminal window using `xterm ws3`
2. Run the command `python3 arp_cache_poisoning.py `
3. Once the attack is launched, traffic from and destined for `ws2` will first go through `ws3`.

To see the traffic being redirected by `ws3`, enable ip forwarding on the workstation using the command: 

`echo 1 > /proc/sys/net/ipv4/ip_forward`




### SYN flooding


ws2 python3 SYN_flooding_attack.py 10.12.0.10


### Reflected DNS DOS

This attack take advantage of the fact that a DNS answer is larger han the request. The attacker (by default launched from the internet) send DNS requests to the DNS server spoofing the IP address of the victim. We use the scapy library to craft DNS packets with the specified target IP address as the source IP. The DNS server send the response back to the victim, which receives a large amount of traffic causing it to be unavailable. To run this attack run the following commands: 

1. Open the terminal window of the internet node: `xterm ws2`
2. Run: `python3 DNS_Reflected_DOS.py`



### TCP Scan




## Defense mechanisms

### ARP Cache poisoning defense




### SYN flooding defense

To prevent SYN flooding attacks, we make use of SYN proxy/ cookies.

First hardening of the sserver

1. Using synproxy requires disabling the conntrack loose tracking option:

% echo 0 > /proc/sys/net/netfilter/nf_conntrack_tcp_loose

2. Also, because synproxy relies on syncookies and tcp timestamps, ensure these are enabled:

% echo 1 > /proc/sys/net/ipv4/tcp_syncookies
% echo 1 > /proc/sys/net/ipv4/tcp_timestamps

3. Increase SYN backlog; max half open SYN connections before new ones are dropped.This give the server more buffer against bursty SYN packets.

Reduce  how long the server deals with potentially bogus requests:

4. SYN-ACK retries is set to default 5, number of time the kernel retransmit SYN-ACK if no ACK is received from the client.

5. SYN retries, default is 6 here is 1. Not really iportant here? 

Next configure firewall rules to :

6. SYNPROXY Rule intercepts and completes TCP handshakes for port 80, allowing only legitimate connections through.

7. Drop Rule discards invalid packets early to protect against malformed traffic and reduce load.

To launch it:

http python SYN_flooding_defense.py

To test it: 

time curl http://10.12.0.10



### TCP Scan



### DNS DOS Defense

Completely blocking this type of attack reveal to be challenging, so our approach is to slow down the attacker by rate-limiting DNS requests. This script enforces a limit of 3 DNS requests per second. To activate this defense mechanism, run the following command directly on the DNS server:

python3 DNS_Reflect_DOS.py
