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

UNFINISHED