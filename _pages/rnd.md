---
permalink: /rnd/
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
title: R&D Project
---

## Intro

Our R&D project for this semester is a research for the INTERSCT project ([source](https://www.nwo.nl/en/research-and-results/research-projects/i/00/33700.html)). The goal of the INTERSCT program is to develop better and mostly more secure IoT devices.

Our part in this project is to write guidelines on securing IoT devices. The contents of the guidelines is based on pentests on different devices. These guidelines will describe how to develop secure IoT, but also how to keep your IoT devices secure after purchasing them.

## Research

As briefly mentioned before, our research mainly consists out of looking into current IoT devices and pentesting them to find what the most common problems are. We started with pentesting the AirScrubber project, a small project developed by students. Here we found that this project has quite some problems. The two most serious ones where bad authorization and a DoS exploit. The problem with the authorization is that once anyone is logged in (has a valid session, checked by the JWT-token), the user can access any page, regardless whether they should be able to access it.

Now we are testing a board by CTouch and multiple smartwatches by different manufacturers. 

Furthermore, we did research for the existing guidelines in security and IoT, to get a good vision of what guidelines  look like and should look like.

My role in these research was helping looking for good guidelines and ordering them. I also helped with the code-review of the AirScrubber and I lead the pentest of the AirScrubber and the CTouch board. My only involvement into the smartwatches is occasionally helping the others.

## Methodical

I have worked on the pentest for both the airscrubber and the smart screen. In both of these pentests we used a methodical approach to make sure that we work properly together. For the airscrubber, we all started on the webpage, with different parts. for instance, I started with trying to inject payloads and then I looked into the authorization. Then we got into the other part then the webpage, the MQTT server. Were I worked with one other to build a script that can send different data to the system. There I found out that you can DOS it with a string.

The smart board needed a different approach, because it does not have a single interface. We started with running a full on port-scan. From there we devided the work in services, we started on ADB togethet and then I got into the updater and Marc into UPnP. From there we worked futher and switched the ports up, always communicating on what we are doing. 

I also have a page talking about my methodical approach of HTB: [Link](https://raw.githubusercontent.com/Riqky/riqky.github.io/portfolio/htb_method).

## Communication and feedback

Communication in our group started out a bit rough. Although most of us already knew each other, we suddenly had to do everything in English, which was a bit new. but, we got over it and we are going a lot better now. On Wednesdays we work from home, we meet at 9 o'clock on our own Discord server. There we have a stand-up and discuss what everyone is going to do that day. If you collaborate with someone you remain online, else you can leave the voice-chat, but you must remain reachable. In the afternoon around 4 o'clock we get together again to discuss our work and progress. On Thursdays we do the same, only physical with our teacher Ron. If someone cannot come online for any good reason (usually COVID), they can join our meetings online.

We've had two retrospectives this semester with a feedback moment for everyone. The feedback to me was that I bring a lot of expertise to the table due to my experience. What I could improve is allowing other students to join in on the pentest and understand what I've done during the tests. This is something I understand, as I am very quick to jump on the pentest and find stuff. I have worked on this problem, and it has become less bad. The stand-down and the smartwatches helped a lot with this.