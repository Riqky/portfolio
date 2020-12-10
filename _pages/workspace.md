---
permalink: /portfolio/workspace/
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
title: Workspace
---

## Linux

I spend quite some time in my laptop, because I am running Linux mint 20. I quit Windows last year, after it messed up my disk encryption I was so fed up, I installed Linux mint. I was already using Linux a lot during my internship, so it wasn't that big of a step. Nor have I regretted it ever since. There where a few set-backs along the way, but I will describe my journey here.

### Distribution

The first problem with installing Linux is choosing a distribution. I wanted something beginner friendly and commonly used. This brought me to the most used distro: Ubuntu. This is a great and simple distro, Debian based which is great, because most of my experience with Linux came from Kali (also Debian based). Then a co-worker at HackDefense recommended Linux Mint. Mint Cinnamon to be exact, has an interface that is more like Windows', which I really like. Mint is based on Ubuntu, so it is very similar. Some parts are different, but almost all troubleshooting for Ubuntu works. I choose for mint, after all, I had nothing to loose. It was a great decision, that I never regretted. Cinnamon is the desktop environment for Mint, this means that it handles almost all of the visual aspects of Mint. There are other variants of Mint, but Cinnamon is the most popular and just very good.

### Ricing

Ricing is the art of 'pimping' your Linux Distribution. The term came from people who bought cheap Asian sport-cars and made then look faster then they actually were. Ricing takes a lot of work and time, so I haven't really done a full rice, but I am happy with the results.

I wanted everything to match my background picture, so I started by finding a cool background. This will do great:

![background](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/workspace/background.jpg)

Then, I look for a program that could get the colourscheme of my background. for this I found pywal([https://github.com/dylanaraps/pywal](source)). Pywal is a simple program that exports a bash colourscheme based on your desktop background.

![bash](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/workspace/bash.png)

Then I got to the system itself. I found that there is a css file that colours the system. I decided to edit this file in oder to change the look of everything. It took me a while to understand the undocumented file, but the result was very nice. Unfortunately, I lost this file, so I cannot show my results.

### ZSH

Linux' bash is great, but it can be better, that's why I installed ZSH with Oh-My-ZSH ([https://github.com/ohmyzsh/ohmyzsh](source)) and the powerlevel10k theme. This, combined with multiple plugins gives me a fast and easy shell, allowing me to do tasks very quickly.

![zsh](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/workspace/zsh.png)

I have the following pluings installed in ZSH:

* **safe-paste**: This simple plugin makes sure that your command does not immediately execute after pasting it in the terminal, even if there is a newline at the end, so you can edit the command before executing it.
* **docker** This enables auto-complete for docker commands, last semester I used a lot of docker.
* **common-aliases** Common aliases for your shell, this plugin gives you some extra shortcuts when using the terminal.
* **debian** This gives you extra aliases and autocompletion for the `apt` environment, making installing programs a bit easier.
* **pip** A small plugin that gives you more autocompletion for python's pip tool.
* **sudo** Simple, bit I use it a lot. When you press escape twice, `sudo` is added before the current command. If there is no command, it will copy the previous command, but with sudo in front of it. This is really useful when you forget that a command requires root
* **zsh-syntax-highlighting** Add some syntax highlight, to make it easier to understand and review your commands.
* **zsh-autosuggestions** Suggests commands that you typed earlier, this allows you to re-execute a command without having to do a reverse search or re-type it.

## VMware

For my tools and machines I use VMware workstation 15 Pro. This is in my opinion the best client for virtual machines. I like to run the machines locally, since I have a good laptop that can run the Vm's and schools seclab tends to be slow and have some problems.

### Kali

My most used VM must be the Kali-Linux VM. This is an install of the latest version of Kali (2020.2) with `Xfce` as display manager. I've made very little changes to my Kali install, because I often install a new version from scratch when an update comes out. A few small changes are the things like setting the panel at the bottom and setting a few shortcuts on the panel. Another change is that I install `gobuster`, a faster alternative to `dirbuster` or `dirb`. As soon as these things are set, I make a snapshot and I am done.

### Reverseing

For the reverse engineering challenges, I started with my Kali, but this proved to be somewhat unconventional, so I installed `Remnux`. This is a Ubuntu machine with all Linux based reversing tools installed. Such as Ghidra, Cutter and various checkers. For windows based binaries I have a Windows 10 machine with ollydbg installed.

### Windows 10

I have only one windows 10 machine, on here I test payloads, malware and other windows based tests. This is pretty sloppy, since the malware, tools and payloads might collide with each other. But so far I'm good. When the time comes I will install a new Windows 10, or use snapshots the switch between the states.
