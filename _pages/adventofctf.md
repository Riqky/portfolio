## Day 0

*Do you remember the flag in the teaser website?*

The CTF is kicked of with a callback to a teaser website. I have never looked at this website, but this can be fixed by using wayback machine. There we see that there was a snapshot made on the 12th of November. This sends you to a beautiful website with a banner teaser, and a flag:

[day0banner](/assets/images/advent/day0banner.png)

After decryption `NOVI{HEY_1S_Th1S_@_Fla9?}` comes out.

## Day 1

On day two you are presented with a Login page, where you have to enter Santa's password. A quick peak at the HTML reveals this:

```html
<!-- This is an odd encoded thing right? YWR2ZW50X29mX2N0Zl9pc19oZXJl -->
```

No, it is not. Decoding it with base64 reveals `advent_of_ctf_is_here`. And after entering it we get the flag `NOVI{L3T_7H3_M0NTH_0F_FUN_START}`!

## Day 2

Today, we are greeted by a login page. After an attempt at logging in, `admin-admin` seems correct. At a further inspection, it turns out that it does not matter what you put in. The second page reads `I'm sorry Guest, I can not do that.`, a nice reference, but no flag. After looking around I found a cookie, that seems base64 encoded: `authenticated:eyJndWVzdCI6InRydWUiLCJhZG1pbiI6ImZhbHNlIn0=`. Decoding shows `{"guest":"true","admin":"false"}`. Well, what if we change from guest to admin, by setting guest to false and admin to true? We get a flag of course! The resulting cookie becomes `eyJndWVzdCI6ImZhbHNlIiwiYWRtaW4iOiJ0cnVlIn0=` and after refreshing the flag is: `NOVI{cookies_are_bad_for_auth}`

## Day 3

We find a login page again. But this time it does not accept random input. The form executes `checkPass();` on submit, this method is quickly found in `login.js`.

[day3login](/assets/images/advent/day3login.png)

So, the password is the username plus '`-NOVI`', base64 encoded. Alright, that gives me admin and YWRtaW4tTk9WSQ== as credentials. Logging in gives us `NOVI{javascript_is_not_s@fe}`.

## Day 4

Today, the page just says `Hello, User 0`. After a short period a token is added in the URL: `eyJ1c2VyaWQiOjB9.1074`. It appears to be once again a base64 token, but with something else. The first part decodes to `{"userid":0}"`. Then I found the javascript, and oh boy.

![day4js](assets/images/advent/day4js.png)

The javascript is obfuscated to make it very difficult to read. But, it seems that the important method is readable. `check()` gets a key out of the local storage, checks if it is not in the URL and then validates the checksum in the key against a checksum calculated by `calculate(text)`. So, if we want to fake the key, we also need to get its checksum. I first tried to reverse the code, in order to understand how to bypass the check, until I realised that I can just run it locally. So, that is what I did, assuming that I need to become user 1.

![day4result](assets/images/advent/day4result.png)

Then we set `eyJ1c2VyaWQiOjF9.1075` as a token in our local storage and refresh the page without the GET parameters. This gives us the flag: `NOVI{0bfusc@t3_all_U_w@n7}`.

## Day 5

Yet again, we have a login form. But this time, the HTML and JS deliver nothing. Maybe it is the other biggest exploit in simple challenges? SQL injection. If we inject a quote into the username box, we get an error:

```text
Error description: You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'password'' at line 1
```

So, if we guess the username and then end the query, we are in. How about `admin`? We enter `admin' #` into the username and something into the password field. This gives us the flag: `NOVI{th3_classics_with_a_7wis7}`.

## Day 6

Today we have a search form to search the naughty list. Well, I guess yet another SQL Injection. If we type `' OR 1=1 #` into the search bar, we see all the results, but limited to only the five first characters. But there also an id column, what if we can get the values into that column?

We can do this using union. If we input `' UNION SELECT 1,2,3#` we see 1,2 and 3 in the results. We can abuse this to give other results. Let's gather more information about the database. With `' UNION SELECT DATABASE(),2,3#` we can see that the database is named `testdb`. Then with `' UNION SELECT table_name, 2, 3 FROM information_schema.tables WHERE table_schema='testdb'#` We can see the tables. Flags and secrets. Great, let's get everything from flags. With `' UNION SELECT *, 2, 3 FROM flags#` we dump everything from flags and see the flag! `NOVI{7h1s_flag_w@s_chuncky_right}`

