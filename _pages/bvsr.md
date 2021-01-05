---
permalink: /portfolio/bvsr/
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
title: Blue VS Red Event
---

## Introduction

The Blue VS Red event was organized for the application made by the security engineers to be tested by the red teamers, while the blue teamers monitor the network.
During this event, all the students can test their skills and/or products in practice. The event is held three times so that the changes can become apparent in between the events.

## First Event

The first event was on 26 October 2020. For this event, we made a Discord server for all the red teamers. When we started we got a list of all applications to divide. Arjan, one of the red team students, dived the scope into the groups we had made. These groups then got to work on these given applications. The tests our group were pretty conclusive because we had found nothing in the three applications. It was clear that the security engineers had focussed on the login page and did not add any functionality. Due to this, there was nothing to find.

## Second Event

During the second event, the setup went about the same. The applications were distributed among new students and we got into testing. My application was a Secure Webcam app. This was a Java application running on a Windows VM. I tested the application on the VM and downloaded it to my machine to decompile the jar file. I did not find any real vulnerabilities in the application. However, I had some points on bad practice. For instance, the passwords were hashed client-side. This is not good practice, but there is no real vulnerability.
