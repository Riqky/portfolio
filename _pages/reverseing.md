---
permalink: /portfolio/challenges/
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
title: HTB Challenges
---

## Reversing

### Baby RE

This is the easiest reversing challenge HTB so let's take a look at it.
After downloading I use file to find out what kind of file it is.

```bash
$ file baby
baby: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=25adc53b89f781335a27bf1b81f5c4cb74581022, for GNU/Linux 3.2.0, not stripped
```

Okay, so it is a linux executable, let's just run it:

```bash
$ ./baby
Insert key:
asd
Try again later.
```

So, the program wants a key. We can find this key with multiple methods, we'll look at the most easy one first. `strings` is a simple command-line program that gets all the strings inside a file, even an executable. 

```bash
strings baby
```

Now, strings always outputs a lot, because there are always series of bytes that look like a string inside a executable, but this is the part that seems the most interesting:

```text
Dont run `strings` on this challenge, that is not the way!!!!
Insert key:
abcde122313
Try again later.
```

That looks like our solution, but apparently we are not allowed to use strings, so let's employ another method: static analysis.
My tool of choice for this is Ghidra, an amazing and free static analysis tool. When you open the file and run all the scan, you are greeted by this screen:

![ghidra](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/ghidra.png)

We need to find where the interesting part is, because an executable is filled with a lot of stuff to make it run, that is not interesting to us. We can use two methods for finding the correct function. First there is strings again:

![strings](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/strings.png)

On the left is the window with all of the strings from the application, on the right is the call that is made to the memory-adress where this string is stored, so we find where it is used, and thus the function we where looking for.
The second method is using ghidras functions view. This allows us to look through all the functions of the appilcation.

![functions](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/functions.png)

Now, where would the juicy part be? In main() ofcourse! This is where the execution of the code starts, so here we need to look. This executable has now other functions, but if there where, they would be named `function_ab123` where the `ab123` is mostly random.

Now we have the location of the function, let's take a look:

![main](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/main.png)

On the left you can see the assembly, and on the right is the c-psuedo code generated from the assembly. As you can see, the program reads the `stdin` for your password, then checks it with the hardcoded variant and if it is the same, you will get the flag, which is obfuscated by storing it as bytes. But, now we have the password: `abcde122313`. Let's get that flag.

![flag](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/flag.png)


### Impossible Password

This challenge is similar to baby RE, but a bit more difficult, let's immediately jump into ghidra.

After checking strings, I found a "secret" password. This is not the main function, but the first interesting. This is an important step in reverse engineering, never look for more then you need, it is difficult enough. I then tried to understand this function, by renaming the variables. This revealed that the program checks for a first password, but then asks for a second, build in a different function:

![first_funtion](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/first_funtion.png)

In the second, it builds a second password, but it quickly becomes evident that it is impossible to guess this password, because it uses the epoch timestamp and `random` to generate it. The third function in the "main" functions appears to generate the flag, let's see if we can circumvent the second check.

![second_function](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/second_function.png)

For this, I am going to use `cutter` a FOSS tool, much like IDA Pro, with a debugger, and reverser. Cutter also has a great graph function, which allows you to visualize the jumps the program makes. With this information I went looking for the check if the second password is correct, which I found in the disassembler, and opened in the graph:

![graph](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/graph.png)

Then I used the reverse jump method, this reverses the jump call allowing you the execute the last function when the answer is wrong.
And there we have a flag!

![flag](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/reverse/flag_im.png)