## Day 7

This day we have a naughty list again. If we enter something invalid the response is `Why?`. If we enter a quote, we get no response, so it must be a blind SQL Injection. Well, what if we enter `' OR 1=1 #`. And that gives us the flag... Wait, that was not supposed to happen! After entering the flag, I get an extra challenge to find the username, to prove that I can execute a blind SQL Injection. I reused the payloads from the previous day, so let's make it quick. 

```text
' UNION SELECT 1 #
1

' UNION SELECT DATABASE() #
testdb

' UNION SELECT table_name FROM information_schema.tables WHERE table_schema='testdb'#
naughty


' UNION SELECT column_name FROM information_schema.columns WHERE table_name='naughty' #
id
username
badthing 

' UNION SELECT username FROM naughty #
egische

' UNION SELECT badthing FROM naughty #
NOVI{bl1nd_sql1_is_naughty} 
```

And there we have both the flag and the username.

## Day 8

*Did you know that the fastest robot can solve a rubiks cube in 0.887 seconds?* We are greeted by this text on the main page. The HTML and JS brings me no further. What about other pages? Going to `/robots.txt` gives us more information:

```txt
# robots.txt generated by smallseotools.com
User-agent: *
Disallow: /
Disallow: /cgi-bin/

Disallow: /encryption/is/a/right
Disallow: /fnagn/unf/znal/cynprf/gb/tb
```

Two very interesting entries. The first one gives us a base64 string: `RW5jb2RpbmcgYW5kIGVuY3J5cHRpb24gYXJlIDIgZGlmZmVyZW50IHRoaW5ncy4=` which decodes to `Encoding and encryption are 2 different things.`. So we have to do something with encryption. On the second page we get a poem by Dr Suess: `"Oh, the places you'll go", my favorite poem... but this is the wrong place. Maybe you read that wrong?`. We are wrong again. But looking at the path of the second page with encryption in mind, this might just be `rot13`. And yes, the second page decrypts to `/santa/has/many/places/to/go` in rot13. And this place gives us the flag: `NOVI{you_have_br@1ns_in_your_head}`.

## Day 9

What a surprise, a login page. This time, regardless of what we enter, we are sent to a page with `Hey user your password is incorrect.`. So, something really stupid, but what if we log in with `user` and `incorrect`? Then we get to see `The naughty list is currently empty....`. This helps us a little further. Then looking into the storage we find a JWT Token:

![day9jwt](assets/images/advent/day9jwt.png)

What if we edit the payload to make ourselves admin, and edit the header to set the encryption to none?

![day9edited](assets/images/advent/day9edited.png)

And there we have a flag! `NOVI{Jw7_f@ilure_in_n0ne}`

## Day 10

This time we have a page that says that there is only one person on the naughty list. Once again there is something interesting in the storage: `eyJwYWdlIjoibWFpbiIsInJvbGUiOiIxMmRlYTk2ZmVjMjA1OTM1NjZhYjc1NjkyYzk5NDk1OTY4MzNhZGM5In0=`, which decrypts to `{"page":"main","role":"12dea96fec20593566ab75692c9949596833adc9"}`. Let's see of we can find another page, by changing `"page":"main"` to `"page":"naughty"`. This gives me a error:

![day10error](assets/images/advent/day10error.png)

So, PHP uses includes on the user input. We *could* exploit this with a reverse shell, but I think this is a bit overkill. Let's just try some other payloads, such as `"page":"flag"`.

![day10step](assets/images/advent/day10step.png)

Well, that is great, but we need to get promoted. The second part of the cookie appears to be a hash, a simple online look-up table tells us that this is `user` hashed with sha1. so, what if we hash `admin` with sha1? We end up with the payload `'{"page":"flag","role":"d033e22ae348aeb5660fc2140aec35850c4da997"}'` encoded to `eyJwYWdlIjoiZmxhZyIsInJvbGUiOiJkMDMzZTIyYWUzNDhhZWI1NjYwZmMyMTQwYWVjMzU4NTBjNGRhOTk3In0K`. This gives us the flag `NOVI{LFI_1s_ask1ng_f0r_tr0bl3}`.