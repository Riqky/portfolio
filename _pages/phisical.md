---
permalink: /portfolio/phisical/
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
title: Test apotheek
---

One of our assignments was to test the physical security of a public accessible building. I decided to test my local pharmacy. My mother works here, so I can walk around freely while making notes, without being suspicious. After I got permission from the manager of course.

## Layout

First, I mapped the entire building out with a simple map. I then placed all camera's, movement detectors, computers and doors. This gave me a great insight into the building:

![fireplane](/assets/images/pysik/fireplan.jpg)

![building](/assets/images/pysik/building.jpg)

As you can see, the building has a lot of movement detectors and few camera's in the key places. The detectors are wired to a central alarm system, but we'll get there later.

The front door is open as long as the pharmacy is opened, this door goes to the waiting room and the counter. The back door has a tag-lock and a door-bell, this door comes out in the backroom, from where you quickly enter the main work area. The furthest most back room is the canteen, there is an emergency door there, but it is inaccessible from the outside. However, if this door can be accessed, anyone could shut down the power from the building, as this is right next to the door without any alarm.

## Network

A pharmacy is and has a lot of critical infrastructure and lots of personal information goes over their network, so this needs to be secure. They have a WIFI-network, but this is separated from the local network with all the work-systems. The local network contains about 22 pc's all running Windows 10. Most of these pc's are inaccessible, because there is always an employee around, or it is hard to access them without passing an employee. However, there are consultation rooms, these, especially the left most, are easily accessible, without any lock. Once in there you have unlimited access to an unlocked pc. This is a huge risk, since an attacker only needs a few moments alone to gain complete access to a pc and they can have access to the local network. The software they use is locked by a password, unique for every employee. However, the pc also needs a password, because an attacker could listen on the network, our exploit a badly patched system.


![room](/assets/images/pysik/room.jpg)
![room](/assets/images/pysik/pc.jpg)

## Alarm

The alarm system is pretty secure and contained. Every employee has their personal tag with assigned work hours. If an employee comes in after hours without permission, the alarm goes of. The alarm can only be armed if all doors and windows are closed and it goes off if any of them open. All three outside doors are connected to the alarm and the alarm has its own backup power, in case of a power outage. Almost all of the rooms contain a motion detector and there are four camera's in the main area, filming the important parts. Breaking and entering in this building is pretty much impossible.

## Other problems

### Patch closet

One problem I found is that the patch-closet is unlocked.

![patchkast](/assets/images/pysik/patchkast.jpg)

This poses quite the risk, since this gives access to the entire network. However, this closet is located in the stockroom, so it is not easily accessible. Since this closet in not required to be accessed regularly, it should be locked and its keys should be stored in a safe place.

### Social engineering

Can you talk your way in? In short, no. This is a very small company where everybody knows everybody. So if you walk in, you will be spotted as out of place. Any mechanic gets announced by their company and there is very little other reason to go in there.
