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
title: Android reverse engineering
---

## Intro

Reversing an android app can be pretty useful, because doing vulnerability research is a lot easier with the source code. Even if you must reverse the source yourself. A lot of vulnerabilities in Android app also requires you to edit the code to circumvent front-end-only restrictions.

## Theoretical research

An `APK` file is a `ZIP` file with a certain structure. Every APK contains a `manifest.mf` for all the meta-data, a `lib` folder for all the platform-dependent code, a certificate, a `res` folder containing recourses as `XML` files, assets to use at runtime and `classes.dex`. The last one is the most interesting, because this is the java-code compiled to a dex file. Dex are run on `Dalvik` VM's. Dalvik itself is not used anymore, but its structure has not changed.

But, how do you read these dex files? First there is `apktool` ([link](https://ibotpeaches.github.io/Apktool/documentation/)), a tool that unpacks `.apk` files and also decodes compiled sources such as `XML` files. This gives you more access and insight into the resources. Then there is `dex2jar` ([link](https://github.com/pxb1988/dex2jar)), this tool converts the dex file into a jar-file (a normal java executable). A jar file can be read with `JD-GUI` ([link](http://java-decompiler.github.io/)). If you want you can save this code and recompile with android studio.

## Practical

First, I made a simple hello world app, that I could test the reversing with:

![hello](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/android/hello.gif)

This is all the app does, just a simple function. I compiled it to an APK, now let's pull it apart. First, I run `dex2jar`:

```bash
d2j-dex2jar.sh app-debug.apk
```

Then, we start the decompiler:

```bash
jd-gui
```

And we open the `app-debug.jar`. We are greeted by three folders, one of which looks familiar.

![libs](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/android/libs.png)

`com.example.hello_world` must be the code I've written. The other two folders are filled with obfuscated classes, probably part of the android library. My class looks a little bit different, this is the result of the optimation done by the java compiler.

```java
//original code:

package com.example.hello_world;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void buttonClick(View v)
    {
        TextView tv = findViewById(R.id.hello);
        if(tv.getText().equals("Hello World!")){
            tv.setText("Welcome to android");
        }else{
            tv.setText("Hello World!");
        }
    }
}

//VS the compiled and reversed code:

package com.example.hello_world;

import android.os.Bundle;
import android.view.View;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
  public void buttonClick(View paramView) {
    TextView textView = (TextView)findViewById(2131165276);
    if (textView.getText().equals("Hello World!")) {
      textView.setText("Welcome to android");
      return;
    }
    textView.setText("Hello World!");
  }
  
  protected void onCreate(Bundle paramBundle) {
    super.onCreate(paramBundle);
    setContentView(2131361820);
  }
}

```

A few things that changed:

- The object ids were changed to their corresponding number in the `enum`.
- Some variable names changes (`tv` to `textView`)
- A redundant cast was placed.
- `@override` was removed.
- All useless empty lines where removed.

This makes reversing a bigger app a lot more difficult because  mainly the variable names can tell a lot, without these the code becomes a lot less clear.

## Ionic

Alright, now we know the structure of a normal Java-based Android app, let's look at the other breed of android apps: ionic. Ionic is based on `nodejs` and `angular` and uses Cordova for the native class to android and ios. You build ionic in HTML and JS, so I wonder whether it compiles to java or not.

Alright, so the first steps are the same, `dex2jar` and `jd-gui`

![webview](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/android/webview.png)

Webview? As expected, it did not compile to a java app, this is only the wrapper that executes HTML and js files. So, let's try to find the js files.

After some `greb`-ing, I found the folder `assets/www` filled with JS-files and an `index.html`. Perfect, now we have to check whether or not these are the js-files that I wrote, so I searched for a literal string that I am sure is present in the source:

![js](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/android/js.png)

As you can see, the js is heavily obfuscated during the compiling, but it is still present. With this we could reverse the js and find out how the app works. 