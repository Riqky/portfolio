---
permalink: /portfolio/method/
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
title: Methodical skills
---

## Any given subject

The idea of a methodical approach is that one takes the same or similar steps, regardless of the subject of the faced problem. That is why I will explain my way of tackling the research for every subject in my portfolio.

For many subjects, I had pre-existing knowledge. Like reverse engineering, which I have worked with a few times before this semester. But there were also some instances where I had no prior knowledge, such as the covert channels. For a subject like that, I started by figuring out what I was dealing with. On the DOT-framework this would be placed in Literature study since it is an online study for the definition of the subject. After that, I continued my literature study, but more focussed on my practical work. So I'd look into what I would like to do in practice, what is possible within the given timeframe and what other students have done. From there I made a plan in my head for what I am going to do. Sometimes I wrote the theory in my portfolio before working on the practical part, but I often just wrote everything last, with all the knowledge I had gathered.

Once I am done with the literature study, I can start on the practical aspect. This was done in Lab and Workshop of the DOT-framework. In the library study I figured out what I want to do, now I have to figure out how to do it. For instance, for the research of covert channels, I found a lot of ways to implement a ping-tunnel using programs such as `ptunnel`. But I wanted to do it manually, after some googling I found a script in Python that does exactly what I wanted. I then spend some time debugging the script, since it didn't work immediately. Then I used `ptunnel` to prove that there are programs that can do these things a lot better than I. This way I learned how a ping tunnel worked, by reading the code I found, and tweaking it until it worked. If I had just used a program, I would never have fully understood how the protocol works and can be abused.

After all of this, I always wrote the page for my portfolio. I often tried to do this immediately after the practice, so my memory is still fresh. Writing the explanation often also required some research, to make sure everything I say is correct and going back to my workspace to take some screenshots.

## Hack The Box

After using Hack The Box for about a year, and getting 54 system own's. I have developed a good approach for taking the machines. I always start with some setup. I have a folder on my Kali machine named HTB. Here stuff like the VPN-pack is stored. For every machine I have a dedicated folder, to store all the information and exploits. Is this folder I also save a file for CherryTree. CherryTree is a node-based text editor I use to write down al my findings. In there I store the `nmap` results, users and passwords, possible exploits and commands I'm tweaking to get working. This structure helps me keeping track of everything I've found and for when I need to help someone with the box after I finished it.

The first step on every box is to scan the open ports. I used to only scan the default ports, but after missing an important high port on a box, I now always scan all ports with `nmap -p- <IP>`. After that, I scan the open port with default scripts and version enumeration, `nmap -sC -sV -oN nmap.txt -p <PORTS> <IP>`. I also save the output to a text file and I only scan the ports that have been found in the previous scan. While this is running I start checking out the open ports. I do a lot of Linux machines and these often run a website that I visit first. A website can often have various attack surfaces. If it is running a web application, I often look for as much information as possible. Version numbers, names and vendors. This will help me finding an exploit. Then to find a possible exploit, I use google and `exploitdb`, or `searchsploit` on Kali. Before running an exploit, I always read over the code, to make sure it is not a dud. If the website is custom made or it has not known exploits, enumerating the surface is always a good idea. I use `gobuster` to find all the webpages on the server. With this information, I can often find some information that can be used to find more functions that can be exploited, think of image uploads or various injections in input, or find some information that was "supposed" to be hidden. If the webserver is running an HTTPS-port, check the certificate, this often hides more information, such as a hidden virtual host.

If the exploit is not on a website, it is often just finding information about the running program and exploit it with a known vulnerability, of course, there are exceptions to everything, I have written a custom buffer overflow for a box. On Windows boxes you often have to gain information through SMB-shares, although I have very little experience with these.

Once you have a successful exploit, after either writing, copying or copying and editing, you gain a reverse shell (on Linux machines at least). Sometimes you've found some credentials and logged into the box with SSH, but the process is still the same. Now we have to escalate from the current user to root, sometimes even from `www-data` to a normal user first. One of my favourite programs for this is [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS), it is a script that scans the machine for interesting things to abuse to gain root access. With HTB it is often that anything that stands out of the ordinary is the way to go. I also often quickly check `sudo -l`, this tells you what you can run as root. There are many ways to escalate, but I often check certain folders for interesting files, check processes and default Linux functions, such as cronjobs.

I try to add everything I do into CherryTree, but I often forget to document the escalation, because I am to occupied with finding it that I forget to write down what I have done.

## R&D

I have worked on the pentest for both the Airscrubber and the smart screen. In both of these pentests, we used a methodical approach to make sure that we work properly together. For the Airscrubber, we all started on the webpage, with different parts. for instance, I started with trying to inject payloads and then I looked into the authorization. Then we got into the other part then the webpage, the MQTT server. Where I worked with one other to build a script that can send different data to the system. There I found out that you can DOS it with a string.

The smartboard needed a different approach because it does not have a single interface. We started by running a full-on port-scan. From there we divided the work in services, we started on ADB together and then I got into the updater and Marc into UPnP. From there we worked further and switched the ports up, always communicating on what we are doing.
